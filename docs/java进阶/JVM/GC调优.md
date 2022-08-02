1. JVM参数
- -XX:+PrintFlagsFinal -version | findstr "GC" 查看虚拟机运行参数
2. 调优领域
 - 内存
 - 锁竞争
 - cpu占用
 - io
 3. 调优目标
- 【低延迟】还是【高吞吐量】，选择合适的垃圾回收期
- CMS,G1,ZGC 低延迟
- ParallelGC 高吞吐量
4. 新生代调优
- 新生代特点
> 所有new操作的内存分配非常廉价，
> TLAB (Thread Local Allocation Buffer 线程本地分配缓冲区), 线程在堆中的私有内存区域，创建对象时优先分配在此区域，为了解决多线程环境下内存分配区域的竞争，加速内存分配速度。
> 死亡对象的回收代价是零
> 大部分对象用过即死
> Minor GC 的时间远低于 Full GC

- 建议新生代大小占比在整个堆内存的 1/4 ~ 1/2, 当新生代内存占比过大意味着老年代内存不足以至于容易触发 Full GC
- 新生代内容纳所有【并大量 * (请求 - 响应)】的数据
- servivor 区大到能保留【当前活跃对象+需要晋升对象】
- 晋升阈值配置得当，让长时间存货对象尽快晋升
> -XX:MaxTenuringThreshold=threshold 设置晋升年龄
> -XX:PrintTenuringDistribution 打印晋升详细信息

5. 老年代调优
- CMS 老年代内存越大越好
- 先尝试不做调优，如果没有Full GC 说明不是老年代引起的
- 观察发生 Full GC 时老年代内存占用，将老年代内存预设调大 1/4 ~ 1/3
> -XX:CMSInitiatingOccupancyFration=precent 老年代内存占比达到多少时使用CMS进行垃圾回收

6. [# JAVA 线上故障排查完整套路](https://mp.weixin.qq.com/s/XNJKYJ1BIJeBYax2QTrF6w)

