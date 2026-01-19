# week1
1. 熟悉opengl,glfw,pangolin等点云图渲染库的基本用法，
• 用来渲染出视角图像，根据图像的位置采样表面点，进而计算点云图的sdf值;
2. 训练好camera输入的物体hold，成功率=60%，但蒸馏暂时失效;
3. 基于camera输入的balance训练稳定
4. 折腾硬盘很久艾
5. 折腾hf下载数据集艾
6. 从mesh数据集计算sdf因为硬盘壳失败

# week2
- [x] 1. 稳定训练camera object hold
• 对比已有的camera-based 训练框架, 寻找改进
• 找到bugs，可以训练，但还没训练好
- [x] 2. 如果成功，开始蒸馏walk with object
- [x] 3. 生成标准sdf值，训练deepsdf
• 调用trimesh/open3d 修复劣质点云，多进程计算采样点集到点云表面的距离，完成deepsdf的复现，可估计空间点到同类物体表面的距离及相关应用
- [x] 4. 尝试用diffusion来训练位姿估计
• 尝试失败，当前建模不对
- [x] 5. 下载别人的代码、配环境复现位姿估计
- [x] 下载若干6D位姿估计数据集
• BCOT: 若干物体的rgb图，和相应位姿；
• CAMERA25: 给定mesh, instance-level物体的rgb, depth, mask, NOCS坐标图;
• REAL275: 6 classes, (RGB, depth, mask, category label, pose, scale, NOCS coord)
• linemod: instance-level, (rgb, depth, mask, gt pose, cad model)
• YCBV: instance-level, (rgb, depth, mask, gt pose, cam_k, cad model))
- [x] 6. 完成slambook2.git的环境配置，编译其代码
• 是复现LVI-SLAM的基础，因为我不做slam，所以复现SLAM没什么用，
• 但加深对位姿估计算法的理解有点用

# week3
- [ ] 1. 稳定训练camera-based walk task;
- [ ] 通过调参，完成
- [ ] 2. 完成更新的robot网络架构代码
* 写好了基础的教师网络
- [ ] 3. 实现想好的pose estimate框架
- [ ] 4. 理解+复现3DGS
* 1.15-1.18连着熬夜打了4天游戏，非常抱歉

# week 4 
- [ ] 
