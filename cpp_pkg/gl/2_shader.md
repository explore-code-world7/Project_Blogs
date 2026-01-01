着色=把3D 坐标转为屏幕上2D带颜色点;

着色器 用 GLSL(OpenGL Shading Language) 库编写;
```cpp
#a vertex shader
#version 330 core

layout(location = 0) in vec3 aPos;
layout(location = 1) in vec3 aColor;
layout(location = 2) in vec2 aTex;

layout(location = 0) out vec3 vColor;
layout(location = 1) out vec2 vTex;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    vColor = aColor;
    vTex   = aTex;
}
```
* Vertex Shader 的输出=(gl_Position+ layout out);
* Fragment Shader 的输出=(out + layout)

```cpp
// Fragment Shader
#version 330 core

layout(location = 0) in vec3 vColor;
layout(location = 1) in vec2 vTex;

out vec4 FragColor;

void main()
{
    FragColor = vec4(vColor, 1.0);
}

```
* fragment shader的输出 ✔️ 写入 GPU 显存中的 FBO / 默认 framebuffer


分为若干阶段: 
##
vertex shader:
1. transform 3D coordinates into different 3D coordinates (more on that later) ;
2. allows us to do some basic processing on the vertex attributes.

* vertex先发送到GPU上的VBO(vertex buffer objects)
### 为VBO分配 ID
```cpp
unsigned int VBO;
glGenBuffers(1, &VBO);  
```

### BufferBind: 任何buffer call作用于binded buffer
* VBO的buffer类型=GL_ARRAY_BUFFER
```cpp
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
```
### data>buffer
* GL_STATIC_DEAW = the graphics card to manage the given data

```cpp
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

## 
fragment shader
1. generate other shapes by emitting new vertices to form new (or other) primitive(s).
2. assembles all the vertices (or vertex if GL_POINTS is chosen) from the vertex (or geometry) shader that form one or more primitives in the primitive shape given;


## 
rasterization
1. maps the resulting primitive(s) to the corresponding pixels on the final screen, resulting in fragments for the fragment shader to use.
2. clipping is performed Before discards all fragments that are outside your view, increasing performance.

>  A fragment in OpenGL is all the data required for OpenGL to render a single pixel.


# set IO format = multiple vertex

```glsl
layout ( triangles ) in;
layout ( , max_vertices = 3 ) out;
```

* 设置输入为3个顶点
* triangle_strip是什么数据类型?

## 相应, I=array

```glsl
in vec4 position_world[];
in vec3 viewDirection_camera[];
```
## but O=vertex
```glsl
    normal_camera = normal_cam;
    normal_world = normal;
    xyz_world = position_world[0];
    xyz_camera = V * xyz_world;
    viewDirection_cam = viewDirection_camera[0];
    gl_PrimitiveID = gl_PrimitiveIDIn;
    EmitVertex();
    ...
    EndPrimitive();    
```

* submiited by Emit Vertext() and EndPrimitive();

# transform to pixel space: MVP
1. M: model space > world
2. V: world space > camera
3. P: camera space > pixel
$\in R^{4\times 4}$

# 
