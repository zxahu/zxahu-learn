## JVM

### 内存分区
- 程序计数器：如果是java方法，则是java指令地址；如果是native方法，则为空
- 虚拟机栈：Java线程的内存模型，调用方法是压栈，方法结束是栈帧出栈
- 本地方法栈：对应的native方法的内存模型
- 堆：线程共享的内存空间，用于存储对象
- 方法区(永久代)：存储类信息、常量、静态变量、即时编译后的代码等
- 运行时常量池(方法区一部分)：用于存放编译期生成的各种字面量和符号引用，在类加载后存放到方法区的运行常量池

### 垃圾回收算法
- 标记-清除
- 复制整理
- 分代收集

#### 垃圾确认规则
##### 引用计数法
无法检测互相引用的无用对象
##### GC Root追溯法
从GC Root开始遍历，如果对象没有到GC Root的路径，则说明这个对象是无用对象。   
能称为GC Root的对象：
- 栈中引用的对象，
- 方法区中的类引用的对象，
- 方法区中的常量引用的对象，
- native方法引用的对象

### 垃圾回收器
#### CMS垃圾回收器
将堆内存分为年轻代和年老代，年轻代包括：eden, s0, s1   
##### CMS的回收过程:
- 初始标记(stop the world，时间较短)
- 并行标记
- 重新标记(stop the world，时间较长) ：为了标记上一步遗漏的对象
- 并发清除

##### CMS的缺点
todo

#### G1垃圾回收器