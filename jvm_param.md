# JVm参数配置
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

为什么不建议<=3G的情况下使用CMS GC
http://hellojava.info/?p=142


jvm crash分析参考
http://itindex.net/detail/41705-tomcat-fatal-error

参数含义
http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html
