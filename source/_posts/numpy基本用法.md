---
title: Numpy的基本使用
category: 数据分析
tag:
    - numpy
---

### 创建numpy数组

```python
import numpy as np

# 三种方式创建numpy数组
n1 = np.array([1, 2, 3, 4, 5])
n2 = np.array(range(1, 6))
n3 = np.arange(1, 6)

print(type(n1))  # numpy.ndarray
print(n1.dtype)  # int64

# 创建指定数据类型的numpy数组
n4 = np.array([1, 0, 0, 1, 0], dtype=bool)

```


### numpy的索引和切片
```python
import numpy as np

a = np.array(range(12)).reshape((3, 4))

# 取一行
print(a[2])  # array([ 8,  9, 10, 11])
print(a[2,:])
# 取连续的多行
print(a[1:])
print(a[1:,:])
# 取不连续的多行
print(a[[0,2]])
print(a[[0,2],:])

# 取一列
print(a[:,0])  # array([0, 4, 8])
# 取连续的多列
print(a[:,1:])
# 取不连续的多列
print(a[:,[0, 2]])

# 取多行和多列
# 取第二行第三列的值
print(a[1,2])
# 取第一行到第二行，第三列到第四列
print(a[0:2, 2:4])

# 取多个不相邻的点
# 取第一行第三列的点和第二行第四列的点
print(a[[0,1],[2,3]])
```

### numpy数值的修改
```python

import numpy as np

a = np.array(range(12)).reshape((3, 4))

a[2] = 9  # 找到索引或者切片后直接赋值就可以

# 将小于5的值替换为0
a[a < 5] = 0  # 会改变原来的numpy数组

# 将小于等于5的值替换为-5, 大于5的替换为10
b = np.where(a<=5, -5, 10)  # 不会改变原来的numpy数组

# 把小于3的替换为3，大于9的替换为9
a.clip(3, 9)
```


### 数组的拼接
```python
import numpy as np

t1 = np.array(range(12)).reshape((2, 6))
t2 = np.array(range(12, 24)).reshape((2, 6))

# 竖直拼接  (vsplit()方法竖直分割）
t3 = np.vstack((t1, t2))

# 水平拼接
t4 = np.hstack((t1,t2))
```

### 数组的行列交换
```python
import numpy as np

t = np.array(range(12, 24)).reshape((3, 4))

# 行交换
t[[1,2],:] = t[[2,1],:]  # 会改变原数组

# 列交换
t[:,[0,2]] = t[:,[2,0]]
```