# plt的基本用法

* plot画自变量-因变量关系图;
* imshow presents numpy array images.

Matplotlib 的 pyplot 模块提供了许多函数来创建和自定义图形。以下是一些常用的函数：

    绘图函数:
        plt.plot(): 绘制线图。
        plt.scatter(): 绘制散点图。
        plt.bar(): 绘制条形图。
        plt.hist(): 绘制直方图。
        plt.imshow(): 显示图像。
    
    坐标轴和标签:
        plt.xlabel(): 设置 x 轴标签。
        plt.ylabel(): 设置 y 轴标签。
        plt.title(): 设置图形标题。
        plt.xlim(): 设置 x 轴范围。
        plt.ylim(): 设置 y 轴范围。
    
    图例和网格:
        plt.legend(): 添加图例。
        plt.grid(): 显示网格线。
    
    保存图形:
        plt.savefig(): 将图形保存为文件。
    
    子图:
        plt.subplot(): 创建子图。
        plt.subplots(): 创建多个子图。
    
    其他:
        plt.tight_layout(): 自动调整子图参数以使图形更美观。
        plt.annotate(): 添加注释。

总结

plt.show() 用于显示图形，而 plt.close() 用于关闭图形。Matplotlib 提供了丰富的函数来创建和自定义图形，用户可以根据需要选择合适的函数来实现数据可视化。

# fig&ax

在 Matplotlib 中，`fig, ax = plt.subplots()` 是一个常用的函数，用于创建一个图形（figure）和一个或多个子图（axes）。在这个语句中，`fig` 和 `ax` 的作用如下：

## 1. `fig`（Figure 对象）

- **作用**: `fig` 是一个 `Figure` 对象，代表整个图形窗口或图形区域。它是所有绘图元素的容器，包括坐标轴、标题、图例等。
- **使用**: 你可以使用 `fig` 对象来设置图形的属性，例如大小、标题、保存图形等。

## 2. `ax`（Axes 对象）

- **作用**: `ax` 是一个 `Axes` 对象，代表图形中的一个坐标轴区域。你可以在这个区域内绘制数据。
- **使用**: 你可以使用 `ax` 对象来绘制数据、设置坐标轴标签、标题、刻度等。

## 示例

以下是一个简单的示例，展示如何使用 `fig` 和 `ax`：

```python
import matplotlib.pyplot as plt
import numpy as np

# 创建数据
x = np.linspace(0, 10, 100)
y = np.sin(x)

# 创建图形和坐标轴
fig, ax = plt.subplots(figsize=(8, 4))  # 设置图形大小为 8x4 英寸

# 在坐标轴上绘制数据
ax.plot(x, y, label='Sine Wave')

# 设置坐标轴标签和标题
ax.set_xlabel('X Axis')
ax.set_ylabel('Y Axis')
ax.set_title('Sine Wave Example')

# 添加图例
ax.legend()

# 显示图形
plt.show()

# 保存图形
fig.savefig('sine_wave.png')  # 使用 fig 保存图形
```

## 主要功能

- **设置图形大小**: 通过 `figsize` 参数设置图形的宽度和高度。
- **保存图形**: 使用 `fig.savefig('filename.png')` 将图形保存为文件。
- **设置图形标题**: 可以使用 `fig.suptitle('Title')` 设置整个图形的标题。
- **调整布局**: 使用 `fig.tight_layout()` 自动调整子图参数以使图形更美观。

## 总结

`fig` 是 Matplotlib 中的 `Figure` 对象，代表整个图形窗口。通过 `fig`，你可以设置图形的属性、保存图形等。`ax` 是 `Axes` 对象，代表绘图区域，主要用于绘制数据和设置坐标轴属性。使用 `fig` 和 `ax` 可以更灵活地控制图形的外观和内容。

# fig如何操作？

* https://stackoverflow.com/questions/34162443/why-do-many-examples-use-fig-ax-plt-subplots
* 不用管，只关心plt即可
