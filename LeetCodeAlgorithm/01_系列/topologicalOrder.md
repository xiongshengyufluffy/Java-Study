# 拓扑排序
- 实现
  - 对于一个有向图,所有的边都是从后往前
  - 当图中存在环的时候,就不存在拓扑排序了
  - 一个图可以进行拓扑排序 <=> 有向无环图
- 实现
  - 入度
    - (1)将所有入度为0的点入队
    - (2)
```
while(q 非空){
    t <- q 的头节点
    for t 的所有出边 t -> j
        -- d[j]
        if(d[j] == 0) 表示j的所有前驱节点已经弹出队列
            q <- j 可以把j加入到队列中
}
```

- 原理:
  - 如果一个图中存在环,则图中任何一个节点的入度不可能为0 --> 该节点无法入队, 该节点链接的边无法被删掉

## 冗余连接
- 拓扑排序做法
- 参考链接

https://leetcode-cn.com/problems/redundant-connection/solution/liang-chong-jie-fa-bing-cha-ji-topopai-x-7haq/
  - 
- 虽然是无向图,但是也可以用拓扑排序解决
- 目标是删除度为1的节点(即为单向边)
- 题目中提供的条件:
  - (1) 给定边edges, 边的长度为N
  - (2) 节点在1 到 N 之间
- 初始化
  - 遍历edges,对于每一条edge
    - **邻接表** neighbor ```map<Integer, Set<Integer>>``` key 表示 节点 a, value 表示 与a相邻的节点集合
    - **访问标记** visited ```visit[N+1][N+1]``` 默认为0,表示两点之间的边尚未被访问过
    - 每个节点的**度数** degree ```degree[N+1]```
- 将所有度数为1的节点入队
- 启动bfs
  - 将队列中的每一个节点出队
  - 遍历该节点的邻接节点集合 neighbor 
    - 每一个元素u和相邻节点v的**度数**分别-1
    - 元素u和相邻节点v的 **访问标记** ```visit[u][v]```设置为true,表示该边已经被访问过
    - 如果相邻节点v的度数为1,则让其入队
- 此时所有度数为1的节点已经被删除
- 剩下的edges中的边此时还未被访问过(```u = edge[0], v = edge[1] visit[u][v] == false```对应的边)则为环中的边
- 题目的要求是返回edges中最后在环中出现的边,则可以逆序遍历edges,找出第一个```u = edge[0], v = edge[1] visit[u][v] == false```的边并返回结果

```
class Solution {
    public int[] findRedundantConnection(int[][] edges) {
        int N = edges.length;
        Map<Integer, Set<Integer>> neighbor = new HashMap<>();
        // 为每一个点初始化一个邻接点的空集合
        for (int i = 1; i <= N; i++) {
            neighbor.put(i, new HashSet<>());
        }
        // 表示边 u -- v 是否被访问过
        boolean[][] visited = new boolean[N+1][N+1]; 
        int[] degree = new int[N+1];

        for(int[] edge: edges){
            int u = edge[0], v = edge[1];
            // 建立邻接关系
            Set<Integer> uNeis = neighbor.get(u), vNeis = neighbor.get(v);
            uNeis.add(v);
            vNeis.add(u);
            degree[u]++;
            degree[v]++;
        }
        Queue<Integer> queue = new LinkedList<>();
        // 将度数为1的点加入队列
        for(int i = 1; i <= N; i++){
            if(degree[i] == 1) queue.add(i);
        }
        while(!queue.isEmpty()){
            // 将当前度数为1的点u出队
            int u = queue.poll();
            Set<Integer> uNeis = neighbor.get(u);
            // 遍历u的邻居v
            for(int v: uNeis){
                // 邻接两点度数 - 1
                degree[u] --;
                degree[v] --;
                // 对边u,v 和 v,u做上访问标记
                visited[u][v] = true;
                visited[v][u] = true;
                // 判断v的度数,如果为1,则将之入队、
                if(degree[v] == 1) queue.add(v);
            }
        }

        // 反向遍历edges
        for(int i = edges.length - 1; i > -1; i--){
            int[] edge = edges[i];
            int u = Math.min(edge[0], edge[1]), v = Math.max(edge[0], edge[1]);
            if(visited[u][v] == false) return new int[] {u, v};
        }
        return null;
    }
}
```