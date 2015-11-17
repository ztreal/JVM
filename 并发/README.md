# 并发

因为现在都是多处理器，多处理器系统中每个处理器都有自己的缓存，但是又有共享的主存储main memory。所以共享同一住内存区域时可能导致各自的缓存数据不一致。所以出现了解决此类问题的协议。
* cpu的[缓存一致性](http://zh.wikipedia.org/wiki/%E5%BF%AB%E5%8F%96%E4%B8%80%E8%87%B4%E6%80%A7)cache coherence：

类似协议MSI、MESI、MOSI
[intel文档](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)，IA-32处理器和Intel 64处理器使用MESI（修改，独占，共享，无效）控制协议去维护内部缓存和其他处理器缓存的一致性
![](http://7vzu3j.com1.z0.glb.clouddn.com/MOSI.png)
编译open jdk
参考https://github.com/hgomez/obuildfactory/wiki/Building-and-Packaging-OpenJDK7-for-OSX

## 常用变量
### volatitle
#### 语意

#### 实现原理




### final
#### 语意

#### 实现原理






