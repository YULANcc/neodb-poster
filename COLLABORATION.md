# 协作要点

## 项目信息
- 仓库：https://github.com/YULANcc/neodb-poster
- 主文件：neodb-poster-wall.html（单文件，GitHub Pages 托管）
- 功能：NeoDB 个人收藏海报导出工具（Canvas 2D 渲染）

## 代理配置
- 本地代理端口：7892（非 7890）
- 推送命令：`$env:https_proxy="http://127.0.0.1:7892"; git push`

## 修复记录

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

## 待改进项（建议）
1. **错误处理**：API 请求失败时缺少用户友好的错误提示
2. **加载状态**：按钮点击后缺少 loading 反馈
3. **本地存储**：Token 存储在 localStorage，可考虑加密
4. **响应式**：移动端海报导出预览可能溢出
5. **Canvas 渲染**：导出功能代码不完整，需确认 Canvas 2D 渲染逻辑
6. **代码结构**：单文件较大，可考虑模块化（但会破坏单文件部署）
7. **Demo 数据**：缺少内置 demo 数据，首次使用体验不佳
8. **国际化**：界面混合中英文，建议统一
