EurekaClientConfig要求的eureka客户端配置的默认实现。
在配置文件中提供了配置eureka客户端所需的信息，下面是一些比较重要的配置。



eureka.client.refresh.interval

表示从eureka服务器获取注册表信息的频率（以秒为单位）,默认30s。

```java
    int getRegistryFetchIntervalSeconds();
```



eureka.appinfo.replicate.interval
表示将本地实例信息复制到EurekaServer的频率，默认30s

```java
   int getInstanceInfoReplicationIntervalSeconds();
```


eureka.appinfo.initial.replicate.time

表示最初将实例信息复制到eureka服务器的时间(以秒为单位)。 默认40s。

```java
	int getInitialInstanceInfoReplicationIntervalSeconds();
```


eureka.serviceUrlPollIntervalMs

表示获取EurekaServer的更新信息的频率，默认5*60s

```java
    int getEurekaServiceUrlPollIntervalSeconds();
```





eureka.eurekaServer.gzipContent

表示从EurekaServer获取的内容是否必须进行压缩。压缩EurekaServer的注册表信息可以优化网络流量。

```java
boolean shouldGZipContent();
```


eureka.eurekaServer.readTimeout

表示请求EurekaServer时的读取超时时间

```java
int getEurekaServerReadTimeoutSeconds();
```


eureka.eurekaServer.connectTimeout

表示请求EurekaServer时连接超时时间

```java
int getEurekaServerConnectTimeoutSeconds();
```


eureka.eurekaServer.maxTotalConnections

表示EurekaClient连接EurekaServer时的最大连接数

```java
int getEurekaServerTotalConnections();
```


eureka.eurekaServer.maxConnectionsPerHost

表示EurekaClient连接单个EurekaServer时的最大连接数

```java
int getEurekaServerTotalConnectionsPerHost();
```


eureka.registration.enabled

表示该实例是否需要注册到EurekaServer，以供其他服务实例发现。默认是true

```java
boolean shouldRegisterWithEureka();
```


eureka.preferSameZone

表示是否需要优先使用同一个区域中的EurekaServer,默认true

```java
boolean shouldPreferSameZoneEureka();
```


eureka.printDeltaFullDiff

指示是否在注册表信息方面记录eureka服务器与eureka客户端之间的差异。默认false.
Eureka客户端尝试从Eureka服务器仅检索增量更改，以最大程度地减少网络流量。收到增量后，eureka客户端会协调来自服务器的信息，以确认它没有丢失某些信息。当客户端与服务器通信时出现网络问题时，对帐可能会失败。如果对帐失败，则eureka客户端会获取完整的注册表信息。
在获取完整的注册表信息时，eureka客户端可以记录客户端和服务器之间的差异，此设置对此进行控制。

```java
boolean shouldLogDeltaDiff();
```


eureka.disableDelta

表示是否禁用增量获取，如果禁用增量获取，那么每次从EurekaServer拉取注册表信息时，都将会使用全量拉取的方式。默认false.

```java
boolean shouldDisableDelta();
```





```java
@Nullable
String fetchRegistryForRemoteRegions();
```




```java
String[] getAvailabilityZones(String region);
```


获取EurekaServer的地址集合

```java
List<String> getEurekaServerServiceUrls(String myZone);
```


eureka.shouldFilterOnlyUpInstances

表示是否在仅使用com.netflix.AppInfo.InstanceInfo.Instance.up状态过滤应用程序后获取应用程序。

```java
boolean shouldFilterOnlyUpInstances();
```


eureka.shouldFetchRegistry

表示是否需要从EurekaServer获取注册表信息

```java
boolean shouldFetchRegistry();
```


eureka.registryRefreshSingleVipAddress

表示客户端是否只对单个VIP的注册表信息感兴趣。 获取该VIP地址，默认为null

```java
@Nullable
String getRegistryRefreshSingleVipAddress();
```


eureka.client.heartbeat.threadPoolSize

表示要初始化的服务续约(心跳)线程池的大小，默认为5

```java
int getHeartbeatExecutorThreadPoolSize();
```


eureka.client.heartbeat.exponentialBackOffBound

表示心跳任务在发生超时的情况下，重试延迟的最大乘数值。 默认为10

```java
int getHeartbeatExecutorExponentialBackOffBound();
```


eureka.client.cacheRefresh.threadPoolSize

表示要初始化的缓存更新(注册表更新)线程池的大小，默认为5

```java
int getCacheRefreshExecutorThreadPoolSize();
```


eureka.client.cacheRefresh.exponentialBackOffBound

表示缓存更新(注册表更新)任务在发生超时的情况下，重试延迟的最大乘数值。 默认为10

```java
int getCacheRefreshExecutorExponentialBackOffBound();
```




