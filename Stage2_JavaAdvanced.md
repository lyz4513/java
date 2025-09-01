# 第二阶段：Java进阶技术 (6-12个月)

## 🎯 学习目标
- 掌握多线程并发编程和线程安全
- 熟练使用IO流和NIO进行文件操作
- 理解反射机制和动态代理
- 掌握注解的使用和自定义注解
- 深入理解JVM原理和性能调优
- 熟练应用常用设计模式

## 📚 知识体系

### 1. 多线程并发编程

#### 1.1 线程基础
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 多线程基础演示
 * 包含线程创建、生命周期、同步机制等
 */
public class ThreadBasicsDemo {
    
    public static void main(String[] args) {
        demonstrateThreadCreation();
        demonstrateThreadLifecycle();
        demonstrateThreadSynchronization();
        demonstrateDeadlock();
    }
    
    /**
     * 演示线程创建的三种方式
     */
    private static void demonstrateThreadCreation() {
        System.out.println("=== 线程创建方式演示 ===");
        
        // 方式1：继承Thread类
        class MyThread extends Thread {
            private String name;
            
            public MyThread(String name) {
                this.name = name;
            }
            
            @Override
            public void run() {
                for (int i = 0; i < 5; i++) {
                    System.out.println(name + " 执行第 " + (i + 1) + " 次");
                    try {
                        Thread.sleep(1000); // 休眠1秒
                    } catch (InterruptedException e) {
                        System.out.println(name + " 被中断");
                        return;
                    }
                }
                System.out.println(name + " 执行完毕");
            }
        }
        
        // 方式2：实现Runnable接口
        class MyRunnable implements Runnable {
            private String name;
            
            public MyRunnable(String name) {
                this.name = name;
            }
            
            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    System.out.println(name + " Runnable执行第 " + (i + 1) + " 次");
                    try {
                        Thread.sleep(800);
                    } catch (InterruptedException e) {
                        System.out.println(name + " Runnable被中断");
                        return;
                    }
                }
                System.out.println(name + " Runnable执行完毕");
            }
        }
        
        // 方式3：实现Callable接口（有返回值）
        class MyCallable implements Callable<String> {
            private String name;
            
            public MyCallable(String name) {
                this.name = name;
            }
            
            @Override
            public String call() throws Exception {
                int sum = 0;
                for (int i = 1; i <= 5; i++) {
                    sum += i;
                    System.out.println(name + " Callable计算: " + i);
                    Thread.sleep(500);
                }
                return name + " 计算结果: " + sum;
            }
        }
        
        // 启动线程
        MyThread thread1 = new MyThread("Thread-1");
        thread1.start();
        
        Thread thread2 = new Thread(new MyRunnable("Thread-2"));
        thread2.start();
        
        // 使用线程池执行Callable
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<String> future = executor.submit(new MyCallable("Callable-1"));
        
        try {
            String result = future.get(); // 获取返回值
            System.out.println("Callable返回值: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        
        executor.shutdown();
        
        // Lambda表达式创建线程（Java 8+）
        Thread lambdaThread = new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                System.out.println("Lambda线程执行第 " + (i + 1) + " 次");
                try {
                    Thread.sleep(600);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
            }
            System.out.println("Lambda线程执行完毕");
        });
        lambdaThread.start();
    }
    
    /**
     * 演示线程生命周期
     */
    private static void demonstrateThreadLifecycle() {
        System.out.println("\n=== 线程生命周期演示 ===");
        
        Thread thread = new Thread(() -> {
            System.out.println("线程开始执行，状态: " + Thread.currentThread().getState());
            
            try {
                // 模拟工作
                Thread.sleep(2000);
                System.out.println("线程工作中，状态: " + Thread.currentThread().getState());
                
                // 等待另一个线程
                synchronized (ThreadBasicsDemo.class) {
                    System.out.println("线程等待中，状态: " + Thread.currentThread().getState());
                    ThreadBasicsDemo.class.wait(1000);
                }
                
            } catch (InterruptedException e) {
                System.out.println("线程被中断");
                Thread.currentThread().interrupt();
            }
            
            System.out.println("线程即将结束，状态: " + Thread.currentThread().getState());
        });
        
        System.out.println("线程创建后状态: " + thread.getState()); // NEW
        
        thread.start();
        System.out.println("线程启动后状态: " + thread.getState()); // RUNNABLE
        
        try {
            Thread.sleep(500);
            System.out.println("线程运行中状态: " + thread.getState()); // RUNNABLE或TIMED_WAITING
            
            Thread.sleep(2000);
            System.out.println("线程等待中状态: " + thread.getState()); // WAITING或TIMED_WAITING
            
            thread.join(); // 等待线程结束
            System.out.println("线程结束后状态: " + thread.getState()); // TERMINATED
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 演示线程同步机制
     */
    private static void demonstrateThreadSynchronization() {
        System.out.println("\n=== 线程同步演示 ===");
        
        // 共享资源类
        class Counter {
            private int count = 0;
            private final Object lock = new Object();
            private final ReentrantLock reentrantLock = new ReentrantLock();
            private final AtomicInteger atomicCount = new AtomicInteger(0);
            
            // 非同步方法（线程不安全）
            public void incrementUnsafe() {
                count++;
            }
            
            // synchronized方法（线程安全）
            public synchronized void incrementSafe() {
                count++;
            }
            
            // synchronized代码块（线程安全）
            public void incrementWithBlock() {
                synchronized (lock) {
                    count++;
                }
            }
            
            // 使用ReentrantLock（线程安全）
            public void incrementWithLock() {
                reentrantLock.lock();
                try {
                    count++;
                } finally {
                    reentrantLock.unlock();
                }
            }
            
            // 使用原子类（线程安全）
            public void incrementAtomic() {
                atomicCount.incrementAndGet();
            }
            
            public int getCount() {
                return count;
            }
            
            public int getAtomicCount() {
                return atomicCount.get();
            }
            
            public void reset() {
                count = 0;
                atomicCount.set(0);
            }
        }
        
        Counter counter = new Counter();
        int threadCount = 10;
        int incrementsPerThread = 1000;
        
        // 测试非同步方法
        testConcurrency("非同步方法", counter, threadCount, incrementsPerThread, 
                       () -> counter.incrementUnsafe());
        
        counter.reset();
        
        // 测试synchronized方法
        testConcurrency("synchronized方法", counter, threadCount, incrementsPerThread, 
                       () -> counter.incrementSafe());
        
        counter.reset();
        
        // 测试synchronized代码块
        testConcurrency("synchronized代码块", counter, threadCount, incrementsPerThread, 
                       () -> counter.incrementWithBlock());
        
        counter.reset();
        
        // 测试ReentrantLock
        testConcurrency("ReentrantLock", counter, threadCount, incrementsPerThread, 
                       () -> counter.incrementWithLock());
        
        counter.reset();
        
        // 测试原子类
        testConcurrency("原子类", counter, threadCount, incrementsPerThread, 
                       () -> counter.incrementAtomic(), true);
    }
    
    /**
     * 测试并发性能和正确性
     */
    private static void testConcurrency(String method, Object counter, int threadCount, 
                                      int incrementsPerThread, Runnable task) {
        testConcurrency(method, counter, threadCount, incrementsPerThread, task, false);
    }
    
    private static void testConcurrency(String method, Object counter, int threadCount, 
                                      int incrementsPerThread, Runnable task, boolean useAtomic) {
        System.out.println("\n测试 " + method + ":");
        
        CountDownLatch latch = new CountDownLatch(threadCount);
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                for (int j = 0; j < incrementsPerThread; j++) {
                    task.run();
                }
                latch.countDown();
            }).start();
        }
        
        try {
            latch.await(); // 等待所有线程完成
            long endTime = System.currentTimeMillis();
            
            int expectedCount = threadCount * incrementsPerThread;
            int actualCount;
            
            if (useAtomic && counter instanceof Counter) {
                actualCount = ((Counter) counter).getAtomicCount();
            } else if (counter instanceof Counter) {
                actualCount = ((Counter) counter).getCount();
            } else {
                actualCount = 0;
            }
            
            System.out.println("期望结果: " + expectedCount);
            System.out.println("实际结果: " + actualCount);
            System.out.println("是否正确: " + (expectedCount == actualCount));
            System.out.println("耗时: " + (endTime - startTime) + "ms");
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 演示死锁
     */
    private static void demonstrateDeadlock() {
        System.out.println("\n=== 死锁演示 ===");
        
        final Object lock1 = new Object();
        final Object lock2 = new Object();
        
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("线程1获得lock1");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                
                System.out.println("线程1等待lock2");
                synchronized (lock2) {
                    System.out.println("线程1获得lock2");
                }
            }
        }, "DeadLock-Thread-1");
        
        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("线程2获得lock2");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                
                System.out.println("线程2等待lock1");
                synchronized (lock1) {
                    System.out.println("线程2获得lock1");
                }
            }
        }, "DeadLock-Thread-2");
        
        thread1.start();
        thread2.start();
        
        // 等待一段时间后检查死锁
        try {
            Thread.sleep(2000);
            if (thread1.isAlive() && thread2.isAlive()) {
                System.out.println("检测到死锁！");
                System.out.println("线程1状态: " + thread1.getState());
                System.out.println("线程2状态: " + thread2.getState());
                
                // 中断线程以解除死锁
                thread1.interrupt();
                thread2.interrupt();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### 1.2 线程池和并发工具
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.List;
import java.util.ArrayList;
import java.util.Random;

/**
 * 线程池和并发工具演示
 * 包含各种线程池类型和并发工具类的使用
 */
public class ConcurrencyToolsDemo {
    
    public static void main(String[] args) {
        demonstrateThreadPools();
        demonstrateCountDownLatch();
        demonstrateCyclicBarrier();
        demonstrateSemaphore();
        demonstrateBlockingQueue();
        demonstrateCompletableFuture();
    }
    
    /**
     * 演示各种线程池
     */
    private static void demonstrateThreadPools() {
        System.out.println("=== 线程池演示 ===");
        
        // 1. 固定大小线程池
        System.out.println("\n1. 固定大小线程池:");
        ExecutorService fixedPool = Executors.newFixedThreadPool(3);
        
        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            fixedPool.submit(() -> {
                System.out.println("固定线程池任务 " + taskId + " 在线程 " + 
                                 Thread.currentThread().getName() + " 中执行");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        fixedPool.shutdown();
        
        // 2. 缓存线程池
        System.out.println("\n2. 缓存线程池:");
        ExecutorService cachedPool = Executors.newCachedThreadPool();
        
        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            cachedPool.submit(() -> {
                System.out.println("缓存线程池任务 " + taskId + " 在线程 " + 
                                 Thread.currentThread().getName() + " 中执行");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        cachedPool.shutdown();
        
        // 3. 单线程线程池
        System.out.println("\n3. 单线程线程池:");
        ExecutorService singlePool = Executors.newSingleThreadExecutor();
        
        for (int i = 0; i < 3; i++) {
            final int taskId = i;
            singlePool.submit(() -> {
                System.out.println("单线程池任务 " + taskId + " 在线程 " + 
                                 Thread.currentThread().getName() + " 中执行");
                try {
                    Thread.sleep(800);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        singlePool.shutdown();
        
        // 4. 定时线程池
        System.out.println("\n4. 定时线程池:");
        ScheduledExecutorService scheduledPool = Executors.newScheduledThreadPool(2);
        
        // 延迟执行
        scheduledPool.schedule(() -> {
            System.out.println("延迟2秒执行的任务");
        }, 2, TimeUnit.SECONDS);
        
        // 固定频率执行
        ScheduledFuture<?> future = scheduledPool.scheduleAtFixedRate(() -> {
            System.out.println("每1秒执行一次的任务: " + System.currentTimeMillis());
        }, 1, 1, TimeUnit.SECONDS);
        
        // 5秒后取消定时任务
        scheduledPool.schedule(() -> {
            future.cancel(true);
            System.out.println("定时任务已取消");
            scheduledPool.shutdown();
        }, 5, TimeUnit.SECONDS);
        
        // 5. 自定义线程池
        System.out.println("\n5. 自定义线程池:");
        ThreadPoolExecutor customPool = new ThreadPoolExecutor(
            2,                      // 核心线程数
            4,                      // 最大线程数
            60L,                    // 空闲线程存活时间
            TimeUnit.SECONDS,       // 时间单位
            new LinkedBlockingQueue<>(10), // 工作队列
            new ThreadFactory() {   // 线程工厂
                private AtomicInteger counter = new AtomicInteger(0);
                @Override
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(r, "CustomPool-" + counter.incrementAndGet());
                    t.setDaemon(false);
                    return t;
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
        );
        
        for (int i = 0; i < 8; i++) {
            final int taskId = i;
            customPool.submit(() -> {
                System.out.println("自定义线程池任务 " + taskId + " 在线程 " + 
                                 Thread.currentThread().getName() + " 中执行");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        // 监控线程池状态
        System.out.println("线程池状态:");
        System.out.println("核心线程数: " + customPool.getCorePoolSize());
        System.out.println("当前线程数: " + customPool.getPoolSize());
        System.out.println("活跃线程数: " + customPool.getActiveCount());
        System.out.println("队列中任务数: " + customPool.getQueue().size());
        
        customPool.shutdown();
    }
    
    /**
     * 演示CountDownLatch
     * 用于等待多个线程完成
     */
    private static void demonstrateCountDownLatch() {
        System.out.println("\n=== CountDownLatch演示 ===");
        
        int workerCount = 5;
        CountDownLatch latch = new CountDownLatch(workerCount);
        
        System.out.println("主线程等待 " + workerCount + " 个工作线程完成...");
        
        // 启动工作线程
        for (int i = 0; i < workerCount; i++) {
            final int workerId = i + 1;
            new Thread(() -> {
                try {
                    // 模拟工作
                    int workTime = new Random().nextInt(3000) + 1000;
                    Thread.sleep(workTime);
                    System.out.println("工作线程 " + workerId + " 完成工作，耗时 " + workTime + "ms");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    latch.countDown(); // 计数器减1
                }
            }, "Worker-" + workerId).start();
        }
        
        try {
            latch.await(); // 等待所有工作线程完成
            System.out.println("所有工作线程已完成，主线程继续执行");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * 演示CyclicBarrier
     * 用于让多个线程在某个点等待，直到所有线程都到达
     */
    private static void demonstrateCyclicBarrier() {
        System.out.println("\n=== CyclicBarrier演示 ===");
        
        int participantCount = 4;
        CyclicBarrier barrier = new CyclicBarrier(participantCount, () -> {
            System.out.println("所有参与者都到达了屏障点，开始下一阶段！");
        });
        
        for (int i = 0; i < participantCount; i++) {
            final int participantId = i + 1;
            new Thread(() -> {
                try {
                    // 第一阶段工作
                    int workTime1 = new Random().nextInt(2000) + 500;
                    Thread.sleep(workTime1);
                    System.out.println("参与者 " + participantId + " 完成第一阶段工作");
                    
                    barrier.await(); // 等待其他参与者
                    
                    // 第二阶段工作
                    int workTime2 = new Random().nextInt(1500) + 500;
                    Thread.sleep(workTime2);
                    System.out.println("参与者 " + participantId + " 完成第二阶段工作");
                    
                    barrier.await(); // 再次等待
                    
                    System.out.println("参与者 " + participantId + " 完成所有工作");
                    
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, "Participant-" + participantId).start();
        }
    }
    
    /**
     * 演示Semaphore
     * 用于控制同时访问资源的线程数量
     */
    private static void demonstrateSemaphore() {
        System.out.println("\n=== Semaphore演示 ===");
        
        // 模拟停车场，只有3个停车位
        Semaphore parkingLot = new Semaphore(3);
        
        // 10辆车要停车
        for (int i = 0; i < 10; i++) {
            final int carId = i + 1;
            new Thread(() -> {
                try {
                    System.out.println("车辆 " + carId + " 到达停车场，等待停车位...");
                    
                    parkingLot.acquire(); // 获取停车位
                    System.out.println("车辆 " + carId + " 获得停车位，开始停车");
                    
                    // 模拟停车时间
                    Thread.sleep(new Random().nextInt(3000) + 1000);
                    
                    System.out.println("车辆 " + carId + " 离开停车位");
                    
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    parkingLot.release(); // 释放停车位
                }
            }, "Car-" + carId).start();
        }
    }
    
    /**
     * 演示阻塞队列
     * 用于生产者-消费者模式
     */
    private static void demonstrateBlockingQueue() {
        System.out.println("\n=== BlockingQueue演示 ===");
        
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);
        
        // 生产者
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    String product = "产品-" + i;
                    queue.put(product); // 阻塞式添加
                    System.out.println("生产者生产: " + product + "，队列大小: " + queue.size());
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Producer");
        
        // 消费者1
        Thread consumer1 = new Thread(() -> {
            try {
                while (true) {
                    String product = queue.take(); // 阻塞式获取
                    System.out.println("消费者1消费: " + product + "，队列大小: " + queue.size());
                    Thread.sleep(800);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer-1");
        
        // 消费者2
        Thread consumer2 = new Thread(() -> {
            try {
                while (true) {
                    String product = queue.take();
                    System.out.println("消费者2消费: " + product + "，队列大小: " + queue.size());
                    Thread.sleep(1200);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer-2");
        
        producer.start();
        consumer1.start();
        consumer2.start();
        
        // 10秒后停止
        try {
            Thread.sleep(10000);
            producer.interrupt();
            consumer1.interrupt();
            consumer2.interrupt();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * 演示CompletableFuture
     * 用于异步编程和组合多个异步操作
     */
    private static void demonstrateCompletableFuture() {
        System.out.println("\n=== CompletableFuture演示 ===");
        
        // 1. 简单异步执行
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "异步任务1完成";
        });
        
        future1.thenAccept(result -> System.out.println("结果: " + result));
        
        // 2. 链式调用
        CompletableFuture<String> future2 = CompletableFuture
            .supplyAsync(() -> {
                System.out.println("执行第一步");
                return "第一步结果";
            })
            .thenApply(result -> {
                System.out.println("执行第二步，输入: " + result);
                return "第二步结果";
            })
            .thenApply(result -> {
                System.out.println("执行第三步，输入: " + result);
                return "最终结果";
            });
        
        try {
            String finalResult = future2.get();
            System.out.println("链式调用最终结果: " + finalResult);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        
        // 3. 组合多个异步操作
        CompletableFuture<String> task1 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "任务1";
        });
        
        CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "任务2";
        });
        
        CompletableFuture<String> combined = task1.thenCombine(task2, (result1, result2) -> {
            return result1 + " + " + result2 + " = 组合结果";
        });
        
        try {
            String combinedResult = combined.get();
            System.out.println("组合结果: " + combinedResult);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        
        // 4. 等待所有任务完成
        List<CompletableFuture<String>> futures = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(new Random().nextInt(2000) + 500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                return "并行任务" + taskId + "完成";
            });
            futures.add(future);
        }
        
        CompletableFuture<Void> allTasks = CompletableFuture.allOf(
            futures.toArray(new CompletableFuture[0])
        );
        
        allTasks.thenRun(() -> {
            System.out.println("所有并行任务都已完成");
            futures.forEach(f -> {
                try {
                    System.out.println("- " + f.get());
                } catch (InterruptedException | ExecutionException e) {
                    e.printStackTrace();
                }
            });
        });
        
        try {
            allTasks.get(); // 等待所有任务完成
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

### 2. IO流和NIO

#### 2.1 传统IO流
```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Java IO流详解
 * 包含字节流、字符流、缓冲流等的使用
 */
public class IOStreamDemo {
    
    private static final String BASE_PATH = "io_demo/";
    
    public static void main(String[] args) {
        // 创建演示目录
        createDemoDirectory();
        
        demonstrateByteStreams();
        demonstrateCharacterStreams();
        demonstrateBufferedStreams();
        demonstrateObjectStreams();
        demonstrateFileOperations();
        demonstrateSystemStreams();
    }
    
    /**
     * 创建演示目录
     */
    private static void createDemoDirectory() {
        File dir = new File(BASE_PATH);
        if (!dir.exists()) {
            dir.mkdirs();
            System.out.println("创建演示目录: " + dir.getAbsolutePath());
        }
    }
    
    /**
     * 演示字节流
     */
    private static void demonstrateByteStreams() {
        System.out.println("\n=== 字节流演示 ===");
        
        String fileName = BASE_PATH + "byte_stream_demo.txt";
        String content = "这是字节流演示内容\nHello World\n你好世界";
        
        // 使用FileOutputStream写入数据
        try (FileOutputStream fos = new FileOutputStream(fileName)) {
            byte[] bytes = content.getBytes(StandardCharsets.UTF_8);
            fos.write(bytes);
            System.out.println("使用FileOutputStream写入数据成功");
        } catch (IOException e) {
            System.err.println("写入失败: " + e.getMessage());
        }
        
        // 使用FileInputStream读取数据
        try (FileInputStream fis = new FileInputStream(fileName)) {
            byte[] buffer = new byte[1024];
            int bytesRead = fis.read(buffer);
            String readContent = new String(buffer, 0, bytesRead, StandardCharsets.UTF_8);
            System.out.println("使用FileInputStream读取的内容:");
            System.out.println(readContent);
        } catch (IOException e) {
            System.err.println("读取失败: " + e.getMessage());
        }
        
        // 逐字节读取（效率较低，仅作演示）
        try (FileInputStream fis = new FileInputStream(fileName)) {
            System.out.println("\n逐字节读取（十六进制）:");
            int byteData;
            int count = 0;
            while ((byteData = fis.read()) != -1 && count < 20) {
                System.out.printf("%02X ", byteData);
                count++;
            }
            System.out.println();
        } catch (IOException e) {
            System.err.println("逐字节读取失败: " + e.getMessage());
        }
    }
    
    /**
     * 演示字符流
     */
    private static void demonstrateCharacterStreams() {
        System.out.println("\n=== 字符流演示 ===");
        
        String fileName = BASE_PATH + "character_stream_demo.txt";
        String content = "字符流演示\n支持中文字符\nCharacter Stream Demo\n";
        
        // 使用FileWriter写入字符数据
        try (FileWriter writer = new FileWriter(fileName, StandardCharsets.UTF_8)) {
            writer.write(content);
            writer.write("追加的内容\n");
            writer.flush(); // 强制刷新缓冲区
            System.out.println("使用FileWriter写入字符数据成功");
        } catch (IOException e) {
            System.err.println("写入失败: " + e.getMessage());
        }
        
        // 使用FileReader读取字符数据
        try (FileReader reader = new FileReader(fileName, StandardCharsets.UTF_8)) {
            char[] buffer = new char[1024];
            int charsRead = reader.read(buffer);
            String readContent = new String(buffer, 0, charsRead);
            System.out.println("使用FileReader读取的内容:");
            System.out.println(readContent);
        } catch (IOException e) {
            System.err.println("读取失败: " + e.getMessage());
        }
        
        // 逐字符读取
        try (FileReader reader = new FileReader(fileName, StandardCharsets.UTF_8)) {
            System.out.println("逐字符读取:");
            int charData;
            int count = 0;
            while ((charData = reader.read()) != -1 && count < 10) {
                System.out.print((char) charData);
                count++;
            }
            System.out.println();
        } catch (IOException e) {
            System.err.println("逐字符读取失败: " + e.getMessage());
        }
    }
    
    /**
     * 演示缓冲流
     */
    private static void demonstrateBufferedStreams() {
        System.out.println("\n=== 缓冲流演示 ===");
        
        String fileName = BASE_PATH + "buffered_stream_demo.txt";
        
        // 使用BufferedWriter写入数据
        try (BufferedWriter writer = new BufferedWriter(
                new FileWriter(fileName, StandardCharsets.UTF_8))) {
            
            for (int i = 1; i <= 5; i++) {
                writer.write("这是第 " + i + " 行数据");
                writer.newLine(); // 写入系统相关的换行符
            }
            
            writer.write("缓冲流可以提高IO性能");
            writer.newLine();
            
            System.out.println("使用BufferedWriter写入数据成功");
        } catch (IOException e) {
            System.err.println("写入失败: " + e.getMessage());
        }
        
        // 使用BufferedReader读取数据
        try (BufferedReader reader = new BufferedReader(
                new FileReader(fileName, StandardCharsets.UTF_8))) {
            
            System.out.println("使用BufferedReader逐行读取:");
            String line;
            int lineNumber = 1;
            while ((line = reader.readLine()) != null) {
                System.out.println(lineNumber + ": " + line);
                lineNumber++;
            }
        } catch (IOException e) {
            System.err.println("读取失败: " + e.getMessage());
        }
        
        // 性能对比：缓冲流 vs 普通流
        performanceComparison();
    }
    
    /**
     * 性能对比
     */
    private static void performanceComparison() {
        System.out.println("\n性能对比测试:");
        
        String fileName1 = BASE_PATH + "performance_test1.txt";
        String fileName2 = BASE_PATH + "performance_test2.txt";
        int iterations = 10000;
        
        // 测试普通流性能
        long startTime = System.currentTimeMillis();
        try (FileWriter writer = new FileWriter(fileName1)) {
            for (int i = 0; i < iterations; i++) {
                writer.write("测试数据 " + i + "\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long normalStreamTime = System.currentTimeMillis() - startTime;
        
        // 测试缓冲流性能
        startTime = System.currentTimeMillis();
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(fileName2))) {
            for (int i = 0; i < iterations; i++) {
                writer.write("测试数据 " + i + "\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long bufferedStreamTime = System.currentTimeMillis() - startTime;
        
        System.out.println("写入 " + iterations + " 行数据:");
        System.out.println("普通流耗时: " + normalStreamTime + "ms");
        System.out.println("缓冲流耗时: " + bufferedStreamTime + "ms");
        System.out.println("性能提升: " + (normalStreamTime / (double) bufferedStreamTime) + " 倍");
    }
    
    /**
     * 演示对象流（序列化）
     */
    private static void demonstrateObjectStreams() {
        System.out.println("\n=== 对象流演示 ===");
        
        String fileName = BASE_PATH + "object_stream_demo.ser";
        
        // 创建可序列化的对象
        Person person = new Person("张三", 25, "北京市");
        
        // 序列化对象
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream(fileName))) {
            
            oos.writeObject(person);
            oos.writeInt(12345);
            oos.writeUTF("序列化字符串");
            
            System.out.println("对象序列化成功: " + person);
        } catch (IOException e) {
            System.err.println("序列化失败: " + e.getMessage());
        }
        
        // 反序列化对象
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream(fileName))) {
            
            Person deserializedPerson = (Person) ois.readObject();
            int number = ois.readInt();
            String text = ois.readUTF();
            
            System.out.println("对象反序列化成功: " + deserializedPerson);
            System.out.println("反序列化的数字: " + number);
            System.out.println("反序列化的字符串: " + text);
            
        } catch (IOException | ClassNotFoundException e) {
            System.err.println("反序列化失败: " + e.getMessage());
        }
    }
    
    /**
     * 演示文件操作
     */
    private static void demonstrateFileOperations() {
        System.out.println("\n=== 文件操作演示 ===");
        
        String fileName = BASE_PATH + "file_operations_demo.txt";
        File file = new File(fileName);
        
        try {
            // 创建文件
            if (file.createNewFile()) {
                System.out.println("文件创建成功: " + file.getAbsolutePath());
            } else {
                System.out.println("文件已存在: " + file.getAbsolutePath());
            }
            
            // 写入内容
            try (PrintWriter writer = new PrintWriter(file, StandardCharsets.UTF_8)) {
                writer.println("文件操作演示");
                writer.println("当前时间: " + System.currentTimeMillis());
                writer.printf("格式化输出: %s = %d\n", "数字", 42);
            }
            
            // 文件信息
            System.out.println("\n文件信息:");
            System.out.println("文件名: " + file.getName());
            System.out.println("文件路径: " + file.getPath());
            System.out.println("绝对路径: " + file.getAbsolutePath());
            System.out.println("文件大小: " + file.length() + " 字节");
            System.out.println("是否可读: " + file.canRead());
            System.out.println("是否可写: " + file.canWrite());
            System.out.println("是否可执行: " + file.canExecute());
            System.out.println("最后修改时间: " + new java.util.Date(file.lastModified()));
            
            // 目录操作
            File dir = new File(BASE_PATH + "subdir");
            if (dir.mkdir()) {
                System.out.println("\n子目录创建成功: " + dir.getAbsolutePath());
            }
            
            // 列出目录内容
            File baseDir = new File(BASE_PATH);
            System.out.println("\n目录内容:");
            File[] files = baseDir.listFiles();
            if (files != null) {
                for (File f : files) {
                    String type = f.isDirectory() ? "[目录]" : "[文件]";
                    System.out.println(type + " " + f.getName());
                }
            }
            
        } catch (IOException e) {
            System.err.println("文件操作失败: " + e.getMessage());
        }
    }
    
    /**
     * 演示系统流
     */
    private static void demonstrateSystemStreams() {
        System.out.println("\n=== 系统流演示 ===");
        
        // 标准输出
        System.out.println("这是标准输出");
        
        // 标准错误输出
        System.err.println("这是标准错误输出");
        
        // 重定向标准输出到文件
        String fileName = BASE_PATH + "system_output.txt";
        try {
            PrintStream fileOut = new PrintStream(new FileOutputStream(fileName));
            PrintStream originalOut = System.out;
            
            System.setOut(fileOut);
            System.out.println("这行输出被重定向到文件");
            System.out.println("当前时间: " + new java.util.Date());
            
            // 恢复标准输出
            System.setOut(originalOut);
            fileOut.close();
            
            System.out.println("输出已重定向到文件: " + fileName);
            
            // 读取重定向的内容
            try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
                System.out.println("重定向文件的内容:");
                String line;
                while ((line = reader.readLine()) != null) {
                    System.out.println("  " + line);
                }
            }
            
        } catch (IOException e) {
            System.err.println("重定向失败: " + e.getMessage());
        }
        
        // 标准输入演示（注释掉避免阻塞）
        /*
        System.out.print("请输入一些文本: ");
        try (Scanner scanner = new Scanner(System.in)) {
            String input = scanner.nextLine();
            System.out.println("你输入的是: " + input);
        }
        */
    }
    
    /**
     * 可序列化的Person类
     */
    static class Person implements Serializable {
        private static final long serialVersionUID = 1L;
        
        private String name;
        private int age;
        private String address;
        
        public Person(String name, int age, String address) {
            this.name = name;
            this.age = age;
            this.address = address;
        }
        
        @Override
        public String toString() {
            return String.format("Person{name='%s', age=%d, address='%s'}", 
                               name, age, address);
        }
        
        // Getter方法
        public String getName() { return name; }
        public int getAge() { return age; }
        public String getAddress() { return address; }
    }
}
```

#### 2.2 NIO (New IO)
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.Iterator;
import java.util.Set;

/**
 * Java NIO演示
 * 包含Channel、Buffer、Selector的使用
 */
public class NIODemo {
    
    private static final String BASE_PATH = "nio_demo/";
    
    public static void main(String[] args) {
        createDemoDirectory();
        
        demonstrateBuffer();
        demonstrateFileChannel();
        demonstratePathAndFiles();
        demonstrateSelector();
    }
    
    private static void createDemoDirectory() {
        try {
            Path dir = Paths.get(BASE_PATH);
            if (!Files.exists(dir)) {
                Files.createDirectories(dir);
                System.out.println("创建NIO演示目录: " + dir.toAbsolutePath());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 演示Path和Files API
     */
    private static void demonstratePathAndFiles() {
        System.out.println("\n=== Path和Files API演示 ===");
        
        // Path操作
        System.out.println("\n1. Path操作:");
        Path path1 = Paths.get(BASE_PATH, "test", "example.txt");
        Path path2 = Paths.get(BASE_PATH + "test/example.txt");
        
        System.out.println("路径1: " + path1);
        System.out.println("路径2: " + path2);
        System.out.println("路径是否相等: " + path1.equals(path2));
        
        System.out.println("\n路径信息:");
        System.out.println("文件名: " + path1.getFileName());
        System.out.println("父目录: " + path1.getParent());
        System.out.println("根目录: " + path1.getRoot());
        System.out.println("路径元素数量: " + path1.getNameCount());
        
        for (int i = 0; i < path1.getNameCount(); i++) {
            System.out.println("路径元素[" + i + "]: " + path1.getName(i));
        }
        
        // 路径解析
        Path basePath = Paths.get(BASE_PATH);
        Path relativePath = Paths.get("subdir/file.txt");
        Path resolvedPath = basePath.resolve(relativePath);
        System.out.println("\n解析后的路径: " + resolvedPath);
        
        // Files操作
        System.out.println("\n2. Files操作:");
        Path testFile = Paths.get(BASE_PATH + "files_demo.txt");
        
        try {
            // 创建文件
            if (!Files.exists(testFile)) {
                Files.createFile(testFile);
                System.out.println("文件创建成功: " + testFile);
            }
            
            // 写入内容
            String content = "Files API演示\n支持更简洁的文件操作\n";
            Files.write(testFile, content.getBytes(StandardCharsets.UTF_8));
            System.out.println("内容写入成功");
            
            // 读取内容
            byte[] bytes = Files.readAllBytes(testFile);
            String readContent = new String(bytes, StandardCharsets.UTF_8);
            System.out.println("读取内容:\n" + readContent);
            
            // 文件属性
            BasicFileAttributes attrs = Files.readAttributes(testFile, BasicFileAttributes.class);
            System.out.println("\n文件属性:");
            System.out.println("创建时间: " + attrs.creationTime());
            System.out.println("最后修改时间: " + attrs.lastModifiedTime());
            System.out.println("文件大小: " + attrs.size() + " 字节");
            System.out.println("是否为目录: " + attrs.isDirectory());
            System.out.println("是否为常规文件: " + attrs.isRegularFile());
            
            // 目录遍历
            System.out.println("\n3. 目录遍历:");
            Path baseDir = Paths.get(BASE_PATH);
            
            System.out.println("直接子项:");
            try (DirectoryStream<Path> stream = Files.newDirectoryStream(baseDir)) {
                for (Path entry : stream) {
                    String type = Files.isDirectory(entry) ? "[目录]" : "[文件]";
                    System.out.println(type + " " + entry.getFileName());
                }
            }
            
            // 递归遍历
            System.out.println("\n递归遍历:");
            Files.walkFileTree(baseDir, new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                    System.out.println("文件: " + file);
                    return FileVisitResult.CONTINUE;
                }
                
                @Override
                public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
                    System.out.println("目录: " + dir);
                    return FileVisitResult.CONTINUE;
                }
            });
            
        } catch (IOException e) {
             e.printStackTrace();
         }
     }
     
     /**
      * 演示Selector（非阻塞IO）
      */
     private static void demonstrateSelector() {
         System.out.println("\n=== Selector演示 ===");
         
         try {
             // 创建Selector
             Selector selector = Selector.open();
             
             // 创建ServerSocketChannel
             ServerSocketChannel serverChannel = ServerSocketChannel.open();
             serverChannel.configureBlocking(false); // 设置为非阻塞模式
             serverChannel.bind(new InetSocketAddress(8080));
             
             // 注册到Selector
             SelectionKey serverKey = serverChannel.register(selector, SelectionKey.OP_ACCEPT);
             System.out.println("服务器启动，监听端口8080");
             
             // 模拟客户端连接
             Thread clientThread = new Thread(() -> {
                 try {
                     Thread.sleep(1000); // 等待服务器启动
                     
                     SocketChannel clientChannel = SocketChannel.open();
                     clientChannel.connect(new InetSocketAddress("localhost", 8080));
                     
                     // 发送数据
                     String message = "Hello from NIO client!";
                     ByteBuffer buffer = ByteBuffer.wrap(message.getBytes());
                     clientChannel.write(buffer);
                     
                     // 读取响应
                     ByteBuffer responseBuffer = ByteBuffer.allocate(1024);
                     int bytesRead = clientChannel.read(responseBuffer);
                     if (bytesRead > 0) {
                         responseBuffer.flip();
                         String response = StandardCharsets.UTF_8.decode(responseBuffer).toString();
                         System.out.println("客户端收到响应: " + response);
                     }
                     
                     clientChannel.close();
                     
                 } catch (IOException | InterruptedException e) {
                     e.printStackTrace();
                 }
             });
             
             clientThread.start();
             
             // 服务器事件循环
             int eventCount = 0;
             while (eventCount < 5) { // 限制循环次数
                 int readyChannels = selector.select(2000); // 2秒超时
                 
                 if (readyChannels == 0) {
                     System.out.println("没有就绪的通道");
                     continue;
                 }
                 
                 Set<SelectionKey> selectedKeys = selector.selectedKeys();
                 Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
                 
                 while (keyIterator.hasNext()) {
                     SelectionKey key = keyIterator.next();
                     
                     if (key.isAcceptable()) {
                         // 处理连接请求
                         ServerSocketChannel server = (ServerSocketChannel) key.channel();
                         SocketChannel client = server.accept();
                         client.configureBlocking(false);
                         
                         // 注册读事件
                         client.register(selector, SelectionKey.OP_READ);
                         System.out.println("接受新连接: " + client.getRemoteAddress());
                         
                     } else if (key.isReadable()) {
                         // 处理读事件
                         SocketChannel client = (SocketChannel) key.channel();
                         ByteBuffer buffer = ByteBuffer.allocate(1024);
                         
                         int bytesRead = client.read(buffer);
                         if (bytesRead > 0) {
                             buffer.flip();
                             String message = StandardCharsets.UTF_8.decode(buffer).toString();
                             System.out.println("服务器收到消息: " + message);
                             
                             // 发送响应
                             String response = "Echo: " + message;
                             ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                             client.write(responseBuffer);
                             
                             // 注册写事件
                             key.interestOps(SelectionKey.OP_WRITE);
                         } else if (bytesRead == -1) {
                             // 客户端关闭连接
                             System.out.println("客户端断开连接");
                             key.cancel();
                             client.close();
                         }
                         
                     } else if (key.isWritable()) {
                         // 处理写事件
                         System.out.println("数据写入完成");
                         key.interestOps(SelectionKey.OP_READ); // 重新关注读事件
                     }
                     
                     keyIterator.remove();
                     eventCount++;
                 }
             }
             
             // 清理资源
             serverChannel.close();
             selector.close();
             clientThread.join();
             
             System.out.println("Selector演示完成");
             
         } catch (IOException | InterruptedException e) {
             e.printStackTrace();
         }
     }
}
```

### 3. 反射机制

```java
import java.lang.reflect.*;
import java.util.Arrays;

/**
 * Java反射机制演示
 * 包含Class对象、构造器、方法、字段的反射操作
 */
public class ReflectionDemo {
    
    public static void main(String[] args) {
        demonstrateClassObject();
        demonstrateConstructors();
        demonstrateMethods();
        demonstrateFields();
        demonstrateAnnotations();
        demonstrateDynamicProxy();
    }
    
    /**
     * 演示Class对象的获取和使用
     */
    private static void demonstrateClassObject() {
        System.out.println("=== Class对象演示 ===");
        
        try {
            // 获取Class对象的三种方式
            
            // 方式1：通过类名.class
            Class<?> clazz1 = String.class;
            System.out.println("方式1 - 类名.class: " + clazz1.getName());
            
            // 方式2：通过对象.getClass()
            String str = "Hello";
            Class<?> clazz2 = str.getClass();
            System.out.println("方式2 - 对象.getClass(): " + clazz2.getName());
            
            // 方式3：通过Class.forName()
            Class<?> clazz3 = Class.forName("java.lang.String");
            System.out.println("方式3 - Class.forName(): " + clazz3.getName());
            
            // 验证三种方式获取的是同一个Class对象
            System.out.println("三个Class对象是否相等: " + (clazz1 == clazz2 && clazz2 == clazz3));
            
            // Class对象信息
            Class<?> personClass = Person.class;
            System.out.println("\nClass信息:");
            System.out.println("类名: " + personClass.getName());
            System.out.println("简单类名: " + personClass.getSimpleName());
            System.out.println("包名: " + personClass.getPackage().getName());
            System.out.println("父类: " + personClass.getSuperclass().getName());
            System.out.println("是否为接口: " + personClass.isInterface());
            System.out.println("是否为数组: " + personClass.isArray());
            System.out.println("是否为枚举: " + personClass.isEnum());
            
            // 接口信息
            Class<?>[] interfaces = personClass.getInterfaces();
            System.out.println("实现的接口: " + Arrays.toString(interfaces));
            
        } catch (ClassNotFoundException e) {
             e.printStackTrace();
         }
     }
     
     /**
      * 演示构造器反射
      */
     private static void demonstrateConstructors() {
         System.out.println("\n=== 构造器反射演示 ===");
         
         try {
             Class<?> personClass = Person.class;
             
             // 获取所有构造器
             Constructor<?>[] constructors = personClass.getConstructors();
             System.out.println("所有public构造器:");
             for (Constructor<?> constructor : constructors) {
                 System.out.println("  " + constructor);
                 
                 // 构造器参数信息
                 Class<?>[] paramTypes = constructor.getParameterTypes();
                 System.out.println("    参数类型: " + Arrays.toString(paramTypes));
             }
             
             // 获取特定构造器并创建对象
             Constructor<?> constructor = personClass.getConstructor(String.class, int.class);
             Object person = constructor.newInstance("反射创建", 30);
             System.out.println("\n通过反射创建的对象: " + person);
             
             // 获取默认构造器
             try {
                 Constructor<?> defaultConstructor = personClass.getConstructor();
                 Object defaultPerson = defaultConstructor.newInstance();
                 System.out.println("通过默认构造器创建: " + defaultPerson);
             } catch (NoSuchMethodException e) {
                 System.out.println("没有默认构造器");
             }
             
         } catch (Exception e) {
             e.printStackTrace();
         }
     }
     
     /**
      * 演示方法反射
      */
     private static void demonstrateMethods() {
         System.out.println("\n=== 方法反射演示 ===");
         
         try {
             Class<?> personClass = Person.class;
             Object person = personClass.getConstructor(String.class, int.class)
                                      .newInstance("方法测试", 25);
             
             // 获取所有public方法（包括继承的）
             Method[] methods = personClass.getMethods();
             System.out.println("所有public方法数量: " + methods.length);
             
             // 获取声明的方法（不包括继承的）
             Method[] declaredMethods = personClass.getDeclaredMethods();
             System.out.println("声明的方法:");
             for (Method method : declaredMethods) {
                 System.out.println("  " + method.getName() + " - " + method);
             }
             
             // 调用getter方法
             Method getNameMethod = personClass.getMethod("getName");
             String name = (String) getNameMethod.invoke(person);
             System.out.println("\n通过反射调用getName(): " + name);
             
             // 调用setter方法
             Method setNameMethod = personClass.getMethod("setName", String.class);
             setNameMethod.invoke(person, "新名字");
             System.out.println("设置新名字后: " + person);
             
             // 调用私有方法
             Method privateMethod = personClass.getDeclaredMethod("privateMethod");
             privateMethod.setAccessible(true); // 设置可访问
             String result = (String) privateMethod.invoke(person);
             System.out.println("调用私有方法结果: " + result);
             
             // 调用静态方法
             Method staticMethod = personClass.getMethod("getSpecies");
             String species = (String) staticMethod.invoke(null); // 静态方法传null
             System.out.println("调用静态方法结果: " + species);
             
         } catch (Exception e) {
             e.printStackTrace();
         }
     }
     
     /**
      * 演示字段反射
      */
     private static void demonstrateFields() {
         System.out.println("\n=== 字段反射演示 ===");
         
         try {
             Class<?> personClass = Person.class;
             Object person = personClass.getConstructor(String.class, int.class)
                                      .newInstance("字段测试", 28);
             
             // 获取所有字段
             Field[] fields = personClass.getDeclaredFields();
             System.out.println("所有声明的字段:");
             for (Field field : fields) {
                 System.out.println("  " + field.getName() + " - " + field.getType().getSimpleName());
                 System.out.println("    修饰符: " + Modifier.toString(field.getModifiers()));
             }
             
             // 访问私有字段
             Field nameField = personClass.getDeclaredField("name");
             nameField.setAccessible(true); // 设置可访问
             
             String originalName = (String) nameField.get(person);
             System.out.println("\n原始name值: " + originalName);
             
             // 修改私有字段
             nameField.set(person, "反射修改的名字");
             System.out.println("修改后的对象: " + person);
             
             // 访问静态字段
             Field staticField = personClass.getDeclaredField("species");
             staticField.setAccessible(true);
             String speciesValue = (String) staticField.get(null); // 静态字段传null
             System.out.println("静态字段species值: " + speciesValue);
             
             // 修改final字段（注意：这在某些JVM版本中可能不起作用）
             try {
                 Field finalField = personClass.getDeclaredField("id");
                 finalField.setAccessible(true);
                 
                 // 移除final修饰符
                 Field modifiersField = Field.class.getDeclaredField("modifiers");
                 modifiersField.setAccessible(true);
                 modifiersField.setInt(finalField, finalField.getModifiers() & ~Modifier.FINAL);
                 
                 long originalId = finalField.getLong(person);
                 System.out.println("原始ID: " + originalId);
                 
                 finalField.setLong(person, 99999L);
                 System.out.println("修改final字段后的对象: " + person);
                 
             } catch (Exception e) {
                 System.out.println("无法修改final字段: " + e.getMessage());
             }
             
         } catch (Exception e) {
             e.printStackTrace();
         }
     }
      
      /**
       * 演示注解反射
       */
      private static void demonstrateAnnotations() {
          System.out.println("\n=== 注解反射演示 ===");
          
          try {
              Class<?> personClass = Person.class;
              
              // 类级别注解
              if (personClass.isAnnotationPresent(Entity.class)) {
                  Entity entity = personClass.getAnnotation(Entity.class);
                  System.out.println("类注解 @Entity: tableName=" + entity.tableName());
              }
              
              // 字段注解
              Field nameField = personClass.getDeclaredField("name");
              if (nameField.isAnnotationPresent(Column.class)) {
                  Column column = nameField.getAnnotation(Column.class);
                  System.out.println("字段注解 @Column: name=" + column.name() + 
                                   ", nullable=" + column.nullable());
              }
              
              // 方法注解
              Method setNameMethod = personClass.getMethod("setName", String.class);
              if (setNameMethod.isAnnotationPresent(Deprecated.class)) {
                  System.out.println("方法 setName 被标记为 @Deprecated");
              }
              
              // 获取所有注解
              System.out.println("\n类上的所有注解:");
              Annotation[] annotations = personClass.getAnnotations();
              for (Annotation annotation : annotations) {
                  System.out.println("  " + annotation);
              }
              
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
      
      /**
       * 演示动态代理
       */
      private static void demonstrateDynamicProxy() {
          System.out.println("\n=== 动态代理演示 ===");
          
          // 创建目标对象
          UserService userService = new UserServiceImpl();
          
          // 创建代理对象
          UserService proxy = (UserService) Proxy.newProxyInstance(
              userService.getClass().getClassLoader(),
              userService.getClass().getInterfaces(),
              new InvocationHandler() {
                  @Override
                  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                      System.out.println("代理前置处理: 调用方法 " + method.getName());
                      
                      long startTime = System.currentTimeMillis();
                      
                      // 调用目标方法
                      Object result = method.invoke(userService, args);
                      
                      long endTime = System.currentTimeMillis();
                      System.out.println("代理后置处理: 方法执行耗时 " + (endTime - startTime) + "ms");
                      
                      return result;
                  }
              }
          );
          
          // 使用代理对象
          System.out.println("\n使用代理对象:");
          proxy.save("张三");
          User user = proxy.findById(1L);
          System.out.println("查询结果: " + user);
      }
      
      // 支持类和接口定义
      
      /**
       * 自定义注解
       */
      @interface Entity {
          String tableName() default "";
      }
      
      @interface Column {
          String name() default "";
          boolean nullable() default true;
      }
      
      /**
       * 测试用的Person类
       */
      @Entity(tableName = "person")
      static class Person {
          private final long id;
          
          @Column(name = "person_name", nullable = false)
          private String name;
          
          private int age;
          private static final String species = "Homo sapiens";
          
          public Person() {
              this.id = System.currentTimeMillis();
              this.name = "默认名字";
              this.age = 0;
          }
          
          public Person(String name, int age) {
              this.id = System.currentTimeMillis();
              this.name = name;
              this.age = age;
          }
          
          public String getName() {
              return name;
          }
          
          @Deprecated
          public void setName(String name) {
              this.name = name;
          }
          
          public int getAge() {
              return age;
          }
          
          public void setAge(int age) {
              this.age = age;
          }
          
          public long getId() {
              return id;
          }
          
          public static String getSpecies() {
              return species;
          }
          
          private String privateMethod() {
              return "这是私有方法的返回值";
          }
          
          @Override
          public String toString() {
              return String.format("Person{id=%d, name='%s', age=%d}", id, name, age);
          }
      }
      
      /**
       * 用户服务接口
       */
      interface UserService {
          void save(String name);
          User findById(Long id);
      }
      
      /**
       * 用户服务实现
       */
      static class UserServiceImpl implements UserService {
          @Override
          public void save(String name) {
              System.out.println("保存用户: " + name);
              try {
                  Thread.sleep(100); // 模拟数据库操作
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
              }
          }
          
          @Override
          public User findById(Long id) {
              System.out.println("查询用户ID: " + id);
              try {
                  Thread.sleep(50); // 模拟数据库查询
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
              }
              return new User(id, "用户" + id);
          }
      }
      
      /**
       * 用户实体类
       */
      static class User {
          private Long id;
          private String name;
          
          public User(Long id, String name) {
              this.id = id;
              this.name = name;
          }
          
          @Override
          public String toString() {
              return String.format("User{id=%d, name='%s'}", id, name);
          }
      }
  }
  ```
  
  ### 4. 注解详解
  
  ```java
  import java.lang.annotation.*;
  import java.lang.reflect.Method;
  
  /**
   * Java注解详解
   * 包含注解定义、使用和处理
   */
  public class AnnotationDemo {
      
      public static void main(String[] args) {
          demonstrateBuiltInAnnotations();
          demonstrateCustomAnnotations();
          processAnnotations();
      }
      
      /**
       * 演示内置注解
       */
      private static void demonstrateBuiltInAnnotations() {
          System.out.println("=== 内置注解演示 ===");
          
          AnnotationExample example = new AnnotationExample();
          example.deprecatedMethod();
          example.suppressWarningsMethod();
          
          System.out.println("内置注解演示完成");
      }
      
      /**
       * 演示自定义注解
       */
      private static void demonstrateCustomAnnotations() {
          System.out.println("\n=== 自定义注解演示 ===");
          
          TestRunner runner = new TestRunner();
          runner.runTests();
      }
      
      /**
       * 处理注解
       */
      private static void processAnnotations() {
          System.out.println("\n=== 注解处理演示 ===");
          
          Class<?> clazz = TestClass.class;
          
          // 处理类级别注解
          if (clazz.isAnnotationPresent(TestSuite.class)) {
              TestSuite suite = clazz.getAnnotation(TestSuite.class);
              System.out.println("测试套件: " + suite.name());
              System.out.println("描述: " + suite.description());
          }
          
          // 处理方法级别注解
          Method[] methods = clazz.getDeclaredMethods();
          for (Method method : methods) {
              if (method.isAnnotationPresent(Test.class)) {
                  Test test = method.getAnnotation(Test.class);
                  System.out.println("\n测试方法: " + method.getName());
                  System.out.println("超时时间: " + test.timeout() + "ms");
                  System.out.println("期望异常: " + test.expected().getSimpleName());
                  
                  try {
                      method.invoke(clazz.newInstance());
                  } catch (Exception e) {
                      System.out.println("执行异常: " + e.getCause().getClass().getSimpleName());
                  }
              }
          }
      }
      
      // 注解定义
      
      /**
       * 测试注解
       * @Retention(RetentionPolicy.RUNTIME) - 运行时保留
       * @Target(ElementType.METHOD) - 只能用于方法
       */
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      @interface Test {
          long timeout() default 0L;
          Class<? extends Throwable> expected() default Test.None.class;
          
          class None extends Throwable {
              private static final long serialVersionUID = 1L;
          }
      }
      
      /**
       * 测试套件注解
       */
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.TYPE)
      @interface TestSuite {
          String name();
          String description() default "";
      }
      
      /**
       * 性能测试注解
       */
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      @interface Performance {
          int iterations() default 1;
          boolean warmup() default false;
      }
      
      /**
       * 可重复注解（Java 8+）
       */
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      @Repeatable(Authors.class)
      @interface Author {
          String name();
      }
      
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      @interface Authors {
          Author[] value();
      }
      
      // 使用注解的类
      
      /**
       * 演示内置注解的类
       */
      static class AnnotationExample {
          
          @Deprecated
          public void deprecatedMethod() {
              System.out.println("这是一个被废弃的方法");
          }
          
          @SuppressWarnings({"unchecked", "rawtypes"})
          public void suppressWarningsMethod() {
              java.util.List list = new java.util.ArrayList();
              list.add("未指定泛型的集合操作");
              System.out.println("抑制警告的方法: " + list.get(0));
          }
          
          @Override
          public String toString() {
              return "AnnotationExample实例";
          }
      }
      
      /**
       * 测试类
       */
      @TestSuite(name = "基础功能测试", description = "测试基本功能是否正常")
      static class TestClass {
          
          @Test(timeout = 1000)
          public void testNormalMethod() {
              System.out.println("正常测试方法执行");
          }
          
          @Test(expected = IllegalArgumentException.class)
          public void testExceptionMethod() {
              System.out.println("期望异常的测试方法");
              throw new IllegalArgumentException("测试异常");
          }
          
          @Test(timeout = 500)
          @Performance(iterations = 3, warmup = true)
          @Author(name = "张三")
          @Author(name = "李四")
          public void testComplexMethod() {
              System.out.println("复杂测试方法执行");
              try {
                  Thread.sleep(100);
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
              }
          }
      }
      
      /**
       * 简单的测试运行器
       */
      static class TestRunner {
          
          public void runTests() {
              Class<?> testClass = TestClass.class;
              
              System.out.println("开始运行测试...");
              
              Method[] methods = testClass.getDeclaredMethods();
              int testCount = 0;
              int passCount = 0;
              
              for (Method method : methods) {
                  if (method.isAnnotationPresent(Test.class)) {
                      testCount++;
                      
                      Test test = method.getAnnotation(Test.class);
                      System.out.println("\n运行测试: " + method.getName());
                      
                      try {
                          Object instance = testClass.newInstance();
                          
                          // 检查性能注解
                          if (method.isAnnotationPresent(Performance.class)) {
                              Performance perf = method.getAnnotation(Performance.class);
                              System.out.println("性能测试 - 迭代次数: " + perf.iterations());
                          }
                          
                          // 检查作者注解
                          if (method.isAnnotationPresent(Authors.class)) {
                              Authors authors = method.getAnnotation(Authors.class);
                              System.out.print("作者: ");
                              for (Author author : authors.value()) {
                                  System.out.print(author.name() + " ");
                              }
                              System.out.println();
                          }
                          
                          long startTime = System.currentTimeMillis();
                          method.invoke(instance);
                          long endTime = System.currentTimeMillis();
                          
                          long executionTime = endTime - startTime;
                          if (test.timeout() > 0 && executionTime > test.timeout()) {
                              System.out.println("测试超时: " + executionTime + "ms > " + test.timeout() + "ms");
                          } else {
                              System.out.println("测试通过，耗时: " + executionTime + "ms");
                              passCount++;
                          }
                          
                      } catch (Exception e) {
                          Throwable cause = e.getCause();
                          if (cause != null && test.expected().isInstance(cause)) {
                              System.out.println("测试通过，捕获到期望的异常: " + cause.getClass().getSimpleName());
                              passCount++;
                          } else {
                              System.out.println("测试失败: " + (cause != null ? cause.getMessage() : e.getMessage()));
                          }
                      }
                  }
              }
              
              System.out.println("\n测试结果: " + passCount + "/" + testCount + " 通过");
          }
      }
  }
  ```
  
  ## 🎯 经典面试题
  
  ### 多线程相关
  
  **1. 什么是线程安全？如何实现线程安全？**

**答案：**

**线程安全的定义：**
线程安全是指在多线程环境下，对共享资源的访问不会导致数据不一致、程序异常或产生竞态条件。当多个线程同时访问同一个对象时，如果不用考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获得正确的结果，那这个对象就是线程安全的。

**线程不安全的示例：**
```java
public class UnsafeCounter {
    private int count = 0;
    
    // 线程不安全的方法
    public void increment() {
        count++;  // 这是三个操作：读取、加1、写入
    }
    
    public int getCount() {
        return count;
    }
    
    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter counter = new UnsafeCounter();
        
        // 创建1000个线程，每个线程执行1000次increment
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }
        
        // 等待所有线程完成
        for (Thread thread : threads) {
            thread.join();
        }
        
        // 期望结果是1000000，但实际结果通常小于这个值
        System.out.println("最终计数：" + counter.getCount());
    }
}
```

**实现线程安全的方式：**

**1. synchronized关键字**
```java
public class SynchronizedCounter {
    private int count = 0;
    
    // 同步方法
    public synchronized void increment() {
        count++;
    }
    
    // 同步代码块
    public void increment2() {
        synchronized (this) {
            count++;
        }
    }
    
    // 静态同步方法
    public static synchronized void staticMethod() {
        // 锁的是类对象
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

**2. volatile关键字（适用于特定场景）**
```java
public class VolatileExample {
    private volatile boolean flag = false;
    private volatile int count = 0;
    
    // volatile保证可见性，适用于状态标志
    public void setFlag(boolean flag) {
        this.flag = flag;
    }
    
    public boolean isFlag() {
        return flag;
    }
    
    // 注意：volatile不保证原子性
    public void increment() {
        count++;  // 这仍然不是线程安全的！
    }
}
```

**3. Lock接口及其实现类**
```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class LockCounter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
    
    // 读写锁示例
    public int getCountWithReadLock() {
        rwLock.readLock().lock();
        try {
            return count;
        } finally {
            rwLock.readLock().unlock();
        }
    }
    
    public void incrementWithWriteLock() {
        rwLock.writeLock().lock();
        try {
            count++;
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```

**4. 原子类（AtomicXXX）**
```java
import java.util.concurrent.atomic.*;

public class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    private final AtomicLong longCount = new AtomicLong(0L);
    private final AtomicReference<String> reference = new AtomicReference<>("initial");
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public void add(int delta) {
        count.addAndGet(delta);
    }
    
    public boolean compareAndSet(int expect, int update) {
        return count.compareAndSet(expect, update);
    }
    
    public int getCount() {
        return count.get();
    }
    
    // 原子引用示例
    public void updateReference(String newValue) {
        reference.set(newValue);
    }
    
    public String getReference() {
        return reference.get();
    }
}
```

**5. ThreadLocal（线程本地变量）**
```java
public class ThreadLocalExample {
    private static final ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };
    
    // Java 8+ 简化写法
    private static final ThreadLocal<Integer> threadLocal2 = ThreadLocal.withInitial(() -> 0);
    
    public void increment() {
        Integer value = threadLocal.get();
        threadLocal.set(value + 1);
    }
    
    public int get() {
        return threadLocal.get();
    }
    
    public void remove() {
        threadLocal.remove();  // 防止内存泄漏
    }
    
    public static void main(String[] args) throws InterruptedException {
        ThreadLocalExample example = new ThreadLocalExample();
        
        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            final int threadId = i;
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 3; j++) {
                    example.increment();
                    System.out.println("线程" + threadId + "的值：" + example.get());
                }
                example.remove();
            });
            threads[i].start();
        }
        
        for (Thread thread : threads) {
            thread.join();
        }
    }
}
```

**6. 不可变对象**
```java
public final class ImmutablePerson {
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    public ImmutablePerson(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // 防御性拷贝
        this.hobbies = Collections.unmodifiableList(new ArrayList<>(hobbies));
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public List<String> getHobbies() {
        return hobbies;  // 已经是不可变的
    }
    
    // 修改操作返回新对象
    public ImmutablePerson withAge(int newAge) {
        return new ImmutablePerson(this.name, newAge, new ArrayList<>(this.hobbies));
    }
}
```

**7. 并发集合类**
```java
import java.util.concurrent.*;

public class ConcurrentCollectionExample {
    // 线程安全的Map
    private final ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
    
    // 线程安全的队列
    private final BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>();
    
    // 线程安全的List（读多写少场景）
    private final CopyOnWriteArrayList<String> copyOnWriteList = new CopyOnWriteArrayList<>();
    
    public void demonstrateConcurrentCollections() {
        // ConcurrentHashMap操作
        concurrentMap.put("key1", 1);
        concurrentMap.computeIfAbsent("key2", k -> 2);
        concurrentMap.merge("key1", 1, Integer::sum);
        
        // BlockingQueue操作
        try {
            blockingQueue.put("item1");  // 阻塞式添加
            String item = blockingQueue.take();  // 阻塞式获取
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // CopyOnWriteArrayList操作
        copyOnWriteList.add("item1");
        copyOnWriteList.add("item2");
    }
}
```

**线程安全级别：**
1. **不可变（Immutable）**：最高级别，如String、Integer等
2. **绝对线程安全**：在任何情况下都不需要额外同步
3. **相对线程安全**：通常意义上的线程安全，如Vector、HashTable
4. **线程兼容**：对象本身不是线程安全的，但可以通过同步手段使其安全
5. **线程对立**：无论是否采取同步措施，都无法在多线程环境中安全使用

**选择建议：**
- **性能要求高**：优先考虑原子类、ConcurrentHashMap等
- **复杂同步逻辑**：使用Lock接口
- **简单同步**：使用synchronized
- **状态标志**：使用volatile
- **线程隔离**：使用ThreadLocal
- **不变性**：设计不可变对象
  
  **2. synchronized和Lock的区别？**

**答案：**

**synchronized关键字详解：**
```java
public class SynchronizedExample {
    private int count = 0;
    private final Object lock = new Object();
    
    // 1. 修饰实例方法（锁的是this对象）
    public synchronized void incrementMethod() {
        count++;
        System.out.println("同步方法：" + Thread.currentThread().getName() + ", count=" + count);
    }
    
    // 2. 修饰静态方法（锁的是Class对象）
    public static synchronized void staticMethod() {
        System.out.println("静态同步方法：" + Thread.currentThread().getName());
    }
    
    // 3. 修饰代码块（this锁）
    public void incrementBlock1() {
        synchronized (this) {
            count++;
            System.out.println("同步代码块(this)：" + Thread.currentThread().getName() + ", count=" + count);
        }
    }
    
    // 4. 修饰代码块（自定义锁对象）
    public void incrementBlock2() {
        synchronized (lock) {
            count++;
            System.out.println("同步代码块(自定义锁)：" + Thread.currentThread().getName() + ", count=" + count);
        }
    }
    
    // 5. 修饰代码块（类锁）
    public void incrementBlock3() {
        synchronized (SynchronizedExample.class) {
            count++;
            System.out.println("同步代码块(类锁)：" + Thread.currentThread().getName() + ", count=" + count);
        }
    }
    
    // wait/notify机制
    private boolean ready = false;
    
    public synchronized void waitForReady() throws InterruptedException {
        while (!ready) {
            System.out.println(Thread.currentThread().getName() + " 等待中...");
            wait();  // 释放锁并等待
        }
        System.out.println(Thread.currentThread().getName() + " 被唤醒，继续执行");
    }
    
    public synchronized void setReady() {
        ready = true;
        System.out.println(Thread.currentThread().getName() + " 设置ready=true");
        notifyAll();  // 唤醒所有等待的线程
    }
}
```

**Lock接口详解：**
```java
import java.util.concurrent.locks.*;
import java.util.concurrent.TimeUnit;

public class LockExample {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();
    private final ReentrantLock fairLock = new ReentrantLock(true);  // 公平锁
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    
    // 1. 基本的Lock使用
    public void increment() {
        lock.lock();
        try {
            count++;
            System.out.println("Lock基本使用：" + Thread.currentThread().getName() + ", count=" + count);
        } finally {
            lock.unlock();  // 必须在finally中释放锁
        }
    }
    
    // 2. 尝试获取锁（非阻塞）
    public boolean tryIncrement() {
        if (lock.tryLock()) {
            try {
                count++;
                System.out.println("tryLock成功：" + Thread.currentThread().getName() + ", count=" + count);
                return true;
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println("tryLock失败：" + Thread.currentThread().getName());
            return false;
        }
    }
    
    // 3. 超时获取锁
    public boolean incrementWithTimeout() {
        try {
            if (lock.tryLock(2, TimeUnit.SECONDS)) {
                try {
                    count++;
                    Thread.sleep(1000);  // 模拟耗时操作
                    System.out.println("超时锁获取成功：" + Thread.currentThread().getName() + ", count=" + count);
                    return true;
                } finally {
                    lock.unlock();
                }
            } else {
                System.out.println("超时锁获取失败：" + Thread.currentThread().getName());
                return false;
            }
        } catch (InterruptedException e) {
            System.out.println("获取锁被中断：" + Thread.currentThread().getName());
            Thread.currentThread().interrupt();
            return false;
        }
    }
    
    // 4. 可中断的锁获取
    public void incrementInterruptibly() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            count++;
            Thread.sleep(2000);  // 模拟长时间操作
            System.out.println("可中断锁：" + Thread.currentThread().getName() + ", count=" + count);
        } finally {
            lock.unlock();
        }
    }
    
    // 5. 读写锁示例
    public int read() {
        readLock.lock();
        try {
            System.out.println("读操作：" + Thread.currentThread().getName() + ", count=" + count);
            Thread.sleep(1000);  // 模拟读操作耗时
            return count;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return -1;
        } finally {
            readLock.unlock();
        }
    }
    
    public void write(int value) {
        writeLock.lock();
        try {
            count = value;
            System.out.println("写操作：" + Thread.currentThread().getName() + ", count=" + count);
            Thread.sleep(1000);  // 模拟写操作耗时
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            writeLock.unlock();
        }
    }
    
    // 6. 条件变量示例（生产者-消费者模式）
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
    private final Object[] buffer = new Object[10];
    private int putIndex = 0, takeIndex = 0, size = 0;
    
    public void put(Object item) throws InterruptedException {
        lock.lock();
        try {
            while (size == buffer.length) {
                System.out.println("缓冲区满，生产者等待：" + Thread.currentThread().getName());
                notFull.await();  // 等待不满条件
            }
            buffer[putIndex] = item;
            putIndex = (putIndex + 1) % buffer.length;
            size++;
            System.out.println("生产者放入：" + item + ", 当前大小：" + size);
            notEmpty.signal();  // 通知不空条件
        } finally {
            lock.unlock();
        }
    }
    
    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (size == 0) {
                System.out.println("缓冲区空，消费者等待：" + Thread.currentThread().getName());
                notEmpty.await();  // 等待不空条件
            }
            Object item = buffer[takeIndex];
            buffer[takeIndex] = null;
            takeIndex = (takeIndex + 1) % buffer.length;
            size--;
            System.out.println("消费者取出：" + item + ", 当前大小：" + size);
            notFull.signal();  // 通知不满条件
            return item;
        } finally {
            lock.unlock();
        }
    }
    
    // 7. 公平锁vs非公平锁演示
    public void demonstrateFairness() {
        // 非公平锁（默认）
        ReentrantLock unfairLock = new ReentrantLock(false);
        // 公平锁
        ReentrantLock fairLock = new ReentrantLock(true);
        
        System.out.println("非公平锁：" + !unfairLock.isFair());
        System.out.println("公平锁：" + fairLock.isFair());
    }
}
```

**详细对比表：**
| 特性 | synchronized | Lock |
|------|--------------|------|
| **本质** | Java关键字，JVM内置 | 接口，JDK提供的API |
| **锁的获取** | 自动获取 | 手动调用lock() |
| **锁的释放** | 自动释放（JVM保证） | 手动调用unlock()（必须在finally中） |
| **中断响应** | 不支持，无法中断等待锁的线程 | 支持（lockInterruptibly()） |
| **超时获取** | 不支持 | 支持（tryLock(time, unit)） |
| **非阻塞获取** | 不支持 | 支持（tryLock()） |
| **公平性** | 非公平锁（无法选择） | 可选择公平锁或非公平锁 |
| **条件变量** | 单一条件（wait/notify/notifyAll） | 多个条件（Condition） |
| **重入性** | 支持重入 | 支持重入（ReentrantLock） |
| **性能** | JDK1.6后优化，轻量级锁、偏向锁等 | 功能丰富，但开销稍大 |
| **使用复杂度** | 简单，语法层面支持 | 相对复杂，需要手动管理 |
| **死锁检测** | 无内置机制 | 可以通过tryLock避免死锁 |
| **锁状态查询** | 无法查询锁状态 | 可以查询锁状态（isLocked()等） |
| **读写分离** | 不支持 | 支持（ReadWriteLock） |

**性能对比测试：**
```java
public class PerformanceComparison {
    private int count = 0;
    private final Object syncLock = new Object();
    private final ReentrantLock reentrantLock = new ReentrantLock();
    
    // synchronized性能测试
    public void synchronizedIncrement() {
        synchronized (syncLock) {
            count++;
        }
    }
    
    // ReentrantLock性能测试
    public void lockIncrement() {
        reentrantLock.lock();
        try {
            count++;
        } finally {
            reentrantLock.unlock();
        }
    }
    
    public static void performanceTest() {
        PerformanceComparison test = new PerformanceComparison();
        int threadCount = 10;
        int iterations = 1000000;
        
        // 测试synchronized
        long startTime = System.currentTimeMillis();
        Thread[] syncThreads = new Thread[threadCount];
        for (int i = 0; i < threadCount; i++) {
            syncThreads[i] = new Thread(() -> {
                for (int j = 0; j < iterations; j++) {
                    test.synchronizedIncrement();
                }
            });
            syncThreads[i].start();
        }
        
        for (Thread thread : syncThreads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        long syncTime = System.currentTimeMillis() - startTime;
        
        // 重置计数器
        test.count = 0;
        
        // 测试ReentrantLock
        startTime = System.currentTimeMillis();
        Thread[] lockThreads = new Thread[threadCount];
        for (int i = 0; i < threadCount; i++) {
            lockThreads[i] = new Thread(() -> {
                for (int j = 0; j < iterations; j++) {
                    test.lockIncrement();
                }
            });
            lockThreads[i].start();
        }
        
        for (Thread thread : lockThreads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        long lockTime = System.currentTimeMillis() - startTime;
        
        System.out.println("synchronized耗时：" + syncTime + "ms");
        System.out.println("ReentrantLock耗时：" + lockTime + "ms");
    }
}
```

**使用场景建议：**

**选择synchronized的情况：**
- 简单的同步需求
- 代码简洁性要求高
- 不需要高级功能（超时、中断等）
- JVM自动优化（偏向锁、轻量级锁等）

**选择Lock的情况：**
- 需要尝试获取锁（tryLock）
- 需要超时获取锁
- 需要可中断的锁获取
- 需要公平锁
- 需要多个条件变量
- 需要读写分离（ReadWriteLock）
- 复杂的同步场景
  
  **3. volatile关键字的作用？**

**答案：**

**volatile的核心作用：**

**1. 保证可见性：**
```java
public class VisibilityExample {
    private volatile boolean flag = false;
    private int count = 0;
    
    // 线程1执行
    public void writer() {
        count = 42;        // 1
        flag = true;       // 2 - volatile写
    }
    
    // 线程2执行
    public void reader() {
        if (flag) {        // 3 - volatile读
            System.out.println(count);  // 4 - 保证能看到42
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        VisibilityExample example = new VisibilityExample();
        
        Thread writerThread = new Thread(() -> {
            try {
                Thread.sleep(1000);
                example.writer();
                System.out.println("Writer完成");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        Thread readerThread = new Thread(() -> {
            while (!example.flag) {
                // 忙等待，如果flag不是volatile，可能永远看不到true
            }
            example.reader();
        });
        
        readerThread.start();
        writerThread.start();
        
        writerThread.join();
        readerThread.join();
    }
}
```

**2. 禁止指令重排序：**
```java
public class ReorderingExample {
    private int a = 0;
    private volatile boolean flag = false;
    
    // 线程1
    public void method1() {
        a = 1;           // 1
        flag = true;     // 2 - volatile写，禁止1和2重排序
    }
    
    // 线程2
    public void method2() {
        if (flag) {      // 3 - volatile读
            int i = a;   // 4 - 保证能看到a=1
            System.out.println("a的值：" + i);
        }
    }
    
    // 演示没有volatile的情况
    private int x = 0;
    private boolean normalFlag = false;
    
    public void unsafeMethod1() {
        x = 1;              // 可能与下一行重排序
        normalFlag = true;  // 可能与上一行重排序
    }
    
    public void unsafeMethod2() {
        if (normalFlag) {
            int i = x;  // 可能看到x=0
            System.out.println("x的值：" + i);
        }
    }
}
```

**3. 不保证原子性：**
```java
public class AtomicityExample {
    private volatile int count = 0;
    
    // 这个方法不是线程安全的！
    public void increment() {
        count++;  // 这是三个操作：读取、加1、写入
    }
    
    // 演示volatile不保证原子性
    public static void main(String[] args) throws InterruptedException {
        AtomicityExample example = new AtomicityExample();
        
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    example.increment();
                }
            });
            threads[i].start();
        }
        
        for (Thread thread : threads) {
            thread.join();
        }
        
        // 期望结果是10000，但实际结果通常小于这个值
        System.out.println("最终计数：" + example.count);
    }
    
    // 正确的做法
    private final Object lock = new Object();
    
    public void safeIncrement() {
        synchronized (lock) {
            count++;
        }
    }
    
    // 或者使用原子类
    private final AtomicInteger atomicCount = new AtomicInteger(0);
    
    public void atomicIncrement() {
        atomicCount.incrementAndGet();
    }
}
```

**volatile的内存语义：**
```java
public class VolatileMemorySemantics {
    private int a = 0;
    private int b = 0;
    private volatile int v = 0;
    private int c = 0;
    private int d = 0;
    
    public void writer() {
        a = 1;    // 1
        b = 2;    // 2
        v = 3;    // 3 - volatile写
        c = 4;    // 4
        d = 5;    // 5
        
        // 内存屏障效果：
        // 1、2不能重排序到3之后
        // 4、5不能重排序到3之前
        // 但1、2之间可以重排序，4、5之间也可以重排序
    }
    
    public void reader() {
        int i = v;  // volatile读
        int j = a;  // 能看到最新的a值
        int k = b;  // 能看到最新的b值
        // 但不能保证看到c、d的最新值
        
        System.out.println("v=" + i + ", a=" + j + ", b=" + k);
    }
}
```

**使用场景详解：**

**1. 状态标志：**
```java
public class StatusFlag {
    private volatile boolean shutdown = false;
    private volatile boolean ready = false;
    
    public void shutdown() {
        shutdown = true;
        System.out.println("设置shutdown标志");
    }
    
    public void setReady() {
        ready = true;
        System.out.println("设置ready标志");
    }
    
    public void doWork() {
        while (!shutdown) {
            if (ready) {
                // 执行工作
                System.out.println("Working...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            } else {
                // 等待ready
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
        System.out.println("工作线程退出");
    }
}
```

**2. 双重检查锁定（DCL）：**
```java
public class Singleton {
    private volatile static Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {                    // 第一次检查
            synchronized (Singleton.class) {
                if (instance == null) {            // 第二次检查
                    instance = new Singleton();   // volatile确保正确发布
                }
            }
        }
        return instance;
    }
    
    // 错误的实现（没有volatile）
    private static Singleton unsafeInstance;
    
    public static Singleton getUnsafeInstance() {
        if (unsafeInstance == null) {
            synchronized (Singleton.class) {
                if (unsafeInstance == null) {
                    // 可能发生指令重排序，导致返回未完全初始化的对象
                    unsafeInstance = new Singleton();
                }
            }
        }
        return unsafeInstance;
    }
}
```

**3. 独立观察：**
```java
public class VolatileObserver {
    private volatile long lastUpdateTime = System.currentTimeMillis();
    private volatile int temperature = 20;
    private volatile String status = "NORMAL";
    
    // 更新方法
    public void updateTemperature(int newTemp) {
        temperature = newTemp;
        lastUpdateTime = System.currentTimeMillis();
        
        if (newTemp > 35) {
            status = "HIGH";
        } else if (newTemp < 10) {
            status = "LOW";
        } else {
            status = "NORMAL";
        }
    }
    
    // 读取方法
    public boolean isStale(long maxAge) {
        return (System.currentTimeMillis() - lastUpdateTime) > maxAge;
    }
    
    public void printStatus() {
        System.out.println("温度：" + temperature + "°C, 状态：" + status + 
                          ", 更新时间：" + lastUpdateTime);
    }
}
```

**4. 一次性安全发布：**
```java
public class SafePublication {
    private volatile Resource resource;
    
    public void initializeResource() {
        Resource temp = new Resource();
        temp.initialize();  // 完全初始化
        resource = temp;    // 一次性安全发布
    }
    
    public Resource getResource() {
        return resource;  // 可能返回null，但不会返回部分初始化的对象
    }
    
    private static class Resource {
        private String data;
        private boolean initialized = false;
        
        public void initialize() {
            data = "初始化数据";
            initialized = true;
        }
        
        public String getData() {
            return data;
        }
        
        public boolean isInitialized() {
            return initialized;
        }
    }
}
```

**volatile vs synchronized vs AtomicXXX：**
| 特性 | volatile | synchronized | AtomicXXX |
|------|----------|--------------|----------|
| **原子性** | 不保证 | 保证 | 保证 |
| **可见性** | 保证 | 保证 | 保证 |
| **有序性** | 部分保证（禁止重排序） | 保证 | 保证 |
| **性能** | 最高（几乎无开销） | 较低（可能阻塞） | 较高（CAS操作） |
| **阻塞性** | 非阻塞 | 可能阻塞 | 非阻塞 |
| **适用场景** | 状态标志、一次性发布 | 复合操作、临界区 | 简单原子操作 |
| **内存开销** | 无额外开销 | 对象头开销 | 对象开销 |

**使用建议：**
- **简单状态标志**：使用volatile
- **需要原子性**：使用synchronized或AtomicXXX
- **高并发计数器**：使用AtomicXXX
- **复杂业务逻辑**：使用synchronized
- **一次性发布**：使用volatile
- **读多写少**：考虑volatile（如果不需要原子性）

**注意事项：**
1. volatile不能修饰局部变量
2. volatile不能保证复合操作的原子性
3. volatile变量的运算在并发下不是安全的
4. volatile适用于一个线程写，多个线程读的场景
  
  **4. 什么是死锁？如何避免？**
  
  **答案：**
  
  **死锁定义：**
  死锁是指两个或多个线程互相等待对方释放资源，导致程序无法继续执行的状态。
  
  **死锁示例：**
  ```java
  public class DeadlockExample {
      private static final Object lock1 = new Object();
      private static final Object lock2 = new Object();
      
      public static void main(String[] args) {
          Thread thread1 = new Thread(() -> {
              synchronized (lock1) {
                  System.out.println("线程1获得lock1");
                  try {
                      Thread.sleep(100);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  
                  System.out.println("线程1等待lock2...");
                  synchronized (lock2) {
                      System.out.println("线程1获得lock2");
                  }
              }
          }, "Thread-1");
          
          Thread thread2 = new Thread(() -> {
              synchronized (lock2) {
                  System.out.println("线程2获得lock2");
                  try {
                      Thread.sleep(100);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  
                  System.out.println("线程2等待lock1...");
                  synchronized (lock1) {
                      System.out.println("线程2获得lock1");
                  }
              }
          }, "Thread-2");
          
          thread1.start();
          thread2.start();
          
          // 这两个线程会发生死锁
      }
  }
  ```
  
  **死锁的四个必要条件（Coffman条件）：**
  
  **1. 互斥条件（Mutual Exclusion）：**
  资源不能被多个线程同时使用，同一时间只能有一个线程访问资源。
  
  **2. 请求和保持条件（Hold and Wait）：**
  线程已经获得了至少一个资源，同时又请求其他被别的线程占用的资源。
  
  **3. 不可剥夺条件（No Preemption）：**
  已经分配给线程的资源不能被强制性地剥夺，只能由持有资源的线程主动释放。
  
  **4. 循环等待条件（Circular Wait）：**
  存在一个线程等待链，每个线程都在等待下一个线程所持有的资源。
  
  **避免死锁的方法：**
  
  **1. 按固定顺序获取锁（锁排序）：**
  ```java
  public class LockOrdering {
      private static final Object lock1 = new Object();
      private static final Object lock2 = new Object();
      
      // 为锁分配唯一ID
      private static final int LOCK1_ID = System.identityHashCode(lock1);
      private static final int LOCK2_ID = System.identityHashCode(lock2);
      
      public void transferMoney(Account from, Account to, int amount) {
          // 按照账户ID排序来获取锁
          Account firstLock = from.getId() < to.getId() ? from : to;
          Account secondLock = from.getId() < to.getId() ? to : from;
          
          synchronized (firstLock) {
              synchronized (secondLock) {
                  from.debit(amount);
                  to.credit(amount);
              }
          }
      }
      
      // 通用的锁排序方法
      public void orderedLock() {
          Object firstLock = LOCK1_ID < LOCK2_ID ? lock1 : lock2;
          Object secondLock = LOCK1_ID < LOCK2_ID ? lock2 : lock1;
          
          synchronized (firstLock) {
              synchronized (secondLock) {
                  // 业务逻辑
                  System.out.println("按顺序获取锁");
              }
          }
      }
  }
  ```
  
  **2. 使用超时机制：**
  ```java
  import java.util.concurrent.TimeUnit;
  import java.util.concurrent.locks.Lock;
  import java.util.concurrent.locks.ReentrantLock;
  
  public class TimeoutLocking {
      private final Lock lock1 = new ReentrantLock();
      private final Lock lock2 = new ReentrantLock();
      
      public boolean transferWithTimeout(Account from, Account to, int amount) {
          long timeout = 1000; // 1秒超时
          
          try {
              // 尝试获取第一个锁
              if (lock1.tryLock(timeout, TimeUnit.MILLISECONDS)) {
                  try {
                      // 尝试获取第二个锁
                      if (lock2.tryLock(timeout, TimeUnit.MILLISECONDS)) {
                          try {
                              // 执行转账操作
                              from.debit(amount);
                              to.credit(amount);
                              return true;
                          } finally {
                              lock2.unlock();
                          }
                      } else {
                          System.out.println("获取lock2超时");
                          return false;
                      }
                  } finally {
                      lock1.unlock();
                  }
              } else {
                  System.out.println("获取lock1超时");
                  return false;
              }
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
              return false;
          }
      }
  }
  ```
  
  **3. 死锁检测和恢复：**
  ```java
  import java.lang.management.ManagementFactory;
  import java.lang.management.ThreadInfo;
  import java.lang.management.ThreadMXBean;
  
  public class DeadlockDetector {
      private final ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
      
      public void detectDeadlock() {
          long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();
          
          if (deadlockedThreads != null) {
              ThreadInfo[] threadInfos = threadMXBean.getThreadInfo(deadlockedThreads);
              
              System.out.println("检测到死锁！");
              for (ThreadInfo threadInfo : threadInfos) {
                  System.out.println("线程名称: " + threadInfo.getThreadName());
                  System.out.println("线程状态: " + threadInfo.getThreadState());
                  System.out.println("阻塞信息: " + threadInfo.getLockInfo());
                  System.out.println("等待的锁: " + threadInfo.getLockName());
                  System.out.println("锁的拥有者: " + threadInfo.getLockOwnerName());
                  System.out.println("---");
              }
              
              // 可以选择中断某些线程来打破死锁
              handleDeadlock(deadlockedThreads);
          }
      }
      
      private void handleDeadlock(long[] deadlockedThreads) {
          System.out.println("处理死锁：中断线程 " + deadlockedThreads[0]);
          
          // 注意：这只是示例，实际中断线程需要谨慎
          for (Thread thread : Thread.getAllStackTraces().keySet()) {
              if (thread.getId() == deadlockedThreads[0]) {
                  thread.interrupt();
                  break;
              }
          }
      }
  }
  ```
  
  **4. 避免嵌套锁：**
  ```java
  public class AvoidNestedLocks {
      private final Object lock1 = new Object();
      private final Object lock2 = new Object();
      
      // 不好的做法：嵌套锁
      public void badMethod() {
          synchronized (lock1) {
              synchronized (lock2) {
                  // 业务逻辑
              }
          }
      }
      
      // 好的做法：避免嵌套
      public void goodMethod() {
          // 方法1：分别处理
          doWithLock1();
          doWithLock2();
      }
      
      private void doWithLock1() {
          synchronized (lock1) {
              // 只使用lock1的逻辑
          }
      }
      
      private void doWithLock2() {
          synchronized (lock2) {
              // 只使用lock2的逻辑
          }
      }
  }
  ```
  
  **死锁预防策略总结：**
  
  | 策略 | 破坏条件 | 实现方法 | 优缺点 |
  |------|----------|----------|--------|
  | **一次性分配** | 请求和保持 | 线程启动时获取所有需要的资源 | 简单但资源利用率低 |
  | **资源排序** | 循环等待 | 按固定顺序获取资源 | 有效但需要全局协调 |
  | **超时机制** | 不可剥夺 | 设置锁获取超时时间 | 实用但可能影响性能 |
  | **银行家算法** | 循环等待 | 动态检查资源分配安全性 | 理论完美但实现复杂 |
  | **避免嵌套锁** | 请求和保持 | 重构代码避免同时持有多个锁 | 最佳实践 |
  
  **最佳实践建议：**
  1. **设计阶段**：避免需要多个锁的设计
  2. **编码阶段**：使用锁排序和超时机制
  3. **测试阶段**：进行死锁压力测试
  4. **运行阶段**：监控死锁并及时处理
  5. **优先使用**：高级并发工具类而非底层锁
  
  **5. 线程池的核心参数和工作原理？**
  
  **答案：**
  
  **线程池核心参数详解：**
  
  ```java
  public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler)
  ```
  
  **1. corePoolSize（核心线程数）：**
  ```java
  public class CorePoolSizeExample {
      public static void main(String[] args) {
          // 核心线程数为2
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              2,  // corePoolSize
              4,  // maximumPoolSize
              60L, TimeUnit.SECONDS,
              new LinkedBlockingQueue<>(2),
              new ThreadPoolExecutor.CallerRunsPolicy()
          );
          
          // 提交5个任务
          for (int i = 1; i <= 5; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("任务" + taskId + " 在线程 " + 
                      Thread.currentThread().getName() + " 中执行");
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
              
              System.out.println("提交任务" + taskId + 
                  ", 当前线程数: " + executor.getPoolSize() +
                  ", 队列大小: " + executor.getQueue().size());
          }
          
          executor.shutdown();
      }
  }
  ```
  
  **2. maximumPoolSize（最大线程数）：**
  ```java
  public class MaximumPoolSizeExample {
      public static void main(String[] args) {
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1,  // 核心线程数
              3,  // 最大线程数
              60L, TimeUnit.SECONDS,
              new ArrayBlockingQueue<>(1), // 队列容量为1
              new ThreadPoolExecutor.AbortPolicy()
          );
          
          // 提交5个任务测试最大线程数
          for (int i = 1; i <= 5; i++) {
              final int taskId = i;
              try {
                  executor.submit(() -> {
                      System.out.println("任务" + taskId + " 开始执行，线程: " + 
                          Thread.currentThread().getName());
                      try {
                          Thread.sleep(3000);
                      } catch (InterruptedException e) {
                          Thread.currentThread().interrupt();
                      }
                      System.out.println("任务" + taskId + " 执行完成");
                  });
                  
                  System.out.println("任务" + taskId + " 提交成功，当前线程数: " + 
                      executor.getPoolSize());
              } catch (RejectedExecutionException e) {
                  System.out.println("任务" + taskId + " 被拒绝: " + e.getMessage());
              }
              
              try {
                  Thread.sleep(500); // 稍微延迟，观察线程池状态变化
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
              }
          }
          
          executor.shutdown();
      }
  }
  ```
  
  **3. keepAliveTime（空闲线程存活时间）：**
  ```java
  public class KeepAliveTimeExample {
      public static void main(String[] args) throws InterruptedException {
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1,  // 核心线程数
              3,  // 最大线程数
              2L, TimeUnit.SECONDS, // 空闲线程2秒后回收
              new LinkedBlockingQueue<>(),
              new ThreadFactory() {
                  private final AtomicInteger threadNumber = new AtomicInteger(1);
                  
                  @Override
                  public Thread newThread(Runnable r) {
                      Thread t = new Thread(r, "CustomThread-" + threadNumber.getAndIncrement());
                      System.out.println("创建新线程: " + t.getName());
                      return t;
                  }
              }
          );
          
          // 允许核心线程也被回收
          executor.allowCoreThreadTimeOut(true);
          
          // 提交3个任务，创建3个线程
          for (int i = 1; i <= 3; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("任务" + taskId + " 在 " + 
                      Thread.currentThread().getName() + " 中执行");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
          }
          
          // 监控线程池大小变化
          for (int i = 0; i < 10; i++) {
              System.out.println("第" + i + "秒，线程池大小: " + executor.getPoolSize());
              Thread.sleep(1000);
          }
          
          executor.shutdown();
      }
  }
  ```
  
  **4. workQueue（工作队列）：**
  ```java
  public class WorkQueueExample {
      public static void main(String[] args) {
          // 1. ArrayBlockingQueue - 有界队列
          testArrayBlockingQueue();
          
          // 2. LinkedBlockingQueue - 无界队列
          testLinkedBlockingQueue();
          
          // 3. SynchronousQueue - 直接交换队列
          testSynchronousQueue();
          
          // 4. PriorityBlockingQueue - 优先级队列
          testPriorityBlockingQueue();
      }
      
      private static void testArrayBlockingQueue() {
          System.out.println("=== ArrayBlockingQueue 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 2, 60L, TimeUnit.SECONDS,
              new ArrayBlockingQueue<>(2), // 容量为2的有界队列
              new ThreadPoolExecutor.CallerRunsPolicy()
          );
          
          for (int i = 1; i <= 5; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("ArrayBlockingQueue任务" + taskId + " 执行");
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
          }
          
          executor.shutdown();
      }
      
      private static void testLinkedBlockingQueue() {
          System.out.println("=== LinkedBlockingQueue 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 2, 60L, TimeUnit.SECONDS,
              new LinkedBlockingQueue<>(), // 无界队列
              new ThreadPoolExecutor.AbortPolicy()
          );
          
          for (int i = 1; i <= 10; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("LinkedBlockingQueue任务" + taskId + " 执行");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
          }
          
          executor.shutdown();
      }
      
      private static void testSynchronousQueue() {
          System.out.println("=== SynchronousQueue 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              0, 3, 60L, TimeUnit.SECONDS,
              new SynchronousQueue<>(), // 直接交换队列
              new ThreadPoolExecutor.CallerRunsPolicy()
          );
          
          for (int i = 1; i <= 5; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("SynchronousQueue任务" + taskId + " 执行");
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
          }
          
          executor.shutdown();
      }
      
      private static void testPriorityBlockingQueue() {
          System.out.println("=== PriorityBlockingQueue 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 2, 60L, TimeUnit.SECONDS,
              new PriorityBlockingQueue<>(),
              new ThreadPoolExecutor.AbortPolicy()
          );
          
          // 提交不同优先级的任务
          for (int i = 5; i >= 1; i--) {
              final int priority = i;
              executor.submit(new PriorityTask(priority, () -> {
                  System.out.println("优先级" + priority + "任务执行");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              }));
          }
          
          executor.shutdown();
      }
      
      static class PriorityTask implements Runnable, Comparable<PriorityTask> {
          private final int priority;
          private final Runnable task;
          
          public PriorityTask(int priority, Runnable task) {
              this.priority = priority;
              this.task = task;
          }
          
          @Override
          public void run() {
              task.run();
          }
          
          @Override
          public int compareTo(PriorityTask other) {
              return Integer.compare(other.priority, this.priority); // 高优先级先执行
          }
      }
  }
  ```
  
  **5. threadFactory（线程工厂）：**
  ```java
  public class ThreadFactoryExample {
      public static void main(String[] args) {
          // 自定义线程工厂
          ThreadFactory customThreadFactory = new ThreadFactory() {
              private final AtomicInteger threadNumber = new AtomicInteger(1);
              private final String namePrefix = "CustomPool-";
              
              @Override
              public Thread newThread(Runnable r) {
                  Thread t = new Thread(r, namePrefix + threadNumber.getAndIncrement());
                  
                  // 设置为守护线程
                  t.setDaemon(false);
                  
                  // 设置优先级
                  t.setPriority(Thread.NORM_PRIORITY);
                  
                  // 设置异常处理器
                  t.setUncaughtExceptionHandler((thread, ex) -> {
                      System.err.println("线程 " + thread.getName() + " 发生异常: " + ex.getMessage());
                      ex.printStackTrace();
                  });
                  
                  System.out.println("创建线程: " + t.getName());
                  return t;
              }
          };
          
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              2, 4, 60L, TimeUnit.SECONDS,
              new LinkedBlockingQueue<>(),
              customThreadFactory,
              new ThreadPoolExecutor.AbortPolicy()
          );
          
          // 提交正常任务
          executor.submit(() -> {
              System.out.println("正常任务在 " + Thread.currentThread().getName() + " 中执行");
          });
          
          // 提交会抛异常的任务
          executor.submit(() -> {
              System.out.println("异常任务在 " + Thread.currentThread().getName() + " 中执行");
              throw new RuntimeException("模拟异常");
          });
          
          executor.shutdown();
      }
  }
  ```
  
  **6. handler（拒绝策略）：**
  ```java
  public class RejectedExecutionHandlerExample {
      public static void main(String[] args) {
          // 测试不同的拒绝策略
          testAbortPolicy();
          testCallerRunsPolicy();
          testDiscardPolicy();
          testDiscardOldestPolicy();
          testCustomPolicy();
      }
      
      // 1. AbortPolicy - 抛出异常
      private static void testAbortPolicy() {
          System.out.println("=== AbortPolicy 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 1, 0L, TimeUnit.MILLISECONDS,
              new ArrayBlockingQueue<>(1),
              new ThreadPoolExecutor.AbortPolicy()
          );
          
          try {
              for (int i = 1; i <= 3; i++) {
                  final int taskId = i;
                  executor.submit(() -> {
                      System.out.println("AbortPolicy任务" + taskId + " 执行");
                      try {
                          Thread.sleep(2000);
                      } catch (InterruptedException e) {
                          Thread.currentThread().interrupt();
                      }
                  });
              }
          } catch (RejectedExecutionException e) {
              System.out.println("任务被拒绝: " + e.getMessage());
          }
          
          executor.shutdown();
      }
      
      // 2. CallerRunsPolicy - 调用者执行
      private static void testCallerRunsPolicy() {
          System.out.println("=== CallerRunsPolicy 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 1, 0L, TimeUnit.MILLISECONDS,
              new ArrayBlockingQueue<>(1),
              new ThreadPoolExecutor.CallerRunsPolicy()
          );
          
          for (int i = 1; i <= 3; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("CallerRunsPolicy任务" + taskId + 
                      " 在线程 " + Thread.currentThread().getName() + " 中执行");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
          }
          
          executor.shutdown();
      }
      
      // 3. DiscardPolicy - 静默丢弃
      private static void testDiscardPolicy() {
          System.out.println("=== DiscardPolicy 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 1, 0L, TimeUnit.MILLISECONDS,
              new ArrayBlockingQueue<>(1),
              new ThreadPoolExecutor.DiscardPolicy()
          );
          
          for (int i = 1; i <= 3; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("DiscardPolicy任务" + taskId + " 执行");
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
              System.out.println("提交任务" + taskId);
          }
          
          executor.shutdown();
      }
      
      // 4. DiscardOldestPolicy - 丢弃最老的任务
      private static void testDiscardOldestPolicy() {
          System.out.println("=== DiscardOldestPolicy 测试 ===");
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 1, 0L, TimeUnit.MILLISECONDS,
              new ArrayBlockingQueue<>(1),
              new ThreadPoolExecutor.DiscardOldestPolicy()
          );
          
          for (int i = 1; i <= 4; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("DiscardOldestPolicy任务" + taskId + " 执行");
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
              System.out.println("提交任务" + taskId);
          }
          
          executor.shutdown();
      }
      
      // 5. 自定义拒绝策略
      private static void testCustomPolicy() {
          System.out.println("=== 自定义拒绝策略测试 ===");
          
          RejectedExecutionHandler customHandler = new RejectedExecutionHandler() {
              @Override
              public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                  System.out.println("自定义处理被拒绝的任务: " + r.toString());
                  
                  // 可以选择:
                  // 1. 记录日志
                  // 2. 存储到数据库或消息队列
                  // 3. 降级处理
                  // 4. 重试机制
                  
                  // 这里演示简单的重试
                  try {
                      Thread.sleep(100);
                      if (!executor.isShutdown()) {
                          executor.getQueue().offer(r);
                          System.out.println("任务重新入队成功");
                      }
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              }
          };
          
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              1, 1, 0L, TimeUnit.MILLISECONDS,
              new ArrayBlockingQueue<>(1),
              customHandler
          );
          
          for (int i = 1; i <= 3; i++) {
              final int taskId = i;
              executor.submit(() -> {
                  System.out.println("自定义策略任务" + taskId + " 执行");
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
              });
          }
          
          executor.shutdown();
      }
  }
  ```
  
  **线程池工作流程详解：**
  
  ```java
  public class ThreadPoolWorkflowExample {
      public static void main(String[] args) {
          // 创建线程池：核心2，最大4，队列容量2
          ThreadPoolExecutor executor = new ThreadPoolExecutor(
              2,  // corePoolSize
              4,  // maximumPoolSize
              60L, TimeUnit.SECONDS,
              new ArrayBlockingQueue<>(2), // 队列容量2
              new ThreadPoolExecutor.CallerRunsPolicy()
          );
          
          // 提交8个任务，观察执行流程
          for (int i = 1; i <= 8; i++) {
              final int taskId = i;
              
              System.out.println("\n=== 提交任务" + taskId + " ===");
              System.out.println("提交前 - 线程数: " + executor.getPoolSize() + 
                  ", 队列大小: " + executor.getQueue().size());
              
              executor.submit(() -> {
                  System.out.println("任务" + taskId + " 开始执行，线程: " + 
                      Thread.currentThread().getName());
                  try {
                      Thread.sleep(3000); // 模拟任务执行时间
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  }
                  System.out.println("任务" + taskId + " 执行完成");
              });
              
              System.out.println("提交后 - 线程数: " + executor.getPoolSize() + 
                  ", 队列大小: " + executor.getQueue().size());
              
              try {
                  Thread.sleep(500); // 稍微延迟，便于观察
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
              }
          }
          
          executor.shutdown();
      }
  }
  ```
  
  **工作流程图解：**
  
  ```
  任务提交
      ↓
  线程数 < corePoolSize？
      ↓ 是
  创建核心线程执行任务
      ↓ 否
  工作队列未满？
      ↓ 是
  任务加入队列等待
      ↓ 否
  线程数 < maximumPoolSize？
      ↓ 是
  创建非核心线程执行任务
      ↓ 否
  执行拒绝策略
  ```
  
  **线程池参数配置建议：**
  
  | 任务类型 | corePoolSize | maximumPoolSize | workQueue | 说明 |
  |----------|--------------|-----------------|-----------|------|
  | **CPU密集型** | CPU核数 | CPU核数 | LinkedBlockingQueue | 避免线程切换开销 |
  | **IO密集型** | 2 * CPU核数 | 2 * CPU核数 | LinkedBlockingQueue | 充分利用IO等待时间 |
  | **混合型** | CPU核数 + 1 | 2 * CPU核数 | ArrayBlockingQueue | 平衡CPU和IO |
  | **突发型** | 较小值 | 较大值 | SynchronousQueue | 快速响应突发请求 |
  
  **最佳实践：**
  1. **合理设置核心线程数**：根据任务类型和系统资源
  2. **选择合适的队列**：有界队列防止内存溢出
  3. **自定义线程工厂**：便于问题排查和监控
  4. **选择合适的拒绝策略**：根据业务需求处理任务溢出
  5. **监控线程池状态**：定期检查线程池健康状况
  6. **优雅关闭**：使用shutdown()而非shutdownNow()
  
  ### IO相关
  
  **6. BIO、NIO、AIO的区别？**
  
  **答案：**
  
  **BIO（Blocking IO - 同步阻塞IO）：**
  
  ```java
  // BIO服务器示例
  public class BIOServer {
      public static void main(String[] args) throws IOException {
          ServerSocket serverSocket = new ServerSocket(8080);
          System.out.println("BIO服务器启动，监听端口8080");
          
          while (true) {
              // 阻塞等待客户端连接
              Socket clientSocket = serverSocket.accept();
              System.out.println("客户端连接：" + clientSocket.getRemoteSocketAddress());
              
              // 为每个客户端创建一个线程
              new Thread(() -> handleClient(clientSocket)).start();
          }
      }
      
      private static void handleClient(Socket clientSocket) {
          try (BufferedReader reader = new BufferedReader(
                  new InputStreamReader(clientSocket.getInputStream()));
               PrintWriter writer = new PrintWriter(
                  clientSocket.getOutputStream(), true)) {
              
              String inputLine;
              while ((inputLine = reader.readLine()) != null) {
                  System.out.println("收到消息：" + inputLine);
                  
                  // 模拟处理时间
                  Thread.sleep(1000);
                  
                  writer.println("Echo: " + inputLine);
                  
                  if ("bye".equalsIgnoreCase(inputLine)) {
                      break;
                  }
              }
          } catch (IOException | InterruptedException e) {
              e.printStackTrace();
          } finally {
              try {
                  clientSocket.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```
  
  **NIO（Non-blocking IO - 同步非阻塞IO）：**
  
  ```java
  // NIO服务器示例
  public class NIOServer {
      private Selector selector;
      private ServerSocketChannel serverChannel;
      private static final int PORT = 8080;
      
      public static void main(String[] args) {
          new NIOServer().start();
      }
      
      public void start() {
          try {
              // 创建选择器
              selector = Selector.open();
              
              // 创建服务器通道
              serverChannel = ServerSocketChannel.open();
              serverChannel.bind(new InetSocketAddress(PORT));
              serverChannel.configureBlocking(false); // 设置为非阻塞
              
              // 注册到选择器，监听连接事件
              serverChannel.register(selector, SelectionKey.OP_ACCEPT);
              
              System.out.println("NIO服务器启动，监听端口" + PORT);
              
              while (true) {
                  // 阻塞等待事件
                  int readyChannels = selector.select();
                  
                  if (readyChannels == 0) {
                      continue;
                  }
                  
                  Set<SelectionKey> selectedKeys = selector.selectedKeys();
                  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
                  
                  while (keyIterator.hasNext()) {
                      SelectionKey key = keyIterator.next();
                      
                      if (key.isAcceptable()) {
                          handleAccept(key);
                      } else if (key.isReadable()) {
                          handleRead(key);
                      }
                      
                      keyIterator.remove();
                  }
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      
      private void handleAccept(SelectionKey key) throws IOException {
          ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
          SocketChannel clientChannel = serverChannel.accept();
          
          if (clientChannel != null) {
              clientChannel.configureBlocking(false);
              clientChannel.register(selector, SelectionKey.OP_READ);
              System.out.println("客户端连接：" + clientChannel.getRemoteAddress());
          }
      }
      
      private void handleRead(SelectionKey key) throws IOException {
          SocketChannel clientChannel = (SocketChannel) key.channel();
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          
          try {
              int bytesRead = clientChannel.read(buffer);
              
              if (bytesRead > 0) {
                  buffer.flip();
                  String message = new String(buffer.array(), 0, bytesRead);
                  System.out.println("收到消息：" + message.trim());
                  
                  // 回写数据
                  String response = "Echo: " + message;
                  ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                  clientChannel.write(responseBuffer);
                  
              } else if (bytesRead < 0) {
                  // 客户端断开连接
                  System.out.println("客户端断开连接：" + clientChannel.getRemoteAddress());
                  key.cancel();
                  clientChannel.close();
              }
          } catch (IOException e) {
              System.out.println("客户端异常断开：" + e.getMessage());
              key.cancel();
              clientChannel.close();
          }
      }
  }
  ```
  
  **AIO（Asynchronous IO - 异步非阻塞IO）：**
  
  ```java
  // AIO服务器示例
  public class AIOServer {
      private AsynchronousServerSocketChannel serverChannel;
      private static final int PORT = 8080;
      
      public static void main(String[] args) {
          new AIOServer().start();
      }
      
      public void start() {
          try {
              serverChannel = AsynchronousServerSocketChannel.open();
              serverChannel.bind(new InetSocketAddress(PORT));
              
              System.out.println("AIO服务器启动，监听端口" + PORT);
              
              // 异步接受连接
              serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
                  @Override
                  public void completed(AsynchronousSocketChannel clientChannel, Void attachment) {
                      // 继续接受下一个连接
                      serverChannel.accept(null, this);
                      
                      try {
                          System.out.println("客户端连接：" + clientChannel.getRemoteAddress());
                          handleClient(clientChannel);
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
                  
                  @Override
                  public void failed(Throwable exc, Void attachment) {
                      exc.printStackTrace();
                  }
              });
              
              // 保持主线程运行
              Thread.currentThread().join();
              
          } catch (IOException | InterruptedException e) {
              e.printStackTrace();
          }
      }
      
      private void handleClient(AsynchronousSocketChannel clientChannel) {
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          
          // 异步读取数据
          clientChannel.read(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
              @Override
              public void completed(Integer result, ByteBuffer attachment) {
                  if (result > 0) {
                      attachment.flip();
                      String message = new String(attachment.array(), 0, result);
                      System.out.println("收到消息：" + message.trim());
                      
                      // 异步写回数据
                      String response = "Echo: " + message;
                      ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                      
                      clientChannel.write(responseBuffer, responseBuffer, 
                          new CompletionHandler<Integer, ByteBuffer>() {
                              @Override
                              public void completed(Integer result, ByteBuffer attachment) {
                                  if (attachment.hasRemaining()) {
                                      clientChannel.write(attachment, attachment, this);
                                  } else {
                                      // 继续读取下一条消息
                                      ByteBuffer newBuffer = ByteBuffer.allocate(1024);
                                      clientChannel.read(newBuffer, newBuffer, 
                                          AIOServer.this.new ReadHandler(clientChannel));
                                  }
                              }
                              
                              @Override
                              public void failed(Throwable exc, ByteBuffer attachment) {
                                  exc.printStackTrace();
                                  try {
                                      clientChannel.close();
                                  } catch (IOException e) {
                                      e.printStackTrace();
                                  }
                              }
                          });
                  } else {
                      // 客户端断开连接
                      try {
                          System.out.println("客户端断开连接");
                          clientChannel.close();
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
              }
              
              @Override
              public void failed(Throwable exc, ByteBuffer attachment) {
                  exc.printStackTrace();
                  try {
                      clientChannel.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          });
      }
      
      private class ReadHandler implements CompletionHandler<Integer, ByteBuffer> {
          private AsynchronousSocketChannel clientChannel;
          
          public ReadHandler(AsynchronousSocketChannel clientChannel) {
              this.clientChannel = clientChannel;
          }
          
          @Override
          public void completed(Integer result, ByteBuffer attachment) {
              if (result > 0) {
                  attachment.flip();
                  String message = new String(attachment.array(), 0, result);
                  System.out.println("收到消息：" + message.trim());
                  
                  // 异步写回数据
                  String response = "Echo: " + message;
                  ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                  
                  clientChannel.write(responseBuffer, responseBuffer, 
                      new CompletionHandler<Integer, ByteBuffer>() {
                          @Override
                          public void completed(Integer result, ByteBuffer attachment) {
                              if (attachment.hasRemaining()) {
                                  clientChannel.write(attachment, attachment, this);
                              } else {
                                  // 继续读取下一条消息
                                  ByteBuffer newBuffer = ByteBuffer.allocate(1024);
                                  clientChannel.read(newBuffer, newBuffer, ReadHandler.this);
                              }
                          }
                          
                          @Override
                          public void failed(Throwable exc, ByteBuffer attachment) {
                              exc.printStackTrace();
                              try {
                                  clientChannel.close();
                              } catch (IOException e) {
                                  e.printStackTrace();
                              }
                          }
                      });
              } else {
                  // 客户端断开连接
                  try {
                      System.out.println("客户端断开连接");
                      clientChannel.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
          
          @Override
          public void failed(Throwable exc, ByteBuffer attachment) {
              exc.printStackTrace();
              try {
                  clientChannel.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```
  
  **详细对比分析：**
  
  | 特性 | BIO | NIO | AIO |
  |------|-----|-----|-----|
  | **同步/异步** | 同步 | 同步 | 异步 |
  | **阻塞/非阻塞** | 阻塞 | 非阻塞 | 非阻塞 |
  | **线程模型** | 一连接一线程 | 一线程处理多连接 | 事件驱动 |
  | **内存拷贝** | 多次拷贝 | 减少拷贝（DirectBuffer） | 减少拷贝 |
  | **适用场景** | 连接数少且稳定 | 连接数多但活跃度不高 | 连接数多且活跃度高 |
  | **编程复杂度** | 简单 | 复杂 | 最复杂 |
  | **性能** | 差 | 好 | 最好 |
  | **资源消耗** | 高（线程多） | 中等 | 低 |
  
  **使用场景建议：**
  
  1. **BIO适用场景：**
     - 连接数比较小且固定的架构
     - 服务器资源充足
     - 开发时间紧张，需要快速实现
     - 对性能要求不高的内部系统
  
  2. **NIO适用场景：**
     - 连接数目多且连接比较短（轻操作）
     - 聊天服务器、弹幕系统、服务器间通讯
     - 需要维持很多连接但不是很活跃的情况
  
  3. **AIO适用场景：**
     - 连接数目多且连接比较长（重操作）
     - 文件服务器、数据库连接
     - 对性能要求极高的系统
  
  **总结：**
  - **BIO**：简单易用，但性能有限，适合小规模应用
  - **NIO**：复杂但高效，适合大多数高并发场景
  - **AIO**：最复杂但性能最好，适合极高性能要求的场景
  
  **7. NIO的核心组件？**
  
  **答案：**
  
  NIO的三大核心组件是Channel、Buffer和Selector，它们协同工作实现高效的非阻塞IO操作。
  
  **1. Channel（通道）**
  
  Channel是数据传输的双向管道，类似于传统IO中的Stream，但有以下特点：
  - 既可以读也可以写（Stream是单向的）
  - 可以异步读写
  - 总是基于Buffer进行读写
  
  ```java
  // Channel示例
  public class ChannelExample {
      public static void main(String[] args) throws IOException {
          // 1. FileChannel - 文件操作
          fileChannelExample();
          
          // 2. SocketChannel - TCP客户端
          socketChannelExample();
          
          // 3. ServerSocketChannel - TCP服务端
          serverSocketChannelExample();
          
          // 4. DatagramChannel - UDP
          datagramChannelExample();
      }
      
      // FileChannel示例
      private static void fileChannelExample() throws IOException {
          // 读取文件
          try (RandomAccessFile file = new RandomAccessFile("test.txt", "rw");
               FileChannel channel = file.getChannel()) {
              
              // 分配缓冲区
              ByteBuffer buffer = ByteBuffer.allocate(1024);
              
              // 从Channel读取数据到Buffer
              int bytesRead = channel.read(buffer);
              while (bytesRead != -1) {
                  buffer.flip(); // 切换到读模式
                  
                  while (buffer.hasRemaining()) {
                      System.out.print((char) buffer.get());
                  }
                  
                  buffer.clear(); // 清空缓冲区
                  bytesRead = channel.read(buffer);
              }
              
              // 写入数据
              String data = "Hello NIO Channel!";
              ByteBuffer writeBuffer = ByteBuffer.allocate(1024);
              writeBuffer.put(data.getBytes());
              writeBuffer.flip();
              
              channel.write(writeBuffer);
          }
      }
      
      // SocketChannel示例
      private static void socketChannelExample() throws IOException {
          SocketChannel socketChannel = SocketChannel.open();
          socketChannel.configureBlocking(false);
          
          // 连接服务器
          socketChannel.connect(new InetSocketAddress("localhost", 8080));
          
          // 等待连接完成
          while (!socketChannel.finishConnect()) {
              // 可以做其他事情
              System.out.println("连接中...");
          }
          
          // 发送数据
          String message = "Hello Server";
          ByteBuffer buffer = ByteBuffer.wrap(message.getBytes());
          socketChannel.write(buffer);
          
          // 读取响应
          ByteBuffer readBuffer = ByteBuffer.allocate(1024);
          int bytesRead = socketChannel.read(readBuffer);
          if (bytesRead > 0) {
              readBuffer.flip();
              String response = new String(readBuffer.array(), 0, bytesRead);
              System.out.println("服务器响应：" + response);
          }
          
          socketChannel.close();
      }
      
      // ServerSocketChannel示例
      private static void serverSocketChannelExample() throws IOException {
          ServerSocketChannel serverChannel = ServerSocketChannel.open();
          serverChannel.configureBlocking(false);
          serverChannel.bind(new InetSocketAddress(8080));
          
          while (true) {
              SocketChannel clientChannel = serverChannel.accept();
              if (clientChannel != null) {
                  clientChannel.configureBlocking(false);
                  System.out.println("客户端连接：" + clientChannel.getRemoteAddress());
                  
                  // 处理客户端请求
                  ByteBuffer buffer = ByteBuffer.allocate(1024);
                  int bytesRead = clientChannel.read(buffer);
                  if (bytesRead > 0) {
                      buffer.flip();
                      String message = new String(buffer.array(), 0, bytesRead);
                      System.out.println("收到消息：" + message);
                      
                      // 回复客户端
                      String response = "Echo: " + message;
                      ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                      clientChannel.write(responseBuffer);
                  }
                  
                  clientChannel.close();
              }
              
              // 避免CPU空转
              Thread.sleep(100);
          }
      }
      
      // DatagramChannel示例
      private static void datagramChannelExample() throws IOException {
          DatagramChannel channel = DatagramChannel.open();
          channel.configureBlocking(false);
          channel.bind(new InetSocketAddress(9999));
          
          // 发送UDP数据
          String message = "Hello UDP";
          ByteBuffer buffer = ByteBuffer.wrap(message.getBytes());
          channel.send(buffer, new InetSocketAddress("localhost", 8888));
          
          // 接收UDP数据
          ByteBuffer receiveBuffer = ByteBuffer.allocate(1024);
          SocketAddress senderAddress = channel.receive(receiveBuffer);
          if (senderAddress != null) {
              receiveBuffer.flip();
              String receivedMessage = new String(receiveBuffer.array(), 0, receiveBuffer.remaining());
              System.out.println("收到UDP消息：" + receivedMessage + " 来自：" + senderAddress);
          }
          
          channel.close();
      }
  }
  ```
  
  **2. Buffer（缓冲区）**
  
  Buffer是数据存储的容器，所有数据都通过Buffer进行读写。Buffer有以下重要属性：
  - **capacity**：容量，Buffer的最大数据容量
  - **limit**：界限，Buffer中可操作数据的边界
  - **position**：位置，下一个要操作的数据位置
  - **mark**：标记，记住某个特定的position
  
  ```java
  // Buffer示例
  public class BufferExample {
      public static void main(String[] args) {
          // 1. ByteBuffer示例
          byteBufferExample();
          
          // 2. 其他类型Buffer示例
          otherBufferExample();
          
          // 3. 直接内存Buffer vs 堆内存Buffer
          directVsHeapBuffer();
      }
      
      private static void byteBufferExample() {
          // 创建Buffer
          ByteBuffer buffer = ByteBuffer.allocate(10);
          System.out.println("初始状态：" + bufferStatus(buffer));
          
          // 写入数据
          buffer.put("Hello".getBytes());
          System.out.println("写入Hello后：" + bufferStatus(buffer));
          
          // 切换到读模式
          buffer.flip();
          System.out.println("flip后：" + bufferStatus(buffer));
          
          // 读取数据
          byte[] data = new byte[buffer.remaining()];
          buffer.get(data);
          System.out.println("读取数据：" + new String(data));
          System.out.println("读取后：" + bufferStatus(buffer));
          
          // 重置到mark位置
          buffer.rewind(); // 重置position到0
          System.out.println("rewind后：" + bufferStatus(buffer));
          
          // 清空缓冲区
          buffer.clear();
          System.out.println("clear后：" + bufferStatus(buffer));
          
          // 使用mark和reset
          buffer.put("ABCDEF".getBytes());
          buffer.flip();
          
          buffer.get(); // 读取A
          buffer.get(); // 读取B
          buffer.mark(); // 标记当前位置
          
          buffer.get(); // 读取C
          buffer.get(); // 读取D
          
          buffer.reset(); // 回到mark位置
          System.out.println("reset后下一个字符：" + (char)buffer.get()); // 应该是C
      }
      
      private static void otherBufferExample() {
          // IntBuffer
          IntBuffer intBuffer = IntBuffer.allocate(5);
          intBuffer.put(new int[]{1, 2, 3, 4, 5});
          intBuffer.flip();
          
          while (intBuffer.hasRemaining()) {
              System.out.print(intBuffer.get() + " ");
          }
          System.out.println();
          
          // CharBuffer
          CharBuffer charBuffer = CharBuffer.allocate(10);
          charBuffer.put("Hello");
          charBuffer.flip();
          
          System.out.println("CharBuffer内容：" + charBuffer.toString());
          
          // DoubleBuffer
          DoubleBuffer doubleBuffer = DoubleBuffer.allocate(3);
          doubleBuffer.put(new double[]{1.1, 2.2, 3.3});
          doubleBuffer.flip();
          
          while (doubleBuffer.hasRemaining()) {
              System.out.print(doubleBuffer.get() + " ");
          }
          System.out.println();
      }
      
      private static void directVsHeapBuffer() {
          // 堆内存Buffer
          ByteBuffer heapBuffer = ByteBuffer.allocate(1024);
          System.out.println("堆内存Buffer isDirect: " + heapBuffer.isDirect());
          
          // 直接内存Buffer
          ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);
          System.out.println("直接内存Buffer isDirect: " + directBuffer.isDirect());
          
          // 性能测试
          long startTime, endTime;
          int iterations = 1000000;
          
          // 堆内存Buffer性能测试
          startTime = System.nanoTime();
          for (int i = 0; i < iterations; i++) {
              heapBuffer.putInt(0, i);
          }
          endTime = System.nanoTime();
          System.out.println("堆内存Buffer耗时：" + (endTime - startTime) / 1000000 + "ms");
          
          // 直接内存Buffer性能测试
          startTime = System.nanoTime();
          for (int i = 0; i < iterations; i++) {
              directBuffer.putInt(0, i);
          }
          endTime = System.nanoTime();
          System.out.println("直接内存Buffer耗时：" + (endTime - startTime) / 1000000 + "ms");
      }
      
      private static String bufferStatus(ByteBuffer buffer) {
          return String.format("position=%d, limit=%d, capacity=%d, remaining=%d",
                  buffer.position(), buffer.limit(), buffer.capacity(), buffer.remaining());
      }
  }
  ```
  
  **3. Selector（选择器）**
  
  Selector是NIO的核心组件，实现了多路复用，可以用一个线程管理多个Channel。
  
  ```java
  // Selector示例
  public class SelectorExample {
      public static void main(String[] args) throws IOException {
          // 创建选择器
          Selector selector = Selector.open();
          
          // 创建服务器通道
          ServerSocketChannel serverChannel = ServerSocketChannel.open();
          serverChannel.configureBlocking(false);
          serverChannel.bind(new InetSocketAddress(8080));
          
          // 注册到选择器
          SelectionKey serverKey = serverChannel.register(selector, SelectionKey.OP_ACCEPT);
          
          System.out.println("服务器启动，监听端口8080");
          
          while (true) {
              // 阻塞等待事件
              int readyChannels = selector.select(1000); // 超时1秒
              
              if (readyChannels == 0) {
                  System.out.println("没有就绪的通道，继续等待...");
                  continue;
              }
              
              // 获取就绪的SelectionKey
              Set<SelectionKey> selectedKeys = selector.selectedKeys();
              Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
              
              while (keyIterator.hasNext()) {
                  SelectionKey key = keyIterator.next();
                  
                  try {
                      if (key.isAcceptable()) {
                          handleAccept(key, selector);
                      } else if (key.isReadable()) {
                          handleRead(key);
                      } else if (key.isWritable()) {
                          handleWrite(key);
                      } else if (key.isConnectable()) {
                          handleConnect(key);
                      }
                  } catch (IOException e) {
                      System.out.println("处理事件时发生异常：" + e.getMessage());
                      key.cancel();
                      if (key.channel() != null) {
                          key.channel().close();
                      }
                  }
                  
                  keyIterator.remove();
              }
          }
      }
      
      private static void handleAccept(SelectionKey key, Selector selector) throws IOException {
          ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
          SocketChannel clientChannel = serverChannel.accept();
          
          if (clientChannel != null) {
              clientChannel.configureBlocking(false);
              
              // 注册读事件
              SelectionKey clientKey = clientChannel.register(selector, SelectionKey.OP_READ);
              
              // 可以在SelectionKey中附加数据
              clientKey.attach(new ClientData(clientChannel.getRemoteAddress().toString()));
              
              System.out.println("客户端连接：" + clientChannel.getRemoteAddress());
          }
      }
      
      private static void handleRead(SelectionKey key) throws IOException {
          SocketChannel clientChannel = (SocketChannel) key.channel();
          ClientData clientData = (ClientData) key.attachment();
          
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          int bytesRead = clientChannel.read(buffer);
          
          if (bytesRead > 0) {
              buffer.flip();
              String message = new String(buffer.array(), 0, bytesRead);
              System.out.println("收到来自 " + clientData.getAddress() + " 的消息：" + message.trim());
              
              // 准备回写数据
              String response = "Echo: " + message;
              clientData.setResponse(response);
              
              // 注册写事件
              key.interestOps(SelectionKey.OP_WRITE);
              
          } else if (bytesRead < 0) {
              // 客户端断开连接
              System.out.println("客户端断开连接：" + clientData.getAddress());
              key.cancel();
              clientChannel.close();
          }
      }
      
      private static void handleWrite(SelectionKey key) throws IOException {
          SocketChannel clientChannel = (SocketChannel) key.channel();
          ClientData clientData = (ClientData) key.attachment();
          
          if (clientData.getResponse() != null) {
              ByteBuffer buffer = ByteBuffer.wrap(clientData.getResponse().getBytes());
              clientChannel.write(buffer);
              
              if (!buffer.hasRemaining()) {
                  // 写完了，重新注册读事件
                  clientData.setResponse(null);
                  key.interestOps(SelectionKey.OP_READ);
              }
          }
      }
      
      private static void handleConnect(SelectionKey key) throws IOException {
          SocketChannel clientChannel = (SocketChannel) key.channel();
          
          if (clientChannel.finishConnect()) {
              System.out.println("连接建立成功");
              key.interestOps(SelectionKey.OP_READ);
          }
      }
      
      // 客户端数据类
      private static class ClientData {
          private String address;
          private String response;
          
          public ClientData(String address) {
              this.address = address;
          }
          
          public String getAddress() {
              return address;
          }
          
          public String getResponse() {
              return response;
          }
          
          public void setResponse(String response) {
              this.response = response;
          }
      }
  }
  ```
  
  **核心组件关系图：**
  
  ```
  ┌─────────────┐    register    ┌──────────────┐
  │   Channel   │ ──────────────→ │   Selector   │
  │             │                 │              │
  │ ┌─────────┐ │                 │ ┌──────────┐ │
  │ │ Buffer  │ │                 │ │Selection │ │
  │ │         │ │                 │ │   Key    │ │
  │ └─────────┘ │                 │ └──────────┘ │
  └─────────────┘                 └──────────────┘
        │                                │
        │ read/write                     │ select
        ▼                                ▼
  ┌─────────────┐                 ┌──────────────┐
  │    Data     │                 │    Events    │
  │             │                 │              │
  └─────────────┘                 └──────────────┘
  ```
  
  **总结：**
  
  | 组件 | 作用 | 特点 |
  |------|------|------|
  | **Channel** | 数据传输通道 | 双向、异步、基于Buffer |
  | **Buffer** | 数据缓冲区 | 有状态、可重用、类型安全 |
  | **Selector** | 多路复用器 | 事件驱动、单线程管理多连接 |
  
  这三个组件协同工作，实现了高效的非阻塞IO操作，是NIO高性能的核心所在。
  
  **8. 什么是零拷贝？**
  
  **答案：**
  
  零拷贝（Zero-Copy）是指数据在传输过程中，减少数据在内存中的拷贝次数，避免CPU将数据从一个存储区域复制到另一个存储区域，从而提高传输效率和系统性能。
  
  **传统IO的数据拷贝过程：**
  
  ```java
  // 传统IO读取文件并发送到网络
  public class TraditionalIO {
      public static void main(String[] args) throws IOException {
          // 传统方式：需要4次拷贝
          try (FileInputStream fis = new FileInputStream("large_file.txt");
               SocketOutputStream sos = socket.getOutputStream()) {
              
              byte[] buffer = new byte[8192];
              int bytesRead;
              
              while ((bytesRead = fis.read(buffer)) != -1) {
                  sos.write(buffer, 0, bytesRead);
              }
          }
      }
  }
  ```
  
  **传统IO的拷贝过程：**
  ```
  1. 磁盘 → 内核缓冲区（DMA拷贝）
  2. 内核缓冲区 → 用户缓冲区（CPU拷贝）
  3. 用户缓冲区 → Socket缓冲区（CPU拷贝）
  4. Socket缓冲区 → 网卡（DMA拷贝）
  
  总共：4次拷贝（2次CPU拷贝 + 2次DMA拷贝）
  ```
  
  **零拷贝的实现方式：**
  
  **1. mmap（内存映射）**
  
  ```java
  // 使用内存映射文件
  public class MmapExample {
      public static void main(String[] args) throws IOException {
          try (RandomAccessFile file = new RandomAccessFile("large_file.txt", "r");
               FileChannel channel = file.getChannel()) {
              
              // 将文件映射到内存
              MappedByteBuffer mappedBuffer = channel.map(
                  FileChannel.MapMode.READ_ONLY, 0, channel.size());
              
              // 直接从映射内存读取数据
              byte[] data = new byte[1024];
              mappedBuffer.get(data);
              
              System.out.println("读取数据：" + new String(data));
          }
      }
      
      // 大文件处理示例
      public static void processLargeFile(String fileName) throws IOException {
          try (RandomAccessFile file = new RandomAccessFile(fileName, "r");
               FileChannel channel = file.getChannel()) {
              
              long fileSize = channel.size();
              long position = 0;
              long chunkSize = 1024 * 1024; // 1MB chunks
              
              while (position < fileSize) {
                  long remainingSize = fileSize - position;
                  long currentChunkSize = Math.min(chunkSize, remainingSize);
                  
                  // 映射文件的一部分
                  MappedByteBuffer buffer = channel.map(
                      FileChannel.MapMode.READ_ONLY, position, currentChunkSize);
                  
                  // 处理数据
                  processChunk(buffer);
                  
                  position += currentChunkSize;
              }
          }
      }
      
      private static void processChunk(MappedByteBuffer buffer) {
          // 处理数据块
          while (buffer.hasRemaining()) {
              byte b = buffer.get();
              // 处理字节数据
          }
      }
  }
  ```
  
  **2. sendfile系统调用（通过FileChannel.transferTo()）**
  
  ```java
  // 使用transferTo实现零拷贝
  public class ZeroCopyExample {
      public static void main(String[] args) throws IOException {
          // 零拷贝文件传输
          transferFileZeroCopy();
          
          // 性能对比测试
          performanceComparison();
      }
      
      // 零拷贝文件传输
      private static void transferFileZeroCopy() throws IOException {
          try (FileChannel sourceChannel = FileChannel.open(
                  Paths.get("source.txt"), StandardOpenOption.READ);
               FileChannel targetChannel = FileChannel.open(
                  Paths.get("target.txt"), StandardOpenOption.WRITE, 
                  StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING)) {
              
              long fileSize = sourceChannel.size();
              long position = 0;
              
              // 使用transferTo进行零拷贝传输
              while (position < fileSize) {
                  long transferred = sourceChannel.transferTo(
                      position, fileSize - position, targetChannel);
                  position += transferred;
              }
              
              System.out.println("零拷贝传输完成，文件大小：" + fileSize + " 字节");
          }
      }
      
      // 网络零拷贝传输
      public static void networkZeroCopy(String fileName, SocketChannel socketChannel) 
              throws IOException {
          try (FileChannel fileChannel = FileChannel.open(
                  Paths.get(fileName), StandardOpenOption.READ)) {
              
              long fileSize = fileChannel.size();
              long position = 0;
              
              // 直接从文件传输到网络
              while (position < fileSize) {
                  long transferred = fileChannel.transferTo(
                      position, fileSize - position, socketChannel);
                  position += transferred;
              }
              
              System.out.println("网络零拷贝传输完成");
          }
      }
      
      // 性能对比测试
      private static void performanceComparison() throws IOException {
          String sourceFile = "large_test_file.txt";
          createTestFile(sourceFile, 100 * 1024 * 1024); // 100MB测试文件
          
          // 传统IO测试
          long startTime = System.currentTimeMillis();
          traditionalCopy(sourceFile, "traditional_copy.txt");
          long traditionalTime = System.currentTimeMillis() - startTime;
          
          // 零拷贝测试
          startTime = System.currentTimeMillis();
          zeroCopy(sourceFile, "zero_copy.txt");
          long zeroCopyTime = System.currentTimeMillis() - startTime;
          
          System.out.println("传统IO耗时：" + traditionalTime + "ms");
          System.out.println("零拷贝耗时：" + zeroCopyTime + "ms");
          System.out.println("性能提升：" + (traditionalTime * 100.0 / zeroCopyTime) + "%");
      }
      
      // 传统IO拷贝
      private static void traditionalCopy(String source, String target) throws IOException {
          try (FileInputStream fis = new FileInputStream(source);
               FileOutputStream fos = new FileOutputStream(target)) {
              
              byte[] buffer = new byte[8192];
              int bytesRead;
              
              while ((bytesRead = fis.read(buffer)) != -1) {
                  fos.write(buffer, 0, bytesRead);
              }
          }
      }
      
      // 零拷贝
      private static void zeroCopy(String source, String target) throws IOException {
          try (FileChannel sourceChannel = FileChannel.open(
                  Paths.get(source), StandardOpenOption.READ);
               FileChannel targetChannel = FileChannel.open(
                  Paths.get(target), StandardOpenOption.WRITE, 
                  StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING)) {
              
              sourceChannel.transferTo(0, sourceChannel.size(), targetChannel);
          }
      }
      
      // 创建测试文件
      private static void createTestFile(String fileName, int sizeInBytes) throws IOException {
          try (FileChannel channel = FileChannel.open(
                  Paths.get(fileName), StandardOpenOption.WRITE, 
                  StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING)) {
              
              ByteBuffer buffer = ByteBuffer.allocate(8192);
              byte[] data = "This is test data for zero copy performance testing.\n".getBytes();
              
              int written = 0;
              while (written < sizeInBytes) {
                  buffer.clear();
                  int remaining = Math.min(buffer.remaining(), sizeInBytes - written);
                  
                  for (int i = 0; i < remaining; i += data.length) {
                      int copyLength = Math.min(data.length, remaining - i);
                      buffer.put(data, 0, copyLength);
                  }
                  
                  buffer.flip();
                  written += channel.write(buffer);
              }
          }
      }
  }
  ```
  
  **3. DirectByteBuffer（直接内存）**
  
  ```java
  // DirectByteBuffer示例
  public class DirectBufferExample {
      public static void main(String[] args) throws IOException {
          // 直接内存vs堆内存性能对比
          performanceTest();
          
          // 网络传输中使用DirectBuffer
          networkTransferWithDirectBuffer();
      }
      
      private static void performanceTest() throws IOException {
          int bufferSize = 1024 * 1024; // 1MB
          int iterations = 1000;
          
          // 堆内存Buffer测试
          ByteBuffer heapBuffer = ByteBuffer.allocate(bufferSize);
          long startTime = System.nanoTime();
          
          for (int i = 0; i < iterations; i++) {
              heapBuffer.clear();
              // 模拟数据操作
              for (int j = 0; j < bufferSize; j++) {
                  heapBuffer.put((byte) (j % 256));
              }
          }
          
          long heapTime = System.nanoTime() - startTime;
          
          // 直接内存Buffer测试
          ByteBuffer directBuffer = ByteBuffer.allocateDirect(bufferSize);
          startTime = System.nanoTime();
          
          for (int i = 0; i < iterations; i++) {
              directBuffer.clear();
              // 模拟数据操作
              for (int j = 0; j < bufferSize; j++) {
                  directBuffer.put((byte) (j % 256));
              }
          }
          
          long directTime = System.nanoTime() - startTime;
          
          System.out.println("堆内存Buffer耗时：" + heapTime / 1000000 + "ms");
          System.out.println("直接内存Buffer耗时：" + directTime / 1000000 + "ms");
          System.out.println("性能提升：" + (heapTime * 100.0 / directTime) + "%");
      }
      
      private static void networkTransferWithDirectBuffer() throws IOException {
          // 使用DirectBuffer进行网络传输
          try (ServerSocketChannel serverChannel = ServerSocketChannel.open()) {
              serverChannel.bind(new InetSocketAddress(8080));
              serverChannel.configureBlocking(false);
              
              Selector selector = Selector.open();
              serverChannel.register(selector, SelectionKey.OP_ACCEPT);
              
              ByteBuffer directBuffer = ByteBuffer.allocateDirect(8192);
              
              while (true) {
                  if (selector.select(1000) > 0) {
                      Set<SelectionKey> keys = selector.selectedKeys();
                      Iterator<SelectionKey> iterator = keys.iterator();
                      
                      while (iterator.hasNext()) {
                          SelectionKey key = iterator.next();
                          iterator.remove();
                          
                          if (key.isAcceptable()) {
                              SocketChannel clientChannel = serverChannel.accept();
                              if (clientChannel != null) {
                                  clientChannel.configureBlocking(false);
                                  clientChannel.register(selector, SelectionKey.OP_READ);
                              }
                          } else if (key.isReadable()) {
                              SocketChannel clientChannel = (SocketChannel) key.channel();
                              
                              directBuffer.clear();
                              int bytesRead = clientChannel.read(directBuffer);
                              
                              if (bytesRead > 0) {
                                  directBuffer.flip();
                                  // 直接从DirectBuffer写回客户端
                                  clientChannel.write(directBuffer);
                              } else if (bytesRead < 0) {
                                  key.cancel();
                                  clientChannel.close();
                              }
                          }
                      }
                  }
              }
          }
      }
  }
  ```
  
  **零拷贝技术对比：**
  
  | 技术 | 原理 | 优点 | 缺点 | 适用场景 |
  |------|------|------|------|----------|
  | **mmap** | 内存映射文件 | 减少拷贝，随机访问快 | 占用虚拟内存，大文件可能OOM | 大文件读取，随机访问 |
  | **sendfile** | 内核直接传输 | 完全零拷贝，CPU占用低 | 只支持文件到Socket | 文件服务器，静态资源传输 |
  | **DirectBuffer** | 直接内存 | 减少JVM堆拷贝 | 内存管理复杂，GC压力 | 网络IO，大数据处理 |
  | **transferTo** | 系统调用优化 | 简单易用，性能好 | 平台相关性 | 文件传输，数据复制 |
  
  **零拷贝的优势：**
  
  1. **减少CPU开销**：避免不必要的数据拷贝
  2. **提高传输效率**：减少内存带宽占用
  3. **降低延迟**：减少数据在内存中的停留时间
  4. **节省内存**：避免重复的数据副本
  
  **使用建议：**
  
  1. **文件传输**：优先使用`FileChannel.transferTo()`
  2. **网络IO**：使用`DirectByteBuffer`
  3. **大文件处理**：考虑使用`mmap`
  4. **高并发场景**：结合NIO和零拷贝技术
  
  **注意事项：**
  
  1. 零拷贝技术依赖于操作系统支持
  2. 不是所有场景都适合零拷贝
  3. 需要考虑内存使用和GC影响
  4. 要根据具体业务场景选择合适的技术
  
  ### 反射相关
  
  **9. 反射的优缺点？**
  
  **答案：**
  
  反射（Reflection）是Java提供的一种在运行时检查和操作类、方法、字段等的机制。
  
  **反射的优点：**
  
  **1. 运行时动态获取类信息**
  
  ```java
  // 反射基础示例
  public class ReflectionExample {
      public static void main(String[] args) throws Exception {
          // 获取Class对象的三种方式
          Class<?> clazz1 = String.class;
          Class<?> clazz2 = "hello".getClass();
          Class<?> clazz3 = Class.forName("java.lang.String");
          
          // 获取类信息
          System.out.println("类名：" + clazz1.getName());
          System.out.println("简单类名：" + clazz1.getSimpleName());
          System.out.println("包名：" + clazz1.getPackage().getName());
          System.out.println("父类：" + clazz1.getSuperclass().getName());
          
          // 获取接口信息
          Class<?>[] interfaces = clazz1.getInterfaces();
          System.out.println("实现的接口：");
          for (Class<?> intf : interfaces) {
              System.out.println("  " + intf.getName());
          }
      }
  }
  ```
  
  **2. 动态创建对象和调用方法**
  
  ```java
  // 动态操作示例
  public class DynamicOperationExample {
      public static void main(String[] args) throws Exception {
          // 动态创建对象
          Class<?> clazz = ArrayList.class;
          Object list = clazz.getDeclaredConstructor().newInstance();
          
          // 动态调用方法
          Method addMethod = clazz.getMethod("add", Object.class);
          addMethod.invoke(list, "Hello");
          addMethod.invoke(list, "World");
          
          Method sizeMethod = clazz.getMethod("size");
          int size = (Integer) sizeMethod.invoke(list);
          System.out.println("列表大小：" + size);
          
          // 动态访问字段
          Field elementDataField = clazz.getDeclaredField("elementData");
          elementDataField.setAccessible(true);
          Object[] elementData = (Object[]) elementDataField.get(list);
          System.out.println("内部数组长度：" + elementData.length);
      }
  }
  ```
  
  **3. 提高程序的灵活性和扩展性**
  
  ```java
  // 插件化架构示例
  public interface Plugin {
      void execute();
  }
  
  public class PluginA implements Plugin {
      @Override
      public void execute() {
          System.out.println("执行插件A的功能");
      }
  }
  
  public class PluginB implements Plugin {
      @Override
      public void execute() {
          System.out.println("执行插件B的功能");
      }
  }
  
  // 插件管理器
  public class PluginManager {
      private Map<String, Class<? extends Plugin>> plugins = new HashMap<>();
      
      // 注册插件
      public void registerPlugin(String name, String className) throws ClassNotFoundException {
          Class<?> clazz = Class.forName(className);
          if (Plugin.class.isAssignableFrom(clazz)) {
              plugins.put(name, (Class<? extends Plugin>) clazz);
          }
      }
      
      // 执行插件
      public void executePlugin(String name) throws Exception {
          Class<? extends Plugin> pluginClass = plugins.get(name);
          if (pluginClass != null) {
              Plugin plugin = pluginClass.getDeclaredConstructor().newInstance();
              plugin.execute();
          }
      }
      
      public static void main(String[] args) throws Exception {
          PluginManager manager = new PluginManager();
          
          // 动态注册插件
          manager.registerPlugin("pluginA", "PluginA");
          manager.registerPlugin("pluginB", "PluginB");
          
          // 动态执行插件
          manager.executePlugin("pluginA");
          manager.executePlugin("pluginB");
      }
  }
  ```
  
  **反射的缺点：**
  
  **1. 性能开销大**
  
  ```java
  // 性能对比测试
  public class ReflectionPerformanceTest {
      private static final int ITERATIONS = 10_000_000;
      
      public static void main(String[] args) throws Exception {
          // 直接调用性能测试
          long startTime = System.nanoTime();
          for (int i = 0; i < ITERATIONS; i++) {
              String str = "test";
              int length = str.length();
          }
          long directTime = System.nanoTime() - startTime;
          
          // 反射调用性能测试
          Class<?> clazz = String.class;
          Method lengthMethod = clazz.getMethod("length");
          
          startTime = System.nanoTime();
          for (int i = 0; i < ITERATIONS; i++) {
              String str = "test";
              int length = (Integer) lengthMethod.invoke(str);
          }
          long reflectionTime = System.nanoTime() - startTime;
          
          System.out.println("直接调用耗时：" + directTime / 1_000_000 + "ms");
          System.out.println("反射调用耗时：" + reflectionTime / 1_000_000 + "ms");
          System.out.println("性能差异：" + (reflectionTime / (double) directTime) + "倍");
      }
  }
  ```
  
  **2. 破坏封装性**
  
  ```java
  // 破坏封装性示例
  public class EncapsulationBreakExample {
      private String secret = "这是私有字段";
      private int count = 0;
      
      private void privateMethod() {
          System.out.println("这是私有方法");
      }
      
      public static void main(String[] args) throws Exception {
          EncapsulationBreakExample obj = new EncapsulationBreakExample();
          Class<?> clazz = obj.getClass();
          
          // 访问私有字段
          Field secretField = clazz.getDeclaredField("secret");
          secretField.setAccessible(true);
          String secret = (String) secretField.get(obj);
          System.out.println("私有字段值：" + secret);
          
          // 修改私有字段
          secretField.set(obj, "被修改的私有字段");
          System.out.println("修改后的值：" + secretField.get(obj));
          
          // 调用私有方法
          Method privateMethod = clazz.getDeclaredMethod("privateMethod");
          privateMethod.setAccessible(true);
          privateMethod.invoke(obj);
      }
  }
  ```
  
  **3. 代码可读性差和编译时无法检查错误**
  
  ```java
  // 代码可读性和错误检查问题
  public class ReflectionProblemsExample {
      public static void main(String[] args) {
          try {
              // 字符串硬编码，容易出错
              Class<?> clazz = Class.forName("java.util.ArrayLis"); // 拼写错误
              Method method = clazz.getMethod("ad", Object.class); // 方法名错误
              
              // 运行时才能发现错误
              Object list = clazz.getDeclaredConstructor().newInstance();
              method.invoke(list, "test");
              
          } catch (Exception e) {
              System.out.println("运行时错误：" + e.getMessage());
              // 这些错误在编译时无法发现
          }
      }
  }
  ```
  
  **反射优缺点总结：**
  
  | 方面 | 优点 | 缺点 |
  |------|------|------|
  | **灵活性** | 运行时动态操作，支持插件化架构 | 代码复杂，难以维护 |
  | **性能** | 无 | 比直接调用慢10-100倍 |
  | **安全性** | 可以访问任何类和成员 | 破坏封装性，安全风险 |
  | **错误检查** | 无 | 编译时无法检查，运行时才发现错误 |
  | **代码质量** | 提高扩展性 | 降低可读性和可维护性 |
  
  **10. 如何提高反射性能？**
  
  **答案：**
  
  **1. 缓存Class对象、Method对象等**
  
  ```java
  // 反射缓存优化
  public class ReflectionCache {
      // 缓存Class对象
      private static final Map<String, Class<?>> classCache = new ConcurrentHashMap<>();
      
      // 缓存Method对象
      private static final Map<String, Method> methodCache = new ConcurrentHashMap<>();
      
      // 缓存Field对象
      private static final Map<String, Field> fieldCache = new ConcurrentHashMap<>();
      
      // 获取缓存的Class对象
      public static Class<?> getCachedClass(String className) throws ClassNotFoundException {
          return classCache.computeIfAbsent(className, name -> {
              try {
                  return Class.forName(name);
              } catch (ClassNotFoundException e) {
                  throw new RuntimeException(e);
              }
          });
      }
      
      // 获取缓存的Method对象
      public static Method getCachedMethod(Class<?> clazz, String methodName, Class<?>... paramTypes) 
              throws NoSuchMethodException {
          String key = clazz.getName() + "#" + methodName + "#" + Arrays.toString(paramTypes);
          return methodCache.computeIfAbsent(key, k -> {
              try {
                  Method method = clazz.getMethod(methodName, paramTypes);
                  method.setAccessible(true); // 预先设置访问权限
                  return method;
              } catch (NoSuchMethodException e) {
                  throw new RuntimeException(e);
              }
          });
      }
      
      // 获取缓存的Field对象
      public static Field getCachedField(Class<?> clazz, String fieldName) throws NoSuchFieldException {
          String key = clazz.getName() + "#" + fieldName;
          return fieldCache.computeIfAbsent(key, k -> {
              try {
                  Field field = clazz.getDeclaredField(fieldName);
                  field.setAccessible(true); // 预先设置访问权限
                  return field;
              } catch (NoSuchFieldException e) {
                  throw new RuntimeException(e);
              }
          });
      }
  }
  ```
  
  **2. 使用setAccessible(true)减少安全检查**
  
  ```java
  // 访问权限优化
  public class AccessibilityOptimization {
      public static void main(String[] args) throws Exception {
          Class<?> clazz = String.class;
          Method method = clazz.getDeclaredMethod("charAt", int.class);
          
          // 预先设置访问权限，避免每次调用时检查
          method.setAccessible(true);
          
          String str = "Hello World";
          long startTime = System.nanoTime();
          
          // 多次调用，只需要一次setAccessible
          for (int i = 0; i < 1_000_000; i++) {
              char c = (Character) method.invoke(str, 0);
          }
          
          long endTime = System.nanoTime();
          System.out.println("优化后耗时：" + (endTime - startTime) / 1_000_000 + "ms");
      }
  }
  ```
  
  **3. 避免频繁的反射调用**
  
  ```java
  // 反射调用优化策略
  public class ReflectionOptimization {
      
      // 方法句柄优化（Java 7+）
      public static void methodHandleExample() throws Throwable {
          MethodHandles.Lookup lookup = MethodHandles.lookup();
          MethodType methodType = MethodType.methodType(int.class);
          MethodHandle lengthHandle = lookup.findVirtual(String.class, "length", methodType);
          
          String str = "Hello";
          
          // MethodHandle比反射更快
          long startTime = System.nanoTime();
          for (int i = 0; i < 1_000_000; i++) {
              int length = (int) lengthHandle.invokeExact(str);
          }
          long methodHandleTime = System.nanoTime() - startTime;
          
          // 传统反射
          Method lengthMethod = String.class.getMethod("length");
          startTime = System.nanoTime();
          for (int i = 0; i < 1_000_000; i++) {
              int length = (Integer) lengthMethod.invoke(str);
          }
          long reflectionTime = System.nanoTime() - startTime;
          
          System.out.println("MethodHandle耗时：" + methodHandleTime / 1_000_000 + "ms");
          System.out.println("反射耗时：" + reflectionTime / 1_000_000 + "ms");
      }
      
      // 批量操作优化
      public static void batchOperationExample() throws Exception {
          List<String> strings = Arrays.asList("a", "bb", "ccc", "dddd");
          Method lengthMethod = String.class.getMethod("length");
          lengthMethod.setAccessible(true);
          
          // 批量处理，减少反射调用开销
          List<Integer> lengths = new ArrayList<>();
          for (String str : strings) {
              lengths.add((Integer) lengthMethod.invoke(str));
          }
          
          System.out.println("字符串长度：" + lengths);
      }
  }
  ```
  
  **4. 考虑使用字节码生成技术**
  
  ```java
  // CGLib动态代理示例
  public class CglibExample {
      
      public static class TargetClass {
          public String sayHello(String name) {
              return "Hello, " + name;
          }
      }
      
      public static void main(String[] args) {
          // 使用CGLib创建代理，避免反射调用
          Enhancer enhancer = new Enhancer();
          enhancer.setSuperclass(TargetClass.class);
          enhancer.setCallback(new MethodInterceptor() {
              @Override
              public Object intercept(Object obj, Method method, Object[] args, 
                      MethodProxy proxy) throws Throwable {
                  System.out.println("方法调用前");
                  // 使用MethodProxy，比反射更快
                  Object result = proxy.invokeSuper(obj, args);
                  System.out.println("方法调用后");
                  return result;
              }
          });
          
          TargetClass proxy = (TargetClass) enhancer.create();
          String result = proxy.sayHello("World");
          System.out.println(result);
      }
  }
  ```
  
  **性能优化总结：**
  
  | 优化策略 | 性能提升 | 适用场景 |
  |----------|----------|----------|
  | **缓存反射对象** | 50-80% | 频繁调用同一方法/字段 |
  | **setAccessible预设置** | 20-30% | 访问私有成员 |
  | **MethodHandle** | 30-50% | Java 7+环境 |
  | **字节码生成** | 80-95% | 复杂的动态代理场景 |
  | **批量操作** | 10-20% | 大量相似操作 |
  
  **最佳实践：**
  
  1. **能不用反射就不用**：优先考虑接口、抽象类等设计模式
  2. **缓存反射对象**：避免重复获取Class、Method、Field
  3. **预设置访问权限**：一次setAccessible，多次使用
  4. **考虑替代方案**：MethodHandle、字节码生成、注解处理器
  5. **性能测试**：在关键路径上进行性能测试和优化
  
  ### 注解相关
  
  **11. 注解的元注解有哪些？**
  
  **答案：**
  
  元注解（Meta-Annotation）是用来注解其他注解的注解，Java提供了几个重要的元注解：
  
  **1. @Retention - 保留策略**
  
  ```java
  // 不同保留策略的注解示例
  
  // SOURCE级别：只在源码中保留，编译后丢弃
  @Retention(RetentionPolicy.SOURCE)
  @Target(ElementType.METHOD)
  public @interface SourceAnnotation {
      String value() default "";
  }
  
  // CLASS级别：保留到字节码，运行时不可见（默认策略）
  @Retention(RetentionPolicy.CLASS)
  @Target(ElementType.TYPE)
  public @interface ClassAnnotation {
      String value() default "";
  }
  
  // RUNTIME级别：运行时可见，可通过反射获取
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.FIELD)
  public @interface RuntimeAnnotation {
      String value() default "";
      int priority() default 0;
  }
  
  // 使用示例
  public class AnnotationExample {
      
      @SourceAnnotation("这个注解只在源码中可见")
      public void sourceMethod() {
          // 编译后@SourceAnnotation会被丢弃
      }
      
      @RuntimeAnnotation(value = "运行时注解", priority = 1)
      private String runtimeField;
      
      public static void main(String[] args) throws Exception {
          // 只能获取到RUNTIME级别的注解
          Field field = AnnotationExample.class.getDeclaredField("runtimeField");
          RuntimeAnnotation annotation = field.getAnnotation(RuntimeAnnotation.class);
          
          if (annotation != null) {
              System.out.println("注解值：" + annotation.value());
              System.out.println("优先级：" + annotation.priority());
          }
      }
  }
  ```
  
  **2. @Target - 使用目标**
  
  ```java
  // 不同目标类型的注解
  
  // 只能用于类、接口、枚举
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface TypeAnnotation {
      String value();
  }
  
  // 只能用于方法
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface MethodAnnotation {
      String description() default "";
      boolean required() default true;
  }
  
  // 只能用于字段
  @Target(ElementType.FIELD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface FieldAnnotation {
      String name() default "";
      Class<?> type() default Object.class;
  }
  
  // 可以用于多个目标
  @Target({ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface MultiTargetAnnotation {
      String value();
  }
  
  // 使用示例
  @TypeAnnotation("这是一个类注解")
  public class TargetExample {
      
      @FieldAnnotation(name = "用户名", type = String.class)
      @MultiTargetAnnotation("字段注解")
      private String username;
      
      @MethodAnnotation(description = "获取用户名", required = true)
      public String getUsername(@MultiTargetAnnotation("参数注解") String defaultValue) {
          return username != null ? username : defaultValue;
      }
  }
  ```
  
  **3. @Documented - 文档包含**
  
  ```java
  // 包含在JavaDoc中的注解
  @Documented
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface DocumentedAnnotation {
      /**
       * 方法描述
       * @return 描述信息
       */
      String description();
      
      /**
       * 作者信息
       * @return 作者名称
       */
      String author() default "Unknown";
  }
  
  // 不包含在JavaDoc中的注解
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface NotDocumentedAnnotation {
      String value();
  }
  
  public class DocumentationExample {
      
      /**
       * 这个方法会在JavaDoc中显示注解信息
       */
      @DocumentedAnnotation(description = "重要方法", author = "Developer")
      public void documentedMethod() {
          // 方法实现
      }
      
      /**
       * 这个方法的注解不会在JavaDoc中显示
       */
      @NotDocumentedAnnotation("隐藏注解")
      public void notDocumentedMethod() {
          // 方法实现
      }
  }
  ```
  
  **4. @Inherited - 继承性**
  
  ```java
  // 可继承的注解
  @Inherited
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface InheritedAnnotation {
      String value();
  }
  
  // 不可继承的注解
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface NotInheritedAnnotation {
      String value();
  }
  
  // 继承示例
  @InheritedAnnotation("父类注解")
  @NotInheritedAnnotation("不可继承注解")
  public class ParentClass {
      // 父类实现
  }
  
  // 子类会继承@InheritedAnnotation，但不会继承@NotInheritedAnnotation
  public class ChildClass extends ParentClass {
      
      public static void main(String[] args) {
          // 检查继承的注解
          Class<ChildClass> clazz = ChildClass.class;
          
          InheritedAnnotation inherited = clazz.getAnnotation(InheritedAnnotation.class);
          NotInheritedAnnotation notInherited = clazz.getAnnotation(NotInheritedAnnotation.class);
          
          System.out.println("继承的注解：" + (inherited != null ? inherited.value() : "无"));
          System.out.println("不可继承注解：" + (notInherited != null ? notInherited.value() : "无"));
      }
  }
  ```
  
  **5. @Repeatable - 可重复注解（Java 8+）**
  
  ```java
  // 容器注解
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Roles {
      Role[] value();
  }
  
  // 可重复注解
  @Repeatable(Roles.class)
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Role {
      String value();
      int priority() default 0;
  }
  
  // 使用可重复注解
  @Role(value = "ADMIN", priority = 1)
  @Role(value = "USER", priority = 2)
  @Role(value = "GUEST", priority = 3)
  public class RepeatableExample {
      
      public static void main(String[] args) {
          Class<RepeatableExample> clazz = RepeatableExample.class;
          
          // 获取重复注解的两种方式
          
          // 方式1：直接获取重复注解数组
          Role[] roles = clazz.getAnnotationsByType(Role.class);
          System.out.println("重复注解数量：" + roles.length);
          for (Role role : roles) {
              System.out.println("角色：" + role.value() + ", 优先级：" + role.priority());
          }
          
          // 方式2：通过容器注解获取
          Roles rolesContainer = clazz.getAnnotation(Roles.class);
          if (rolesContainer != null) {
              System.out.println("\n通过容器注解获取：");
              for (Role role : rolesContainer.value()) {
                  System.out.println("角色：" + role.value() + ", 优先级：" + role.priority());
              }
          }
      }
  }
  ```
  
  **元注解总结表：**
  
  | 元注解 | 作用 | 可选值 | 默认值 |
  |--------|------|--------|--------|
  | **@Retention** | 指定注解保留策略 | SOURCE, CLASS, RUNTIME | CLASS |
  | **@Target** | 指定注解使用目标 | TYPE, METHOD, FIELD等 | 所有目标 |
  | **@Documented** | 是否包含在JavaDoc | 无参数 | 不包含 |
  | **@Inherited** | 是否可被子类继承 | 无参数 | 不可继承 |
  | **@Repeatable** | 指定容器注解 | 容器注解Class | 不可重复 |
  
  **12. 注解的处理时机？**
  
  **答案：**
  
  **1. 编译时处理 - 注解处理器（Annotation Processor）**
  
  ```java
  // 编译时注解
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.SOURCE)
  public @interface Entity {
      String tableName() default "";
  }
  
  @Target(ElementType.FIELD)
  @Retention(RetentionPolicy.SOURCE)
  public @interface Column {
      String name() default "";
      boolean nullable() default true;
  }
  
  // 使用编译时注解
  @Entity(tableName = "users")
  public class User {
      @Column(name = "id", nullable = false)
      private Long id;
      
      @Column(name = "username")
      private String username;
      
      @Column(name = "email")
      private String email;
  }
  
  // 注解处理器
  @SupportedAnnotationTypes({"Entity", "Column"})
  @SupportedSourceVersion(SourceVersion.RELEASE_8)
  public class EntityProcessor extends AbstractProcessor {
      
      @Override
      public boolean process(Set<? extends TypeElement> annotations, 
                           RoundEnvironment roundEnv) {
          
          for (Element element : roundEnv.getElementsAnnotatedWith(Entity.class)) {
              if (element.getKind() == ElementKind.CLASS) {
                  TypeElement classElement = (TypeElement) element;
                  Entity entityAnnotation = classElement.getAnnotation(Entity.class);
                  
                  // 生成SQL创建表语句
                  generateCreateTableSQL(classElement, entityAnnotation);
              }
          }
          
          return true;
      }
      
      private void generateCreateTableSQL(TypeElement classElement, Entity entity) {
          String tableName = entity.tableName().isEmpty() ? 
              classElement.getSimpleName().toString().toLowerCase() : entity.tableName();
          
          StringBuilder sql = new StringBuilder();
          sql.append("CREATE TABLE ").append(tableName).append(" (\n");
          
          List<? extends Element> fields = classElement.getEnclosedElements();
          for (Element field : fields) {
              if (field.getKind() == ElementKind.FIELD) {
                  Column column = field.getAnnotation(Column.class);
                  if (column != null) {
                      String columnName = column.name().isEmpty() ? 
                          field.getSimpleName().toString() : column.name();
                      
                      sql.append("  ").append(columnName)
                         .append(" ").append(getColumnType(field))
                         .append(column.nullable() ? "" : " NOT NULL")
                         .append(",\n");
                  }
              }
          }
          
          sql.setLength(sql.length() - 2); // 移除最后的逗号
          sql.append("\n);");
          
          // 输出生成的SQL
          processingEnv.getMessager().printMessage(
              Diagnostic.Kind.NOTE, "Generated SQL: " + sql.toString());
      }
      
      private String getColumnType(Element field) {
          String typeName = field.asType().toString();
          switch (typeName) {
              case "java.lang.Long":
              case "long":
                  return "BIGINT";
              case "java.lang.String":
                  return "VARCHAR(255)";
              case "java.lang.Integer":
              case "int":
                  return "INT";
              default:
                  return "TEXT";
          }
      }
  }
  ```
  
  **2. 运行时处理 - 反射API**
  
  ```java
  // 运行时注解
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Cacheable {
      String key() default "";
      int expireTime() default 3600; // 秒
  }
  
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Transactional {
      boolean readOnly() default false;
      int timeout() default 30;
  }
  
  // 业务类
  public class UserService {
      
      @Cacheable(key = "user:", expireTime = 1800)
      @Transactional(readOnly = true)
      public User getUserById(Long id) {
          // 模拟数据库查询
          System.out.println("从数据库查询用户：" + id);
          return new User(id, "user" + id, "user" + id + "@example.com");
      }
      
      @Transactional(timeout = 60)
      public void updateUser(User user) {
          // 模拟数据库更新
          System.out.println("更新用户：" + user.getUsername());
      }
  }
  
  // 运行时注解处理器
  public class AnnotationProcessor {
      
      public static Object processAnnotations(Object target, String methodName, 
                                             Object[] args) throws Exception {
          Class<?> clazz = target.getClass();
          Method method = clazz.getMethod(methodName, getParameterTypes(args));
          
          // 处理缓存注解
          Cacheable cacheable = method.getAnnotation(Cacheable.class);
          if (cacheable != null) {
              String cacheKey = cacheable.key() + Arrays.toString(args);
              System.out.println("检查缓存：" + cacheKey + ", 过期时间：" + cacheable.expireTime());
              
              // 模拟缓存逻辑
              Object cachedResult = getFromCache(cacheKey);
              if (cachedResult != null) {
                  System.out.println("从缓存返回结果");
                  return cachedResult;
              }
          }
          
          // 处理事务注解
          Transactional transactional = method.getAnnotation(Transactional.class);
          if (transactional != null) {
              System.out.println("开始事务，只读：" + transactional.readOnly() + 
                               ", 超时：" + transactional.timeout() + "秒");
          }
          
          try {
              // 执行原方法
              Object result = method.invoke(target, args);
              
              // 缓存结果
              if (cacheable != null) {
                  String cacheKey = cacheable.key() + Arrays.toString(args);
                  putToCache(cacheKey, result, cacheable.expireTime());
              }
              
              if (transactional != null) {
                  System.out.println("提交事务");
              }
              
              return result;
              
          } catch (Exception e) {
              if (transactional != null) {
                  System.out.println("回滚事务");
              }
              throw e;
          }
      }
      
      private static Class<?>[] getParameterTypes(Object[] args) {
          Class<?>[] types = new Class[args.length];
          for (int i = 0; i < args.length; i++) {
              types[i] = args[i].getClass();
          }
          return types;
      }
      
      private static Object getFromCache(String key) {
          // 模拟缓存查询
          return null;
      }
      
      private static void putToCache(String key, Object value, int expireTime) {
          System.out.println("缓存结果：" + key + ", 过期时间：" + expireTime + "秒");
      }
      
      public static void main(String[] args) throws Exception {
          UserService userService = new UserService();
          
          // 处理注解并执行方法
          Object result = processAnnotations(userService, "getUserById", new Object[]{1L});
          System.out.println("方法执行结果：" + result);
      }
  }
  ```
  
  **3. 类加载时处理 - 字节码操作**
  
  ```java
  // 使用字节码操作工具（如ASM、Javassist）在类加载时处理注解
  
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface PerformanceMonitor {
      String value() default "";
  }
  
  // 字节码增强示例（使用Javassist）
  public class BytecodeEnhancer {
      
      public static void enhanceClass(String className) throws Exception {
          ClassPool pool = ClassPool.getDefault();
          CtClass ctClass = pool.get(className);
          
          CtMethod[] methods = ctClass.getDeclaredMethods();
          for (CtMethod method : methods) {
              // 检查方法是否有性能监控注解
              if (method.hasAnnotation(PerformanceMonitor.class)) {
                  // 在方法前后添加性能监控代码
                  method.insertBefore(
                      "long startTime = System.currentTimeMillis();" +
                      "System.out.println(\"开始执行方法：\" + \"" + method.getName() + "\");"
                  );
                  
                  method.insertAfter(
                      "long endTime = System.currentTimeMillis();" +
                      "System.out.println(\"方法执行耗时：\" + (endTime - startTime) + \"ms\");"
                  );
              }
          }
          
          // 加载增强后的类
          Class<?> enhancedClass = ctClass.toClass();
          ctClass.detach();
      }
  }
  
  // 使用性能监控注解的类
  public class MonitoredService {
      
      @PerformanceMonitor("业务方法")
      public void businessMethod() {
          try {
              Thread.sleep(100); // 模拟业务处理
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }
      }
  }
  ```
  
  **注解处理时机总结：**
  
  | 处理时机 | 保留策略 | 处理工具 | 优点 | 缺点 | 适用场景 |
  |----------|----------|----------|------|------|----------|
  | **编译时** | SOURCE | 注解处理器 | 无运行时开销，代码生成 | 处理复杂，调试困难 | 代码生成，编译检查 |
  | **运行时** | RUNTIME | 反射API | 灵活，易于实现 | 性能开销，运行时错误 | 框架开发，AOP |
  | **类加载时** | RUNTIME | 字节码操作 | 性能好，功能强大 | 复杂度高，兼容性问题 | 性能监控，代码增强 |
  
  **最佳实践：**
  
  1. **编译时处理**：用于代码生成和编译检查
  2. **运行时处理**：用于框架开发和配置驱动
  3. **类加载时处理**：用于性能敏感的AOP场景
  4. **选择合适的保留策略**：根据处理时机选择对应的保留策略
  5. **注解设计原则**：简单明确，职责单一，易于理解
  
  ## 📖 学习建议
  
  ### 学习顺序
  1. **多线程基础** → **并发工具类** → **线程池** → **性能调优**
  2. **传统IO** → **NIO基础** → **NIO高级特性** → **网络编程**
  3. **反射基础** → **动态代理** → **框架原理理解**
  4. **注解使用** → **自定义注解** → **注解处理器**
  
  ### 实践项目
  1. **多线程爬虫**：练习线程池和并发控制
  2. **NIO聊天室**：练习NIO和Selector
  3. **简单ORM框架**：练习反射和注解
  4. **AOP框架**：练习动态代理
  
  ### 推荐资源
  - 《Java并发编程实战》
  - 《Java NIO》
  - 《深入理解Java虚拟机》
  - Oracle官方文档
  
  ## ⏰ 学习时间规划
  
  - **多线程并发**：2-3个月
  - **IO和NIO**：1-2个月
  - **反射机制**：2-3周
  - **注解技术**：1-2周
  - **综合实践**：1个月
  
  **总计：6-8个月**
  
  ## 🎯 阶段目标
  
  完成本阶段学习后，你应该能够：
  - 熟练使用多线程解决并发问题
  - 理解并应用各种同步机制
  - 掌握NIO进行高性能IO操作
  - 运用反射进行动态编程
  - 设计和使用自定义注解
  - 理解主流框架的底层原理
  
  为下一阶段的框架学习打下坚实基础！
      
      private static void demonstrateBuffer() {
         System.out.println("=== Buffer演示 ===");
        
        // ByteBuffer演示
        System.out.println("\n1. ByteBuffer演示:");
        ByteBuffer buffer = ByteBuffer.allocate(10);
        
        System.out.println("初始状态:");
        printBufferInfo(buffer);
        
        // 写入数据
        buffer.put("Hello".getBytes());
        System.out.println("\n写入'Hello'后:");
        printBufferInfo(buffer);
        
        // 切换到读模式
        buffer.flip();
        System.out.println("\nflip()后（切换到读模式）:");
        printBufferInfo(buffer);
        
        // 读取数据
        byte[] data = new byte[buffer.remaining()];
        buffer.get(data);
        System.out.println("\n读取数据: " + new String(data));
        printBufferInfo(buffer);
        
        // 清空缓冲区
        buffer.clear();
        System.out.println("\nclear()后:");
        printBufferInfo(buffer);
        
        // 演示mark和reset
        System.out.println("\n2. mark和reset演示:");
        buffer.put("ABCDEF".getBytes());
        buffer.flip();
        
        System.out.println("读取前两个字节:");
        System.out.println((char) buffer.get());
        System.out.println((char) buffer.get());
        
        buffer.mark(); // 标记当前位置
        System.out.println("\nmark()后，继续读取:");
        System.out.println((char) buffer.get());
        System.out.println((char) buffer.get());
        
        buffer.reset(); // 回到标记位置
        System.out.println("\nreset()后，重新读取:");
        System.out.println((char) buffer.get());
        System.out.println((char) buffer.get());
        
        // 直接缓冲区演示
        System.out.println("\n3. 直接缓冲区演示:");
        ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);
        System.out.println("是否为直接缓冲区: " + directBuffer.isDirect());
        System.out.println("缓冲区容量: " + directBuffer.capacity());
    }
    
    /**
     * 打印Buffer信息
     */
    private static void printBufferInfo(ByteBuffer buffer) {
        System.out.println(String.format("position=%d, limit=%d, capacity=%d, remaining=%d",
                buffer.position(), buffer.limit(), buffer.capacity(), buffer.remaining()));
    }
    
    /**
     * 演示FileChannel
     */
    private static void demonstrateFileChannel() {
        System.out.println("\n=== FileChannel演示 ===");
        
        Path filePath = Paths.get(BASE_PATH + "channel_demo.txt");
        String content = "这是FileChannel演示内容\nNIO提供了更高效的IO操作\n";
        
        // 写入文件
        try (FileChannel writeChannel = FileChannel.open(filePath, 
                StandardOpenOption.CREATE, StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING)) {
            
            ByteBuffer buffer = ByteBuffer.wrap(content.getBytes(StandardCharsets.UTF_8));
            int bytesWritten = writeChannel.write(buffer);
            System.out.println("写入字节数: " + bytesWritten);
            
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 读取文件
        try (FileChannel readChannel = FileChannel.open(filePath, StandardOpenOption.READ)) {
            
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int bytesRead = readChannel.read(buffer);
            
            buffer.flip();
            String readContent = StandardCharsets.UTF_8.decode(buffer).toString();
            
            System.out.println("读取字节数: " + bytesRead);
            System.out.println("读取内容:\n" + readContent);
            
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 文件复制
        Path sourcePath = filePath;
        Path targetPath = Paths.get(BASE_PATH + "channel_copy.txt");
        
        try (FileChannel sourceChannel = FileChannel.open(sourcePath, StandardOpenOption.READ);
             FileChannel targetChannel = FileChannel.open(targetPath, 
                     StandardOpenOption.CREATE, StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING)) {
            
            long bytesTransferred = sourceChannel.transferTo(0, sourceChannel.size(), targetChannel);
            System.out.println("\n文件复制完成，传输字节数: " + bytesTransferred);
            
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 内存映射文件
        demonstrateMemoryMappedFile(filePath);
    }
    
    /**
     * 演示内存映射文件
     */
    private static void demonstrateMemoryMappedFile(Path filePath) {
        System.out.println("\n内存映射文件演示:");
        
        try (RandomAccessFile file = new RandomAccessFile(filePath.toFile(), "rw");
             FileChannel channel = file.getChannel()) {
            
            // 创建内存映射
            long fileSize = channel.size();
            ByteBuffer mappedBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, fileSize);
            
            System.out.println("文件大小: " + fileSize);
            System.out.println("是否为直接缓冲区: " + mappedBuffer.isDirect());
            
            // 读取映射内容
            byte[] data = new byte[(int) fileSize];
            mappedBuffer.get(data);
            System.out.println("映射文件内容:\n" + new String(data, StandardCharsets.UTF_8));
            
            // 修改映射内容（会直接反映到文件）
            mappedBuffer.position(0);
            mappedBuffer.put("Modified: ".getBytes(StandardCharsets.UTF_8));
            
            System.out.println("内存映射修改完成");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }