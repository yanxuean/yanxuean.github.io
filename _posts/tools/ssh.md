---
layout: post
title: ssh
categories: [tools]
description: ssh
keywords: ssh
---

## ssh���Ҫ��
###  һ��ʹ��
####  ʹ��key��֤��ʽ��¼SSH��

1.     ��װSSH
Client��һ�㰲װ��ssh �ͻ��ˡ�Server��Ҫ��װssh �����������£�
Ubunt:   apt-get install openssh-server
CentOS:  yum -y install ssh

2.     ����Key
```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/xxx-user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/xxx-user/.ssh/id_rsa.
Your public key has been saved in /home/xxx-user/.ssh/id_rsa.pub.
```

3.     ������Կ����������ָ���û���~/.ssh/authorized_keys �ļ���
```
$ ssh-copy-id -i .ssh/id_rsa.pub username@xx.xx.xx.xx
```
Ҳ���ֹ�����
   Ҫһ�У��Ҹ��ļ�Ȩ��Ϊ600�����more /etc/ssh/sshd_config������ΪStrictModes yes��
   
1.4     ��֤��¼
$ ssh username@10.62.105.210

1.5     ��ֹ�˻������¼
Զ��Linux������/etc/ssh/sshd_config�����ļ� 
PasswordAuthentication ��Ϊ no 
�޸ĺ�/etc/init.d/sshd reload #�����޸ĺ�������ļ�����Ч 
ubuntu��ִ��service ssh restart

1.6     �޸�SSH�˿�
Port 22 �޸�Ϊָ���˿� xxxx

1.7     ָ���˿ڵ�¼
$ ssh -p xxxx username@10.62.105.210

2       ��װfirefox
��װfirefox
$ sudo apt-get install firefox
�����������������������⣬��װ��������
$ sudo apt-get install ttf-wqy-microhei #��Ȫ��-΢�׺�
$ sudo apt-get install ttf-wqy-zenhei  #��Ȫ��-����
$ sudo apt-get install xfonts-wqy #��Ȫ��-��������
 
3       ubuntu ����������ô��������

��ʹ�� ubuntu �����е�¼��ʱ�򣬳��֣�
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
    LANGUAGE = (unset),
    LC_ALL = (unset),
    LC_MESSAGES = "zh_CN.UTF-8",
    LANG = "zh_CN.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
3.1     ��װ localepurge ���������ļ�

$ sudo apt-get install localepurge
ѡ��������Ҫ�����ԣ����� en_US.UTF-8 �� zh_CN.UTF-8��
 
��ȻҲ����ʹ�����������ٴν������ã�
$ sudo dpkg-reconfigure localepurge
3.2     �����Լ���Ҫ������

$ sudo locale-gen zh_CN.UTF-8 en_US.UTF-8
��ӡ����ǰ��������Ϣ
$ locale

   

###  �����ļ�˵��
####  client��
���ȡ����~/.ssh/ssh_config�ļ���ȡhostname��user��port�����ò�����
һЩ�ļ���Ȩ��Ҫ��
```
    .ssh 700
    .ssh/config   600
    .ssh/id_rsa.pub  600
    ����ļ�  600
```

#### server��
.ssh/known_hosts�ļ�ʱ��server�˵�footprint
ssh server�����־�ļ�Ϊ/var/log/secure
```
    .ssh   700
    .ssh/authorized_keys   600
```

### �÷�˵��
clientҲ����ͨ��ѡ��ִ��identity_file
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
    DSA, ECDSA, ED25519 or RSA algorithms.    ��Public key authenticationʹ�õ��㷨
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
    client  public key��
     ~/.ssh/identity.pub (protocol 1), 
     ~/.ssh/id_dsa.pub (protocol 2 DSA),
     ~/.ssh/id_ecdsa.pub (protocol 2 ECDSA), 
     ~/.ssh/id_ed25519.pub (protocol 2 ED25519), or 
     ~/.ssh/id_rsa.pub (protocol 2 RSA)

    server  public key for user��
    ~/.ssh/authorized_keys      one key  per line����client�Ĺ�Կ���ļ�Ȩ�ޱ�����600��  .sshĿ¼��Ȩ�ޱ�����700 

   server  
   ~/.ssh/known_hosts     server save host key
   /etc/ssh/ssh_known_hosts      is automatically checked for known  hosts.

finally, if other authentication methods fail,  ssh prompts the user for a password

HOSTKEY:
A host key is a cryptographic key used for authenticating computers in the SSH protocol.
Host keys are key pairs, typically using the RSA, DSA, or ECDSA algorithms. Public host keys are stored on and/or distributed to SSH clients, and private keys are stored on SSH servers.
Each host (i.e., computer) should have a unique host key

```

���ϣ��ssh��Կ��Ч��������������3��������
1)  .sshĿ¼��Ȩ�ޱ�����700 
3) �û�Ŀ¼Ȩ��Ϊ 755 ���� 700�����ǲ�����77x
    

