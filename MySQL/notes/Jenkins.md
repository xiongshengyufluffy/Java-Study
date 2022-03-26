# step1 网址
http://192.168.99.62:8888/

![20210817221133](https://i.loli.net/2021/08/17/iWMv9Ragx5PS8TN.png)

# step2打包


eg.工程hycoupon【trunk】
- tagname:coupon.hongyuan.com
- aimVerson:1.0.0
- aimPName:hongyuan2.0(不能有汉字)
- 发版(和武清有所不同)
  - deploy直接发布到测试环境,目前只有Aries,Gemini,Capricomus
  - build只构建一个镜像(没有办法直接发版到k8s)
    - 所以只能先构建一个镜像
    - 镜像可选环境有58,docker，k8s-leo,k8s-sagittarius,k8s-cancer
    - 当测试需要提发版的包,选择