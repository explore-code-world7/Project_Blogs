## batch training

# cuda out of memory

0. 按批load to gpu
   
   ```py
   for batch_idx, batch_data in enumerate(Test_Loader):
    batch_data = batch_data.to(device)
   ```

对比实验
2. 仅仅将batch_data存储，正常
3. 将神经网络经过模型预测后，无论是否释放模型的输出，都会报错cuda out of memory

* 模型forward的时候保存了计算图,并未清除，可能是这个原因
  
  ```py
    gcnet.eval()
    with torch.no_grad():
        pos, _, _, _ = gcnet(batch_data)
  ```
3. 训练的时候
* first define a optimizer
  
  ```py
  optimizer = torch.optim.SGD(model.parameters(), lr=learning_rate)
  ```

* 参数梯度清零
  
  ```py
    optimizer.zero_grad()
  ```

* 参数迭代不计算梯度
  
  ```py
    optimizer.step()
  ```

* [some little stupid but rich functions](https://stackoverflow.com/questions/59129812/how-to-avoid-cuda-out-of-memory-in-pytorch)

# gradient

在 PyTorch 中，`model.evaluate()`、`torch.no_grad()` 和 `model.eval()` 是三个相关但功能不同的概念，主要区别如下：

### model.eval() - 模型模式切换

- **作用**：将模型设置为**评估模式**

- **影响**：
  
  - 关闭特定层的训练行为：如 Dropout（停止随机丢弃神经元）、BatchNorm（使用移动平均的统计量而非当前批次）
  - **不**影响梯度计算

- **使用场景**：
  
  - 任何模型评估/推理阶段（验证集/测试集）
  - 需要确保模型行为一致时（如生成文本/图像）

- **示例**：
  
  ```python
  model.eval()  # 切换为评估模式
  ```

---

### torch.no_grad() - 梯度计算开关

- **作用**：创建**禁用梯度计算**的上下文

- **影响**：
  
  - 停止自动求导系统跟踪计算图
  - 减少内存消耗（不保存中间变量）
  - 加速计算（跳过梯度相关操作）
  - **不**改变模型层的行为

- **使用场景**：
  
  - 模型推理/预测
  - 计算验证指标（如准确率）
  - 手动更新参数

- **示例**：
  
  ```python
  with torch.no_grad():  # 禁用梯度
      outputs = model(inputs)
  ```

---

### model.evaluate() - 评估流程封装

- **来源**：通常是用户自定义函数或高级封装库（如 PyTorch Lightning）

- **作用**：**封装整个评估流程**

- **包含内容**：
  
  - 内部自动调用 `model.eval()`
  - 内部使用 `torch.no_grad()`
  - 实现数据遍历、指标计算、结果汇总

- **伪代码示例**：
  
  ```python
  def evaluate(model, dataloader):
      model.eval()  # 切换到评估模式
      total_loss, total_acc = 0, 0
  
      with torch.no_grad():  # 禁用梯度
          for inputs, labels in dataloader:
              outputs = model(inputs)
              loss = loss_fn(outputs, labels)
              acc = (outputs.argmax(1) == labels).float().mean()
  
              total_loss += loss.item()
              total_acc += acc.item()
  
      return total_loss / len(dataloader), total_acc / len(dataloader)
  ```

---

### 关键区别总结：

| **功能**     | `model.eval()`        | `torch.no_grad()` | `model.evaluate()` |
| ---------- | --------------------- | ----------------- | ------------------ |
| **主要作用**   | 切换模型层行为               | 禁用梯度计算            | 封装评估流程             |
| **是否原生**   | ✓ (PyTorch 内置)        | ✓ (PyTorch 内置)    | ✗ (需自定义/第三方库)      |
| **影响梯度**   | 不影响梯度计算               | 禁用梯度计算            | 内部禁用梯度             |
| **改变模型行为** | ✓ (Dropout/BatchNorm) | ✗                 | 依赖内部调用             |
| **典型使用位置** | 评估前调用                 | 评估代码块上下文          | 直接调用评估函数           |

---

## no_grad for isgym

### 1. release caches of gym simulation

### 2. accumulate no grad of network

```python
    def rollout_step(self, obs, critic_obs):
        actions = self.alg.act(obs, critic_obs)
        with torch.no_grad():
            obs, privileged_obs, rewards, dones, infos = self.env.step(actions)
            torch.cuda.empty_cache()
        critic_obs = privileged_obs if privileged_obs is not None else obs
        obs, critic_obs, rewards, dones = obs.to(self.device), critic_obs.to(self.device), rewards.to(self.device), dones.to(self.device)
        self.alg.process_env_step(rewards, dones, infos, obs, critic_obs)
        return obs, critic_obs, rewards, dones, infos
```

# gpu管理

1. GPU创建多个中间变量，如何删除引用
* tensor的运算会自动释放无引用对象（中间变量）；

* torch.empty_cache()足以，用来释放删除对象后的缓存清理
2. detach和torch.no_grad()的区别

在PyTorch中，`detach()`和`with torch.no_grad()`都是用于控制梯度计算的，但它们的用途和行为有所不同。

* detach()
- **功能**：`detach()`方法用于从当前计算图中分离出一个张量。调用`detach()`后，返回的张量将不再与原始张量的计算图关联，因此在后续的操作中不会计算梯度。

- **用法**：通常用于在需要保留原始张量的同时，避免对其进行梯度计算。例如，在进行某些操作时希望不影响梯度更新。

- **示例**：
  
  ```python
  import torch
  
  x = torch.tensor([1.0, 2.0], requires_grad=True)
  y = x.detach()  # y与x不再关联
  ```
* with torch.no_grad()
- **功能**：`with torch.no_grad()`是一个上下文管理器，用于在其作用域内禁用梯度计算。所有在该上下文内的操作都不会记录梯度信息。

- **用法**：通常用于评估模型或进行推理时，以减少内存使用和加快计算速度。

- **示例**：
  
  ```python
  import torch
  
  x = torch.tensor([1.0, 2.0], requires_grad=True)
  with torch.no_grad():
      y = x * 2  # 在这个上下文中，y不会计算梯度
  ```
* 总结
- detach保留原张量的计算梯度，拷贝一个无梯度副本;

- torch.no_grad()禁用计算图，不改变所有梯度属性，但不记录运算的梯度信息，所有的计算都不考虑梯度；

- torch.inference_mode()可以提供更好的内存优化, 特别是在使用 TorchScript 或其他优化时；

- pytorch所有含梯度的张量的运算，默认记录运算的梯度信息；

这两者可以结合使用，但它们的目的和使用场景不同。

3. gc.collect和torch.cuda.empty_cache()的区别
* gc.collect只能释放CPU上的无引用对象，torch.cuda.empty_cache()释放GPU上的无引用对象
4. 实时GPU显存消耗检测
* 只能检测当前程序GPU消耗

```python
print(f"Allocated memory: {torch.cuda.memory_allocated() / (1024 ** 2)} MB")
print(f"Cached memory: {torch.cuda.memory_reserved() / (1024 ** 2)} MB")
print(torch.cuda.memory_summary())
```

* 检测gpu所有消耗

```python
import GPUtil
gpus = GPUtil.getGPUs()
for gpu in gpus:
    print(f"GPU ID: {gpu.id}, Memory Free: {gpu.memoryFree} MB, Memory Used: {gpu.memoryUsed} MB, Memory Total: {gpu.memoryTotal} MB")
```

5. torch.empty_cache()就是清除所有torch缓存(是我output位置不对)

# vmap

## 使用 `vmap` 的示例

`vmap` 是一个功能强大的工具，可以在 PyTorch 中实现向量化的计算，特别是在处理批次数据时。下面是一些使用 `vmap` 的具体示例。

### 示例 1: 批量计算点积

```python
import torch
from torch.func import vmap

# 定义两个随机张量
x = torch.randn(2, 5)  # (2, 5)
y = torch.randn(2, 5)  # (2, 5)

# 使用 vmap 批量计算点积
batched_dot = vmap(torch.dot)  # [N, D], [N, D] -> [N]
result = batched_dot(x, y)
print(result)  # 输出的形状为 (2,)
```

### 示例 2: 批量模型预测

假设我们有一个简单的线性模型：

```python
import torch
from torch.func import vmap

# 模型权重
weights = torch.randn(5, requires_grad=True)

# 定义模型
def model(feature_vec):
    return feature_vec.dot(weights).relu()

# 创建一些样本数据
batch_size = 3
feature_size = 5
examples = torch.randn(batch_size, feature_size)  # (3, 5)

# 使用 vmap 批量运行模型
result = vmap(model)(examples)
print(result)  # 输出的形状为 (3,)
```

### 示例 3: 计算雅可比矩阵

`vmap` 也可以用于更复杂的梯度计算，例如雅可比矩阵：

```python
import torch
from torch.func import vmap

# 定义函数
def f(x):
    return x ** 2

# 创建一个随机张量
x = torch.randn(5, requires_grad=True)  # (5,)

# 计算函数值
y = f(x)

# 创建单位矩阵
I_N = torch.eye(5)

# 使用 vmap 计算雅可比矩阵的每一行
def get_vjp(v):
    return torch.autograd.grad(y, x, v)[0]

jacobian = vmap(get_vjp)(I_N)  # (5, 5)
print(jacobian)
```

### 示例 4: 嵌套的 `vmap`

可以嵌套使用 `vmap`，进行多重向量化计算：

```python
import torch
from torch.func import vmap

# 定义两个随机张量
x = torch.randn(2, 3, 5)  # (2, 3, 5)
y = torch.randn(2, 3, 5)  # (2, 3, 5)

# 嵌套 vmap 计算点积
batched_dot = vmap(vmap(torch.dot))  # [N1, N0, D], [N1, N0, D] -> [N1, N0]
result = batched_dot(x, y)
print(result.shape)  # 输出的形状为 (2, 3)
```

这些示例展示了如何使用 `vmap` 来批量处理不同的操作。你可以根据需求调整输入和输出的维度，以实现你的功能需求。如果你有其他问题，请告诉我！
