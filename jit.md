# JIT

如果多次使用某一个正则表达式，建议先对其进行compile，再使用compile之后得到的对象来做正则匹配。而这个compile的过程就是JIT(即时编译)。
举个栗子，正则表达式匹配有两种方式

```py
pat = re.compile(pattern)
result = pat.match(string)
```

或

```py
result = re.match(pattern, string)
```

前者先编译，再匹配，如果多次使用该正则表达匹配，建议先编译，之后就只用调用transfer匹配了

# torch.jit > TorchScript

PyTorch 的 1.0 版本发布的最核心的两个新特性就是 JIT 和 C++ API，这两个特性一起发布不是没有道理的，JIT 是 Python 和 C++ 的桥梁。开发者可以使用 Python 训练模型，然后通过 JIT 将模型转为语言无关的模块， 从而让 C++ 可以非常方便得调用，从此使用 Python 训练模型，使用 C++ 将模型部署到生产环境对 PyTorch 来说成为了一件很容易的事。而因为使用了 C++， 开发者几乎可以把 PyTorch 模型部署到任意平台和设备上：树莓派、iOS、Android 等。

TorchScript是PyTorch模型的一种中间表示(即常见的intermediate representation，IR)，是nn.Module的子类，基于这种模型格式可以在高性能的环境如C++中运行。TorchScript这种PyTorch模型表示格式可以由 TorchScript 编译器理解、编译和序列化而成。从根本上说，TorchScript 本身就是一种编程语言。它是使用 PyTorch API 的 Python 的子集。简单来说，TorchScript 软件栈可以将 Python 代码转换成 C++ 代码。TorchScript 软件栈包括两部分：TorchScript(Python)和 LibTorch(C++)。TorchScript负责将 Python 代码转成一个模型文件，LibTorch负责解析运行这个模型文件。后续会从代码层面介绍如何保存为TorchScript模型格式及其在Python和C++中如何加载使用

* 有torch.jit.trace、torch.jit.script
* torch.jit.trace适用于静态图模型，torch.jit.script适用于动态图模型（模型的结构和输入在运行时变换）

# trace

> trace 模式顾名思义就是跟踪模型的执行，然后记录执行过程中的路径。在使用 trace 模式时，需要构造一个符合要求的输入，然后使用 TorchScript tracer 运行一遍，记录整个运行过程。在 trace 模式中运行时，每执行一个算子，就会往当前的 graph 加入一个 node。所有代码执行完毕，每一步的操作就会以一个计算图里的某个节点的形式被保存下来。

值得一提的是，PyTorch 导出 ONNX 也是使用了这部分代码，所以理论上能够导出 ONNX 的模型也能够使用 trace 模式导出 TorchScript 格式模型。

在VCAP中，将onnx模型转为vaimlite（CPU/GPU)或bin+params(NPU），之后的输入形状必须固定，猜测之前是可以的

trace 模式有以下2点限制：

* 不能有 if-else 等控制流
* 只支持 Tensor 操作。不支持非Tensor 操作，如List、Tuple、Map 等容器操作

通过 trace 模式的特点可以看出，trace 模式通常用于深度模型的导出。常见的深度模型通常没有 if-else 控制流且通常==没有非 Tensor 操作==。

实例：

```py
import torch
torch.manual_seed(42)


class TestTraceCell(torch.nn.Module):
    def __init__(self):
        super(TestTraceCell, self).__init__()
        self.linear = torch.nn.Linear(4, 4)

    def forward(self, x, h):
        new_h = torch.tanh(self.linear(x) + h)
        return new_h, new_h

test_cell = TestTraceCell()
x, h = torch.rand(3, 4), torch.rand(3, 4)
traced_cell = torch.jit.trace(test_cell, (x, h))
print(traced_cell)

print("trace module=",traced_cell(x, h))
```

# script

script 模式和trace 模式的使用方式很接近， 但是实现原理却大相径庭。TorchScript 实现了一个完整的编译器以支持 script 模式。保存模型阶段对应着编译器的前端(语法分析、语义分析、类型检查、中间代码生成)。即，对code进行词法分析、语法分析、句法分析，形成AST树(抽象语法树)，最后再将AST树线性化。在保存模型时，TorchScript 编译器解析 Python 代码，并构建代码的AST。script 模式在的限制比较小，不仅支持 if-else 等控制流，还支持非 Tensor 操作，如 List、Tuple、Map 等容器操作。script相当于一个嵌入在Python/Pytorch的DSL，其语法只是pytorch语法的子集，这意味着存在一些op和语法script不支持，这样在编译的时候就会遇到问题。此外，script的编译优化方式更像是CPU上的传统编译优化，重点对于图进行硬件无关优化，并对IF、loop这样的statement进行优化。

```py
import torch
torch.manual_seed(42)


class TestDecisionGate(torch.nn.Module):
    def forward(self, x):
        if x.sum() > 0:
            return x
        else:
            return -x

class TestCell(torch.nn.Module):
    def __init__(self, dg):
        super(TestCell, self).__init__()
        self.dg = dg
        self.linear = torch.nn.Linear(4, 4)

    def forward(self, x, h):
        new_h = torch.tanh(self.dg(self.linear(x)) + h)
        return new_h, new_h

test_cell = TestCell(TestDecisionGate())
x, h = torch.rand(3, 4), torch.rand(3, 4)
traced_cell = torch.jit.trace(test_cell, (x, h))
print(traced_cell.code)
```

# trace+script

```py
import torch
torch.manual_seed(42)

# 无if-else，仅仅tensor op，用torch.jit.trace
class TestTraceCell(torch.nn.Module):
    def __init__(self):
        super(TestTraceCell, self).__init__()
        self.linear = torch.nn.Linear(4, 4)
    def forward(self, x, h):
        new_h = torch.tanh(self.linear(x) + h)
        return new_h, new_h

# 有if-else，用script
class TestDecisionGate(torch.nn.Module):
    def forward(self, x):
        if x.sum() > 0:
            return x
        else:
            return -x

# no if-else，only tensor，use trace
class TestCell(torch.nn.Module):
    def __init__(self, dg):
        super(TestCell, self).__init__()
        self.dg = dg
        self.linear = torch.nn.Linear(4, 4)

    def forward(self, x, h):
        new_h = torch.tanh(self.dg(self.linear(x)) + h)
        return new_h, new_h

x, h = torch.rand(3, 4), torch.rand(3, 4)
scripted_gate = torch.jit.script(TestDecisionGate())

# for = if-else，use script，but member member can use trace
class TestRNNLoop(torch.nn.Module):
    def __init__(self):
        super(TestRNNLoop, self).__init__()
        self.cell = torch.jit.trace(TestCell(scripted_gate), (x, h))        # TestCell本身用trace，但transfer的dg用script

    def forward(self, xs):
        h, y = torch.zeros(3, 4), torch.zeros(3, 4)
        for i in range(xs.size(0)):
            y, h = self.cell(xs[i], h)
        return y, h

rnn_loop = torch.jit.script(TestRNNLoop())            
print(rnn_loop.code)
```

在某些情况下，需要使用Tracing而不是Scripting，例如，module中有许多决策，这些决策是基于Python中固定值做出的，而我们又不希望在TorchScript中出现。在这种情况下，可以通过tracing来组成scripting：torch.jit.script将内联被traced module 的代码，而tracing将内联被scripted module 的代码。

# torch.compile

torch.compile是PyTorch库中的一个函数，它主要用于将一段定义好的PyTorch脚本转换为可以高效运行的低级张量表达式（Tensor Expressions）。当你对一些计算过程进行了多次同样的操作，如前向传播、损失计算等，并且这个过程相对固定，torch.compile可以帮助优化这部分代码，使其在后续运行时速度更快，尤其是在分布式环境中或者模型部署阶段。

编译的过程会将动态图模式下的Python代码转化为静态图模式，这有助于消除不必要的函数调用开销和垃圾回收，提高性能。然而，需要注意的是，只有那些不会依赖于外部状态（如随机数生成或变量初始化）的纯函数才适合编译。

## orient dynamic input

现在有一个输入的batch_size维度可能发生变化的pytorch模型，它适合用torch.compile进行编译码

> 对于一个输入batch_size可能会变化的PyTorch模型，通常不太适合直接使用torch.compile进行编译。因为torch.compile是针对静态图模式设计的，它期望所有的输入形状在编译时已经确定下来，以便生成高效的执行计划。如果模型的输入batch_size部分是动态的，那么每次传入的数据都会影响到计算路径，这种情况下静态图并不能很好地优化性能。
> 
> 相反，如果你的模型有大量可重复使用的前向传播逻辑，但是batch_size的变化仅限于网络的部分层（例如卷积神经网络的池化层），你可以考虑把这部分逻辑封装成一个独立的模块，并在该模块内部处理batch_size的变化。然后，这个模块就可以作为静态图的一部分来编译，而其他部分继续保持动态图的灵活性。
> 所以，在这种情况下，更好的做法可能是拆分模型，只对那些可以提前确定计算流程的部分进行编译。

# Questions

> Q1:torch.jit.trace支持列表的索引操作吗

C知道说torch.jit.trace不支持，torch.jit.script支持，但本地试验的时候，都是支持的

> Q2:torch.jit.trace如何接受多级input？

* 直接按照教程的拼接方法再用torch.jit.trace()会bloom
  
  ```sh
  :List inputs to traced functions must have consistent element type. Found Tensor and Tuple[Tensor, Tensor]
  ```
  
  [discuss1](https://github.com/facebookresearch/detr/issues/208)
  
  ```sh
  RuntimeError: Only tensors, lists, tuples of tensors, or dictionary of tensors can be output from traced functions
  ```
  
  script
  
  ```sh
  2024-10-14 22:03:51.730 | ERROR    | actor.agent:reset:102 - could not get source code
  2024-10-14 22:03:51.730 | ERROR    | actor.agent:reset:103 - get_model error, try again...
  ```
  
  # Reference
1. [模型推理加速系列 | 05: 推理加速格式TorchScript简介及其应用](https://developer.aliyun.com/article/1258481)
2. [torch.jit.trace与torch.jit.script的区别](https://cloud.tencent.com/developer/article/2067810)
3. [torch.jit.trace Official Guide](https://pytorch.org/docs/stable/generated/torch.jit.trace.html)

# implement

> 我也不知道有什么意义，但好歹推进了一点

## Bug1

> RuntimeError: Type '<class 'actor.model.Model'>' cannot be compiled since it inherits from nn.Module, pass an instance instead  Model类前加了@torch.jit.script，请问这是什么原因

**GPT**：
在 PyTorch 中，`@torch.jit.script` 装饰器用于将 Python 函数或类编译为 TorchScript，这是一种可以在 C++ 环境中运行的中间表示形式。TorchScript 允许你将 PyTorch 模型导出并在没有 Python 解释器的环境中运行。

当你在一个继承自 `nn.Module` 的类上使用 `@torch.jit.script` 时，可能会遇到 `RuntimeError: Type '<class 'actor.model.Model'>' cannot be compiled since it inherits from nn.Module, pass an instance instead` 这个错误。这是因为 ==`@torch.jit.script` 期望的是一个函数或类的实例==，而不是类本身。

要解决这个问题，你可以使用 `@torch.jit.script` 装饰器来装饰类中的方法，而不是整个类。以下是一个示例：

```python
import torch
import torch.nn as nn

class MyModel(nn.Module):
    def __init__(self):
        super(MyModel, self).__init__()
        self.linear = nn.Linear(10, 5)

    @torch.jit.script_method
    def forward(self, x):
        return self.linear(x)

# 创建模型实例
model = MyModel()

# 使用 TorchScript 编译模型
scripted_model = torch.jit.script(model)

# 现在可以使用 scripted_model
input_tensor = torch.randn(1, 10)
output = scripted_model(input_tensor)
print(output)
```

在这个示例中，`forward` 方法被装饰为 `@torch.jit.script_method`，而不是直接在类上使用 `@torch.jit.script`。这样可以避免你遇到的错误。

总结一下，确保你在 `nn.Module` 的实例方法上使用 `@torch.jit.script_method`，而不是在类定义上使用 `@torch.jit.script`。

### 方案：

在network.forward()前加@torch.jit.script

## Bug2

> RuntimeError:
> 'Tensor (inferred)' object has no attribute or method 'all_hero_feature_dim'.:
>   File ".\aiarena\process\..\code\common\algorithm_torch.py", line 222
>         feature_vec_split_list = feature_vec.split(
>             [
>                 self.all_hero_feature_dim,
>                 ~~~~~~~~~~~~~~~~~~~~~~~~~ <--- HERE
>                 self.all_soldier_feature_dim,
>                 self.all_organ_feature_dim,

方案：
修改forward中的所有成员变量为具体值

## Bug3

> RuntimeError:
> 'Tensor (inferred)' object has no attribute or method 'hero_main_mlp'.:
>   File ".\aiarena\process\..\code\common\algorithm_torch.py", line 323
>         """ real computations"""
>         hero_main_result = self.hero_main_mlp(_hero_main)
>                            ~~~~~~~~~~~~~~~~~~ <--- HERE
>         hero_emy_mlp_out = self.hero_mlp(_hero_emy)

network.forward()引用成员模块，但这也要手动写，需要把network的所有子模块当作实参传入，

* 用@torch.jit.script修饰复杂类的成员函数不现实
* 但可以把torch.nn的实现复制过来，这样不用继承了

## Bug4

转投torch.jit.trace

> :List inputs to traced functions must have consistent element type. Found Tensor and Tuple[Tensor, Tensor]

torch.jit.trace()只接受 Tuple[Tensor,...,Tensor] or Tensor

## Commit 0 = torch.jit.trace()编译成功！

把所有的tensor的组合(集合、列表、字典）全展成Tensor列表，一个个输入，终于编译成功了

* 但效果未知，因为实际的模型包含了for循环，可能不支持，

* 但本模块中的所有for均为固定格式的判断，没有if-else路径的切换，理论上是可以的
  
  ## Q1
  
  > torch.jit.trace优化的模型的forward可以包含for循环吗
  > torch.jit.trace 是PyTorch提供的一种模型优化技术，它主要用于将Python函数式的模型转换为高效的低级张量操作，以便在推理阶段获得更快的速度。在使用trace()对模型进行追踪时，它通常假设输入数据是静态的，并且函数体中的大部分计算应该是可以直接编译的。

关于forward函数是否能包含for循环，这取决于循环内的内容能否被静态地推断出来并且转化为等效的张量操作。如果循环内部的操作是对单个输入元素进行简单变换并累积结果（例如卷积或累加），那么理论上是可以支持的。但是，复杂的条件控制、随机数生成或动态图操作通常不在trace的支持范围内，因为它们难以确定其对输入的所有可能影响。

总的来说，对于那些能够简化为固定序列运算的循环，torch.jit.trace可能会工作，但对于依赖于循环变量状态或不确定性的部分，可能会失败或者效果不佳。

## Extension: trace VS script

torch.jit.trace 和 torch.jit.script 都是PyTorch用于将Python函数转换为 TorchScript（高效的低级神经网络中间表示）的方法，但它们的工作原理和性能特点有所不同。

torch.jit.trace 是基于采样（tracing）的方式，它会记录函数在特定输入上运行的路径，并生成对应的执行图。当输入在trace期间发生变化时，trace 的效果可能会受到影响，因为它是针对单个或一组输入进行优化的。如果输入数据集是静态的或者变化不大，trace 可能能提供接近原生性能的速度，因为它避免了完整的静态分析。

相比之下，torch.jit.script 则会对整个函数进行全面的静态分析，然后生成一个通用的执行计划。这种方法通常会产生更高效的代码，但它需要完整地理解和构建函数的结构，所以对于动态操作或复杂控制流，script 有时可能会比较慢，但如果模型结构不变并且可以完全分析，其速度通常优于trace。

总的来说，trace 更适合于轻量级、输入固定或者变化较小的情况，而script 更适合那些可以提前分析并优化的场景。实际性能取决于具体的用例和模型特性。

==重要的是，torch.jit.script不能编译继承类!!!==

## solution: 考虑把nn.Module源码搬到要继承的类下面
