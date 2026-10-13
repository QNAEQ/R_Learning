# ggplot2 知识点速查表

> **快速索引**：按 `Ctrl+F` 搜索关键词，或点击下方目录跳转

## 📋 目录
- [基础概念](#基础概念)
- [核心函数](#核心函数)
- [图形类型](#图形类型)
- [映射与设定](#映射与设定)
- [图层控制](#图层控制)
- [坐标与主题](#坐标与主题)
- [数据操作](#数据操作)
- [常见错误](#常见错误)
- [实用技巧](#实用技巧)
- [函数速查](#函数速查)

---

## 基础概念

### 图形两种用途
| 类型 | 用途 | 函数 | 特点 |
|------|------|------|------|
| **探查型** | 给自己看，快速了解数据 | `hist()`, `boxplot()`, `plot()` | 简单快速，不求美观 |
| **展示型** | 给别人看，正式报告/论文 | `ggplot2` 所有函数 | 精致清晰，专业美观 |

### ggplot2 三要素
```r
ggplot(data = 数据框) +      # 1. 数据
  aes(x = 列名, y = 列名) +   # 2. 映射
  geom_point()               # 3. 几何图层
```

### Grammar of Graphics（图形语法）
- **数据**：要可视化的数据源
- **映射**：数据到视觉属性的映射
- **几何对象**：点、线、柱等基本图形
- **统计变换**：数据聚合（如均值、计数）
- **坐标系**：坐标轴的类型和范围
- **分面**：将数据分割成子图
- **主题**：非数据元素的外观

---

## 核心函数

### ggplot()
```r
ggplot(data = mtcars)              # 基础写法
ggplot(mtcars, aes(x = wt, y = mpg))  # 常用紧凑写法
```

### aes() - 映射函数
```r
aes(x = wt, y = mpg)                    # 基础映射
aes(x = wt, y = mpg, color = cyl)       # 添加颜色映射
aes(x = wt, y = mpg, size = hp)        # 添加大小映射
aes(x = wt, y = mpg, shape = am)       # 添加形状映射
aes(x = wt, y = mpg, alpha = 0.5)       # 添加透明度映射
```

### labs() - 标签设置
```r
labs(title = "标题",
     subtitle = "副标题",
     caption = "注释",
     x = "X轴标签",
     y = "Y轴标签",
     color = "图例标题",
     shape = "图例标题",
     size = "图例标题")
```

---

## 图形类型

### 1. 散点图
```r
# 基础散点图
ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point()

# 带颜色映射
ggplot(mtcars, aes(x = wt, y = mpg, color = factor(cyl))) +
  geom_point(size = 3)

# 带大小映射（气泡图）
ggplot(mtcars, aes(x = wt, y = mpg, size = hp)) +
  geom_point(alpha = 0.6)
```

### 2. 柱状图
#### geom_col() - 需要预汇总
```r
# 先汇总
汇总 <- mtcars %>%
  group_by(cyl) %>%
  summarise(平均油耗 = mean(mpg))

# 再绘图
ggplot(汇总, aes(x = factor(cyl), y = 平均油耗)) +
  geom_col(fill = "steelblue")

# 管道操作
mtcars %>%
  group_by(cyl) %>%
  summarise(平均油耗 = mean(mpg)) %>%
  ggplot(aes(x = factor(cyl), y = 平均油耗)) +
  geom_col(fill = "steelblue")
```

#### geom_bar() - 自动计数
```r
# 自动计数
ggplot(mtcars, aes(x = factor(cyl))) +
  geom_bar(fill = "steelblue")
```

### 3. 折线图
```r
# 基础折线图
ggplot(月度[月度$城市 == "北京", ], aes(x = 月份, y = AQI)) +
  geom_line(color = "blue") +
  geom_point(color = "blue")  # 加点便于读数

# 多条折线（必须分组）
ggplot(月度, aes(x = 月份, y = AQI, color = 城市)) +
  geom_line() +
  scale_x_continuous(breaks = 1:12)  # 显示所有月份刻度
```

### 4. 直方图
```r
# base R 直方图
hist(mtcars$mpg)

# ggplot2 直方图（下一讲）
ggplot(mtcars, aes(x = mpg)) +
  geom_histogram(bins = 8, fill = "steelblue")
```

### 5. 箱线图
```r
# base R 箱线图
boxplot(mpg ~ cyl, data = mtcars)

# ggplot2 箱线图（下一讲）
ggplot(mtcars, aes(x = factor(cyl), y = mpg)) +
  geom_boxplot()
```

---

## 映射与设定

### 映射 vs 设定
| 类型 | 位置 | 作用 | 示例 |
|------|------|------|------|
| **映射** | aes()内 | 数据驱动视觉属性 | `color = factor(cyl)` |
| **设定** | aes()外 | 固定视觉属性 | `color = "red"` |

### 分类变量映射
```r
# 颜色映射
aes(color = factor(cyl))      # 转为因子，离散颜色
aes(color = cyl)              # 数值变量，渐变色带

# 形状映射（黑白打印重要）
aes(shape = factor(am))       # 自动变速箱类型

# 大小映射（气泡图）
aes(size = hp)                # 马力越大，点越大
```

### 连续变量映射
```r
# 大小映射
aes(size = hp)                # 马力

# 透明度映射
aes(alpha = density)          # 密度越大，越不透明
```

---

## 图层控制

### 图层叠加
```r
# 基础图层
ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point(color = "red", size = 2) +    # 第一层：散点
  geom_smooth(method = "lm", se = FALSE) + # 第二层：拟合线
  labs(title = "车重与油耗关系")             # 标题
```

### 常用几何图层
| 函数 | 用途 | 示例 |
|------|------|------|
| `geom_point()` | 散点 | `geom_point(color = "red", size = 2)` |
| `geom_line()` | 折线 | `geom_line(color = "blue")` |
| `geom_smooth()` | 平滑线 | `geom_smooth(method = "lm", se = FALSE)` |
| `geom_col()` | 柱状图 | `geom_col(fill = "steelblue")` |
| `geom_bar()` | 计数柱状图 | `geom_bar(fill = "lightblue")` |
| `geom_hline()` | 水平线 | `geom_hline(yintercept = 100, color = "red")` |
| `geom_text()` | 文本标签 | `geom_text(aes(label = 城市), vjust = -1)` |

---

## 坐标与主题

### 坐标轴控制
```r
# X轴刻度
scale_x_continuous(breaks = 1:12)  # 显示1-12
scale_x_discrete(labels = c("A", "B", "C"))  # 自定义标签

# Y轴刻度
scale_y_log10()  # 对数坐标
scale_y_continuous(limits = c(0, 50))  # 范围限制

# 坐标轴翻转
coord_flip()  # 横条图
```

### 主题控制
```r
# 基础主题
theme_minimal()      # 简约主题
theme_bw()           # 黑白主题
theme_classic()      # 经典主题

# 自定义主题
theme(plot.title = element_text(hjust = 0.5),  # 居中标题
      axis.text.x = element_text(angle = 45))   # X轴文字旋转
```

---

## 数据操作

### 排序
```r
# reorder函数（重要！）
reorder(列名, -数值)  # 降序
reorder(列名, 数值)   # 升序

# 使用示例
ggplot(aqi, aes(x = reorder(城市, -AQI均值), y = AQI均值)) +
  geom_col()
```

### 管道操作
```r
# 数据流操作
数据 %>%
  操作1 %>%
  操作2 %>%
  ggplot(...) +
  geom_...

# 示例
mtcars %>%
  group_by(cyl) %>%
  summarise(平均油耗 = mean(mpg)) %>%
  ggplot(aes(x = factor(cyl), y = 平均油耗)) +
  geom_col()
```

### 数据筛选
```r
# 单个城市
月度[月度$城市 == "北京", ]

# 多个城市
月度[月度$城市 %in% c("上海", "广州"), ]
```

---

## 常见错误

### 1. 管道与加号混淆
```r
# ❌ 错误
mtcars %>% ggplot(...) + geom_point()

# ✅ 正确
mtcars %>% 
  group_by(cyl) %>% 
  summarise(...) %>% 
  ggplot(...) + 
  geom_point()
```

### 2. aes()内外参数混淆
```r
# ❌ 映射写成设定（错误）
aes(color = "red")  # 这不是映射，是设定

# ✅ 正确写法
# 映射（数据驱动）
aes(color = factor(cyl))  
# 设定（固定值）
geom_point(color = "red")
```

### 3. factor()遗漏
```r
# ❌ 数值变量，渐变色
aes(color = cyl)  # 会产生渐变色带

# ✅ 转为因子，离散色
aes(color = factor(cyl))  # 产生离散颜色
```

### 4. 折线图忘记分组
```r
# ❌ 忘记分组，所有点连成一条线
ggplot(月度, aes(x = 月份, y = AQI)) + geom_line()

# ✅ 必须指定分组
ggplot(月度, aes(x = 月份, y = AQI, color = 城市)) + geom_line()
```

### 5. geom_col vs geom_bar
```r
# ❌ geom_col需要y值
ggplot(mtcars, aes(x = factor(cyl))) + geom_col()

# ✅ geom_col需要预汇总
汇总 <- mtcars %>% group_by(cyl) %>% summarise(均值 = mean(mpg))
ggplot(汇总, aes(x = factor(cyl), y = 均值)) + geom_col()

# ✅ geom_bar自动计数
ggplot(mtcars, aes(x = factor(cyl))) + geom_bar()
```

### 6. 机房版本兼容
```r
# 机房是ggplot2 3.3.x
geom_smooth(size = 1)     # ✅ 正确

# 不要使用（3.4+才有）
geom_smooth(linewidth = 1)  # ❌ 机房不支持
```

### 7. 加号位置错误
```r
# ❌ 加号在行首
ggplot(mtcars) 
+ aes(x = wt, y = mpg)

# ✅ 加号在行尾
ggplot(mtcars) + 
  aes(x = wt, y = mpg)
```

---

## 实用技巧

### 1. 添加参考线
```r
# 水平参考线
geom_hline(yintercept = 100, 
           color = "red", 
           linetype = "dashed")

# 垂直参考线
geom_vline(xintercept = 5, 
           color = "blue")
```

### 2. 数据标签
```r
# 基础标签
geom_text(aes(label = 城市), vjust = -1)

# 带条件标签
geom_text(aes(label = ifelse(AQI > 100, 城市, "")), 
          vjust = -1)
```

### 3. 分面
```r
# 单变量分面
facet_wrap(~区域)

# 双变量分面
facet_grid(区域 ~ 季节)
```

### 4. 颜色方案
```r
# 离散颜色
scale_color_brewer(palette = "Set1")

# 连续颜色
scale_color_gradient(low = "blue", high = "red")
```

### 5. 图例控制
```r
# 隐藏图例
geom_point(show.legend = FALSE)

# 自定义图例标题
labs(color = "气缸数", shape = "变速箱")
```

---

## 函数速查

### 核心函数
| 函数 | 功能 | 参数示例 |
|------|------|----------|
| `ggplot()` | 创建图形对象 | `ggplot(data = df, aes(x = x, y = y))` |
| `aes()` | 设置映射 | `aes(x = wt, y = mpg, color = cyl)` |
| `labs()` | 设置标签 | `labs(title = "标题", x = "X轴")` |
| `geom_point()` | 散点图 | `geom_point(color = "red", size = 3)` |
| `geom_col()` | 柱状图 | `geom_col(fill = "steelblue")` |
| `geom_bar()` | 计数柱状图 | `geom_bar(fill = "lightblue")` |
| `geom_line()` | 折线图 | `geom_line(color = "blue")` |
| `geom_smooth()` | 平滑线 | `geom_smooth(method = "lm", se = FALSE)` |
| `geom_hline()` | 水平线 | `geom_hline(yintercept = 100)` |
| `geom_text()` | 文本标签 | `geom_text(aes(label = "文本"))` |

### 数据操作函数
| 函数 | 功能 | 示例 |
|------|------|------|
| `group_by()` | 分组 | `group_by(cyl)` |
| `summarise()` | 汇总 | `summarise(均值 = mean(mpg))` |
| `reorder()` | 排序 | `reorder(城市, -AQI均值)` |
| `filter()` | 筛选 | `filter(城市 == "北京")` |
| `%>%` | 管道 | `mtcars %>% group_by(cyl)` |

### 坐标与主题
| 函数 | 功能 | 示例 |
|------|------|------|
| `coord_flip()` | 翻转坐标 | `coord_flip()` |
| `scale_x_continuous()` | X轴连续刻度 | `scale_x_continuous(breaks = 1:12)` |
| `scale_y_log10()` | Y轴对数刻度 | `scale_y_log10()` |
| `theme_minimal()` | 简约主题 | `theme_minimal()` |
| `theme_bw()` | 黑白主题 | `theme_bw()` |

---

## 快速备忘

### 一行代码示例
```r
# 散点图
ggplot(mtcars, aes(x = wt, y = mpg)) + geom_point()

# 分组散点图
ggplot(mtcars, aes(x = wt, y = mpg, color = factor(cyl))) + geom_point()

# 柱状图（已排序）
mtcars %>% group_by(cyl) %>% summarise(均值 = mean(mpg)) %>%
  ggplot(aes(x = reorder(factor(cyl), -均值), y = 均值)) + geom_col()

# 折线图（多条）
ggplot(月度, aes(x = 月份, y = AQI, color = 城市)) + geom_line()

# 带参考线的柱状图
ggplot(aqi, aes(x = reorder(城市, -AQI均值), y = AQI均值)) +
  geom_col(fill = "steelblue") +
  geom_hline(yintercept = 100, color = "red", linetype = "dashed")
```

### 常用颜色
```r
# 基础颜色
"red", "blue", "green", "yellow", "purple", "orange", "pink", "gray"

# 专业配色
"steelblue", "lightblue", "darkblue", "forestgreen", "tomato", "lightcoral"
```

### 常用形状
```r
# 0-25：各种形状
shape = 0:25  # 查看所有形状
```

---

## 更新记录
- v1.0 (2024): 初始版本，包含第5讲核心内容
- 持续更新中...

> **提示**：保存此文档到R工作目录，随时查阅！