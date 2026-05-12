# n8n 智能咨询分类器 - 技术文档补充

## 架构说明

### 工作流结构

```
Webhook (接收咨询)
    ↓
解析Body (Code节点，统一数据格式)
    ↓
AI分析 (HTTP Request，调用智谱AI)
    ↓
清洗输出 (Code节点，提取意向等级)
    ↓
判断意向 (If节点)
    ├→ True (高意向) → 通知销售 (Send Email)
    └→ False (中/低意向) → 自动回复 (Send Email)
```

### 数据流转

**输入格式**：
```json
{
  "name": "客户姓名",
  "email": "客户邮箱",
  "contact": "联系方式",
  "message": "咨询内容"
}
```

**输出结果**：
- 高意向：销售邮箱收到通知邮件
- 中/低意向：客户邮箱收到自动回复

---

## 部署指南

### 方案A：本地部署（测试用）

1. 安装 Node.js 18+
2. 安装 n8n：`npm install -g n8n`
3. 启动：`npx n8n`
4. 访问 `http://localhost:5678`
5. 导入工作流 JSON
6. 配置 API Key 和 SMTP
7. 激活工作流

**限制**：本地部署时，电脑关机后 Webhook 无法访问。

---

### 方案B：云服务器部署（生产用）

#### B1. 购买云服务器

推荐配置：
- **阿里云/腾讯云**：2核4G，约 100元/月
- **操作系统**：Ubuntu 20.04 / CentOS 7
- **带宽**：1-5 Mbps 足够

#### B2. 安装 Node.js 和 n8n

```bash
# 安装 Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 安装 n8n
sudo npm install -g n8n

# 验证安装
n8n --version
```

#### B3. 使用 PM2 管理进程（自动重启）

```bash
# 安装 PM2
sudo npm install -g pm2

# 启动 n8n
pm2 start n8n

# 设置开机自启
pm2 startup
pm2 save

# 查看状态
pm2 status
pm2 logs n8n
```

#### B4. 配置反向代理（可选，推荐）

使用 Nginx 反向代理，支持域名访问和 HTTPS：

```bash
# 安装 Nginx
sudo apt-get install nginx

# 配置反向代理
sudo nano /etc/nginx/sites-available/n8n
```

配置内容：
```
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

启用配置：
```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### B5. 配置 HTTPS（推荐）

使用 Let's Encrypt 免费证书：

```bash
# 安装 Certbot
sudo apt-get install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d your-domain.com

# 自动续期（已自动配置 cron）
sudo certbot renew --dry-run
```

---

### 方案C：Docker 部署（进阶）

创建 `docker-compose.yml`：

```yaml
version: '3'
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your_password
      - N8N_HOST=your-domain.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

启动：
```bash
docker-compose up -d
```

---

## 截图标注说明

根据原文档中的截图位标注，以下是每个截图应展示的内容：

### 截图1：n8n 工作流创建界面
- 展示：登录后点击 "Create Workflow"
- 标注：鼠标点击位置、名称输入框

### 截图2：Webhook 节点配置
- 展示：Method=POST, Path=customer-inquiry
- 标注：Response Mode 选择 "Immediately"

### 截图3：Webhook 测试输出
- 展示：Test Webhook 返回的 JSON 数据
- 标注：body 字段正确解析

### 截图4：Code 节点（解析Body）
- 展示：JavaScript 代码
- 标注：JSON.parse() 处理逻辑

### 截图5：HTTP Request 节点（AI分析）
- 展示：URL、Headers、Body 配置
- 标注：Authorization Header、模型名称

### 截图6：AI 分析输出
- 展示：返回 `{"output": {"text": "高"}}`
- 标注：提取 text 字段

### 截图7：Code 节点（清洗输出）
- 展示：意向等级判断逻辑
- 标注：includes('高') 判断

### 截图8：If 节点配置
- 展示：Left Value=`{{ $json.level }}`, Operation=`equals`, Right Value=`高`
- 标注：True/False 两个输出点

### 截图9：Send Email 节点（通知销售）
- 展示：SMTP 配置、邮件内容
- 标注：To、Subject、Body 中的表达式

### 截图10：Send Email 节点（自动回复）
- 展示：客户邮箱 `{{ $json.email }}`
- 标注：自动回复内容模板

### 截图11：完整工作流截图
- 展示：所有节点连接的完整流程图
- 标注：数据流向、关键节点名称

### 截图12：激活工作流
- 展示：右上角 "Active" 开关为绿色
- 标注：Webhook URL 位置

---

## 维护指南

### 日常检查清单

**每天**：
- [ ] 查看 n8n 执行日志（有无失败）
- [ ] 检查销售邮箱是否收到通知邮件

**每周**：
- [ ] 查看 AI API 余额
- [ ] 检查服务器资源使用（CPU/内存/磁盘）

**每月**：
- [ ] 备份工作流 JSON
- [ ] 分析咨询数据（高/中/低意向占比）
- [ ] 优化提示词（根据误判情况）

---

### 监控配置

#### 添加错误通知

在 n8n 中添加 "Error Trigger" 节点：
1. 点击 "Add Trigger" → "Error Trigger"
2. 连接到通知节点（邮件/企业微信）
3. 工作流出错时自动通知

#### 查看执行统计

访问 n8n 控制台：
- Executions → 查看历史执行记录
- 统计成功/失败率
- 分析平均响应时间

---

### 性能优化

#### 1. 提升 AI 分析速度

- 使用更快的模型（`glm-4-flash` 约 1-2 秒）
- 降低 `temperature`（设为 0.1）
- 简化提示词（减少 token 数）

#### 2. 提升并发能力

- 升级服务器配置（2核4G → 4核8G）
- 使用 n8n 队列模式（需 Redis）
- 切换为付费 AI API（更高 QPS）

#### 3. 减少邮件发送延迟

- 使用企业邮箱（QQ 邮箱有发送频率限制）
- 配置多个 SMTP 账号轮询
- 使用邮件服务的 API（如 SendGrid、阿里云邮件推送）

---

## 安全建议

### 1. 保护 Webhook URL

- 不要公开分享完整 URL
- 在 Webhook 节点添加 `Authentication`（Header Auth 或 Basic Auth）
- 限制 IP 访问（防火墙规则）

### 2. 保护 API Key

- 不要在代码中硬编码 Key
- 使用 n8n 的 `Credential` 功能存储 Key
- 定期轮换 Key

### 3. 数据脱敏

在 "解析Body" 节点中添加脱敏逻辑：

```javascript
// 手机号脱敏
if (body.contact) {
  body.contact = body.contact.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2');
}

// 邮箱脱敏
if (body.email) {
  let [name, domain] = body.email.split('@');
  body.email = name.substring(0, 3) + '***@' + domain;
}
```

---

## 故障排查详细步骤

### 问题1：工作流没有触发

**排查步骤**：
1. 检查 n8n 是否运行：`pm2 status` 或访问 `http://localhost:5678`
2. 检查工作流是否激活：右上角开关是否为绿色
3. 检查 Webhook URL 是否正确：复制节点中的 URL 测试
4. 检查防火墙：确保 5678 端口对外开放
5. 查看 n8n 日志：`pm2 logs n8n`

### 问题2：AI 分析一直失败

**排查步骤**：
1. 手动测试 API：用 curl 或 Postman 调用智谱 AI API
2. 检查 Key 是否正确：在智谱 AI 控制台验证
3. 检查余额：是否充足
4. 查看详细错误：在 n8n 执行日志中查看 HTTP Request 节点的错误输出

### 问题3：邮件发送失败

**排查步骤**：
1. 检查 SMTP 配置：Host/Port/SSL/授权码
2. 测试 SMTP 连接：使用 Telnet 或在线工具测试
3. 检查授权码是否过期：重新获取
4. 查看 n8n 日志：Send Email 节点的错误输出

---

## 进阶功能扩展

### 1. 添加 Google Sheets 记录

在 "判断意向" 节点后添加 "Google Sheets" 节点：
- 认证 Google 账号
- 创建 Sheets（字段：时间、姓名、邮箱、联系方式、咨询内容、意向等级）
- 每次咨询自动追加一行

### 2. 添加企业微信通知

在 "判断意向" 节点的 True 分支后添加 "HTTP Request" 节点：
- URL：企业微信机器人 Webhook URL
- Method：POST
- Body：
```json
{
  "msgtype": "text",
  "text": {
    "content": "高意向客户：{{ $json.name }}\n咨询内容：{{ $json.message }}"
  }
}
```

### 3. 添加数据分析报表

使用 "Schedule" 触发器 + "Google Sheets" 节点：
- 每周一自动读取上周咨询数据
- 统计高/中/低意向占比
- 生成图表（使用 QuickChart API）
- 发送邮件报表

---

最后更新：2026-05-11
