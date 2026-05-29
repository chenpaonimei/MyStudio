# Cloudflare Worker 反代部署教程（中文界面 · 小白版）

> 解决什么问题：APIMart 的 official 渠道被 Cloudflare bot 防护拦截，浏览器直连失败。
> 用 Worker 中转：浏览器 → 你的 Worker → APIMart 后端。Worker 是服务器调用不触发 challenge，且自动加 CORS 头。
> 全程**网页操作**，不用命令行
> 预计用时：**10 分钟**
> 费用：**完全免费**（Cloudflare Workers 免费档每天 10 万次请求，足够个人 + 5-10 朋友用）

> 📌 本教程按 **Cloudflare 中文界面** 写。每个按钮 / 菜单名后面括号里是英文原名，万一翻译有出入按英文找。

---

## Step 1：注册 Cloudflare 账号（如已有账号跳过）

1. 浏览器打开 https://dash.cloudflare.com/sign-up
2. 邮箱 + 密码注册
3. 验证邮箱（收件箱点 confirm 链接）

---

## Step 2：进入 Workers 控制台

1. 登录后访问 https://dash.cloudflare.com
2. **左侧菜单**找 **「计算 (Workers)」** 或 **「Workers 和 Pages」**（Workers & Pages）点开
   - 新版界面可能在 **「计算 (Compute)」** 分组下
   - 老版本可能直接叫 **「Workers」**
3. **第一次进会让你选一个 subdomain（子域名前缀）**
   - 中文界面提示：**「选择您的子域」** 或 **「Choose your subdomain」**
   - 例如填 `chenpaonimei` → 你以后的 Worker URL 就是 `xxx.chenpaonimei.workers.dev`
   - 这个名字**全网唯一**，被占就换一个
   - 点 **「设置 (Set up)」**

---

## Step 3：创建 Worker

1. 进入「Workers 和 Pages」页面，右上角点蓝色按钮 **「创建 (Create)」**
2. 选 **「Workers」** 选项卡（不是「Pages」）
3. 点 **「创建 Worker (Create Worker)」**
4. 给 Worker 起名字：例如 `apimart-proxy`
   - 中文界面提示：**「Worker 名称」** 或 **「Name your Worker」**
   - 这个名字会变成 Worker URL 的一部分，**全 subdomain 内唯一**，被占就换
5. 其他选项保持默认，直接点底部蓝色按钮 **「部署 (Deploy)」**
   - 这一步先用 Cloudflare 自带的 Hello World 代码部署一下占位
6. 部署成功后会看到 **「继续处理项目 (Continue to project)」** 或 **「编辑代码 (Edit code)」** 按钮

---

## Step 4：替换 Worker 代码

1. 上一步结束后点 **「编辑代码 (Edit code)」**
   - 如果跳过了，可以从「Workers 和 Pages」列表里找到 `apimart-proxy` → 右上角 **「编辑代码」**
2. 进入在线代码编辑器，左侧文件树有 `worker.js`（约 8 行 Hello World 代码）
3. 鼠标点到代码区，按 **Ctrl+A 全选**，再按 **Delete 删除**（清空）
4. 打开 outputs 文件夹的 [`apimart-cors-proxy.worker.js`](./apimart-cors-proxy.worker.js)，**Ctrl+A 全选 + Ctrl+C 复制**
5. 回到 Cloudflare 编辑器，**Ctrl+V 粘贴**
6. 右上角点蓝色按钮 **「部署 (Deploy)」**
7. 弹出确认框 → 点 **「保存并部署 (Save and Deploy)」**

---

## Step 5：拿到 Worker URL 并验证

1. 部署完成后页面顶部 / 右侧会显示 Worker URL：
   ```
   https://apimart-proxy.<你的-subdomain>.workers.dev
   ```
   例如：`https://apimart-proxy.chenpaonimei.workers.dev`

   也可以在「Workers 和 Pages」列表里找 `apimart-proxy`，旁边会显示这个 URL

2. **复制这个 URL，在浏览器新标签页打开它验证**：
   - 直接访问 → 应该看到一个 JSON：
     ```json
     {
       "ok": true,
       "proxy": "apimart-cors-proxy",
       "upstream": "https://api.apimart.ai",
       "usage": "Use this Worker URL + /v1/... as Base URL in your client"
     }
     ```
   - 看到这个 = Worker 跑起来了 ✅
   - 看到 `Error 1101` / `Worker threw exception` / 一片空白 = 脚本没粘干净，回 Step 4 重新粘 + Deploy

---

## Step 6：在 Studio 里改 Base URL

1. 打开你的 Studio（本地双击 HTML 或 GitHub Pages 链接）
2. 点右上角 **「⚙️ 设置」**
3. 找到 **「Base URL」** 输入框，把：
   ```
   https://api.apimart.ai/v1
   ```
   改成：
   ```
   https://apimart-proxy.<你的-subdomain>.workers.dev/v1
   ```
   ⚠️ **末尾的 `/v1` 必须保留**（APIMart 所有 endpoint 都在 `/v1/...` 下）

4. 输入框边框**不能是红色**。如果变红了说明 HTTPS 校验不通过 —— 检查 URL 开头是不是 `https://`

5. 试着提交一个 official 任务 —— 应该不再被 CORS 拦截 🎉

---

## Step 7：（可选）锁定只允许自己用

默认 Worker 允许任何人调用，理论上别人拿到你的 URL 可以用它当自己的 CORS 代理（消耗你的免费额度）。

⚠️ 注意：别人**还是需要他们自己的 APIMart Key** 才能成功调用，不会动你的钱包，但会占你 Cloudflare 免费额度。

想限制只允许自己的设备：

1. 回 Cloudflare → **「Workers 和 Pages」** → `apimart-proxy` → **「编辑代码 (Edit code)」**
2. 找到这一段（约第 30-32 行）：
   ```js
   const ALLOWED_ORIGINS = '*';
   ```
3. 改成：
   ```js
   const ALLOWED_ORIGINS = [
     'https://chenpaonimei.github.io',  // GitHub Pages 部署版
     'http://localhost:8080',           // 本地开发
     'null'                              // 本地双击 HTML（origin 是 null）
   ];
   ```
4. 右上 **「部署 (Deploy)」** → **「保存并部署」**

---

## 常见问题（FAQ）

### Q：找不到「Workers 和 Pages」菜单？
A：Cloudflare 改版几次，可能在：
- 左侧侧栏「计算 (Compute)」分组下
- 左侧侧栏直接的「Workers 和 Pages」
- 顶部账户名旁边
- 实在找不到：浏览器直接打开 https://dash.cloudflare.com/?to=/:account/workers-and-pages

### Q：第 2 步没让我选 subdomain，直接进了 Workers 列表？
A：说明你之前用过 Cloudflare 已经设过了。直接进 Step 3 创建 Worker，subdomain 默认用之前那个。

### Q：第 5 步打开 Worker URL 看到 `Error 1101` / `Worker exceeded CPU` / 空白页？
A：脚本粘贴时漏了某段。回 **「编辑代码」**，再把 [`apimart-cors-proxy.worker.js`](./apimart-cors-proxy.worker.js) 完整内容**重新全选粘贴一次**，**「保存并部署」**。

### Q：第 6 步改了 Base URL 还是 CORS 报错？
A：检查清单：
- [ ] Base URL 末尾有没有 `/v1`？
- [ ] Worker URL 拼对了？回 Cloudflare 看 Worker 列表里那个完整 URL
- [ ] 浏览器有缓存 → **Ctrl+Shift+R 强制刷新**
- [ ] 设置面板看输入框边框是不是红色？红色说明你 URL 格式错（必须 https://）
- [ ] F12 → Network → 提交一个任务 → 看 `generations` 那行的 Request URL，应该是 `https://xxx.workers.dev/v1/...`，不是 `https://api.apimart.ai/v1/...`

### Q：怎么知道请求真的走了 Worker？
A：F12 → **「网络 (Network)」** → 提交一个任务 → 找到 `generations` 那一行：
- ✅ 走 Worker：**Request URL** 显示 `https://xxx.workers.dev/v1/...`
- ❌ 还在直连：Request URL 显示 `https://api.apimart.ai/v1/...`（说明 Base URL 没改对）

### Q：Worker 免费额度够吗？
A：Cloudflare 免费档每天 **10 万次** Worker 请求。一次图像生成大概包含 1-3 次 API 调用（提交 + 轮询 + 单查询）。每天能跑 3-10 万张图，完全够个人 + 朋友用。

### Q：Worker 会看到我的 API Key 吗？
A：技术上 Cloudflare 边缘节点能看到 `Authorization` header（HTTPS 在边缘解密再加密发到上游）。但：
- 这个 Worker 脚本**不打任何日志**，Cloudflare 默认也不存请求体
- Cloudflare 自身有商业信誉
- 上游 APIMart 同样能看到 Key（这是 API 调用本身的事）
- 如果完全不想任何第三方触碰 → 自建反代服务器（VPS + Nginx），但成本和复杂度高很多

### Q：换电脑 / 重装浏览器后 Worker 还在吗？
A：在。Worker 部署在 Cloudflare 服务器上，跟你本地无关。只要你不去 Cloudflare 后台主动删，它一直运行。

### Q：想停掉 Worker 怎么办？
A：Cloudflare → **「Workers 和 Pages」** → 找到 `apimart-proxy` → 右侧 **「⋯」** → **「删除 (Delete)」**

### Q：左侧菜单没有「计算」也没有「Workers 和 Pages」，但有「Compute」/「Functions」？
A：Cloudflare 不同账号 / 不同时期界面有差异。**直接用 URL 进**：
- https://dash.cloudflare.com/?to=/:account/workers-and-pages
- 或 https://workers.cloudflare.com/

---

## 给朋友用

朋友也想用反代？两种方式：

- **共享你的 Worker**：把 Worker URL 给朋友，让他们也在 Studio 的 Base URL 里填这个地址
  - 简单
  - 消耗你的免费额度（10 万次/天，朋友少没问题）
  - 朋友还是要用自己的 APIMart Key

- **朋友自己部署一份**：把 `WORKER_SETUP.md` + `apimart-cors-proxy.worker.js` 发给朋友，让他们各自走 Step 1-6
  - 每人自己 10 万次额度
  - 互不影响

我建议第二种 —— 每人自己一份 Worker。

---

## 部署完后告诉我

部署完后给我：
- Worker URL（去掉 subdomain 也行，比如 `https://apimart-proxy.****.workers.dev`）
- Studio 里改完 Base URL 后能否正常跑 official 任务

如果有报错截 **F12 → Console** 给我，我帮诊断。


---

## v0.17.6 新增：视频/音频上传代理 `/upload`

### 这一步解决什么

如果你是**直接双击本地 HTML** 打开 Studio，浏览器来源是 `file://`，catbox.moe 会因为 CORS 拒绝视频/音频上传。v0.17.6 在同一个 Cloudflare Worker 里新增了 `/upload`：

```
Studio(file:// 或 GitHub Pages) → 你的 Worker /upload → catbox.moe
```

这样浏览器只和你自己的 Worker 通信，Worker 再服务器侧上传到 catbox，绕开 null origin 的限制。

### 你需要做的事

1. 打开 Cloudflare → Workers 和 Pages → 进入你之前创建的 `apimart-proxy` Worker。
2. 点击右上角 **编辑代码 / Edit code**。
3. 回到本地文件夹，打开 `apimart-cors-proxy.worker.js`。
4. 全选 Worker 编辑器里的旧代码，粘贴这个文件的完整新代码。
5. 点击 **Deploy / 部署**。
6. 部署完成后，浏览器打开：

```
https://你的-worker.workers.dev/health
```

能看到 JSON 说明 Worker 正常。

7. Studio 设置里的 Base URL 仍然填：

```
https://你的-worker.workers.dev/v1
```

不要填 `/upload`。Studio 会自动把视频/音频上传发到同域：

```
https://你的-worker.workers.dev/upload
```

### 怎么确认 `/upload` 生效

- 不配 Pixeldrain Key。
- 本地双击打开 `gpt-image-2-studio.html`。
- 视频 tab 上传一个小视频或音频。
- 如果上传框显示 catbox 返回的 URL，说明 Worker `/upload` 已接管。
- 如果看到「Worker /upload 未部署，请重部署」，说明你还在用旧 Worker：旧 Worker 会把 `/upload` 代理到 APIMart，返回一整段 HTML 首页。重新按上面步骤粘贴新版 `apimart-cors-proxy.worker.js` 并 Deploy。
- 如果 Worker 网络错误，前端会尝试回退到直连 catbox；`file://` 下仍可能看到 CORS 警告。

