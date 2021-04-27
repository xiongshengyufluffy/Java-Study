# 1625. 执行操作后字典序最小的字符串
- 思路
- (1)累加操作
  - 原始字符串中奇数位的字符串需要累加:
    - 被选中的索引为: 0,[1],2,[3],...
    - 当轮转值b为奇数,那么原始字符串中的奇数索引对应的元素,偶数索引对应的元素都会被选中累加
    - 当轮转值b为偶数,那么原始字符串中的奇数索引对应的元素会被选中累加
      - 再怎么转,原始字符串中的偶数索引对应的元素始终无法被选取到
  
- (2)轮转操作
  - 字符串向右移动b位
    - eg:当```b==1```时
    - ```3456``` --> 右移一位 ```6345```
    - ```|【3456】|3456|``` --> 右移一位 ```|345【6|345】6|```

- (3)如果按照暴力的思路
  - 累加和轮转的次序不影响最终结果 
  - 即可以
    - (3.1)先**只做累加**得到累加的结果集
    - (3.2)再基于累加的结果集,**只做轮转**

- 容易思考得到:
  - (1)累加操作
    - (1.1)累加重复现象
    - 一组数字,基于某个给定的a进行累加,会出现重复
      - 初始值```【(6,5,0)】```,累加值```a==7```
        - 累加1次 ```(3, 2, 7)```
        - 累加2次 ```(0, 9, 4)```
        - 累加3次 ```(7, 6, 1)```
        - 累加4次 ```(4, 3, 8)```
        - 累加5次 ```(1, 0, 5)```
        - 累加6次 ```(8, 7, 2)```
        - 累加7次 ```(2, 1, 6)```
        - 累加8次 ```(9, 8, 3)```
        - 累加9次 ```【(6, 5, 0)】```
          - 一旦发现累加值重复,停止生成累加结果集的迭代
  - (2)轮转操作
    - (2.1)数位问题 
      - 当轮转操作b为奇数时,原字符串中**奇数位,偶数位**都可以进行累加
      - 当轮转操作b为偶数时,原字符串中只有**奇数位**可以累加 
    - (2.1)轮转重复现象
      - 一个字符串,基于某个给定的b进行轮转,会出现重复
        - 初始字符串为```"[105655]"```,轮转移位值```b==2```
          - 轮转1次 ```|1056【55|1056】55|```
          - 轮转2次 ```|10【5655|10】5655|```
          - 轮转3次 ```|【105655】|105655|```
            - 一旦发现轮转结果重复,停止生成轮转结果集的迭代

![925_1](/leetcode\photos\stimulation\1625_1.png)

```
class Solution:
    def findLexSmallestString(self, s: str, a: int, b: int) -> str:
        # 思路,先累加,再进行轮转
        odds = []
        evens = []
        for i in range(len(s)):
            if i % 2 == 1:
                odds.append(int(s[i]))
            else:
                evens.append(int(s[i]))
        # print(odds,evens)
        # 无论b为奇数或者b为偶数,原字符串的奇数位始终要累加
        odds_sets = list(self.accumulation(odds, a))
        # b为奇数,则s的奇数位偶数位都可以进行累加
        if b % 2 == 1:
            evens_sets = list(self.accumulation(evens, a))
        # b为偶数
        else:
            evens_sets = [tuple(evens)]

        min_result = ""
        for odds_set in odds_sets:
            for evens_set in evens_sets:
                # 还原累加后的字符串
                accumulated_str = self._accumulated_str(odds_set, evens_set)
                # 得到旋转集合
                rotated_set = self.rotate(accumulated_str, b)
                cur_min_result = min(rotated_set)
                if not min_result:
                    min_result = cur_min_result
                else:
                    min_result = min(min_result, cur_min_result)
        return min_result



    # 对nums中的每一个元素实现累加
    def accumulation(self, nums, a):

        set_nums = set()
        temps_nums = [(num + a) % 10 for num in nums]

        while True:
            tuple_temps = tuple(temps_nums)
            if tuple_temps not in set_nums:
                set_nums.add(tuple_temps)
            else:
                break
            # 更新累加
            temps_nums = [(num + a) % 10 for num in temps_nums]
        return set_nums

    def _accumulated_str(self, odds_set, evens_set):
        length = len(odds_set) + len(evens_set)
        accumulated_str = ""
        for i in range(length):
            if i % 2 == 0:
                accumulated_str += str(evens_set[i // 2])
            else:
                accumulated_str += str(odds_set[i // 2])
        return accumulated_str

    def rotate(self, accumulated_str, b):

        length = len(accumulated_str)
        temp_set = set()
        temp_set.add(accumulated_str)

        while True:
            accumulated_str = accumulated_str[length-b:]+accumulated_str[:length-b]
            if accumulated_str not in temp_set:
                temp_set.add(accumulated_str)
            else:
                break

        return temp_set
```

# 31.下一个排列

```
...
635214
635241
635412
[635421]
[641235]
641253
641325
641352
641523
641532
642135
642153
642315
...
```
- 分析
  - ```63|5421```的下一个为```64|1235```
    - 其中```5421|```局部降序,```63|```局部降序,```3|5```升序
    - 交换```3```和```4``` -- > ```64|5321```
    - 转置```64|[5321]``` -- > ```64|[1235]```
- 思路
  - 得到```635421```的下一个排序```641235```
  - (1)从倒数第2个元素开始向前遍历用index1开始遍历
    - (1.1)找到当前元素```nums[index1]```>后面元素```nums[index1-1]```的组合
        - nums[1], nums[2] == ["3", "5"],当前元素下标为index1,为nums[1]
      - (1.1.1)找到大于当前元素的最后一个元素，记录其下标index2,该元素为nums[3] == "4"
      - (1.1.2)交换2个数组元素 ```6[3]5[4]21``` --> ```6[4]5[3]21``` ```64|5321```
      - (1.1.3)转置后续元素 ```64|5321``` --> ```64|1235```
      - 找到下一个元素,return 
    - (1.2)```nums[index1] <= nums[index1]```时,index1 --;
  - 若遍历结束,未曾return,则表明nums升序,类似于```123456```,返回转置```123456``

- 代码参考连接
  - https://leetcode-cn.com/problems/next-permutation/solution/java-tong-su-yi-dong-cai-neng-ji-hu-shuang-bai-ji-/
- Java实现
```
class Solution {
    public void nextPermutation(int[] nums) {
        if (nums == null || nums.length < 2) {
            return;
        }

        int length = nums.length;
        int index1 = length - 2;

        /*
            从 倒数第2个元素 开始，向前遍历：
                1、若 当前元素 < 后面的元素(存在下一个更大的排列)：
                    1）找到大于当前元素的最后一个元素，记录其下标
                    2）交换 选中的两个数组元素
                    3）转置后续的元素(保证 后续元素“升序排列”，即 当前排列表示的数“最小”)
                2、反之(不存在下一个更大的排列):
                    则 继续循环
            若 未找到，则 当前排列为 最大排列，转置后返回即可
         */
        while (index1 >= 0) {
            if (nums[index1] < nums[index1 + 1]) {
                int index2 = length - 1;
                /*
                    找到 大于当前元素 的 最后一个元素
                 */
                while (index2 > index1 && nums[index2] <= nums[index1]) {
                    index2--;
                }
                exchange(nums, index1, index2);
                reverse(nums, index1 + 1, length - 1);
                return;
            }
            index1--;
        }
        reverse(nums, 0, length - 1);
    }

    /**
     * 将 指定数组，从 指定开始位置，到 指定结束位置，进行 原地转置
     * @param nums 指定数组
     * @param startIndex 指定开始位置
     * @param endIndex 指定结束位置
     */
    private void reverse(int[] nums, int startIndex, int endIndex) {
        while (startIndex < endIndex) {
            exchange(nums, startIndex++, endIndex--);
        }
    }

    /**
     * 将 指定数组 的 指定下标的两个元素，进行 原地交换
     * @param nums 指定数组
     * @param index1 指定元素下标
     * @param index2 指定元素下标
     */
    private void exchange(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```

```
class Solution:
    def nextPermutation(self, nums):
        length = len(nums)
        index1 = length - 2

        while (index1 >= 0):
            if (nums[index1] < nums[index1 + 1]):
                index2 = length - 1
                while (index2 > index1 and nums[index2] <= nums[index1]):
                    index2 -= 1
                self.exchange(nums, index1, index2)
                self.reverse(nums, index1 + 1, length - 1)
                return
            index1 -= 1

        self.reverse(nums, 0, length - 1)
        return

    def exchange(self, nums, index1, index2):
        nums[index1], nums[index2] = nums[index2], nums[index1]

    def reverse(self, nums, startIndex, endIndex):
        while startIndex < endIndex:
            self.exchange(nums, startIndex, endIndex)
            startIndex += 1
            endIndex -= 1
```