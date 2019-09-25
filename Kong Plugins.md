

> 再次重申多个插件对于某个服务或者路由或其他服务的使用，存在优先级概念！
>
> 以下操作是在 DB 模式下进行。
>
> 以下所有内容均使用命令行操作解决。如需使用 Postman 自行转换。



### 身份验证



> 我们主要使用身份验证来保护我们的服务



#### Basic Auth



**step1: 为服务开启插件**

```shell
curl -X POST http://10.20.11.117:8001/services/77458fdb-deeb-4b66-8b42-ba749cdd9838/plugins --data "name=basic-auth" --data "config.hide_credentials=true"
```

*注意事项: service id 可以为 name*



**step2: 为路由开启插件**

```shell
curl -X POST http://10.20.11.117:8001/routes/d82735e1-b3a4-4ff5-afd7-908d7e79c07c/plugins --data "name=basic-auth" --data "config.hide_credentials=true"
```

*注意事项: route id 可以为 name*



**step3: 创建证书**

```shell
curl -X POST http://kong:8001/consumers/andy/basic-auth \
             --data "username=andy" \
             --data "password=onceas"
```

*注意事项: andy 是我的测试角色，请更换为实际角色，username 和 password 请更换为自己的*



**step4: 测试证书**

```shell
curl http://10.20.11.117:8000/baidu -H 'Authorization: Basic YW5keTpvbmNlYXM='
```

```html
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesh<title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=trform name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rs=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一ews.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=htt.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=/品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&//jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

*注意: 我的这个服务是转发到百度，可以看到是正确的，上文中的 "YW5keTpvbmNlYXM=" 是  "andy:onceas" 的 base64 加密结果，如果你不想加密可以尝试访问 web，可以正常输入用户名密码*



```shell
curl http://10.20.11.117:8000/baidu -H 'Authorization: Basic YW5keTpvbm='
```

```json
{"message":"Invalid authentication credentials"}
```

*注意，这里是账号密码错误的结果！*



####  Key Auth



**step1: 为服务开启插件**

```shell
curl -X POST http://10.20.11.117:8001/services/1b14c838-b98a-4f11-b21e-66ce2480510d/plugins \
			 --data "name=key-auth"	
```

*注意事项: service id 可以为 name*



**step2: 为路由开启插件** 

```shell
curl -X POST http://10.20.11.117:8001/routes/7985e3a3-2908-46f2-9510-d51e60e636df/plugins  \                --data "name=key-auth"
```

*注意事项: route id 可以为 name*



**step3: 自定义一个 API key 给消费者**

```shell
curl -X POST http://10.20.11.117:8001/consumers/json/key-auth/ --data "key=test_api_key_json"
```

*注意事项: json 是我的测试角色，请更换为实际角色，test_api_key_json 是我的测试 key，请更换为实际 key*



**setp4: 测试 key**

```shell
curl -i -X GET --url http://10.20.11.117:8000/demoapi --header "apikey: test_api_key_json" --data "name=sss"
```

```yaml
HTTP/1.1 200 
Content-Type: text/plain;charset=UTF-8
Content-Length: 78
Connection: keep-alive
Date: Wed, 25 Sep 2019 03:10:11 GMT
X-Kong-Upstream-Latency: 4
X-Kong-Proxy-Latency: 1
Via: kong/1.3.0rc1

success  from dubbo v2.7: Hello null, response from provider: 172.17.0.7:27880
```

*可以看出携带自定义 key 访问是正确的，这已经证明我们启动插件成功*



```shell
curl -i -X GET --url http://10.20.11.117:8000/demoapi --data "name=sss"
```

```yaml
HTTP/1.1 401 Unauthorized
Date: Wed, 25 Sep 2019 02:18:40 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
WWW-Authenticate: Key realm="kong"
Content-Length: 41
Server: kong/1.3.0rc1

{"message":"No API key found in request"}
```

*如果不携带 key 很明显是错误的！*



**step5：删除 key**

```shell
curl -X DELETE http://10.20.11.117:8001/consumers/json/key-auth/04e94904-f040-491a-83f5-119fb078eac8
```

*注意事项: json 是我的测试角色，请更换为实际角色，04e94904-f040-491a-83f5-119fb078eac8 id是我的测试 id，请更换为实际 id*



### 监控



#### Prometheus



> 本文使用 promethues 监控，默认你已经配置好了 prometheus 配置文件。



**step1: 为某个服务启用 Prometheus**

```shell
curl -X POST http://10.20.11.117:8001/services/1b14c838-b98a-4f11-b21e-66ce2480510d/plugins \
    --data "name=prometheus" 
```



**step2: 为全部服务启用 Prometheus**

```shell
curl http://10.20.11.117:8001/plugins -d name=prometheus
```



**访问 Prometheus**

```javascript
10.20.11.117:9090/targets
```



### 日志



#### nginx



> 推荐直接使用 access.log，可以在部署时挂载出来



#### http-log



**step1: 为某个服务启用 http-log**

```shell
curl -X POST http://10.20.11.117:8001/services/1b14c838-b98a-4f11-b21e-66ce2480510d/plugins \
    --data "name=http-log"  \
    --data "config.http_endpoint=http://10.20.11.117:9200/kong/kong-log" \
    --data "config.method=POST" \
    --data "config.timeout=1000" \
    --data "config.keepalive=1000"
```

*注意：id 请替换为自己的 id，http_endpoint 端点本文选择储存到 elasticsearch 里，这里可以任意选择*



**step2: 为某个路由启用 http-log**

```shell
curl -X POST http://10.20.11.117:8001/routes/7985e3a3-2908-46f2-9510-d51e60e636df/plugins \
    --data "name=http-log"  \
    --data "config.http_endpoint=http://10.20.11.117:9200/kong/kong-log" \
    --data "config.method=POST" \
    --data "config.timeout=1000" \
    --data "config.keepalive=1000"
```

*注意：id 请替换为自己的 id，http_endpoint 端点本文选择储存到 elasticsearch 里，这里可以任意选择*



**step3: 为某个消费者启用 http-log**

```shell
curl -X POST http://10.20.11.117:8001/consumers/json/plugins \
    --data "name=http-log" \
     \
    --data "config.http_endpoint=http://10.20.11.117:9200/kong/kong-log" \
    --data "config.method=POST" \
    --data "config.timeout=1000" \
    --data "config.keepalive=1000"
```

*注意：json 用户请替换为自己的 用户名，http_endpoint 端点本文选择储存到 elasticsearch 里，这里可以任意选择*



**step4：验证插件**

```shell
curl 10.20.11.117:9200/kong/kong-log/_search
```

```json
{
    .......
    "hits": {
        "total": 24,
        "max_score": 1.0,
        "hits": [
			.......
            {
                "_index": "kong-es",
                "_type": "log",
                "_id": "7KfzkmwBId3qyRdicnSi",
                "_score": 1.0,
                "_source": {
                    "latencies": {
                        "request": 3,
                        "kong": 1,
                        "proxy": 1
                    },
                    "service": {
                        "host": "10.20.11.117",
                        "created_at": 1565339929,
                        "connect_timeout": 60000,
                        "id": "455c02b5-3c92-4d50-a9fd-6801debcb4df",
                        "protocol": "http",
                        "name": "log-service2",
                        "read_timeout": 60000,
                        "port": 8001,
                        "updated_at": 1565593966,
                        "write_timeout": 60000,
                        "retries": 5
                    },
                    "request": {
                        "querystring": {},
                        "size": "288",
                        "uri": "/",
                        "url": "http://konghq:8000/",
                        "headers": {
                            "host": "konghq",
                            "content-type": "application/json",
                            "postman-token": "7a4a6958-efd4-4f89-b37f-5fe5a1c72bf6",
                            "apikey": "aZdNzdLSmqZNYTWfebbFVT3WuNNxpFoh",
                            "accept": "*/*",
                            "cache-control": "no-cache",
                            "accept-encoding": "gzip, deflate",
                            "user-agent": "PostmanRuntime/7.15.2",
                            "connection": "keep-alive"
                        },
                        "method": "GET"
                    },
                    "client_ip": "10.20.50.137",
                    "tries": [
                        {
                            "balancer_latency": 0,
                            "port": 8001,
                            "balancer_start": 1565833523872,
                            "ip": "10.20.11.117"
                        }
                    ],
                    "upstream_uri": "/",
                    "response": {
                        "headers": {
                            "content-type": "application/json; charset=utf-8",
                            "date": "Thu, 15 Aug 2019 01:45:23 GMT",
                            "x-kong-proxy-latency": "1",
                            "content-length": "6242",
                            "server": "kong/1.2.1",
                            "via": "kong/1.2.1",
                            "access-control-allow-origin": "*",
                            "x-kong-upstream-latency": "1",
                            "connection": "close"
                        },
                        "status": 200,
                        "size": "6513"
                    },
                    "route": {
                        "id": "562bc712-325f-4006-897d-92f3d211a119",
                        "paths": [],
                        "protocols": [
                            "http",
                            "https"
                        ],
                        "created_at": 1565340038,
                        "hosts": [
                            "konghq"
                        ],
                        "name": "log2-route",
                        "preserve_host": false,
                        "regex_priority": 0,
                        "strip_path": true,
                        "updated_at": 1565764672,
                        "https_redirect_status_code": 426,
                        "service": {
                            "id": "455c02b5-3c92-4d50-a9fd-6801debcb4df"
                        },
                        "methods": [
                            "GET"
                        ]
                    },
                    "started_at": 1565833523871
                }
            },

           .......
        ]
    }
}
```



### 安全



#### ACL



> 1: 为消费者分配授权策略组，2: 为 API 添加授权策略分组插件，3: 只有拥有 API 授权策略分组的消费者才可以调用该 API
>
> 4: 授权策略分组，建立在认证机制上，该策略生效的前提，API 至少要开启任意一个 auth 认证插件





**step1: 为服务开启 acl**

```shell
curl -X POST http://10.20.11.117:8001/services/77458fdb-deeb-4b66-8b42-ba749cdd9838/plugins \
    --data "name=acl"  \
    --data "config.whitelist=group1" \
    --data "config.whitelist=group2" \
    --data "config.hide_groups_header=true"
```



**step2: 为路由开启 acl**

```shell
curl -X POST http://10.20.11.117:8001/routes/d82735e1-b3a4-4ff5-afd7-908d7e79c07c/plugins \
    --data "name=acl"  \
    --data "config.whitelist=group1" \
    --data "config.whitelist=group2" \
    --data "config.hide_groups_header=true"
```



**step3: 消费者**

```shell
curl -X POST http://10.20.11.117:8001/consumers/andy/acls --data "group=group1"
```



**step4: 验证**

```shell
curl -X GET http://kong:8001/acls/andy/consumer
```

*检索与 acl 关联的消费者*



### 限流管理



#### Rate Limiting



> 限流在给定的几秒钟，几分钟，几小时，几天，几月或几年内，开发人员可以发出多少个 HTTP 请求。如果基础服务/路由没有身份验证层，则将使用**客户端 IP** 地址，如果已配置身份验证插件，使用消费者。



**step1: 为服务开启 acl**

```shell
curl -X POST http://10.20.11.117:8001/services/77458fdb-deeb-4b66-8b42-ba749cdd9838/plugins --data "name=rate-limiting"  --data "config.second=5" --data "config.hour=10000"
```



**step2: 为路由开启 acl**

```shell
curl -X POST http://10.20.11.117:8001/routes/d82735e1-b3a4-4ff5-afd7-908d7e79c07c/plugins --data "name=rate-limiting"  --data "config.second=5" --data "config.hour=10000"
```



**step3: 消费者**

```shell
curl -X POST http://10.20.11.117:8001/consumers/andy/plugins --data "name=rate-limiting" --data "config.second=5" --data "config.hour=10000"
```



> config.second: 表述每秒请求不能超过 5 次，参数可选
>
> config.hour: 表示每小时请求不能大于 10000，参数可选



**step4: 验证插件**



> 随意写一个脚本不停访问我们的服务（我们每次请求次数远超出五次），结果抛出如下所示：

```json
{"message":"API rate limit exceeded"}
{"message":"API rate limit exceeded"}
{"message":"API rate limit exceeded"}
{"message":"API rate limit exceeded"}
{"message":"API rate limit exceeded"}
{"message":"API rate limit exceeded"}
{"message":"API rate limit exceeded"}
```
