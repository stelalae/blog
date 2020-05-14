# UIWebview ITMS-90809: Deprecated API Usage

> ITMS-90809: Deprecated API Usage - New apps that use UIWebView are no longer accepted. Instead, use WKWebView for improved security and reliability. Learn more (https://developer.apple.com/documentation/uikit/uiwebview).

* 2020年4月起App Store将不再接受使用UIWebView的新App上架；
* 2020年12月起将不再接受使用UIWebView的App更新；

解决步骤：
1. 全局搜索UIWebview，将相关代码删除或者替换。
2. 终端cd到要检查的项目根目录，然后执行`grep -r UIWebView .`
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/eUihU4.png)

像上面就是极光认证的第三方库里面包含UIWebView的API，于是在Podfile中`pod 'JVerification', '2.5.3'`改为`pod 'JVerification', '2.6.2'`，然后再次打包上传，问题解决！

补充，在项目根目录下终端执行`find . -type f | grep -e ".a" -e ".framework" | xargs grep -s UIWebView`也可以查找哪些SDK包含了UIWebView。