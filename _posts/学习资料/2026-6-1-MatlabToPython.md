---
title: 从 MATLAB 到 Python：信号处理学习指南
author: lixinghui
date: 2026-6-1 12:00:00 +0800
categories: [Note, Python]
tags: [学习资料]
---

# 从 MATLAB 到 Python：信号处理学习指南

> 本指南面向有 MATLAB 基础的工程师，系统讲解 NumPy、SciPy、Matplotlib、Pandas 四大库在信号处理中的应用。

---

# 第一章：从 MATLAB 到 Python —— 思维转换速查

## 1.1 最核心的一个区别

```
MATLAB：一切从 1 开始计数
Python：一切从 0 开始计数
```
```matlab
% MATLAB
a = [10, 20, 30];
a(1)    % → 10（第一个元素）
a(3)    % → 30（第三个元素）
```
```python
# Python
import numpy as np
a = np.array([10, 20, 30])
a[0]    # → 10（第一个元素）
a[2]    # → 30（第三个元素）
```
## 1.2 MATLAB vs Python 语法对照表

| 操作           | MATLAB             | Python (NumPy)            |
| -------------- | ------------------ | ------------------------- |
| 注释           | `% 注释`           | `# 注释`                  |
| 打印           | `disp(x)`          | `print(x)`                |
| 行向量         | `[1 2 3]`          | `np.array([1, 2, 3])`     |
| 列向量         | `[1;2;3]`          | `np.array([[1],[2],[3]])` |
| 等差序列       | `1:10`             | `np.arange(1, 11)`        |
| 等差序列(步长) | `0:0.1:1`          | `np.arange(0, 1.01, 0.1)` |
| 线性等分       | `linspace(0,1,11)` | `np.linspace(0, 1, 11)`   |
| 全零矩阵       | `zeros(3,4)`       | `np.zeros((3, 4))`        |
| 全一矩阵       | `ones(3,4)`        | `np.ones((3, 4))`         |
| 单位矩阵       | `eye(3)`           | `np.eye(3)`               |
| 随机数         | `rand(3,4)`        | `np.random.rand(3, 4)`    |
| 矩阵大小       | `size(A)`          | `A.shape`                 |
| 长度           | `length(v)`        | `len(v)`                  |
| 转置           | `A'`               | `A.T`                     |
| 逐元素乘法     | `A .* B`           | `A * B`                   |
| 矩阵乘法       | `A * B`            | `A @ B`                   |
| 逐元素幂       | `A .^ 2`           | `A ** 2`                  |
| 索引           | `A(2,3)`           | `A[1, 2]`                 |
| 切片           | `A(1:3, :)`        | `A[0:3, :]`               |
| 逻辑索引       | `A(A>0)`           | `A[A>0]`                  |
| FFT            | `fft(x)`           | `np.fft.fft(x)`           |
| IFFT           | `ifft(x)`          | `np.fft.ifft(x)`          |
| 卷积           | `conv(a,b)`        | `np.convolve(a, b)`       |
| 求解线性方程   | `A \ b`            | `np.linalg.solve(A, b)`   |
| 伪代码块       | `if ... end`       | `if ...:`（缩进代替 end） |

## 1.3 MATLAB 的 `end` 不见了

MATLAB 用 `end` 关闭代码块，Python 用 **缩进**：
```matlab
% MATLAB
for i = 1:10
    if mod(i, 2) == 0
        disp(i);
    end
end
```
```python
# Python
for i in range(1, 11):
    if i % 2 == 0:
        print(i)
```

> **关键：Python 的缩进是语法的一部分，不是装饰。** 混用 Tab 和空格会直接报错。建议编辑器设置为 **4 个空格缩进**。

## 1.4 Python 的列表 ≠ MATLAB 的数组

这是最容易踩的坑：
```python
# Python 列表 — 不是数学上的向量！
a = [1, 2, 3]
a * 2       # → [1, 2, 3, 1, 2, 3]  ← 重复，不是乘法！
a + [4]     # → [1, 2, 3, 4]        ← 拼接，不是加法！
# NumPy 数组 — 这才是 MATLAB 意义上的数组
import numpy as np
a = np.array([1, 2, 3])
a * 2       # → array([2, 4, 6])    ← 逐元素乘法 ✅
a + 4       # → array([5, 6, 7])    ← 逐元素加法 ✅
```
**结论：做数值计算，永远用 `np.array`，不要用 Python 原生列表。**

## 1.5 环境搭建（5 分钟搞定）

```bash
# 1. 安装 Miniconda（推荐）
#    下载：https://docs.conda.io/en/latest/miniconda.html
# 2. 创建虚拟环境
conda create -n signal python=3.11 -y
conda activate signal
# 3. 一键安装所有需要的库
conda install numpy scipy matplotlib pandas jupyter -y
# 4. 启动 Jupyter（交互式笔记本，类似 MATLAB 的 live script）
jupyter notebook
```

---

# 第二章：NumPy —— 一切计算的基石

> NumPy = Numerical Python。它是 Python 中所有科学计算的基础，对应 MATLAB 的核心矩阵运算能力。

## 2.1 创建数组

### 2.1.1 从列表创建

```python
import numpy as np
# 一维数组（向量）
a = np.array([1, 2, 3, 4, 5])
print(a)          # [1 2 3 4 5]
print(a.dtype)    # int64
print(a.shape)    # (5,)
# 二维数组（矩阵）
B = np.array([[1, 2, 3],
              [4, 5, 6]])
print(B.shape)    # (2, 3)
# 指定数据类型
c = np.array([1, 2, 3], dtype=np.float64)
print(c.dtype)    # float64
# 信号处理中常用：复数数组
d = np.array([1+2j, 3+4j, 5+6j])
print(d.dtype)    # complex128
```

### 2.1.2 快速创建

```python
import numpy as np
# ─── 等差序列 ───
# MATLAB: 0:0.5:2  → [0, 0.5, 1.0, 1.5, 2.0]
# Python 写法1: arange（左闭右开，类似 range）
a = np.arange(0, 2.01, 0.5)     # [0.  0.5 1.  1.5 2. ]
# 注意：arange 对浮点步长可能漏掉端点，推荐用 linspace
# Python 写法2: linspace（左闭右闭，推荐！）
b = np.linspace(0, 2, 5)        # [0.  0.5 1.  1.5 2. ]
# ─── 特殊矩阵 ───
zeros = np.zeros(5)              # [0. 0. 0. 0. 0.]       一维
zeros2d = np.zeros((3, 4))      # 3×4 全零矩阵
ones = np.ones((2, 3))          # 2×3 全一矩阵
eye = np.eye(4)                 # 4×4 单位矩阵
diag = np.diag([1, 2, 3])      # 对角矩阵
# ─── 随机数 ───
rng = np.random.default_rng(42) # 推荐的新式随机数生成器
uniform = rng.random(5)         # [0,1) 均匀分布  ← 对应 MATLAB rand(1,5)
normal = rng.standard_normal(5) # 标准正态分布    ← 对应 MATLAB randn(1,5)
integers = rng.integers(0, 10, size=5)  # [0,10) 整数
```

> **MATLAB 习惯提醒**：MATLAB 的 `rand(3,4)` 生成 3×4 矩阵，NumPy 的 `rng.random((3,4))` 参数是**元组**，注意多一层括号。

### 2.1.3 信号处理常用：时间向量

```python
# 生成采样时间向量（最常用）
fs = 1000          # 采样率 1000 Hz
T = 1.0            # 总时长 1 秒
N = int(fs * T)    # 采样点数 = 1000
# 方式1：用 arange
t1 = np.arange(N) / fs
# 方式2：用 linspace（推荐，更直观）
t2 = np.linspace(0, T, N, endpoint=False)  # endpoint=False 不包含终点
print(t1[:5])  # [0.    0.001 0.002 0.003 0.004]
print(t2[:5])  # [0.    0.001 0.002 0.003 0.004]
```
> **`endpoint=False` 很重要**：信号处理中，$$N$$ 个采样点覆盖 $$[0, T)$$，不包含 $$T$$ 本身，这样 FFT 的频率点才对齐。

---

## 2.2 数组属性

```python
a = np.array([[1, 2, 3, 4],
              [5, 6, 7, 8],
              [9, 10, 11, 12]])
print(a.ndim)      # 2        维度数
print(a.shape)     # (3, 4)   各维度大小
print(a.size)      # 12       总元素数
print(a.dtype)     # int64    数据类型
print(a.itemsize)  # 8        每个元素字节数
print(a.nbytes)    # 96       总字节数 (12 × 8)
# ─── 类型转换 ───
b = a.astype(np.float64)    # 转为 float64
c = a.astype(np.complex128) # 转为复数
```

---

## 2.3 索引与切片（重点！和 MATLAB 差异最大）

### 2.3.1 基本索引

```python
a = np.array([10, 20, 30, 40, 50])
# MATLAB: a(1)=10, a(3)=30, a(end)=50
# Python:
a[0]      # 10    第一个
a[2]      # 30    第三个
a[-1]     # 50    最后一个  ← MATLAB 没有这个！
a[-2]     # 40    倒数第二个
```
### 2.3.2 切片

```python
a = np.array([10, 20, 30, 40, 50])
# MATLAB: a(2:4) → [20, 30, 40]
# Python:
a[1:4]    # array([20, 30, 40])   左闭右开！
# 完整对照：
# MATLAB        Python        含义
# a(1:3)        a[0:3]       前三个元素
# a(2:end)      a[1:]        第二个到最后
# a(1:end-1)    a[:-1]       除最后一个外
# a(end:-1:1)   a[::-1]      倒序
# a(1:2:end)    a[::2]       隔一个取一个（步长2）
# a(2:2:end)    a[1::2]      从第二个开始隔一个取
# ─── 2D 切片 ───
B = np.array([[1, 2, 3, 4],
              [5, 6, 7, 8],
              [9, 10, 11, 12]])
# MATLAB: B(2:3, 1:2)
# Python:
B[1:3, 0:2]    # array([[5, 6],
                #        [9, 10]])
# 取整行/整列
B[0, :]        # array([1, 2, 3, 4])   第一行
B[:, 0]        # array([1, 5, 9])       第一列
```
### 2.3.3 花式索引与布尔索引

```python
a = np.array([10, 20, 30, 40, 50])
# ─── 花式索引（用数组当索引）───
indices = np.array([0, 2, 4])
a[indices]        # array([10, 30, 50])
# ─── 布尔索引（信号处理中极常用）───
# MATLAB: a(a > 25)
mask = a > 25
print(mask)       # [False False  True  True  True]
a[mask]           # array([30, 40, 50])
# 一步写法
a[a > 25]         # array([30, 40, 50])
# 实战：把信号中的异常值截断
signal = np.array([1.2, 0.8, 5.0, 0.3, 4.8, 0.9])  # 5.0 和 4.8 是尖峰
signal[signal > 3.0] = 3.0   # 截断到 3.0
print(signal)     # [1.2 0.8 3.  0.3 3.  0.9]
```

---
## 2.4 数组运算

### 2.4.1 逐元素运算（对应 MATLAB 的 `.` 运算符）

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])
# 逐元素四则运算
a + b       # array([11, 22, 33, 44])
a - b       # array([ -9, -18, -27, -36])
a * b       # array([ 10,  40,  90, 160])   ← 逐元素，不是矩阵乘法！
a / b       # array([0.1, 0.1, 0.1, 0.1])
a ** 2      # array([ 1,  4,  9, 16])       ← 逐元素平方
np.sqrt(a)  # array([1.   , 1.414, 1.732, 2.   ])
# 逐元素比较
a > 2       # array([False, False,  True,  True])
```

> **MATLAB 对应**：Python 中 `*` 就是 MATLAB 的 `.*`（逐元素乘），`@` 才是 MATLAB 的 `*`（矩阵乘）。

### 2.4.2 矩阵运算

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
# 矩阵乘法
# MATLAB: A * B
A @ B            # array([[19, 22], [43, 50]])
# 等价写法
A.dot(B)         # 同上
np.matmul(A, B)  # 同上
# 转置
A.T              # array([[1, 3], [2, 4]])
# 共轭转置（处理复数信号时用）
C = np.array([[1+2j, 3+4j]])
C.conj().T       # 共轭转置，对应 MATLAB 的 C'
```
### 2.4.3 广播机制（Broadcasting）

这是 NumPy 比 MATLAB 强大的地方——**不同形状的数组也能运算**：
```python
# 信号加偏置
signal = np.array([1.0, 2.0, 3.0, 4.0])   # shape (4,)
bias = 0.5                                   # 标量
signal + bias   # array([1.5, 2.5, 3.5, 4.5])
# bias 自动"广播"到和 signal 一样的形状
# 2D 示例：每行加不同的偏置
matrix = np.zeros((3, 4))       # shape (3, 4)
row_offsets = np.array([1, 2, 3]).reshape(3, 1)  # shape (3, 1)
matrix + row_offsets
# row_offsets 自动从 (3,1) 广播到 (3,4)
# array([[1., 1., 1., 1.],
#        [2., 2., 2., 2.],
#        [3., 3., 3., 3.]])
# 信号处理实战：对多通道信号去均值
# 3 个通道，每个通道 1000 个采样点
data = rng.random((3, 1000))
mean_per_channel = data.mean(axis=1, keepdims=True)  # shape (3, 1)
centered = data - mean_per_channel  # 广播！每个通道减去自己的均值
```
广播规则图解：
```
shape (3, 4)  +  shape (3, 1)  →  shape (3, 4)  ✅
shape (3, 4)  +  shape (4,)    →  shape (3, 4)  ✅
shape (3, 4)  +  shape (3,)    →  ❌ 报错！维度不匹配
口诀：从右向左对齐，维度相等或其中一个为1即可广播
```

---
## 2.5 常用数学函数

```python
x = np.array([0, np.pi/6, np.pi/4, np.pi/3, np.pi/2])
# ─── 三角函数 ───
np.sin(x)        # 正弦
np.cos(x)        # 余弦
np.tan(x)        # 正切
np.arcsin(x)     # 反正弦（输入需在 [-1, 1]）
# ─── 指数与对数 ───
np.exp(x)        # e^x
np.log(x)        # ln(x)     ← 注意：MATLAB 的 log 也是自然对数
np.log10(x)      # log10(x)  ← MATLAB 的 log10
np.log2(x)       # log2(x)   ← MATLAB 的 log2
# ─── 取整 ───
a = np.array([-2.7, -1.3, 0.5, 1.3, 2.7])
np.round(a)      # [-3., -1.,  0.,  1.,  3.]   四舍五入
np.floor(a)      # [-3., -2.,  0.,  1.,  2.]   向下取整
np.ceil(a)       # [-2., -1.,  1.,  2.,  3.]   向上取整
np.abs(a)        # [ 2.7,  1.3,  0.5,  1.3,  2.7]  绝对值
# ─── 信号处理常用 ───
np.angle(1+1j)   # 0.7854  复数相位角（弧度）
np.abs(3+4j)     # 5.0     复数模
np.real(3+4j)    # 3.0     实部
np.imag(3+4j)    # 4.0     虚部
np.conj(3+4j)    # (3-4j)  共轭
# ─── dB 换算（极常用！）───
def mag2db(mag):
    """幅度转 dB"""
    return 20 * np.log10(np.maximum(mag, 1e-20))
def db2mag(db):
    """dB 转幅度"""
    return 10 ** (db / 20)
def pow2db(pwr):
    """功率转 dB"""
    return 10 * np.log10(np.maximum(pwr, 1e-20))
```

---
## 2.6 聚合运算（统计量）

```python
a = np.array([3, 1, 4, 1, 5, 9, 2, 6])
# ─── 基本统计 ───
a.sum()          # 31     求和
a.mean()         # 3.875  均值
a.std()          # 2.56   标准差（默认 ddof=0，总体标准差）
a.std(ddof=1)    # 2.74   样本标准差（MATLAB 默认 ddof=1）
a.var()          # 6.56   方差
a.min()          # 1
a.max()          # 9
a.argmin()       # 1      最小值的索引
a.argmax()       # 5      最大值的索引
a.median()       # np.median(a) = 3.5  ← 中位数是函数不是方法
# ─── 累积运算 ───
a.cumsum()       # [ 3  4  8  9 14 23 25 31]  累积和
a.cumprod()      # 累积积
# ─── 沿指定轴运算（2D 及以上）───
B = np.array([[1, 2, 3],
              [4, 5, 6]])
B.sum(axis=0)    # array([5, 7, 9])     沿行求和 → 每列的和
B.sum(axis=1)    # array([ 6, 15])      沿列求和 → 每行的和
B.mean(axis=0)   # array([2.5, 3.5, 4.5])  每列的均值
# 记忆：axis=i 意味着"消除第 i 个维度"
# B.shape = (2, 3)
# axis=0 → 结果 shape=(3,)  消除行
# axis=1 → 结果 shape=(2,)  消除列
```

---
## 2.7 形状操作

```python
a = np.arange(12)   # [0 1 2 3 4 5 6 7 8 9 10 11]
# ─── reshape ───
b = a.reshape(3, 4)
# array([[ 0,  1,  2,  3],
#        [ 4,  5,  6,  7],
#        [ 8,  9, 10, 11]])
# 用 -1 自动推算某个维度
c = a.reshape(3, -1)   # 等价于 reshape(3, 4)
d = a.reshape(-1, 6)   # 等价于 reshape(2, 6)
# ─── 转置 ───
b.T            # 4×3 矩阵
b.transpose()  # 同上
# ─── 展平 ───
b.flatten()    # [0 1 2 ... 11]  返回副本
b.ravel()      # [0 1 2 ... 11]  返回视图（更省内存）
# ─── 堆叠（信号处理中拼接多通道信号）───
ch1 = np.array([1, 2, 3])
ch2 = np.array([4, 5, 6])
ch3 = np.array([7, 8, 9])
np.vstack([ch1, ch2, ch3])   # 垂直堆叠 → 3×3
# array([[1, 2, 3],
#        [4, 5, 6],
#        [7, 8, 9]])
np.hstack([ch1, ch2, ch3])   # 水平堆叠 → 1×9
# array([1, 2, 3, 4, 5, 6, 7, 8, 9])
np.stack([ch1, ch2, ch3], axis=0)  # 新增维度堆叠
# shape (3, 3)，三个通道
np.stack([ch1, ch2, ch3], axis=1)  # 沿第二维堆叠
# shape (3, 3)，每行是一个时刻的三个通道值
```

---
## 2.8 NumPy 中的 FFT（最关键的信号处理功能）

```python
import numpy as np
import matplotlib.pyplot as plt
# ─── 生成测试信号 ───
fs = 1000                        # 采样率 1000 Hz
T = 1.0                          # 信号时长 1 秒
N = int(fs * T)                  # 采样点数
t = np.arange(N) / fs            # 时间向量
# 信号 = 50Hz(幅度1) + 120Hz(幅度0.5) + 噪声
x = 1.0 * np.sin(2 * np.pi * 50 * t) + \
    0.5 * np.sin(2 * np.pi * 120 * t) + \
    0.2 * rng.standard_normal(N)
# ─── FFT 计算 ───
X = np.fft.fft(x)               # 复数频谱
# 频率轴
freqs = np.fft.fftfreq(N, d=1/fs)
# 幅度谱（归一化）
mag = np.abs(X) / N              # 除以 N 得到真实幅度
# ─── 只取正频率部分 ───
pos_mask = freqs >= 0
freqs_pos = freqs[pos_mask]
mag_pos = mag[pos_mask]
# 单边谱幅度需要乘以 2（直流分量除外）
mag_pos[1:] *= 2
# 打印峰值频率
peak_idx = np.argmax(mag_pos[1:]) + 1  # 跳过直流
print(f"峰值频率: {freqs_pos[peak_idx]:.1f} Hz, 幅度: {mag_pos[peak_idx]:.4f}")
# 峰值频率: 50.0 Hz, 幅度: ~1.0 ✅
# ─── IFFT：从频域回到时域 ───
x_reconstructed = np.fft.ifft(X).real   # 取实部（消除微小虚部误差）
# 验证重构误差
print(f"重构误差: {np.max(np.abs(x - x_reconstructed)):.2e}")
# 重构误差: ~1e-15  ✅
```
### FFT 完整对照表

| 操作            | MATLAB              | Python (NumPy)                |
| --------------- | ------------------- | ----------------------------- |
| FFT             | `X = fft(x)`        | `X = np.fft.fft(x)`           |
| IFFT            | `x = ifft(X)`       | `x = np.fft.ifft(X)`          |
| 频率轴          | `f = (0:N-1)*fs/N`  | `f = np.fft.fftfreq(N, 1/fs)` |
| 实数FFT（优化） | `X = fft(x, N/2+1)` | `X = np.fft.rfft(x)`          |
| 实数IFFT        | `x = ifft(X, N)`    | `x = np.fft.irfft(X)`         |
| FFT 长度补零    | `X = fft(x, 2048)`  | `X = np.fft.fft(x, 2048)`     |
| FFT 移位        | `fftshift(X)`       | `np.fft.fftshift(X)`          |

---
## 2.9 NumPy 信号处理实战案例

### 案例：生成调频信号（Chirp）并计算频谱图

```python
import numpy as np
fs = 4000                        # 采样率
T = 2.0                          # 时长 2 秒
t = np.arange(int(fs * T)) / fs  # 时间向量
# 线性调频：频率从 100Hz 扫到 800Hz
f0, f1 = 100, 800
freq_inst = f0 + (f1 - f0) * t / T    # 瞬时频率
phase = 2 * np.pi * np.cumsum(freq_inst) / fs   # 相位 = 频率积分
chirp = np.sin(phase)
# 简易频谱图（短时傅里叶变换的手动实现）
def simple_spectrogram(signal, fs, window_size=256, hop=128):
    """手动实现 STFT 频谱图"""
    N = len(signal)
    n_fft = window_size
    window = np.hanning(window_size)
    
    n_frames = (N - window_size) // hop + 1
    S = np.zeros((n_fft // 2 + 1, n_frames))
    
    for i in range(n_frames):
        start = i * hop
        frame = signal[start:start + window_size] * window
        X = np.fft.rfft(frame, n_fft)
        S[:, i] = np.abs(X) / window_size
    
    freqs = np.fft.rfftfreq(n_fft, 1/fs)
    times = np.arange(n_frames) * hop / fs
    return freqs, times, S
freqs, times, S = simple_spectrogram(chirp, fs)
print(f"频率轴: {freqs.shape}, 时间轴: {times.shape}, 频谱图: {S.shape}")
print(f"频率范围: [{freqs[0]:.1f}, {freqs[-1]:.1f}] Hz")
```

---
## 2.10 NumPy 性能技巧

```python
# ─── 规则1：避免 Python 循环，用向量化 ───
# ❌ 慢：Python 循环
x = np.zeros(1000000)
for i in range(1000000):
    x[i] = np.sin(2 * np.pi * 50 * i / 1000)
# ✅ 快：向量化运算
i = np.arange(1000000)
x = np.sin(2 * np.pi * 50 * i / 1000)   # 快 100 倍以上！
# ─── 规则2：预分配数组 ───
# ❌ 慢：动态追加
result = []
for i in range(100000):
    result.append(np.sin(i))
result = np.array(result)
# ✅ 快：预分配
result = np.empty(100000)
for i in range(100000):
    result[i] = np.sin(i)
# ✅✅ 最快：直接向量化
result = np.sin(np.arange(100000))
# ─── 规则3：用 np.where 替代 if/else ───
# ❌ 慢：逐元素判断
x = np.array([-2, 5, -1, 3, -4])
y = np.empty_like(x)
for i in range(len(x)):
    if x[i] > 0:
        y[i] = x[i]
    else:
        y[i] = 0
# ✅ 快：向量化
y = np.where(x > 0, x, 0)    # array([0, 5, 0, 3, 0])
# 等价写法
y = np.clip(x, 0, None)      # 同上（截断到 [0, ∞)）
```

---
## 2.11 本章速查表

| 需求         | 代码                                   |
| ------------ | -------------------------------------- |
| 创建等距向量 | `np.linspace(0, 1, 100)`               |
| 创建时间向量 | `t = np.arange(N) / fs`                |
| 生成正弦波   | `np.sin(2 * np.pi * f * t)`            |
| 添加高斯噪声 | `x + sigma * rng.standard_normal(N)`   |
| 计算FFT      | `X = np.fft.fft(x)`                    |
| 频率轴       | `f = np.fft.fftfreq(N, 1/fs)`          |
| 取正频率     | `f[f >= 0]`, `X[f >= 0]`               |
| 幅度谱(dB)   | `20 * np.log10(np.abs(X) / N + 1e-20)` |
| IFFT         | `np.fft.ifft(X).real`                  |
| 逐元素乘     | `a * b`                                |
| 矩阵乘       | `A @ B`                                |
| 复数模       | `np.abs(z)`                            |
| 复数相位     | `np.angle(z)`                          |
| 共轭         | `np.conj(z)`                           |
| 卷积         | `np.convolve(a, b)`                    |
| 相关         | `np.correlate(a, b)`                   |

---
> **第二章结束。** 接下来是第三章：SciPy 信号处理核心工具箱，将覆盖滤波器设计（FIR/IIR）、频谱分析、重采样、窗函数等。回复"继续"获取下一章。

# 第三章：SciPy —— 信号处理核心工具箱
> SciPy = Scientific Python。在 NumPy 的基础上提供信号处理、优化、插值、线性代数等高级算法。对应 MATLAB 的 Signal Processing Toolbox + Optimization Toolbox + ...

---
## 3.1 SciPy 模块总览
```python
# SciPy 的结构（只导入需要的子模块）
from scipy import signal       # 信号处理 ← 本章重点
from scipy import fft          # FFT（比 numpy.fft 更快更完善）
from scipy import interpolate  # 插值
from scipy import optimize     # 优化
from scipy import io           # 文件 I/O（读写 .mat 文件！）
from scipy import linalg       # 线性代数
from scipy import stats        # 统计
```

---
## 3.2 读写 MATLAB 的 .mat 文件（迁移第一步！）

从 MATLAB 转 Python，第一步就是把数据搬过来：
```python
from scipy import io
import numpy as np
# ─── 读取 .mat 文件 ───
data = io.loadmat('signal_data.mat')
# 返回一个字典，key 是变量名，value 是 numpy 数组
print(data.keys())
# dict_keys(['__header__', '__version__', '__globals__', 'ecg_signal', 'fs', 'labels'])
ecg = data['ecg_signal']    # 取出变量
fs = data['fs'].item()      # .item() 把 1×1 数组转为标量
print(f"信号形状: {ecg.shape}, 采样率: {fs} Hz")
# ─── 保存为 .mat 文件（给 MATLAB 同事用）───
processed = np.random.randn(1000, 1)
io.savemat('processed.mat', {
    'signal': processed,
    'fs': fs,
    'method': 'bandpass'    # 字符串也能保存
})
# ─── 读取 MATLAB v7.3 格式（HDF5）───
# 如果 loadmat 报错 "Not a valid MATLAB 5.0 file"，说明是 v7.3 格式
import h5py
with h5py.File('large_data.mat', 'r') as f:
    print(list(f.keys()))
    data = f['my_signal'][:]
```

> **重要提示**：MATLAB 的 `.mat` 文件中，向量默认是列向量 `(N, 1)`，不是行向量 `(N,)`。使用时经常需要 `.flatten()` 或 `.squeeze()`。

```python
# MATLAB 列向量 → Python 一维数组
ecg = data['ecg_signal'].flatten()   # 或 .squeeze() 或 .ravel()
print(ecg.shape)  # (1000,) ✅ 而不是 (1000, 1)
```

---
## 3.3 窗函数
窗函数是信号处理的基石，几乎所有频谱分析和滤波器设计都离不开它。
### 3.3.1 常用窗函数一览
```python
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt
N = 51  # 窗长度
# ─── 矩形窗（无窗）───
rect = signal.windows.boxcar(N)
# ─── 汉宁窗（最常用）───
hann = signal.windows.hann(N)
# ─── 汉明窗 ───
hamm = signal.windows.hamming(N)
# ─── 布莱克曼窗（旁瓣更低）───
blackman = signal.windows.blackman(N)
# ─── 平顶窗（幅度精度最高）───
flattop = signal.windows.flattop(N)
# ─── 凯撒窗（可调参数 β）───
kaiser = signal.windows.kaiser(N, beta=8)   # β 越大旁瓣越低
# ─── 高斯窗 ───
gauss = signal.windows.gaussian(N, std=7)
# ─── 可调参数的 DPSS 窗（多锥法用）───
dpss = signal.windows.dpss(N, NW=3, Kmax=5)  # 返回多个窗
```

### 3.3.2 窗函数对比可视化
```python
N = 256
windows = {
    '矩形窗 (Rect)':  signal.windows.boxcar(N),
    '汉宁窗 (Hann)':   signal.windows.hann(N),
    '汉明窗 (Hamming)': signal.windows.hamming(N),
    '布莱克曼窗':       signal.windows.blackman(N),
    '凯撒窗 (β=8)':    signal.windows.kaiser(N, beta=8),
}
fig, axes = plt.subplots(1, 2, figsize=(14, 5))
# 左图：时域波形
for name, w in windows.items():
    axes[0].plot(w, label=name)
axes[0].set_title('窗函数时域波形')
axes[0].legend(fontsize=8)
axes[0].grid(True, alpha=0.3)
# 右图：频域特性（归一化幅度 dB）
for name, w in windows.items():
    # 补零到 4096 点提高频率分辨率
    W = np.fft.fft(w, 4096)
    W_mag = 20 * np.log10(np.abs(W[:2048]) / np.max(np.abs(W)) + 1e-20)
    freqs = np.linspace(0, 0.5, 2048)  # 归一化频率
    axes[1].plot(freqs, W_mag, label=name)
axes[1].set_title('窗函数频域特性')
axes[1].set_ylim([-120, 3])
axes[1].set_xlabel('归一化频率 (×π rad/sample)')
axes[1].set_ylabel('幅度 (dB)')
axes[1].legend(fontsize=8)
axes[1].grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('windows_comparison.png', dpi=150)
plt.show()
```

### 3.3.3 窗函数选择指南
```
┌──────────────────────────────────────────────────────────────┐
│                    窗函数选择指南                             │
├───────────────┬──────────────┬──────────────┬────────────────┤
│ 窗函数        │ 主瓣宽度     │ 旁瓣衰减     │ 适用场景       │
├───────────────┼──────────────┼──────────────┼────────────────┤
│ 矩形窗        │ 最窄 (1 bin) │ -13 dB       │ 频率分辨率优先 │
│ 汉宁窗        │ 1.5 bin      │ -31 dB       │ 通用频谱分析   │
│ 汉明窗        │ 1.5 bin      │ -42 dB       │ 语音处理       │
│ 布莱克曼窗    │ 2.0 bin      │ -58 dB       │ 低旁瓣需求     │
│ 平顶窗        │ 3.8 bin      │ -44 dB       │ 幅度精度优先   │
│ 凯撒窗(β可调) │ 可调         │ 可调         │ 灵活折中       │
└───────────────┴──────────────┴──────────────┴────────────────┘
MATLAB 对照：
  rectwin(N)   → signal.windows.boxcar(N)
  hann(N)      → signal.windows.hann(N)
  hamming(N)   → signal.windows.hamming(N)
  blackman(N)  → signal.windows.blackman(N)
  kaiser(N,β)  → signal.windows.kaiser(N, beta=β)
  flattopwin(N)→ signal.windows.flattop(N)
```

---
## 3.4 滤波器设计
这是 SciPy 信号处理模块最核心的功能。
### 3.4.1 IIR 滤波器设计
```python
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt
fs = 1000  # 采样率
# ═══════════════════════════════════════════
#  方法1：指定阶数的滤波器（类似 MATLAB 的 butter, cheby1...）
# ═══════════════════════════════════════════
# ─── 巴特沃斯低通 ───
# MATLAB: [b, a] = butter(4, 100/(fs/2))
b, a = signal.butter(N=4, Wn=100, btype='low', fs=fs)
# ─── 巴特沃斯带通 ───
# MATLAB: [b, a] = butter(4, [20 200]/(fs/2))
b, a = signal.butter(N=4, Wn=[20, 200], btype='band', fs=fs)
# ─── 切比雪夫 I 型 ───
# MATLAB: [b, a] = cheby1(4, 1, 100/(fs/2))
b, a = signal.cheby1(N=4, rp=1, Wn=100, btype='low', fs=fs)   # rp=通带波纹(dB)
# ─── 切比雪夫 II 型 ───
# MATLAB: [b, a] = cheby2(4, 40, 100/(fs/2))
b, a = signal.cheby2(N=4, rs=40, Wn=100, btype='low', fs=fs)  # rs=阻带衰减(dB)
# ─── 椭圆滤波器 ───
# MATLAB: [b, a] = ellip(4, 1, 40, 100/(fs/2))
b, a = signal.ellip(N=4, rp=1, rs=40, Wn=100, btype='low', fs=fs)
# ═══════════════════════════════════════════
#  方法2：指定指标的滤波器（推荐！自动确定阶数）
# ═══════════════════════════════════════════
# 巴特沃斯：指定通带和阻带指标
# MATLAB: [N, Wn] = buttord([20 200]/(fs/2), [10 300]/(fs/2), 3, 40)
N_ord, Wn = signal.buttord(
    wp=[20, 200],    # 通带截止频率 (Hz)
    ws=[10, 300],    # 阻带起始频率 (Hz)
    gpass=3,         # 通带最大衰减 (dB)
    gstop=40,        # 阻带最小衰减 (dB)
    fs=fs
)
print(f"自动计算阶数: {N_ord}, 截止频率: {Wn}")
# 然后用计算出的阶数设计滤波器
b, a = signal.butter(N_ord, Wn, btype='band', fs=fs)
# 同理还有 cheb1ord, cheb2ord, ellipord
```

> **关键参数说明**：
> - `Wn`：截止频率。当 `fs` 参数指定时，单位是 Hz；否则是归一化频率（0~1，1 = fs/2）。
> - `btype`：`'low'`(低通), `'high'`(高通), `'band'`(带通), `'bandstop'`(带阻)。
> - 带通/带阻时，`Wn` 是 `[低频, 高频]` 列表。

### 3.4.2 FIR 滤波器设计

```python
# ═══════════════════════════════════════════
#  方法1：firwin（最常用，类似 MATLAB 的 fir1）
# ═══════════════════════════════════════════
# 低通 FIR，101 阶
# MATLAB: b = fir1(100, 100/(fs/2))
b = signal.firwin(numtaps=101, cutoff=100, fs=fs)
# 带通 FIR
# MATLAB: b = fir1(100, [20 200]/(fs/2))
b = signal.firwin(numtaps=101, cutoff=[20, 200], pass_zero=False, fs=fs)
# 高通 FIR
b = signal.firwin(numtaps=101, cutoff=50, pass_zero=False, fs=fs)
# 带阻 FIR
b = signal.firwin(numtaps=101, cutoff=[45, 55], pass_zero=True, fs=fs)
# 指定窗函数
b = signal.firwin(numtaps=101, cutoff=100, window='hamming', fs=fs)
b = signal.firwin(numtaps=101, cutoff=100, window=('kaiser', 8), fs=fs)
# ═══════════════════════════════════════════
#  方法2：firwin2（指定任意频率-增益对）
# ═══════════════════════════════════════════
# 设计一个在 0-100Hz 增益为1，100-150Hz 线性衰减到0，150-500Hz 增益为0 的滤波器
freq_points = [0, 100, 150, 500]       # 频率点 (Hz)
gain_points = [1, 1,   0,   0  ]       # 对应增益
b = signal.firwin2(numtaps=101, freq=freq_points, gain=gain_points, fs=fs)
# ═══════════════════════════════════════════
#  方法3：最小二乘法 FIR（firls，指定指标）
# ═══════════════════════════════════════════
# MATLAB: b = firls(100, [0 20 25 200 220 500], [1 1 0 0 0 0])
b = signal.firls(
    numtaps=101,
    bands=[0, 20, 25, 200, 220, 500],  # 频率带边界
    desired=[1, 1, 0, 0, 0, 0],        # 每个带内的期望增益
    fs=fs
)
```

### 3.4.3 IIR 滤波器的二阶级联形式（SOS）— 重要！

```python
# ⚠️ 警告：高阶 IIR 滤波器直接用 b, a 形式会不稳定！
# 解决方案：使用 SOS（Second-Order Sections）形式
# ❌ 不推荐：b, a 形式（高阶时数值不稳定）
b, a = signal.butter(10, 100, fs=fs)
# ✅ 推荐：SOS 形式
sos = signal.butter(10, 100, fs=fs, output='sos')
# sos 是一个 (n_sections, 6) 的数组
# 每行 = [b0, b1, b2, a0, a1, a2]，代表一个二阶节
print(f"SOS 形状: {sos.shape}")  # (5, 6) → 5 个二阶节
# ─── 所有 IIR 设计函数都支持 output='sos' ───
sos = signal.butter(N=8, Wn=[20, 200], btype='band', fs=fs, output='sos')
sos = signal.cheby1(N=6, rp=1, Wn=100, fs=fs, output='sos')
sos = signal.ellip(N=6, rp=1, rs=40, Wn=100, fs=fs, output='sos')
# ─── b,a 与 sos 的稳定性对比 ───
# 16 阶巴特沃斯
b, a = signal.butter(16, 100, fs=fs)
sos = signal.butter(16, 100, fs=fs, output='sos')
# 检查极点是否在单位圆内
z_ba, p_ba, k_ba = signal.tf2zpk(b, a)
z_sos, p_sos, k_sos = signal.sos2zpk(sos)
print(f"b,a 形式最大极点半径: {np.max(np.abs(p_ba)):.6f}")   # 可能 > 1（不稳定！）
print(f"SOS 形式最大极点半径: {np.max(np.abs(p_sos)):.6f}")   # 总是 < 1（稳定）
```

---
## 3.5 滤波器应用（实际滤波）
### 3.5.1 一次性滤波（离线处理）

```python
import numpy as np
from scipy import signal
fs = 1000
t = np.arange(0, 5, 1/fs)
# 生成含噪声的信号：10Hz 信号 + 50Hz 工频干扰 + 200Hz 高频噪声
x = np.sin(2 * np.pi * 10 * t) + \
    0.5 * np.sin(2 * np.pi * 50 * t) + \
    0.3 * np.random.randn(len(t))
# ═══════════════════════════════════════════
#  方法1：lfilter（类似 MATLAB 的 filter）
# ═══════════════════════════════════════════
# IIR 带通滤波器：保留 5-30Hz
b, a = signal.butter(4, [5, 30], btype='band', fs=fs)
y_lfilter = signal.lfilter(b, a, x)
# SOS 版本（推荐）
sos = signal.butter(4, [5, 30], btype='band', fs=fs, output='sos')
y_sosfilt = signal.sosfilt(sos, x)
# FIR 滤波器
b_fir = signal.firwin(101, [5, 30], pass_zero=False, fs=fs)
y_fir = signal.lfilter(b_fir, [1.0], x)  # FIR 的 a = [1.0]
# ═══════════════════════════════════════════
#  方法2：filtfilt（零相位滤波，强烈推荐！）
# ═══════════════════════════════════════════
# lfilter 有相位延迟，filtfilt 正反向滤波消除相位延迟
# MATLAB: y = filtfilt(b, a, x)
y_filtfilt = signal.filtfilt(b, a, x)
# SOS 版本
y_sosfiltfilt = signal.sosfiltfilt(sos, x)
```

### 3.5.2 lfilter vs filtfilt 对比（关键！）
```python
fig, axes = plt.subplots(2, 1, figsize=(12, 8), sharex=True)
# 原始信号
axes[0].plot(t[:500], x[:500], 'gray', alpha=0.5, label='原始信号')
axes[0].plot(t[:500], y_lfilter[:500], 'r', linewidth=1.5, label='lfilter (有相位延迟)')
axes[0].plot(t[:500], y_sosfiltfilt[:500], 'b', linewidth=1.5, label='sosfiltfilt (零相位)')
axes[0].set_title('时域对比')
axes[0].legend()
axes[0].grid(True, alpha=0.3)
# 相位差异
axes[1].plot(t[:500], np.sin(2 * np.pi * 10 * t[:500]), 'k--', label='理想10Hz信号', alpha=0.5)
axes[1].plot(t[:500], y_lfilter[:500], 'r', label='lfilter')
axes[1].plot(t[:500], y_sosfiltfilt[:500], 'b', label='filtfilt')
axes[1].set_title('与理想信号的相位对比')
axes[1].legend()
axes[1].grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('lfilter_vs_filtfilt.png', dpi=150)
plt.show()
```

```
┌──────────────────────────────────────────────────────────────┐
│            lfilter vs filtfilt 对照表                        │
├────────────────┬─────────────────────┬───────────────────────┤
│ 特性           │ lfilter / sosfilt   │ filtfilt / sosfiltfilt│
├────────────────┼─────────────────────┼───────────────────────┤
│ 相位           │ 有延迟（因果）       │ 零相位（非因果）       │
│ 滤波次数       │ 1 次（正向）         │ 2 次（正向+反向）      │
│ 滤波器阶数等效 │ N 阶                 │ 2N 阶（幅频特性更陡）  │
│ 实时性         │ ✅ 可用于实时         │ ❌ 必须有完整信号      │
│ 边界效应       │ 仅开头               │ 开头和结尾都有         │
│ MATLAB 对应    │ filter(b,a,x)       │ filtfilt(b,a,x)       │
│ 推荐场景       │ 实时流式处理         │ 离线分析              │
└────────────────┴─────────────────────┴───────────────────────┘
```

### 3.5.3 实时流式滤波（逐样本处理）
```python
class RealtimeIIRFilter:
    """
    实时 IIR 滤波器（逐样本处理）
    对应 MATLAB: filter(b, a, x) 在循环中逐样本调用
    """
    def __init__(self, b, a):
        self.b = np.asarray(b)
        self.a = np.asarray(a)
        self.order = len(b) - 1
        # 状态缓冲区（保存历史输入和输出）
        self.z = signal.lfilter_zi(b, a) * 0  # 初始化为零
        # 也可以初始化为信号均值以减少瞬态：
        # self.z = signal.lfilter_zi(b, a) * initial_value
    def process_sample(self, x):
        """处理单个样本"""
        y, self.z = signal.lfilter(self.b, self.a, [x], zi=self.z)
        return y[0]
    def process_block(self, block):
        """处理一个数据块"""
        y, self.z = signal.lfilter(self.b, self.a, block, zi=self.z)
        return y
# 使用示例
fs = 1000
b, a = signal.butter(4, 50, btype='low', fs=fs)
rt_filter = RealtimeIIRFilter(b, a)
# 模拟逐样本到达
output_samples = []
for sample in x:   # x 是输入信号
    y = rt_filter.process_sample(sample)
    output_samples.append(y)
# 或按块处理
rt_filter2 = RealtimeIIRFilter(b, a)
output_blocks = []
block_size = 100
for i in range(0, len(x), block_size):
    block = x[i:i+block_size]
    y_block = rt_filter2.process_block(block)
    output_blocks.append(y_block)
y_stream = np.concatenate(output_blocks)
```

---
## 3.6 频率响应分析
### 3.6.1 滤波器频率响应
```python
# ═══════════════════════════════════════════
#  b, a 形式
# ═══════════════════════════════════════════
# MATLAB: [H, w] = freqz(b, a, 2048, fs)
w, H = signal.freqz(b, a, worN=2048, fs=fs)
# 幅度响应（dB）
mag_db = 20 * np.log10(np.abs(H) + 1e-20)
# 相位响应
phase = np.angle(H)
# ═══════════════════════════════════════════
#  SOS 形式
# ═══════════════════════════════════════════
w, H = signal.sosfreqz(sos, worN=2048, fs=fs)
# ═══════════════════════════════════════════
#  FIR 滤波器
# ═══════════════════════════════════════════
b_fir = signal.firwin(101, 50, fs=fs)
w, H = signal.freqz(b_fir, worN=2048, fs=fs)
# ═══════════════════════════════════════════
#  绘制完整的频率响应图
# ═══════════════════════════════════════════
fig, axes = plt.subplots(2, 1, figsize=(10, 8), sharex=True)
# 幅度响应
axes[0].plot(w, mag_db, 'b', linewidth=1.5)
axes[0].axvline(50, color='r', linestyle='--', alpha=0.5, label='截止频率')
axes[0].axhline(-3, color='g', linestyle='--', alpha=0.5, label='-3 dB')
axes[0].set_ylabel('幅度 (dB)')
axes[0].set_title('幅度响应')
axes[0].legend()
axes[0].grid(True, alpha=0.3)
axes[0].set_ylim([-80, 5])
# 相位响应
axes[1].plot(w, np.degrees(phase), 'b', linewidth=1.5)
axes[1].set_xlabel('频率 (Hz)')
axes[1].set_ylabel('相位 (°)')
axes[1].set_title('相位响应')
axes[1].grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('freq_response.png', dpi=150)
plt.show()
```

### 3.6.2 群延迟
```python
# MATLAB: [gd, w] = grpdelay(b, a, 2048, fs)
w, gd = signal.group_delay((b, a), w=2048, fs=fs)
plt.figure(figsize=(10, 4))
plt.plot(w, gd, 'b')
plt.xlabel('频率')
plt.ylabel('群延迟')
plt.title('群延迟')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
# FIR 滤波器的群延迟 = (N-1)/2（常数），即线性相位
w_fir, gd_fir = signal.group_delay((b_fir, [1.0]), w=2048, fs=fs)
print(f"FIR 群延迟理论值: {(len(b_fir)-1)/2:.1f} 样本")
print(f"FIR 群延迟实测值: {np.mean(gd_fir[w_fir < 30]):.1f} 样本")
```

### 3.6.3 零极点图
```python
# ═══════════════════════════════════════════
#  从 b, a 获取零极点
# ═══════════════════════════════════════════
z, p, k = signal.tf2zpk(b, a)
# ═══════════════════════════════════════════
#  从 SOS 获取零极点
# ═══════════════════════════════════════════
z, p, k = signal.sos2zpk(sos)
# 绘制零极点图
fig, ax = plt.subplots(figsize=(6, 6))
# 单位圆
theta = np.linspace(0, 2*np.pi, 200)
ax.plot(np.cos(theta), np.sin(theta), 'k-', linewidth=1)
# 零点（圆圈）和极点（叉号）
ax.scatter(z.real, z.imag, marker='o', facecolors='none', 
           edgecolors='b', s=80, linewidths=2, label='零点')
ax.scatter(p.real, p.imag, marker='x', 
           color='r', s=80, linewidths=2, label='极点')
ax.set_aspect('equal')
ax.set_xlabel('实部')
ax.set_ylabel('虚部')
ax.set_title('零极点图')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('pole_zero.png', dpi=150)
plt.show()
```

---
## 3.7 频谱分析
### 3.7.1 功率谱密度（PSD）

```python
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt
fs = 1000
t = np.arange(0, 10, 1/fs)
# 生成测试信号
rng = np.random.default_rng(42)
x = np.sin(2 * np.pi * 50 * t) + \
    0.5 * np.sin(2 * np.pi * 120 * t) + \
    0.3 * rng.standard_normal(len(t))
# ═══════════════════════════════════════════
#  方法1：Welch 法（最常用）
# ═══════════════════════════════════════════
# MATLAB: [pxx, f] = pwelch(x, hamming(256), 128, 256, fs)
f_welch, Pxx_welch = signal.welch(
    x,
    fs=fs,
    window='hamming',    # 窗函数
    nperseg=256,         # 每段长度
    noverlap=128,        # 重叠长度（默认 50%）
    nfft=256,            # FFT 长度
    scaling='density'    # 'density': V²/Hz, 'spectrum': V²
)
plt.figure(figsize=(10, 4))
plt.semilogy(f_welch, Pxx_welch, 'b')
plt.xlabel('频率')
plt.ylabel('PSD (V^2/Hz)')
plt.title('Welch 功率谱密度')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
# ═══════════════════════════════════════════
#  方法2：周期图法（Periodogram）
# ═══════════════════════════════════════════
# MATLAB: [pxx, f] = periodogram(x, hamming(length(x)), [], fs)
f_per, Pxx_per = signal.periodogram(
    x,
    fs=fs,
    window='hamming',
    scaling='density'
)
# ═══════════════════════════════════════════
#  方法3：多锥法（Multitaper）
# ═══════════════════════════════════════════
f_mt, Pxx_mt = signal.welch(x, fs=fs, nperseg=512)  # 简化
# SciPy 没有直接的 multitaper，但可以这样手动实现：
def multitaper_psd(x, fs, NW=3, k=None, nfft=None):
    """多锥法功率谱估计"""
    N = len(x)
    if nfft is None:
        nfft = N
    if k is None:
        k = int(2 * NW - 1)  # Slepian 序列数
    
    # 生成 DPSS 窗
    dpss_windows = signal.windows.dpss(N, NW, k)
    
    psd_accum = np.zeros(nfft // 2 + 1)
    for w in dpss_windows:
        X = np.fft.rfft(x * w, n=nfft)
        psd_accum += np.abs(X) ** 2
    
    freqs = np.fft.rfftfreq(nfft, 1/fs)
    psd = psd_accum / (k * N * fs)
    return freqs, psd

f_mt, Pxx_mt = multitaper_psd(x, fs, NW=3)
```

### 3.7.2 短时傅里叶变换（STFT）
```python
# ═══════════════════════════════════════════
#  SciPy 内置 STFT
# ═══════════════════════════════════════════
# MATLAB: [S, F, T] = spectrogram(x, hamming(256), 128, 256, fs)
f_stft, t_stft, Zxx = signal.stft(
    x,
    fs=fs,
    window='hann',
    nperseg=256,
    noverlap=128,
    nfft=256
)
# Zxx 是复数 STFT 矩阵，shape = (n_freqs, n_times)
print(f"频率轴: {f_stft.shape}, 时间轴: {t_stft.shape}, STFT: {Zxx.shape}")
# 绘制频谱图
plt.figure(figsize=(10, 6))
plt.pcolormesh(t_stft, f_stft, np.abs(Zxx), shading='gouraud', cmap='viridis')
plt.colorbar(label='幅度')
plt.xlabel('时间')
plt.ylabel('频率')
plt.title('STFT 频谱图')
plt.ylim([0, 200])
plt.tight_layout()
plt.show()
# ═══════════════════════════════════════════
#  逆 STFT（从频域恢复时域信号）
# ═══════════════════════════════════════════
_, x_reconstructed = signal.istft(Zxx, fs=fs)
min_len = min(len(x), len(x_reconstructed))
reconstruction_error = np.max(np.abs(x[:min_len] - x_reconstructed[:min_len]))
print(f"ISTFT 重构误差: {reconstruction_error:.2e}")
```

### 3.7.3 三种 PSD 方法对比
```python
fig, axes = plt.subplots(3, 1, figsize=(10, 9), sharex=True)
axes[0].semilogy(f_per, Pxx_per, 'b', linewidth=0.5)
axes[0].set_title('周期图法 (方差大)')
axes[1].semilogy(f_welch, Pxx_welch, 'r', linewidth=1)
axes[1].set_title('Welch 法 (推荐，方差小)')
axes[2].semilogy(f_mt, Pxx_mt, 'g', linewidth=1)
axes[2].set_title('多锥法 (平滑，低方差)')
for ax in axes:
    ax.set_ylabel('PSD (V²/Hz)')
    ax.grid(True, alpha=0.3)
    ax.set_xlim([0, 200])
axes[2].set_xlabel('频率')
plt.tight_layout()
plt.savefig('psd_comparison.png', dpi=150)
plt.show()
```

---
## 3.8 重采样

```python
# ═══════════════════════════════════════════
#  上采样 / 下采样 / 任意倍率重采样
# ═══════════════════════════════════════════
fs_orig = 1000
t_orig = np.arange(0, 1, 1/fs_orig)
x_orig = np.sin(2 * np.pi * 10 * t_orig)
# ─── resample：基于 FFT 的高质量重采样 ───
# MATLAB: y = resample(x, P, Q)  (P/Q 倍)
fs_new = 1500
ratio = fs_new / fs_orig  # 1.5 倍
P, Q = 3, 2               # 分子分母互质
x_resamp = signal.resample(x_orig, int(len(x_orig) * ratio))
t_resamp = np.arange(len(x_resamp)) / fs_new
# 等价写法（指定目标点数）
x_resamp2 = signal.resample(x_orig, int(len(x_orig) * P / Q))
# ─── resample_poly：基于多相滤波器的重采样 ───
# MATLAB: y = resample(x, P, Q)
x_poly = signal.resample_poly(x_orig, up=P, down=Q)
# ─── decimate：降采样（先低通再抽取）───
# MATLAB: y = decimate(x, D)
D = 5  # 降采样因子
x_dec = signal.decimate(x_orig, D, ftype='iir')  # IIR 抗混叠
x_dec_fir = signal.decimate(x_orig, D, ftype='fir')  # FIR 抗混叠
# ─── upfirdn：上采样 → FIR 滤波 → 下采样 ───
# MATLAB: y = upfirdn(h, x, P, Q)
h = signal.firwin(30, 0.5)  # 低通滤波器
x_upfirdn = signal.upfirdn(h, x_orig, up=P, down=Q)
```

```
┌──────────────────────────────────────────────────────────────────┐
│                    重采样方法选择指南                              │
├──────────────┬──────────────────┬────────────────────────────────┤
│ 方法         │ 特点             │ 适用场景                       │
├──────────────┼──────────────────┼────────────────────────────────┤
│ resample     │ 基于 FFT，质量高  │ 有理数倍率，信号较短           │
│ resample_poly│ 多相滤波，效率高  │ 有理数倍率，实时/长信号        │
│ decimate     │ 整数倍降采样     │ 只需降采样，简单快捷           │
│ upfirdn      │ 自定义滤波器     │ 需要精确控制抗混叠滤波器       │
└──────────────┴──────────────────┴────────────────────────────────┘
```

---
## 3.9 相关与卷积
```python
# ═══════════════════════════════════════════
#  卷积
# ═══════════════════════════════════════════
a = np.array([1, 2, 3])
b = np.array([4, 5])
# MATLAB: conv(a, b)
np.convolve(a, b)         # [4, 13, 22, 15]  完整卷积
np.convolve(a, b, 'full') # 同上
np.convolve(a, b, 'same') # [13, 22, 22]     与长输入同长
np.convolve(a, b, 'valid')# [22]              完全重叠部分
# SciPy 的 fftconvolve（大数据量快很多）
from scipy.signal import fftconvolve
y = fftconvolve(a, b, mode='full')  # 基于 FFT，O(N log N)
# SciPy 的 oaconvolve（超长信号的分块卷积）
from scipy.signal import oaconvolve
y = oaconvolve(a, b, mode='full')   # 适合非常长的信号
# ═══════════════════════════════════════════
#  互相关（信号对齐、时延估计）
# ═══════════════════════════════════════════
# MATLAB: xcorr(a, b)
# NumPy
correlation = np.correlate(a, b, 'full')  # [5, 14, 23, 12]
# SciPy 的 correlate（更灵活）
from scipy.signal import correlate
corr = correlate(a, b, mode='full', method='auto')
# ─── 时延估计实战 ───
rng = np.random.default_rng(42)
fs = 1000
t = np.arange(0, 2, 1/fs)
original = np.sin(2 * np.pi * 30 * t) + 0.2 * rng.standard_normal(len(t))
# 制造一个延迟版本（延迟 50 个采样点 = 50ms）
delay_samples = 50
delayed = np.roll(original, delay_samples)
delayed[:delay_samples] = 0  # 前面补零
# 互相关找延迟
corr = correlate(delayed, original, mode='full')
lag = np.arange(-len(original)+1, len(original))
peak_idx = np.argmax(corr)
estimated_delay = lag[peak_idx]
print(f"真实延迟: {delay_samples} 样本 ({delay_samples/fs*1000:.1f} ms)")
print(f"估计延迟: {estimated_delay} 样本 ({estimated_delay/fs*1000:.1f} ms)")
# ─── 归一化互相关 ───
# MATLAB: xcorr(a, b, 'coeff')
def normalized_cross_correlation(x, y):
    corr = correlate(x - x.mean(), y - y.mean(), mode='full')
    norm = np.sqrt(np.sum((x - x.mean())**2) * np.sum((y - y.mean())**2))
    return corr / norm
ncc = normalized_cross_correlation(original, delayed)
```

---
## 3.10 信号生成
```python
from scipy.signal import chirp, gausspulse, sweep_poly, unit_impulse
import numpy as np
fs = 1000
t = np.arange(0, 2, 1/fs)
# ─── 扫频信号（Chirp）───
# MATLAB: chirp(t, f0, t1, f1)
y_chirp = chirp(t, f0=50, t1=2, f1=500, method='linear')    # 线性调频
y_chirp2 = chirp(t, f0=50, t1=2, f1=500, method='quadratic') # 二次调频
y_chirp3 = chirp(t, f0=50, t1=2, f1=500, method='logarithmic') # 对数调频
# ─── 高斯脉冲 ───
y_gauss = gausspulse(t, fc=100, bw=0.5)
# ─── 单位脉冲 ───
# MATLAB: [1 zeros(1,99)]
impulse = unit_impulse(100, idx=0)
# ─── 方波 ───
y_square = signal.square(2 * np.pi * 5 * t)  # 5Hz 方波
y_square_duty = signal.square(2 * np.pi * 5 * t, duty=0.3)  # 30% 占空比
# ─── 锯齿波 ───
y_sawtooth = signal.sawtooth(2 * np.pi * 5 * t)        # 上升锯齿
y_sawtooth2 = signal.sawtooth(2 * np.pi * 5 * t, 0.5)  # 三角波（对称）
```

---
## 3.11 峰值检测
```python
from scipy.signal import find_peaks, peak_prominences, peak_widths
# 生成含噪声的信号
rng = np.random.default_rng(42)
x = np.sin(2 * np.pi * 5 * np.arange(0, 2, 1/1000))
x += 0.3 * rng.standard_normal(len(x))
# ─── 基本峰值检测 ───
# MATLAB: findpeaks(x)
peaks, properties = find_peaks(x)
# ─── 带条件的峰值检测 ───
peaks, properties = find_peaks(
    x,
    height=0.5,          # 最小高度
    distance=50,         # 峰值之间最小距离（采样点数）
    prominence=0.3,      # 最小突出度
    width=5,             # 最小宽度
)
print(f"找到 {len(peaks)} 个峰值")
print(f"峰值位置: {peaks}")
print(f"峰值高度: {properties['peak_heights']}")
# ─── 突出度（Peak Prominences）───
prominences, left_bases, right_bases = peak_prominences(x, peaks)
# ─── 峰值宽度 ───
widths, width_heights, left_ips, right_ips = peak_widths(x, peaks, rel_height=0.5)
# 可视化
plt.figure(figsize=(12, 5))
plt.plot(x, 'b-', linewidth=0.5)
plt.plot(peaks, x[peaks], 'rx', markersize=10, label='峰值')
plt.vlines(x=peaks, ymin=x[peaks]-prominences, ymax=x[peaks],
           color='r', alpha=0.5, label='突出度')
plt.legend()
plt.grid(True, alpha=0.3)
plt.title('峰值检测')
plt.tight_layout()
plt.show()
```

---
## 3.12 信号包络与希尔伯特变换
```python
from scipy.signal import hilbert
# ─── 解析信号与包络 ───
# MATLAB: envelope(x)
x = np.sin(2 * np.pi * 50 * t) * (1 + 0.5 * np.sin(2 * np.pi * 3 * t))  # AM信号
analytic_signal = hilbert(x)     # 解析信号（复数）
envelope = np.abs(analytic_signal)  # 包络
instantaneous_phase = np.unwrap(np.angle(analytic_signal))  # 瞬时相位
instantaneous_freq = np.diff(instantaneous_phase) / (2 * np.pi) * fs  # 瞬时频率
plt.figure(figsize=(12, 6))
plt.subplot(2, 1, 1)
plt.plot(t, x, 'b', linewidth=0.5, alpha=0.7)
plt.plot(t, envelope, 'r', linewidth=2, label='包络')
plt.plot(t, -envelope, 'r', linewidth=2)
plt.title('AM信号与包络')
plt.legend()
plt.grid(True, alpha=0.3)
plt.subplot(2, 1, 2)
plt.plot(t[1:], instantaneous_freq, 'g', linewidth=0.5)
plt.title('瞬时频率')
plt.xlabel('时间
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---
## 3.13 本章速查表

| 需求           | MATLAB                        | Python (SciPy)                           |
| -------------- | ----------------------------- | ---------------------------------------- |
| 巴特沃斯滤波器 | `[b,a]=butter(N,Wn)`          | `b,a=signal.butter(N,Wn,fs=fs)`          |
| 自动定阶       | `[N,Wn]=buttord(wp,ws,gp,gs)` | `N,Wn=signal.buttord(wp,ws,gp,gs,fs=fs)` |
| FIR 滤波器     | `b=fir1(N,Wn)`                | `b=signal.firwin(N+1,Wn,fs=fs)`          |
| 滤波（有相位） | `y=filter(b,a,x)`             | `y=signal.lfilter(b,a,x)`                |
| 零相位滤波     | `y=filtfilt(b,a,x)`           | `y=signal.sosfiltfilt(sos,x)`            |
| 频率响应       | `[H,w]=freqz(b,a,N,fs)`       | `w,H=signal.freqz(b,a,worN=N,fs=fs)`     |
| 功率谱         | `[pxx,f]=pwelch(x,...)`       | `f,Pxx=signal.welch(x,fs,...)`           |
| STFT           | `spectrogram(x,...)`          | `f,t,Z=signal.stft(x,fs,...)`            |
| 重采样         | `y=resample(x,P,Q)`           | `y=signal.resample_poly(x,P,Q)`          |
| 互相关         | `xcorr(a,b)`                  | `signal.correlate(a,b)`                  |
| 峰值检测       | `findpeaks(x)`                | `find_peaks(x,height=...)`               |
| 包络           | `envelope(x)`                 | `np.abs(signal.hilbert(x))`              |
| 窗函数         | `hann(N)`                     | `signal.windows.hann(N)`                 |
| 群延迟         | `grpdelay(b,a)`               | `signal.group_delay((b,a))`              |

---

> **第三章结束。** 接下来是第四章：Matplotlib 信号可视化，将覆盖时域波形、频谱图、多子图布局、交互式绘图、导出出版级图片等。回复"继续"获取下一章。

# 第四章：Matplotlib —— 信号可视化
> Matplotlib 是 Python 的绘图标准库，对应 MATLAB 的 `plot`, `subplot`, `figure` 等全部绑图功能。本章聚焦信号处理场景，教你画出出版级的图表。

---
## 4.1 核心：两种绘图接口
Matplotlib 有两套 API，**必须搞清楚区别**：
```python
import matplotlib.pyplot as plt
import numpy as np
# ═══════════════════════════════════════════
#  接口1：MATLAB 风格（pyplot / plt）
# ═══════════════════════════════════════════
# 适合：快速探索、简单图表
plt.figure()
plt.plot([1, 2, 3], [4, 5, 6])
plt.title('Quick Plot')
plt.show()
# ═══════════════════════════════════════════
#  接口2：面向对象风格（Figure + Axes）  ← 推荐！
# ═══════════════════════════════════════════
# 适合：复杂布局、多子图、精确控制
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])
ax.set_title('OO Plot')
plt.show()
```
```
┌──────────────────────────────────────────────────────────────┐
│            两种接口对照                                      │
├──────────────────────┬───────────────────────────────────────┤
│  MATLAB 风格 (plt)   │  面向对象风格 (fig, ax)  ← 推荐       │
├──────────────────────┼───────────────────────────────────────┤
│  plt.figure()        │  fig, ax = plt.subplots()             │
│  plt.plot(x, y)      │  ax.plot(x, y)                        │
│  plt.title('...')    │  ax.set_title('...')                  │
│  plt.xlabel('...')   │  ax.set_xlabel('...')                 │
│  plt.ylabel('...')   │  ax.set_ylabel('...')                 │
│  plt.xlim(0, 10)     │  ax.set_xlim(0, 10)                   │
│  plt.ylim(-1, 1)     │  ax.set_ylim(-1, 1)                   │
│  plt.legend()        │  ax.legend()                           │
│  plt.grid(True)      │  ax.grid(True)                         │
│  plt.savefig('a.png')│  fig.savefig('a.png')                  │
│  plt.subplot(2,1,1)  │  fig, axes = plt.subplots(2, 1)       │
└──────────────────────┴───────────────────────────────────────┘
记忆：plt.xxx() 画图 → ax.set_xxx() 设置属性
注意：ax.set_title() 有 set_ 前缀！plt.title() 没有！
```

> **本指南统一使用面向对象接口**，这是专业用户的共识。

---
## 4.2 最基础的信号图：时域波形
```python
import numpy as np
import matplotlib.pyplot as plt
# ─── 生成信号 ───
fs = 1000
t = np.arange(0, 0.5, 1/fs)
x = np.sin(2 * np.pi * 50 * t) + 0.3 * np.sin(2 * np.pi * 150 * t)
# ─── 基本绘图 ───
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(t, x)
ax.set_xlabel('时间')
ax.set_ylabel('幅度')
ax.set_title('时域波形')
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### 4.2.1 线条样式完全控制
```python
fig, ax = plt.subplots(figsize=(12, 4))
# 完整的 plot 参数
ax.plot(t, x,
    color='#1f77b4',      # 颜色：十六进制 / 命名色 / 'r','g','b'...
    linewidth=1.5,         # 线宽（别名 lw）
    linestyle='-',         # 线型：'-', '--', '-.', ':'
    marker='o',            # 标记：'.', 'o', 's', '^', 'v', 'x', '+', '*'
    markersize=3,          # 标记大小（别名 ms）
    markerfacecolor='red', # 标记填充色（别名 mfc）
    markeredgecolor='k',   # 标记边框色（别名 mec）
    markeredgewidth=0.5,   # 标记边框宽度（别名 mew）
    alpha=0.8,             # 透明度 0~1
    label='信号1',         # 图例名称
)
# ─── 常用线型速查 ───
# linestyle: '-' 实线, '--' 虚线, '-.' 点划线, ':' 点线
# MATLAB:    '-'    '--'      '-.'       ':'
# ─── 常用标记速查 ───
# marker: '.' 点, 'o' 圆, 's' 方, '^' 三角上, 'v' 三角下
#         'D' 菱形, '*' 星, 'x' 叉, '+' 加号, '1' 三叉下
# MATLAB:  '.'     'o'     's'     '^'          'v'
ax.set_xlabel('时间')
ax.set_ylabel('幅度')
ax.set_title('线条样式详解')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### 4.2.2 颜色方案
```python
# ─── 颜色的 N 种写法 ───
ax.plot(t, x, color='r')                     # 单字符：'r','g','b','c','m','y','k','w'
ax.plot(t, x, color='blue')                  # 英文名
ax.plot(t, x, color='#FF5733')               # 十六进制
ax.plot(t, x, color=(0.1, 0.2, 0.5))         # RGB 元组（0~1）
ax.plot(t, x, color=(0.1, 0.2, 0.5, 0.8))    # RGBA 元组（含透明度）
# ─── 信号处理推荐配色（色盲友好）───
colors = {
    '蓝': '#0072B2',
    '橙': '#E69F00',
    '绿': '#009E73',
    '红': '#D55E00',
    '紫': '#CC79A7',
    '青': '#56B4E9',
    '黄': '#F0E442',
    '黑': '#000000',
}
# ─── 使用 colormap 生成渐变色 ───
n_lines = 5
cmap = plt.cm.viridis
for i in range(n_lines):
    color = cmap(i / (n_lines - 1))
    ax.plot(t, np.sin(2 * np.pi * (20 + i*20) * t), color=color, label=f'{20+i*20}Hz')
```

---
## 4.3 多子图布局
### 4.3.1 基本多子图
```python
import numpy as np
import matplotlib.pyplot as plt
fs = 1000
t = np.arange(0, 1, 1/fs)
rng = np.random.default_rng(42)
x = np.sin(2 * np.pi * 50 * t) + 0.5 * rng.standard_normal(len(t))
# ═══════════════════════════════════════════
#  方式1：plt.subplots（最常用）
# ═══════════════════════════════════════════
# MATLAB: subplot(3,1,1); plot(...); subplot(3,1,2); plot(...)
# 3 行 1 列
fig, axes = plt.subplots(3, 1, figsize=(10, 8), sharex=True)
# axes 是数组：axes[0], axes[1], axes[2]
axes[0].plot(t, x, 'b', linewidth=0.5)
axes[0].set_ylabel('原始信号')
axes[0].set_title('信号处理流程')
from scipy import signal as sig
b, a = sig.butter(4, 100, fs=fs)
y = sig.filtfilt(b, a, x)
axes[1].plot(t, y, 'r', linewidth=0.5)
axes[1].set_ylabel('滤波后')
# 包络
from scipy.signal import hilbert
env = np.abs(hilbert(y))
axes[2].plot(t, y, 'b', linewidth=0.5, alpha=0.5)
axes[2].plot(t, env, 'r', linewidth=1.5, label='包络')
axes[2].plot(t, -env, 'r', linewidth=1.5)
axes[2].set_ylabel('包络')
axes[2].set_xlabel('时间
for ax in axes:
    ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('signal_processing_pipeline.png', dpi=150)
plt.show()
# ═══════════════════════════════════════════
#  方式2：2×2 网格
# ═══════════════════════════════════════════
fig, axes = plt.subplots(2, 2, figsize=(12, 8))
# axes 是 2D 数组：axes[0,0], axes[0,1], axes[1,0], axes[1,1]
axes[0, 0].plot(t, x, 'b')
axes[0, 0].set_title('时域波形')
f, Pxx = sig.welch(x, fs=fs, nperseg=256)
axes[0, 1].semilogy(f, Pxx, 'r')
axes[0, 1].set_title('功率谱')
axes[1, 0].plot(t, y, 'g')
axes[1, 0].set_title('滤波后')
axes[1, 1].plot(t, env, 'm')
axes[1, 1].set_title('包络')
for ax in axes.flat:    # .flat 将 2D 数组展平为一维迭代器
    ax.grid(True, alpha=0.3)
    ax.set_xlabel('')
plt.tight_layout()
plt.show()
```

### 4.3.2 不等大子图（GridSpec）
```python
from matplotlib.gridspec import GridSpec
# 经典布局：上方大图（频谱图），下方两个小图（时域 + 频域）
fig = plt.figure(figsize=(12, 9))
gs = GridSpec(3, 2, figure=fig, height_ratios=[2, 1, 1], hspace=0.35, wspace=0.3)
# 频谱图占满第一行
ax_spec = fig.add_subplot(gs[0, :])
# 时域波形占左下
ax_time = fig.add_subplot(gs[1, 0])
# 频域占右下
ax_freq = fig.add_subplot(gs[1, 1])
# 包络占底部
ax_env = fig.add_subplot(gs[2, :])
# 填充内容
f_stft, t_stft, Zxx = sig.stft(x, fs=fs, nperseg=128)
ax_spec.pcolormesh(t_stft, f_stft, np.abs(Zxx), shading='gouraud', cmap='jet')
ax_spec.set_title('STFT 频谱图')
ax_spec.set_ylabel('频率')
ax_time.plot(t, x, 'b', linewidth=0.5)
ax_time.set_title('时域波形')
ax_freq.semilogy(f, Pxx, 'r')
ax_freq.set_title('功率谱')
ax_freq.set_xlim([0, 200])
ax_env.plot(t, env, 'm')
ax_env.set_title('包络')
ax_env.set_xlabel('时间')
plt.savefig('complex_layout.png', dpi=150, bbox_inches='tight')
plt.show()
```

### 4.3.3 共享坐标轴
```python
# sharex / sharey：共享坐标轴，缩放时联动
# 共享 X 轴（信号处理最常用：上下对齐时间轴）
fig, axes = plt.subplots(3, 1, figsize=(10, 8), sharex=True, sharey=False)
#                              sharex=True → 只有最下面的子图显示 x 轴标签
#                              sharey=True → 所有子图 y 轴范围相同
# 共享 Y 轴（左右对齐幅度轴）
fig, axes = plt.subplots(1, 3, figsize=(15, 4), sharey=True)
```

---
## 4.4 频域绘图（信号处理核心图表）
### 4.4.1 幅度谱与相位谱

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal
fs = 1000
N = 2048
t = np.arange(N) / fs
rng = np.random.default_rng(42)
x = np.sin(2 * np.pi * 50 * t) + 0.5 * np.sin(2 * np.pi * 120 * t) + 0.1 * rng.standard_normal(N)
# FFT
X = np.fft.rfft(x)
freqs = np.fft.rfftfreq(N, 1/fs)
mag = np.abs(X) / N * 2  # 单边谱幅度
mag[0] /= 2              # 直流分量不乘2
phase = np.angle(X)
# ═══════════════════════════════════════════
#  标准双图：幅度谱 + 相位谱
# ═══════════════════════════════════════════
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 6), sharex=True)
# 幅度谱（线性）
ax1.stem(freqs, mag, linefmt='b-', markerfmt='bo', basefmt='k-')
ax1.set_ylabel('幅度')
ax1.set_title('幅度谱（线性）')
ax1.grid(True, alpha=0.3)
ax1.set_xlim([0, 200])
# 相位谱（只画幅度显著的点）
threshold = 0.01
significant = mag > threshold
ax2.stem(freqs[significant], np.degrees(phase[significant]),
         linefmt='r-', markerfmt='ro', basefmt='k-')
ax2.set_ylabel('相位 (°)')
ax2.set_xlabel('频率')
ax2.set_title('相位谱（仅显著分量）')
ax2.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### 4.4.2 dB 刻度频谱
```python
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 6), sharex=True)
# 幅度谱
ax1.plot(freqs, 20 * np.log10(mag + 1e-20), 'b', linewidth=1)
ax1.set_ylabel('幅度')
ax1.set_title('幅度谱（dB）')
ax1.set_ylim([-80, 10])
ax1.grid(True, alpha=0.3)
# 功率谱密度
f_psd, Pxx = signal.welch(x, fs=fs, nperseg=512)
ax2.semilogy(f_psd, Pxx, 'r', linewidth=1)
ax2.set_ylabel('PSD (V²/Hz)')
ax2.set_xlabel('频率')
ax2.set_title('功率谱密度（Welch 法）')
ax2.set_xlim([0, 200])
ax2.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### 4.4.3 滤波器频率响应标准图
```python
# 信号处理论文/报告中最常见的图
fs = 1000
# 设计几种滤波器做对比
filters = {
    '巴特沃斯(4阶)': signal.butter(4, [20, 200], btype='band', fs=fs, output='sos'),
    '切比雪夫I(4阶,1dB)': signal.cheby1(4, 1, [20, 200], btype='band', fs=fs, output='sos'),
    '椭圆(4阶,1dB,40dB)': signal.ellip(4, 1, 40, [20, 200], btype='band', fs=fs, output='sos'),
}
fig, axes = plt.subplots(2, 1, figsize=(10, 8), sharex=True)
for name, sos in filters.items():
    w, H = signal.sosfreqz(sos, worN=4096, fs=fs)
    mag_db = 20 * np.log10(np.abs(H) + 1e-20)
    phase_deg = np.degrees(np.unwrap(np.angle(H)))
    axes[0].plot(w, mag_db, label=name, linewidth=1.5)
    axes[1].plot(w, phase_deg, label=name, linewidth=1.5)
# 标注通带和阻带
axes[0].axvline(20, color='gray', linestyle='--', alpha=0.5)
axes[0].axvline(200, color='gray', linestyle='--', alpha=0.5)
axes[0].axhline(-3, color='green', linestyle=':', alpha=0.5, label='-3 dB')
axes[0].fill_betweenx([-100, 5], 20, 200, alpha=0.05, color='green', label='通带')
axes[0].set_ylabel('幅度')
axes[0].set_title('带通滤波器对比')
axes[0].set_ylim([-80, 5])
axes[0].legend(fontsize=9)
axes[0].grid(True, alpha=0.3)
axes[1].set_ylabel('相位 (°)')
axes[1].set_xlabel('频率')
axes[1].legend(fontsize=9)
axes[1].grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('filter_comparison.png', dpi=150)
plt.show()
```

---
## 4.5 频谱图（Spectrogram / STFT 热力图）

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal
# ─── 生成调频信号 ───
fs = 4000
t = np.arange(0, 3, 1/fs)
f0, f1 = 100, 1500
chirp_signal = signal.chirp(t, f0, t[-1], f1, method='linear')
# ─── STFT ───
nperseg = 256
f, t_stft, Sxx = signal.spectrogram(
    chirp_signal, fs,
    window='hann',
    nperseg=nperseg,
    noverlap=nperseg // 2,
    nfft=nperseg,
    scaling='density'
)
# ═══════════════════════════════════════════
#  标准频谱图
# ═══════════════════════════════════════════
fig, ax = plt.subplots(figsize=(10, 6))
# pcolormesh 比 imshow 更适合非均匀频率轴
pcm = ax.pcolormesh(t_stft, f, 10 * np.log10(Sxx + 1e-20),
                     shading='gouraud', cmap='jet')
cbar = fig.colorbar(pcm, ax=ax, label='功率谱密度')
ax.set_ylabel('频率')
ax.set_xlabel('时间')
ax.set_title('频谱图 (Spectrogram)')
ax.set_ylim([0, 2000])
plt.tight_layout()
plt.savefig('spectrogram.png', dpi=150)
plt.show()
# ═══════════════════════════════════════════
#  多 colormap 选择
# ═══════════════════════════════════════════
cmaps = {
    'jet':     '经典彩虹（不推荐论文，但工程常用）',
    'viridis': '默认，色盲友好（推荐）',
    'plasma':  '暖色调，色盲友好',
    'hot':     '热力图',
    'gray':    '灰度',
    'coolwarm':'冷暖对比',
    'RdBu':    '红蓝对比（适合相关函数）',
}
# 预览所有 colormap
fig, axes = plt.subplots(2, 4, figsize=(16, 6))
for ax, (cmap_name, desc) in zip(axes.flat, cmaps.items()):
    pcm = ax.pcolormesh(t_stft, f, 10*np.log10(Sxx+1e-20),
                         shading='gouraud', cmap=cmap_name)
    ax.set_title(cmap_name, fontsize=10)
    ax.set_ylim([0, 2000])
    fig.colorbar(pcm, ax=ax)
plt.tight_layout()
plt.show()
```

---
## 4.6 注释与标注（标记峰值、关键点）

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.signal import find_peaks
fs = 1000
t = np.arange(0, 0.2, 1/fs)
x = np.sin(2 * np.pi * 25 * t) * np.exp(-10 * t)
peaks, _ = find_peaks(x)
fig, ax = plt.subplots(figsize=(10, 5))
ax.plot(t, x, 'b-', linewidth=1.5, label='衰减正弦')
# ─── 标注峰值 ───
for i, peak_idx in enumerate(peaks):
    ax.annotate(
        f'峰值{i+1}: {x[peak_idx]:.3f}',
        xy=(t[peak_idx], x[peak_idx]),           # 箭头指向的点
        xytext=(t[peak_idx] + 0.02, x[peak_idx] + 0.1),  # 文字位置
        fontsize=9,
        arrowprops=dict(
            arrowstyle='->',           # 箭头样式
            color='red',
            lw=1.5,
        ),
        color='red',
    )
# ─── 标注特定区域 ───
ax.axvspan(0, 0.02, alpha=0.1, color='red', label='起始区域')
ax.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
ax.axvline(x=0.05, color='green', linestyle=':', alpha=0.7, label='t=50ms')
# ─── 文本框 ───
ax.text(0.1, 0.8, f'频率: 25 Hz\n衰减系数: 10',
        transform=ax.transAxes,   # 坐标系：0~1 归一化
        fontsize=10,
        verticalalignment='top',
        bbox=dict(boxstyle='round', facecolor='wheat', alpha=0.5))
ax.set_xlabel('时间')
ax.set_ylabel('幅度')
ax.set_title('信号标注示例')
ax.legend(loc='upper right')
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---
## 4.7 双 Y 轴图（幅度 + 相位 / 信号 + 包络）
```python
import numpy as np
import matplotlib.pyplot as plt
fs = 1000
t = np.arange(0, 0.5, 1/fs)
x = np.sin(2 * np.pi * 50 * t)
# ═══════════════════════════════════════════
#  双 Y 轴：幅度(dB) + 相位
# ═══════════════════════════════════════════
from scipy import signal
b, a = signal.butter(4, 100, fs=fs)
w, H = signal.freqz(b, a, worN=2048, fs=fs)
fig, ax1 = plt.subplots(figsize=(10, 5))
color1 = 'tab:blue'
ax1.plot(w, 20 * np.log10(np.abs(H) + 1e-20), color=color1, linewidth=1.5)
ax1.set_xlabel('频率')
ax1.set_ylabel('幅度', color=color1)
ax1.tick_params(axis='y', labelcolor=color1)
ax1.set_ylim([-60, 5])
ax1.grid(True, alpha=0.3)
# 创建共享 X 轴的第二个 Y 轴
ax2 = ax1.twinx()
color2 = 'tab:red'
ax2.plot(w, np.degrees(np.unwrap(np.angle(H))), color=color2, linewidth=1.5, linestyle='--')
ax2.set_ylabel('相位 (°)', color=color2)
ax2.tick_params(axis='y', labelcolor=color2)
plt.title('滤波器频率响应：幅度 + 相位')
fig.tight_layout()
plt.show()
```

> **警告：双 Y 轴容易误导读者，论文中慎用。** 更推荐分成上下两个子图。

---
## 4.8 中文字体配置（必须设置！）
```python
# ═══════════════════════════════════════════
#  方法1：全局设置（推荐，放在脚本开头）
# ═══════════════════════════════════════════
import matplotlib.pyplot as plt
import matplotlib
# Windows
matplotlib.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'DejaVu Sans']
# macOS
# matplotlib.rcParams['font.sans-serif'] = ['Arial Unicode MS', 'PingFang SC']
# Linux
# matplotlib.rcParams['font.sans-serif'] = ['WenQuanYi Micro Hei', 'Noto Sans CJK SC']
# 解决负号显示问题
matplotlib.rcParams['axes.unicode_minus'] = False
# ═══════════════════════════════════════════
#  方法2：临时设置
# ═══════════════════════════════════════════
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False
# ═══════════════════════════════════════════
#  方法3：查看系统可用字体
# ═══════════════════════════════════════════
from matplotlib.font_manager import fontManager
fonts = sorted(set(f.name for f in fontManager.ttflist))
chinese_fonts = [f for f in fonts if any(kw in f.lower() for kw in
    ['hei', 'song', 'kai', 'ming', 'yahei', 'yuan', 'fang', 'cjk', 'noto', 'wenquan'])]
print("可用中文字体:", chinese_fonts)
```

---
## 4.9 导出出版级图片
```python
# ═══════════════════════════════════════════
#  基本导出
# ═══════════════════════════════════════════
fig.savefig('output.png')                          # 默认 100 DPI
fig.savefig('output.png', dpi=300)                 # 高分辨率
fig.savefig('output.png', dpi=300, bbox_inches='tight')  # 去除白边
fig.savefig('output.pdf')                          # 矢量图（论文推荐）
fig.savefig('output.svg')                          # 矢量图（网页推荐）
fig.savefig('output.eps')                          # 矢量图（LaTeX 推荐）
# ═══════════════════════════════════════════
#  不同场景的推荐格式
# ═══════════════════════════════════════════
# 论文/报告：PDF 或 EPS（矢量，无限缩放）
# PPT/文档：PNG 300dpi
# 网页/博客：SVG 或 PNG 150dpi
# 预览/调试：plt.show()
# ═══════════════════════════════════════════
#  论文风格预设
# ═══════════════════════════════════════════
plt.style.use('seaborn-v0_8-whitegrid')  # 或 'seaborn-v0_8', 'ggplot', 'bmh'
# 查看所有可用风格
print(plt.style.available)
# ['Solarize_Light2', '_classic_test_patch', 'bmh', 'classic',
#  'dark_background', 'fast', 'fivethirtyeight', 'ggplot',
#  'grayscale', 'seaborn-v0_8', 'seaborn-v0_8-bright',
#  'seaborn-v0_8-colorblind', 'seaborn-v0_8-dark',
#  'seaborn-v0_8-darkgrid', 'seaborn-v0_8-dark-palette',
#  'seaborn-v0_8-deep', 'seaborn-v0_8-muted', ...]
```

---
## 4.10 交互式绘图（数据探索）
```python
# ═══════════════════════════════════════════
#  方法1：matplotlib 内置交互（鼠标悬停显示坐标）
# ═══════════════════════════════════════════
# 在 plt.show() 前添加格式化器
import matplotlib.ticker as ticker
fig, ax = plt.subplots()
ax.plot(t, x)
ax.format_coord = lambda x, y: f't={x:.4f}s, x={y:.4f}'
# ═══════════════════════════════════════════
#  方法2：mplcursors 库（点击标注数值）
# ═══════════════════════════════════════════
# pip install mplcursors
import mplcursors
fig, ax = plt.subplots()
line, = ax.plot(t, x)
cursor = mplcursors.cursor(line, hover=True)  # 悬停自动标注
plt.show()
# ═══════════════════════════════════════════
#  方法3：Plotly（推荐，交互性最强）
# ═══════════════════════════════════════════
import plotly.graph_objects as go
fig = go.Figure()
fig.add_trace(go.Scatter(x=t, y=x, mode='lines', name='信号'))
fig.update_layout(
    title='交互式信号图',
    xaxis_title='时间
    yaxis_title='幅度',
    hovermode='x unified'   # 所有曲线在同一X处对齐显示
)
fig.show()   # 自动在浏览器中打开
# Plotly 导出静态图片（需要 kaleido）
# pip install kaleido
# fig.write_image('plotly_output.png', width=1200, height=600, scale=2)
```

---
## 4.11 信号处理常用图表模板
### 模板1：完整的滤波效果展示
```python
def plot_filtering_result(t, x_original, x_filtered, fs, sos):
    """
    一键绘制滤波效果：时域对比 + 频域对比 + 滤波器响应
    """
    from scipy import signal
    fig = plt.figure(figsize=(14, 10))
    gs = GridSpec(3, 2, figure=fig, hspace=0.4, wspace=0.3)
    # ─── 1. 原始信号 ───
    ax1 = fig.add_subplot(gs[0, 0])
    ax1.plot(t, x_original, 'b', linewidth=0.5)
    ax1.set_title('原始信号')
    ax1.set_ylabel('幅度')
    ax1.grid(True, alpha=0.3)
    # ─── 2. 滤波后信号 ───
    ax2 = fig.add_subplot(gs[0, 1])
    ax2.plot(t, x_filtered, 'r', linewidth=0.5)
    ax2.set_title('滤波后信号')
    ax2.set_ylabel('幅度')
    ax2.grid(True, alpha=0.3)
    # ─── 3. 原始频谱 ───
    ax3 = fig.add_subplot(gs[1, 0])
    f_orig, Pxx_orig = signal.welch(x_original, fs=fs, nperseg=512)
    ax3.semilogy(f_orig, Pxx_orig, 'b', linewidth=1)
    ax3.set_title('原始功率谱')
    ax3.set_ylabel('PSD (V²/Hz)')
    ax3.set_xlim([0, fs/2])
    ax3.grid(True, alpha=0.3)
    # ─── 4. 滤波后频谱 ───
    ax4 = fig.add_subplot(gs[1, 1])
    f_filt, Pxx_filt = signal.welch(x_filtered, fs=fs, nperseg=512)
    ax4.semilogy(f_filt, Pxx_filt, 'r', linewidth=1)
    ax4.set_title('滤波后功率谱')
    ax4.set_ylabel('PSD (V²/Hz)')
    ax4.set_xlim([0, fs/2])
    ax4.grid(True, alpha=0.3)
    # ─── 5. 滤波器幅频响应 ───
    ax5 = fig.add_subplot(gs[2, 0])
    w, H = signal.sosfreqz(sos, worN=4096, fs=fs)
    ax5.plot(w, 20 * np.log10(np.abs(H) + 1e-20), 'g', linewidth=1.5)
    ax5.set_title('滤波器幅频响应')
    ax5.set_xlabel('频率')
    ax5.set_ylabel('幅度')
    ax5.set_ylim([-80, 5])
    ax5.grid(True, alpha=0.3)
    # ─── 6. 残差（被滤除的部分）───
    ax6 = fig.add_subplot(gs[2, 1])
    residual = x_original - x_filtered
    ax6.plot(t, residual, 'gray', linewidth=0.5)
    ax6.set_title('残差（被滤除的成分）')
    ax6.set_xlabel('时间')
    ax6.set_ylabel('幅度')
    ax6.grid(True, alpha=0.3)
    return fig
# 使用
fs = 1000
t = np.arange(0, 2, 1/fs)
rng = np.random.default_rng(42)
x = np.sin(2*np.pi*10*t) + 0.5*np.sin(2*np.pi*50*t) + 0.2*rng.standard_normal(len(t))
sos = signal.butter(4, [5, 30], btype='band', fs=fs, output='sos')
y = signal.sosfiltfilt(sos, x)
fig = plot_filtering_result(t, x, y, fs, sos)
plt.savefig('filtering_result.png', dpi=150)
plt.show()
```

### 模板2：多通道信号对比
```python
def plot_multi_channel(signals, channel_names, fs, time_range=None):
    """
    绘制多通道信号（自动偏移不重叠）
    signals: shape (n_channels, n_samples) 或 list of arrays
    """
    signals = np.atleast_2d(signals)
    n_channels = signals.shape[0]
    t = np.arange(signals.shape[1]) / fs
    if time_range is not None:
        mask = (t >= time_range[0]) & (t <= time_range[1])
        t = t[mask]
        signals = signals[:, mask]
    # 计算每个通道的偏移量
    offsets = np.zeros(n_channels)
    for i in range(1, n_channels):
        offsets[i] = offsets[i-1] + np.max(np.abs(signals[i-1])) * 1.5
    fig, ax = plt.subplots(figsize=(14, n_channels * 1.5))
    for i in range(n_channels):
        ax.plot(t, signals[i] + offsets[i], linewidth=0.8, label=channel_names[i])
        ax.axhline(y=offsets[i], color='gray', linestyle='-', alpha=0.2)
        # 通道名称标注在左侧
        ax.text(t[0] - 0.01 * (t[-1]-t[0]), offsets[i], channel_names[i],
                ha='right', va='center', fontsize=10)
    ax.set_xlabel('时间
    ax.set_yticks([])  # 隐藏 Y 轴刻度
    ax.set_title('多通道信号')
    ax.grid(True, alpha=0.2)
    plt.tight_layout()
    return fig
# 使用
fs = 500
t = np.arange(0, 3, 1/fs)
eeg_channels = np.array([
    np.sin(2*np.pi*10*t) + 0.2*np.random.randn(len(t)),
    np.sin(2*np.pi*10*t + np.pi/4) + 0.3*np.random.randn(len(t)),
    np.sin(2*np.pi*10*t + np.pi/2) + 0.15*np.random.randn(len(t)),
    np.sin(2*np.pi*10*t + np.pi) + 0.25*np.random.randn(len(t)),
])
names = ['Fp1', 'Fp2', 'C3', 'C4']
fig = plot_multi_channel(eeg_channels, names, fs, time_range=(0.5, 2.0))
plt.savefig('multi_channel.png', dpi=150)
plt.show()
```

---
## 4.12 保存不显示（服务器/批量绘图）
```python
# 在无显示器的服务器上绘图
import matplotlib
matplotlib.use('Agg')   # 必须在 import pyplot 之前！
import matplotlib.pyplot as plt
import numpy as np
# 绘图...
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [4, 5, 6])
fig.savefig('output.png')
plt.close(fig)  # 释放内存！批量绘图时必须关闭
# ─── 批量绘图模板 ───
for i in range(100):
    fig, ax = plt.subplots()
    ax.plot(data[i])
    fig.savefig(f'frame_{i:03d}.png', dpi=100)
    plt.close(fig)  # 不 close 会内存泄漏！
```

---
## 4.13 MATLAB vs Matplotlib 绘图命令完整对照

| 功能        | MATLAB                    | Matplotlib (OO)                       |
| ----------- | ------------------------- | ------------------------------------- |
| 新建图      | `figure`                  | `fig, ax = plt.subplots()`            |
| 折线图      | `plot(x,y)`               | `ax.plot(x,y)`                        |
| 散点图      | `scatter(x,y)`            | `ax.scatter(x,y)`                     |
| 柱状图      | `bar(x,y)`                | `ax.bar(x,y)`                         |
| 横向柱状图  | `barh(x,y)`               | `ax.barh(x,y)`                        |
| 直方图      | `hist(x,n)`               | `ax.hist(x, bins=n)`                  |
| 茎叶图      | `stem(x,y)`               | `ax.stem(x,y)`                        |
| 填充区域    | `fill(x,y,'b')`           | `ax.fill_between(x,y1,y2)`            |
| 对数Y轴     | `semilogy(x,y)`           | `ax.semilogy(x,y)`                    |
| 双对数      | `loglog(x,y)`             | `ax.loglog(x,y)`                      |
| 图像/热力图 | `imagesc(Z)`              | `ax.imshow(Z)` / `ax.pcolormesh(...)` |
| 等高线      | `contour(Z)`              | `ax.contour(Z)`                       |
| 子图        | `subplot(2,1,1)`          | `fig, axes = plt.subplots(2,1)`       |
| 标题        | `title('text')`           | `ax.set_title('text')`                |
| X轴标签     | `xlabel('text')`          | `ax.set_xlabel('text')`               |
| 图例        | `legend`                  | `ax.legend()`                         |
| 网格        | `grid on`                 | `ax.grid(True)`                       |
| X轴范围     | `xlim([0 10])`            | `ax.set_xlim(0, 10)`                  |
| 保存        | `saveas(gcf,'a.png')`     | `fig.savefig('a.png')`                |
| 清除        | `clf`                     | `plt.clf()` / `ax.cla()`              |
| 关闭        | `close all`               | `plt.close('all')`                    |
| 保持叠加    | `hold on`                 | 不需要（默认叠加）                    |
| 颜色条      | `colorbar`                | `fig.colorbar(im, ax=ax)`             |
| 双Y轴       | `yyaxis left/right`       | `ax2 = ax1.twinx()`                   |
| 文字        | `text(x,y,'str')`         | `ax.text(x,y,'str')`                  |
| 箭头        | `annotation('arrow',...)` | `ax.annotate(...)`                    |

---

> **第四章结束。** 接下来是第五章：Pandas —— 信号数据管理，将覆盖时间序列处理、多通道数据管理、数据读写、滚动窗口运算等。回复"继续"获取下一章。

# 第五章：Pandas —— 信号数据管理
> Pandas = Panel Data。它是 Python 的表格数据处理库，对应 MATLAB 的 timetable + table + dataframes。对于信号处理工程师，Pandas 最核心的价值是**时间序列管理**和**数据清洗**。

---
## 5.1 为什么信号处理需要 Pandas？
```
┌──────────────────────────────────────────────────────────────┐
│          NumPy vs Pandas：什么时候用哪个？                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  NumPy：纯数值计算                                           │
│  ├── 信号滤波、FFT、卷积                                     │
│  ├── 矩阵运算、线性代数                                      │
│  └── 需要极致性能的场景                                      │
│                                                              │
│  Pandas：带标签的数据管理                                    │
│  ├── 时间戳对齐（不同采样率的信号同步）                       │
│  ├── 缺失值处理（传感器丢数据）                              │
│  ├── 数据读写（CSV、Excel、HDF5、SQL）                       │
│  ├── 分组统计（按小时/按通道聚合）                            │
│  └── 多通道数据管理（类似 MATLAB timetable）                  │
│                                                              │
│  实际工作流：                                                │
│  原始数据 → Pandas(读取/清洗/对齐) → NumPy(计算) → Pandas(存储)│
└──────────────────────────────────────────────────────────────┘
```

---
## 5.2 核心数据结构：Series 与 DataFrame
### 5.2.1 Series — 带标签的一维数组
```python
import numpy as np
import pandas as pd
# ─── 从列表创建 ───
s = pd.Series([1.2, 2.5, 3.1, 4.8, 5.3])
print(s)
# 0    1.2
# 1    2.5
# 2    3.1
# 3    4.8
# 4    5.3
# dtype: float64
# 左边是索引（Index），右边是值
print(s.index)   # RangeIndex(start=0, stop=5, step=1)
print(s.values)  # array([1.2, 2.5, 3.1, 4.8, 5.3])  ← NumPy 数组！
# ─── 自定义索引 ───
s = pd.Series([100, 200, 300], index=['CH1', 'CH2', 'CH3'])
print(s['CH2'])  # 200
# ─── 从字典创建 ───
s = pd.Series({'alpha': 0.5, 'beta': 1.2, 'gamma': 3.4})
# ─── 从 NumPy 数组创建（最常用）───
data = np.sin(np.linspace(0, 2*np.pi, 100))
s = pd.Series(data, name='signal')
```

### 5.2.2 DataFrame — 带标签的二维表格
```python
# ═══════════════════════════════════════════
#  创建 DataFrame 的多种方式
# ═══════════════════════════════════════════
# 方式1：从字典创建（最常用）
df = pd.DataFrame({
    'time':   [0.0, 0.1, 0.2, 0.3, 0.4],
    'CH1':    [1.2, 2.3, 3.1, 2.8, 1.5],
    'CH2':    [0.5, 1.1, 2.0, 1.8, 0.9],
    'label':  ['A', 'A', 'B', 'B', 'A'],
})
print(df)
#    time   CH1   CH2 label
# 0   0.0   1.2   0.5      A
# 1   0.1   2.3   1.1      A
# 2   0.2   3.1   2.0      B
# 3   0.3   2.8   1.8      B
# 4   0.4   1.5   0.9      A
# 方式2：从 NumPy 数组创建
data = np.random.randn(100, 4)
df = pd.DataFrame(data, columns=['CH1', 'CH2', 'CH3', 'CH4'])
# 方式3：从列表的列表创建
df = pd.DataFrame([
    [0.0, 1.2, 0.5],
    [0.1, 2.3, 1.1],
    [0.2, 3.1, 2.0],
], columns=['time', 'CH1', 'CH2'])
# ─── 基本属性 ───
print(df.shape)      # (5, 4)    行数×列数
print(df.columns)    # 列名
print(df.index)      # 行索引
print(df.dtypes)     # 每列数据类型
print(df.info())     # 完整信息
print(df.describe()) # 统计摘要（类似 MATLAB 的 summary）
```

### 5.2.3 MATLAB table 对照
```matlab
% MATLAB
T = table(time, CH1, CH2, CH3);
T.time = seconds(T.time);
summary(T);
% Python 等价
df = pd.DataFrame({'time': time, 'CH1': CH1, 'CH2': CH2, 'CH3': CH3})
df['time'] = pd.to_timedelta(df['time'], unit='s')
df.describe()
```

---
## 5.3 数据读取与写入
### 5.3.1 CSV 文件
```python
# ─── 读取 CSV ───
# MATLAB: T = readtable('data.csv')
df = pd.read_csv('signal_data.csv')
# 常用参数
df = pd.read_csv('data.csv',
    sep=',',              # 分隔符（默认逗号）
    header=0,             # 第一行是列名
    index_col=0,          # 第一列作为索引
    usecols=['CH1','CH2'],# 只读指定列
    nrows=1000,           # 只读前 1000 行（大文件探索）
    encoding='utf-8',     # 编码
    na_values=['NaN','--'],# 自定义缺失值标记
    parse_dates=['timestamp'],  # 自动解析日期列
    dtype={'CH1': np.float64},  # 指定列类型
)
# ─── 写入 CSV ───
df.to_csv('output.csv', index=False)  # index=False 不写行号
# ─── 大文件分块读取 ───
chunk_iter = pd.read_csv('huge_file.csv', chunksize=10000)
for chunk in chunk_iter:
    process(chunk)  # 每次处理 1 万行
```

### 5.3.2 Excel 文件
```python
# 读取
# MATLAB: T = readtable('data.xlsx')
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')
# 写入
df.to_excel('output.xlsx', sheet_name='结果', index=False)
# 多 sheet 写入
with pd.ExcelWriter('multi_sheet.xlsx') as writer:
    df_raw.to_excel(writer, sheet_name='原始数据', index=False)
    df_filtered.to_excel(writer, sheet_name='滤波后', index=False)
    df_stats.to_excel(writer, sheet_name='统计指标', index=False)
```

### 5.3.3 HDF5 文件（大规模信号数据推荐）
```python
# HDF5：适合 GB 级数据，读写极快，支持压缩
# MATLAB: T = readtable('data.h5')
# 写入
df.to_hdf('signal_store.h5', key='raw_data', mode='w')
# 追加
df_filtered.to_hdf('signal_store.h5', key='filtered_data', mode='a')
# 读取
df_raw = pd.read_hdf('signal_store.h5', key='raw_data')
# 查看文件中有哪些 key
with pd.HDFStore('signal_store.h5') as store:
    print(store.keys())  # ['/raw_data', '/filtered_data']
# ─── 大数据性能对比 ───
# 100 万行 × 10 列的数据
# CSV 写入: ~5 秒, 读取: ~2 秒, 文件大小: ~80 MB
# Excel:   ~30 秒, 读取: ~10 秒, 文件大小: ~50 MB
# HDF5:    ~0.3 秒, 读取: ~0.1 秒, 文件大小: ~30 MB (压缩后)
```

### 5.3.4 其他格式
```python
# ─── JSON ───
df.to_json('data.json', orient='records', indent=2)
df = pd.read_json('data.json')
# ─── Parquet（大数据生态推荐）───
df.to_parquet('data.parquet', compression='gzip')
df = pd.read_parquet('data.parquet')
# ─── Pickle（Python 专用，最快但不可跨语言）───
df.to_pickle('data.pkl')
df = pd.read_pickle('data.pkl')
# ─── 剪贴板（从 Excel 复制后直接读取）───
df = pd.read_clipboard()  # 直接 Ctrl+C 然后 read_clipboard
# ─── SQL 数据库 ───
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM signals WHERE channel="CH1"', conn)
df.to_sql('results', conn, if_exists='replace', index=False)
```

---
## 5.4 时间序列处理（信号处理最核心功能）
### 5.4.1 创建时间索引
```python
import pandas as pd
import numpy as np
# ─── 方式1：从采样率和点数创建 ───
fs = 1000  # 采样率
N = 5000   # 采样点数
# MATLAB: t = seconds(0:1/fs:(N-1)/fs)
t_index = pd.date_range(start='2024-01-01 08:00:00',
                         periods=N,
                         freq=f'{1000//fs}ms')  # 1ms 间隔
print(t_index[:5])
# DatetimeIndex(['2024-01-01 08:00:00', '2024-01-01 08:00:00.001',
#                '2024-01-01 08:00:00.002', ...])
# ─── 方式2：指定起止时间和频率 ───
t_index = pd.date_range(start='2024-01-01', end='2024-01-02',
                         freq='1s')  # 每秒一个点
print(f"点数: {len(t_index)}")  # 86401
# ─── 方式3：从字符串解析 ───
timestamps = ['2024-01-01 08:00:00.000',
              '2024-01-01 08:00:00.001',
              '2024-01-01 08:00:00.002']
t_index = pd.to_datetime(timestamps)
# ─── 频率字符串速查 ───
# '1ms'   毫秒
# '100us' 100微秒
# '1s'    秒
# '1min'  分钟
# '1h'    小时
# '1D'    天
# '1W'    周
# '1ME'   月末
```

### 5.4.2 带时间索引的 DataFrame
```python
fs = 100
N = 1000
t_index = pd.date_range('2024-01-01', periods=N, freq='10ms')
rng = np.random.default_rng(42)
df = pd.DataFrame({
    'ECG': np.sin(2 * np.pi * 1 * np.arange(N)/fs) + 0.2 * rng.standard_normal(N),
    'EMG': 0.5 * rng.standard_normal(N),
    'temp': 36.5 + 0.1 * np.cumsum(rng.standard_normal(N)) / 50,
}, index=t_index)
df.index.name = 'timestamp'
print(df.head())
#                             ECG       EMG      temp
# timestamp
# 2024-01-01 00:00:00  0.089765 -0.347621  36.50000
# 2024-01-01 00:00:00.010  0.241669  0.175424  36.50028
# 2024-01-01 00:00:00.020  0.389418 -0.140754  36.49972
# ...
```

### 5.4.3 时间索引的选择与切片
```python
# ─── 按时间范围切片（最强大！）───
# MATLAB: T(timerange(start, end), :)
# 方式1：字符串切片
subset = df['2024-01-01 00:00:01':'2024-01-01 00:00:05']
# 方式2：部分匹配（自动补全）
df['2024-01-01']              # 整天的数据
df['2024-01']                 # 整个月的数据
df['2024']                    # 整年的数据
# 方式3：loc 精确选择
subset = df.loc['2024-01-01 00:00:01':'2024-01-01 00:00:05']
# ─── 按条件选择 ───
# 选择温度 > 36.6 的时间段
hot = df[df['temp'] > 36.6]
# 选择 ECG 信号幅度超过阈值的时刻
threshold = 0.8
peaks_time = df[df['ECG'].abs() > threshold]
```

### 5.4.4 重采样（Resample）— 不同采样率转换
```python
# ═══════════════════════════════════════════
#  降采样：高频 → 低频
# ═══════════════════════════════════════════
# MATLAB: retime(T, '5s', 'mean')
# 原始: 100Hz → 降采样到 10Hz
df_10hz = df.resample('100ms').mean()       # 每 100ms 取均值
df_10hz = df.resample('100ms').last()       # 每 100ms 取最后一个值
df_10hz = df.resample('100ms').first()      # 每 100ms 取第一个值
# 自定义聚合
df_10hz = df.resample('100ms').agg({
    'ECG': 'mean',          # ECG 取均值
    'EMG': 'rms',           # EMG 取 RMS（需自定义）
    'temp': 'last',         # 温度取最后一个
})
# 自定义 RMS 聚合函数
def rms(x):
    return np.sqrt(np.mean(x**2))
df_10hz = df.resample('100ms').agg({
    'ECG': 'mean',
    'EMG': rms,
    'temp': 'last',
})
# ═══════════════════════════════════════════
#  升采样：低频 → 高频（插值）
# ═══════════════════════════════════════════
# 原始: 100Hz → 升采样到 1000Hz
df_1000hz = df.resample('1ms').interpolate(method='linear')
# 插值方法：
# 'linear'    线性插值（默认）
# 'cubic'     三次样条
# 'quadratic' 二次样条
# 'spline'    B样条
# 'pad'       前向填充（用前一个值）
# 'nearest'   最近邻
# ─── OHLC 重采样（金融/振动监测）───
# 每秒的开高低均
df_1s = df['ECG'].resample('1s').ohlc()
# 返回: open, high, low, close 四列
```

### 5.4.5 滑动窗口（Rolling）— 实时特征提取
```python
# ═══════════════════════════════════════════
#  基本滑动窗口
# ═══════════════════════════════════════════
# MATLAB: movmean(x, window), movstd(x, window)
# 窗口大小：指定采样点数
df['ECG_mean'] = df['ECG'].rolling(window=50).mean()       # 50点滑动均值
df['ECG_std']  = df['ECG'].rolling(window=50).std()        # 50点滑动标准差
df['ECG_max']  = df['ECG'].rolling(window=50).max()        # 50点滑动最大值
df['ECG_min']  = df['ECG'].rolling(window=50).min()        # 50点滑动最小值
df['ECG_sum']  = df['ECG'].rolling(window=50).sum()        # 50点滑动求和
# 窗口大小：指定时间长度（时间索引专有！）
df['ECG_200ms_mean'] = df['ECG'].rolling('200ms').mean()   # 200ms 滑动均值
# ═══════════════════════════════════════════
#  滑动 RMS（信号处理极常用）
# ═══════════════════════════════════════════
def rolling_rms(series, window):
    """计算滑动 RMS 值"""
    return np.sqrt((series ** 2).rolling(window=window).mean())
df['EMG_rms'] = rolling_rms(df['EMG'], window=50)
# ═══════════════════════════════════════════
#  滑动窗口参数详解
# ═══════════════════════════════════════════
df['ECG_roll'] = df['ECG'].rolling(
    window=50,             # 窗口大小
    min_periods=1,         # 最少需要的观测值（默认=window，设1避免开头NaN）
    center=True,           # 窗口居中（默认False，即只看过去）
    win_type='hann',       # 窗函数类型！信号处理利器
)
# ─── 带窗函数的滑动平均 ───
# 类似加窗平滑滤波
df['ECG_smooth'] = df['ECG'].rolling(
    window=51, center=True, win_type='hann'
).mean()
# 其他窗函数：'boxcar'(矩形), 'triang'(三角), 'hamming', 'blackman', 'kaiser'
# Kaiser 窗需要参数
df['ECG_kaiser'] = df['ECG'].rolling(
    window=51, center=True, win_type='kaiser', alpha=8
).mean()
# ═══════════════════════════════════════════
#  指数加权移动平均（EWMA）
# ═══════════════════════════════════════════
# MATLAB: smoothdata(x, 'ewma', alpha)
df['ECG_ewm'] = df['ECG'].ewm(
    span=50,          # 跨度（类似窗口大小）
    # 或 alpha=0.1,   # 平滑因子
).mean()
# ═══════════════════════════════════════════
#  扩展窗口（累积统计）
# ═══════════════════════════════════════════
df['ECG_cummean'] = df['ECG'].expanding().mean()   # 累积均值
df['ECG_cummax']  = df['ECG'].expanding().max()    # 累积最大值
```

### 5.4.6 滑动窗口可视化
```python
import matplotlib.pyplot as plt
fig, axes = plt.subplots(3, 1, figsize=(14, 9), sharex=True)
# 原始信号
axes[0].plot(df.index, df['EMG'], 'b', linewidth=0.3, alpha=0.5, label='原始 EMG')
axes[0].set_ylabel('幅度')
# 滑动 RMS
axes[1].plot(df.index, df['EMG_rms'], 'r', linewidth=1.5, label='滑动RMS (50点)')
axes[1].set_ylabel('RMS')
# 滑动均值 vs 窗函数平滑
axes[2].plot(df.index, df['ECG'], 'b', linewidth=0.3, alpha=0.3, label='原始 ECG')
axes[2].plot(df.index, df['ECG_mean'], 'r', linewidth=1, label='矩形窗滑动均值')
axes[2].plot(df.index, df['ECG_smooth'], 'g', linewidth=1.5, label='汉宁窗滑动均值')
axes[2].set_ylabel('幅度')
axes[2].set_xlabel('时间')
for ax in axes:
    ax.legend(loc='upper right')
    ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('rolling_features.png', dpi=150)
plt.show()
```

---
## 5.5 缺失值处理
```python
import pandas as pd
import numpy as np
# ─── 创建含缺失值的 DataFrame ───
df = pd.DataFrame({
    'CH1': [1.0, np.nan, 3.0, np.nan, 5.0, 6.0],
    'CH2': [0.5, 1.2, np.nan, np.nan, 2.1, 2.5],
    'CH3': [7.0, 7.1, 7.2, 7.3, np.nan, 7.5],
})
# ─── 检测缺失值 ───
print(df.isna())          # 布尔矩阵
print(df.isna().sum())    # 每列缺失数量
print(df.isna().any())    # 每列是否有缺失
# ═══════════════════════════════════════════
#  删除缺失值
# ═══════════════════════════════════════════
df_drop_row = df.dropna()                  # 删除任何含 NaN 的行
df_drop_col = df.dropna(axis=1)            # 删除任何含 NaN 的列
df_drop_all = df.dropna(how='all')         # 只删除全为 NaN 的行
df_drop_thresh = df.dropna(thresh=2)       # 至少保留 2 个非 NaN 值
# ═══════════════════════════════════════════
#  填充缺失值（信号处理常用）
# ═══════════════════════════════════════════
# 方式1：常数填充
df_fill0 = df.fillna(0)
# 方式2：前向/后向填充（传感器数据最常用）
df_ffill = df.fillna(method='ffill')   # 用前一个有效值填充
df_bfill = df.fillna(method='bfill')   # 用后一个有效值填充
# 新版 Pandas 推荐写法：
df_ffill = df.ffill()
df_bfill = df.bfill()
# 方式3：插值填充
df_linear  = df.interpolate(method='linear')     # 线性插值（最常用）
df_spline  = df.interpolate(method='spline', order=3)  # 三次样条
df_cubic   = df.interpolate(method='cubic')       # 三次插值
df_nearest = df.interpolate(method='nearest')     # 最近邻
# 方式4：用统计量填充
df_mean = df.fillna(df.mean())          # 用列均值填充
df_median = df.fillna(df.median())      # 用列中位数填充
# ─── 信号处理实战：传感器丢包补全 ───
# 策略：短间隔用线性插值，长间隔标记为无效
max_gap = 5  # 最大允许插值间隔（采样点数）
def smart_fill(series, max_gap=5):
    """智能填充：只填充短间隔缺失，长间隔保持NaN"""
    return series.interpolate(method='linear').where(
        series.isna().groupby(
            (series.notna() != series.shift().notna()).cumsum()
        ).transform('sum') <= max_gap
    )
# 更简单的方法：限制连续填充数
df_filled = df.interpolate(method='linear', limit=3)  # 最多连续填充3个
```

---
## 5.6 数据选择与过滤
### 5.6.1 列选择
```python
df = pd.DataFrame({
    'time': np.arange(0, 1, 0.001),
    'ECG': np.random.randn(1000),
    'EMG': np.random.randn(1000),
    'EEG': np.random.randn(1000),
    'label': np.random.choice(['rest', 'active'], 1000),
})
# ─── 单列（返回 Series）───
ecg = df['ECG']          # 或 df.ECG
# ─── 多列（返回 DataFrame）───
subset = df[['ECG', 'EMG']]
# ─── 列名模式匹配 ───
e_channels = df.filter(like='E')         # 列名含 'E' 的
e_channels = df.filter(regex='^E[CM]')   # 正则匹配 ECG, EMG
# ─── 排除列 ───
features = df.drop(columns=['time', 'label'])
# ─── 选择数值列 ───
numeric_cols = df.select_dtypes(include=[np.number])
```

### 5.6.2 行选择
```python
# ─── 按位置 ───
row = df.iloc[0]              # 第 0 行
rows = df.iloc[10:20]         # 第 10-19 行
rows = df.iloc[-5:]           # 最后 5 行
# ─── 按标签 ───
row = df.loc[0]               # 索引标签为 0 的行
rows = df.loc[10:20]          # 标签 10 到 20（含两端！）
# ─── 布尔索引（最常用！）───
# MATLAB: T(T.label == 'active', :)
active = df[df['label'] == 'active']                # 等于
big_ecg = df[df['ECG'].abs() > 2]                   # 条件筛选
combo = df[(df['ECG'] > 1) & (df['label'] == 'active')]  # 组合条件
# ─── query 方法（可读性更好）───
active = df.query("label == 'active'")
big_ecg = df.query("abs(ECG) > 2")
combo = df.query("ECG > 1 and label == 'active'")
# ─── 采样 ───
sample = df.sample(n=100)      # 随机抽 100 行
sample = df.sample(frac=0.1)   # 随机抽 10%
```

### 5.6.3 同时选择行和列
```python
# ─── loc: 标签选择 ───
df.loc[0:10, 'ECG']                    # 行0-10, ECG列
df.loc[0:10, ['ECG', 'EMG']]          # 行0-10, ECG和EMG列
df.loc[df['label']=='active', 'ECG']   # 布尔行 + 列名
# ─── iloc: 位置选择 ───
df.iloc[0:10, 1]                       # 行0-9, 第1列
df.iloc[0:10, 1:4]                    # 行0-9, 第1-3列
df.iloc[:5, :]                         # 前5行，所有列
# ─── at / iat: 取单个值（最快）───
val = df.at[0, 'ECG']                  # 按标签取单个值
val = df.iat[0, 1]                     # 按位置取单个值
```

---
## 5.7 分组聚合（GroupBy）
```python
import pandas as pd
import numpy as np
# ─── 创建多通道多条件的信号数据 ───
rng = np.random.default_rng(42)
N = 10000
df = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=N, freq='10ms'),
    'channel':   np.random.choice(['ECG','EMG','EEG'], N),
    'condition': np.random.choice(['rest','stress','exercise'], N),
    'value':     rng.standard_normal(N) * 10 + 50,
})
# ═══════════════════════════════════════════
#  单列分组
# ═══════════════════════════════════════════
# MATLAB: groupsummary(T, 'channel', 'mean')
# 每个通道的统计量
df.groupby('channel')['value'].mean()       # 每通道均值
df.groupby('channel')['value'].agg(['mean', 'std', 'min', 'max'])
# ═══════════════════════════════════════════
#  多列分组
# ═══════════════════════════════════════════
# 每个通道×条件的统计量
result = df.groupby(['channel', 'condition'])['value'].agg(
    mean_val='mean',
    std_val='std',
    count='count',
)
print(result)
# ═══════════════════════════════════════════
#  自定义聚合函数
# ═══════════════════════════════════════════
def rms(x):
    return np.sqrt(np.mean(x**2))
def crest_factor(x):
    """波峰因子 = 峰值 / RMS"""
    return np.max(np.abs(x)) / rms(x) if rms(x) > 0 else 0
def zero_crossing_rate(x):
    """过零率"""
    return np.sum(np.diff(np.sign(x)) != 0) / len(x)
result = df.groupby('channel')['value'].agg(
    mean='mean',
    rms=rms,
    crest=crest_factor,
    zcr=zero_crossing_rate,
)
print(result)
# ═══════════════════════════════════════════
#  按时间分组
# ═══════════════════════════════════════════
# 每分钟统计
df.set_index('timestamp', inplace=True)
minute_stats = df.groupby('channel')['value'].resample('1min').agg(
    mean='mean', rms=rms, max='max'
)
print(minute_stats)
# 按小时分组
hourly = df.groupby('channel')['value'].resample('1h').mean()
# ═══════════════════════════════════════════
#  透视表（Pivot Table）
# ═══════════════════════════════════════════
# MATLAB: pivot(T, 'channel', 'condition', 'value', 'mean')
df_reset = df.reset_index()
pivot = pd.pivot_table(
    df_reset,
    values='value',
    index='channel',
    columns='condition',
    aggfunc=['mean', 'std'],
)
print(pivot)
```

---
## 5.8 数据变换与特征工程
### 5.8.1 列运算

```python
import pandas as pd
import numpy as np
fs = 100
N = 5000
t = np.arange(N) / fs
df = pd.DataFrame({
    'time': t,
    'ECG': np.sin(2*np.pi*1*t) + 0.2*np.random.randn(N),
    'EMG': 0.5*np.random.randn(N),
})
# ─── 基本运算 ───
df['ECG_squared'] = df['ECG'] ** 2           # 逐元素平方
df['ECG_abs'] = df['ECG'].abs()              # 绝对值
df['EMG_dB'] = 20 * np.log10(df['EMG'].abs() + 1e-20)  # 转dB
# ─── 通道间运算 ───
df['ECG_plus_EMG'] = df['ECG'] + df['EMG']   # 通道相加
df['ECG_normalized'] = (df['ECG'] - df['ECG'].mean()) / df['ECG'].std()  # Z-score 归一化
# ─── 差分（离散导数）───
# MATLAB: diff(x)
df['ECG_diff'] = df['ECG'].diff()             # 一阶差分
df['ECG_diff2'] = df['ECG'].diff(2)           # 二阶差分
df['ECG_pct_change'] = df['ECG'].pct_change() # 百分比变化
# ─── 累积运算 ───
df['ECG_cumsum'] = df['ECG'].cumsum()         # 累积和
df['ECG_cummax'] = df['ECG'].cummax()         # 累积最大值
df['ECG_cummin'] = df['ECG'].cummin()         # 累积最小值
# ─── 排名 ───
df['ECG_rank'] = df['ECG'].rank()             # 排名
```
### 5.8.2 apply — 逐行/逐列自定义函数
```python
# ═══════════════════════════════════════════
#  逐列应用
# ═══════════════════════════════════════════
# 对所有数值列计算 RMS
numeric_df = df.select_dtypes(include=[np.number]).drop(columns=['time'])
rms_values = numeric_df.apply(lambda col: np.sqrt(np.mean(col**2)))
print(rms_values)
# ═══════════════════════════════════════════
#  逐行应用
# ═══════════════════════════════════════════
# 计算每行的多通道联合特征
def row_features(row):
    return pd.Series({
        'ECG_EMG_ratio': row['ECG'] / (row['EMG'] + 1e-10),
        'total_energy': row['ECG']**2 + row['EMG']**2,
    })
# 注意：逐行 apply 很慢，大数据量避免使用
df_features = df.apply(row_features, axis=1)
```
### 5.8.3 transform — 分组变换不改变形状
```python
# ─── Z-score 归一化（按通道）───
# 每个通道独立归一化，但保持原始行数
df['ECG_zscore'] = df.groupby('channel')['value'].transform(
    lambda x: (x - x.mean()) / x.std()
)
# ─── 减去组内均值（去基线）───
df['value_centered'] = df.groupby('channel')['value'].transform(
    lambda x: x - x.mean()
)
# ─── 组内排名 ───
df['within_group_rank'] = df.groupby('channel')['value'].transform('rank')
```
### 5.8.4 信号特征提取模板
```python
def extract_features(signal_array, fs):
    """从一段信号中提取时域和频域特征"""
    from scipy import signal as sig
    from scipy.stats import kurtosis, skew
    N = len(signal_array)
    t_features = {}
    # ─── 时域特征 ───
    t_features['mean'] = np.mean(signal_array)
    t_features['std'] = np.std(signal_array)
    t_features['rms'] = np.sqrt(np.mean(signal_array**2))
    t_features['max'] = np.max(signal_array)
    t_features['min'] = np.min(signal_array)
    t_features['peak_to_peak'] = np.max(signal_array) - np.min(signal_array)
    t_features['crest_factor'] = np.max(np.abs(signal_array)) / t_features['rms']
    t_features['kurtosis'] = kurtosis(signal_array)
    t_features['skewness'] = skew(signal_array)
    t_features['zcr'] = np.sum(np.diff(np.sign(signal_array)) != 0) / N
    # ─── 频域特征 ───
    f, Pxx = sig.welch(signal_array, fs=fs, nperseg=min(256, N))
    Pxx_norm = Pxx / (np.sum(Pxx) + 1e-20)
    t_features['spectral_centroid'] = np.sum(f * Pxx_norm)
    t_features['spectral_bandwidth'] = np.sqrt(np.sum((f - t_features['spectral_centroid'])**2 * Pxx_norm))
    t_features['spectral_flatness'] = np.exp(np.mean(np.log(Pxx + 1e-20))) / (np.mean(Pxx) + 1e-20)
    # 带宽能量
    low_mask = f < 10
    mid_mask = (f >= 10) & (f < 50)
    high_mask = f >= 50
    t_features['low_freq_energy'] = np.sum(Pxx[low_mask])
    t_features['mid_freq_energy'] = np.sum(Pxx[mid_mask])
    t_features['high_freq_energy'] = np.sum(Pxx[high_mask])
    return t_features
# ─── 对 DataFrame 滑动窗口提取特征 ───
window_size = 500  # 5秒窗口
step = 100         # 1秒步进
features_list = []
for start in range(0, len(df) - window_size + 1, step):
    window = df['ECG'].iloc[start:start+window_size].values
    feat = extract_features(window, fs)
    feat['start_time'] = df['time'].iloc[start]
    features_list.append(feat)
df_features = pd.DataFrame(features_list)
print(df_features.head())
print(f"特征矩阵形状: {df_features.shape}")
```

---
## 5.9 多 DataFrame 操作
### 5.9.1 合并（Merge）
```python
# 类似 SQL 的 JOIN
# MATLAB: outerjoin(T1, T2, 'Keys', 'time')
df_left = pd.DataFrame({
    'time': [0.0, 0.1, 0.2, 0.3],
    'ECG':  [1.0, 1.2, 1.5, 1.3],
})
df_right = pd.DataFrame({
    'time': [0.0, 0.1, 0.2, 0.3],
    'EMG':  [0.5, 0.6, 0.4, 0.7],
    'label': ['A', 'A', 'B', 'B'],
})
# ─── 按键合并 ───
merged = pd.merge(df_left, df_right, on='time', how='outer')  # 外连接
merged = pd.merge(df_left, df_right, on='time', how='inner')  # 内连接
merged = pd.merge(df_left, df_right, on='time', how='left')   # 左连接
# ─── 多键合并 ───
merged = pd.merge(df1, df2, on=['time', 'channel'], how='outer')
```
### 5.9.2 拼接（Concat）
```python
# ─── 垂直拼接（加行）───
# MATLAB: [T1; T2]
df_combined = pd.concat([df1, df2], axis=0, ignore_index=True)
# ─── 水平拼接（加列）───
# MATLAB: [T1, T2]
df_combined = pd.concat([df1, df2], axis=1)
# ─── 多个 DataFrame 合并 ───
dfs = [df1, df2, df3, df4]
df_all = pd.concat(dfs, axis=0, ignore_index=True)
# ─── 按键去重合并 ───
df_combined = pd.concat(dfs).drop_duplicates(subset='timestamp').sort_values('timestamp')
```
### 5.9.3 时间序列对齐（不同采样率同步）
```python
# ─── 场景：ECG 500Hz, EMG 100Hz, 温度 1Hz，需要对齐到同一时间轴 ───
# 创建不同采样率的数据
fs_ecg, fs_emg, fs_temp = 500, 100, 1
T = 10.0  # 10 秒
t_ecg = np.arange(0, T, 1/fs_ecg)
t_emg = np.arange(0, T, 1/fs_emg)
t_temp = np.arange(0, T, 1/fs_temp)
df_ecg = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=len(t_ecg), freq='2ms'),
    'ECG': np.sin(2*np.pi*5*t_ecg) + 0.1*np.random.randn(len(t_ecg)),
}).set_index('timestamp')
df_emg = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=len(t_emg), freq='10ms'),
    'EMG': 0.5*np.random.randn(len(t_emg)),
}).set_index('timestamp')
df_temp = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=len(t_temp), freq='1s'),
    'temp': 36.5 + 0.01*np.random.randn(len(t_temp)),
}).set_index('timestamp')
# ─── 方法1：join（对齐到某一索引）───
# 以 ECG 的 500Hz 时间轴为基准
df_aligned = df_ecg.join(df_emg).join(df_temp)
# EMG 和 temp 在非采样时刻自动填充 NaN
# 然后插值补全
df_aligned['EMG'] = df_aligned['EMG'].interpolate(method='linear')
df_aligned['temp'] = df_aligned['temp'].fillna(method='ffill')
# ─── 方法2：merge_asof（最近邻对齐）───
df_ecg_reset = df_ecg.reset_index()
df_emg_reset = df_emg.reset_index()
df_temp_reset = df_temp.reset_index()
# 对齐 EMG 到 ECG（允许最大 5ms 偏差）
df_step1 = pd.merge_asof(
    df_ecg_reset, df_emg_reset,
    on='timestamp', direction='nearest',
    tolerance=pd.Timedelta('5ms')
)
# 对齐 temp 到 ECG
df_aligned2 = pd.merge_asof(
    df_step1, df_temp_reset,
    on='timestamp', direction='nearest',
    tolerance=pd.Timedelta('1s')
)
print(df_aligned2.head(10))
print(f"对齐后缺失值:\n{df_aligned2.isna().sum()}")
```

---
## 5.10 Pandas 与 NumPy/SciPy 的无缝协作
```python
# ═══════════════════════════════════════════
#  核心：DataFrame.values 或 DataFrame.to_numpy() → NumPy 数组
# ═══════════════════════════════════════════
df = pd.DataFrame({
    'ECG': np.sin(2*np.pi*5*np.arange(1000)/500) + 0.1*np.random.randn(1000),
    'EMG': 0.5*np.random.randn(1000),
})
# ─── 取出 NumPy 数组做滤波 ───
from scipy import signal
ecg_array = df['ECG'].values   # 或 df['ECG'].to_numpy()
b, a = signal.butter(4, 10, fs=500)
ecg_filtered = signal.filtfilt(b, a, ecg_array)
# 写回 DataFrame
df['ECG_filtered'] = ecg_filtered
# ─── 取出多列做矩阵运算 ───
data_matrix = df[['ECG', 'EMG']].to_numpy()  # shape (1000, 2)
# 现在可以对 data_matrix 做任何 NumPy/SciPy 运算
# ─── 一键对多列做滤波 ───
channels = ['ECG', 'EMG']
for ch in channels:
    df[f'{ch}_filtered'] = signal.filtfilt(b, a, df[ch].values)
# ─── 使用 pipe 链式操作 ───
def bandpass_filter(series, lowcut, highcut, fs, order=4):
    """对 Series 做带通滤波"""
    sos = signal.butter(order, [lowcut, highcut], btype='band', fs=fs, output='sos')
    return pd.Series(
        signal.sosfiltfilt(sos, series.values),
        index=series.index,
        name=series.name
    )
# 链式调用
df_clean = (df
    .pipe(bandpass_filter, lowcut=0.5, highcut=40, fs=500)  # ❌ pipe 作用于整个 df
)
# 正确方式：用 transform 或 apply
df_filtered = df[['ECG', 'EMG']].apply(
    lambda col: bandpass_filter(col, lowcut=0.5, highcut=40, fs=500)
)
```

---
## 5.11 性能优化
```python
# ═══════════════════════════════════════════
#  规则1：少用 iterrows，多用向量化
# ═══════════════════════════════════════════
# ❌ 极慢：逐行遍历
for idx, row in df.iterrows():
    df.loc[idx, 'ECG_sq'] = row['ECG'] ** 2
# ✅ 快 100 倍：向量化
df['ECG_sq'] = df['ECG'] ** 2
# ═══════════════════════════════════════════
#  规则2：少用 apply，多用内置方法
# ═══════════════════════════════════════════
# ❌ 慢：apply
df['ECG_abs'] = df['ECG'].apply(np.abs)
# ✅ 快：内置方法
df['ECG_abs'] = df['ECG'].abs()
# ═══════════════════════════════════════════
#  规则3：大数据用 NumPy 做计算再放回
# ═══════════════════════════════════════════
# ❌ 慢：在 DataFrame 上做复杂计算
result = df[['ECG', 'EMG']].rolling(100).apply(lambda x: np.sqrt(np.mean(x**2)))
# ✅ 快：用 NumPy 算完再放回
ecg_arr = df['ECG'].values
# 用 scipy 或 numpy 实现高速 rolling RMS
from scipy.signal import lfilter
squared = ecg_arr ** 2
window = np.ones(100) / 100
mean_sq = lfilter(window, 1, squared)
rms_result = np.sqrt(mean_sq)
df['ECG_rms_fast'] = rms_result
# ═══════════════════════════════════════════
#  规则4：指定 dtype 节省内存
# ═══════════════════════════════════════════
df = pd.DataFrame({'value': np.random.randn(1_000_000)})
print(f"默认 float64 内存: {df.memory_usage(deep=True).sum() / 1e6:.1f} MB")  # ~7.6 MB
df['value'] = df['value'].astype(np.float32)
print(f"float32 内存: {df.memory_usage(deep=True).sum() / 1e6:.1f} MB")      # ~3.8 MB
# ═══════════════════════════════════════════
#  规则5：读取时指定 dtype
# ═══════════════════════════════════════════
df = pd.read_csv('large_file.csv', dtype={
    'ECG': np.float32,
    'EMG': np.float32,
    'label': 'category',   # 分类类型，大幅节省内存
})
```

---
## 5.12 完整实战：多通道生理信号分析流水线
```python
import numpy as np
import pandas as pd
from scipy import signal
import matplotlib.pyplot as plt
# ═══════════════════════════════════════════
#  Step 1: 读取数据
# ═══════════════════════════════════════════
# 假设原始数据在 CSV 中
# df = pd.read_csv('physio_data.csv', parse_dates=['timestamp'], index_col='timestamp')
# 这里用模拟数据
fs = 500
N = 30000  # 60 秒数据
t = np.arange(N) / fs
rng = np.random.default_rng(42)
t_index = pd.date_range('2024-01-01 08:00:00', periods=N, freq='2ms')
df = pd.DataFrame({
    'ECG': np.sin(2*np.pi*1*t) + 0.3*rng.standard_normal(N),
    'EMG': 0.5*rng.standard_normal(N),
    'EDA': 0.1*np.cumsum(rng.standard_normal(N)/100) + 5,
    'label': np.repeat(['rest','stress','exercise','recovery'], N//4),
}, index=t_index)
df.index.name = 'timestamp'
print(f"数据形状: {df.shape}")
print(df.head())
# ═══════════════════════════════════════════
#  Step 2: 数据清洗
# ═══════════════════════════════════════════
# 检查缺失值
print(f"\n缺失值:\n{df.isna().sum()}")
# 异常值检测与截断
for ch in ['ECG', 'EMG']:
    q99 = df[ch].quantile(0.99)
    q01 = df[ch].quantile(0.01)
    df[ch] = df[ch].clip(q01, q99)
# ═══════════════════════════════════════════
#  Step 3: 滤波
# ═══════════════════════════════════════════
def filter_channel(series, filter_type, fs, **kwargs):
    """通用滤波函数"""
    sos = signal.butter(**kwargs, fs=fs, output='sos')
    return pd.Series(signal.sosfiltfilt(sos, series.values), index=series.index, name=series.name)
# ECG: 0.5-40 Hz 带通
df['ECG_filt'] = filter_channel(df['ECG'], 'bandpass', fs, N=4, Wn=[0.5, 40], btype='band')
# EMG: 20-200 Hz 带通
df['EMG_filt'] = filter_channel(df['EMG'], 'bandpass', fs, N=4, Wn=[20, 200], btype='band')
# EDA: 0.05-5 Hz 低通
df['EDA_filt'] = filter_channel(df['EDA'], 'lowpass', fs, N=4, Wn=5, btype='low')
# ═══════════════════════════════════════════
#  Step 4: 特征提取（滑动窗口）
# ═══════════════════════════════════════════
def rms(x): return np.sqrt(np.mean(x**2))
window = '1s'  # 1 秒窗口
features = pd.DataFrame({
    'ECG_rms':     df['ECG_filt'].rolling(window).apply(rms),
    'EMG_rms':     df['EMG_filt'].rolling(window).apply(rms),
    'EDA_mean':    df['EDA_filt'].rolling(window).mean(),
    'EDA_slope':   df['EDA_filt'].rolling(window).apply(lambda x: np.polyfit(np.arange(len(x)), x, 1)[0]),
}, index=df.index)
# 降采样到 1Hz（每秒一个特征点）
features_1hz = features.resample('1s').last()
# ═══════════════════════════════════════════
#  Step 5: 按条件分组统计
# ═══════════════════════════════════════════
# 给 features 加上 label
features_1hz['label'] = df['label'].resample('1s').first()
group_stats = features_1hz.groupby('label')[['ECG_rms','EMG_rms','EDA_mean']].agg(
    ['mean', 'std', 'min', 'max']
)
print("\n按条件分组统计:")
print(group_stats.round(4))
# ═══════════════════════════════════════════
#  Step 6: 可视化
# ═══════════════════════════════════════════
fig, axes = plt.subplots(4, 1, figsize=(14, 12), sharex=True)
colors = {'rest':'green', 'stress':'red', 'exercise':'orange', 'recovery':'blue'}
# 绘制信号 + 条件背景色
for i, ch in enumerate(['ECG_filt', 'EMG_filt', 'EDA_filt']):
    axes[i].plot(df.index, df[ch], linewidth=0.3, color='black', alpha=0.5)
    # 添加条件背景
    for label, color in colors.items():
        mask = df['label'] == label
        if mask.any():
            starts = df.index[mask & ~mask.shift(1, fill_value=False)]
            ends = df.index[mask & ~mask.shift(-1, fill_value=False)]
            for s, e in zip(starts, ends):
                axes[i].axvspan(s, e, alpha=0.1, color=color)
    axes[i].set_ylabel(ch.replace('_filt', ''))
    axes[i].grid(True, alpha=0.2)
# 绘制 RMS 特征
axes[3].plot(features_1hz.index, features_1hz['ECG_rms'], label='ECG RMS', color='blue')
axes[3].plot(features_1hz.index, features_1hz['EMG_rms'], label='EMG RMS', color='red')
axes[3].set_ylabel('RMS')
axes[3].set_xlabel('时间')
axes[3].legend()
axes[3].grid(True, alpha=0.2)
# 添加图例
from matplotlib.patches import Patch
legend_elements = [Patch(facecolor=c, alpha=0.3, label=l) for l, c in colors.items()]
axes[0].legend(handles=legend_elements, loc='upper right', ncol=4)
plt.tight_layout()
plt.savefig('physio_pipeline.png', dpi=150)
plt.show()
# ═══════════════════════════════════════════
#  Step 7: 保存结果
# ═══════════════════════════════════════════
df.to_hdf('analysis_results.h5', key='filtered_data', mode='w')
features_1hz.to_hdf('analysis_results.h5', key='features_1hz', mode='a')
group_stats.to_csv('group_statistics.csv')
print("\n✅ 分析完成，结果已保存！")
```

---
## 5.13 MATLAB vs Pandas 速查表

| 功能       | MATLAB                        | Pandas                                   |
| ---------- | ----------------------------- | ---------------------------------------- |
| 创建表格   | `table(col1,col2)`            | `pd.DataFrame({'col1':..., 'col2':...})` |
| 读取CSV    | `readtable('f.csv')`          | `pd.read_csv('f.csv')`                   |
| 写入CSV    | `writetable(T,'f.csv')`       | `df.to_csv('f.csv')`                     |
| 查看前N行  | `T(1:N,:)`                    | `df.head(N)`                             |
| 查看后N行  | `T(end-N+1:end,:)`            | `df.tail(N)`                             |
| 表格大小   | `height(T), width(T)`         | `df.shape`                               |
| 列名       | `T.Properties.VariableNames`  | `df.columns`                             |
| 选列       | `T.ECG`                       | `df['ECG']`                              |
| 选行       | `T(1:10,:)`                   | `df.iloc[0:10]`                          |
| 条件筛选   | `T(T.ECG>1,:)`                | `df[df['ECG']>1]`                        |
| 排序       | `sortrows(T,'ECG')`           | `df.sort_values('ECG')`                  |
| 去重       | `unique(T)`                   | `df.drop_duplicates()`                   |
| 缺失值检测 | `ismissing(T)`                | `df.isna()`                              |
| 删除缺失   | `rmmissing(T)`                | `df.dropna()`                            |
| 填充缺失   | `fillmissing(T,'linear')`     | `df.interpolate()`                       |
| 分组统计   | `groupsummary(T,'ch','mean')` | `df.groupby('ch').mean()`                |
| 横向合并   | `[T1, T2]`                    | `pd.concat([df1,df2], axis=1)`           |
| 纵向合并   | `[T1; T2]`                    | `pd.concat([df1,df2], axis=0)`           |
| 键合并     | `outerjoin(T1,T2,'Keys','t')` | `pd.merge(df1,df2,on='t')`               |
| 时间索引   | `timetable(Time, data)`       | `df.set_index('timestamp')`              |
| 重采样     | `retime(T,'1s','mean')`       | `df.resample('1s').mean()`               |
| 滑动均值   | `movmean(x,50)`               | `df.rolling(50).mean()`                  |
| 数据类型   | `varfun(@double, T)`          | `df.astype(float)`                       |
| 透视表     | `pivot(T,...)`                | `pd.pivot_table(df,...)`                 |
| 统计摘要   | `summary(T)`                  | `df.describe()`                          |

---

> **第五章结束。** 接下来是最终章第六章：综合实战 —— 完整信号处理流水线，将把 NumPy + SciPy + Matplotlib + Pandas 四大库串联起来，做一个端到端的项目。回复"继续"获取最终章。

# 第六章：综合实战 —— 完整信号处理流水线
> 本章将 NumPy + SciPy + Matplotlib + Pandas 四大库串联，完成一个端到端的信号处理项目。以 **旋转机械振动信号故障诊断** 为案例，覆盖从数据读取到报告生成的全流程。

---

## 6.1 项目总览
```
┌─────────────────────────────────────────────────────────────────────┐
│                  旋转机械振动信号故障诊断流水线                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ① 数据读取          ② 预处理           ③ 特征提取                  │
│  ┌─────────┐        ┌─────────┐        ┌─────────┐                │
│  │ CSV/HDF5│───────→│ 去趋势  │───────→│ 时域特征 │                │
│  │ 多通道  │        │ 滤波    │        │ 频域特征 │                │
│  └─────────┘        │ 去异常  │        │ 时频特征 │                │
│                     └─────────┘        └─────────┘                │
│                                                                     │
│  ④ 特征分析          ⑤ 可视化           ⑥ 报告输出                  │
│  ┌─────────┐        ┌─────────┐        ┌─────────┐                │
│  │ 分组统计 │───────→│ 波形图  │───────→│ 图片    │                │
│  │ 相关分析 │        │ 频谱图  │        │ CSV     │                │
│  │ 异常检测 │        │ 趋势图  │        │ HDF5    │                │
│  └─────────┘        └─────────┘        └─────────┘                │
└─────────────────────────────────────────────────────────────────────┘
```

---
## 6.2 Step 1：数据生成与读取
### 6.2.1 模拟工业振动信号
```python
"""
模拟旋转机械振动信号
- 正常状态：转频 + 少量谐波 + 随机噪声
- 不平衡故障：1×转频幅值增大
- 不对中故障：2×转频幅值增大
- 轴承外圈故障：高频共振 + 特征频率调制
"""
import numpy as np
import pandas as pd
from scipy import signal
def generate_vibration_signal(condition, fs=10000, duration=10.0, seed=None):
    """
    生成模拟振动信号
    
    Parameters
    ----------
    condition : str
        'normal', 'imbalance', 'misalignment', 'bearing_fault'
    fs : int
        采样率 Hz
    duration : float
        信号时长 秒
    seed : int
        随机种子
        
    Returns
    -------
    df : DataFrame
        包含时间戳和多通道振动数据
    """
    rng = np.random.default_rng(seed)
    N = int(fs * duration)
    t = np.arange(N) / fs
    
    # 转子参数
    rpm = 3000                           # 转速 3000 RPM
    f_rotor = rpm / 60                   # 转频 50 Hz
    
    # 轴承参数（SKF 6205 典型值）
    n_balls = 9                           # 滚动体数
    bpfo = 3.58 * f_rotor                 # 外圈故障特征频率 ~179 Hz
    
    # 基础信号：转频 + 谐波 + 噪声
    x = np.zeros(N)
    
    if condition == 'normal':
        x = (0.5 * np.sin(2*np.pi*f_rotor*t) +         # 1×转频
             0.1 * np.sin(2*np.pi*2*f_rotor*t) +        # 2×转频
             0.05 * np.sin(2*np.pi*3*f_rotor*t) +       # 3×转频
             0.3 * rng.standard_normal(N))               # 噪声
             
    elif condition == 'imbalance':
        x = (2.0 * np.sin(2*np.pi*f_rotor*t) +          # 1×转频增大！
             0.15 * np.sin(2*np.pi*2*f_rotor*t) +
             0.05 * np.sin(2*np.pi*3*f_rotor*t) +
             0.3 * rng.standard_normal(N))
             
    elif condition == 'misalignment':
        x = (0.5 * np.sin(2*np.pi*f_rotor*t) +
             1.5 * np.sin(2*np.pi*2*f_rotor*t) +        # 2×转频增大！
             0.8 * np.sin(2*np.pi*3*f_rotor*t) +        # 3×转频增大
             0.3 * rng.standard_normal(N))
             
    elif condition == 'bearing_fault':
        # 轴承故障：周期性冲击 → 激发高频共振
        # 冲击周期 = 1/BPFO
        impact_period = 1.0 / bpfo
        # 生成冲击序列
        impact_train = np.zeros(N)
        impact_indices = np.arange(0, N, int(impact_period * fs))
        impact_train[impact_indices] = 1.0
        
        # 高频共振（衰减振荡）
        f_resonance = 3000  # 共振频率 3kHz
        resonance_decay = 200  # 衰减系数
        impulse_response = np.exp(-resonance_decay * t[:int(0.02*fs)]) * \
                           np.sin(2*np.pi*f_resonance*t[:int(0.02*fs)])
        
        # 冲击通过共振系统
        bearing_signal = np.convolve(impact_train, impulse_response, mode='full')[:N]
        bearing_signal = bearing_signal / np.max(np.abs(bearing_signal))  # 归一化
        
        x = (0.3 * np.sin(2*np.pi*f_rotor*t) +
             0.05 * np.sin(2*np.pi*2*f_rotor*t) +
             1.5 * bearing_signal +                      # 轴承故障信号！
             0.3 * rng.standard_normal(N))
    
    else:
        raise ValueError(f"未知工况: {condition}")
    
    # 第二个传感器（径向，相位偏移）
    y = x * 0.8 + 0.2 * np.roll(x, int(fs*0.001)) + 0.2 * rng.standard_normal(N)
    
    # 构造 DataFrame
    timestamps = pd.date_range('2024-01-01 08:00:00', periods=N, freq=f'{1e6/fs:.0f}us')
    
    df = pd.DataFrame({
        'vibration_axial':     x,
        'vibration_radial':    y,
        'rpm':                 rpm + 5 * rng.standard_normal(N),  # RPM 传感器有噪声
        'condition':           condition,
    }, index=timestamps)
    df.index.name = 'timestamp'
    
    return df
# ═══════════════════════════════════════════
#  生成四种工况的数据
# ═══════════════════════════════════════════
fs = 10000  # 10 kHz 采样率
data = {}
for i, cond in enumerate(['normal', 'imbalance', 'misalignment', 'bearing_fault']):
    data[cond] = generate_vibration_signal(cond, fs=fs, duration=10.0, seed=42+i)
    print(f"{cond:20s}: shape={data[cond].shape}, "
          f"RMS={np.sqrt(np.mean(data[cond]['vibration_axial']**2)):.4f}")
# 合并所有工况（带时间间隔）
df_all = pd.concat([
    data['normal'].assign(run_id=0),
    data['imbalance'].assign(run_id=1),
    data['misalignment'].assign(run_id=2),
    data['bearing_fault'].assign(run_id=3),
], axis=0)
print(f"\n合并数据: {df_all.shape}")
print(df_all.head())
```

### 6.2.2 数据存储与加载
```python
# ═══════════════════════════════════════════
#  保存为 HDF5（生产环境推荐）
# ═══════════════════════════════════════════
df_all.to_hdf('vibration_raw.h5', key='raw', mode='w')
# 加载
df_loaded = pd.read_hdf('vibration_raw.h5', key='raw')
print(f"加载完成: {df_loaded.shape}")
# ═══════════════════════════════════════════
#  也可以保存为 CSV（小数据量）
# ═══════════════════════════════════════════
df_all.to_csv('vibration_raw.csv')
# CSV 读取（指定时间索引）
df_csv = pd.read_csv('vibration_raw.csv', index_col='timestamp', parse_dates=True)
```

---
## 6.3 Step 2：信号预处理
```python
import numpy as np
import pandas as pd
from scipy import signal
import matplotlib.pyplot as plt
# ═══════════════════════════════════════════
#  预处理流水线
# ═══════════════════════════════════════════
class VibrationPreprocessor:
    """振动信号预处理器"""
    
    def __init__(self, fs=10000, rpm=3000):
        self.fs = fs
        self.rpm = rpm
        self.f_rotor = rpm / 60
        self.processing_log = []
        
    def remove_trend(self, x):
        """去除线性趋势"""
        t = np.arange(len(x))
        coeffs = np.polyfit(t, x, deg=1)
        trend = np.polyval(coeffs, t)
        return x - trend
    
    def remove_mean(self, x):
        """去均值"""
        return x - np.mean(x)
    
    def design_bandpass(self, lowcut=5, highcut=4500, order=4):
        """设计带通滤波器"""
        sos = signal.butter(order, [lowcut, highcut], 
                           btype='band', fs=self.fs, output='sos')
        return sos
    
    def design_envelope_filter(self, center_freq=3000, bandwidth=1000, order=4):
        """设计包络分析用带通滤波器（轴承故障诊断）"""
        sos = signal.butter(order, 
                           [center_freq - bandwidth/2, center_freq + bandwidth/2],
                           btype='band', fs=self.fs, output='sos')
        return sos
    
    def apply_filter(self, x, sos):
        """零相位滤波"""
        return signal.sosfiltfilt(sos, x)
    
    def compute_envelope(self, x):
        """计算信号包络（Hilbert 变换）"""
        analytic = signal.hilbert(x)
        return np.abs(analytic)
    
    def remove_outliers(self, x, n_sigma=4):
        """基于 4-sigma 准则去除异常值"""
        mu = np.median(x)
        sigma = np.median(np.abs(x - mu)) * 1.4826  # MAD 估计标准差
        mask = np.abs(x - mu) > n_sigma * sigma
        x_clean = x.copy()
        x_clean[mask] = mu + n_sigma * sigma * np.sign(x[mask])
        return x_clean
    
    def preprocess(self, df, channel='vibration_axial', 
                   do_bandpass=True, do_detrend=True, do_outlier=True,
                   lowcut=5, highcut=4500):
        """
        完整预处理流水线
        
        Returns
        -------
        df_out : DataFrame  含预处理后的数据
        info : dict         预处理信息
        """
        x = df[channel].values.copy()
        info = {'original_length': len(x), 'steps': []}
        
        # Step 1: 去均值
        x = self.remove_mean(x)
        info['steps'].append('remove_mean')
        
        # Step 2: 去趋势
        if do_detrend:
            x = self.remove_trend(x)
            info['steps'].append('remove_trend')
        
        # Step 3: 带通滤波
        if do_bandpass:
            sos = self.design_bandpass(lowcut, highcut)
            x = self.apply_filter(x, sos)
            info['steps'].append(f'bandpass_{lowcut}-{highcut}Hz')
            info['sos_bandpass'] = sos
        
        # Step 4: 去异常
        if do_outlier:
            x = self.remove_outliers(x)
            info['steps'].append('remove_outliers')
        
        # 写回 DataFrame
        df_out = df.copy()
        df_out[f'{channel}_clean'] = x
        
        # 计算包络
        df_out[f'{channel}_envelope'] = self.compute_envelope(x)
        info['steps'].append('envelope')
        
        info['rms_before'] = np.sqrt(np.mean(df[channel].values**2))
        info['rms_after'] = np.sqrt(np.mean(x**2))
        
        return df_out, info
# ═══════════════════════════════════════════
#  对所有工况执行预处理
# ═══════════════════════════════════════════
preprocessor = VibrationPreprocessor(fs=fs, rpm=3000)
processed = {}
for cond, df in data.items():
    df_proc, info = preprocessor.preprocess(df, channel='vibration_axial')
    processed[cond] = df_proc
    print(f"\n{cond}:")
    print(f"  步骤: {' → '.join(info['steps'])}")
    print(f"  RMS: {info['rms_before']:.4f} → {info['rms_after']:.4f}")
# 保存预处理结果
for cond, df in processed.items():
    df.to_hdf('vibration_processed.h5', key=cond, mode='a')
```

---
## 6.4 Step 3：时域与频域特征提取
```python
"""
特征提取模块
- 时域特征：RMS、峰值、峭度、波峰因子、形状因子...
- 频域特征：谱质心、谱峰频率、特定频带能量比...
- 包络特征：包络谱峰值（轴承故障诊断核心）
"""
import numpy as np
from scipy import signal, stats
class FeatureExtractor:
    """振动信号特征提取器"""
    
    def __init__(self, fs=10000, rpm=3000):
        self.fs = fs
        self.rpm = rpm
        self.f_rotor = rpm / 60
    
    # ═══════════════════════════════════════════
    #  时域特征
    # ═══════════════════════════════════════════
    def time_domain_features(self, x):
        """提取 12 个时域特征"""
        N = len(x)
        x_abs = np.abs(x)
        rms = np.sqrt(np.mean(x**2))
        
        features = {}
        features['mean']       = np.mean(x)
        features['std']        = np.std(x, ddof=1)
        features['rms']        = rms
        features['max']        = np.max(x)
        features['min']        = np.min(x)
        features['peak2peak']  = np.max(x) - np.min(x)
        features['crest']      = np.max(x_abs) / (rms + 1e-20)         # 波峰因子
        features['shape']      = rms / (np.mean(x_abs) + 1e-20)        # 形状因子
        features['impulse']    = np.max(x_abs) / (np.mean(x_abs) + 1e-20)  # 脉冲因子
        features['margin']     = np.max(x_abs) / (np.mean(np.sqrt(x_abs))**2 + 1e-20)  # 裕度因子
        features['kurtosis']   = stats.kurtosis(x, fisher=True)        # 峭度（超额峰度）
        features['skewness']   = stats.skew(x)                          # 偏度
        
        return features
    
    # ═══════════════════════════════════════════
    #  频域特征
    # ═══════════════════════════════════════════
    def frequency_domain_features(self, x, nperseg=2048):
        """提取频域特征"""
        f, Pxx = signal.welch(x, fs=self.fs, nperseg=nperseg, window='hann')
        Pxx_sum = np.sum(Pxx) + 1e-20
        Pxx_norm = Pxx / Pxx_sum
        
        features = {}
        
        # 谱质心（重心频率）
        features['spectral_centroid'] = np.sum(f * Pxx_norm)
        
        # 谱方差
        features['spectral_variance'] = np.sum((f - features['spectral_centroid'])**2 * Pxx_norm)
        
        # 谱偏度
        features['spectral_skewness'] = np.sum(
            ((f - features['spectral_centroid'])**3) * Pxx_norm
        ) / (features['spectral_variance']**1.5 + 1e-20)
        
        # 谱峭度
        features['spectral_kurtosis'] = np.sum(
            ((f - features['spectral_centroid'])**4) * Pxx_norm
        ) / (features['spectral_variance']**2 + 1e-20)
        
        # 主频（谱峰值频率）
        peak_idx = np.argmax(Pxx)
        features['dominant_freq'] = f[peak_idx]
        
        # 特定频带能量比
        def band_energy(f_arr, pxx, low, high):
            mask = (f_arr >= low) & (f_arr <= high)
            return np.sum(pxx[mask]) / Pxx_sum
        
        features['E_sub_rotor']    = band_energy(f, Pxx, 0, self.f_rotor * 0.9)
        features['E_1x']           = band_energy(f, Pxx, self.f_rotor * 0.9, self.f_rotor * 1.1)
        features['E_2x']           = band_energy(f, Pxx, self.f_rotor * 1.9, self.f_rotor * 2.1)
        features['E_3x']           = band_energy(f, Pxx, self.f_rotor * 2.9, self.f_rotor * 3.1)
        features['E_high']         = band_energy(f, Pxx, self.f_rotor * 3.1, self.fs / 2)
        
        # 1×/2× 频率比（诊断不对中的关键指标）
        features['ratio_1x_2x'] = features['E_1x'] / (features['E_2x'] + 1e-20)
        
        return features, f, Pxx
    
    # ═══════════════════════════════════════════
    #  包络谱特征（轴承故障诊断核心）
    # ═══════════════════════════════════════════
    def envelope_spectrum_features(self, envelope, nperseg=4096):
        """
        包络谱分析
        
        轴承故障 → 周期性冲击 → 包络谱在故障特征频率处出现峰值
        """
        f_env, Pxx_env = signal.welch(envelope, fs=self.fs, 
                                        nperseg=nperseg, window='hann')
        
        # 轴承特征频率（理论值）
        n_balls = 9
        bpfo = 3.58 * self.f_rotor   # 外圈 ~179 Hz
        bpfi = 5.42 * self.f_rotor   # 内圈 ~271 Hz
        bsf  = 2.36 * self.f_rotor   # 滚动体 ~118 Hz
        ftf  = 0.40 * self.f_rotor   # 保持架 ~20 Hz
        
        features = {}
        
        # 在各特征频率处搜索谱峰值（容差 ±5Hz）
        tolerance = 5.0
        def peak_near(freq_array, pxx, target_freq, tol):
            mask = (freq_array >= target_freq - tol) & (freq_array <= target_freq + tol)
            if np.any(mask):
                return np.max(pxx[mask])
            return 0.0
        
        features['BPFO_amplitude']     = peak_near(f_env, Pxx_env, bpfo, tolerance)
        features['BPFI_amplitude']     = peak_near(f_env, Pxx_env, bpfi, tolerance)
        features['BSF_amplitude']      = peak_near(f_env, Pxx_env, bsf, tolerance)
        features['FTF_amplitude']      = peak_near(f_env, Pxx_env, ftf, tolerance)
        
        # 谐波能量（1×BPFO 到 4×BPFO）
        harmonic_energy = 0
        for k in range(1, 5):
            harmonic_energy += peak_near(f_env, Pxx_env, k * bpfo, tolerance)
        features['BPFO_harmonics'] = harmonic_energy
        
        # 包络谱总能量
        features['envelope_total'] = np.sum(Pxx_env)
        
        return features, f_env, Pxx_env
    
    # ═══════════════════════════════════════════
    #  全部特征一键提取
    # ═══════════════════════════════════════════
    def extract_all(self, x, x_envelope=None):
        """
        提取全部特征
        
        Returns
        -------
        features : dict     全部特征
        spectra  : dict     频谱数据（用于绘图）
        """
        # 时域特征
        t_features = self.time_domain_features(x)
        
        # 频域特征
        f_features, f, Pxx = self.frequency_domain_features(x)
        
        # 包络谱特征
        if x_envelope is not None:
            e_features, f_env, Pxx_env = self.envelope_spectrum_features(x_envelope)
        else:
            e_features, f_env, Pxx_env = {}, np.array([]), np.array([])
        
        # 合并
        all_features = {**t_features, **f_features, **e_features}
        spectra = {
            'freq': f, 'psd': Pxx,
            'freq_env': f_env, 'psd_env': Pxx_env,
        }
        
        return all_features, spectra
# ═══════════════════════════════════════════
#  对所有工况提取特征
# ═══════════════════════════════════════════
extractor = FeatureExtractor(fs=fs, rpm=3000)
results = {}
spectra_data = {}
for cond, df in processed.items():
    x_clean = df['vibration_axial_clean'].values
    x_env   = df['vibration_axial_envelope'].values
    
    features, spectra = extractor.extract_all(x_clean, x_env)
    
    results[cond] = features
    spectra_data[cond] = spectra
    
    print(f"\n{'='*60}")
    print(f"工况: {cond}")
    print(f"{'='*60}")
    print(f"  时域: RMS={features['rms']:.4f}, 峭度={features['kurtosis']:.4f}, "
          f"波峰因子={features['crest']:.4f}")
    print(f"  频域: 主频={features['dominant_freq']:.1f}Hz, "
          f"1×能量={features['E_1x']:.4f}, 2×能量={features['E_2x']:.4f}")
    print(f"  包络: BPFO={features['BPFO_amplitude']:.4e}, "
          f"BPFI={features['BPFI_amplitude']:.4e}")
# ═══════════════════════════════════════════
#  构建特征矩阵
# ═══════════════════════════════════════════
df_features = pd.DataFrame(results).T
df_features.index.name = 'condition'
print("\n特征矩阵:")
print(df_features.round(4))
print(f"\n形状: {df_features.shape}")
```

---
## 6.5 Step 4：滑动窗口特征提取（连续监测）
```python
"""
对长信号做分段特征提取，模拟在线监测场景
"""
import numpy as np
import pandas as pd
from scipy import signal, stats
def sliding_window_features(df, channel='vibration_axial_clean',
                             fs=10000, window_sec=1.0, step_sec=0.5):
    """
    滑动窗口特征提取
    
    Parameters
    ----------
    df : DataFrame        预处理后的信号数据
    channel : str         信号列名
    fs : int              采样率
    window_sec : float    窗口长度（秒）
    step_sec : float      步进长度（秒）
    
    Returns
    -------
    df_feat : DataFrame   每个窗口的特征
    """
    x = df[channel].values
    envelope = df[f'{channel.replace("_clean","")}_envelope'].values
    
    window_size = int(window_sec * fs)
    step_size = int(step_sec * fs)
    n_windows = (len(x) - window_size) // step_size + 1
    
    extractor = FeatureExtractor(fs=fs, rpm=3000)
    feat_list = []
    
    for i in range(n_windows):
        start = i * step_size
        end = start + window_size
        
        x_win = x[start:end]
        env_win = envelope[start:end]
        
        # 快速特征（不用 Welch，用简单 FFT 提高频域特征速度）
        t_feat = extractor.time_domain_features(x_win)
        
        # 简化频域特征：用 FFT 代替 Welch
        N_fft = len(x_win)
        X = np.fft.rfft(x_win)
        freqs = np.fft.rfftfreq(N_fft, 1/fs)
        mag = np.abs(X) / N_fft * 2
        
        # 快速频域指标
        f_rotor = 3000 / 60
        
        feat = {**t_feat}
        feat['dominant_freq'] = freqs[np.argmax(mag[1:]) + 1]  # 跳过直流
        feat['E_1x'] = np.sum(mag[(freqs >= f_rotor*0.9) & (freqs <= f_rotor*1.1)]**2)
        feat['E_2x'] = np.sum(mag[(freqs >= f_rotor*1.9) & (freqs <= f_rotor*2.1)]**2)
        feat['E_3x'] = np.sum(mag[(freqs >= f_rotor*2.9) & (freqs <= f_rotor*3.1)]**2)
        
        # 包络 RMS
        feat['envelope_rms'] = np.sqrt(np.mean(env_win**2))
        
        # 时间戳
        feat['window_start'] = start / fs
        feat['window_end'] = end / fs
        
        feat_list.append(feat)
    
    df_feat = pd.DataFrame(feat_list)
    
    return df_feat
# ═══════════════════════════════════════════
#  对所有工况做滑动窗口特征提取
# ═══════════════════════════════════════════
window_features = {}
for cond, df in processed.items():
    window_features[cond] = sliding_window_features(
        df, fs=fs, window_sec=1.0, step_sec=0.5
    )
    print(f"{cond}: {window_features[cond].shape}")
# 合并所有工况的特征
df_all_features = pd.concat([
    wf.assign(condition=cond) for cond, wf in window_features.items()
], axis=0).reset_index(drop=True)
print(f"\n全部特征: {df_all_features.shape}")
print(df_all_features.groupby('condition')['rms'].agg(['mean', 'std']).round(4))
```

---
## 6.6 Step 5：综合可视化
### 6.6.1 四工况对比图
```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
from matplotlib.patches import Patch
plt.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False
def plot_condition_comparison(data, processed, spectra_data, fs=10000):
    """
    绘制四种工况的全面对比图
    布局：4行 × 4列
    - 行：normal, imbalance, misalignment, bearing_fault
    - 列1：时域波形
    - 列2：频谱
    - 列3：包络谱
    - 列4：时域特征趋势
    """
    conditions = ['normal', 'imbalance', 'misalignment', 'bearing_fault']
    cond_labels = ['正常', '不平衡', '不对中', '轴承故障']
    
    fig = plt.figure(figsize=(20, 16))
    gs = gridspec.GridSpec(4, 4, figure=fig, 
                           hspace=0.35, wspace=0.3,
                           width_ratios=[2, 2, 2, 1.5])
    
    f_rotor = 3000 / 60  # 50 Hz
    
    for row, (cond, label) in enumerate(zip(conditions, cond_labels)):
        df = processed[cond]
        x = df['vibration_axial_clean'].values
        t = np.arange(len(x)) / fs
        spec = spectra_data[cond]
        
        # ─── 列1：时域波形（前 0.2 秒）───
        ax1 = fig.add_subplot(gs[row, 0])
        mask_t = t <= 0.2
        ax1.plot(t[mask_t] * 1000, x[mask_t], linewidth=0.5, color='#1f77b4')
        ax1.set_ylabel(f'{label}\n幅度', fontsize=9)
        if row == 0:
            ax1.set_title('时域波形 (0~200ms)', fontsize=11)
        if row == 3:
            ax1.set_xlabel('时间
        ax1.grid(True, alpha=0.3)
        ax1.set_xlim([0, 200])
        
        # ─── 列2：频谱 ───
        ax2 = fig.add_subplot(gs[row, 1])
        f, Pxx = spec['freq'], spec['psd']
        Pxx_db = 10 * np.log10(Pxx + 1e-20)
        ax2.plot(f, Pxx_db, linewidth=0.8, color='#d62728')
        
        # 标注转频及谐波
        for k in range(1, 6):
            ax2.axvline(k * f_rotor, color='green', linestyle='--', 
                       alpha=0.4, linewidth=0.8)
            if row == 0:
                ax2.text(k * f_rotor, ax2.get_ylim()[1], f'{k}×',
                        ha='center', fontsize=7, color='green')
        
        ax2.set_xlim([0, 500])
        ax2.set_ylim([Pxx_db[f<500].min() - 5, Pxx_db[f<500].max() + 5])
        if row == 0:
            ax2.set_title('功率谱', fontsize=11)
        if row == 3:
            ax2.set_xlabel('频率
        ax2.grid(True, alpha=0.3)
        
        # ─── 列3：包络谱 ───
        ax3 = fig.add_subplot(gs[row, 2])
        f_env, Pxx_env = spec['freq_env'], spec['psd_env']
        Pxx_env_db = 10 * np.log10(Pxx_env + 1e-20)
        ax3.plot(f_env, Pxx_env_db, linewidth=0.8, color='#9467bd')
        
        # 标注轴承特征频率
        bpfo = 3.58 * f_rotor
        bpfi = 5.42 * f_rotor
        ax3.axvline(bpfo, color='red', linestyle='--', alpha=0.6, label=f'BPFO={bpfo:.0f}Hz')
        ax3.axvline(bpfi, color='orange', linestyle='--', alpha=0.6, label=f'BPFI={bpfi:.0f}Hz')
        
        ax3.set_xlim([0, 500])
        ax3.set_ylim([Pxx_env_db[f_env<500].min() - 5, Pxx_env_db[f_env<500].max() + 5])
        if row == 0:
            ax3.set_title('包络谱', fontsize=11)
            ax3.legend(fontsize=7, loc='upper right')
        if row == 3:
            ax3.set_xlabel('频率
        ax3.grid(True, alpha=0.3)
        
        # ─── 列4：特征指标柱状图 ───
        ax4 = fig.add_subplot(gs[row, 3])
        feat = results[cond]
        indicators = {
            'RMS': feat['rms'],
            '峭度': feat['kurtosis'],
            '波峰因子': feat['crest'],
            '1×/2×': feat['ratio_1x_2x'],
        }
        colors = ['#2ca02c', '#d62728', '#ff7f0e', '#1f77b4']
        bars = ax4.barh(list(indicators.keys()), list(indicators.values()), 
                        color=colors, alpha=0.7, edgecolor='black', linewidth=0.5)
        if row == 0:
            ax4.set_title('关键指标', fontsize=11)
        ax4.grid(True, alpha=0.3, axis='x')
    
    fig.suptitle('旋转机械振动信号 — 四种工况对比分析', fontsize=14, fontweight='bold', y=0.98)
    plt.savefig('condition_comparison.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("✅ 工况对比图已保存")
# 调用
plot_condition_comparison(data, processed, spectra_data, fs)
```

### 6.6.2 特征趋势图（连续监测）
```python
def plot_feature_trends(df_all_features):
    """绘制滑动窗口特征随时间的变化趋势"""
    
    conditions = ['normal', 'imbalance', 'misalignment', 'bearing_fault']
    cond_labels = {'normal': '正常', 'imbalance': '不平衡',
                   'misalignment': '不对中', 'bearing_fault': '轴承故障'}
    colors = {'normal': '#2ca02c', 'imbalance': '#d62728',
              'misalignment': '#ff7f0e', 'bearing_fault': '#9467bd'}
    
    feature_cols = ['rms', 'kurtosis', 'crest', 'dominant_freq', 'envelope_rms']
    feature_names = ['RMS', '峭度', '波峰因子', '主频', '包络RMS']
    
    fig, axes = plt.subplots(len(feature_cols), 1, 
                              figsize=(14, 3*len(feature_cols)), sharex=True)
    
    for ax, col, name in zip(axes, feature_cols, feature_names):
        for cond in conditions:
            subset = df_all_features[df_all_features['condition'] == cond]
            ax.plot(subset['window_start'], subset[col], 
                    marker='o', markersize=2, linewidth=1,
                    color=colors[cond], label=cond_labels[cond], alpha=0.8)
        
        ax.set_ylabel(name, fontsize=11)
        ax.legend(fontsize=8, ncol=4, loc='upper right')
        ax.grid(True, alpha=0.3)
    
    axes[-1].set_xlabel('时间
    axes[0].set_title('特征趋势图 — 滑动窗口（1s窗口，0.5s步进）', fontsize=13)
    
    plt.tight_layout()
    plt.savefig('feature_trends.png', dpi=150)
    plt.show()
    print("✅ 特征趋势图已保存")
plot_feature_trends(df_all_features)
```

### 6.6.3 雷达图（特征对比）
```python
def plot_radar_chart(df_features, features_to_plot=None):
    """雷达图对比不同工况的特征"""
    
    if features_to_plot is None:
        features_to_plot = ['rms', 'kurtosis', 'crest', 'shape', 
                           'E_1x', 'E_2x', 'BPFO_amplitude']
    
    labels_short = {
        'rms': 'RMS', 'kurtosis': '峭度', 'crest': '波峰因子',
        'shape': '形状因子', 'E_1x': '1×能量', 'E_2x': '2×能量',
        'BPFO_amplitude': 'BPFO',
    }
    
    cond_labels = {'normal': '正常', 'imbalance': '不平衡',
                   'misalignment': '不对中', 'bearing_fault': '轴承故障'}
    colors = {'normal': '#2ca02c', 'imbalance': '#d62728',
              'misalignment': '#ff7f0e', 'bearing_fault': '#9467bd'}
    
    # 归一化到 [0, 1]
    df_norm = df_features[features_to_plot].copy()
    for col in df_norm.columns:
        col_min = df_norm[col].min()
        col_max = df_norm[col].max()
        df_norm[col] = (df_norm[col] - col_min) / (col_max - col_min + 1e-20)
    
    # 雷达图
    n_vars = len(features_to_plot)
    angles = np.linspace(0, 2*np.pi, n_vars, endpoint=False).tolist()
    angles += angles[:1]  # 闭合
    
    fig, ax = plt.subplots(figsize=(8, 8), subplot_kw=dict(polar=True))
    
    for cond in df_norm.index:
        values = df_norm.loc[cond].values.tolist()
        values += values[:1]  # 闭合
        ax.plot(angles, values, 'o-', linewidth=2, 
                color=colors.get(cond, 'gray'), label=cond_labels.get(cond, cond))
        ax.fill(angles, values, alpha=0.1, color=colors.get(cond, 'gray'))
    
    ax.set_xticks(angles[:-1])
    ax.set_xticklabels([labels_short.get(f, f) for f in features_to_plot], fontsize=11)
    ax.set_ylim(0, 1.1)
    ax.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1), fontsize=10)
    ax.set_title('工况特征雷达图', fontsize=13, pad=20)
    
    plt.tight_layout()
    plt.savefig('radar_chart.png', dpi=150, bbox_inches='tight')
    plt.show()
plot_radar_chart(df_features)
```

---
## 6.7 Step 6：统计分析与特征选择
```python
"""
分析哪些特征最能区分不同工况
"""
import numpy as np
import pandas as pd
from scipy import stats
# ═══════════════════════════════════════════
#  单因素方差分析（ANOVA）
# ═══════════════════════════════════════════
def anova_analysis(df, feature_cols, group_col='condition'):
    """
    对每个特征做 ANOVA 检验，评估不同组间的差异显著性
    
    Returns
    -------
    df_anova : DataFrame  含 F 值、p 值、显著性标记
    """
    groups = df[group_col].unique()
    results = []
    
    for col in feature_cols:
        group_data = [df[df[group_col] == g][col].dropna().values for g in groups]
        
        # 正态性检验（Shapiro-Wilk）
        normality = all(stats.shapiro(g)[1] > 0.05 for g in group_data if len(g) > 20)
        
        # ANOVA
        f_val, p_val = stats.f_oneway(*group_data)
        
        # 效应量 η²
        ss_between = sum(len(g) * (g.mean() - df[col].mean())**2 for g in group_data)
        ss_total = np.sum((df[col] - df[col].mean())**2)
        eta_sq = ss_between / (ss_total + 1e-20)
        
        results.append({
            'feature': col,
            'F_value': f_val,
            'p_value': p_val,
            'eta_squared': eta_sq,
            'significant': '***' if p_val < 0.001 else '**' if p_val < 0.01 else '*' if p_val < 0.05 else 'n.s.',
        })
    
    df_anova = pd.DataFrame(results).sort_values('F_value', ascending=False)
    return df_anova
# 数值特征列
feature_cols = [c for c in df_all_features.columns 
                if c not in ['window_start', 'window_end', 'condition']
                and df_all_features[c].dtype in [np.float64, np.float32, np.int64]]
df_anova = anova_analysis(df_all_features, feature_cols)
print("ANOVA 分析结果（按 F 值排序）：")
print(df_anova[['feature', 'F_value', 'p_value', 'eta_squared', 'significant']].to_string(index=False))
# 选出最显著的 5 个特征
top_features = df_anova.head(5)['feature'].tolist()
print(f"\nTop 5 区分特征: {top_features}")
# ═══════════════════════════════════════════
#  特征相关性热力图
# ═══════════════════════════════════════════
corr_matrix = df_all_features[feature_cols].corr()
fig, ax = plt.subplots(figsize=(12, 10))
im = ax.imshow(corr_matrix, cmap='RdBu_r', vmin=-1, vmax=1, aspect='auto')
ax.set_xticks(range(len(feature_cols)))
ax.set_yticks(range(len(feature_cols)))
ax.set_xticklabels(feature_cols, rotation=45, ha='right', fontsize=8)
ax.set_yticklabels(feature_cols, fontsize=8)
fig.colorbar(im, ax=ax, label='相关系数')
# 标注数值
for i in range(len(feature_cols)):
    for j in range(len(feature_cols)):
        text = ax.text(j, i, f'{corr_matrix.iloc[i, j]:.2f}',
                       ha='center', va='center', fontsize=6,
                       color='white' if abs(corr_matrix.iloc[i, j]) > 0.5 else 'black')
ax.set_title('特征相关性矩阵', fontsize=13)
plt.tight_layout()
plt.savefig('correlation_matrix.png', dpi=150)
plt.show()
```

---
## 6.8 Step 7：简易故障分类器
```python
"""
基于规则的故障分类器 + 简单机器学习分类器
"""
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler
# ═══════════════════════════════════════════
#  方法1：基于专家规则的分类器
# ═══════════════════════════════════════════
def rule_based_classifier(features_dict):
    """
    基于领域知识的规则分类器
    
    规则：
    - 正常：RMS 低，峭度接近 3，1×/2× 比值正常
    - 不平衡：1× 频率能量显著增大
    - 不对中：2× 频率能量显著增大（1×/2× 比值 < 1）
    - 轴承故障：BPFO 包络谱幅值显著增大
    """
    f = features_dict
    
    # 特征阈值
    rms_high = 1.5
    ratio_1x_2x_low = 1.0
    bpfo_high = 1e-4
    
    # 决策逻辑
    if f['BPFO_amplitude'] > bpfo_high and f['kurtosis'] > 4:
        return 'bearing_fault'
    elif f['E_2x'] > f['E_1x'] * 1.5:
        return 'misalignment'
    elif f['E_1x'] > 0.05 and f['ratio_1x_2x'] > 2:
        return 'imbalance'
    else:
        return 'normal'
# 测试规则分类器
print("规则分类器测试：")
for cond, features in results.items():
    prediction = rule_based_classifier(features)
    correct = "✅" if prediction == cond else "❌"
    print(f"  实际: {cond:20s} → 预测: {prediction:20s} {correct}")
# ═══════════════════════════════════════════
#  方法2：随机森林分类器
# ═══════════════════════════════════════════
# 准备数据
X = df_all_features[feature_cols].values
y = df_all_features['condition'].values
# 标准化
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
# 训练 + 5 折交叉验证
clf = RandomForestClassifier(n_estimators=100, random_state=42, max_depth=5)
scores = cross_val_score(clf, X_scaled, y, cv=5, scoring='accuracy')
print(f"\n随机森林分类器：")
print(f"  5折交叉验证准确率: {scores.mean():.4f} ± {scores.std():.4f}")
# 训练最终模型
clf.fit(X_scaled, y)
# 特征重要性
importance = pd.Series(clf.feature_importances_, index=feature_cols)
importance = importance.sort_values(ascending=True)
# 绘制特征重要性
fig, ax = plt.subplots(figsize=(10, 6))
importance.tail(10).plot.barh(ax=ax, color='steelblue', edgecolor='black', linewidth=0.5)
ax.set_xlabel('特征重要性')
ax.set_title('随机森林 — Top 10 特征重要性')
ax.grid(True, alpha=0.3, axis='x')
plt.tight_layout()
plt.savefig('feature_importance.png', dpi=150)
plt.show()
```

---
## 6.9 Step 8：报告输出

```python
"""
生成分析报告：图片 + CSV + 摘要文本
"""
import numpy as np
import pandas as pd
from datetime import datetime
def generate_report(df_features, df_anova, results, spectra_data, fs):
    """生成完整分析报告"""
    
    report_lines = []
    report_lines.append("=" * 70)
    report_lines.append("   旋转机械振动信号分析报告")
    report_lines.append(f"   生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    report_lines.append("=" * 70)
    report_lines.append("")
    
    # 1. 数据概览
    report_lines.append("一、数据概览")
    report_lines.append("-" * 40)
    report_lines.append(f"  采样率: {fs} Hz")
    report_lines.append(f"  信号时长: 10 秒/工况")
    report_lines.append(f"  工况数量: {len(results)}")
    report_lines.append("")
    
    # 2. 各工况特征汇总
    report_lines.append("二、各工况关键特征汇总")
    report_lines.append("-" * 40)
    
    for cond in df_features.index:
        f = df_features.loc[cond]
        report_lines.append(f"\n  [{cond}]")
        report_lines.append(f"    RMS:        {f['rms']:.4f}")
        report_lines.append(f"    峭度:       {f['kurtosis']:.4f}  (正常≈3)")
        report_lines.append(f"    波峰因子:   {f['crest']:.4f}  (正常≈1.414)")
        report_lines.append(f"    主频:       {f['dominant_freq']:.1f} Hz")
        report_lines.append(f"    1×能量比:   {f['E_1x']:.4f}")
        report_lines.append(f"    2×能量比:   {f['E_2x']:.4f}")
        report_lines.append(f"    1×/2×比:    {f['ratio_1x_2x']:.4f}")
        report_lines.append(f"    BPFO幅值:   {f['BPFO_amplitude']:.4e}")
    
    report_lines.append("")
    
    # 3. 诊断结论
    report_lines.append("三、诊断结论")
    report_lines.append("-" * 40)
    
    for cond, features in results.items():
        diagnosis = rule_based_classifier(features)
        report_lines.append(f"\n  [{cond}] → 诊断: {diagnosis}")
        
        if diagnosis == 'imbalance':
            report_lines.append("    依据: 1×转频能量显著增大，1×/2×比值偏高")
            report_lines.append("    建议: 检查转子动平衡")
        elif diagnosis == 'misalignment':
            report_lines.append("    依据: 2×转频能量超过1×，1×/2×比值 < 1")
            report_lines.append("    建议: 检查联轴器对中")
        elif diagnosis == 'bearing_fault':
            report_lines.append("    依据: 包络谱 BPFO 特征频率峰值显著，峭度异常")
            report_lines.append("    建议: 更换轴承或安排检修")
        else:
            report_lines.append("    依据: 各项指标在正常范围内")
            report_lines.append("    建议: 继续监测")
    
    report_lines.append("")
    
    # 4. ANOVA 结果
    report_lines.append("四、特征显著性分析 (ANOVA)")
    report_lines.append("-" * 40)
    report_lines.append(f"  最显著特征 (Top 5):")
    for _, row in df_anova.head(5).iterrows():
        report_lines.append(f"    {row['feature']:25s} F={row['F_value']:.2f}, "
                          f"p={row['p_value']:.2e}, η²={row['eta_squared']:.4f} {row['significant']}")
    
    report_lines.append("")
    report_lines.append("=" * 70)
    report_lines.append("报告结束")
    
    # 写入文件
    report_text = "\n".join(report_lines)
    
    with open('analysis_report.txt', 'w', encoding='utf-8') as f:
        f.write(report_text)
    
    print(report_text)
    return report_text
# ═══════════════════════════════════════════
#  执行报告生成
# ═══════════════════════════════════════════
report = generate_report(df_features, df_anova, results, spectra_data, fs)
# ═══════════════════════════════════════════
#  保存所有结果
# ═══════════════════════════════════════════
# 特征矩阵
df_features.to_csv('features_matrix.csv')
df_features.to_excel('features_matrix.xlsx')
# 滑动窗口特征
df_all_features.to_csv('window_features.csv')
df_all_features.to_hdf('analysis_results.h5', key='window_features', mode='w')
# ANOVA 结果
df_anova.to_csv('anova_results.csv', index=False)
print("\n✅ 所有结果已保存：")
print("  📄 analysis_report.txt")
print("  📊 features_matrix.csv / .xlsx")
print("  📊 window_features.csv")
print("  📊 anova_results.csv")
print("  🖼️ condition_comparison.png")
print("  🖼️ feature_trends.png")
print("  🖼️ radar_chart.png")
print("  🖼️ correlation_matrix.png")
print("  🖼️ feature_importance.png")
```

---
## 6.10 完整流水线一键运行
```python
"""
一键运行全部流水线
"""
import numpy as np
import pandas as pd
from scipy import signal
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
from datetime import datetime
def main():
    """完整信号处理流水线"""
    
    print("=" * 60)
    print("  旋转机械振动信号故障诊断 — 自动分析流水线")
    print(f"  开始时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print("=" * 60)
    
    # ─── 参数设置 ───
    fs = 10000
    rpm = 3000
    duration = 10.0
    
    # ─── Step 1: 数据生成 ───
    print("\n[1/7] 生成模拟数据...")
    conditions = ['normal', 'imbalance', 'misalignment', 'bearing_fault']
    data = {}
    for i, cond in enumerate(conditions):
        data[cond] = generate_vibration_signal(cond, fs=fs, duration=duration, seed=42+i)
        print(f"  {cond}: ✓")
    
    # ─── Step 2: 预处理 ───
    print("\n[2/7] 信号预处理...")
    preprocessor = VibrationPreprocessor(fs=fs, rpm=rpm)
    processed = {}
    for cond, df in data.items():
        processed[cond], info = preprocessor.preprocess(df)
        print(f"  {cond}: RMS {info['rms_before']:.4f} → {info['rms_after']:.4f}")
    
    # ─── Step 3: 特征提取 ───
    print("\n[3/7] 特征提取...")
    extractor = FeatureExtractor(fs=fs, rpm=rpm)
    results = {}
    spectra_data = {}
    for cond, df in processed.items():
        x_clean = df['vibration_axial_clean'].values
        x_env = df['vibration_axial_envelope'].values
        results[cond], spectra_data[cond] = extractor.extract_all(x_clean, x_env)
    df_features = pd.DataFrame(results).T
    print(f"  特征矩阵: {df_features.shape}")
    
    # ─── Step 4: 滑动窗口特征 ───
    print("\n[4/7] 滑动窗口特征提取...")
    window_features = {}
    for cond, df in processed.items():
        window_features[cond] = sliding_window_features(df, fs=fs)
    df_all_features = pd.concat([
        wf.assign(condition=cond) for cond, wf in window_features.items()
    ], axis=0).reset_index(drop=True)
    print(f"  窗口特征: {df_all_features.shape}")
    
    # ─── Step 5: ANOVA 分析 ───
    print("\n[5/7] 统计分析...")
    feature_cols = [c for c in df_all_features.columns 
                    if c not in ['window_start', 'window_end', 'condition']
                    and df_all_features[c].dtype in [np.float64, np.float32, np.int64]]
    df_anova = anova_analysis(df_all_features, feature_cols)
    print(f"  显著特征: {df_anova.head(5)['feature'].tolist()}")
    
    # ─── Step 6: 可视化 ───
    print("\n[6/7] 生成可视化图表...")
    plot_condition_comparison(data, processed, spectra_data, fs)
    plot_feature_trends(df_all_features)
    plot_radar_chart(df_features)
    print("  图表已保存")
    
    # ─── Step 7: 报告 ───
    print("\n[7/7] 生成分析报告...")
    generate_report(df_features, df_anova, results, spectra_data, fs)
    
    print(f"\n{'='*60}")
    print(f"  ✅ 分析完成！耗时: {datetime.now().strftime('%H:%M:%S')}")
    print(f"{'='*60}")
# 运行
if __name__ == '__main__':
    main()
```

---
## 6.11 全书知识体系总结
```
┌─────────────────────────────────────────────────────────────────────┐
│                  Python 信号处理知识体系                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │  NumPy   │  │  SciPy   │  │Matplotlib│  │  Pandas  │           │
│  │  基础运算 │  │  专业算法 │  │   可视化  │  │  数据管理 │           │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘           │
│       │              │              │              │                 │
│  ┌────┴──────────────┴──────────────┴──────────────┴─────┐         │
│  │                    协作流程                            │         │
│  │                                                       │         │
│  │  Pandas ──读取──→ NumPy ──计算──→ SciPy ──分析──→     │         │
│  │    ↑                                          │       │         │
│  │    └──────────────存储─────────────────────────┘       │         │
│  │                         │                              │         │
│  │                    Matplotlib ──可视化                  │         │
│  └───────────────────────────────────────────────────────┘         │
│                                                                     │
│  核心一句话：                                                       │
│  Pandas 管数据，NumPy 做计算，SciPy 算算法，Matplotlib 画图表      │
└─────────────────────────────────────────────────────────────────────┘
```
### 各库核心 10 函数速记

| #    | NumPy                     | SciPy (signal)         | Matplotlib                 | Pandas           |
| ---- | ------------------------- | ---------------------- | -------------------------- | ---------------- |
| 1    | `np.array()`              | `signal.butter()`      | `fig, ax = plt.subplots()` | `pd.DataFrame()` |
| 2    | `np.linspace()`           | `signal.firwin()`      | `ax.plot()`                | `pd.read_csv()`  |
| 3    | `np.fft.fft()`            | `signal.sosfiltfilt()` | `ax.set_title()`           | `df['col']`      |
| 4    | `np.abs()`                | `signal.welch()`       | `ax.set_xlabel()`          | `df.loc[]`       |
| 5    | `np.angle()`              | `signal.stft()`        | `ax.legend()`              | `df.groupby()`   |
| 6    | `np.convolve()`           | `signal.freqz()`       | `ax.grid()`                | `df.rolling()`   |
| 7    | `np.argmax()`             | `signal.hilbert()`     | `ax.set_xlim()`            | `df.resample()`  |
| 8    | `np.random.default_rng()` | `signal.find_peaks()`  | `fig.savefig()`            | `df.merge()`     |
| 9    | `np.where()`              | `signal.chirp()`       | `ax.stem()`                | `df.describe()`  |
| 10   | `np.reshape()`            | `signal.resample()`    | `ax.pcolormesh()`          | `df.to_hdf()`    |

---

> **全文完。** 从 MATLAB 到 Python 的信号处理学习指南全部六章已输出完毕。涵盖了：思维转换、NumPy 基础运算与 FFT、SciPy 滤波器设计与频谱分析、Matplotlib 专业绑图、Pandas 时间序列管理，以及一个端到端的综合实战项目。祝学习顺利！
