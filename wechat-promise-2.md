# WeChat Promise Session 2
Promise的顺序执行和并行执行

#### 前言
小程序最大并发是10个request,所以顺序执行request在小程序里面就十分重要了. 
所以一开始我就在找轮子看看在小程序上有没有已经实现顺序执行request的方法.
然后找到了一个wx-queue-request的一个轮子, 万事大吉!然后点到人家github页面一看使用介绍, 尼玛不会啊!怎么用都不会!! T ^ T.
而且还要导文件, 小程序本来就有2M源文件限制,寸土寸金的地方除非万不得已绝不导包! 抱着这样的原则, 继续找啊找,然后就发现了Promise
如何做顺序执行请求~下面让我我慢慢道来～

#### 场景
* 对与Promise的同步请求其实就是把Promise请求放到一个数组里面, [Promise1, Promise2, Promsie3],然后使用使用Promise.all(),统一处理即可.
* 对于Promise的顺序请求,简单来说就是不断then就可以.

假设有个任务场景, 你需要下载一个产品列表, 同时每个产品又需要下载图片、产品资料、产品其他资料信息. 过程可以理解成这样:

* 请求产品列表
* 请求对应产品信息、资料、以及其他资料(对应产品列表中的每一个产品)

```bash
获取产品列表
|-----------------------------| [Product1]
[Product1,Product2,Product3]   |------------------|
                                Product Image
                                Product Info
                                Product Others Info  [Product2]
                                                   |------------------|
                                                    Product Image
                                                    Product Info
                                                    Product Others Info  [Product3]
                                                                        |------------------|
                                                                          Product Image
                                                                          Product Info
                                                                          Product Others Info
```

#### Promise并行执行实现
关于 RequestWithAccessToken()方法可以参考 Promise Session 1
https://github.com/PandaQQ/WeChat/blob/master/wechat-promise-1.md

```javascript

function requestTheProductDetailWithProductID(product_id){

  let product_detail_url = '';
  return api.RequestWithAccessToken(product_detail_url);
}

function requestTheProductImagesWithProductID(product_id){
  
  let product_image_url = '';
  return api.RequestWithAccessToken(product_image_url);
}

function requestTheProductOthersDetailsWithProductID(product_id){

  let product_diamond_detail_url = '';
  return api.RequestWithAccessToken(product_diamond_detail_url);
}

function requestTheProductWithProductID(product_id){

  let getProductDetail = requestTheProductDetailWithProductID(product_id);
  let getProductImages = requestTheProductImagesWithProductID(product_id);
  let getOthersDetails = requestTheProductOthersDetailsWithProductID(product_id);
  let requests = [getProductDetail,getProductImages,getOthersDetails];
  return Promise.all(requests);
}

```

#### Promise顺序执行的正确使用方式

按照顺序执行，容易想到的有以下几种方法：
* then.then.then，从头then到尾…
* then(then(then()))，then的嵌套…
####
promise链的本质其实就是从头then到尾，但是第一种方法怎么用程序来实现，看到实现方法的时候只有一句话，犀利啊！

 ```javascript
 
 function requestToGetProductList(){

  let product_array;
  let url = '';
  api.TRequestWithAccessToken(url).then((res)=>{
  
    product_array = res.data;
    let sequence = Promise.resolve();
    product_array.forEach(function(item){
      
      let product_id = item.product_id;
      sequence = sequence.then(function(){
        
        return requestTheProductWithProductID(product_id);
      }).then((res)=>{
        
        console.log(res);
      });
    });
  }).catch((err)=>{
    
    console.log(err);
  });
}

 ```

#### 参考
* http://sabrinaluo.com/tech/2016/01/23/excecute-parallel-promise-and-sequential-promise/ 
* https://mp.weixin.qq.com/debug/wxadoc/dev/api/api-network.html
* https://github.com/zhengjunxin/wx-queue-request
