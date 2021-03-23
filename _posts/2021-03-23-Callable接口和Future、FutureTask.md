---
layout:     post
title:      Callable接口和Future、FutureTask
subtitle:   概述
date:       2021-03-23
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - 多线程
---

### Callable接口和Future、FutureTask

​	继承Thread类或者实现Runnable接口可以创建线程，但是这两种方式没有返回值，执行完任务后无法获得返回结果。所以自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。

### Callable和Runnable

先说一下java.lang.Runnable吧，它是一个接口，在它里面只声明了一个run()方法,由于run()方法返回值为void类型，所以在执行完任务之后无法返回任何结果。

~~~java
public interface Runnable {
    public abstract void run();
}
~~~

Callable位于java.util.concurrent包下，它也是一个接口，在它里面也只声明了一个方法，只不过这个方法叫做call()

~~~java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
~~~

可以看到，这是一个泛型接口，call()函数返回的类型就是传递进来的V类型。

　　那么怎么使用Callable呢？一般情况下是配合ExecutorService来使用的，在ExecutorService接口中声明了若干个submit方法的重载版本：

~~~java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
~~~

第一个submit方法里面的参数类型就是Callable。

　　暂时只需要知道Callable一般是和ExecutorService配合来使用的，具体的使用方法讲在后面讲述。

　　一般情况下我们使用第一个submit方法和第三个submit方法，第二个submit方法很少使用。

### Future

Future是一个接口，就是对于具体的Runnable接口或Callable接口任务的执行结果进行取消、查询是否完成、获取结果。必要时可以用get()方法获取执行的结果，该方法会阻塞到任务返回结果。

~~~java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
~~~

- ancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。
- isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。
- isDone方法表示任务是否已经完成，若任务完成，则返回true；
- get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
- get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

也就是说Future有三个功能

- 判断任务是否已经完成
- 中断任务
- 获取任务的执行结果

### FutureTask

Future只是一个接口，无法用来直接创建对象，所以有了FutureTask

~~~java
public class FutureTask<V> implements RunnableFuture<V>
~~~

由上可见，FutureTask实现了RunnableFuture接口

~~~java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
~~~

由上可见，RunnableFuture接口继承了Runnable和Future两个接口



~~~java
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
~~~

上面为FutureTask的两个构造方法

### 使用Callable+Future获取执行结果

~~~java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        executor.shutdown();
         
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
         
        System.out.println("主线程在执行任务");
         
        try {
            System.out.println("task运行结果"+result.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
         
        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
~~~

输出

~~~
子线程在进行计算
主线程在执行任务
task运行结果4950
所有任务执行完毕
~~~



### 使用Callable+FutureTask获取执行结果

~~~java
public class Test {
    public static void main(String[] args) {
        //第一种方式
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        executor.submit(futureTask);
        executor.shutdown();
         
        //第二种方式，注意这种方式和第一种方式效果是类似的，只不过一个使用的是ExecutorService，一个使用的是Thread
        /*Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        Thread thread = new Thread(futureTask);
        thread.start();*/
         
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
         
        System.out.println("主线程在执行任务");
         
        try {
            System.out.println("task运行结果"+futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
         
        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
~~~

