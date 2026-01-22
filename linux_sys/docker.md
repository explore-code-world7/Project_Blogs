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

## run 命令参数详解
🖥️ 命令参数详解
参数	作用	详细说明
docker run -it	交互式运行容器	-i 保持标准输入打开，-t 分配伪终端，让你可以像在真实终端中一样与容器交互
--env="DISPLAY=host.docker.internal:0.0"	设置显示环境变量（最关键）	告诉容器内的图形程序：
1. host.docker.internal 是 Docker 为 Windows 主机保留的特殊域名
2. :0.0 表示使用主机的第一个显示器
--env="QT_X11_NO_MITSHM=1"	Qt 图形库兼容性设置	防止 Qt 程序（如 RViz）使用共享内存时出错
-v /tmp/.X11-unix:/tmp/.X11-unix	挂载 X11 套接字目录	让容器能访问主机的 X11 显示服务（虽然 Windows 没有原生的 /tmp/.X11-unix，但 Docker Desktop 会处理）
-v C:\Users\firefox\catkin_ws:/home/catkin_ws	挂载工作目录	将 Windows 上的代码目录映射到容器内，实现文件同步
ros:melodic	使用的 Docker 镜像	ROS Melodic 版本的官方基础镜像
/bin/bash	启动命令	容器启动后自动进入 Bash shell


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



