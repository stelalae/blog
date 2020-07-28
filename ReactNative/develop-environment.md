# React Native环境踩坑

## 开发环境

### Xcode相关

- ### Unknown argument type '__attribute__' in method

xcode 11内容变更后引起的RN源码运行闪退，找到文件：项目位置/node_modules/react-native/React/Base/RCTModuleMethod.mm. (或者.m）中的`static BOOL RCTParseUnuse`方法替换为下面的内容：

```objective-c
static BOOL RCTParseUnused(const char **input)
{
  return RCTReadString(input, "__unused") ||
         RCTReadString(input, "__attribute__((__unused__))") ||
         RCTReadString(input, "__attribute__((unused))");
}
```



- #### fatal error: 'boost/config/user.hpp' file not found

这种情况大多是boost包下载不完整或者本地解压失败（硬盘空间不够）。

根据国内网络情况，前者情况居多，所以在`ios/Podfile`文件里其他依赖前添加boost-for-react-native的国内下载地址：

```bash
# Uncomment the next line to define a global platform for your project
platform :ios, '9.0'

target 'app' do
  # Uncomment the next line if you're using Swift or would like to use dynamic frameworks
  # use_frameworks!
  
  pod 'boost-for-react-native', :git => 'https://gitee.com/damon-s/boost-for-react-native.git’  
  pod 'yoga', :path => '../node_modules/react-native/ReactCommon/yoga'

end
```

然后重新执行`pod install`即可。

如果pod install在下载boost卡住时，也可用上面办法解决。

## 生产环境





---

参考资料：

- [Xcode11.beta版, 遇到React Native启动报错的问题 getCurrentAppState:error 和 objectAtIndexedSubscript: 的解决方案](https://blog.csdn.net/kuangdacaikuang/article/details/94579312)
- [ Boost for React Native](https://gitee.com/damon-s/boost-for-react-native)

