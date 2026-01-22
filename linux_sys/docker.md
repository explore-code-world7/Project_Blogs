# 1. 初始配置
* 拉取image
```bash
docker pull tyoung96/lvi_sam:1.0
```
* 创建image的container
```bash
# 正确方式
docker run -it --name hello_sam tyoung96/lvi_sam:1.0 /bin/bash

# 创建完后删除
docker run -it --rm tyoung96/lvi_sam:1.0 /bin/bash
```

# 2. 进入创建好的container
```bash
docker exec -it <container_id> /bin/bash
```


# 3. 传输文件
```bash
# 1. 复制单个文件到容器
docker cp C:\Users\firefox\data.txt my_container:/home/

# 2. 复制整个目录到容器（自动创建目录）
docker cp C:\Users\firefox\project my_container:/home/

# 3. 从容器复制文件到主机
docker cp my_container:/var/log/app.log C:\Users\firefox\logs\

# 4. 复制时保持文件权限
docker cp --archive C:\config\ my_container:/etc/
```

# ?修改docker的swap space size



# docker不自带图形界面
🔄 WSL2 原生 vs Docker 容器：图形支持对比
场景	图形支持机制	配置要求
WSL2 原生 Ubuntu	自动通过 DISPLAY=:0 连接到 Windows 显示	几乎无需配置，安装 GUI 应用即可运行
Docker 容器	隔离环境，默认无显示连接	必须手动配置环境变量和卷挂载

关键区别：Docker 容器是完全隔离的沙箱环境，它不会自动继承 WSL2 的显示设置。



