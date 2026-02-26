# Dashboard Cleanup Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 清理控制台无用内容与噪声日志，修复图标/对齐问题，并让重启与更新检测具备真实功能。

**Architecture:** 在渲染层精简仪表盘数据流与卡片布局，新增更新/关于弹窗与红点状态；主进程暴露重启服务 IPC，并在预加载层提供安全 API 给渲染层调用。

**Tech Stack:** Electron (main/preload), React, TypeScript, Redux, Vite

---

### Task 1: 清理仪表盘数据流与移除最近对话卡片

**Files:**
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.tsx`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.css`

**Step 1: 识别并移除最近对话相关状态与定时器**
- 删除 conversations 状态、loadConversations、定时刷新对话的 interval。
- 移除最近对话卡片 JSX 与对应的条件渲染。

**Step 2: 移除运行环境检测与噪声日志**
- 删除 runtime 检测状态与日志追加（如 “正在刷新仪表盘数据...”“最近会话已刷新”）。
- 保留必要的错误日志与用户触发日志。

**Step 3: 清理 CSS**
- 删除 `.conversations-card` 与相关样式块。
- 保留其它卡片样式不动。

**Step 4: 手动检查**
- 打开控制台页面，确认最近对话卡片已移除，日志区仍存在且不再刷出噪声日志。

---

### Task 2: 修复日志工具图标可见性

**Files:**
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.css`

**Step 1: 定位“导出/清空”按钮图标不可见原因**
- 检查 `.btn-export`/`.btn-clear` 的颜色、背景、SVG 继承颜色。

**Step 2: 修复样式**
- 调整颜色或 SVG 继承颜色规则，确保图标在浅/深背景都可见。

**Step 3: 手动检查**
- 确认两枚图标正常显示。

---

### Task 3: 更新检测与版本号 UI 行为

**Files:**
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\App.tsx`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.tsx`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.css`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\window\WindowTitleBar.tsx`

**Step 1: App 层暴露更新状态**
- 使用 `updateInfo` 推导 `hasUpdate`，传给 `NexusDashboard`。
- 更新 `app:showAbout` 逻辑，改为打开关于/更新弹窗而非 toast。

**Step 2: 状态区按钮与版本号**
- 删除“设置”按钮。
- 在“检测更新”按钮右上角添加红点（仅 `hasUpdate` 时显示）。
- 版本号移动至“检测更新”下方小字并可点击打开更新弹窗。

**Step 3: 手动检查**
- 有更新时出现红点；点击“检测更新”触发检查；点击版本号打开关于/更新弹窗。

---

### Task 4: 关于/更新弹窗改为旧版内容

**Files:**
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\update\AppUpdateModal.tsx`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\App.tsx`

**Step 1: 扩展 AppUpdateModal**
- 支持显示当前版本、最新版本、官网链接、检查更新按钮。

**Step 2: 绑定入口**
- 顶部“帮助→关于”打开弹窗。
- 版本号点击打开弹窗。

**Step 3: 手动检查**
- 弹窗信息正确显示，按钮交互正常。

---

### Task 5: 让“重启”具备真实功能

**Files:**
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\main\main.ts`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\main\preload.ts`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.tsx`

**Step 1: 主进程 IPC**
- 新增 `ipcMain.handle('app:restartServices')`：
  - `coworkRunner.stopAllSessions()`
  - `stopCoworkOpenAICompatProxy()` 后再 `startCoworkOpenAICompatProxy()`
  - `imGatewayManager.stopAll()` 后再 `startAllEnabled()`
- 返回执行结果给渲染层。

**Step 2: 预加载桥接**
- 在 `window.electron` 暴露 `app.restartServices()`。

**Step 3: 渲染层按钮**
- “重启”按钮调用 `window.electron.app.restartServices()`，并显示“重启中”状态。
- 错误场景弹 toast 并恢复按钮。

**Step 4: 手动检查**
- 点击“重启”后按钮进入加载态，完成后恢复。

---

### Task 6: 统一图标尺寸与对齐

**Files:**
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusDashboard.css`
- Modify: `c:\markovbot\markovbot\nexusbot-main\src\renderer\components\nexus\NexusHardwareInfo.tsx`

**Step 1: 统一图标尺寸**
- 调整“重启/检测更新/展开详情/导出报告”按钮内 SVG 高度与对齐。

**Step 2: 修复下拉按钮对齐与外框**
- 调整 “展开详情” 按钮样式，确保右对齐并移除多余外框。

**Step 3: 手动检查**
- 图标高度一致且按钮对齐正确。

---

### Task 7: 验证

**Step 1: 运行 lint**
- Run: `npm run lint`
- Expected: 若出现既有历史错误，记录并说明未由本次修改引入。

**Step 2: 运行 build**
- Run: `npm run build`
- Expected: 构建成功。

**Step 3: 手动验证**
- 控制台 UI 无多余卡片、图标可见、更新红点正常、版本号可点击、重启可用。

---

**Note:** 按项目约束不提交 git commit，除非你明确要求。
