# WeChat Promise 1
如何使用Promise请求更新 Access Token 和 Guest Token

#### 1. 前言

在开发小程序之前一直是一个iOS开发者, 还是写Objective-C的, 所以一直会比较抵触类似javascript平台的技术.但这次小程序项目的开发,不单单让我认识到前端的博大精深, 更是让我慢慢开始拥抱React以及React Native方面的技术。小程序开发,让我从一个固执与Native开发慢慢转向大前端方向.

获取Access Token 和获取 Guest Token 功能之前在iOS端有实现过一次, 使用的是AFNetworking封装之后可以自动对 401的响应重新发起刷新token的请求之后继续执行401之前最初的请求. 但是实现并不是特别理想, 同时AFNetworking 3.0不支持, 一直使用的是AFNetworking 2.0 .这次有机会写小程序，初次接触到Promise, 就大胆尝试了下. 

#### 2. 
