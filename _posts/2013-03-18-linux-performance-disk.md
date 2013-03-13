---
layout : post
category : manage
tags : [Linux]
title : Linux 系统优化之Disk 
---


### 3.磁盘I/O性能评估

	在对磁盘I/O性能做评估之前，必须知道的几个方面是：
	
	熟悉RAID存储方式，可以根据应用的不同，选择不同的RAID方式，例如，如果一个应用经常有大量的读操作，可以选择RAID5方式构建磁盘阵列存储数据，如果应用有大量的、频繁的写操作，可以选择raid0存取方式，如果应用对数据安全要求很高，同时对读写也有要求的话，可以考虑raid01存取方式等等。
	尽可能用内存的读写代替直接磁盘I/O，使频繁访问的文件或数据放入内存中进行操作处理，因为内存读写操作比直接磁盘读写的效率要高千倍。
	将经常进行读写的文件与长期不变的文件独立出来，分别放置到不同的磁盘设备上。
	对于写操作频繁的数据，可以考虑使用裸设备代替文件系统。这里简要讲述下文件系统与裸设备的对比：
	
	使用裸设备的优点有：
	
	数据可以直接读写，不需要经过操作系统级的缓存，节省了内存资源，避免了内存资源争用。
	避免了文件系统级的维护开销，比如文件系统需要维护超级块、I-node等。
	避免了操作系统的cache预读功能，减少了I/O请求。
	
	使用裸设备的缺点是：
	
	数据管理、空间管理不灵活，需要很专业的人来操作。
	其实裸设备的优点就是文件系统的缺点，反之也是如此，这就需要我们做出合理的规划和衡量，根据应用的需求，做出对应的策略。
	
	下面接着介绍对磁盘IO的评估标准。
#### 3.1 sar -d命令组合
	
	通过“sar –d”组合，可以对系统的磁盘IO做一个基本的统计，请看下面的一个输出：
	
	[root@webserver ~]# sar -d 2 3
	Linux 2.6.9-42.ELsmp (webserver)        11/30/2008      _i686_  (8 CPU)

	11:09:33 PM  DEV   tps   rd_sec/s wr_sec/s  avgrq-sz  avgqu-sz  await  svctm   %util
	11:09:35 PM dev8-0  0.00  0.00     0.00      0.00      0.00      0.00   0.00    0.00

	11:09:35 PM  DEV   tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz  await   svctm   %util
	11:09:37 PM dev8-0  1.00  0.00     12.00     12.00      0.00     0.00    0.00    0.00

	11:09:37 PM   DEV   tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz  await  svctm  %util
	11:09:39 PM dev8-0  1.99   0.00    47.76     24.00     0.00      0.50    0.25    0.05

	Average:  DEV     tps    rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz  await  svctm   %util
	Average:  dev8-0  1.00   0.00      19.97     20.00      0.00     0.33    0.17    0.02

	对上面每项的输出解释如下：
	
	DEV表示磁盘设备名称。
	tps表示每秒到物理磁盘的传送数，也就是每秒的I/O流量。一个传送就是一个I/O请求，多个逻辑请求可以被合并为一个物理I/O请求。
	rd_sec/s表示每秒从设备读取的扇区数（1扇区=512字节）。
	wr_sec/s表示每秒写入设备的扇区数目。
	avgrq-sz表示平均每次设备I/O操作的数据大小（以扇区为单位）。
	avgqu-sz表示平均I/O队列长度。
	await表示平均每次设备I/O操作的等待时间（以毫秒为单位）。
	svctm表示平均每次设备I/O操作的服务时间（以毫秒为单位）。
	%util表示一秒中有百分之几的时间用于I/O操作。

	Linux中I/O请求系统与现实生活中超市购物排队系统有很多类似的地方，通过对超市购物排队系统的理解，可以很快掌握linux中I/O运行机制。比如：
	avgrq-sz类似与超市排队中每人所买东西的多少。
	avgqu-sz类似与超市排队中单位时间内平均排队的人数。
	await类似与超市排队中每人的等待时间。
	svctm类似与超市排队中收银员的收款速度。
	%util类似与超市收银台前有人排队的时间比例。

	对以磁盘IO性能，一般有如下评判标准：
	正常情况下svctm应该是小于await值的，而svctm的大小和磁盘性能有关，CPU、内存的负荷也会对svctm值造成影响，过多的请求也会间接的导致svctm值的增加。
	await值的大小一般取决与svctm的值和I/O队列长度以及I/O请求模式，如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢，此时可以通过更换更快的硬盘来解决问题。
	%util项的值也是衡量磁盘I/O的一个重要指标，如果%util接近100%，表示磁盘产生的I/O请求太多，I/O系统已经满负荷的在工作，该磁盘可能存在瓶颈。长期下去，势必影响系统的性能，可以通过优化程序或者通过更换更高、更快的磁盘来解决此问题。


#### 3.2 iostat –d命令组合
	
	通过“iostat –d”命令组合也可以查看系统磁盘的使用状况，请看如下输出：
	
	[root@webserver ~]#   iostat -d 2 3
	Linux 2.6.9-42.ELsmp (webserver)        12/01/2008      _i686_  (8 CPU)

	Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
	sda               1.87         2.58       114.12    6479462  286537372

	Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
	sda               0.00         0.00         0.00          0          0

	Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
	sda               1.00         0.00        12.00          0         24

	对上面每项的输出解释如下：
	
	Blk_read/s表示每秒读取的数据块数。
	Blk_wrtn/s表示每秒写入的数据块数。
	Blk_read表示读取的所有块数
	Blk_wrtn表示写入的所有块数。

	这里需要注意的一点是：上面输出的第一项是系统从启动以来到统计时的所有传输信息，从第二次输出的数据才代表在检测的时间段内系统的传输值。
	
	可以通过Blk_read/s和Blk_wrtn/s的值对磁盘的读写性能有一个基本的了解，如果Blk_wrtn/s值很大，表示磁盘的写操作很频繁，可以考虑优化磁盘或者优化程序，如果Blk_read/s值很大，表示磁盘直接读取操作很多，可以将读取的数据放入内存中进行操作。对于这两个选项的值没有一个固定的大小，根据系统应用的不同，会有不同的值，但是有一个规则还是可以遵循的：长期的、超大的数据读写，肯定是不正常的，这种情况一定会影响系统性能。
	
	“iostat –x”组合还提供了对每个磁盘的单独统计，如果不指定磁盘，默认是对所有磁盘进行统计，请看下面的一个输出：

	[root@webserver ~]#   iostat -x /dev/sda  2 3
	Linux 2.6.9-42.ELsmp (webserver)        12/01/2008      _i686_  (8 CPU)

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
				2.45    0.00    0.30    0.24    0.00   97.03

	Device: rrqm/s  wrqm/s  r/s  w/s  rsec/s  wsec/s avgrq-sz avgqu-sz   await  svctm  %util
	   sda   0.01     12.48    0.10  1.78  2.58   114.03    62.33   0.07    38.39   1.30   0.24

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
			   3.97    0.00    1.83    8.19    0.00   86.14

	Device:rrqm/s wrqm/s   r/s  w/s   rsec/s  wsec/s avgrq-sz avgqu-sz   await  svctm  %util
	   sda    0.00   195.00  0.00 18.00  0.00  1704.00    94.67     0.04    2.50   0.11   0.20

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               4.04    0.00    1.83    8.01    0.00   86.18

	Device: rrqm/s  wrqm/s  r/s  w/s   rsec/s   wsec/s avgrq-sz avgqu-sz     await  svctm  %util
	  sda    0.00     4.50    0.00   7.00   0.00    92.00    13.14     0.01    0.79   0.14   0.10

	这个输出基本与“sar –d”相同，需要说明的几个选项的含义为：
	
	rrqm/s表示每秒进行merged的读操作数目。
	wrqm/s表示每秒进行 merge 的写操作数目。
	r/s表示每秒完成读I/O设备的次数。
	w/s表示每秒完成写I/O设备的次数。
	rsec/s表示每秒读取的扇区数。
	wsec/s表示每秒写入的扇区数。

#### 3.3 vmstat –d 组合

	通过“vmstat –d”组合也可以查看磁盘的统计数据，情况下面的一个输出：

	[root@webserver ~]# vmstat -d 3 2|grep sda
	disk- ------------reads------------ ------------writes----------- -----IO------
		total  merged sectors    ms    total    merged   sectors      ms     cur    sec
	sda  239588 29282  6481862  1044442 4538678  32387680 295410812  186025580  0   6179
	disk- ------------reads------------ ------------writes----------- -----IO------
		total  merged  sectors  ms    total     merged    sectors     ms     cur   sec
	sda  239588 29282  6481862 1044442 4538680   32387690 295410908 186025581  0   6179
	
	这个输出显示了磁盘的reads、writes和IO的使用状况。