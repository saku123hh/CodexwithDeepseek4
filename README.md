<img width="1311" height="255" alt="Pasted image 20260611204819" src="https://github.com/user-attachments/assets/553176cd-102f-438d-8172-bc31545c82ff" />
# CodexwithDeepseek4
Integrate DeepSeek AI into OpenAI Codex using Moonbridge as a compatibility layer
**# 重新理清moonbridge和codex直接连接

## 参考文献
https://github.com/deepseek-ai/awesome-deepseek-agent/blob/main/docs/codex.zh-CN.md
https://cloud.tencent.com/developer/article/2671457


moonbridge需要配置deepseek相关的，还有就是监听端口 http://127.0.0.1:38440
codex发消息给端口 moonbridge要监听并让deepseek干活  最后codex接受干活的响应：
 http://127.0.0.1:38440/v1/response
## 配置moonbridge
主要是新建config.yml(yaml)
Server: http://127.0.0.34480
config.yaml:
```python
mode: "Transform"

server:
  addr: "127.0.0.1:38440"

models:
  deepseek-v4-pro:
    context_window: 1000000
    max_output_tokens: 384000
    default_reasoning_level: "high"
    supported_reasoning_levels:
      - effort: "high"
        description: "High reasoning effort"
      - effort: "xhigh"
        description: "Extra high reasoning effort"
    supports_reasoning_summaries: true
    default_reasoning_summary: "auto"
    extensions:
      deepseek_v4:
        enabled: true
  deepseek-v4-flash:
    context_window: 1000000
    max_output_tokens: 384000
    default_reasoning_level: "high"
    supported_reasoning_levels:
      - effort: "high"
        description: "High reasoning effort"
      - effort: "xhigh"
        description: "Extra high reasoning effort"
    supports_reasoning_summaries: true
    default_reasoning_summary: "auto"
    extensions:
      deepseek_v4:
        enabled: true

providers:
  deepseek:
    base_url: "https://api.deepseek.com/anthropic"
    api_key: "sk-your-deepseek-api-key"
    offers:
      - model: deepseek-v4-pro
      - model: deepseek-v4-flash

routes:
  moonbridge:
    model: deepseek-v4-pro
    provider: deepseek

defaults:
  model: moonbridge
  max_tokens: 65536
```
然后moonbridge就开一个监听窗口，新建powershell
```
cd moon-bridge
go run ./cmd/moonbridge --config config.yaml
```

产生
```
Moon Bridge 监听于 127.0.0.1:38440
time=2026-06-11T20:21:14.015+08:00 level=INFO msg="HTTP 服务器监听中" addr=127.0.0.1:38440
```

## codex配置
最重要的是要配置codex的环境变量，要设置为目标的./codex文件夹
```
$CODEX_HOME_DIR = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { "$HOME\.codex" }
New-Item -ItemType Directory -Force -Path $CODEX_HOME_DIR | Out-Null
```
`-Force`是强制执行存在$CODEX_HOME_DIR文件夹 `| Out-Null` 输出不打印到终端
如果$CODEX_HOME_DIR存在就不新建文件夹，不存在就新建
备份现在./codex文件夹的config.toml配置
```
# 备份当前config.toml
if (Test-Path "$CODEX_HOME_DIR\config.toml") {
  Copy-Item "$CODEX_HOME_DIR\config.toml" "$CODEX_HOME_DIR\config.toml.bak" -Force
}
```
创建config.toml和models_catalog.json
这个会打印出来codex的model是moonbridge
```
$MODEL = go run ./cmd/moonbridge --config config.yml --print-codex-model
```
```
go run ./cmd/moonbridge --config config.yaml --print-codex-config "$MODEL" --codex-base-url "http://127.0.0.1:38440/v1" --codex-home "$CODEX_HOME_DIR" | Set-Content -Path "$CODEX_HOME_DIR\config.toml"
```
models_catalog.json的路径被保存为"$CODEX_HOME\models_catalog.json"
在config.toml里面为：

```
model_catalog_json = "C:\\Users\\Administrator\\.codex\\models_catalog.json"
```
<img width="1292" height="555" alt="Pasted image 20260611204546" src="https://github.com/user-attachments/assets/d1ff1897-e69f-4f1a-9c52-3605ab8085d3" />
<img width="1311" height="255" alt="Pasted image 20260611204819" src="https://github.com/user-attachments/assets/b06f9e92-97ad-4b1e-a6b2-b476526dc469" />


然后启动codex就可以了

codex cd myProject
```**
