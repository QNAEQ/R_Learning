# dplyr六大动词知识点总结

## 📋 目录索引

1. [核心概念](#1-核心概念)
2. [六大动词详解](#2-六大动词详解)
   - [filter() - 筛选行](#21-filter---筛选行)
   - [arrange() - 排序](#22-arrange---排序)
   - [select() - 选择列](#23-select---选择列)
   - [mutate() - 创建新列](#24-mutate---创建新列)
   - [group_by() + summarise() - 分组汇总](#25-group_by--summarise---分组汇总)
   - [其他重要函数](#26-其他重要函数)
3. [管道操作符](#3-管道操作符)
4. [常用函数速查表](#4-常用函数速查表)
5. [易错点总结](#5-易错点总结)
6. [实际应用技巧](#6-实际应用技巧)
7. [与base R的对比](#7-与base-r的对比)
8. [练习题答案参考](#8-练习题答案参考)

---

## 1. 核心概念

### dplyr是什么？
- dplyr是R语言中用于数据处理的强大包
- 提供了一套一致、简洁的语法来操作数据框
- 六大动词构成了数据处理的核心框架

### 管道操作符 `%>%`
- 将左边的结果作为右边函数的第一个参数
- 实现函数的链式调用
- 代码从左到右读，符合阅读习惯
- **快捷键**：Ctrl + Shift + M (Windows) / Cmd + Shift + M (Mac)

---

## 2. 六大动词详解

### 2.1 filter() - 筛选行

**功能**：根据条件筛选符合条件的行

**语法**：
```r
filter(数据框, 条件1, 条件2, ...)
```

**示例**：
```r
# 筛选4缸车
filter(mtcars, cyl == 4)

# 筛选4缸且马力大于90的车
filter(mtcars, cyl == 4, hp > 90)

# 筛选4缸或油耗大于30的车
filter(mtcars, cyl == 4 | mpg > 30)

# 使用&符号
filter(mtcars, cyl == 4 & hp > 90)

# 使用%in%判断是否在集合中
filter(mtcars, cyl %in% c(4, 6))
```

**特点**：
- 多个条件用逗号分隔（表示"且"）
- 使用"&"也可以表示"且"
- 使用"|"表示"或"
- 支持%in%操作符判断是否在集合中
- 列名直接写，不需要$符号
- NA值会被自动排除

### 2.2 arrange() - 排序

**功能**：对数据框进行排序

**语法**：
```r
arrange(数据框, 排序列1, 排序列2, ...)
```

**示例**：
```r
# 按mpg升序排列
arrange(mtcars, mpg)

# 按mpg降序排列（使用负号）
arrange(mtcars, -mpg)

# 使用desc()函数降序
arrange(mtcars, desc(mpg))

# 多条件排序：先按cyl升序，cyl相同时按mpg降序
arrange(mtcars, cyl, -mpg)
```

**特点**：
- 默认升序排列
- 使用负号表示降序
- 使用desc()函数降序（推荐）
- 多条件排序时，前面的列优先级更高

### 2.3 select() - 选择列

**功能**：选择特定的列

**语法**：
```r
select(数据框, 列名1, 列名2, ...)
```

**示例**：
```r
# 选择特定列
select(iris, Species, Sepal.Length)

# 使用冒号选择连续列
select(iris, Sepal.Length:Petal.Length)

# 使用减号剔除列
select(iris, -Species)

# 剔除多列
select(iris, -c(Sepal.Length, Sepal.Width))

# 辅助函数：
# starts_with("前缀")：选择以指定前缀开头的列
select(iris, starts_with("Sepal"))

# ends_with("后缀")：选择以指定后缀结尾的列
select(iris, ends_with("Width"))

# contains("字符串")：选择包含指定字符串的列
select(iris, contains("Petal"))

# 重命名列
select(iris, 种类 = Species, 花萼长 = Sepal.Length)
```

**特点**：
- 使用冒号选择连续列
- 使用减号剔除列
- 提供多个辅助函数简化列选择
- 可以同时进行重命名

### 2.4 mutate() - 创建新列

**功能**：在数据框中创建新列

**语法**：
```r
mutate(数据框, 新列名 = 计算式, ...)
```

**示例**：
```r
# 创建单列
mutate(mtcars, 单位重量马力 = hp / wt)

# 创建多列
mutate(mtcars, 
       单位重量马力 = hp / wt,
       马力等级 = 单位重量马力 / 10)

# 结合ifelse进行二值转换
mutate(mtcars,
       变速箱类型 = ifelse(am == 0, "自动挡", "手动挡"))

# 复杂条件判断
mutate(mtcars,
       车辆类型 = ifelse(cyl %in% c(6, 8), "费油车", "省油车"))
```

**特点**：
- 可以一次创建多列
- 新列可以在后续列中使用
- 结合ifelse进行条件判断
- 不会修改原始数据框，返回新结果

### 2.5 group_by() + summarise() - 分组汇总

**功能**：按指定列分组，然后对每组进行汇总计算

**语法**：
### 4 个可选参数（考试常考）

1. `.groups = "drop"`：**删除分组（最常用）**，汇总结果变成普通 data.frame，没有分组信息。
2. `.groups = "keep"`：保持原来的分组变量（依然是分组 tbl）
3. `.groups = "rowwise"`：转为按行分组
4. `.groups = "drop_last"`：只去掉最后一层分组（多分组时用）
```r
数据框 %>% 
  group_by(分组列1, 分组列2, ...) %>% 
  summarise(汇总列1 = 计算函数(列), 
           汇总列2 = 计算函数(列),
           .groups = "drop")
```

**示例**：
```r
# 按气缸数分组，计算平均油耗和台数
mtcars %>% 
  group_by(cyl) %>% 
  summarise(平均油耗 = mean(mpg),
           车辆台数 = n(),
           .groups = "drop")

# 多个统计函数
mtcars %>% 
  group_by(cyl) %>% 
  summarise(台数 = n(),
           平均油耗 = mean(mpg),
           油耗中位 = median(mpg),
           油耗标准差 = sd(mpg),
           最大马力 = max(hp),
           总重量 = sum(wt),
           .groups = "drop")

# 处理NA值
df %>% 
  group_by(性别) %>% 
  summarise(语文均分 = round(mean(语文, na.rm = TRUE), 1),
           实考人数 = sum(!is.na(语文)),
           .groups = "drop")
```

**常用统计函数**：
- `n()`：计算行数
- `mean()`：计算平均值
- `median()`：计算中位数
- `sd()`：计算标准差
- `sum()`：求和
- `max()`：最大值
- `min()`：最小值
- `first()`：第一个值
- `last()`：最后一个值
- `nth()`：第n个值

### 2.6 其他重要函数

#### rename() - 改列名
```r
# 语法：rename(新名 = 旧名)
iris %>% rename(花萼长度 = Sepal.Length, 花萼宽度 = Sepal.Width)
```

#### relocate() - 调整列顺序
```r
# 移到最前
iris %>% relocate(Species)

# 移到指定列之后
iris %>% relocate(Species, .after = Sepal.Width)
```

#### distinct() - 去重
```r
# 删除重复行
distinct(a)

# 查看唯一组合
cb %>% distinct(商品类别)

# 查看多个列的唯一组合
cb %>% distinct(入境口岸, 商品类别)
```

#### na.omit() - 删除含NA的行
```r
# 删除包含NA的整行
df_clean <- na.omit(df)
```

#### count() - 计数
```r
# 等价于 group_by + summarise(n())
cb %>% count(商品类别)

# 计数并排序
cb %>% count(入境口岸, sort = TRUE)
```

---

## 3. 管道操作符

### 基本用法
```r
# 普通写法
mean(mtcars$mpg)

# 管道写法
mtcars$mpg %>% mean()

# 多步操作
mtcars %>% 
  filter(cyl == 4) %>% 
  select(mpg, hp) %>% 
  mutate(mph = mpg / hp) %>% 
  arrange(-mph)
```

### 管道操作的优势
- 减少中间变量
- 代码从左到右读，符合阅读习惯
- 提高代码可读性和可维护性
- 方便调试（可以随时在管道中间插入head()查看结果）

---

## 4. 常用函数速查表

### 筛选和排序
| 函数 | 功能 | 示例 |
|------|------|------|
| filter() | 筛选行 | filter(df, x > 10) |
| arrange() | 排序 | arrange(df, x) |
| desc() | 降序 | arrange(df, desc(x)) |

### 列操作
| 函数 | 功能 | 示例 |
|------|------|------|
| select() | 选择列 | select(df, x, y) |
| rename() | 改列名 | rename(df, 新名 = 旧名) |
| relocate() | 调整列顺序 | relocate(df, x) |
| distinct() | 去重 | distinct(df) |

### 数据转换
| 函数 | 功能 | 示例 |
|------|------|------|
| mutate() | 创建新列 | mutate(df, z = x + y) |
| transmute() | 创建新列并保留新列 | transmute(df, z = x + y) |
| ifelse() | 条件判断 | ifelse(x > 0, "正", "负") |

### 分组汇总
| 函数 | 功能 | 示例 |
|------|------|------|
| group_by() | 分组 | group_by(df, category) |
| summarise() | 汇总 | summarise(df, mean = mean(x)) |
| count() | 计数 | count(df, category) |

### 缺失值处理
| 函数 | 功能 | 示例 |
|------|------|------|
| na.omit() | 删除含NA的行 | na.omit(df) |
| is.na() | 判断是否为NA | is.na(df$x) |
| na.rm | 在函数中移除NA | mean(df$x, na.rm = TRUE) |

---

## 5. 易错点总结

### 5.1 常见错误

#### 1. 列名拼写错误
```r
# 错误
filter(mtcars, CYL == 4)  # 大小写错误

# 正确
filter(mtcars, cyl == 4)
```

#### 2. 缺少逗号
```r
# 错误
filter(mtcars cyl == 4 hp > 90)

# 正确
filter(mtcars, cyl == 4, hp > 90)
```

#### 3. 管道符使用错误
```r
# 错误
mtcars %>% filter cyl == 4

# 正确
mtcars %>% filter(cyl == 4)
```

#### 4. 括号不匹配
```r
# 错误
mutate(mtcars, 
       单位重量马力 = hp / wt
       马力等级 = 单位重量马力 / 10)

# 正确
mutate(mtcars, 
       单位重量马力 = hp / wt,
       马力等级 = 单位重量马力 / 10)
```

#### 5. na.rm参数缺失
```r
# 错误（如果数据中有NA）
df %>% group_by(性别) %>% summarise(语文均分 = mean(语文))

# 正确
df %>% group_by(性别) %>% summarise(语文均分 = mean(语文, na.rm = TRUE))
```

### 5.2 重要注意事项

#### 1. rename的方向
```r
# 错误（方向反了）
rename(iris, Sepal.Length = 花萼长度)

# 正确
rename(iris, 花萼长度 = Sepal.Length)
```

#### 2. group_by后的.groups参数
```r
# 错误（忘记解除分组）
mtcars %>% group_by(cyl) %>% summarise(平均油耗 = mean(mpg))

# 正确
mtcars %>% group_by(cyl) %>% summarise(平均油耗 = mean(mpg), .groups = "drop")
```

#### 3. 管道操作的顺序
```r
# 效率较低（先选择再筛选）
mtcars %>% select(mpg, cyl, hp) %>% filter(cyl == 4)

# 效率较高（先筛选再选择）
mtcars %>% filter(cyl == 4) %>% select(mpg, hp)
```

#### 4. ifelse的语法
```r
# 错误
ifelse(mpg > 20, "高油耗")

# 正确
ifelse(mpg > 20, "高油耗", "低油耗")
```

#### 5. %in%的使用
```r
# 错误
filter(mtcars, cyl == 4 or cyl == 6)

# 正确
filter(mtcars, cyl %in% c(4, 6))
```

#### 6. desc()函数的使用
```r
# 错误
arrange(mtcars, -mpg)

# 可以，但推荐使用desc()
arrange(mtcars, desc(mpg))
```

---

## 6. 实际应用技巧

### 6.1 数据处理最佳实践

#### 1. 筛选优化
```r
# 先筛选再选择，提高效率
mtcars %>% 
  filter(cyl == 4 & hp > 90) %>%  # 先筛选
  select(mpg, hp, wt)            # 再选择
```

#### 2. 使用辅助函数
```r
# 使用starts_with简化列选择
iris %>% select(starts_with("Sepal"))

# 使用ends_with选择特定后缀的列
iris %>% select(ends_with("Width"))

# 使用contains包含特定字符串
iris %>% select(contains("Petal"))
```

#### 3. 条件判断技巧
```r
# 复杂条件判断
cb %>% 
  mutate(优先级 = ifelse(
    商品类别 %in% c("数码电子", "美妆护肤") & 通关时间 >= 18,
    "高",
    "低"
  ))
```

#### 4. 分组汇总技巧
```r
# 同时计算多个统计量
mtcars %>% 
  group_by(cyl) %>% 
  summarise(
    数量 = n(),
    均值 = mean(mpg, na.rm = TRUE),
    中位数 = median(mpg, na.rm = TRUE),
    标准差 = sd(mpg, na.rm = TRUE),
    最大值 = max(mpg, na.rm = TRUE),
    最小值 = min(mpg, na.rm = TRUE),
    .groups = "drop"
  )
```

### 6.2 调试技巧

#### 1. 分步查看
```r
# 在管道中间插入head()查看结果
mtcars %>% 
  filter(cyl == 4) %>% head() %>%  # 查看筛选结果
  select(mpg, hp) %>% head() %>%   # 查看选择结果
  mutate(mph = mpg / hp)           # 继续处理
```

#### 2. 使用变量存储中间结果
```r
# 将复杂操作拆解
filtered <- mtcars %>% filter(cyl == 4)
selected <- filtered %>% select(mpg, hp)
result <- selected %>% mutate(mph = mpg / hp)
```

---

## 7. 与base R的对比

| 任务 | base R | dplyr | 优势 |
|------|--------|-------|------|
| 筛选行 | df[df$x > 3, ] | filter(df, x > 3) | dplyr更易读 |
| 选择列 | df[, c("a", "b")] | select(df, a, b) | dplyr语法更简洁 |
| 排序 | df[order(df$x), ] | arrange(df, x) | dplyr支持降序更直观 |
| 降序 | df[order(-df$x), ] | arrange(df, -x) | dplyr的desc()更清晰 |
| 新增列 | df$新列 <- df$a / df$b | mutate(df, 新列 = a / b) | dplyr支持链式操作 |
| 改列名 | names(df)[2] <- "新名" | rename(df, 新名 = 旧名) | dplyr不易出错 |
| 去重 | df[!duplicated(df), ] | distinct(df) | dplyr更直观 |
| 分组汇总 | tapply()之后手工拼表 | group_by() %>% summarise() | dplyr一步到位 |

### 重要说明
- base R不是"过时的错的"，dplyr也不是"唯一正确的"
- base R更底层，写函数、做包时离不开它
- dplyr更适合日常做分析，代码读起来像说话
- 两套都会，才叫真正掌握

---

## 8. 练习题答案参考

### 练习1-1
```r
filter(mtcars, cyl == 4)                    # 4缸车
filter(mtcars, cyl == 4, hp > 90)           # ★ 逗号就是「且」，比 & 好打
filter(mtcars, cyl == 4 & hp > 90)          # 写 & 也行，一样的结果
filter(mtcars, cyl == 4 | mpg > 30)         # 「或」还是要写 |
mtcars %>% filter(cyl == 4, hp > 90)        # 管道版
mtcars %>% filter(cyl %in% c(4, 6))         # %in% 一样能用
```

### 练习2-3
```r
iris %>% select(Species, Sepal.Length, Petal.Length) %>% arrange(-Sepal.Length)
```

### 练习3-4
```r
mtcars %>% mutate(单位重量马力 = hp / wt,
                  是否高性能 = ifelse(单位重量马力 > 50, "是", "否"))
```

### 练习4-8
```r
# mtcars按气缸数计算
mtcars %>% group_by(cyl) %>% summarise(平均油耗 = mean(mpg), 台数 = n(), .groups = "drop")

# students按性别计算三科平均分
df %>% group_by(性别) %>%
  summarise(语文 = mean(语文, na.rm = TRUE),
            数学 = mean(数学, na.rm = TRUE),
            英语 = mean(英语, na.rm = TRUE), 
            .groups = "drop")
```

### 综合竞速题（简化版）
```r
竞赛包裹 %>%
  filter(商品类别 %in% c("数码电子", "美妆护肤"),
         !is.na(申报价值), !is.na(通关时间), 通关时间 >= 18) %>%
  mutate(优先积分 = ifelse(申报价值 >= 3000, 3, 1)) %>%
  group_by(入境口岸) %>%
  summarise(待处置数 = n(),
            总积分 = sum(优先积分),
            .groups = "drop") %>%
  filter(待处置数 >= 30) %>%
  arrange(desc(总积分), desc(待处置数)) %>%
  head(1)
```

---

## 总结

dplyr六大动词是数据处理的利器，掌握它们可以让你的数据分析工作事半功倍：

1. **filter** - 筛选你想要的数据
2. **arrange** - 排序让数据更有序
3. **select** - 选择需要的列
4. **mutate** - 创建新的变量
5. **group_by + summarise** - 分组汇总的核心
6. **管道符 %>%** - 连接所有操作的魔法

记住常见的易错点，多加练习，你就能熟练使用dplyr进行高效的数据分析！