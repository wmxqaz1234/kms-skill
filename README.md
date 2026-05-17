# KMS Skill

Confluence 知识库（KMS）交互 Skill，支持查看、创建和更新页面。

## 功能

- **查看页面**：通过 pageId 或页面标题获取页面内容
- **创建页面**：在指定父页面下创建子页面
- **更新页面**：修改现有页面的内容和标题
- **搜索页面**：按标题精确搜索页面
- **删除页面**：删除指定页面（需管理员权限）

## 前置配置

凭据通过 Corevo 连接中心管理（slug: `kms`）：

```bash
export KMS_BASE_URL="https://kms.fineres.com"
export KMS_API_TOKEN="your-token"
export KMS_USERNAME="your-username"
```

## 安装

```bash
git clone https://github.com/wmxqaz1234/kms-skill.git ~/.claude/skills/kms
```

## 使用示例

```bash
# 查看页面
curl -H "Authorization: Bearer $KMS_API_TOKEN" \
  "$KMS_BASE_URL/rest/api/content/{PAGE_ID}?expand=body.storage"

# 创建页面
curl -X POST \
  -H "Authorization: Bearer $KMS_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"page","title":"标题","space":{"key":"jiandaoyun"}}' \
  "$KMS_BASE_URL/rest/api/content"
```

## 文档

完整文档请查看 [SKILL.md](./SKILL.md)

## 约束

- 创建/修改页面仅限在 **Avince**（pageId: 1352284265）下
- 任何创建/修改/删除操作需先确认

## License

MIT
