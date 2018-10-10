##rocketMQ命令

###1 准备

进入 `/RocketMQ/bin`

查看mqadmin 的已有命令:`sh mqadmin`

查看命令怎么使用 ：`sh mqadmin help 命令名称`

```
例如，查看 consumerProgress 的使用
sh mqadmin help consumerProgress

```

###2 命令
 
- 查询集群消息
`sh mqadmin clusterList -n localhost:9876`

- 查看topicList:
 `sh mqadmin topicList -n localhost:9876`


- 查看topic路由信息

`sh mqadmin topicRoute –n localhost:9876 –t topicName`
 
- 查看所有消费组group: `sh mqadmin consumerProgress -n localhost:9876`

- 查看指定消费组下的所有topic数据堆积情况： 
`sh mqadmin consumerProgress -n localhost:9876 -g consumerGroupName`

- 查看topic信息列表详情统计
`sh mqadmin topicstatus -n localhost:9876 -t topicTemp1`

- 根据消息ID查询消息
sh mqadmin queryMsgById –n localhost:9876 –i 6449371000002A9F0000000000000912

- 看topic是否有生产
 sh mqadmin topicStatus -n localhost:9876 -t T_BonusCountsNotifyDC


- 查看某个topic的消费者连接的IP
sh mqadmin consumerConnection -n localhost:9876 -g S_DH_ES_INDEX_DCALICE

- 查看某个topic的生产者IP
sh mqadmin producerConnection -n localhost:9876 -g P_AL_CASE_PRODUCER_BONUS_ALICE -t T_BonusCountsNotifyDC

- 查看消费进度
sh mqadmin consumerProgress -n localhost:9876 -g S_dcalice_CG_T_cmOrgNotify









- 新增topic
`sh mqadmin updateTopic –n localhost:9876 –c DefaultCluster –t topicTemp`
- 删除topic
`sh mqadmin deleteTopic –n localhost:9876 –c DefaultCluster –t topicTemp`

 %RETRY%S_dcalice_CG_T_cmOrgNotify

 

注：需指定ip和端口


参考：
1、[mq基本命令](https://blog.csdn.net/jenny8080/article/details/53466738)
2、[mq基本命令](https://www.cnblogs.com/gmq-sh/p/6232633.html)