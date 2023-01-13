# JUC-其它组件

### FutureTask

在介绍Callable时我们知道它可以有返回值，返回值通过Future<V>进行封装。FutureTask实现了RunnableFuture接口，该接口继承自Runnable和Future<V>接口，这使得FutureTask既可以当做一个任务执行，也可以有返回值。

```java
public class FutureTask<V> implements RunnableFuture<V>
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用FutureTask来封装这个任务，主线程在完成自己的人之后在去获取结果。

```java

public class FutureTaskExample {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(100);
                    result += i;
                }
                LocalDateTime now = LocalDateTime.now();
                System.out.println(now.getHour()+":"+now.getMinute()+":"+now.getSecond());
                return result;
            }
        });

        Thread computeThread = new Thread(futureTask);
        computeThread.start();

        Thread otherThread = new Thread(() -> {
            LocalDateTime now = LocalDateTime.now();
            System.out.println(now.getHour()+":"+now.getMinute()+":"+now.getSecond());
            System.out.println("other task is running...");
            // try {
            //     // Thread.sleep(1000);
            // } catch (InterruptedException e) {
            //     e.printStackTrace();
            // }
        });
        otherThread.start();
        System.out.println(futureTask.get());
    }
}
```

```java
9:7:20
other task is running...
9:7:31
4950
```

FutureTask异步的执行任务


### BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

- FIFO队列：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
- 优先级队列：PriorityBlockingQueue

提供了阻塞的take()和put()方法：如果队列为空take()阻塞，直到队列中有内容；如果队列为满put()将阻塞，直到队列有空闲位置。

使用 BlockingQueue 实现生产者消费者问题

```java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }
    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }
        for (int i = 0; i < 5; i++) {
            //到i=2的时候阻塞了，这时候要等队列中有值才会执行 当生产者不执行了，消费者还在执行，阻塞了，运行不停止
            Consumer consumer = new Consumer();
            consumer.start();
        }
        for (int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }
    }
    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}

produce..produce..consume..consume..produce..produce..consume..consume..
```

### ForkJoin

主要用于并行计算中，和MapReduce原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```java
public class ForkJoinExample extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinExample example = new ForkJoinExample(1, 20);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Future result = forkJoinPool.submit(example);
        System.out.println(result.get());
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分成小任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }
}
```

ForkJoin使用ForkJoinPool来启动，它是一个特殊的线程池，线程数量取决于CPU核数。
```java
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool 实现了工作窃取算法来提高 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。例如下图中，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。但是如果队列中只有一个任务时还是会发生竞争。

![pSKeFSg.png](https://s1.ax1x.com/2023/01/13/pSKeFSg.png)

