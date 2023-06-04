#### JVM常见参数

###### 标准参数
```
-verbose:class 打印每个class信息    
-verbose:gc 打印每次gc信息
```
###### 非标准参数
```
-Xloggc:filename 设置GC log文件的位置 -Xloggc:log/gc.log    
-Xms大小 设置堆的初始化大小 -Xmx2048m =-XX:InitialHeapSize    
-Xmx大小 设置堆的最大大小 -Xms1024m =-XX:MaxHeapSize 一般Xms=Xmx,防止扩容和缩容   
-Xmn大小 设置年轻代大小（初始化和最大） -Xmn256m    
-XX:NewSize -XX:MaxNewSize     分别指定年轻代的初始化和最大大小，建议年轻代占堆大小的1/4 ~ 1/2    
-Xss大小 设置线程栈大小 -Xss1m =-XX:ThreadStackSize  
```
###### 不稳定参数
```
-XX:ErrorFile=文件 设置错误日志路径 -XX:ErrorFile=./hs_err_pid%p.log %p为当前进程号    
-XX:OnError=命令 错误发生时执行命令 -XX:OnError="gcore %p;dbx - %p"    
-XX:OnOutOfMemoryError=命令 内存溢出时执行命令    
-XX:MaxDirectMemorySize=size 设置直接内存最大值 -XX:MaxDirectMemorySize=100m 默认为0 当直接内存达到设置的最大值会FullGC    
-XX:ObjectAlignmentInBytes=alignment 设置java对象的内存对齐，默认是8字节    
-XX:ThreadStackSize 设置线程栈大小 -XX:ThreadStackSize=1m = -Xss    
-XX:-UseBiasedLocking 禁用偏向锁 默认开启，不禁用 如果使用的是大量的没有竞争的同步，使用偏向锁会提升性能    
-XX:-UseCompressedOops 禁用压缩指针堆内存小于32G时默认开启 开启后，对象引用是32位而不是64位，可以提升性能。只有64位的jvm才生效    
-XX:+DoEscapeAnalysis 开启逃逸分析 默认开启    
-XX:+Inline 开启方法内联 默认开启    
-XX:InlineSmallCode=大小 设置应内联的已编译方法的最大代码大小，只有小于此数值的才会内联 默认1000字节  
-XX:MaxInlineSize=大小 设置要内联方法的最大字节码大小 默认35字节    
-XX:+OptimizeStringConcat 开启字符串连接优化 默认开启    
-XX:+PrintInlining 打印方法内联 默认关闭，需和-XX:+UnlockDiagnosticVMOptions 一起使用    
-XX:-TieredCompilation 关闭分层编译 默认开启    
-XX:+HeapDumpOnOutOfMemoryError  OOM时堆内存dump到当前目录    
-XX:HeapDumpPath=路径 设置堆内存dump的路径 -XX:HeapDumpPath=/var/java_pid%p.hprof    
-XX:+UnlockDiagnosticVMOptions 开启jvm诊断功能选项    
```
###### 垃圾回收相关参数
```
-XX:+AggressiveHeap 开启堆最优化设置 默认关闭
-XX:+CMSClassUnloadingEnabled 当使用CMS垃圾收集器时，允许类卸载 默认开启
-XX:CMSExpAvgFactor=percent 指定垃圾收集消耗的时间百分比 默认这个数是25%，就是25
-XX:CMSInitiatingOccupancyFraction=percent 设置CMS回收开始的老年代百分比 默认-1，任何的负值表示会使用-XX:CMSTriggerRatio选项来定义这个百分比数
-XX:+CMSScavengeBeforeRemark 在CMS重新标记之前执行ygc操作  默认关闭 在remark时间过长时可以开启；开启减少remark的STW时间
-XX:CMSTriggerRatio=percent 设置CMS开始的百分比 默认80，((100 - MinHeapFreeRatio) + (double)( CMSTriggerRatio * MinHeapFreeRatio) / 100.0) / 100.0=92%
-XX:+UseCMSCompactAtFullCollection  在FULL GC的时候，对年老代的压缩
-XX:CMSFullGCsBeforeCompaction=0  上面配置开启的情况下,这里设置多少次Full GC后,对年老代进行压缩，
-XX:MinHeapFreeRatio=percent GC之后堆内存最小剩余百分比，如果小于此值，则自动扩容 默认40%
-XX:MaxHeapFreeRatio=percent GC之后堆内存最大剩余百分比，如果小于此值，则自动缩容 默认70%
-XX:ParallelGCThreads=threads 设置Parallel GC的线程数 默认根据cpu个数，&lt;=8,则使用8个，&gt;8个3+5N/81台服务器只有1个jvm时使用默认值较好，如果有n个jvm，cpu个数 / n比较合适
-XX:ConcGCThreads=个数 并发GC的线程数 默认值取决于cpu个数 ConcGCThreads = (ParallelGCThreads + 3)/4
-XX:+DisableExplicitGC 使System.gc()显式gc失效 默认不开启，
-XX:G1HeapRegionSize=size 当使用G1收集器时，设置java堆被分割的region大小 1M~32M默认根据堆内存最优化设置
-XX:+G1PrintHeapRegions 打印G1收集器收集的区域  默认关闭
-XX:G1ReservePercent=percent  设置堆内存保留大小，以防晋升失败 0~50 默认10%
-XX:InitialHeapSize=size 堆初始大小 =-Xms
-XX:MaxHeapSize=size 堆最大大小 =-Xmx
-XX:InitialSurvivorRatio=ratio 设置伊甸园区和幸存区初始比例 默认为8:1
-XX:SurvivorRatio=ratio 设置伊甸园区和幸存区比例 默认为8:1  8:1:1
XX:TargetSurvivorRatio YGC之后，幸存区期望百分比 默认 50%
XX:InitiatingHeapOccupancyPercent=percent 堆占用达到多少开始并发垃圾回收 只有并发垃圾回收器生效
-XX:MaxGCPauseMillis=time GC最大暂停时间 ms 默认没有最大暂停时间
-XX:MetaspaceSize=size 元空间多次扩容后超过此值就会full gc 默认大约20M 元空间使用本地内存。 尽量设置足够，避免因为这个区域内存不够引发Full GC
-XX:MaxMetaspaceSize=size 设置元空间最大大小 默认不限制 建议和MetaspaceSize一样大，一般256M
-XX:NewSize=size 设置年轻代初始大小 建议年轻代占堆大小的1/4 ~ 1/2
-XX:MaxNewSize=size 设置年轻代最大大小 默认根据最大效能分配
-XX:MaxTenuringThreshold=threshold 最大晋升年龄，从年轻代到老年代 默认：15 - 并行回收器 6 - CMS
-XX:NewRatio=ratio 设置老年代和新生代比例 默认2 老年代: （伊甸园 + 2个幸存区）
-XX:+PrintGC 打印GC信息 默认关闭
-XX:+PrintGCDetails 打印GC详细信息 默认关闭
-XX:+PrintTenuringDistribution 打印晋升分配
-XX:+ScavengeBeforeFullGC Full gc之前先ygc 默认开启
-XX:+UseTLAB 年轻代使用线程局部缓存 默认开启  效率高
-XX:TLABSize=size 设置初始化thread-local allocation buffer (TLAB)大小
-XX:+UseAdaptiveSizePolicy  JDK 1.8 默认使用 UseParallelGC 垃圾回收器，该垃圾回收器默认启动了 AdaptiveSizePolicy
-XX:+UseParallelGC 年轻代使用并行回收器
-XX:+UseParallelOldGC 老年代使用并行回收器
-XX:+UseParNewGC 为配置CMS，年轻代使用ParNew回收器，CMS会默认开启
-XX:+UseConcMarkSweepGC 使用CMS回收器
-XX:+UseG1GC 使用G1回收器
-XX:+UseGCOverheadLimit 限制GC的运行时间，通过统计GC时间来预测是否要OOM了，提前抛出异常，防止OOM发生 并行/并发回收器在GC回收时间过长时会抛出OutOfMemroyError。过长的定义是，超过98%的时间用来做GC并且回收了不到2%的堆内存。用来避免内存过小造成应用不能正常工作
-XX:+UseStringDeduplication 开启字符串去重 G1回收器生效
-XX:AutoBoxCacheMax=20000 加大Integer Cache
-XX:+PrintPromotionFailure 知道是多大的新生代对象晋升到老生代失败从而引发Full GC时的。
```
###### G1垃圾回收器JVM参数样例

```
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=4M
-XX:MaxTenuringThreshold=12
-XX:InitiatingHeapOccupancyPercent=40
-XX:ConcGCThreads=4 当前核心数的一半
-XX:+UseStringDeduplication
-XX:AutoBoxCacheMax=20000
-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m
-XX:+PrintPromotionFailure
-XX:+HeapDumpOnOutOfMemoryErro
-XX:HeapDumpPath=${LOGDIR}/java_pid${pid}.hprof
-Xloggc:/data/log/gc-myapp.log 
-verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails
-XX:+DisableExplicitGC 
```
##### 常用命令：

```
查询进程中各个线程占用CPU的百分比
ps -mp [PID] -o THREAD,tid,time

将进程号转换为16禁止
printf “%x\n” [tid]

打印线程的堆栈信息
jstack pid | grep [tid] -A 30

查看内存回收情况
jstat -gcutil [pid] 250 6

查看当前java进程中创建的活跃对象的数目和占用内存的大小
jmap -histo:live [PID] > a.log

导出当前java进程的内存占用情况
jmap -dump:live,format=b,file=[filePath] [PID]
jmap -F -dump,format=b,file=20210426-24254.hprof 24254

ClassName 代表说明

C:char
D:double
F:float
I:int
J:long
Z:boolean
[:数组，如[I表示<span class="hljs-keyword">int</span>[]
[L+类名:其他对象
```
