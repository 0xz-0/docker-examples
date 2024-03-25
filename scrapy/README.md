# scrapyd & scrapydweb

基于docker，部署爬虫（scrapy）运行服务（scrapyd）和可视化界面（scrapydweb）

## Source

* [scrapyd](https://github.com/scrapy/scrapyd)

* [scrapydweb](https://github.com/my8100/scrapydweb)

## Getting started

1. 进入当前目录：```cd scrapy```

2. 复制并修改**scrapyd**的配置文件：```cp scrapyd/scrapyd_example.conf scrapyd/scrapyd.conf```
    * 开启用户密码登录
      * 修改参数`username`和`password`
    * 在[scrapydweb/requirements.txt](/scrapy/scrapydweb/requirements.txt)增加未来爬虫所需库的依赖

3. 复制并修改**scraydweb**的配置文件：```cp scrapydweb/scrapydweb_settings_v10_example.py  scrapydweb/scrapydweb_settings_v10.py```
     * `__dc_gateway`，等同于`docker-compose.yaml`中`networks`的`gateway`
     * 开启WEB使用的密码登录
       * 修改参数`USERNAME`和`PASSWORD`
     * 开启数据库连接
       * 在[scrapyd/requirements.txt](/scrapy/scrapyd/requirements.txt)中增加对应依赖
       * 修改参数`DATABASE_URL`

4. 启动容器组
   ```docker-compose up -d --build --force-recreate```

5. 打开[scrapydweb](http://localhost:5000)