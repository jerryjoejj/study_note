# 谈谈对Java平台理解
## Java语言特性
- 书写一次，到处运行 Write once,run anywhere
- 垃圾收集 GC
- Java源代码通过JVM内嵌解释器将字节码转换成最终执行的机器代码，但是Oralce Hotsopt JVM提供JIT(Just-In-Time)编译器，部分代码在运行时将热点代码编译成机器码
# Exception和Error区别
均集成自Throwable类