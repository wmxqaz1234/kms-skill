# Confluence 文档生成风格指南

本指南用于创建/更新 KMS 页面时，生成结构清晰、样式美观的 Confluence 文档。使用 Confluence Storage Format (XML/HTML) 及特定宏实现高级视觉效果。

## 适用场景

- 调研报告（市场调研、用户访谈、竞品分析等）
- 项目总结与复盘（项目回顾、经验沉淀、问题分析等）
- 方案设计文档（技术方案、业务方案、产品设计等）
- 会议纪要与决策记录（会议总结、行动项跟踪、决策依据等）
- 知识沉淀文档（最佳实践、操作手册、问题解决方案等）

## 核心技巧

### 1. 结构化宏 (Structured Macros)

利用 `ac:structured-macro` 标签实现提示框效果，是提升文档颜值的关键。

#### 常用宏类型

| 宏名称 | ac:name | 用途 | 视觉效果 |
| :--- | :--- | :--- | :--- |
| **信息** | `info` | 核心观点、摘要 | 蓝色背景/边框 |
| **提示** | `note` | 补充说明、引用 | 黄色背景/边框 |
| **警告** | `warning` | 风险、痛点、注意 | 红色背景/边框 |
| **建议** | `tip` | 成功经验、最佳实践 | 绿色背景/边框 |
| **代码** | `code` | 代码块 | 代码高亮 |

#### 示例代码

```xml
<p>
  <ac:structured-macro ac:name="info" ac:schema-version="1">
    <ac:rich-text-body>
      <p><strong>核心观点：</strong> 这里放置文档最重要的结论或摘要。</p>
    </ac:rich-text-body>
  </ac:structured-macro>
</p>
```

### 2. 高级表格 (Advanced Tables)

使用 `class="wrapped"` 创建标准边框表格，并结合内联样式进行语义化表达。

- **基础结构**: 使用标准的 HTML `table`, `thead`, `tbody`, `tr`, `th`, `td` 标签。
- **样式增强**: 
    - `<table class="wrapped">`: 添加标准表格边框。
    - `<colgroup>`: 控制列宽。
    - **颜色标记**: 使用 `<span style="color: rgb(R,G,B);">` 高亮关键文本。
        - 🟢 优势/高优先级: `rgb(0,128,0)`
        - 🔴 劣势/风险: `rgb(255,0,0)` or `rgb(255,102,0)`
        - 🟠 中等/关注: `rgb(255,102,0)`
        - ⚫️ 弱化信息: `rgb(153,153,153)`

#### 示例代码

```xml
<table class="wrapped">
  <colgroup><col style="width: 150px;" /><col /></colgroup>
  <thead>
    <tr><th>维度</th><th>说明</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>优先级</strong></td>
      <td><span style="color: rgb(255,0,0);"><strong>高 (High)</strong></span></td>
    </tr>
  </tbody>
</table>
```

### 3. 排版与布局细节

- **层级标题**: 严格使用 `<h2>`, `<h3>` 等构建文档骨架。
- **列表**: 使用 `<ul>` (无序) 和 `<ol>` (有序) 保持条理。
- **文本强调**: 使用 `<strong>` 加粗，`<em>` 斜体。
- **链接与附件**: 
    - 链接: `<a href="...">Text</a>`
    - 附件引用: `<ac:link><ri:attachment ri:filename="file.txt" /></ac:link>`

### 4. 表格背景色应用场景（必须掌握）

**关键规则**：必须将背景色应用到 `<td>` 标签上（Confluence 不支持直接给 `<tr>` 加背景色），同时设置 `class` 和 `data-highlight-colour`。

**典型应用场景**：

#### 场景 A：量化数据表

关键指标（活跃用户数、ARR、续约意向等）用 **浅绿**，增购机会用 **浅蓝**：

```xml
<tr>
  <td class="highlight-green" data-highlight-colour="#e3fcef"><strong>人事场景活跃用户</strong></td>
  <td class="highlight-green" data-highlight-colour="#e3fcef"><span style="color: rgb(0,128,0);"><strong>1,100+（稳定使用）</strong></span></td>
</tr>
<tr>
  <td class="highlight-blue" data-highlight-colour="#e6fcff"><strong>BI增购机会</strong></td>
  <td class="highlight-blue" data-highlight-colour="#e6fcff"><span style="color: rgb(0,0,255);"><strong>已挖掘（客户目前用观远BI）</strong></span></td>
</tr>
```

#### 场景 B：态度/状态变化表

按趋势加颜色渐变——冷态/风险用 **浅红**，僵局/间接触达用 **浅黄**，破冰/认可/信任用 **浅绿**：

```xml
<!-- 冷态 → 红色 -->
<tr>
  <td class="highlight-red" data-highlight-colour="#ffebe6"><strong>4月初</strong></td>
  <td class="highlight-red" data-highlight-colour="#ffebe6"><span style="color: rgb(255,102,0);"><strong>加微信不通过，完全冷态</strong></span></td>
  <td class="highlight-red" data-highlight-colour="#ffebe6">原对接人离职，服务断层</td>
</tr>
<!-- 僵局 → 黄色 -->
<tr>
  <td class="highlight-yellow" data-highlight-colour="#fffae6"><strong>4月15日</strong></td>
  <td class="highlight-yellow" data-highlight-colour="#fffae6">加了微信但不回</td>
  <td class="highlight-yellow" data-highlight-colour="#fffae6">建联僵局</td>
</tr>
<!-- 破冰 → 绿色 -->
<tr>
  <td class="highlight-green" data-highlight-colour="#e3fcef"><strong>4月21日</strong></td>
  <td class="highlight-green" data-highlight-colour="#e3fcef"><span style="color: rgb(0,128,0);"><strong>破冰 + 主动提出P0问题</strong></span></td>
  <td class="highlight-green" data-highlight-colour="#e3fcef">上门回访，发现核心痛点</td>
</tr>
```

#### 场景 C：运营手段/策略表

核心标 **浅绿**，待推进标 **浅黄**，无需标 **浅蓝（灰色文字）**：

```xml
<tr>
  <td class="highlight-green" data-highlight-colour="#e3fcef"><strong>技术能力建联</strong></td>
  <td class="highlight-green" data-highlight-colour="#e3fcef"><span style="color: rgb(0,128,0);"><strong>✅ 核心</strong></span></td>
  <td class="highlight-green" data-highlight-colour="#e3fcef">通过解决技术问题建立信任</td>
</tr>
<tr>
  <td class="highlight-yellow" data-highlight-colour="#fffae6"><strong>高层建联</strong></td>
  <td class="highlight-yellow" data-highlight-colour="#fffae6"><span style="color: rgb(255,102,0);"><strong>待推进</strong></span></td>
  <td class="highlight-yellow" data-highlight-colour="#fffae6">销售准备联系CTO/CEO/CIO</td>
</tr>
<tr>
  <td class="highlight-blue" data-highlight-colour="#e6fcff"><strong>竞品防御</strong></td>
  <td class="highlight-blue" data-highlight-colour="#e6fcff"><span style="color: rgb(122,134,154);"><strong>无需</strong></span></td>
  <td class="highlight-blue" data-highlight-colour="#e6fcff">客户未提及评估其他产品</td>
</tr>
```

## 最佳实践：文档结构模板

适用于调研报告、项目复盘、方案设计等多种场景的通用文档结构：

1.  **核心观点提炼 (Info Macro)** — 置顶展示关键结论，让读者快速把握核心内容。
2.  **基本信息表 (Table)** — 包含时间、参与人、相关链接等元数据。
3.  **正文内容 (H2/H3 + List)** — 分章节阐述背景、现状、分析、结论。使用 `<ul>` 或 `<ol>` 罗列要点。使用灰色小字 `<span style="font-size: 12.0px; color: rgb(153,153,153);">` 标注时间戳或引用来源。
4.  **对比分析 (Table + Color)** — 针对多方案对比或维度分析，使用表格进行横向对比。关键评价使用颜色高亮（绿色表示优势，红色/橙色表示风险或问题）。
5.  **问题与洞察 (Warning/Note Macro)** — 使用 Warning 宏强调风险、阻碍或需要注意的问题。使用 Note 宏展示重要洞察或关键引用。
6.  **行动计划 (Table)** — 列出优先级、行动项、负责人、时间节点等。

## 写作与逻辑规范

### 1. 行文风格 (Wording)

- **拒绝黑话**：使用平实、直接的语言，拒绝"赋能"、"抓手"、"底层逻辑"等互联网黑话。
- **减少括号英文**：除非是专有名词的首次定义，否则不要在中文后紧跟英文括号。
- **克制加粗**：仅高亮真正的关键数据或核心结论，避免满篇黑体。
- **简单直接**：用大白话讲清楚事情，不要过度包装。

### 2. 内容逻辑 (Logic)

- **整体框架清晰**：在文档开头建立清晰的分析框架（时间维度、对象维度、层级维度等）。
- **分层分类明确**：根据内容特点进行合理分层或分类，并明确说明各层级或类别的判断标准和关键差异。
- **主次分明**：核心主线重点阐述，辅助信息适度提及，避免喧宾夺主。

### 3. 视觉增强 (Visual Enhancement)

- **色彩语义化**：使用特定颜色传递明确的情感色彩，避免乱用颜色。
    - 🔵 **核心成果/正向/重点** (`color: rgb(0,0,255)`): 强调关键数据、核心价值、重要结论。
    - 🟠 **问题/风险/注意** (`color: rgb(255,102,0)`): 标注痛点、阻塞点、待解决问题。
    - 🟢 **关键路径/概念** (`color: rgb(0,128,128)`): 强调流程节点、战略动作。
    - 🔘 **辅助/次要信息** (`color: rgb(122,134,154)`): 小标题注释、补充说明，配合 *斜体* 使用。
- **斜体使用**：仅用于次要信息的标注、引用或补充说明，核心内容禁止使用斜体。
- **表格背景色 (Row/Cell Highlighting)**：
    - **关键规则**：必须将背景色应用到 `<td>` 标签上（Confluence 不支持直接给 `<tr>` 加背景色）。
    - **属性要求**：同时设置 `class` 和 `data-highlight-colour`。
    - **常用色值**：
        - 🟢 **浅绿 (成果/优势)**: `class="highlight-green" data-highlight-colour="#e3fcef"`
        - 🔵 **浅蓝 (现状/普通)**: `class="highlight-blue" data-highlight-colour="#e6fcff"`
        - 🟡 **浅黄 (突出/注意)**: `class="highlight-yellow" data-highlight-colour="#fffae6"`
        - 🔴 **浅红 (问题/风险)**: `class="highlight-red" data-highlight-colour="#ffebe6"`

### 4. 版本管理与协作规范 (Read-Before-Write)

为防止 AI 操作覆盖用户在浏览器端的手动修改，必须遵循以下协作流程：

1.  **先读后写**：在对页面进行任何更新操作前，必须先调用 API 读取最新内容。
2.  **增量修改**：必须在读取到的最新 HTML 代码基础上进行修改，而不是使用旧的上下文记忆。
3.  **用户指令**：用户修改完页面后，应提示"我修改了页面，请先读取最新版"。AI 接到修改指令时，若不确定页面是否被手动修改过，应主动读取一次。