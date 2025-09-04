# 🔧 API参数修复说明

## 🚨 发现的关键问题

### 1. **GPT模型参数错误**
- **问题**: 使用了 `instructions` + `input` 参数
- **正确**: 应该使用 `input` 参数（可以是字符串或数组）
- **修复**: 新增 `use_input: true` 标识，使用单一 `input` 参数

### 2. **模型分类错误**
- **问题**: 所有模型都用 `use_messages` 或 `instructions` 二分法
- **正确**: 需要三种模式：`messages`、`input`、`prompt`
- **修复**: 更新模型配置，正确分类每个模型

### 3. **参数范围和名称错误**
- **问题**: 使用了错误的参数名称和超出范围的值
- **正确**: 严格按照官方API文档的参数范围
- **修复**: 更新所有参数到正确范围

## ✅ 修复详情

### GPT模型 (gpt-oss-120b, gpt-oss-20b)
```javascript
// 之前错误的方式
{
  instructions: "系统提示词",
  input: "用户输入",
  temperature: 0.7,
  top_p: 0.95,           // ❌ GPT不支持
  presence_penalty: 0.1  // ❌ GPT不支持
}

// 现在正确的方式
{
  input: "完整的上下文 + 用户问题",  // ✅ 单一input参数
  temperature: 0.7,
  reasoning: {           // ✅ GPT特有的reasoning参数
    effort: "medium",    // low/medium/high
    summary: "auto"      // auto/concise/detailed
  }
}
```

### DeepSeek模型 (deepseek-r1-distill-qwen-32b)
```javascript
// 支持messages格式，参数范围严格按官方文档
{
  messages: [...],
  max_tokens: 8192,
  temperature: 0.8,        // 范围: 0-5
  top_p: 0.9,              // 范围: 0.001-1
  top_k: 50,               // 范围: 1-50
  repetition_penalty: 1.1, // 范围: 0-2
  frequency_penalty: 0.1,  // 范围: -2到2
  presence_penalty: 0.1    // 范围: -2到2
}
```

### Llama模型 (llama-4-scout-17b-16e-instruct)
```javascript
// 支持messages格式，使用正确的参数名
{
  messages: [...],
  max_tokens: 4096,
  temperature: 0.75,        // 范围: 0-5
  top_p: 0.95,              // 范围: 0-2
  top_k: 40,                // 范围: 1-50
  repetition_penalty: 1.1,  // ✅ 正确参数名(不是repeat_penalty)
  frequency_penalty: 0.1,   // 范围: 0-2
  presence_penalty: 0.1     // 范围: 0-2
}
```

### Qwen模型 (qwen2.5-coder-32b-instruct)
```javascript
// 支持messages格式，代码优化参数
{
  messages: [...],
  max_tokens: 8192,
  temperature: 0.3,         // 代码生成需要低随机性
  top_p: 0.8,               // 范围: 0-2
  top_k: 30,                // 范围: 1-50
  repetition_penalty: 1.1,  // 范围: 0-2
  frequency_penalty: 0.1,   // 范围: 0-2
  presence_penalty: 0.1     // 范围: 0-2
}
```

### Gemma模型 (gemma-3-12b-it)
```javascript
// 支持messages格式，多语言优化
{
  messages: [...],
  max_tokens: 4096,
  temperature: 0.8,         // 范围: 0-5
  top_p: 0.9,               // 范围: 0-2
  top_k: 40,                // 范围: 1-50
  repetition_penalty: 1.0,  // 范围: 0-2
  frequency_penalty: 0.1,   // 范围: 0-2
  presence_penalty: 0.1     // 范围: 0-2
}
```

## 🔄 调用逻辑修复

### 新的三分支逻辑
```javascript
if (selectedModel.use_input) {
  // GPT模型: 使用input参数
  const inputParams = {
    input: "完整上下文",
    ...optimalParams
  };
  response = await env.AI.run(selectedModel.id, inputParams);
  
} else if (selectedModel.use_messages) {
  // DeepSeek/Llama/Qwen/Gemma: 使用messages参数
  const messagesParams = {
    messages: [...],
    ...optimalParams
  };
  response = await env.AI.run(selectedModel.id, messagesParams);
  
} else {
  // 未知模型类型
  throw new Error(`未知的模型类型: ${selectedModel.name}`);
}
```

## 📊 修复前后对比

| 模型 | 修复前 | 修复后 | 主要变化 |
|------|--------|--------|----------|
| **GPT-OSS-120B** | ❌ instructions+input | ✅ input only | 使用单一input参数 + reasoning |
| **GPT-OSS-20B** | ❌ instructions+input | ✅ input only | 使用单一input参数 + reasoning |
| **DeepSeek-R1** | ✅ messages | ✅ messages | 修正参数范围 |
| **Llama Scout** | ✅ messages | ✅ messages | repeat_penalty → repetition_penalty |
| **Qwen Coder** | ✅ messages | ✅ messages | 添加缺失的penalty参数 |
| **Gemma 3** | ✅ messages | ✅ messages | 修正参数范围 |

## 🎯 预期效果

### 1. **GPT模型不再报错**
- 使用正确的 `input` 参数格式
- 移除不支持的 `top_p`、`presence_penalty` 参数
- 添加 GPT 特有的 `reasoning` 配置

### 2. **所有模型参数在官方范围内**
- `temperature`: 严格按每个模型的范围设置
- `top_p`、`top_k`: 在允许范围内
- `penalty` 参数: 使用正确名称和范围

### 3. **更好的调试信息**
- 为每种调用模式显示不同的日志
- 清晰显示使用的参数类型

### 4. **更稳定的调用**
- 减少参数错误导致的失败
- 提高响应成功率

## 🚀 部署后测试

1. **GPT模型测试**: 确认不再报错，能正常回复
2. **DeepSeek测试**: 验证思维链推理输出完整
3. **其他模型测试**: 检查参数是否在正确范围内
4. **调试日志**: 查看控制台显示的参数信息

---
**修复完成时间**: 2025年1月  
**参考文档**: Cloudflare Workers AI 官方API文档
