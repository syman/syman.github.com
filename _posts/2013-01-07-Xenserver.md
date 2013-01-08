---
layout : post
category : manage
tags : [VM]
title : Xenserver 单网口实现发布虚拟机
    
    Xenserver的主要工作接口


---
[思维导图文件下载](#)
# Xenserver 从哪里来？

        Xenserver 是Citrix基本Linux开发的虚拟化产品，可以使用Xenserver构建简单易于管理的虚拟化环境
		
        Xenserver 是基于强大的 Xen Hypervisor 程序之上
		
# Xenserver 的主要工作接口

        Xenserver 本身运行于Linux系统之上

        主要工作接口：

        资源管理
        
        存储管理
        
        虚拟交换机

# 如何利用Xenserver 单网口实现发布虚拟机

        在IDC机房，大家都知道，如果仅是在一台服务器托管的情况下，那么IDC运营商只会提供一根网线和一个IP，那么
        
        在想节约成本的前提下，如何使用这一个接口将运行在Xenserver之上的服务器，发布到公网之外那？

        首先，我们来看一下，Xenserver 本身就是一个Linux服务器，只不过是Citrix 公司在上面运行了一个虚拟化套件并

        对其实现了部分接口的优化和虚拟化底层的改写而已，那么既然是Linux 我们运维工作人员，在将Linux当做路由器时，
        
        是使用的什么来做地址NAT的哪？ 

        毫无疑问这里大家肯定会想到是：IPTables，对的，没错，就是IPtables。

        在这里我在扯一些废话，为什么我们要用Xenserver 而不用VMWare的ESXi那？ ESXi的虚拟交换机功能不必他差吗？

        没错，VMware的交换机是比Xenserver好一些，但他本是只是一个虚拟化产品，并且做得是够深度精简，几乎所有Linux

        的模块但VMware 没有IPtables的功能。

        OK，到这里我们的目标地址转换的功能已经找到了，那么另外一个还有一个接口从哪里来那？

        这个就很简单了，安装好Xenserver以后，大家只需要使用客户端连接上去以后，去看看他的网络接口就明白该如何去做了。

        温馨提示： 加入对外的网络连接的是你的Eth0接口，那么你就基于Eth0接口在创建一个接口就ok了，另外这个接口还必须设置

        IP地址，IP地址的作用是为运行在Xenserver上面的虚拟机提供网关功能，其实这里在新建一个接口就是Linux里面的子接口。

        到这里是不是既有防火墙又有两个接口又可以运行虚拟化又节省了成本那？

        另外在填写IPTables NAT chain的时候注意一下接口的设置，外网的IP地址在哪个接口上就设置out 端口是那个端口。


        iptables -t nat -A PREROUTING -p tcp -d 192.168.110.120 --dport 8022 -j DNAT --to-destination 192.168.205.110:22

        iptables -t nat -A POSTROUTING -s 192.168.205.0/24 -o xenbr0 -j SNAT  --to 192.168.110.120 