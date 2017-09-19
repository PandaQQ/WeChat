# WeChat Promise 1
如何使用Promise请求更新 Access Token 和 Guest Token

####  前言

在开发小程序之前一直是一个iOS开发者, 还是写Objective-C的, 所以一直会比较抵触类似javascript平台的技术.但这次小程序项目的开发,不单单让我认识到前端的博大精深, 更是让我慢慢开始拥抱React以及React Native方面的技术。小程序开发,让我从一个固执与Native开发慢慢转向大前端方向.

####  实现

获取Access Token 和获取 Guest Token 功能之前在iOS端有实现过一次, 使用的是AFNetworking封装之后可以自动对 401的响应重新发起刷新token的请求之后继续执行401之前最初的请求. 但是实现并不是特别理想, 同时AFNetworking 3.0不支持, 一直使用的是AFNetworking 2.0 .这次有机会写小程序，初次接触到Promise, 就大胆尝试了下. 

一般这个请求是所有request的基类, 所以我将这个请求的方法单独放到一个api.js里面.
所以一些基本的配置信息也就放在这个js文件里面.
```javascript
  ### api.js ###
  
const APIRequestURL = 'http://192.168.101.104/api/v1/';
const ClientID = 'iOS';
const ClientSecert = 'secretiOS';
const GuestTokenRefresh = APIRequestURL + 'tokens/guest';
const AccessTokenRefresh = APIRequestURL + 'tokens/refresh';
```

接下来是获取 Guest Token的两个方法.
```javascript
  /**
  * 封装TMark Refresh Guest Token Request
  */
 function RequestWithRefreshToken(url, data = {}, method = "GET"){
  return new Promise(function (reslove, reject){
    wx.request({
      url: url,
      data: data,
      method: method,
      header:{
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Authorization':wx.getStorageSync('guestToken')
      },
      success:function(res){
        console.log("success");
        if(res.statusCode == 201){
          reslove(res);
        }
        else if(res.statusCode == 401){
          //需要获取guestToken之后才可以登录
          let guestToken = null;
          return RefreshGuestToken().then(res =>{
          
            wx.setStorageSync('guestToken', guestToken);
            return RequestWithRefreshToken(url,data,method);
          }).then((res) =>{
            reslove(res);
          }).catch((err) => {
            reject(err);
          });
        }
        else{
          console.log(res.statusCode);
          reject(res);
        }
      },
      fail:function(err){
        console.log(err);
        reject(err);
      }
    })
  });
}

/**
 * 封装Refresh GuestToken功能
*/
function RefreshGuestToken(){
  return new Promise(function(reslove, reject){
    wx.request({
      url: GuestTokenRefresh,
      data:{
      /*自定义数据*/
      },
      header:{
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      method:'POST',
      success:function(res){
        console.log(res);
        if (res.statusCode == 201) {
          reslove(res);
        }
        else{
          reject(res);
        }
      },
      fail:function(err){
        reject(err);
      }
    })
  });
}
  
```
接下来是获取 Access Token的两个方法.
```javascript
/**
 * 封装 TMark Refresh AccessToken Request 
 */
function RequestWithAccessToken(url, data = {}, method = "GET"){
  return new Promise(function (reslove, reject) {
    wx.request({
      url: url,
      data: data,
      method: method,
      header: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Authorization': wx.getStorageSync('accessToken')
      },
      success: function (res) {
        console.log("success");
        if (res.statusCode == 201) {
          reslove(res);
        }
        else if (res.statusCode == 401) {
          //需要获取accessToken之后才访问
          let accessToken = null;
          return RefreshAccessToken().then(res => {
          
            wx.setStorageSync('accessToken', accessToken);
            return RequestWithAccessToken(url, data, method);

          }).then((res) => {
            reslove(res);
          }).catch((err) => {
            reject(err);
          });
        }
        else if (res.statusCode == 200){
          reslove(res);
        }
        else{
          console.log(res.statusCode);
          reject(res);
        }
      },
      fail: function (err) {
        console.log(err);
        reject(err);
      }
    })
  });
}


/**
 * 封装Refresh AccessToken功能
*/
function RefreshAccessToken() {
  return new Promise(function (reslove, reject) {
    wx.request({
      url: AccessTokenRefresh,
      data: {
        /*自定义数据*/
        "refresh_token":wx.getStorageSync('refreshToken')
      },
      header: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      method: 'POST',
      success: function (res) {
        console.log(res);
        if (res.statusCode == 201) {
          reslove(res);
        }
        else {
          reject(res);
        }
      },
      fail: function (err) {
        reject(err);
      }
    })
  });
}

```
最后是export, 让其他类可以访问Request方法.
```javascript
 module.exports = {
  RequestWithRefreshToken,
  RequestWithAccessToken
 }
```


####  如何使用？

这个就十分方便了.

```javascript
  /*首先引入js文件*/
  const api = requrire('./api.js');
  let info = {};
  let url = '';
  
  //调用 refresh guest token
  api.RequestWithRefreshToken(url, info, "POST").then((res) => {
  
  }).catch((err) => {
     
  });
  //调用 refresh access token
  api.RequestWithAccessToken(url, info, "POST").then((res) => {
  
  }).catch((err) => {
     
  });
  
```
