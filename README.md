# 深入理解Java虚拟机 JVM高级特性与最佳实践
# 第三章 垃圾收集器与内存分配策略
## GC需要完成的3件事情
1. 哪些内存需要回收
2. 什么时候回收
3. 如何回收
## 需要回收的内存区域
```
程序计数器、栈、本地方法栈3个区域随线程而生，随线程而灭，所以不需要进行垃圾回收。
程序处于运行期间才知道会创建哪些对象，java堆上的内存分配是动态分配的，所以收集器关注
的是这部分内存。
```
## 判断对象可回收的算法
1. 引用计数法
```
这个算法解决不了对象循环引用问题，所以java没有采用此种算法。
```
2. 可达性算法
```
这个算法基本思路是通过一系列称为"GC Roots"的对象为起始点，从这些节点开始向下搜索，搜索的路径称为引用链，
当一个对象没有任何引用链相连，则证明此对象不可用，可以被回收。
可作为GC Roots的对象包括下面几种：
1. 栈中引用的对象
2. 方法区中类静态属性引用的对象
3. 方法区中常量引用对象
4. 本地方法栈中JNI引用的对象
```
## 对象死亡之前的"挣扎"
```
要真正宣告一个对象死亡，至少要经历两次标记过程，如果对象没有相连接的引用链，那它会被第一次
标记并且进行一次筛选，筛选的条件是些对象是否有必要执行finalize()方法。当对象没有覆盖finalize()
方法，或者finalize()方法已经被虚拟机调用过，虚拟机将这两种情况视为没必要执行，直接回收。
但不鼓励使用finalize()方法。
```
## 垃圾收集算法
1. 标记-清除算法
```
此算法分为标记和清除两个阶段。
首先标记出所以需要回收的对象，然后统一回收所有被标记的对象。
缺点:
1. 效率不高
2. 收集完成后产生的内存碎片太多
```
2. 复制算法
```
它将可用内存按容量划分为大小相等的两块，每次只使用其中一块。当一块内存用完了，就
将还存活着的对象复制到另外一块上面，然后清除掉使用过的内存空间。
缺点:
1. 使用内存被缩小了一半
2. 复制的对象过多过大耗时严重
优点：
1. 不会产生内存碎片
```
3. 标记-整理算法
```
标记整理与标记清除相似，标记整理只是标记后，让所以存活的对象都向一端移动，然后
直接清理掉端边界以外的内存。
```
4. 分代收集算法
```
当前的商业虚拟机的垃圾收集器都采用分代收集算法。一般是把java堆分为新生代和老年代，
这样可以根据各个年代的特点采用最适当的收集算法。
在新生代中，每次垃圾收集时都有大量对象死去，那就选用复制算法，只需要付出少量存活
对象复制成本就可以完成收集。面老年代中因为存活率高，就必须使用标记-清理或标记整理
来回收。
```
