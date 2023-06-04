```java
package cn.edu.buaa.data.cache.lettuce;


import io.lettuce.core.*;
import io.lettuce.core.cluster.ClusterClientOptions;
import io.lettuce.core.cluster.ClusterTopologyRefreshOptions;
import io.lettuce.core.cluster.RedisClusterClient;
import io.lettuce.core.cluster.api.StatefulRedisClusterConnection;
import io.lettuce.core.cluster.api.async.RedisAdvancedClusterAsyncCommands;
import io.lettuce.core.cluster.api.sync.RedisAdvancedClusterCommands;
import io.lettuce.core.resource.DefaultClientResources;
import io.lettuce.core.resource.Delay;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.annotation.Resource;
import java.time.Duration;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@Service
public class LettuceRedisCache implements IRedisCluster {

    private static final Logger LOGGER = LoggerFactory.getLogger(LettuceRedisCache.class);

    private static final int BATCH_SIZE = 500;

    @Value("${redis.cluster.host.port}")
    private String hostAndPort;

    /**
     * 重定向次数配置
     */
    @Value("${redis.cluster.max.redirects}")
    private int maxRedirects;

    /**
     * 命令执行超时时间配置
     */
    @Value("${redis.cluster.command.timeout}")
    private int commandTimeoutMillions;

    /**
     * 连接超时时间配置
     */
    @Value("${redis.cluster.connect.timeout}")
    private int connectTimeoutMillions;

    /**
     * 重连等待时间
     */
    @Value("${redis.cluster.reconnect.delay}")
    private int reconnectDelayMillions;

    /**
     * 刷新拓扑间隔时间
     */
    @Value("${redis.cluster.refresh.duration.seconds}")
    private int refreshDurationSeconds;

    /**
     * 连接重连触发拓扑刷新的次数
     */
    @Value("${redis.cluster.refresh.attempts}")
    private int refreshTriggersAttempts;

    /**
     * 默认过期时间
     */
    @Value("${redis.cluster.default.expire}")
    private int defaultExpire;

    private DefaultClientResources clientResources;

    private RedisClusterClient redisClusterClient;

    private StatefulRedisClusterConnection<String, String> connection;

    private RedisAdvancedClusterCommands<String, String> sync;

    private RedisAdvancedClusterAsyncCommands<String, String> async;

    @Resource
    private ISerialization serialization;

    @PostConstruct
    public void init() {
        //  Build Redis URI
        if (StringUtils.isBlank(hostAndPort)) {
            throw new IllegalArgumentException("Redis Cluster Host & Port Empty!");
        }
        List<RedisURI> redisURIList = Stream.of(hostAndPort.split(",")).map(item -> item.split(":")).map(item -> RedisURI.builder().withHost(item[0]).withPort(Integer.parseInt(item[1])).withTimeout(Duration.ofMillis(connectTimeoutMillions)).build()).collect(Collectors.toList());

        //  Customize Client Resource
        clientResources = DefaultClientResources.builder().reconnectDelay(Delay.constant(Duration.ofMillis(reconnectDelayMillions))).build();

        //  Customize Refresh Options
        ClusterTopologyRefreshOptions topologyRefreshOptions = ClusterTopologyRefreshOptions.builder().enablePeriodicRefresh(Duration.ofSeconds(refreshDurationSeconds)).enableAllAdaptiveRefreshTriggers().enablePeriodicRefresh().refreshTriggersReconnectAttempts(refreshTriggersAttempts).build();

        SocketOptions socketOptions = SocketOptions.builder().connectTimeout(Duration.ofMillis(connectTimeoutMillions)).tcpNoDelay(true).build();

        //  Customize Timeout Options
        TimeoutOptions timeoutOptions = TimeoutOptions.builder().fixedTimeout(Duration.ofMillis(commandTimeoutMillions)).timeoutCommands().build();

        //  Customize Client Options
        ClusterClientOptions clusterClientOptions = ClusterClientOptions.builder().topologyRefreshOptions(topologyRefreshOptions).timeoutOptions(timeoutOptions).maxRedirects(maxRedirects)
                //  断开连接时，将拒绝所有请求
                .disconnectedBehavior(ClientOptions.DisconnectedBehavior.REJECT_COMMANDS).autoReconnect(true)
                //  重连失败时，将忽略之前堆积的请求
                .cancelCommandsOnReconnectFailure(true).socketOptions(socketOptions).build();

        //  Init Lettuce Client
        redisClusterClient = RedisClusterClient.create(clientResources, redisURIList);
        redisClusterClient.setOptions(clusterClientOptions);

        //  Init Connection
        connection = redisClusterClient.connect();

        //  Get Sync/Async API
        sync = connection.sync();
        async = connection.async();
    }

    @PreDestroy
    public void close() {
        connection.close();
        redisClusterClient.shutdown();
        clientResources.shutdown();
    }

    //redis del
    @Override
    public void del(String key) {
        try {
            sync.del(key);
        } catch (Exception e) {
            LOGGER.error("del key value from cache error,key:{}", key, e);
        }
    }

    //redis expire
    public void expire(String key, long timeout) {
        if (StringUtils.isBlank(key)) {
            return;
        }
        try {
            sync.expire(key, timeout);
        } catch (Exception e) {
            LOGGER.error("set key expire time error,key:{}", key, e);
        }
    }

    //redis set
    @Override
    public void set(String key, String objs, long timeout) {
        if (StringUtils.isBlank(key)) {
            return;
        }
        if (objs != null) {
            try {
                sync.setex(key, timeout, objs);
            } catch (Exception e) {
                LOGGER.error("Save cache error. cacheKey:{}", key, e);
            }
        }
    }

    //redis setnx
    public boolean setnx(String key, String objs) {
        boolean setnx = false;
        if (null == objs) {
            return setnx;
        }

        try {
            setnx = sync.setnx(key, objs);
        } catch (Exception e) {
            LOGGER.error("Setnx cache error. cacheKey:{}", key, e);
        }
        return setnx;
    }

    //redis get
    @Override
    public String get(String key) {
        String result = null;
        try {
            result = sync.get(key);
        } catch (Exception e) {
            LOGGER.error("get value from cache error,key:{}", key, e);
        }
        return result;
    }

    //redis sadd
    @Override
    public void sadd(String key, List<String> values, long timeout) {
        for (String value : values) {
            if (null == value) {
                continue;
            }
            try {
                sync.sadd(key, value);
                long expireTime = 0 == timeout ? defaultExpire : timeout;
                sync.expire(key, expireTime);
            } catch (Exception e) {
                LOGGER.error("Save cache error. cacheKey:{}", String.valueOf(key), e);
            }
        }
    }

    //redis smembers
    @Override
    public Set<String> smembers(String key) {
        Set<String> result = new HashSet<>();
        if (StringUtils.isBlank(key)) {
            return Collections.emptySet();
        }
        try {
            RedisFuture<Set<String>> setRedisFuture = async.smembers(key);
            Set<String> resultSet = setRedisFuture.get();
            result.addAll(resultSet);
        } catch (Exception e) {
            LOGGER.error("smembers cache error. cacheKey:{}", key, e);
        }
        return result;
    }

    public void srem(String key, List<String> values) {

        for (String value : values) {
            if (null == value) {
                continue;
            }
            try {
                sync.srem(key, value);
            } catch (Exception e) {
                LOGGER.error("srem cache error. cacheKey:{}", key, e);
            }
        }
    }

    //redis hset
    @Override
    public void hset(String key, String field, String value) {
        boolean success = false;
        try {
            sync.hset(key, field, value);
        } catch (Exception e) {
            LOGGER.error("");
        }
    }

    //redis hmset
    @Override
    public void hmset(String key, Map<String, String> map) {
        try {
            sync.hmset(key, map);
        } catch (Exception e) {
            LOGGER.error("hmset cache error. cacheKey:{}", key, e);
        }
    }

    //redis hget
    @Override
    public String hget(String key, String field) {
        if (StringUtils.isBlank(key) || StringUtils.isBlank(field)) {
            return null;
        }
        String result = null;
        try {
            result = sync.hget(key, field);
        } catch (Exception e) {
            LOGGER.error("hget cache error. cacheKey:{},field:{}", key, field, e);
        }
        return result;
    }

    @Override
    public List<KeyValue<String, String>> hmget(String key, String... fields) {
        try {
            return sync.hmget(key, fields);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public Long hincrBy(String key, String field, long value) {
        if (null == key || StringUtils.isEmpty(field)) {
            return;
        }
        try {
            sync.hincrby(key, field, value);
        } catch (Exception e) {
            LOGGER.error("hincrBy cache error. cacheKey:{},field:{}", key, field, e);
        }
        return  null;
    }
}

```
