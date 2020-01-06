## task_loader 任务加载模块
#### 1. 可以起多个节点，通过配置区分不同节点；模块配置有：
```javascript
{
	node: conf.cluster_node,// 集群节点名称:打印系统日志、task表使用到
	collection: "bi_task", // 任务数据表名
	intervalMs: 10 * 1000,// 轮询间隔ms
	limit: 10, // 单次启动限制个数
	limitTasks: []// 限定任务
	tasks : Map() //运行中的任务map
	status: READY/RUNNING //运行状态
	stopSignal //退出信号
}
```
#### 2. 轮询从mongo里读取符合条件的任务，条件如下：
```javascript
{
	 status: READY, //任务状态
	 IsDeleted: false, 
	 nodes.name: {'$ne': this,node}, //此节点还没加载任务
	 _id:  {'$in': this,limitTasks}, //限定某些任务，没有限定时默认不限
	 limit: this.limit, //每次读取的任务个数
}
```
#### 3. 加载从mongo里读取出来的任务
	(1). 已加载的任务，先unload，再加载;所以需要重启的任务，可以把task.status 设为ready,清空nodes.name字段;
	(2). 调用task模块创建任务、任务检查,检查内容包括任务的各项配置是否必填、配置格式是否合法;检查通过，则启动任务、并把任务保存在map里,然后进入步骤(3);否则任务加载失败,然后进入步骤(4).
	(3). 加载成功，更新mongo表的task状态为running;
	(4). 加载失败，task状态设为error,并记录错误信息condition。
#### 4. 暂停任务
	(1). 接收到退出信号，设置退出信号，然后等待步骤2、3的轮询任务结束。
	(2). 遍历运行中的任务map，停止任务、从任务map移除任务、任务状态设为READY.

## task 任务模块
#### 1. 运行任务
		(1). 遍历inputs,订阅多个消息队列。
		(2). 有两个消息输入源：订阅kafka/redis 对应的topic; 订阅内建消息池的消息，用taskId对应。
#### 2. 订阅消息队列的消息。
		(1). 读取消息队列的消息，队列里的消息经过了input模块的解析处理,具体处理过程看input模块;
		(2). 判断msg.taskId是否存在，不存在则赋值 msg.taskId = this._id；注：从消息池进来的消息有taskId，其他来源的没有。
#### 3. 消息处理，具体消息处理过程如下：
		(1). 消息过滤，用到task对象的filter，filter是一个方法。
			过滤通过，进入下一步。
			过滤不通过，丢弃消息，结束消息处理。
		(2). 消息追踪，追踪过程：
			a. 如果没有utmId(对应msg.common.utm_id,这个在解析消息的parse方法里赋值)
				如果有 utm_mapping_db ，则通过mysql找回，utm_mapping_db是表名，查询条件是 common.id；如果没有找到，丢弃这条消息，结束消息处理。
				没有 utm_mapping_db,则：
					有 utm_mapping_id ， 则通过redis找回，utm_mapping_id是key；
					如果还没有找到utmId, 则通过ip ua找回
			b. 找到utmId，则往下走。否则丢弃这条消息，结束消息处理。
			c. 根据utmId，通过redis找回源数据,保存在msg.trace；
			d. utm_mapping_id && 通过ip ua找回的utmId  && umtId && 在归因时间之内 ,则将 utm_mapping_id 和 umtId的映射保存到redis。
		(3). 追踪后过滤
			任务用户过滤：trace.dmp_user_id === task.user_id,通过往下走，否则丢弃消息，结束消息处理。
		(4). 消息补全
			有三种方式(看task对象extras字段说明)补全数据;
			补全结果情况：
				a.server 返回 delay ,延迟处理消息，设置 expires/time,放入消息池;
				b.server 返回 abandon,则丢弃消息，结束消息处理；
				c. 其他情况出错，放入消息池。
				d. 成功补全消息，往下走。
		(5). 消息操作，对应task对象的operations,输出消息到kafka、redis、mysql。
#### 4. 步骤3中，消息处理过程出错或者需要延迟处理，则做如下处理：
		(1). 设置消息出错次数、下次处理的时间点、消息最迟的处理时间，然后放进消息池
#### 5. task 对象字段说明
```javascript
	{
		input 输入源，必填，数组或字符串，对应配置的app.input对象的属性名。
		filter 消息过滤方法，必须是方法，可选。
		user_id 所属账号，可选，字符串
		extras [] 需要补全的信息，数组
		[{
			field //字段名	必填
			// server method func 三者必填一个
			server // 查询数据的服务 对应 module/query/;
			method //方法名，已定义的工具类方法modle/msg_tools.js
			func //一个可执行的方法
			args {//传递给func server method 的参数
				map
				keys
				period
			}
		}]
		operations [] //对消息要进行的操作。
		{
			condition //执行操作的前提条件	可选，一个可执行的方法
			outputs: [] 必填
				{
					target:  //对应处理模块 /module/output/	必填，调用模块的do 方法输出消息。
					fields: []
						{
							name:  //字段名
							value: //值
							ttl: 1 //有效期
						}
					values: {

					}
				}
		}

	}
```
## 消息输入模块
#### 1.输入源:kafka\redis，订阅多个集群多个topic的消息。
#### 2.不同输入源，由不同模块处理，通过配置设置
		{
            module: 'kafka/bird', //对应处理的模块，module/input/
            cluster: 'comm_user_behaviors', //集群
            topic: 'BIRD_SH'
        } //多个集群 <--对应--> 多个topic <--对应--> 多个业务类型日志(比如手机银行登录日志 需要监听两个topic LA04_L_Mobile_Login_FLM_SZ\LA04_L_Mobile_Login_FLM_SH)
#### 3. 处理模块的parse方法，解析消息，获取统一数据放到common字段里。
#### 4. 一个tasks 有多个input , 一个input 对应一个消息队列，有多个队列订阅同一个topic；一个topic1消息进来， 会 put 到所有topic === topic1 的队列。
#### 5. tasks的input字段，对应配置的 app.input 对象的属性。

## 消息输出模块
#### 1. 输出到：kafka/redis/mysql;
#### 2. 不同输出源，由不同模块处理，通过配置设置，对应task的operations设置(详情看task对象operations字段说明);
#### 3. kafka: 需要统计服务入库的消息;
#### 4. redis: 存在依赖的消息，把相关数据保存到redis,供其它消息使用;
#### 5. mysql: 一些T+1数据无法从数仓直接获取，需要数仓T+1作业获取到相应数据后，才能生成T+1的数据。


## 内建消息池模块
#### 1. 延迟处理和系统处理出错的消息，会放到消息池,保存在mongo，再轮询读取消息处理；
#### 2. 读取消息过滤条件
```javascript 
{
	status: MsgStatus,Ready, //消息有两种状态：准备就绪1，已消费2,
	taskId: {"$in": taskIds},
	time: {"$lt": now},
	errorTimes: {"$lte": this,errorLimit},
	$or: [{expires: {"$gt": now}}, {expires: null}]
}
```
#### 3. 将消息放到对应的taskId队列。
####