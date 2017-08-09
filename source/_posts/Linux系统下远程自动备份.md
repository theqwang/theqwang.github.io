title: Linux系统下远程自动备份
date: 2017-08-09 08:32:40
categories: linux
tags:
---
公司的两个网站都部署在阿里云上，这两个网站以文章展示为主，之前也没有做过数据备份。随着文章数量的增加，实现数据的自动备份也就越来越重要了。后来这个工作就落到了我的头上，也感谢这次机会，让我重新拾起了《鸟哥的Linux私房菜》，狠狠加强了一把我的Linux基础。
<!--more-->

## 1.背景
这里以自动备份网站的数据库和代码为例讲解我的实现思路。
要备份的网站部署在了阿里云服务器上，简称该服务器为A；备份服务器是我们公司的一台物理机，简称为B，备份的数据都要统一放在B上。

## 2.备份思路
1. 使用**SSH**实现主机B到A的免密码登录
2. 使用**rsync**实现代码的增量备份
3. 使用**mysqldump+tar**备份数据库，再通过**scp**把备份的数据库文件从A拷贝到备份主机B的指定位置并按时间命名，保留5天之内的数据库备份文件其余删除
4. 使用**crontab**实现定时备份

## 3. 备份频率
* 数据库每天备份一次，备份的时间点是每天`2:00`
* 代码每周备份一次，备份的时间点是每周的星期天`3:00`

## 4. SSH免密码登录
要把主机A上的数据定时备份到主机B上，要先实现SSH免密码登录。具体步骤：

1) 用root登录B，生成SSH密钥

```bash
ssh-keygen
```

执行完该命令后，会在`/root/.ssh`目录下生成`id-rsa`、`id-rsa.pub`，其中`id-rsa`是私钥文件，`id-rsa.pub`是公钥文件。

2) 把`id-rsa.pub`上传到A上

```
scp /root/.ssh/id_rsa.pub  root@A.com:/root/id_rsa.pub
```

3) 把公钥内容附件到`authorized_keys`文件中

```shell
// authorized_keys的路径是：/root/.ssh/authorized_keys，如果路径不存在，需要执行以下命令创建。

mkdir -p /root/.ssh
chmod 700 /root/.ssh
touch /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
cat /root/id_rsa.pub >> /root/.ssh/authorized_keys
```

注意：因为涉及到权限的问题，在A上也使用root登录；请一定要给`.ssh`和`authorized_keys`设置700和600的权限，不然用SSH登录时还是需要密码

4) 验证SSH免密码登录是否成功

```shell
ssh -v root@A
```



## 5.增量备份代码

使用**rsync**增量备份代码。假设A上部署的项目路径是`/wwwXXXcom`， 把A上的代码备份到B的`/data/backup_code`目录下，在该目录下新建备份脚本文件`backup_code.sh`，在该文件中写入：

```shell
#!/bin/bash
rsync -avz --delete -e ssh root@A:/wwwXXXcom /data/backup_code
```

`--delete`参数表示如果A上的某文件删除了，备份时B上也会删除对应文件

给`backup_code.sh`增加执行权限：

```shell
chmod +x backup_code.sh
```



## 6.备份数据库

备份数据库的思路是首先在主机A上使用**mysqldump+tar**把数据库备份到本地，然后通过**scp**把备份文件拷贝到主机B上。

### 6.1 在主机A上备份数据库

假设在主机A上，数据库备份文件放在`/backup_origin`目录下，在该目录下新建数据库备份脚本`backup_origin.sh`，文件中写入：

```shell
#!/bin/bash

# 删除之前的数据库备份文件
find /backup_origin -name "DB_name.sql.gz" -type f -exec rm {} \; > /dev/null 2>&1

# 备份数据库
mysqldump -uroot -ppassword DB_name | gzip > /backup_origin/DB_name.sql.gz
```

给`backup_origin.sh`增加执行权限：

```shell
chmod +x backup_origin.sh
```

### 6.2 把备份文件拷贝到主机B

假设数据库备份文件放在主机B的`/data/backup`目录下 ，在该目录下新建脚本文件`backup_DB.sh`，文件中写入：

```bash
#!/bin/bash

time=`date +%Y-%m-%d_%H:%M:%S`

# 通过SSH调用主机A上的数据库备份脚本
/usr/bin/ssh root@A "/backup_origin/backup_origin.sh"

# 通过scp把A上新生成的数据库备份文件拷贝到B
scp root@A:/backup_origin/DB_name.sql.gz /data/backup/DB_name$time.sql.gz

# 删除5天之前的备份文件
find /data/backup -name "DB_name*.sql.gz" -type f -mtime +5 -exec rm {} \; > /dev/null 2>&1
```

给`backup_DB.sh`增加执行权限：

```shell
chmod +x backup_DB.sh
```



## 7.定时备份

使用**crontab**实现定时备份功能。

```shell
crontab -e
```

在主机B上输入以上命令编辑`crontab`命令的内容，键入以下内容实现备份脚本的定时执行：

```shell
# 备份数据库，每天2:00执行
0 2 * * * bash /data/backup/backup_DB.sh

# 备份项目代码，每周的周天3:00执行
0 3 * * 0 bash /data/backup_code/backup_code.sh
```



## 8. 最后

至此，我们就实现了数据的远程自动备份。但是，数据还不是绝对的安全，现在数据都备份到了主机B上，但是当主机B出现问题时，备份也就没有意义了。更近一步，可以在主机B上做**RAID5**或者把主机B上的备份数据上传到云上。条件所限，这些措施也就没有做，以后做了我会再来补充。