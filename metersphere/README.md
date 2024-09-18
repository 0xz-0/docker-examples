# MeterSphere

* 自动化测试平台
* 官方地址：
  * https://metersphere.io/index.html
  * https://github.com/metersphere/metersphere

## Getting started

* 安装
   ```cmd
   ## cmd
   docker run -d ^
      -p 8081:8081 ^
      --name=metersphere ^
      --volume .\\data:/opt/metersphere/data ^
      cr2.fit2cloud.com/metersphere/metersphere-ce-allinone
   ```
* 登录：http://localhost:8081
  * 用户名: admin
  * 密码: metersphere
  

* **目前安装报错**！！！