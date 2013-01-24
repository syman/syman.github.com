---
layout : post
category : manage
tags : [VM]
title : 开发人员必须知道的Linux 命令
---
[思维导图文件下载](#)
# cat – 连接文件，并输出结果

            jfields$ cat order.out.log   
			8:22:19 111, 1, Patterns of Enterprise Architecture, Kindle edition, 39.99  
			8:23:45 112, 1, Joy of Clojure, Hardcover, 29.99  
			8:24:19 113, -1, Patterns of Enterprise Architecture, Kindle edition, 39.99 
		
# sort – 文件里的文字按行排序

			jfields$ cat order.* | sort  
			8:22:19 111, 1, Patterns of Enterprise Architecture, Kindle edition, 39.99  
			8:22:20 111, Order Complete  
			8:23:45 112, 1, Joy of Clojure, Hardcover, 29.99  
			8:23:50 112, Order sent to fulfillment  
			8:24:19 113, -1, Patterns of Enterprise Architecture, Kindle edition, 39.99  
			8:24:20 113, Refund sent to processing 
			
			就像上面例子显示的，文件里的数据已经经过排序。对于一些小文件，你可以读取整个
			文件来处理它们，然而，真正的log文件通常有大量的内容，你不能不考虑这个情况。
			此时你应该考虑过滤出某些内容，把cat、sort后的内容通过管道传递给过滤工具。

# grep, egrep, fgrep – 打印出匹配条件的文字行

        假设我们只对Patterns of Enterprise Architecture这本书的订单感兴趣。
		
		使用grep，我们能限制只输出含有Patterns字符的订单。
        
        jfields$ cat order.* | sort | grep Patterns  
		8:22:19 111, 1, Patterns of Enterprise Architecture, Kindle edition, 39.99  
		8:24:19 113, -1, Patterns of Enterprise Architecture, Kindle edition, 39.99 
		
		假设退款订单113出了一些问题，你希望查看所有相关订单——你又需要使用grep了。
		
		jfields$ cat order.* | sort | grep ":\d\d 113, "  
		8:24:19 113, -1, Patterns of Enterprise Architecture, Kindle edition, 39.99  
		8:24:20 113, Refund sent to processing 