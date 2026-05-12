# n8n 项目总仓库

本仓库用于存放多个 n8n 自动化工作流项目。

## 项目列表

| 项目 | 说明 | 状态 | 目录 |
|------|------|------|------|
| 客户咨询分类器 | AI 自动分析客户意向并分类处理 | ✅ 已完成 | `project1-customer-classifier/` |

## 客户咨询分类器

**功能**：自动接收客户咨询 → AI 分析意向（高/中/低）→ 高意向通知销售 + 低意向自动回复

**价值**：响应时间从 2 小时缩短至 1 分钟，销售只跟进高意向客户。

**快速开始**：进入 `project1-customer-classifier/` 目录，查看 `README.md`。

## 仓库结构

```
n8n_project/
├── README.md                       # 总仓库说明（本文件）
├── .gitignore
├── project1-customer-classifier/   # 项目1：客户咨询分类器
│   ├── README.md
│   ├── workflow.json
│   ├── user-manual.md
│   ├── faq.md
│   └── technical-docs.md
├── project2-xxx/                   # 项目2（待添加）
└── project3-yyy/                   # 项目3（待添加）
```

## 使用说明

每个项目独立存放在 `projectN-name/` 目录中，包含：
- `README.md` - 项目说明和快速开始
- `workflow.json` - n8n 工作流配置
- `user-manual.md` - 用户使用手册
- `faq.md` - 常见问题解答
- `technical-docs.md` - 技术文档

## 贡献指南

添加新项目：
1. 创建 `projectN-name/` 目录
2. 添加项目文件
3. 更新本 README.md 的项目列表
4. 提交并推送

## 相关资源

- n8n 官网：https://n8n.io/
- n8n 文档：https://docs.n8n.io/
- n8n 工作流示例：https://n8n.io/workflows/

## License

MIT License

---

创建时间：2026-05-12
最后更新：2026-05-12
