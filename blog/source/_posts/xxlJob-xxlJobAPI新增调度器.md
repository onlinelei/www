---
title: xxlJob:API 方式新增调度器
date: 2020-02-12 16:20:57
tags: xxlJob
---

#  一、API 方式新增调度器

xxlJob文档中 5.11.2提供了可以Api的方式对调度中心进行控制，原文如下：

>5.11.2 提供给业务的API服务：
1.任务列表查询；
2.任务新增；
3.任务更新；
4.任务删除；
5.任务启动；
6.任务停止；
7.任务触发；
>API服务位置：`com.xxl.job.admin.controller.JobInfoController.java`
API服务请求参考代码：可参考任务界面操作的ajax请求。任何ajax接口均可配置成为API服务，只需在待启用的API服务上添加 `@PermissionLimit(limit = false)` 注解取消登陆态拦截即可；



## 1.任务列表查询

接口地址：/jobinfo/pageList
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/pageList

请求参数：
``` json
jobGroup: 4
triggerStatus: -1
jobDesc: 
executorHandler: 
author: 
start: 0
length: 10
```



## 2. 任务新增

接口地址：/jobinfo/add
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/add

请求参数：(标星为必填项)
``` json
jobGroup: 4  // * 执行器，需要查询到自己的执行器id
jobDesc: 同步组织架构API 测试  // * 调度器描述信息
executorRouteStrategy: FIRST // * 路由策略:(FIRST:第一个,LAST:最后一个,ROUND:轮询,RANDOM:随机,CONSISTENT_HASH:一致性HASH,LEAST_FREQUENTLY_USED:最不经常使用,LEAST_RECENTLY_USED:最近最久未使用,FAILOVER:故障转移,BUSYOVER:忙碌转移,SHARDING_BROADCAST:分片广播)
cronGen_display: 0 0 1 * * ? // * Cron表达式
jobCron: 0 0 1 * * ? // * Cron表达式
glueType: BEAN  // * 运行模式(BEAN:BEAN,GLUE_GROOVY:GLUE(Java),GLUE_SHELL:GLUE(Shell),GLUE_PYTHON:GLUE(Python),GLUE_PHP:GLUE(PHP),GLUE_NODEJS:GLUE(Nodejs),GLUE_POWERSHELL:GLUE(PowerShell))
executorHandler: syncOrganizationJobHandler	// * JobHandler
executorBlockStrategy: SERIAL_EXECUTION // * 阻塞处理策略(SERIAL_EXECUTION:单机串行,DISCARD_LATER:丢弃后续调度,COVER_EARLY:覆盖之前调度)
childJobId: // 子任务ID
executorTimeout: 0 // 任务超时时间
executorFailRetryCount: 0 // 失败重试次数
author: qianlei // * 负责人
alarmEmail: suqianlei@gmail.com // 报警邮件
executorParam: 0 // 任务参数
glueRemark: GLUE代码初始化 // 
glueSource: // 
```


## 3. 任务更新

接口地址：/jobinfo/update
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/update

请求参数：
``` json
jobGroup: 4
jobDesc: 同步组织架构API 测试
executorRouteStrategy: FIRST
cronGen_display: 0 0 1 * * ?
jobCron: 0 0 1 * * ?
executorHandler: syncOrganizationJobHandler	
executorBlockStrategy: SERIAL_EXECUTION
childJobId: 
executorTimeout: 0
executorFailRetryCount: 0
author: qianlei
alarmEmail: suqianlei@gmail.com
executorParam: 0
id: 55
```



## 4. 任务删除

接口地址：/jobinfo/remove
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/remove

请求参数：
``` json
id: 55
```



## 5.任务启动

接口地址：/jobinfo/start
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/start

请求参数：
``` json
id: 55
```



## 6.任务停止

接口地址：/jobinfo/stop
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/stop

请求参数：
``` json
id: 55
```

## 7.任务触发

接口地址：/jobinfo/trigger
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/trigger

请求参数：
``` json
id: 55
executorParam: 0
```

## 8.其他接口


### 8.1查询注册节点

接口地址：/jobgroup/loadById
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobgroup/loadById

``` json
id: 4
```

### 8.2下次执行时间
接口地址：/jobinfo/nextTriggerTime
测试环境：http://172.16.117.2:12380/xxl-job-admin/jobinfo/nextTriggerTime

请求参数：
``` json
cron: 0 0 1 * * ?
```

# 二、直接操作数据库方式新增

或者直接操作数据库表增加任务，

任务表：xxl_job_info
执行器表：xxl_job_group

``` sql
-- 一下标星内容为必填项
-- * 商户名称
SET @groupName = 'LQ科技';
-- * 商户ID
SET @groupID = '7282937789';
-- * 创建人
SET @author = 'qianlei';
-- 报警邮件
SET @alarmEmail = 'qianlei@sunlands.com';

-- * 执行器主键ID (xxl_job_group中主键)
SET @jobGroup = 3;
-- * 路由策略:(FIRST:第一个,LAST:最后一个,ROUND:轮询,RANDOM:随机,CONSISTENT_HASH:一致性HASH,LEAST_FREQUENTLY_USED:最不经常使用,LEAST_RECENTLY_USED:最近最久未使用,FAILOVER:故障转移,BUSYOVER:忙碌转移,SHARDING_BROADCAST:分片广播)
SET @executorRouteStrategy = 'FIRST';
-- * 阻塞处理策略(SERIAL_EXECUTION:单机串行,DISCARD_LATER:丢弃后续调度,COVER_EARLY:覆盖之前调度)
SET @executorBlockStrategy = 'SERIAL_EXECUTION';
-- * 运行模式(BEAN:BEAN,GLUE_GROOVY:GLUE(Java),GLUE_SHELL:GLUE(Shell),GLUE_PYTHON:GLUE(Python),GLUE_PHP:GLUE(PHP),GLUE_NODEJS:GLUE(Nodejs),GLUE_POWERSHELL:GLUE(PowerShell))
SET @glueType = 'BEAN';
-- * 任务状态(1:开启，0关闭)
SET @triggerStatus = 1;


-- 创建job
INSERT INTO `xxl_job_info` (`job_group`, `job_cron`, `job_desc`, `add_time`, `update_time`, `author`, `alarm_email`, `executor_route_strategy`, `executor_handler`, `executor_param`, `executor_block_strategy`, `executor_timeout`, `executor_fail_retry_count`, `glue_type`, `glue_source`, `glue_remark`, `glue_updatetime`, `child_jobid`, `trigger_status`, `trigger_last_time`, `trigger_next_time`) VALUES
(@jobGroup, '0 0 1 * * ?', CONCAT('同步组织架构（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'syncOrganizationJobHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0 1 * * ?', CONCAT('A-重新分配机会的定时任务（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'aRetryAllocateOpportunityHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0 1 * * ?', CONCAT('B-重新分配机会的定时任务（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'bRetryAllocateOpportunityHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0 1 * * ?', CONCAT('C-重新分配机会的定时任务（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'cRetryAllocateOpportunityHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0/3 * * * ?', CONCAT('分配机会的定时任务（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'loopAllocateOpportunityHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0 3 ? * *', CONCAT('回收机会定时任务（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'recycleOpportunityTaskHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0/3 7-23 ? * *', CONCAT('话单同步（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'callRecordSynMinuteJobHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 10 * ? * *', CONCAT('每小时聚合微信聊天记录（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'wechatRecordHourMergeJobHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0 2 ? * *', CONCAT('微信有效跟进统计（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'wechatValidFollowJobHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '30 0/5 * ? * *', CONCAT('增量更新机会es（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'compareCacheOppJobHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 0 5 ? * *', CONCAT('重建机会es（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'reindexCacheOppJobHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0),
(@jobGroup, '0 10 0 * * ?', CONCAT('回收机会预警（',@groupName,'）'), NOW(), NOW(), @author, @alarmEmail, @executorRouteStrategy, 'recycleEarlyWarningHandler', @groupID, @executorBlockStrategy, 0, 0, @glueType, '', 'GLUE代码初始化', NOW(), '', @triggerStatus, 0, 0);
```