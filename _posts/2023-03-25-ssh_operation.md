---
layout: mypost
title: ssh operation
categories: [ssh, linux, shell]
---

# SSH远程相关操作

### ssh连接

```shell
ssh -p 22 chen@172.0.0.1

#登陆时发送私钥，保存登陆信息
ssh-copy-id -p port username@ip

#服务器文件配置ubuntu
vi /etc/ssh/sshd_config 
#ssh重启（更改配置文件后）
service sshd reload

# 通过curl查询ssh端口技巧
curl ip:22
#如果该服务器默认ssh端口为22，则返回信息
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4
curl: (56) Recv failure: Connection reset by peer
#返回
Connection refused
#代表该端口没有运行中的程序
```

### 传送文件

```shell
scp -P 22 文件名 user@172.0.0.1:/root/1.txt   #传送文件
scp -r 文件夹 user@172.0.0.1:/root/    #传送文件夹
```

### 端口转发

```shell
# 图1 正向转发
ssh -L 123:localhost:456 remotehost
# 123 -> 本地端口
# 图2 正向转发通过服务器访问它能访问的另外一台服务器

# 图3 反向转发 远程用户可通过本机端口访问本机的内网
ssh -R 123:farawayhost:456 remotehost
```

![01](01.png)

![02](02.png)

![03](03.png)

![04](04.png)