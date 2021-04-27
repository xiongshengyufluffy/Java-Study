# 注意要点:

- (1) visited的设置位置
- (2) 参数x,y与bfs中pop()出来的点的curx和cury

# 岛屿数量,朋友圈
# 127. 单词接龙

```
输入:
beginWord = "hit",
endWord = "cog",
wordList = ["hot","dot","dog","lot","log","cog"]

输出: 5

解释: 一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog",
     返回它的长度 5。
```
<img src ="./photos/bfs/127.png"/>

- 思路(单向,bfs)
  - 初始化
    - 集合visited,记录当前生成的单词是否满足存在于wordList中以及曾经出现过
    - step = 1
      - 表示需要多少步beginWord --> endWord
  - 将给出的首个单词beginWord加入队列
  - visited.add(beginWord)
  - 启动bfs
    - 对于当前层,共有current_size个元素
      - 对于每一个当前层的元素(即一个字符串,如"")
        - 尝试修改其中的一个字符modify
          - next_word = modify(word):
          - 如果next_word == endWord:
            - return step + 1
          - 否则:
            - 若next_word满足,则visited.add(next_word)
              - (1)在wordList中
              - (2)未在visited中出现过
    - 每一层迭代结束, step = step + 1 
  
```
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        # 如果不将列表转化为集合,会超时
        wordList = set(wordList)
        if endWord not in wordList:
            return 0
        step = 1
        visited = set()
        queue = deque()

        visited.add(beginWord)
        queue.append(beginWord)

        while queue:
            # print(step, queue)
            cur_length = len(queue) # 当前层中单词数量
            for i in range(cur_length):
                cur_word = queue.popleft() # 当前层中对应的单词
                next_words = self.modify(cur_word) # 尝试对当前单词改动一个字符
                # if cur_word == "hit":
                #     print(next_words)
                for next_word in next_words:
                    if next_word != endWord:
                        if next_word in wordList and next_word not in visited:
                            visited.add(next_word)
                            queue.append(next_word)
                    else:
                        return step + 1
            step += 1
        
        return 0
    # 生成一组next_words
    def modify(self, word):
        next_words = []
        length = len(word)
        word_list = list(word) # 将单词转化为列表
        for j in range(length):
            for alpha in string.ascii_lowercase:
                if alpha != word_list[j]:
                    next_word_list = word_list[:j] + [alpha] + word_list[j+1:]
                    next_words.append("".join(next_word_list))
        return next_words
```

# 126.单词接龙
```
class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        vector<vector<string>> res;
        unordered_set<string> dict(wordList.begin(), wordList.end());
        vector<string> p{beginWord};
        queue<vector<string>> paths;
        paths.push(p);
        int level = 1, minLevel = INT_MAX;
        unordered_set<string> words;
        while (!paths.empty()) {
            auto t = paths.front(); paths.pop();
            if (t.size() > level) {
                for (string w : words) dict.erase(w);
                words.clear();
                level = t.size();
                if (level > minLevel) break;
            }
            string last = t.back();
            for (int i = 0; i < last.size(); ++i) {
                string newLast = last;
                for (char ch = 'a'; ch <= 'z'; ++ch) {
                  //修改了一个字符的新的字符串
                    newLast[i] = ch;
                    //如果不存在wordList中,则跳过
                    if (!dict.count(newLast)) continue;
                    words.insert(newLast);
                    vector<string> nextPath = t;
                    nextPath.push_back(newLast);
                    if (newLast == endWord) {
                        res.push_back(nextPath);
                        minLevel = level;
                    } else paths.push(nextPath);
                }
            }
        }
        return res;
    }
};
```