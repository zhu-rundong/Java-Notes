# LockSupport

## LockSupport是什么？

LockSupport是用来创建锁和其他同步类的基本线程阻塞原语，其中的park()和unpark(thread)分别是用来阻塞线程和解除阻塞线程。

LockSupport类使用了一种名为Permit（许可）的概念来做到阻塞和唤醒线程的功能， 每个线程都有一个许可(permit)，permit只有两个值1和零，默认是零。可以把许可看成是一种(0,1)信号量（Semaphore），但与 Semaphore 不同的是，许可的累加上限是1。

## 线程等待和唤醒

1. 使用Object中的wait()方法让线程等待，使用Object中的notify()方法唤醒线程
2. 使用JUC包中Condition的await()方法让线程等待，使用signal()方法唤醒线程
3. 使用LockSupport类来中的park()方法来阻塞当前线程以及unpark(thread)方法唤醒指定被阻塞的线程

### Object和Condition使用的限制条件

1. 线程先要获得并持有锁，必须在锁块(synchronized或lock)中
2. 线程必须要先等待后唤醒，线程才能够被唤醒

## 使用示例

```java
public static void lockSupport(){
    Thread a = new Thread(() -> {
        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println(Thread.currentThread().getName() + "\t" + " come in");
        LockSupport.park();
        //LockSupport.park();
        System.out.println(Thread.currentThread().getName() + "\t" + " 被唤醒");
    }, "a");
    a.start();

    new Thread(() -> {
        LockSupport.unpark(a);
        //permit的上限是1，第二个LockSupport.park()被阻塞
        //LockSupport.unpark(a);
        System.out.println(Thread.currentThread().getName()+"\t"+" 发出通知");
    },"b").start();
}
```

