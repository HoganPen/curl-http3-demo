# Ubuntu 18.04使用root 帐号开启 SSH 登录

### Ubuntu 16.04 同样适用该方法

#### 一、安装ssh

18.04

```
apt install openssh-server  
```

16.04

```
apt-get install openssh-server
```

#### 二、更改Ubuntu的root密码

首先应该更改root密码，因为默认安装时并不是固定的root密码
切换到root用户

```
sudo su 
```

修改root用户的密码

```
passwd
```

两次输入root密码即可

#### 三、修改ssh的配置文件

找到文件

```
/etc/ssh/sshd_config
```

找到配置文件下面的地方

```
# Authentication:
LoginGraceTime 120
PermitRootLogin prohibit-password
StrictModes yes
```

修改为

```
# Authentication:
#LoginGraceTime 2m
#PermitRootLogin prohibit-password
PermitRootLogin yes
StrictModes yes
```

#### 四、重新启动ssh服务

```
service sshd restart
```

接下来就可以在windows进行登录了。