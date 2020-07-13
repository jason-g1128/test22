---
title: "线程"
date: 2020-07-12T13:44:11+08:00
draft: true
---

# 线程

## 主要内容

*   线程基础
*   线程池

## 教学目标

-[ ] 能够理解线程概念

-[ ] 学会使用线程的创建方式

-[ ] 能够理解线程的状态

-[ ] 能够理解线程通信概念

-[ ] 能够理解等待唤醒机制

-[ ] 能够描述Java中线程池运行原理

# 第一章 线程基础

## 1.1 线程简介

#### 概念:

现代操作系统在运行一个程序时，会为其创建一个进程。例如，启动一个Java程序，操作系统就会创建一个Java进程。现代操作系统调度的最小单元是线程，也叫轻量级进程（LightWeight Process），在一个进程里可以创建多个线程，这些线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。处理器在这些线程上高速切换，让使用者感觉到这些线程在同时执行。

Java线程的实现方式：

Java线程使用操作系统的内核线程实现，内核线程（Kernel-Level Thread, KLT）是直接由操作系统内核（Kernel，内核）支持的线程，这种线程由内核来完成线程切换，内核通过操纵调度器（Scheduler）对线程进行调度，并负责将线程的任务映射到各个处理器上。每个内核线程可以视为内核的一个分身（孙悟空的分身术），这样操作系统就有能力同时处理多件事情，支持多线程的内核就叫做多线程内核（Muti-Threads Kernel)。

[^用户线程]: java在jdk 1.2之前基于用户线程实现，在1.2之后，基于操作系统的原生线程模型来实现.用户线程指不需要内核支持而在用户程序中实现的线程，其不依赖于操作系统核心，应用进程利用线程库提供创建、同步、调度和管理线程的函数来控制用户线程。不需要用户态/核心态切换，速度快，操作系统内核不知道多线程的存在，因此一个线程阻塞将使得整个进程（包括它的所有线程）阻塞。使用用户线程实现的程序一般都比较复杂，java曾经用过，不过最后还是放弃了。

Java程序如何使用内核线程：

程序一般通过使用内核线程的高级接口-----轻量级进程（Light Weight Process, LWP），也就是我们通常意义上的线程。每个LWP都由一个内核线程支持。也就是说任何时候使用Java代码创建线程，调用Thread.start()的时候，都是通过LWP接口创建了KLT内核线程，然后通过OS的Thread Scheduler对内核线程进行调度分配CPU。线程模型如下图所示：

![1589742311569](assets\1589742311569.png)

内核线程的优点：

（1）每一个内核线程都是独立的轻量级进程，一个线程的阻塞不会影响整个进程的工作。

内核线程的缺点：

（1）由于是基于内核线程实现，各种线程的操作，如创建、析构、中断、休眠和同步，都需要系统调度（频繁从用户态切换进内核态），而系统调度的代价相对较高；

（2）占用内核资源，同时轻量级进程的数量有限。

主内存与工作内存：

JAVA内存模型规定了所有的变量都存储在主内存（Main Memory）中。所有的线程都有自己的工作内存，工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在工作内存中执行，而不能直接读写主内存中的变量。同时，线程之间也无法读写对方的工作内存。关系图：
![1589742638665](assets\1589742638665.png)

## 1.2 实现线程的方式

### 1.2.1 Runnable接口

一个接口，定义了一个run()方法，重写run()实现线程的主体逻辑

### 1.2.2 Thread类

Java中线程的抽象，将继承Runnable类的对象放入Thread中，Thread中定义很多控制管理线程的方法

```
为什么通常创建线程使用Runnable?
第一点：结构简单，层次分明，Runnable中只有一个run(),实现线程的主体逻辑只需要在run()中实现即可
第二点：不占用java单一继承的名额,在实现Runnable接口的同时还能继承其他的类
```

#### 1.2.2.1 Thread类的常用方法

1. setName()/getName() 设置/获取线程名称

   由于在一个进程中可能有多个线程，而多线程的运行状态又是不确定的，即不知道在多线程中当前执行的线程是哪个线程，所以在多线程操作中需要有一个明确的标识符标识出当前线程对象的信息，这个信息往往通过线程的名称来描述.当然也可以通过Thread的构造函数来设置**public Thread(Runnable target,String name)**

   若没有手动设置线程名称时，会自动分配一个线程的名称，例如自动分配线程名称为Thread-0

   需要注意的是，由于设置线程名称是为了区分当前正在执行的线程是哪一个线程，所以在设置线程名称时应避免重复

2. sleep(long millis) 线程休眠

   指的是让线程暂停执行，等到预计时间之后再恢复执行

   （1）线程休眠会交出CPU，让CPU去执行其他的任务。

   （2）调用sleep()方法让线程进入休眠状态后，sleep()方法并不会释放锁，即当前线程持有某个对象锁时，即使调用sleep()方法其他线程也无法访问这个对象。

   （3）调用sleep()方法让线程从运行状态转换为阻塞状态；sleep()方法调用结束后，线程从阻塞状态转换为可执行状态。

   （4）sleep()方法的休眠时间是以毫秒作为单位。

3. yield() 线程让步

   暂停当前正在执行的线程对象，并执行其他线程。使用该方法，可以防止线程对CPU的过度使用，提高系统性能。

   （1）调用yield()方法让当前线程交出CPU权限，让CPU去执行其他线程。

   （2）yield()方法和sleep()方法类似，不会释放锁，但yield()方法不能控制具体交出CPU的时间。

   （3）yield()方法只能让拥有相同优先级的线程获取CPU执行的机会。

   （4）使用yield()方法不会让线程进入阻塞状态，而是让线程从运行状态转换为就绪状态，只需要等待重新获取CPU执行的机会。

   案例:

   ```java
   class MyThread implements Runnable{
   	@Override
   	public void run() {
   		for(int i=0;i<5;i++)
   		{
   			//使用Thread类的yield()方法
   			Thread.yield();
   			System.out.println("当前线程："+Thread.currentThread().getName()+"-----i="+i);
   		}
   	}
   }
   public class Test1 {
   	public static void main(String[] args){
   		MyThread myThread=new MyThread();
   		//利用myThread对象分别创建三个线程
   		Thread thread1=new Thread(myThread);
   		thread1.start();
   		Thread thread2=new Thread(myThread);
   		thread2.start();
   		Thread thread3=new Thread(myThread);
   		thread3.start();
   	}
   }
   ```

4. join()/join(long time) 等待线程终止

   A线程调用B线程的join()方法，将会使A等待B执行，直到B线程终止。如果传入time参数，将会使A等待B执行time的时间，如果time时间到达，将会切换进A线程，继续执行A线程。

   案例:

   ```java
   class MyThread implements Runnable{
   	@Override
   	public void run() {
   		for(int i=0;i<2;i++)
   		{
   			try {
   				Thread.sleep(3000);
   			} catch (InterruptedException e) {
   				e.printStackTrace();
   			}
   			System.out.println("当前线程："+Thread.currentThread().getName()+"-----i="+i);
   		}
   	}
   }
   public class Test2 {
   	public static void main(String[] args) throws InterruptedException{
   		MyThread myThread=new MyThread();
   		Thread thread1=new Thread(myThread,"自己创建的线程");
   		thread1.start();
   		System.out.println("主线程："+Thread.currentThread().getName());
   		//线程对象thread1调用join()方法
   		thread1.join();
   		System.out.println("代码结束");
   	}
   }
   ```

   运行结果如下所示：

   ```
   主线程：main
   当前线程：自己创建的线程-----i=0
   当前线程：自己创建的线程-----i=1
   代码结束
   ```

   若不调用join()方法的话，运行结果如下所示：

   ```
   主线程：main
   代码结束
   当前线程：自己创建的线程-----i=0
   当前线程：自己创建的线程-----i=1
   ```

5. 线程停止

   多线程中停止线程有三种方式：

   （1）设置标记位，让线程正常停止。

   ```java
   class MyThread implements Runnable{
   	//设置标记位
   	private boolean flag=true;
   	public void setFlag(boolean flag) {
   		this.flag = flag;
   	}
   	@Override
   	public void run() {
   		int i=0;
   		while(flag)
   		{
   			System.out.println("第"+(i++)+"次执行-----"+"线程名称:"+Thread.currentThread().getName());
   		}
   	}
   }
   public class Test3 {
   	public static void main(String[] args) throws InterruptedException{
   		MyThread myThread=new MyThread();
   		Thread thread1=new Thread(myThread,"自己创建的线程");
   		thread1.start();
   		//让主线程sleep一毫秒
   		Thread.sleep(1);
   		//修改标记位的值，让自己创建的线程停止
   		myThread.setFlag(false);
   		System.out.println("代码结束");
   	}
   }
   ```

   运行结果如下所示：

   ```
   第0次执行-----线程名称:自己创建的线程
   第1次执行-----线程名称:自己创建的线程
   第2次执行-----线程名称:自己创建的线程
   第3次执行-----线程名称:自己创建的线程
   第4次执行-----线程名称:自己创建的线程
   第5次执行-----线程名称:自己创建的线程
   第6次执行-----线程名称:自己创建的线程
   第7次执行-----线程名称:自己创建的线程
   第8次执行-----线程名称:自己创建的线程
   第9次执行-----线程名称:自己创建的线程
   第10次执行-----线程名称:自己创建的线程
   第11次执行-----线程名称:自己创建的线程
   第12次执行-----线程名称:自己创建的线程
   第13次执行-----线程名称:自己创建的线程
   第14次执行-----线程名称:自己创建的线程
   第15次执行-----线程名称:自己创建的线程
   第16次执行-----线程名称:自己创建的线程
   第17次执行-----线程名称:自己创建的线程
   第18次执行-----线程名称:自己创建的线程
   第19次执行-----线程名称:自己创建的线程
   第20次执行-----线程名称:自己创建的线程
   第21次执行-----线程名称:自己创建的线程
   第22次执行-----线程名称:自己创建的线程
   第23次执行-----线程名称:自己创建的线程
   第24次执行-----线程名称:自己创建的线程
   第25次执行-----线程名称:自己创建的线程
   第26次执行-----线程名称:自己创建的线程
   代码结束
   ```

   （2）使用stop()方法强制使线程退出，但是使用该方法不安全，已经被废弃了！

   ```java
   class MyThread implements Runnable{
   	@Override
   	public void run() {
   		for(int i=0;i<100;i++)
   		{
   			System.out.println("线程名称:"+Thread.currentThread().getName()+"------i="+i);
   		}
   	}
   }
   public class Test6 {
   	public static void main(String[] args) throws InterruptedException{
   		MyThread myThread=new MyThread();
   		Thread thread1=new Thread(myThread,"自己创建的线程");
   		thread1.start();
   		//让主线程sleep2毫秒
   		Thread.sleep(2);
   		//调用已被弃用的stop()方法去强制让线程退出
   		thread1.stop();
   		System.out.println("代码结束");
   	}
   }
   ```

   **某次**运行结果如下所示：

   ```
   线程名称:自己创建的线程------i=0
   线程名称:自己创建的线程------i=1
   线程名称:自己创建的线程------i=2
   线程名称:自己创建的线程------i=3
   线程名称:自己创建的线程------i=4
   线程名称:自己创建的线程------i=5
   线程名称:自己创建的线程------i=6
   线程名称:自己创建的线程------i=7
   线程名称:自己创建的线程------i=8
   线程名称:自己创建的线程------i=9线程名称:自己创建的线程------i=9代码结束
   
   ```

   从上述代码和运行结果可以看出，原本线程对象thread1的run()方法中应该执行100次语句“System.out.println("线程名称:"+Thread.currentThread().getName()+"------i="+i);”，但现在没有执行够100次，所以说stop()方法起到了让线程终止的作用。再从运行结果上可以看出，i=9被执行了两次且没有换行，这就体现了调用stop()方法的不安全性！

   stop()方法为什么不安全？

   因为stop()方法会解除由线程获得的所有锁，当在一个线程对象上调用stop()方法时，这个线程对象所运行的线程会立即停止，假如一个线程正在执行同步方法：

   ```java
   public synchronized void fun(){
   	x=3;
   	y=4;
   }
   
   ```

   由于方法是同步的，多线程访问时总能保证x,y被同时赋值，而如果一个线程正在执行到x=3;时，被调用的stop()方法使得线程即使在同步方法中也要停止，这就造成了数据的不完整性。故，stop()方法不安全，已经被废弃，不建议使用！

   （3）使用Thread类的interrupt()方法中断线程。

   ```java
   class MyThread implements Runnable{
   	@Override
   	public void run() {
   		int i=0;
   		while(true)
   		{
   			//使用sleep()方法，使得线程由运行状态转换为阻塞状态
   			try {
   				Thread.sleep(1000);
   				//调用isInterrupted()方法，用于判断当前线程是否被中断
   				boolean bool=Thread.currentThread().isInterrupted();
   				if(bool) {
   					System.out.println("非阻塞状态下执行该操作，当前线程被中断!");
   					break;
   				}
   				System.out.println("第"+(i++)+"次执行"+" 线程名称："+Thread.currentThread().getName());
   			} catch (InterruptedException e) {
   				System.out.println("退出了！");
   				//这里退出了阻塞状态，且中断标志bool被系统自动清除设置为false，所以此处的bool为false
   				boolean bool=Thread.currentThread().isInterrupted();
   				System.out.println(bool);
   				//退出run()方法，中断进程
   				return;
   			}
   		}
   	}
   }
   public class Test6 {
   	public static void main(String[] args) throws InterruptedException{
   		MyThread myThread=new MyThread();
   		Thread thread1=new Thread(myThread,"自己创建的线程");
   		thread1.start();
   		//让主线程sleep三秒
   		Thread.sleep(3000);
   		//调用interrupt()方法
   		thread1.interrupt();
   		 System.out.println("代码结束");
   	}
   }
   
   ```

   运行结果如下所示 ：

   ```
   第0次执行 线程名称：自己创建的线程
   第1次执行 线程名称：自己创建的线程
   第2次执行 线程名称：自己创建的线程
   代码结束
   退出了！
   false
   
   ```

   （1）interrupt()方法只是改变中断状态而已，它不会中断一个正在运行的线程。具体来说就是，调用interrupt()方法只会给线程设置一个为true的中断标志，而设置之后，则根据线程当前状态进行不同的后续操作。

   （2）如果线程的当前状态处于非阻塞状态，那么仅仅将线程的中断标志设置为true而已；

   （3）如果线程的当前状态处于阻塞状态，那么将在中断标志设置为true后，还会出现wait()、sleep()、join()方法之一引起的阻塞，那么会将线程的中断标志位重新设置为false，并抛出一个InterruptedException异常。

   （4）如果在中断时，线程正处于非阻塞状态，则将中断标志修改为true，而在此基础上，一旦进入阻塞状态，则按照阻塞状态的情况来进行处理。例如，一个线程在运行状态时，其中断标志设置为true之后，一旦线程调用了wait()、sleep()、join()方法中的一种，立马抛出一个InterruptedException异常，且中断标志被程序自动清除，重新设置为false。

   总结：调用Thread类的interrupted()方法，其本质只是设置该线程的中断标志，将中断标志设置为true，并根据线程状态决定是否抛出异常。因此，通过interrupted()方法真正实现线程的中断原理是 ：开发人员根据中断标志的具体值来决定如何退出线程。

### 1.2.3 Callable

Callable:前面两种实现线程的方式中run()方法都是void的类型，没有返回值，线程在运行的过程中无法得知运行结果，如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果，这样使用起来就比较麻烦。Callable类在Runnable类的基础上增加了返回值和异常两点，是Runnable的增强版

Callable是一个接口,里面只有一个call()方法

![1589707709352](assets\1589707709352.png)

call()使用泛型作为返回值，并且抛出Exception异常

#### Future

Callable里面只是定义了一个任务执行的方法，通过重写call()方法，来编写任务逻辑主体。Future主要用来获取任务结果或者中断任务。

在Future接口中声明了5个方法，下面依次解释每个方法的作用：

cancel方法:用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

isCancelled方法:表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

isDone方法:表示任务是否已经完成，若任务完成，则返回true；

get()方法:用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

get(long timeout, TimeUnit unit)方法:用来获取执行结果，如果在指定时间内，还没获取到结果抛出超时异常。

也就是说Future提供了三种功能：

　　1）判断任务是否完成；

　　2）能够中断任务；

　　3）能够获取任务执行结果。

　　因为Future只是一个接口，所以是无法直接用来创建对象使用的，因此就有了下面的FutureTask。

#### FutureTask

FutureTask继承RunnableFuture

![img](assets\16015500-a44901e7d7ce4f67.png)

RunnableFuture继承Runnable和Future

![img](assets\16015500-87b8cec86e5ff7ea.png)

在FutureTask的成员变量中定义一个state,这个state管理提交到FutureTask中的线程或者任务的生命周期。

![img](assets\16015500-9c8c3540c17a0c79.png)

生命周期有下面几种情况

\* NEW -> COMPLETING -> NORMAL (通常情况)

\* NEW -> COMPLETING -> EXCEPTIONAL

\* NEW -> CANCELLED

\* NEW -> INTERRUPTING -> INTERRUPTED

#### 两种使用Callable的方式

第一种Callable和FutureTask

FutureTask的构造函数可以提交实现Callable类的对象

![img](assets\16015500-02f8d982f52c3c4e.png)

例子：Task类继承Callable

```java
public class Task implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println("运行call.....");
        int result = 0;
        for (int i=0;i<1000;i++){
            result += i;
        }
        return result;
    }
}

```

```java
public class Main {
    public static void main(String[] args) {
        Task task = new Task();
        FutureTask<Integer> ft = new FutureTask<Integer>(task);
        Thread thread = new Thread(ft,"FutureTask-Thread");
        thread.start();
        System.out.println("运行main....");

        int result = 0;
        try {
            result = ft.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println("执行结果:"+result);

    }
}

```

提示:由于FutureTask继承Runnable，所以可以把实现Callable类的对象提交到FutureTask，然后把FutureTask对象提交到Thread进行管理和启动。

```
执行结果如下:
运行main....
运行call.....
执行结果:499500

```

第二种：Callable和ExecutorService(通常情况)

在ExecutorService中的submit()方法，用于提交任务

例子:Task类不变，修改Main类

```java
public class Main {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> future = service.submit(task);
        service.shutdown();
        System.out.println("运行main....");

        int result = 0;
        try {
            result = future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println("执行结果:"+result);

    }
}

```

Task类不变，修改Main类

运行结果与第一种方式一致

## 1.3 线程的状态

1：初始状态(NEW)：创建一个线程对象，但是还没有调用start()。

2：运行状态(RUNNABLE)：Java中将就绪和运行两种状态统称为“运行中”。

3：阻塞状态(BLOCKED)：表示线程阻塞于锁。

4：等待状态(WAITING)：表示线程进入等待状态，进入该状态的线程必须等待其他线程做出一些特定动作(如通知或者中断)。

5：等待超时(TIME_WAITING)：表示线程进入等待状态，与WAITING不同的是会在指定时间自行返回。

6：终止状态(TERMINATED)：表示线程执行完毕。

![img](assets\16015500-8a7f3be3537e5c50.jpg)



### 1.3.1 查看线程状态的小工具-jstack

使用jstack查看一个进程中所有线程的运行状态

编写一个一直运行的线程

运行该程序，然后cmd 打开命令提示符，输入jps查看各个进程的PID，然后使用jstack PID 就可以查看到一个进程中不同线程的运行状态（如果看到不到，跳转到JDK的bin目录下执行jps和jstack命令）。

## 1.4 线程的种类

 在Java中有两类线程：User Thread(用户线程)、Daemon Thread(守护线程) 

 用个比较通俗的比如，任何一个守护线程都是整个JVM中所有非守护线程的保姆

 只要当前JVM实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。
 Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)，它就是一个很称职的守护者。

 User和Daemon两者几乎没有区别，唯一的不同之处就在于虚拟机的离开：如果  User Thread已经全部退出运行了，只剩下Daemon Thread存在了，虚拟机也就退出了。  因为没有了被守护者，Daemon也就没有工作可做了，也就没有继续运行程序的必要了。

值得一提的是，守护线程并非只有虚拟机内部提供，用户在编写程序时也可以自己设置守护线程。通过Thread.setDaemon(true)设置线程为守护线程。

这里有几点需要注意：

(1) Thread.setDaemon(true)必须在thread.start()之前设置，否则会抛出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。 

(2) 在Daemon线程中产生的新线程也是Daemon的。 

(3) 不要认为所有的应用都可以分配给Daemon来进行服务，比如读写操作或者计算逻辑

因为你不可能知道在所有的User完成之前，Daemon是否已经完成了预期的服务任务。一旦User退出了，可能大量数据还没有来得及读入或写出，计算任务也可能多次运行结果不一样。这对程序是毁灭性的。造成这个结果的原因就是：一旦所有User Thread离开了，虚拟机也就退出运行了

案例:

```java
class MyDeamon implements Runnable{
    public void run(){   
        try{
            System.out.println("开始准备写文件");
            Thread.sleep(1000);//守护线程阻塞1秒后运行
            File f=new File("d:\\daemon.txt");
            FileOutputStream os=new FileOutputStream(f,true);
            os.write("daemon".getBytes());
            os.close();
        }
        catch(IOException e1){
            e1.printStackTrace();
        }
        catch(InterruptedException e2){
            e2.printStackTrace();
        }
    }   
}   
public class TestDeamon{
    public static void main(String[] args) throws InterruptedException   
    {   
        Runnable tr=new MyDeamon();
        Thread thread=new Thread(tr);   
        thread.setDaemon(true); //设置守护线程
        thread.start(); //开始执行守护线程
    }   
}   
//运行结果：daemon.txt没有生成

```

## 1.5 线程间通信

**概念：**多个线程在处理同一个资源，但是处理的动作（线程的任务）却不相同,为了让这同一资源协调一致(数据的一致性),这多个线程就必须进行协商(通信).

比如：线程A用来生成包子的，线程B用来吃包子的，包子可以理解为同一资源，线程A与线程B处理的动作，一个是生产，一个是消费，那么线程A与线程B之间就存在线程通信问题。

![](img\线程间通信.bmp)

**为什么要处理线程间通信：**

多个线程并发执行时, 在默认情况下CPU是随机切换线程的，当我们需要多个线程来共同完成一件任务，并且我们希望他们有规律的执行, 那么多线程之间需要一些协调通信，以此来帮我们达到多线程共同操作一份数据。

**如何保证线程间通信有效利用资源：**

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作。 就是多个线程在操作同一份数据时， 避免对同一共享变量的争夺。也就是我们需要通过一定的手段使各个线程能有效的利用资源。而这种手段即—— **等待唤醒机制。**

## 1.6 等待唤醒机制

**什么是等待唤醒机制**

这是多个线程间的一种**协作**机制。谈到线程我们经常想到的是线程间的**竞争（race）**，比如去争夺锁，但这并不是故事的全部，线程间也会有协作机制。就好比在公司里你和你的同事们，你们可能存在晋升时的竞争，但更多时候你们更多是一起合作以完成某些任务。

就是在一个线程进行了规定操作后，就进入等待状态（**wait()**）， 等待其他线程执行完他们的指定代码过后 再将其唤醒（**notify()**）;在有多个线程进行等待时， 如果需要，可以使用 notifyAll()来唤醒所有的等待线程。

wait/notify 就是线程间的一种协作机制。

**等待唤醒中的方法**

等待唤醒机制就是用于解决线程间通信的问题的，使用到的3个方法的含义如下：

1. wait：线程不再活动，不再参与调度，进入 wait set 中，因此不会浪费 CPU 资源，也不会去竞争锁了，这时的线程状态即是 WAITING。它还要等着别的线程执行一个**特别的动作**，也即是“**通知（notify）**”在这个对象上等待的线程从wait set 中释放出来，重新进入到调度队列（ready queue）中
2. notify：则选取所通知对象的 wait set 中的一个线程释放；例如，餐馆有空位置后，等候就餐最久的顾客最先入座。
3. notifyAll：则释放所通知对象的 wait set 上的全部线程。

>注意：
>
>哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不持有锁，所以她需要再次尝试去获取锁（很可能面临其它线程的竞争），成功后才能在当初调用 wait 方法之后的地方恢复执行。
>
>总结如下：
>
>- 如果能获取锁，线程就从 WAITING 状态变成 RUNNABLE 状态；
>- 否则，从 wait set 出来，又进入 entry set，线程就从 WAITING 状态又变成 BLOCKED 状态

**调用wait和notify方法需要注意的细节**

1. wait方法与notify方法必须要由同一个锁对象调用。因为：对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
2. wait方法与notify方法是属于Object类的方法的。因为：锁对象可以是任意对象，而任意对象的所属类都是继承了Object类的。
3. wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为：必须要通过锁对象调用这2个方法。

## 1.7 synchronized和Lock

在java中，为解决同步问题，很多时候都会使用到synchronized和Lock，这两者都是在多线程并发时候常使用的锁机制。

### 1.7.1 synchronized

synchronized是java中的一个**关键字**，也就是说是java内置的一个特性。当一个线程访问一个被synchronized修饰的代码块，会自动获取对应的一个锁，并在执行该代码块时，其他线程想访问这个代码块，会一直处于等待状态，等该线程释放锁后，其他线程进行资源竞争，竞争获取到锁的线程才能访问该代码块。

线程释放synchronized修饰的代码块锁的方式有两种：

1. 该线程执行完对应代码块，自动释放锁。
2. 在执行该代码块是发生了异常，JVM会自动释放锁。

采用synchronized关键字来实现同步，会导致如果存在多个线程想执行该代码块，而当前获取到锁的线程又没有释放锁，可想而知，其他线程只有一直等待，这将严重影响执行效率。Lock锁机制的出现就是为了解决该现象。

### 1.7.2 Lock

Lock是一个java接口，通过这个接口可以实现同步，使用Lock时，用户必须手动进行锁的释放，否则容易出现死锁。

Lock与synchronized的区别

![img](assets\1222507-20190603205436977-1451348357.png)

ReentranLock是Lock的实现类。下面简单介绍一下ReentranLock与synchronized的区别：

1. Synchronized是一个同步锁。当一个线程A访问synchronized修饰的代码块时，线程A就会获取该代码块的锁，如果这时存在其他线程访问该代码块时，将会阻塞，但是不影响这些线程访问其他非同步代码块。

2. ReentranLock是可重入锁。该锁支持两种锁模式，公平锁和非公平锁。默认是非公平的。

   公平锁：当线程A获取访问该对象，获取到锁后，此时内部存在一个计数器num+1，其他线程想访问该对象，就会进行排队等待(等待队列最前一个线程处于待唤醒状态)，直到线程A释放锁(num  =  0)，此时会唤醒处于待唤醒状态的线程进行获取锁的操作。

   非公平锁：当线程A在释放锁后，等待对象的线程会进行资源竞争，竞争成功的线程将获取该锁，其他线程继续睡眠。

   公平锁是以FIFO的方式进行锁的竞争，但是非公平锁是无序的锁竞争，刚释放锁的线程很大程度上能比较快的获取到锁，队列中的线程只能等待，所以非公平锁可能会有“饥饿”的问题。但是重复的锁获取能减小线程之间的切换，而公平锁则是线程切换，这样对操作系统的影响是比较大的，所以非公平锁的吞吐量是大于公平锁的，这也是为什么JDK将非公平锁作为默认的实现。

下面是接口Lock的方法：

lock()：获取锁，如果锁被暂用则一直等待

unlock():释放锁

tryLock(): 注意返回类型是boolean，如果获取锁的时候锁被占用就返回false，否则返回true

tryLock(long time, TimeUnit unit)：比起tryLock()就是给了一个时间期限，保证等待参数时间

newCondition()   返回用来与此 Lock 实例一起使用的 Condition 实例,用于线程间通信。

测试案例:

```java
public class TestLock {
    // ReentrantLock为Lock的实现类
    private Lock lock = new ReentrantLock();

    /**
     * 测试使用lock 的 lock()方法 ：如果锁已经被其他线程获取，则等待
     * @param thread
     */
    public void testLock(Thread thread){
        try {
            // 1.获取锁
            lock.lock();
            System.out.println("线程 " + thread.getName() + " 获取了锁！");
            Thread.sleep(2000);
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            System.out.println("线程 " + thread.getName() + " 释放了锁！");
            // 必须在 finally 中释放锁，防止死锁
            lock.unlock();
        }
    }


    /**
     * 测试使用lock 的 tryLock()方法 ：如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false
     * @param thread
     */
    public void testTryLock(Thread thread){
        if(lock.tryLock()){// 如果获取到了锁
            try {
                System.out.println("线程 " + thread.getName() + " 获取了锁！");
                Thread.sleep(3000);
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                System.out.println("线程 " + thread.getName() + " 释放了锁！");
                // 必须在 finally 中释放锁，防止死锁
                lock.unlock();
            }
        }else {
            // 没有获取到锁
            System.out.println("线程 " + thread.getName() + " 没有获取到锁！");
        }
    }

    /**
     * 测试使用lock 的 tryLock(long time, TimeUnit unit)方法 ：和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，
     * 在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。
     * @param thread
     */
    public void testTryLock_time_unit(Thread thread){
        try {
            if(lock.tryLock(1000, TimeUnit.MILLISECONDS)){// 如果获取到了锁
                try {
                    System.out.println("线程 " + thread.getName() + " 获取了锁！");
                    //Thread.sleep(2000);
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    System.out.println("线程 " + thread.getName() + " 释放了锁！");
                    // 必须在 finally 中释放锁，防止死锁
                    lock.unlock();
                }
            }else {
                // 没有获取到锁
                System.out.println("线程 " + thread.getName() + " 没有获取到锁！");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args){
        TestLock testLock = new TestLock();
        Thread a = new Thread("A") {
            @Override
            public void run() {
                // 测试 lock()
//                testLock.testLock(Thread.currentThread());
                // 测试 tryLock()
//                testLock.testTryLock(Thread.currentThread());
                // 测试 tryLock(long time, TimeUnit unit)
                testLock.testTryLock_time_unit(Thread.currentThread());
            }
        };
        Thread b = new Thread("B") {
            @Override
            public void run() {
//                testLock.testLock(Thread.currentThread());
//                testLock.testTryLock(Thread.currentThread());
                testLock.testTryLock_time_unit(Thread.currentThread());
            }
        };
        a.start();
        b.start();
    }
}

```

公平锁/非公平锁测试案例:

```java
public class TestFair {
    public static void main(String[] args) {
        C c = new C();
        Thread t1 = new Thread(c);
        Thread t2 = new Thread(c);
        Thread t3 = new Thread(c);
        t1.start();
        t2.start();
        t3.start();

    }
}
class C implements Runnable{
    private static Lock lock = new ReentrantLock(true);
    @Override
    public void run() {
        for(int i=1;i<=10;i++){
            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName());
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }
    }
}

```

### 1.7.3 Lock和synchronized的选择

- synchronized是内置语言实现的关键字，Lock是为了实现更高级锁功能而提供的接口。
- Lock实现了tryLock等方法，线程可以不用一直等待。
- synchronized发生异常时自动释放占有的锁，Lock需要在finally块中手动释放锁。因此从安全性角度讲，既可以用Lock又可以用synchronized时(即不需要锁的更高级功能时)使用synchronized更保险。
- 线程激烈竞争时Lock的性能远优于synchronized，即有大量线程时推荐使用Lock。在竞争不激烈时，由于synchronized的编译器优化更好，性能更佳。

## 1.8 生产者与消费者问题

等待唤醒机制其实就是经典的“生产者与消费者”的问题。

就拿生产包子消费包子来说等待唤醒机制如何有效利用资源：

~~~java
包子铺线程生产包子，吃货线程消费包子。当包子没有时（包子状态为false），吃货线程等待，包子铺线程生产包子（即包子状态为true），并通知吃货线程（解除吃货的等待状态）,因为已经有包子了，那么包子铺线程进入等待状态。接下来，吃货线程能否进一步执行则取决于锁的获取情况。如果吃货获取到锁，那么就执行吃包子动作，包子吃完（包子状态为false），并通知包子铺线程（解除包子铺的等待状态）,吃货线程进入等待。包子铺线程能否进一步执行则取决于锁的获取情况。

~~~

**代码演示：**

包子资源类：

~~~java
public class BaoZi {
     String  pier ;
     String  xianer ;
     boolean  flag = false ;//包子资源 是否存在  包子资源状态
}

~~~

吃货线程类：

~~~java
public class ChiHuo extends Thread{
    private BaoZi bz;

    public ChiHuo(String name,BaoZi bz){
        super(name);
        this.bz = bz;
    }
    @Override
    public void run() {
        while(true){
            synchronized (bz){
                if(bz.flag == false){//没包子
                    try {
                        bz.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("吃货正在吃"+bz.pier+bz.xianer+"包子");
                bz.flag = false;
                bz.notify();
            }
        }
    }
}

~~~

包子铺线程类：

~~~java
public class BaoZiPu extends Thread {

    private BaoZi bz;

    public BaoZiPu(String name,BaoZi bz){
        super(name);
        this.bz = bz;
    }

    @Override
    public void run() {
        int count = 0;
        //造包子
        while(true){
            //同步
            synchronized (bz){
                if(bz.flag == true){//包子资源  存在
                    try {
                        bz.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                // 没有包子  造包子
                System.out.println("包子铺开始做包子");
                if(count%2 == 0){
                    // 冰皮  五仁
                    bz.pier = "冰皮";
                    bz.xianer = "五仁";
                }else{
                    // 薄皮  牛肉大葱
                    bz.pier = "薄皮";
                    bz.xianer = "牛肉大葱";
                }
                count++;

                bz.flag=true;
                System.out.println("包子造好了："+bz.pier+bz.xianer);
                System.out.println("吃货来吃吧");
                //唤醒等待线程 （吃货）
                bz.notify();
            }
        }
    }
}

~~~

测试类：

~~~java
public class Demo {
    public static void main(String[] args) {
        //等待唤醒案例
        BaoZi bz = new BaoZi();

        ChiHuo ch = new ChiHuo("吃货",bz);
        BaoZiPu bzp = new BaoZiPu("包子铺",bz);

        ch.start();
        bzp.start();
    }
}

~~~

执行效果：

~~~java
包子铺开始做包子
包子造好了：冰皮五仁
吃货来吃吧
吃货正在吃冰皮五仁包子
包子铺开始做包子
包子造好了：薄皮牛肉大葱
吃货来吃吧
吃货正在吃薄皮牛肉大葱包子
包子铺开始做包子
包子造好了：冰皮五仁
吃货来吃吧
吃货正在吃冰皮五仁包子

~~~

# 第二章 线程池

## 2.1 线程池思想概述

![](img\游泳池.jpg)

我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。

## 2.2 线程池概念

* **线程池：**其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。Java中的线程池是运用场景最多的并发框架，几乎所有需要异步或并发执行任务的程序都可以使用线程池。

由于线程池中有很多操作都是与优化资源相关的，我们在这里就不多赘述。我们通过一张图来了解线程池的工作原理：

![](img\线程池原理.bmp)

合理利用线程池能够带来三个好处：

1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. 提高响应速度。当任务到达时，任务可以不用等到线程创建就能立即执行。
3. 提高线程的可管理性。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。

什么时候使用线程池

1. 单个任务处理时间比较久

2. 需要处理的任务数量很大

## 2.3 线程池的使用

![img](assets\16015500-a2526cfaa1a933f6.png)

### 2.3.1 Exectutor  

Java的线程既是工作单元，也是执行机制。从JDK 5开始，把工作单元与执行机制分离开来。工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

在上层，Java多线程程序通常把应用分解为若干个任务，然后使用用户级的调度器（Executor框架）将这些任务映射为固定数量的线程；在底层，操作系统内核将这些线程映射到硬件处理器上。

### 2.3.2 Executor框架的结构

Executor框架主要由3大部分组成如下。

1. **任务**。包括被执行任务需要实现的接口：Runnable接口或Callable接口。

   Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。

2. **任务的执行**。包括任务执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor框架有两个关键类实现了ExecutorService接口（ThreadPoolExecutor和ScheduledThreadPoolExecutor）。

   Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具,它将任务的提交与任务的执行分离开来。真正的线程池接口是`java.util.concurrent.ExecutorService`。

   ThreadPoolExecutor是线程池的核心实现类，用来执行被提交的任务。

   ScheduledThreadPoolExecutor是另一个实现类，可以在给定的延迟后运行任务，或者定期执行任务。

3. **异步计算的结果**。包括接口Future和实现Future接口的FutureTask类，代表异步计算的结果。

### 2.3.3 Executor类和接口示意图

![img](assets\16015500-4dc0d547170f70db.png)

### 2.3.4 Executor框架的使用示意图

![img](assets\16015500-7f6d6bab677b51b8.png)

### 2.3.5 ThreadPoolExecutor

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。

ThreadPoolExecutor通常使用工厂类Executors来创建。Executors可以创建3种类型的：`SingleThreadExecutor`、`FixedThreadPool`和`CachedThreadPool`

1. **FixedThreadPool**。创建使用固定线程数的FixedThreadPool的API

   FixedThreadPool适用于为了满足资源管理的需求，而需要限制当前线程数量的应用场景，它适用于负载比较重的服务器。

2. **SingleThreadExecutor**。创建使用单个线程的SingleThreadExecutor的API。

   SingleThreadExecutor适用于需要保证顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的应用场景。

3. **CachedThreadPool**。创建一个会根据需要创建新线程的CachedThreadPool的API。

   CachedThreadPool是大小无界的线程池，适用于执行很多的短期异步任务的小程序，或者是负载较轻的服务器

```
注意：使用Executors来创建线程池，失去了线程池的灵活性，而且存在一定的隐患，根据阿里巴巴规范的提示，使用Executors来创建线程池存在资源耗尽的可能，因为使用Executors来创建线程池默认的最大容量是Integer.Max，也就是Integer的最大值作为线程池的最大容量，这样在程序中可能出现错误，导致创建了Integer.Max个线程，存在内存溢出的风险。所以在熟练掌握线程池原理后可以使用ThreadPoolExecutor根据添加不同的参数创建不同类型的线程池。

```

public ThreadPoolExecutor(int corePoolSize, 

​                                            int maximumPoolSize,

​                                            long keepAliveTime,

​                                             TimeUnit unit,

​                                             BlockingQueue workQueue,

​                                             ThreadFactory threadFactory,

​                                             RejectedExecutionHandler handler) 

下面解释构造函数的参数含义

**corePoolSize**：核心线程数量

**maximumPoolSize**：最大线程数量；

**workQueue**：等待队列，当任务提交时，如果线程池中的线程数量大于等于corePoolSize的时候，把该任务封装成一个Worker对象放入等待队列；

**keepAliveTime**：线程池维护线程所允许的空闲时间。当线程池中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了keepAliveTime；

**TimeUnit** ：时间级别

**threadFactory**：它是ThreadFactory类型的变量，用来创建新线程。

**handler**：它是RejectedExecutionHandler类型的变量，表示线程池的饱和策略。如果阻塞队列满了并且没有空闲的线程，这时如果继续提交任务，就需要采取一种策略处理该任务。

虽然使用Executors创建线程池存在一定的风险，但是在不是很复杂的场景下，合理使用Executors还是可行的。下面使用Executors来创建不同类型的线程池

#### 2.3.5.1 固定数量线程池（newFixedThreadPool）

Executors构造newFixedThreadPool方式

![img](assets\16015500-0b3d713bac891678.png)

查看源码

![img](assets\16015500-84d89f43d24bca17.png)

corePoolSize = maximumPoolSize =初始化的参数

workQueue：使用无界队列LinkedBlockingQueue链表阻塞队列

keepAliveTime = 0 由于使用无界队列LinkedBlockingQueue作为缓存队列，所以当corePoolSize满后，后面添加的线程任务都会添加到LinkedBlockingQueue中去，所以maximumPoolSize 就失去了意义，这样也就没有必要设置空闲时间

**使用无界队列的影响，这也是为什么使用Eexcutors来创建线程池存在一定风险的原因**

1）当线程池中的线程数达到corePoolSize后，新任务将在无界队列中等待，因此线程池中的线程数不会超过corePoolSize。

2）使用无界队列时maximumPoolSize将是一个无效参数。

3）使用无界队列时keepAliveTime将是一个无效参数。

4）由于使用无界队列，运行中的FixedThreadPool（未执行方法shutdown()或shutdownNow()）不会拒绝任务。

使用线程池中线程对象的步骤：

1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

代码示例:

~~~java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("教练来了： " + Thread.currentThread().getName());
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
public class FixedPoolTest {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.submit(r);
        // 再获取个线程对象，调用MyRunnable中的run()
        service.submit(r);
        service.submit(r);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}

~~~

运行结果:

```
我要一个教练
我要一个教练
教练来了： pool-1-thread-1
教练来了： pool-1-thread-2
教我游泳,交完后，教练回到了游泳池
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-1
教我游泳,交完后，教练回到了游泳池

```

为什么线程名称会重复：这正是线程池的原理，因为线程池会重复利用已创建的线程，当一个任务Runnable被挂载到线程池中的一个线程，这个任务执行完毕后，会有另一个任务继续挂载到这个线程上面，所以会出现线程名称重复。

#### 2.3.5.2 单例线程池（newSingleThreadExecutor）

Executors构造newSingleThreadExecutor方式

![img](assets\16015500-73200d75b1e3b715.png)

源代码

![img](assets\16015500-06b15fae6ab7e51c.png)

corePoolSize = maximumPoolSize =1  由于是单例线程池，所以线程池中是有一个重用的线程

workQueue：使用无界队列LinkedBlockingQueue链表阻塞队列

keepAliveTime:0 原因上面已经阐述

代码示例:

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("教练来了： " + Thread.currentThread().getName());
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
public class SinglePoolTest {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newSingleThreadExecutor();
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.submit(r);
        // 再获取个线程对象，调用MyRunnable中的run()
        service.submit(r);
        service.submit(r);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}

```

运行结果:

```
我要一个教练
教练来了： pool-1-thread-1
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-1
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-1
教我游泳,交完后，教练回到了游泳池

```

只有一个可重用的线程，任务的执行顺序和添加顺序一致

#### 2.3.5.3 缓存线程池（newCachedThreadPool）

Executors构造newCachedThreadPool方式

![img](assets\16015500-19ff018088a4c6e1.png)

源代码

![img](assets\16015500-e7cf72e43d80209e.png)

corePoolSize:0 表示线程池中没有核心线程，都是非核心线程

maximumPoolSize ：线程池容量Integer最大值

keepAliveTime：60秒 由于没有核心线程的存在，线程池中创建的线程都是非核心线程，所以设置空闲时间60秒，当非核心线程60秒后没有被重用，将会被销毁，如果没有线程提交给该线程池，超过空闲时间，该线程池就没有非空闲线程，那么该线程池也就不会消耗过多的资源，

workQueue:SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。

代码示例:

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("教练来了： " + Thread.currentThread().getName());
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
public class CachePoolTest {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newCachedThreadPool();
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.submit(r);
        // 再获取个线程对象，调用MyRunnable中的run()
        service.submit(r);
        service.submit(r);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}

```

执行结果:

```
我要一个教练
我要一个教练
我要一个教练
教练来了： pool-1-thread-1
教我游泳,交完后，教练回到了游泳池
教练来了： pool-1-thread-2
教我游泳,交完后，教练回到了游泳池
教练来了： pool-1-thread-3
教我游泳,交完后，教练回到了游泳池

```

可以看出每一个任务都安排了一个线程

### 2.3.6 ScheduledThreadPoolExecutor

ScheduledThreadPoolExecutor：通常用来创建定时线程任务的线程池，例如定时轮询数据库中的表的数据

ScheduledThreadPoolExecutor通常使用工厂类Executors来创建。Executors可以创建2种类型的ScheduledThreadPoolExecutor，如下。

1. **ScheduledThreadPoolExecutor**: 创建固定个数线程的ScheduledThreadPoolExecutor的API。

   ScheduledThreadPoolExecutor适用于需要多个后台线程执行周期任务，同时为了满足资源管理的需求而需要限制后台线程的数量的应用场景。

2. **SingleThreadScheduledExecutor**: 创建单个线程的SingleThreadScheduledExecutor的API。

   SingleThreadScheduledExecutor适用于需要单个后台线程执行周期任务，同时需要保证顺序地执行各个任务的应用场景。

#### 2.3.6.1 定时线程池（newScheduledThreadPool）

它主要用来在给定的延迟之后运行任务，或者定期执行任务，例如定时轮询数据库中的表的数据

Executors构造newScheduledThreadPool方式

![img](assets\16015500-c808740ab57f1c0e.png)

源代码

![img](assets\16015500-dc075e671c66036d.png)

workQeueu：delayWorkQueue，使用延迟队列作为缓存队列

任务提交方式

![img](assets\16015500-5f14d6a87f592285.png)

**schedule**(Callable<E> callable, long delay, TimeUnit unit);

callable：提交Callable或者Runnable任务

delay：延迟时间

unit：时间级别

该方法表示在给定的delay延迟时间后**执行一次，有返回结果**

**scheduleAtFixedRate**(Runnable command, long initialDelay, long period, TimeUnit unit);

command：提交Runnable任务

initialDelay：初始延迟时间

period：**表示这个任务连续执行的时间周期，第一个任务开始到第二个任务的开始，包含了任务的执行时间**

unit：时间级别

该方法在initialDelay时间后开始周期性的按period时间间隔执行任务

**scheduleWithFixedDelay**(Runnable command,long initialDelay,long delay,TimeUnit unit);

command：提交Runnable任务

initialDelay：初始延迟时间

delay:**表示延迟时间 第一个任务结束到第二个任务开始的时间间隔**

unit:时间级别

代码示例:

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("教练来了： " + Thread.currentThread().getName()+" "+sdf.format(new Date()));
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
public class SchedulePoolTest {
    public static void main(String[] args) {
        // 创建线程池对象
        ScheduledExecutorService service = Executors.newScheduledThreadPool(5);
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.scheduleWithFixedDelay(r,5,3, TimeUnit.SECONDS);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}

```

运行结果:

```
我要一个教练
教练来了： pool-1-thread-1 2020-05-19 20:03:04
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-1 2020-05-19 20:03:09
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-2 2020-05-19 20:03:14
教我游泳,交完后，教练回到了游泳池

```

可以看出任务以指定的时间间隔执行

#### 2.3.6.2 单例延迟线程池（newSingleThreadScheduledExecutor）

Executors构造newSingleThreadScheduledExecutor方式

![img](assets\16015500-29486026bd3e7594.png)

源代码

![img](assets\16015500-6017e5da94d8e7d1.png)

corePoolSize ：1由于是单例线程池，所以核心线程为1，线程池中只有一个重用线程

代码示例:

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("教练来了： " + Thread.currentThread().getName()+" "+sdf.format(new Date()));
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}

public class SingleSchedulePoolTest {
    public static void main(String[] args) {
        // 创建线程池对象
        ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.scheduleWithFixedDelay(r,5,3, TimeUnit.SECONDS);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}

```

运行结果:

```
我要一个教练
教练来了： pool-1-thread-1 2020-05-19 20:26:30
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-1 2020-05-19 20:26:35
教我游泳,交完后，教练回到了游泳池
我要一个教练
教练来了： pool-1-thread-1 2020-05-18 20:26:40
教我游泳,交完后，教练回到了游泳池

```

可以看出任务以指定的时间间隔执行

# 第三章 Java并发工具类

## 3.1 CountDownLatch

CountDownLatch允许一个或者多个线程等待其他线程完成操作。

在多线程中可能出现A线程需要等待B线程和C线程执行完毕才会执行，而A线程又不知道B和C线程具体什么时候能够结束，所以需要一种通知方式告诉A线程B和C线程已经执行完毕，在Java中可以通过先使A线程wait()，等待B和C线程执行完毕再notifyAll()唤醒A线程。

Thread类有一个join()方法可以实现这个功能，

join()：在一个线程中调用另一个线程的join()，会使得被调用join()的线程执行完毕，调用线程才会继续执行下去。

代码案例：main线程需等待thread1和thread2线程执行完毕才会继续执行

```java
public class TestJoin {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Runnable() {
            public void run() {
                System.out.println("Thread1 run......");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "Thread1");

        Thread thread2 = new Thread(new Runnable() {
            public void run() {
                System.out.println("Thread2 run......");
            }
        }, "Thread2");

        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();

        System.out.println("Main Thread run......");

    }
}

```

结果：

```
Thread1 run......
Thread2 run......
Main Thread run......

```

**CountDownLatch的使用**

CountDownLatch有着和join()相同的功能，但是比join()更加灵活，功能也更加强大。CountDownLatch可以在不同线程不同位置使用。

初始化

CountDownLatch countDownLatch = new CountDownLatch(int count);

count:表示一个计数值，每调用一次countDown()方法，该计数值会减一，当count=0时，其他线程执行完毕，唤醒主线程。

使用CountDownLatch 完成上面的例子

```
public class TestCountDownLatch  {
    public static void main(String[] args) throws InterruptedException {
        final CountDownLatch countDownLatch = new CountDownLatch(2);
        Thread thread1 = new Thread(new Runnable() {
            public void run() {
                System.out.println("Thread1 run......");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            }
        }, "Thread1");

        Thread thread2 = new Thread(new Runnable() {
            public void run() {
                try {
                    System.out.println("Thread2 run......");
                }finally {
                    countDownLatch.countDown();
                }
            }
        }, "Thread2");

        thread1.start();
        thread2.start();
        countDownLatch.await();//等待thread1和thread2线程执行完毕
        System.out.println("Main Thread run......");

    }
}

```

结果:

```
Thread1 run......
Thread2 run......
Main Thread run......

```

```
注意：使用CountDownLatch的时候确保计数值在最后为零，要不然主线程会一直等待下去，所以调用countDown()方法的时候，确保线程会执行到该方法。也可以设置超时时间await(long timeout, TimeUnit unit),超时就不再等待

```

## 3.2 CyclicBarrier

CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

在不同线程之间设置屏障

![img](assets\16015500-670d7281d1fd3667.png)

先到达的线程进入等待，等待屏障打开

![img](assets\16015500-bb540a26440bd7d2.png)

当所有线程到达屏障后，屏障打开

**初始化**

第一种 

CyclicBarrier c = new CyclicBarrier(int parties);

parties:表示拦截线程的数量

每个线程调用CyclicBarrier 的await()，表示当前线程到达屏障，当前线程会进入阻塞。当所有规定数量的线程到达屏障后，之前阻塞的线程线程都会唤醒。继续执行。（屏障打开后，所有线程的执行顺序是不确定的，根据线程竞争cpu，获取执行时间）

第二种

CyclicBarrier c = new CyclicBarrier(int parties, Runnable barrierAction)

barrierAction：屏障线程

设置屏障线程，既然所有线程都在同一个点阻塞（同步点），那么这个阻塞点就很有意义，所以就有了屏障线程的出现，在所有线程到达屏障，但是屏障还没有打开，这个时间就是屏障线程的工作时间。

例如有一个5000大小的数组，统计数组中数字的和。可以将这个数组分成5分，每个线程处理1000个数字之和，当5个线程执行完毕，再有一个屏障线程处理这五个线程的和，这样得到最终的结果。

对上面的例子进行编码

```java
public class CyclicBarrierTest {
    static int[] numArr = new int[5000];
    static int[] resultArr = new int[5];

    public static void main(String[] args) {
        Random random = new Random();
        //随机填充5000个数
        int i;
        for (i=0;i<5000;i++){
            numArr[i] = random.nextInt(5000);
        }
        //创建一个线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        //创建CyclicBarrier
        final CyclicBarrier c = new CyclicBarrier(5,new BarrierThread());

        //创建5个线程,进行计算,并提交给线程池进行管理
        int j;
        for (j=0;j<5;j++){
            final int[] arr = Arrays.copyOfRange(numArr,j*1000,(j+1)*1000);
            final int index = j;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    //计算
                    for(int num : arr){
                        resultArr[index] += num;
                    }
                    //计算完毕,设置屏障
                    try {
                        c.await();
                        //当屏障线程计算完毕,打开屏障,输出各个线程计算之和
                        System.out.println("线程"+index+"的和:"+resultArr[index]);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        //关闭线程池
        threadPool.shutdown();
    }
    public static class BarrierThread implements Runnable{
        @Override
        public void run() {
            int result = 0;
            for(int i : resultArr){
                result += i;
            }
            System.out.println("总和为:"+result);
        }
    }
}

```

结果:

```
总和为:12433020
线程4的和:2522329
线程0的和:2430680
线程2的和:2493498
线程3的和:2517489
线程1的和:2469024

```

## 3.3 SemaPhore

Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。

比如**虎门大桥**异常震动要限制流量，只允许同时有一百辆车在桥上行使，其他的都必须在桥口等待，所以前一百辆车会看到绿灯，可以开进虎门大桥，后面的车会看到红灯，不能驶入虎门大桥，但是如果前一百辆中有5辆车已经离开了虎门大桥，那么后面就允许有5辆车驶入大桥，这个例子里说的车就是线程，驶入大桥就表示线程在执行，离开大桥就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。

例子模拟100个线程 ，SemaPhore限制10个线程。

```java
public class SemaPhoreTest {
    public static void main(String[] args) {
        final Semaphore s = new Semaphore(10);
        ExecutorService threadPool = Executors.newFixedThreadPool(100);
        int i;
        for (i=0;i<100;i++){
            final int num = i;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    try {
                        //获取一个认证,认证现在占有公共资源的线程还没有达到上限
                        s.acquire();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    try {
                        //睡眠2秒,模拟任务执行
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("Thread :"+num);
                    //释放认证
                    s.release();
                }
            });
        }
        threadPool.shutdown();
    }
}

```

运行结果:每隔2秒打印10个线程

```
Thread :1
Thread :2
Thread :0
Thread :3
Thread :4
Thread :6
Thread :5
Thread :7
Thread :8
Thread :9
.....

```

## 3.4 Exchanger

Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange方法交换数据，如果第一个线程先执行exchange()方法，它会一直等待第二个线程也执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

例子：模拟A，B两个线程之间信息的交换

```java
public class ExchangerTest {
    public static void main(String[] args) {
        final Exchanger<Integer> e = new Exchanger<Integer>();
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        threadPool.submit(new Runnable() {
            @Override
            public void run() {
                int result = 0;
                for (int i=0;i<100;i++){
                    result += i;
                }
                try {
                    int intB = e.exchange(result);
                    System.out.println("线程A中获取线程B的数据:"+intB);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
            }
        });

        threadPool.submit(new Runnable() {
            @Override
            public void run() {
                int result = 0;
                for (int i=0;i<10;i++){
                    result += i;
                }
                try {
                    int intA = e.exchange(result);
                    System.out.println("线程B中获取线程A的数据:"+intA);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }

            }
        });
        threadPool.shutdown();
    }
}

```

运行结果:

```
线程A中获取线程B的数据:45
线程B中获取线程A的数据:4950

```

如果两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用exchange（V x，long timeout，TimeUnit unit）设置最大等待时长。

# 第四章 Java原子操作类

当程序更新一个变量时，如果多线程同时更新这个变量，可能得到期望之外的值，比如变量i=1，A线程更新i+1，B线程也更新i+1，经过两个线程操作之后可能i不等于3，而是等于2。因为A和B线程在更新变量i的时候拿到的i都是1，这就是线程不安全的更新操作，通常我们会使用synchronized来解决这个问题，synchronized会保证多线程不会同时更新变量i。

而Java从JDK 1.5开始提供了java.util.concurrent.atomic包（以下简称Atomic包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。

因为变量的类型有很多种，所以在Atomic包里一共提供了12个类，属于4种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段）

## 4.1 原子更新基本类型

**AtomicInteger**：原子更新整形

**AtomicBoolean**：原子更新布尔形

**AtomicLong**：原子更新长整形

上面三个原子操作类的使用方法大多都相同，下面使用AtomicInteger来介绍常用的API

初始化 AtomicInteger atomic = new AtomicInteger(100);

get()：获取当前值

addAndGet(int delta) :以原子方式将给定值添加到当前值，并返回添加后的值

decrementAndGet()：以原子的方式将当前值减一，返回减后的值

incrementAndGet()：以原子的方式将当前值加一，返回加后的值

getAndDecrement()：以原子的方式将当前值减一，返回减前的值

getAndIncrement()：以原子的方式将当前值加一，返回加前的值

代码示例:

```java
public class Test1 {
    public AtomicInteger ai = new AtomicInteger(0);
    public static void main(String[] args) throws InterruptedException {
        Test1 t = new Test1();
        Thread t1 = new Thread(new A(t));
        Thread t2 = new Thread(new B(t));
        t2.start();
        t1.start();
    }
}

class A implements Runnable{
    private Test1 t;

    public A(Test1 t) {
        this.t = t;
    }

    @Override
    public void run() {
        for (int i=0;i<10;i++){
            System.out.println("A:"+t.ai.incrementAndGet());
        }

    }
}

class B implements Runnable{
    private Test1 t;

    public B(Test1 t) {
        this.t = t;
    }
    @Override
    public void run() {
        for (int i=0;i<10;i++) {
            System.out.println("B==" + t.ai.incrementAndGet());
        }

    }
}

```

## 4.2 原子更新数组

过原子的方式更新数组里的某个元素，Atomic包提供了以下3个类。

**AtomicIntegerArray**：原子更新整型数组里的元素。

**AtomicLongArray**：原子更新长整型数组里的元素。

**AtomicReferenceArray**：原子更新引用类型数组里的元素

AtomicIntegerArray类主要是提供原子的方式更新数组里的整型，其常用方法如下。

addAndGet（int i，int delta）：以原子方式将输入值与数组中索引i的元素相加。

decrementAndGet(int i)：以原子的方式将索引为i的值减一，返回减后的值

incrementAndGet(int i)：以原子的方式将索引为i的值加一，返回加后的值

getAndDecrement(int i)：以原子的方式将索引为i的值减一，返回减前的值

getAndIncrement(int i)：以原子的方式将索引为i的值加一，返回加前的值

例子:

![img](assets\16015500-7b5f442a2bdd5500.png)

结果:

![img](assets\16015500-e9eb96ecd97fbcd3.png)

```
需要注意的是，数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组

```

## 4.3 原子更新引用类型

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类。

**AtomicReference**：原子更新引用类型。

**AtomicReferenceFieldUpdater**：原子更新引用类型里的字段。

**AtomicMarkableReference**：原子更新带有标记位的引用类型。

上面三个类的使用方法都类似，使用AtomicReference的常用方法

set(V newValue)：设置一个值

getAndSet(V newValue)：原子地设置为给定值并返回旧值

compareAndSet(V expect, V update)：如果当前值是期望值，则原子化地将该值设置为给定的更新值

例子:

![img](assets\16015500-e97dcc2d8c938283.png)

结果:

![img](assets\16015500-0deb4cb0997cd616.png)

## 4.4 原子更新字段类

如果需原子地更新某个类里的某个字段时，就需要使用原子更新字段类，Atomic包提供了以下3个类进行原子字段更新

**AtomicIntegerFieldUpdater**：原子更新整型的字段的更新器。

**AtomicLongFieldUpdater**：原子更新长整型字段的更新器。

**AtomicStampedReference**：原子更新带有版本号的引用类型

要想原子地更新字段类需要两步。第一步，因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。第二步，更新类的字段（属性）必须使用public volatile修饰符。

例子:

![img](assets\16015500-9ca7a7b18b27193b.png)

结果:

![img](assets\16015500-80649cd7e953ca36.png)



