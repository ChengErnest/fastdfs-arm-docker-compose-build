# ARM架构DockerCompose构建单机FastDFS

1. 安装`Docker`与`Docker Compose`

2. `environment`：必要安装包与配置文件

   ![](https://github.com/nicollcheng/fastdfs-arm-docker-compose-build/blob/main/imgs/environment.png)

3. 编写Docker Compose构建启动文件`docker-compose.yml`

   ```yaml
   version: '3.1'
   services:
     fastdfs:
       build: environment
       restart: always
       container_name: fastdfs
       volumes:
         - ./storage:/fastdfs/storage
       network_mode: host
       environment:
         FASTDFS_IPADDR: trackerIP:trackerPort
   ```

4. 执行`docker-compose up -d`构建并启动

5. 启动成功后，进入容器中

   ```bash
   docker exec -it fastdfs bash
   ```

6. 验证

   - 测试文件上传

     ```bash
     /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/fastdfs-5.11/INSTALL
     ```

   - 服务器反馈上传地址

     ```bash
     group1/M00/00/00/wKhLi1oHVMCAT2vrAAAeSwu9TgM3976771
     ```

   - 访问地址

     ```bash
     http://192.168.0.59:8888/group1/M00/00/00/wKhLi1oHVMCAT2vrAAAeSwu9TgM3976771
     ```

   - 查看storage状态

     ```bash
     fdfs_monitor /etc/fdfs/storage.conf
     ```

     ![](https://github.com/nicollcheng/fastdfs-arm-docker-compose-build/blob/main/imgs/storage-info.png)

     确保该状态为`ACTIVE`状态

   - 查看client状态

     ```bash
     fdfs_monitor /etc/fdfs/client.conf
     ```

     ![](https://github.com/nicollcheng/fastdfs-arm-docker-compose-build/blob/main/imgs/client-info.png)

     确保该状态为`ACTIVE`状态

   > FDFS_STORAGE_STATUS：INIT      :初始化，尚未得到同步已有数据的源服务器
   > FDFS_STORAGE_STATUS：WAIT_SYNC :等待同步，已得到同步已有数据的源服务器
   > FDFS_STORAGE_STATUS：SYNCING   :同步中
   > FDFS_STORAGE_STATUS：DELETED   :已删除，该服务器从本组中摘除
   > FDFS_STORAGE_STATUS：OFFLINE   :离线
   > FDFS_STORAGE_STATUS：ONLINE    :在线，尚不能提供服务
   > FDFS_STORAGE_STATUS：ACTIVE    :在线，可以提供服务
   >
   >
   > 若这两个服务的状态不为`ACTIVE`状态时，FastDFS不会提供服务，可以尝试以下操作：
   >
   > ```bash
   > # 从集群中删除
   > fdfs_monitor /etc/fdfs/client.conf delete group1 192.168.0.59
   > # 在服务器中，删除数据文件夹
   > rm -rf /fastdfs/storage/data
   > # 重启节点
   > fdfs_storaged /etc/fdfs/storage.conf
   > # 重新查状态
   > fdfs_monitor /etc/fdfs/client.conf
   > ```

