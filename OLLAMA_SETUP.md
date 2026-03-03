# OpenClaw 本地 Ollama 配置指南

## 已完成配置

我已经成功配置了 OpenClaw 以使用本地部署的 Ollama 模型。以下是配置详情：

### 1. 配置文件位置

- 主配置文件: `~/.openclaw/openclaw.json`
- 环境变量文件: `~/.openclaw/.env`

### 2. 配置内容

#### openclaw.json

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "ollama/qwen3:8b"
      }
    }
  },
  "models": {
    "mode": "merge",
    "providers": {
      "ollama": {
        "baseUrl": "http://127.0.0.1:11434",
        "api": "ollama",
        "apiKey": "ollama-local",
        "models": [
          {
            "id": "qwen3:8b",
            "name": "Qwen3 8B",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 128000,
            "maxTokens": 8192
          },
          {
            "id": "deepseek-r1:1.5b",
            "name": "DeepSeek R1 1.5B",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 128000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "channels": {
    "webchat": {
      "enabled": true
    }
  }
}
```

#### .env 文件

```bash
# OpenClaw .env configuration for local Ollama

# Gateway authentication (recommended if the gateway binds beyond loopback)
OPENCLAW_GATEWAY_TOKEN=change-me-to-a-long-random-token

# Ollama configuration
OLLAMA_API_KEY="ollama-local"

# Optional: If you want to use other model providers alongside Ollama
# OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...
# GEMINI_API_KEY=...

# Optional: If you want to use webchat interface
OPENCLAW_WEBCHAT_ENABLED=true
```

### 3. 支持的模型

根据您的 Ollama 实例，当前检测到以下模型：

1. **qwen3:8b** - Qwen3 8B 模型 (8.2B 参数)
2. **deepseek-r1:1.5b** - DeepSeek R1 1.5B 模型 (1.8B 参数，支持推理)

### 4. 如何使用

#### 启动 OpenClaw

```bash
cd /Users/zwj/openclaw
./openclaw.mjs start
```

#### 使用 webchat 界面

1. 启动 OpenClaw
2. 访问 webchat 界面（默认端口）
3. 开始与配置的 Ollama 模型对话

#### 切换模型

要切换模型，修改 `openclaw.json` 文件中的 `agents.defaults.model.primary` 字段：

```json
"primary": "ollama/deepseek-r1:1.5b"
```

### 5. 添加更多 Ollama 模型

如果您在 Ollama 中安装了更多模型，OpenClaw 会自动发现它们。您也可以手动添加到配置中：

1. 首先检查 Ollama 中的可用模型：

```bash
ollama list
```

2. 将新模型添加到 `openclaw.json` 的 `models.providers.ollama.models` 数组中：

```json
{
  "id": "模型名称",
  "name": "显示名称",
  "reasoning": false,  # 如果模型支持推理则为 true
  "input": ["text"],   # 支持的输入类型
  "cost": {
    "input": 0,
    "output": 0,
    "cacheRead": 0,
    "cacheWrite": 0
  },
  "contextWindow": 128000,  # 上下文窗口大小
  "maxTokens": 8192         # 最大输出令牌数
}
```

### 6. 故障排除

#### Ollama 服务未运行

```bash
# 启动 Ollama 服务
ollama serve

# 在另一个终端中拉取模型
ollama pull llama3.2
```

#### 配置验证

运行测试脚本验证配置：

```bash
cd /Users/zwj/openclaw
node test-ollama-simple.js
```

#### 查看日志

```bash
# 查看 OpenClaw 日志
tail -f ~/.openclaw/logs/openclaw.log
```

### 7. 高级配置

#### 使用多个模型提供者

您可以在配置中添加其他模型提供者（如 OpenAI、Anthropic 等）：

```json
"models": {
  "mode": "merge",
  "providers": {
    "ollama": { ... },
    "openai": {
      "apiKey": "${OPENAI_API_KEY}",
      "models": [...]
    }
  }
}
```

#### 配置回退模型

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "ollama/qwen3:8b",
      "fallbacks": ["ollama/deepseek-r1:1.5b", "openai/gpt-4o"]
    }
  }
}
```

### 8. 安全注意事项

1. **网关令牌**：在生产环境中，请将 `OPENCLAW_GATEWAY_TOKEN` 更改为安全的随机字符串
2. **网络访问**：默认配置仅绑定到本地回环地址，如需外部访问请谨慎配置
3. **模型权限**：确保只有授权用户可以访问您的 Ollama 实例

### 9. 性能优化建议

1. **上下文窗口**：根据模型能力调整 `contextWindow` 值
2. **最大令牌数**：根据使用场景调整 `maxTokens` 值
3. **批处理**：对于大量请求，考虑调整批处理设置
4. **缓存**：启用模型响应缓存以提高性能

### 10. 扩展功能

OpenClaw 支持通过以下方式扩展：

- **技能（Skills）**：添加自定义功能模块
- **插件（Plugins）**：集成第三方服务
- **通道（Channels）**：添加更多消息平台支持（如 Telegram、Discord 等）

---

**配置验证状态**: ✅ 成功
**Ollama 连接**: ✅ 正常
**模型可用性**: ✅ 2 个模型已配置
**主模型**: ollama/qwen3:8b
