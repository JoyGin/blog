Java中实现线程间通信的方式主要有以下几种：

1. **等待/通知机制（wait/notify）**：
   - `wait()`：使当前线程等待，直到另一个线程调用同一个对象的`notify()`或`notifyAll()`方法。
   - `notify()`：唤醒在此对象监视器上等待的单个线程。
   - `notifyAll()`：唤醒在此对象监视器上等待的所有线程。

   使用时，这些方法必须在同步代码块或同步方法中调用，因为它们需要对象锁。

   ```java
   class SharedObject {
       public synchronized void waitForSignal() throws InterruptedException {
           while (/* condition not met */) {
               wait();
           }
           // Perform action appropriate to condition
       }

       public synchronized void doNotify() {
           // Change condition
           notify();
           // or notifyAll();
       }
   }
   ```

2. **管道通信（PipedInputStream/PipedOutputStream）**：
   - 主要用于在不同线程间传输数据，一个线程发送数据到输出管道，另一个线程从输入管道读取数据。

   ```java
   PipedOutputStream output = new PipedOutputStream();
   PipedInputStream input = new PipedInputStream(output);

   new Thread(new Runnable() {
       public void run() {
           try {
               output.write("Hello, world!".getBytes());
               output.close();
           } catch (IOException e) {}
       }
   }).start();

   new Thread(new Runnable() {
       public void run() {
           try {
               int data = input.read();
               while (data != -1) {
                   System.out.print((char) data);
                   data = input.read();
               }
               input.close();
           } catch (IOException e) {}
       }
   }).start();
   ```

3. **信号量（Semaphore）**：
   - 信号量可以控制对共享资源的访问，通过acquire()和release()方法来获取和释放许可。

   ```java
   Semaphore semaphore = new Semaphore(1);

   new Thread(new Runnable() {
       public void run() {
           try {
               semaphore.acquire();
               // Critical section
           } catch (InterruptedException e) {
               // Handle exception
           } finally {
               semaphore.release();
           }
       }
   }).start();
   ```

4. **CountDownLatch/CyclicBarrier**：
   - `CountDownLatch`：允许一个或多个线程等待一系列指定操作的完成。
   - `CyclicBarrier`：允许一组线程互相等待，达到一个公共屏障点后再继续执行。

   ```java
   CountDownLatch latch = new CountDownLatch(1);

   new Thread(new Runnable() {
       public void run() {
           // Perform task
           latch.countDown();
       }
   }).start();

   new Thread(new Runnable() {
       public void run() {
           try {
               latch.await();
               // Continue with execution
           } catch (InterruptedException e) {
               // Handle exception
           }
       }
   }).start();
   ```

5. **BlockingQueue**：
   - 阻塞队列提供了线程安全的队列操作，线程可以安全地从队列中插入和移除元素，如果队列为空或满时，线程会阻塞。

   ```java
   BlockingQueue<Integer> queue = new LinkedBlockingQueue<>();

   new Thread(new Runnable() {
       public void run() {
           try {
               queue.put(1);
           } catch (InterruptedException e) {
               // Handle exception
           }
       }
   }).start();

   new Thread(new Runnable() {
       public void run() {
           try {
               int value = queue.take();
               // Process value
           } catch (InterruptedException e) {
               // Handle exception
           }
       }
   }).start();
   ```

6. **FutureTask/Callable**：
   - `Callable`接口代表一个可以返回结果的任务，而`FutureTask`可以包装`Callable`，并且可以用来查询任务是否完成，等待任务完成，以及获取任务的结果。

   ```java
   Callable<Integer> callable = new Callable<Integer>() {
       public Integer call() throws Exception {
           // Perform computation and return result
           return 42;
       }
   };

   FutureTask<Integer> futureTask = new FutureTask<>(callable);
   new Thread(futureTask).start();

   try {
       Integer result = futureTask.get(); // This will block until the result is available
   } catch (InterruptedException | ExecutionException e) {
       // Handle exception
   }
   ```

这些是Java中实现线程间通信的一些常用方式。在实际应用中，应根据具体需求选择最合适的通信机制。