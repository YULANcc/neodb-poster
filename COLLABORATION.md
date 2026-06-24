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
- **GitHub Pages 部署**：push 到 main 分支自动部署
- **双筛选器架构**：桌面端 `.toolbar / .sub-toolbar / .date-bar` 和移动端 `.mobile-filters` 是两套完全独立的 DOM，通过 CSS 媒体查询控制显隐，JS 共享同一个 state 对象和 `applyFilters()`

## 协作流程
1. 本地修改代码
2. `git add -A; git commit -m "描述"`
3. `$env:https_proxy="http://127.0.0.1:7892"; git push`
4. 等待 GitHub Pages 部署（约 1-2 分钟）
5. 刷新线上页面验证效果
6. ⚠️ CDN 有缓存延迟，建议 URL 加 `?ts=时间戳` 参数绕过缓存

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

## 修复记录

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

### 高优先级
1. **单张卡片导出** - 在详情模态框添加"Export Card"按钮，导出 400x600 PNG
   - 复用 Canvas 2D 渲染能力
   - 移动端支持长按触发
   - 保持与整体设计语言一致

### 中优先级
2. **错误处理优化** - API 请求失败时显示用户友好的错误提示
3. **加载状态反馈** - 按钮点击后显示 loading 状态
4. **Token 安全** - localStorage 加密或提示风险

### 低优先级
5. **响应式优化** - 小屏幕（<480px）海报网格优化为 2 列
6. **图片懒加载** - 海报封面添加懒加载
7. **国际化** - 界面中英混合，建议统一
