---
name: obsidian-bases
description: 创建和编辑 Obsidian Bases（.base 文件），支持视图、筛选器、公式和汇总。在处理 .base 文件、创建笔记的数据库视图，或用户提到 Bases、表格视图、卡片视图、筛选器、公式时使用。
---

# Obsidian Bases

## 工作流程

1. **创建文件**：在库中创建包含合法 YAML 内容的 `.base` 文件
2. **定义范围**：添加 `filters` 筛选哪些笔记出现在视图中（按标签、文件夹、属性或日期）
3. **添加公式**（可选）：在 `formulas` 区域定义计算属性
4. **配置视图**：添加一个或多个视图（`table`、`cards`、`list` 或 `map`），用 `order` 指定显示的属性顺序
5. **验证**：确认文件是合法 YAML 且无语法错误；检查所有引用的属性和公式均已定义。常见问题：含特殊 YAML 字符的字符串未加引号、公式表达式中的引号不匹配、`order` 中引用了 `formula.X` 却未在 `formulas` 中定义
6. **在 Obsidian 中测试**：在 Obsidian 中打开 `.base` 文件确认视图渲染正常。若提示 YAML 错误，请检查下方的引号规则

## 结构

Base 文件使用 `.base` 扩展名，内容须为合法 YAML。

```yaml
# 全局筛选器作用于 Base 中的所有视图
filters:
  # 可以是单条筛选字符串
  # 也可以是带 and/or/not 的嵌套筛选对象
  and: []
  or: []
  not: []

# 定义可在所有视图中使用的公式属性
formulas:
  formula_name: 'expression'

# 配置属性的显示名称和设置
properties:
  property_name:
    displayName: "Display Name"
  formula.formula_name:
    displayName: "Formula Display Name"
  file.ext:
    displayName: "Extension"

# 定义自定义汇总公式
summaries:
  custom_summary_name: 'values.mean().round(3)'

# 定义一个或多个视图
views:
  - type: table | cards | list | map
    name: "View Name"
    limit: 10                    # 可选：限制结果数量
    groupBy:                     # 可选：分组显示
      property: property_name
      direction: ASC | DESC
    filters:                     # 视图级筛选器
      and: []
    order:                       # 按顺序显示的属性
      - file.name
      - property_name
      - formula.formula_name
    summaries:                   # 将属性映射到汇总公式
      property_name: Average
```

## 筛选语法

筛选器用于缩小结果范围，可全局应用或按视图设置。

### 筛选结构

```yaml
# 单条筛选
filters: 'status == "done"'

# AND - 所有条件必须同时满足
filters:
  and:
    - 'status == "done"'
    - 'priority > 3'

# OR - 任一条件满足即可
filters:
  or:
    - 'file.hasTag("book")'
    - 'file.hasTag("article")'

# NOT - 排除匹配项
filters:
  not:
    - 'file.hasTag("archived")'

# 嵌套筛选
filters:
  or:
    - file.hasTag("tag")
    - and:
        - file.hasTag("book")
        - file.hasLink("Textbook")
    - not:
        - file.hasTag("book")
        - file.inFolder("Required Reading")
```

### 筛选运算符

| 运算符 | 含义 |
|----------|-------------|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` | 大于 |
| `<` | 小于 |
| `>=` | 大于等于 |
| `<=` | 小于等于 |
| `&&` | 逻辑与 |
| `\|\|` | 逻辑或 |
| <code>!</code> | 逻辑非 |

## 属性

### 三种属性类型

1. **笔记属性** — 来自 frontmatter：`note.author` 或简写为 `author`
2. **文件属性** — 文件元数据：`file.name`、`file.mtime` 等
3. **公式属性** — 计算值：`formula.my_formula`

### 文件属性速查

| 属性 | 类型 | 说明 |
|----------|------|-------------|
| `file.name` | String | 文件名 |
| `file.basename` | String | 不含扩展名的文件名 |
| `file.path` | String | 完整路径 |
| `file.folder` | String | 所在文件夹路径 |
| `file.ext` | String | 文件扩展名 |
| `file.size` | Number | 文件大小（字节） |
| `file.ctime` | Date | 创建时间 |
| `file.mtime` | Date | 修改时间 |
| `file.tags` | List | 文件中的所有标签 |
| `file.links` | List | 文件中的内部链接 |
| `file.backlinks` | List | 链接到该文件的笔记 |
| `file.embeds` | List | 笔记中的嵌入 |
| `file.properties` | Object | 所有 frontmatter 属性 |

### `this` 关键字

- 在主内容区：指代 Base 文件本身
- 被嵌入时：指代嵌入它的文件
- 在侧边栏：指代主内容区的当前活动文件

## 公式语法

公式用于从属性计算值，在 `formulas` 区域定义。

```yaml
formulas:
  # 简单算术
  total: "price * quantity"

  # 条件逻辑
  status_icon: 'if(done, "✅", "⏳")'

  # 字符串格式化
  formatted_price: 'if(price, price.toFixed(2) + " dollars")'

  # 日期格式化
  created: 'file.ctime.format("YYYY-MM-DD")'

  # 计算创建至今的天数（Duration 类型用 .days）
  days_old: '(now() - file.ctime).days'

  # 计算距离截止日期的天数
  days_until_due: 'if(due_date, (date(due_date) - today()).days, "")'
```

## 关键函数

以下为最常用函数。所有类型的完整参考（Date、String、Number、List、File、Link、Object、RegExp）参见 [FUNCTIONS_REFERENCE.md](references/FUNCTIONS_REFERENCE.md)。

| 函数 | 签名 | 说明 |
|----------|-----------|-------------|
| `date()` | `date(string): date` | 将字符串解析为日期（`YYYY-MM-DD HH:mm:ss`） |
| `now()` | `now(): date` | 当前日期时间 |
| `today()` | `today(): date` | 当前日期（时间为 00:00:00） |
| `if()` | `if(condition, trueResult, falseResult?)` | 条件判断 |
| `duration()` | `duration(string): duration` | 解析时长字符串 |
| `file()` | `file(path): file` | 获取文件对象 |
| `link()` | `link(path, display?): Link` | 创建链接 |

### Duration 类型

两个日期相减的结果为 **Duration** 类型（不是数字）。

**Duration 字段：** `duration.days`、`duration.hours`、`duration.minutes`、`duration.seconds`、`duration.milliseconds`

**重要：** Duration 不支持直接调用 `.round()`、`.floor()`、`.ceil()`。需先访问数字字段（如 `.days`），再应用数字函数。

```yaml
# 正确：计算日期间隔天数
"(date(due_date) - today()).days"                    # 返回天数
"(now() - file.ctime).days"                          # 创建至今的天数
"(date(due_date) - today()).days.round(0)"           # 四舍五入后的天数

# 错误 —— 会导致报错：
# "((date(due) - today()) / 86400000).round(0)"      # Duration 不支持先除法再取整
```

### 日期运算

```yaml
# 时长单位：y/year/years, M/month/months, d/day/days,
#           w/week/weeks, h/hour/hours, m/minute/minutes, s/second/seconds
"now() + \"1 day\""       # 明天
"today() + \"7d\""        # 一周后
"now() - file.ctime"      # 返回 Duration
"(now() - file.ctime).days"  # 获取天数（数字）
```

## 视图类型

### 表格视图

```yaml
views:
  - type: table
    name: "My Table"
    order:
      - file.name
      - status
      - due_date
    summaries:
      price: Sum
      count: Average
```

### 卡片视图

```yaml
views:
  - type: cards
    name: "Gallery"
    order:
      - file.name
      - cover_image
      - description
```

### 列表视图

```yaml
views:
  - type: list
    name: "Simple List"
    order:
      - file.name
      - status
```

### 地图视图

需要经纬度属性及 Maps 社区插件。

```yaml
views:
  - type: map
    name: "Locations"
    # 地图专用设置，指定 lat/lng 属性
```

## 默认汇总公式

| 名称 | 输入类型 | 说明 |
|------|------------|-------------|
| `Average` | Number | 算术平均值 |
| `Min` | Number | 最小值 |
| `Max` | Number | 最大值 |
| `Sum` | Number | 求和 |
| `Range` | Number | 最大值减最小值 |
| `Median` | Number | 中位数 |
| `Stddev` | Number | 标准差 |
| `Earliest` | Date | 最早日期 |
| `Latest` | Date | 最晚日期 |
| `Range` | Date | 最晚减最早 |
| `Checked` | Boolean | true 值计数 |
| `Unchecked` | Boolean | false 值计数 |
| `Empty` | Any | 空值计数 |
| `Filled` | Any | 非空值计数 |
| `Unique` | Any | 唯一值计数 |

## 完整示例

### 任务追踪 Base

```yaml
filters:
  and:
    - file.hasTag("task")
    - 'file.ext == "md"'

formulas:
  days_until_due: 'if(due, (date(due) - today()).days, "")'
  is_overdue: 'if(due, date(due) < today() && status != "done", false)'
  priority_label: 'if(priority == 1, "🔴 High", if(priority == 2, "🟡 Medium", "🟢 Low"))'

properties:
  status:
    displayName: Status
  formula.days_until_due:
    displayName: "Days Until Due"
  formula.priority_label:
    displayName: Priority

views:
  - type: table
    name: "Active Tasks"
    filters:
      and:
        - 'status != "done"'
    order:
      - file.name
      - status
      - formula.priority_label
      - due
      - formula.days_until_due
    groupBy:
      property: status
      direction: ASC
    summaries:
      formula.days_until_due: Average

  - type: table
    name: "Completed"
    filters:
      and:
        - 'status == "done"'
    order:
      - file.name
      - completed_date
```

### 阅读清单 Base

```yaml
filters:
  or:
    - file.hasTag("book")
    - file.hasTag("article")

formulas:
  reading_time: 'if(pages, (pages * 2).toString() + " min", "")'
  status_icon: 'if(status == "reading", "📖", if(status == "done", "✅", "📚"))'
  year_read: 'if(finished_date, date(finished_date).year, "")'

properties:
  author:
    displayName: Author
  formula.status_icon:
    displayName: ""
  formula.reading_time:
    displayName: "Est. Time"

views:
  - type: cards
    name: "Library"
    order:
      - cover
      - file.name
      - author
      - formula.status_icon
    filters:
      not:
        - 'status == "dropped"'

  - type: table
    name: "Reading List"
    filters:
      and:
        - 'status == "to-read"'
    order:
      - file.name
      - author
      - pages
      - formula.reading_time
```

### 每日笔记索引

```yaml
filters:
  and:
    - file.inFolder("Daily Notes")
    - '/^\d{4}-\d{2}-\d{2}$/.matches(file.basename)'

formulas:
  word_estimate: '(file.size / 5).round(0)'
  day_of_week: 'date(file.basename).format("dddd")'

properties:
  formula.day_of_week:
    displayName: "Day"
  formula.word_estimate:
    displayName: "~Words"

views:
  - type: table
    name: "Recent Notes"
    limit: 30
    order:
      - file.name
      - formula.day_of_week
      - formula.word_estimate
      - file.mtime
```

## 嵌入 Base

在 Markdown 文件中嵌入：

```markdown
![[MyBase.base]]

<!-- 指定视图 -->
![[MyBase.base#View Name]]
```

## YAML 引号规则

- 公式中含有双引号时，外层用单引号：`'if(done, "Yes", "No")'`
- 简单字符串用双引号：`"My View Name"`
- 复杂表达式中嵌套引号需正确转义

## 故障排查

### YAML 语法错误

**未加引号的特殊字符**：包含 `:`、`{`、`}`、`[`、`]`、`,`、`&`、`*`、`#`、`?`、`|`、`-`、`<`、`>`、`=`、`!`、`%`、`@`、`` ` `` 的字符串必须加引号。

```yaml
# 错误 —— 未加引号字符串中的冒号
displayName: Status: Active

# 正确
displayName: "Status: Active"
```

**公式中引号不匹配**：公式含双引号时，整体用单引号包裹。

```yaml
# 错误 —— 双引号内又嵌套双引号
formulas:
  label: "if(done, "Yes", "No")"

# 正确 —— 单引号包裹双引号
formulas:
  label: 'if(done, "Yes", "No")'
```

### 常见公式错误

**Duration 运算未取字段**：日期相减返回 Duration，不是数字。务必先访问 `.days`、`.hours` 等字段。

```yaml
# 错误 —— Duration 不是数字
"(now() - file.ctime).round(0)"

# 正确 —— 先取 .days 再取整
"(now() - file.ctime).days.round(0)"
```

**缺少空值检查**：并非所有笔记都有该属性。使用 `if()` 进行保护。

```yaml
# 错误 —— due_date 为空时会崩溃
"(date(due_date) - today()).days"

# 正确 —— 用 if() 保护
'if(due_date, (date(due_date) - today()).days, "")'
```

**引用未定义的公式**：`order` 或 `properties` 中的每个 `formula.X` 都必须在 `formulas` 中有对应定义。

```yaml
# 若 'total' 未在 formulas 中定义，此处会静默失败
order:
  - formula.total

# 修正：先定义它
formulas:
  total: "price * quantity"
```

## 参考

- [Bases Syntax](https://help.obsidian.md/bases/syntax)
- [Functions](https://help.obsidian.md/bases/functions)
- [Views](https://help.obsidian.md/bases/views)
- [Formulas](https://help.obsidian.md/formulas)
- [Complete Functions Reference](references/FUNCTIONS_REFERENCE.md)
