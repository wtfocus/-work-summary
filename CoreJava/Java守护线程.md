[toc]

### Java 守护线程

### 作用

1.  Java 提供的线程类型：
    -   **用户线程**：JVM 终止前会等待任何用户线程结束。
    -   **守护线程**：唯一的作用是**为用户线程提供服务**。
2.  守护线程不会阻止 JVM 退出。
3.  **不能在守护线程中执行 I/O 任务**，JVM 退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。

### 实现

1.  代码

    -   ```java
        public class DaemonTest {
            public static void main(String[] args) {
                Thread thread = new TimerThread();
                // 必须在启动线程前调用
                thread.setDaemon(true);
                thread.start();
        
                System.out.println("isDaemon = " + thread.isDaemon());
        
                try {
                    // 程序阻塞，一旦接收到用户输入，main线程结束，守护线程自动结束
                    System.in.read();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
                System.out.println("main thread ending.");
            }
        }
        
        
        class TimerThread extends Thread {
            public void run() {
                int n = 0;
                while (true) {
        
                    System.out.println(Thread.currentThread().getName() + " -- " + ++n);
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        ```

        