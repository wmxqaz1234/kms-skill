---
name: kms
description: 当用户提到 KMS、Confluence、知识库页面，或提供 kms.fineres.com 链接时，使用此 skill 查看、创建和更新页面。自动识别包含 pageId 的 URL。创建/更新页面时，按 references/confluence-doc-style.md 的风格指南生成高质量文档。
allowed-tools: Bash, Read, Write, WebFetch
auto-trigger-keywords: kms, confluence, 知识库, kms.fineres.com, pageId, 查看页面, 创建页面, 更新页面, 写文档, 生成文档, 调研报告, 会议纪要
---

# KMS (Knowledge Management System) Skill

这个 skill 用于与 Confluence KMS 系统交互，支持查看、创建和更新页面。

**自动触发条件：**
- 用户提到 "KMS"、"知识库"、"Confluence"
- 用户提供 kms.fineres.com 的 URL
- 用户请求查看/创建/更新页面
- 消息中包含 pageId

## 前置配置

凭据通过 Corevo 连接中心管理（slug: `kms`）。bash/execute_python 中通过环境变量读取：

```bash
# 环境变量自动注入，无需手动配置
KMS_BASE_URL → $KMS_BASE_URL
KMS_API_TOKEN → $KMS_API_TOKEN
KMS_USERNAME → $KMS_USERNAME
```

**认证方式：** 使用 Bearer Token 认证（不是 Basic Auth）

### 如何获取 API Token

1. 登录 KMS 系统
2. 点击右上角头像 → 设置 → 个人访问令牌
3. 创建新令牌并保存
4. 在 Corevo 连接中心填写 Token

## 核心约束

- 必须先在连接中心配置 KMS 凭据才能使用
- 所有操作需要相应的 KMS 权限
- 更新页面时必须提供正确的版本号
- 页面内容使用 Confluence Storage Format (HTML-like)
- **⚠️ 重要：URL 中的中文参数必须进行 URL 编码**
- **🔴 创建/修改范围限制：只允许在 Avince（https://kms.fineres.com/display/jiandaoyun/Avince，pageId: 1352284265）下创建和修改子页面。禁止在其他空间或页面下创建/修改内容。查看和搜索不受此限制。**
- **⚠️ 操作确认：任何创建、修改、删除操作必须先向用户确认内容和意图，获得明确同意后才执行。不得未经确认直接写入/修改/删除页面。**

## 执行步骤

### 功能 1: 查看页面

当用户请求查看 KMS 页面时：

**步骤 1: 提取页面 ID**

从 URL 或直接的页面 ID 中提取 pageId：
- URL 格式: `https://kms.fineres.com/pages/viewpage.action?pageId=72143735`
- 提取 pageId: `72143735`

**步骤 2: 调用 API 获取页面内容**

```bash
# 获取页面详细信息和内容
curl -X GET \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$KMS_BASE_URL/rest/api/content/${PAGE_ID}?expand=body.storage,version,space" \
  -s
```

**步骤 3: 解析并展示结果**

从返回的 JSON 中提取：
- `title`: 页面标题
- `body.storage.value`: 页面内容（HTML 格式）
- `version.number`: 当前版本号
- `space.name`: 所属空间名称
- `_links.webui`: 页面链接

以清晰的格式展示给用户。

### 功能 2: 创建页面

当用户请求创建新页面时：

**步骤 1: 收集必要信息**

需要以下信息：
- 页面标题 (title)
- 页面内容 (content)
- 父页面 ID (ancestorId) - 可选
- 空间 Key (spaceKey) - 必需

**步骤 2: 准备页面内容**

将用户提供的内容转换为 Confluence Storage Format。如果是 Markdown，需要转换为 HTML。
- 空页面使用：`""`（空字符串）
- 简单段落：`"<p>内容</p>"`

**步骤 3: 调用 API 创建页面（推荐使用 heredoc 格式）**

```bash
# 创建新页面 - 使用 heredoc 格式更可靠
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  -d @- \
  "$KMS_BASE_URL/rest/api/content" -s << 'EOF'
{
  "type": "page",
  "title": "页面标题",
  "space": {
    "key": "jiandaoyun"
  },
  "ancestors": [
    {
      "id": "1352284265"
    }
  ],
  "body": {
    "storage": {
      "value": "<p>页面内容</p>",
      "representation": "storage"
    }
  }
}
EOF
```

**注意事项：**
- 使用 heredoc (`<< 'EOF'`) 避免 shell 变量替换问题
- `ancestors` 必须指定 Avince pageId: `1352284265`
- `space` 固定为 `jiandaoyun`
- `body.storage.value` 可以是空字符串 `""`

**步骤 4: 返回结果**

返回新创建页面的：
- 页面 ID
- 完整 URL (`_links.webui`)
- 标题
- 创建时间

### 功能 3: 更新页面

当用户请求更新现有页面时：

**步骤 1: 获取当前页面信息**

首先获取页面的当前版本号和内容（更新时必需）：

```bash
curl -X GET \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/content/${PAGE_ID}?expand=body.storage,version" \
  -s
```

从返回结果中提取：
- `version.number`: 当前版本号
- `body.storage.value`: 当前内容（如需追加）
- `title`: 当前标题

**步骤 2: 准备更新内容**

根据用户需求：
- 完全替换内容
- 追加内容到现有内容后（需要先获取当前内容）
- 修改特定部分

**步骤 3: 调用 API 更新页面**

```bash
# 更新页面 - 使用 heredoc 格式
NEW_VERSION=$((CURRENT_VERSION + 1))

curl -X PUT \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  -d @- \
  "$KMS_BASE_URL/rest/api/content/${PAGE_ID}" -s << EOF
{
  "id": "${PAGE_ID}",
  "type": "page",
  "title": "更新后的标题",
  "version": {
    "number": ${NEW_VERSION},
    "message": "Updated via Claude Code"
  },
  "body": {
    "storage": {
      "value": "<p>更新后的内容</p>",
      "representation": "storage"
    }
  }
}
EOF
```

**重要提示：**
- `version.number` 必须是当前版本号 + 1
- 如果版本号不正确会返回 409 Conflict 错误
- `version.message` 是可选的更新说明

**步骤 4: 确认更新**

返回更新后的页面信息和链接。

### 功能 4: 搜索页面（按标题精确匹配）

当用户提供页面标题需要查找页面时：

**步骤 1: URL 编码标题（关键步骤）**

⚠️ **必须进行 URL 编码，否则搜索会失败！**

```bash
# 对中文标题进行 URL 编码
ENCODED_TITLE=$(python -c "import urllib.parse; print(urllib.parse.quote('工作相关-赵文轩'))")
```

**步骤 2: 调用搜索 API**

```bash
# 按标题精确搜索（必须指定 spaceKey）
curl -X GET \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/content?spaceKey=jiandaoyun&title=${ENCODED_TITLE}&expand=body.storage,version,space" \
  -s
```

**关键参数：**
- `spaceKey`: 必需，指定搜索的空间（如 DR）
- `title`: 精确匹配的标题（必须 URL 编码）
- `expand`: 扩展返回的信息

**步骤 3: 解析结果**

从 `results` 数组中提取页面信息。如果 `size` 为 0，说明没找到匹配的页面。

### 功能 5: 获取 Space 列表

当不确定页面所属空间时：

```bash
# 获取所有可访问的 space
curl -X GET \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/space?limit=50" \
  -s
```

从结果中查看：
- `key`: Space Key（用于搜索和创建页面）
- `name`: Space 名称
- `_links.webui`: Space 链接

**常用 Space：**
- `jiandaoyun`: 2. 简道云（Avince 页面所在空间）
- `DR`: 3.2 研发测试组
- `FIN`: 2. FineReport
- `BI`: 2. FineBI
- `team`: 3.2 产品组

### 功能 6: 获取页面子页面

```bash
# 获取某页面的所有子页面
curl -X GET \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/content/${PAGE_ID}/child/page?expand=version&limit=100" \
  -s
```

### 功能 7: 删除页面

```bash
# 删除页面（需要管理员权限）
curl -X DELETE \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/content/${PAGE_ID}" \
  -s
```

## 内容格式处理

### Markdown 转 Confluence Storage Format

当用户提供 Markdown 格式内容时，使用以下映射：

```
# 标题       → <h1>标题</h1>
## 子标题    → <h2>子标题</h2>
**粗体**     → <strong>粗体</strong>
*斜体*       → <em>斜体</em>
[链接](url)  → <a href="url">链接</a>
- 列表项     → <ul><li>列表项</li></ul>
1. 编号列表  → <ol><li>项目</li></ol>
```

### 文档风格指南（创建/更新页面时必读）

**创建或更新页面内容时，必须先读取 `references/confluence-doc-style.md`**，按其中的风格指南生成文档。核心要点速览：

1. **结构化宏**：核心结论用 `info` 宏，风险用 `warning`，洞察用 `note`，最佳实践用 `tip`
2. **高级表格**：使用 `class="wrapped"`，关键单元格用背景色区分（`highlight-green/blue/yellow/red` 加在 `<td>` 上）
3. **色彩语义化**：蓝色 `rgb(0,0,255)` 标核心成果，橙色 `rgb(255,102,0)` 标风险，青色 `rgb(0,128,128)` 标关键路径，灰色 `rgb(122,134,154)` 标辅助信息
4. **文档结构模板**：Info 宏摘要 → 基本信息表 → H2/H3 正文 → 对比分析表 → Warning/Note 洞察 → 行动计划表
5. **写作规范**：拒绝黑话、克制加粗、主次分明、先读后写（更新前必须先获取最新内容）

### 自动排版增强（生成内容时必须应用）

当生成 Confluence Storage Format 内容时，**必须主动应用以下排版增强**，不能只做简单的 Markdown→HTML 映射：

#### 1. 顶部摘要 Info 宏

每个文档开头必须加 `info` 宏包裹核心结论：

```xml
<ac:structured-macro ac:name="info" ac:schema-version="1">
  <ac:parameter ac:name="title">核心结论</ac:parameter>
  <ac:rich-text-body>
    <p>这里放置文档最重要的结论或摘要，让读者快速把握核心内容。</p>
  </ac:rich-text-body>
</ac:structured-macro>
```

#### 2. 表格背景色语义化

**所有表格必须加 `class="wrapped"`**，并根据内容语义给 `<td>` 加背景色：

| 场景 | 背景色 | class + data-highlight-colour |
|------|--------|-------------------------------|
| 核心成果/优势 | 🟢 浅绿 | `class="highlight-green" data-highlight-colour="#e3fcef"` |
| 现状/普通信息 | 🔵 浅蓝 | `class="highlight-blue" data-highlight-colour="#e6fcff"` |
| 待关注/待推进 | 🟡 浅黄 | `class="highlight-yellow" data-highlight-colour="#fffae6"` |
| 问题/风险 | 🔴 浅红 | `class="highlight-red" data-highlight-colour="#ffebe6"` |

**典型应用场景：**

- **量化数据表**：关键指标（活跃用户数、ARR、续约意向等）用 **浅绿**；增购机会用 **浅蓝**
- **态度/状态变化表**：按趋势加颜色渐变——冷态/风险用 **浅红**，僵局/间接触达用 **浅黄**，破冰/认可/信任用 **浅绿**
- **运营手段/策略表**：核心标 **浅绿**，待推进标 **浅黄**，无需/已放弃标 **浅蓝（灰色文字）**

#### 3. 关键数据自动高亮

以下内容必须用 `<span style="color: rgb(0,0,255);"><strong>...</strong></span>` 高亮：
- 金额类：ARR、合同金额、预算等
- 数量类：活跃用户数、员工数、覆盖率等
- 比例类：增长率、转化率、满意度等
- 时间类：关键里程碑日期

#### 4. 风险/警示内容用 Warning 宏

涉及流失风险、产品缺陷、服务断层等负面信息时，用 `warning` 宏包裹：

```xml
<ac:structured-macro ac:name="warning" ac:schema-version="1">
  <ac:parameter ac:name="title">风险警示</ac:parameter>
  <ac:rich-text-body>
    <p>这里描述风险内容...</p>
  </ac:rich-text-body>
</ac:structured-macro>
```

#### 5. 最佳实践用 Tip 宏

成功经验、创新做法、可复用方法论等内容用 `tip` 宏包裹。

#### 6. 色彩语义化速查

| 颜色 | rgb | 用途 | 示例 |
|------|-----|------|------|
| 🔵 蓝色 | `rgb(0,0,255)` | 核心成果、关键数据、重要结论 | ARR 314,000元 |
| 🟠 橙色 | `rgb(255,102,0)` | 问题、风险、待解决 | 流失风险、P0问题 |
| 🟢 绿色 | `rgb(0,128,0)` | 优势、正向结果、已达成 | 续约意向明显加强 |
| 🔘 灰色 | `rgb(122,134,154)` | 辅助信息、次要注释 | *数据来源：简道云系统* |

## 常见问题 FAQ

### ❌ 问题 1: 搜索中文标题失败

**症状：** 搜索返回空结果或 400 错误

**原因：** 中文字符未进行 URL 编码

**解决方案：**
```bash
# ❌ 错误方式
curl "$KMS_BASE_URL/rest/api/content?title=工作相关-赵文轩"

# ✅ 正确方式
ENCODED_TITLE=$(python -c "import urllib.parse; print(urllib.parse.quote('工作相关-赵文轩'))")
curl "$KMS_BASE_URL/rest/api/content?title=${ENCODED_TITLE}"
```

### ❌ 问题 2: 创建页面返回 500 错误

**症状：** `{"statusCode":500,"message":"","reason":"Internal Server Error"}`

**可能原因：**
1. JSON 格式错误（引号嵌套问题）
2. `ancestors` ID 不存在
3. 权限不足

**解决方案：**
- 使用 heredoc 格式避免引号嵌套问题
- 验证父页面 ID 是否存在
- 确认对该 space 有创建权限

### ❌ 问题 3: 更新页面返回 409 Conflict

**症状：** 版本号冲突错误

**原因：** 提供的版本号不是当前版本号 + 1

**解决方案：**
```bash
# 1. 先获取当前版本号
CURRENT=$(curl -s -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/content/${PAGE_ID}?expand=version" | \
  python -m json.tool | grep '"number"' | head -1 | grep -o '[0-9]*')

# 2. 计算新版本号
NEW_VERSION=$((CURRENT + 1))

# 3. 使用正确的版本号更新
```

### ❌ 问题 4: 401 Unauthorized 错误

**原因：** 认证信息错误

**检查清单：**
- [ ] 环境变量 `KMS_API_TOKEN` 是否设置
- [ ] Token 是否有效（未过期）
- [ ] 使用的是 Bearer Token 认证（不是 Basic Auth）

**验证命令：**
```bash
echo "Token: $KMS_API_TOKEN"
```

### ❌ 问题 5: 找不到页面（按标题搜索）

**原因：** 没有指定 `spaceKey`

**解决方案：**
```bash
# ❌ 可能搜不到
curl "$KMS_BASE_URL/rest/api/content?title=${TITLE}"

# ✅ 指定 space
curl "$KMS_BASE_URL/rest/api/content?spaceKey=jiandaoyun&title=${TITLE}"
```

## 错误处理

### 401 Unauthorized
- 检查环境变量 `KMS_API_TOKEN` 是否正确配置
- 验证 API Token 是否有效
- 确认使用的是 Bearer Token 认证

### 403 Forbidden
- 用户没有该页面的访问权限
- 检查空间权限设置
- 确认有创建/编辑权限

### 404 Not Found
- 页面 ID 不存在
- 检查 URL 和页面 ID 是否正确
- 页面可能已被删除

### 409 Conflict
- 版本号冲突（其他人同时编辑了页面）
- 重新获取最新版本号后再次尝试

### 500 Internal Server Error
- JSON 格式错误
- 检查请求体的引号嵌套
- 使用 heredoc 格式避免格式问题

## 使用示例

### 示例 1: 查看页面

```
User: 帮我查看 KMS 页面 https://kms.fineres.com/pages/viewpage.action?pageId=72143735
Claude: [提取 pageId=72143735，调用查看 API，展示页面内容]
```

### 示例 2: 按标题查找页面

```
User: 查看"工作相关-赵文轩"这个页面
Claude: [URL 编码标题，在 DR space 中搜索，找到页面 ID，展示内容]
```

### 示例 3: 创建子页面

```
User: 在"工作相关-赵文轩"下创建一个子页面，标题是"26年-james"，正文为空
Claude: [先搜索父页面获取 ID，使用 heredoc 格式创建空白子页面，返回新页面链接]
```

### 示例 4: 更新页面内容

```
User: 更新页面 72143735，在末尾添加新的一节
Claude: [获取当前内容和版本号，追加新内容，使用版本号+1 更新页面]
```

### 示例 5: 获取 Space 列表

```
User: KMS 中有哪些 space？
Claude: [调用 space API，列出所有可访问的空间及其 key]
```

## 最佳实践

1. **URL 编码优先**: 任何中文参数都必须 URL 编码
2. **使用 heredoc**: 创建和更新页面时使用 heredoc 避免引号问题
3. **指定 spaceKey**: 搜索时总是指定 spaceKey 提高准确性
4. **更新前备份**: 重要页面更新前，先获取并保存当前内容
5. **版本号检查**: 更新前总是获取最新版本号
6. **权限确认**: 确认对目标空间/页面有适当权限
7. **增量更新**: 大量更新时，分批次进行，避免超时
8. **测试优先**: 复杂操作先在测试页面验证

## 工作流程总结

### 典型流程 1: 查找并查看页面

```
1. 用户提供页面标题
2. URL 编码标题
3. 在指定 space 中搜索
4. 获取页面 ID
5. 调用查看 API
6. 展示页面内容
```

### 典型流程 2: 创建页面

```
1. 确定父页面 ID（如有）
2. 确定 space key
3. 准备页面内容（Markdown → HTML）
4. 使用 heredoc 格式调用创建 API
5. 返回新页面链接
```

### 典型流程 3: 更新页面

```
1. 获取当前页面信息（内容 + 版本号）
2. 根据需求准备新内容
3. 计算新版本号（当前 + 1）
4. 使用 heredoc 格式调用更新 API
5. 确认更新成功
```

## 注意事项

- ✅ API Token 是敏感信息，使用环境变量管理
- ✅ 所有中文参数必须 URL 编码
- ✅ 使用 Bearer Token 认证，不是 Basic Auth
- ✅ 创建/更新页面推荐使用 heredoc 格式
- ✅ 更新页面时版本号必须正确
- ✅ 搜索时指定 spaceKey 提高准确性
- ⚠️ 大规模操作前先在测试环境验证
- ⚠️ 遵守公司的 KMS 使用规范
- ⚠️ 删除操作不可逆，谨慎使用
