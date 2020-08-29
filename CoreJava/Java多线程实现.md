[toc]

## Java 多线程实现

### 继承 Thread 类

1.  代码

    -   ```java
        public class ThreadTest {
            public static void main(String[] args) throws ExecutionException, InterruptedException {
                
                // 继承 Thread 类
                MyThread myThreadA = new MyThread();
                MyThread myThreadB = new MyThread();
                MyThread myThreadC = new MyThread();
                myThreadA.start();
                myThreadB.start();
                myThreadC.start();
            }
        }
        
        
        class MyThread extends Thread {
            public void run() {
                System.out.println(this.getClass().getName() + "--" + Thread.currentThread().getName());
            }
        }
        ```

### 实现 Runnable 接口

1.  代码

    -   ```java
        public class ThreadTest {
            public static void main(String[] args) throws ExecutionException, InterruptedException {
        
                // 实现 Runnable 接口
                MyRunnable myRunnable = new MyRunnable();
                new Thread(myRunnable).start();
                new Thread(myRunnable).start();
                new Thread(myRunnable).start();
            }
        }
        
        
        class MyRunnable implements Runnable {
        
            @Override
            public void run() {
                System.out.println(this.getClass().getName() + "--" + Thread.currentThread().getName());
            }
        }
        ```

    -   

### 实现 Callable 接口

1.  代码

    -   ```java
        public class ThreadTest {
            public static void main(String[] args) throws ExecutionException, InterruptedException {
        
                // 实现 Callable 接口
                ExecutorService executorService = Executors.newSingleThreadExecutor();
                // 在一个新线程上执行 MyCallable 的 call 方法
                Future<Integer> future = executorService.submit(new MyCallable());
                // 阻塞，直到得到返回值
                System.out.println(future.get());
                executorService.shutdown();
            }
        }
        
        
        class MyCallable implements Callable<Integer> {
        
            @Override
            public Integer call() throws Exception {
                System.out.println(this.getClass().getName() + "--" + Thread.currentThread().getName());
        
                Thread.sleep(2000);
        
                return 123;
            }
        }
        ```

        