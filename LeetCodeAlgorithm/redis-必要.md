# Redis
- 当前卡券系统6个站点都是使用哨兵模式，不涉及集群模式

## 依赖的数据类型
### card
- set:用于存卡券的id集合

![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202108302057283.png)

### coupon2c
- String
  - 一类是缓存对象的JSON字符串
  - 一类是"1",比如说用户是否存在过..的状态

![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202108302112773.png)


### couponspt
- hash:某个固定字符串的hash
![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202108302113666.png)

### coupon/coupondw
- list:特权金id的list(感觉list一般都存放池子id)

![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202108302116066.png)


- todo:一些操作
  - setex
  - pipeline

- todo: redis分布式锁的应用场景
- todo: redis分布式自旋锁的应用场景