# 在Linux实现科学上网

*本教程存在的意义是方便实验室的学弟学妹下载与更新Linux包和ROS功能包，软件不必以2kb/s的速度下载，费时且不成功，并解决了学术资源访问不成功的问题。*

**🈲️请勿在其他地方传播此教程，仅限实验室内部人员查看与使用**

**⚠️再次提醒⚠️：勿作他途，仅限学术研究与下载更新ROS**

> 本次演示的环境为：Ubuntu Server 18.04.1 LTS 64bit
> 本文参考了[在 Linux 上使用 Clash 作代理🔗](https://einverne.github.io/post/2021/03/linux-use-clash.html)

> 注：`$`作为shell中特殊变量使用，复制到终端中运行时不要把`$`复制进去

## Clash简介

[Clash🔗](https://github.com/Dreamacro/clash)是一款用Go开发的支持Linux/MacOS/Windows等多平台的网络工具。

## 下载安装

> 以下是`v1.7.1`，若需要最新版自行替换以下链接或前往官网页面[下载🔗](https://github.com/Dreamacro/clash/releases/latest)，注意选择`clash-linux-amd64`这个版本

```shell
$ cd ~/Downloads
$ wget -O clash.gz https://github.com/Dreamacro/clash/releases/download/v1.7.1/clash-linux-amd64-v1.7.1.gz
```

若网络不畅请`crtl-c`结束进程
将`Attachment/clash.gz`拖入到Downloads文件夹中

```shell
$ cd ~/Downloads
$ gzip -d clash.gz
$ sudo cp clash /usr/local/bin
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
$ curl "这里填写自己的订阅转换后的链接" > config.yaml
```

该配置中的端口分别为http: 7890，socks: 7891

再运行`clash`,若没有报错，则配置成功

## 设置系统与终端代理

### 系统代理

若浏览器无法访问谷歌，这说明系统代理没有设置

以Ubuntu 18.04为例，在设置(Settings)-网络(Network)-网络代理(Network Proxy)中，选择手动，在HTTP Proxy与HTTPS Proxy设置为代理对应的地址与端口，我这里地址均为 `127.0.0.1`，端口为 `7890`，即可设置系统代理

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

在配置开机启动之前，将配置文件移动到`/etc`目录：
以后修改配置都记住修改`/etc/clash`目录下的这个配置文件

```shell
$ sudo mv ~/.config/clash /etc
```

然后使用`vim`增加`systemd`配置

```shell
$ sudo vim /etc/systemd/system/clash.service 
```

放入如下内容：

```
[Unit]
Description=Clash Daemon

[Service]
ExecStart=/usr/local/bin/clash -d /etc/clash/
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

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

Clash提供了默认的`9090`端口作为远端管理端口,可以使用[Clash远程管理界面](http://clash.razord.top/#/proxies)进行管理
<http://clash.razord.top/#/proxies>
这个页面要求提供Host,Port,Secret三个参数：

```
Host: 127.0.0.1
Port: 9090
Secret: 配置文件配置的secret，若没有则留空
```

> 自此Clash就已经配置完成





