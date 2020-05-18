# ReactNative工程搭建和改造

[官方教程](https://facebook.github.io/react-native/docs/getting-started)，[中文教程](https://reactnative.cn/docs/getting-started/)。网上也有很多其他资料，可自行搜索。

两种教程里都提到2种方式：沙盒方式（[expo](https://expo.io/)）、与原生混合（Native Code），他们的区别在于是否会编写原生代码。expo方式中集成了部分原生的功能，直接引用expo sdk即可。也可使用`yarn ejected`根据提示对工程进行转原生混合方式，应该不能逆转吧。

考虑到国内对APP的要求，如个性化、性能要求等等，还是选择与原生混合方式较好，方便后期扩展。

下面以**iOS**为例，记录下我的改造过程：

#### 一、工程构建

1、工程目录构建

```
$ yarn global add react-native-cli
$ react-native init doctor_app
$ cd doctor_app
$ yarn install	// 此步可能在init时已经自动执行过了
```

2、iOS添加pod支持

```
$ cd ios
$ pod init
$ vim Podfile	// 更改platform为9.0，可以移除其他未用的target
$ pod install
```

双击打开生成的doctor_app.xcworkspace后，会在doctor_app工程中的Libraries看到React的默认支持。这时iOS工程项目已经初步完成。如果需要添加原生支持的RN三方依赖，以[react-native-device-info](https://github.com/react-native-community/react-native-device-info/)为例

```
$ yarn add react-native-device-info
$ react-native link react-native-device-info	// 推荐用link自动方式，也有其他多种方式，按照库的问题操作即可
$ cd ios
$ pod install
```

最后，就可以在RN代码里react-native-device-info的API，编译后APP能正常运行。

#### 二、[!] React has been deprecated

 在上面`pod install`之后可能会注意到控制台有黄色的提示`[!] React has been deprecated`，同时发现pod过程中安装**React 0.11.0**，但明明package.json里使用的`"react-native": "0.59.8"`。

有强迫症的表示不能接受，哈哈…..如果此时通过pod方式安装[code-push](https://github.com/Microsoft/react-native-code-push)，编译还会报错`React/RCTEventEmitter.h file not found in CodePush.h`，此问题在code-push的issues上有提到很多次。我实际工作中遇到很多类似报RN的头文件未找到的问题。是不是很尴尬！明明装了RN，而且还安装了两个版本。

大致原因就是头文件引用的问题。下面用白话简单阐述，虽说工程构建时添加了*RN版本A*，走的Library形式，但是用pod安装的RN依赖并不能准确找到*RN版本A*的头文件，导致会自动安装一个*RN版本B(即0.11.0)*。这时用pod方式安装的code-push会自动首先找到*RN版本B*，报上面的错误也能理解，毕竟code-push发布时依赖RN版本最低要求0.xx.x（仅是我猜测，未验证！）。

解决方式就是将Library里的RN版本A移到Podfile里。

#### 三、转移React Native到Podfile

即把下面这些Library转移到Podfile里去，这时有没有想起什么？转移原理其实是[集成到现有原生应用](https://facebook.github.io/react-native/docs/integration-with-existing-apps)，只不过转移后还保留了react-native-cli脚手架在Xcode中的配置，比如打包RN代码的脚本等。一般情况下将RN集成到现有原生APP里，在打包APP时需要首先打包RN的离线文件，更详细见[React Native iOS打包详解](https://www.jianshu.com/p/36db88ef118d)。

![16adebab8ea98007](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/nfHuol.jpg)


在Podfile里添加下面的代码，然后pod install即可会安装两个RN版本的问题。

```
  pod 'React', :path => '../node_modules/react-native', :subspecs => [
    'Core',
    'CxxBridge',
    'DevSupport',
    'RCTText',
    'RCTImage',
    'RCTNetwork',
    'RCTWebSocket',
    'RCTAnimation',
    'RCTLinkingIOS',
    'RCTSettings',
    'RCTGeolocation',
    'RCTActionSheet',
    'RCTVibration',
    'RCTBlob',
  ]
    
  pod 'yoga', :path => '../node_modules/react-native/ReactCommon/yoga'
  
  pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
  pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec'
  pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'
```

为了保险起见，将上述截图里的RN子模块全部移除掉**Remove Reference**，不是**Move to Trash**。接着用Xcode编译试试APP看能不能正常运行。

到此，RN工程的基本改造完成，尽情使用Podfile安装RN第三方原生依赖吧。
    
<br/>

**2019-07-11 18:57:57 更新**

*最新的RN版本0.60里关于iOS的部分改动，和我此篇文章改造思路几乎一致，可能Facebook自己也意识到此问题。详情见 [React Native 0.60 新特性](https://juejin.im/post/5d1d7e07e51d4577565367f5)*

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b427e02f29d?w=697&h=225&f=png&s=54768)





