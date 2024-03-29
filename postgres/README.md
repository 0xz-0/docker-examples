# Postgres

## Reference

* https://www.postgresql.org/
  
  * GIS：  https://postgis.net/
  
* https://registry.hub.docker.com/_/postgres/
  
  * https://registry.hub.docker.com/r/postgis/postgis
  
    * https://github.com/postgis/docker-postgis

## Getting started

1. 进入当前目录：```cd postgres```

2. 启动 - ```docker run```
   * 注：提前修改环境变量

   ```bash
   ## bash
   docker run -d \
      --name local_pg14 \
      --restart always \
      --publish 5432:5432 \
      --env POSTGRES_USER=postgres \
      --env POSTGRES_PASSWORD=postgres \
      --env PGDATA=/var/lib/postgresql/data/pgdata \
      --volume ./data:/var/lib/postgresql/data \
      postgis/postgis:14-3.4
   ```
   ```cmd
   ## cmd
   docker run -d ^
      --name local_pg14 ^
      --restart always ^
      --publish 5432:5432 ^
      --env POSTGRES_USER=postgres ^
      --env POSTGRES_PASSWORD=postgres ^
      --env PGDATA=/var/lib/postgresql/data/pgdata ^
      --volume .\\data:/var/lib/postgresql/data ^
      postgis/postgis:14-3.4
   ```
   ```powershell
   ## powershell
   docker run -d `
       --name local_pg14 `
       --restart always `
       --publish 5432:5432 `
       --env POSTGRES_USER=postgres `
       --env POSTGRES_PASSWORD=postgres `
       --env PGDATA=/var/lib/postgresql/data/pgdata `
       --volume .\\data:/var/lib/postgresql/data `
       postgis/postgis:14-3.4
    ```