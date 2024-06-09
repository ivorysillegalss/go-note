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

当实现多线程具体逻辑时，一般都在run()方法中实现执行。但两run并非同一方法。其运行示意图可见下图。当继承Thread为父类，其运行时的start()方法内调用了为native方法的start0()方法。最终运行run()方法。

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