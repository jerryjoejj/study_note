# Synchronized实现原理
```java
public class SynchronizedTest {

    public synchronized void doSth(){
        System.out.println("Hello World");
    }

    public void doSth1(){
        synchronized (SynchronizedTest.class){
            System.out.println("Hello World");
        }
    }
}
```

同步代码块使用**monitorenter**和**monitorexit**两个指令实现。执行**monitorenter**指令加锁，执行**monitorexit**指令释放锁。
同步方法使用**ACC_SYNCHRONIZED**关键字隐式的对方法加锁，当线程执行的方法被标上**ACC_SYNCHRONIZED**时，需要先获得锁才能执行该方法。
每个对象维护一个计数器，记录对象被锁次数，当一个线程获得锁时，该计数器自增1，当同一个线程释放锁是，该计数器减1。

# Java对象模型
每一个Java类在被JVM加载的时候，JVM会为这个类创建一个**instanceKlass**，保存在方法区，用于在JVM层表示该Java类。当我们使用new创建一个对象时，JVM会创建一个**instanceOopDesc**对象，包含两部分信息：对象头及元数据。对象头中有一些运行时数据，其中包括多线程相关锁的信息。元数据维护的指针指向对象所属类的**instanceKlass**。

# Java对象头
``` c++
class oopDesc {
    friend class VMStructs;
    private:
    volatile markOop  _mark;
    union _metadata {
        wideKlassOop    _klass;
        narrowOop       _compressed_klass;
  } _metadata;
}
```
markword设计是将存储空间划分为多个比特位，并在不同对象状态下赋予比特位含义。下图为32为虚拟机
![2018-12-12.16.04.25-ObjectHead.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.16.04.25-ObjectHead.png)
# Monitor实现原理
同步方法和同步代码块都是基于**monitor**实现的。
## 操作系统中管程
**管程**是一种程序结构，结构内多个多个子程序（对象或模块）形成多个共享线程互斥访问共享资源。这些共享资源一般是硬件资源或者共享变量。管程实现了在一个时间点最多只有一个线程执行管程中某个子程序。
## Java线程同步相关之Monitor
对象的所有方法互斥执行，一个monitor只有一个运行许可，任一线程进入任何方法都需要获得这个许可，离开时归还许可。提供**singal**机制，允许正在持有许可的线程放弃许可，等待某个条件成立后当前线程可以通知等待这个条件变量的线程去重新获取许可。
# Monitor实现
Java虚拟机（HotSpot）的monitor是基于C++实现的，主要数据结构如下
``` c++
 ObjectMonitor() {
    _header       = NULL;
    _count        = 0;
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL;
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```
关键属性如下：
``` c++
_owner 		// 指向持有ObjectMonitor对象的线程
_WaitSet	// 存放处于wait状态的线程队列
_EntryList	// 存在处于wait状态的线程队列
_recursion	// 锁的重入次数
_count 		// 用来记录该线程获取锁的次数
```
当多个线程同时访问一段同步代码时，会首先进入**EntryList** 队列中，当某个线程获取到对象的monitor后进入**Owner** 区域并把**ower** 变量设置为当前线程，同时monitor中计数器加1。即线程获得锁。
当持有monitor的线程调用**wait()** 方法，将释放当前持有monitor，**owner**变量恢复为null，**count**减1，同时**WaitSet**集合中等待线程会被唤醒。当前线程执行完毕也会释放monitor。如下图所示。
![2018-12-12.17.11.38-monitor.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.17.11.38-monitor.png)

## 获取锁
```c++
void ATTR ObjectMonitor::enter(TRAPS) {
	Thread * const Self = THREAD ;
	void * cur ;
	//通过CAS尝试把monitor的`_owner`字段设置为当前线程
	cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
	//获取锁失败
	if (cur == NULL) {assert (_recursions == 0, "invariant") ;
		 assert (_owner      == Self, "invariant") ;
		 // CONSIDER: set or assert OwnerIsThread == 1
		 return ;
	}
	// 如果旧值和当前线程一样，说明当前线程已经持有锁，此次为重入，_recursions自增，并获得锁。
	if (cur == Self) { 
		// TODO-FIXME: check for integer overflow!  BUGID 6557169.
		_recursions ++ ;
		return ;
	}

	// 如果当前线程是第一次进入该monitor，设置_recursions为1，_owner为当前线程
	if (Self->is_lock_owned ((address)cur)) { 
		assert (_recursions == 0, "internal state error");
		_recursions = 1 ;
		// Commute owner from a thread-specific on-stack BasicLockObject address to
		// a full-fledged "Thread *".
		_owner = Self ;
		OwnerIsThread = 1 ;
		return ;
	}

	// 省略部分代码。
	// 通过自旋执行ObjectMonitor::EnterI方法等待锁的释放
	for (;;) {
	  jt->set_suspend_equivalent();
	  // cleared by handle_special_suspend_equivalent_condition()
	  // or java_suspend_self()

	  EnterI (THREAD) ;

	  if (!ExitSuspendEquivalent(jt)) break ;

	  //
	  // We have acquired the contended monitor, but while we were
	  // waiting another thread suspended us. We don't want to enter
	  // the monitor while suspended because that would surprise the
	  // thread that suspended us.
	  //
		  _recursions = 0 ;
	  _succ = NULL ;
	  exit (Self) ;

	  jt->java_suspend_self();
	}
}
```

![2018-12-12.17.13.35-lockenter.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.17.13.35-lockenter.png)

## 释放锁
``` c++
void ATTR ObjectMonitor::exit(TRAPS) {
   Thread * Self = THREAD ;
   //如果当前线程不是Monitor的所有者
   if (THREAD != _owner) { 
     if (THREAD->is_lock_owned((address) _owner)) { // 
       // Transmute _owner from a BasicLock pointer to a Thread address.
       // We don't need to hold _mutex for this transition.
       // Non-null to Non-null is safe as long as all readers can
       // tolerate either flavor.
       assert (_recursions == 0, "invariant") ;
       _owner = THREAD ;
       _recursions = 0 ;
       OwnerIsThread = 1 ;
     } else {
       // NOTE: we need to handle unbalanced monitor enter/exit
       // in native code by throwing an exception.
       // TODO: Throw an IllegalMonitorStateException ?
       TEVENT (Exit - Throw IMSX) ;
       assert(false, "Non-balanced monitor enter/exit!");
       if (false) {
          THROW(vmSymbols::java_lang_IllegalMonitorStateException());
       }
       return;
     }
   }
    // 如果_recursions次数不为0.自减
   if (_recursions != 0) {
     _recursions--;        // this is simple recursive enter
     TEVENT (Inflated exit - recursive) ;
     return ;
   }

   //省略部分代码，根据不同的策略（由QMode指定），从cxq或EntryList中获取头节点，通过ObjectMonitor::ExitEpilog方法唤醒该节点封装的线程，唤醒操作最终由unpark完成。
```
![2018-12-12.17.13.53-lockexit.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.17.13.53-lockexit.png)

>**注意** sychronized操作是重量级操作，需要将用户态转换到内核态。
# Java虚拟机锁优化技术
## 线程状态
分为以下五种，分别为：**初始状态（New）**，**就绪状态（Runnable）**，**运行状态（Running）**，**阻塞状态（Blocked）**，**死亡状态（Dead）**。各种状态间转换如下图所示：
![2018-12-12.17.27.29-thread.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.17.27.29-thread.png)
## 自旋锁
线程不放弃处理器执行时间，等待共享资源可访问后继续执行，自旋锁只是将当前线程不停执行循环体并检查共享资源，不改变线程状态。
>注意：线程数不停增加是性能会下降。
## 锁消除
在使用synchronized时，如果使用JIT逃逸分析发现并无线程安全问题，则会使用锁消除。


