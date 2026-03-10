# [AIFileManager] AI 文件管理器协作指南

你是一位精通 Python 和前端开发的资深软件工程师。你的任务是协助我，以高质量、可维护的方式完成本项目的开发。

---

## 1. 技术栈与环境 (Tech Stack & Environment)

### 后端
- **语言**: Python (>= 3.10+)
- **桌面框架**: PyWebView (Python 与前端通信桥梁)
- **文件监控**: watchdog (监控文件系统变化)
- **依赖管理**: uv (虚拟环境和包管理)

### 前端
- **框架**: React 18
- **构建工具**: Vite
- **样式**: Tailwind CSS
- **图标**: Lucide React

### 开发工具
- **代码规范**: black (格式化)、isort (导入排序)
- **静态检查**: mypy (类型检查)、flake8 (代码风格)
- **测试**: pytest
- **运行说明（macOS）**: 使用项目根目录 `.venv/bin/python` 执行 Python 文件

---

## 2. 架构与代码规范 (Architecture & Code Style)

### Python 后端
- **模块化**: 每个模块职责单一（API、配置、扫描、搜索分离）
- **类型注解**: **[强制]** 所有公共函数必须添加类型注解
- **错误处理**: 使用自定义异常类，避免直接抛出 `Exception`
- **日志**: 使用 `logging` 模块，避免 `print()`
- **异步**: 文件扫描和搜索使用后台线程，避免阻塞 UI

### React 前端
- **组件化**: 每个组件职责单一，可复用
- **状态管理**: 使用 React Hooks (useState, useEffect)
- **样式**: Tailwind CSS 实用类，避免自定义 CSS
- **API 调用**: 通过 `window.pywebview.api` 调用后端方法

### PyWebView 通信
- **后端暴露 API**: 在 `api.py` 中定义类方法，通过 `webview.create_window(api=api)` 暴露
- **前端调用**: `await window.pywebview.api.method_name(args)`
- **数据格式**: 统一使用 JSON 格式传递数据

---

## 3. Git 与版本控制 (Git & Version Control)

- **Commit Message 规范**: **[严格遵循]** Conventional Commits 规范
  - 格式: `<type>(<scope>): <subject>`
  - 示例: `feat(backend): add file scanner module`
  - 类型: feat, fix, refactor, docs, test, chore, perf

---

## 4. AI 协作指令 (AI Collaboration Directives)

- **[原则] 最小化代码**: 只写必要的代码，避免过度设计和冗余实现
- **[原则] 优先标准库**: 优先使用 Python 标准库（os, pathlib, json）和 React 内置功能
- **[流程] 分段输出**: 创建文件时分段输出，避免一次性输出大量内容
- **[流程] 审查优先**: 实现新功能前先阅读相关代码，理解现有逻辑，提出计划后再编码
- **[实践] 类型安全**: Python 代码必须通过 mypy 检查，React 使用 PropTypes 或 TypeScript（可选）
- **[实践] 错误处理**: 前端调用后端 API 时必须处理异常，后端方法必须返回统一的响应格式
- **[产出] 解释代码**: 生成复杂代码后，简要解释核心逻辑和设计思想

---

## 补充说明

1. **依赖管理**: 使用 `uv` 管理 Python 依赖，前端使用 `npm` 或 `pnpm`
2. **打包**: 使用 PyInstaller 打包成 macOS .app 应用
3. **路径处理**: 使用 `pathlib.Path` 处理路径，支持 `~` 展开
4. **性能优化**: 大量文件时使用分页或虚拟滚动，搜索使用后台线程
5. **用户体验**: 加载状态、错误提示、空状态提示
