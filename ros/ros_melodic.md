在 Ubuntu 18.04 上安装 ROS 和 RViz 的完整指南：

## 1. 系统更新和准备

```bash
# 更新系统
sudo apt update
sudo apt upgrade -y

# 设置时区（可选）
sudo timedatectl set-timezone Asia/Shanghai

# 安装必要工具
sudo apt install -y curl wget git build-essential
```

## 2. 设置 ROS 软件源

```bash
# 添加ROS软件源
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

# 添加ROS密钥
sudo apt install -y curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

## 3. 安装 ROS Melodic（Ubuntu 18.04 对应版本）

```bash
# 更新软件包索引
sudo apt update

# 完整桌面版安装（包括RViz、Gazebo、rqt等）
sudo apt install -y ros-melodic-desktop-full

# 或者最小安装
# sudo apt install -y ros-melodic-ros-base
```

## 4. 初始化 rosdep

```bash
# 初始化rosdep
sudo rosdep init
rosdep update
```

如果 `sudo rosdep init` 失败（网络问题），可以尝试：

```bash
# 方法1：使用国内镜像
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'

# 方法2：手动设置
sudo mkdir -p /etc/ros/rosdep/sources.list.d/
sudo sh -c 'echo "yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/osx-homebrew.yaml osx-homebrew" > /etc/ros/rosdep/sources.list.d/20-default.list'
sudo sh -c 'echo "yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml" >> /etc/ros/rosdep/sources.list.d/20-default.list'
sudo sh -c 'echo "yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml" >> /etc/ros/rosdep/sources.list.d/20-default.list'
sudo sh -c 'echo "yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/ruby.yaml" >> /etc/ros/rosdep/sources.list.d/20-default.list'
sudo sh -c 'echo "yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list" >> /etc/ros/rosdep/sources.list.d/20-default.list'
```

## 5. 设置环境变量

```bash
# 将ROS环境变量添加到bashrc
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc

# 验证环境变量
echo $ROS_PACKAGE_PATH
```

## 6. 安装构建工具和依赖

```bash
# 安装rosinstall和其他构建工具
sudo apt install -y python-rosinstall python-rosinstall-generator python-wstool build-essential

# 安装catkin构建工具
sudo apt install -y python-catkin-tools

# 安装常用ROS包
sudo apt install -y ros-melodic-rviz ros-melodic-rqt ros-melodic-rqt-common-plugins
```

## 7. 测试 ROS 和 RViz 安装

```bash
# 测试ROS核心
roscore &
# 如果正常，会看到master启动信息

# 新终端测试RViz
rosrun rviz rviz
# 或直接运行
rviz
```

## 8. 创建 ROS 工作空间（可选但推荐）

```bash
# 创建工作空间目录
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws

# 初始化工作空间
catkin_make

# 设置工作空间环境
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc

# 验证工作空间
echo $ROS_PACKAGE_PATH
```

## 9. 安装常用工具和包

```bash
# 安装ROS导航包
sudo apt install -y ros-melodic-navigation

# 安装SLAM工具
sudo apt install -y ros-melodic-slam-gmapping ros-melodic-cartographer

# 安装TF工具
sudo apt install -y ros-melodic-tf-tools ros-melodic-tf2-*

# 安装可视化工具
sudo apt install -y ros-melodic-plotjuggler ros-melodic-rosbridge-server

# 安装Gazebo（如果未安装）
sudo apt install -y ros-melodic-gazebo-ros-pkgs ros-melodic-gazebo-ros-control
```

## 10. 故障排除

### 常见问题1：`rosdep update` 失败
```bash
# 使用国内镜像
export ROS_DISTRO=melodic
sudo sh -c 'echo "yaml https://gitee.com/zhao-xuzuo/rosdistro/raw/master/rosdep/sources.list.d/20-default.list" > /etc/ros/rosdep/sources.list.d/20-default.list'
sudo sh -c 'echo "yaml https://gitee.com/zhao-xuzuo/rosdistro/raw/master/rosdep/python.yaml" > /etc/ros/rosdep/sources.list.d/21-python.list'
sudo sh -c 'echo "yaml https://gitee.com/zhao-xuzuo/rosdistro/raw/master/rosdep/base.yaml" > /etc/ros/rosdep/sources.list.d/22-base.list'
rosdep update
```

### 常见问题2：缺少依赖
```bash
# 自动修复依赖
rosdep install --from-paths src --ignore-src -r -y
```

### 常见问题3：RViz 无法启动
```bash
# 安装OGRE依赖
sudo apt install -y libogre-1.9-dev libogre-1.9.0v5

# 安装OpenGL支持
sudo apt install -y mesa-utils freeglut3-dev

# 测试OpenGL
glxinfo | grep "OpenGL version"
```

## 11. 快速验证脚本

创建测试脚本 `test_ros.sh`：

```bash
#!/bin/bash
echo "=== ROS安装测试 ==="

# 测试roscore
echo "1. 启动roscore..."
roscore &
sleep 3
ROSCORE_PID=$!

# 测试RViz
echo "2. 启动RViz..."
rosrun rviz rviz -d /opt/ros/melodic/share/rviz/default.rviz &
sleep 5
RVIZ_PID=$!

# 测试ROS节点
echo "3. 运行测试节点..."
rosrun roscpp_tutorials talker &
sleep 2
rosrun roscpp_tutorials listener &
sleep 3

echo "4. 清理进程..."
kill $ROSCORE_PID $RVIZ_PID
echo "=== 测试完成 ==="
```

## 12. 卸载 ROS（如果需要）

```bash
# 卸载ROS
sudo apt remove --purge ros-melodic-*
sudo apt autoremove

# 删除环境变量
sed -i '/source \/opt\/ros\/melodic\/setup.bash/d' ~/.bashrc
sed -i '/source ~\/catkin_ws\/devel\/setup.bash/d' ~/.bashrc
source ~/.bashrc
```

## 安装完成后的建议

1. **学习资源**：
   ```bash
   # 安装教程包
   sudo apt install -y ros-melodic-ros-tutorials
   # 运行小乌龟示例
   roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
   ```

2. **配置优化**：
   ```bash
   # 提高编译速度
   echo "export MAKEFLAGS='-j$(nproc)'" >> ~/.bashrc
   
   # 设置Catkin工具别名
   echo "alias cm='catkin_make'" >> ~/.bashrc
   echo "alias cb='catkin build'" >> ~/.bashrc
   source ~/.bashrc
   ```

3. **安装最新 RViz 插件**（可选）：
   ```bash
   # 安装RViz插件
   sudo apt install -y ros-melodic-rviz-imu-plugin \
                      ros-melodic-jsk-rviz-plugins \
                      ros-melodic-hector-slam
   ```

按照以上步骤，你应该能在 Ubuntu 18.04 上成功安装 ROS Melodic 和 RViz。安装完成后，可以通过运行 `roscore` 和 `rviz` 来验证安装是否成功。
