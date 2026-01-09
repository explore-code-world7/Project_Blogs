# 安装gcc并用soft link设置
```bash
# 11-gcc,g++
sudo apt install gcc-11 g++-11
# 然后设置默认版本
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 110
```

# /usr/local下目录的区别

| 目录                   | 内容      | 作用                                 |
| -------------------- | ------- | ---------------------------------- |
| `/usr/local/bin`     | 可执行程序   | 用户安装的软件的可执行文件（命令行程序）               |
| `/usr/local/lib`     | 库文件     | 编译好的 **静态库（.a）或动态库（.so）**，供链接器使用   |
| `/usr/local/include` | 头文件     | **C/C++ 的头文件**（.h/.hpp），供编译器在编译时引用 |
| `/usr/local/share`   | 架构无关的数据 | 文档、配置文件、pkgconfig、样例数据等，不依赖 CPU 架构 |


* include是头文件库，用于编译成.o文件
* lib是链接库，用于链接库文件.a成.so
* share是用于头文件库的CMake配置文件用于索引，或者存放架构无关数据

## /usr/local/share/eigen3/cmake

CMake 使用 find_package(Eigen3) 查找 Eigen3。

/usr/local/share/eigen3/cmake 里面通常有：

Eigen3Config.cmake

Eigen3ConfigVersion.cmake

这些文件告诉 CMake：

EEigen3 的 头文件路径（Eigen 是头文件库，没有 .so 或 .a）
版本信息

包含目录设置方式

举例：

```cpp
find_package(Eigen3 3.4 REQUIRED)
target_include_directories(my_target PRIVATE ${EIGEN3_INCLUDE_DIR})
```

CMake 就会读取 /usr/local/share/eigen3/cmake/Eigen3Config.cmake，自动找到头文件路径。
