# 游戏大厅

一个静态首页，把做的所有小游戏列成卡片，一个链接分享全部。

- 线上地址：**https://merge-games.github.io/hall/**
- 想加/改游戏：编辑 `index.html` 顶部的 `GAMES` 数组即可（标题 / 简介 / 链接 / 封面 / 标签），保存刷新就能看到。

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

## 相关仓库

| 仓库 | 内容 |
|---|---|
| [merge-yanglukai](https://github.com/merge-games/merge-yanglukai) | 合成杨璐恺（游戏本体） |
