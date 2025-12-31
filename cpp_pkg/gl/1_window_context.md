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
