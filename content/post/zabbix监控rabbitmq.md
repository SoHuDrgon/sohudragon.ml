---
title: "Zabbix监控rabbitmq"
date: 2019-04-26T14:31:47+08:00
tags: [“zabbix”,"rabbitmq"]
categories: [“zabbix”,"rabbitmq"]
draft: false
---
## 文章介绍
近期上线了rabbitmq，需要在zabbix上进行监控设置，所以Google搜索到一个项目，采用这个项目配置了监控rabbitmq。

[开源项目地址](https://github.com/jasonmcintosh/rabbitmq-zabbix)

由于作者开始想把监控转向Prometheus了，很久都没有更新了。但是对于我来说够用了。


## rabbitmq监控要点

具体监控什么东西，主要是rabbitmq-web监控页面的overview内容
![屏幕快照 2019-04-26 下午2.58.42](http://wx4.sinaimg.cn/mw690/79b5b049gy1g2g3x96y62j20qv0na0vh.jpg)  

还有就是队列堆积数，如果超过某个数值，比如5000个就立马报警

因为SNMP插件不是官方支持的插件，并且基于rabbitmqctl的监视器相比之下真的很慢。

所以作者就写了python脚本来监控rabbitmq。

## rabbitmq监控部署步骤
### 克隆项目到本地
`git clone https://github.com/jasonmcintosh/rabbitmq-zabbix.git`  

### 脚本复制
复制项目中scripts目录中所以文件移动至 zabbix_agentd 端服务器的/etc/zabbix/scripts/  目录下  
```
mkdir /etc/zabbix/scripts

cp rabbitmq-zabbix/scripts/rabbitmq/* /etc/zabbix/scripts/
ls /etc/zabbix/scripts/
api.py  list_rabbit_nodes.sh  list_rabbit_queues.sh  list_rabbit_shovels.sh  rabbitmq-status.sh
```


注：放置文件的服务器需要能与rabbitmq服务器通讯  


### 配置zabbix-agent扩展配置文件

zabbix_agentd 扩展配置文件目录需要打开  
```
grep "^Include" /etc/zabbix/zabbix_agentd.conf 
Include=/etc/zabbix/zabbix_agentd.d/
```

复制项目中的zabbix-rabbitmq.conf到/etc/zabbix/zabbix_agentd.d/  
`cp rabbitmq-zabbix/zabbix_agentd.d/zabbix-rabbitmq.conf /etc/zabbix/zabbix_agentd.d/userparameter_rabbitmq.conf`

查看配置文件内容，保证里面各脚本的路径与我们移动的目录一致。  
```
# cat /etc/zabbix/zabbix_agentd.d/userparameter_rabbitmq.conf 
UserParameter=rabbitmq.discovery_queues,/etc/zabbix/scripts/rabbitmq/list_rabbit_queues.sh
UserParameter=rabbitmq.discovery_shovels,/etc/zabbix/scripts/rabbitmq/list_rabbit_shovels.sh
UserParameter=rabbitmq.discovery_nodes,/etc/zabbix/scripts/rabbitmq/list_rabbit_nodes.sh
UserParameter=rabbitmq[*],/etc/zabbix/scripts/rabbitmq/rabbitmq-status.sh $1 $2 $3
```

### 导入模板到zabbix server服务器
In 'Configuration > Templates' click on 'Import' and select rabbitmq.template.xml  
如图所示：  
![屏幕快照 2019-04-26 下午3.30.23](http://ws1.sinaimg.cn/mw690/79b5b049gy1g2g3xm39nfj20ld0gjwfy.jpg)

### 修改zabbix-agent.conf配置文件
修改 zabbix_agentd 端和 zabbix_server 端，修改Timeout超时时间为10秒，默认为3秒，因为脚本有的执行时间超过3秒，这样会超时，无法获取数据，定义具体多大，视情况而定，最大30秒
```
# grep "^Timeout" /etc/zabbix/zabbix_agentd.conf
Timeout=10
```

### 配置.rab.auth
在api.py文件所在目录创建一个文件，名称为“.rab.auth”注意，此文件是隐藏文件，前面有个点，文件内容为  
```
USERNAME=guest    
PASSWORD=guest
CONF=/etc/zabbix/zabbix_agentd.conf
LOGLEVEL=DEBUG
LOGFILE=/var/log/zabbix/rabbitmq_zabbix.log
PORT=15672
```
您还可以在此文件中添加过滤器以限制监视哪些队列。此项是JSON编码的字符串。该格式为其使用提供了一些灵活性。您可以提供单个对象或要过滤的对象列表。可用的键是：*status，node，name，consumers，vhost，durable，exclusive_consumer_tag，auto_delete，memory，policy*    


例如，以下过滤器可以找到所有持久队列： *FILTER='{"durable": true}'*  

要仅为给定的vhost使用持久队列，过滤器将是： *FILTER='{"durable": true, "vhost": "mine"}'*  

要提供队列名称列表，过滤器将是： *FILTER='[{"name": "mytestqueuename"}, {"name": "queue2"}]'*  

要调试任何潜在的问题，请确保日志目录存在并且可以由zabbix写入，然后在.rab.auth文件中设置LOGLEVEL = DEBUG并且您将获得非常详细的输出   

### 修改api.py文件

全文替换　*/etc/zabbix/zabbix_agentd.conf*　此路径为你自己的路径，如果相同就不必替换    
全文替换　*/var/log/zabbix/rabbitmq_zabbix.log*　此路径为你自己的路径    

修改 RabbitMQAPI 类中的 __init__ 方法中:    
**user_name:rabbitmq**		管理界面登录用户名  
**password:rabbitmq** 		管理界面登录密码  
**host_name:rabbitmq**		管理界面地址  
**port:rabbitmq**			管理界面端口  
**conf:zabbix_agentd** 		配置文件路径  
**senderhostname：**			此选项为zabbix_sender往zabbix_server推送数据的地址，默认此模板是按“Zabbix客户端（主动式）”把数据传输到服务端的，我推荐使用服务端主动式，所以这里默认即可  
**protocol**:rabbitmq管理界面使用协议，比如http或者https，默认为http    
```
class RabbitMQAPI(object):
    '''Class for RabbitMQ Management API'''

    def __init__(self, user_name='zabbix', password='zabbix', host_name='172.16.xx.xx',
                 port=15672, conf='/etc/zabbix/zabbix_agentd.conf', senderhostname=None, protocol='http'):
        self.user_name = user_name
        self.password = password
        self.host_name = host_name or socket.gethostname()
        self.port = port
        self.conf = conf or '/etc/zabbix/zabbix_agentd.conf'
        self.senderhostname = senderhostname or socket.gethostname()
        self.protocol = protocol or 'http'
```

### 权限配置
授权一个zabbix监控rabbitmq的用户：  
```
rabbitmqctl add_user zabbix pass
rabbitmqctl set_user_tags zabbix monitoring
rabbitmqctl set_permissions -p / zabbix '.*' '.*' '.*'
```

修改api.py文件所在目录的所有文件，包括.rab.auth隐藏文件的所属组和所属主为zabbix，然后修改权限为755   
`chmod 755 .rab.auth`   
附上我的.rab.auth  
```
cat .rab.auth
USERNAME=zabbix
PASSWORD=zabbix
CONF=/etc/zabbix/zabbix_agentd.conf
LOGLEVEL=DEBUG
LOGFILE=/var/log/zabbix/rabbitmq_zabbix.log
PORT=15672
```

### Macros配置

您可以通过更改以下Macros来调整消息量的关键和警告级别的值：  

>RABBIT_QUEUE_MESSAGES_CRIT定义队列中消息量的临界值。默认设置为200000条消息
>RABBIT_QUEUE_MESSAGES_WARN定义队列中消息量的警告值。默认设置为100000条消息

### 配置完成后启动zabbix-agent客户端

`# /etc/init.d/zabbix-agent restart `     


### 完成监控后如图
![屏幕快照 2019-04-26 下午5.35.43](http://ws4.sinaimg.cn/mw690/79b5b049gy1g2g67k0to8j20vp0fi76j.jpg)

## 注意事项
脚本是根据python2.x编写的.  
服务器的hosts文件一定要配好.  
调试方法：  
```
./list_rabbit_nodes.sh 
{"data": [{"{#NODETYPE}": "disc", "{#NODENAME}": "xxx-vapp-xxx"}]}
```
或者去zabbix server执行:  
`zabbix_get -s 172.16.xx.xx -k rabbitmq.discovery_queues`