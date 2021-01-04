# Java多线程
## 线程/进程 的概念
- 线程就是程序中单独顺序的流控制。线程本身不能运行，它只能用于程序中。
- 多线程： 多线程则指的是在单个程序中可以同时运行多个不同的线程执行不同的任务。
- 进程：执行中的程序（程序是静态的概念，进程是动态的概念）。
- 说明：线程是程序内的顺序控制流，只能使用分配给程序的资源和环境。
- Java 中如果我们自己没有产生线程，那么系统就会给我们产生一个线程（主线程，main方法就在主线程上运行），我们的程序都是由线程来执行的。

## 多线程
- 一个进程可以包含一个或多个线程。
- 一个程序实现了多个代码同时交替运行就需要产生多个线程。
- CPU随机的抽出时间，让我们的程序一会做这件事就，一会做另外一件事情。

## 线程的实现
1. 方式一：继承 Thread 类，然后重写 run 方法；
2. 方式二：实现 Runnable 接口，然后实现 run 方法。将我们希望线程执行的代码放到 run 方法中，然后通过 start 方法来启动线程，start方法首先为线程的执行准备好系统资源，然后再去调用 run 方法。当某个类继承了 Thread 类之后，该类就叫做一个线程类。

### 继承Thread类
```java
public class ThreadTest {
	public static void main(String[] args) {
		Thread1 t1 = new Thread1();
		Thread2 t2 = new Thread2();

		t1.start();		//启动线程，会自动调用run方法
		t2.start();
//		t2.start();		//结果每次相同。必须使用start方法才会被认为是以线程方式启动

	}
}
```
```java
class Thread1 extends Thread{
	//将需要执行的代码放在run方法中
	@Override
	public void run() {
		for(int i = 0; i < 10; i++){
			System.out.println("hello:" + i);
		}
	}
}
```
```java
class Thread2 extends Thread{
	//将需要执行的代码放在run方法中
	@Override
	public void run() {
		for(int i = 0; i < 10; i++){
			System.out.println("welcome:" + i);
		}
	}
} 
```
- 对于单核 CPU 来说，某一时刻只能有一个线程在执行（微观串行），从宏观角度来看，多个线程在同时进行（宏观并行）。对于双核或双核以上的CPU来说，可以做到真正做到微观并行。

### 实现Runnable接口
```java
public class ThreadTest2 {
	public static void main(String[] args) {
		Thread t1 = new Thread(new MyThread());
		Thread t2 = new Thread(new MyThread2());
		
		t1.start();
		t2.start();
	}
}
```
```java
class MyThread implements Runnable {
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("hello:" + i);
		}
	}

}
```
```java
class MyThread2 implements Runnable {
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("welcome:" + i);
		}
	}
}
```
## Thread 类源码分析
1. Thread 类也实现了 Runnable 接口，因此实现了Runnable 接口中的 run 方法；
2. 当生成一个线程对象时，如果没有为其设定名字，那么线程对象的名字将使用如下形式：Thread-number，该number是自动增加的，并被所有的 Thread 对象所共享（因为它是 static 的成员变量）。
3. 当使用第一种方式来生成线程对象时，我们需要重写run 方法，因为 Thread 类的run 方法此时什么事情都不做。当使用第二种方式来生成线程对象时，我们需要实现 Runnable 接口的 run 方法，然后使用 new Thread(new MyThread())（假设MyThread已经实现了runnable接口）来生成线程对象，这时的线程对象 run 方法就会调用 MyThread 类的 run 方法，这样我们自己编写的run方法就执行了。

## 线程的生命周期
- 线程的生命周期可分为四个状态： 1. 创建状态； 2. 可运行状态； 3. 不可运行状态； 4. 消亡状态。
![](http://cloudnotes.nos-eastchina1.126.net/20181226112217-419275.jpg)
```java
public class ThreadTest3
{
	public static void main(String[] args)
	{
		Runnable r = new HelloThread();
		
		Thread t1 = new Thread(r);
		
		//r = new HelloThread();
		
		Thread t2 = new Thread(r);
		
		t1.start();
		t2.start();
	}
}
```
```java
class HelloThread implements Runnable
{
	int i;
	
	@Override
	public void run()
	{
		int i = 0;
		
		while(true)
		{
			System.out.println("number: " + this.i++);
			
			try
			{
				Thread.sleep((long)(Math.random() * 1000));
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
			
			if(50 == this.i)
			{
				break;
			}
		}
	}
}
```
- 关于成员变量与局部变量： 如果一个变量是成员变量，那么多个线程对同一个对象的成员变量进行操作时，他们对该成员变量是彼此影响的（也就是说一个线程对成员变量的改变会影响到另一个线程） 。如果一个变量是局部变量，那么每个线程都会有一个该局部变量的拷贝，一个线程对该局部变量的改变不会影响到其他的线程。
- 停止线程的方式：不能使用 Thread 类的 stop 方法来终止线程的执行。一般要设定一个变量，在 run 方法中是一个循环，循环每次检查该变量，如果满足条件则继续执行，否则跳出循环，线程结束。
- 不能依靠线程的优先级来决定线程的执行顺序。

## 多线程的同步
- 在多线程环境中，可能会有两个甚至更多的线程是同时访问一个有限的资源，这样会造成资源的冲突。解决方法：在线程使用一个资源时为其加锁即可。访问资源的第一个线程为其加上锁以后，其他线程便不能再使用那个资源，除非被解锁。
- synchronized 关键字：当 synchronized 关键字修饰一个方法的时候，该方法叫做同步方法。
```java
public class ThreadTest4
{
	public static void main(String[] args)
	{
		Example example = new Example();
		
		Thread t1 = new TheThread(example);
		
		example = new Example();
		
		Thread t2 = new TheThread2(example);
		
		t1.start();
		t2.start();
	}
}
```
```java
class Example
{
	public synchronized void execute()
	{
		for(int i = 0; i < 20; i++)
		{
			try
			{
				Thread.sleep((long)(Math.random() * 1000));
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
			
			System.out.println("hello: " + i);
		}
	}
	
	public synchronized static void execute2()
	{
		for(int i = 0; i < 20; i++)
		{
			try
			{
				Thread.sleep((long)(Math.random() * 1000));
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
			
			System.out.println("world: " + i);
		}
	}
}
```
```java
class TheThread extends Thread
{
	private Example example;
	
	public TheThread(Example example)
	{	
		this.example = example;
	}
	
	@Override
	public void run()
	{
		this.example.execute();
	}
}
```
```java
class TheThread2 extends Thread
{
	private Example example;
	
	public TheThread2(Example example)
	{	
		this.example = example;
	}
	
	@Override
	public void run()
	{
		this.example.execute2();
	}
}
```
- Java 中的每个对象都有一个锁（lock）或者叫做监视器（monitor），当访问某个对象的 synchronized 方法时，表示将该对象上锁，此时其他任何线程都无法再去访问该 synchronized 方法了，直到之前的那个线程执行方法完毕后（或者是抛出了异常），那么将该对象的锁释放掉，其他线程才有可能再去访问该 synchronized 方法。
- 如果一个对象有多个 synchronized 方法，某一时刻某个线程已经进入到了某个 synchronized 方法，那么在该方法没有执行完毕前，其他线程都是无法访问该对象的任何 synchronized 方法的。
- 如果某个 synchronized 方法是 static 的，那么当线程访问该方法时，它锁的并不是 synchronized 方法所在的对象，而是 synchronized 方法所在的对象所对应的 Class 对象，因为java中无论一个类有多少个对象，这些对象都会对应唯一一个 Class 对象，因此当线程分别访问同一个类的两个对象的 static，synchronized方法时，他们的执行顺序也是顺序的，也就是说一个线程先去执行方法，执行完毕后另一个线程才开始执行。
- synchronized 块，写法： synchronized(object){.....}。 表示线程在执行的时候会对 object 对象上锁。
- synchronized 方法是中粗粒度的并发控制，某一时刻，只能有一个线程执行该 synchronized方法；synchronized块则是一种细粒度的并发控制，只会将块中的代码同步，位于方法内、synchronized 块之外的代码是可以被多个线程同时访问到的。
![](http://cloudnotes.nos-eastchina1.126.net/20181226113659-117852.jpg)
## 线程间的相互作用
使用wait()、notify()
```java
public class Sample
{
	private int number;

	public synchronized void increase()
	{
//		if (0 != number)		//两个线程的时候用if没有问题，多了就有问题了
		while (0 != number)
		{
			try
			{
				wait();
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
		}

		number++;

		System.out.println(number);

		notify();
	}

	public synchronized void decrease()
	{
//		if (0 == number)
		while (0 == number)
		{
			try
			{
				wait();
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
		}

		number--;

		System.out.println(number);

		notify();
	}
}
```
```java
public class IncreaseThread extends Thread
{
	private Sample sample;
	
	public IncreaseThread(Sample sample)
	{
		this.sample = sample;
	}
	
	@Override
	public void run()
	{
		for(int i = 0; i < 20; i++)
		{
			try
			{
				Thread.sleep((long)(Math.random() * 1000));
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
			
			sample.increase();
		}
	}
	
}
```
```java
public class DecreaseThread extends Thread
{
	private Sample sample;
	
	public DecreaseThread(Sample sample)
	{
		this.sample = sample;
	}
	
	@Override
	public void run()
	{
		for(int i = 0; i < 20; i++)
		{
			try
			{
				Thread.sleep((long)(Math.random() * 1000));
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
			
			sample.decrease();
		}
	}
}
```
```java
public class MainTest
{
	public static void main(String[] args)
	{
		Sample sample = new Sample();
		
		Thread t1 = new IncreaseThread(sample);
		Thread t2 = new DecreaseThread(sample);
		
		Thread t3 = new IncreaseThread(sample);
		Thread t4 = new DecreaseThread(sample);
		
		Thread t5 = new IncreaseThread(sample);
		Thread t6 = new DecreaseThread(sample);
		
		t1.start();
		t2.start();
		t3.start();
		t4.start();
		t5.start();
		t6.start();
		
	}
}
```
- wait 与 notify 方法都是定义在 Object 类中，而且是 final 的，因此会被所有的 java类所继承并且无法重写。这两个方法要求在调用时线程应该已经获得了对象的锁，因此对这两个方法的调用需要放在 synchronized 方法或块当中。当线程执行了 wait 方法时，它会释放掉对象的锁。
- wait 和 sleep 区别：另一个会导致线程暂停的方法就是 Thread 类的 sleep 方法，它会导致线程睡眠指定的毫秒数，但线程在睡眠过程中是不会释放掉对象的锁的。
