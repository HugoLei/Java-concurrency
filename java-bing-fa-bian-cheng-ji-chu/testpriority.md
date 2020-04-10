# TestPriority

```text
package com.cn.uncle.base.TestJava;
import java.util.concurrent.atomic.AtomicInteger;

public class TestThread {

    private static volatile Integer start = 0;
    private static volatile Integer end = 0;

    public static void main(String[] args) {
        testPriority();
    }

    public static void testPriority() {

        MyThread myThread2 = new TestThread().new MyThread();
        myThread2.setPriority(Thread.MIN_PRIORITY);
        myThread2.start();

        MyThread myThread3 = new TestThread().new MyThread();
        myThread3.setPriority(Thread.NORM_PRIORITY);
        myThread3.start();

        MyThread myThread1 = new TestThread().new MyThread();
        myThread1.setPriority(Thread.MAX_PRIORITY);
        myThread1.start();

        start = 1;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        end = 1;
    }

    public class MyThread extends Thread {
        AtomicInteger count = new AtomicInteger(0);

        @Override
        public void run() {
            while (TestThread.start <= 0) {
            }
            while (TestThread.start > 0) {
                count.addAndGet(1);
                if (TestThread.end > 0) {
                    break;
                }
            }
            System.out.println(Thread.currentThread().getName() + ", priority=" + Thread.currentThread().getPriority()
                    + ", count=" + count.get());
        }
    }
}
```

