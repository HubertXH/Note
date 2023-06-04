## G1-日志说明

JVM中GC日志的参数配置

| 参数                                    | 说明                        | 
|---------------------------------------|---------------------------|
| -Xloggc:/path/to/gc.log               | 日志存储路径                    | 
| -XX:+UseGCLogFileRotation             | 是否开启GC日志自动切换(true /false) | 
| -XX:NumberOfGCLogFiles=<value>        | GC 日志保留的数量(num)           | 
| -XX:GCLogFileSize=<size>              | GC日志切换的大小(100M)           | 
| -XX:+PrintGCDetails                   | 打印详细GC日志                  | 
| -XX:+PrintGCDateStamps                | 打印GC发生的真实日期及时间            | 
| -XX:+PrintGCApplicationStoppedTime    | 打印在垃圾回收期间应用停顿的时长          | 
| -XX:+PrintGCApplicationConcurrentTime | 打印GC时程序的运行时间              | 
| -XX:-PrintCommandLineFlags            | 在日志中打印所有命令标识              | 

```
2023-04-27T16:02:10.150+0800: 2203.486: [GC pause (G1 Evacuation Pause) (young), 0.0041826 secs] -- ①
   [Parallel Time: 2.8 ms, GC Workers: 10] -- ②
      [GC Worker Start (ms): Min: 2203486.0, Avg: 2203486.0, Max: 2203486.0, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.7, Avg: 0.9, Max: 2.0, Diff: 1.3, Sum: 9.0]
      [Update RS (ms): Min: 0.0, Avg: 0.4, Max: 0.6, Diff: 0.6, Sum: 4.3]
         [Processed Buffers: Min: 0, Avg: 12.6, Max: 23, Diff: 23, Sum: 126]
      [Scan RS (ms): Min: 0.0, Avg: 0.1, Max: 0.1, Diff: 0.1, Sum: 1.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.2]
      [Object Copy (ms): Min: 0.6, Avg: 1.1, Max: 1.2, Diff: 0.6, Sum: 11.4]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 10]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.5]
      [GC Worker Total (ms): Min: 2.6, Avg: 2.7, Max: 2.7, Diff: 0.1, Sum: 26.5]
      [GC Worker End (ms): Min: 2203488.6, Avg: 2203488.6, Max: 2203488.7, Diff: 0.1]
   [Code Root Fixup: 0.2 ms] -- ③
   [Code Root Purge: 0.0 ms] -- ④
   [Clear CT: 0.2 ms] -- ⑤
   [Other: 1.0 ms] -- ⑥
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.2 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.4 ms]
   [Eden: 1214.0M(1214.0M)->0.0B(1212.0M) Survivors: 14.0M->16.0M Heap: 1531.6M(2048.0M)->318.7M(2048.0M)] -- ⑦
 [Times: user=0.02 sys=0.00, real=0.00 secs] -- ⑧ 
```
- ① 
  - ***2023-04-27T16:02:10.150+0800***:GC发生的实际时间
  - ***2203.486***:距离JVM启动的时间
  - ***(G1 Evacuation Pause) (young)***：GC类型 young GC
  - ***0.0041826 secs***:此次GC耗费时间
- ②
  - ***Parallel Time: 2.8 ms***:此次GC 执行期间发送的Stop The Word的时间，从垃圾收集开始直到最后一个线程结束为止
  - ***GC Workers: 10***: GC工作线程数，可以使用参数：XX:ParallelGCThreads来设置。默认情况下为CPU的数量，对于8核以上的CPU默认值为CPU数量的5/8
  - ***GC Worker Start***:GC线程开始工作的时间-距离JVM启动的时间(毫秒值),理想情况下所有的线程应该同一时间开始工作，Diff应该为0
  - ***Ext Root Scanning***:扫描根节点，找到任何能够进入当前收集的对象
  - 
  - 
