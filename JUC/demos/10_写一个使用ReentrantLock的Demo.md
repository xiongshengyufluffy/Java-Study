- 重入锁
  - ReentrantLock
  - synchronized关键字


- ReentrantLock

```


import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;


public class Phone implements Runnable{
    Lock lock = new ReentrantLock();
    @Override
    public void run() {
        open();
    }

    public void open(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\topen phone");
            sendWeChat();
        }finally {
            lock.unlock();
        }
    }

    public void sendWeChat(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\tsend WeChat");
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(phone).start();
        new Thread(phone).start();
    }
}

```


- Synchronized关键字

```
public class Phone {
    public synchronized void open(){
        System.out.println(Thread.currentThread().getName() + "\topen phone");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("\t 3s later");
        sendWeChat();

    }
    public synchronized void sendWeChat(){
        System.out.println(Thread.currentThread().getName() + "\tsendWeChat");
    }

    public static void main(String[] args) {
        Phone phone = new Phone();
//        new Thread(() -> {
//            phone.open();
//        }).start();
        new Thread(phone::open).start();
        new Thread(phone::open).start();
    }
}
```