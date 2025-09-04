# 🤖 CF AI Chat - 多模型聊天助手

基于Cloudflare Workers AI的单Worker聊天应用，支持6种AI模型，密码保护，历史记录等功能。

**📺 项目作者：YouTube：康康的订阅天地**

## ✨ 功能特点

- 🔐 **密码保护**: 配置文件设置访问密码
- 🤖 **6种AI模型**: DeepSeek、OpenAI、Llama、Qwen、Gemma
- 📚 **历史记录**: KV存储自动保存聊天记录
- 💰 **费用透明**: 显示每个模型的收费和限制
- 📱 **响应式UI**: 支持桌面和移动设备
- ⚡ **纯文本回复**: 返回纯文本，非JSON格式
- 🚀 **单Worker部署**: 一个Worker包含前后端所有功能

## 支持的AI模型

| 模型 | 上下文窗口 | 输入价格 | 输出价格 | 适用场景 |
|------|------------|----------|----------|----------|
| **DeepSeek-R1-Distill-Qwen-32B** | 80,000 tokens | $0.50/M | $4.88/M | 复杂推理、数学计算 |
| **OpenAI GPT-OSS-120B** | 128,000 tokens | $0.35/M | $0.75/M | 高推理需求的通用任务 |
| **OpenAI GPT-OSS-20B** | 128,000 tokens | $0.20/M | $0.30/M | 低延迟、专用应用 |
| **Meta Llama 4 Scout** | 131,000 tokens | $0.27/M | $0.85/M | 多模态、图像理解 |
| **Qwen2.5-Coder-32B** | 32,768 tokens | $0.66/M | $1.00/M | 代码生成和理解 |
| **Gemma 3 12B** | 80,000 tokens | $0.35/M | $0.56/M | 多语言、文本生成 |

## 部署步骤

### 方式一：GitHub + Cloudflare Workers 自动部署（推荐）

#### 1. Fork 本项目

点击右上角的 "Fork" 按钮将项目复制到你的GitHub账户

#### 2. 配置 GitHub Secrets

在你的GitHub仓库中，进入 **Settings** > **Secrets and variables** > **Actions**，添加以下secrets：

- `CLOUDFLARE_API_TOKEN`: 你的Cloudflare API Token
- `CLOUDFLARE_ACCOUNT_ID`: 你的Cloudflare账户ID

#### 3. 修改配置

编辑 `wrangler.toml` 文件：
- 将KV命名空间ID替换为你的真实ID
- 修改密码和其他配置

#### 4. 推送代码

```bash
git add .
git commit -m "Deploy CF AI Chat"
git push
```

GitHub Actions 会自动：
- 部署Worker到Cloudflare
- 部署前端页面到GitHub Pages

#### 5. 访问应用

- 前端页面：`https://yourusername.github.io/cf-gpt-oss`
- API端点：你的Worker域名

### 方式二：手动部署

#### 1. 部署Worker

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **Workers & Pages**
3. 点击 **Create application** > **Create Worker**
4. 将 `src/worker.js` 中的代码复制到编辑器中
5. 点击 **Save and Deploy**

#### 2. 配置Worker

在Worker设置中配置：

**KV Namespace Bindings:**
- Variable name: `CHAT_HISTORY`
- KV namespace: 选择你的KV命名空间

**Environment Variables:**
- `CHAT_PASSWORD`: 你的访问密码
- `MODEL_CONFIG`: 模型配置JSON（见CF_DASHBOARD_CONFIG.md）

**AI Bindings:**
- Variable name: `AI`

#### 3. 部署前端

将 `index.html` 文件上传到任何静态网站托管服务：
- GitHub Pages
- Netlify
- Vercel
- Cloudflare Pages



## 🚀 快速部署

### 5分钟部署步骤
1. **Fork本项目**到您的GitHub账户
2. **创建KV命名空间**在Cloudflare Dashboard
3. **修改wrangler.toml**：更新KV ID和密码
4. **设置GitHub Secrets**：CLOUDFLARE_API_TOKEN 和 CLOUDFLARE_ACCOUNT_ID
5. **提交代码**：自动部署完成

📋 **详细步骤**: [部署说明.md](./部署说明.md)

## 📚 其他文档
- [简单部署指南.md](./简单部署指南.md) - 快速参考
- [作者信息保护说明.md](./作者信息保护说明.md) - 重要！必读！

## 使用说明

1. **访问应用**: 使用Worker域名（如：https://cf-ai-chat.youraccount.workers.dev）
2. **身份验证**: 输入wrangler.toml中设置的密码
3. **选择模型**: 从6个内置模型中选择合适的AI模型
4. **开始聊天**: 输入问题并发送，AI返回纯文本回复
5. **查看历史**: 自动保存在KV存储，可加载历史记录

## API接口说明

### POST /api/chat
发送聊天消息

```json
{
  "message": "你好",
  "model": "deepseek-r1",
  "password": "your_password",
  "history": []
}
```

### GET /api/models
获取支持的模型列表

### GET /api/history
获取聊天历史记录

### POST /api/history
保存聊天历史记录

## 模型参数说明

根据官方文档，不同模型使用不同的参数格式：

- **使用 `messages` 参数的模型**:
  - DeepSeek-R1-Distill-Qwen-32B
  - Meta Llama 4 Scout
  - Qwen2.5-Coder-32B
  - Gemma 3 12B

- **使用 `instructions` 参数的模型**:
  - OpenAI GPT-OSS-120B
  - OpenAI GPT-OSS-20B

## 注意事项

1. **成本控制**: 不同模型收费不同，请根据需求选择合适的模型
2. **上下文限制**: 每个模型都有上下文窗口限制，超出后会自动截断
3. **访问安全**: 请妥善保管访问密码，不要在代码中硬编码
4. **KV存储**: 历史记录保存在Cloudflare KV中，免费套餐有一定限制

## 故障排除

- **部署失败**: 检查KV命名空间ID是否正确配置
- **密码错误**: 确认 `wrangler.toml` 中的密码设置
- **模型调用失败**: 检查Cloudflare Workers AI是否已启用
- **历史记录丢失**: 检查KV绑定是否正确

## 更新日志

- **v1.0.0**: 初始版本，支持6种AI模型和基础聊天功能
