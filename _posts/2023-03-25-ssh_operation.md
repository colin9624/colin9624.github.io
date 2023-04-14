---
layout: mypost
title: ssh operation
categories: [ssh, linux, shell]
---

# SSH remote related operation

### ssh connect

```shell
ssh -p 22 chen@172.0.0.1

#Send the private key when logging in and save the login information
ssh-copy-id -p port username@ip

#Ubuntu server file configuration
vi /etc/ssh/sshd_config 
#ssh restart: after changing the configuration file
service sshd reload

# Skills of querying ssh ports through curl
curl ip:22
# If the default ssh port of the server is 22, the information is returned.
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4
curl: (56) Recv failure: Connection reset by peer
# Return
Connection refused
# Indicates that the port does not have a running program
```

### Upload file

```shell
scp -P 22 filename user@172.0.0.1:/root/1.txt
scp -r foldername user@172.0.0.1:/root/
```

### Port forwarding

```shell
# Figure 1 Forward forwarding
ssh -L 123:localhost:456 remotehost
# 123 -> local port
# Figure 2 Forward forwarding accesses another server that it can access through the server.

# Figure 3 Reverse forwarding allows remote users to access the local intranet through the local port
ssh -R 123:farawayhost:456 remotehost
```

![01](01.png)

![02](02.png)

![03](03.png)

![04](04.png)