# 需求

首先我们明确我们的目标，我们是希望通过SFTP代替FTP，需求是这样的:

1. 这些用户**只能通过SFT**访问服务器
2. 用户要**锁定**在相应的目录下

我想通常的思路是这样的:

1. 打开OpenSSH的SFTP
2. 将这些用户设置在一个*组*中
3. 让OpenSSH识别这个组，只允许这些用户使用SFTP
4. 最后系统要自动的将用户*chroot*在用户目录下

# 配置

## SSHD_CONFIG

sshd通常是打开了sftp的，不过我们应该使用**internal-sftp**在sshd_conf中作如下配置:

```
# Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp

##
Match group sftponly
	ChrootDirectory %u
	X11Forwarding no
	AllowTcpForwarding no
	ForceCommand internal-sftp
```



# 创建用户

```
## -G 表示新用户属于sftponly组 
## -d 表示新用户的HOME_DIR
## -s 表示新用户登录的shell
#useradd -G sftponly -d /home/sftphome  -s /usr/bin/bash sftpuser
#vim /etc/password

sftpuser:x:1001:1002::/home/sftphome:/usr/bin/bash
#passwd sftpuser
## 然后输入密码
```

# 启动

```
#sftp sftpuser@127.0.0.1
sftp>
sftp>
sftp>
```

