# 个人主页 · 接手说明

> 给后续接手的 AI 或人看的。看完这一页再动代码，能避开 90% 的坑。
> 最后更新：2026-11-18

---

## 1. 项目是什么

一个单文件、零依赖、零构建的个人主页。整站只有 `index.html`（约 2400 行）+ `assets/` 图片，没有 npm、没有框架、没有打包步骤。所有 CSS 和 JS 都内联在 HTML 里。

- 本地路径：`D:\01_Work\Projects\01_Portfolio\PersonalPage`
- 主域名：https://home-cqf.pages.dev
- 镜像：https://codecatio.github.io/home
- 仓库：https://github.com/CodeCatIO/home

**主域名是 Cloudflare Pages，不是 GitHub Pages。** GitHub Pages 只是镜像，两边内容完全一样。

## 2. 部署方式

```
改本地文件 → git add -A → git commit → git push origin main
```

- Cloudflare Pages 检测到 push 后自动构建，约 40–60 秒生效
- GitHub Pages 也会同时重新部署，但国内访问很慢，不要用它做验收
- 没有构建命令，没有输出目录，构建配置是「Framework preset: None / Build command: 空 / Output directory: /」
- 部署完验收：`https://home-cqf.pages.dev/` 返回 200，且 HTML 里能找到你刚改的内容

**注意：GitHub Pages 和浏览器都会缓存，验收时加 `?v=随便什么` 或强制刷新，否则会看到旧版本。**

## 3. 架构约定

### 路由

hash 路由：`#/home`、`#/about`、`#/work`、`#/tools`、`#/notes`、`#/contact`，工具详情还有 `#/tools/utoolbar`、`#/tools/minitodo`、`#/tools/minwallpaper`。

- 路由表：`const ROUTES = [...]`
- 特殊路由映射写在 `parseRoute()` 里
- 每个页面是 `<section class="page" data-page="xxx" hidden>`
- 切换靠 `hidden` 属性，不销毁 DOM

### 多语言

四语言 zh / en / ja / ko，靠 `data-t="key"` + `I18N` 字典。新增任何文案，**四种语言都要补齐**，否则回退到中文。

### 主题

CSS 变量在 `:root`（亮色）和 `[data-theme="dark"]`（暗色）里。**不要写死颜色**，一律用 `var(--ink)`、`var(--panel)`、`var(--line)` 这类变量。

## 4. 新增一个工具页的标准步骤

漏任何一步都会出现「页面打不开」或「切语言变英文」的问题：

1. 加 `<section class="page" data-page="工具名" hidden>`，放在工具详情区
2. `parseRoute()` 加一行 `if (raw === 'tools/xxx') return 'xxx';`
3. `TITLES` 四种语言各加一个 title
4. 导航高亮：`const navKey = (...)` 里把这个 key 归到 `tools`
5. 工具列表加卡片 `<a class="tool card-3d" href="#/tools/xxx">`
6. 卡片文案加进 `I18N` 四种语言
7. 图片放进 `assets/工具名/`，**必须是 WebP**，见第 5 节
8. 本地打开 `#/tools/xxx` 走一遍：图片、按钮、FAQ、暗色、切语言

## 5. 性能红线（必须遵守）

之前因为图片问题被用户投诉过一次，别再犯：

- **图片一律 WebP**，PNG/JPG 只作为源文件保留，页面里不许直接引用
- 截图按实际显示尺寸压：1180×740 的界面截图压到 60–70 KB 是正常水平，超过 200 KB 说明没压
- 首屏之外的所有 `<img>` 必须加 `loading="lazy" decoding="async"`
- 首屏主图加 `fetchpriority="high"`，并在 head 里 `preload`
- 所有 `<img>` 写死 `width` / `height`，防止布局跳动
- **视频不要 `autoplay`**，用 `preload="none"` + `data-src`，进入那个页面时再赋值播放
- 首屏图片预算：**150 KB 以内**

## 6. 已知的坑

| 坑 | 说明 |
|---|---|
| `hidden` 不阻止图片下载 | 隐藏页面里的 `<img>` 一样会被请求，必须靠 `loading="lazy"` |
| CSS 变量不存在会静默回退 | 曾经写过 `var(--card,#fff)`，但 `--card` 根本没定义，暗色下白底白字 |
| hash 路由的 SEO 局限 | 搜索引擎只把整站当一个 URL，子页面收录不了，这是架构代价，不是 bug |
| GitHub Pages 国内很慢 | 新访客 TTFB 曾测到 8.8 秒，所以主域换成了 Cloudflare Pages |
| `workers.dev` 国内被墙 | 不要用它做部署，`pages.dev` 可以 |
| Gitee 是导入副本 | 不会自动同步 GitHub，改完代码要去 Gitee 点「强制同步」 |
| Pages 构建有延迟 | push 后立刻访问可能还是旧版，等 40–60 秒 |

## 7. 文案规范

- 全站中文为主，禁止出现真实姓名、学校、专科经历
- 不要写「汇报进度」「对仗总结」「有坑位」这类 AI 味表达
- 不要时间线
- 语气可以自嘲，不要喊口号
- 英文只用于技术名词、标签、版本号

## 8. 设计规范

- 主色 `--accent: #8f7bc4`（紫），辅色 `--accent-2: #d99ab4`（粉）
- 樱花：桌面 34 片，宽度 < 700px 时 16 片
- 流星：同屏最多 3 条，SVG 描边动画，不要改成整体平移
- 看板娘：只有站立和点击害羞两个状态，`assets/mascot-stand.webp` / `mascot-dust.webp`
- 鼠标礼花：点击炸 6–9 个四角星，播完必须 `remove()`
- 所有动效必须尊重 `prefers-reduced-motion`

## 9. 外部资源

| 用途 | 地址 |
|---|---|
| 主页仓库 | https://github.com/CodeCatIO/home |
| UToolBar | https://github.com/CodeCatIO/UToolBar |
| MINITODO | https://github.com/CodeCatIO/MINITODO |
| MINWallPaper | https://github.com/CodeCatIO/MINWallPaper |
| MINWallPaper 国内镜像 | https://gitee.com/codecatio/minwallpaper |
| 博客 | https://codecatio.github.io/blog.html |

## 10. 验收清单

改完任何东西，至少在浏览器里过一遍：

- [ ] 六个主页面能进能出，刷新不白屏
- [ ] 浏览器后退 / 前进正常
- [ ] 四种语言切换后没有中文残留、没有空文本
- [ ] 亮色 / 暗色都看一遍
- [ ] 手机宽度（< 820px）布局不溢出
- [ ] 控制台无 404、无报错
- [ ] 新增图片是 WebP 且体积正常
- [ ] 部署后线上 HTML 里能搜到你改的关键字

## 11. 当前待办

- [ ] GSC / Bing 新域名（home-cqf.pages.dev）提交站点地图
- [ ] 笔记页 6 条占位文章换成真实文章
- [ ] Gitee 每次更新后手动强制同步
