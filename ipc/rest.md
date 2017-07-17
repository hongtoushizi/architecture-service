# 同步请求 RESTFUL

## HTTP 消息总线

假设服务之间直接通讯。每个服务暴露一组REST API，外部的服务或者客户端通过REST API来调用。明显的，这种模型对于简单的微服务架构应用有效。但是随着服务数量的增加，它会慢慢变得复杂。这也是为什么在SOA里面要用ESB来避免杂乱的点对点的连接。让我们试着总结一下点对点模式的弊端。

- 非功能需求，比如用户认证、流控、监控等必须在每个微服务里实现；
- 由于通用功能的重复，每个微服务的实现变得复杂；
- 在服务和客户端之间没有通讯控制（甚至对于监控、跟踪、过滤等都没有）；
- 对于大的微服务实现来说直接的通讯形式通常被认为是[反模式](http://www.infoq.com/articles/seven-uservices-antipatterns)。


因此， 在复杂的微服务应用场景下，不要使用点对点直连或者中央的ESB，我们可以使用一个**轻量级的中央消息总线**给所有微服务提供一个抽象层，而且可以用来实现各种非功能的能力。这种风格也叫做API Gateway风格。

![](http://img.dockerinfo.net/2016/07/20160718114652.jpg)

#### 功能点

1. 授权认证和访问控制
1. 流控、熔断
1. 日志、跟踪

#### 项目地址 
[httpgateway](https://github.com/ifintech/httpgateway)


## API设计规范

> [规范地址](https://ifentech.gitbooks.io/rdbuild/content/rule/api.html)

#### header头约定

> x-source 来源应用  
> x-time 时间戳  
> x-m 加密串  
> x-field 参与校验的字段  
> x-rid 请求唯一标识  

## 实施

#### 与服务注册发现结合
![](/images/httpgateway.png)

#### consul-template 安装配置

```shell
wget https://releases.hashicorp.com/consul-template/0.18.2/consul-template_0.18.2_linux_amd64.zip
unzip consul-template_0.18.2_linux_amd64.zip
cp consul-template /bin/
```

vim /etc/nginx/consul-template/upstream.tpl

```shell
{{range services}} {{$name := .Name}} {{$service := service .Name}}
upstream {{$name}} {
  {{range $service}}server {{.Address}}:{{.Port}} weight=1;
  {{end}}
} {{end}}
```

更新upstream

```shell
consul-template -consul-addr 10.0.4.13:8500 -template "/etc/nginx/consul-template/upstream.tpl:/etc/nginx/upstream.conf:/usr/local/openresty/nginx/sbin/nginx -s reload" -once
```
自启动 vim rc.local

```shell
consul-template -consul-addr 10.0.4.13:8500 -template "/etc/nginx/consul-template/upstream.tpl:/etc/nginx/upstream.conf" -once
/usr/local/openresty/nginx/sbin/nginx
consul-template -consul-addr 10.0.4.13:8500 -template "/etc/nginx/consul-template/upstream.tpl:/etc/nginx/upstream.conf:/usr/local/openresty/nginx/sbin/nginx -s reload" >> /var/log/consul-template 2>&1 &
```

