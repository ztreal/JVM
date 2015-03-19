# 变量在运行时内存的位置
使用HSDB来查看JAVA运行时内存的情况

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
➜  java  jdb
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


