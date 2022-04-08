# 1.现象

OpenSSH版本：8.8p1

在 openssh配置文件中开启 pam，如下：

```
UsePAM yes
```

发现在 windows上用远程连接工具无法以 root用户登录，其他的 linux服务器也无法以 ssh的方式登录该服务器。

远程连接工具报错：Access denied

# 2.分析

从报错来看只能猜测是权限问题，具体原因还需要查看 Linux系统日志，这里简要介绍两个系统日志：

- **/var/log/messages**

  这里存放的是一些常规系统日志，一般的操作都能在这里找到对应日志，但是有的日志不是很明确

- **/var/log/secure**

  存放安全类的日志，常见的就是登录登出，有些日志比较详细，更能帮助定位问题

当 root用户无法登录时，查看这两个日志相关输出如下：

```
# /var/log/messages报错
sshd[32475]: error: PAM: Authentication failure for root from 192.168.xx.xx
sshd[32475]: Failed password for root from 192.168.xx.xx port 64392 ssh2

# /var/log/secure报错
login: pam_unix(login:auth): check pass; user unknown
login: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=tty1 ruser= rhost=
login: FAILED LOGIN 1 FROM tty1 FOR (unknown), User not known to the underlying authentication module
```

日志中还是很明确的，由于 pam的权限管控，在登录时认证失败。

查询资料后，发现在 openssh中不能仅仅打开 pam开关，还需要增加 sshd的相关配置才行。

# 3.解决

**备注**：此时虽然无法远程登录服务器，但是可以直连服务器进行相关操作

**第一步，在 pam目录中增加 sshd的配置**

```shell
vim /etc/pam.d/sshd

# 文件内容如下
#%PAM-1.0
auth	   required	pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
-auth      optional     pam_reauthorize.so prepare
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
session    include      postlogin
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
```

**第二步，重启 sshd**

```
systemctl restart sshd
```

**第三步，验证能否远程以 root账户登录该服务器**

