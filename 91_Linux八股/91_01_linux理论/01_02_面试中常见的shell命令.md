
https://testerhome.com/topics/32655

# 1 内存和cpu

**3、top 如何查看Linux系统的CPU占用率？**

可以使用以下命令查看Linux系统的CPU占用率：

```text
top
```

该命令会打印出系统正在运行的进程和相应的资源占用情况，包括CPU占用率、内存占用率等。

**回答：**  top命令显示和更新排序的过程信息。使用此top命令来确定正在运行的进程以及它们消耗了多少内存和CPU。

# 2 磁盘使用情况

**4、如何查看Linux系统的磁盘使用情况？**

可以使用以下命令查看Linux系统的磁盘使用情况：

```text
df
```

该命令会打印出系统中每个文件系统的使用情况，包括已用空间、剩余空间等。
回答：  用户可以使用df命令对磁盘空间问题进行故障排除。此处df表示显示可用磁盘空间。

92.什么是df -h命令？
回答：此命令显示已挂载文件系统上的可用空间。

93.什么是df -i命令？
回答： df -I命令显示已挂载文件系统上的空闲索引节点。



2  du
82.在Linux中du命令是什么？
Linux中的Ans： du命令用于检索有关哪些文件使用目录中磁盘空间的更多详细信息。

**du命令示例：**

```
`$ du -sh /``var``/log/*`

`1.8M  /``var``/log/anaconda`

`384K  /``var``/log/audit`

`4.0K  /``var``/log/boot.log`

`0 /``var``/log/chrony`

`4.0K  /``var``/log/cron`

`4.0K  /``var``/log/maillog`

`64K /``var``/log/messages`
```


3 fdisk -l命令的作用是什么？

回答：  fdisk -I命令显示磁盘分区的大小和类型（以root身份运行）。

# 3 网络 

**5、netstat 如何查看Linux系统的网络连接情况？**

可以使用以下命令查看Linux系统的网络连接情况：

```text
netstat
```

该命令会打印出系统中当前的网络连接状态，包括建立的连接、监听的端口等。
回答：  Linux中的netstat命令显示网络状态。此netstat命令显示正在使用的网络端口及其传入连接。

**37.如何查看默认路由和路由表**
回答：   要显示默认路由和路由表，我们使用以下命令。
$ route-n 
$ nestat-rn
$ ip


 38.如何检查Linux服务器中正在侦听哪些端口？
**回答：**我们有两个命令来检查Linux Server中正在侦听哪些端口。以下是两个命令 

|   |   |
|---|---|
|1个<br><br>2|`# netstat --listen`<br><br>`# netstat -l`|

# 4 软件包

**2、如何查看Linux系统中安装的软件包？**

在Debian/Ubuntu系统中，可以使用以下命令查看已安装的软件包：

```text
dpkg --list
```

在CentOS/RHEL系统中，可以使用以下命令查看已安装的软件包：

```text
yum list installed

**6、如何在Linux系统中安装软件包？**

在Debian/Ubuntu系统中，可以使用以下命令安装软件包：

```text
apt-get install <package_name>
```

在CentOS/RHEL系统中，可以使用以下命令安装软件包：

```text
yum install <package_name>
```


# 5 文件 

**7、如何在Linux系统中检查系统日志？**

可以使用以下命令查看Linux系统的系统日志：

```text
tail /var/log/syslog
```

该命令会打印出系统的系统日志，包括系统重要事件、错误信息等。


**8、如何在Linux系统中创建临时文件？**

可以使用以下命令在Linux系统中创建临时文件：

```text
mktemp
```

该命令会创建一个唯一的临时文件，并打印出该文件的名称。

**9、如何在Linux系统中修改文件权限？**

可以使用以下命令在Linux系统中修改文件权限：

```text
chmod
```

该命令可以修改文件的读、写、执行权限，例如，将某个文件设置为只读：

```text
chmod 400 <filename>
```

