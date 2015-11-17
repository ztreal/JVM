# 语法糖介绍

## 常用语法糖示例

### 泛型

### 自动拆箱/装箱

### 循环历遍

### 多条件


### 枚举




### 变长参数




### switch支持字符串

java代码
```
public class SwitchTest {

    public void test(String condition){
        switch (condition){
            case "a":
                System.out.println("a");
        }
    }

}

```
条件为int时候的字节码

```
Classfile /Users/zhangtan/github/zt-example/zt-example-util/src/main/java/SwitchTest.class
  Last modified 2015-3-11; size 447 bytes
  MD5 checksum 594966c5d284103833088deef012a577
  Compiled from "SwitchTest.java"
public class SwitchTest
  SourceFile: "SwitchTest.java"
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#16         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #17.#18        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #19            //  a
   #4 = Methodref          #20.#21        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #22            //  SwitchTest
   #6 = Class              #23            //  java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               test
  #12 = Utf8               (I)V
  #13 = Utf8               StackMapTable
  #14 = Utf8               SourceFile
  #15 = Utf8               SwitchTest.java
  #16 = NameAndType        #7:#8          //  "<init>":()V
  #17 = Class              #24            //  java/lang/System
  #18 = NameAndType        #25:#26        //  out:Ljava/io/PrintStream;
  #19 = Utf8               a
  #20 = Class              #27            //  java/io/PrintStream
  #21 = NameAndType        #28:#29        //  println:(Ljava/lang/String;)V
  #22 = Utf8               SwitchTest
  #23 = Utf8               java/lang/Object
  #24 = Utf8               java/lang/System
  #25 = Utf8               out
  #26 = Utf8               Ljava/io/PrintStream;
  #27 = Utf8               java/io/PrintStream
  #28 = Utf8               println
  #29 = Utf8               (Ljava/lang/String;)V
{
  public SwitchTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public void test(int);
    descriptor: (I)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: iload_1
         1: lookupswitch  { // 1
                       1: 20
                 default: 28
            }
        20: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        23: ldc           #3                  // String a
        25: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        28: return
      LineNumberTable:
        line 4: 0
        line 6: 20
        line 8: 28
      StackMapTable: number_of_entries = 2
           frame_type = 20 /* same */
           frame_type = 7 /* same */

}

```
上面看来switch还是比较简单的，直接根据值，如果是1跳转到#20，否则跳转到#28


字节码
```
Classfile /Users/zhangtan/github/zt-example/zt-example-util/src/main/java/SwitchTest.class
  Last modified 2015-3-11; size 585 bytes
  MD5 checksum 671e045b71d04359530af0c4434e265a
  Compiled from "SwitchTest.java"
public class SwitchTest
  SourceFile: "SwitchTest.java"
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #8.#19         //  java/lang/Object."<init>":()V
   #2 = Methodref          #20.#21        //  java/lang/String.hashCode:()I
   #3 = String             #22            //  a
   #4 = Methodref          #20.#23        //  java/lang/String.equals:(Ljava/lang/Object;)Z
   #5 = Fieldref           #24.#25        //  java/lang/System.out:Ljava/io/PrintStream;
   #6 = Methodref          #26.#27        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #7 = Class              #28            //  SwitchTest
   #8 = Class              #29            //  java/lang/Object
   #9 = Utf8               <init>
  #10 = Utf8               ()V
  #11 = Utf8               Code
  #12 = Utf8               LineNumberTable
  #13 = Utf8               test
  #14 = Utf8               (Ljava/lang/String;)V
  #15 = Utf8               StackMapTable
  #16 = Class              #30            //  java/lang/String
  #17 = Utf8               SourceFile
  #18 = Utf8               SwitchTest.java
  #19 = NameAndType        #9:#10         //  "<init>":()V
  #20 = Class              #30            //  java/lang/String
  #21 = NameAndType        #31:#32        //  hashCode:()I
  #22 = Utf8               a
  #23 = NameAndType        #33:#34        //  equals:(Ljava/lang/Object;)Z
  #24 = Class              #35            //  java/lang/System
  #25 = NameAndType        #36:#37        //  out:Ljava/io/PrintStream;
  #26 = Class              #38            //  java/io/PrintStream
  #27 = NameAndType        #39:#14        //  println:(Ljava/lang/String;)V
  #28 = Utf8               SwitchTest
  #29 = Utf8               java/lang/Object
  #30 = Utf8               java/lang/String
  #31 = Utf8               hashCode
  #32 = Utf8               ()I
  #33 = Utf8               equals
  #34 = Utf8               (Ljava/lang/Object;)Z
  #35 = Utf8               java/lang/System
  #36 = Utf8               out
  #37 = Utf8               Ljava/io/PrintStream;
  #38 = Utf8               java/io/PrintStream
  #39 = Utf8               println
{
  public SwitchTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public void test(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=2
         0: aload_1
         1: astore_2
         2: iconst_m1
         3: istore_3
         4: aload_2
         5: invokevirtual #2                  // Method java/lang/String.hashCode:()I
         8: lookupswitch  { // 1
                      97: 28
                 default: 39
            }
        28: aload_2
        29: ldc           #3                  // String a
        31: invokevirtual #4                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
        34: ifeq          39
        37: iconst_0
        38: istore_3
        39: iload_3
        40: lookupswitch  { // 1
                       0: 60
                 default: 68
            }
        60: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
        63: ldc           #3                  // String a
        65: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        68: return
      LineNumberTable:
        line 4: 0
        line 6: 60
        line 8: 68
      StackMapTable: number_of_entries = 4
           frame_type = 253 /* append */
          offset_delta = 28
          locals = [ class java/lang/String, int ]
             frame_type = 10 /* same */
             frame_type = 20 /* same */
             frame_type = 249 /* chop */
            offset_delta = 7

  }

```
1、aload_1把第二个引用类型局部变量即为"a"推送至栈顶，第一个是this。
2、astore_2将栈顶的元素保存在第 3 个局部变量中。
3、iconst_m1将 int 型-1 推送至栈顶。
4、istore_3 把栈顶的值存入第三个局部变量
5、aload_2 把第三个变量入栈
6、invokevirtual #2 调用常量池中的java/lang/String.hashCode:()I的引用
7、lookupswitch 比较hash值是否为97，如果相等则跳转到#28
8、28: aload_2 把第3个变push到栈顶
8、ldc #3读取"a"
9、调用java/lang/String.equals
10、34: ifeq          39
8、调用 java/lang/System.out:Ljava/io/PrintStream
9、ldc #3读取"a"



##使用HSDB查看class文件位置
```
sudo java -cp /Library/Java/JavaVirtualMachines/jdk1.8.0.jdk/Contents/Home/lib/sa-jdi.jar sun.jvm.hotspot.HSDB
```