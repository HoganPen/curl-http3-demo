# WSL2 docker镜像存储位置修改

## 背景说明

由于最新的windows提供了新的虚拟化技术，（WSL/WSL2）适用于 Linux 的 Windows 子系统;
[官方文档说明](https://docs.microsoft.com/zh-cn/windows/wsl/)
相应的docker也及时跟进了对WSL的支持,这是一个相当大的优化,尤其是增加了对windows家庭版的支持;
[Docker官方文档](https://docs.docker.com/docker-for-windows/wsl/)

基于WSL2安装docker后,会发现大量的docker镜像文件,使系统文件C盘容量激增,所以我们需要手动修改docker的镜像地址保存到另外的非系统盘.
![docker镜像文件占据了大量的系统盘空间](https://img-blog.csdnimg.cn/20201130110730615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F1c3RpbmRldg==,size_16,color_FFFFFF,t_70)

## 处理步骤

![相关文章](https://img-blog.csdnimg.cn/20201130140851974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F1c3RpbmRldg==,size_16,color_FFFFFF,t_70)

- 退出Docker桌面版
  Icon右键点击 Quit Docker Desktop
- 确认所有wsl应用都已退出

```
wsl --list -v
```

 ![确认退出](https://img-blog.csdnimg.cn/20201130111045966.png)

- 导出docker镜像文件

```
wsl --export docker-desktop-data "F:\wsl\docker-data\docker-desktop-data.tar"
```

- 注销docker-desktop-data

```
wsl --unregister docker-desktop-data
```

- 指定文件夹重新导入

```
wsl --import docker-desktop-data F:\wsl\docker\data "F:\wsl\docker-data\docker-desktop-data.tar" --version 2
```

- 同样的道理,迁移 docker-desktop

```
wsl --export docker-desktop "F:\wsl\docker-data\docker-desktop.tar"
wsl --unregister docker-desktop
wsl --import docker-desktop F:\wsl\docker\desktop "F:\wsl\docker-data\docker-desktop.tar" --version 2
```

- 重启Docker即可