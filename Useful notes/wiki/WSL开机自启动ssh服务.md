# WSL开机自启动ssh服务

1. win+R键调出运行，输入shell:startup确定进入开始菜单启动程序目录(大致是C:\Users\用户名\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup);

2. startWSL.vbs放到上述目录;

3. 打开wsl，将init.sh放到/目录，即/init.sh，记得要提前安装openssh-server(apt-get install openssh-server)，应该默认安装好了;

4. 下次开机将自动启动wsl，并运行着ssh的进程，任务管理器中可以看到如下图内容。

  此时，可以使用xshell等连接localhost，可以使用密码或密钥连接，方式同linux，这里不予累述。

![p](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcuc2FpbnRpYy5jb20vRWF1RG91Y2UvYmxvZy8yMDE4MDQyNzEwMDQyOTM1NTAucG5n?x-oss-process=image/format,png)

1. startWSL.vbs

```bash
Set ws = WScript.CreateObject("WScript.Shell")
cmd = "C:\Windows\System32\bash.exe -c ""bash /init.sh"""
'运行命令不显示cmd窗口
ws.Run cmd, 0, false
Set ws = Nothing
WScript.quit
```

 

2. init.sh

```
#!/bin/bash
pn=$(ps aux|grep -v grep|grep sshd|wc -l)
[ -d /var/run/sshd ] || mkdir /var/run/sshd
chmod 744 /var/run/sshd

if [ "${pn}" != "0" ]; then
    pid=$(ps aux|grep -v grep|grep /usr/sbin/sshd|awk '{print $2}')
    kill $pid
fi

/usr/sbin/sshd -D
```

 需要默认root运行

```
在cmd 下：

wslconfig /list # 查看wsl版本
ubuntu1604 config --default-user root
Ubuntu 18.04

在cmd 目录下：
    ubuntu1804 config --default-user root
```

配置文件github地址： https://github.com/yinshangqing/WSL

#### 1、准备工作

**win+R**键调出运行，输入**shell:startup**确定进入开始菜单启动程序目录(大致是C:\Users\用户名\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup);

#### 2、放置文件

将startWSL放到上述目录 将打开wsl，将init.sh放到/目录（Ubuntu的根目录），即/init.sh，记得要提前安装openssh-server(apt-get install openssh-server)，应该默认安装好了;

#### 3、设置权限

需要默认以root运行，所以在cmd命令下，先查看安装的Ubuntu版本 （cmd下）使用 ubuntu config --default-user root **Ubuntu18.04**版本时使用下面的命令： ubuntu1804 config --default-user root

#### 4、重启电脑

重启电脑后，wsl默认开始运行了，就可以使用xshell直接连了，不需要别的操作