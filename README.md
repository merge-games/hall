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
    tag: "合成",                                             // 右上角小标签，可留空
    on: true                                                 // 是否在大厅显示；不写 = 显示
  }
];
```

| 字段 | 含义 | 缺省 |
|---|---|---|
| `title` | 游戏名 | 必填 |
| `desc` | 一句话简介 | 空 |
| `url` | 游戏地址（`https://…` 或相对路径） | 必填 |
| `cover` | 封面图，留空用默认渐变底 | 空 |
| `tag` | 右上角小标签 | 空 |
| `on` | `true` 显示 / `false` 在大厅里隐藏 | **不写 = `true`（显示）** |

> `on: false` 只是**不在大厅里露出这张卡片**，游戏页面本身还在（知道链接的人照样能打开）。想彻底下线请删掉这一条或删掉对应仓库。


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

面板默认**完全不渲染**（不开面板时页面 DOM、样式与普通访客看到的完全一致），触发后也要先过一道**管理员密码门**。

### 怎么打开

| 方式 | 操作 |
|---|---|
| 网址参数 | 打开 `.../hall/?admin=1` 或 `.../hall/index.html?admin=1` |
| Hash | 打开 `.../hall/#admin` |
| 连点标题 | **2 秒内连续点击页面标题「游戏大厅」5 次** |

三种方式都只会**先弹出密码输入框**（页面内的输入框 + 「进入管理面板」按钮，不会用浏览器那个丑陋的 `window.prompt`）。密码对了才构建管理面板；输了：
- 立刻红字提示，**输入框自动清空**；
- **连续输错 3 次 → 冷却 10 秒**，期间输入框禁用、按钮上直接显示「冷却中 N 秒」倒计时，即使这时候输对密码也进不去；10 秒过后机会重置。

打开后地址栏里的 `?admin=1` / `#admin` 会自动清掉（不影响已打开的面板）。点面板右上角**「退出管理」**或按 `Esc` 即可关闭；关闭后注入的样式和诊断属性也会一并移除，页面恢复成普通访客看到的样子。

### 管理员密码

口令**不存明文**，源码里只有两个常量（在文件偏上位置，`GAMES` 数组之前）：

```js
var ADMIN_SALT = "74743da0ee0c";          // 随机盐（12 位十六进制）
var ADMIN_PASS_HASH = "a930eb04b461c6c8"; // hallPassHash(密码) 的结果，16 位十六进制
```

- **默认密码：`banzhongwang`** —— 第一次进面板后请立刻改掉（面板 **F · 安全设置**，或下面的手工改法）。
- 哈希算法：`hallPassHash(密码, 盐)`，**纯 JS 手写**，不依赖 `crypto.subtle`（`file://` 双击打开时浏览器不提供它）：
  1. 取字符串 `盐 + "|" + 密码`；
  2. 对它同时算 **FNV-1a(32)** 和 **djb2(32)** 两个哈希；
  3. 把这两个哈希的十六进制拼起来当作下一轮的盐，**重复 3 轮**（每轮字符值额外 `+ 131×轮次`）；
  4. 最后输出 `h1` 与 `h2` 各 8 位十六进制，拼成 16 位，全小写。

  > 定位说明：这是**挡住随手翻源码的人**的轻量校验，不是强加密。源码是公开的，真想进来的人可以离线爆破；它保护的是「别人拿到链接不会顺手点进编辑面板」。

**改密码的两种办法**

1. **面板里改（推荐）**：打开面板 → **F · 安全设置** → 填「新密码」「再输一次」（≥ 4 位）→ 勾上「同时换一个随机盐」→ 点**「更新密码并发布」**。会依次做：算新哈希 → 把 `ADMIN_SALT` / `ADMIN_PASS_HASH` 写回源码 → **走一遍产物自检** → 用上面的 GitHub Token 发布（没填 Token 时会把改好的整页复制到剪贴板，并提示你点「② 下载 index.html」手动替换）。自检如果没过会**自动回滚**，不会把你锁在门外。旁边还有个「只算新哈希（不动文件）」可以先看看结果。
2. **手工改**：在浏览器控制台（打开本页后）执行
   ```js
   hallPassHash("你的新密码")            // 用当前 ADMIN_SALT
   // 或者连盐一起换：
   hallPassHash("你的新密码", "自己编的12位十六进制")
   ```
   把返回值填进 `ADMIN_PASS_HASH`（换盐的话 `ADMIN_SALT` 也一起改）。也可以直接用面板 **D · 源码编辑器**读取 `hall/index.html`、改这两行、提交。

**「记住本机」与「锁定」**

- 密码门上有个「记住本机」勾选框。勾上并成功进入后，浏览器 `localStorage` 会写一个解锁标记，键名 **`hall-admin:unlocked:v1`**，值就是 `"1"`——**只存这个标记，不存密码也不存哈希**。以后在**同一台机器、同一个网址**再触发入口就直接进面板，不再问密码。
- 面板右上角的**「锁定」**按钮会清掉这个标记并立刻退回密码门（本机下次要重新输密码）。
- 用 `file://` 双击打开时，浏览器把每个本地文件当成独立源、禁止写 `localStorage`，所以「记住本机」会被**自动禁用**并给出提示，每次都要重新输密码（密码门本身照常可用）。
- 公共电脑上别勾；想取消随时可以点「锁定」，或手动清掉这个键。

### 面板功能

**A · 游戏清单** — 列出 `GAMES` 的全部条目，每条都能改标题 / 简介 / 链接 / 封面 / 标签，并支持新增、删除、上移、下移。改动**实时反映到下方页面卡片**（所见即所得）。标题为空、链接为空或链接格式不对（必须以 `http` / `https` / `/` / 相对路径开头）会红字提示，并且**不会**把非法内容写进页面。

**每条的显示开关** — 每条右侧有一个「**在大厅显示**」勾选框（对应源码里的 `on` 字段）：勾掉立刻联动卡片，那张卡片会从大厅消失（勾选框所在条目会变成虚线灰底）；重新勾上就回来。工具条上还有「**全部显示 / 全部隐藏**」两个按钮。导出或发布的代码里**永远会显式写出 `on: true/false`**，不会丢字段。

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

   下载与发布前都会做一次**产物自检**（doctype、GAMES 块、写入标记数量、是否混进面板 DOM、`ADMIN_SALT` / `ADMIN_PASS_HASH` 是否与当前生效值一致且格式合法），任何一项不过就直接中止，避免把页面写坏。

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

**F · 安全设置** — 改管理员密码，详见上面「[管理员密码](#管理员密码)」。会显示当前生效的 `ADMIN_SALT` / `ADMIN_PASS_HASH`、本机解锁标记状态，以及「连错几次冷却多久」。

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
- **密码门是「挡一下」，不是「加密」**：口令只存加盐哈希（`ADMIN_SALT` + `ADMIN_PASS_HASH`），源码公开、纯前端比对，认真的人可以离线爆破。它能拦住的是「拿到链接随手点进编辑面板」；真正保密的东西（比如 Token）不要指望它。
- 两种「记住」都不写密码：`hall-admin:unlocked:v1` 只存一个 `"1"` 解锁标记，`hall-admin:token:v1` 才存 Token 明文。
- 就算进了面板，**没有 Token 也只能看，不能发布**。
- 忘记密码时不用慌：删掉 `ADMIN_PASS_HASH` 那一行的值改成新哈希，或者干脆把这一行改回出厂值 `"a930eb04b461c6c8"`（对应密码 `banzhongwang`），提交后即可重进。

## 相关仓库

| 仓库 | 内容 |
|---|---|
| [merge-yanglukai](https://github.com/merge-games/merge-yanglukai) | 合成杨璐恺（游戏本体） |
| [jump-yanglukai](https://github.com/merge-games/jump-yanglukai) | 跳一跳 · 璐恺版 |
| [fruit-yanglukai](https://github.com/merge-games/fruit-yanglukai) | 切西瓜 · 璐恺版 |
| [galgame-yanglukai](https://github.com/merge-games/galgame-yanglukai) | 璐恺十二钗 |
| [hall](https://github.com/merge-games/hall) | 本仓库（游戏大厅） |
