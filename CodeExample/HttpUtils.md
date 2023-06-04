
```
package cn.edu.buaa.utils;

import org.apache.commons.collections4.MapUtils;
import org.apache.commons.lang.StringUtils;
import org.apache.http.HttpEntity;
import org.apache.http.HttpStatus;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.config.SocketConfig;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.DefaultHttpRequestRetryHandler;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.util.EntityUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.URI;
import java.net.URISyntaxException;
import java.nio.charset.Charset;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.TimeUnit;

public class HttpUtil {

    private static final Logger logger = LoggerFactory.getLogger(HttpUtil.class);
    /**
     * 默认连接超时时间
     */
    private static final int DEFAULT_CONNECTION_TIMEOUT = 5000;

    /**
     * 默认连接读取超时时间
     */
    private static final int DEFAULT_READ_TIMEOUT = 1000;

    private static final String DEFAULT_CONFIG_FILE = "conf/api.properties";
    //使用多例
    private static Map<String, CloseableHttpClient> httpClientMap = new HashMap<>();

    private HttpUtil() {
        // hide public constructor
    }


    //获取url的HostAndPort的httpClient
    private static CloseableHttpClient getHttpClient(String url, RequestConfig requestConfig) {
        String hostAndPort = getHostAndPort(url);
        CloseableHttpClient httpClient = httpClientMap.get(hostAndPort);
        if (null != httpClient) {
            return httpClient;
        }

        synchronized (HttpUtil.class) {
            //再获取一次
            httpClient = httpClientMap.get(hostAndPort);
            if (null != httpClient) {
                return httpClient;
            }
            //获取properties
            Properties properties = loadProperties();
            int maxTotal = Integer.parseInt(properties.getProperty("httpclient.pooling.maxTotal", "400"));
            int defaultMaxPerRoute = Integer.parseInt(properties.getProperty("httpclient.pooling.maxPerRoute", "200"));
            int defaultSoTimeout = Integer.parseInt(properties.getProperty("httpclient.pooling.soTimeout", "3000"));//3s
            int defaultTimeToLive = Integer.parseInt(properties.getProperty("httpclient.pooling.timeToLive", "1800"));//1800s=30m
            int defaultRetry = Integer.parseInt(properties.getProperty("httpclient.retry", "0"));
            //仍未获取到，则生成新的httpClient
            // 设置连接池
            final PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(defaultTimeToLive, TimeUnit.SECONDS);
            cm.setMaxTotal(maxTotal); // 应用允许的最大并发线程数
            cm.setDefaultMaxPerRoute(defaultMaxPerRoute);
            cm.setDefaultSocketConfig(SocketConfig.custom()
                    .setSoTimeout(defaultSoTimeout)//3秒超时
                    .setSoKeepAlive(true)
                    .setSoReuseAddress(true)
                    .build());

            // 钩子连接客户端
            final CloseableHttpClient newHttpClient = HttpClients.custom()
                    .setConnectionManager(cm)
                    .setDefaultRequestConfig(requestConfig) //使用传入的requestConfig
                    .setRetryHandler(new DefaultHttpRequestRetryHandler(defaultRetry, false))
                    .build();
            //put into map
            httpClientMap.put(hostAndPort, newHttpClient);
            httpClient = newHttpClient;

            //调整shutdownhook的线程方法，仅增加一个thread，且在第一个client初始化时
            if (!httpClientMap.isEmpty() && httpClientMap.size() == 1) {
                // 注册关闭钩子
                Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            if (httpClientMap != null && !httpClientMap.isEmpty()) {
                                logger.info("shutdown hook is running for all httpClient");
                                for (CloseableHttpClient httpClient : httpClientMap.values())
                                    if (null != httpClient) {
                                        httpClient.close();
                                    }
                            }
                        } catch (Exception e) {
                            logger.error("关闭http连接池时出错：" + e.getMessage(), e);
                        }
                    }
                }));
            }
        }
        return httpClient;
    }

    private static Properties loadProperties() {
        Properties properties = new Properties();
        try {
            InputStream inStream = Thread.currentThread().getContextClassLoader().getResourceAsStream(DEFAULT_CONFIG_FILE);
            properties.load(inStream);
            return properties;
        } catch (Exception e) {
            logger.warn("load properties from {} error. default value will be used. please check it.", DEFAULT_CONFIG_FILE);
        }
        return properties;
    }

    private static String getHostAndPort(String url) {
        String tmp = StringUtils.removeStartIgnoreCase(url, "http://");
        return StringUtils.substringBefore(tmp, "/");
    }

    public static String httpGet(String requestUrl, Map<String, String> params, Charset charset) {
        return httpGet(requestUrl, params, charset, false, DEFAULT_CONNECTION_TIMEOUT, DEFAULT_READ_TIMEOUT);
    }

    public static String httpGet(String requestUrl, Map<String, String> params, Charset charset, int connectionTimeout, int readTimeout) {
        return httpGet(requestUrl, params, charset, false, connectionTimeout, readTimeout);
    }

    public static String httpGet(String requestUrl, Map<String, String> params, Charset charset, boolean writeAccessLog, int connectionTimeout, int readTimeout) {
        return httpGetByHttpClient(requestUrl, params, charset, writeAccessLog, connectionTimeout, readTimeout);
    }

    public static File httpGetDownloadFile(String requestUrl, int connectionTimeout, int readTimeout, String dirPath, String fileName) {
        RequestConfig defaultRequestConfig = RequestConfig.custom()
                .setSocketTimeout(connectionTimeout > 0 ? connectionTimeout : DEFAULT_CONNECTION_TIMEOUT)
                .setConnectTimeout(readTimeout > 0 ? readTimeout : DEFAULT_READ_TIMEOUT)
                .setConnectionRequestTimeout(readTimeout > 0 ? readTimeout : DEFAULT_READ_TIMEOUT)
                .setContentCompressionEnabled(true)
                .build();
        CloseableHttpClient httpClient = getHttpClient(requestUrl, defaultRequestConfig);
        CloseableHttpResponse httpResponse = null;
        HttpGet httpget;
        File file = null;
        try {
            httpget = new HttpGet(requestUrl);
            httpget.setConfig(defaultRequestConfig);
            httpget.setHeader("Cache-Control", "no-cache");
            httpget.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.61 Safari/537.36");
            httpget.setHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8");
            httpResponse = httpClient.execute(httpget);
            if (HttpStatus.SC_OK == httpResponse.getStatusLine().getStatusCode()) {
                HttpEntity entity = httpResponse.getEntity();
                InputStream in = entity.getContent();
                file = savePicToDisk(in, dirPath, fileName);
            }
        } catch (Exception e) {
            logger.error("access url={} error.", requestUrl, e);
        } finally {
            try {
                if (httpResponse != null) {
                    httpResponse.close();
                }
            } catch (IOException e) {
                logger.error("HttpUtil.httpGet close connection for url={} error.", requestUrl, e);
            }
        }
        return file;
    }

    private static File savePicToDisk(InputStream in, String dirPath, String filePath) {
        File file = null;
        try {
            File dir = new File(dirPath);
            if (dir == null || !dir.exists()) {
                dir.mkdirs();
            }

            String realPath = dirPath.concat(filePath);
            file = new File(realPath);
            if (file == null || !file.exists()) {
                file.createNewFile();
            }

            FileOutputStream fos = new FileOutputStream(file);
            byte[] buf = new byte[1024];
            int len = 0;
            while ((len = in.read(buf)) != -1) {
                fos.write(buf, 0, len);
            }
            fos.flush();
            fos.close();

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return file;
    }

    public static String httpGetByHttpClient(String requestUrl, Map<String, String> params, Charset charset, boolean writeAccessLog, int connectionTimeout, int readTimeout) {
        RequestConfig defaultRequestConfig = RequestConfig.custom()
                .setSocketTimeout(connectionTimeout > 0 ? connectionTimeout : DEFAULT_CONNECTION_TIMEOUT)
                .setConnectTimeout(readTimeout > 0 ? readTimeout : DEFAULT_READ_TIMEOUT)
                .setConnectionRequestTimeout(readTimeout > 0 ? readTimeout : DEFAULT_READ_TIMEOUT)
                .setContentCompressionEnabled(true)//设置自动检测gzip压缩
                .build();
        CloseableHttpClient httpClient = getHttpClient(requestUrl, defaultRequestConfig);
        HttpGet httpGet;
        CloseableHttpResponse httpResponse = null;
        try {
            httpGet = new HttpGet(buildUri(requestUrl, params));
            httpGet.setHeader("Cache-Control", "no-cache");
            RequestConfig requestConfig = RequestConfig.copy(defaultRequestConfig)
                    .build();
            httpGet.setConfig(requestConfig);
            httpResponse = httpClient.execute(httpGet);
            //status==200
            if (httpResponse != null && httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity httpEntity = httpResponse.getEntity();
                return EntityUtils.toString(httpEntity, charset);
            }
        } catch (Exception e) {
            if (writeAccessLog) {
                logger.error("access url={} error.", requestUrl, e);
            }
        } finally {
            try {
                if (httpResponse != null) {
                    httpResponse.close();
                }
            } catch (IOException e) {
                if (writeAccessLog) {
                    logger.error("HttpUtil.httpGet close connection for url={} error.", requestUrl, e);
                }
            }
        }
        return "";
    }

    private static URI buildUri(String baseUrl, Map<String, String> props) throws URISyntaxException {
        URIBuilder uriBuilder = new URIBuilder(baseUrl);
        if (MapUtils.isNotEmpty(props)) {
            for (Map.Entry<String, String> prop : props.entrySet()) {
                uriBuilder.addParameter(prop.getKey(), prop.getValue());
            }
        }
        return uriBuilder.build();
    }

    public static String httpPost(String requestUrl, String data, String contentType, Charset charset) {
        return httpPost(requestUrl, data, contentType, charset, true, DEFAULT_CONNECTION_TIMEOUT, DEFAULT_READ_TIMEOUT);
    }

    public static String httpPost(String requestUrl, String data, String contentType, Charset charset, int connectionTimeout, int readTimeout) {
        return httpPost(requestUrl, data, contentType, charset, false, connectionTimeout, readTimeout);
    }

    public static String httpPost(String requestUrl, String data, String contentType, Charset charset, boolean writeAccessLog, int connectionTimeout, int readTimeout) {
        return httpPostByHttpClient(requestUrl, data, contentType, charset, writeAccessLog, connectionTimeout, readTimeout);
    }

    public static String httpPostByHttpClient(String requestUrl, String data, String contentType, Charset charset, boolean writeAccessLog, int connectionTimeout, int readTimeout) {
        RequestConfig defaultRequestConfig = RequestConfig.custom()
                .setSocketTimeout(connectionTimeout > 0 ? connectionTimeout : DEFAULT_CONNECTION_TIMEOUT)
                .setConnectTimeout(readTimeout > 0 ? readTimeout : DEFAULT_READ_TIMEOUT)
                .setConnectionRequestTimeout(readTimeout > 0 ? readTimeout : DEFAULT_READ_TIMEOUT)
                .setContentCompressionEnabled(true)//设置自动检测gzip压缩
                .setExpectContinueEnabled(false)
                .build();
        CloseableHttpClient httpClient = getHttpClient(requestUrl, defaultRequestConfig);
        HttpPost httpPost;
        CloseableHttpResponse httpResponse = null;
        try {
            httpPost = new HttpPost(requestUrl);
            httpPost.setHeader("Cache-Control", "no-cache");
            StringEntity se = new StringEntity(data, ContentType.create(contentType, charset.name()));
            httpPost.setEntity(se);
            RequestConfig requestConfig = RequestConfig.copy(defaultRequestConfig)
                    .build();
            httpPost.setConfig(requestConfig);
            httpResponse = httpClient.execute(httpPost);
            //status==200
            if (httpResponse != null && httpResponse.getStatusLine() != null && httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity httpEntity = httpResponse.getEntity();
                return EntityUtils.toString(httpEntity, charset);
            }
        } catch (Exception e) {
            if (writeAccessLog) {
                logger.error("access url={} with data={} error.", requestUrl, data, e);
            }
        } finally {
            try {
                if (httpResponse != null) {
                    httpResponse.close();
                }
            } catch (IOException e) {
                if (writeAccessLog) {
                    logger.error("HttpUtil.httpPostByHttpClient close connection for url={} error.", requestUrl, e);
                }
            }
        }
        return "";
    }


}

```
