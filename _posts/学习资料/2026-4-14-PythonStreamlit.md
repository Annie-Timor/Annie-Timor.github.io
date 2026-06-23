---
title: Streamlit 绝对实战指南
author: lixinghui
date: 2026-4-14 12:00:00 +0800
categories: [Note, Python]
tags: [学习资料]
---

# 🚀 Streamlit 绝对实战指南：从脚本到专业级 App

你是否有过这样的经历：用 Python 写了一个非常棒的数据分析或机器学习模型，但只能自己对着 Jupyter Notebook 欣赏？如果你想把它变成一个**任何人都能通过浏览器使用的漂亮网页应用**，而且**不需要写任何 HTML/CSS/JS**，那么 Streamlit 就是你的终极武器。
## 🧠 第一部分：颠覆认知的“执行模型”
在写代码之前，你必须搞懂 Streamlit 是怎么运行的，否则会遇到无数玄学 Bug。
### 1. 自顶向下的执行流
传统的 Web 框架（如 Flask/Django）是“等待触发->执行对应函数”。而 Streamlit 是**从头到尾顺序执行**整个 Python 脚本。每次你点击按钮、滑动滑块，整个脚本就会**从头到尾重新运行一遍**。

### 2. 玩转 `st.session_state`（核心中的核心）

因为脚本会无限重跑，如果我想保留一个变量（比如用户登录状态、累加的计数器），怎么办？这就需要 `session_state`（会话状态）。它是一个跨重跑存在的字典。
**❌ 错误写法（永远停留在 1）：**
```python
count = 0
if st.button("点我加1"):
    count += 1
st.write(f"当前数值: {count}") 
```
**✅ 正确写法：**
```python
# 初始化（只在第一次运行时生效）
if 'count' not in st.session_state:
    st.session_state.count = 0
if st.button("点我加1"):
    st.session_state.count += 1
st.write(f"当前数值: {st.session_state.count}")
```
**💡 高级技巧：用 Callback 绑定按钮**
当你在输入框按回车，或者点击按钮时，不想触发整个脚本的重跑，而是先执行一段特定逻辑：
```python
def form_submit():
    st.session_state.submitted = True
st.text_input("输入内容", on_change=form_submit)
```
---
## 🎨 第二部分：布局与排版（让 App 像个真正的产品）
不要把所有东西都堆叠在一起，掌握以下布局组件，你的界面立刻专业起来。
### 1. 侧边栏 `st.sidebar`
任何放在 `with st.sidebar:` 下面的组件，都会自动跑到页面左边。
```python
with st.sidebar:
    st.file_uploader("上传数据")
    st.selectbox("选择模型", ["A", "B"])
```
### 2. 分栏布局 `st.columns`
这是最常用的布局！将页面横向切分。
```python
col1, col2, col3 = st.columns(3) # 分成 3 等份
# 也可以设置比例: col1, col2 = st.columns([2, 1]) col1占2/3宽
with col1:
    st.metric("总收入", "10万", "+1.2万")
with col2:
    st.metric("总支出", "8万", "-0.5万")
with col3:
    st.metric("利润", "2万", "+0.7万")
```
### 3. 标签页 `st.tabs`
当内容太多，或者属于不同类别时，千万不要堆砌，用 Tabs：
```python
tab1, tab2, tab3 = st.tabs(["📊 数据概览", "📈 可视化分析", "⚙️ 参数设置"])
with tab1:
    st.write("这里是数据概览")
with tab2:
    st.line_chart(data_df)
with tab3:
    st.slider("调整阈值", 0, 100)
```

### 4. 容器与展开框 `st.container` & `st.expander`
*   `st.container()`：把一些元素打包，方便整体控制（原代码中用到过）。
*   `st.expander("标题")`：默认收起，点击展开。**非常适合放原始数据表格、冗长的日志、代码说明**，保持主界面清爽。

---

## 🛠️ 第三部分：交互组件全家桶
交互组件的作用是：**接收用户输入 -> 赋值给变量 -> 触发脚本重跑 -> 影响下方代码**。

| 需求场景           | 使用的组件                                                | 返回值类型           |
| :----------------- | :-------------------------------------------------------- | :------------------- |
| 选择单一选项       | `st.selectbox("选啥", ['A', 'B'])`                        | `str`                |
| 多选标签           | `st.multiselect("选啥", ['A', 'B', 'C'])`                 | `list`               |
| 滑动数字           | `st.slider("年龄", 0, 100, value=25)`                     | `int/float`          |
| 输入文本           | `st.text_input("名字")`                                   | `str`                |
| 输入数字           | `st.number_input("金额", step=0.01)`                      | `int/float`          |
| 日期选择           | `st.date_input("生日")`                                   | `datetime.date`      |
| 开关/复选框        | `st.checkbox("同意协议")`                                 | `bool`               |
| 上传文件           | `st.file_uploader("传文件", type=['csv'])`                | `BytesIO` 对象       |
| 下载文件           | `st.download_button("下载", data=csv, file_name='f.csv')` | 触发下载动作         |
| **【隐藏大杀器】** | **`st.number_input(step=0, format="%d")`**                | **伪装成只读展示框** |

---
## 📊 第四部分：数据与图表可视化
Streamlit 对 Pandas 和主流图表库做了深度集成。
### 1. 数据展示
```python
# 静态表格（适合少量数据展示）
st.table(df.head())
# 交互式表格（自带排序、搜索、全屏，极力推荐！）
st.dataframe(df, use_container_width=True, hide_index=True)
# 可编辑表格（Streamlit 1.30+ 新特性，超级强大！）
edited_df = st.data_editor(df, num_rows="dynamic")
# 用户修改后，edited_df 会返回修改后的结果
```
### 2. 原生图表（极速出图）
不需要写复杂的配置，直接把 DataFrame 传进去：
```python
st.line_chart(df[['日期', '销售额']]) # 折线图
st.bar_chart(df[['城市', '人口']])    # 柱状图
st.area_chart(df)                     # 面积图
```
### 3. 复杂图表
当你需要双 Y 轴、子图、复杂的 Tooltip（像你之前的耳温枪代码），必须用 Plotly：
```python
import plotly.express as px
import plotly.graph_objects as go
fig = px.scatter(df, x="长度", y="重量", color="种类")
# 注意：在 Streamlit 中，统一使用 use_container_width=True 让图表自适应宽度
st.plotly_chart(fig, use_container_width=True) 
```
---
## ⚡ 第五部分：性能优化（决定你的 App 是能用还是卡死）
这是区分新手和高手的最重要分水岭。如果你的数据有 10 万行，每次滑一下滑块都要等 3 秒，体验会极差。
### 1. 缓存装饰器（魔法指令）
*   **`@st.cache_data`**：用于缓存**数据**（DataFrame、字符串、字典）。比如读取 CSV、爬虫请求。
*   **`@st.cache_resource`**：用于缓存**资源**（数据库连接池、机器学习模型）。只要加载到内存中，不管重跑多少次都不变。
```python
@st.cache_data  # 加上这行，无论怎么点按钮，只有第一次会去读文件，后续直接从内存拿
def load_data(path):
    df = pd.read_csv(path) # 假设这步很慢
    return df
data = load_data("large_file.csv")
```
**如果我想强制刷新缓存怎么办？**
在侧边栏加一个按钮：
```python
if st.button("清除缓存"):
    st.cache_data.clear()
```
### 2. 避免在循环中渲染组件
❌ **大忌**：不要在 `for` 循环里面写 `st.write()` 或 `st.button()`。组件太多，重跑会严重卡顿。
✅ **替代方案**：把循环里的数据聚合成一个 DataFrame，最后用一次 `st.dataframe()` 或 `st.metric()` 展示。（就像你之前展示 8 个变化率，用 `st.columns(8)` 是没问题的，因为是固定数量，但如果动态生成 1000 个 column 就会崩）。

---

## 🗂️ 第六部分：多页面应用架构
当一个 App 超过 200 行代码，千万别全塞在一个 `app.py` 里！
1. 在项目根目录创建一个文件夹，必须命名为 `pages`。
2. 在 `pages` 文件夹里创建你的子页面，比如 `1_数据清洗.py`, `2_模型训练.py`, `3_系统设置.py`。
3. Streamlit 会自动识别，并在左侧边栏生成多页面导航菜单！
**目录结构：**
```text
my_app/
├── app.py               (主页：首页概览)
└── pages/
    ├── 1_数据清洗.py
    ├── 2_模型训练.py
    └── 3_系统设置.py
```
**跨页面传参：** 直接使用 `st.session_state`，它在所有页面中都是共享的！

---
## 🎯 第七部分：高频实战场景代码模板
### 场景 1：带进度条的耗时任务
```python
import time
st.write("开始处理大量数据...")
progress_bar = st.progress(0)
status_text = st.empty()
for i in range(100):
    time.sleep(0.05) # 模拟耗时操作
    progress_bar.progress(i + 1)
    status_text.text(f"当前进度: {i+1}%")
status_text.text("处理完成！✅")
```
### 场景 2：文件上传与动态预览
```python
uploaded = st.file_uploader("上传CSV", type="csv")
if uploaded is not None:
    df = pd.read_csv(uploaded)
    st.success("文件读取成功！")
    
    tab1, tab2 = st.tabs(["原始数据", "数据描述"])
    with tab1:
        st.dataframe(df)
    with tab2:
        st.write(df.describe())
```
### 场景 3：优雅的表单提交
默认情况下，输入框只要改动一个字就会触发重跑。如果表单有 10 个输入框，体验极差。用 `st.form` 可以实现“填完所有内容后，点击提交按钮才重跑”。
```python
with st.form("my_form"):
    st.write("请在下方填写参数")
    name = st.text_input("姓名")
    age = st.number_input("年龄", 0, 120)
    submitted = st.form_submit_button("提交")
    
    if submitted:
        st.success(f"保存成功：{name}, {age}岁")
```
---
## 🚢 第八部分：部署上线给别人看
写完了代码，怎么让老板/客户看到？
**最省事的方法：Streamlit Community Cloud (免费)**
1. 把代码传到 GitHub。
2. 登录 [share.streamlit.io](https://share.streamlit.io)。
3. 授权 GitHub，选择你的仓库，点击部署。
4. 2 分钟后，你会获得一个类似 `https://your-app.streamlit.app` 的公网链接。
**如果是企业内网（无外网环境）：**
```bash
# 在服务器上直接运行，指定端口
streamlit run app.py --server.port 8080 --server.address 0.0.0.0
```
---
## 🎓 结语与进阶建议
这篇教程覆盖了 Streamlit 90% 的日常开发场景。现在，你可以尝试：
1. 把之前写过的某个分析脚本，加上 `st.slider` 和 `st.line_chart` 改造成 App。
2. 遇到不知道怎么实现的 UI，直接去 [Streamlit Gallery](https://streamlit.io/gallery) 找类似的项目，看别人的源码是怎么组织的。
**避坑口诀总结：**

> 变量重置找 `session_state`，布局排版靠 `columns/tabs`，机器学习模型一定加 `cache_resource`，图表自适应记死 `use_container_width=True`。