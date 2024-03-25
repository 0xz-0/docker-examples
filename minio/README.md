# MinIO

* 高性能对象存储系统

## Reference

* https://min.io/
  
  * https://min.io/docs/minio/container/operations/install-deploy-manage/deploy-minio-single-node-single-drive.html
  
* https://github.com/minio/minio
  
* https://hub.docker.com/r/minio/minio



## Getting started

1. 进入当前目录：```cd minio```
   
2. 复制并修改配置文件：```cp config_example.env config.env```
   * 修改管理界面的账号密码：`MINIO_ROOT_USER`和`MINIO_ROOT_PASSWORD`
   * PS：配置文件中的```MINIO_OPTS="--console-address :9001"```发现未生效，先注释。在容器启动时指定该参数值
   
3. 启动容器组
   ```docker-compose up -d --force-recreate```

4. 打开MinIO的[WebUI](http://localhost:9001)