# 协作要点

## 项目信息
- 仓库：https://github.com/YULANcc/neodb-poster
- 线上地址：https://yulancc.github.io/neodb-poster/neodb-poster-wall.html
- 主文件：neodb-poster-wall.html（单文件，GitHub Pages 托管）
- 功能：NeoDB 个人收藏海报导出工具（Canvas 2D 渲染）

## 推送配置
- 本地代理端口：7892（非 7890）
- 推送命令：`$env:https_proxy="http://127.0.0.1:7892"; git push`

## 项目概述
这是一个为 NeoDB 个人收藏页定制的海报导出工具。用户在网页上配置好布局后，一键导出与页面布局完全一致的静态图片。

**核心功能**：
- 从 NeoDB 读取用户收藏数据（书籍、电影、剧集、音乐、游戏），按选择的分类和布局渲染海报
- 支持 Canvas 2D 直接渲染导出，无需依赖 html2canvas
- 海报布局与 DOM 预览完全对齐（间距、字体、分隔线等）
- 支持封面图片、本地图片、纯色背景三种海报样式
- 右上角按钮：视图切换、使用指南、设置、同步、筛选（移动端）

## 技术架构
- **单文件架构**：所有 HTML/CSS/JS 都在 neodb-poster-wall.html 中
- **Canvas 2D 渲染**：绕过 html2canvas 的兼容性问题，直接渲染导出
- **无外部依赖**：仅使用 Google Fonts（Playfair Display + Inter）
- **GitHub Pages 部署**：推送到 `dev` 分支不触发部署，合并到 `main` 分支自动部署
- **双筛选器架构**：桌面端 `.toolbar / .sub-toolbar / .date-bar` 和移动端 `.mobile-filters` 是两套完全独立的 DOM，通过 CSS 媒体查询控制显隐，JS 共享同一个 state 对象和 `applyFilters()`

## 协作流程

### 日常开发（不触发线上部署）
1. 本地修改代码
2. `git add -A; git commit -m "描述"`
3. `$env:https_proxy="http://127.0.0.1:7892"; git push origin dev`
4. 开发分支不会触发 GitHub Pages 部署，可以随意调试

### 发布上线（触发部署）
1. 确认 `dev` 分支代码已稳定
2. `$env:https_proxy="http://127.0.0.1:7892"; git checkout main; git merge dev; git push`
3. 推送到 main 分支后，GitHub Pages 自动部署（约 1-2 分钟）
4. 刷新线上页面验证效果
5. ️ CDN 有缓存延迟，建议 URL 加 `?ts=时间戳` 参数绕过缓存

### 线上地址
- GitHub Pages: `https://yulancc.github.io/neodb-poster/`
- GitHub 仓库: `https://github.com/YULANcc/neodb-poster`
- 本地代理端口: 7892（非 7890）

## 踩过的坑

### 1. 桌面端和移动端筛选器互相影响
**问题**：只改一端CSS，另一端也跟着变。
**根因**：共用一套筛选器 DOM 结构。
**解决方案**：HTML 中放两套独立筛选器（`.toolbar/.sub-toolbar/.date-bar` 桌面端 + `.mobile-filters` 移动端），CSS 媒体查询控制显隐。JS 侧的 `updateChipStates()` 需要同时更新两套 DOM 的 active 状态。

### 2. edit 工具匹配换行符
**问题**：`edit` 的 `oldText` 必须和文件中实际内容完全一致，包括 `\r\n` vs `\n` 差异。
**解决**：先用 `Select-String` 或 `read` 定位准确文本再替换。

### 3. CDN 缓存
**问题**：push 后线上看不到最新改动。
**解决**：URL 加 `?ts=时间戳` 或硬刷新 `Ctrl+Shift+R`。

### 4. 改视图切换不要动 sub-toolbar
**教训**：`sub-toolbar` 是桌面端和移动端共用的组件。视图切换按钮整合到 masthead-actions 后，`sub-toolbar` 在移动端下已经隐藏，但桌面端需要 `display: none` 来彻底移除。

### 5. 移动端筛选按钮整合
**问题**：移动端 masthead-actions 只有 3 个按钮（指南/设置/同步），筛选按钮在独立的 `.mobile-filters` 里。
**解决**：在 masthead-actions 末尾加 `#filterToggleBtn`（桌面端隐藏、移动端显示），移动端的独立 filter-toggle 隐藏。两个按钮绑定同一个 `toggleFilterPanel()` 函数。

---

## 修复记录

### 2026-06-24 P0 修复（Token + 加载 + 卡片导出 + 详情模态优化）
- **Token 安全增强**：localStorage 改为 base64 混淆存储（新键），旧 token 首次自动迁移，设置面板加安全提示
- **首次加载逻辑修正**：单次点击 → recent 秒出 → 后台自动续跑全量加载，不再需要点两次
- **详情模态操作增强**：在 modal-body 底部加入"Export Card"按钮；Canvas 导出完整模态内容（封面、标题、分类、评分、状态、评语、标签、底部栏）
  - NeoDB 外链和复制链接功能暂缓，待后续排期

### 2026-06-24 密集优化
- 桌面端/移动端筛选器完全分离（两套独立 DOM）
- 移动端标题：32px 居中非斜体，按钮 44px 触控尺寸无底色
- 桌面端标题去斜体
- 视图切换整合到右上角图标栏
- help-link 统一为透明按钮样式
- 统计栏底部横线移除、导出按钮居中
- 海报预览溢出修复

### 2026-06-22 修复按钮无法交互
**问题原因**：JavaScript 语法错误导致整个脚本执行失败
1. `buildCell` 函数缺少闭合括号 `}`
2. `exportPoster` 函数中有多余的 `}` 导致函数提前结束
3. `escapeHTML` 函数未定义但被调用
4. html2canvas 依赖已移除（改用 Canvas 2D 直接渲染）

**修复内容**：
- 添加 `escapeHTML` 函数定义
- 修复 `buildCell` 函数闭合
- 修复 `exportPoster` 函数结构
- 移除未使用的 html2canvas script 标签

## 待实现功能

### P2 — 体验打磨

1. **图片代理稳定性**
   - 所有封面走 `https://images.weserv.nl/?url=...` 代理，服务不属于本项目
   - 方案：Canvas 导出时优先尝试直连原图；直连失败再走代理；代理也失败则用占位色块
   - 网页端预览的 `<img>` 保持走代理（性能考虑），加 `onerror` fallback

2. **海报预览弹窗缩放优化**
   - 导出弹窗打开后 `setTimeout 200ms` 重新算 scale，有可见的内容跳动
   - 方案：使用 `ResizeObserver` 实时监听容器宽度变化，或先渲染到离屏再一次性插入

3. **移动端筛选面板体验**
   - 当前展开式面板操作路径长，分类按钮用纯 emoji（📖🎬📺🎵🎮）中文用户可能困惑
   - 方案：改为底部弹出 sheet（iOS 风格），分类按钮用文字+emoji

4. **escapeHTML 去重**
   - 当前脚本中 `escapeHTML` 有两次独立定义：一处 `textContent` 方案 + 一处正则替换方案
   - 统一为一个实现，删除冗余

### P3 — 远期迭代（工程债务 + 锦上添花）

5. **虚拟滚动 — 大数据量性能**
   - `renderGrid()` 一次性 DOM 渲染所有卡片，几百上千条收藏直接卡死
   - 方案：Intersection Observer + 视口内渲染；或至少分页加载

6. **空状态视觉升级**
   - 当前空状态仅标题 + 文字 + 按钮，单调
   - 方案：加 SVG 装饰插画 + 示例卡片骨架引诱连接

7. **统计栏视觉层次**
   - 统计数据一横排偏平淡，avg 空数据显示 `-` 不够友好
   - 方案：统计栏轻微背景区分、数字等宽字体对齐

8. **响应式优化**
   - 移动端海报网格在一些中间尺寸（480-640px）表现不理想
   - `poster-date` 标签在移动端完全隐藏，可考虑改为更紧凑的日期格式

9. **图片懒加载**
   - 海报封面添加 `loading="lazy"`（当前已部分实现，需全面覆盖）

10. **国际化**
    - 界面中英混合，建议统一为中文

11. **代码架构拆分**
    - 单文件约 2400 行，CSS/HTML/JS 混在一起，维护困难
    - 短期：添加注释分区标记 + table of contents
    - 长期：拆为独立 CSS + JS 模块（受限于 GitHub Pages 单文件部署，可能需要构建步骤）

12. **单元测试覆盖**
    - Canvas 渲染逻辑复杂，手动验证成本高
    - 至少给纯函数写测试：`getPresetRange`、`monthKey`、`passesDateFilter`、`wrapText`
