# n8n 客户咨询分类器

基于 n8n 的自动化客户咨询处理工作流，使用 AI 自动分析客户意向并分类处理。

## 功能特性

- 自动接收客户咨询（Webhook）
- AI 分析客户意向（高/中/低）
- 高意向客户：自动邮件通知销售团队
- 低意向客户：自动回复确认邮件
- 响应时间：从 2 小时缩短至 1 分钟

## 文件说明

| 文件 | 说明 |
|------|------|
| `workflow.json` | n8n 工作流配置（已清理敏感数据） |
| `user-manual.md` | 用户使用手册 |
| `faq.md` | 常见问题解答（23 个 FAQ） |
| `technical-docs.md` | 技术文档（部署指南、维护指南） |

## 快速开始

### 1. 导入工作流

1. 启动 n8n：`n8n`
2. 访问 `http://localhost:5678`
3. 点击 "Create Workflow" → "Import from JSON"
4. 粘贴 `workflow.json` 内容

### 2. 配置 API Key

在 "HTTP Request" 节点配置阿里云百炼 API：
- 注册：https://bailian.console.aliyun.com/
- 创建 API Key
- 在 n8n 中配置 "Alibaba Cloud account" credential

### 3. 配置 SMTP

在 "Send an Email" 节点配置邮箱：
- Host: `smtp.qq.com`
- Port: `465`
- 使用 QQ 邮箱授权码（非密码）

### 4. 激活工作流

1. 点击右上角 "Active" 开关
2. 复制 Webhook URL
3. 集成到你的咨询表单

## 技术架构

```
Webhook (接收咨询)
    ↓
HTTP Request (AI 分析意向)
    ↓
If (判断高/中/低)
    ├→ True (高意向) → Send Email (通知销售)
    └→ False (中/低) → Send Email (自动回复客户)
```

## 定价参考

| 版本 | 功能 | 定价 |
|------|------|------|
| 基础版 | 核心功能 | 800-1500 元 |
| 进阶版 | + 数据记录 + 短信通知 | 2000-3000 元 |
| 企业版 | + 企业微信/钉钉 | 5000-12000 元/年 |

## 相关资源

- n8n 官方文档：https://docs.n8n.io/
- 阿里云百炼：https://bailian.console.aliyun.com/
- QQ 邮箱 SMTP 帮助：https://service.mail.qq.com/

## 许可证

MIT License

---

创建时间：2026-05-11
最后更新：2026-05-12
