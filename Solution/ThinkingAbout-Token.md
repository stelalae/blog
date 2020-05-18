# 对Token的思考

阅读本文之前，推荐先阅读下面文章：
* [傻傻分不清之 Cookie、Session、Token、JWT](https://juejin.im/post/5e055d9ef265da33997a42cc)
* [基于token的多平台身份认证架构设计](https://www.cnblogs.com/beer/p/6029861.html)

如上面文章里提到，帐号体系已经成为一个互联网产品的必备基石，往往是一个服务器（或单独的账户服务器），N个客户端的架构。

![20200425](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/s8RUis.png)

下面重点说说Token、JWT的区别：

> 相同：
* 都是访问资源的令牌
* 都可以记录用户的信息
* 都是使服务端无状态化
* 都是只有验证成功后，客户端才能访问服务端上受保护的资源

> 区别：
* Token：服务端验证客户端发送过来的Token时，还需要查询数据库获取用户信息，然后验证 Token 是否有效。
* JWT：将Token和Payload加密后存储于客户端，服务端只需要使用密钥解密进行校验（校验也是JWT自己实现的）即可，不需要查询或者减少查询数据库，因为JWT自包含了用户信息和加密的数据。

常见的前后端鉴权方式：
* Session-Cookie：前后端分离后，使用频率大幅降低；
* Token 验证（包括 JWT，SSO）：中小型项目中常用；
* OAuth2.0（开放授权）：中大型项目中常用，特别是会对外开放的服务；

## OAuth2.0

笔者推荐OAuth2.0，该模型设计成熟，应用广泛，对后端来说有很多现成的第三方包，降低了接入难度。

一般情况下，access_token有效期较短，refresh_token拥有较长的有效期（如一个月、一年等）。而且第三方平台的响应字段里没有refresh_token的过期时间，需关注文档，自己记录refresh_token的过期时间。

*另外，不同开发平台对refresh_token管理不一样，如：豆瓣网、百度云，refresh_token仅能使用一次，因此当用refresh_token更新access_token后，会返回一个新的refresh_token。*

下面摘自微信的开发文档：

```JSON
{ 
    "access_token":"ACCESS_TOKEN", 
    "expires_in":7200, 
    "refresh_token":"REFRESH_TOKEN",
    "openid":"OPENID", 
    "scope":"SCOPE",
    "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
```
> 1、刷新access_token有效期
access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新，access_token刷新结果有两种：
* 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间；
* 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。

> 2、refresh_token拥有较长的有效期（30天），当refresh_token失效的后，需要用户重新授权。

所以正确的登录流程应该是：（以refresh_token有效期30天、access_token有效期2小时为例）
1. 判断是否有refresh_token的时间缓存：
    1. 若无，执行登录流程。
    2. 若有，则判断refresh_token是否过期。
    if 当前时间 - 缓存时间戳 >= 30，说明refresh_token已过期，则重新登录。
2. 若refresh_token没有过期，走判断access_token是否过期：
    1. 读取expire_date，判断access_token是否过期。
    if 当前时间 - expire_date > 2小时，说明已经过期，需要去获取新的access_token和expire_date，把结果缓存到本地。
    2. 若access_token没有过期，走调用接口流程。
3. 利用有效的access_token，进行接口请求，获取不同业务场景数据。

对前端来说，就要思考下面几个问题：
* 执行业务请求时，access_token失效，自动执行refresh_token，携带最新access_token重试之前的业务请求；
* 多业务请求并发访问时，所有请求均失效，保证仅有一次refresh_token操作；
* 对refresh_token进行合理的节流；
* 业务请求+refresh_token合理的降级策略；
* 特殊场景：no refresh_token白名单策略、单点在线业务；

简单的说，当access_token失效时，要自动刷新access_token，并且不能影响正常业务请求，而且要考虑并发刷新的问题。

分析前面的登录流程，发现有2个时间点是可以调整的，那我们是不是可以改变时间点，来降低因token临近过期而引起的程序健壮问题呢？比如现实中90%以上用户会在01:00~04:00期间睡觉为例，那么一个app、web页面等常驻前台不会超过1天的：
* refresh_token过期判断，由`>= 30 改为 >= 28`；
* access_token过期由`固定2小时 改为 变长时间差（第二天凌晨三点 - 当前时间时间）`；

在如IM等可能存在会限制单点在线的业务里，这是单token模式（不限制有效期）会更合适，因为IM往往是长连接模式，而且服务器会记录每个登录设备会记录唯一ID，token失效后服务器会主动通知客户端。

## Token在访客模式中的应用

1. 防止爬虫和机器人，下面摘自[API 接口设计中Token设计讨论](https://www.jianshu.com/p/9fdcfd950292)：
![2630656-ed1961ad7d41ad22](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/yr9aGu.png)

    * 其中签名的算法中的{Session}是通过加密的信息，用于判断当前 页面的请求。
    * {浏览器摘要}中会记录访问者的ip及浏览器信息，用于防止token被不同的机器使用

    使用场景：
    * 媒体临时访问页面，用于防止机器爬虫获取token后无限访问其他界面
    * 动态临时界面，防止机器不断获取动态临时页面信息中的数据

2. 访客模式下的用户行为如何转移到登录用户下
在某个电商产品里，用户小明在访客模式下使用了该产品一段时间，比如搜索了自行车、笔记本电脑等商品，突然某一天注册并登录了产品，这时就需要把之前的浏览器数据关联到小明的账户下，以便后台给小明继续推荐他感兴趣的商品。


参考资料：
* [App之OAuth授权登录、access token、refresh token](https://blog.csdn.net/LVXIANGAN/article/details/78020674)
* [网站应用微信登录开发指南](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)
* [自动刷新token方案](https://juejin.im/post/5b56bbdcf265da0f8524f094)
* [傻傻分不清之 Cookie、Session、Token、JWT](https://juejin.im/post/5e055d9ef265da33997a42cc)
* [基于token的多平台身份认证架构设计](https://www.cnblogs.com/beer/p/6029861.html)
* [API 接口设计中Token设计讨论](https://www.jianshu.com/p/9fdcfd950292)