## Day1

#### IO流程

作用：传输数据 --> Java代码与现实世界连接的桥梁

方向：主角 --> java

##### 分类：

- 字节流：操作任何类型的文件
- 接口:InputStream / OutputStream
  - FileInputStream / FileOutputStream
  - BufferedInputStream / BufferedOutputStream
    - 一次读一个字符
    - 一次一个字符数组
- 字符流：操作文本
- 接口：InputReader / OutputWriter
  - InputStreamReader / OutputStreamWriter:转换流
    - 构造传入 字节流
  - FileReader / FileWriter
  - BufferedReader / BufferedWriter
    - 一次读一个字符
    - 一次一个字符数组
    - 高效的 一次一行（不读换行）

> *字节流中还有以下：*
>
> ​	System.in：标准的字节输入流
>
> ​		只做键盘录入
>
> ​	System.out：标准的字节输出流
>
> ​		控制台打印

> *追加写？*
>
> ​	在FileOutputStream / FileWriter构造的第二个参数传入true

> *设置编码格式：*
>
> ​	装换流：构造的第二个实参传入编码格式。
>
> ​		“ASCII”：
>
> ​		“UTF-8”:可变长编码集
>
> ​		“GBK”：中文编码
>
> ​		“ANSI”:本地编码集（随系统改变）

*以下是一个复制追加的操作*

1. ```java
   public static void main(String[] args) throws IOException {
   	BufferedWriter bWriter = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("1.txt",true), "UTF-8"));
   	BufferedReader bReader = new BufferedReader(new FileReader("in.txt"));
   		
   	String line;
   	while ((line = bReader.readLine()) != null) {
   		bWriter.write(line);
   		bWriter.newLine();
   		bWriter.flush();
   	}
   		
   	bWriter.close();
   	bReader.close();
   }
   ```



#### 多线程

*进程与线程的关系*

- 进程：正在运行的程序，计算机资源分配的最小单元

- 线程：线程是存在于进程中的顺序执行流

  ​	进程允许有多个线程



在java中有3中实现方式

1. 继承Thread
2. 实现Runnable接口
3. 实现Callable接口(带有返回值可以抛异常)

##### 1.继承Thread

继承Thread类可以直接使用Thread类的方法，将需要实现的功能封装起来在重写的run()方法中执行即可

```java
public class MyTread extends Thread{
	@Override
	public void run() {
		for (int i = 1; i < 101; i++) {
			System.out.println(this.getName() + " " + i);
		}
	}
}
```

##### 2.实现Runnable接口

实现runnable接口的类无法调用Thread的方法，但是可以通过Thread构造器将其传入构造Thr对象加以使用

```java
public class MyTreadRunnable implements Runnable{
	@Override
	public void run() {
		for (int i = 1; i < 101; i++) {
			System.out.println(Thread.currentThread().getName() + " " + i);
		}
	}
}
```

实际上可以使用匿名内部类来实现

```java
new Thread(new Runnable() {
			public void run() {
				for (int i = 1; i < 101; i++) {
					System.out.println(Thread.currentThread().getName() + " " + i);
				}
			}
		},"线程3").start();
```

##### 3.实现Callable接口

```java
package com.mulTread;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;


/*实现callable接口
 * 1.实现Callable接口的call方法
 * 2.使用FutureTask接收
 * 3.new Thread(futureTask, "有返回").start();
 * */
public class ThirdThread {
	public static void main(String[] args) {
		ThirdThread rt = new ThirdThread();
		//使用lambda表达式实现callable接口
		FutureTask<Integer> task = new FutureTask<>(()->{
				int i;
				for (i = 0; i < 10; i++) {
					System.out.println(i);
				}
				return i;
			}
		);
		
		for (int i = 0; i < 100; i++) {
			if (i == 20) {
				new Thread(task, "有返回").start();
				try {
					int re = task.get();
					System.out.println("-----"+re);
				} catch (InterruptedException e) {
					e.printStackTrace();
				} catch (ExecutionException e) {
					e.printStackTrace();
				}
			}
		}
	}
}
```



#### 加锁

加锁包括使用sychronized关键字对方法或者代码块加锁，或者使用Lock同步锁来加锁

Thread的wait()会释放锁

Thread的yield() sleep()不会释放锁



Lock对象需要显示的加锁和释放

Lock.lock和Lock.unlock

#### 线程池

##### Executors工厂类来产生线程池

> ```
> public class Executors extends ObjectFactory and utility methods for Executor, ExecutorService, ScheduledExecutorService, ThreadFactory, and Callable classes defined in this package. 
> ```

newCachedThreadPool()	自动扩展

newFixedThreadPool()	有限数量

newSingleThreadPool()	一个

newScheduledThreadPool()	固定数量，可以延迟或者定时的去执行任务

....

通过submit提交执行

```java
java.util.concurrent.Executor:负责线程的使用和调度的 根接口
	|--ExecutorService 子接口：线程池主要 接口
		|--ThreadPoolExecutor 线程池的 实现类
		|--ScheduledExecutorService 子接口：负责线程的调度
			|--ScheduledThreadPoolExecutor 
				继承ThreadPoolExecutor实现ScheduledExecutorService
```



##### ForkJoin框架

![1536066658372](..\图片\1536066658372.png)

```java
package ForkJionDemo;

import java.util.concurrent.ForkJoinPool;

import org.junit.jupiter.api.Test;

public class ForkJionDemo implements ForkJionInterface {
	@Test
	public void test1() {
		ForkJoinPool fPool = new ForkJoinPool();

		ForkJionTaskDemo forkJionTask = new ForkJionTaskDemo(0l, 5000000000l);

		Long result = fPool.invoke(forkJionTask);

		System.out.println(result);
	}

	@Test
	public void test2() {
		long sum = 0;

		for (int i = 0; i <= 5000000000l; i++) {
			sum += i;
		}

		System.out.println(sum);
	}
}
```

```java
package ForkJionDemo;

import java.util.concurrent.RecursiveTask;
import java.util.function.BiFunction;

public class ForkJionTaskDemo extends RecursiveTask<Long> {
	/**
	 * 
	 */
	private static final long serialVersionUID = 9085118290088942449L;
	private static final long THRESHSHOLD = 10L;
	private long start;
	private long end;

	public ForkJionTaskDemo(long start, long end) {
		super();
		this.start = start;
		this.end = end;
	}

	@Override
	protected Long compute() {
		if (end - start < 1000) {

			return getResult(start, end, (start1, end1) -> {
				long sum = 0;
				for (long i = start1; i <= end1; i++) {
					sum += i;
				}
				return sum;
			});

			// return getResult(start, end, (start,end)->(start+end)*(end-start+1)/2);
		} else {
			long mid = (start + end) / 2;
			ForkJionTaskDemo left = new ForkJionTaskDemo(start, mid);
			left.fork();
			ForkJionTaskDemo right = new ForkJionTaskDemo(mid + 1, end);
			right.fork();

			return left.join() + right.join();
		}
	}

	private Long getResult(long start, long end, BiFunction<Long, Long, Long> bf) {
		// TODO Auto-generated method stub
		return bf.apply(start, end);
	}

}
```



把线程池和ForkJion加一起玩一下

动态代理很简单，自己写一下就是了

```java
package ForkJionDemo;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MainTest {
	public static void main(String[] args) {
		
		ForkJionDemo forkJionDemo = new ForkJionDemo();
		ForkJionInterface fjdProxy = MyProxyFactory.get(forkJionDemo);
		
		ExecutorService poll = Executors.newFixedThreadPool(2);	
		poll.submit(new Runnable() {
			@Override
			public void run() {
				fjdProxy.test1();
			}
		});
		
		poll.submit(new Runnable() {
			@Override
			public void run() {
				fjdProxy.test2();
			}
		});
		
		poll.shutdown();
	}
}
```

