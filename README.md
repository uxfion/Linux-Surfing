# 在Linux下实现科学上网

[TOC]

*本教程存在的意义是方便实验室的学弟学妹下载与更新Linux包和ROS功能包，软件不必以2kb/s的速度下载，费时且不成功，并解决了学术资源访问不成功的问题。*

**🈲️请勿在其他地方传播此教程，仅限实验室内部人员查看与使用**

**⚠️再次提醒⚠️：勿作他途，仅限学术研究与下载更新ROS**

> 本次演示的环境为：Ubuntu Server 18.04.1 LTS 64bit
> 本文参考了[在 Linux 上使用 Clash 作代理🔗](https://einverne.github.io/post/2021/03/linux-use-clash.html)
> 注：`$`作为shell中特殊变量使用，复制到终端中运行时不要把`$`复制进去

## Clash简介

[Clash🔗](https://github.com/Dreamacro/clash)是一款用Go开发的支持Linux/MacOS/Windows等多平台的网络工具。

## 下载安装

> 以下是`v1.7.1`，若需要最新版自行替换代码框中的文件名和下载链接
> 前往官网页面[下载🔗](https://github.com/Dreamacro/clash/releases/latest)最新发布的版本，注意选择`clash-linux-amd64`这个版本
> 

```shell
$ cd ~/Downloads
$ wget https://github.com/Dreamacro/clash/releases/download/v1.7.1/clash-linux-amd64-v1.7.1.gz
```

若网络不畅，下载一直没有速度，如下所示，一直在等待，请`crtl-c`结束进程

```
ubuntu@VM-12-15-ubuntu:~/Downloads$ wget https://github.com/Dreamacro/clash/releases/download/v1.7.1/clash-linux-amd64-v1.7.1.gz--2021-09-18 22:22:57--  https://github.com/Dreamacro/clash/releases/download/v1.7.1/clash-linux-amd64-v1.7.1.gzResolving github.com (github.com)... 13.250.177.223Connecting to github.com (github.com)|13.250.177.223|:443... connected.HTTP request sent, awaiting response...
```

将`Attachment/clash-linux-amd64-v1.7.1.gz`拖入到Downloads文件夹中

```shell
$ cd ~/Downloads
$ gzip -d clash-linux-amd64-v1.7.1.gz
$ sudo cp clash-linux-amd64-v1.7.1 /usr/local/bin/clash
$ sudo chmod +x /usr/local/bin/clash
$ clash
```

运行成功结果，Clash启动后会在`~/.config/clash`目录生成配置文件

```shell
ubuntu@VM-12-15-ubuntu:~$ ./clash 
INFO[0000] Can't find config, create a initial config file 
INFO[0000] Can't find MMDB, start download              
INFO[0000] Mixed(http+socks) proxy listening at: 127.0.0.1:7890 
```

## 修改配置

> 大家可以使用此[链接🔗](https://www.v2aky.jp.net/#/register?code=voCjUSdi)注册
> <https://www.v2aky.jp.net/#/register?code=voCjUSdi>

比如说对于我使用的机场✈️，点击`仪表盘-我的订阅-一键订阅`，复制订阅地址，之后会用
如果自己有机场，复制自己机场✈️的订阅地址放进订阅转换中

**保管好自己的订阅，不要发送给其他人**

打开[订阅转换🔗](https://bianyuan.xyz)

选择进阶模式，把刚才复制的订阅地址粘贴到订阅链接中，客户端选择Clash，后端地址保持默认，远程配置选择`ACL4SSR_Online_Mini精简版`（大家也可尝试其他配置），其他保持默认，点击生成链接，复制该链接

可以在浏览器打开上述订阅转换后的链接，即为Clash的配置文件

```shell
$ curl "这里填写自己的订阅转换后的链接" > ~/.config/clash/config.yaml
```

该配置中的端口分别为http: 7890，socks: 7891

再运行`clash`，若没有报错，则配置成功，有如下输出：

```
ubuntu@VM-12-15-ubuntu:~/Downloads$ clashINFO[0000] Start initial compatible provider ♻️ 自动选择    INFO[0000] Start initial compatible provider 🚀 节点选择     INFO[0000] Start initial compatible provider 🎯 全球直连     INFO[0000] Start initial compatible provider 🛑 全球拦截     INFO[0000] Start initial compatible provider 🐟 漏网之鱼     INFO[0000] HTTP proxy listening at: [::]:7890           INFO[0000] SOCKS proxy listening at: [::]:7891          INFO[0000] RESTful API listening at: [::]:9090
```

保持该终端运行，之后的命令重新开一个终端运行


## 设置系统与终端代理

### 系统代理

若浏览器无法访问谷歌，这说明系统代理没有设置

以Ubuntu 18.04为例，在设置(Settings)-网络(Network)-网络代理(Network Proxy)中，选择手动(Manual)，在HTTP Proxy与HTTPS Proxy设置为代理对应的地址与端口，这里均为 `127.0.0.1`，端口为 `7890`，即可设置系统代理

### 终端代理

> 终端一般不会走系统代理，因此想命令行走代理的需要自己手动设置

设置终端走代理

```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7891
export no_proxy=localhost
```

每次打开终端都需要重新输入一遍，过于麻烦，因此我写了个本地代理开关函数，只需要在终端中输入`proxy`即可使终端开启代理，`unproxy`即可关闭代理

```
# 本地代理开关函数
function proxy() {
    export https_proxy=http://127.0.0.1:7890
    export http_proxy=http://127.0.0.1:7890
    export all_proxy=socks5://127.0.0.1:7891
    export no_proxy=localhost
    echo -e '\n          **已开启代理**          \n'}

function unproxy(){
    unset https_proxy
    unset http_proxy
    echo -e '\n          **已关闭代理**          \n'
}
```

在`～/.bashrc`中添加以下内容（复制一下内容到终端中按回车键），并`source ~/.bashrc`

```shell
$ echo "
# 本地代理开关函数
function proxy() {
    export https_proxy=http://127.0.0.1:7890
    export http_proxy=http://127.0.0.1:7890
    export all_proxy=socks5://127.0.0.1:7891
    export no_proxy=localhost
    echo -e '\n          **已开启代理**          \n'
}

function unproxy(){
    unset https_proxy
    unset http_proxy
    echo -e '\n          **已关闭代理**          \n'
}
" >> ~/.bashrc
```
    
### 代理测试

* 访问各大网站，如果都有网页源码输出说明代理没问题

```
$ curl -sL www.baidu.com
$ curl -sL www.google.com
```
    
* 获取当前 IP 地址

由于`cip.cc`策略组里没有走代理因此返回本地ip

```shell
$ curl cip.cc
```

## 开机启动

> 开机启动配置完成后，不再需要一个终端运行`clash`并保持打开状态

将上文提到运行`clash`的终端按`ctrl+c`结束进程
在配置开机启动之前，将配置文件移动到`/etc`目录中
以后修改配置都记住修改`/etc/clash`目录下的这个配置文件

```shell
$ sudo mv ~/.config/clash /etc
```

> 若没有找到vim，运行`sudo apt install vim`

然后使用`vim`增加`systemd`配置

```shell
$ sudo vim /etc/systemd/system/clash.service 
```

按`i`进入编辑模式，放入如下内容：

```
[Unit]
Description=Clash Daemon

[Service]
ExecStart=/usr/local/bin/clash -d /etc/clash/
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
按`esc`退出编辑模式，然后输入`:wq`保存文件
运行一次`sudo systemctl enable clash.service`即开启了自动启动服务，之后不需要再次开启

以下为`systemctl`的一些命令：
启用与终止clash.servic

```shell
$ sudo systemctl enable clash.service
$ sudo systemctl disable clash.service
```

手动开启与停止clash.servic

```shell
$ sudo systemctl start clash.service
$ sudo systemctl stop clash.service
```

如果要查看clash.service的日志可以使用：

```shell
$ journalctl -e -u clash.service
```
    
## 远程管理

> 此项远程管理面板在火狐浏览器上无法使用，待解决

Clash提供了默认的`9090`端口作为远端管理端口，可以使用以下Clash远程管理面板对Clash进行节点选择等操作
[Yadc](http://yacd.haishan.me)
[razord](http://clash.razord.top/#/proxies)

这个页面要求提供Host，Port，Secret三个参数：

```
Host: 127.0.0.1
Port: 9090
Secret: 配置文件配置的secret，若没有则留空
```

自此Clash就已经配置完成

## 常见问题

### sudo rosdep init 失败

> init不成功的根本原因是网络问题，找一个能用的DNS替换即可
> 参考自[ROS:sudo rosdep init出错常规方法都无效后解决办法记录](https://zhuanlan.zhihu.com/p/77483614)、[sudo rosdep init ERROR](http://obgeqwh.top/sudo-rosdep-init-error)

`sudo rosdep init`出现问题的输出结果如下：

```
ubuntu@VM-12-15-ubuntu:~$ sudo rosdep init
ERROR: cannot download default sources list from:
https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
Website may be down.
```

解决方法：

编辑hosts文件

```
$ sudo vim /etc/hosts
```

在末尾添加如下内容

```
151.101.84.133  raw.githubusercontent.com
```

保存后再次运行`sudo rosdep init`即可






