![![2](42F03A027DE548EB835857C927020EA9)](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/![2](42F03A027DE548EB835857C927020EA9).png)

- 使用CountDownLatch

```
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 执行完毕,马上-1");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + "\t 继续执行相应的逻辑");
    }
}
```

