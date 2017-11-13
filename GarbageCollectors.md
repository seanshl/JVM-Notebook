### References
* [Understand Java Garbage Collection](https://www.cubrid.org/blog/understanding-java-garbage-collection)
* [理解java垃圾回收机制](http://jayfeng.com/2016/03/11/%E7%90%86%E8%A7%A3Java%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6/)

### 问题
1. 为什么需要垃圾回收
2. 回收什么？
	1. 哪里的数据
	2. 如何判断对象已死
3. 怎么回收?
	1. 垃圾回收算法
	2. 垃圾回收器

### 为什么需要垃圾回收
1. 程序员无需手动释放内存

### 收集哪里的数据
1. java堆中存储的程序构建的对象

### 判断对象已死

1. **Reference Counter**
	1. 给对象添加一个引用计数器,引用计数器为0的对象不可能再被使用.
	2. +: 实现简单，判定效率高
	3. -: 循环引用问题无法解决
2. **可达性分析/Reachability Analysis**
	1. 算法：通过一系列被称为**GC Roots**的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称为引用链(reference chain), 当一个对象到GC Roots之间没有任何reference chain相连，则证明此对象是不可用的.
	2. **GC Roots**：
		* jvm heap(栈帧中的本地变量表)中引用的对象
		* 方法区中类静态属性引用的对象
		* 本方法区中常量引用的对象
		* 本地方法栈中JNI(native 方法)引用的对象
3. **强引用和弱引用**
	1. jdk1.2中，判断引用的方法就是如果是reference类型的数据中存储的数值代表的是另外一块内存的起始地址。
	2. 1.2之后进行扩展，对引用扩充为**Strong Reference, Soft Reference, Weak Reference, Phantom Reference**. 强度依次下降
		* 强引用: 普遍存在，Object obj = new Object()这类的引用，只要强引用存在，GC永远不会回收掉被引用的对象.
		* 软引用： 描述一些还有用但并非必需的对象.被软引用关联的对象，在系统将要发生内存溢出时，将会把这些对象列进回收范围进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常.
		* 弱引用: 也用来描述非必需对象，但是强度更弱，被弱引用关联的对象之鞥呢生存岛下一次垃圾回收发生之前。当GC工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的独享.
		* 虚引用： 对其生存时间不构成影响，唯一目的是该对象被回收时毁收到一个系统通知.
4. **生存还是死亡**
	* 即使是在可达性分析中被判断为不可达的对象，也不是非死不可.
	* 宣告死亡必须进行至少两次标记过程：
		1. 如果对象经过可达性分析之后发现没有与GC Roots连接的引用链，将会被第一次标记并进行一次筛选，筛选的条件是此对象是否有必要执行finalize()方法. 对象没有覆盖finalize()方法或者该方法已经被虚拟机调用过了，虚拟机将这两种情况视为没有必要执行.
		2. 如果被判断有必要执行finalize 对象会被放到一个**F-Queue**的地方.
			* 虚拟机自动建立一个低优先级的Finalizer线程去异步执行它.
			* finalize方法是对象最后一次逃脱死亡的机会，GC之后会对F-Queue中的对象进行第二次小规模的标记，如果对象在finalize中重新与引用链上的任何一个对象建立关联，例如把自己(this)赋值给某个类变量或者对象的成员变量，在第二次标记时它会被移除出机将会受到的集合，否则，基本上就被回收了。自救的机会只有一次，因为finalize方法最多会被系统自动调用一次.
			* 这种方法尽量不要使用.finalize方法最好不用触碰.
			
### 垃圾回收算法
1. **Mark and Sweep**
	1. 可达性分析进行标记
	2. 垃圾回收器进行回收
	3. 问题：产生大量的内存空间不连续碎片
	4. 适用于old generation
2. **Copying**
	1. 将堆空间分成五五开的两个部分，当需要垃圾回收时，将survivors挪到第二个空空间，然后清空第一个空间
	2. 问题：内存效率低
	3. 适用于new generation (重新划分堆空间后)
3. **Generation Collection**
	1. 根据copying算法，将空间化为：
		* permanent generation
		* old generation (若干次垃圾回收的幸存者)
		* new generation
			* eden (80%)的比例
			* survivor 1 (和eden共同组成copying算法的第一个空间)
			* survivor 2 (copying算法的第二个空间)
	2. 根据不同的generation进行垃圾回收
		1. 新生代: copying
		2. 老年代：mark and sweep
	
		
	
