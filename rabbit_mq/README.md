# RabbitMQ

* 参考:
  * https://www.cnblogs.com/shanfeng1000/p/16261011.html
  
## Getting started

* 启动
```shell
docker run -d --name rabbit -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456 rabbitmq:management
```
* 说明
  * 端口5672，是AMPQ协议端口
  * 端口15672，是后台管理页面端口
  * RABBITMQ_DEFAULT_USER：设置后台管理登录账号
  * RABBITMQ_DEFAULT_PASS：设置后台管理登录账号的密码

* 访问地址：[http://localhost:15672](http://localhost:15672)