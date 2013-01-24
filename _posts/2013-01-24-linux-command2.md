---
layout : post
category : manage
tags : [Linux]
title : Linux Shell 常用小技巧
---
[思维导图文件下载](#)



### 1.按内存从大到小排列进程:  
	
	ps -eo "%C : %p : %z : %a"|sort -k5 -nr

### 2.查看当前有哪些进程；查看进程打开的文件:
	
	ps -A ；lsof -p PID

### 3.获取当前IP地址（从中学习grep,awk,cut的作用）
	
	ifconfig eth0 |grep "inet addr:" |awk '{print $2}'|cut -c 6-

### 4.统计每个单词出现的频率，并排序

	awk '{arr[$1]+=1 }END{for(i in arr){print arr"\t"i}}' 文件名 | sort -rn

### 5.显示10条最常用的命令

	sed -e "s/| /\n/g" ~/.bash_history | cut -d ' ' -f 1 | sort | uniq -c | sort -nr | head

### 6.杀死Nginx进程(杀死某一进程)

	ps -ef|grep -v grep |grep nginx|awk '{print $2}' 或
	
	for i in `ps aux | grep nginx | grep -v grep | awk {'print $2'}` ; do kill $i; done

### 7.列出当前文件夹目录大小，以G，M，K显示。
	
	du -b --max-depth 1 | sort -nr | perl -pe 's{([0-9]+)}{sprintf"%.1f%s", $1>=2**30? ( $1/2**30, "G" ) : $1>=2**20? ( $1/2**20, "M" ) : $1>=2**10? ( $1/2**10, "K" ) : ( $1, "" ）}e'

	du -hs $(du -sk ./`ls -F |grep /` |sort -nr |awk '{print $NF}') 也可 以实现，不过不是特别完美。但好记。

### 8.清空linux buffer cache
	
	sync && echo 3 > /proc/sys/vm/drop_caches

### 9.将当前目录文件名全部转换成小写

	for i in *; do mv "$i" "$(echo $i|tr A-Z a-z)"; done

### 10.消除vim中的^M的几种方法

	1)dos2uninx filename
	2)sed -e 's/^M//' filename
	3)vim中 :s/^M//gc
	4)col -bx < dosfile > newfile
	5)tr -s "\r\n" "\n" < file > newfile

### 11. 清除所有arp缓存
	
	arp -n|awk '/^[1-9]/ {print "arp -d "$1}'|sh

### 12. 绑定已知机器的arp地址

	cat /proc/net/arp | awk '{print $1 " " $4}' |sort -t. -n +3 -4 > /etc/ethers