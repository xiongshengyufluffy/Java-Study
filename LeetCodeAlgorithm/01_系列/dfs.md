

## dfs + 二分
### 1631. 最小体力消耗路径
- 数据范围
  - rows == heights.length
  - columns == heights[i].length
  - 1 <= rows, columns <= 100
  - 1 <= heights[i][j] <= 106

- 代码
```
// java实现
class Solution {
    static int M, N;
    static boolean[][] visited;
    public int minimumEffortPath(int[][] heights) {
        M = heights.length;
        N = heights[0].length;
        if(M == N && M == 0) return 0;
       
        int left = 0, right = 1 << 20; // (1 << 20) > 10 ** 6
        while (left < right){
            int mid = left + (right - left) / 2; 
            visited = new boolean[M][N];
            visited[0][0] = true;
            if (dfs(0, 0, mid, heights)) right = mid;   
            else left = mid + 1;
        }
        return left;
    }

    public boolean dfs(int x, int y, int target, int[][] h){
        int[][] ps = {{-1, 0}, {0, -1}, {1, 0}, {0, 1}};
        if (x == M - 1 && y == N - 1) return true;
        for(int i = 0; i < 4; i++){
            int nx = x + ps[i][0], ny = y + ps[i][1];
            if (0 > nx || nx >= M || 0 > ny || ny >= N) continue;
            if (!visited[nx][ny] && target >= Math.abs(h[nx][ny] - h[x][y])) {
                visited[nx][ny] = true;
                if(dfs(nx, ny, target, h)) return true;
            }
        }
        return false;
    }
}
```

# <font color=red>双向dfs</font>

## 5675.最接近目标值的子序列和

给你一个整数数组 ```nums``` 和一个目标值 ```goal``` 。

你需要从 ```nums``` 中选出一个子序列，使子序列元素总和最接近 ```goal``` 。也就是说，如果子序列元素和为 ```sum``` ，你需要 最小化绝对差 ```abs(sum - goal)``` 。

返回 ```abs(sum - goal)``` 可能的 最小值 。

注意，数组的子序列是通过移除原始数组中的某些元素（可能全部或无）而形成的数组。

 

### 示例 1：
```
输入：nums = [5,-7,3,5], goal = 6
输出：0
解释：选择整个数组作为选出的子序列，元素和为 6 。
子序列和与目标值相等，所以绝对差为 0 。
```
### 示例 2：
```
输入：nums = [7,-9,15,-2], goal = -5
输出：1
解释：选出子序列 [7,-9,-2] ，元素和为 -4 。
绝对差为 abs(-4 - (-5)) = abs(1) = 1 ，是可能的最小值。
```

### 示例 3：
```
输入：nums = [1,2,3], goal = -7
输出：7
```

### 提示：

- $1 <= nums.length <= 40$
- $-10^7 <= nums[i] <= 10^7$
- $-10^9 <= goal <= 10^9$

### 分析
- 如果元素个数与数值大小的乘积  $nums.length * goal$在$10^5,10^6$左右,可以使用背包问题来做
- 在本题中具有以下特征
  - 元素个数非常少
    - $1 <= nums.length <= 40$
  - 数值范围非常大
    - $-10^9 <= goal <= 10^9$
  - 如果使用背包问题的解法,则容易超时,因此一般使用暴搜来做
    - 如果$nums.length <= 20$,直接暴搜即可,时间复杂度为$2^{20}$
    - 此时对于本题来说有$nums.length <= 40$,可以使用**双向dfs**

### 思路 

- $1 <= nums.length <= 40$
  - 可以先搜索前$\frac{n}{2}$,再搜索后$\frac{n}{2}$
  - 每一个方案必然是来自前$\frac{n}{2}$个元素中的选择结果,以及来自后$\frac{n}{2}$个元素的选择结果 的结合
    - 可以先将前$\frac{n}{2}$个元素的选法的结果(范围为$2^{20}$)放在一个数组中,并排序,结果集为$S_L$ 
    - 再去搜索后$\frac{n}{2}$个元素的选法的结果(范围为$2^{20}$),结果集为$S_R$
      - 当通过dfs计算出$s_r$时
      - 使用<font color=red>二分</font>从<font color=red>有序</font>的$S_L$中找到距一个$s_l$,使得$s_l, s_r$离$goal$最接近
      - 所谓的<font color=red>**最接近goal**</font> 的$s_l, s_r$组合,当$s_r$固定时,查找到的$s_l$有两种情况,这两种情况其一可能会使得$abs(s_l + s_r - goal)$达到最小
        - 当$s_l$较小时,$s^{'}_{l} + s_r <= goal$($set(s_l)为满足所有s_{l} + s_r <= goal中s_l解集, 而s^{'}_{l}为最大者,如果有多个元素的值都为s^{'}_{l},则返回最靠右的坐标$)
        - 当$s_l$较大时,$s^{''}_{l} + s_r> goal$($set(s_l)为满足所有s_{l} + s_r > goal中s_l解集, 而s^{''}_{l}为最小者,如果有多个元素的值都为s^{''}_{l},则返回最靠左的坐标$)
        - $s^{'}_{l}, s^{''}_{l}$在排序后的$S_L$中相邻
      - 用$abs(s_l + s_r - goal)$来更新最终结果
      - ps: 后$\frac{n}{2}$个元素的选法结果的可以不用存放在数组中,直接用来更新全局结果即可
  
<img src="photos\dfs\5675_1.jpg">

- 整体的时间复杂度可以降为$2^{20} * 20$

### 代码
```
// java实现
class Solution {
    int N = 2100000;
    int[] q;
    int cnt;
    int n;
    int result;
    int target;
    
    public int minAbsDifference(int[] nums, int goal) {
        result = Integer.MAX_VALUE;
        target = goal;
        
        cnt = 0;
        q = new int[N];
        n = nums.length;
        dfs1(nums, 0, 0);
        Arrays.sort(q, 0, cnt);
        dfs2(nums, (n + 1) / 2, 0);
        return result;
    }
    
    public void dfs1(int[] nums, int idx, int Sl){
        if(idx == (n + 1) / 2){
            q[cnt++] = Sl;
            return;
        }
        dfs1(nums, idx + 1, Sl); // 不选nums[idx]
        dfs1(nums, idx + 1, Sl + nums[idx]); // 选择nums[idx]
    }
    
    public void dfs2(int[] nums, int idx, int Sr){
        if(idx == n){
            int r = binSearch(Sr);
            // minAbsdifference可能由Sl <- q[r] 或者 Sl <- q[r+1] 时abs(Sl + Sr - goal)得到
            int Sl = q[r];
            // 更新
            result = Math.min(Math.abs(Sl + Sr - target), result);
            // 当r+1没有越界时
            if(r + 1 < cnt){
                Sl = q[r + 1];
                // 更新
                result = Math.min(Math.abs(Sl + Sr - target), result);
            } 
            return;
        }
        dfs2(nums, idx + 1, Sr);
        dfs2(nums, idx + 1, Sr + nums[idx]);
    }
    
    public int binSearch(int Sr){
        int l = 0, r = cnt - 1;
        while (l < r){
            int mid = l + (r - l + 1) / 2;
            int Sl = q[mid];
            if(Sl + Sr <= target) l = mid;
            else r = mid - 1;
        }
        return r;
    }
    
    
}
```