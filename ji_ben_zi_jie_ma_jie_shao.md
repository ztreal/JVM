# 基本字节码介绍



###基本概念
u1,u2 和 u4,分别代 表了 1、2 和 4 个字节的无符号数



#### 类型
i 代表对 int 类型的数据操作,l 代表 long,s 代表 short,b 代表 byte, c 代表 char,f 代表 float,d 代表 double,a 代表 reference。也有一些指令的助记符中没 有明确的指明操作类型的字母
#### 操作
加载和存储指令用于将数据从栈帧(§2.6)的局部变量表(§2.6.1)和操作数栈之间来回 传输
* 将一个局部变量加载到操作栈的指令包括有Xload
* 将一个数值从操作数栈存储到局部变量表 Xstore
* 将一个常量加载到操作数栈 Xpush Xconst
* 扩充局部变量表 wide




* 问题：虚拟机如何识别此字节码文件是java文件，并且判断当前虚拟机 可以支持执行此JDK版本编译的java字节码文件?
* 问题：程序报错的时候如何知道对应java代码是哪一行出的错呢？


字节码格式
```
ClassFile {
   u4 magic;
   u2 minor_version;
   u2 major_version;
   u2 constant_pool_count;
   cp_info constant_pool[constant_pool_count-1];
   u2 access_flags;
   u2 this_class;
   u2 super_class;
   u2 interfaces_count;
   u2 interfaces[interfaces_count];
   u2 fields_count;
   field_info fields[fields_count];
   u2 methods_count;
   method_info methods[methods_count];
   u2 attributes_count;
   attribute_info attributes[attributes_count];
}
```

类
```
public class LoopClass {
    public static void main(String[] args) {
        int i;
        for (i = 0; i < 100; i++) {
            ;
        }
    }
}
```


编译后的字节码为
```
Classfile /Users/zhangtan/github/zt-example/zt-example-util/src/main/java/LoopClass.class
  Last modified 2015-3-3; size 310 bytes
  MD5 checksum 7c7c7d28fd342023356b0aac31cfda0d
  Compiled from "LoopClass.java"
public class LoopClass
  SourceFile: "LoopClass.java"
  minor version: 0
  major version: 50
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #3.#13         //  java/lang/Object."<init>":()V
   #2 = Class              #14            //  LoopClass
   #3 = Class              #15            //  java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               main
   #9 = Utf8               ([Ljava/lang/String;)V
  #10 = Utf8               StackMapTable
  #11 = Utf8               SourceFile
  #12 = Utf8               LoopClass.java
  #13 = NameAndType        #4:#5          //  "<init>":()V
  #14 = Utf8               LoopClass
  #15 = Utf8               java/lang/Object
{
  public LoopClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iload_1
         3: bipush        100
         5: if_icmpge     14
         8: iinc          1, 1
        11: goto          2
        14: return
      LineNumberTable:
        line 4: 0
        line 7: 14
      StackMapTable: number_of_entries = 2
           frame_type = 252 /* append */
          offset_delta = 2
          locals = [ int ]
             frame_type = 11 /* same */

  }
```
上面是使用javap查看的，使用二进制查看文件开头会显示
![](http://7vzu3j.com1.z0.glb.clouddn.com/magic.png)
[0xCAFEBABE](http://zh.wikipedia.org/wiki/Hexspeak)（“cafebabe”）在Mach-O格式文件中用于标识通用二进制目标文件，同时也在Java中用于识别Java字节码类文件。

*
minor_vesion、major_version
分别表示 Class 文件 的副、主版本。一个 Java 虚拟机实例只能支持特定范围内的主版本号(Mi 至 Mj)和 0 至特定范围 内(0 至 m)的副版本号。
此版本号jdk版本生成，例如上图的例子使用JDK1.6，出来的版本号为50。

下面写个JDK1.8特性的例子
```
public class Test {
    public static void main(String[] args) {
        //这里有{}和return 以及 ;
        Runnable r = () -> { System.out.println("hello world"); };

        //这里不需要{}和return
        java.util.Comparator<String> c = (String s1, String s2) -> s2.length()-s1.length();
        r.run();
        System.out.println(c.compare("s1", "12323"));
    }

}

```
查看变以后结果
```
➜  java git:(master) ✗ javap -verbose LoopClass
Classfile /Users/zhangtan/github/zt-example/zt-example-util/src/main/java/LoopClass.class
  Last modified 2015-3-3; size 310 bytes
  MD5 checksum c9a2bea90f79a07ba64bd0cb0dbaf7db
  Compiled from "LoopClass.java"
public class LoopClass
  SourceFile: "LoopClass.java"
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #3.#13         //  java/lang/Object."<init>":()V
   #2 = Class              #14            //  LoopClass
   #3 = Class              #15            //  java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               main
   #9 = Utf8               ([Ljava/lang/String;)V
  #10 = Utf8               StackMapTable
  #11 = Utf8               SourceFile
  #12 = Utf8               LoopClass.java
  #13 = NameAndType        #4:#5          //  "<init>":()V
  #14 = Utf8               LoopClass
  #15 = Utf8               java/lang/Object
{
  public LoopClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iload_1
         3: bipush        100
         5: if_icmpge     14
         8: iinc          1, 1
        11: goto          2
        14: return
      LineNumberTable:
        line 4: 0
        line 7: 14
      StackMapTable: number_of_entries = 2
           frame_type = 252 /* append */
          offset_delta = 2
          locals = [ int ]
             frame_type = 11 /* same */

  }
```
####magic

JDK 版本为 1. k 时(k ≥2)时,对应的 Class 文件格式版本号的范围是 45.0 至 44+k.0
major version是52，上面一个例子为50

如果jvm不支持，则会报错 ![](http://7vzu3j.com1.z0.glb.clouddn.com/minorVersionError.png)


####fags:ACC_PUBLIC, ACC_SUPER
ACC_PUBLIC 可以被包的类外访问
ACC_SUPER 当用到 invokespecial 指令时,需要特殊处理3的 父类方法。
￼

####Constant pool
常量池,如例子所示，共有15个常量(从1开始计数)  0010 = 16-1个
![](http://7vzu3j.com1.z0.glb.clouddn.com/constant_pool.png)￼￼
常量池内容可以在HSDB中查看![](http://7vzu3j.com1.z0.glb.clouddn.com/constants_HSDB.png)￼

####其他属性讲解
stack=2, locals=2, args_size=1
stack为栈的深度
locals为存储空间单位为Slot，一个Slot能存储32bit的数据类型、引用
args_size为参数个数
*LineNumberTable
标记源文件中行号表示的内容在 Java 虚拟机的 code[]数组中对应的部分
*tackMapTable
属性包󿰄 0 至多个栈映射帧(Stack Map Frames),每个栈映射帧都显 式或隐式地指定了一个字节码偏移量,用于表示局部变量表和操作数栈的验证类型
￼可以被包的类外访问。
*invokespecial
指令用于调用一些需要特殊处理的实例方法,包括实例初始化方法(§ 2.9)、私有方法和父类方法。
*StackMapTable
StackMapTable 属性包󿰄 0 至多个栈映射帧(Stack Map Frames),每个栈映射帧都显 式或隐式地指定了一个字节码偏移量,用于表示局部变量表和操作数栈的验证类型
*SourceFile
sourcefile_index 项引用字符串表示被编译的 Class 文件的源文件的名字。不包括 源文件所在目录的目录名,也不包括源文件的绝对路径名。
平台相关的绝对路径等)的 附加信息必须是运行时解释器(Runtime Interpreter)或开发工具在文件名实际使 用时提供。
*LocalVariableTable
记录所有的本地变量在帧中的位置

####执行步骤
```
         0: iconst_0
         1: istore_1
         2: iload_1
         3: bipush        100
         5: if_icmpge     14
         8: iinc          1, 1
        11: goto          2
        14: return
```
*
问题：3: bipush 后面为什么是5: if_icmpge，4去哪里了呐？
因为此命令会占用两个字节，一个字节用于存储操作码bipush，另一个存储操作数100

下面是对执行步骤的讲解
![](http://7vzu3j.com1.z0.glb.clouddn.com/opscode.png)




常量池tag
![](http://7vzu3j.com1.z0.glb.clouddn.com/常量池tag.png)

常量池表类型
![](http://7vzu3j.com1.z0.glb.clouddn.com/常量池.png)