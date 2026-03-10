# 技术实现计划：AI 文件管理器核心功能

**版本**: 1.0
**日期**: 2026-03-09
**状态**: Draft

---

## 1. 技术选型 (Technology Stack)

### 1.1 后端：Python + PyWebView

**选择理由**：
- **PyWebView**：轻量级桌面框架，无需 Electron 的庞大体积（~200MB vs ~100MB）
- **Python 3.10+**：丰富的文件处理库，类型注解支持，异步能力
- **watchdog**：成熟的文件系统监控库，跨平台支持
- **标准库优先**：pathlib、json、threading 满足核心需求

### 1.2 前端：React 18 + Vite + Tailwind CSS

**选择理由**：
- **React 18**：组件化开发，Hooks 简化状态管理
- **Vite**：快速构建（< 1s），热更新体验好
- **Tailwind CSS**：实用类优先，减少自定义 CSS，符合 macOS 设计风格
- **Lucide React**：轻量图标库（< 50KB）
- **react-markdown**（可选）：Markdown 渲染支持
- **prism-react-renderer**（可选）：代码语法高亮

### 1.3 通信机制：PyWebView API Bridge

- 后端通过类方法暴露 API
- 前端通过 `window.pywebview.api` 调用
- 数据格式：统一 JSON，类型安全

---

## 2. 关键设计决策 (Key Design Decisions)

### 2.1 架构模式：分层架构

```
┌─────────────────────────────────────┐
│  Presentation Layer (React)         │  ← UI 组件、状态管理
├─────────────────────────────────────┤
│  API Layer (PyWebView Bridge)       │  ← 前后端通信
├─────────────────────────────────────┤
│  Business Logic Layer (Python)      │  ← 扫描、搜索、配置
├─────────────────────────────────────┤
│  Data Layer (File System + Cache)   │  ← 文件读写、内存缓存
└─────────────────────────────────────┘
```

### 2.2 通信策略：同步 API + 异步任务

- **同步 API**：配置读写（< 100ms）
- **异步任务**：文件扫描、全文搜索（后台线程 + 加载状态反馈）
- **文件监控**：可选增强功能，失败时降级到手动刷新
- **错误处理**：统一响应格式 `{"success": bool, "data": any, "error": str}`

### 2.3 性能优化策略

- **内存缓存**：扫描结果缓存（dict），避免重复扫描
- **增量更新**：watchdog 监控文件变化，仅更新变更部分
- **搜索限制**：文件大小 < 10MB，内容搜索使用生成器
- **前端优化**：虚拟滚动（> 100 文件）、搜索防抖（300ms）

---

## 3. 项目结构 (Project Structure)

```
AIFileManager/
├── backend/
│   ├── __init__.py
│   ├── main.py              # 应用入口
│   ├── api.py               # PyWebView API 类
│   ├── config.py            # 配置管理
│   ├── scanner.py           # 文件扫描
│   ├── searcher.py          # 搜索引擎
│   ├── watcher.py           # 文件监控
│   ├── exceptions.py        # 自定义异常
│   └── utils.py             # 工具函数
├── frontend/
│   ├── src/
│   │   ├── App.jsx          # 根组件
│   │   ├── components/
│   │   │   ├── Sidebar.jsx      # 类别列表
│   │   │   ├── FileList.jsx     # 文件表格
│   │   │   └── SearchBar.jsx    # 搜索框
│   │   ├── hooks/
│   │   │   └── useApi.js        # API 调用封装
│   │   └── main.jsx
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
├── tests/
│   ├── test_scanner.py
│   ├── test_searcher.py
│   └── test_config.py
├── pyproject.toml           # Python 依赖
└── README.md

# 用户配置文件路径（运行时）
~/.aifilemanager/config.json  # 用户配置（支持打包后应用）
```

---

## 4. 异常体系 (Exception System)

### 4.1 异常层次结构

```python
AIFileManagerError (基类)
├── ConfigError              # 配置错误
│   ├── ConfigNotFoundError
│   └── ConfigValidationError
├── ScannerError             # 扫描错误
│   ├── PathNotFoundError
│   └── PermissionDeniedError
└── SearchError              # 搜索错误
    └── SearchTimeoutError
```

### 4.2 错误处理原则

- **后端**：捕获异常 → 记录日志 → 返回统一错误响应
- **前端**：捕获 API 错误 → 显示用户友好提示 → 记录控制台
- **不静默吞噬**：所有异常必须处理或向上传播

---

## 5. 模块职责与依赖关系 (Module Responsibilities & Dependencies)

### 5.1 后端模块

| 模块 | 职责 | 依赖 | 输出 |
|------|------|------|------|
| `config.py` | 配置读写、验证、默认值 | pathlib, json | Config 对象 |
| `scanner.py` | 递归扫描目录、提取文件元数据（后台线程） | pathlib, os, threading | List[FileInfo] |
| `searcher.py` | 文件名/内容搜索、结果排序（后台线程） | scanner, threading | List[SearchResult] |
| `watcher.py` | 监控文件变化、触发增量更新 | watchdog, scanner | 事件回调 |
| `api.py` | 暴露前端 API、协调各模块 | 所有模块 | JSON 响应 |
| `main.py` | 初始化 PyWebView、启动应用 | api, webview | - |

### 5.2 前端组件

| 组件 | 职责 | 状态 | API 调用 |
|------|------|------|----------|
| `App.jsx` | 全局状态、路由 | categories, files, selectedCategory | getCategories, getFiles |
| `Sidebar.jsx` | 类别列表、选择 | - | - |
| `SearchBar.jsx` | 搜索输入、防抖 | query | searchFiles |
| `FileList.jsx` | 文件表格、排序 | sortBy | - |

### 5.3 依赖关系图

```
main.py → api.py → {config, scanner, searcher, watcher}
                       ↓
                   scanner ← watcher
                       ↓
                   searcher
```

---

## 6. TDD 模式 (TDD Approach)

### 6.1 测试策略

- **单元测试**：每个模块独立测试（80%+ 覆盖率）
- **集成测试**：API 层测试（模拟前端调用）
- **E2E 测试**：PyWebView 启动测试（可选）

### 6.2 测试用例设计

#### 6.2.1 config.py
```python
# RED: 测试配置加载
def test_load_default_config()
def test_load_custom_config()
def test_save_config()
def test_validate_config_schema()
def test_add_category()
def test_remove_category()
```

#### 6.2.2 scanner.py
```python
# RED: 测试文件扫描
def test_scan_directory()
def test_scan_recursive()
def test_scan_empty_directory()
def test_scan_permission_denied()
def test_extract_file_metadata()
```

#### 6.2.3 searcher.py
```python
# RED: 测试搜索功能
def test_search_by_filename()
def test_search_by_content()
def test_search_case_insensitive()
def test_search_large_file_skip()
def test_search_empty_query()
```

### 6.3 测试工具

- **pytest**：测试框架
- **pytest-cov**：覆盖率报告
- **pytest-mock**：模拟文件系统
- **mypy**：类型检查

### 6.4 TDD 工作流

1. **RED**：编写失败测试 → 运行 `pytest` → 确认失败
2. **GREEN**：最小实现 → 运行 `pytest` → 确认通过
3. **REFACTOR**：优化代码 → 运行 `pytest` → 确认通过
4. **COVERAGE**：运行 `pytest --cov` → 确认 80%+

---

## 7. 其他技术要点 (Other Technical Considerations)

### 7.1 性能指标

| 指标 | 目标 | 实现方式 |
|------|------|----------|
| 扫描 1000 文件 | < 2s | 多线程扫描、缓存结果 |
| 搜索响应 | < 1s | 索引缓存、限制文件大小 |
| 内存占用 | < 200MB | 按需加载、清理缓存 |
| CPU 空闲 | < 5% | 后台线程、防抖 |

### 7.2 安全考虑

- **路径验证**：防止路径遍历攻击（`Path.resolve()`）
- **文件大小限制**：搜索文件 < 10MB
- **权限检查**：捕获 `PermissionError`，友好提示
- **输入验证**：配置 JSON schema 验证

### 7.3 打包部署

- **工具**：PyInstaller
- **配置**：`--onefile --windowed --icon=icon.icns`
- **资源打包**：前端构建产物（`frontend/dist`）
- **签名**：macOS 代码签名（可选）

### 7.4 日志策略

```python
# 日志级别
DEBUG: 详细调试信息（开发环境）
INFO: 关键操作（扫描完成、配置更新）
WARNING: 非致命错误（文件权限不足）
ERROR: 致命错误（配置损坏、模块加载失败）

# 日志输出
开发环境: 控制台 + 文件（app.log）
生产环境: 文件（~/Library/Logs/AIFileManager/app.log）
```

### 7.5 用户体验优化

- **加载状态**：骨架屏、进度条
- **错误提示**：Toast 通知、错误边界
- **空状态**：友好提示、引导操作
- **快捷键**：Cmd+F（搜索）、Cmd+R（刷新）

---

## 8. 风险与缓解 (Risks & Mitigation)

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 大量文件扫描性能差 | 高 | 多线程、增量更新、缓存 |
| PyWebView 兼容性问题 | 中 | 测试多个 macOS 版本 |
| 打包后体积过大 | 低 | 排除不必要依赖、压缩资源 |
| 文件权限问题 | 中 | 捕获异常、友好提示 |

---

**计划制定人**: AI Assistant
**审核人**: zhuyu
**批准日期**: zhuyu
