# ProxyPool

一个带有打分机制的代理IP池

抓取了21个代理IP网址的高匿代理

环境：Windows 7、Python2.7、MySQL5.7

原理：


	1、使用Python抓取网页上的代理IP，去重检测后将可用的代理IP保存到数据库，并置分数为1，默认失败次数为0。


	2、定时从数据库中读取代理IP，对于检测到依然能用的代理进行+1分操作，不可用的代理-1分操作，并将失败次数+1


	3、定时删除数据库中失败次数>=3次的代理

	4、定时运行的方式使用的是Windows的任务计划,每30分钟运行一次，命令如下：
	   schtasks /create /sc minute /mo 30 /tn "代理IP池" /tr "路径Path\IP_Proxy_Spider.py"

	5、数据库建表规则：
	   mysql> show create table proxyippool;
	   +-------------+------------------------------------------------------------------------------------+
	   | Table       | Create Table                                                                       |
	   +-------------+------------------------------------------------------------------------------------+
	   | proxyippool | CREATE TABLE `proxyippool` (
	     `protocol` enum('http','https') NOT NULL,
	     `ip` varchar(15) NOT NULL,
	     `port` varchar(5) NOT NULL,
	     `speed` float(5,3) DEFAULT NULL,
	     `position` varchar(200) DEFAULT NULL,
	     `score` tinyint(4) NOT NULL,
	     `failtimes` tinyint(1) unsigned NOT NULL DEFAULT '0',
	     `lasttime` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
	     PRIMARY KEY (`ip`,`port`)
	   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
	   +-------------+------------------------------------------------------------------------------------+
	   1 row in set


BUG修正及说明：


	1、删除分数低于某个特定值的方式不可取，对于分数较高的代理，如果失效，则需要验证很多次之后才能被删除，不合理（更新已修复）

	2、新的问题待解决：考虑一种情况，如果某个代理IP成功100次，失败1次，那么是不是成功300次之后就被删除掉了了呢？

获得经验：

	1、从数据库中来看，80和8080端口的IP似乎更稳定


