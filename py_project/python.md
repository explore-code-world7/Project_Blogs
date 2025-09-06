# *

```python
def model(x, *, t, y, name, num_classes, reuse=tf.AUTO_REUSE, ch, out_ch, ch_mult=(1, 2, 4, 8), num_res_blocks,
          attn_resolutions, dropout=0., resamp_with_conv=True):
  B, S, _, _ = x.shape
  assert x.dtype == tf.float32 and x.shape[2] == S
  assert t.dtype in [tf.int32, tf.int64]
  num_resolutions = len(ch_mult)
```

在 Python 中，* 的使用在函数定义中有特定的含义。它用于指示在 * 之后的参数必须以关键字参数的形式传递。这意味着在调用该函数时，所有在 * 后面的参数都不能通过位置参数传递，必须使用关键字来指定。

在你提供的 model 函数定义中，* 的作用如下：

    强制使用关键字参数：在 * 之后的参数（如 t, y, name, num_classes, reuse, ch, out_ch, ch_mult, num_res_blocks, attn_resolutions, dropout, resamp_with_conv）必须以关键字的形式传递。例如，调用该函数时，必须使用 model(x, t=t_value, y=y_value, name='model_name', ...) 的形式，而不能简单地使用位置参数 model(x, t_value, y_value, 'model_name', ...)。
    
    提高可读性：这种方式可以提高函数调用的可读性，尤其是当函数参数较多时。使用关键字参数可以清楚地表明每个参数的含义，减少混淆。
    
    避免参数顺序错误：强制使用关键字参数可以避免在调用函数时因参数顺序错误而导致的错误。

# =

* 使用 = 将一个对象赋值给一个变量时，实际上是将该对象的引用（而不是对象本身）赋给变量。

* 对b 的修改也会影响到 对a的修改，因为b,a引用同一个对象。

* copy()则复制了一个新的对象。
  
  ## case
  
  ```python
  a=[10,2]
  b=a
  a[1]=5
  b
  ```

* 但对a的重新赋值，并不会影响到b

# list comprehension

使用列表推导式（list comprehension）构建列表通常比在 `for` 循环中逐个添加元素的方式更高效。这是因为列表推导式在内部实现上是优化过的，能够减少 Python 的函数调用开销和循环的开销。

## 性能比较

1. 列表推导式：
   
   - 列表推导式在创建列表时会在内存中一次性分配所需的空间，这样可以减少内存的重新分配次数。
   - 由于它是用 C 语言实现的，执行速度通常比 Python 的 `for` 循环快。

2. `for` 循环：
   
   - 在 `for` 循环中，每次添加元素时，Python 可能需要多次重新分配内存，尤其是在列表增长时，这会导致性能下降。
   - `for` 循环的语法相对较长，且每次迭代都涉及到 Python 的函数调用。

# @classmethod

在 Python 中，`@classmethod` 是一个装饰器，用于定义类方法。类方法的第一个参数是类本身（通常命名为 `cls`），而不是实例（通常命名为 `self`）。这使得类方法可以访问类的属性和方法，而不需要实例化该类。

## 主要作用

1. **访问类属性**: 类方法可以访问和修改类的属性，而不需要创建类的实例。

2. **工厂方法**: 类方法常用于定义工厂方法，这些方法可以根据不同的输入参数返回类的实例。

3. **与实例无关**: 类方法可以在没有实例的情况下被调用，这使得它们在某些情况下比实例方法更方便。

## 示例

以下是一个使用 `@classmethod` 的示例：

```python
class MyClass:
    class_variable = 0

    def __init__(self, value):
        self.instance_variable = value

    @classmethod
    def increment_class_variable(cls):
        cls.class_variable += 1

    @classmethod
    def create_instance(cls, value):
        return cls(value)

# 使用类方法访问和修改类变量
MyClass.increment_class_variable()
print(MyClass.class_variable)  # 输出: 1

# 使用类方法创建实例
instance = MyClass.create_instance(10)
print(instance.instance_variable)  # 输出: 10
```

## 解释

- `increment_class_variable` 是一个类方法，它通过 `cls` 参数访问和修改类变量 `class_variable`。
- `create_instance` 是一个工厂方法，它使用类方法创建并返回 `MyClass` 的实例。

## 总结

`@classmethod` 使得你可以在类级别上定义方法，这些方法可以访问类的属性和方法，而不需要实例化类。它在需要访问类状态或创建类实例时非常有用。

# sh

## 删除

* 文件夹+所有文件
  
  ```py
  import shutil,os
  ```

if os.path.exists(folder_path):
    # 删除文件夹及其所有内容
    shutil.rmtree(folder_path)

# numpy.void

在 Python 的 NumPy 库中，`numpy.void` 是一种特殊的数据类型，用于表示结构化数组中的元素。它通常用于存储具有不同数据类型的复合数据结构。`numpy.void` 实际上是一个封装了多个字段的对象，类似于 C 语言中的结构体。

## 特点

- `numpy.void` 对象可以包含多个字段，每个字段可以是不同的数据类型。
- 它通常用于处理结构化数组（structured arrays）或记录数组（record arrays），这些数组的每个元素可以包含多个不同类型的值。

## 如何构造 `numpy.void` 类型的元素

要构造 `numpy.void` 类型的元素，通常需要先定义一个结构化数组的 dtype，然后使用这个 dtype 创建数组。以下是一个示例：

```python
import numpy as np

# 定义结构化数组的 dtype

dtype = np.dtype([('field1', np.int32), ('field2', np.float64), ('field3', 'U10')])

# 创建一个结构化数组

structured_array = np.zeros(1, dtype=dtype)

# 访问和设置字段

structured_array[0]['field1'] = 42
structured_array[0]['field2'] = 3.14
structured_array[0]['field3'] = 'Hello'

# 获取 numpy.void 对象

void_element = structured_array[0]

print("Void Element:", void_element) # 输出: Void Element: (42, 3.14, 'Hello')
print("Type of void_element:", type(void_element)) # 输出: <class 'numpy.void'>
```

## 说明

1. **定义 dtype**：使用 `np.dtype` 定义一个包含多个字段的结构化数据类型。每个字段都有名称和数据类型。
2. **创建结构化数组**：使用 `np.zeros` 创建一个结构化数组，指定 dtype。
3. **访问和设置字段**：可以通过字段名称访问和设置结构化数组中的字段值。
4. **获取 `numpy.void` 对象**：访问结构化数组的元素时，会返回一个 `numpy.void` 对象。

## 总结

`numpy.void` 是用于表示结构化数组中元素的类型，允许存储不同数据类型的复合数据。通过定义结构化数组的 dtype，可以方便地创建和操作这种类型的元素。

# package installation

```bash
# 安装古老版本
pip install torch==1.12.1+cu113 --index-url https://download.pytorch.org/whl/cu118


# 安装最新版本
pip install torchaudio==2.5.1+cu118 -f https://download.pytorch.org/whl/torch_stable.html## 说明
```

# yield

```python
def test_yield():
    for i in range(5):
        yield 10, 20

res= test_yield()
print(res)
```

```bash
<generator object test_yield at 0x78f0e4450190>
```

```python
c= list(res)
print(c)

# 用list转义完后，迭代器的指针到达最末端

for a, b  in res:
    print(a,b)
```

```bash
[(10, 20), (10, 20), (10, 20), (10, 20), (10, 20)]
```

# eval

## usage

```python
eval(expression, globals=None, locals=None)
```

- **expression**：一个字符串，表示要计算的 Python 表达式。
- **globals**（可选）：一个字典，用于指定全局命名空间。
- **locals**（可选）：一个字典，用于指定局部命名空间。

## 作用

1. 计算表达式的值

2. 使用全局和局部命名空间
* 构造神经网络

```python
# 模型类的名称
model_class_name = "SimpleNN"

# 使用 eval() 创建模型实例
model = eval(model_class_name)()  # 注意这里的 () 是调用构造函数

print(model)  # 输出模型结构
```

# Sequence对象的使用

* 定义

```python
# to build Sequence object
from collections.abc import Sequence

class MySequence(Sequence):
    def __init__(self, data):
        self._data = list(data)

    def __len__(self):
        return len(self._data)

    def __getitem__(self, index):
        return self._data[index]
```

* 调用

```python
print(self.robot.find_joints(name_keys=MySequence([".*_wrist_.*_joint", ".*_base_link_joint", ".*_index_.*_joint", 
".*_middle_.*_joint", ".*_ring_.*_joint"])))
```

# wrapper

```python
import functools

def height_field_to_mesh(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(difficulty: float, cfg: HfTerrainBaseCfg):
        ...
        return xxx, xxx
    return wrapper

@height_field_to_mesh
def random_uniform_terrain(difficulty: float, cfg: hf_terrains_cfg.HfRandomUniformTerrainCfg) -> np.ndarray:
```

# Json

```python
            with open(os.path.join(log_dir, "config.json"), "w") as f:
                json.dump(log_cfg_dict, f, indent= 4)
```

# torch

* 双树组索引和slice索引不同

```python
import torch

a = torch.tensor([[1, 2, 3],
                  [4, 5, 6],
                  [7, 8, 9]])

b = torch.tensor([0, 1, 2])  # 行索引
c = torch.tensor([1, 2, 0])  # 列索引

result = a[b, c]
print(result)  # 输出: tensor([2, 6, 7])
```

在 Python 中，特别是使用 PyTorch 或 NumPy 等库时，`a[b, c]` 表示对张量 `a` 进行索引操作，其中 `b` 和 `c` 是索引数组或张量。

## 解释

- `a` 是一个多维张量（数组）。
- `b` 和 `c` 是索引数组，通常是整数数组，表示要从 `a` 中提取的元素的位置。
- `a[b, c]` 将返回一个新的张量，其中包含 `a` 中在位置 `(b[i], c[i])` 的元素，`i` 是索引的循环变量。

# 多继承init函数书写

* init函数
