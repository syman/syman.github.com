---
layout : post
category : manage
tags : [Linux]
title : Linux 系统优化之Network
---

### 4. 网络性能评估

	网络性能的好坏直接影响应用程序对外提供服务的稳定性和可靠性，监控网络性能，可以从以下几个方面进行管理和优化。

#### 4.1 通过ping命令检测网络的连通性

	如果发现网络反应缓慢，或者连接中断，可以通过ping来测试网络的连通情况，请看下面的一个输出：

	[root@webserver ~]# ping 10.10.1.254
	PING 10.10.1.254 (10.10.1.254) 56(84) bytes of data.
	64 bytes from 10.10.1.254: icmp_seq=0 ttl=64 time=0.235 ms
	64 bytes from 10.10.1.254: icmp_seq=1 ttl=64 time=0.164 ms
	64 bytes from 10.10.1.254: icmp_seq=2 ttl=64 time=0.210 ms
	64 bytes from 10.10.1.254: icmp_seq=3 ttl=64 time=0.178 ms
	64 bytes from 10.10.1.254: icmp_seq=4 ttl=64 time=0.525 ms
	64 bytes from 10.10.1.254: icmp_seq=5 ttl=64 time=0.571 ms
	64 bytes from 10.10.1.254: icmp_seq=6 ttl=64 time=0.220 ms
	--- 10.10.1.254 ping statistics ---
	7 packets transmitted, 7 received, 0% packet loss, time 6000ms
	rtt min/avg/max/mdev = 0.164/0.300/0.571/0.159 ms, pipe 2

	在这个输出中，time值显示了两台主机之间的网络延时情况，如果此值很大，则表示网络的延时很大，单位为毫秒。在这个输出的最后，是对上面输出信息的一个总结，packet loss表示网络的丢包率，此值越小，表示网络的质量越高。


	4.2 通过netstat –i组合检测网络接口状况
	
	netstat命令提供了网络接口的详细信息，请看下面的输出：
	
	[root@webserver ~]# netstat -i
	Kernel Interface table
	Iface MTU  Met RX-OK     RX-ERR RX-DRP RX-OVR   TX-OK    TX-ERR TX-DRP TX-OVR       Flg
	eth0  1500  0 1313129253  0      0       0     1320686497    0      0      0        BMRU
	eth1  1500  0 494902025   0      0       0     292358810     0      0      0        BMRU
	lo   16436  0 41901601    0      0       0     41901601      0      0      0        LRU
	
	对上面每项的输出解释如下：
	Iface表示网络设备的接口名称。
	MTU表示最大传输单元，单位字节。
	RX-OK/TX-OK表示已经准确无误的接收/发送了多少数据包。
	RX-ERR/TX-ERR表示接收/发送数据包时产生了多少错误。
	RX-DRP/TX-DRP表示接收/发送数据包时丢弃了多少数据包。
	RX-OVR/TX-OVR表示由于误差而遗失了多少数据包。
	
	Flg表示接口标记，其中：
	L：表示该接口是个回环设备。
	B：表示设置了广播地址。
	M：表示接收所有数据包。
	R：表示接口正在运行。
	U：表示接口处于活动状态。
	O：表示在该接口上禁用arp。
	P：表示一个点到点的连接。

	正常情况下，RX-ERR/TX-ERR、RX-DRP/TX-DRP和RX-OVR/TX-OVR的值都应该为0，如果这几个选项的值不为0，并且很大，那么网络质量肯定有问题，网络传输性能也一定会下降。
	当网络传输存在问题是，可以检测网卡设备是否存在故障，如果可能，可以升级为千兆网卡或者光纤网络，还可以检查网络部署环境是否合理。

#### 4.3 通过netstat –r组合检测系统的路由表信息

	在网络不通，或者网络异常时，首先想到的就是检查系统的路由表信息，“netstat –r”的输出结果与route命令的输出完全相同，请看下面的一个实例：

	[root@webserver ~]#  netstat -r
	Kernel IP routing table
	Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
	10.10.1.0       *               255.255.255.0   U         0   0       0  eth0
	192.168.200.0   *               255.255.255.0   U         0   0       0  eth1
	169.254.0.0     *               255.255.0.0     U         0   0       0  eth1
	default         10.10.1.254     0.0.0.0         UG        0   0       0  eth0
	
	关于输出中每项的具体含义，已经在前面章节进行过详细介绍，这里不再多讲，这里我们重点关注的是default行对应的值，default项表示系统的默认路由，对应的网络接口为eth0。


####4.4 通过sar –n组合显示系统的网络运行状态

	sar提供四种不同的选项来显示网络统计信息，通过“-n”选项可以指定4个不同类型的开关：DEV、EDEV、SOCK和FULL。DEV显示网络接口信息，EDEV显示关于网络错误的统计数据，SOCK显示套接字信息，FULL显示所有三个开关。请看下面的一个输出：
	
	[root@webserver ~]# sar -n DEV 2 3
	Linux 2.6.9-42.ELsmp (webserver)        12/01/2008      _i686_  (8 CPU)

	02:22:31 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
	02:22:33 PM        lo     31.34     31.34     37.53     37.53      0.00      0.00      0.00
	02:22:33 PM      eth0    199.50    279.60     17.29    344.12      0.00      0.00      0.00
	02:22:33 PM      eth1      5.47      4.98      7.03      0.36      0.00      0.00      0.00
	02:22:33 PM      sit0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

	02:22:33 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
	02:22:35 PM        lo     67.66     67.66     74.34     74.34      0.00      0.00      0.00
	02:22:35 PM      eth0    159.70    222.39     19.74    217.16      0.00      0.00      0.00
	02:22:35 PM      eth1      3.48      4.48      0.44      0.51      0.00      0.00      0.00
	02:22:35 PM      sit0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

	02:22:35 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
	02:22:37 PM        lo      4.52      4.52      9.25      9.25      0.00      0.00      0.00
	02:22:37 PM      eth0    102.51    133.67     20.67    116.14      0.00      0.00      0.00
	02:22:37 PM      eth1     27.14     67.34      2.42     89.26      0.00      0.00      0.00
	02:22:37 PM      sit0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

	Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
	Average:           lo     34.61     34.61     40.48     40.48      0.00      0.00      0.00
	Average:         eth0    154.08    212.15     19.23    226.17      0.00      0.00      0.00
	Average:         eth1     11.98     25.46      3.30     29.85      0.00      0.00      0.00
	Average:         sit0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
	
	对上面每项的输出解释如下：
	
	IFACE表示网络接口设备。
	rxpck/s表示每秒钟接收的数据包大小。
	txpck/s表示每秒钟发送的数据包大小。
	rxkB/s表示每秒钟接收的字节数。
	txkB/s表示每秒钟发送的字节数。
	rxcmp/s表示每秒钟接收的压缩数据包。
	txcmp/s表示每秒钟发送的压缩数据包。
	rxmcst/s表示每秒钟接收的多播数据包。
	
	通过“sar –n”的输出，可以清楚的显示网络接口发送、接收数据的统计信息。此外还可以通过“sar -n EDEV 2 3”来统计网络错误信息等。
