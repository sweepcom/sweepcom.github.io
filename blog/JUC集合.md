---
title: JUC系列之集合
date: 2018/8/24 11:02:59  
categories: "JUC系列"
---
	ArrayBlockingQueue ：由数组结构组成的有界阻塞队列。
	LinkedBlockingQueue ：由链表结构组成的有界阻塞队列。
	PriorityBlockingQueue ：支持优先级排序的无界阻塞队列。
	DelayQueue：使用优先级队列实现的无界阻塞队列。
	SynchronousQueue：不存储元素的阻塞队列。
	LinkedTransferQueue：由链表结构组成的无界阻塞队列。
	LinkedBlockingDeque：由链表结构组成的双向阻塞队列。

# 1.CopyOnWriteArrayList和CopyOnWriteArraySet #

它相当于线程安全的ArrayList。和ArrayList一样，它是个可变数组；但是和ArrayList不同的时，

	它具有以下特性：
	1. 它最适合于具有以下特征的应用程序：List 大小通常保持很小，只读操作远多于可变操作，需要
	   在遍历期间防止线程间的冲突。
	2. 它是线程安全的。
	3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
	4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等操作。
	5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

下面从“动态数组”和“线程安全”两个方面对CopyOnWriteArrayList的原理进行说明。

1. CopyOnWriteArrayList的“动态数组”机制 -- 它内部有个“volatile数组”(array)来保持数据。在“添加/修改/删除”数据时，都会新建一个数组，并将更新后的数据拷贝到新建的数组中，最后再将该数组赋值给“volatile数组”。这就是它叫做CopyOnWriteArrayList的原因！CopyOnWriteArrayList就是通过这种方式实现的动态数组；不过正由于它在“添加/修改/删除”数据时，都会新建数组，所以涉及到修改数据的操作，CopyOnWriteArrayList效率很
低；但是单单只是进行遍历查找的话，效率比较高。

2. CopyOnWriteArrayList的“线程安全”机制 -- 是通过volatile和互斥锁来实现的。

 (01) CopyOnWriteArrayList是通过“volatile数组”来保存数据的。一个线程读取volatile数组时，总能看到其它线程对该volatile变量最后的写入；就这样，通过volatile提供了“读取到的数据总是最新的”这个机制的保证。

 (02) CopyOnWriteArrayList通过互斥锁来保护数据。在“添加/修改/删除”数据时，会先“获取互斥锁”，再修改完毕之后，先将数据更新到“volatile数组”中，然后再“释放互斥锁”；这样，就达到了保护数据的目的。 

CopyOnWriteArraySet类似于CopyOnWriteArrayList它是线程安全的无序的集合，可以将它理解成线程安全的HashSet

# ArrayBlockingQueue #

ArrayBlockingQueue是数组实现的线程安全的有界的阻塞队列。

线程安全是指，ArrayBlockingQueue内部通过“互斥锁”保护竞争资源，实现了多线程对竞争资源的互斥访问。而有界，则是指ArrayBlockingQueue对应的数组是有界限的。 

阻塞队列，是指多线程访问竞争资源时，当竞争资源已被某线程获取时，其它要获取该资源的线程需要阻塞等待；而且，ArrayBlockingQueue是按 FIFO（先进先出）原则对元素进行排序，元素都是从尾部插入到队列，从头部开始返回。

 1. ArrayBlockingQueue继承于AbstractQueue，并且它实现了BlockingQueue接口。
   
 2. ArrayBlockingQueue内部是通过Object[]数组保存数据的，也就是说ArrayBlockingQueue本质上是通过数组实现的。ArrayBlockingQueue的大小，即数组的容量是创建ArrayBlockingQueue时指定的。
    
 3. ArrayBlockingQueue与ReentrantLock是组合关系，ArrayBlockingQueue中包含一个ReentrantLock对象(lock)。ReentrantLock是可重入的互斥锁，ArrayBlockingQueue就是根据该互斥锁实现“多线程对竞争资源的互斥访问”。而且，ReentrantLock分为公平锁和非公平锁，关于具体使用公平锁还是非公平锁，在创建ArrayBlockingQueue时可以指定；而且，ArrayBlockingQueue默认会使用非公平锁。
    
 4. ArrayBlockingQueue与Condition是组合关系，ArrayBlockingQueue中包含两个Condition对象(notEmpty和notFull)。而且，Condition又依赖于ArrayBlockingQueue而存在，通过Condition可以实现对ArrayBlockingQueue的更精确的访问 -- (01)若某线程(线程A)要取数据时，数组正好为空，则该线程会执行notEmpty.await()进行等待；当其它某个线程(线程B)向数组中插入了数据之后，会调用notEmpty.signal()唤醒“notEmpty上的等待线程”。此时，线程A会被唤醒从而得以继续运行。(02)若某线程(线程H)要插入数据时，数组已满，则该线程会它执行notFull.await()进行等待；当其它某个线程(线程I)取出数据之后，会调用notFull.signal()唤醒“notFull上的等待线程”。此时，线程H就会被唤醒从而得以继续运行。

#  LinkedBlockingQueue、LinkedBlockingDeque #

	1. LinkedBlockingQueue继承于AbstractQueue，它本质上是一个FIFO(先进先出)的队列。
	2. LinkedBlockingQueue实现了BlockingQueue接口，它支持多线程并发。当多线程竞争同一个资源时，某线程获取到该资源之后，其它线程需要阻塞等待。
	3. LinkedBlockingQueue是通过单链表实现的。
	(01) head是链表的表头。取出数据时，都是从表头head处插入。
	(02) last是链表的表尾。新增数据时，都是从表尾last处插入。
	(03) count是链表的实际大小，即当前链表中包含的节点个数。
	(04) capacity是列表的容量，它是在创建链表时指定的。
	(05) putLock是插入锁，takeLock是取出锁；notEmpty是“非空条件”，notFull是“未满条件”。	通过它们对链表进行并发控制。
       LinkedBlockingQueue在实现“多线程对竞争资源的互斥访问”时，对于“插入”和“取出(删除)”操作分别使用了不同的锁。对于插入操作，通过“插入锁putLock”进行同步；对于取出操作，通过“取出锁takeLock”进行同步。
       此外，插入锁putLock和“非满条件notFull”相关联，取出锁takeLock和“非空条件notEmpty”相关联。通过notFull和notEmpty更细腻的控制锁。

LinkedBlockingDeque是双向链表实现的双向并发阻塞队列。该阻塞队列同时支持FIFO和FILO两种操作方式，即可以从队列的头和尾同时操作(插入/删除)；并且，该阻塞队列是支持线程安全
