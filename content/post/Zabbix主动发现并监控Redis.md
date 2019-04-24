---
title: "Zabbix主动发现并监控Redis"
date: 2019-04-24T18:00:44+08:00
tags: [“zabbix”,"redis"]
categories: [“zabbix”,"redis"]
draft: false
---

## 准备工作
最近需要监控redis，就Google搜索了一下，发现了这个项目，但是是英文的，我就把实际操作的步骤整理成文章记录一下：   
**Github地址**：  
https://github.com/allenta/zabbix-template-for-redis

安装zabbix-sender  
`yum -y install zabbix-sender`

## 英文README说明

This is a Zabbix template + discovery & sender script useful to monitor Redis Server & Redis Sentinel instances:  
## 1. Copy zabbix-redis.py to /usr/local/bin/.
拷贝文件夹里面的zabbix-redis.py 到 /usr/local/bin/ 目录下面，并授予执行权限  
`cp zabbix-redis.py /usr/local/bin/zabbix-redis.py`
`chmod +x zabbix-redis.py` 

### 2. 添加UserParameter到zabbix的客户端：

Add the redis_server.discovery and / or redis_sentinel.discovery user parameters to Zabbix:  
将redis_server.discovery和/或redis_sentinel.discovery用户参数添加到Zabbix：

UserParameter=redis_server.discovery[*],/usr/local/bin/zabbix-redis.py -i '$1' -t server discover $2 2> /dev/null
UserParameter=redis_sentinel.discovery[*],/usr/local/bin/zabbix-redis.py -i '$1' -t sentinel discover $2 2> /dev/null
	

```	
cat /etc/zabbix/zabbix_agentd.d/userparameter_redis.conf 
UserParameter=redis_server.discovery[*],/usr/local/bin/zabbix-redis.py -i '$1' -t server --redis-password admin discover $2 2> /dev/null
UserParameter=redis_sentinel.discovery[*],/usr/local/bin/zabbix-redis.py -i '$1' -t sentinel --redis-password admin  discover $2 2> /dev/null
```
	
### 3. 定时任务添加
Add required jobs to the zabbix user crontab (beware of the -i, -t and -s options). This will submit Redis Server and / or Redis Sentinel metrics through Zabbix Sender:

将所需作业添加到zabbix用户crontab（注意-i，-t和-s选项）。 这将通过Zabbix Sender提交Redis Server和/或Redis Sentinel指标：  
```
* * * * * /usr/local/bin/zabbix-redis.py -i '6379, 6380, 6381, 7000, 7001, 7002, 7003, 7004, 7005' -t server send -c /etc/zabbix/zabbix_agentd.conf -s dev > /dev/null 2>&1
* * * * * /usr/local/bin/zabbix-redis.py -i '26379, 26380, 26381' -t sentinel send -c /etc/zabbix/zabbix_agentd.conf -s dev > /dev/null 2>&1
```

由于没有哨兵模式：  
```
0-59 * * * * /usr/local/bin/zabbix-redis.py -i '6379' -t server --redis-password admin send -c /etc/zabbix/zabbix_agentd.conf -s dev > /dev/null 2>&1
# * * * * * /usr/local/bin/zabbix-redis.py -i '26379, 26380, 26381' -t sentinel send -c /etc/zabbix/zabbix_agentd.conf -s dev > /dev/null 2>&1
```

### 4. 导入模板到zabbix server里面
Import the required templates (template-app-redis-server.xml and / or template-app-redis-sentinel.xml files).

In 'Configuration > Templates' click on 'Import' and select template-app-redis-server.xml.  
In 'Configuration > Templates' click on 'Import' and select template-app-redis-sentinel.xml.


### 5. 监控设置

Add an existing / new host to the Redis servers group and link it to the right template (Template App Redis Server for Redis Server and Template App Redis Sentinel for Redis Sentinel). Beware depending on the used template you must set a value for the {$REDIS_SERVER_LOCATIONS} or {$REDIS_SENTINEL_LOCATIONS} macro (comma-delimited list of Redis instances; port, host:port and unix:///path/to/socket formats are allowed). There are defined macros for Template App Redis Server and Template App Redis Sentinel.  

将现有/新主机添加到Redis服务器组并将其链接到正确的模板（用于Redis服务器的模板应用程序Redis服务器和用于Redis Sentinel的模板应用程序Redis Sentinel）。 请注意，根据使用的模板，您必须为{$ REDIS_SERVER_LOCATIONS}或{$ REDIS_SENTINEL_LOCATIONS}宏设置一个值（以逗号分隔的Redis实例列表; port，host：port和unix：/// path / to / socket格式为允许）。 为Template App Redis服务器和模板应用程序Redis Sentinel定义了宏。    

添加变量参数：  
	
	In 'Configuration > Hosts' click on 'Create host':
		○ Host name: dev
		○ Group: Redis servers
		○ Linked templates: Template App Redis Server and / or Template App Redis Sentinel
		○ Macros: {$REDIS_SERVER_LOCATIONS} => 6379, 6380, 6381, 7000, 7001, 7002, 7003, 7004, 7005 and/or
		○  {$REDIS_SENTINEL_LOCATIONS} => 26379, 26380, 26381

![屏幕快照 2019-04-24 下午6.14.23](http://wx4.sinaimg.cn/mw690/79b5b049gy1g2dw2750rej20ph09mgmo.jpg)
![屏幕快照 2019-04-24 下午6.14.34](http://ws4.sinaimg.cn/mw690/79b5b049gy1g2dw2fe2h1j20pt0a2wft.jpg)
	
	For Template App Redis Server:
		○ {$REDIS_SERVER_EVICTED_KEYS_ALLOWED}
		○ {$REDIS_SERVER_HISTORY_STORAGE_PERIOD}
		○ {$REDIS_SERVER_INSTANCES}
		○ {$REDIS_SERVER_KEEP_LOST_RESOURCES_PERIOD}
		○ {$REDIS_SERVER_LOCATIONS}
		○ {$REDIS_SERVER_MIN_UPTIME_AFTER_RESTART}
		○ {$REDIS_SERVER_TREND_STORAGE_PERIOD}
		○ {$REDIS_SERVER_UPDATE_INTERVAL_DISCOVERY}
		○ {$REDIS_SERVER_UPDATE_INTERVAL_ITEM}

	For Template App Redis Sentinel:
		○ {$REDIS_SENTINEL_HISTORY_STORAGE_PERIOD}
		○ {$REDIS_SENTINEL_INSTANCES}
		○ {$REDIS_SENTINEL_LOCATIONS}
		○ {$REDIS_SENTINEL_MIN_UPTIME_AFTER_RESTART}
		○ {$REDIS_SENTINEL_TREND_STORAGE_PERIOD}
		○ {$REDIS_SENTINEL_UPDATE_INTERVAL_DISCOVERY}
		○ {$REDIS_SENTINEL_UPDATE_INTERVAL_ITEM}

	It's also possible to use contexts on macros, for example:
		○ {$REDIS_SERVER_HISTORY_STORAGE_PERIOD:cluster-cluster_slots_assigned}
		○ {$REDIS_SENTINEL_HISTORY_STORAGE_PERIOD:server-uptime_in_seconds}
		
### 6. Adjust triggers and trigger prototypes according with your preferences.

根据您的喜好调整触发器并触发原型

## 最后看看监控出图
![屏幕快照 2019-04-24 下午6.18.23](http://wx4.sinaimg.cn/mw690/79b5b049gy1g2dw6f67djj218z0f1q4o.jpg)

ok，完工！

