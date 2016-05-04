%%:uuid=160503232905006

## 1 JAVA线程: 创建与启动
###1.1 继承java.lang.Thread类
```
java.lang.Object
  |-- java.lang.Thread
```
创建线程：
``` java
 class PrimeThread extends Thread {
         long minPrime;
         PrimeThread(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
              . . .
         }
     }

```
启用线程
``` java
PrimeThread p = new PrimeThread(143);
     p.start();
```
###1.2 实现java.lang.Runnable接口。
*方法摘要 *
```
 void run() 
          使用实现接口 Runnable 的对象创建一个线程时，启动该线程将导致在独立执行的线程中调用对象的 run 方法。 
```
创建线程：
```java
class PrimeRun implements Runnable {
         long minPrime;
         PrimeRun(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
              . . .
         }
     }
```
使用线程
``` java
PrimeRun p = new PrimeRun(143);
     new Thread(p).start();
```

## 2. 线程状态
java.lang 
枚举 Thread.State
java.lang.Object
  |-- java.lang.Enum<Thread.State>

### 2.1 枚举常量摘要
BLOCKED  
          受阻塞并且正在等待监视器锁的某一线程的线程状态。 
NEW 
          至今尚未启动的线程的状态。 
RUNNABLE 
          可运行线程的线程状态。 
TERMINATED 
          已终止线程的线程状态。 
TIMED_WAITING 
          具有指定等待时间的某一等待线程的线程状态。 
WAITING 
          某一等待线程的线程状态。 
### 2.2 状态转换
![线程状态如](res:\attach\160503232905006\线程状态如.jpg)

## 3. 终止线程
###3.1 Thread.stop()***（废弃方法）***
Thread.stop()方法在结束线程时，会直接终止线程，并且会立即释放这个线程所有的锁。
>**结果：**导致数据不一致

```java
package com.cc.training.thread;

/**
 * 用于测试使用Thread.stop()方法终止线程导致的数据不一致问题
 * 
 * @author cc
 *
 */
public class StopThreadUnsafeTest {
	/**
	 * 定义共享的user用于全局调用，下面两个测试线程都可以使用当前user
	 */
	private static User user = new User();

	/**
	 * 声明一个user，包括uername和id 两个线程同时操作更新这个user的数据
	 */
	public static class User {
		private int id;
		private String username;

		public User() {
			setId(0);
			setUsername("0");
		}

		@Override
		public String toString() {
			// TODO Auto-generated method stub
			return "User {id=" + getId() + ", username=" + getUsername() + "}";
		}

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}

		public String getUsername() {
			return username;
		}

		public void setUsername(String username) {
			this.username = username;
		}
	}

	/**
	 * 使用extends Thread的模式创建一个线程 该线程用于设置user的属性
	 * 
	 * @author cc
	 *
	 */
	public static class ChangeObjectThread extends Thread {

		@Override
		public void run() {
			// TODO Auto-generated method stub
			
			while (true) {
				synchronized (user) {
					//在当前进程中使用同一变量，分别赋值给id和user 以用于检查数据不一致性
					int v = (int) (System.currentTimeMillis() / 1000);
					user.setId(v);
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					user.setUsername(String.valueOf(v));
				}
				// 暂停当前正在执行的线程对象，并执行其他线程。
				Thread.yield();
			}
		}
	}
	
	/**
	 * 使用extends Thread的模式创建另一个线程 该线程用于读取user的属性
	 * @author cc
	 */
	public static class ReadObjectThread extends Thread {
		/**
		 * 当user下的id和name不一致的时候，读出user
		 */
		@Override
		public void run() {
			System.out.println("Read Thread Start!");
			// TODO Auto-generated method stub
			while (true) {
				synchronized (user) {
					if(user.getId() != Integer.parseInt(user.getUsername())){
						System.out.println(user.toString());
					}
			}
			// 暂停当前正在执行的线程对象，并执行其他线程。
			Thread.yield();
		}
	  }
	}

	public static void main(String[] args) throws InterruptedException {
		new ReadObjectThread().start();
		//每隔1.5秒，启动一个changeOjectThread的线程
		while(true){
			Thread t  = new ChangeObjectThread();
			t.start();
			Thread.sleep(1500);
			t.stop();
		}
	}
}

```

>**执行结果**
![image](res:\attach\160503232905006\image.jpg)

###3.2 正确终止线程
> 使用标志位，让线程自行结束

```java
package com.cc.training.thread;


/**
 * 使用标志位来正确终止线程
 * 
 * @author cc
 *
 */
public class StopTheadSafeTest {
	/**
	 * 定义共享的user用于全局调用，下面两个测试线程都可以使用当前user
	 */
	private static User user = new User();

	/**
	 * 声明一个user，包括uername和id 两个线程同时操作更新这个user的数据
	 */
	public static class User {
		private int id;
		private String username;

		public User() {
			setId(0);
			setUsername("0");
		}

		@Override
		public String toString() {
			// TODO Auto-generated method stub
			return "User {id=" + getId() + ", username=" + getUsername() + "}";
		}

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}

		public String getUsername() {
			return username;
		}

		public void setUsername(String username) {
			this.username = username;
		}
	}

	/**
	 * 使用extends Thread的模式创建一个线程 该线程用于设置user的属性
	 * 
	 * @author cc
	 *
	 */
	public static class ChangeObjectThread extends Thread {
		// 线程终止标志位
		volatile static boolean stopFlag = false;
		/**
		 * 提供给外部调用，用于终止线程
		 */
		public static void stopThread(){
			stopFlag = true;
		}
		@Override
		public void run() {
			// TODO Auto-generated method stub
			
			while (!stopFlag) {
				synchronized (user) {
					//在当前进程中使用同一变量，分别赋值给id和user 以用于检查数据不一致性
					int v = (int) (System.currentTimeMillis() / 1000);
					user.setId(v);
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					user.setUsername(String.valueOf(v));
				}
				// 暂停当前正在执行的线程对象，并执行其他线程。
				Thread.yield();
			}
		}
	}
	
	/**
	 * 使用extends Thread的模式创建另一个线程 该线程用于读取user的属性
	 * @author cc
	 */
	public static class ReadObjectThread extends Thread {
		/**
		 * 当user下的id和name不一致的时候，读出user
		 */
		@Override
		public void run() {
			System.out.println("Read Thread Start!");
			// TODO Auto-generated method stub
			while (true) {
				synchronized (user) {
					if(user.getId() != Integer.parseInt(user.getUsername())){
						System.out.println(user.toString());
					}
			}
			// 暂停当前正在执行的线程对象，并执行其他线程。
			Thread.yield();
		}
	  }
	}

	public static void main(String[] args) throws InterruptedException {
		new ReadObjectThread().start();
		//每隔1.5秒，启动一个changeOjectThread的线程
		while(true){
			Thread t  = new ChangeObjectThread();
			t.start();
			Thread.sleep(1500);
			((ChangeObjectThread) t).stopThread();
		}
	}
}

```