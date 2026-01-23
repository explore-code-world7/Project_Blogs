在 **Ubuntu 18.04** 上安装 ROS，其对应的官方发行版是 **ROS Melodic Morenia**。它是ROS 1的最后一个长期支持版本，非常适合在Ubuntu 18.04上进行稳定开发。

### 📦 ROS Melodic 安装步骤

**1. 配置Ubuntu软件源**
确保你的“universe”、“restricted”、“multiverse”软件源已启用。这通常在安装Ubuntu时默认完成，但可以检查或启用：
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository restricted
sudo add-apt-repository multiverse
```

**2. 设置ROS软件源**
将ROS官方的软件源添加到你的系统，并添加其GPG密钥：
```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

**3. 安装ROS Melodic**
更新软件包列表并安装。对于大多数用户，推荐安装**完整桌面版**，它包含了ROS、RViz、rqt、机器人通用库等常用工具和功能。
```bash
sudo apt update
# 安装完整桌面版（推荐，包含GUI工具和模拟器）
sudo apt install ros-melodic-desktop-full -y

# 或者，安装基础版（仅核心ROS和构建工具）
# sudo apt install ros-melodic-ros-base -y
```

**4. 初始化 rosdep**
`rosdep` 是安装ROS包的依赖所必需的工具。
```bash
sudo rosdep init
rosdep update
```

**5. 设置环境变量**
为了方便使用ROS命令，需要将ROS环境变量添加到你的bash会话中。
```bash
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

**6. 安装构建工具**
为了能够编译和构建你自己的ROS工作空间，需要安装一些依赖。
```bash
sudo apt install python-rosinstall python-rosinstall-generator python-wstool build-essential python-catkin-tools -y
```

### 🧪 验证安装
安装完成后，可以通过以下简单命令测试ROS核心功能是否正常：

1.  **启动ROS核心（Master）**：在一个终端中运行：
    ```bash
    roscore
    ```
    看到类似 `started core service [/rosout]` 的日志输出，说明ROS核心启动成功。

2.  **运行一个简单节点**：保持`roscore`运行，**新开一个终端**，运行一个内置的“小乌龟”模拟节点：
    ```bash
    rosrun turtlesim turtlesim_node
    ```
    此时应该会弹出一个带有随机小乌龟的图形窗口。

3.  **控制小乌龟**：**再新开一个终端**，运行控制节点，然后你可以使用键盘方向键控制乌龟移动：
    ```bash
    rosrun turtlesim turtle_teleop_key
    ```

### 📂 创建你的工作空间（可选但推荐）
标准的ROS开发都是在“工作空间”中进行的。可以按以下步骤创建一个：
```bash
# 1. 创建并进入工作空间目录
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
# 2. 使用 catkin 构建系统初始化工作空间
catkin init
# 3. 构建整个工作空间
catkin build
# 4. 将工作空间的环境变量也加入bashrc，这样你编写的包才能被找到
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
