# 自顶向下回溯--理解:叶子节点即为所求的解
## n皇后
- 思路1
- main方法
  - 初始化chessbord,visited(表示某个位置是否**被放置过**皇后),putable(表示某个位置是否**可以放置**皇后)
  - 棋盘为n*n
  - 从第0行(start)开始尝试
    - self.backtrack(chessbord,visited,putable,start)
  - 对得到的结果进行格式转换
- backtrack(chessboard,visited,putable,start)
  - 获得解集条件:当start==len(chessboard) (**而非start==len(chessboard)-1**)
    - 即:start在0~len(chessboard)-1都有匹配结果
  - 遍历0~n-1列
    - if not visited[start][col] and putable[start][col]:
      - chessboard[start][col] = "Q"
      - visited[start][col] = True
      - temp_putable 保存putable的临时深拷贝
      - self.change(putable)根据(start,col)修改putable
      - 此时可以尝试在下一行进行放置: self.backtrack(chessbord, visited, putable, start + 1)
      - putable = temp_putable 
      - visited[start][col] = False
      - chessborad[start][col] = "."
- change(putable,x,y):
  - 对于给定的x,y,将和x,y同一
    - 行元素置为False
    - 列元素置为False
    - 同一对角线上的元素置为False,方向↖↘
    - 同一对角线上的元素置为False,方向↙↗
```
import copy
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        chessBoard = [["."] * n for _ in range(n)]
        visited = [[False] * n for _ in range(n)]
        putable = [[True] * n for _ in range(n)]
        start = 0
        self.solutions = []
        self.backtrack(chessBoard, visited, putable, start)
        
        # 修改输出格式
        result = []
        for solution in self.solutions:
            res = []
            for row in solution:
                res.append("".join(row))
            result.append(res)
        return result

        
    
    def backtrack(self, chessBoard, visited, putable, start):
        if start == len(chessBoard):
            self.solutions.append(copy.deepcopy(chessBoard))
            return 
        
        for col in range(len(chessBoard)):
            if not visited[start][col] and putable[start][col]:
                chessBoard[start][col] = "Q"
                visited[start][col] = True
                temp_putable = copy.deepcopy(putable)
                self.change(putable,start,col)
                self.backtrack(chessBoard, visited, putable, start + 1)
                putable = temp_putable
                visited[start][col] = False
                chessBoard[start][col] = "."
    
    # change的逻辑
    def change(self, putable, x, y):
        n = len(putable)
        # (1)x,y所处的同一行上的元素设置为False
        for j in range(n):
            putable[x][j] = False
        # (2)x,y所处的同一列上的元素设置为False
        for i in range(n):
            putable[i][y] = False
        # (3)将x,y所处于同一对角线上的元素设置为False,方向为\
        # [x-1,y-1],[x,y],[x+1,y+1]
        i, j = x, y
        while i >= 0 and j >= 0:
            putable[i][j] = False
            i -= 1
            j -= 1
        i, j = x, y
        while i <= n - 1 and j <= n - 1:
            putable[i][j] = False
            i += 1
            j += 1
        # (4)将x,y所处于同一对角线上的元素设置为False,方向为/
        # [x+1,y-1],[x,y],[x-1,y+1]
        i, j = x, y
        while i >= 0 and j <= n - 1:
            putable[i][j] = False
            i -= 1
            j += 1
        i, j = x, y
        while i <= n - 1 and j >= 0:
            putable[i][j] = False
            i += 1
            j -= 1
```

- 思路2:
  - 在思路1的基础上,使用putable替代visited
  - 不改变change方法
```
import copy
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        chessBoard = [["."] * n for _ in range(n)]
        putable = [[True] * n for _ in range(n)]
        start = 0
        self.solutions = []
        self.backtrack(chessBoard, putable, start)
        
        # 修改输出格式
        result = []
        for solution in self.solutions:
            res = []
            for row in solution:
                res.append("".join(row))
            result.append(res)
        return result

        
    
    def backtrack(self, chessBoard, putable, start):
        if start == len(chessBoard):
            self.solutions.append(copy.deepcopy(chessBoard))
            return 
        
        for col in range(len(chessBoard)):
            if putable[start][col]:
                chessBoard[start][col] = "Q"
                temp_putable = copy.deepcopy(putable)
                self.change(putable,start,col)
                self.backtrack(chessBoard, putable, start + 1)
                putable = temp_putable
                chessBoard[start][col] = "."
```

- 思路3
  - 每个皇后一旦放上一个单元格,则其唯一占据棋盘的一行,一列,一个左对角线↖↘(left_diagonal),一个右对角线(right_diagonal)↙↗
  - 使用集合记录皇后放置过的行,列,左对角线
![NQueen](/leetcode\photos\backtrack\NQueen.png)
  - 从第0行出发,递归的选择没有使用过的列,直到匹配到第n-1行
    - left_diagonal = row - col
    - right_diagonal = row + col
  - 递归终点start==n,即已经匹配完第0-n-1行 
  - solution使用列表维护,其中元素为匹配到第i行时对应的列col
```
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        
        self.n = n
        self.cols = set()
        self.left_diagonals = set()
        self.right_diagonals = set()
        start = 0
        self.solutions = []
        solution = []
        self.backtrack(start, solution)
        # n == 4时
        # self.solutions ==> [[1, 3, 0, 2], [2, 0, 3, 1]]
        result = []
        for solution in self.solutions:
            res = []
            for col in solution:
                temp = ["."] * n
                temp[col] = "Q"
                res.append("".join(temp)) 
            result.append(res)
        return result


        
    
    def backtrack(self, start, solution):
        if start == self.n:
            self.solutions.append(solution.copy())
            return 
        
        for col in range(self.n):
            left_diagonal, right_diagonal = start - col, start + col
            if col not in self.cols \
            and left_diagonal not in self.left_diagonals \
            and right_diagonal not in self.right_diagonals:
                self.cols.add(col)
                self.left_diagonals.add(left_diagonal)
                self.right_diagonals.add(right_diagonal)
                self.backtrack(start + 1, solution + [col])
                self.right_diagonals.remove(right_diagonal)
                self.left_diagonals.remove(left_diagonal)
                self.cols.remove(col)
```
   

# 自底向上回溯--当前回溯的结果需要依赖于叶子节点的回溯结果
```
首先说明一下，这个回溯算法为什么要有布尔返回值：因为本题只要求获得一个结果。


    // 循环：从start 往后拆分
    // 剪枝：满足斐波那契数列条件、满足不以0开头（除非是0）
    // 结束：start==len ans内元素大于2个
    private boolean backtracking(String S, int start, List<Integer> ans) {
        if (start == S.length() && ans.size() > 2) {
            return true;
        }
        for (int i = start; i < S.length(); i++) {
            String segment = S.substring(start, i+1);
            // 剪枝
            // 数值超过范围
            if (Long.parseLong(segment) > Integer.MAX_VALUE) break;
            // 防止开头为 0
            if (!"0".equals(segment) && segment.startsWith("0")) break;
            if (isFibonacciSequence(Integer.valueOf(segment), ans)) {
                ans.add(Integer.valueOf(segment));
                // 找到一个结果就返回
                if (backtracking(S, i+1, ans)) return true;
                ans.remove(ans.size()-1);
            }
        }
        return false;
    }
    // 判断是否能组成斐波那契序列
    private boolean isFibonacciSequence(Integer num, List<Integer> ans) {
        int size = ans.size();
        if (size < 2) return true;
        return (ans.get(size-2) + ans.get(size-1) == num);
    }
    public List<Integer> splitIntoFibonacci(String S) {
        List<Integer> ans =  new ArrayList<>();
        backtracking(S, 0, ans);
        return ans;
    }

作者：caipengbo
链接：https://leetcode-cn.com/problems/split-array-into-fibonacci-sequence/solution/chao-ji-jian-dan-qing-xi-de-javadai-fan-hui-zhi-de/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
- bad case
```
输入
"1011"
预期
[1,0,1,1]
```

- Java代码
```
public class Solution {
    List<Integer> result;
    public List<Integer> splitIntoFibonacci(String S) {
        result = new ArrayList<>();
        backtrack(S, 0);
        return result;
    }

    public boolean backtrack(String S, int start){
        if (start == S.length()) return result.size() > 2;
        for(int i = start; i < S.length(); i++){
            String nextNum = S.substring(start, i + 1);
            if (Long.parseLong(nextNum) > Integer.MAX_VALUE) break; // 超过 2 ** 31 - 1
            if ( (nextNum.length() > 1) && (nextNum.startsWith("0")) ) break; // "01","045"非法,但是单独的"0"是ok的
            int len = result.size();
            if ((len < 2) || ( result.get(len-1) + result.get(len-2) == Integer.parseInt(nextNum) ) ) {
                //回溯结果
                result.add(Integer.parseInt(nextNum));
                if(backtrack(S, i+1)) return true;
                result.remove(result.size()-1);
            }
        }
        return false;
    }
}
```