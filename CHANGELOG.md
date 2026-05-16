# Ling's GPT-Image-2 Studio · 更新日志

> 项目：`gpt-image-2-studio.html`（基于 APIMart 网关的 GPT-Image-2 / GPT-Image-2-Official 图像生成工作台）
>
> 版本号方案：`主.次.补丁`（语义化），破坏性变更 +主；新功能 +次；修复或小补丁 +补丁。
>
> 关联接口文档：[image2-generate-apimart-v5.md](./image2-generate-apimart-v5.md)

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

`https://chenpaonimei.github.io/My-GPT-Image-2-Studio/gpt-image-2-studio.html`

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
- **懒加载**：首次点击对应按钮时才下载模型（5–15 MB），之后秒开
- **兜底**：CDN 不可达 → toast 提示，按钮可重试；其他按钮独立加载，互不影响
- **合并策略**：AI 结果默认合并到现有遮罩（OR），可撤销 / 重做
- **状态条**：面板上方显示「未加载 / 已就绪 / 运行中… / 失败可重试」
- **配套留底**：新增 `MASK_AI_ASSESSMENT.md` 评估报告，详述 v0.12.0+ 后续路线（含 head/clothing 多类分割、Grounded-SAM 文字驱动等）

## v0.10.0 — 2026-05-15 · Mask 编辑器：拖拽载入 + 清除参考图

- **新增**：拖拽载入 — 把本地图片或网页图片直接拖到画布即可载入；拖入时画布上叠蓝色蒙层 + 虚线提示框
- **新增**：左侧工具区「🗑 清除参考图」按钮（独立于主页面参考图，仅清编辑器内的图）
- **兼容**：拖入 `text/uri-list` / `text/plain` 中的 http(s) 或 data:image URL 也会被识别（来自其他网页拖图）
- **保护**：拖入新图替换、清除参考图，都会先检查当前是否已有涂抹，有则 confirm
- **文案**：空态从「⬅ 在左侧上传」改为更醒目的「拖到这里 / 上传 / 用主页参考图」三选一图标

## v0.9.0 — 2026-05-15 · Mask 编辑器缩放（Photopea / SD WebUI 风格）

- **新增**：Mask 编辑器画布支持自由缩放，范围 10% – 1000%
- **新增**：浮动工具栏（画布右下角胶囊）— `−` / 缩放百分比 / `+` / `⛶ 适应` / `1:1`，点击百分比即恢复 100%
- **新增**：`Ctrl/⌘ + 滚轮` 以光标位置为中心放大缩小（屏幕锚点保持不动，符合 Photopea 体感）
- **新增**：键盘快捷键 — `Ctrl/⌘ +=` 放大、`Ctrl/⌘ + -` 缩小、`Ctrl/⌘ + 0` 适应窗口、`Ctrl/⌘ + 1` 100% 实际像素（拦截浏览器默认页面缩放）
- **增强**：放大后通过 wrap 自带滚动条平移；不带 Ctrl/⌘ 的普通滚轮仍走默认滚动行为，方便平移
- **增强**：光标圆圈线宽根据缩放自适应（始终 ~1.5 屏幕像素），笔刷大小提示与画布像素一致
- **增强**：画布信息条实时显示当前缩放百分比（替代旧的「显示 N%」一次性文本）
- **重构**：把 `loadImage` 中临时的「按 wrap 适配」逻辑统一到 `fitToWindow()`，后续缩放、初次适配走同一路径

## v0.8.0 — 2026-05-14 · 预览图修复 + 链接显示 + 选择性下载 + Mask Alpha 校验

- **Bug 修复**：任务卡图片预览不显示的 bug — 原因是 `sm:grid-cols-${N}` 这种插值类名 Tailwind CDN 的 JIT 编译不出来（特别是后注入的 DOM），改成静态 switch 枚举类
- **新增**：每张生成图下方显示完整 URL（只读 input，点击全选），右侧有单独「📋」复制按钮
- **新增**：每张生成图左上角加复选框，可跨任务选中
- **新增**：批量操作栏新增「⬇️ 下载选中 (N)」按钮，实时显示选中数；「下载全部」保留并改名
- **新增**：批量操作栏在有选中时显示「×」清除选中按钮
- **增强**：Mask 编辑器导出 / 保存到主页面时，自动统计透明像素数；toast 给「✓ Alpha 通道有效（N px 透明）」或「⚠ 没有透明像素」提示
- **增强**：Mask 预览区也会显示 Alpha 校验结果（绿色对勾或琥珀色警告）
- **增强**：图片角上的复制 / 下载按钮改为始终可见（之前 hover 显示，触屏不可见）
- **增强**：getContext('2d', { alpha: true }) 显式声明，对应文档「Alpha 通道为是」的要求

## v0.7.0 — 2026-05-13 · 批量轮询 + URL 倒计时 + ETA

- 新增 `POST /v1/tasks/batch` 批量轮询：所有进行中任务合并为一次请求，并发场景大幅减少请求数与 429 风险
- 接口不可用 / 返回格式不符时，自动回退到 `GET /v1/tasks/{id}` 单任务轮询，并 toast 提示
- 已完成任务卡显示 URL 失效倒计时（24h 边界，< 6h 琥珀色，< 1h 红色，过期红色加粗）
- 进行中任务进度条下显示「预计 / 剩余」秒数（基于 `estimated_time`）
- 每分钟自动重渲一次已完成任务，保持倒计时准确

## v0.6.0 — 2026-05-13 · 独立滚动 + 任务折叠 + Mask 编辑器

- 右侧任务队列独立滚动条（lg 屏幕下），bulk 操作栏置顶
- 每个任务可折叠 / 展开，折叠态显示首图缩略图 + 多图 `+N` 计数
- 新增「展开全部 / 折叠全部」批量按钮
- 新增 🎨 Mask 编辑器独立 tab，基于 canvas 实现
  - 涂抹 / 擦除 两种工具
  - 笔刷大小（2-400px）、硬度（10-100%）、遮罩预览强度
  - 撤销 / 重做（最多 30 步历史）
  - 清空 / 反转 / 全部涂抹
  - 快捷键：`B` / `E` 切工具，`[` `]` 调大小，`Ctrl/⌘+Z` 撤销，`Ctrl/⌘+Shift+Z` 重做
- Mask 导出符合 API 契约：涂抹区透明、未涂区白色不透明
- 「用作此次生成」一键写入主页面，自动切到 official 模型；若主页面无参考图自动补上编辑器源图

## v0.5.0 — 2026-05-12 · 任务级重试覆盖 + 清空关闭全局重试

- 失败 / 等待重试的任务可单独配置重试规则，覆盖全局设置
- 任务级 Modal：模式 / 次数 / 时长 / 间隔 / 跳过 4xx / 立即停止重试
- 卡片头部显示紫色 `⚙ 重试覆盖` 徽章
- 清空任务时同步关闭全局自动重试，避免下次提交不小心触发

## v0.4.0 — 2026-05-11 · 余额展示 + 成本估算

- 顶部实时显示用户余额 + Key 余额，颜色随余额变化
- 自动刷新触发点：页面加载、API Key 改动、任务提交后、任务完成后（均防抖）
- 提交按钮上方显示预估成本（任务数 × n × 单价），超出余额会弹确认框
- 价格表（参考值 / 实测均值），任务完成后从 `data.cost` 自动学习单价
- 设置面板新增可折叠「价格表」面板 + 重置实测均值按钮

## v0.3.1 — 2026-05-11 · 内容审核失败也允许重试

- 去掉对 `content_policy` / `safety` / `moderation` 关键词的硬拦
- 这类失败按所选模式正常重试（实测多试几次往往能过）
- 同步更新接口文档 §8.3 重试策略表

## v0.3.0 — 2026-05-10 · 自动重试 + Mask 本地上传

- 失败自动重试：关闭 / 按次数 / 按时长 / 直到成功 四种模式
- 重试间隔可设，重试任务和新提交共用同一并发上限
- Mask 字段支持本地文件上传：自动 base64 嵌入、缩略图预览、尺寸校验
- 与第 1 张参考图尺寸不一致会标红警告
- Mask 内联使用说明（折叠面板，6 条要点）

## v0.2.0 — 2026-05-10 · 具体像素选择器

- 比例 + 分辨率下面新增「具体像素」选择器
- 按 1K / 2K / 4K 分组列出预设值（1:1 / 16:9 / 9:16 / 3:4 / 4:3 / 9:21 / 21:9）
- 支持自定义宽高输入，超长边（>3840）或总像素（>8.29MP）自动校验拦截
- 直接以 `size: "WxH"` 调用 API 的高级用法

## v0.1.0 — 2026-05-10 · 初始版本

- 基础生成表单：模型（gpt-image-2 / official）/ prompt / 比例 / 分辨率 / 参考图
- 13 种比例 + 1K / 2K / 4K 分辨率档位
- 参考图最多 16 张，支持本地拖拽 / 点击 / 粘贴 / URL
- 任务队列 + 自动轮询（首次 8s，间隔 4s，硬超时 8 分钟）
- 失败原因展示（code + message + type）
- 批量下载（每任务 ZIP / 全部任务总 ZIP）
- API Key 本地保存（localStorage）+ 并发上限可调
- 单 HTML 文件，零依赖（除 Tailwind + JSZip via CDN）

---

## 版本号策略

- **主版本（0.x → 1.x）**：当本工具发布到公开域名或与他人共享时。
- **次版本（0.6 → 0.7）**：新增一组功能（如新 tab、新 API 端点）。
- **补丁（0.3.0 → 0.3.1）**：行为修正或文档更新，无新功能。

## 文件清单

| 文件 | 用途 |
|------|------|
| `gpt-image-2-studio.html` | 主程序，双击在浏览器打开即可使用 |
| `CHANGELOG.md` | 本文件 |
| `image2-generate-apimart-v5.md` | 接口契约文档，给其他 LLM 读了即可调用 |
