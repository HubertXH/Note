#### JAVA-SPI    
SPI 英文为 Service Provider Interface,提供的一套用于被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件。 SPI的作用就是为这些被扩展的API寻找服务实现。  

即：在各个不同模块之间通过接口编程交互，同一个接口在不同的组件中有着不相同的实现，而SPI就是根据不同的组件配置加载接口对应的实现类，从而实现灵活的扩展及插拔。

例：spring的slf4j-Logger接口，其实现有logback，log4j等。这些组建都实现了Logger接口的功能，在项目中使用其中可以既可以实现功能。其切换容易，不需要修改现有代码，只需要修改pom文件中的依赖。    

java中的SPI是通过ServiceLoader这个泪来实现
``` java
    public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
```
参数service通常为要实现的接口类。
```java

    private static final String PREFIX = "META-INF/services/";

    public static <S> ServiceLoader<S> load(Class<S> service,ClassLoader loader){
        return new ServiceLoader<>(service,loader);
    }
    
    
    private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }
    
    public void reload() {
        providers.clear();
        lookupIterator = new LazyIterator(service, loader);
    }
    
    private S nextService() {
        if (!hasNextService())
            throw new NoSuchElementException();
        String cn = nextName;
        nextName = null;
        Class<?> c = null;
        try {
            c = Class.forName(cn, false, loader);
        } catch (ClassNotFoundException x) {
            fail(service,
                 "Provider " + cn + " not found");
        }
        if (!service.isAssignableFrom(c)) {
            fail(service,
                 "Provider " + cn  + " not a subtype");
        }
        try {
            S p = service.cast(c.newInstance());
            providers.put(cn, p);
            return p;
        } catch (Throwable x) {
            fail(service,
                 "Provider " + cn + " could not be instantiated",
                 x);
        }
        throw new Error();          // This cannot happen
    }
    
    private boolean hasNextService() {
        if (nextName != null) {
            return true;
        }
        if (configs == null) {
            try {
                String fullName = PREFIX + service.getName();
                if (loader == null)
                    configs = ClassLoader.getSystemResources(fullName);
                else
                    configs = loader.getResources(fullName);
            } catch (IOException x) {
                fail(service, "Error locating configuration files", x);
            }
        }
        while ((pending == null) || !pending.hasNext()) {
            if (!configs.hasMoreElements()) {
                return false;
            }
            pending = parse(service, configs.nextElement());
        }
        nextName = pending.next();
        return true;
    }
    
```
根据以上代码可以看到，ServiceLoader读取了META-INF/services/这个路径下 名字为要实现的接口名称的文件。读取文件里面的内容，根据配置的类名称通过反射加载实现，并将其放到缓存中。