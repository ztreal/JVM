# 字节码操作

常见的操作字节码的框架为asm，spring可能使用它实现AOP功能.由于asm的高效，限制低的特性，还是很受欢迎的。java.lang.ref.proxy的效率低(使用反射)，限制多(必须实现接口)


asm官方文档
http://download.forge.objectweb.org/asm/asm4-guide.pdf



cglib封装了asm