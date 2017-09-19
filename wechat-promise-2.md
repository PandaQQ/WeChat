# WeChat Promise Session 2
Promise的顺序执行和并行执行

#### 前言
小程序最大并发是10个request,所以顺序执行request在小程序里面就十分重要了. 
所以一开始我就在找轮子看看在小程序上有没有已经实现顺序执行request的方法.
然后找到了一个wx-queue-request的一个轮子, 万事大吉!然后点到人家github页面一看使用介绍, 尼玛不会啊!怎么用都不会!! T ^ T.
而且还要导文件, 小程序本来就有2M源文件限制,寸土寸金的地方除非万不得已绝不导包! 抱着这样的原则, 继续找啊找,然后就发现了Promise
如何做顺序执行请求~下面让我我慢慢道来～


#### 参考
* http://sabrinaluo.com/tech/2016/01/23/excecute-parallel-promise-and-sequential-promise/ 
* https://mp.weixin.qq.com/debug/wxadoc/dev/api/api-network.html
* https://github.com/zhengjunxin/wx-queue-request
