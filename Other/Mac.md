# Mac装机

- 安装代理，开启正确上网模式，保证下载速度；
- 手动安装[Google Chrome](https://www.google.com/intl/zh-CN/chrome/)、[印象笔记](https://yinxiang.com/)、[坚果云](https://www.jianguoyun.com/)，保证LastPass、SwitchyOmega、笔记等信息自动同步；
- 执行`xcode-select --install`安装git，如果报错，就去[developer.apple.com](https://developer.apple.com/download/more/)下载，配置git config和curlrc；
- 登录github，拉去ssh配置：.ssh - 700，rsa_id - 600，rsa_id.pub及authorized_keys - 644，然后使用`ssh -T gitee.com`测试；
- 下载软件：
  - [typora](https://typora.io/)
  - [输入法](https://srf.baidu.com/input/mac.html)，登录帐号，同步字库和配置
  - [QQ](https://im.qq.com/pcqq/)
  - [CleanMyMac](https://www.macwk.com/soft/cleanmymac-x)
  - [iterm2](https://www.iterm2.com/)，关闭警告声音（Preferences -> Profiles -> Terminal -> silence bell）
  - [微信](https://mac.weixin.qq.com/?t=mac&lang=zh_CN)：[WeChatExtension-ForMac](https://github.com/MustangYM/WeChatExtension-ForMac)
  - [uPic](https://github.com/gee1k/uPic)：导入配置
  - [iShot](https://apps.apple.com/cn/app/id1485844094)
  - [XMind](https://www.macwk.com/soft/xmind-2020)
- 开发环境：
  - [brew](https://brew.sh/index_zh-cn)

  - [oh-my-zsh](https://ohmyz.sh/)

  - [yarn](https://www.jianshu.com/p/4611d8f1544f)，官网那个不再适用，见[issue](https://github.com/yarnpkg/website/issues/913)

  - [nvm](https://github.com/nvm-sh/nvm)，安装最新版node

  - [nrm](https://github.com/Pana/nrm)，管理npm的registry，不支持yarn
    - `npm config set registry https://registry.npm.taobao.org`
    - `yarn config set registry https://registry.npm.taobao.org`
    - `npm config get registry`、`yarn config get registry`
    
  - vim，`vim ~/.vimrc`

    ```bash
    set tabstop=2
    set shiftwidth=2
    set expandtab
    set autoindent
    set nu
    set backspace=2
    ```

- 其他：
  - 关闭提醒音量

