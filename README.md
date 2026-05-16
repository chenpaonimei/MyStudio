# Ling's GPT-Image-2 Studio

> 一个**纯前端、单 HTML 文件**的 GPT-Image-2 / GPT-Image-2-Official 图像生成工作台。
> 数据全部存于浏览器本地，零后端，零编译，双击即可使用。

---

## 这是什么

基于 [APIMart](https://apimart.ai) 网关调用 OpenAI **GPT-Image-2** 模型的网页工具，集成图像生成 + Mask 编辑器 + Prompt 库三大功能。

主要特性：

- **🚀 图像生成** — 13 种比例 × 1K/2K/4K 分辨率，最多 16 张参考图，任务队列 / 自动重试 / 批量轮询 / URL 倒计时
- **🎨 Mask 编辑器** — 涂抹/擦除/缩放/快捷键，AI 智能选区（人物/背景/头部/双手/双脚），一键写入主页面
- **📚 Prompt 库** — 双数据源汇集 200+ 条爆款 prompt（ApiMartAI + EvoLinkAI），按分类/语言/热度筛选，一键填入
- **💰 余额监控** — 顶部实时显示余额，提交前预估成本，价格表自动从 `cost` 字段学习
- **🔒 隐私** — API Key 仅保存浏览器 localStorage，图像数据不上传任何第三方服务器

---

## 三种使用方式

### 方式 A：本地双击使用（最简单，适合个人）

1. 下载 [`gpt-image-2-studio.html`](./gpt-image-2-studio.html)
2. 双击在浏览器打开
3. 点右上角「⚙️ 设置」填入 API Key + Base URL
4. 开始用

✅ 完全离线（除了调 API 的网络请求）
✅ 单文件无依赖（CDN 资源浏览器自动加载）
❌ 不能给别人远程访问

### 方式 B：访问 GitHub Pages 部署版本（朋友共享）

1. 浏览器打开 `https://chenpaonimei.github.io/My-GPT-Image-2-Studio/gpt-image-2-studio.html`
2. 同样填入自己的 API Key 即可使用

✅ 不用下载
✅ 朋友分享链接就能用
⚠️ 每个用户需要自己的 API Key（工具不收集任何账号信息）

### 方式 C：自部署到自己的服务器

把 `gpt-image-2-studio.html` 上传到任意 HTTPS 服务器目录即可。零特殊依赖。

---

## 申请 API Key

需要一个 OpenAI 兼容 API Key（gpt-image-2 模型）。常见获取方式：

- **APIMart**（推荐，默认配置）：[apimart.ai](https://apimart.ai) 注册账号，充值后在控制台拿 Key，默认 Base URL `https://api.apimart.ai/v1`
- **OpenAI 官方**：[platform.openai.com](https://platform.openai.com) 申请，但需要 OpenAI 开放 gpt-image-2 访问权限
- **其他兼容网关**：把 Base URL 改成对应的 endpoint 即可

---

## 安全说明

### 你 API Key 的去向

- **Key 保存在你的浏览器 localStorage**（明文），不会上传给我或任何第三方
- 每次生成请求，浏览器**直接**把 Key 发到你填的 Base URL（默认 `apimart.ai`）
- 我看不到你的 Key、prompt、生成结果
- **但如果你打开恶意网站 / 装恶意浏览器扩展 / 共用电脑，Key 可能被人偷**

### 这个工具做了什么安全加固

- ✅ **CSP（Content Security Policy）**：限制脚本只能从白名单 CDN 加载，即使页面被 XSS 注入，恶意 JS 也无法 fetch 到陌生服务器
- ✅ **`<meta name="robots" content="noindex">`**：搜索引擎不会收录，陌生人很难偶然找到
- ✅ **Base URL HTTPS 强制校验**：仅允许 `https://` 或 `http://localhost`（防 HTTP 明文泄漏 Key）
- ✅ **「清除所有本地数据」按钮**：公用电脑用完一键清干净
- ✅ **API Key 输入框红框警告**：醒目提示风险
- ✅ **noindex 默认开启**：URL 不会被搜索引擎收录

### 还可以更安全的选项（进阶）

如果你担心 CDN 被劫持（jsdelivr / cdnjs / tailwindcss 任一被 hack），可以加 **SRI 校验**：

```html
<!-- 把这一行（在 HTML 第 9 行附近） -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>

<!-- 改成（hash 去 https://www.srihash.org/ 或 cdnjs.com 该版本页查） -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"
        integrity="sha512-XXXXXX..."
        crossorigin="anonymous"></script>
```

CDN 内容被改过 → 浏览器拒绝执行。

更彻底：把 JSZip / Tailwind / MediaPipe 全部下载到本地 `vendor/` 目录，HTML 改成相对路径引用 —— 完全去除 CDN 信任。

### 不要做的事

- ❌ 不要把这个工具部署成「公开给陌生人用」的 SaaS。当前架构下，陌生用户的 API Key 仍然在他们自己的浏览器，但每个人都依赖 3 个 CDN 没被劫持
- ❌ 不要在公用电脑保存 Key 不清就走
- ❌ 不要修改 Base URL 到不信任的网关

---

## 数据来源说明

Prompt 库的数据来自两个公开 GitHub 仓库（CDN 直连，不经过任何中间服务器）：

- [`ApiMartAI/best-gpt-image-2-prompts`](https://github.com/ApiMartAI/best-gpt-image-2-prompts) — 107 条爆款 Twitter prompt
- [`EvoLinkAI/awesome-gpt-image-2-API-and-Prompts`](https://github.com/EvoLinkAI/awesome-gpt-image-2-API-and-Prompts) — 100+ 条按分类组织的 prompt

数据通过 jsdelivr CDN 拉取，本地缓存 7 天。可在「Prompt 库」面板点「↻ 刷新」强制重拉，或「清缓存」彻底清空。

---

## 技术栈

- **HTML + Tailwind CSS** (via CDN，生产环境推荐换成 PostCSS 编译版)
- **原生 JS ES2020**（async/await/optional chaining/template literals）
- **JSZip** 3.10.1（批量打包下载）
- **MediaPipe Tasks Vision** 0.10.14（AI 智能选区，纯前端跑 selfie/pose/hand/face 检测）
- **存储** localStorage（API Key / 配置 / 价格表 / Prompt 库缓存 / 收藏）

---

## 文件清单

| 文件 | 用途 |
|---|---|
| `gpt-image-2-studio.html` | **主程序**，双击或部署都用这一个 |
| `CHANGELOG.md` | 完整更新日志（v0.1 → v0.14+） |
| `README.md` | 本文件 |
| `LICENSE` | MIT 协议 |
| `image2-generate-apimart-v5.md` | API 契约文档（给其他 LLM 读了即可调用） |
| `MASK_AI_ASSESSMENT.md` | 智能化路线评估 |
| `PROMPT_LIBRARY_PLAN.md` | Prompt 库集成方案 |
| `PUBLISH_AND_SECURITY.md` | 发布与安全分析 |
| `DEPLOY_GUIDE.md` | 部署到 GitHub Pages 的图文教程 |

---

## 贡献 / Issues

- 发现 bug 或想加功能 → [提 Issue](https://github.com/chenpaonimei/My-GPT-Image-2-Studio/issues)
- 想自己改的 → fork / PR
- 看不懂 / 需要帮助 → Issue 里 @ 我

---

## 致谢

- 数据源：[ApiMartAI](https://github.com/ApiMartAI) / [EvoLinkAI](https://github.com/EvoLinkAI)
- API 网关：[APIMart](https://apimart.ai)
- AI 选区：[MediaPipe](https://developers.google.com/mediapipe)

---

## License

MIT License — 自由使用、修改、分发，不承担任何责任。详见 [LICENSE](./LICENSE)。
