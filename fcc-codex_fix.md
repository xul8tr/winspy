Free Claude Code × OpenAI Codex (fcc-codex) 魔改与完美闭环备忘录

## 📋 一、 核心架构全景
本方案实现了 OpenAI Codex 官方终端客户端 (v0.140.0) 与本地魔改代理 free-claude-code (fcc-server) 的无缝对接。

* 客户端层：OpenAI Codex 以为自己在调用最顶尖的 gpt-5.5 (medium)，并完美继承原版客户端的本地工作区读写、脚本执行等高阶权限。
* 代理中转层：本地 127.0.0.1:8082 拦截 Codex 的 responses 私有协议流，将其翻译为标准的大模型对话协议。
* 算力提供层（真身）：流量通过 NVIDIA NIM 平台无缝路由至满血版的 DeepSeek V4 Pro（代码与中文梗核心担当）以及 Nemotron-3 Ultra (550B) 等顶级大模型，实现零成本/低成本的高效开发。

---

## 🛠️ 二、 终端环境配置 (Zsh)

通过配置 ~/.zshrc 别名，免去每次手动敲击冗长参数的烦恼。根据对“终端画面清爽度”的要求，提供以下两种配置方案（二选一即可）：

### 方案 A：极致纯净版（推荐 🌟）
> 优点：完美隐藏所有模型元数据不匹配的警告，终端输出最清爽。
> 代价：在终端内部输入 /model 时无法可视化浏览完整的模型池，但完全不影响盲敲命令切换。

打开 ~/.zshrc 并写入以下别名：
```bash
# OpenAI Codex 纯净版免警告别名
alias fcc-codex='codex -c "model_provider=\"fcc\"" -c "model_providers.fcc.name=\"Free Claude Code\"" -c "model_providers.fcc.base_url=\"[http://127.0.0.1:8082/v1](http://127.0.0.1:8082/v1)\"" -c "model_providers.fcc.env_key=\"FCC_CODEX_API_KEY\"" -c "model_providers.fcc.wire_api=\"responses\""'

### 方案 B：完整模型池版
优点：在交互界面输入 /model 时，能看到密密麻麻的可用模型列表。
代价：因为本地 JSON 缺少特定描述字段，每次启动时会弹出一行无害的黄字警告 warning: Model metadata for gpt-5.5 not found...。

打开 ~/.zshrc 并写入以下别名：

Bash
# OpenAI Codex 完整模型池版别名（包含本地 Catalog 路径）
alias fcc-codex='codex -c "model_provider=\"fcc\"" -c "model_providers.fcc.name=\"Free Claude Code\"" -c "model_providers.fcc.base_url=\"[http://127.0.0.1:8082/v1](http://127.0.0.1:8082/v1)\"" -c "model_providers.fcc.env_key=\"FCC_CODEX_API_KEY\"" -c "model_providers.fcc.wire_api=\"responses\"" -c "model_catalog_json=\"/Users/guwj/.fcc/codex-model-catalog.json\""'

🚀 三、 核心生效与验证命令

每当你修改了别名配置文件，或者需要日常使用时，请按顺序执行以下命令：

使配置文件立即生效（极其重要，改完必跑）：

Bash
source ~/.zshrc
单次命令执行模式（快速验证/单次任务）：

Bash
fcc-codex exec "hello"
常驻交互式 TUI 模式（沉浸式对话、写代码）：

Bash
fcc-codex

🎛️ 四、 幕后真实模型路由与切换指南

虽然客户端面板显示为 gpt-5.5，但流量的生杀大权完全由 free-claude-code 后台的 Model Routing 控制：

1. 默认的阶梯映射规则
Sonnet Override -> nvidia_nim/deepseek-ai/deepseek-v4-pro （当前实际主力模型）

Default / Opus Override -> nvidia_nim/nvidia/nemotron-3-ultra-550b-a55b

Haiku Override -> nvidia_nim/minimaxai/minimax-m3

2. 运行时动态切换模型的 3 种姿势
姿势一：在交互界面（TUI）内部秒换
在 fcc-codex 对话框中，直接输入指令并回车：

Plaintext
/model sonnet   # 切换至 DeepSeek V4 Pro 满血版
/model haiku    # 切换至 MiniMax 轻量版速度之王
/model opus     # 切换至英伟达 550B 巨无霸
姿势二：在 CLI 命令中临时换挡

Bash
fcc-codex exec "帮我评审这段架构" --model opus
姿势三：Web 后台一劳永逸修改
访问本地 Web 后台：http://127.0.0.1:8082/admin，在 Model Routing 页面中直接修改对应标签下的真实模型字符串（如换成全新的 llama-3.3-70b），点击 Save 即可。终端完全不需要重启或修改。

⚠️ 五、 常见故障排查（避坑指南）

问题：为什么我改了别名，终端里还是报错或者弹出警告？

原因：忘记刷新当前终端会话。

解决：立刻执行 source ~/.zshrc，或者直接关掉当前终端标签页，新建一个窗口重新执行。

问题：提示网络连接拒绝或无法连接到 127.0.0.1:8082？

原因：幕后的 free-claude-code 核心服务没有启动。

解决：检查你的常驻进程（如 tmux 窗口或 launchctl 服务），确保 fcc-server 正在后台健康运行。

祝：代码纵横，Coding 愉快！ 🚀🥂
