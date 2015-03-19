# JVm参数配置
##配置讲解
先看一个线上配置
```
#!/bin/sh

export TOMCAT_USER="tomcat"
export JAVA_OPTS="-Xms3072m -Xmx3072m -XX:NewSize=675m -XX:PermSize=256m -XX:MaxPermSize=256M -server -XX:+DisableExplicitGC  -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGC -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=1 -XX:+PrintSafepointStatistics -XX:SurvivorRatio=8  -Xloggc:$CATALINA_BASE/logs/gc.log "
chown -R tomcat:tomcat $CATALINA_BASE/logs
chown -R tomcat:tomcat $CATALINA_BASE/cache
chown -R tomcat:tomcat $CATALINA_BASE/conf
chown -R tomcat:tomcat $CATALINA_BASE/work
chown -R tomcat:tomcat $CATALINA_BASE/temp
```
-Xms3072m -Xmx3072m 使堆的最大最小内存一致，避免垃圾回收后重新分配内存

-XX:NewSize=675m  设置新生代的大小

-XX:PermSize=256m 设置永久代大小

-server 以server模式启动

-XX:+DisableExplicitGC 禁用System.gc()
使用nio的情况下，此参数会引发问题，因为nio中的bits类的reserveMemory方法使用了System.gc()，如果禁用，会引发oom

-XX:+UseConcMarkSweepGC 使用cms收集器

-XX:CMSFullGCsBeforeCompaction=1 一次full gc后对内存进行压缩  默认是0，即每次cms gc转入ful gc的时候都启用压缩

java –verbose:gc 虚拟机发生内存回收的时候输出设备显示信息

-XX:SurvivorRatio=8  设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10


为什么不建议<=3G的情况下使用CMS GC
http://hellojava.info/?p=142


jvm crash分析参考
http://itindex.net/detail/41705-tomcat-fatal-error

参数含义
http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html

##日志查看
下面看个例子，首先jvm参数配置为
```
export JAVA_OPTS="-Xms2048m -Xmx2048m -XX:NewSize=256m -XX:PermSize=256m -server -XX:+DisableExplicitGC -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:$CATALINA_BASE/logs/gc.log"
```
输出日志内容为
```
2015-03-13T15:02:18.056+0800: 67361.219: [GC [PSYoungGen: 696640K->192K(697856K)] 1126663K->430247K(2096128K), 0.0074560 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
```
[GC [PSYoungGen: 696640K->192K(697856K)]：
为gc类型:gc前内存占用大小->gc后内存占用大小(新生代总内存大小)
1126663K->430247K(2096128K), 0.0074560 secs]:
gc前堆大小->gc后堆大小(堆总大小)

为什么
HotSpot VM在显示某个空间的capacity（容量）的时候，如果里面包含有survivor space，则会减掉一个survivor space的大小（具体来说是减掉了当前to space的大小；由于from和to的大小不一定总是完全一样大，提一下具体减去的是哪个survivor space的大小还是有必要的）。
所以YoungGen的capacity是eden()->capacity() + from()->capacity()，
而整个Java heap的capacity是YoungGen的capacity + OldGen的capacity。

jvm配置
```
export JAVA_OPTS="-Xms3072m -Xmx3072m -XX:NewSize=675m -XX:PermSize=256m -XX:MaxPermSize=256M -server -XX:+DisableExplicitGC  -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGC -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=1 -XX:+PrintSafepointStatistics -XX:SurvivorRatio=8  -Xloggc:$CATALINA_BASE/logs/gc.log "
```
*
一次CMS为什么算两次FGC？
因为initial mark与final re-mark会stop the world 2次一共
![](http://7vzu3j.com1.z0.glb.clouddn.com/CMS.png)
参考：http://rednaxelafx.iteye.com/blog/1108768

## JVM参数优化

gc日志结果
```
2015-03-16T11:10:02.291+0800: 117373.544: [GC 117373.545: [ParNew (promotion failed): 622080K->622080K(622080K), 0.3934210 secs]117373.938: [CMS2015-03-16T11:10:02.940+0800: 117374.193: [CMS-concurrent-mark: 0.319/0.802 secs] [Times: user=5.09 sys=0.19, real=0.81 secs]
 (concurrent mode failure): 2145434K->751705K(2454528K), 2.9668350 secs] 2683000K->751705K(3076608K), [CMS Perm : 107533K->105533K(262144K)], 3.3615860 secs] [Times: user=5.22 sys=0.07, real=3.36 secs]
Total time for which application threads were stopped: 3.3715080 seconds
Application time: 0.0006130 seconds
Total time for which application threads were stopped: 0.0589920 seconds
Application time: 0.1627490 seconds
```
发现promotion failed 和 concurrent mode failure ，前者是进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的，后者是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的。


查看当前进程的gc情况使用命令
``` sudo /home/q/java/default/bin/jstat -gcutil  7967  1000```
相关命令说明 http://docs.oracle.com/javase/1.5.0/docs/tooldocs/share/jstat.html
结果如下
```
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
100.00   0.00  38.86  37.29  23.41  10496  102.379     1    0.375  102.754
100.00   0.00  44.30  37.29  23.41  10496  102.379     1    0.375  102.754
100.00   0.00  50.39  37.29  23.41  10496  102.379     1    0.375  102.754
100.00   0.00  55.14  37.29  23.41  10496  102.379     1    0.375  102.754
100.00   0.00  63.48  37.29  23.41  10496  102.379     1    0.375  102.754
100.00   0.00  72.32  37.29  23.41  10496  102.379     1    0.375  102.754
```
S0 Survivor0区域使用空间百分比
S1 Survivor1区域使用空间百分比
E  Eden已经使用百分比
O  old已经使用百分比
P  Perm使用百分比
YGC  启动以来Young GC次数
YGCT 启动以来Young GC总耗时秒
FGC  Full GC总次数
FGCT Full GC总耗时
GCT GC总耗时

dump内存快照java/bin/jmap -dump:format=b,file=dump.bin pid
使用MAT工具分析


