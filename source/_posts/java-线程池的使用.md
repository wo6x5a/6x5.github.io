---
title: java 线程池的使用
date: 2017-03-27 17:25:25
tags: [java,线程池]
categories: 技术
---
```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 线程池.
 * 
 * @author chenwulou
 *
 */
public class TestThreadPool {
	/**
	 * 下面解释下一下构造器中各个参数的含义：
	 * 
	 * corePoolSize：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，</br>
	 * 线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()
	 * 或者prestartCoreThread()方法，</br>
	 * 从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。</br>
	 * 默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，
	 * 当线程池中的线程数目达到corePoolSize后，</br>
	 * 就会把到达的任务放到缓存队列当中；</br>
	 * maximumPoolSize：线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；</br>
	 * keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。</br>
	 * 默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，</br>
	 * 直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，</br>
	 * 如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。</br>
	 * 但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，</br>
	 * keepAliveTime参数也会起作用，直到线程池中的线程数为0；</br>
	 * unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：</br>
	 * TimeUnit.DAYS; //天</br>
	 * TimeUnit.HOURS; //小时</br>
	 * TimeUnit.MINUTES; //分钟</br>
	 * TimeUnit.SECONDS; //秒</br>
	 * TimeUnit.MILLISECONDS; //毫秒</br>
	 * TimeUnit.MICROSECONDS; //微妙</br>
	 * TimeUnit.NANOSECONDS; //纳秒</br>
	 * 
	 * @param args
	 */
	public static void main(String[] args) {
		//自动创建线程池
//		ExecutorService executorService = Executors.newCachedThreadPool();
//		ExecutorService executorService = Executors.newSingleThreadExecutor();
//		ExecutorService executorService = Executors.newFixedThreadPool(3);
		ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 15, 200, TimeUnit.MILLISECONDS,
				new ArrayBlockingQueue<Runnable>(5));

		for (int i = 0; i < 15; i++) {
			MyTask myTask = new MyTask(i);
//			Thread t = new Thread(myTask);
			//设置优先级
//			t.setPriority(1);
			executor.execute(myTask);
			System.out.println("线程池中线程数目：" + executor.getPoolSize() + "，队列中等待执行的任务数目："
					+ executor.getQueue().size() + "，已执行完毕的任务数目：" + executor.getCompletedTaskCount());
		}
		executor.shutdown();
//		executor.shutdownNow();
	}
}

class MyTask implements Runnable {
	private int taskNum;

	public MyTask(int num) {
		this.taskNum = num;
	}

	@Override
	public void run() {
		System.out.println("正在执行task " + taskNum);
		try {
//			Thread.sleep(10);
			Thread.currentThread().sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("task " + taskNum + "执行完毕");
	}
}
```