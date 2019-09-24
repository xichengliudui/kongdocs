> 注意，1、本文档 API 版本基本 Kong V1.13+，2、本文档 Admin API 适用于 DB（Postgres 或 Cassandra）模式下，如果是无 DB 模式，将不适用本文档， 

* Kong 提供了用于管理的内部 RESTful  Admin API。对 Admin API 的请求可以发送到集群中的任何节点，Kong 会保持所有节点的配置一致
* `8001`，是 Admin API 的默认端口
* `8001`是到 Admin API 的 HTTPS 流量的默认端口，以下内容皆使用 8001 端口


[TOC]


### 路由信息



#### 检索节点信息



> 检索有关节点的常规详细信息。



| 请求方式 | 请求路径 |
| -------- | -------- |
| *GET*    | */*      |



**响应**

```shell
HTTP 200 OK
```

```json
{
    "hostname": "",
    "node_id": "6a72192c-a3a1-4c8d-95c6-efabae9fb969",
    "lua_version": "LuaJIT 2.1.0-beta3",
    "plugins": {
        "available_on_server": [
            ...
        ],
        "enabled_in_cluster": [
            ...
        ]
    },
    "configuration" : {
        ...
    },
    "tagline": "Welcome to Kong",
    "version": "0.14.0"
}
```



* node_id: 表示正在运行的 Kong 节点的 UUID。这个 UUID 是在 Kong 启动时随机生成的，所以每次重新启动节点时都有一个不同的 node_id
* available_on_server: 节点上安装的插件的名称
* enabled_in_cluster: 已启用/配置的插件的名称。所有 Kong 节点共享的数据存储中当前的插件配置



#### 检索节点状态



> 检索关于节点的使用信息，以及关于底层 nginx 进程正在处理的连接的一些基本信息、数据库连接的状态和节点的内存使用情况
>



| 请求方式 | 请求路径  |
| -------- | --------- |
| *GET*    | */status* |



**响应**

```json
HTTP 200 OK
```

```json
{
    "database": {
      "reachable": true
    },
    "memory": {
        "workers_lua_vms": [{
            "http_allocated_gc": "0.02 MiB",
            "pid": 18477
          }, {
            "http_allocated_gc": "0.02 MiB",
            "pid": 18478
        }],
        "lua_shared_dicts": {
            "kong": {
                "allocated_slabs": "0.04 MiB",
                "capacity": "5.00 MiB"
            },
            "kong_db_cache": {
                "allocated_slabs": "0.80 MiB",
                "capacity": "128.00 MiB"
            },
        }
    },
    "server": {
        "total_requests": 3,
        "connections_active": 1,
        "connections_accepted": 1,
        "connections_handled": 1,
        "connections_reading": 0,
        "connections_writing": 1,
        "connections_waiting": 0
    }
}
```



* memory: 关于内存使用的指标
  *  workers_lua_vms: 包含所有 Kong 工作节点的数组，其中每个条目包含:
  * http_allocated_gc: HTTP 子模块的 Lua 虚拟机的内存使用信息（由 collectgarbage("count") 报告），针对每个活动 worker，即在过去 10 秒内接收代理调用的 worker
  * pid: 工作进程号
  * lua_shared_dicts: 在 Kong 节点中与所有工作共享的关于字典的信息数组，其中，每个数组节点包含有多少内存专用于特定的共享字典（capacity），以及有多少内存正在使用（allocated_slabs）。然而，对于某些字典，例如 HIT/MISS 字典，增加它们的大小可能有利于 Kong 节点的整体性能
* 内存使用 unit 和 precision 可以使用 querystring 参数 unit  和 scale 进行更改:
  * unit:  `b/B`, `k/K`, `m/M`, `g/G` 中的一个，它将分别以 bytes、kibibytes、mebibytes 或 gibibytes 返回结果。当请求 bytes 时，响应中的内存值将使用数字类型不是字符串。默认为 m。
  * scale: 以可读的内存字符串（除 bytes 外的单位）给出值小数点右侧的位数。默认为 2。可以通过以下操作获得以 kibibytes 为单位的共享字典内存使用情况（精度为 4 位）: GET /status?unit=k&scale=4
* server: 关于 nginx HTTPS 服务的指标
  * total_requests: 客户端请求总数
  * connections_active: 当前可用客户端来连接的总数
  * connections_accepted: 接受的客户端连接的总数
  * connections_handled: 处理的连接总数，通常除非达到某些资源限制。否则这个值与 accept 相同
  * connections_reading: Kong: 正在读取请求头的当前连接数
  * connections_writing: nginx: 等待请求的空闲客户端连接数
* database: 关于数据库的指标
  	* reachable: 数据库连接状态的 boolean 值，注意：这个参数不返回数据库本身健康状态



### 标签



> 标签是与 Kong 中的实体相关联的字符串，每个参数必须有一个或者多个字母数字字符组成如：`_`, `-`, `.`
>
> 另外，标签还可以过滤实体



#### 查看标签列表



> 返回所有标签的分页列表，实体列表将不限于单一的实体类型: 所有带有标记的实体都将出现在这个列表中，如果一个实体具有多个标记，则使用 entity_id



| 请求方式 | 请求路径 |
| -------- | -------- |
| *GET*    | */tags*  |



**响应**

```json
HTTP 200 OK
```

```json
{
    {
      "data": [
        { "entity_name": "services",
          "entity_id": "acf60b10-125c-4c1a-bffe-6ed55daefba4",
          "tag": "s1",
        },
        { "entity_name": "services",
          "entity_id": "acf60b10-125c-4c1a-bffe-6ed55daefba4",
          "tag": "s2",
        },
        { "entity_name": "routes",
          "entity_id": "60631e85-ba6d-4c59-bd28-e36dd90f6000",
          "tag": "s1",
        },
        ...
      ],
      "offset" = "c47139f3-d780-483d-8a97-17e9adc5a7ab",
      "next" = "/tags?offset=c47139f3-d780-483d-8a97-17e9adc5a7ab",
    }
}
```



#### 根据标签查看实例



> 返回已使用指定标签标记的实体，实体列表将不限于单一的实体类型:所有带有标记的实体都将出现在这个列表中。
>





| 请求方式 | 请求路径      |
| -------- | ------------- |
| *GET*    | */tags/:tags* |



**响应**

```json
HTTP 200 OK
```

```json
{
    {
      "data": [
        { "entity_name": "services",
          "entity_id": "c87440e1-0496-420b-b06f-dac59544bb6c",
          "tag": "example",
        },
        { "entity_name": "routes",
          "entity_id": "8a99e4b1-d268-446b-ab8b-cd25cff129b1",
          "tag": "example",
        },
        ...
      ],
      "offset" = "1fb491c4-f4a7-4bca-aeba-7f3bcee4d2f9",
      "next" = "/tags/example?offset=1fb491c4-f4a7-4bca-aeba-7f3bcee4d2f9",
    }
}
```



### 服务对象



> 顾名思义，服务实体是你自己上游服务的抽象，服务的实例包括微服务、计费 API 等.......
>
> 服务的主要属性是 URL（Kong 应该将代理流量发送到这个 URL），可以将其设置为单个字符串，也可以是单独指定协议、主机、端口和路径。
>
> 服务与路由相关连（一个服务可以有多个与之关联的路由），路由是 Kong 中的入口点，定义了匹配客户端请求的规则。一旦匹配了路由 Kong 会将请求代理到它的关联服务



```json
{
    "id": "9748f662-7711-4a90-8186-dc02f10eb0f5",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-service",
    "retries": 5,
    "protocol": "http",
    "host": "example.com",
    "port": 80,
    "path": "/some_api",
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["user-level", "low-priority"],
    "client_certificate": {"id":"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b"}
}
```



#### 添加服务



##### 创建服务

| 请求方式 | 请求路径   |
| -------- | ---------- |
| *POST*   | *services* |



##### 创建与特定证书关联的服务

| 请求方式 | 请求路径                                          |
| -------- | ------------------------------------------------- |
| *POST*   | */certificates/{certificate name or id}/services* |

| 属性                   | 描述                                          |
| ---------------------- | --------------------------------------------- |
| certificate name or id | 证书 id 或名称，id 或名称与新创建的服务相关联 |



**请求体**

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| name               | 服务名称                                                     |
| retries            | 代理失败后要执行的重试次数。默认为 5                         |
| protocol           | 与上游通信协议，http 或者 https，默认 http 协议              |
| host               | 上游服务的主机                                               |
| port               | 上游服务的端口，默认 80                                      |
| path               | 请求上游服务的路径。                                         |
| connect_timeout    | 建立到上游服务连接的超时（以毫秒为单位）。默认为60000。      |
| write_timeout      | 两个连续写操作之间的超时（以毫秒为单位），用于将请求发送到上游服务。默认 60000 |
| read_timeout       | 两个连续读取操作之间的超时（以毫秒为单位），用于将请求传输到上游服务。默认 60000 |
| tags               | 与服务相关的一组可选字符串，用于分组和过滤                   |
| client_certificate | 当 TLS 与上游服务握手时，证书用作客户端证书。对于 form-encoded，notation 是 client_certificate.id=<client_certificate_id>。对于JSON，使用 "client_certificate":{"id":"<client_certificate_id>"} |
| url                | 简写属性，用于同时设置 protocol、host、port、path 这个属性是只写的（ Admin API从不 return url） |



**响应**

```json
HTTP 201 Created
```

```
{
    "id": "9748f662-7711-4a90-8186-dc02f10eb0f5",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-service",
    "retries": 5,
    "protocol": "http",
    "host": "example.com",
    "port": 80,
    "path": "/some_api",
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["user-level", "low-priority"],
    "client_certificate": {"id":"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b"}
}
```



#### 服务列表



##### 所有服务

| 请求方式 | 请求路径    |
| -------- | ----------- |
| *GET*    | */services* |



##### 与特定证书关联的服务列表

| 请求方式 | 请求路径                                          |
| -------- | ------------------------------------------------- |
| *GET*    | */certificates/{certificate name or id}/services* |

| 属性                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| certificate name or id | 要检索服务的证书的 id 或名称。使用此端点时，只列出与指定证书关联的服务 |



**响应**

```json
HTTP 200 OK
```

```json
{
"data": [{
    "id": "a5fb8d9b-a99d-40e9-9d35-72d42a62d83a",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-service",
    "retries": 5,
    "protocol": "http",
    "host": "example.com",
    "port": 80,
    "path": "/some_api",
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["user-level", "low-priority"],
    "client_certificate": {"id":"51e77dc2-8f3e-4afa-9d0e-0e3bbbcfd515"}
}, {
    "id": "fc73f2af-890d-4f9b-8363-af8945001f7f",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-service",
    "retries": 5,
    "protocol": "http",
    "host": "example.com",
    "port": 80,
    "path": "/another_api",
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["admin", "high-priority", "critical"],
    "client_certificate": {"id":"4506673d-c825-444c-a25b-602e3c2ec16e"}
}],

    "next": "http://localhost:8001/services?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索服务



##### 检索服务

| 请求方式 | 请求路径                 |
| -------- | ------------------------ |
| *GET*    | */services/{name or id}* |

| 属性       | 描述                     |
| ---------- | ------------------------ |
| name or id | 要检索的服务的 id 或名称 |



##### 检索与特定路由关联的服务

| 请求方式 | 请求路径                             |
| -------- | ------------------------------------ |
| *GET*    | */routes/{route name or id}/service* |

| 属性             | 描述                               |
| ---------------- | ---------------------------------- |
| route name or id | 与要检索的服务关联路由的 id 或名称 |



##### 检索与特定插件关联的服务

| 请求方式 | 请求路径                       |
| -------- | ------------------------------ |
| *GET*    | */plugins/{plugin id}/service* |

| 属性      | 描述                        |
| --------- | --------------------------- |
| plugin id | 与要检索的服务关联的插件 id |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "9748f662-7711-4a90-8186-dc02f10eb0f5",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-service",
    "retries": 5,
    "protocol": "http",
    "host": "example.com",
    "port": 80,
    "path": "/some_api",
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["user-level", "low-priority"],
    "client_certificate": {"id":"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b"}
}

```



#### 更新服务



##### 更新服务

| 请求方式 | 请求路径                 |
| -------- | ------------------------ |
| *PATCH*  | */services/{name or id}* |

| 属性       | 描述                    |
| ---------- | ----------------------- |
| name or id | 要更新服务的 id 或 name |



##### 与特定路由关联的更新服务

| 请求方式 | 请求路径                             |
| -------- | ------------------------------------ |
| *PATCH*  | */routes/{route name or id}/service* |

| 属性             | 描述                                  |
| ---------------- | ------------------------------------- |
| route name or id | 与要更新的服务关联路由的 id 或者 name |



##### 与特定插件关联的更新服务

| 请求方式 | 请求路径                       |
| -------- | ------------------------------ |
| *PATCH*  | */plugins/{plugin id}/service* |

| 属性      | 描述                        |
| --------- | --------------------------- |
| plugin id | 与要检索的服务关联的插件 id |



**请求体**

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| name               | 服务名称                                                     |
| retries            | 代理失败后要执行的重试次数。默认为 5                         |
| protocol           | 与上游通信协议，http 或者 https，默认 http 协议              |
| host               | 上游服务的主机                                               |
| port               | 上游服务的端口，默认 80                                      |
| path               | 请求上游服务的路径。                                         |
| connect_timeout    | 建立到上游服务连接的超时（以毫秒为单位）。默认为60000。      |
| write_timeout      | 两个连续写操作之间的超时（以毫秒为单位），用于将请求发送到上游服务。默认 60000 |
| read_timeout       | 两个连续读取操作之间的超时（以毫秒为单位），用于将请求传输到上游服务。默认 60000 |
| tags               | 与服务相关的一组可选字符串，用于分组和过滤                   |
| client_certificate | 当 TLS 与上游服务握手时，证书用作客户端证书。对于 form-encoded，notation 是 client_certificate.id=<client_certificate_id>。对于JSON，使用 "client_certificate":{"id":"<client_certificate_id>"} |
| url                | 简写属性，用于同时设置 protocol、host、port、path 这个属性是只写的（ Admin API从不 return url） |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "9748f662-7711-4a90-8186-dc02f10eb0f5",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-service",
    "retries": 5,
    "protocol": "http",
    "host": "example.com",
    "port": 80,
    "path": "/some_api",
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["user-level", "low-priority"],
    "client_certificate": {"id":"4e3ad2e4-0bc4-4638-8e34-c84a417ba39b"}
}
```



#### 更新或创建服务



##### 更新或创建服务

| 请求方式 | 请求路径                 |
| -------- | ------------------------ |
| *PUT*    | */services/{name or id}* |

| 属性       | 描述                    |
| ---------- | ----------------------- |
| name or id | 要更新服务的 id 或 name |



##### 创建或更新与特定路由关联的服务

| 请求方式 | 请求路径                             |
| -------- | ------------------------------------ |
| *PUT*    | */routes/{route name or id}/service* |

| 属性             | 描述                                  |
| ---------------- | ------------------------------------- |
| route name or id | 与要更新的服务关联路由的 id 或者 name |



##### 创建或更新与特定插件相关联的服务

| 请求方式 | 请求路径                       |
| -------- | ------------------------------ |
| *PUT*    | */plugins/{plugin id}/service* |

| 属性      | 描述                        |
| --------- | --------------------------- |
| plugin id | 与要检索的服务关联的插件 id |



**请求体**

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| name               | 服务名称                                                     |
| retries            | 代理失败后要执行的重试次数。默认为 5                         |
| protocol           | 与上游通信协议，http 或者 https，默认 http 协议              |
| host               | 上游服务的主机                                               |
| port               | 上游服务的端口，默认 80                                      |
| path               | 请求上游服务的路径。                                         |
| connect_timeout    | 建立到上游服务连接的超时（以毫秒为单位）。默认为60000。      |
| write_timeout      | 两个连续写操作之间的超时（以毫秒为单位），用于将请求发送到上游服务。默认 60000 |
| read_timeout       | 两个连续读取操作之间的超时（以毫秒为单位），用于将请求传输到上游服务。默认 60000 |
| tags               | 与服务相关的一组可选字符串，用于分组和过滤                   |
| client_certificate | 当 TLS 与上游服务握手时，证书用作客户端证书。对于 form-encoded，notation 是 client_certificate.id=<client_certificate_id>。对于JSON，使用 "client_certificate":{"id":"<client_certificate_id>"} |
| url                | 简写属性，用于同时设置 protocol、host、port、path 这个属性是只写的（ Admin API从不 return url） |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除服务



##### 删除服务

| 请求方式 | 请求路径                 |
| -------- | ------------------------ |
| *DELETE* | */services/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要删除的服务的 id 或 name |



##### 删除与特定路由关联的服务

| 请求方式 | 请求路径                             |
| -------- | ------------------------------------ |
| *DELETE* | */routes/{route name or id}/service* |

| 属性             | 描述                                |
| ---------------- | ----------------------------------- |
| route name or id | 与要删除的服务关联的路由 id 或 name |



**响应**

```json
HTTP 204 No Content
```



### 路由对象



> 路由实体定义匹配客户端请求的规则。每个路由都与一个服务相关联，并且一个服务可能有多个与之关联的路由。每个与给定路由匹配的请求都将代理到其关联的服务。
>
> 路由和服务的组合（以及它们之间的关注点分离）提供了一个强大的路由机制，通过该机制可以在 Kong 中定义细粒度的入口点，从而指向基础设施的不同上游服务。
>
> 至少需要一个匹配规则，该规则应用于路由匹配的协议。根据路由所配置的协议（由 protocols 字段定义），这意味着至少必须设置以下属性之一:



* 对于 http，至少有一个 methods, hosts, headers 或 paths
* 对于 https，至少有一个 methods、hosts、headers 、paths 或 snis;
* 对于 tcp，至少有一个 sources 或 destinations;
* 对于 tls，至少有一个 sources、destinations 或 snis;
* 对于 grpc，至少有一个 hosts、headers  或 paths;
* 对于 grpc，至少有一个 hosts、headers 、paths 或 snis。



```json
{
    "id": "d35165e2-d03e-461a-bdeb-dad0a112abfe",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-route",
    "protocols": ["http", "https"],
    "methods": ["GET", "POST"],
    "hosts": ["example.com", "foo.test"],
    "paths": ["/foo", "/bar"],
    "headers": {"x-another-header":["bla"], "x-my-header":["foo", "bar"]},
    "https_redirect_status_code": 426,
    "regex_priority": 0,
    "strip_path": true,
    "preserve_host": false,
    "tags": ["user-level", "low-priority"],
    "service": {"id":"af8330d3-dbdc-48bd-b1be-55b98608834b"}
}
```



#### 添加路由



##### 创建路由

| 请求方式 | 请求路径  |
| -------- | --------- |
| *POST*   | */routes* |



#####  创建与特定服务关联的路由

| 请求方式 | 请求路径                                |
| -------- | --------------------------------------- |
| *POST*   | */services/{service name or id}/routes* |

| 属性               | 描述                                      |
| ------------------ | ----------------------------------------- |
| service name or id | 应该与新创建的路由关联的服务的 id 或 name |



**请求体**

| 属性                       | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| neme                       | 路由的名称                                                   |
| protocols                  | 此路由应该允许的协议列表。当设置为 ["https"] 时，HTTP 请求会得到一个升级到 https 的请求的响应。默认为 ["http", "https"] |
| methods                    | 匹配此路由的 HTTP 方法列表。                                 |
| hosts                      | 匹配此路由的域名列表。使用 form-encoded，这个 notation 是 `hosts[]=example.com&hosts[]=foo.test`。对于 JSON，使用数组 |
| paths                      | 匹配此路由的路径列表。使用 form-encoded，notation 是`paths[]=/foo&paths[]=/bar`。对于JSON，使用数组 |
| headers                    | 一个或多个按 header 名称索引的值列表，如果在请求中出现，将导致此路由匹配，Host header 不能与此属性一起使用: 应使用 hosts 属性指定主机 |
| https_redirect_status_code | 当一个路由的所有属性都匹配时，状态码 Kong 会做出响应，但协议除外，也就是说，如果请求的协议是 HTTP 而不是 HTTPS。如果字段设置为 301、302、307 或 308，则位置报头由 Kong 注入。默认为 426。 |
| regex_priority             | 当多个路由同时使用 regexes 匹配给定的请求时，用于选择哪条路由的一个数字将解析该请求。当两条路由匹配该路径并具有相同的 regex_priority 时，将使用较旧的一条（最低 created_at）。注意，非正则表达式路由的优先级是不同的（较长的非正则表达式路由在较短的路由之前匹配）。默认值为 0 |
| strip_path                 | 当通过其中一条路径匹配路由时，从上游请求 URL 中去掉匹配前缀。默认值为 true |
| preserve_host              | 当通过主机域名之一匹配路由时，请在上游请求 header 中使用请求主机 header。如果设置为 false，则上游主机 header 将是服务主机的 header |
| snis                       | 使用流路由时匹配此路由的 SNIs 列表                           |
| sources                    | 使用流路由时匹配此路由的传入连接的 IP 源列表。每个条目都是一个对象，具有字段 "ip"（可选在 CIDR 范围表示法中）和 / 或 "port" |
| destinations               | 使用流路由时匹配此路由的传入连接的 IP 源列表。每个条目都是一个对象，具有字段 "ip"（可选在 CIDR 范围表示法中）和 / 或 "port" |
| tags                       | 可选的一组与路由相关联的字符串，用于分组和筛选。             |
| service                    | 此路由所关联的服务。这是路由代理通信到的地方。对于 form-encoded，notation 是 service.id=<service_id>。对于JSON，使用 "service":{"id":"<service_id>"} |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "d35165e2-d03e-461a-bdeb-dad0a112abfe",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-route",
    "protocols": ["http", "https"],
    "methods": ["GET", "POST"],
    "hosts": ["example.com", "foo.test"],
    "paths": ["/foo", "/bar"],
    "headers": {"x-another-header":["bla"], "x-my-header":["foo", "bar"]},
    "https_redirect_status_code": 426,
    "regex_priority": 0,
    "strip_path": true,
    "preserve_host": false,
    "tags": ["user-level", "low-priority"],
    "service": {"id":"af8330d3-dbdc-48bd-b1be-55b98608834b"}
}
```



#### 路由列表



##### 路由列表

| 请求方式 | 请求路径  |
| -------- | --------- |
| *GET*    | */routes* |



##### 与特定服务关联的路由列表

| 请求方式 | 请求路径                                |
| -------- | --------------------------------------- |
| *GET*    | */services/{service name or id}/routes* |

| 属性               | 描述                                      |
| ------------------ | ----------------------------------------- |
| service name or id | 应该与新创建的路由关联的服务的 id 或 name |



**响应**

```json
HTTP 200 OK
```

````json
{
"data": [{
    "id": "a9daa3ba-8186-4a0d-96e8-00d80ce7240b",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-route",
    "protocols": ["http", "https"],
    "methods": ["GET", "POST"],
    "hosts": ["example.com", "foo.test"],
    "paths": ["/foo", "/bar"],
    "headers": {"x-another-header":["bla"], "x-my-header":["foo", "bar"]},
    "https_redirect_status_code": 426,
    "regex_priority": 0,
    "strip_path": true,
    "preserve_host": false,
    "tags": ["user-level", "low-priority"],
    "service": {"id":"127dfc88-ed57-45bf-b77a-a9d3a152ad31"}
}, {
    "id": "9aa116fd-ef4a-4efa-89bf-a0b17c4be982",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-route",
    "protocols": ["tcp", "tls"],
    "https_redirect_status_code": 426,
    "regex_priority": 0,
    "strip_path": true,
    "preserve_host": false,
    "snis": ["foo.test", "example.com"],
    "sources": [{"ip":"10.1.0.0/16", "port":1234}, {"ip":"10.2.2.2"}, {"port":9123}],
    "destinations": [{"ip":"10.1.0.0/16", "port":1234}, {"ip":"10.2.2.2"}, {"port":9123}],
    "tags": ["admin", "high-priority", "critical"],
    "service": {"id":"ba641b07-e74a-430a-ab46-94b61e5ea66b"}
}],

    "next": "http://localhost:8001/routes?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
````



#### 检索路由



##### 检索路由

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *GET*    | */routes/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的路由的 id 或 name |



##### 检索与特定插件关联的路由

| 请求方式 | 请求路径                     |
| -------- | ---------------------------- |
| *GET*    | */plugins/{plugin id}/route* |

| 属性      | 描述                          |
| --------- | ----------------------------- |
| plugin id | 与要检索的路由关联的插件的 id |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "d35165e2-d03e-461a-bdeb-dad0a112abfe",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-route",
    "protocols": ["http", "https"],
    "methods": ["GET", "POST"],
    "hosts": ["example.com", "foo.test"],
    "paths": ["/foo", "/bar"],
    "headers": {"x-another-header":["bla"], "x-my-header":["foo", "bar"]},
    "https_redirect_status_code": 426,
    "regex_priority": 0,
    "strip_path": true,
    "preserve_host": false,
    "tags": ["user-level", "low-priority"],
    "service": {"id":"af8330d3-dbdc-48bd-b1be-55b98608834b"}
}

```



#### 更新路由



##### 更新路由

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *PATCH*  | */routes/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的路由的 id 或 name |



##### 更新与特定插件关联的路由

| 请求方式 | 请求路径                     |
| -------- | ---------------------------- |
| *PATCH*  | */plugins/{plugin id}/route* |

| 属性      | 描述                          |
| --------- | ----------------------------- |
| plugin id | 与要检索的路由关联的插件的 id |



**请求体**

| 属性                       | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| neme                       | 路由的名称                                                   |
| protocols                  | 此路由应该允许的协议列表。当设置为 ["https"] 时，HTTP 请求会得到一个升级到 https 的请求的响应。默认为 ["http", "https"] |
| methods                    | 匹配此路由的 HTTP 方法列表。                                 |
| hosts                      | 匹配此路由的域名列表。使用 form-encoded，这个 notation 是 `hosts[]=example.com&hosts[]=foo.test`。对于 JSON，使用数组 |
| paths                      | 匹配此路由的路径列表。使用 form-encoded，notation 是`paths[]=/foo&paths[]=/bar`。对于JSON，使用数组 |
| headers                    | 一个或多个按 header 名称索引的值列表，如果在请求中出现，将导致此路由匹配，Host header 不能与此属性一起使用: 应使用 hosts 属性指定主机 |
| https_redirect_status_code | 当一个路由的所有属性都匹配时，状态码 Kong 会做出响应，但协议除外，也就是说，如果请求的协议是 HTTP 而不是 HTTPS。如果字段设置为 301、302、307 或 308，则位置报头由 Kong 注入。默认为 426。 |
| regex_priority             | 当多个路由同时使用 regexes 匹配给定的请求时，用于选择哪条路由的一个数字将解析该请求。当两条路由匹配该路径并具有相同的 regex_priority 时，将使用较旧的一条（最低 created_at）。注意，非正则表达式路由的优先级是不同的（较长的非正则表达式路由在较短的路由之前匹配）。默认值为 0 |
| strip_path                 | 当通过其中一条路径匹配路由时，从上游请求 URL 中去掉匹配前缀。默认值为 true |
| preserve_host              | 当通过主机域名之一匹配路由时，请在上游请求 header 中使用请求主机 header。如果设置为 false，则上游主机 header 将是服务主机的 header |
| snis                       | 使用流路由时匹配此路由的 SNIs 列表                           |
| sources                    | 使用流路由时匹配此路由的传入连接的 IP 源列表。每个条目都是一个对象，具有字段 "ip"（可选在 CIDR 范围表示法中）和 / 或 "port" |
| destinations               | 使用流路由时匹配此路由的传入连接的 IP 源列表。每个条目都是一个对象，具有字段 "ip"（可选在 CIDR 范围表示法中）和 / 或 "port" |
| tags                       | 可选的一组与路由相关联的字符串，用于分组和筛选。             |
| service                    | 此路由所关联的服务。这是路由代理通信到的地方。对于 form-encoded，notation 是 service.id=<service_id>。对于JSON，使用 "service":{"id":"<service_id>"} |



**响应**

```json
HTTP 200 OK
```

```
{
    "id": "d35165e2-d03e-461a-bdeb-dad0a112abfe",
    "created_at": 1422386534,
    "updated_at": 1422386534,
    "name": "my-route",
    "protocols": ["http", "https"],
    "methods": ["GET", "POST"],
    "hosts": ["example.com", "foo.test"],
    "paths": ["/foo", "/bar"],
    "headers": {"x-another-header":["bla"], "x-my-header":["foo", "bar"]},
    "https_redirect_status_code": 426,
    "regex_priority": 0,
    "strip_path": true,
    "preserve_host": false,
    "tags": ["user-level", "low-priority"],
    "service": {"id":"af8330d3-dbdc-48bd-b1be-55b98608834b"}
}
```



#### 更新或创建路由



##### 更新或创建路由

| 请求方式 | 请求路径             |
| -------- | -------------------- |
| *PUT*    | /routes/{name or id} |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的路由的 id 或 name |



##### 创建或更新与特定插件关联的路由

| 请求方式 | 请求路径                   |
| -------- | -------------------------- |
| *PUT*    | /plugins/{plugin id}/route |

| 属性      | 描述                          |
| --------- | ----------------------------- |
| plugin id | 与要检索的路由关联的插件的 id |



**请求体**

| 属性                       | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| neme                       | 路由的名称                                                   |
| protocols                  | 此路由应该允许的协议列表。当设置为 ["https"] 时，HTTP 请求会得到一个升级到 https 的请求的响应。默认为 ["http", "https"] |
| methods                    | 匹配此路由的 HTTP 方法列表。                                 |
| hosts                      | 匹配此路由的域名列表。使用 form-encoded，这个 notation 是 `hosts[]=example.com&hosts[]=foo.test`。对于 JSON，使用数组 |
| paths                      | 匹配此路由的路径列表。使用 form-encoded，notation 是`paths[]=/foo&paths[]=/bar`。对于JSON，使用数组 |
| headers                    | 一个或多个按 header 名称索引的值列表，如果在请求中出现，将导致此路由匹配，Host header 不能与此属性一起使用: 应使用 hosts 属性指定主机 |
| https_redirect_status_code | 当一个路由的所有属性都匹配时，状态码 Kong 会做出响应，但协议除外，也就是说，如果请求的协议是 HTTP 而不是 HTTPS。如果字段设置为 301、302、307 或 308，则位置报头由 Kong 注入。默认为 426。 |
| regex_priority             | 当多个路由同时使用 regexes 匹配给定的请求时，用于选择哪条路由的一个数字将解析该请求。当两条路由匹配该路径并具有相同的 regex_priority 时，将使用较旧的一条（最低 created_at）。注意，非正则表达式路由的优先级是不同的（较长的非正则表达式路由在较短的路由之前匹配）。默认值为 0 |
| strip_path                 | 当通过其中一条路径匹配路由时，从上游请求 URL 中去掉匹配前缀。默认值为 true |
| preserve_host              | 当通过主机域名之一匹配路由时，请在上游请求 header 中使用请求主机 header。如果设置为 false，则上游主机 header 将是服务主机的 header |
| snis                       | 使用流路由时匹配此路由的 SNIs 列表                           |
| sources                    | 使用流路由时匹配此路由的传入连接的 IP 源列表。每个条目都是一个对象，具有字段 "ip"（可选在 CIDR 范围表示法中）和 / 或 "port" |
| destinations               | 使用流路由时匹配此路由的传入连接的 IP 源列表。每个条目都是一个对象，具有字段 "ip"（可选在 CIDR 范围表示法中）和 / 或 "port" |
| tags                       | 一组与路由相关联的字符串，用于分组和筛选。                   |
| service                    | 此路由所关联的服务。这是路由代理通信到的地方。对于 form-encoded，notation 是 service.id=<service_id>。对于JSON，使用 "service":{"id":"<service_id>"} |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```
HTTP 201 Created or HTTP 200 OK
```



#### 删除路由

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *DELETE* | */routes/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的路由的 id 或 name |



**响应**

```json
HTTP 204 No Content
```



### 消费者对象



> 消费者对象表示服务的使用者或用户。你可以依赖 Kong 作为主数据存储，也可以将消费者列表映射到数据库，以保持 Kong 与现有主数据存储之间的一致性



```json
{
    "id": "ec1a1f6f-2aa4-4e58-93ff-b56368f19b27",
    "created_at": 1422386534,
    "username": "my-username",
    "custom_id": "my-custom-id",
    "tags": ["user-level", "low-priority"]
}
```



#### 添加消费者



| 请求方式 | 请求路径     |
| -------- | ------------ |
| *POST*   | */consumers* |



**请求体**

| 属性      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| username  | 消费者的 username。您必须随请求发送此字段或 custom_id        |
| custom_id | 用于存储消费者的 custom_id，用于将 Kong 映射到现有数据库中的用户。必须将此字段或 username 与请求一起发送 |
| tags      | 与消费者关联的一组可选字符串，用于分组和筛选                 |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "ec1a1f6f-2aa4-4e58-93ff-b56368f19b27",
    "created_at": 1422386534,
    "username": "my-username",
    "custom_id": "my-custom-id",
    "tags": ["user-level", "low-priority"]
}

```



#### 消费者列表

| 请求方式 | 请求路径     |
| -------- | ------------ |
| *GET*    | */consumers* |



**响应**

```json
HTTP 200 OK
```

```
{
"data": [{
    "id": "a4407883-c166-43fd-80ca-3ca035b0cdb7",
    "created_at": 1422386534,
    "username": "my-username",
    "custom_id": "my-custom-id",
    "tags": ["user-level", "low-priority"]
}, {
    "id": "01c23299-839c-49a5-a6d5-8864c09184af",
    "created_at": 1422386534,
    "username": "my-username",
    "custom_id": "my-custom-id",
    "tags": ["admin", "high-priority", "critical"]
}],

    "next": "http://localhost:8001/consumers?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索消费者



##### 检索消费者

| 请求方式 | 请求路径                      |
| -------- | ----------------------------- |
| *GET*    | */consumers/{username or id}* |

| 属性           | 描述                            |
| -------------- | ------------------------------- |
| username or id | 要检索的消费者的 id 或 username |



##### 检索与特定插件相关的消费者

| 请求方式 | 请求路径                        |
| -------- | ------------------------------- |
| *GET*    | */plugins/{plugin id}/consumer* |

| 属性      | 描述                            |
| --------- | ------------------------------- |
| plugin id | 与要检索的消费者关联的插件的 id |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "ec1a1f6f-2aa4-4e58-93ff-b56368f19b27",
    "created_at": 1422386534,
    "username": "my-username",
    "custom_id": "my-custom-id",
    "tags": ["user-level", "low-priority"]
}
```



#### 更新消费者



##### 更新消费者

| 请求方式 | 请求路径                      |
| -------- | ----------------------------- |
| *PATCH*  | */consumers/{username or id}* |

| 属性           | 描述                            |
| -------------- | ------------------------------- |
| username or id | 要检索的消费者的 id 或 username |



##### 更新与特定插件关联的使用者

| 请求方式 | 请求路径                        |
| -------- | ------------------------------- |
| *PATCH*  | */plugins/{plugin id}/consumer* |

| 属性      | 描述                            |
| --------- | ------------------------------- |
| plugin id | 与要检索的消费者关联的插件的 id |



**请求体**

| 属性      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| username  | 消费者的 username。您必须随请求发送此字段或 custom_id        |
| custom_id | 用于存储消费者的 custom_id，用于将 Kong 映射到现有数据库中的用户。必须将此字段或 username 与请求一起发送 |
| tags      | 与消费者关联的一组可选字符串，用于分组和筛选                 |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "ec1a1f6f-2aa4-4e58-93ff-b56368f19b27",
    "created_at": 1422386534,
    "username": "my-username",
    "custom_id": "my-custom-id",
    "tags": ["user-level", "low-priority"]
}

```



#### 更新或创建消费者



##### 更新或创建消费者

| 请求方式 | 请求路径                      |
| -------- | ----------------------------- |
| *PUT*    | */consumers/{username or id}* |

| 属性           | 描述                            |
| -------------- | ------------------------------- |
| username or id | 要检索的消费者的 id 或 username |



#####  创建或更新与特定插件关联的使用者

| 请求方式 | 请求路径                        |
| -------- | ------------------------------- |
| *PUT*    | */plugins/{plugin id}/consumer* |

| 属性      | 描述                            |
| --------- | ------------------------------- |
| plugin id | 与要检索的消费者关联的插件的 id |



**请求体**

| 属性      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| username  | 消费者的 username。您必须随请求发送此字段或 custom_id        |
| custom_id | 用于存储消费者的 custom_id，用于将 Kong 映射到现有数据库中的用户。必须将此字段或 username 与请求一起发送 |
| tags      | 与消费者关联的一组可选字符串，用于分组和筛选                 |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除消费者

| 请求方式 | 请求路径                      |
| -------- | ----------------------------- |
| *DELETE* | */consumers/{username or id}* |

| 属性           | 描述                            |
| -------------- | ------------------------------- |
| username or id | 要检索的消费者的 id 或 username |



**响应**

```json
HTTP 204 No Content
```



### 组件对象 



> 插件实体表示在 HTTP request/response 生命周期中执行的插件配置。这就是如何向运行在 Kong 之后的服务添加功能，例如身份验证或限流。
>
> 当向服务添加插件配置时，客户端向该服务发出的每个请求都将运行该插件。如果需要为某些特定的使用者将插件调优到不同的值，可以通过创建一个单独的插件实例来实现，该实例通过服务和使用者字段指定服务和使用者

> 组件对象存在优先级概念

```json
{
    "id": "ce44eef5-41ed-47f6-baab-f725cecf98c7",
    "name": "rate-limiting",
    "created_at": 1422386534,
    "route": null,
    "service": null,
    "consumer": null,
    "config": {"hour":500, "minute":20},
    "run_on": "first",
    "protocols": ["http", "https"],
    "enabled": true,
    "tags": ["user-level", "low-priority"]
}
```



#### 组件优先级



> 注意：插件具有优先级概念，本文主要讲述 Admin API 的使用，不涉及组件优先级。详情请参考[组件优先级](<https://docs.konghq.com/1.3.x/admin-api/#precedence>)。



#### 创建插件



##### 创建插件

| 请求方式 | 请求路径   |
| -------- | ---------- |
| *POST*   | */plugins* |



##### 创建与特定路由关联的插件

| 请求方式 | 请求路径                     |
| -------- | ---------------------------- |
| *POST*   | */routes/{route id}/plugins* |

| 属性     | 描述                                    |
| -------- | --------------------------------------- |
| route id | 路由的 id，该 id 会与新创建的插件相关联 |



##### 创建与特定服务关联的插件

| 请求方式 | 请求路径                         |
| -------- | -------------------------------- |
| *POST*   | */services/{service id}/plugins* |

| 属性       | 描述                                    |
| ---------- | --------------------------------------- |
| service id | 服务的 id，该 id 会与新创建的插件相关联 |



##### 创建与特定消费者关联的插件

| 请求方式 | 请求路径                          |
| -------- | --------------------------------- |
| *POST*   | */consumers/{consumer id}/plugins |

| 属性        | 描述                                       |
| ----------- | ------------------------------------------ |
| consumer id | x消费者的 id，该 id 会与新创建的插件相关联 |



**请求体**

| 属性      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| name      | 要添加的插件的名称。目前插件必须分别安装在每个 Kong 实例中   |
| route     | 如果设置，插件将只在通过指定的路由接收请求时激活。不管使用什么路由，都不要设置插件来激活它。默认为 null。使用  form-encoded 的 notation 是 route.id=<route_id> 对于 JSON，使用 "route":{"id":"<route_id>"} |
| service   | 如果设置了，插件将只在通过属于指定服务的路由之一接收请求时激活。不管服务是否匹配，都不要设置插件激活。默认为空。使用 form-encoded，notation 是 service.id=<service_id> 对于 JSON 使用 "service":{"id":"<service_id>"} |
| consumer  | 如果设置了，插件将只对已验证指定的请求激活插件。(注意，有些插件不能以这种方式仅限于消费者)。不管经过身份验证的使用者是谁，都不要设置插件激活。默认为空。对于 form-encoded，notation 是 consumer.id=<consumer_id> 对于 JSON，使用 "consumer":{"id":"<consumer_id>"}。 |
| config    | 插件的配置属性，可以在 Kong Hub 的插件文档页面上找到         |
| run_on    | 给定服务网格场景，控制此插件将运行在哪个 Kong 节点上。接受的值是：                                             * first 意思是 "在请求遇到的第一个 Kong 节点上运行"。在 API 逃逸场景中，这是正常的操作，因为在源和目标之间只有一个 Kong 节点。在 sidecar-to-sidecar 服务网格场景中，这意味着只在出站连接的 Kong sidecar 上运行插件。                                                                                                                            * second 意思是 "在请求遇到的第二个节点上运行"。此选项仅适用于 sidecar-to-sidecar 服务网格场景中，这意味着仅在入站连接的 Kong sidecar 上运行插件。                                                                     * all 意思是 "在所有节点上运行"，即在 sidecar-to-sidecar 场景中同时运行两个 sidecar。这对于 tracing/logging （跟踪记录）插件非常有用。默认为 "first" |
| protocols | 将触发此插件的请求协议列表。可能的值是 http、https、tcp 和 tls。根据插件类型的不同，默认值以及该字段允许的可能值可能会发生变化。例如，仅在流模式下工作的插件可能只支持 tcp 和 tls。默认为 ["grpc","grpcs","http", "https"] |
| enabled   | 是否应用插件。默认值为 true                                  |
| tags      | 与插件相关联的可选字符串，用于分组和过滤                     |



**响应**

```json
HTTP 201 Created
```

````json
{
    "id": "ce44eef5-41ed-47f6-baab-f725cecf98c7",
    "name": "rate-limiting",
    "created_at": 1422386534,
    "route": null,
    "service": null,
    "consumer": null,
    "config": {"hour":500, "minute":20},
    "run_on": "first",
    "protocols": ["http", "https"],
    "enabled": true,
    "tags": ["user-level", "low-priority"]
}

````



#### 插件列表



##### 所有插件

| 请求方式 | 请求路径   |
| -------- | ---------- |
| *GET*    | */plugins* |



##### 与特定路由关联的插件

| 请求方式 | 请求路径                     |
| -------- | ---------------------------- |
| *GET*    | */routes/{route id}/plugins* |

| 属性     | 描述                                    |
| -------- | --------------------------------------- |
| route id | 路由的 id，该 id 会与新创建的插件相关联 |



##### 与特定服务关联的插件

| 请求方式 | 请求路径                         |
| -------- | -------------------------------- |
| *GET*    | */services/{service id}/plugins* |

| 属性       | 描述                                    |
| ---------- | --------------------------------------- |
| service id | 服务的 id，该 id 会与新创建的插件相关联 |



##### 与特定消费者关联的插件

| 请求方式 | 请求路径                           |
| -------- | ---------------------------------- |
| *GET*    | */consumers/{consumer id}/plugins* |

| 属性        | 描述                                       |
| ----------- | ------------------------------------------ |
| consumer id | x消费者的 id，该 id 会与新创建的插件相关联 |



**响应**

```json
HTTP 200 OK
```

```
{
"data": [{
    "id": "02621eee-8309-4bf6-b36b-a82017a5393e",
    "name": "rate-limiting",
    "created_at": 1422386534,
    "route": null,
    "service": null,
    "consumer": null,
    "config": {"hour":500, "minute":20},
    "run_on": "first",
    "protocols": ["http", "https"],
    "enabled": true,
    "tags": ["user-level", "low-priority"]
}, {
    "id": "66c7b5c4-4aaf-4119-af1e-ee3ad75d0af4",
    "name": "rate-limiting",
    "created_at": 1422386534,
    "route": null,
    "service": null,
    "consumer": null,
    "config": {"hour":500, "minute":20},
    "run_on": "first",
    "protocols": ["tcp", "tls"],
    "enabled": true,
    "tags": ["admin", "high-priority", "critical"]
}],

    "next": "http://localhost:8001/plugins?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索插件

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *GET*    | */plugins/{plugin id}* |

| 属性      | 描述              |
| --------- | ----------------- |
| plugin id | 要检索的插件的 id |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "ce44eef5-41ed-47f6-baab-f725cecf98c7",
    "name": "rate-limiting",
    "created_at": 1422386534,
    "route": null,
    "service": null,
    "consumer": null,
    "config": {"hour":500, "minute":20},
    "run_on": "first",
    "protocols": ["http", "https"],
    "enabled": true,
    "tags": ["user-level", "low-priority"]
}

```



#### 更新插件

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *PATCH*  | */plugins/{plugin id}* |

| 属性      | 描述              |
| --------- | ----------------- |
| plugin id | 要检索的插件的 id |



**请求体**

| 属性      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| name      | 要添加的插件的名称。目前插件必须分别安装在每个 Kong 实例中   |
| route     | 如果设置，插件将只在通过指定的路由接收请求时激活。不管使用什么路由，都不要设置插件来激活它。默认为 null。使用  form-encoded 的 notation 是 route.id=<route_id> 对于 JSON，使用 "route":{"id":"<route_id>"} |
| service   | 如果设置了，插件将只在通过属于指定服务的路由之一接收请求时激活。不管服务是否匹配，都不要设置插件激活。默认为空。使用 form-encoded，notation 是 service.id=<service_id> 对于 JSON 使用 "service":{"id":"<service_id>"} |
| consumer  | 如果设置了，插件将只对已验证指定的请求激活插件。(注意，有些插件不能以这种方式仅限于消费者)。不管经过身份验证的使用者是谁，都不要设置插件激活。默认为空。对于 form-encoded，notation 是 consumer.id=<consumer_id> 对于 JSON，使用 "consumer":{"id":"<consumer_id>"}。 |
| config    | 插件的配置属性，可以在 Kong Hub 的插件文档页面上找到         |
| run_on    | 给定服务网格场景，控制此插件将运行在哪个 Kong 节点上。接受的值是：                                             * first 意思是 "在请求遇到的第一个 Kong 节点上运行"。在 API 逃逸场景中，这是正常的操作，因为在源和目标之间只有一个 Kong 节点。在 sidecar-to-sidecar 服务网格场景中，这意味着只在出站连接的 Kong sidecar 上运行插件。                                                                                                                            * second 意思是 "在请求遇到的第二个节点上运行"。此选项仅适用于 sidecar-to-sidecar 服务网格场景中，这意味着仅在入站连接的 Kong sidecar 上运行插件。                                                                     * all 意思是 "在所有节点上运行"，即在 sidecar-to-sidecar 场景中同时运行两个 sidecar。这对于 tracing/logging （跟踪记录）插件非常有用。默认为 "first" |
| protocols | 将触发此插件的请求协议列表。可能的值是 http、https、tcp 和 tls。根据插件类型的不同，默认值以及该字段允许的可能值可能会发生变化。例如，仅在流模式下工作的插件可能只支持 tcp 和 tls。默认为 ["grpc","grpcs","http", "https"] |
| enabled   | 是否应用插件。默认值为 true                                  |
| tags      | 与插件相关联的可选字符串，用于分组和过滤                     |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "ce44eef5-41ed-47f6-baab-f725cecf98c7",
    "name": "rate-limiting",
    "created_at": 1422386534,
    "route": null,
    "service": null,
    "consumer": null,
    "config": {"hour":500, "minute":20},
    "run_on": "first",
    "protocols": ["http", "https"],
    "enabled": true,
    "tags": ["user-level", "low-priority"]
}
```



#### 更新或创建插件

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *PUT*    | */plugins/{plugin id}* |

| 属性      | 描述              |
| --------- | ----------------- |
| plugin id | 要检索的插件的 id |



**请求体**

| 属性      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| name      | 要添加的插件的名称。目前插件必须分别安装在每个 Kong 实例中   |
| route     | 如果设置，插件将只在通过指定的路由接收请求时激活。不管使用什么路由，都不要设置插件来激活它。默认为 null。使用  form-encoded 的 notation 是 route.id=<route_id> 对于 JSON，使用 "route":{"id":"<route_id>"} |
| service   | 如果设置了，插件将只在通过属于指定服务的路由之一接收请求时激活。不管服务是否匹配，都不要设置插件激活。默认为空。使用 form-encoded，notation 是 service.id=<service_id> 对于 JSON 使用 "service":{"id":"<service_id>"} |
| consumer  | 如果设置了，插件将只对已验证指定的请求激活插件。(注意，有些插件不能以这种方式仅限于消费者)。不管经过身份验证的使用者是谁，都不要设置插件激活。默认为空。对于 form-encoded，notation 是 consumer.id=<consumer_id> 对于 JSON，使用 "consumer":{"id":"<consumer_id>"}。 |
| config    | 插件的配置属性，可以在 Kong Hub 的插件文档页面上找到         |
| run_on    | 给定服务网格场景，控制此插件将运行在哪个 Kong 节点上。接受的值是：                                             * first 意思是 "在请求遇到的第一个 Kong 节点上运行"。在 API 逃逸场景中，这是正常的操作，因为在源和目标之间只有一个 Kong 节点。在 sidecar-to-sidecar 服务网格场景中，这意味着只在出站连接的 Kong sidecar 上运行插件。                                                                                                                            * second 意思是 "在请求遇到的第二个节点上运行"。此选项仅适用于 sidecar-to-sidecar 服务网格场景中，这意味着仅在入站连接的 Kong sidecar 上运行插件。                                                                     * all 意思是 "在所有节点上运行"，即在 sidecar-to-sidecar 场景中同时运行两个 sidecar。这对于 tracing/logging （跟踪记录）插件非常有用。默认为 "first" |
| protocols | 将触发此插件的请求协议列表。可能的值是 http、https、tcp 和 tls。根据插件类型的不同，默认值以及该字段允许的可能值可能会发生变化。例如，仅在流模式下工作的插件可能只支持 tcp 和 tls。默认为 ["grpc","grpcs","http", "https"] |
| enabled   | 是否应用插件。默认值为 true                                  |
| tags      | 与插件相关联的可选字符串，用于分组和过滤                     |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除插件

| 请求方式 | 请求路径               |
| -------- | ---------------------- |
| *DELETE* | */plugins/{plugin id}* |



**响应**

```json
HTTP 204 No Content
```



#### 检索启用插件

| 请求方式 | 请求路径           |
| -------- | ------------------ |
| *GET*    | */plugins/enabled* |



**响应**

```json
HTTP 200 OK
```

```json
{
    "enabled_plugins": [
        "jwt",
        "acl",
        "cors",
        "oauth2",
        "tcp-log",
        "udp-log",
        "file-log",
        "http-log",
        "key-auth",
        "hmac-auth",
        "basic-auth",
        "ip-restriction",
        "request-transformer",
        "response-transformer",
        "request-size-limiting",
        "rate-limiting",
        "response-ratelimiting",
        "aws-lambda",
        "bot-detection",
        "correlation-id",
        "datadog",
        "galileo",
        "ldap-auth",
        "loggly",
        "statsd",
        "syslog"
    ]
}
```



#### 检索插件配置



| 请求方式 | 请求路径                        |
| -------- | ------------------------------- |
| *GET*    | */plugins/schema/{plugin name}* |



**响应**

```json
HTTP 200 OK
```

```json
{
    "fields": {
        "hide_credentials": {
            "default": false,
            "type": "boolean"
        },
        "key_names": {
            "default": "function",
            "required": true,
            "type": "array"
        }
    }
}
```



### 证书对象



> 证书对象表示公共证书，可以选择与对应的私钥配对。Kong 使用这些对象来处理加密请求的 SSL/TLS，或者在验证客户端/服务的对等证书时用作受信任的 CA 存储。证书可选与 SNI 对象关联，以将证书/密钥（cert/key）对绑定到一个或多个主机名。
>



```json
{
    "id": "7fca84d6-7d37-4a74-a7b0-93e576089a41",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "key": "-----BEGIN RSA PRIVATE KEY-----...",
    "tags": ["user-level", "low-priority"]
}
```



#### 添加证书

| 请求方式 | 请求路径        |
| -------- | --------------- |
| *POST*   | */certificates* |



**请求体**

| 属性 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| cert | Pmm-encoded 的 SSL 密钥对公共证书链                          |
| key  | SSL 密钥对的 Pmm-encoded 的私钥                              |
| tags | 与插件相关联的可选字符串，用于分组和过滤                     |
| snis | 一个包含零个或多个主机名的数组，用于将此证书关联为 SNIs。这是一个 sugar 参数，它将在底层创建一个 SNI 对象，并将其与此证书关联，以便您方便使用。设置此属性的话，此证书必须具有与其关联的有效私钥 |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "7fca84d6-7d37-4a74-a7b0-93e576089a41",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "key": "-----BEGIN RSA PRIVATE KEY-----...",
    "tags": ["user-level", "low-priority"]
}
```



#### 证书列表

| 请求方式 | 请求路径        |
| -------- | --------------- |
| *GET*    | */certificates* |



**响应**

```json
HTTP 200 OK
```

```json
{
"data": [{
    "id": "d044b7d4-3dc2-4bbc-8e9f-6b7a69416df6",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "key": "-----BEGIN RSA PRIVATE KEY-----...",
    "tags": ["user-level", "low-priority"]
}, {
    "id": "a9b2107f-a214-47b3-add4-46b942187924",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "key": "-----BEGIN RSA PRIVATE KEY-----...",
    "tags": ["admin", "high-priority", "critical"]
}],

    "next": "http://localhost:8001/certificates?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索证书

| 请求方式 | 请求路径                         |
| -------- | -------------------------------- |
| *GET*    | */certificates/{certificate id}* |

| 属性           | 描述    |
| -------------- | ------- |
| certificate id | 证书 id |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "7fca84d6-7d37-4a74-a7b0-93e576089a41",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "key": "-----BEGIN RSA PRIVATE KEY-----...",
    "tags": ["user-level", "low-priority"]
}
```



#### 更新证书

| 请求方式 | 请求路径                         |
| -------- | -------------------------------- |
| *PATCH*  | */certificates/{certificate id}* |

| 属性           | 描述    |
| -------------- | ------- |
| certificate id | 证书 id |



**请求体**

| 属性 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| cert | Pmm-encoded 的 SSL 密钥对公共证书链                          |
| key  | SSL 密钥对的 Pmm-encoded 的私钥                              |
| tags | 与插件相关联的可选字符串，用于分组和过滤                     |
| snis | 一个包含零个或多个主机名的数组，用于将此证书关联为 SNIs。这是一个 sugar 参数，它将在底层创建一个 SNI 对象，并将其与此证书关联，以便您方便使用。设置此属性的话，此证书必须具有与其关联的有效私钥 |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "7fca84d6-7d37-4a74-a7b0-93e576089a41",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "key": "-----BEGIN RSA PRIVATE KEY-----...",
    "tags": ["user-level", "low-priority"]
}

```



#### 更新或创建证书

| 请求方式 | 请求路径                         |
| -------- | -------------------------------- |
| *PUT*    | */certificates/{certificate id}* |

| 属性           | 描述    |
| -------------- | ------- |
| certificate id | 证书 id |



**请求体**

| 属性 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| cert | Pmm-encoded 的 SSL 密钥对公共证书链                          |
| key  | SSL 密钥对的 Pmm-encoded 的私钥                              |
| tags | 与插件相关联的可选字符串，用于分组和过滤                     |
| snis | 一个包含零个或多个主机名的数组，用于将此证书关联为 SNIs。这是一个 sugar 参数，它将在底层创建一个 SNI 对象，并将其与此证书关联，以便您方便使用。设置此属性的话，此证书必须具有与其关联的有效私钥 |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除证书

| 请求方式 | 请求路径                         |
| -------- | -------------------------------- |
| *DELETE* | */certificates/{certificate id}* |

| 属性           | 描述    |
| -------------- | ------- |
| certificate id | 证书 id |



**响应**

```json
HTTP 204 No Content
```



### CA 证书对象



> CA 证书对象表示受信任的 CA。Kong 使用这些对象来验证客户端或服务器证书的有效性



```json
{
    "id": "04fbeacf-a9f1-4a5d-ae4a-b0407445db3f",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "tags": ["user-level", "low-priority"]
}

```



#### 添加 CA 证书

| 请求方式 | 请求路径           |
| -------- | ------------------ |
| *POST*   | */ca_certificates* |



**请求体**

| 属性 | 描述                               |
| ---- | ---------------------------------- |
| cert | PEM-encoded 编码的公共证书         |
| tags | 与证书关联的字符串，用于分组和筛选 |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "04fbeacf-a9f1-4a5d-ae4a-b0407445db3f",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "tags": ["user-level", "low-priority"]
}
```



#### CA 证书列表

| 请求方式 | 请求路径           |
| -------- | ------------------ |
| *GET*    | */ca_certificates* |



**响应**

```json
HTTP 200 OK
```

```json
{
"data": [{
    "id": "43429efd-b3a5-4048-94cb-5cc4029909bb",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "tags": ["user-level", "low-priority"]
}, {
    "id": "d26761d5-83a4-4f24-ac6c-cff276f2b79c",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "tags": ["admin", "high-priority", "critical"]
}],

    "next": "http://localhost:8001/ca_certificates?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索 CA 证书

| 请求方式 | 请求路径                               |
| -------- | -------------------------------------- |
| *GET*    | */ca_certificates/{ca_certificate id}* |

| 属性              | 描述                  |
| ----------------- | --------------------- |
| ca_certificate id | 要检索的 CA 证书的 id |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "04fbeacf-a9f1-4a5d-ae4a-b0407445db3f",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "tags": ["user-level", "low-priority"]
}

```



#### 更新 CA 证书

| 请求方式 | 请求路径                               |
| -------- | -------------------------------------- |
| *PATCH*  | */ca_certificates/{ca_certificate id}* |

| 属性              | 描述                  |
| ----------------- | --------------------- |
| ca_certificate id | 要检索的 CA 证书的 id |



**请求体**

| 属性 | 描述                               |
| ---- | ---------------------------------- |
| cert | PEM-encoded 编码的公共证书         |
| tags | 与证书关联的字符串，用于分组和筛选 |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "04fbeacf-a9f1-4a5d-ae4a-b0407445db3f",
    "created_at": 1422386534,
    "cert": "-----BEGIN CERTIFICATE-----...",
    "tags": ["user-level", "low-priority"]
}
```



#### 更新或创建 CA 证书

| 请求方式 | 请求路径                               |
| -------- | -------------------------------------- |
| *PUT*    | */ca_certificates/{ca_certificate id}* |

| 属性              | 描述                  |
| ----------------- | --------------------- |
| ca_certificate id | 要检索的 CA 证书的 id |



**请求体**

| 属性 | 描述                               |
| ---- | ---------------------------------- |
| cert | PEM-encoded 编码的公共证书         |
| tags | 与证书关联的字符串，用于分组和筛选 |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除 CA 证书

| 请求方式 | 请求路径                               |
| -------- | -------------------------------------- |
| *DELETE* | */ca_certificates/{ca_certificate id}* |

| 属性              | 描述                  |
| ----------------- | --------------------- |
| ca_certificate id | 要检索的 CA 证书的 id |



**响应**

```json
HTTP 204 No Content
```



### SNI 对象



> SNI 对象表示主机名到证书的多对一映射。也就是说，一个证书对象可以有许多与之关联的主机名；当 Kong 接收到 SSL 请求时，它使用客户端 Hello 中的 SNI 字段根据与证书关联的 SNI 查找证书对象。



```json
{
    "id": "91020192-062d-416f-a275-9addeeaffaf2",
    "name": "my-sni",
    "created_at": 1422386534,
    "tags": ["user-level", "low-priority"],
    "certificate": {"id":"a2e013e8-7623-4494-a347-6d29108ff68b"}
}
```

#### 创建 SNI



##### 创建 SNI

| 请求方式 | 请求路径 |
| -------- | -------- |
| *POST*   | */snis*  |



##### 创建与特定证书关联的 SNI

| 请求方式 | 请求路径                                      |
| -------- | --------------------------------------------- |
| *POST*   | */certificates/{certificate name or id}/snis* |

| 属性                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| certificate name or id | 证书的 id 或 name，该 id 或 name 属性应该与新创建的 SNI 关联 |



**请求体**

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| name        | 与给定证书关联的 SNI 名称                                    |
| tags        | 与证书关联的字符串，用于分组和筛选                           |
| certificate | 用于关联 SNI 主机名的证书的 id (UUID)。证书必须具有与之关联的有效私钥，以便 SNI 对象使用。对于 form-encoded，notation是 certificate.id=<certificate_id>。对于 JSON，使用 "certificate":{"id":"<certificate_id>"} |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "91020192-062d-416f-a275-9addeeaffaf2",
    "name": "my-sni",
    "created_at": 1422386534,
    "tags": ["user-level", "low-priority"],
    "certificate": {"id":"a2e013e8-7623-4494-a347-6d29108ff68"}
}
```



#### SNI 列表



##### SNI 列表

| 请求方式 | 请求路径 |
| -------- | -------- |
| *GET*    | */snis*  |



##### 与特定证书关联的 SNI 列表

| 请求方式 | 请求路径                                      |
| -------- | --------------------------------------------- |
| *GET*    | */certificates/{certificate name or id}/snis* |

| 属性                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| certificate name or id | 证书的 id 或 name，该 id 或 name 属性应该与新创建的 SNI 关联 |



**响应**

```json
HTTP 200 OK
```

```json
{
"data": [{
    "id": "147f5ef0-1ed6-4711-b77f-489262f8bff7",
    "name": "my-sni",
    "created_at": 1422386534,
    "tags": ["user-level", "low-priority"],
    "certificate": {"id":"a3ad71a8-6685-4b03-a101-980a953544f6"}
}, {
    "id": "b87eb55d-69a1-41d2-8653-8d706eecefc0",
    "name": "my-sni",
    "created_at": 1422386534,
    "tags": ["admin", "high-priority", "critical"],
    "certificate": {"id":"4e8d95d4-40f2-4818-adcb-30e00c349618"}
}],

    "next": "http://localhost:8001/snis?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索 SNI

| 请求方式 | 请求路径             |
| -------- | -------------------- |
| *GET*    | */snis/{name or id}* |

| 属性       | 描述                    |
| ---------- | ----------------------- |
| name or id | 要检索的 SNI id 或 name |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "91020192-062d-416f-a275-9addeeaffaf2",
    "name": "my-sni",
    "created_at": 1422386534,
    "tags": ["user-level", "low-priority"],
    "certificate": {"id":"a2e013e8-7623-4494-a347-6d29108ff68b"}
}

```



#### 更新 SNI

| 请求方式 | 请求路径             |
| -------- | -------------------- |
| *PATCH*  | */snis/{name or id}* |

| 属性       | 描述                    |
| ---------- | ----------------------- |
| name or id | 要检索的 SNI id 或 name |



**请求体**

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| name        | 与给定证书关联的 SNI 名称                                    |
| tags        | 与证书关联的字符串，用于分组和筛选                           |
| certificate | 用于关联 SNI 主机名的证书的 id (UUID)。证书必须具有与之关联的有效私钥，以便 SNI 对象使用。对于 form-encoded，notation是 certificate.id=<certificate_id>。对于 JSON，使用 "certificate":{"id":"<certificate_id>"} |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "91020192-062d-416f-a275-9addeeaffaf2",
    "name": "my-sni",
    "created_at": 1422386534,
    "tags": ["user-level", "low-priority"],
    "certificate": {"id":"a2e013e8-7623-4494-a347-6d29108ff68b"}
}
```



#### 更新或创建 SNI

| 请求方式 | 请求路径             |
| -------- | -------------------- |
| *PUT*    | */snis/{name or id}* |

| 属性       | 描述                    |
| ---------- | ----------------------- |
| name or id | 要检索的 SNI id 或 name |



**请求体**

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| name        | 与给定证书关联的 SNI 名称                                    |
| tags        | 与证书关联的字符串，用于分组和筛选                           |
| certificate | 用于关联 SNI 主机名的证书的 id (UUID)。证书必须具有与之关联的有效私钥，以便 SNI 对象使用。对于 form-encoded，notation是 certificate.id=<certificate_id>。对于 JSON，使用 "certificate":{"id":"<certificate_id>"} |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除 SNI

| 请求方式 | 请求路径             |
| -------- | -------------------- |
| *DELET*  | */snis/{name or id}* |

| 属性       | 描述                    |
| ---------- | ----------------------- |
| name or id | 要检索的 SNI id 或 name |



**响应**

```json
HTTP 204 No Content
```



### 上游对象

 

> 上游对象表示一个虚拟主机名，可用于在多个服务（target）上对传入请求进行负载均衡。例如，为主机为 service.v1.xyz 的服务对象的上游命名为 service.v1.xyz。对该服务的请求将代理到上游定义的 target。
>
> 
>
> 上游还包括一个健康检查，它能够根据 target 是否能够服务请求来启用和禁用目标。健康检查的配置存储在上游对象中，并应用于它的所有目标。



```json
{
    "id": "58c8ccbb-eafb-4566-991f-2ed4f678fa70",
    "created_at": 1422386534,
    "name": "my-upstream",
    "algorithm": "round-robin",
    "hash_on": "none",
    "hash_fallback": "none",
    "hash_on_cookie_path": "/",
    "slots": 10000,
    "healthchecks": {
        "active": {
            "https_verify_certificate": true,
            "unhealthy": {
                "http_statuses": [429, 404, 500, 501, 502, 503, 504, 505],
                "tcp_failures": 0,
                "timeouts": 0,
                "http_failures": 0,
                "interval": 0
            },
            "http_path": "/",
            "timeout": 1,
            "healthy": {
                "http_statuses": [200, 302],
                "interval": 0,
                "successes": 0
            },
            "https_sni": "example.com",
            "concurrency": 10,
            "type": "http"
        },
        "passive": {
            "unhealthy": {
                "http_failures": 0,
                "http_statuses": [429, 500, 503],
                "tcp_failures": 0,
                "timeouts": 0
            },
            "type": "http",
            "healthy": {
                "successes": 0,
                "http_statuses": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]
            }
        }
    },
    "tags": ["user-level", "low-priority"]
}

```



#### 添加上游

| 请求方式 | 请求路径     |
| -------- | ------------ |
| *POST*   | */upstreams* |



**请求体**

| 属性                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| name                                         | 这是一个主机名，它必须等于（equal）服务的主机                |
| algorithm                                    | 使用哪种负载均衡算法，round-robin、consistent-hashing 或 least-connections，默认：round-robin |
| hash_on                                      | 使用什么作为哈希输入：none（返回没有散列的加权循环方案）、consumer、ip、header 或 cookie。默认为 "none" |
| hash_fallback                                | 如果 primary hash_on 不返回哈希值，应该使用什么作为哈希输入（例如：缺少 header，或者没有标识消费者）。使用其中之一：none、consumer、ip、header 或 cookie。如果将hash_on 设置为 cookie，则不可用。默认为 "none" |
| hash_on_header                               | 要将值作为哈希输入的 header。只有当 hash_on 被设置为header 时才需要 |
| hash_fallback_header                         | 要将值作为哈希输入的 header。只有当 hash_fallback 设置为 header 时才需要 |
| hash_on_cookie                               | 获取值作为哈希输入的 cookie 名称。只有当 hash_on 或 hash_fallback 设置为 cookie 时才需要。如果指定的 cookie 不在请求中，Kong 将生成一个值并在响应中设置 cookie。 |
| hash_on_cookie_path                          | 要在响应 header 中设置的 cookie 路径。只有当 hash_on 或 hash_fallback 设置为 cookie 时才需要。默认为"/" |
| slots                                        | 负载均衡器算法中的槽数（10-65536）。默认为 10000             |
| healthchecks.active.https_verify_certificate | 使用 HTTPS 执行活跃健康检查时，是否检查远程主机的 SSL 证书的有效性。默认值为 true |
| healthchecks.active.unhealthy.http_statuses  | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以监听到这一点。默认为 `[429, 404, 500, 501, 502, 503, 504, 505]`，使用 form-encoded 表示为 http_statuses[]=429&http_statuses[]=404，使用 JSON 则是数组 |
| healthchecks.active.unhealthy.tcp_failures   | 活跃探测中 TCP 失败的次数认为目标不健康。默认值为 0          |
| healthchecks.active.unhealthy.timeouts       | 活跃探测中 TCP 超时的次数认为目标不健康。默认值为0           |
| healthchecks.active.unhealthy.http_failures  | 活跃探测中 HTTP 失败的次数（由 healthcheck .active.unhealth .http_statuses 定义），认为目标不健康。默认值为 0 |
| healthchecks.active.unhealthy.interval       | 对不健康目标进行主动健康检查的间隔(以秒为单位)。值为 0 表示不健康目标的活跃探测不应该执行。默认值为 0 |
| healthchecks.active.http_path                | 在 GET HTTP 请求中使用的路径，在活跃健康检查上作为探针运行。默认为 "/" |
| healthchecks.active.timeout                  | 活跃健康检查的套接字超时（以秒为单位）。默认为 1             |
| healthchecks.active.healthy.http_statuses    | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以观察到这一点。默认为 `[200, 302]`，notation 使用 form-encoded 表示为 http_statuses[]=200&http_statuses[]=302，使用 JSON 则是数组 |
| healthchecks.active.healthy.interval         | 对健康目标进行主动健康检查的间隔（以秒为单位）。值为 0 表示不应该执行针对健康目标的活动探测。默认值为 0 |
| healthchecks.active.healthy.successes        | 在活跃探测（由 healthcheck .active.health .http_statuses 定义）中考虑目标是否健康的成功次数。默认值为 0 |
| healthchecks.active.https_sni                | 使用 HTTPS 执行活跃健康检查时用作 SNI （服务名标识）的主机名。当使用 ip 配置目标时，尤其有用，这样就可以使用适当的 SNI 验证目标主机的证书 |
| healthchecks.active.concurrency              | 在活跃健康检查中同时检查的目标数量。默认为 10                |
| healthchecks.active.type                     | 是否使用 HTTP 或 HTTPS 执行活跃健康检查，或者只是尝试 TCP 连接。可能的值是 tcp、http 或 https。默认为 "http" |
| healthchecks.passive.unhealthy.http_failures | 代理流量中 HTTP 失败的次数（由 healthcheck .passive.unhealth .http_statuses 定义）认为目标不健康，由被动健康检查监听到。默认值为 0 |
| healthchecks.passive.unhealthy.http_statuses | 代理流量产生的 HTTP 状态数组，如被动健康检查所监听到的。默认值为 [429,500,503]。使用 form-encoded 的 notation 是 http_statuses[]=429&http_statuses[]=500。对于 JSON，使用数组 |
| healthchecks.passive.unhealthy.tcp_failures  | 代理流量中 TCP 失败的次数，认为目标不健康，如被动健康检查所监听到的。默认值为 0 |
| healthchecks.passive.unhealthy.timeouts      | 在代理通信中，认为目标不健康的超时次数，如被动健康检查所监听到的。默认值为 0 |
| healthchecks.passive.type                    | 是否执行解释 HTTP/HTTPS 状态的被动健康检查，或者只是检查 TCP 连接是否成功。可能的值是 tcp、http 或 https（在被动检查中，http 和 https 选项是等效的）。默认为 http |
| healthchecks.passive.healthy.successes       | 代理流量（由 healthchecks.passive.healthy.http_statuses 定义）中考虑目标健康的成功次数（由被动健康检查监听到）。默认值为 0 |
| healthchecks.passive.healthy.http_statuses   | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以观察到这一点。默认为 `[200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]`，使用 form-encoded 表示为 http_statuses[]=200&http_statuses[]=201，使用 JSON 则是数组 |
| tags                                         | 与证书关联的字符串，用于分组和筛选                           |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "58c8ccbb-eafb-4566-991f-2ed4f678fa70",
    "created_at": 1422386534,
    "name": "my-upstream",
    "algorithm": "round-robin",
    "hash_on": "none",
    "hash_fallback": "none",
    "hash_on_cookie_path": "/",
    "slots": 10000,
    "healthchecks": {
        "active": {
            "https_verify_certificate": true,
            "unhealthy": {
                "http_statuses": [429, 404, 500, 501, 502, 503, 504, 505],
                "tcp_failures": 0,
                "timeouts": 0,
                "http_failures": 0,
                "interval": 0
            },
            "http_path": "/",
            "timeout": 1,
            "healthy": {
                "http_statuses": [200, 302],
                "interval": 0,
                "successes": 0
            },
            "https_sni": "example.com",
            "concurrency": 10,
            "type": "http"
        },
        "passive": {
            "unhealthy": {
                "http_failures": 0,
                "http_statuses": [429, 500, 503],
                "tcp_failures": 0,
                "timeouts": 0
            },
            "type": "http",
            "healthy": {
                "successes": 0,
                "http_statuses": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]
            }
        }
    },
    "tags": ["user-level", "low-priority"]
}
```



#### 上游列表

| 请求方式 | 请求路径     |
| -------- | ------------ |
| *GET*    | */upstreams* |



**响应**

```json
HTTP 200 OK
```

```json
{
"data": [{
    "id": "ea29aaa3-3b2d-488c-b90c-56df8e0dd8c6",
    "created_at": 1422386534,
    "name": "my-upstream",
    "algorithm": "round-robin",
    "hash_on": "none",
    "hash_fallback": "none",
    "hash_on_cookie_path": "/",
    "slots": 10000,
    "healthchecks": {
        "active": {
            "https_verify_certificate": true,
            "unhealthy": {
                "http_statuses": [429, 404, 500, 501, 502, 503, 504, 505],
                "tcp_failures": 0,
                "timeouts": 0,
                "http_failures": 0,
                "interval": 0
            },
            "http_path": "/",
            "timeout": 1,
            "healthy": {
                "http_statuses": [200, 302],
                "interval": 0,
                "successes": 0
            },
            "https_sni": "example.com",
            "concurrency": 10,
            "type": "http"
        },
        "passive": {
            "unhealthy": {
                "http_failures": 0,
                "http_statuses": [429, 500, 503],
                "tcp_failures": 0,
                "timeouts": 0
            },
            "type": "http",
            "healthy": {
                "successes": 0,
                "http_statuses": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]
            }
        }
    },
    "tags": ["user-level", "low-priority"]
}, {
    "id": "4fe14415-73d5-4f00-9fbc-c72a0fccfcb2",
    "created_at": 1422386534,
    "name": "my-upstream",
    "algorithm": "round-robin",
    "hash_on": "none",
    "hash_fallback": "none",
    "hash_on_cookie_path": "/",
    "slots": 10000,
    "healthchecks": {
        "active": {
            "https_verify_certificate": true,
            "unhealthy": {
                "http_statuses": [429, 404, 500, 501, 502, 503, 504, 505],
                "tcp_failures": 0,
                "timeouts": 0,
                "http_failures": 0,
                "interval": 0
            },
            "http_path": "/",
            "timeout": 1,
            "healthy": {
                "http_statuses": [200, 302],
                "interval": 0,
                "successes": 0
            },
            "https_sni": "example.com",
            "concurrency": 10,
            "type": "http"
        },
        "passive": {
            "unhealthy": {
                "http_failures": 0,
                "http_statuses": [429, 500, 503],
                "tcp_failures": 0,
                "timeouts": 0
            },
            "type": "http",
            "healthy": {
                "successes": 0,
                "http_statuses": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]
            }
        }
    },
    "tags": ["admin", "high-priority", "critical"]
}],

    "next": "http://localhost:8001/upstreams?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### 检索上游



##### 检索上游

| 请求方式 | 请求路径                  |
| -------- | ------------------------- |
| *GET*    | */upstreams/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的上游的 id 或 name |



##### 检索与特定目标相关的上游

| 请求方式 | 请求路径                                     |
| -------- | -------------------------------------------- |
| *GET*    | */targets/{target host:port or id}/upstream* |

| 属性                   | 描述                                       |
| ---------------------- | ------------------------------------------ |
| target host:port or id | 与要检索的上游关联的目标的 id 或 host:port |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "58c8ccbb-eafb-4566-991f-2ed4f678fa70",
    "created_at": 1422386534,
    "name": "my-upstream",
    "algorithm": "round-robin",
    "hash_on": "none",
    "hash_fallback": "none",
    "hash_on_cookie_path": "/",
    "slots": 10000,
    "healthchecks": {
        "active": {
            "https_verify_certificate": true,
            "unhealthy": {
                "http_statuses": [429, 404, 500, 501, 502, 503, 504, 505],
                "tcp_failures": 0,
                "timeouts": 0,
                "http_failures": 0,
                "interval": 0
            },
            "http_path": "/",
            "timeout": 1,
            "healthy": {
                "http_statuses": [200, 302],
                "interval": 0,
                "successes": 0
            },
            "https_sni": "example.com",
            "concurrency": 10,
            "type": "http"
        },
        "passive": {
            "unhealthy": {
                "http_failures": 0,
                "http_statuses": [429, 500, 503],
                "tcp_failures": 0,
                "timeouts": 0
            },
            "type": "http",
            "healthy": {
                "successes": 0,
                "http_statuses": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]
            }
        }
    },
    "tags": ["user-level", "low-priority"]
}

```



#### 更新上游



##### 更新上游

| 请求方式 | 请求路径                  |
| -------- | ------------------------- |
| *PATCH*  | */upstreams/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的上游的 id 或 name |



##### 更新与特定 target 关联的上游

| 请求方式 | 请求路径                                     |
| -------- | -------------------------------------------- |
| *PATCH*  | */targets/{target host:port or id}/upstream* |

| 属性                   | 描述                                       |
| ---------------------- | ------------------------------------------ |
| target host:port or id | 与要检索的上游关联的目标的 id 或 host:port |



**请求体**

| 属性                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| name                                         | 这是一个主机名，它必须等于（equal）服务的主机                |
| algorithm                                    | 使用哪种负载均衡算法，round-robin、consistent-hashing 或 least-connections，默认：round-robin |
| hash_on                                      | 使用什么作为哈希输入：none（返回没有散列的加权循环方案）、consumer、ip、header 或 cookie。默认为 "none" |
| hash_fallback                                | 如果 primary hash_on 不返回哈希值，应该使用什么作为哈希输入（例如：缺少 header，或者没有标识消费者）。使用其中之一：none、consumer、ip、header 或 cookie。如果将hash_on 设置为 cookie，则不可用。默认为 "none" |
| hash_on_header                               | 要将值作为哈希输入的 header。只有当 hash_on 被设置为header 时才需要 |
| hash_fallback_header                         | 要将值作为哈希输入的 header。只有当 hash_fallback 设置为 header 时才需要 |
| hash_on_cookie                               | 获取值作为哈希输入的 cookie 名称。只有当 hash_on 或 hash_fallback 设置为 cookie 时才需要。如果指定的 cookie 不在请求中，Kong 将生成一个值并在响应中设置 cookie。 |
| hash_on_cookie_path                          | 要在响应 header 中设置的 cookie 路径。只有当 hash_on 或 hash_fallback 设置为 cookie 时才需要。默认为"/" |
| slots                                        | 负载均衡器算法中的槽数（10-65536）。默认为 10000             |
| healthchecks.active.https_verify_certificate | 使用 HTTPS 执行活跃健康检查时，是否检查远程主机的 SSL 证书的有效性。默认值为 true |
| healthchecks.active.unhealthy.http_statuses  | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以监听到这一点。默认为 `[429, 404, 500, 501, 502, 503, 504, 505]`，使用 form-encoded 表示为 http_statuses[]=429&http_statuses[]=404，使用 JSON 则是数组 |
| healthchecks.active.unhealthy.tcp_failures   | 活跃探测中 TCP 失败的次数认为目标不健康。默认值为 0          |
| healthchecks.active.unhealthy.timeouts       | 活跃探测中 TCP 超时的次数认为目标不健康。默认值为0           |
| healthchecks.active.unhealthy.http_failures  | 活跃探测中 HTTP 失败的次数（由 healthcheck .active.unhealth .http_statuses 定义），认为目标不健康。默认值为 0 |
| healthchecks.active.unhealthy.interval       | 对不健康目标进行主动健康检查的间隔(以秒为单位)。值为 0 表示不健康目标的活跃探测不应该执行。默认值为 0 |
| healthchecks.active.http_path                | 在 GET HTTP 请求中使用的路径，在活跃健康检查上作为探针运行。默认为 "/" |
| healthchecks.active.timeout                  | 活跃健康检查的套接字超时（以秒为单位）。默认为 1             |
| healthchecks.active.healthy.http_statuses    | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以观察到这一点。默认为 `[200, 302]`，notation 使用 form-encoded 表示为 http_statuses[]=200&http_statuses[]=302，使用 JSON 则是数组 |
| healthchecks.active.healthy.interval         | 对健康目标进行主动健康检查的间隔（以秒为单位）。值为 0 表示不应该执行针对健康目标的活动探测。默认值为 0 |
| healthchecks.active.healthy.successes        | 在活跃探测（由 healthcheck .active.health .http_statuses 定义）中考虑目标是否健康的成功次数。默认值为 0 |
| healthchecks.active.https_sni                | 使用 HTTPS 执行活跃健康检查时用作 SNI （服务名标识）的主机名。当使用 ip 配置目标时，尤其有用，这样就可以使用适当的 SNI 验证目标主机的证书 |
| healthchecks.active.concurrency              | 在活跃健康检查中同时检查的目标数量。默认为 10                |
| healthchecks.active.type                     | 是否使用 HTTP 或 HTTPS 执行活跃健康检查，或者只是尝试 TCP 连接。可能的值是 tcp、http 或 https。默认为 "http" |
| healthchecks.passive.unhealthy.http_failures | 代理流量中 HTTP 失败的次数（由 healthcheck .passive.unhealth .http_statuses 定义）认为目标不健康，由被动健康检查监听到。默认值为 0 |
| healthchecks.passive.unhealthy.http_statuses | 代理流量产生的 HTTP 状态数组，如被动健康检查所监听到的。默认值为 [429,500,503]。使用 form-encoded 的 notation 是 http_statuses[]=429&http_statuses[]=500。对于 JSON，使用数组 |
| healthchecks.passive.unhealthy.tcp_failures  | 代理流量中 TCP 失败的次数，认为目标不健康，如被动健康检查所监听到的。默认值为 0 |
| healthchecks.passive.unhealthy.timeouts      | 在代理通信中，认为目标不健康的超时次数，如被动健康检查所监听到的。默认值为 0 |
| healthchecks.passive.type                    | 是否执行解释 HTTP/HTTPS 状态的被动健康检查，或者只是检查 TCP 连接是否成功。可能的值是 tcp、http 或 https（在被动检查中，http 和 https 选项是等效的）。默认为 http |
| healthchecks.passive.healthy.successes       | 代理流量（由 healthchecks.passive.healthy.http_statuses 定义）中考虑目标健康的成功次数（由被动健康检查监听到）。默认值为 0 |
| healthchecks.passive.healthy.http_statuses   | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以观察到这一点。默认为 `[200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]`，使用 form-encoded 表示为 http_statuses[]=200&http_statuses[]=201，使用 JSON 则是数组 |
| tags                                         | 与证书关联的字符串，用于分组和筛选                           |



**响应**

```json
HTTP 200 OK
```

```json
{
    "id": "58c8ccbb-eafb-4566-991f-2ed4f678fa70",
    "created_at": 1422386534,
    "name": "my-upstream",
    "algorithm": "round-robin",
    "hash_on": "none",
    "hash_fallback": "none",
    "hash_on_cookie_path": "/",
    "slots": 10000,
    "healthchecks": {
        "active": {
            "https_verify_certificate": true,
            "unhealthy": {
                "http_statuses": [429, 404, 500, 501, 502, 503, 504, 505],
                "tcp_failures": 0,
                "timeouts": 0,
                "http_failures": 0,
                "interval": 0
            },
            "http_path": "/",
            "timeout": 1,
            "healthy": {
                "http_statuses": [200, 302],
                "interval": 0,
                "successes": 0
            },
            "https_sni": "example.com",
            "concurrency": 10,
            "type": "http"
        },
        "passive": {
            "unhealthy": {
                "http_failures": 0,
                "http_statuses": [429, 500, 503],
                "tcp_failures": 0,
                "timeouts": 0
            },
            "type": "http",
            "healthy": {
                "successes": 0,
                "http_statuses": [200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]
            }
        }
    },
    "tags": ["user-level", "low-priority"]
}
```



#### 更新或创建上游



##### 创建或更新上游

| 请求方式 | 请求路径                  |
| -------- | ------------------------- |
| *PUT*    | */upstreams/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的上游的 id 或 name |



##### 创建或更新与特定 target 相关联的上游

| 请求方式 | 请求路径                                     |
| -------- | -------------------------------------------- |
| *PUT*    | */targets/{target host:port or id}/upstream* |

| 属性                   | 描述                                       |
| ---------------------- | ------------------------------------------ |
| target host:port or id | 与要检索的上游关联的目标的 id 或 host:port |



**请求体**

| 属性                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| name                                         | 这是一个主机名，它必须等于（equal）服务的主机                |
| algorithm                                    | 使用哪种负载均衡算法，round-robin、consistent-hashing 或 least-connections，默认：round-robin |
| hash_on                                      | 使用什么作为哈希输入：none（返回没有散列的加权循环方案）、consumer、ip、header 或 cookie。默认为 "none" |
| hash_fallback                                | 如果 primary hash_on 不返回哈希值，应该使用什么作为哈希输入（例如：缺少 header，或者没有标识消费者）。使用其中之一：none、consumer、ip、header 或 cookie。如果将hash_on 设置为 cookie，则不可用。默认为 "none" |
| hash_on_header                               | 要将值作为哈希输入的 header。只有当 hash_on 被设置为header 时才需要 |
| hash_fallback_header                         | 要将值作为哈希输入的 header。只有当 hash_fallback 设置为 header 时才需要 |
| hash_on_cookie                               | 获取值作为哈希输入的 cookie 名称。只有当 hash_on 或 hash_fallback 设置为 cookie 时才需要。如果指定的 cookie 不在请求中，Kong 将生成一个值并在响应中设置 cookie。 |
| hash_on_cookie_path                          | 要在响应 header 中设置的 cookie 路径。只有当 hash_on 或 hash_fallback 设置为 cookie 时才需要。默认为"/" |
| slots                                        | 负载均衡器算法中的槽数（10-65536）。默认为 10000             |
| healthchecks.active.https_verify_certificate | 使用 HTTPS 执行活跃健康检查时，是否检查远程主机的 SSL 证书的有效性。默认值为 true |
| healthchecks.active.unhealthy.http_statuses  | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以监听到这一点。默认为 `[429, 404, 500, 501, 502, 503, 504, 505]`，使用 form-encoded 表示为 http_statuses[]=429&http_statuses[]=404，使用 JSON 则是数组 |
| healthchecks.active.unhealthy.tcp_failures   | 活跃探测中 TCP 失败的次数认为目标不健康。默认值为 0          |
| healthchecks.active.unhealthy.timeouts       | 活跃探测中 TCP 超时的次数认为目标不健康。默认值为0           |
| healthchecks.active.unhealthy.http_failures  | 活跃探测中 HTTP 失败的次数（由 healthcheck .active.unhealth .http_statuses 定义），认为目标不健康。默认值为 0 |
| healthchecks.active.unhealthy.interval       | 对不健康目标进行主动健康检查的间隔(以秒为单位)。值为 0 表示不健康目标的活跃探测不应该执行。默认值为 0 |
| healthchecks.active.http_path                | 在 GET HTTP 请求中使用的路径，在活跃健康检查上作为探针运行。默认为 "/" |
| healthchecks.active.timeout                  | 活跃健康检查的套接字超时（以秒为单位）。默认为 1             |
| healthchecks.active.healthy.http_statuses    | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以观察到这一点。默认为 `[200, 302]`，notation 使用 form-encoded 表示为 http_statuses[]=200&http_statuses[]=302，使用 JSON 则是数组 |
| healthchecks.active.healthy.interval         | 对健康目标进行主动健康检查的间隔（以秒为单位）。值为 0 表示不应该执行针对健康目标的活动探测。默认值为 0 |
| healthchecks.active.healthy.successes        | 在活跃探测（由 healthcheck .active.health .http_statuses 定义）中考虑目标是否健康的成功次数。默认值为 0 |
| healthchecks.active.https_sni                | 使用 HTTPS 执行活跃健康检查时用作 SNI （服务名标识）的主机名。当使用 ip 配置目标时，尤其有用，这样就可以使用适当的 SNI 验证目标主机的证书 |
| healthchecks.active.concurrency              | 在活跃健康检查中同时检查的目标数量。默认为 10                |
| healthchecks.active.type                     | 是否使用 HTTP 或 HTTPS 执行活跃健康检查，或者只是尝试 TCP 连接。可能的值是 tcp、http 或 https。默认为 "http" |
| healthchecks.passive.unhealthy.http_failures | 代理流量中 HTTP 失败的次数（由 healthcheck .passive.unhealth .http_statuses 定义）认为目标不健康，由被动健康检查监听到。默认值为 0 |
| healthchecks.passive.unhealthy.http_statuses | 代理流量产生的 HTTP 状态数组，如被动健康检查所监听到的。默认值为 [429,500,503]。使用 form-encoded 的 notation 是 http_statuses[]=429&http_statuses[]=500。对于 JSON，使用数组 |
| healthchecks.passive.unhealthy.tcp_failures  | 代理流量中 TCP 失败的次数，认为目标不健康，如被动健康检查所监听到的。默认值为 0 |
| healthchecks.passive.unhealthy.timeouts      | 在代理通信中，认为目标不健康的超时次数，如被动健康检查所监听到的。默认值为 0 |
| healthchecks.passive.type                    | 是否执行解释 HTTP/HTTPS 状态的被动健康检查，或者只是检查 TCP 连接是否成功。可能的值是 tcp、http 或 https（在被动检查中，http 和 https 选项是等效的）。默认为 http |
| healthchecks.passive.healthy.successes       | 代理流量（由 healthchecks.passive.healthy.http_statuses 定义）中考虑目标健康的成功次数（由被动健康检查监听到）。默认值为 0 |
| healthchecks.passive.healthy.http_statuses   | 对于 JSON，使用 HTTP 状态数组，当代理流量生成时，这些状态表示健康状态，通过被动健康检查可以观察到这一点。默认为 `[200, 201, 202, 203, 204, 205, 206, 207, 208, 226, 300, 301, 302, 303, 304, 305, 306, 307, 308]`，使用 form-encoded 表示为 http_statuses[]=200&http_statuses[]=201，使用 JSON 则是数组 |
| tags                                         | 与证书关联的字符串，用于分组和筛选                           |



> 使用请求体中指定的定义在请求的资源下 Inserts（或 replaces） 服务。服务将通过 name 或 id 属性进行标识。
>
> 当 name 或 id 属性具有 UUID 结构时，inserted/replaced 的服务将通过其 id 标识，否则将通过其名称标识。
>
> 当创建一个新服务而不指定 id（既不在 URL 中也不在正文中）时，它将自动生成。
>
> 注意，不允许在 URL 中指定名称，而在请求体中指定另一个名称。



**响应**

```json
HTTP 201 Created or HTTP 200 OK
```



#### 删除上游



##### 删除上游

| 请求方式 | 请求路径                  |
| -------- | ------------------------- |
| *DELETE* | */upstreams/{name or id}* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的上游的 id 或 name |



##### 删除与特定 target 相关的上游

| 请求方式 | 请求路径                                     |
| -------- | -------------------------------------------- |
| *DELETE* | */targets/{target host:port or id}/upstream* |

| 属性                   | 描述                                       |
| ---------------------- | ------------------------------------------ |
| target host:port or id | 与要检索的上游关联的目标的 id 或 host:port |



**响应**

```json
HTTP 204 No Content
```



#### 节点上游健康状况

| 请求方式 | 请求路径                          |
| -------- | --------------------------------- |
| *GET*    | */upstreams/{name or id}/health/* |

| 属性       | 描述                      |
| ---------- | ------------------------- |
| name or id | 要检索的上游的 id 或 name |



**响应**

```json
HTTP 200 OK
```

```json
{
    "total": 2,
    "node_id": "cbb297c0-14a9-46bc-ad91-1d0ef9b42df9",
    "data": [
        {
            "created_at": 1485524883980,
            "id": "18c0ad90-f942-4098-88db-bbee3e43b27f",
            "health": "HEALTHY",
            "target": "127.0.0.1:20000",
            "upstream_id": "07131005-ba30-4204-a29f-0927d53257b4",
            "weight": 100
        },
        {
            "created_at": 1485524914883,
            "id": "6c6f34eb-e6c3-4c1f-ac58-4060e5bca890",
            "health": "UNHEALTHY",
            "target": "127.0.0.1:20002",
            "upstream_id": "07131005-ba30-4204-a29f-0927d53257b4",
            "weight": 200
        }
    ]
}
```



### Target 对象



> target  是一个 ipaddress/hostname，端口标识后端服务的实例。每个上游都可以有许多 target，并且可以动态地添加 target。变化是动态执行的。
>
> 因为上游维护 target 更改的历史记录，所以不能删除或修改 target。若要禁用 target，请发布权重为 0 的新 target；或者，使用 `DELETE` convenient 方法来完成相同的操作。



```json
{
    "id": "a3395f66-2af6-4c79-bea2-1b6933764f80",
    "created_at": 1422386534,
    "upstream": {"id":"885a0392-ef1b-4de3-aacf-af3f1697ce2c"},
    "target": "example.com:8000",
    "weight": 100,
    "tags": ["user-level", "low-priority"]
}
```



#### 添加 target 



##### 创建与特定上有关联的 target 

| 请求方式 | 请求路径                                        |
| -------- | ----------------------------------------------- |
| *POST*   | */upstreams/{upstream host:port or id}/targets* |

| 属性                     | 描述                                                    |
| ------------------------ | ------------------------------------------------------- |
| upstream host:port or id | 应该与新创建的 target 关联的上游的 id 或 host:port 属性 |



**请求体**

| 属性   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| target | target 地址（ip 或主机名）和端口。如果主机名解析为 SRV 记录，则端口值将被 DNS 记录中的值覆盖 |
| weight | target 在上游负载均衡器（0-1000）中获得的权重。如果主机名解析为 SRV 记录，则权重值将被 DNS 记录中的值覆盖。默认为 100 |
| tags   | 与证书关联的字符串，用于分组和筛选                           |



**响应**

```json
HTTP 201 Created
```

```json
{
    "id": "a3395f66-2af6-4c79-bea2-1b6933764f80",
    "created_at": 1422386534,
    "upstream": {"id":"885a0392-ef1b-4de3-aacf-af3f1697ce2c"},
    "target": "example.com:8000",
    "weight": 100,
    "tags": ["user-level", "low-priority"]
}
```



#### target  列表



##### 与特定上游相关的 target  

| 请求方式 | 请求路径                                        |
| -------- | ----------------------------------------------- |
| *GET*    | */upstreams/{upstream host:port or id}/targets* |

| 属性                     | 描述                                                    |
| ------------------------ | ------------------------------------------------------- |
| upstream host:port or id | 应该与新创建的 target 关联的上游的 id 或 host:port 属性 |



**响应**

```json
HTTP 200 OK
```

```json
{
"data": [{
    "id": "f5a9c0ca-bdbb-490f-8928-2ca95836239a",
    "created_at": 1422386534,
    "upstream": {"id":"173a6cee-90d1-40a7-89cf-0329eca780a6"},
    "target": "example.com:8000",
    "weight": 100,
    "tags": ["user-level", "low-priority"]
}, {
    "id": "bdab0e47-4e37-4f0b-8fd0-87d95cc4addc",
    "created_at": 1422386534,
    "upstream": {"id":"f00c6da4-3679-4b44-b9fb-36a19bd3ae83"},
    "target": "example.com:8000",
    "weight": 100,
    "tags": ["admin", "high-priority", "critical"]
}],

    "next": "http://localhost:8001/targets?offset=6378122c-a0a1-438d-a5c6-efabae9fb969"
}
```



#### DELETE target

| 请求方式 | 请求路径                                                     |
| -------- | ------------------------------------------------------------ |
| *DELETE* | */upstreams/{upstream name or id}/targets/{host:port or id}* |

| 属性                | 描述                                                       |
| ------------------- | ---------------------------------------------------------- |
| upstream name or id | 要删除 target 的 id 或上游的 name                          |
| host:port or id     | 要删除的 target 的 host:port 组合，或现有 target 条目的 id |



**响应**

```json
HTTP 204 No Content
```



#### 设置 target  地址为健康



> 将负载均衡器中的 target 解析的单个地址的当前健康状态设置为整个 Kong 集群中 "healthy"。
>
> 此端点可用于手动重新启用由 target 解析的地址，该 target 以前被上游的健康检查禁用。上游只将请求转发给正常节点，因此这个调用告诉 Kong 重新开始使用这个地址。
>
> 这将重置运行在 Kong 节点的所有工作中的健康检查的健康计数器，并广播集群范围内的消息，以便将 "health" 状态传播到整个 Kong 集群。



| 请求方式 | 请求路径                                                     |
| -------- | ------------------------------------------------------------ |
| *POST*   | */upstreams/{upstream name or id}/targets/{target or id}/{address}/healthy* |

| 属性                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| upstream name or id | 上游的 id 或 name                                            |
| target or id        | 要设置为健康状态的 target 的 host/port 组合，或现有 target 条目的 id |
| address             | 要设置为健康的地址的 host/port 组合                          |



**响应**

```json
HTTP 204 No Content
```



#### 设置 target  地址为不健康



> 将负载均衡器中的 TARGET 解析的单个地址的当前健康状态设置为整个 Kong 集群中 "unhealthy"。
>
> 此端点可用于手动禁用地址并使其停止响应请求。上游只将请求转发给正常节点，因此这个调用告诉 Kong 开始跳过这个地址。
>
> 这个调用重置运行在 Kong 节点的所有工作中的健康检查程序的健康计数器，并广播集群范围内的消息，以便将 "unhealthy" 状态传播到整个 Kong 集群。
>
> 对于不健康的地址，将继续执行活跃健康检查。注意，如果启用了活跃健康检查，并且探针检测到地址实际上是健康的，它将自动重新启用它。要从均衡器中永久删除 target，应该删除 target。



| 请求方式 | 请求路径                                                     |
| -------- | ------------------------------------------------------------ |
| *POST*   | */upstreams/{upstream name or id}/targets/{target or id}/unhealthy* |

| 属性                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| upstream name or id | 上游的 id 或 name                                            |
| target or id        | 要设置为健康状态的 target 的 host/port 组合，或现有 target 条目的 id |
|                     |                                                              |



**响应**

```json
HTTP 204 No Content
```



#### 设置 target  为健康



> 将负载均衡器中的 target  的当前健康状态设置为整个Kong集群中的 "healthy"。由该 target 解析的所有地址将设置为 "healthy"
>
> 此端点可用于手动重新启用以前被上游的健康检查禁用的 target。上游只将请求转发给健康的节点，因此这个调用告诉 Kong 再次开始使用这个 target。
>
> 这将重置运行在 Kong 节点的所有工作中的健康检查的健康计数器，并广播集群范围内的消息，以便将 "health" 状态传播到整个 Kong 集群。



| 请求方式 | 请求路径                                                     |
| -------- | ------------------------------------------------------------ |
| *POST*   | */upstreams/{upstream name or id}/targets/{target or id}/healthy* |

| 属性                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| upstream name or id | 上游的 id 或 name                                            |
| target or id        | 要设置为健康状态的 target 的 host/port 组合，或现有 target 条目的 id |
|                     |                                                              |



**响应**

```json
HTTP 204 No Content
```



#### 设置 target  为不健康



> 将负载均衡器中 target 的当前健康状态设置为整个 Kong 集群中的 "unhealthy"。由该 target 解析的所有地址将会设置为 "unhealthy" 
>
> 此端点可用于手动禁用 target 并使其停止响应请求。上游只将请求转发给健康节点，因此这个调用告诉 Kong 开始跳过这个目标。
>
> 此调用重置运行在 Kong 节点的所有工作中的健康检查程序的健康计数器，并广播集群范围内的消息，以便将 "unhealthy" 状态传播到整个 Kong 集群。
>
> 对 "unhealthy" 的 target 继续执行活跃的健康检查。注意，如果启用了活跃健康检查，并且探针检测到 target 实际上是健康的，它将自动重新启用它。要从均衡器中永久删除 target，应该删除 target。



| 请求方式 | 请求路径                                                     |
| -------- | ------------------------------------------------------------ |
| *POST*   | */upstreams/{upstream name or id}/targets/{target or id}/unhealthy* |

| 属性                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| upstream name or id | 上游的 id 或 name                                            |
| target or id        | 要设置为健康状态的 target 的 host/port 组合，或现有 target 条目的 id |
|                     |                                                              |



**响应**

```json
HTTP 204 No Content
```



#### 列出所有 target 



> 列出上游的所有 target。可以返回同一 target 的多个 target 对象，显示特定 target 的更改历史。具有最新 created_at 的 target 对象是当前定义。



| 请求方式 | 请求路径                               |
| -------- | -------------------------------------- |
| *GET*    | */upstreams/{name or id}/targets/all/* |

| 属性       | 描述                            |
| ---------- | ------------------------------- |
| name or id | 用于列出目标的 id 或上游的 name |



**响应**

```json
HTTP 200 OK
```

```json
{
    "total": 2,
    "data": [
        {
            "created_at": 1485524883980,
            "id": "18c0ad90-f942-4098-88db-bbee3e43b27f",
            "target": "127.0.0.1:20000",
            "upstream_id": "07131005-ba30-4204-a29f-0927d53257b4",
            "weight": 100
        },
        {
            "created_at": 1485524914883,
            "id": "6c6f34eb-e6c3-4c1f-ac58-4060e5bca890",
            "target": "127.0.0.1:20002",
            "upstream_id": "07131005-ba30-4204-a29f-0927d53257b4",
            "weight": 200
        }
    ]
}
```

















