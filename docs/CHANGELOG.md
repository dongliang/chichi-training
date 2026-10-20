# CHANGELOG

## 手机阅读改造 — 2026-10-20

原来页面在 iPhone 上是坏的：`<meta name="viewport">` 写的是 `width=390`，把**所有**设备强制按 390px 布局——iPad、安卓大屏手机、桌面窄窗口全都变成放大的手机版面，iOS 上 Safari 还会按 390 的固定宽度做缩放。同时手机上几处版式直接不能用。

### 改动

- **viewport** 换成 `width=device-width, initial-scale=1, viewport-fit=cover`，并补 `theme-color`、`format-detection`（防电话号码自动识别）、`apple-mobile-web-app-*` 和纯 SVG 的 `apple-touch-icon`（不新增图片文件）。
- **断点统一到 900px**：原来左栏在 900px 折叠、但移动端优化只写到 760px，中间 761–900px 是一段「单栏 + 桌面字号」的尴尬区间。
- **速查表（`#drills`）在 ≤600px 改成卡片列表**：原来 5 列表格塞进 390px，名称和「一句话」被压成一列一个字。现在每条是「编号 + 章 + 视频数 / 名称 / 一句话」三段式，整条可点。
- **视频库（`#videos`）在手机上改成横排卡**：缩略图 118px 靠左（≤420px 收到 96px），文字在右，一屏能看到 4 条而不是 1 条。课程链接只留编号（完整标题在 `title` 语义下放 `.lk-t` 里，手机隐藏）。
- **首页 5 个数据卡**：双列会剩半行，改成一张卡片里的紧凑列表。
- **触控尺寸**：顶栏按钮、分页、chip、视频 chip、「标记为已学」全部 ≥44px；段落内行内链接用 `display:inline-block;padding` 撑到可点，不改版面。
- **iOS 输入放大**：搜索框 `font-size:16px`（低于 16px 时 Safari 聚焦会自动放大整页）。
- **抽屉（目录）**：`html{overflow:hidden}` 在 iOS Safari 不生效，改成 `body{position:fixed;top:-scrollY}` 的经典锁滚动写法，关闭时恢复滚动位置；补 `aria-expanded` / `aria-controls`、「Esc 关闭」。
- **`.rail.open{transform:none}` 改成 `translateX(0)`**：`none` 在某些引擎里和带过渡的 `translateX` 之间插值不出结果，抽屉会卡在屏幕外。
- **知识地图**：给横滑提示，`hover:hover` 不成立时把「悬停高亮」的说明换成「左右滑动」。`scroll-snap` 让横滑停在卡片列上。
- **字号**：所有 <12px 的小字在手机上统一抬到 11.5px，正文 15/15.5px，行高 1.75。
- **安全区**：`env(safe-area-inset-*)` 补到顶栏、底部留白；底部留 56px + 安全区，避免和 Home 指示条打架。
- **平板/横屏**：单栏时正文 `max-width:720px` 居中，一行不会拉到 80 多个字。
- **底部间距**：`.btn-done` 和 `.pager` 之间原来没有间距（HTML 里没有 `.lesson-foot` 容器），补 `margin-top`。

### 验证：实测的视口

| 宽度 | 设备对应 | 结果 |
|---|---|---|
| 320×568 | iPhone SE 1 代 / 最小屏 | 无横向溢出，视频卡不裁切 |
| 390×844 | iPhone 14/15 | 五个页面（首页/地图/速查/视频/课程）均无横向溢出（`scrollWidth === 390`） |
| 768×1024 | iPad 竖 | 表格仍是表格（600 断点以上），不挤 |
| 834×1000 | iPad Air 竖 | 同上 |
| 844×390 | 手机横屏 | 正文收窄居中，没有一行过长 |
| 1440×900 | 桌面 | 左栏固定目录、原版布局输出无变化 |

检查方法：`execute_js` 里遍历 `#view *` 比对 `getBoundingClientRect().right` 与 `innerWidth`，并统计 `button/a` 高度 <36px 的元素。

### 已知未做

- 知识地图的 SVG 仍是 780px 最小宽度、靠横滑看，没有改成「一列一条」的纵向列表。改成纵向会丢掉「连线表示依赖」这个核心表达，暂不做。

## 视频标题中英调换 — 2026-10-20

30 个视频原来只有 YouTube 的英文标题。现在每条加了 `zh` 字段（中文标题），列表卡、课程页视频条、播放器三处都是**中文当主标题、英文原题当副标题**，样式沿用原来的两级字号（主标题加粗、副标题灰字），没有新增视觉层。

- 中文标题是翻译，不是官方译名。邓巴学院没有中文频道，所以译名是按主题直译的，比如 `The Dog Training 3 Second Rule` → 「训练的三秒法则」。
- `aria-label` 和 iframe 的 `title` 也换成中文，读屏时不念一长串英文。
- 副标题长的地方单行截断（`-webkit-line-clamp:1`），完整英文标题在播放器里是全的。
- 英文副标题加了 `overflow-wrap:anywhere`，避免 `Mailbag` 这类长单词顶出卡片边界。
