- 参考链接

https://github.com/labuladong/fucking-algorithm/tree/master/%E7%AE%97%E6%B3%95%E6%80%9D%E7%BB%B4%E7%B3%BB%E5%88%97

### 452. 用最少数量的箭引爆气球
- Java版本
```
class Solution {
    public int findMinArrowShots(int[][] points) {
        int length = points.length;
        if (length < 1) return length;
            
        Arrays.sort(points, new Comparator<int[]>(){
            public int compare(int[] o1, int[] o2){             
                return o1[0] < o2[0] ? -1 : 1;
            }
        });
        int cnt = 1;
        int lastCrossBegin = points[0][0], lastCrossEnd = points[0][1];
        for (int i = 1; i < length; i ++){ 
            int curBegin = points[i][0], curEnd = points[i][1];
            // System.out.println("before   cur_interval:"+curBegin+","+curEnd+"  cross:"+lastCrossBegin+","+lastCrossEnd);
            if (curBegin > lastCrossEnd){
                cnt ++;
                // System.out.println("add");
                lastCrossBegin = curBegin;
                lastCrossEnd = curEnd;
            }else{
                lastCrossBegin = Math.max(lastCrossBegin, curBegin);
                lastCrossEnd = Math.min(lastCrossEnd, curEnd);
            }
            // System.out.println("after  cur_interval:"+curBegin+","+curEnd+"  cross:"+lastCrossBegin+","+lastCrossEnd);
        }
        return cnt;
    }
}
```

- Python版本
```
class Solution:
    def findMinArrowShots(self, points: List[List[int]]) -> int:
        length = len(points)
        if length < 1:
            return length
        # 先按照区间左端点进行排序
        points.sort(key=lambda x:x[0])
        # print(points)
        cnt = 1
        last_cross_begin, last_cross_end = points[0][0], points[0][1] # 交集区间 [1,3] --> [1,3]; [1,3], [2,3] --> [2,3]
        for i in range(1, length):
            cur_begin, cur_end = points[i][0], points[i][1]
            # print("before", "cur_interval:", cur_begin, cur_end, "cross:", last_cross_begin, last_cross_end, end="")
            if cur_begin > last_cross_end:
                cnt += 1
                # print("add interval")
                last_cross_begin, last_cross_end = cur_begin, cur_end
            else:
                # 更新交集区间左右端点
                last_cross_begin, last_cross_end = max(last_cross_begin, cur_begin), min(last_cross_end, cur_end)
            # print()
            # print("after", "cur_interval:", cur_begin, cur_end, "cross:", last_cross_begin, last_cross_end)
        return cnt
```

### 上下车算法