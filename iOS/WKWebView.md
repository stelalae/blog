# WKWebView

## 加载

在H5在App、公众号、小程序内等多端复用的背景下，提高加载速度的思考。
* SSR：结合Serverless速度应该非常快！首选方案！
* App离线加载
    * 本地拦截：首先加载index.html，然后拦截js/css文件请求，用原生方式去下载该文件缓存到本地，然后响应被拦截的请求。
    * Zip包：此方式只适合App端。首先管理端配置H5离线包，App端根据入口json文件下载zip包后解压，最后加载。

1. 针对目前常用的SPA框架，利用①webpack打包文件自带hash值、②app启动时预加载webview内容。考虑到多端复用的情形，SSR应该是首选，在成本和效益之间平衡！
2. App离线加载的本地拦截，大致分为利用①自定义NSURLProtocol、②代理WKURLSchemeHandler，zip包可以参考蚂蚁金服科技的[支付宝离线包](https://tech.antfin.com/docs/2/59594)。

## 通信

可以直接看[从零收拾一个hybrid框架（一）-- 从选择JS通信方案开始](http://awhisper.github.io/2018/01/02/hybrid-jscomunication/)，里面详细的列出了各种通信方案。

我带过的项目里都是用的假跳转，原因是业务和设计不复杂，Android、iOS、公众号三端都会使用到同一份H5，所以假跳转更适合我。

另外推荐几篇好文：
* [iOS WKWebView的使用及Demo](https://github.com/wsl2ls/WKWebView)
* [\[iOS\]WKWebView的使用--API篇 - 简书](https://www.jianshu.com/p/833448c30d70)
* [WKWebView实现浏览历史恢复](https://oldoldb.com/2019/01/16/Session-restoration/)
* [深入理解JSCore - 美团技术团队（推荐）](https://tech.meituan.com/2018/08/23/deep-understanding-of-jscore.html)