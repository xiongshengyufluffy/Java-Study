# 406. 根据身高重建队列
- 模拟过程
```
原始输入：
[[7,0] [4,4] [7,1] [5,0] [6,1] [5,2]]
sort处理：
[[7 0] [7 1] [6 1] [5 0] [5 2] [4 4]]
遍历people：
===== i=0
   ↓：p[0] 应该在index=0的位置
[[7 0] [7 1] [6 1] [5 0] [5 2] [4 4]]
[[7 0] [7 1] [6 1] [5 0] [5 2] [4 4]] ok

===== i=1
         ↓：p[1]应该在index=1的位置
[[7 0] [7 1] [6 1] [5 0] [5 2] [4 4]]
[[7 0] [7 1] [6 1] [5 0] [5 2] [4 4]] ok

===== i=2
               ↓：p[2]应该在index=1的位置
[[7 0] [7 1] [6 1] [5 0] [5 2] [4 4]]
[[7 0] [6 1] [7 1] [5 0] [5 2] [4 4]] ok

===== i=3
                     ↓：p[3]应该在index=0的位置
[[7 0] [6 1] [7 1] [5 0] [5 2] [4 4]]
[[5 0] [7 0] [6 1] [7 1] [5 2] [4 4]] ok

===== i=4
                           ↓：p[4]应该在index=2的位置
[[5 0] [7 0] [6 1] [7 1] [5 2] [4 4]]
[[5 0] [7 0] [5 2] [6 1] [7 1] [4 4]] ok

===== i=5
                                 ↓：p[5]应该在index=4的位置
[[5 0] [7 0] [5 2] [6 1] [7 1] [4 4]]
[[5 0] [7 0] [5 2] [6 1] [4 4] [7 1]] ok

最终结果：
[[5 0] [7 0] [5 2] [6 1] [4 4] [7 1]]

作者：DCCooper
链接：https://leetcode-cn.com/problems/queue-reconstruction-by-height/solution/gojie-fa-xiang-xi-zhu-shi-xian-pai-xu-zai-cha-ru-s/
```
  - ps:
    - 本道题是按照参考链接给的过程实现的代码
    - 确定people[i]的插入位置idx的实现思路参考链接不同
- Ps:
  - 这道题其实不太涉及插入排序的内容,只不过涉及插入部分
- Python实现
```
class Solution:
    def reconstructQueue(self, people: List[List[int]]) -> List[List[int]]:
        # 实现按照第一列降序,在第一列相等的时候按照第二列升序
        people.sort(key=lambda x:(-x[0],x[1]))
        # 一定要这么写,按照之后的思路,不能写成只按照第一列降序
        # people.sort(key=lambda x:-x[0])
        length = len(people)
        for i in range(length):
            k = people[i][1]  # 比people[i]高的人数个数
            count_k = 0 # 当前循环中比people[i]高的人数个数
            for idx in range(i+1):
                # 用来判断people[i]的插入位置idx
                if count_k < k:
                    if people[idx][0] >= people[i][0]:
                        count_k += 1
                    continue
                # 当前元素为i,插入位置为idx,idx <= i
                # [0][...][idx-1][idx][..][i-1][i]
                # 1. 保留temp = nums[i] 2. nums[idx, i-1] --> nums[idx+1, i] 3.nums[idx] = temp
                temp = people[i]
                while i > idx:
                    people[i] = people[i-1]
                    i -= 1
                people[idx] = temp
                break # 插入完毕结束循环
        
        return people
```
- Java实现

## 1210 复习
```
class Solution {
    public int[] sortArray(int[] nums) {
        // 将nums[i]插入到nums[0,i)中
        for(int i = 1; i < nums.length; i ++){
            int temp = nums[i], j = i;
            while ((j > 0) && (nums[j-1] > temp)){
                nums[j] = nums[j-1];
                j --;
            }
            nums[j] = temp;
        }
        return nums;
    }
}
```