```
from collections import deque
class Solution:
    def allCellsDistOrder(self, R: int, C: int, r0: int, c0: int) -> List[List[int]]:
        self.R = R
        self.C = C
        # 思路:
        # 从给定点出发向外bfs,每一次bfs将对应的周围一层的点加入队列
        # 加入队列之前,检查点的坐标是否合法
        # 按层将属于[0, r-1] [0, c-1]范围的点加入到返回列表中
        visited = [[False] * C for _ in range(R)]
        queue = deque()
        poss = [[-1, 0], [1, 0], [0, -1], [0, 1]]
        queue.append((r0, c0))
        visited[r0][c0] = True
        ret = []
        while queue:
            length = len(queue)
            for i in range(length):
                point = queue.popleft()
                x, y = point[0], point[1]
                ret.append([x, y])
                for pos in poss:
                    next_x, next_y = x + pos[0], y + pos[1]
                    if self.check(next_x, next_y) and not visited[next_x][next_y]:
                        queue.append((next_x, next_y))
                        visited[next_x][next_y] = True
        return ret
    
    def check(self, x, y):
        if 0 <= x < self.R and 0 <= y < self.C:
            return True
        return False
```