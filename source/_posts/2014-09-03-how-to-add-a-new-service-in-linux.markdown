---
layout: post
title: "如何在Linux上新建一个Service"
date: 2014-09-03 16:27:37 +0800
comments: true
categories: Linux
author: jackson,andy
---

From Andy, Jackson at SuZhou sklcc.com

### 简介

linux中的service是用于方便对系统服务进行统一标准的管理，比如启动(start)，停止(stop)，重启(restart)和查看状态(status)等。

service本身命令是一个shell脚本。它在/etc/init.d/目录中查找指定的服务脚本，然后调用该服务脚本进行服务。例如当我们输入`service ssh status`时，service程序其实是调用/etc/init.d/ssh脚本来完成获取sshd状态的功能。所以`service ssh status`也等同于`/etc/init.d/ssh status`
 
有时候我们需要某个程序能够开机自启动，例如apache程序。同时又要求程序在运行中可以随时停止，重启和查看状态。

这时写一个脚本，并把它注册为一个service,就变得非常有用。

<!--more-->

### service的写法

service的写法有固定的模板，自己只要完成`start()`,`stop()`,`restart()`等方法中的逻辑就行。以/etc/init.d/ssh脚本为例来说：

```
case "$1" in  
  start)
        ...
        ;;
  stop)
        ...
        ;;
  reload|force-reload)
        ...
        ;;
  restart)
        ...
        ;;
  try-restart)
        ...
        ;;
  status)
        ...
        ;;
  *)
        ...
esac
```

一般自定义service脚本只需要约定俗成的实现start,stop,restart,status和\*5种情况就可以了。其中\*情况是用于输出service的用法的，例如ssh的\*的输出就是：

> Usage: /etc/init.d/ssh {start|stop|reload|force-reload|restart|try-restart|status}

至于service脚本中具体每个情况的实现需要根据具体情况而编写，这里就不赘述了。如有疑问可以参考/etc/init.d/下的其他的脚本。

### 注册

编写完service脚本之后，并不就意味着可以直接使用了，还需要把自定义的service脚本注册到系统里。

这里所谓的注册其实就是：

> 1. 把脚本拷贝至/etc/init.d/下（本例中自定义service脚本叫test）  
>    `sudo cp test /etc/init.d/ & cd /etc/init.d`  
> 2. 修改脚本的权限    
>    `sudo chmod 755 test`  
> 3. 配置开机启动  
>    在ubuntu10.04之前版本和在redhat/centos/fedora中都是使用chkconfig来管理的，但是在ubuntu之后的版本就没有了，在这里我们使用update-rc.d来替代chkconfig。具体可以参考[这里](http://blog.csdn.net/dante_k7/article/details/7213151)  
>    `sudo update-rc.d test defaults`  

经过上述步骤test已经成功注册成service了,可以通过`sudo service test start|stop|restart`或者`sudo /etc/init.d/test start|stop|restart`启动，停止，重启服务
过了一段时间你不再需要服务了,运行`sudo update-rc.d -f test remove`卸载服务

