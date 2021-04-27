
# 1207. 独一无二的出现次数
- 思路:两遍遍历
  - (1):先得到每个元素出现次数的字典times_map
  - (2):遍历字典的value,将每个元素出现次数对应的value加入集合values_set中
    - 如果len(times_map) == len(values_set) 返回true
      - 否则返回false
```
from collections import defaultdict
class Solution:
    def uniqueOccurrences(self, arr: List[int]) -> bool:
        times_map = defaultdict(int)
        values_set = set()
        for _ in arr:
            times_map[_] += 1
        
        for key in times_map.keys():
            values_set.add(times_map[key])
        

        return len(times_map) == len(values_set)
```

```
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }
        int[] table = new int[26];
        for (int i = 0; i < s.length(); i++) {
            table[s.charAt(i) - 'a']++;
        }
        for (int i = 0; i < t.length(); i++) {
            table[t.charAt(i) - 'a']--;
            if (table[t.charAt(i) - 'a'] < 0) {
                return false;
            }
        }
        return true;
    }
}
```
```
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }
        Map<Character, Integer> table = new HashMap<Character, Integer>();
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            table.put(ch, table.getOrDefault(ch, 0) + 1);
        }
        for (int i = 0; i < t.length(); i++) {
            char ch = t.charAt(i);
            table.put(ch, table.getOrDefault(ch, 0) - 1);
            if (table.get(ch) < 0) {
                return false;
            }
        }
        return true;
    }
}
```

# 219.存在重复元素Ⅱ

- 思路
  - 使用hashmap来存每个元素的最后一个的出现坐标<元素,坐标>
  - 当遍历到一个新的元素时,检查这个元素是否在之前的hashmap中出现过
    - 如果出现过,则检查当前坐标i - lastidx 是否 <= k 若符合要求,则返回ture
  - 返回false

```
class Solution {
    Map<Integer, Integer> map;
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        map = new HashMap<>();
        int length = nums.length;
        for(int i = 0; i < length; i++){
            int lastIdx = map.getOrDefault(nums[i], -1);
            if(lastIdx == -1){
                // 从未出现过
                map.put(nums[i], i);
            }else{
                if(i - lastIdx <= k) return true;
                else map.put(nums[i], i);
            }
        }
        return false;
    }
}
``` 

# 220.存在重复元素3
- 在一个长度为k的窗口内,寻找和某个数字最接近的一个数
  - 两个方向 
    - 大于等于 k的最小数 --> lower bound
    - < k的最大数
  
![20210417124329](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210417124329.png)


```
class Solution {
    public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
        TreeSet<Long> ts = new TreeSet<>(); 
        // nums中的数字较大,可能存在数据溢出的问题
        for(int i = 0; i < nums.length; i++){
            Long cur = nums[i] * 1L;
            // 找到集合中<= cur的最大值
            Long lower = ts.floor(cur); // 理解为对cur取底
            Long upper = ts.ceiling(cur);
            if(lower != null && cur - lower <= t)  return true;
            if(upper != null && upper - cur <= t) return true;
            ts.add(nums[i] * 1L);
            if(i >= k) ts.remove(nums[i - k] * 1L);
        }
        return false;
    }
}
```