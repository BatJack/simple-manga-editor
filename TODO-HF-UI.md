# HuggingFace 风格界面统一 — 剩余待办

> 范围：只改 GUI 样式，不改功能 JS 逻辑。CSS 颜色必须走 `css/root.css` 中的 token；除 token 声明外不新增硬编码 hex/rgba；圆角用 `--radius-*`；动画用 `--dur-*`/`--ease`；兼容 `prefers-reduced-transparency` 与 `prefers-reduced-motion`；不使用固定像素宽度做布局。
> 缓存规则：改 CSS/JS 后必须升 `index.html` 中对应 `?v=`，并在必要时升 `service-worker.js` 的 `CACHE_VERSION`。
> 状态标记：`[ ]` 未开始 / `[x]` 已完成。

---

## 0. 已完成批次（存档）

- [x] toast / shortcut-modal / unified-settings / floating-window
- [x] dashboard / role-assign-modal / overlay-progress / image-prompt-helper
- [x] inpaint-editor / angle-editor / object-menu / mode-change / share / font-manager
- [x] blend / bottom-bar / tutorial / intro
- [x] gallery（检查后已合规）、custom-html-components（检查后已合规）
- [x] ComfyUI 工作流编辑器 / form / tagify / tippy / controls-mini / image-control-manager / layout / layout-layer / responsive
  - 版本：上述 CSS 为 `?v=7.4`，`CACHE_VERSION` = `manga-editor-v12`
- [x] P0 批次：`html/common.css`（帮助页主题重做）、`html/functionList.html` 内联样式、`css/ui/custom-html-components.css` 死代码清理、`css/styles-css.css` token 化
  - 版本：`css/styles-css.css` = `?v=7.4`，`CACHE_VERSION` = `manga-editor-v13`
- [x] P1 批次：floating-window 面板重做、`components.css` 视觉细节、图层列表 token 化、导航栏 mask 图标、`.input-container` 弹性最小宽、启动引导层 token 化
  - 版本：`main-component.css` / `common.css` / `components.css` / `custom-html-components.css` = `?v=7.4`，`CACHE_VERSION` = `manga-editor-v14`
- [x] P2 批次：`--margin-right-5px` 变量修复、`toast.css` 时长注明、legacy 别名清零
  - 版本：`layout.css` / `layout-layer.css` / `form.css` / `common.css` = `?v=7.5`，`toast.css` / `image-prompt-helper.css` = `?v=7.4`，`gallery.css` = `?v=7.6`，`CACHE_VERSION` = `manga-editor-v15`

---

## P0 — 完全未统一（下一批优先）

### [x] 1. 帮助页共享样式表（71 个帮助页）

- 文件：`html/common.css`
- 问题：
  - `:root`（L1-13）自建一套**紫色主题**：`--accent:#7c4dff`、`--bg:#121212`、`--bg-card:#1e1e1e`、`--bg-hover:#2a2a2a` 等 14 处硬编码色值
  - 无浅色主题支持，与主界面 HF 黄体系完全脱节
  - `transition: all 0.2s`（L52、L225、L261）非 token
  - `.lang-btn.active { color:#fff }`（L63）
  - `.diff-remove` / `.diff-add` 硬编码 `rgba(248,113,113,.2)` / `rgba(74,222,128,.2)`（L298、L303）
  - `code` 字体 `'SF Mono', Monaco, monospace`，未用 `--font-mono`
- 建议：
  - 引入 `css/root.css`，删除自建色板，全部改用 `--bg-*`、`--text-*`、`--border-*`、`--accent`、`--ok/warn/danger`
  - 圆角换 `--radius-sm/md/lg`，动画换 `--dur-*`、`--ease`
  - 正文 `--text-1`、弱化文字 `--text-2`，卡片 `--bg-0` + `--border-default` + `--elev-1`
  - 表格 hover `--hover`；`code` 用 `--bg-2` + `--font-mono`
- 验收：深/浅两主题下帮助页可读、无紫色残留、无硬编码色值；`npm` 校验无未定义变量；窄屏 768px 不溢出

### [x] 2. functionList.html 内联样式

- 文件：`html/functionList.html`（L8-18 内联 `<style>`）
- 问题：`color:#fff`（L11）、`background:rgba(124,77,255,.2)`（L17）
- 建议：改用 token（`--accent` / `--accent-dim` / `--text-0`），必要时把内联样式并入 `html/common.css`

### [x] 3. 清理注释死代码

- 文件：`css/ui/custom-html-components.css`（L3-10）
- 问题：注释块内的旧输入框样式含 `#3a3a3a` / `#4a4a4a` / `#fff`
- 建议：直接删除该注释块

### [x] 4. 通用页面模板确认

- 文件：`css/styles-css.css`
- 问题：header / nav / footer 全部走 legacy 别名（`--color-secondary`、`--color-text-primary`），无新 token
- 建议：先确认是否被任何页面引用；未使用则删除，正在使用则改为 `--bg-*` / `--text-*`

---

## P1 — 部分处理 / 修复真实缺陷

### [x] 5. floating-window 提示面板对比度 bug

- 文件：`css/common.css`（L36-73）
- 问题：
  - `.floating-windowPromptClass h4 { color: var(--background-color-A) }` → 即 `--bg-0` 落在 `--bg-1` 上，**深色模式下标题几乎不可见**
  - 面板 `width:40%` + `padding:5px`，无圆角、无 `--elev-*`
  - 输入框边框恒为强调黄 `--color-input-focus-border`，无法区分常态与 focus
- 建议：标题 `--bg-1` 底 + `--text-0` 字 + 底部 `--border-subtle`；面板 `--bg-0` + `--radius-md` + `--elev-3` + 内边距 `--space-3`；输入框常态 `--border-default`，focus 才用 `--accent` + `--accent-dim` 光晕

### [x] 6. 主栏组件视觉细节

- 文件：`css/components.css`
- 问题：
  - `.param-group-header`（L285）、`.provider-status-label`（L308）用强调黄 `--text-color-C` 作正文色
  - `#svg_icon_results:empty::after`（L114）用 `--color-tertiary`（= `--bg-3`）作文字，对比度极低
  - `#svg_icon_results`（L105）仍用 legacy 圆角
- 建议：分组标题改 11px / 600 / `--text-2` / 大写 + letter-spacing；状态标签改 `--text-2`；空态文字改 `--text-2`；圆角 `--radius-sm`

### [x] 7. 图层列表面板收尾

- 文件：`css/layout-layer.css`
- 问题：图层名称/按钮仍用 legacy（`--text-color-B`、`--fjm-button-border-radius`、`--layer-*` 别名）
- 建议：按需替换为 `--text-0`、`--radius-sm`、`--bg-2`、`--hover-strong`；`ai-spin 0.8s`（L214）可保留

### [x] 8. 导航栏残留硬编码

- 文件：`css/core/main-component.css`
- 问题：L245 / L249 汉堡图标 SVG data-URI 内 stroke `rgba(229,231,235,.9)` / `rgba(17,24,39,.85)`；L273 `inset 0 0 0 1px rgba(255,210,30,.45)`
- 建议：前者属 data-URI 无法引用 CSS 变量，可用 `mask-image` + `background-color:var(--text-0)` 方案替代；后者改 `--accent-dim` / `--accent`

### [x] 9. 固定像素宽度

- 文件：`css/ui/custom-html-components.css`（L16、L25）
- 问题：`.input-container` `min-width:150px`、`.input-container-leftSpace` `min-width:140px`
- 建议：改为弹性（`flex:1 1 auto` / `min-width:0`）或统一为百分比/`ch` 基准

---

## P2 — 变量与细节

### [x] 10. 未定义变量

- 文件：`css/layout.css`（L225）
- 问题：`.info-item .fab { margin-right: var(--margin-right-5) }`，root.css 只有 `--margin-right-5px`，当前实际失效
- 建议：改为 `var(--margin-right-5px)`

### [x] 11. 非 token 时长

- 文件：`css/toast.css`（L76）：`transition: width 20ms linear` → `var(--dur-1) var(--ease)` 或保留但注明原因
- `css/ui/overlay-progress.css`（L5）`1.5s` 与 `ai-spin 0.8s` 为 spinner 时长，可保留

### [x] 12. legacy 别名统一（可选，不影响显示）

已清零：`layout-layer.css`、`gallery.css`、`form.css`、`common.css`、`layout.css`、`image-prompt-helper.css` 中 60 处 legacy 别名全部替换为 `--bg-*` / `--text-*` / `--border-*` / `--accent` / `--radius-*`；`root.css` 中的别名定义保留（JS 仍通过 `getCssValue` 读取）。

### [x] 13. 启动引导层内联色值

- 文件：`index.html`（boot 内联 `<style>` + SVG 生成逻辑）
- 处理：head 提前引入 `css/root.css`（渲染阻塞，首屏前即可用 token）；`.hf-boot` 直接使用 `--bg-0` / `--text-0`，局部变量映射为 `--border-default` / `--text-2` / `--bg-1` / `--bg-2`；删除 `.hf-boot--light/--dark` 两套硬编码底色与 `isLight` 判断，跟随 html 主题类自动切换；SVG 多边形描边改 CSS `stroke:var(--border-strong)`，就绪高亮改 `style.stroke="var(--hf-yellow-strong)"`
- 验证：深色 `rgb(11,15,25)` / `rgb(229,231,235)`，浅色 `rgb(255,255,255)` / `rgb(17,24,39)` 随主题自动切换；polygon 初始 `rgba(255,255,255,.22)`、就绪 `rgb(255,157,0)`；无控制台错误与 404

---

## P3 — 新发现（非本次样式改造引入）

### [x] 14. 852px 宽度下顶部导航栏溢出

- 现象：视口 852px 时 `scrollWidth=899`，超出元素为 `#navbarNavDropdown.collapse`（right≈899、width≈845），其下 `.navbar-nav.ms-auto`（right≈889）
- 根因：导航使用 `navbar-expand`（Bootstrap 5 无断点版本 = 任何宽度都强制单行展开），且汉堡按钮仍是 Bootstrap 4 的 `data-toggle/data-target`，在 5.2.3 bundle 下实际不可用
- 处理：
  - `index.html`：`navbar-expand` → `navbar-expand-lg`；汉堡按钮改 `data-bs-toggle="collapse" data-bs-target="#navbarNavDropdown"`
  - `css/core/main-component.css`：新增 `@media (max-width:991.98px)`，导航固定 48px 高，`.navbar-collapse` 改为绝对定位下拉面板（`top:100%`、`left/right:0`、`--bg-0`、`--elev-2`、`--z-dropdown`），左菜单竖向、右侧 `.navbar-nav.ms-auto` 横向可换行；toggler 右对齐 + `--accent-dim` 焦点环
- 验证：852px 下 `scrollWidth=844 = clientWidth`，导航高度 48px；菜单可展开/关闭，`File` 下拉 `clientHeight=390px` 未被裁剪；无控制台错误与 404
- 版本：`main-component.css` = `?v=7.6`，`CACHE_VERSION` = `manga-editor-v17`
- 备注：≥992px 桌面布局不变（原 `navbar-expand` 的展开规则与 `navbar-expand-lg` 在宽屏下等价）

---

## 验证清单（每批完成后执行）

- [ ] 桌面宽度（1440px）深色主题截图正常
- [ ] `applyTheme('light-mode')` 后无白字白底、无纯黑异常块
- [ ] 976px 视口下 `document.documentElement.scrollWidth <= window.innerWidth`（无横向溢出）
- [ ] 控制台无 CSS 解析错误、无未定义变量告警
- [ ] 页面 HTML 走 network-first；确认改动样式表 `?v=` 已升级，必要时 `CACHE_VERSION` +1
- [ ] 未运行 `npm run format`；新增 CSS 未缩进

---

## 排除项（不处理）

- `css/all.min.css`、`css/flag-icon.min.css`、`third/**`：vendored 文件
- `js/core/debug.js`、`js/core/logger.js` 中的 `console.log`：属调试输出
- `json_js/`、`test/`、`01_build/`、`02_images_svg/`、`03_images/`、`99_doc/`、`font/`、`node_modules/`：按项目规范排除
