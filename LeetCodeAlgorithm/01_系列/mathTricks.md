## 204.计算质数

https://leetcode-cn.com/problems/count-primes/solution/ru-he-gao-xiao-pan-ding-shai-xuan-su-shu-by-labula/

```
class Solution {
    public int countPrimes(int n) {
        if (n <= 2) return 0;
        int count = 0;
        boolean[] ifPrimes = new boolean[n];
        Arrays.fill(ifPrimes, true);
        // ifPrimes[0] = false;
        // ifPrimes[1] = false;

        for(int i = 2; i < n; i++){
            if((ifPrimes[i])&&(isPrime(i))){
                count ++;
                for(int j = i; (long)i * j < n; j++){
                    ifPrimes[i*j] = false;
                    // System.out.print("i:"+i+"j:"+j+":"+"i*j"+i*j+" ");
                }
            }
            // System.out.println(" i: " + i + "prime state " + ifPrimes[i]);
        }
        return count;
    }

    public boolean isPrime(int n){
       
        for(int i = 2; (long)i * i < n; i++){
            if(n % i == 0) return false;
        }
        return true;
    }
}
```

## 剪绳子 2


https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/


- 和剪绳子1不同的地方在于需要用到大数运算
```
import java.math.BigInteger;
class Solution {
    public int cuttingRope(int n) {
        BigInteger dp[] = new BigInteger[n + 1];
        Arrays.fill(dp, BigInteger.valueOf(1));
        for (int i = 3; i <= n; i++) {
            for (int j = 1; j < i; j++) {
                dp[i] = dp[i].max(BigInteger.valueOf(j * (i - j))).max(dp[i - j].multiply(BigInteger.valueOf(j)));
            }
        }
        return  dp[n].mod(BigInteger.valueOf(1000000007)).intValue();
    }
}
```
