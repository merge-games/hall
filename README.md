# 游戏大厅

一个静态首页，把做的所有小游戏列成卡片，一个链接分享全部。

- 线上地址：**https://merge-games.github.io/hall/**
- 想加/改游戏：编辑 `index.html` 顶部的 `GAMES` 数组即可（标题 / 简介 / 链接 / 封面 / 标签），保存刷新就能看到。
- 不想改代码：用内置的**隐藏管理员面板**（见下文），点点鼠标就能改清单、改标题、一键发布。

```js
var GAMES = [
  {
    title: "合成杨璐恺",
    desc: "同学头像合成小游戏：两个相同头像碰到一起就升级，合出最大级触发全屏庆祝。",
    url: "https://merge-games.github.io/merge-yanglukai/",   // 游戏地址，也可以是本地相对路径
    cover: "https://merge-games.github.io/merge-yanglukai/assets/level_11.jpg",  // 可留空
    tag: "合成"                                              // 右上角小标签，可留空
  }
];
```

## 本地预览

直接双击 `index.html` 即可（纯静态、无依赖）；也可以用 `python -m http.server 8000` 起服务。

## 部署

```bash
git add -A && git commit -m "更新大厅"
git push
```

已带 `.nojekyll`，Pages 的 Source 选 `main` / `(root)` 就不会卡 Jekyll 构建。

---

## 隐藏管理员面板

面板默认**完全不渲染**（不开面板时页面 DOM、样式与普通访客看到的完全一致），需要手动触发。

### 怎么打开

| 方式 | 操作 |
|---|---|
| 网址参数 | 打开 `.../hall/?admin=1` 或 `.../hall/index.html?admin=1` |
| Hash | 打开 `.../hall/#admin` |
| 连点标题 | **2 秒内连续点击页面标题「游戏大厅」5 次** |

打开后地址栏里的 `?admin=1` / `#admin` 会自动清掉（不影响已打开的面板）。点面板右上角**「退出管理」**或按 `Esc` 即可关闭；关闭后注入的样式和诊断属性也会一并移除，页面恢复成普通访客看到的样子。

### 面板功能

**A · 游戏清单** — 列出 `GAMES` 的全部条目，每条都能改标题 / 简介 / 链接 / 封面 / 标签，并支持新增、删除、上移、下移。改动**实时反映到下方页面卡片**（所见即所得）。标题为空、链接为空或链接格式不对（必须以 `http` / `https` / `/` / 相对路径开头）会红字提示，并且**不会**把非法内容写进页面。

**B · 大厅设置** — 改标题 `HALL_TITLE`、副标题 `HALL_SUB`、底部纪念文字，和清单一并保存/发布。

**C · 保存方式（三种）**

1. **复制 GAMES 代码** — 把当前清单生成 `var GAMES = [ … ];` 整段复制到剪贴板，自己贴回文件即可。
2. **下载 index.html** — 生成一份改好的整页文件直接下载，替换仓库里的同名文件。
3. **发布到 GitHub** — 用 GitHub Token 一键提交：先 `GET /repos/merge-games/hall/contents/index.html?ref=main` 取当前 `sha`，再 `PUT` 同路径提交 `{message, content(base64), sha, branch:"main"}`。成功后显示本次提交链接。失败会区分原因：
   - `401` Token 无效/过期
   - `403` Token 无写权限或触发限流
   - `404` 仓库/路径不对，或 Token 看不到这个仓库（私有仓库需单独授权）
   - `409` 分支状态冲突（`sha` 过期）→ **自动重新拉取最新 `sha` 再重试一次**，仍失败会提示重新读取
   - `422` 提交被拒绝

   下载与发布前都会做一次**产物自检**（doctype、GAMES 块、写入标记数量、是否混进面板 DOM），任何一项不过就直接中止，避免把页面写坏。

**D · 源码编辑器** — 仓库下拉（`hall` / `merge-yanglukai` / `jump-yanglukai` / `fruit-yanglukai` / `galgame-yanglukai` / `game-kit`）+ 预置文件下拉（默认给出各仓库的 `index.html`），也可以手填任意路径（例如 `assets/main.js`、`README.md`）。点「读取文件」把内容拉进编辑器，改完点「提交到 GitHub」。提交前会显示**改动行数摘要**（新增 / 删除多少行、总行数变化）并弹**二次确认**。另有：

- **复制当前内容（备份）** — 提交前先把原文复制一份留底
- **撤销未保存的修改** — 还原成最近一次读取/提交的内容
- 路径不存在时可以直接写内容提交，等于**新建文件**

**E · 数据看板** — 同源读取各小游戏写在本机的 `localStorage` 成绩（只读）：

| 游戏 | 读取的键 |
|---|---|
| 合成杨璐恺 | `merge-yanglukai:best:v2`、`merge-yanglukai:codex:v1` |
| 跳一跳 · 璐恺版 | `jump-yanglukai:best:v1`、`jump-yanglukai:chai:v1` |
| 切西瓜 · 璐恺版 | `fruit-yanglukai:best:v1`、`fruit-yanglukai:progress:v1` |
| 璐恺十二钗 | `galgame-yanglukai:ends:v1`、`galgame-yanglukai:save:v1` |

读不到的项显示「—」。**用 `file://` 双击打开时读不到**：浏览器把每个本地文件当成独立源，大厅页面访问不到各游戏页面写下的 localStorage（同源限制）。把大厅放到线上（`https://merge-games.github.io/hall/`）再打开面板才能看到真实数据。

### Token 权限建议

推荐用 **fine-grained personal access token**（GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens）：

- **Repository access**：只选 `merge-games` 下的这几个仓库（不要选 All repositories）
- **Permissions → Repository permissions → Contents: Read and write**（这是提交文件唯一需要的权限，其余全部留 No access）
- **Expiration**：设一个较短的有效期（如 30 天），到期重新生成

经典 token（`ghp_…`）也能用，但至少要勾 `repo` 范围，权限比 fine-grained 大，不推荐。

### 安全提示

- Token **默认只存在内存里**，刷新/关页面就没了。
- 只有勾选「**记住到本机（localStorage）**」才会把 Token 明文写进浏览器 `localStorage`，键名 `hall-admin:token:v1`，同一个站点下的任何脚本都能读到 —— **公共电脑 / 共享浏览器上不要勾**。
- Token 只会发给 `api.github.com`，页面本身不向其它域名发任何请求。
- 怀疑泄露：立刻去 GitHub 把该 Token 删除（Delete/Revoke），再重新生成一个。
- 面板入口是「隐藏」不是「加密」：任何知道 `?admin=1` 的人都能打开编辑器，但**没有 Token 就只能看，不能发布**。

## 相关仓库

| 仓库 | 内容 |
|---|---|
| [merge-yanglukai](https://github.com/merge-games/merge-yanglukai) | 合成杨璐恺（游戏本体） |
| [jump-yanglukai](https://github.com/merge-games/jump-yanglukai) | 跳一跳 · 璐恺版 |
| [fruit-yanglukai](https://github.com/merge-games/fruit-yanglukai) | 切西瓜 · 璐恺版 |
| [galgame-yanglukai](https://github.com/merge-games/galgame-yanglukai) | 璐恺十二钗 |
| [hall](https://github.com/merge-games/hall) | 本仓库（游戏大厅） |
