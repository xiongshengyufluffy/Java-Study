- hashMap 在jdk1.7之前,在扩容时使用头插法来复制链表,但是在多线程时可能造成的问题是形成有环链表
```
void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
            	//获取旧表的下一个元素
                Entry<K,V> next = e.next;
              //
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
              // 这两步是计算新的链表节点插入位置的索引i
              // 为了简便起见,可以考虑i不发生改变
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }

```
- 以上代码可以抽取为
```
while(null != e){
  1: Node next = e.next;
  // compute i
  2: e.next = tables[i];
  3: tables[i] = e;
  4: e = next;
} 
```
# Thread A:
  - 1: next = e.next;
  - 线程a 的 e 指向 key(3), 线程 a 的 next 指向key(7), 准备扩容,此时时间片切换

  - <img width = '40%' height ='40%' style="middle" src ="./photos/hashMap/1.jpg"/> 

# Thread B:

- 此时将原数组old_tables[1]中的链表复制到新的扩容后的tables[3]中

- 循环开始
- 1: next = e.next;

 <img width = '40%' height ='40%' src ="./photos/hashMap/1.jpg"/>

- 2: e.next = tables[3];

<img width = '40%' height ='40%' src ="./photos/hashMap/2.jpg"/>

- 3: tables[3] = e;

<img width = '40%' height ='40%' src ="./photos/hashMap/3.jpg"/>
  
- 4: e = next;  

- 循环结束

- 1: next = e.next;

<img width = '40%' height ='40%' style="middle" src ="./photos/hashMap/4.jpg"/>

  - 2: e.next = tables[3]; 

<img width = '40%' height ='40%' src ="./photos/hashMap/5.jpg"/>

  - 3: tables[3] = e;

<img width = '40%' height ='40%' src ="./photos/hashMap/6.jpg"/>

  - 4: e = next;
  - 循环结束
  - 忽略之前old_table[1]中的引用,此时完成头插法后的tables为

<img width = '40%' height ='40%' src ="./photos/hashMap/7.jpg"/>

  - 时间片切换
# Thread A
- 线程a 的 e 指向key(3), 线程 a 的 next 指向key(7)

<img width = '40%' height ='40%' src ="./photos/hashMap/8.jpg"/>

  - 2: e.next = tables[3];

<img width = '40%' height ='40%' src ="./photos/hashMap/9.jpg"/>

  - 3: tables[3] = e;


<img width = '40%' height ='40%' src ="./photos/hashMap/10.jpg"/>

  - 4: e = next;

<img width = '40%' height ='40%' src ="./photos/hashMap/11.jpg"/>

  - 循环结束
  - 循环开始
  - 1: next = e.next;

<img width = '40%' height ='40%' src ="./photos/hashMap/12.jpg"/>

  - 2: e.next = tables[3];

  - 此时tables[3]中存放的是指向next的引用, 所以在这一步,相当于无事发生(将e的next指针指向next,但是e的next指针本来就指向next)

<img width = '40%' height ='40%' src ="./photos/hashMap/12.jpg"/>

  - 3: tables[3] = e;

<img width = '40%' height ='40%' src ="./photos/hashMap/13.jpg"/>
  
  - 4: e = next;
  - 循环结束
  - 循环开始
  - 1: next = e.next; 此时next 为null

<img width = '40%' height ='40%' src ="./photos/hashMap/14.jpg"/>

  - 2: e.next = tables[3];
  - tables[3]存放的指向key(7)的引用,此时发生环形链表


<img width = '40%' height ='40%' src ="./photos/hashMap/15.jpg"/>











