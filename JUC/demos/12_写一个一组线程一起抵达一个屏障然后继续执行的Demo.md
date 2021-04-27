![![2](42F03A027DE548EB835857C927020EA9)](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/![2](42F03A027DE548EB835857C927020EA9).png)

```
public class CyclicBarrierDemo {
    private static CyclicBarrier cyclicBarrier = new CyclicBarrier(6, new afterBarrierGo());
    public static void print(int tid) throws BrokenBarrierException, InterruptedException {
        System.out.println(tid + "\t is ready");
        cyclicBarrier.await();
        System.out.println(tid + "\t go");
    }
    private static class afterBarrierGo implements Runnable{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "\t after barrier go!");
        }
    }
    public static void main(String[] args) {
        for (int i = 1; i <= 6 ; i++) {
            final int tid = i;
            new Thread(()->{
                try {
                    print(tid);
                } catch (BrokenBarrierException | InterruptedException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```