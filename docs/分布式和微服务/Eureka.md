#### Eureka 和 Zookeeper 的区别

Eureka 基于 AP，只要有任意一个 server 节点可用集群即可用，Zookeeper 基于 CP，主节点失效后需要等待新的主节点选举完成后才能对外提供服务。

### Eureka Client

主要类是 `org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient `，该类通过组合的方式引入了 `com.netflix.discovery.EurekaClient`， 实际实现类是 `com.netflix.discovery.DiscoveryClient` 。

#### org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient

```java
public class EurekaDiscoveryClient implements DiscoveryClient {
	private final EurekaClient eurekaClient;
	/**
	* 获取描述
	*/ 
	@Override
	public String description() {
        return "Spring Cloud Eureka Discovery Client";
    }
    
    /**
	* 根据服务 id 获取实例列表
	*/ 
    @Override
    public List<ServiceInstance> getInstances(String serviceId) {
        List<InstanceInfo> infos = this.eurekaClient.getInstancesByVipAddress(serviceId, false);
        List<ServiceInstance> instances = new ArrayList();
        Iterator var4 = infos.iterator();

        while(var4.hasNext()) {
            InstanceInfo info = (InstanceInfo)var4.next();
            instances.add(new EurekaDiscoveryClient.EurekaServiceInstance(info));
        }

        return instances;
    }
    
    /**
	* 获取全部实例 id 列表
	*/ 
    @Override
    public List<String> getServices() {
        Applications applications = this.eurekaClient.getApplications();
        if (applications == null) {
            return Collections.emptyList();
        } else {
            List<Application> registered = applications.getRegisteredApplications();
            List<String> names = new ArrayList();
            Iterator var4 = registered.iterator();

            while(var4.hasNext()) {
                Application app = (Application)var4.next();
                if (!app.getInstances().isEmpty()) {
                    names.add(app.getName().toLowerCase());
                }
            }

            return names;
        }
    }
}
```

#### com.netflix.discovery.EurekaClient

```java
// 指定实现类为
@ImplementedBy(DiscoveryClient.class)
public interface EurekaClient extends LookupService {
    // 根据实例名称(spring.application.name)获取实例容器
    Application getApplication(String appName);
    // 返回所有实例信息
    Applications getApplications();
	// 根据服务 id (eureka.instance.instance-id)获取实例信息
    List<InstanceInfo> getInstancesById(String id);
}
```

#### com.netflix.discovery.DiscoveryClient

```java
@Singleton
public class DiscoveryClient implements EurekaClient {
    // 构造函数：拉取注册表信息、服务注册、初始化心跳、缓存刷新、按需注册定时任务
    @Inject
    DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args, Provider<BackupRegistry> backupRegistryProvider) {
        // eureka.client.fetch-registry
        if (config.shouldFetchRegistry()) {
            this.registryStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRY_PREFIX + "lastUpdateSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.registryStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }
		// eureka.server.enable-self-preservation
        if (config.shouldRegisterWithEureka()) {
            this.heartbeatStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRATION_PREFIX + "lastHeartbeatSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.heartbeatStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }
		... ...
         //  定义定时器线程池，线程大小2，一个线程用于发送心跳，一个线程用于刷新缓存
        scheduler = Executors.newScheduledThreadPool(2, new ThreadFactoryBuilder()
                            .setNameFormat("DiscoveryClient-%d")
                            .setDaemon(true)
                            .build());
        heartbeatExecutor = new ThreadPoolExecutor( 1, clientConfig.getHeartbeatExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(),
            new ThreadFactoryBuilder()
            .setNameFormat("DiscoveryClient-HeartbeatExecutor-%d")
            .setDaemon(true)
            .build()
        );  // use direct handoff

        cacheRefreshExecutor = new ThreadPoolExecutor(
            1, clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(),
            new ThreadFactoryBuilder()
            .setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d")
            .setDaemon(true)
            .build()
        ); // use direct handoff
        // 初始化 Eureka CLient 和 Eureka Server 进行 Http 请求的 Jersey 客户端
        eurekaTransport = new EurekaTransport();
        scheduleServerEndpointTask(eurekaTransport, args);
        ... ...
        // 从 Eureka Server 拉取注册信息
        if (clientConfig.shouldFetchRegistry() && !fetchRegistry(false)) {
            fetchRegistryFromBackup();
        }
        // 注册本服务实例到 Eureka Server
        if (this.preRegistrationHandler != null) {
            this.preRegistrationHandler.beforeRegistration();
        }

        if (clientConfig.shouldRegisterWithEureka() && clientConfig.shouldEnforceRegistrationAtInit()) {
            try {
                if (!register() ) {
                    throw new IllegalStateException("Registration error at startup. Invalid server response.");
                }
            } catch (Throwable th) {
                throw new IllegalStateException(th);
            }
        }
        // 初始化定时任务，如集群解析、心跳、实例注册、拉取
        initScheduledTasks();
    }
}
```

#### 拉取服务实例

```java
private boolean fetchRegistry(boolean forceFullRegistryFetch) {
    Stopwatch tracer = FETCH_REGISTRY_TIMER.start();
    try {
        // 如果增量式拉取被禁止，则使用全量拉取
        Applications applications = getApplications();
        if (clientConfig.shouldDisableDelta()
            || (!Strings.isNullOrEmpty(clientConfig.getRegistryRefreshSingleVipAddress()))
            || forceFullRegistryFetch
            || (applications == null)
            || (applications.getRegisteredApplications().size() == 0)
            || (applications.getVersion() == -1)) //Client application does not have latest library supporting delta
        {
            clientConfig.getRegistryRefreshSingleVipAddress());
            // 全量拉取
            getAndStoreFullRegistry();
        } else {
            // 增量拉取
            getAndUpdateDelta(applications);
        }
        // 计算应用集合一致性哈希码
        applications.setAppsHashCode(applications.getReconcileHashCode());
        logTotalInstances();
    } catch (Throwable e) {
        return false;
    } finally {
        if (tracer != null) {
            tracer.stop();
        }
    }
    // 更新远程实例状态之前推送缓存刷新事件，但是 Eureka 中没有提供默认的事件监听器
    onCacheRefreshed();
    // 基于缓存中被刷新的数据更新远程实例状态
    updateInstanceRemoteStatus();
    return true;
}
```

#### 全量拉取

```java
// 全量拉取，第一次拉取注册表信息是全量拉取
private void getAndStoreFullRegistry() throws Throwable {
    // 获取版本号，避免多线程拉取的情况下出现旧数据覆盖新数据的情况
    long currentUpdateGeneration = fetchRegistryGeneration.get();
    Applications apps = null;
    EurekaHttpResponse<Applications> httpResponse = clientConfig.getRegistryRefreshSingleVipAddress() == null
        ? eurekaTransport.queryClient.getApplications(remoteRegionsRef.get())
        : eurekaTransport.queryClient.getVip(clientConfig.getRegistryRefreshSingleVipAddress(), remoteRegionsRef.get());
    if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
        apps = httpResponse.getEntity();
    }
    if (apps == null) {
        // 检查版本号，确保版本号一直的情况才更新
    } else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
        localRegionApps.set(this.filterAndShuffle(apps));
        logger.debug("Got full registry with apps hashcode {}", apps.getAppsHashCode());
    } else {
        logger.warn("Not updating applications as another thread is updating it already");
    }
}
```

#### 增量拉取

```java
// 增量拉取，某个时间点后一段事件内（通常 3 分钟）发生的变更信息
private void getAndUpdateDelta(Applications applications) throws Throwable {
    // 版本号
    long currentUpdateGeneration = fetchRegistryGeneration.get();
    Applications delta = null;
    EurekaHttpResponse<Applications> httpResponse = eurekaTransport.queryClient.getDelta(remoteRegionsRef.get());
    if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
        delta = httpResponse.getEntity();
    }
    if (delta == null) {
        // 增量拉取失败则全量拉取
        getAndStoreFullRegistry();
    } else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
        String reconcileHashCode = "";
        if (fetchRegistryUpdateLock.tryLock()) {
            try {
                // 更新本地缓存，根据 instance 的 actionType 选择不同的处理方式
                updateDelta(delta);
                // 计算应用集合一致性哈希
                reconcileHashCode = getReconcileHashCode(applications);
            } finally {
                fetchRegistryUpdateLock.unlock();
            }
        } else {
            logger.warn("Cannot acquire update lock, aborting getAndUpdateDelta");
        }
        // 一致性哈希不一致则说明本次拉取的数据是脏数据，将全量拉取
        if (!reconcileHashCode.equals(delta.getAppsHashCode()) || clientConfig.shouldLogDeltaDiff()) {
            reconcileAndLogDifference(delta, reconcileHashCode);  // this makes a remoteCall
        }
    } 
}
```

#### 服务注册

```java
// 服务注册
boolean register() throws Throwable {
    EurekaHttpResponse<Void> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    return httpResponse.getStatusCode() == 204;
}
```

#### 初始化定时任务

```java
// 初始化定时任务
private void initScheduledTasks() {
    // 缓存刷新定时器
    if (clientConfig.shouldFetchRegistry()) {
        // 刷新间隔，默认 30 秒
        int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
        // 最大时延，默认 10
        int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
        scheduler.schedule(
            new TimedSupervisorTask("cacheRefresh", scheduler, cacheRefreshExecutor, registryFetchIntervalSeconds, TimeUnit.SECONDS, expBackOffBound, new CacheRefreshThread()),
            registryFetchIntervalSeconds, TimeUnit.SECONDS);
    }
	// 心跳定时器
    if (clientConfig.shouldRegisterWithEureka()) {
        // 刷新间隔，默认 30 秒
        int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
        int expBackOffBound = clientConfig.getHeartbeatExecutorExponentialBackOffBound();

        scheduler.schedule(
            new TimedSupervisorTask( "heartbeat", scheduler, heartbeatExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new HeartbeatThread()),
            renewalIntervalInSecs, TimeUnit.SECONDS);

        // 按需注册定时任务，client 信息或者状态发生改变时发起重新注册请求
        // 定时检查服务实例信息
        instanceInfoReplicator = new InstanceInfoReplicator(this, instanceInfo, clientConfig.getInstanceInfoReplicationIntervalSeconds(), 2); // burstSize
        // 监控应用状态变化
        statusChangeListener = new ApplicationInfoManager.StatusChangeListener() {
            @Override
            public String getId() {
                return "statusChangeListener";
            }

            @Override
            public void notify(StatusChangeEvent statusChangeEvent) {

                instanceInfoReplicator.onDemandUpdate();
            }
        };
        // 注册应用状态改变监控器
        if (clientConfig.shouldOnDemandUpdateStatusChange()) {
            applicationInfoManager.registerStatusChangeListener(statusChangeListener);
        }
        // 启动按需注册定时器
       instanceInfoReplicator.start(clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
}
```

#### 服务下线

```java
// 下线服务 
@PreDestroy
public synchronized void shutdown() {
    if (isShutdown.compareAndSet(false, true)) {
        if (statusChangeListener != null && applicationInfoManager != null) {
            // 注销状态监听器
            applicationInfoManager.unregisterStatusChangeListener(statusChangeListener.getId());
        }
		// 取消定时任务
        cancelScheduledTasks();
        if (applicationInfoManager != null
            && clientConfig.shouldRegisterWithEureka()
            && clientConfig.shouldUnregisterOnShutdown()) {
            // 服务下线
            applicationInfoManager.setInstanceStatus(InstanceStatus.DOWN);
            unregister();
        }
		// 关闭 Jersy 客户端
        if (eurekaTransport != null) {
            eurekaTransport.shutdown();
        }
		// 关闭监视器
        heartbeatStalenessMonitor.shutdown();
        registryStalenessMonitor.shutdown();
    }
}
```

#### com.netflix.discovery.TimedSupervisorTask

```java
public class TimedSupervisorTask extends TimerTask {
    public void run() {
        Future<?> future = null;
        try {
            // 执行任务
            future = executor.submit(task);
            threadPoolLevelGauge.set((long) executor.getActiveCount());
            // 等待任务执行完成
            future.get(timeoutMillis, TimeUnit.MILLISECONDS);  // block until done or timeout
            // 设置下次任务执行时间
            delay.set(timeoutMillis);
            threadPoolLevelGauge.set((long) executor.getActiveCount());
        } catch (TimeoutException e) {
            // 任务超市
            timeoutCounter.increment();
		   // 设置下次任务的时间间隔
            long currentDelay = delay.get();
            long newDelay = Math.min(maxDelay, currentDelay * 2);
            delay.compareAndSet(currentDelay, newDelay);
        } catch (RejectedExecutionException e) {
		   // 统计任务拒绝次数
            rejectedCounter.increment();
        } catch (Throwable e) {
		   // 统计其他异常
            throwableCounter.increment();
        } finally {
            // 取消未完成任务
            if (future != null) {
                future.cancel(true);
            }
		   // 如果任务未关闭执行下次任务
            if (!scheduler.isShutdown()) {
                scheduler.schedule(this, delay.get(), TimeUnit.MILLISECONDS);
            }
        }
    }
}
```

### Eureka Server

功能：

- 服务注册
- 接受服务心跳
- 服务剔除
- 服务下线
- 集群同步
- 获取注册表中服务实例信息

主要代码结构如下：

![](images/EurekaServer.png)

LeaseManager 对注册到 Eureka Server 的服务实例租约管理，包括服务注册、服务下线、服务租约更新、服务剔除。服务实例信息的对象是 Lease，定义了对租约的注册、下线、更新等操作。租约默认时长 90 秒。

InstanceRegistry 接口中增加了管理服务实例租约和查询注册表中的服务实例信息。

PeerAwareInstanceRegistry 定义了集群同步操作

PeerAwareInstanceRegistryImpl 在对本地注册表操作的基础上添加了对其 Peer 节点的同步复制操作。

#### 服务注册(`com.netflix.eureka.registry.AbstractInstanceRegistry`)

```java
public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
    try {
        read.lock();
        // 根据 appName 对服务实例进行分类
        Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
        REGISTER.increment(isReplication);
        if (gMap == null) {
            final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
            gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
            if (gMap == null) {
                gMap = gNewMap;
            }
        }
        // 根据 instanceId 获取服务实例
        Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
        // 如果已经存在租约，比较最后更新的时间戳，保留最后更新的时间戳最新的那个实例
        if (existingLease != null && (existingLease.getHolder() != null)) {
            Long existingLastDirtyTimestamp = existingLease.getHolder().getLastDirtyTimestamp();
            Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();
            if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
                registrant = existingLease.getHolder();
            }
        } else {
            // 如果租约不存在则新注册租约
            synchronized (lock) {
                if (this.expectedNumberOfRenewsPerMin > 0) {
                    this.expectedNumberOfRenewsPerMin = this.expectedNumberOfRenewsPerMin + 2;
                    this.numberOfRenewsPerMinThreshold =
                        (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
                }
            }
        }
        // 注册新租约
        Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
        if (existingLease != null) {
            lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
        }
        gMap.put(registrant.getId(), lease);
        synchronized (recentRegisteredQueue) {
            recentRegisteredQueue.add(new Pair<Long, String>(
                System.currentTimeMillis(),
                registrant.getAppName() + "(" + registrant.getId() + ")"));
        }
        // This is where the initial state transfer of overridden status happens
        if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
            logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the "
                         + "overrides", registrant.getOverriddenStatus(), registrant.getId());
            if (!overriddenInstanceStatusMap.containsKey(registrant.getId())) {
                logger.info("Not found overridden id {} and hence adding it", registrant.getId());
                overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
            }
        }
        InstanceStatus overriddenStatusFromMap = overriddenInstanceStatusMap.get(registrant.getId());
        if (overriddenStatusFromMap != null) {
            logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
            registrant.setOverriddenStatus(overriddenStatusFromMap);
        }

        // Set the status based on the overridden status rules
        InstanceStatus overriddenInstanceStatus = getOverriddenInstanceStatus(registrant, existingLease, isReplication);
        registrant.setStatusWithoutDirty(overriddenInstanceStatus);

        // If the lease is registered with UP status, set lease service up timestamp
        if (InstanceStatus.UP.equals(registrant.getStatus())) {
            lease.serviceUp();
        }
        registrant.setActionType(ActionType.ADDED);
        recentlyChangedQueue.add(new RecentlyChangedItem(lease));
        registrant.setLastUpdatedTimestamp();
        invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
        logger.info("Registered instance {}/{} with status {} (replication={})",
                    registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication);
    } finally {
        read.unlock();
    }
}
```

#### 接受服务心跳(`com.netflix.eureka.registry.AbstractInstanceRegistry`)

```java
public boolean renew(String appName, String id, boolean isReplication) {
    RENEW.increment(isReplication);
    Map<String, Lease<InstanceInfo>> gMap = registry.get(appName);
    Lease<InstanceInfo> leaseToRenew = null;
    if (gMap != null) {
        leaseToRenew = gMap.get(id);
    }
    if (leaseToRenew == null) {
        RENEW_NOT_FOUND.increment(isReplication);
        logger.warn("DS: Registry: lease doesn't exist, registering resource: {} - {}", appName, id);
        return false;
    } else {
        InstanceInfo instanceInfo = leaseToRenew.getHolder();
        if (instanceInfo != null) {
            // touchASGCache(instanceInfo.getASGName());
            InstanceStatus overriddenInstanceStatus = this.getOverriddenInstanceStatus(
                instanceInfo, leaseToRenew, isReplication);
            if (overriddenInstanceStatus == InstanceStatus.UNKNOWN) {
                return false;
            }
            if (!instanceInfo.getStatus().equals(overriddenInstanceStatus)) {
                    instanceInfo.getOverriddenStatus().name(),
                    instanceInfo.getId());
                instanceInfo.setStatusWithoutDirty(overriddenInstanceStatus);

            }
        }
        renewsLastMin.increment();
        // 更新租约中的有效时间
        leaseToRenew.renew();
        return true;
    }
}
```

#### 服务剔除(`com.netflix.eureka.registry.AbstractInstanceRegistry`)

```java
public void evict() {
    evict(0l);
}

public void evict(long additionalLeaseMs) {
    // 自我保护机制，该状态下不允许剔除服务
    if (!isLeaseExpirationEnabled()) {
        return;
    }
    // 遍历注册表获取所有过期租约
    List<Lease<InstanceInfo>> expiredLeases = new ArrayList<>();
    for (Entry<String, Map<String, Lease<InstanceInfo>>> groupEntry : registry.entrySet()) {
        Map<String, Lease<InstanceInfo>> leaseMap = groupEntry.getValue();
        if (leaseMap != null) {
            for (Entry<String, Lease<InstanceInfo>> leaseEntry : leaseMap.entrySet()) {
                Lease<InstanceInfo> lease = leaseEntry.getValue();
                if (lease.isExpired(additionalLeaseMs) && lease.getHolder() != null) {
                    expiredLeases.add(lease);
                }
            }
        }
    }

    // 计算最大允许剔除租约数量，获取租约表总数量
    int registrySize = (int) getLocalRegistrySize();
    // 计算注册表租约阈值
    int registrySizeThreshold = (int) (registrySize * serverConfig.getRenewalPercentThreshold());
    // 计算剔除租约数量
    int evictionLimit = registrySize - registrySizeThreshold;

    int toEvict = Math.min(expiredLeases.size(), evictionLimit);
    if (toEvict > 0) {
	    // 逐个随机剔除
        Random random = new Random(System.currentTimeMillis());
        for (int i = 0; i < toEvict; i++) {
            // Pick a random item (Knuth shuffle algorithm)
            int next = i + random.nextInt(expiredLeases.size() - i);
            Collections.swap(expiredLeases, i, next);
            Lease<InstanceInfo> lease = expiredLeases.get(i);

            String appName = lease.getHolder().getAppName();
            String id = lease.getHolder().getId();
            EXPIRED.increment();
            logger.warn("DS: Registry: expired lease for {}/{}", appName, id);
            internalCancel(appName, id, false);
        }
    }
}
```

#### 服务下线(`com.netflix.eureka.registry.AbstractInstanceRegistry`)

```java
public boolean cancel(String appName, String id, boolean isReplication) {
    return internalCancel(appName, id, isReplication);
}

protected boolean internalCancel(String appName, String id, boolean isReplication) {
    try {
        read.lock();
        CANCEL.increment(isReplication);
        Map<String, Lease<InstanceInfo>> gMap = registry.get(appName);
        Lease<InstanceInfo> leaseToCancel = null;
        // 移除服务实例的租约
        if (gMap != null) {
            leaseToCancel = gMap.remove(id);
        }
        // 将服务实例信息添加到最近要下线服务实例统计队列
        synchronized (recentCanceledQueue) {
            recentCanceledQueue.add(new Pair<Long, String>(System.currentTimeMillis(), appName + "(" + id + ")"));
        }
        InstanceStatus instanceStatus = overriddenInstanceStatusMap.remove(id);
        if (instanceStatus != null) {
            logger.debug("Removed instance id {} from the overridden map which has value {}", id, instanceStatus.name());
        }
        // 租约不存在返回 false
        if (leaseToCancel == null) {
            CANCEL_NOT_FOUND.increment(isReplication);
            logger.warn("DS: Registry: cancel failed because Lease is not registered for: {}/{}", appName, id);
            return false;
        } else {
            // 设置租约的下线时间
            leaseToCancel.cancel();
            InstanceInfo instanceInfo = leaseToCancel.getHolder();
            String vip = null;
            String svip = null;
            if (instanceInfo != null) {
                // 添加最近租约变更队列，标识 ActionTYpe 为 DELETED，用于增量拉取注册表
                instanceInfo.setActionType(ActionType.DELETED);
                recentlyChangedQueue.add(new RecentlyChangedItem(leaseToCancel));
                instanceInfo.setLastUpdatedTimestamp();
                vip = instanceInfo.getVIPAddress();
                svip = instanceInfo.getSecureVipAddress();
            }
            invalidateCache(appName, vip, svip);
            logger.info("Cancelled instance {}/{} (replication={})", appName, id, isReplication);
            return true;
        }
    } finally {
        read.unlock();
    }
}
```

#### 集群同步

```java
public int syncUp() {
    // 从临近的 peer 节点复制整个注册表
    int count = 0;
	// 获取不到则线程等待
    for (int i = 0; ((i < serverConfig.getRegistrySyncRetries()) && (count == 0)); i++) {
        if (i > 0) {
            try {
                Thread.sleep(serverConfig.getRegistrySyncRetryWaitMs());
            } catch (InterruptedException e) {
                logger.warn("Interrupted during registry transfer..");
                break;
            }
        }
        // 获取所有服务实例
        Applications apps = eurekaClient.getApplications();
        for (Application app : apps.getRegisteredApplications()) {
            for (InstanceInfo instance : app.getInstances()) {
                try {
                    if (isRegisterable(instance)) {
                        register(instance, instance.getLeaseInfo().getDurationInSecs(), true);
                        count++;
                    }
                } catch (Throwable t) {
                    logger.error("During DS init copy", t);
                }
            }
        }
    }
    return count;
}
```

初始化本地注册表时，Eureka Server 不接受来自 Client 的请求（如注册、获取服务等）。

每个 Eureka Server 对本地注册表管理时会同步到 peer 节点

```java
// 同步租约状态
public boolean renew(final String appName, final String id, final boolean isReplication) {
    if (super.renew(appName, id, isReplication)) {
        replicateToPeers(Action.Heartbeat, appName, id, null, null, isReplication);
        return true;
    }
    return false;
}
// 注册
public void register(final InstanceInfo info, final boolean isReplication) {
    int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
    if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
        leaseDuration = info.getLeaseInfo().getDurationInSecs();
    }
    super.register(info, leaseDuration, isReplication);
    replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
}
// 服务下线
public boolean cancel(final String appName, final String id,
                      final boolean isReplication) {
    if (super.cancel(appName, id, isReplication)) {
        replicateToPeers(Action.Cancel, appName, id, null, null, isReplication);
        synchronized (lock) {
            if (this.expectedNumberOfRenewsPerMin > 0) {
                // Since the client wants to cancel it, reduce the threshold (1 for 30 seconds, 2 for a minute)
                this.expectedNumberOfRenewsPerMin = this.expectedNumberOfRenewsPerMin - 2;
                this.numberOfRenewsPerMinThreshold =
                    (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
            }
        }
        return true;
    }
    return false;
}
```

Eureka Server 获取注册表也是通过全量拉取和增量拉取的

##### 全量拉取(`com.netflix.eureka.registry.AbstractInstanceRegistry`)

```java
 public Applications getApplicationsFromMultipleRegions(String[] remoteRegions) {

     boolean includeRemoteRegion = null != remoteRegions && remoteRegions.length != 0;

     logger.debug("Fetching applications registry with remote regions: {}, Regions argument {}",
                  includeRemoteRegion, remoteRegions);

     if (includeRemoteRegion) {
         GET_ALL_WITH_REMOTE_REGIONS_CACHE_MISS.increment();
     } else {
         GET_ALL_CACHE_MISS.increment();
     }
     Applications apps = new Applications();
     apps.setVersion(1L);
     for (Entry<String, Map<String, Lease<InstanceInfo>>> entry : registry.entrySet()) {
         Application app = null;

         if (entry.getValue() != null) {
             for (Entry<String, Lease<InstanceInfo>> stringLeaseEntry : entry.getValue().entrySet()) {
                 Lease<InstanceInfo> lease = stringLeaseEntry.getValue();
                 if (app == null) {
                     app = new Application(lease.getHolder().getAppName());
                 }
                 app.addInstance(decorateInstanceInfo(lease));
             }
         }
         if (app != null) {
             apps.addApplication(app);
         }
     }
     if (includeRemoteRegion) {
         for (String remoteRegion : remoteRegions) {
             RemoteRegionRegistry remoteRegistry = regionNameVSRemoteRegistry.get(remoteRegion);
             if (null != remoteRegistry) {
                 Applications remoteApps = remoteRegistry.getApplications();
                 for (Application application : remoteApps.getRegisteredApplications()) {
                     if (shouldFetchFromRemoteRegistry(application.getName(), remoteRegion)) {
                         logger.info("Application {}  fetched from the remote region {}",
                                     application.getName(), remoteRegion);

                         Application appInstanceTillNow = apps.getRegisteredApplications(application.getName());
                         if (appInstanceTillNow == null) {
                             appInstanceTillNow = new Application(application.getName());
                             apps.addApplication(appInstanceTillNow);
                         }
                         for (InstanceInfo instanceInfo : application.getInstances()) {
                             appInstanceTillNow.addInstance(instanceInfo);
                         }
                     } else {
                         logger.debug("Application {} not fetched from the remote region {} as there exists a "
                                      + "whitelist and this app is not in the whitelist.",
                                      application.getName(), remoteRegion);
                     }
                 }
             } else {
                 logger.warn("No remote registry available for the remote region {}", remoteRegion);
             }
         }
     }
     apps.setAppsHashCode(apps.getReconcileHashCode());
     return apps;
 }
```

##### 增量拉取(`com.netflix.eureka.registry.AbstractInstanceRegistry`)

```java
public Applications getApplicationDeltasFromMultipleRegions(String[] remoteRegions) {
    if (null == remoteRegions) {
        remoteRegions = allKnownRemoteRegions; // null means all remote regions.
    }

    boolean includeRemoteRegion = remoteRegions.length != 0;

    if (includeRemoteRegion) {
        GET_ALL_WITH_REMOTE_REGIONS_CACHE_MISS_DELTA.increment();
    } else {
        GET_ALL_CACHE_MISS_DELTA.increment();
    }

    Applications apps = new Applications();
    apps.setVersion(responseCache.getVersionDeltaWithRegions().get());
    Map<String, Application> applicationInstancesMap = new HashMap<String, Application>();
    try {
        write.lock();
        Iterator<RecentlyChangedItem> iter = this.recentlyChangedQueue.iterator();
        logger.debug("The number of elements in the delta queue is :{}", this.recentlyChangedQueue.size());
        while (iter.hasNext()) {
            Lease<InstanceInfo> lease = iter.next().getLeaseInfo();
            InstanceInfo instanceInfo = lease.getHolder();
            logger.debug("The instance id {} is found with status {} and actiontype {}",
                         instanceInfo.getId(), instanceInfo.getStatus().name(), instanceInfo.getActionType().name());
            Application app = applicationInstancesMap.get(instanceInfo.getAppName());
            if (app == null) {
                app = new Application(instanceInfo.getAppName());
                applicationInstancesMap.put(instanceInfo.getAppName(), app);
                apps.addApplication(app);
            }
            app.addInstance(decorateInstanceInfo(lease));
        }

        if (includeRemoteRegion) {
            for (String remoteRegion : remoteRegions) {
                RemoteRegionRegistry remoteRegistry = regionNameVSRemoteRegistry.get(remoteRegion);
                if (null != remoteRegistry) {
                    Applications remoteAppsDelta = remoteRegistry.getApplicationDeltas();
                    if (null != remoteAppsDelta) {
                        for (Application application : remoteAppsDelta.getRegisteredApplications()) {
                            if (shouldFetchFromRemoteRegistry(application.getName(), remoteRegion)) {
                                Application appInstanceTillNow =
                                    apps.getRegisteredApplications(application.getName());
                                if (appInstanceTillNow == null) {
                                    appInstanceTillNow = new Application(application.getName());
                                    apps.addApplication(appInstanceTillNow);
                                }
                                for (InstanceInfo instanceInfo : application.getInstances()) {
                                    appInstanceTillNow.addInstance(instanceInfo);
                                }
                            }
                        }
                    }
                }
            }
        }

        Applications allApps = getApplicationsFromMultipleRegions(remoteRegions);
        apps.setAppsHashCode(allApps.getReconcileHashCode());
        return apps;
    } finally {
        write.unlock();
    }
}
```

