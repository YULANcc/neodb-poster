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
