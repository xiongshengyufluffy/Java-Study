# 减治系列
# 由2个-->n个/k个

# 二数之和,三数之和,四数之和
- Python通解
  
```
def two_sum(nums, target):
    """假定nums升序
    """
    left = 0
    right = len(nums) - 1
    res = []
    # 左右指针分别移动，收缩区间
    while left < right:
        left_num = nums[left]
        right_num = nums[right]
        tmp_sum = left_num + right_num
        if tmp_sum == target:
            res.append([left_num, right_num])
            # left右移，去掉重复的数字
            while left < right and nums[left] == left_num:
                left += 1
            # right左移，去掉重复的数字
            while left < right and nums[right] == right_num:
                right -= 1
        elif tmp_sum > target:
            right -= 1
        elif tmp_sum < target:
            left += 1
    return res

def n_sum(nums, n, target):
    if len(nums) < n or n < 2:  # 非法
        return []

    if n == 2:  # 基本情况，调用two_sum()
        return two_sum(nums, target)

    i = 0
    res = []
    # 枚举nums[i]，在nums[i+1:]计算n-1 sum的组合
    while i < len(nums) - 1:
        tmp_res = n_sum(nums[i+1:], n-1, target-nums[i])
        for tmp in tmp_res:
            tmp[0:0] = [nums[i]]    # 极速头插法
            res.append(tmp)
        # 跳过第一个重复的值
        while i < len(nums) - 1 and nums[i] == nums[i+1]:
            i += 1
        i += 1  # 移动到未计算遍历的下一个值
    return res

class Solution:
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:
        nums.sort()
        return n_sum(nums, 4, target)
```

## 三数之和

- 测试用例
```
[-1,0,1,2,-1,-4]
[1,1,1]
[0,0,0,0]
[-2,0,0,2,2]
```

- Java实现
```
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> ans = new ArrayList();
        int len = nums.length;
        if(nums == null || len < 3) return ans;
        int target = 0;

        Arrays.sort(nums);
        List<List<Integer>> results = new ArrayList<List<Integer>>();
        for(int i = 0; i < nums.length; i++){
            // System.out.println("unique: " + nums[i]);
            List<List<Integer>> curResults = twoSum(nums, target-nums[i], i+1);
            int curNum = nums[i];
            if(curResults != null){
                // 遍历curResults中的每一个curResult
                for(List<Integer> curResult: curResults){
                    List<Integer> result = new ArrayList<>();
                    result.add(nums[i]);
                    result.addAll(curResult);
                    // System.out.println("result"+result.toString());
                    results.add(result);
                    // System.out.println("results"+results.toString());
                }
                // 不断跳过重复元素
                while ((i < nums.length - 1) && (nums[i+1] == curNum)) i++;
            }
        }
        return results;
    }

    public List<List<Integer>> twoSum(int[] nums, int target, int start) {
        int left = start, right = nums.length - 1;
        if (start == nums.length) return null;
        // System.out.println(Arrays.toString(nums));
        List<List<Integer>> results = new ArrayList<List<Integer>>();
        while (left < right){
            int leftNum = nums[left], rightNum = nums[right];
            int twoSum = leftNum + rightNum;
            if (twoSum == target){
                // Arrays.AsList 返回的是一个不可扩展的静态数组
                // new ArrayList(Arrays.AsList()) 返回的是一个可扩展的动态数组
                results.add(new ArrayList(Arrays.asList(leftNum, rightNum)));
                left++;
                right--;
                while ((left < right)&&(nums[left] == leftNum)) left++;
                while ((left < right)&&(nums[right] == rightNum)) right--;
            }
            else if (twoSum > target) right --;
            else left ++;
        }
        // System.out.println(results.toString());
        return results.size() != 0 ? results : null;
    }   
}
```

## 四数之和
- 测试用例
```
[-3,-2,-1,0,0,1,2,3]
0
--
```

- Java实现
```
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        return NSum(4, nums, target, 0);
    }
 
    public List<List<Integer>> NSum(int n, int[] nums, int target, int start) {
        List<List<Integer>> ans = new ArrayList();
        int len = nums.length;
        if(nums == null || len < n) return ans;
        if(n == 2) return twoSum(nums, target, start);
        // int target = 0;

    
        List<List<Integer>> results = new ArrayList();
        for(int i = start; i < nums.length; i++){
            // System.out.println("unique: " + nums[i]);
            List<List<Integer>> curResults = NSum(n - 1, nums, target-nums[i], i+1);
            int curNum = nums[i];
            if(!curResults.isEmpty()){
                // 遍历curResults中的每一个curResult
                for(List<Integer> curResult: curResults){
                    List<Integer> result = new ArrayList<>();
                    result.add(nums[i]);
                    result.addAll(curResult);
                    // System.out.println("result"+result.toString());
                    results.add(result);
                    // System.out.println("results"+results.toString());
                }
                // 不断跳过重复元素
                while ((i < nums.length - 1) && (nums[i+1] == curNum)) i++;
            }
        }
        return results;
    }

    public List<List<Integer>> twoSum(int[] nums, int target, int start) {
        List<List<Integer>> results = new ArrayList();
        int left = start, right = nums.length - 1;
        if (start == nums.length) return results;
        // System.out.println(Arrays.toString(nums));
        while (left < right){
            int leftNum = nums[left], rightNum = nums[right];
            int twoSum = leftNum + rightNum;
            if (twoSum == target){
                // Arrays.AsList 返回的是一个不可扩展的静态数组
                // new ArrayList(Arrays.AsList()) 返回的是一个可扩展的动态数组
                results.add(new ArrayList(Arrays.asList(leftNum, rightNum)));
                left++;
                right--;
                while ((left < right)&&(nums[left] == leftNum)) left++;
                while ((left < right)&&(nums[right] == rightNum)) right--;
            }
            else if (twoSum > target) right --;
            else left ++;
        }
        // System.out.println(results.toString());
        return results;
    }   
}
```