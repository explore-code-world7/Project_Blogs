# t-SNE

t-SNE（t-分布随机邻域嵌入）是一种广泛应用于数据降维和可视化的技术。它特别适用于高维数据的探索，能够有效地将数据映射到二维或三维空间，同时尽可能保持数据点之间的局部关系。

### t-SNE 的工作原理

- **局部结构**：t-SNE通过计算数据点之间的相似性，努力在低维空间中尽可能保留高维空间的数据点的局部结构。
- **概率分布**：t-SNE首先在高维空间中为每对样本计算条件概率，然后在低维空间中为每对样本定义一个类似的概率分布，最后通过最小化这两个分布之间的Kullback-Leibler散度来优化低维表示。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.manifold import TSNE

# 生成随机数据
data = np.random.rand(64, 10)  # 64个样本，每个样本维度为10
target = np.arange(8).repeat(8)  # 64个标签

# 应用t-SNE进行降维
tsne_features = TSNE(n_components=2, learning_rate='auto', init='pca', random_state=2022).fit_transform(data)

# 绘制散点图
plt.scatter(x=tsne_features[:, 0], y=tsne_features[:, 1], c=target, cmap='jet')
plt.xlabel('Component 1')
plt.ylabel('Component 2')
plt.title('t-SNE Visualization')
plt.show()
```

### 关键参数

- **n_components**: 降维后的维度，一般选择 2 或 3。
- **learning_rate**: 学习率，影响收敛速度和结果的稳定性。
- **init**: 初始化方法，通常为 `'pca'` 或 `'random'`。
- **random_state**: 固定随机种子，以便重现结果。

****
