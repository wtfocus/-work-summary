[toc]

## RESTful API 最佳实践

### 参考

1.  [RESTful API 最佳实践- 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)
2.  [13 个设计REST API 的最佳实践- 掘金](https://juejin.im/post/5c1ba471f265da61257812bf)
3.  [Restful API设计最佳实践- Tech For Fun](http://kaelzhang81.github.io/2019/05/24/Restful-API设计最佳实践/)
4.  [REST API规范约定— Jumpserver latest 文档](https://docs.jumpserver.org/zh/1.4.8/api_style_guide.html)

### 补充 。。。

#### 200628 - 确定几种常见的规范


1.  命名	-- 结论：**使用下杠、小写命名**

    >@api    {get}   /MainCategory/      获取主类    驼峰命名
    >
    >@api    {get}   /main-category/     获取主类    中杠、小写命名
    >
    >@api    {get}   /main_category/     获取主类    下杠、小写命名     （正例）

2.  尾斜杠

    >   @api    {get}   /main_category      获取主类    URL结尾不应该包含斜杠“/”     （正例）

3.  名词是否使用复数 -- 结论：**不使用复数**

    >   @api    {get}   /main_category          获取主类    不使用复数   （正例）
    >
    >   @api    {get}   /main_categorys         获取主类    一般复数形式
    >
    >   @api    {get}   /main_categories        获取主类    category 复数

4.  路径与表名对应

    >@api    {get}   /main_category        获取主类  main_category 跟数据库表 main_category 对应    （正例）

#### 200706 - 接口状态码

1.  常见 API 状态码及相关信息

    -    2xx	操作成功

         -    | 状态码         | HTTP 方法 | 描述                                                       |
              | -------------- | --------- | ---------------------------------------------------------- |
              | 200 OK         | GET       | 服务器成功返回用户请求的数据, 该操作是幂等的(Idempotent)。 |
              | 201 CREATED    | POST\|PUT | 成功请求，并创建了新的资源                                 |
              | 202 Accepted   | *         | 表示一个请求已经进入后台排队(异步任务)                     |
              | 204 NO CONTENT | DELETE    | 删除数据成功。                                             |
         
    -    3xx	重定向

         -    | 状态码        | 动作              | 描述                                         |
              | ------------- | ----------------- | -------------------------------------------- |
              | 301           |                   | 永久重定向，API 级别不考虑                   |
              | 302           |                   | 暂时重定向，API 级别不考虑                   |
              | 303 See Other | POST\|PUT\|DELETE | 浏览器不会自动跳转，会让用户决定下一步怎么办 |

    -    4xx	客户端错误

         -    | 状态码                  | 动作      | 描述                                                        |
              | ----------------------- | --------- | ----------------------------------------------------------- |
              | 400 INVALID REQUEST     | *         | 服务器无法理解请求                                          |
              | 401 Unauthorized        | *         | 用户没有权限                                                |
              | 403 Forbidden           | *         | 用户得到授权(与401错误相对), 但访问是被禁止的               |
              | 404 NOT FOUND           | *         | 服务器无法根据客户端的请求找到资源                          |
              | 406 Not Acceptable      | GET       | 用户请求的格式不可得(比如用户请求JSON格式, 但是只有XML格式) |
              | 422 Unprocesable entity | POST\|PUT | 当创建一个对象时, 发生一个验证错误。                        |

         

    -    5xx	服务端错误

         -    | 状态码                    | 动作 | 描述                                     |
              | ------------------------- | ---- | ---------------------------------------- |
              | 500 INTERNAL SERVER ERROR | *    | 服务器内部错误，无法完成请求             |
              | 503 Service Unavailable   | *    | 服务器无法处理请求，一般用于网站维护状态 |

2.  每一种状态码都有标准的（或者约定的）解释，客户端只需查看状态码，就可以判断出发生了什么情况，所以服务器应该返回尽可能精确的状态码。

3.  参考：

    -   [List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)