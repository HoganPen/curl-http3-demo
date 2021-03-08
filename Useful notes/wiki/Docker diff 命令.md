# Docker diff 命令

**docker diff :** 检查容器里文件结构的更改。

### 语法

```
docker diff [OPTIONS] CONTAINER
```

### 实例

查看容器mymysql的文件结构更改。

```
runoob@runoob:~$ docker diff mymysql
A /logs
A /mysql_data
C /run
C /run/mysqld
A /run/mysqld/mysqld.pid
A /run/mysqld/mysqld.sock
C /tmp
```