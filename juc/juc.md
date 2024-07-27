# JUC

创建线程两种方式 及其简易demo：

```java
// 主类
public class JucMain {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start();

        MyThread myThread = new MyThread();
        myThread.start();

        System.out.println("11");
    }
}


// 继承Thread类为父类
public class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println("22");
    }
}


// 实现Runnable接口
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        System.out.println("33");
    }
}
```

当实现多线程具体逻辑时，一般都在run()方法中实现执行。但两run并非同一方法。其运行示意图可见下图。当继承Thread为父类，其运行时的start()方法内调用了为native方法的start0()方法。最终在start0()处理后，通过JVM回调运行run()方法。

而在实现Runnable接口这一方式上，通过构造函数传递多线程任务。跳过了执行start0()等方法的步骤，执行以下的run方法。在对r其简单判空后，调用Runnable.run方法，执行逻辑。

```
public Thread(Runnable task) {
    this(null, null, 0, task, 0, null);
}
```

```
public void start() {
    synchronized (this) {
        // zero status corresponds to state "NEW".
        if (holder.threadStatus != 0)
            throw new IllegalThreadStateException();
        start0();
    }
}
    private native void start0();

```

![image-20240609232939012](C:\Users\chenz\OneDrive\桌面\note\juc\images\image-20240609232939012.png)







![image-20240722150559422](C:\Users\chenz\OneDrive\桌面\note\juc\images\image-20240722150559422.png)

ThreadLocal中的变量和对象是每个线程所特有的，彼此之间不共享。

但是在使用完之后要注意释放回收，因为线程对象是通过强引用指向ThreadLocalMap的。

而在java中，强引用是不会被自动回收的。也就是说这里线程对象的回收时机实际上就是线程的回收时机。若线程不被回收，其ThreadLocal中的对象就会一直存在，造成内存泄漏。

所以应该主动手动调用ThreadLocal的remove方法，手动释放其中的资源。