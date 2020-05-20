# iOS中位置权限选项的区别

现在隐私保护是越来越重视，获取地理位置难度+++，APP Store在审核时容易发生因位置权限配置不对导致被拒。目前iOS的 `info.plist` 里有4个位置权限设置项及对应版本：
* Privacy - Location Usage Description：iOS 6.0–8.0
* Privacy - Location When In Use Usage Description：iOS 11.0+
* Privacy - Location Always Usage Description：iOS 8.0–10.0
* Privacy - Location Always and When In Use Usage Description：iOS 11.0+

以下测试分别在iOS的 12.3.1、13.1.2 环境下进行，下文中分别简称iOS12、iOS13。

## 详细说明

### 1. Privacy - Location Usage Description

[官网说明](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocationusagedescription), 适用于iOS 6.0–8.0，所以新项目中不再使用，写不写都无所谓。

### 2. Privacy - Location When In Use Usage Description

只允许使用应用期间APP获取地理位置，[官网说明](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocationwheninuseusagedescription)，适用于iOS 11.0+。

**在iOS 12中**
* 用户提示框
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/ZWOwNY.jpg) 
* 设置菜单
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/lvZ1xd.jpg)

**在iOS 13中**
* 用户提示框
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/tpWq7a.jpg) 
* 设置菜单
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/t5aI3h.jpg)

### 3. Privacy - Location Always Usage Description 和 Privacy - Location Always and When In Use Usage Description

`Privacy - Location Always Usage Description`和`Privacy - Location Always and When In Use Usage Description`，都是申请允许APP在前后台时都能获取地理位置，下面来看看具体区别。

先看官网的说明：
* [NSLocationAlwaysUsageDescription](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocationalwaysusagedescription)，iOS 8.0–10.0，已废弃，即兼容iOS 11以前的需要配置这个。

> Use this key if your iOS app accesses location information in the background, and you deploy to a target earlier than iOS 11. In that case, add both this key and NSLocationAlwaysAndWhenInUseUsageDescription to your app’s Info.plist file with the same message. Apps running on older versions of the OS use the message associated with NSLocationAlwaysUsageDescription, while apps running on later versions use the one associated with NSLocationAlwaysAndWhenInUseUsageDescription.
> If your app only needs location information when in the foreground, use NSLocationWhenInUseUsageDescription instead. For more information, see Choosing the Location Services Authorization to Request.
> If you need location information in a macOS app, use NSLocationUsageDescription instead.

* [NSLocationAlwaysAndWhenInUseUsageDescription](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocationalwaysandwheninuseusagedescription)，iOS 11.0+，即是用来在iOS 11及以后代替`NSLocationAlwaysUsageDescription`的。

> Use this key if your iOS app accesses location information while running in the background. If your app only needs location information when in the foreground, use NSLocationWhenInUseUsageDescription instead. For more information, see Choosing the Location Services Authorization to Request.
> If you need location information in a macOS app, use NSLocationUsageDescription instead. If your iOS app deploys to versions earlier than iOS 11, see NSLocationAlwaysUsageDescription.

**在iOS 13中**
* 设置菜单
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/HL5wbd.jpg)

## 结论

* 如果app需要前台运行定位权限，需要配置NSLocationWhenInUseUsageDescription；
* 如果app需要后台运行定位权限，需要配置NSLocationAlwaysAndWhenInUseUsageDescription；（如果适配iOS 11之前版本，还需要配置NSLocationAlwaysUsageDescription）

官方建议：[Choosing the Location Services Authorization to Request](https://developer.apple.com/documentation/corelocation/choosing_the_location_services_authorization_to_request)
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/H5EFJK.png)
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/lZyNXx.jpg)

2020年05月上架一个新包，之前按照别人文档来囫囵吞枣的把位置权限全部加上，于是收到被拒的大礼包：
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/18/Zhe5M4.jpg)
后经大佬指点，被拒的真正原因是因为xcode 11里Location updates被勾选了，去掉重新提交审核就好了。
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/20/RZdJLT.png)
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/20/7RlP7c.png)

-------
参考资料：
* [iOS中各种类型的位置权限的选项有什么区别](https://www.codenong.com/js54242b509c85/)
* [\[资源分享\] iOS定位权限请求时易犯的错误小结](https://iambigboss.top/post/54258_1_1.html)
