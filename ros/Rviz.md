# wsl2
* window1
```bash
roscore
```
* window2
```bash
rviz
```

<img width="1400" height="1333" alt="image" src="https://github.com/user-attachments/assets/bedb433c-41c0-4350-b46a-6e467a77a380" />


# docker启用GUI支持
方案三：为 Docker 容器启用 GUI 支持（需要额外配置）
对于 Windows Docker Desktop：

需要安装 X11 服务器并配置 Docker 容器：

    安装 X11 服务器（如 VcXsrv）：

        下载地址：https://sourceforge.net/projects/vcxsrv/

        安装后启动 XLaunch，选择 "Disable access control"

    重新运行容器时添加显示参数：
    powershell

    # 停止并删除当前容器
    docker stop fd05d0c2a87a
    docker rm fd05d0c2a87a

    # 重新运行容器，添加显示参数
    docker run -it `
      --env="DISPLAY=host.docker.internal:0.0" `
      --env="QT_X11_NO_MITSHM=1" `
      -v /tmp/.X11-unix:/tmp/.X11-unix `
      -v C:\Users\firefox\catkin_ws:/home/catkin_ws `
      ros:melodic `
      /bin/bash

    在容器内安装必要的 GUI 依赖：
    bash

    apt-get update
    apt-get install -y x11-apps mesa-utils
    # 测试显示
    xeyes  # 应该能看到眼睛图案

对于 Linux 主机：
bash

docker run -it \
  --env="DISPLAY" \
  --env="QT_X11_NO_MITSHM=1" \
  --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
  -v /path/to/workspace:/home/catkin_ws \
  ros:melodic


# usage
这里推荐一些适合学习 RViz 点云可视化的简单 ROS 项目，从易到难排列：

## 1. **基础点云可视化项目**

### a) ROS Tutorials - 官方教程
```bash
# 安装ROS教程包
sudo apt install ros-melodic-ros-tutorials

# 运行turtlesim（不是点云但简单）
rosrun turtlesim turtlesim_node

# 点云相关教程
sudo apt install ros-melodic-pcl-ros
```
**特点**: 官方维护，文档完整，最基础

### b) PCL 基础示例
```bash
# 安装PCL和ROS接口
sudo apt install ros-melodic-pcl-ros ros-melodic-pcl-conversions

# 克隆简单示例
cd ~/catkin_ws/src
git clone https://github.com/strawlab/pcl_ros_tutorial.git
```
**特点**: 学习 PCL 库和 ROS 的结合使用

## 2. **仿真环境项目**

### a) TurtleBot3 仿真
```bash
# 安装TurtleBot3
sudo apt install ros-melodic-turtlebot3 ros-melodic-turtlebot3-simulations

# 运行Gazebo仿真
export TURTLEBOT3_MODEL=waffle_pi
roslaunch turtlebot3_gazebo turtlebot3_world.launch

# 启动RViz
roslaunch turtlebot3_gazebo turtlebot3_gazebo_rviz.launch
```
**特点**: 完整的移动机器人仿真，有激光雷达和深度相机点云

### b) Husky 仿真
```bash
# 安装Husky仿真包
sudo apt install ros-melodic-husky-simulator

# 运行仿真
roslaunch husky_gazebo husky_empty_world.launch

# 启动RViz
roslaunch husky_viz view_robot.launch
```
**特点**: 户外机器人，有多种传感器点云

## 3. **简单点云生成项目**

### a) 基础点云发布器
创建一个最简单的点云发布节点：

```bash
# 创建新包
cd ~/catkin_ws/src
catkin_create_pkg simple_pointcloud std_msgs rospy roscpp sensor_msgs pcl_ros

# 创建节点代码
cat > src/simple_pointcloud_publisher.cpp << 'EOF'
#include <ros/ros.h>
#include <pcl/point_cloud.h>
#include <pcl/point_types.h>
#include <pcl_conversions/pcl_conversions.h>
#include <sensor_msgs/PointCloud2.h>

int main(int argc, char** argv)
{
    ros::init(argc, argv, "simple_pointcloud_publisher");
    ros::NodeHandle nh;
    
    ros::Publisher pub = nh.advertise<sensor_msgs::PointCloud2>("my_pointcloud", 1);
    ros::Rate rate(1);
    
    while (ros::ok()) {
        pcl::PointCloud<pcl::PointXYZ> cloud;
        
        // 创建一个简单的立方体点云
        for (float x = -1.0; x <= 1.0; x += 0.1) {
            for (float y = -1.0; y <= 1.0; y += 0.1) {
                for (float z = -1.0; z <= 1.0; z += 0.1) {
                    cloud.push_back(pcl::PointXYZ(x, y, z));
                }
            }
        }
        
        sensor_msgs::PointCloud2 output;
        pcl::toROSMsg(cloud, output);
        output.header.frame_id = "map";
        output.header.stamp = ros::Time::now();
        
        pub.publish(output);
        rate.sleep();
    }
    
    return 0;
}
EOF
```

### b) 2D激光转3D点云
```bash
cd ~/catkin_ws/src
git clone https://github.com/ros-perception/pointcloud_to_laserscan.git
```
**特点**: 学习2D到3D的转换，理解不同传感器的数据格式

## 4. **数据集可视化项目**

### a) KITTI 数据集可视化
```bash
cd ~/catkin_ws/src
git clone https://github.com/yanii/kitti-pcl

# 或使用更完整的版本
git clone https://github.com/koide3/ndt_omp
```
**特点**: 真实数据，大规模点云，适合学习处理实际数据

### b) TUM RGB-D 数据集
```bash
git clone https://github.com/tum-vision/dvo_slam
```
**特点**: 彩色点云，RGB-D数据，适合学习彩色点云可视化

## 5. **简单SLAM项目**

### a) hector_slam
```bash
# 安装Hector SLAM
sudo apt install ros-melodic-hector-slam

# 运行示例（需要激光数据）
roslaunch hector_slam_launch tutorial.launch
```
**特点**: 基于激光的2D/3D SLAM，点云地图构建简单直观

### b) Cartographer
```bash
# 安装Cartographer
sudo apt install ros-melodic-cartographer-ros

# 运行2D建图示例
roslaunch cartographer_ros demo_backpack_2d.launch
```
**特点**: Google开源的SLAM，可视化效果优秀，文档完整

## 6. **传感器数据可视化项目**

### a) 深度相机示例（RealSense/ Kinect）
```bash
# 安装RealSense ROS包
sudo apt install ros-melodic-realsense2-camera

# 运行（需要有RealSense相机）
roslaunch realsense2_camera rs_camera.launch

# 在RViz中查看点云
rosrun rviz rviz
# 添加 PointCloud2，话题选择 /camera/depth/color/points
```

### b) Velodyne 激光雷达
```bash
# 安装Velodyne仿真
sudo apt install ros-melodic-velodyne-simulator

# 运行仿真
roslaunch velodyne_gazebo_plugins example.launch
```
**特点**: 机械式激光雷达，点云模式典型

## 7. **学习项目推荐（Github）**

### a) **beginner_tutorials** - 最适合入门
```bash
git clone https://github.com/ros/ros_tutorials.git
# 查看 beginner_tutorials
```

### b) **rosbook** - 配套书籍的代码
```bash
git clone https://github.com/rosbook/effective_robotics_programming_with_ros.git
```

### c) **Autoware 基础可视化**
```bash
git clone https://github.com/CPFL/Autoware.git
# 只运行可视化部分
roslaunch runtime_manager runtime_manager.launch
```
**特点**: 自动驾驶系统，点云可视化非常完整

## 8. **实践项目建议**

按照以下顺序学习：

### **第1周：基础**
```bash
# 1. 学习发布简单点云
# 2. 在RViz中显示
# 3. 调整颜色、大小、坐标系
```

### **第2周：传感器数据**
```bash
# 1. 仿真激光雷达数据
# 2. 仿真深度相机数据
# 3. 真实数据（如果有设备）
```

### **第3周：数据处理**
```bash
# 1. 点云滤波
# 2. 点云配准
# 3. 点云分割
```

### **第4周：实际应用**
```bash
# 1. 简单SLAM建图
# 2. 物体检测
# 3. 自主导航
```

## 9. **快速开始的最小项目**

创建一个最简单的点云可视化项目：

```bash
# 1. 创建ROS工作空间
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin_make
source devel/setup.bash

# 2. 创建示例包
cd ~/catkin_ws/src
catkin_create_pkg learn_pointcloud roscpp std_msgs sensor_msgs pcl_ros

# 3. 下载示例代码
cd learn_pointcloud
wget https://raw.githubusercontent.com/ros/common_tutorials/kinetic-devel/roscpp_tutorials/listener_class/listener_class.cpp
# 修改为点云发布器

# 4. 编译运行
cd ~/catkin_ws
catkin_make

# 5. 运行节点
rosrun learn_pointcloud simple_pointcloud_publisher
```

## 10. **在线学习资源**

- **ROS Wiki 点云教程**: http://wiki.ros.org/pcl_ros/Tutorials
- **PCL 官方教程**: http://pointclouds.org/documentation/tutorials/
- **YouTube 频道推荐**:
  - ROS Conferences
  - The Construct (有免费课程)
  - Chris Wang的ROS教程（中文）

## **推荐新手路线**：

1. **第一天**: 安装 ROS 和 RViz，运行 turtlesim
2. **第二、三天**: 学习发布自定义点云，调整 RViz 显示
3. **第四、五天**: 运行仿真（TurtleBot3），查看激光和深度点云
4. **第六、七天**: 处理真实数据集（KITTI 或 TUM）
5. **第二周**: 实现简单功能（滤波、配准、建图）

**最简单的起点**: 从 `TurtleBot3` 仿真开始，它有完整的传感器配置和 RViz 显示，文档详细，社区支持好。

**最实用起点**: 从 `hector_slam` 开始，可以立即看到建图效果，成就感强。

**最适合学习**: 从 `PCL 基础示例` 开始，理解点云数据结构，再逐步扩展到应用。
