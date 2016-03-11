# 对象在运行时内存的位置
使用HSDB来查看JAVA运行时内存的情况

* 怎么查看jdk8，已经没有perm gen？

HSDB中执行universe查看输出结果
```
hsdb> universe
Heap Parameters:
ParallelScavengeHeap [ PSYoungGen [ eden =  [0x000000076ab00000,0x000000076ab78ec8,0x000000076ab80000] , from =  [0x000000076ab80000,0x000000076abf0000,0x000000076ac00000] , to =  [0x000000076ac00000,0x000000076ac00000,0x000000076ac80000]  ] PSOldGen [  [0x00000006c0000000,0x00000006c0000000,0x00000006cfe80000]  ]  ] hsdb>
```
对比在jdk7中执行的结果输出
```
Heap Parameters:
ParallelScavengeHeap [ PSYoungGen [ eden =  [0x00000007bab00000,0x00000007bb440f00,0x00000007bb600000] , from =  [0x00000007bb980000,0x00000007bbbd2288,0x00000007bbc80000] , to =  [0x00000007bb600000,0x00000007bb600000,0x00000007bb980000]  ] PSOldGen [  [0x00000007b0000000,0x00000007b11013c0,0x00000007b1700000]  ]  ] hsdb>
hsdb> scanoops 0x00000007bab00000 0x00000007b1700000 VarInJVM2
No such type.
hsdb>  scanoops 0x00000007bab00000 0x00000007b1700000 VarInJVM

```


## 查看对象在内存中的位置



首先编写
```
public class Main {
    public static void main(String[] args) {
        VarInJVM test = new VarInJVM();
        test.fn();
    }
}
```

```
public class VarInJVM {
    static VarInJVM2 t1 = new VarInJVM2();
    VarInJVM2 t2 = new VarInJVM2();

    public void fn() {
        VarInJVM2 t3 = new VarInJVM2();
    }
}

class VarInJVM2 {

}

```
运行命令
```
➜  jdb   -XX:+UseSerialGC
正在初始化jdb...
> stop in VarInJVM.fn
正在延迟断点VarInJVM.fn。
将在加载类后设置。
> run Main
运行 Main
设置未捕获的java.lang.Throwable
设置延迟的未捕获的java.lang.Throwable
>
VM 已启动: 设置延迟的断点VarInJVM.fn

断点命中: "线程=main", VarInJVM.fn(), 行=11 bci=0
11            VarInJVM2 t3 = new VarInJVM2();

main[1] next
>
已完成的步骤: "线程=main", VarInJVM.fn(), 行=12 bci=8
12        }

main[1]
```
不加UseSerialGC的话，可能查看虚拟内存地址的时候会有问题
查看线程pid
```
➜  java  jps
5474 TTY
3412 RemoteJdbcServer
379
509 RemoteMavenServer
5503 Main
5567 Jps
```
启动HSDB
```
sudo java -cp /Library/Java/JavaVirtualMachines/jdk1.8.0.jdk/Contents/Home/lib/sa-jdi.jar sun.jvm.hotspot.HSDB
```
注意需要sudo权限

在HSDB中只执行
```
hsdb> universe
Heap Parameters:
ParallelScavengeHeap [ PSYoungGen [ eden =  [0x00000007bab00000,0x00000007bb440f00,0x00000007bb600000] , from =  [0x00000007bb980000,0x00000007bbbd2288,0x00000007bbc80000] , to =  [0x00000007bb600000,0x00000007bb600000,0x00000007bb980000]  ] PSOldGen [  [0x00000007b0000000,0x00000007b11013c0,0x00000007b1700000]  ]  ] hsdb>
hsdb> scanoops 0x00000007bab00000 0x00000007b1700000 VarInJVM2
No such type.
hsdb>  scanoops 0x00000007bab00000 0x00000007b1700000 VarInJVM
```

然后在1.7环境下查找对象所在位置

```
hsdb> scanoops 0x00000007aaa80000 0x00000006fc300000 VarInJVM2
hsdb> scanoops 0x00000007aaa80000 0x00000007aeb00000 VarInJVM2
0x00000007aab2c330 VarInJVM2
0x00000007aab2c350 VarInJVM2
0x00000007aab2c360 VarInJVM2
```
找到了3个对象，然后执行whatis的时候挂了。。。出不来东西。然后使用open jdk的再尝试
```
 sudo /Users/zhangtan/develop/openjdk7-24.80-b04-fastdebug/bin/java -cp /Users/zhangtan/develop/openjdk7-24.80-b04-fastdebug/lib/sa-jdi.jar sun.jvm.hotspot.HSDB
```
唉，open jdk的1.7和1.8的都不好使，各种报错。下面去linux下面的jdk试试

查找yum安装的文件
rpm -qa |grep java
rpm -ql java-1.7.0-openjdk-devel-1.7.0.75-2.5.4.2.el7_0.x86_64

linux下的openjdk 也不行，太坑了
下了个jdk，终于成功了！！！！

检查虚拟内存中对象位置
```
hsdb> universe
Heap Parameters:
Gen 0:   eden [0x00000000fa400000,0x00000000fa460a90,0x00000000fa6b0000) space capacity = 2818048, 14.049441315406977 used
  from [0x00000000fa6b0000,0x00000000fa6b0000,0x00000000fa700000) space capacity = 327680, 0.0 used
  to   [0x00000000fa700000,0x00000000fa700000,0x00000000fa750000) space capacity = 327680, 0.0 usedInvocations: 0

Gen 1:   old  [0x00000000fa750000,0x00000000fa750000,0x00000000fae00000) space capacity = 7012352, 0.0 usedInvocations: 0

  perm [0x00000000fae00000,0x00000000fb066fa0,0x00000000fc2c0000) space capacity = 21757952, 11.577119022966867 usedInvocations: 0
hsdb> scanoops 0x00000000fa400000 0x00000000fc2c0000 VarInJVM2
0x00000000fa45b4b0 VarInJVM2
0x00000000fa45b4d0 VarInJVM2
0x00000000fa45b4e0 VarInJVM2
```
查看内存分配
```
hsdb> whatis 0x00000000fa45b4b0
Address 0x00000000fa45b4b0: In thread-local allocation buffer for thread "main" (25062)  [0x00000000fa452e38,0x00000000fa45b4f0,0x00000000fa460a80)

hsdb> whatis 0x00000000fa45b4d0
Address 0x00000000fa45b4d0: In thread-local allocation buffer for thread "main" (25062)  [0x00000000fa452e38,0x00000000fa45b4f0,0x00000000fa460a80)

hsdb> whatis 0x00000000fa45b4e0
Address 0x00000000fa45b4e0: In thread-local allocation buffer for thread "main" (25062)  [0x00000000fa452e38,0x00000000fa45b4f0,0x00000000fa460a80)
```
检查对象内容
```
hsdb> inspect 0x00000000fa45b4b0
instance of Oop for VarInJVM2 @ 0x00000000fa45b4b0 @ 0x00000000fa45b4b0 (size = 16)
_mark: 1
```
查看内容
```
hsdb> mem 0x00000000fa45b4b0 2
0x00000000fa45b4b0: 0x0000000000000001
0x00000000fa45b4b8: 0x00000000fb066d30
```

Klass内容如下图
![](http://7vzu3j.com1.z0.glb.clouddn.com/Klass.png)


由于java对象是指针引用，所以找到上面三个对象引用的指针就可以了。
首先找第一个对象的指针引用，看哪里引用他
```
hsdb> revptrs 0x00000000fa45b4b0
Computing reverse pointers...
Done.
null
Oop for java/lang/Class @ 0x00000000fa45a488
```
然后查看引用所在的位置
```
hsdb> whatis 0x00000000fa45a488
Address 0x00000000fa45a488: In thread-local allocation buffer for thread "main" (25062)  [0x00000000fa452e38,0x00000000fa45b4f0,0x00000000fa460a80)
```
查看这个引用的内容
```
hsdb> inspect 0x00000000fa45a488
instance of Oop for java/lang/Class @ 0x00000000fa45a488 @ 0x00000000fa45a488 (size = 96)
<<Reverse pointers>>:
t1: Oop for VarInJVM2 @ 0x00000000fa45b4b0 Oop for VarInJVM2 @ 0x00000000fa45b4b0
```
根据上面内容可以发现引用这个指针的对象中包含一个VarInJVM2对象的引用。

至此我们找打了t1的位置了终于。t1在eden区域中，main线程的tlab中。

查找第二个实例的位置，以及内容
```
hsdb> revptrs 0x00000000fa45b4d0
Oop for VarInJVM @ 0x00000000fa45b4c0
hsdb> whatis 0x00000000fa45b4c0
Address 0x00000000fa45b4c0: In thread-local allocation buffer for thread "main" (25062)  [0x00000000fa452e38,0x00000000fa45b4f0,0x00000000fa460a80)

hsdb> inspect 0x00000000fa45b4c0
instance of Oop for VarInJVM @ 0x00000000fa45b4c0 @ 0x00000000fa45b4c0 (size = 16)
<<Reverse pointers>>:
_mark: 1
t2: Oop for VarInJVM2 @ 0x00000000fa45b4d0 Oop for VarInJVM2 @ 0x00000000fa45b4d0

```
找到了t2，是作为VarInJVM的成员变量存在。


t3命令有bug找不到了，根据图片![](http://7vzu3j.com1.z0.glb.clouddn.com/t3Frame.png)可以看到 t3在栈帧里面

常量池内容和字节码是对应的，参见下图
![](http://7vzu3j.com1.z0.glb.clouddn.com/constants_HSDB.png)


main线程的栈内存信息
![](http://7vzu3j.com1.z0.glb.clouddn.com/stackFrame.png)

