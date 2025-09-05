# CF AI Chat - Cloudflare Workers AI 聊天应用

一个基于 Cloudflare Workers AI 的多模型智能聊天应用，支持多种 AI 模型，具备代码自动识别和复制功能。

## ✨ 主要功能

- 🤖 **多模型支持**：DeepSeek-R1、GPT-OSS-120B/20B、Llama-4-Scout、Qwen2.5-Coder、Gemma-3
- 💬 **实时聊天**：流畅的对话体验，支持历史记录
- 🔐 **密码保护**：安全的访问控制
- 💾 **历史记录**：按模型分别保存聊天历史
- 🎨 **代码高亮**：自动识别代码块，支持多种编程语言
- 📋 **一键复制**：完整保持代码格式的复制功能
- 📱 **响应式设计**：支持桌面和移动端

## 🚀 完整部署流程

### 第一步：准备 Cloudflare 账号

1. **注册 Cloudflare 账号**
   - 访问 [https://cloudflare.com](https://cloudflare.com) 注册账号
   - 完成邮箱验证
   - 确保账号可以访问 Workers AI 功能

2. **检查权限和余额**
   - 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
   - 确认账号有 Workers AI 使用权限
   - 检查账号余额（AI 调用会产生费用）

### 第二步：创建 Cloudflare Worker

1. **创建新 Worker**
   - 登录 Cloudflare Dashboard
   - 选择 "Workers"
   - 点击 "Create application"
   - 选择 "Create Worker"
   - 输入应用名称（推荐：`cf-ai-chat`）
   - 点击 "Deploy"

2. **获取 Worker URL**
   - 部署完成后会显示 Worker 的访问 URL
   - 记录这个 URL，稍后需要访问测试

### 第三步：配置环境变量

#### 必需的环境变量

| 变量名 | 类型 | 必需 | 说明 | 示例值 |
|--------|------|------|------|--------|
| `CHAT_PASSWORD` | 字符串 | ✅ | 聊天应用的访问密码 | `MySecurePass123!` |

#### 在 Cloudflare Dashboard 中设置环境变量

1. **进入 Worker 设置页面**
   - 在 Worker 列表中点击你刚创建的 Worker
   - 点击 "Settings" 标签

2. **添加环境变量**
   - 找到 "Environment Variables" 部分
   - 点击 "Add variable"
   - 输入以下信息：
     ```
     Variable name: CHAT_PASSWORD
     Value: your_secure_password_123
     ```
   - 点击 "Save and deploy"

3. **环境变量安全建议**
   - 使用至少12位字符的强密码
   - 包含大小写字母、数字和特殊字符
   - 避免使用常见密码或个人信息
   - 定期更换密码

### 第四步：配置 KV 存储（可选 - 用于聊天历史）

如果需要保存聊天历史记录功能：

1. **创建 KV 命名空间**
   - 在 Cloudflare Dashboard 中选择 "Workers & Pages"
   - 点击 "KV" 标签
   - 点击 "Create a namespace"
   - 输入命名空间名称：`CHAT_HISTORY`
   - 点击 "Add"
   - 记录生成的 Namespace ID

2. **绑定 KV 到 Worker**
   - 回到你的 Worker 设置页面
   - 在 "Settings" > "Variables" 中找到 "KV Namespace Bindings"
   - 点击 "Add binding"
   - 输入以下信息：
     ```
     Variable name: CHAT_HISTORY
     KV namespace: 选择刚创建的 CHAT_HISTORY 命名空间
     ```
   - 点击 "Save and deploy"

### 第五步：部署应用代码

#### 方法一：在线编辑器部署（推荐）

1. **编辑 Worker 代码**
   - 在 Worker 详情页面点击 "Quick edit"
   - 删除编辑器中的所有默认代码

2. **复制应用代码**
   - 打开本项目的 `src/worker.js` 文件
   - 复制全部内容
   - 粘贴到 Cloudflare 在线编辑器中

3. **保存并部署**
   - 点击 "Save and deploy"
   - 等待部署完成

#### 方法二：使用 Wrangler CLI 部署

1. **安装 Wrangler**
   ```bash
   npm install -g wrangler
   ```

2. **登录 Cloudflare**
   ```bash
   wrangler auth login
   ```

3. **下载项目代码**
   ```bash
   git clone <your-repo-url>
   cd cf-gpt-oss
   ```

4. **修改配置文件**
   编辑 `wrangler.toml` 文件：
   ```toml
   name = "cf-ai-chat"
   main = "src/worker.js"
   compatibility_date = "2024-01-15"
   
   [ai]
   binding = "AI"
   
   # 如果配置了 KV 存储，替换下面的 ID
   [[kv_namespaces]]
   binding = "CHAT_HISTORY"
   id = "your_kv_namespace_id_here"
   preview_id = "your_kv_namespace_id_here"
   
   [vars]
   CHAT_PASSWORD = "your_secure_password_here"
   ```

5. **部署到 Cloudflare**
   ```bash
   wrangler deploy
   ```

### 第六步：配置自定义域名（可选）

如果你有自己的域名：

1. **添加域名到 Cloudflare**
   - 在 Cloudflare Dashboard 中点击 "Add a Site"
   - 输入你的域名
   - 按照指引更新 DNS 服务器

2. **设置 Worker 路由**
   - 在 Worker 设置页面点击 "Triggers" 标签
   - 点击 "Add Custom Domain"
   - 输入子域名（如：`ai-chat.yourdomain.com`）
   - 点击 "Add Custom Domain"

## 🔧 支持的 AI 模型

| 模型名称 | 模型 ID | 特点 | 适用场景 | 成本 |
|----------|---------|------|----------|------|
| **DeepSeek-R1** | `@cf/deepseek-ai/deepseek-r1-distill-qwen-32b` | 思维链推理，支持复杂逻辑 | 数学计算、逻辑推理、复杂分析 | 中等 |
| **GPT-OSS-120B** | `@cf/openai/gpt-oss-120b` | 大参数量，高质量输出 | 创意写作、文本分析、通用对话 | 较高 |
| **GPT-OSS-20B** | `@cf/openai/gpt-oss-20b` | 快速响应，低延迟 | 实时对话、快速问答、简单任务 | 较低 |
| **Llama-4-Scout** | `@cf/meta/llama-4-scout-17b-16e-instruct` | 多模态支持 | 长文档分析、图像理解 | 中等 |
| **Qwen2.5-Coder** | `@cf/qwen/qwen2.5-coder-32b-instruct` | 代码专家模型 | 编程、代码生成、技术问题 | 中等 |
| **Gemma-3** | `@cf/google/gemma-3-12b-it` | 多语言支持 | 翻译、多语言对话、文化理解 | 中等 |

## 📱 使用指南

### 基本使用流程

1. **访问应用**
   - 在浏览器中打开你的 Worker URL
   - 或使用配置的自定义域名

2. **身份验证**
   - 输入你在环境变量中设置的 `CHAT_PASSWORD`
   - 点击"验证"按钮

3. **选择 AI 模型**
   - 在左侧边栏选择适合的 AI 模型
   - 查看模型的特点和定价信息

4. **开始聊天**
   - 在底部输入框中输入你的问题
   - 按 Enter 键或点击"发送"按钮
   - AI 会用中文回复你的问题

### 代码功能特性

#### 自动代码识别
- 系统自动识别 Python、JavaScript、HTML、CSS、SQL 等多种编程语言
- 即使 AI 没有用 ``` 标记，系统也会自动识别并格式化代码

#### 语法高亮
- 支持多种编程语言的语法高亮显示
- 自动检测代码语言类型
- 清晰的代码块展示

#### 一键复制功能
- 每个代码块右上角都有"复制"按钮
- 点击后完整复制代码，保持原始格式
- 支持缩进、换行的完整保持
- 复制的代码可以直接在编辑器中使用

### 历史记录功能

- **自动保存**：每个模型的对话会自动保存到 KV 存储
- **模型隔离**：不同模型的历史记录分别保存
- **快速切换**：切换模型时自动加载对应的历史记录
- **手动管理**：可以手动清空当前模型的聊天记录

## 🛠️ 故障排除

### 常见问题及解决方案

#### 1. 403 错误 - 密码验证失败
**现象**: 输入密码后提示错误
**解决方案**:
- 检查环境变量 `CHAT_PASSWORD` 是否正确设置
- 确认密码输入时没有多余空格
- 重新保存环境变量并部署

#### 2. AI 模型调用失败
**现象**: 选择模型后无响应或报错
**解决方案**:
- 检查 Cloudflare 账号是否有 Workers AI 权限
- 确认账号余额充足
- 检查网络连接是否正常

#### 3. 历史记录不保存
**现象**: 刷新页面后聊天记录丢失
**解决方案**:
- 检查 KV 命名空间是否正确创建
- 确认 KV 绑定名称为 `CHAT_HISTORY`
- 检查 KV namespace ID 是否正确

#### 4. 代码复制功能不工作
**现象**: 点击复制按钮无反应
**解决方案**:
- 确认网站运行在 HTTPS 协议下
- 检查浏览器是否支持剪贴板 API
- 尝试手动选择代码进行复制

#### 5. 页面加载缓慢
**现象**: 打开应用时加载时间过长
**解决方案**:
- 检查网络连接
- 清除浏览器缓存
- 尝试使用 CDN 或自定义域名

### 调试方法

#### 查看浏览器控制台
1. 按 F12 打开开发者工具
2. 切换到 "Console" 标签
3. 查看错误信息和调试日志
4. 尝试复制代码时会显示详细的调试信息

#### 查看 Worker 日志
1. 在 Cloudflare Dashboard 中进入你的 Worker
2. 点击 "Logs" 标签
3. 查看实时日志输出
4. 分析错误信息和调用详情

## 🔒 安全和成本控制

### 安全最佳实践

#### 密码安全
- ✅ 使用至少12位字符的强密码
- ✅ 包含大小写字母、数字和特殊字符
- ✅ 避免使用常见密码或个人信息
- ✅ 定期更换访问密码
- ✅ 不要在代码中硬编码密码

#### 访问控制
- ✅ 考虑使用自定义域名限制访问
- ✅ 配置 Cloudflare WAF 规则
- ✅ 监控异常访问模式
- ✅ 定期检查访问日志

### 成本控制

#### Cloudflare Workers 费用
- **免费额度**: 每天 100,000 次请求
- **付费计划**: 超出后 $0.50/百万请求
- **建议**: 设置请求限制和预算警告

#### AI 模型调用费用
- **计费方式**: 按 token 使用量计费
- **价格范围**: $0.20-$4.88 每百万 tokens
- **优化建议**:
  - 选择合适的模型（简单任务用小模型）
  - 控制对话长度
  - 避免重复无效请求

#### KV 存储费用
- **免费额度**: 每月 100,000 次读取，1,000 次写入
- **付费价格**: $0.50/百万读取，$5.00/百万写入
- **优化**: 合理设置历史记录保留数量

## ⚙️ 高级配置

### 模型参数优化

不同模型支持不同的参数配置，应用已内置最优参数：

- **DeepSeek-R1**: 优化思维链推理参数
- **GPT 系列**: 简化参数避免异步响应
- **Llama-4-Scout**: 多模态优化配置
- **Qwen-Coder**: 代码生成专用参数
- **Gemma-3**: 多语言优化设置

### 自定义功能

如需自定义功能，可以修改 `src/worker.js`：

1. **添加新模型**: 在 `MODEL_CONFIG` 中添加配置
2. **修改界面**: 调整 HTML 和 CSS 样式
3. **扩展功能**: 添加新的 API 端点
4. **优化性能**: 调整缓存策略

## 📊 监控和维护

### 性能监控
- 使用 Cloudflare Analytics 监控访问量
- 检查 Worker 响应时间
- 监控 AI 调用成功率
- 跟踪错误频率

### 定期维护
- 更新 Cloudflare Worker 运行时
- 检查和更新模型配置
- 清理过期的 KV 数据
- 更新安全配置

## 🔄 更新日志

### 最新版本功能
- ✅ 支持 GPT-OSS 120B/20B 模型
- ✅ 完善的代码自动识别系统
- ✅ 可靠的代码复制功能
- ✅ 响应式用户界面设计
- ✅ 按模型分类的历史记录
- ✅ 详细的错误处理和用户反馈

### 计划功能
- 🔄 支持文件上传功能
- 🔄 对话导出和分享
- 🔄 主题切换功能
- 🔄 流式响应支持
- 🔄 更多 AI 模型支持

## 📞 技术支持

### 获取帮助
- 检查本文档的故障排除部分
- 查看 Cloudflare Workers 官方文档
- 访问 Cloudflare 社区论坛

### 反馈问题
- 详细描述问题现象
- 提供错误日志信息
- 说明部署环境和配置

---

**重要提醒**: 
1. 使用本应用产生的 AI 调用费用由用户承担，请合理使用以控制成本
2. 定期检查和更新访问密码以确保安全
3. 建议在生产环境中使用自定义域名和 HTTPS
4. 遵守相关法律法规和 Cloudflare 服务条款

**许可证**: 本项目采用 MIT 许可证，允许自由使用、修改和分发。

