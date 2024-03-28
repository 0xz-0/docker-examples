# Redis

## Reference

* https://redis.io/
    
* https://github.com/redis/redis
  
* https://hub.docker.com/_/redis



## Getting started

1. 进入当前目录：```cd redis```
   
2. 复制并修改配置文件：
   ```bash
   cp redis_example.conf redis.conf
   ```
   * 常用参数：
     * **requirepass**: 密码。与```aclfile```选项和```ACL LOAD```命令不兼容，它们会导致```requirepass```被忽略。
  
     * **bind**：监听。默认情况下是只能主机地址环纹。
   
3. 启动
   ```bash
   # docker-compose.yaml
   docker-compose up -d


   # docker run
   ## Liunx bash
   docker run -d \
      --name local_redis \
      --restart always \
      --publish 6379:6379 \
      --volume ./data:/data \
      --volume ./redis.conf:/etc/redis/redis.conf
      redis:7.2.4 \
      redis.conf

   ## windows-wsl2 cmd
   docker run -d ^
      --name local_redis ^
      --restart always ^
      --publish 6379:6379 ^
      --volume .\\redis.conf:/etc/redis/redis.conf ^
      --volume .\\data:/data ^
      redis:7.2.4
   ```
