#### <center>**java并发之线程封闭**</center>
&ensp;&ensp;JVM运行时数据区分为线程共享部分、线程独占部分。多线程程访问共享可变数据时，涉及到线程间数据同步的问题。但并不是所有时候都要用到共享数据，所以线程封闭概念就提出来了。
数据都被封闭在各自的线程之中，就不需要同步，这种通过将数据封闭在线程中而避免使用同步的技术称为**线程封闭**。

&ensp;&ensp;线程封闭具体的体现有:
* 栈封闭(局部变量)
* ThreadLocal类

##### 栈封闭
&ensp;&ensp;栈封闭是我们编程当中遇到的最多的线程封闭。其实就是局部变量。多个线程访问一个方法，此方法中的局部变量都会被拷贝一分儿到线程栈中。编译时在虚拟机栈中存储，虚拟机栈是线程独享的，所以局部变量是不被多个线程所共享的，也就不会出现并发问题。
##### ThreadLocal封闭
&ensp;&ensp;ThreadLocal类是JDK提供的一种特殊的变量。它是一个线程级别变量，每个线程都有一个ThreadLocal，即每个线程都拥用了自己独立的一个变量，这个类能防止线程内的可变变量的共享，通过ThreadLocal类可保证线程特有的变量封闭在线程内而不会逸出到该线程外。

**用法：** ThreadLocal<T> var = new ThreadLocal<T>();
会自动在每个线程上创建一个T的副本，副本之间彼此独立，互不影响。可以用ThreadLocal存储一些参数，以便在线程中多个方法中使用，用来代替方法传参的做法。

代码示列
```
/** 线程封闭示例 */
public class Demo {
	/** threadLocal变量，每个线程都有一个副本，互不干扰 */
	public static ThreadLocal<String> value = new ThreadLocal<>();

	/**
	 * threadlocal测试
	 *
	 * @throws Exception
	 */
	public void threadLocalTest() throws Exception {

		// threadlocal线程封闭示例
		value.set("这是主线程设置的MainThreadTest"); // 主线程设置值
		String v = value.get();
		System.out.println("线程1执行之前，主线程取到的值：" + v);

		new Thread(new Runnable() {
			@Override
			public void run() {
				String v = value.get();
				System.out.println("线程1取到的值：" + v);
				// 设置 threadLocal
				value.set("这是线程1设置的Thread1");

				v = value.get();
				System.out.println("重新设置之后，线程1取到的值：" + v);
				System.out.println("线程1执行结束");
			}
		}).start();

		Thread.sleep(5000L); // 等待所有线程执行结束

		v = value.get();
		System.out.println("线程1执行之后，主线程取到的值：" + v);

	}

	public static void main(String[] args) throws Exception {
		new Demo().threadLocalTest();
	}
}
```
运行结果如下：
```
线程1执行之前，主线程取到的值：这是主线程设置的MainThreadTest
线程1取到的值：null
重新设置之后，线程1取到的值：这是线程1设置的Thread1
线程1执行结束
线程1执行之后，主线程取到的值：这是主线程设置的MainThreadTest
```
&ensp;&ensp;从运行结果可以看出，在主线程中给赋的值，子线程获取不到。子线程中赋的值，主线程也获取不到。这个值对线程是独立的,每个线程都拥用了自己独立的一个变量，互不影响。
