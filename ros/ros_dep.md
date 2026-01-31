
```bash
sudo apt install python-pip
sudo pip install rosdep
sudo rosdep init
sudo rosdep update
```

# ros安装第三方依赖
```bash
rosdep install -y   --from-paths ./3rdparty   --ignore-src
```
