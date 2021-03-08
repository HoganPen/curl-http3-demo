# 修改ubuntu 18.04的sources.list源为阿里或清华镜像

## 1. 备份源列表

Ubuntu缺省的配置的源并不是国内的服务器，下载更新软件都比较慢，本文介绍如何设置源列表，选择比较快的源以节省下载时间。



```bash
# 首先备份源列表
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
```

## 2. 打开sources.list文件修改

选择合适的源，替换原文件的内容，保存编辑好的文件, 以阿里云更新服务器为例（从实际测试上结果分析，个人认为阿里云比网易和搜狐的服务器要快）：



```bash
sudo vim /etc/apt/sources.list
```

## 3. 阿里云镜像源-清华镜像源

注意根据具体使用的Ubuntu的版本不同，将文本中trusty替换为下面对应版本的字符串:

| 版本                      | 版本号 | 代号    |
| ------------------------- | ------ | ------- |
| Lucid(10.04)              | 10.04  | lucid   |
| Precise(12.04):  precise  | 12.04  | precise |
| Trusty(14.04):     trusty | 14.04  | trusty  |
| Utopic(14.10):     utopic | 14.10  | utopic  |
| Ubuntu 16.04 TLS： xenial | 16.04  | xenial  |
| Ubuntu 18.04 TLS： bionic | 18.04  | bionic  |

### Ubuntu 18.04 TLS版本阿里云镜像源：



```bash
# https://opsx.alibaba.com/mirror
deb https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse 

# 仿照清华镜像源，注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# deb-src https://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse 
# deb-src https://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse 
# deb-src https://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse 
# deb-src https://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse 
# deb-src https://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
```

### Ubuntu 18.04 TLS版本清华镜像源：



```bash
# https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

## 4. 刷新列表，一定要执行刷新



```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential
```

## 5. 参考

1. [修改ubuntu的sources.list源-http://blog.csdn.net/u010053463/article/details/49300625](http://blog.csdn.net/u010053463/article/details/49300625)
2. [修改ubuntu的sources.list源-https://blog.csdn.net/wang725/article/details/79902004](https://blog.csdn.net/wang725/article/details/79902004)
3. [如何修改Ubuntu的源列表(source list)-https://blog.csdn.net/jinguangliu/article/details/46539639](https://blog.csdn.net/jinguangliu/article/details/46539639)