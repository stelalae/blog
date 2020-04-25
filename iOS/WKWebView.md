# WKWebView

## 加载

在H5在App、公众号、小程序内等多端复用的背景下，提高加载速度的思考，内容要点：
* SSR：结合Serverless速度应该非常快！首选方案！
* App离线加载
    * 本地拦截：首先加载index.html，然后拦截js/css文件请求，用原生方式去下载该文件缓存到本地，然后响应被拦截的请求。
    * Zip包：此方式只适合App端。首先管理端配置H5离线包，App端根据入口json文件下载zip包后解压，最后加载。

目前Web开发都会是使用React、Vue等SPA框架，结合流行或定制的脚手架进行开发。如果存在多端复用代码，且两端业务要尽量保持一致的情况下，需要处理更多的优化问题。正常情况下，一般会考虑如下方法：
1. webpack打包文件自带hash值；
2. 独立image等静态资源，分拆js和css，懒加载等方式，尽可能减小文件体积，然后使用CDN分发；
3. 针对重要业务入口页面，使用SSR解决首次加载白屏和等待时间过长；

网上有很多解决SPA加载问题的问题，读者可自行搜索了解。

笔者最近三年主要在医疗行业带领团队负责前端开发，涉及到的终端：
* 患者
    * 患者App - person_app
    * 患者H5（微信公众号、支付宝生活号、其他渠道的H5）- person_h5
* 医生
    * 医生App - doctor_app
    * 医生PC - doctor_pc
* 管理端
    * 管理后台用户端（医院、科室、机构等用户）- admin_pc
    * 管理后台运营端（自己公司的运营和运维）- superadmin_pc

其中几乎所有的用户、大部分业务集中在患者的两个端和医生App，所以终端的优化和兼容性也是集中再次。考虑到机型性能的差一和多端复用情形，所以我为啥推崇SSR，毕竟这是。
    
2. App离线加载的本地拦截，大致分为利用①自定义NSURLProtocol、②代理WKURLSchemeHandler，zip包可以参考蚂蚁金服科技的[支付宝离线包](https://tech.antfin.com/docs/2/59594)。

## 通信

可以直接看[从零收拾一个hybrid框架（一）-- 从选择JS通信方案开始](http://awhisper.github.io/2018/01/02/hybrid-jscomunication/)，里面详细的列出了各种通信方案。

我带过的项目里都是用的假跳转，原因是业务和设计不复杂，Android、iOS、公众号三端都会使用到同一份H5，所以假跳转更适合我。

另外推荐几篇好文：
* [iOS WKWebView的使用及Demo](https://github.com/wsl2ls/WKWebView)
* [\[iOS\]WKWebView的使用--API篇 - 简书](https://www.jianshu.com/p/833448c30d70)
* [WKWebView实现浏览历史恢复](https://oldoldb.com/2019/01/16/Session-restoration/)
* [深入理解JSCore - 美团技术团队（推荐）](https://tech.meituan.com/2018/08/23/deep-understanding-of-jscore.html)