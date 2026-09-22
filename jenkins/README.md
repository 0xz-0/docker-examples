# Jenkins

* 开源持续集成/持续交付（CI/CD）服务

## Reference

* https://www.jenkins.io/

  * https://www.jenkins.io/doc/book/installing/docker/

* https://github.com/jenkinsci/docker

* https://hub.docker.com/r/jenkins/jenkins



## Getting started

1. 进入当前目录：```cd jenkins```

2. 启动容器组（首次会根据```Dockerfile```构建镜像，为官方镜像附加```docker``` CLI）
   ```bash
   # docker-compose.yaml
   docker-compose up -d --build


   # docker run（不含docker-cli，仅用官方镜像；如需在容器内操作宿主机Docker，请用docker-compose方式）
   ## windows-wsl2 cmd
   docker run -d ^
      --name local_jenkins ^
      --restart always ^
      --publish 8080:8080 ^
      --publish 50000:50000 ^
      --env TZ=Asia/Shanghai ^
      --env JAVA_OPTS="-Duser.timezone=Asia/Shanghai" ^
      --volume .\\data:/var/jenkins_home ^
      jenkins/jenkins:2.568.3-lts-jdk21
   ```

3. 获取初始管理员密码（任选其一）
   ```bash
   # 从容器日志中获取
   docker logs local_jenkins

   # 从挂载目录中获取
   cat data/secrets/initialAdminPassword
   ```

4. 打开Jenkins的[WebUI](http://localhost:8080)，输入初始管理员密码，选择“安装推荐的插件”，并创建管理员账号

## 说明

* `8080`：Web界面端口；`50000`：Agent（节点）连接端口，不使用分布式构建可去掉
* 所有数据（配置、插件、任务、构建记录）保存在`./data`目录（即容器内`/var/jenkins_home`），删除容器不会丢失数据
* 插件下载慢时，可在`Manage Jenkins -> Plugins -> Advanced settings`中修改`Update Site`为国内镜像源
* 本示例已挂载宿主机的`/var/run/docker.sock`，并通过```Dockerfile```在镜像内安装了`docker` CLI（官方静态二进制，已校验sha256），使Jenkins流水线可以直接`docker build` / `docker push`，也能操作宿主机上的其他容器
  * `docker-compose.yaml`中的`group_add: ["0"]`让`jenkins`用户能访问归属`root:root`的socket，避免把整个Jenkins进程跑成root
  * **安全提示**：挂载`docker.sock`等价于给Jenkins（以及任何能在其中执行Pipeline脚本的人）宿主机的完全控制权限（可通过它挂载宿主机根目录逃逸）。仅适合本机隔离的开发环境，不要把该端口暴露到公网或不受信任的网络
