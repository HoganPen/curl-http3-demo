# Docker create 命令

**docker create ：**创建一个新的容器但不启动它

用法同 [docker run](https://www.runoob.com/docker/docker-run-command.html)

### 语法

```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

语法同 [docker run](https://www.runoob.com/docker/docker-run-command.html)

### 实例

使用docker镜像nginx:latest创建一个容器,并将容器命名为myrunoob

```
runoob@runoob:~$ docker create  --name myrunoob  nginx:latest      
09b93464c2f75b7b69f83d56a9cfc23ceb50a48a9db7652ee4c27e3e2cb1961f
```