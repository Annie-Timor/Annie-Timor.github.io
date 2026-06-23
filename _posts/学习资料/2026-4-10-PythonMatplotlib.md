---
title: Python 数据可视化完全指南
author: lixinghui
date: 2026-4-10 12:00:00 +0800
categories: [Note, Python]
tags: [学习资料]
---

# 📈 Python 数据可视化完全指南：从入门到专业的 Matplotlib 实战教程
> 在数据分析和上位机开发中，数据可视化是不可或缺的一环。Matplotlib 是 Python 中最基础、最强大的 2D 绘图库。本文将从零开始，带你一步步掌握 Matplotlib 的核心逻辑，并结合实战代码，教你如何绘制出专业级、可直接用于报告或论文的图表。
> 本教程涵盖：基础画图、线条美化、坐标轴控制（线性/对数）、多子图布局、双Y轴共用X轴、辅助参考线、区域填充、智能刻度格式化等核心技能。

---

## 〇、前置准备：解决中文显示“方块”问题
Matplotlib 默认字体不支持中文字符，直接写中文会显示为一个个方框 `□□□`。在写任何绘图代码前，**必须**先配置中文字体。

```python
import matplotlib.pyplot as plt
import numpy as np
# --- 最通用的中文设置方法 ---
# Windows 用户通常使用 'SimHei' (黑体) 或 'Microsoft YaHei'
# Mac 用户通常使用 'Arial Unicode MS' 或 'PingFang SC'
plt.rcParams['font.sans-serif'] = ['SimHei']  # 设置默认字体为黑体
plt.rcParams['axes.unicode_minus'] = False    # 解决保存图像时负号'-'显示为方块的问题
```
---
## 一、核心概念：Figure 与 Axes
很多新手喜欢用 `plt.plot()` 直接画图，这在画简单图形时没问题，但在绘制复杂图表时会遇到瓶颈。专业做法是区分 **Figure（画板）** 和 **Axes（坐标系/子图）**。
*   **Figure**：相当于一块物理画布，可以包含一个或多个 Axes。
*   **Axes**：相当于画布上的一块区域（带有 X 轴、Y 轴），**这是你真正画图和数据操作的地方**。

```python
# 创建一个 Figure 对象和一个 Axes 对象
fig, ax = plt.subplots(figsize=(8, 4), dpi=100) 
# figsize: 设置画板大小 (宽, 高) 单位英寸
# dpi: 分辨率 (每英寸点数)
# 准备测试数据
x = np.linspace(0, 10, 100)
y = np.sin(x)
# 画线
ax.plot(x, y)
plt.show()
```

---

## 二、线条、颜色与标记（基础美化）
使用 `plot` 函数的格式化字符串 `fmt = '[marker][line][color]'` 可以快速设置样式，但更推荐使用关键字参数进行精细控制。
### 2.1 快捷设置 (格式化字符串)
```python
fig, ax = plt.subplots()
ax.plot(x, y, 'r--') 
# 'r' = red (红色), '--' = 虚线
# 'go:' = green (绿色), 'o' = 圆点标记, ':' = 点线

plt.show()
```
### 2.2 完整属性设置（推荐，更灵活）
通过关键字参数可以精细控制线条的每一个细节。
```python
fig, ax = plt.subplots()

# plot 的主要参数
ax.plot(x, y, 
        color='#FF5733',       # 线条颜色 (支持英文单词、十六进制、RGB元组)
        linestyle='-.',        # 线型: '-', '--', '-.', ':', 'None'
        linewidth=2.5,         # 线宽
        marker='s',            # 标记点形状: 'o'圆, 's'方, '^'三角, 'D'菱形, '*'星
        markersize=8,          # 标记点大小
        markerfacecolor='blue',# 标记点填充色
        markeredgecolor='black',# 标记点边缘色
        label='正弦波曲线'      # 图例名称 (后续会用到)
)

plt.show()
```
---
## 三、坐标轴设置、标题与网格
这部分是提升图表专业度、可读性的关键。
### 3.1 标题、轴名称与显示范围
```python
fig, ax = plt.subplots()
ax.plot(x, y)

# 设置标题
ax.set_title("这是一幅正弦波图像", fontsize=16, color='blue', loc='center')

# 设置 X/Y 轴名称
ax.set_xlabel("时间 (秒)", fontsize=12)
ax.set_ylabel("电压 (V)", fontsize=12)

# 设置 X/Y 轴的数值显示范围 (尺度)
ax.set_xlim(0, 15)   # X 轴从 0 到 15
ax.set_ylim(-2, 2)   # Y 轴从 -2 到 2

# 3.2 添加网格线
# axis='both' 表示X/Y轴都加网格 (可选 'x', 'y')
# linestyle/-- 设置网格线型，alpha 设置透明度
ax.grid(True, axis='both', linestyle='--', alpha=0.6) 

plt.show()
```
### 3.2 自定义刻度 与网格
有时候默认的数字太多或格式不对，我们需要自定义。
```python
fig, ax = plt.subplots()
ax.plot(x, y)

# 方法1：只改变文本，不改变位置
# plt.xticks(rotation=45) # 将X轴标签旋转45度

# 方法2：使用 set_xticks 设置位置，set_xticklabels 设置文本
ax.set_xticks([0, 2.5, 5, 7.5, 10])
ax.set_xticklabels(['开始', '阶段1', '阶段2', '阶段3', '结束'])

plt.show()
```
---
## 四、线性轴 与 对数轴 切换
在科学计算、信号处理或上位机频率分析中，对数坐标极其常见。Matplotlib 提供了极其方便的 `set_xscale` 和 `set_yscale` 方法。
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

# 准备指数增长数据
x_exponential = np.linspace(0, 5, 100)
y_exponential = np.exp(x_exponential)

# 左图：标准的线性轴
ax1.plot(x_exponential, y_exponential, 'r-', label='线性轴')
ax1.set_title("标准线性轴 (Linear)")
ax1.set_yscale('linear') # 也可以不写，默认就是linear
ax1.grid(True)

# 右图：Y轴对数轴 (logarithmic)
ax2.plot(x_exponential, y_exponential, 'b-', label='对数轴')
ax2.set_title("Y轴对数轴 (Log)")
ax2.set_yscale('log')   # 设置Y轴为对数
# 可选参数: base=10 或 base=2 (默认以10为底)
ax2.grid(True, which="both") # which='both' 显示主次网格线，对数坐标下非常实用

plt.tight_layout() # 自动调整子图间距，防止重叠
plt.show()
```
---
## 五、图例 的使用
当一张图里有多条线时，图例是必不可少的。前提是你在 `plot` 时传入了 `label` 参数。
```python
fig, ax = plt.subplots()
ax.plot(x, np.sin(x), label='Sin(x)')
ax.plot(x, np.cos(x), label='Cos(x)')
# 添加图例
# loc 控制位置: 'best' (自动找最佳位置), 'upper right', 'lower left' 等
ax.legend(loc='best', fontsize=12, framealpha=0.9)
plt.show()
```
---
## 六、多子图 布局详解
使用 `plt.subplots` 可以轻松创建网格布局。
### 6.1 基础多子图 (2行2列)
`plt.subplots(行数, 列数)` 返回一个二维的 Axes 数组。
```python
fig, axes = plt.subplots(2, 2, figsize=(10, 8)) 
# 通过 axes[行, 列] 访问具体的子图
axes[0, 0].plot(x, np.sin(x), 'r')
axes[0, 0].set_title("子图 1")
axes[0, 1].plot(x, np.cos(x), 'b')
axes[0, 1].set_title("子图 2")
axes[1, 0].plot(x, np.tan(x), 'g')
axes[1, 0].set_ylim(-5, 5) # 截断 tan 函数的极值
axes[1, 0].set_title("子图 3")
axes[1, 1].axis('off') # 隐藏没用的右下角
plt.tight_layout()
plt.show()
```
### 6.2 上下排列共用 X 轴
如果两个子图上下排列且 X 轴数据相同，为了简洁美观，通常让上面子图不显示 X 轴刻度。使用 `sharex=True` 可以实现**联动缩放**。
```python
# sharex=True: 共享X轴刻度和缩放比例
fig, (ax_top, ax_bottom) = plt.subplots(2, 1, figsize=(8, 6), sharex=True)
ax_top.plot(x, np.sin(x), 'r', label='正弦波')
ax_top.set_title("上方子图")
ax_top.legend()
ax_top.grid(True)
ax_bottom.plot(x, np.cos(x), 'b', label='余弦波')
ax_bottom.set_title("下方子图")
# 只需在最下方的子图设置 X 轴名称即可
ax_bottom.set_xlabel("X 轴数据") 
ax_bottom.legend()
ax_bottom.grid(True)
plt.tight_layout()
# 注意：使用 sharex 时，如果调用 tight_layout()，可能会出现底部标签被截断，
# 此时可以在 tight_layout() 中加入 rect 参数，或者不用 tight_layout，改用 constrained_layout=True
plt.show()
```
---
## 七、双 Y 轴共用 X 轴 (极其常用)
有时候左边的 Y 轴是温度，右边的 Y 轴是湿度，但它们共享同一个时间 X 轴。可以使用 `twinx()` 方法完美实现。
```python
fig, ax1 = plt.subplots(figsize=(8, 4))

# 第一条线，绑定到左侧 Y 轴
color = 'tab:red'
ax1.set_xlabel('时间')
ax1.set_ylabel('温度 (°C)', color=color)
ax1.plot(x, np.sin(x) * 10 + 20, color=color, label='温度')
ax1.tick_params(axis='y', labelcolor=color) # 让Y轴刻度颜色也变红

# 实例化第二个 Y 轴对象，共用同一个 X 轴
ax2 = ax1.twinx()  

# 第二条线，绑定到右侧 Y 轴
color = 'tab:blue'
ax2.set_ylabel('湿度 (%)', color=color)
ax2.plot(x, np.cos(x) * 10 + 50, color=color, linestyle='--', label='湿度')
ax2.tick_params(axis='y', labelcolor=color)

# 共用 X 轴的图例合并稍微复杂一点，需要手动收集 handles 和 labels
lines1, labels1 = ax1.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(lines1 + lines2, labels1 + labels2, loc='upper right')

plt.title("温度与湿度随时间变化 (共用X轴)")
plt.show()
```
---
## 八、辅助参考线 (`hline` 与 `vline`)
在数据分析中，我们经常需要在图表上画一条水平或垂直的线来表示**警戒线、目标值、平均值或特定的时间节点**。
### 8.1 贯穿全图的参考线 (`axhline` 和 `axvline`)
这两个函数的特点是：**无论你的数据范围是多少，它们都会横跨/纵跨整个坐标系**。
```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, y, label='信号数据')
# 绘制水平参考线 axhline (在Y=0处画灰色实线)
ax.axhline(y=0, color='gray', linestyle='-', linewidth=0.8, label='零位基线')
# 绘制水平警戒线 (在Y=3处画红色虚线)
ax.axhline(y=3, color='red', linestyle='--', linewidth=2, label='上限警戒线')
# 绘制垂直参考线 axvline (在X=5处画蓝色点划线)
ax.axvline(x=5, color='blue', linestyle='-.', linewidth=2, label='系统重启时刻')
# 补充：绘制跨越指定 Y 轴范围的区域色块
ax.axhspan(ymin=-2, ymax=2, facecolor='green', alpha=0.1, label='安全波动区间')
ax.set_title("使用 axhline / axvline 绘制辅助参考线", fontsize=14)
ax.legend(loc='upper right')
ax.grid(True, linestyle=':', alpha=0.6)
plt.show()
```
### 8.2 任意两点之间的连线
如果你不想让线贯穿全图，而是**从坐标轴上的 A 点精确画到 B 点**，直接使用 `plot` 传入两个端点坐标即可。
```python
fig, ax = plt.subplots(figsize=(6, 4))
ax.plot(x, y)
# 从 (5, -10) 画到 (5, 10) 的垂直线 (Y范围给大点，靠 ylim 裁剪)
ax.plot([5, 5], [-10, 10], color='purple', linestyle='--', label='局部垂直线')
# 从 (0, 3) 画到 (10, 3) 的水平线
ax.plot([0, 10], [3, 3], color='orange', linewidth=3, label='局部水平线')
ax.set_ylim(-6, 6) # 限制Y轴显示范围，超出部分会被自动裁剪
ax.legend()
plt.show()
```
---
## 九、区域填充 (`fill_between`)
当你想表达数据的上下限（如误差带、置信区间）或者想看两条曲线之间的面积差异时，区域填充是最好的选择。
```python
x = np.linspace(0, 10, 100)
y_mean = np.sin(x)
y_upper = y_mean + 0.2
y_lower = y_mean - 0.2
fig, ax = plt.subplots(figsize=(8, 4))
# 1. 绘制均值线
ax.plot(x, y_mean, 'r-', label='均值', linewidth=2)
# 2. 绘制误差带 (上下边界之间)
ax.fill_between(x, y1=y_upper, y2=y_lower, color='red', alpha=0.2, label='误差范围 (±0.2)')
# 3. 填充曲线下方到 Y=0 的所有区域
ax.fill_between(x, y1=y_mean, y2=0, color='blue', alpha=0.1, label='正面积区域')
ax.set_title("fill_between 区域填充应用")
ax.legend()
ax.grid(True)
plt.show()
```
---
## 十、文字标注与箭头 (`text` 与 `annotate`)
数据本身可能不够直观，我们需要在图表上直接写文字说明或指出异常点。
```python
fig, ax = plt.subplots()
ax.plot(x, np.sin(x))
# 方法1：直接在某坐标放下纯文本
ax.text(2.5, 0.8, "这是局部最高点", fontsize=12, color='blue')
# 方法2：带箭头的高阶注释
ax.annotate("波谷", 
            xy=(4.7, -1.0),             # 箭头指向的数据点坐标
            xytext=(2, -1.5),           # 文本所在的坐标
            arrowprops=dict(facecolor='black', shrink=0.05, width=1, headwidth=6), 
            fontsize=12, 
            # 给文本加个背景框，显得更专业
            bbox=dict(boxstyle="round,pad=0.3", fc="yellow", ec="black", alpha=0.8) 
           )
plt.show()
```
---
## 十一、数据轴的智能格式化 (百分比、千分位)
有时候 Y 轴的数据是 `0.03`，我们希望它显示为 `3%`；或者数据是 `1500000`，我们希望显示为 `1,500,000`。通过 `ticker` 模块可以轻松实现。
```python
import matplotlib.ticker as ticker
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))
x_index = range(10)
# --- 案例 1：Y轴格式化为百分比 ---
y_percent = np.random.rand(10) # 0~1 之间的随机数
ax1.plot(x_index, y_percent, 'o-')
ax1.set_title("Y轴格式化为百分比")
# 自定义格式化函数
def to_percent(y, position):
    return f"{y * 100:.0f}%"
ax1.yaxis.set_major_formatter(ticker.FuncFormatter(to_percent))
# --- 案例 2：Y轴加上千位分隔符 ---
y_big = np.random.randint(100000, 5000000, 10)
ax2.plot(x_index, y_big, 's-')
ax2.set_title("Y轴格式化为千分位数字")
# 使用匿名函数直接格式化
ax2.yaxis.set_major_formatter(ticker.FuncFormatter(lambda x, pos: f"{x:,.0f}"))
plt.tight_layout()
plt.show()
```
---
## 十二、保存高质量图片 (论文/报告必备)
写代码画图最终往往是为了写报告。**强烈建议不要直接截图，而是保存为矢量图。**
```python
fig, ax = plt.subplots()
ax.plot(x, y)
# dpi: 仅对位图 有效)
# bbox_inches='tight': 裁剪掉图片周围多余的空白边缘
# format: 'svg' (矢量图，放大不失真，极力推荐), 'pdf', 'png'
plt.savefig("my_professional_plot.svg", format="svg", dpi=300, bbox_inches='tight')
plt.show()
```
---
## 结语：构建专业图表的“黄金模板”
将前面学到的所有核心技能融合在一起，当你需要画一张专业图时，直接复制下面这个模板并修改数据即可。
```python
import matplotlib.pyplot as plt
import numpy as np
import matplotlib.ticker as ticker
# ================= 1. 全局配置 =================
plt.rcParams['font.sans-serif'] = ['SimHei'] 
plt.rcParams['axes.unicode_minus'] = False   
# ================= 2. 准备数据 =================
x = np.linspace(0, 10, 50)
y1 = np.sin(x) * 10 + 50 # 模拟温度数据
y2 = np.cos(x) * 5 + 60  # 模拟湿度数据
# ================= 3. 创建画板和子图 =================
fig, ax1 = plt.subplots(figsize=(10, 6))
# ================= 4. 绘制左侧 Y 轴数据 =================
color = 'tab:red'
ax1.set_xlabel('时间轴 (小时)', fontsize=12, fontweight='bold')
ax1.set_ylabel('温度 (°C)', color=color, fontsize=12)
line1 = ax1.plot(x, y1, color=color, linestyle='-', linewidth=2, marker='o', markersize=4, label='温度')[0]
ax1.tick_params(axis='y', labelcolor=color)
# ================= 5. 创建右侧 Y 轴数据 (共用X轴) =================
ax2 = ax1.twinx()  
color = 'tab:blue'
ax2.set_ylabel('湿度 (%)', color=color, fontsize=12)
line2 = ax2.plot(x, y2, color=color, linestyle='--', linewidth=2, label='湿度')[0]
ax2.tick_params(axis='y', labelcolor=color)
# ================= 6. 美化、辅助线与填充 =================
ax1.set_title("24小时温湿度协同监测分析", fontsize=16, pad=20)
ax1.grid(True, linestyle=':', alpha=0.6)
# 添加参考线与区间标注
ax1.axhline(y=60, color='red', linewidth=1.5, linestyle='--', alpha=0.5)
ax1.axhspan(ymin=40, ymax=60, facecolor='green', alpha=0.05)
ax1.text(0.5, 62, '高温警戒线', color='red', fontsize=10)
# ================= 7. 合并图例 =================
lines = [line1, line2]
labels = [l.get_label() for l in lines]
ax1.legend(lines, labels, loc='upper left', framealpha=0.9)
# ================= 8. 智能刻度格式化 =================
ax1.xaxis.set_major_locator(ticker.MaxNLocator(integer=True)) # X轴显示整数
ax2.yaxis.set_major_formatter(ticker.FuncFormatter(lambda y, _: f'{y:.0f}%')) # Y轴显示百分号
# ================= 9. 导出与展示 =================
plt.tight_layout()
plt.savefig("professional_chart.svg", dpi=300, bbox_inches='tight')
plt.show()
```
只要掌握了这套组合拳，足以应对 90% 以上的日常绘图与上位机数据显示需求！