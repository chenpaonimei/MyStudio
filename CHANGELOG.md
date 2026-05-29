# Ling's AIGC Studio · 更新日志

> 项目：`gpt-image-2-studio.html`（图像 + 视频 双形态 AIGC 工作台，基于 APIMart 网关）
>
> 版本号方案：`主.次.补丁`（语义化），破坏性变更 +主；新功能 +次；修复或小补丁 +补丁。
>
> 关联接口文档：[image2-generate-apimart-v5.md](./image2-generate-apimart-v5.md) · [STUDIO_DEV.md](./STUDIO_DEV.md)（开发交接文档）

---

## v0.17.6 — 2026-05-29 · 视频链路补完：续写 / 真人认证 / 素材入库 / Worker 上传

本版把 v0.17.0-v0.17.5 已经出现在界面和更新日志里的视频链路补成真实行为，避免“按钮在、逻辑空”的半成品状态。

### 修复 / 补完

- **用尾帧续写**：完成任务返回 `last_frame_url` 时，「用尾帧续写」按钮会切到「图生（首帧）」模式，把尾帧写入首帧上传位，并保留原 prompt 作为续写起点。
- **真人认证步骤 1**：新增「开始认证」按钮，POST `/v1/seedance2/real-avatar` 创建会话；拿到 `task_id` 后轮询 `/v1/tasks/{id}`，提取 `h5_link` / `byted_token`，生成真实 QR 并保存 token，步骤 3 查询可直接复用。
- **素材库审核轮询**：虚拟人像批量上传并提交 `/v1/seedance2/private-avatar` 后自动轮询审核任务；解析 `usable_assets` / `assets` / `asset_urls` 等常见返回结构，写入 `localStorage('studio.seedance.assets.v1')` 并渲染素材卡片。
- **asset 选择 UI**：用素材库模式从 prompt 粘贴升级为下拉选择 + 卡片「选用」按钮；仍保留手动粘贴 `asset_id` / `asset://...` 兜底。
- **catbox file:// CORS 真修法**：Cloudflare Worker 新增 `/upload` 路由。视频/音频上传会优先走 `{Worker Base URL 去掉 /v1}/upload`，由 Worker 服务器侧转发到 catbox.moe；未部署 Worker 时继续走 Pixeldrain / catbox / URL paste 兜底。
- **上传结果硬校验 hotfix**：旧 Worker 未重部署时，`/upload` 会被旧代理转发到 APIMart 并返回 HTML 首页；现在前端只接受真正的 `http(s)` URL，遇到 HTML 会明确提示「Worker /upload 未部署，请重部署」，不会再把网页源码塞进参考视频框。
- **版本同步**：`APP_VERSION` / header / 文档统一到 v0.17.6。

### 需要用户操作

如果想让 `file://` 本地双击场景下的视频/音频上传不再被 catbox CORS 拦截，需要按 `WORKER_SETUP.md` 重新部署一次 `apimart-cors-proxy.worker.js`。部署后 Studio 的 Base URL 仍填 `https://你的-worker.workers.dev/v1`，前端会自动把上传发到同域的 `/upload`。

---

## v0.17.5 — 2026-05-29 · 上传识别修复 + asset 模式 picker + file:// CORS 警告 + 开发文档

把 v0.17.1-v0.17.4 累积的 3 个用户反馈一次性收敛。同时产出 `STUDIO_DEV.md` 完整开发文档，让 Codex / 其他 agent 能直接接手。

### 修复

- **多模态混合模式下视频/音频被误识别为图片**：`detectKind` 之前读 `.label-title` 文字判断（"视频"/"音频" 关键字），但多模态模式下文字是「最多 3 段」「最多 9 张」不含关键字。改为读 `box > svg > use` 的 `href` 属性（`#i-image` / `#i-video` / `#i-music`），跨模式可靠。
- **「用素材库」模式下的「从素材库选」走文件上传**：v0.17.3 给所有 `.ref-uploader-compact` 统一上 file picker，但 asset 模式应该是从已审核素材里**选**而不是上传新文件。新写 `upgradeAssetPicker`：点击 → 弹 prompt 让粘 `asset_id` 或 `asset://` URL → 自动补前缀 → 显示 ✓ + ✕ 删除按钮。下个版本会做下拉选择 UI。
- **catbox.moe 在 `file://` 协议下 CORS 被拒**：catbox.moe 的 `Access-Control-Allow-Origin` 不接受 null origin（file:// 协议）。现在检测到 `location.protocol === 'file:'` 且没配 Pixeldrain Key 时，弹 confirm 友好解释三条出路：① 配 Pixeldrain API Key 走 Pixeldrain；② 用 `python -m http.server 8000` 起本地 HTTP；③ URL paste 兜底。**注意**：真正干净的修法是让用户在 Cloudflare Worker 加 `/upload` 路由代理，STUDIO_DEV.md 记录了 5 个选项让 Codex 决定。

### 开发文档

新产出 `STUDIO_DEV.md`，给 Codex / 其他 coding agent 接手用。13 个 section 覆盖：

- 项目结构 + 文件层次
- 主 HTML 按**行号**导航的内容地图（CSS 在哪一行、各 view 在哪一段、JS IIFE 在哪个范围）
- 版本演进时间线 v0.15.0 → v0.17.5
- 关键 JS 模块组织 + 30+ 核心函数的行号定位表
- **5 个已知 bug**：每个写「症状 / 根因 / 修法 / 位置 / 推荐方案」四段
- 代码风格规范（强调用 Python 脚本字符串替换 + assert anchor）
- localStorage keys 清单 / API endpoints 清单 / 外部上传服务对比表
- 17 项行为级测试清单
- Tailwind / CSS 已知坑（特别是 hotfix6 body 桥接、hotfix8 按钮类规范）
- 用户口味提醒（反感重复 bug 修不好 / 反感 mock 假装真功能）

文件位置：[STUDIO_DEV.md](./STUDIO_DEV.md)

### 文件统计

- 主 HTML：539 832 → 544 375 字节（+4.5 KB），7 843 → 7 938 行
- 新增文件：STUDIO_DEV.md（~600 行）

---

## v0.17.4 — 2026-05-29 · AI caret 修复 + 系统提示词模板 + 按钮重设计

针对用户的 5 项反馈一次性收齐。重点是 AI 优化的 caret popover 终于点得开了。

### 修复

- **AI 优化的 ▾ caret 点不开**：之前 `cloneNode` 只换了 button，caret 是兄弟元素，v0.17.0 mock 时代的 click listener 还在。每次点 caret 都触发 v0.17.0 + v0.17.4 两个 toggle，两个 toggle 互相 cancel 导致 popover 不显示。改成 cloneNode 整个 `.ai-split` 父级，一次性清掉所有老 listener，再统一重绑 caret + popover + 模型 item click。
- **参考视频上传后预览 + 删除按钮缺失**：之前删除按钮放在 `.upload-thumb` 内部，被 `<video>` 元素绝对定位的内置控件遮挡。现在放在 box 右上角（24×24 圆形红 ×），不论 thumb 里是 `<img>` 还是 `<video>` 还是 `<svg>` 音符都不会被挡。
- **「提交生成」+「从库选」按钮设计崩**：缺 `inline-flex items-center justify-center gap-2`，导致 svg 跟文字垂直堆叠。从库选额外加 `whitespace-nowrap` 避免「从」和「库选」断行。
- **下载按钮位置/设计差**：之前下载按钮孤零零浮在视频下方，没有 action bar 容器。重做：现在是统一 action bar（带 `border-top` 分隔线），蓝色 primary「下载 mp4」+ 灰色「用尾帧续写」+ 浅色「新标签打开」三按钮并排，右侧显示 task_id（mono 字体）。

### 新功能：AI 优化系统提示词模板编辑

设置面板加 `<details>` 折叠区「AI 优化 prompt 模板（高级）」：

- 图像 + 视频两个独立 textarea，各 6 行
- 「↺ 恢复默认」按钮一键重置
- `localStorage('studio.ai.sysImage' / 'studio.ai.sysVideo')` 持久化
- `callChatCompletions` 优先读自定义，留空走内置默认
- 内置默认仍是 v0.17.1 的扩写策略（图像 150-400 字 / 视频 200-500 字电影分镜）

用法：去设置 → 滚到底 → 展开模板区 → 改 textarea → 失焦自动存。

---

## v0.17.3 — 2026-05-28 · 完整上传模块（ImgBB + catbox + Pixeldrain + 缩略图 + 素材库）

替换 v0.17.2 部分实现的 URL paste 模式，给上传模块做完整重写。

### 新功能

**图床多源支持**：

| 文件类型 | 主图床 | 备用 | 需 Key | 大小限制 |
|---|---|---|---|---|
| 图片 | ImgBB | URL paste | ✅ ImgBB Key | ≤32 MB |
| 视频 / 音频 | catbox.moe | Pixeldrain（若配 Key） / URL paste | ❌ 默认 / ✅ Pixeldrain Key 可选 | ≤200 MB（catbox）/ ≤1 GB（Pixeldrain） |

**上传交互**：

- 点击上传框任意位置 / 点「上传」按钮 / 拖拽文件 — 三种方式都触发
- 上传中显示「⏳ 上传 ImgBB...」「⏳ 上传 catbox...」等进度文案
- 上传成功后 box 内左侧显示 64×64 缩略图：图片用本地 dataURI 即时预览（不等服务器）；视频用 `<video preload="metadata">` 第一帧；音频显示音符 svg 图标
- 缩略图右上角小红 ✕ 删除按钮（hover 放大 1.15x）

**素材库批量上传**（虚拟人像 + 真人人像）：

- 虚拟人像「批量上传 (≤20)」按钮：弹文件选择器（multiple）→ 选最多 20 张图 → 弹组名 prompt → 并行 ImgBB 上传（每 4 张一批避限流）→ 拿到 URL 列表 → POST `/v1/seedance2/private-avatar` 提交审核
- 真人认证步骤 4「批量上传真人视频」：弹 `group_id` prompt → 文件选择器（多文件）→ 串行 catbox/Pixeldrain 上传 → POST `/v1/seedance2/real-avatar`

### 修复

- v0.17.1 的 `attachUrlUploaders`（URL paste 老 handler）跟 v0.17.2 的 cloneNode 替换打架，导致点击没反应。彻底注释掉 v0.17.1 的旧 handler，全部用 v0.17.3 的新模块。
- v0.17.1 的 `bindVirtualAvatarUpload`（旧 URL prompt 模式）early return，让 v0.17.3 的文件选择版本接管。

### v0.17.3 hotfix1（同日）

- **CSP 阻止视频结果加载**：CSP meta 缺 `media-src`，`<video>` 元素加载远端 mp4 被 CSP 用 `default-src 'self'` fallback 拦截。修：加 `media-src 'self' https: blob: data:`；顺便加 `https://fonts.googleapis.com` 到 `style-src`、`https://fonts.gstatic.com` 到 `font-src`，修掉之前控制台一直警告的 Google Fonts 阻塞。
- **AI 优化 SSE 解析失败**：APIMart 的 chat completions 即使没传 `stream: true` 也返回 SSE 格式（`data: {...}\n` 流式），导致 `resp.json()` 抛 `Unexpected token 'd', "data: {"id"... is not valid JSON`。修：先 `resp.text()` 拿原文，判断开头是不是 `data:` → SSE 模式逐行拼 delta.content；否则走 `JSON.parse`。同时显式带 `stream: false` 在 body 里试图让网关走 JSON。
- **Pixeldrain 401**：Pixeldrain 现在匿名上传需付费 key 了。改默认走 catbox.moe（免 key、≤200MB、永久），Pixeldrain 改成可选高阶（设置面板加 Pixeldrain API Key 字段，配了就走 Pixeldrain ≤1GB）。
- **删除按钮设计**：原来是「✕ 删除」红边框白底按钮浮在右上角，太抢眼。改成 18×18 圆形红 ×，hover 放大到 1.15x，无确认弹窗。

### v0.17.3 hotfix2（同日）

- **提交按钮点击无反应**：v0.17.0 hotfix8 把视频区的 `.btn-primary` 改成 Tailwind 类后，v0.17.1 的 `bindSubmitButton` 还在用 `.btn-primary` 选择器找不到按钮。改用文本匹配 `b.textContent.includes('提交生成')`，加 `dataset.submitBound` 防重复绑定。
- **AI caret popover 在 cloneNode 后失效**：v0.17.1 cloneNode 清掉了 v0.17.0 的 popover 选项 click listener。在 `bindRealEnhance` 内增加 caret + popover + model item 的重绑代码。
- **上传 box 缺删除功能**：完整版上线（后被 v0.17.4 进一步优化位置）。
- **提交按钮 / submitVideo 增加详细 console 日志**（`[v0.17.1] submitVideo called` / `payload built` / `submit button clicked`），方便用户反馈时定位。

---

## v0.17.2 — 2026-05-28 · ImgBB 图床集成 + Prompt 灵感库 SVG 截图临摹

### 新功能

- **设置面板新增「图床（ImgBB）」section**：API Key 字段 + 注册链接 + 友好说明。`localStorage('studio.imgbbKey')` 持久化。
- **上传按钮文件选择器**：点视频区的上传按钮 → 弹原生 file picker → 选图片 → 自动转 base64 → POST 到 `https://api.imgbb.com/1/upload` → 拿真实 URL 存到 `box.dataset.url`。没配 key 时降级到 URL paste prompt。
- **Prompt 灵感库 SVG 视觉临摹**：之前两张大卡只有单色 SVG 图标 + 渐变背景。换上对应站点的 SVG 视觉临摹 — youmind 用彩虹渐变 + 大字「SEEDANCE 2.0」+ 黑框白底「提示词」标签；atlascloud 用纯黑底 + 三角 logo + 渐变星标 + 大字 + 底部分类 chip 模拟条。识别度大幅提升。

### v0.17.2 hotfix（同日）

- v0.17.1 的 URL paste handler 跟新文件选择器打架。彻底删 v0.17.1 旧逻辑，新写完整模块：① 点击文件选择器；② 拖拽文件；③ ImgBB 上传；④ 上传中状态显示；⑤ 失败 4 秒后自动复位。
- 之前缺**拖拽**功能完全没实现，hotfix 补上。

---

## v0.17.1 — 2026-05-27 · 真 API 接通：AI 优化 + 视频提交 + 任务轮询

替换 v0.17.0 的所有 mock，接通真 APIMart endpoints。

### 真 API 集成

- **AI 优化 prompt**：POST `/v1/chat/completions` · OpenAI 兼容协议 · 3 个模型（gpt-4o-mini 默认 / claude-3-5-sonnet / gpt-4o）· 各自 system prompt 针对 GPT-Image-2 vs Seedance 2.0 调优。
- **视频任务提交**：POST `/v1/videos/generations` · 表单读字段 → 按互斥规则组 body → 拿 task_id。
- **视频任务轮询**：GET `/v1/tasks/{id}?language=zh` · 提交后 3s 起轮，处理中 5s，未知状态 8s，网络错误 10s 退避。
- **本地价格预估**：4 模型 × 3 分辨率 heuristic 价表，改字段实时刷新。
- **虚拟人像批量提交**：POST `/v1/seedance2/private-avatar` · 弹 URL 列表 prompt + 组名（v0.17.3 换成文件选择器）。
- **真人认证步骤 2 查询**：POST `/v1/seedance2/real-avatar` · 粘 byted_token → 返回 group_id。

### 任务卡渲染

- 动态生成 `<article class="solid-card task-card">` DOM
- 状态徽章：提交中 / 已提交 / 处理中 / 已完成 / 失败 / 已取消
- 完成后 `<video>` 内嵌播放器 + 下载 mp4 按钮 + 续写按钮（条件显示）
- 失败时 banner 显示错误信息

### 视频区布局重构

- tab nav 改成 inline 横排：图像 / 视频 是主 tab，Mask + Prompt 库是图像的 sub-tab，Prompt 灵感库是视频的 sub-tab
- 视频 tab 内的 main-grid 跟图像 tab 用同款 Tailwind `lg:grid-cols-12 lg:col-span-5/7`，视觉一致

### v0.17.1 hotfix（v0.17.0 → v0.17.1 期间多次小修）

| hotfix | 修了什么 |
|---|---|
| 1 | sub-tab 点击时丢 group 上下文 → 用 `closest('.sub-tabs-inline')` 反推父级 |
| 2 | 补 `.main-grid` CSS（合并漏带），清 demo 测试数据（4 mock task / 4 mock asset / prompt 默认值） |
| 3 | 浅色 solid-card 边框 / 阴影增强 |
| 4 | violet 染色边框 + 4 层阴影 |
| 5 | 用 `#viewVideo` 高特异性选择器强制覆盖 |
| 6 | **真正根因**：v0.16.0 的桥接 `.bg-slate-50,.bg-white{...!important}` 把 body bg 强制成白色（body 用 bg-slate-50 类），白底白卡看不出边。拆开两条规则：bg-slate-50 → `var(--bg-deep)` lavender，bg-white → `var(--bg-elevated)` white |
| 7 | 任务队列 section 没 col-span（默认占 1/12 列宽 ~133px 导致"视频任务队列"被压成竖排），Prompt 灵感库错挂 col-span，JS 任务卡模板里的 col-span 都修了 |
| 8 | 视频区表单元素改用图像区同款 Tailwind utility 类（textarea / select / input / button） |

---

## v0.17.0 — 2026-05-26 · Seedance 2.0 视频生成 + AI 优化 prompt + 品牌升级

整合 APIMart 的 Doubao Seedance 2.0 视频生成接口，把 Studio 从纯图像扩到「图像 + 视频」双形态 AIGC 工作台。同时上线 AI 优化 prompt 功能、外链 Prompt 灵感库、tab 导航重构。

### 品牌升级

- **产品名：** Ling's GPT-Image-2 Studio → **Ling's AIGC Studio**（图像 + 视频 + 未来音频，统一旗下）
- 文件名保持 `gpt-image-2-studio.html` 不动，避免书签 / GitHub Pages URL 中断
- header h1、`<title>`、meta description、og:title、动态 document.title 一并改

### 视频生成功能（Doubao Seedance 2.0）

四个模型变体覆盖所有场景：

- `doubao-seedance-2.0` — 标准版，1080p 支持
- `doubao-seedance-2.0-fast` — 快速版（默认选中，省钱）
- `doubao-seedance-2.0-face` — 真人版，1080p
- `doubao-seedance-2.0-fast-face` — 真人快速版

**6 种输入模式**（segmented 切换，UI 自动隐藏不兼容字段）：

1. 文生视频
2. 图生视频（首帧 1 张图）
3. 首尾帧（first_frame + last_frame）
4. 视频参考（≤3 段，总时长 ≤15s）
5. 多模态混合（图 + 视频 + 音频）
6. 用素材库（asset_id 引用审核通过的人像）

**字段互斥规则可视化**（避免用户手动组合出 400 请求）：

- `image_urls` ↔ `image_with_roles` 互斥
- 用 `image_with_roles`（首尾帧）→ `video_urls` / `audio_urls` 全禁
- `audio_urls` 必须配 `image_urls` 或 `video_urls`
- `1080p` 仅 `2.0` / `2.0-face` 支持，fast 系列没有
- `asset://` URL 仅 `2.0` / `2.0-fast` 可用；face 系列必须走真人素材库

**视频任务卡：**

- `<video>` 播放器内嵌（controls + preload="metadata"，海报用 last_frame）
- 状态徽章：已完成 / 处理中 / 重试 / 失败 / 续写链
- 任务卡折叠（`<details>`）+ 右上「全部折叠」按钮
- 任务队列独立滚动（`task-queue-scroll`，max-height + overflow-y）
- 左侧表单 sticky（宽屏下不跟着滚走）

**素材库（v0.17.0 初版 UI）：**

- 虚拟人像（POST `/v1/seedance2/private-avatar`）：批量上传 ≤20 张 → 审核任务卡 → asset_id 卡片列表（含 Active / Failed 状态）
- 真人人像（POST `/v1/seedance2/real-avatar`）：4 步进度条 + QR code 大图 + 跨设备轮询 byted_token + 步骤 4 真人视频上传
- QR code 用纯 SVG 手画，无外部依赖
- `callback_url` 字段填占位符 `https://example.com/seedance-callback`（纯前端拿不到 webhook，靠轮询查 byted_token）

### Prompt 灵感库（外链卡片导航）

skip iframe，原因：youmind.com 设 `X-Frame-Options: DENY` 拒绝嵌入，atlascloud 也一样。改成 2 张大渐变卡片导航：

- 视频：[youmind](https://youmind.com/seedance-2-0-prompts) + [atlascloud](https://www.atlascloud.ai/prompts-hub/seedance-2-prompt?locale=en)
- 图像：本地 Prompt 库（v0.16.0 已有 36 条）+ [atlascloud](https://www.atlascloud.ai/prompts-hub/gpt-image-2-prompt?locale=en) 外链卡片

点视频右侧 sub-tab「Prompt 灵感库」自动平滑滚到卡片区。本工具不抓取这两个站的内容、不缓存截图，仅做嵌入/跳转。

### AI 优化 prompt

Prompt 输入框上方加 **split button**（左主按钮触发 + 右 caret 弹 popover 选模型）：

- **GPT-4o-mini**（默认，~$0.0005 / 次）
- **Claude 3.5 Sonnet**（~$0.01，中文文采好）
- **GPT-4o**（~$0.02，顶级）

选择 `localStorage('studio.aiModel')` 持久化。按钮 label 实时显示当前模型（`AI 优化 · 4o-mini`）。优化中显示 spinner，完成后 textarea 有 1.2s accent 高亮边框。

**目前是 mock 行为**（fake 1.2-2.4s spinner + 写好的扩写文本）。**v0.17.1 计划接真 API**：POST `{baseUrl}/chat/completions`，复用现有 API Key + Worker 反代，视频 / 图像分别有不同 system prompt。

### Tab 导航重构（inline 横排）

旧：3 个 tab 平铺。新：

```
[图像生成 ●]│ Mask 编辑器  Prompt 库 │ [视频生成]
[图像生成]   │ [视频生成 ●]│ Prompt 灵感库 │
```

主 tab 选中后，对应 sub-tab inline 长出来；切换主 tab，sub-tab 跟着换。视频 sub-tab「Prompt 灵感库」点击 → 平滑滚到 viewVideo 内的卡片区。

### UI / 交互细节补充

- 宽屏：container `max-width` 1280 → 1600px；≥1920 时升到 1800px
- 深色模式下拉 `<option>` 可读性修复（v0.16.0 选中后未选项接近不可见）
- 视频价格预估 chip：实时显示「预计 $0.18 · 5s · 720p · fast」
- 互斥规则速查卡片（底部 7 条规则一目了然）

### 文件 / 备份

- 备份：`gpt-image-2-studio-v0.16.0.bak.html`（v0.16.0 原版，可一键回滚）
- 主 HTML 体积：408 945 → **479 074 字节**（+70 KB，含视频 tab 全套 UI + token + 22 条新 CSS + 14 个新 SVG icon + AI 优化 JS）
- 行数：5434 → 6518
- 设计来源 demo：`studio-video-preview.html`（108 KB / 2088 行，独立可点击预览）

### 回滚方法

```
cp gpt-image-2-studio-v0.16.0.bak.html gpt-image-2-studio.html
```

### v0.17.1 路线图（短期）

- AI 优化接 APIMart `/v1/chat/completions` 真实调用 + 视频 / 图像分别的 system prompt
- 视频任务真接 `/v1/videos/generations` + 轮询 `/v1/tasks/{id}`
- 视频价格表加进设置面板
- 视频任务专用轮询超时（默认 15 分，face 模型可达 5+ 分钟）
- 素材库真接 `/v1/seedance2/private-avatar` + `/v1/seedance2/real-avatar`
- 真人认证 QR 用 `qrcode-svg` 库（~3KB）生成真 QR
- 视频任务、素材列表、AI 模型选择 localStorage 持久化

---

## v0.16.0 — 2026-05-22 · UI 大改版：克制玻璃（Restrained Glass）

跟着 `nextlevelbuilder/ui-ux-pro-max-skill` v2.5.0 把界面从「亮色 Tailwind 默认」升到「Cinema Dark + 局部 Glassmorphism + Drawing Canvas 紫调色板」。整体走 skill 推荐的工作台风：Modern Dark (Cinema) 作主体、Glassmorphism 只用在悬浮层（Toast / Tabs / Modal），避免长时间高密度阅读时玻璃后背景跟着滚动闪烁。

### 视觉系统

- **配色：** 双主题。深色版底 `#0a0612 → #120a1f` 紫渐变 + violet `#7c3aed` 主色 + cyan `#06b6d4` 次色；浅色版底 `#faf5ff` lavender + 同主色。两套都自带 ambient blob 慢振荡（28-42 s 周期），`prefers-reduced-motion` 一关全停。
- **字体：** Inter 300-700 主字体（替换原系统默认）、JetBrains Mono 用作代码 / prompt 显示。中文 fallback 仍是 PingFang / 微软雅黑。
- **token 体系：** 写成 CSS custom properties。间距 4 pt 基线（`--space-1..8`）、圆角四档（`--radius-sm..xl`）、字阶 8 档、运动 `cubic-bezier(0.16,1,0.3,1)` 全局 easing。
- **卡片策略：** 大卡片（任务卡、表单、设置面板、Pre-delivery checklist）走 solid `--bg-elevated`，悬浮层（Toast / Tabs / Compare bar / chip）走 `backdrop-filter: blur(20px) saturate(160%)`。原 18 处 blur 收敛到 5-6 处大型surface，性能影响显著降低。

### 主题切换

- 头部新增 sun/moon 切换按钮。
- `localStorage('studio.theme')` 持久化用户选择。
- 首次访问按 `prefers-color-scheme` 自动跟随系统。
- `<html data-theme="dark|light">` 切换驱动所有 token 变化，无需任何脚本重新渲染组件。

### 图标：emoji → Lucide SVG

按 skill `no-emoji-icons` 规则替换 26 个 UI 图标 emoji 为内联 Lucide SVG（25 个 symbol，零外部请求）：

| 原 emoji | Lucide 名 | 用在 |
|---|---|---|
| `⚙️ / ⚙` | settings | 设置按钮 |
| `🚀 / ✨` | sparkles | 生成 tab / AI 提示 |
| `🎨` | palette | Mask 编辑器 tab |
| `📚 / 📖` | book-open | Prompt 库 tab / 说明 |
| `🔁 / 🔄` | refresh-cw | 重试 / 刷新 |
| `🛠️` | wrench | 高级设置 |
| `💰` | coins | 价格 / 余额 |
| `⚠️ / ⚠` | alert-triangle | 警告横幅 |
| `🔑` | key | API Key |
| `🗑` | trash-2 | 删除 / 清除 |
| `👁` | eye | 显示密码 |
| `📋` | clipboard | 复制 |
| `✎` | pencil | 编辑 |
| `❤️` | heart | 收藏 |
| `🔍` | search | 搜索 |
| `⏳` | clock | 等待 |
| `❓` | help-circle | 帮助 |
| `⬇️` | download | 下载 |
| `👍` | thumbs-up | 点赞 |
| `✓` | check | 确认 |
| `⛶` | maximize | 全屏 |
| `🧲` | magnet | 磁性套索 |
| `📁` | folder | 上传 |
| (新) | sun / moon | 主题切换 |

**保留 emoji**（这些是内容 / 语义分类，不是装饰图标，所以不替换）：
- `⌘` —— Mac 键盘 Cmd 键提示
- `👤 🌄 🙂 ✋ 🦶` —— Mask 编辑器 Quick Mask 预设按钮（人物 / 背景 / 头部 / 双手 / 双脚）
- Prompt 库分类 emoji 🛒 📣 🍌 🧪 等 —— 在 `<script>` 内的字符串字面量里，是 CHANGELOG 文本和分类标签

### 交互：focus / hover / press / motion

- `button:focus-visible` 统一 4 px 双层 ring（外层底色 2 px + 内层 accent 2 px），键盘可见。
- `button:active` 全局 `scale(0.97)` 反馈，120 ms。
- Hover transition 180 ms，`var(--ease-out)` 收敛。
- 文本/数字输入 focus 时边框换 accent + 3 px accent-glow。

### Tailwind 桥接策略

主 HTML 5254 行 / 388 KB 有海量 Tailwind 工具类。这次没改 HTML 上的 class，改用 CSS 选择器把所有 `bg-white / bg-slate-*` / `text-slate-*` / `border-slate-*` / `bg-blue-* / border-blue-*` / `bg-amber-* / bg-rose-*` 重定向到新 token。这样：

- 结构零破坏，回滚成本低
- 双主题切换瞬间生效
- 第三方 prompt 渲染（JS 动态生成的 markdown 节点）也自动适配

### 文件 / 备份

- 备份：`gpt-image-2-studio-v0.15.0.bak.html`（与上次 `gpt-image-2-studio-v0.15.0.html` 内容相同，多一份保险）
- 迁移脚本：`migrate_v0.16.0.py` 保留在 outputs/，可复盘
- 设计来源 demo：`gpt-image-2-studio-ui-preview.html`（全玻璃 v1）+ `gpt-image-2-studio-ui-preview-v2.html`（克制玻璃 v2，最终采纳）
- 主 HTML 体积：388 537 → 408 945 字节（+20 KB，含 v2 token + SVG sprite + 主题脚本）

### 回滚方法

```
cp gpt-image-2-studio-v0.15.0.bak.html gpt-image-2-studio.html
```

或者用 `gpt-image-2-studio-v0.15.0.html`，二者内容相同。

---

## v0.15.0 — 2026-05-18 · Mask 编辑器三件套：矩形 / 多边形套索 / 磁性套索

之前 Mask 编辑器只有「涂抹」和「擦除」两个画笔工具，加上 AI 快速选区（人物 / 背景 / 头 / 手 / 脚），但缺少**几何 / 半自动**选区。这次补齐三个最常用的工具，应对画笔太慢、AI 不够精的场景。

### ▢ 矩形选区（快捷键 R）

最简单：按住鼠标拖动出矩形 → 松开自动填充到 mask。覆盖整片背景、全屏 inpaint、或者快速画一个 ROI 都用它。

### ✎ 多边形套索（快捷键 L）

不规则物体边界手动选区。

- 单击：锁定一个锚点
- 拖动：实时虚线预览（从最后锚点到光标）
- 双击 / Enter / 点回起点（自动吸附 12px）：闭合多边形 → 内部填充
- ESC：取消当前选区
- Backspace：撤回最后一个锚点

UI 上锚点是蓝色小圆点；起点画一个空心大圆，提示用户「点这里能闭合」。

### 🧲 磁性套索（快捷键 M）

基于 **Sobel 边缘检测 + 局部吸附**的半自动工具，灵感来自 PS 的磁性套索。

实现思路：

1. 切到这个工具时，先对原图运行 Sobel 3×3 算子，生成「边缘强度图」（Float32 灰度 → Sobel → Uint8ClampedArray 缓存）。性能优化：对超过 1024 长边的图先 downscale 到 1024，4K 大图也只要 50-200 ms 算完。
2. 鼠标移动时，在光标周围 12px 半径圆形邻域内找 Sobel 强度最高的像素 → 把自由端「吸」到那个点
3. 边缘强度低于阈值 30 时不吸（防止纯色区域瞎吸）
4. 点击锁定锚点（也吸附），Enter / 双击闭合

**对比 PS**：

- ✅ 简单边界（前景物体 vs 反差大的背景，比如人 vs 天空）效果接近
- ✅ 性能很好，4K 图也不卡
- ❌ 半透明 / 模糊 / 低对比度边缘可能跑偏
- ❌ 没做 Dijkstra 最短路径，所以两个锚点之间是直线，不会沿边缘走（PS 完整版会）。补救：多点几个锚点

### 共享设计

- 三个工具共享：锚点管理、闭合检测、多边形 fill('evenodd') 填充
- 切换工具时如果有未闭合选区，静默丢弃（不打断用户）
- 换图 / 清空图时自动失效 Sobel 缓存 + 重新算（如果磁性是当前工具）
- 历史撤销/重做：选区填充后正常进入 history，跟画笔一样可撤

### 状态栏

工具按钮组下方加了一个动态状态栏，会显示：

- 当前工具 + 操作提示（「点击锁点 · Enter 闭合 · ESC 取消」）
- 套索模式下：「已锁 N 点」实时数
- 磁性模式 + Sobel 计算中：「正在分析图像边缘…」

### 文件

- 主程序：`gpt-image-2-studio.html`（约 +500 行）
- 备份：`gpt-image-2-studio-v0.15.0.html`

---

## v0.14.2 — 2026-05-18 · 官方通道大图修复 + 拥堵自适应 + 超时可配

### 🚨 Worker 修复（必须重新部署）

- **修复 Cloudflare Worker 反代转发 body 丢失的 bug**：之前 `apimart-cors-proxy.worker.js` 直接把 `request.body`（ReadableStream）传给 fetch，在 Cloudflare runtime 中不显式声明 `duplex: 'half'` 时 stream 会被「静默跳过」。后果：APIMart 上游收到的 body 是空的，所有 official 渠道请求报 `Missing required parameter: 'model'` / `empty string`。
- **修法**：改成 `await request.arrayBuffer()` 先读到内存再转发，同时去掉 `Content-Length` / `Transfer-Encoding` 头让 fetch 自动重算。**如果你在用 Worker 反代，请重新粘贴 `apimart-cors-proxy.worker.js` 到 Cloudflare 编辑器并 Deploy**。

### 📦 智能参考图压缩（新）

发现问题：`gpt-image-2-official` 渠道对单次请求体大小有阈值，参考图（特别是 4K PNG）base64 后超过约 2-3 MB 就会触发上游 OpenAI 静默丢字段。中转通道（`gpt-image-2`）因 APIMart 自己重新编码所以不受影响。

**自动压缩功能**：

- 上传 > 1 MB 的图自动判定是否需要压缩
- 分级降级策略：JPEG q0.92（保持原尺寸）→ q0.88 + 最长边 2048 → q0.85 + 1536 → q0.82 + 1280 → q0.80 + 1024
- 目标输出 < 2 MB，几乎一次到位
- Toast 显示压缩前后体积 + 采用的策略
- 设置面板「🛠️ 高级」有开关，默认开启
- 对 AI 生图模型来说，参考图 > 2048 px 的部分本来就是浪费（模型内部 downsample 到 1024-1536），所以压缩几乎不损失实际效果

### 🚦 拥堵自适应（503 / 429 Retry-After）

APIMart 高峰期（晚 7-11 点）经常返回 `Service temporarily overloaded (queue=prod_default pending=12089 > 3000). Please retry after 10 seconds`。之前默认重试间隔 5 秒会反复被拒。

**改进**：

- 自动解析 `Retry-After` HTTP header（标准做法）
- 同时解析错误体里的 `retry after N seconds` 文本（APIMart 用这种）
- 收到 503/429 + 解析到秒数 → 本次重试用 `max(用户配的间隔, 服务器要求的秒数)`，封顶 5 分钟
- Toast 提示「服务过载，X 秒后自动重试（服务器要求 Y s）」
- 错误消息加「服务过载」前缀，更友好
- 设置面板有开关「服务过载（503/429）时自动遵守服务器要求的等待时间」，默认开启

### ⏱ 单任务轮询超时可配置

之前 `POLL_TIMEOUT_MS = 8 分钟` 硬编码。APIMart 官方通道生成慢时（10-15 分钟很常见）会被前端误判为超时然后重试，浪费一份额度。

- 设置面板「🛠️ 高级」加「单任务轮询超时（分）」输入框
- 范围 1-60 分钟，默认 8 分钟
- 硬封顶 1 小时（避免 state 损坏导致无限轮询）
- 超时报错信息提示去哪里改

### 其他

- **默认重试间隔从 5 秒改为 15 秒**：覆盖 APIMart 拥堵时通常要求的 10 秒
- 持久化新增 `autoCompress` / `pollTimeoutMin` / `respectRetryAfter` 三个字段

### 关联文件

- 主程序：`gpt-image-2-studio.html`
- Worker 反代：`apimart-cors-proxy.worker.js`（也已修复，部署的话需要重新粘）
- 备份：`gpt-image-2-studio-v0.14.2.html`

---

## v0.14.0 — 2026-05-16 · 发布准备：CSP / HTTPS 校验 / 危险区 / API Key 风险提示 + 配套文档

为「自己 + 5-10 个朋友」级别的发布做完整安全加固，配套 README / LICENSE / 部署教程。

### 安全加固

- **CSP（Content Security Policy）meta**：限制脚本来源到 `jsdelivr.net / cdnjs.cloudflare.com / tailwindcss.com` 白名单，即使页面被 XSS 注入也无法 fetch 到陌生服务器外泄 API Key
- **`meta robots noindex+nofollow`**：搜索引擎不收录这个工具页面，陌生人不会偶然找到
- **Base URL HTTPS 强制校验**：保存时校验必须 `https://` 或 `http://localhost / 127.0.0.1 / [::1]`；不符合 → 输入框红框 + toast 警告。**作用**：HTTP 是明文传输，Authorization header 里的 API Key 会被路径上任何 WiFi 热点 / 网络代理 / 运营商抓包看到
- **API Key 输入框红框警告**：醒目提示「仅私人设备使用 / 怀疑泄漏立即撤销重发」
- **「⚠️ 危险区」面板**：折叠式（避免误点），含「🔑 清除已保存的 Key」+「🗑 清除所有本地数据」（双重 confirm）
- **额外 meta**：`X-Content-Type-Options: nosniff`、`referrer-policy: strict-origin-when-cross-origin`

### 发布素材

- **OG / Twitter Card 标签**：分享链接到 Slack/微信/Twitter 有预览卡片
- **README.md**（纯中文）：三种使用方式（本地双击 / GitHub Pages / 自部署）、API Key 申请引导、安全说明、技术栈、文件清单
- **LICENSE**（MIT）：自由使用 / 修改 / 分发
- **robots.txt**：`Disallow: /` 双保险（noindex meta + robots.txt）
- **DEPLOY_GUIDE.md**：小白版 GitHub Pages 部署图文教程（Step 1 创建 repo → Step 5 分享给朋友），常见 FAQ

### XSS 审查（无新增改动，验证通过）

- 全部 `innerHTML` 含用户输入处都已用 `escapeHtml()` 包装
- `toast` 用 `textContent`
- Prompt 库的 `_escapeHtml` 覆盖卡片 / 详情 Modal 所有字段

### 部署目标

`https://chenpaonimei.github.io/MyStudio/gpt-image-2-studio.html`

## v0.13.7 — 2026-05-15 · 修复 EvoLink parser：图片是 HTML img 标签不是 markdown 图片语法

- 根因：EvoLink cases/*.md 里图片用 `<a><img src="..."></a>` HTML 形式而不是 `![](url)` markdown 形式，原 regex 一律不匹配，27 个 case 全被跳过
- 修复：imgMatch 正则改为先匹配 HTML `<img src="...">`，fallback markdown 形式
- cache key V4 → V5 强制刷新

## v0.13.6 — 2026-05-15 · EvoLink parser 加详细诊断（text 长度 / Case 头数 / parse 数）

- v0.13.5 fetch 都成功但每个分类 parse 出 0 条，加 `console.info` 输出每个分类的 text.length / `### Case` 头数 / parse 数 + 前 200 字预览
- 排查思路：text=0b=拉到空 / Case头=0=不是 markdown / parsed<header=正则不匹配
- cache key V3 → V4 强制刷新

## v0.13.5 — 2026-05-15 · EvoLink fetch 兜底：加 GitHub raw 备用 URL + 详细日志

v0.13.4 实测「全部 7 个 EvoLink 分类都拉取失败」。原因可能是 jsdelivr 主 URL 在某些网络下不可达。本版加兜底链路 + 详细诊断。

- **双 URL 候选**：每个分类先试 `cdn.jsdelivr.net`，失败自动降级到 `raw.githubusercontent.com`；都失败才把此分类标失败
- **详细日志**：
  - `console.info` 打印每个分类的成功条数（例：`[EvoLink] 成功: ecommerce(20), ad(8), portrait(35)...`）
  - `console.warn` 打印失败分类的具体 URL 和错误（HTTP 状态码 / 网络错误信息）
- **错误提示更新**：从「全部 7 个失败」改成「详情见 F12 console」，引导用户开发者工具排查

> 如果两个 URL 都不可达，通常是网络拦截 jsdelivr 和 raw.githubusercontent.com 两端。可以挂代理或者反馈给我，我考虑放镜像。

## v0.13.4 — 2026-05-15 · EvoLink 数据源升级：从 36 条扩展到 100+ 条完整分类

**数据扩容**：v0.13.0 只 parse 了 EvoLink 的 `README_zh-CN.md`（只含 36 条精选），现在改为并行拉 `cases/` 目录下 7 个分类的完整 markdown 文件。

- **7 个分类文件**：`ecommerce / ad-creative / portrait / poster / character / ui / comparison`，每个一个 `*_zh-CN.md`
- **并行 fetch**：`Promise.allSettled` 拉 7 个文件，任一失败不影响其他；console.warn 列出失败的分类
- **parser 兼容中英文 prompt 标记**：`**Prompt:**` 和 `**提示词：**`
- **简化逻辑**：每个文件 = 一个分类，不再需要 H2 切片，category 直接打标
- **cache key 升级**：`promptLibV2` → `promptLibV3`，老用户打开 Prompt 库会自动重新拉取
- **预期数据量**：ApiMart 107 + EvoLink 100+ ≈ **210+ 条 prompt**，覆盖 7 大类 + 未分类

## v0.13.3 — 2026-05-15 · 紧急修复：提交任务报 `debugHtml is not defined`

**严重 bug**：点击「🚀 提交生成」立即报 `Uncaught ReferenceError: debugHtml is not defined`，整个任务提交流程失效。

- **根因**：v0.13.1 在 `taskHTML` 函数底部加了 `${!isCollapsed ? debugHtml : ''}` 引用，但**对应的 `const debugHtml` 定义没成功插入到函数体内**（python patch 当时的 `assert` 不知怎么没触发，导致 replace 假装成功了；node syntax check 也没能识破因为引用 undef 变量是运行时错误）
- **修复**：补回 `const debugHtml = (...) ? \`...\` : ''` 到 `taskHTML` 函数内的 `errorBg` 定义之前
- **教训**：以后大段 python `replace` 之后必须 grep 验证新增 token 真的存在，光靠 syntax check 不够

## v0.13.2 — 2026-05-15 · 修复任务图核心 bug：batch 接口只返回 {id, status} 时自动 fallback

**根因（用户提供 console payload 后实测确认）**：APIMart 的 `POST /v1/tasks/batch` 实际返回的对象**只有 `{id, status}` 两个字段**，没有 `result.images`。这与 `image2-generate-apimart-v5.md` 文档 §5.5 说的「响应是任务对象数组，结构同 §5.2」**不一致**。完整的 result 数据需要再调单查询 `GET /v1/tasks/{id}` 拿。

**修复**：
- `applyTaskUpdate` 的 completed 分支在检测到 `t.images.length === 0` 时，自动 fetch `GET /v1/tasks/{id}?language=zh` 拿完整 payload，再走 `extractImages` 提图
- 新增 `fetchTaskFullDetails(t)` 辅助函数 —— 不动 `state.activeCount` / `pumpQueue`（避免 batch 已经处理过的状态被重复扣减），只更新 `t.images / t.actualTime / t.cost`，调度重渲
- 加 `t._fallbackTried` 守卫，单个任务只 fallback 一次防无限循环
- 单查询也提不出图时仍存最新 payload 到 `_debugPayload`，琥珀色调试 UI 照样显示

> 整个 v0.7.0 引入 batch poll 后这个 bug 就潜伏着，但因为大多数任务在 batch 看到 completed 之前就被单查询拿到了完整数据（pollOne 是兜底分支），所以一直没暴露。直到 batch 完全接管 + 任务很快完成 + 用户截图看到诊断 toast，才让根因浮出水面。

## v0.13.1 — 2026-05-15 · 修任务卡脏状态 + 加调试 payload UI

实测发现：任务卡同时出现「已完成」绿色徽章 + 红色「失败」错误框，这是 **t.error 残留导致的脏数据**（前一次 poll 可能因 batch 接口格式异常误触 failed，后一次 poll 拿到 completed 但没清 error）。

- **修复（关键）**：两层防御：
  1. `applyTaskUpdate` 完成时强制 `t.error = null` + `t.firstFailedAt = null`
  2. `renderTaskRow` 错误框加 `status !== 'completed'` 守卫，即使 error 残留也不再显示
- **诊断 UI**：完成但 `extractImages` 提不出图时，任务卡里直接显示 raw payload 调试块（琥珀色面板）+「📋 复制 raw payload」按钮 —— 不用 F12 也能把数据复制给开发者排查
- **清理**：成功提到图时 `_debugPayload = null`，避免下次显示脏数据

> 根因可能不只一个。如果你看到调试块出现，把里面的 raw payload 复制给我，我对照实际字段精修 extractImages。

## v0.13.0 — 2026-05-15 · Prompt 库双源合并 + 分类筛选 + 任务图修复

### Prompt 库扩展

- **新增数据源**：集成 [`EvoLinkAI/awesome-gpt-image-2-API-and-Prompts`](https://github.com/EvoLinkAI/awesome-gpt-image-2-API-and-Prompts)（约 36 条**已分类** prompt）。两源并行 fetch，任一失败不影响另一个；toast 提示部分失败
- **分类筛选**：左侧新增「分类」下拉 —— 🛒 电商 / 📣 广告创意 / 🍌 人像 / 🎨 海报 / 🧍 角色 / 📱 UI / 🧪 对比 / ❓ 未分类（ApiMart 数据归此）
- **卡片徽章**：「A / E」标识来源（蓝 = ApiMart / 绿 = EvoLink）+ 分类 emoji 徽章
- **详情 Modal**：显示完整来源名 + 分类标签 + 案例标题（EvoLink 独有的 marketing-friendly title）
- **状态条**：显示双源数量 `A:107 + E:36` 实时统计
- **独立滚动条**：右侧网格 `lg:max-h-[calc(100vh-9rem)] lg:overflow-auto`，左侧筛选 sticky 顶部跟随
- **内嵌 markdown parser**：JS 在浏览器里 parse EvoLink 的 README — H2 切章节、`### Case <N>` 提取 case，正则抽 title / author / 原帖 URL / 图片 URL / prompt 文本；自动跳过缺字段的 case
- **自动语言检测**：EvoLink 没有显式 `lang` 字段，按中日韩字符占比识别（>5% 中文 → zh，>1% 假名 → ja，>1% 韩文 → ko，否则 en）

### 任务图显示 / 下载 修复

实测发现已完成任务的预览图、URL、下载按钮全部不展示。根因猜测：API 返回的字段结构和文档示例不同（可能 result 嵌套了一层，或字段命名变化），原代码硬编码 `d.result.images[].url` 提取失败 → `t.images = []` → 整个图区不渲染。

- **抽出 `extractImages(d)` 函数**：兼容多种字段命名（`result.images` / `images` / `result.urls` / `urls`）和多种 url 形式（string / string[] / `{url, ...}` 嵌套对象）
- **自动 unwrap 多层 wrapper**：批量接口偶尔会把任务再包一层 `{ code, data: {...} }`，最多剥 3 层
- **诊断兜底**：完成但提不出图时 `console.warn` 完整 payload + toast 提示「按 F12 看 console」，方便定位 API 字段变化

> 修复是猜测性的：如果还是看不到图，请打开 F12 控制台，截 `[applyTaskUpdate] completed 但无法解析出图片` 后面的 payload 给我，我对照实际字段精修。

## v0.12.0 — 2026-05-15 · Prompt 库（107 条爆款 prompt + 一键填入）

**功能跃迁版本**：从「会用 API」升级到「有创作社区背书」。新增「📚 Prompt 库」独立 tab，集成 [`ApiMartAI/best-gpt-image-2-prompts`](https://github.com/ApiMartAI/best-gpt-image-2-prompts)（107 条来自 Twitter 的真实爆款 prompt，带生成结果预览 + 互动数据）。

- **数据**：运行时从 jsdelivr CDN 拉取 raw JSON（约 70 KB）；localStorage 缓存 7 天；图片来自 Twitter CDN（pbs.twimg.com）
- **筛选**：
  - 搜索（prompt 内容 / 作者名）
  - 语言（zh / en / ja / ko / 其他）
  - 排序（点赞数 / 浏览量 / 转发数 / 最新 / 最早）
  - 只看有参考图 / 只看 ❤️ 收藏
- **网格**：响应式 1 / 2 / 3 列；卡片显示首张结果图 + prompt 摘要（120 字截断）+ 作者 + 点赞数 + 语言徽章 + 多图 `+N` 标记 + 收藏 ❤️ 角标；图片懒加载
- **详情 Modal**：完整 prompt（可滚动 + 可复制）+ 所有结果图横向滚动浏览 + 作者完整信息（粉丝 / 日期 / 语言 / 4 类互动数据 / 媒体数）+ 原帖链接
- **一键载入**：`✨ 用此 prompt 生成` 自动切到生成 tab、填 prompt 框、根据首张图宽高匹配最接近的比例预设
- **收藏**：`❤️` 按钮，存 localStorage（与缓存独立）；左侧筛选可只看收藏
- **维护**：`↻ 刷新` 强制重拉；`清缓存` 一键清空（含收藏）
- **快捷键**：Esc 关闭 Modal

> 实施细节见 `PROMPT_LIBRARY_PLAN.md`。下一步候选：找类似 prompt（embedding 相似度）、模板提取、按需翻译。

## v0.11.3 — 2026-05-15 · 品牌化：改名 + 新 logo + 三份配套 MD

- **改名**：平台名「GPT-Image-2 工作台」→「**Ling's GPT-Image-2 Studio**」
- **slogan**：去掉「APIMart」，改为「纯前端单页 · 数据只在你本地 · AI 智能选区」
- **logo**：左上角换成用户提供的卡通图（54KB JPG → 73KB base64 嵌入 HTML，保持单文件零依赖）
- **favicon**：浏览器标签页图标同步替换为新 logo
- **配套文档**（独立 MD 留底，不在 HTML 里）：
  - `PROJECT_STATUS.md` —— v0.7-v0.11.3 演进、能力清单、技术栈、限制、下一步
  - `PUBLISH_AND_SECURITY.md` —— 发布路径选择、5 个安全风险（API Key 泄漏最严重）、修复清单
  - `PROMPT_LIBRARY_PLAN.md` —— `ApiMartAI/best-gpt-image-2-prompts` 仓库集成方案（v0.12.0 候选）

## v0.11.2 — 2026-05-15 · 坐姿 / 遮挡场景修复 + 头部改用 FaceDetector

实测 6 人室内坐姿图：v0.11.1 只识别到 2 人头部 + 0 手 + 0 脚。原因是 Pose 模型对**坐姿、上半身互相遮挡**的人检出率低，HandLandmarker 对**多人小手**、脚被茶几挡住的情况完全失败。本补丁重点修这类场景：

- **头部改用 FaceDetector**（MediaPipe `blaze_face_short_range`）：多脸 box → 椭圆 mask（包住头发 / 下巴）。比 Pose 的 0-10 关键点子集准很多，多人坐姿能识别 **10+ 人**。Pose 仍作为 fallback
- **阈值放宽**：
  - Pose / Hand / Face 的 `minDetectionConfidence` 全部降到 **0.3**（默认 0.5）
  - Pose / Hand `minPresenceConfidence` 0.3
  - 关键点可见性阈值 0.3 → **0.1**（坐姿 / 半遮挡场景能识别）
- **numPoses: 5→10，numHands: 6→10**，最多覆盖 10 人
- **手部新增 fallback 路径**：HandLandmarker 检测不到时，改用 Pose 的腕/食指/小指/拇指 **4 个关键点（每手）**做凸包，膨胀放大补点少
- **脚部新增 fallback 路径**：6 个脚关键点全被遮挡（坐姿 / 桌面下）时，用**脚踝+膝盖**估算位置 — 沿小腿方向延伸 30%，画一个朝向小腿方向的椭圆作为近似
- **可观测**：toast 现在会标注「`pose fallback`」「`N 只估算`」「`(face)`」等标签，让你知道走的哪条路径

> 注：v0.11.1 你看到的「人物 (2 人, pose)」也会受这波阈值放宽改善，但实测 Pose 模型对完全背对镜头的人仍然检测不到 — 那种情况只能用 selfie segmenter 的整图分割（已经是 fallback 路径）。

## v0.11.1 — 2026-05-15 · 快速选区多人场景修复

实测发现 v0.11.0 在多人图上：「头部 / 双脚」只识别 1 人；「人物」把人和人之间的物体也圈进遮罩。本补丁修复：

- **修复（多人头/脚）**：`PoseLandmarker` 的 `numPoses` 从 1 升到 5；模型从 `pose_landmarker_lite`（5 MB）换成 `pose_landmarker_full`（9 MB），多人精度提升明显
- **修复（关键 bug：中间物体被圈）**：原 `segmentByPose` 把所有人的关键点合并取**一个大凸包**，导致两人之间的桌椅、空气被一起圈住。改为「**每人独立画凸包合并到同一 mask**」，不再串扰
- **修复（多人手）**：`HandLandmarker` 的 `numHands` 从 2 升到 6，支持最多 3 人
- **改进（人物 / 背景）**：原 Selfie Segmenter 多人场景轮廓粗、人与人之间的空隙也被当前景。改为优先用 `PoseLandmarker.segmentationMasks` —— 每人独立分割 mask，OR 合并；失败时 fallback 到 Selfie Segmenter
- **可观测**：toast 提示会显示「N 人 (pose)」或「(selfie)」让你知道实际走的哪条路径，方便排查

## v0.11.0 — 2026-05-15 · 快速选区（纯前端 MediaPipe，AI 智能化第一步）

- **新增**：左侧工具区「✨ 快速选区」面板 — 一键生成 5 类常见遮罩
  - `👤 人物`：MediaPipe Selfie Segmenter（准确率高）
  - `🌄 背景`：上面的反色版本
  - `🙂 头部`：MediaPipe Pose 面部 11 个关键点凸包 + 膨胀
  - `✋ 双手`：MediaPipe Hands 21 关键点凸包 + 膨胀（精度好）
  - `🦶 双脚`：MediaPipe Pose 脚踝/脚跟/脚趾 6 个关键点凸包（粗略，需精修 — 前端没有专门的脚分割模型）
- **技术**：纯前端 `@mediapipe/tasks-vision@0.10.14`（jsdelivr CDN），图片不离开本机
- **懒加载**：首次点击对应按钮时才下载模型（5–15 MB），之后
