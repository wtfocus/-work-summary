[toc]

## opendevops 源码浅析

1.  起因：200605 部门周会时，大家提及此项目，并感觉还不错。所以，我就找了个周末，再进一步了解一下这个项目。
2.  目的：了解这些开源项目实现思路、架构设计、技术实现，不会去追溯业务细节。

### opendevops 

1.  项目简介：[opendevops/README](https://github.com/opendevops-cn/opendevops/blob/master/README-zh.md)

2.  项目模块：

    >前端代码：[codo](https://github.com/opendevops-cn/codo)
    >管理后端：[codo-admin](https://github.com/opendevops-cn/codo-admin)
    >定时任务：[codo-cron](https://github.com/opendevops-cn/codo-cron)
    >任务调度：[codo-task](https://github.com/opendevops-cn/codo-task)
    >资产管理：[codo-cmdb](https://github.com/opendevops-cn/codo-cmdb)
    >配置中心：[codo-kerrigan](https://github.com/opendevops-cn/kerrigan)
    >运维工具：[codo-tools](https://github.com/opendevops-cn/codo-tools)
    >域名管理：[codo-dns](https://github.com/opendevops-cn/codo-dns)

3.  下面主要看看 **code-cron** 和 **code-task** 的实现原理及相关技术。

### code-cron

1.  程序入口：

    -   命令行启动参考 `doc/supervisor_ops.conf` 文件，
    -   指令如下：`python3 startup.py --service=cron --port=99%(process_num)02d`
    -   入口程序 `startup.py`，`startup.MyProgram`

2.  web app 类：`cron.applications.Application`。实现技术，**tornado**。

3.  定时任务实现类，`cron/applications.py:17`。实现技术，`apscheduler.schedulers.tornado.TornadoScheduler`

    -   **TornadoScheduler** 源码摘录

        -   ```python
            from __future__ import absolute_import
            
            from datetime import timedelta
            from functools import wraps
            
            from apscheduler.schedulers.base import BaseScheduler
            from apscheduler.util import maybe_ref
            
            try:
                from tornado.ioloop import IOLoop
            except ImportError:  # pragma: nocover
                raise ImportError('TornadoScheduler requires tornado installed')
            
            
            def run_in_ioloop(func):
                @wraps(func)
                def wrapper(self, *args, **kwargs):
                    self._ioloop.add_callback(func, self, *args, **kwargs)
                return wrapper
            
            
            class TornadoScheduler(BaseScheduler):
                ……
                
                @run_in_ioloop
                def shutdown(self, wait=True):
                    ……
                    
                @run_in_ioloop
                def wakeup(self):
            ```

    -   可以看出，TornadoScheduler 也是基于 **apscheduler** 和 **ioloop** 来实现异步定时任务。

    -   apscheduler 可以参考：[Python APScheduler 分享]([http://gitlab.playcrab-inc.com/gaoyang/operation-document/-/wikis/Python%20APScheduler%20%E5%88%86%E4%BA%AB](http://gitlab.playcrab-inc.com/gaoyang/operation-document/-/wikis/Python APScheduler 分享)

4.  url 接口汇总：

    >   ```python
    >   cron_urls = [
    >      (r"/v1/cron/job/", CronJobs),
    >      (r"/v1/cron/log/", CronLogs),
    >      (r"/are_you_ok/", LivenessProbe),
    >   ]
    >   ```

4.  `/v1/cron/job/` 接口，实现类 `cron.applications.CronJobs`
    -   get，获取 cron job
    -   post，添加 cron job
    -   put，更新 cron job
    -   delete，删除 cron job
    -   patch，暂停作业/恢复作业
5.  `/v1/cron/log/` 接口，实现类 `cron.applications.CronLogs`
    
    -   get，查询 cron log，cron_log 数据库结构 `models.cron.CronLog`

### code-task

1.  程序入口：

    -   命令行启动参考 `doc/supervisor_ops.conf` 文件
    -   指令如下：`python3 startup.py --service=cron --port=99%(process_num)02d`
    -   入口程序 `startup.py`, `startup.MyProgram`

2.  功能分类

    -   `biz.applications.Application`，web 接口
    -   `biz.program.Application`，执行任务
    -   `biz.subscribe.RedisSubscriber`，redis 订阅
    -   `biz.cron_app.Application`，报警
    -   `biz.other_app.Application`，其他，非核心

3.  web app

    -   实现类：`biz.applications.Application`
    -   使用技术：tornado

4.   web 接口汇总

    -   ```python
        temp_urls = [
            (r"/v2/task_layout/command/", CommandHandler),
            (r"/v2/task_layout/temp/", TemplateHandler),
            (r"/v2/task_layout/details/", TempDetailsHandler),
            (r"/v2/task_layout/args/", ArgsHandler),
            (r"/v2/task_layout/user/", ExecutiveUserHandler),
            (r"/v2/task_layout/temp/args/", TempArgsHandler),
            (r"/are_you_ok/", LivenessProbe),
        ]
        # 参考，biz/handlers/templet_handler.py:421
        
        
        task_list_urls = [
            (r"/v2/task/list/", TaskListHandler),
            (r"/v2/task/check/", TaskCheckHandler),
            (r"/v2/task/check_history/", HistoryListHandler),
            (r"/v2/task/statement/", TaskStatementHandler),
        ]
        # 参考，biz/handlers/task_handler.py:394
        
        
        accept_task_urls = [
            (r"/v2/task/accept/", AcceptTaskHandler)
        ]
        # 参考：biz/handlers/task_accept.py:137
        ```

    -   

5.  执行任务

    -   使用一个消费者，订阅 MQ。来实时获取任务，然后去消费。

    -   执行类：`biz.exec_tasks.MyExecute`

    -   相关日志发布到 redis channels 中

        -   ```python
            redis_conn.publish("task_log", json.dumps(
                            {"log_key": log_key, "exec_time": int(round(time.time() * 1000)), "result": result}))
            ```

6.  redis 订阅
    -   实现类：`biz.subscribe.RedisSubscriber`
    -   监听 redis channels，将相关 log 持久化到 scheduler_task_log（`models.scheduler.TaskLog`） 表中。
7.  报警
    -   实现类：`biz.alert_tasks.send_alarm`
    -   相关技术：`tornado.ioloop.PeriodicCallback`，参考：[tornado.ioloop.``PeriodicCallback]([https://tornado-zh.readthedocs.io/zh/latest/ioloop.html?highlight=tornado%20ioloop%20periodiccallbac#tornado.ioloop.PeriodicCallback](https://tornado-zh.readthedocs.io/zh/latest/ioloop.html?highlight=tornado ioloop periodiccallbac#tornado.ioloop.PeriodicCallback))
8.  其他，非核心
    
    -    暂时不去理会