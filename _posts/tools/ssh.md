---
layout: post
title: ssh
categories: [tools]
description: ssh
keywords: ssh
---

## ssh设计要求
###  一般使用
####  使用key验证方式登录SSH：

1.     安装SSH
Client端一般安装了ssh 客户端。Server端要安装ssh 服务器，如下：
Ubunt:   apt-get install openssh-server
CentOS:  yum -y install ssh

2.     创建Key
```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/xxx-user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/xxx-user/.ssh/id_rsa.
Your public key has been saved in /home/xxx-user/.ssh/id_rsa.pub.
```

3.     拷贝公钥到服务器端指定用户的~/.ssh/authorized_keys 文件中
```
$ ssh-copy-id -i .ssh/id_rsa.pub username@xx.xx.xx.xx
```
也可手工拷贝
   要一行，且该文件权限为600（如果more /etc/ssh/sshd_config中配置为StrictModes yes）
   
1.4     验证登录
$ ssh username@10.62.105.210

1.5     禁止账户密码登录
远程Linux主机的/etc/ssh/sshd_config配置文件 
PasswordAuthentication 改为 no 
修改后/etc/init.d/sshd reload #加载修改后的配置文件并生效 
ubuntu上执行service ssh restart

1.6     修改SSH端口
Port 22 修改为指定端口 xxxx

1.7     指定端口登录
$ ssh -p xxxx username@10.62.105.210

2       安装firefox
安装firefox
$ sudo apt-get install firefox
解决浏览器出现中文乱码问题，则安装中文字体
$ sudo apt-get install ttf-wqy-microhei #文泉驿-微米黑
$ sudo apt-get install ttf-wqy-zenhei  #文泉驿-正黑
$ sudo apt-get install xfonts-wqy #文泉驿-点阵宋体
 
3       ubuntu 解决语言设置错误的问题

在使用 ubuntu 命令行登录的时候，出现：
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
    LANGUAGE = (unset),
    LC_ALL = (unset),
    LC_MESSAGES = "zh_CN.UTF-8",
    LANG = "zh_CN.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
3.1     安装 localepurge 管理语言文件

$ sudo apt-get install localepurge
选择我们想要的语言，例如 en_US.UTF-8 和 zh_CN.UTF-8。
 
当然也可以使用以下命令再次进行配置：
$ sudo dpkg-reconfigure localepurge
3.2     生成自己想要的语言

$ sudo locale-gen zh_CN.UTF-8 en_US.UTF-8
打印出当前的配置信息
$ locale

   

###  配置文件说明
####  client侧
会读取本地~/.ssh/ssh_config文件获取hostname，user，port等配置参数。
一些文件的权限要求：
```
    .ssh 700
    .ssh/config   600
    .ssh/id_rsa.pub  600
    别的文件  600
```

#### server侧
.ssh/known_hosts文件时是server端的footprint
ssh server侧的日志文件为/var/log/secure
```
    .ssh   700
    .ssh/authorized_keys   600
```

### 用法说明
client也可以通过选项执行identity_file
```
-i identity_file
             Selects a file from which the identity (private key) for public
             key authentication is read.  The default is ~/.ssh/identity for
             protocol version 1, and ~/.ssh/id_dsa, ~/.ssh/id_ecdsa,
             ~/.ssh/id_ed25519 and ~/.ssh/id_rsa for protocol version 2.
             Identity files may also be specified on a per-host basis in the
             configuration file.  It is possible to have multiple -i options
             (and multiple identities specified in configuration files).  ssh
             will also try to load certificate information from the filename
             obtained by appending -cert.pub to identity filenames.

     The methods available for authentication are: GSSAPI-based
     authentication, host-based authentication, public key authentication,
     challenge-response authentication, and password authentication.
     Authentication methods are tried in the order specified above, though
     protocol 2 has a configuration option to change the default order:
     PreferredAuthentications.

     A variation on public key authentication is available in the form of certificate authentication
 
public key authentication:
    DSA, ECDSA, ED25519 or RSA algorithms.    是Public key authentication使用的算法
    The file ~/.ssh/authorized_keys lists the public keys that are permitted
     for logging in.  When the user logs in, the ssh program tells the server
     which key pair it would like to use for authentication.  The client
     proves that it has access to the private key and the server checks that
     the corresponding public key is authorized to accept the account.

    client  private key
     ~/.ssh/identity (protocol 1), 
     ~/.ssh/id_dsa (protocol 2 DSA), 
     ~/.ssh/id_ecdsa (protocol 2 ECDSA), 
     ~/.ssh/id_ed25519 (protocol 2 ED25519), or 
     ~/.ssh/id_rsa (protocol 2 RSA) 
    client  public key：
     ~/.ssh/identity.pub (protocol 1), 
     ~/.ssh/id_dsa.pub (protocol 2 DSA),
     ~/.ssh/id_ecdsa.pub (protocol 2 ECDSA), 
     ~/.ssh/id_ed25519.pub (protocol 2 ED25519), or 
     ~/.ssh/id_rsa.pub (protocol 2 RSA)

    server  public key for user：
    ~/.ssh/authorized_keys      one key  per line。放client的公钥。文件权限必须是600。  .ssh目录的权限必须是700 

   server  
   ~/.ssh/known_hosts     server save host key
   /etc/ssh/ssh_known_hosts      is automatically checked for known  hosts.

finally, if other authentication methods fail,  ssh prompts the user for a password

HOSTKEY:
A host key is a cryptographic key used for authenticating computers in the SSH protocol.
Host keys are key pairs, typically using the RSA, DSA, or ECDSA algorithms. Public host keys are stored on and/or distributed to SSH clients, and private keys are stored on SSH servers.
Each host (i.e., computer) should have a unique host key

```

如果希望ssh公钥生效需满足至少下面3个条件：
1)  .ssh目录的权限必须是700 
3) 用户目录权限为 755 或者 700，就是不能是77x
    

