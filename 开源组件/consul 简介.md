[toc]

## consul 简介

1.  简介：

    -   `Consul` 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。

2.  特性：

    -   服务发现（主要功能）
    -	健康检测
    -	Key/Value 存储
    -	多数据中心

3.  适用场景

    -   简单的服务发现流程分解图

    -   ![consul运行原理图](imgs/consul运行原理图.png)
    -   较完整的服务发现流程分解图
    -   ![img](imgs/pasted-125.png)

4.  内部原理

    -   ![img](imgs/pasted-124.png)

5.  与其他常见服务发现框架对比

    -   ![image-20210113133603590](imgs/image-20210113133603590.png)

6.  python client 相关文档

    -   [Python client for Consul](https://python-consul.readthedocs.io/en/latest/)
    -   [Python client | Github](https://github.com/cablehead/python-consul)
    -   [使用 Consul 作为 Python 微服务的配置中心](https://juejin.cn/post/6844903797064466445#heading-0)

7.  python demo

    1.  注册/发现

        -   producer

        -   ```python
            #!/usr/bin/env python
            # coding: utf-8
            # __author__ = 'wang tao'
            
            
            import consul
            
            
            host = "10.2.24.196"
            port = 8500
            c = consul.Consul(host=host, port=port)
            
            def register(
                    name="memorial",
                    host="10.2.24.223",
                    port=8002,
                    tags=None
            ):
                c.agent.service.register(
                    name,
                    name,
                    host,
                    port,
                    tags,
                    # 健康检查 url，检查时间：5，超时时间：30，注销时间：30s
                    check=consul.Check().http("http://10.2.24.223:8002/test/get-parameter", "5s", "30s", "30s")
                )
            
            
            if __name__ == "__main__":
                register()
            
            ```

        -   comsumer

        -   ```python
            #!/usr/bin/env python
            # coding: utf-8
            # __author__ = 'wang tao'
            
            
            import consul
            import logging
            
            logging.basicConfig(
                level=logging.INFO,
                format="%(asctime)s - %(name)s [line:%(lineno)d] - %(levelname)s - %(message)s"
            )
            logger = logging.getLogger(__name__)
            
            
            host = "10.2.24.196"
            port = 8500
            c = consul.Consul(host=host, port=port)
            
            
            def get_service(name="memorial"):
            
                services = c.agent.services()
                service = services.get(name)
                logger.info(f"{service.get('Service')} {service.get('Address')} {service.get('Port')}")
            
            
            if __name__ == "__main__":
                
                get_service()
            
            ```

    2.  key/value 存储

        -   producer

        -   ```python
            #!/usr/bin/env python
            # coding: utf-8
            # __author__ = 'wang tao'
            
            
            import consul
            
            
            host = "10.2.24.196"
            port = 8500
            c = consul.Consul(host=host, port=port)
            
            
            def push(key='foo', value="bar"):
            
                c.kv.put(key, value)
            
            
            if __name__ == "__main__":
                push()
            
            ```

        -   comsumer

        -   ```python
            #!/usr/bin/env python
            # coding: utf-8
            # __author__ = 'wang tao'
            
            
            import consul
            import logging
            
            logging.basicConfig(
                level=logging.INFO,
                format="%(asctime)s - %(name)s [line:%(lineno)d] - %(levelname)s - %(message)s"
            )
            logger = logging.getLogger(__name__)
            
            
            host = "10.2.24.196"
            port = 8500
            c = consul.Consul(host=host, port=port)
            
            
            def poll(key='foo'):
            
                # poll a key for updates
                index = None
                while True:
                    # 对应 key 如果没有修改，get() 会阻塞。
                    index, data = c.kv.get(
                        key,
                        index=index,
                        wait="10s"	# 超时，get() 会返回数据
                    )
                    logger.info("{} {}".format(index, data))
            
            
            if __name__ == "__main__":
            
                poll()
            
            ```

        -   

6.  参考：
    -   [使用 Consul 作为 Python 微服务的配置中心](https://juejin.cn/post/6844903797064466445#heading-0)
    -   [使用Consul做服务发现的若干姿势](https://blog.didispace.com/consul-service-discovery-exp/)
    -   [微服务Consul介绍](http://gitlab.playcrab-inc.com/technical-support/operation-document/-/wikis/Introduction-to-Consul)

