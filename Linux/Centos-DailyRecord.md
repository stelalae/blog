# Centos日常记录

[TOC]

## 配置

### Activate the web console with: systemctl enable --now cockpit.socket

[Cockpit](https://github.com/cockpit-project/cockpit)是一个Web端的系统管理工具，只用鼠标点点就能管理系统，事实上也确实如此，我实际使用来说，启动Cockpit服务之后，只需要鼠标点点点就能完成系统很多基础操作，比如查看系统信息，启动/停止服务，新增或者更改账户，系统更新，Web终端及查看网络流量等功能。

```bash
$ ll /etc/motd.d/
总用量 0
lrwxrwxrwx. 1 root root 17 4月  24 2020 cockpit -> /run/cockpit/motd

$ cat /etc/motd.d/cockpit
Activate the web console with: systemctl enable --now cockpit.socket

$ rm /etc/motd.d/cockpit  # 删除改文件即可取消提示
```

想对于我这种Linux菜鸡来说，Cockpit （默认端口9090）应该有很大帮助的，不用再去查找各种命令。

```bash
$ systemctl start cockpit.socket   # 运行Cockpit服务

$ systemctl enable –now cockpit.socket  # 启动该服务，随系统启动一同启动

$ systemctl status cockpit.socket
```

更详细的参考：[centos8 终端登录后提示需要把cockpit服务设置开机自启](https://zhuanlan.zhihu.com/p/113270502)

### nginx

#### 安装

```bash
$ yum install nginx # 简单快捷版


# 或使用repo文件，根据需要选择下面版本
$ vim /etc/yum.repos.d/nginx.repo

# 版本一：官方提供版本 https://www.nginx.com/resources/wiki/start/topics/tutorials/install
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1

# 版本二
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

# 版本三
[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

$ yum info nginx # 查看 可安装/已安装 nginx的版本信息
$ systemctl enable nginx.service # 配置开机启动
$ systemctl start nginx.service # 启动nginx
```



#### 配置