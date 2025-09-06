# C++

## clone source code

```bash
git clone https://github.com/catchorg/Catch2.git
cd Catch2
```

## compile files

```bash
mkdir build
cd build
cmake ..
make
sudo make install
```

## resolution

在使用 CMake 构建项目时，`cmake ..`、`make` 和 `sudo make install` 是三个重要的命令，它们的作用分别如下：

### 1. `cmake ..`

- **作用**：生成构建系统文件。
- **解释**：当您在 `build` 目录中运行 `cmake ..` 时，CMake 会读取上一级目录（通常是源代码目录）中的 `CMakeLists.txt` 文件，并根据其中的指令生成适合您系统的构建文件（如 Makefile 或 Ninja 文件）。这些构建文件将用于后续的编译过程。
- **示例**：如果您在 `Catch2` 的 `build` 目录中运行 `cmake ..`，CMake 会配置项目并准备好构建所需的文件。

### 2. `make`

- **作用**：编译源代码。
- **解释**：在运行 `make` 时，Make 工具会根据 `cmake` 生成的 Makefile 文件来编译项目。它会查找源代码文件，编译它们，并生成可执行文件或库。`make` 会自动处理依赖关系，只重新编译那些自上次构建以来发生变化的文件。
- **示例**：在 `build` 目录中运行 `make`，将会编译 Catch2 的源代码并生成相应的库文件。

### 3. `sudo make install`

- **作用**：将编译好的文件安装到系统目录。
- **解释**：运行 `make install` 会将编译生成的可执行文件、库文件和头文件复制到系统的标准位置（如 `/usr/local/bin`、`/usr/local/lib` 和 `/usr/local/include`）。使用 `sudo` 是因为这些目录通常需要管理员权限才能写入。
- **示例**：在 `build` 目录中运行 `sudo make install`，将会把 Catch2 安装到系统中，使其可以在其他项目中使用。

### 总结

- `cmake ..`：生成构建系统文件（Makefile）。
- `make`：编译源代码，生成可执行文件或库。
- `sudo make install`：将编译好的文件安装到系统目录，以便全局使用。

通过这些步骤，您可以成功构建和安装 CMake 项目。如果您有其他问题或需要进一步的帮助，请随时问我！

# torch.utils.cpp_extension

* link：[Custom C++ and CUDA Extensions — PyTorch Tutorials 2.8.0+cu128 documentation](https://docs.pytorch.org/tutorials/advanced/cpp_extension.html)

```python
from setuptools import setup, Extension
from torch.utils import cpp_extension

setup(name='lltm_cpp',
      ext_modules=[cpp_extension.CppExtension('lltm_cpp', ['lltm.cpp'])],
      cmdclass={'build_ext': cpp_extension.BuildExtension})
```

* name='lltm_cpp'：指定包/分发名称为 lltm_cpp

* ext_modules=[cpp_extension.CppExtension('lltm_cpp', ['lltm.cpp'])]
  
  - 使用 torch.utils.cpp_extension.CppExtension 声明一个需要编译的 C++ 扩展模块。
  - 第一个参数 'lltm_cpp' 是扩展模块的导入名称（即编译后可通过 import lltm_cpp 导入）。
  - 第二个参数 ['lltm.cpp'] 是该扩展的源文件列表；可以包含多个 .cpp/.cu 文件。
  - CppExtension 会为编译设置合适的 include/link 标志以支持与 PyTorch 的 C++/CUDA API 配合（不自动包含 CUDA 支持，CUDA 文件需用 CUDAExtension）。

* cmdclass={'build_ext': cpp_extension.BuildExtension}
  
  * 将setuptools的build_ext命令替换为PyTorch提供的BuildExtension
  
  * BuildExtension 在构建时会调用正确的编译器（g++/nvcc）、添加 PyTorch 的 include/lib 路径、传递适当的编译选项，从而正确编译并链接扩展。

****

* 替换cppExtension的等价写法

```python
Extension(
   name='lltm_cpp',
   sources=['lltm.cpp'],
   include_dirs=cpp_extension.include_paths(),
   language='c++')
```

## torch.utils.cpp\_extension.\_wrap\_function

## nvcc编译指令

```bash
nvcc -c -o build/knn_cuda_kernel.so src/knn_cuda_kernel.cu -x cu -Xcompiler -fPIC -shared -Isrc
```
