# 代码编译
```bash
sudo apt-get install freeglut3-dev
```
```bash
gcc filename.c -lGL -lGLU -lglut
```

* 安装glfw
```bash
sudo apt-get install libglfw3-dev
```

* 安装glm: 按照GLSL的数学规范设计的，C++头文件数学库;
```bash
sudo apt install libglm-dev
```

## 编译glad
* 添加头文件
```bash
add_library(glad glad/src/glad.c)
target_include_directories(glad  PUBLIC glad/include)
target_link_libraries(main glfw ${GL_LIBRARY} glad)
```

* 方法2:
```bash
cd glad/include
sudo mv glad/ /usr/local/include #将glad目录移动到/usr/local/include
sudo mv KHR/ /usr/local/include #将KHR目录移动到/usr/local/include
```
* 添加.c到工程
```bash
cmake_minimum_required(VERSION 2.8.1)

project(window)

find_package(glfw3 REQUIRED)
add_library(glad 3rdparty/glad/src/glad.c)

add_executable(window window.cpp)

target_link_libraries(window  glfw  glad)

```


# gl自带函数

glClear – 清除缓冲区。

glClearColor – 设置清除颜色。

glEnable / glDisable – 启用/禁用某些功能（如深度测试、混合等）。

glGenBuffers / glBindBuffer / glBufferData – 用于创建、绑定和设置缓冲区数据。

glGenTextures / glBindTexture / glTexImage2D – 用于创建和操作纹理。

glUseProgram – 激活着色器程序。

glDrawArrays / glDrawElements – 绘制图形。

glClearDepth – 设置深度缓冲区清除值。

glBlendFunc – 设置混合函数。

glLineWidth / glPointSize – 设置线宽和点大小。

glviewport -设置窗口的大小

# 创建窗口

```cpp
int main()
{
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    //glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
  
    return 0;
}
```

# 窗口尺寸变化时自动调用
```cpp
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```
# 更新窗口内容

```cpp
while(!glfwWindowShouldClose(window))
{
    glfwSwapBuffers(window);
    glfwPollEvents();    
}
```
* glfwPollEvents: checks if any events are triggered (like keyboard input or mouse movement events), updates the window state, and calls the corresponding functions (which we can register via callback methods)

# 清空释放的内存

```cpp
glfwTerminate();
return 0;
```

# 窗口事件的检查-glfwGetKey

```cpp
void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}
```
## 调用

```cpp
while (!glfwWindowShouldClose(window))
{
    processInput(window);

    glfwSwapBuffers(window);
    glfwPollEvents();
}  
```

# set清屏颜色
```cpp

glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);
```
