# 前缀和
## 一维前缀和
- 数组a的下标从1开始
- 前缀和 $s_{i} = a_{1} + a_{2} + ... + a_{i}, s_{0} = 0$
- 区间a[i]~a[j]之和即为$s_{j} - s_{i-1}$

## 系列题
https://leetcode-cn.com/problems/find-pivot-index/solution/de-liao-qian-zhui-he-na-xie-shi-bei-wo-b-f8q2/
## 134. 加油站

- 前缀和思路
  - Python实现
```
# 累加和
class Solution:
    def canCompleteCircuit(self, gas: List[int], cost: List[int]) -> int:
        # gas - cost 得到到达每个加油站--离开每个加油站的油的净增加
        length = len(gas)
        if length == 1:
            return 0 if gas[0] >= cost[0] else -1
          
        gas_incre = [gas[i] - cost[i] for i in range(length)]
        # gas_incre >= 0 的元素对应的索引,才有可能是可行的起始点
        gas_incre_add = gas_incre.copy()
        # print(gas_incre)
        for i in range(1, length):
            gas_incre_add[i] += gas_incre_add[i-1]
        # print(gas_incre_add)
        # add[i] 累加和数组
        # add[0] A[0]
        # add[1] A[0],A[1] --> A[i] = add[i] - add[i-1] (i != 0) add[i] (i == 0)
        # add[2] A[0],A[1],A[2] --> A[1] + A[2] = add[2] - add[0] --> A[i,j] == add[j] - add[i-1] (if i != 0) add[j] (i==0)
        if_exist = False
        for j in range(length):
            # 尝试从该加油站出发,判断能否绕路行驶一周
            if_j_visible = False
            if gas_incre[j] >= 0:
                if_j_visible = True
                # k从j出发,计算j-k的累加和,如果一直>=0,则表明j为可行的出发点
                # if j == 0: 
                if j == 0:
                # [0]-->[length-1]
                    for k in range(j, length):
                        if gas_incre_add[k] < 0:
                            if_j_visible = False
                            break
                else:# j >= 1
                # [j]-->[length-1] [0]-->[j-1]
                    for k in range(j, length):
                        if gas_incre_add[k] - gas_incre_add[j-1] < 0:
                            if_j_visible = False
                            break
                    # 在[j]-->[length-1]的累加过程中,if_j_visible从来没有被置为过False
                    if if_j_visible:
                        for k in range(j):
                            # [上一段的累加和] + [本段的累加和]
                            if gas_incre_add[length-1] - gas_incre_add[j-1] + gas_incre_add[k] < 0:
                                if_j_visible = False
                                break
            if if_j_visible:
                return j
        return -1
```

- Java实现
```
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int length = gas.length;
        if (length == 1) return gas[0] >= cost[0] ? 0 : -1; 
        int[] gasIncre = new int[length];
        int[] gasIncreAdd = new int[length];
        for (int i = 0; i < length; i++) {
            gasIncre[i] = gas[i] - cost[i];
            gasIncreAdd[i] = i > 0 ? gasIncre[i] + gasIncreAdd[i-1] : gasIncre[i];
        }
        // System.out.println(Arrays.toString(gasIncre));
        // System.out.println(Arrays.toString(gasIncreAdd));
        boolean ifJVisible;
        
        for (int j = 0; j < length; j++) {
            ifJVisible = false;
            if(gasIncre[j] >= 0){
                ifJVisible = true;
                if (j == 0){
                    for (int k = j; k < length; k++) {
                        if(gasIncreAdd[k] < 0) {
                           ifJVisible = false;
                           break;
                        }
                    }
                }else{
                    for (int k = j; k < length; k++) {
                       if(gasIncreAdd[k] - gasIncreAdd[j-1] < 0) {
                           ifJVisible = false;
                           break;
                       }
                    }
                    if(ifJVisible){
                        for (int k = 0; k < j; k++) {
                            if(gasIncreAdd[length-1] - gasIncreAdd[j-1] + gasIncreAdd[k] < 0) {
                                ifJVisible = false;  
                                break;
                            }
                        }
                    }
                }
            }
            // System.out.println(ifJVisible);
            if(ifJVisible) return j;
        }
        return -1;
    }
}
```

## 二维前缀和
- 预处理思路
  - 先按行累加一维前缀和
  - 再按列累加一维前缀和

