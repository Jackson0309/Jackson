# 多哥门户、文字助手、AI 坐席与观察台

本包是 2026-11-18 对当前 `http://localhost:4175/` 的应用源码快照，包含页面注入、弹窗资源、业务服务和观察台已构建资源。

## 内容

- `togo-govbot-demo/`：多哥门户代理、三入口按钮、文字助手及其本地办件后台。
- `tangrat-voice/`：保留原 Tang Rat 交互与语音时序的多哥 AI 坐席、观察台、六项业务配置、材料清单和模拟上传。
- `provider-settings.json`：当前非敏感供应商、模型、声音等设置的快照；容器内同名文件在首次启动时初始化配置。
- `.env.example`：需要重新填写的 API 凭据。

门户首页和部分资源由多哥网站 `https://new.service-public.gouv.tg` 动态加载，本包不是该网站的离线镜像。需要网络访问该上游、AI 供应商，以及首次加载语音模型时的模型下载服务。办理、材料和回执均为演示，不向政府系统申报。

## Docker Compose 启动

需要 Docker Desktop 或 Docker Engine + Compose。首次构建会下载 Python、Node、语音依赖及模型，可能较慢。

```sh
cp .env.example .env
# 编辑 .env，填写自己的 DEEPSEEK_API_KEY、ELEVENLABS_API_KEY 等
# 已在使用本机 4175 / 8080 的服务时，先停止旧服务，或调整 compose.yaml 的主机端口
docker compose up -d --build
```

访问门户 `http://localhost:4175/`。三个入口依次为文字助手、AI 坐席、观察台，默认英语。

- 文字助手：`http://localhost:4175/govbot/demo/assistant.html?ui_lang=en`
- AI 坐席：`http://localhost:4175/tangrat/?ui_lang=en`
- 观察台：`http://localhost:4175/tangrat/observer?ui_lang=en`
- Compose 通过 `.env` 配置密钥。`/operator` 仅允许原生本机回环请求；Docker 端口转发请求可能被拒绝，不放宽此访问限制。

主业务 LLM 设置为 `deepseek-v4-pro`，实时 ASR / TTS 保留 ElevenLabs。分段 ASR 仍使用 OpenRouter，需要相应密钥。讯飞配置为备用，能否使用取决于账户授权。模型名和能力以各供应商可用范围为准。

`.env` 由 Compose 传入环境变量；原 Python 服务直接启动时不会自动加载 `.env`。环境变量优先于操作配置页。未配置真实凭据时不能完成真实 AI 语音调用。

```sh
docker compose logs -f
docker compose down
```

办件数据在命名卷中；`docker compose down` 保留数据。请勿把运行数据、真实附件或操作配置上传 GitHub。

## iPhone 与其他手机

手机通过局域网 HTTP 地址无法获得麦克风权限。部署时需要给门户配置设备信任的 HTTPS 证书，并在反向代理中支持 WebSocket 和 SSE。默认 Compose 仅向本机开放端口；手机接入需由部署者配置 HTTPS 入口。旧电脑的 CA、配置描述文件和所有证书私钥均未打包，不能直接复用旧的证书安装地址。

`server/https_gateway.py` 保留了原本机网关实现，但其中证书路径及 8443 监听只供自建本地证书的开发环境参考，不由本次 Compose 自动启用。

## 源码与构建

观察台源码位于 `tangrat-voice/observer-ui`；已附当前构建结果，无需在启动前重新编译。修改后可执行：

```sh
cd tangrat-voice/observer-ui
npm ci
npm run build
```

原项目参考： https://github.com/gov-services-fde/tangrat-voice 。业务适配说明见 `tangrat-voice/docs/togo-adaptation.md`。历史文档中的本机地址、凭据检查结果或旧启动方式只代表当时环境，本 README 为打包启动入口。

未包含：密钥、会话、草稿、用户上传、数据库、虚拟环境、node_modules、Git 历史、备份、截图报告和鸿蒙应用工程。本包聚焦当前 Web 门户与相关弹窗。
