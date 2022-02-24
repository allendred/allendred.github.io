Linux scp 设置nohup后台运行

- [1.正常执行scp命令](https://www.cnblogs.com/jyzhao/p/6253728.html#1)
- [2.输入ctrl + z 暂停任务](https://www.cnblogs.com/jyzhao/p/6253728.html#2)
- [3.bg将其放入后台](https://www.cnblogs.com/jyzhao/p/6253728.html#3)
- [4.disown -h 将这个作业忽略HUP信号](https://www.cnblogs.com/jyzhao/p/6253728.html#4)
- [5.测试会话中断，任务继续运行不受影响](https://www.cnblogs.com/jyzhao/p/6253728.html#5)

# 1.正常执行scp命令

从oradb30机器拷贝一个文件夹到oradb31机器:
scp -r /u01/media/Disk1/ 192.168.1.31:/u01/media/

```perl
[root@oradb30 ~]# scp -r /u01/media/Disk1/ 192.168.1.31:/u01/media/
reverse mapping checking getaddrinfo for bogon failed - POSSIBLE BREAK-IN ATTEMPT!
root@192.168.1.31's password: 
...
filegroup2.jar                                                                                                                                              100%   84KB  83.8KB/s   00:00    
filegroup9.jar                                                                                                                                              100%   16KB  16.1KB/s   00:00    
```

# 2.输入ctrl + z 暂停任务

输入ctrl + z 暂停

```ruby
[1]+  Stopped                 scp -r /u01/media/Disk1/ 192.168.1.31:/u01/media/
[root@oradb30 ~]# 
```

此时查看jobs：

```ruby
[root@oradb30 ~]# jobs
[1]+  Stopped                 scp -r /u01/media/Disk1/ 192.168.1.31:/u01/media/
[root@oradb30 ~]# 
```

# 3.bg将其放入后台

bg将该任务号放入后台:

```ruby
[root@oradb30 media]# bg %1
[1]+ scp -r Disk1/ 192.168.1.31:/u01/media/ &
```

查看任务已经在后台运行:

```ruby
[root@oradb30 media]# jobs
[1]+  Running                 scp -r Disk1/ 192.168.1.31:/u01/media/ &
```

# 4.disown -h 将这个作业忽略HUP信号

使用disown -h 将这个作业忽略HUP信号:

```ruby
[root@oradb30 media]# disown -h %1
[root@oradb30 media]# jobs
[1]+  Running                 scp -r Disk1/ 192.168.1.31:/u01/media/ &
```

查看任务运行状态和父进程号:

```bash
[root@oradb30 media]# ps -ef|grep scp
root     12704 12638  0 05:19 pts/0    00:00:01 scp -r Disk1  192.168.1.31 /u01/media/
root     12705 12704  8 05:19 pts/0    00:00:17 /usr/bin/ssh -x -oForwardAgent no -oPermitLocalCommand no -oClearAllForwardings yes 192.168.1.31 scp -r -t /u01/media/
root     12823 12638  0 05:22 pts/0    00:00:00 grep scp
```

# 5.测试会话中断，任务继续运行不受影响

断开该会话测试任务是否可以继续后台运行:

```perl
[root@oradb30 media]# exit
logout

Last login: Thu Jan  5 05:19:50 2017 from 192.168.1.198

[root@oradb30 ~]# 
[root@oradb30 ~]# 
[root@oradb30 ~]# 
[root@oradb30 ~]# ps -ef|grep scp
root     12704     1  0 05:19 ?        00:00:02 scp -r Disk1  192.168.1.31 /u01/media/
root     12705 12704  8 05:19 ?        00:00:17 /usr/bin/ssh -x -oForwardAgent no -oPermitLocalCommand no -oClearAllForwardings yes 192.168.1.31 scp -r -t /u01/media/
root     12854 12829  0 05:22 pts/2    00:00:00 grep scp
```

发现scp任务继续运行，没有因为会话断开而中断，父进程号变为1。

如果有其他任务需要使用nohup后台运行，但执行时却忘记了使用nohup，也可以参照此方法进行设置。

如果配置好ssh无密码登陆，也可以直接 nohup scp .. & 执行。
