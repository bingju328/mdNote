#### <center>**java并发之线程通信**</center>
&ensp;&ensp;谈到并发我们就会想到多线程，要想实现多个线程之间的协同，如：线程执行先后顺序、获取某个
 线程执行的结果等等。都涉及取线程之间相互通信。
 线程通信分为以下四类：
  1. 文件共享
  2. 网络共享
  3. 共享变量
  4. **jdk提供的线程协调API**&ensp;细分为：~~suspend/resume~~、wait/notify、park/uppack

&ensp;&ensp;文件共享、变量共享这里不做讨论，需要提一点：**变量共享这里的变量是指的类内的全局变量并不是方法内的局部变量。** 我们着重讨论一下上面提到的几种线程协调API.

&ensp;&ensp;JDK中对于需要多线程协作完成某一任务的典型场景是：生产者-消费者模型。(线程阻塞、线程唤醒)。
这里为了说明问题，只是简单的实现生产者-消费者模型。
 示例：线程1去买包子，没有包子，则不再执行。线程2生产出包子，通知线程1继续执行。
下面分别对API里面的 **~~suspend/resume~~、wait/notify、park/uppack** 进行分析。
##### suspend/resume示例。
```
/**
 * 三种线程协作通信的方式之suspend/resume
 * @date 2019/2/21
 */
public class ThreadAPITest {
  /** 包子店 */
  public static Object baozidian = null;

  /**
   * 正常的suspend/resume
   * */
  public void suspendResumeTest1() throws Exception {
      // 启动线程
      Thread consumerThread = new Thread(() -> {
          if (baozidian == null) { // 如果没包子，则进入等待
              System.out.println("1、进入等待");
              System.out.println("suspend()前状态：" + Thread.currentThread().getState().toString());
              Thread.currentThread().suspend();
          }
          System.out.println("4、买到包子，回家");
      });
      consumerThread.start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      System.out.println("suspend()后状态：" + consumerThread.getState().toString());
      baozidian = new Object();
      System.out.println("2、产生一个包子");
      System.out.println("执行resume()。。。");
      consumerThread.resume();
      System.out.println("3、通知消费者");
  }
  /**
   * 死锁的suspend/resume。 suspend并不会像wait一样释放锁，故此容易写出死锁代码
   * */
  public void suspendResumeDeadLockTest2() throws Exception {
      // 启动线程
      Thread consumerThread = new Thread(() -> {
          if (baozidian == null) { // 如果没包子，则进入等待
              System.out.println("1、进入等待");
              // 当前线程拿到锁，然后挂起
              synchronized (this) {
                  System.out.println("获取到synchronized,进入synchronized,执行suspend()。。。");
                  Thread.currentThread().suspend();
              }
          }
          System.out.println("4、买到包子，回家");
      });
      consumerThread.start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      baozidian = new Object();
      System.out.println("2、产生一个包子");
      // 争取到锁以后，再恢复consumerThread
      synchronized (this) {
          System.out.println("获取到synchronized,执行suspend()。。。");
          consumerThread.resume();
      }
      System.out.println("3、通知消费者");
  }

  /**
   * 导致程序永久挂起的suspend/resume
   * */
  public void suspendResumeDeadLockTest3() throws Exception {
      // 启动线程
      Thread consumerThread = new Thread(() -> {
          if (baozidian == null) {
              System.out.println("1、没包子，进入等待");
              try { // 为这个线程加上一点延时
                  Thread.sleep(5000L);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              // 这里的挂起执行在resume后面
              System.out.println("执行suspend()。。。");
              Thread.currentThread().suspend();
          }
          System.out.println("4、买到包子，回家");
      });
      consumerThread.start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      baozidian = new Object();
      System.out.println("2、产生一个包子");
      System.out.println("执行resume()。。。");
      consumerThread.resume();
      consumerThread.join();
      System.out.println("3、通知消费者");

  }
    public static void main(String[] args) throws Exception {
        //1
        new ThreadAPITest().suspendResumeTest1();
        //2
        new ThreadAPITest().suspendResumeDeadLockTest2();
        //3
        new ThreadAPITest().suspendResumeDeadLockTest3();
    }
}
```
第一个调用结果如下：
```
1、进入等待
suspend()前状态：RUNNABLE
suspend()后状态：RUNNABLE
2、产生一个包子
执行resume()。。。
3、通知消费者
4、买到包子，回家
```
 &ensp;&ensp;这个是可以正常调用的，线程状态从NEW-》RUNNABLE后，调用suspend()线程依然是RUNNABLE，这说明suspend()方法并没有改变线程的
 状态，只是挂起(暂停)线程而已。

 第二个调用结果如下：
 ```
 1、进入等待
 获取到synchronized,进入synchronized,执行suspend()。。。
 2、产生一个包子
 .......................
 ```
 &ensp;&ensp;从这个结果可以看到，线程进入synchronized代码块-》持有锁-》执行suspend()挂起-》3秒之后，生产一个包子线程就中止了，并没有执行System.out.println("获取到synchronized,执行suspend()。。。");这一行。**这说明suspend()方法挂起时持有锁，并没有释放锁。** 这样就产生了死锁。

 第三个调用结果如下：
 ```
 1、没包子，进入等待
 2、产生一个包子
 执行resume()。。。
 执行suspend()。。。
 ```
&ensp;&ensp;这个运行是：线程先sleep()5秒-》主线程先执行了resume()-》工作线程执行suspend().这个时候线程就永久挂起了,也就是死锁。所以suspend()必须在resume()之前执行。

>综上suspend/resume方式：
* suspend()方法挂起时线程状态并没有改变，此方法也没有遵循线程状态流程
* 加锁/不加锁都对调用顺序有严格要求，否则容易死锁。
* 加锁时suspend()方法会持锁，并且不释放，这样也容易导致死锁。

##### wait/notify示例。
```
/**
 * 三种线程协作通信的方式之wait/notify
 * @date 2019/2/21
 */
public class ThreadAPITes2 {
    /** 包子店 */
    public static Object baozidian = null;

    /** 正常的wait/notify */
    public void waitNotifyTest1() throws Exception {
        // 启动线程
        Thread waitNotify = new Thread(() -> {
            synchronized (this) {
                while (baozidian == null) { // 如果没包子，则进入等待
                    try {
                        System.out.println("1、进入等待");
                        System.out.println("wait()前状态：" + Thread.currentThread().getState().toString());
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            System.out.println("4、买到包子，回家");
        });
        waitNotify.start();
        // 3秒之后，生产一个包子
        Thread.sleep(3000L);
        System.out.println("2、生产一个包子");
        baozidian = new Object();
        synchronized (this) {
            System.out.println("wait()后状态：" + waitNotify.getState().toString());
            this.notifyAll();
            System.out.println("3、通知消费者");
        }
    }

    /** 会导致程序永久等待的wait/notify */
    public void waitNotifyDeadLockTest2() throws Exception {
        // 启动线程
        new Thread(() -> {
            if (baozidian == null) { // 如果没包子，则进入等待
                try {
                    Thread.sleep(5000L);
                } catch (InterruptedException e1) {
                    e1.printStackTrace();
                }
                synchronized (this) {
                    try {
                        System.out.println("1、进入等待");
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            System.out.println("2、买到包子，回家");
        }).start();
        // 3秒之后，生产一个包子
        Thread.sleep(3000L);
        baozidian = new Object();
        System.out.println("2、生产一个包子");
        synchronized (this) {
            System.out.println("执行notifyAll()方法");
            this.notifyAll();
            System.out.println("3、通知消费者");
        }
    }
    public static void main(String[] args) throws Exception {
        // wait/notify要求再同步关键字里面使用，免去了死锁的困扰，
        //但是一定要先调用wait，再调用notify，否则永久等待了
        //1
        new ThreadAPITes2().waitNotifyTest1();
        //2
        new ThreadAPITes2().waitNotifyDeadLockTest2();
    }
}
```
第一个调用结果如下：
```
1、进入等待
wait()前状态：RUNNABLE
2、生产一个包子
wait()后状态：WAITING
3、通知消费者
4、买到包子，回家
```
&ensp;&ensp;这是wait()/nofity()方式的正确示例。但有一点需要注意，wait()必须在同步关键字里面使用。执行wait()后只是线程等待，等待时wait()方法会释放锁。

第二个调用结果如下：
```
2、生产一个包子
执行notifyAll()方法
3、通知消费者
1、进入等待
。。。。。。。。。。。。。。
```
&ensp;&ensp;这个执行时先调用了nofityAll()方法，然后调用wait()方法进入等待。就会产生死锁。
>综上wait/notify方式：
* wait()方法等待时会释放锁，这样就免去了死锁的困扰。
* 必须在同步关键字里面使用。
* 对调用顺序有严格要求，否则容易死锁。

##### park/uppark示例
```
/**
 * 三种线程协作通信的方式之park/unpark
 * @date 2019/2/21
 */
public class ThreadAPITes3 {
    /** 包子店 */
    public static Object baozidian = null;

    /** 正常的park/unpark */
    public void parkUnparkTest1() throws Exception {
        // 启动线程
        Thread consumerThread = new Thread(() -> {
            while (baozidian == null) { // 如果没包子，则进入等待
                System.out.println("1、进入等待");
                System.out.println("park()前状态：" + Thread.currentThread().getState().toString());
                LockSupport.park();
            }
            System.out.println("4、买到包子，回家");
        });
        consumerThread.start();
        // 3秒之后，生产一个包子
        Thread.sleep(3000L);
        baozidian = new Object();
        System.out.println("2、产生一个包子");
        System.out.println("park()后状态：" + consumerThread.getState().toString());
        System.out.println("调用uppark()");
        LockSupport.unpark(consumerThread);
        System.out.println("3、通知消费者");
    }

    /** 正常的park/unpark */
    public void parkUnparkTest2() throws Exception {
        // 启动线程
        Thread consumerThread = new Thread(() -> {
            while (baozidian == null) { // 如果没包子，则进入等待
                try {
                    System.out.println("1、进入等待");
                    Thread.sleep(3000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("调用park()");
                LockSupport.park();
            }
            System.out.println("4、买到包子，回家");
        });
        consumerThread.start();
        // 1秒之后，生产一个包子
        Thread.sleep(1000L);
        baozidian = new Object();
        System.out.println("2、产生一个包子");
        System.out.println("调用uppark()");
        LockSupport.unpark(consumerThread);
        System.out.println("3、通知消费者");
    }

    /** 死锁的park/unpark */
    public void parkUnparkDeadLockTest3() throws Exception {
        // 启动线程
        Thread consumerThread = new Thread(() -> {
            if (baozidian == null) { // 如果没包子，则进入等待
                System.out.println("1、进入等待");
                // 当前线程拿到锁，然后挂起
                synchronized (this) {
                    LockSupport.park();
                }
            }
            System.out.println("4、买到包子，回家");
        });
        consumerThread.start();
        // 3秒之后，生产一个包子
        Thread.sleep(3000L);
        baozidian = new Object();
        System.out.println("2、产生一个包子");
        // 争取到锁以后，再恢复consumerThread
        synchronized (this) {
            System.out.println("调用uppark()");
            LockSupport.unpark(consumerThread);
        }
        System.out.println("3、通知消费者");
    }
    public static void main(String[] args) throws Exception {
        // park/unpark没有顺序要求，但是park并不会释放锁，
        // 所有再同步代码中使用要注意
        //1
        new ThreadAPITes3().parkUnparkTest1();
        //2
        new ThreadAPITes3().parkUnparkTest2();
        //3
        new ThreadAPITes3().parkUnparkDeadLockTest3();
    }
}
```
第一个调用结果如下
```
1、进入等待
park()前状态：RUNNABLE
2、产生一个包子
park()后状态：WAITING
调用uppark()
3、通知消费者
4、买到包子，回家
```
第二个调用结果如下
```
1、进入等待
2、产生一个包子
调用uppark()
3、通知消费者
调用park()
4、买到包子，回家
```
这两个都是正常的调用，一个是先调用park(),一个是先调用uppark(),都可以正常运行，park()/uppark() 对调用顺序没有要求。

第三个调用结果如下
```
1、进入等待
2、产生一个包子
。。。。。。。。。。。
```
这个是在同步代码块中，先调用park，在调用uppark时获取不到锁，说明park()方法不会释放锁。这时就产生死锁了。
>综上park/uppark方式：
* park()方法等待时不会释放锁，在同步代码块中慎用。
* park/unpark没有顺序要求。

>** park/unpark没有顺序要求，它的设计原理核心是“许可”：park是等待一个许可，unpark是为某线程提供一个许可。也就是说，unpark先提供许可。当某线程调用park时，已经有许可了，它就消费这个许可，然后可以继续运行了。再详细的以后再讨论。**

**以上三种方式，suspend/resume已被弃用，wait(),park()注要有以下几个注意事项。**
* wait()需释放锁，用的时候需要在synchronized中使用.
* wait()必须在nofify之前调用。
* park/unpark在同步代码块中使用要注意park不释放锁的问题。
