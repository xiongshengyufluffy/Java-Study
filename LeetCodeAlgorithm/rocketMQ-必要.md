# rocketmq
## 生命周期
### 发送
### 创建时候
- 卡券的创建依赖于活动批次,本地缓存中有维护
  - 如果本地缓存为空,从数据库中查
  - 如果数据库为空,rpc反查
- (状态变更):状态变为"已领取",异步发送状态变更mq,上报**BI**
  - id列表, 操作类型,
  - 先写入对账表
    - 写入对账表时,如果发生异常、日志
  - 再组装、发送MQ
  - 如果发生异常、打印日志
- (通知主卡):如果该卡是**副卡**,即、挂接在主卡下面、可以**领取**、也会**异步**发送一个主卡挂接的福利领取的MQ
- (通知其他调用发【消息中心】, 例如用于后续推送)
- (红点):给用户发送发送红点MQ

### 退款作废
- (通知下游系统):超级回馈卡发送MQ通知客服系统
### 定时任务过期中也会被用到

![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202108302159883.png)


![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202108302200705.png)

- 扫描漏斗表中状态为未使用的记录,获得已过期列表
- 过期后异步发送过期状态变更MQ

#### 同步发送MQ消息、发送失败有补偿

- 发送MQ
  - 存在补偿
    - 没有补偿、则消息可能丢失
    - 实现补偿
      - 将数据添加到缓存/数据库时,如果发生异常,将异常信息插入异常表
  - 存在对账
  - 存在补偿、存在对账

![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202109062241062.png)
##### 补偿
![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202109012132843.png)

```java
/**
     * 特权金红包状态变更时 维护个人资产以及相关缓存数据
     * @author ws 2019/3/12
     * @param changeVO
     * @return
     * @test 有单元测试
     * @author ws
     */
    public boolean refreshCache(CouponChangeVO changeVO, boolean needUpdateDB){
        int opType = changeVO.getOpType();
        if( opType == RedPacketOperateTypeEnum.Create.getKey()){
            return true;
        }

        boolean result = false;
        String changeVOJson = JSONObject.toJSONString(changeVO);
        try{
            //1.根据rdcId从mysql反查全量数据得到rp
            RedPacketCacheVO cacheVO = this.getCacheVOFromDBById(changeVO.getCouponId());

            /*删除红包是否存在缓存 start*/
            List cacheKeys = new ArrayList<Long>();
            cacheKeys.add(cacheVO.getUserId());
            this.deletePageListCacheBatch(cacheKeys, CacheConfCenter.RedPacket.MY_REDPACKET_EXIST.getKey());
            /*删除红包是否存在缓存 end*/

            if(opType== RedPacketOperateTypeEnum.Gain.getKey()){
                //1.分享人获取红包
                String idListKey =  CacheConfCenter.RedBagCash.MY_READY_REDPACKET.getKey()+cacheVO.getUserId();
                //2.更新用户个人资产 和id集合 缓存 设置红包info信息缓存不在此步 在方法最后
                this.refreshCacheOneMarkChange(cacheVO.getUserId(),changeVO.getCouponId(),CacheConfCenter.UserMark.RP1,idListKey,changeVOJson,needUpdateDB);

            }else if(opType==RedPacketOperateTypeEnum.BeGot.getKey()){
                //红包特权金被领取

            }else if(opType==RedPacketOperateTypeEnum.GotEmpty.getKey()){
                //红包特权金被领完
                String readyListKey = CacheConfCenter.RedBagCash.MY_READY_REDPACKET.getKey()+cacheVO.getUserId();
                String emptyListKey =  CacheConfCenter.RedBagCash.MY_EMPTY_REDPACKET.getKey()+cacheVO.getUserId();

                this.refreshCacheTwoMarkChange(cacheVO.getUserId(),changeVO.getCouponId(),CacheConfCenter.UserMark.RP2,CacheConfCenter.UserMark.RP1,emptyListKey,readyListKey,changeVOJson,needUpdateDB);

            }else if(opType==RedPacketOperateTypeEnum.Expire.getKey()){
                //红包已过期
                String readyListKey = CacheConfCenter.RedBagCash.MY_READY_REDPACKET.getKey()+cacheVO.getUserId();
                String expiredListKey =  CacheConfCenter.RedBagCash.MY_EXPIRED_REDPACKET.getKey()+cacheVO.getUserId();

                this.refreshCacheTwoMarkChange(cacheVO.getUserId(),changeVO.getCouponId(),CacheConfCenter.UserMark.RP3,CacheConfCenter.UserMark.RP1,expiredListKey,readyListKey,changeVOJson,needUpdateDB);
                //特权金红包红点消失处理
                redBagCashStateChangeService.dealRedPackStateChangeMQ(cacheVO.getUserId());
            }
            //3.更新特权金红包info 信息 缓存
            String infoKey = CacheConfCenter.RedBagCash.REDPACKET_CACHE_INFO.getKey()+changeVO.getCouponId();
            redisSentinelService.setex(infoKey,CacheConfCenter.RedBagCash.REDPACKET_CACHE_INFO.getExpireTime(),JSONObject.toJSONString(cacheVO));

        }catch( Exception e){
            e.printStackTrace();
            logger.error("特权金红包：个人资产设置失败,changeVO=" + JSONObject.toJSONString(changeVO)+"exception，"+e);

            throw new ByException(ErrorCode.SysError.REDIS_ERROR, "特权金红包：红包缓存或个人资产设置失败");

        }
        return result;
    }
```
- refreshCacheOneMarkChange
  - 如果缓存中存在idListKey
    - 将<idLiistKey, couponId>添加到缓存中
    - 如果发生异常
      - **dealException**

```java
/**
     *
     * 功能描述:
     * 举例：当分享人获得特权金红包时，状态直接变为未使用，需要操作1个id集合
     * @param:
     * @return:
     * @auther: ws
     * @date: 2019/2/19 15:27
     * @test 有被集成测试
     * @author ws
     */
    public void refreshCacheOneMarkChange(Long userId,Long couponId,String field,String idListKey,String changeVOJson,boolean needUpdateDB){
        Map needDBOpr = new HashMap();
        needDBOpr.put(field,"+1");
        boolean idListExist = redisSentinelService.exists(idListKey);
        if(idListExist){
            //cacheIds 设置uid到券或红包id的映射
            try{
                redisSentinelService.sadd(idListKey,couponId.toString());
            }catch (Exception e){
                this.dealException(true,changeVOJson);
                throw e;
            }
        }else{
        }

    }
```

- dealException是抽象的,有两个实现类
  - 其中一个是RedPacketCacheServiceImpl.dealException
    - 根据isDbException标志
      - 将来自于数据库的异常信息或者来自于redis的异常信息插入到异常表中,返回true

```java
  public boolean dealException(boolean isDbException,String changeVOJson){
        if(isDbException){
            exceptionService.insertException(changeVOJson, ExceptionTypeEnum.RED_PACKET_STATUS_MARK_FAILED_DB.getKey(), "特权金：状态标识设置失败,db exception，changeVO=" + changeVOJson);
        }else{
            exceptionService.insertException(changeVOJson, ExceptionTypeEnum.RED_PACKET_STATUS_MARK_FAILED_REDIS.getKey(), "特权金：状态标识设置失败,redis exception，changeVO=" + changeVOJson);
        }
        return true;
    }
```

- 对异常的补偿处理使用定时任务实现
![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202109012224681.png)

```java
/**
 * 处理异常任务
 * @author Guoxiong
 * @date 2017/12/15.
 */
@DisallowConcurrentExecution
@PersistJobDataAfterExecution
public class ExceptionDealJob implements Job{

    private static final Logger LOGGER = LoggerManager.getLogger(ByConstants.timerJob);

    IExceptionHandlerService exceptionHandlerService = SpringContextHolder.getBean("exceptionHandlerServiceImpl",ExceptionHandlerServiceImpl.class);

    /**
     * 定时处理异常
     * @param jobExecutionContext
     * @throws JobExecutionException
     */
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        String ip = IpUtil.getXForwardedFor();
        LOGGER.error("[异常处理服务][current node ip = {}],异常处理服务开始",ip);
        try{
            //调用异常处理service进行异常处理
            exceptionHandlerService.handle();
        }catch(Exception e){
            LOGGER.error("[异常处理服务] 异常处理服务错误", e);
        }
        LOGGER.error("[异常处理服务][current node ip = {}] 异常处理服务结束",ip);
    }
}

```

# 见couponspt

![](https://gitee.com/fluffyball/blogimage/raw/master/images/localPC/202109062244177.png)