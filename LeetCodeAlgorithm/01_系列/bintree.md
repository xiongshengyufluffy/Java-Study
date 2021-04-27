# 二叉树的后序遍历


- PS:二叉树的遍历可以自己画一个只有三个节点的二叉树进行模拟
  
**迭代版本**

- 参考题解

https://leetcode-cn.com/problems/binary-tree-postorder-traversal/solution/liang-chong-die-dai-by-powcai/

- **思路**
- 前序遍历:VLR,后序遍历:LRV
- 后序遍历:LRV可以看作是前序遍历序列VLR --> 采取先R后L的遍历方式得到VRL --> 再逆序输出 LRV
```
class Solution:
    def postorderTraversal(self, root: TreeNode) -> List[int]:
        stack = []
        res = []
        while root or stack: # ![当根节点**或者**栈非空,这里使用或者是因为在一开始进入循环的时候栈就是空的]
            while root:
                stack.append(root)
                res.append(root.val)
                root = root.right
            root = stack.pop()
            root = root.left
        return res[::-1]
```
- 二叉树的前序遍历(迭代版本)
- VLR
- 沿着左侧链下行,将左侧链的节点值放入列表中(保证列表中根节点的节点值始V优先),同时将左侧链的节点进栈
- 弹栈,左侧链最末端的节点出栈,如果该节点存在右子树,进入右侧分支
```
class Solution:
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        stack = []
        res = []
        while root or stack: # ![当根节点**或者**栈非空,这里使用或者是因为在一开始进入循环的时候栈就是空的]
            while root:
                stack.append(root)
                res.append(root.val)
                root = root.left
            root = stack.pop()
            root = root.right

        return res
```
- 二刷
  - 本题的关键在于while循环的条件
```
# 迭代版本满足循环的两个条件: 辅助栈非空或者当前节点存在
# (1)沿左侧链下行,访问各个节点,同时将右子树加入栈中
# (2)当左侧链到达末端时,弹栈
class Solution:
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        if not root:
            return []
        stack = []
        result = []
        while stack or root:
            if root:
                result.append(root.val)
                if root.right:
                    stack.append(root.right)
                root = root.left
            else:
                root = stack.pop()
        
        return result
```

# 331. 验证二叉树的前序序列化
```
class Solution {
    char[] ss;
    int length;
    int p;
    
    public boolean isValidSerialization(String preorder) {
         length = preorder.length();
         ss = new char[length + 1];
         for(int i = 0; i < length; i++) ss[i] = preorder.charAt(i);
        ss[length] = ',';
        if(!dfs()) return false;
        return p == ss.length;
    }
    
    public boolean dfs(){
        if(p == length + 1) return false;
        if(ss[p] == '#') { // 表明抵达叶子节点
            p += 2;
            return true;
        }
        while(ss[p] != ',') p++; // 执行之后,ss[p]为','
        p++; // 跳过','
        return dfs() && dfs();
        
    }
}
```
# 二叉树的中序遍历
- 二叉树的中序遍历(迭代版本)

- LVR
- 沿着左侧链下行,将左侧链的节点压入栈中
- 左侧链弹栈,将当前节点的节点值放入列表中
- 如果当前节点存在右分支,转入右侧分支
```
class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        if not root:
            return []
        stack = []
        res = []
        while stack or root:
            while root:
                stack.append(root)
                root = root.left
            root = stack.pop()
            res.append(root.val)
            root = root.right
        return res
```

## 二叉搜索树的最小绝对差
- https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/

# 二叉树的层序遍历
## 116. 填充每个节点的下一个右侧节点指针
```
from collections import deque
class Solution:
    def connect(self, root: 'Node') -> 'Node':
        if not root:
            return root
        if not root.left and not root.right:
            return root
        queue = deque()
        queue.append(root)
        while queue:
            length = len(queue)
            
            last_node = None # 上一个节点(每一层开始对应不存在的首节点)
            for i in range(length):
                node = queue.popleft()
                if last_node: 
                    last_node.next = node
                last_node = node
                if node.left:
                    queue.append(node.left)
                if node.right:
                    queue.append(node.right)
        return root
```
- 如果使用bfs借助队列,则需要消耗额外的O(n)的空间
  - 待分析:为什么是O(n)
- 进阶要求: 使用常数级空间
- 参考链接:
  
https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/solution/bfshe-di-gui-zui-hou-liang-chong-ji-bai-liao-100-2/

- 写法1
```
class Solution:
    def connect(self, root: 'Node') -> 'Node':
        if not root:
            return root
        if not root.left and not root.right:
            return root
        
        node = root # 当前层节点
        cur = None # 用于遍历,连接当前层下一层节点的辅助节点
        first_node = None # 当前层下一层的第一个节点
        while node:
            if node.left:
                if not first_node: first_node = node.left
                if cur:
                    cur.next = node.left
                cur = node.left
            if node.right:
                if not first_node: first_node = node.right
                if cur:
                    cur.next = node.right
                cur = node.right
            node = node.next # 当前层节点右移

            if not node: # 当前层遍历完毕
                node = first_node
                cur = None
                first_node = None
        return root
```

- 写法2
```
class Solution:
    def connect(self, root: 'Node') -> 'Node':
        if not root:
            return root
        if not root.left and not root.right:
            return root
        
        node = root # 当前层节点
        cur = None # 用于遍历,连接当前层下一层节点的辅助节点
        first_node = None # 当前层下一层的第一个节点
        while node:
            if node.left:
                if not first_node: first_node = node.left
                if not cur:
                    cur = node.left
                else:
                    cur.next = node.left
                    cur = cur.next
            
            if node.right:
                if not first_node: first_node = node.right
                if not cur:
                    cur = node.right
                else:
                    cur.next = node.right
                    cur = cur.next

            node = node.next # 当前层节点右移

            if not node: # 当前层遍历完毕
                node = first_node
                cur = None
                first_node = None
        return root
```


# 树和他的孩子节点的关系
## 剑指offer和lc上的衍生题目

# 完全二叉树
## 完全二叉树的性质
https://leetcode-cn.com/problems/count-complete-tree-nodes/

**[递归]**
- 思路:
  - **完全二叉树的性质**:
    - 完全二叉树中任意一个节点,其左子树和右子树都是完全二叉树
  - **满二叉树的性质**:
    - 满二叉树为完全二叉树
    - 从满二叉树的根节点出发,根节点的左子树对应的左侧链长度(左子树高度)和根节点对应的右子树的右侧链长度(右子树高度)相等
`

```
    [1]
   / \
  2   3
 / \  /
4  5 6

[1] 的左子树为
  2  
 / \  
4  5 
为完全二叉树以及满二叉树
-- leftBranchLength == 1(左子树高度 == 1)
-- rightBranchLength == 1(右子树高度 == 1)
-- leftBranchLength == rightBranchLength, 总节点数为 2 ** (height + 1) - 1

[1] 的右子树为
    3
   /
  6
为完全二叉树
-- leftBranchLength == 1(左子树高度 == 1)
-- rightBranchLength == 0(右子树高度 == 0)
-- leftBranchLength != rightBranchLength, 总节点数 = 1 + 左子树总节点数 + 右子树总节点数
```

- **PS**:完全二叉树可以拆成1个根节点 + 满二叉树 + 满二叉树 的形式
```
    [1]
    / \
null  null
-- leftBranchLength == 0(左子树高度 == 0)
-- rightBranchLength == 0(右子树高度 == 0)
-- 总节点数为 2 ** (height + 1) - 1 == 0
```