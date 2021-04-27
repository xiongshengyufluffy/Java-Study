- 代码

```
public class DeadLockDemo {
    private static Object lock1 = new Object();
    private static Object lock2 = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
           synchronized (lock1){
               System.out.println(Thread.currentThread().getName() + "\t get lock1 then want lock2");
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               synchronized (lock2){
                   System.out.println(Thread.currentThread().getName() + "\t get lock2");
               }
           }
        }).start();

        new Thread(() -> {
            synchronized (lock2){
                System.out.println(Thread.currentThread().getName() + "\t get lock2 then want lock1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock1){
                    System.out.println(Thread.currentThread().getName() + "\t get lock1");
                }
            }
        }).start();
    }

}

```
