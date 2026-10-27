## MySQL

* 官方镜像地址：https://hub.docker.com/_/mysql

### 5.7 版本

1. 进入当前目录：```cd mysql```

2. 先启动 - ```docker run```
   ```cmd
   ## cmd
   docker run -d --env MYSQL_ALLOW_EMPTY_PASSWORD=true --name local_mysql57 mysql:5.7.44
   ```

3. 复制容器内配置文件到本地，并删除对应容器
    ```cmd
    ## cmd
    mkdir .\5.7\data\conf
    docker cp local_mysql57:/etc/my.cnf  .\5.7\data\conf\
    docker rm --force local_mysql57 
    ```

4. 重新启动完整命令
   * 注：提前修改环境变量(如：```MYSQL_ROOT_PASSWORD=mysql```)
   * 默认用户为：```root```
    ```cmd
    ## cmd
    docker run -d ^
       --name local_mysql57 ^
       --restart always ^
       --publish 3306:3306 ^
       --env MYSQL_ROOT_PASSWORD=mysql ^
       --volume .\\5.7\\data\\log:/var/log/mysql ^
       --volume .\\5.7\\data\\data:/var/lib/mysql ^
       --volume .\\5.7\\data\\conf\\my.cnf:/etc/my.cnf ^
       mysql:5.7.44
    ```

### 8.0 版本

1. 进入当前目录：```cd mysql```
   
2. 先启动 - ```docker run```
   ```cmd
   %cmd%
   docker run -d --env MYSQL_ALLOW_EMPTY_PASSWORD=true --name local_mysql80 mysql:8.0.24
   ```
3. 复制容器内配置文件到本地，并删除对应容器
    ```cmd
    %cmd%
    mkdir .\8.0\data\conf
    docker cp local_mysql80:/etc/mysql/conf.d  .\8.0\data\conf\
    docker rm --force local_mysql80 
    ```

4. 重新启动完整命令
   * 注：提前修改环境变量(如：```MYSQL_ROOT_PASSWORD=mysql```)
   * 默认用户为：```root```
   ```cmd
   %cmd%
   docker run -d ^
      --name local_mysql80 ^
      --restart always ^
      --publish 3306:3306 ^
      --env MYSQL_ROOT_PASSWORD=mysql ^
      --volume .\\8.0\\data\\mysql-files:/var/lib/mysql-files ^
      --volume .\\8.0\\data\\conf:/etc/mysql/conf.d ^
      --volume .\\8.0\\data\\logs:/var/log/mysql ^
      --volume .\\8.0\\data\\data:/var/lib/mysql ^
      mysql:8.0.24
   ```
   ```
   ## bash
   docker run -d \
      --name local_mysql80 \
      --restart always \
      --publish 3306:3306 \
      --env MYSQL_ROOT_PASSWORD=mysql \
      --volume ./8.0/data/mysql-files:/var/lib/mysql-files \
      --volume ./8.0/data/conf:/etc/mysql/conf.d \
      --volume ./8.0/data/logs:/var/log/mysql \
      --volume ./8.0/data/data:/var/lib/mysql \
      mysql:8.0.24
   ```