# Ros-noetic

## install

1. replace source package
   https://robotics.stackexchange.com/questions/99013/ros-cannot-install-noetic-due-to-unmet-dependencies-20-04
2. install
   https://wiki.ros.org/noetic/Installation/Ubuntu

# ROS2

## install

* solution = https://docs.ros.org/en/rolling/Installation/Ubuntu-Install-Debs.html
* remember to sudo update&upgrade with recommended updates

# Start an ros env & compile package
## 1. activate ros command package 
```bash
source /opt/ros/melodic/setup.bash   # 18.04
source /opt/ros/noetic/setup.bash   # 20.04
source /opt/ros/humble/setup.bash   # 22
```

# 2. initialize catkin workspace env
```bash
# 进入你的工作空间根目录
cd ~/catkin_ws
# 编译
catkin_make
# 初始化工作空间环境（将覆盖系统环境中同名的包）
source devel/setup.bash
```

# 3. 运行目标代码库的软件包
```bash
rosrun your_package your_node
roslaunch your_package your_launch_file.launch
```

# 4. 检查关键环境变量是否正确设置
```bash
# 检查ROS版本和环境
echo $ROS_DISTRO
# 检查ROS包路径，应该包含你的工作空间和系统路径
echo $ROS_PACKAGE_PATH
# 应该看到类似输出，且你的catkin_ws路径在前
# /home/yourname/catkin_ws/src:/opt/ros/melodic/share
```

## devel目录
1. 编译生成的所有可执行文件、库、脚本、配置文件都存放在这里，并按 ROS 标准结构组织：
```bash
devel/
├── lib/                    # 最重要的目录：所有编译成果
│   ├── pkg_name/          # 每个包的可执行文件
│   │   └── node_name      # 具体的ROS节点（可执行程序）
│   └── *.so               # 编译生成的共享库（C++）
├── include/               # C++头文件（通常链接到源码）
├── share/                 # 配置文件、launch文件、Python模块等
│   └── pkg_name/
│       ├── cmake/        # 包的CMake配置文件
│       ├── launch/       # launch启动文件
│       └── ...           
└── setup.bash            # 【最关键文件】环境激活脚本
```
2. devel/ 目录下的 setup.bash（或 setup.zsh、setup.sh）是连接你的编译成果与ROS系统的桥梁。
```bash
source ~/catkin_ws/devel/setup.bash
```
3. 实现“环境覆盖”机制
* 执行顺序决定优先级：
```bash
source /opt/ros/melodic/setup.bash   # 基础系统
source ~/catkin_ws_a/devel/setup.bash # 覆盖系统，A包优先
source ~/catkin_ws_b/devel/setup.bash # 覆盖A，B包优先（最后source的优先级最高）
```



