- trie树
  - 用于快速存储和查找字符串集合的数据结构
![20210414095446](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210414095446.png)

- 性质
  - N 表示单词个数(同时也是树中叶子节点的个数)
  - son[N][26] (以纯英文构成的26个小写单词组成的字符串为例)
    - 其中的26表示每个节点最多连向26个下一个节点
  - cnt[N]
    - 代表该节点对应的字符串在整体字符串集合中出现的数量
  - idx
    - 表示下标是哪一个点
      - 当idx == 0时,既是trie树的根节点,又是空节点
  
- 操作
  - (1) 插入操作

![20210414095357](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210414095357.png)


- 
  - (2) 查询操作

![20210414101411](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210414101411.png)


```
class Trie {
    final static int N = 100010;
    static int[] cnt; // 以某个字符串结尾的数量
    static int[][] son; 
    static int idx;
    /** Initialize your data structure here. */
    public Trie() {
        cnt = new int[N + 1];
        son = new int[N + 1][26];
        idx = 1;
    }
    
    /** Inserts a word into the trie. */
    public void insert(String word) {
        // 从第0层出发
        int p = 0;
        for(int i = 0; i < word.length(); i++){
            int u = word.charAt(i) - 'a';
            if(son[p][u] == 0){
                son[p][u] = idx;
                idx++;
            }
            p = son[p][u];
        }
        cnt[p]++;
    }
    
    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        int p = 0;
        for(int i = 0; i < word.length(); i++){
            int u = word.charAt(i) - 'a';
            if(son[p][u] == 0){
                return false;
            }
            p = son[p][u];
        }
        return cnt[p] != 0;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
        int p = 0;
        for(int i = 0; i < prefix.length(); i++){
            int u = prefix.charAt(i) - 'a';
            if(son[p][u] == 0){
                return false;
            }
            p = son[p][u];
        }
        return p != 0;
    }
}

/**
 * Your Trie object will be instantiated and called as such:
 * Trie obj = new Trie();
 * obj.insert(word);
 * boolean param_2 = obj.search(word);
 * boolean param_3 = obj.startsWith(prefix);
 */
```