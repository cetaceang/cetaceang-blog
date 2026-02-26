---
title: Linux 如何安装 Claude Code 和 Codex
published: 2026-02-26
description: 'Linux 下安装 Claude Code 与 Codex 的完整教程：环境准备、安装命令、升级与常见问题'
image: ''
tags: [Linux, Claude Code, Codex, 开发工具]
category: '技术'
draft: false
lang: 'zh-CN'
---

# Linux 安装 Claude Code 和 Codex

## 一、通用前置环境（Node.js）

```bash
# 添加 NodeSource 仓库
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

# 安装 Node.js
sudo apt-get install -y nodejs
```

验证 Node.js 与 npm：

```bash
node --version
npm --version
```

如果显示版本号，则说明安装成功。

## 二、Claude Code

开始前，请先完成上面的 Node.js 前置环境。

### 1. 安装 Claude Code

安装 Claude Code：

```bash
sudo npm install -g @anthropic-ai/claude-code@latest
```

验证 Claude Code 安装：

```bash
claude --version
```

如果显示版本号，则说明安装成功。

### 2. 配置 Claude Code 环境变量

```bash
echo 'export ANTHROPIC_BASE_URL="你的API URL"' >> ~/.bashrc
echo 'export ANTHROPIC_AUTH_TOKEN="你的API密钥"' >> ~/.bashrc
source ~/.bashrc
```

### 3. 启动 Claude Code

启动命令：

```bash
claude
```

## 三、Codex

开始前，请先完成上面的 Node.js 前置环境。

### 1. 安装 Codex

```bash
sudo npm install -g @openai/codex@latest
```

安装后验证：

```bash
codex --version
```

如果显示版本号，则说明安装成功。

### 2. 配置 Codex

在用户目录的 `.codex` 下创建 `auth.json`：

```json
{
  "OPENAI_API_KEY": "key"
}
```

然后在 `.codex` 目录下创建 `config.toml`：

```toml
model_provider = "veloera"
model = "gpt-5.3-codex"
model_reasoning_effort = "xhigh"
model_supports_reasoning_summaries = true
personality = "pragmatic"

[profiles.veloera]
model_provider = "veloera" # 与 model_providers 后缀对应
model = "gpt-5.3-codex"

[model_providers.veloera]
name = "veloera"
base_url = "你的api的URL"
env_key = "veloera"
wire_api = "responses"
requires_openai_auth = true

[mcp_servers.mcphub]
url = "https://mcphub的url/mcp"
http_headers = { "Authorization" = "Bearer your-api-key" }
```

配置环境变量：

```bash
echo 'export veloera="你的API密钥"' >> ~/.bashrc
source ~/.bashrc
```

### 3. 启动 Codex

配置完成后，在终端中输入以下命令启动 Codex：

```bash
codex
```

## 常见问题

### 如何更新 Claude Code 和 Codex

```bash
sudo npm install -g @anthropic-ai/claude-code@latest
sudo npm install -g @openai/codex@latest
```

重新安装即可。
