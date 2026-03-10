# 任务清单：AI 文件管理器核心功能

**版本**: 1.0
**日期**: 2026-03-09
**状态**: Ready for Development

---

## 任务执行原则

**TDD 工作流（强制）**：
1. **RED**: 编写测试 → 运行 `pytest` → 确认失败
2. **GREEN**: 最小实现 → 运行 `pytest` → 确认通过
3. **REFACTOR**: 优化代码 → 运行 `pytest` → 确认通过
4. **COVERAGE**: 运行 `pytest --cov` → 确认 80%+

**代码规范**：
- 所有函数必须添加类型注解
- 使用 `pathlib.Path` 处理路径
- 使用 `logging` 模块记录日志
- 统一错误响应格式：`{"success": bool, "data": any, "error": str}`

---

## 阶段 1: 项目初始化

### Task 1.1: 创建项目结构
**优先级**: P0
**预计时间**: 30min
**依赖**: 无

**目标**：
- 创建后端和前端目录结构
- 初始化 Python 虚拟环境（uv）
- 初始化前端项目（Vite + React）

**验收标准**：
- [ ] 目录结构符合 plan.md 第 3 节
- [ ] `uv venv` 创建成功
- [ ] `npm create vite@latest frontend` 执行成功
- [ ] 安装 Python 依赖：
  - 核心依赖：pywebview, watchdog
  - 测试依赖：pytest, pytest-cov, pytest-mock
  - 代码质量：black, isort, mypy, flake8
  - 性能测试：psutil
- [ ] 安装前端依赖：react, react-dom, tailwindcss, lucide-react
- [ ] 配置 Tailwind CSS（安装依赖、配置文件、导入样式）

**TDD 步骤**：
- 无需测试（基础设施任务）

---

### Task 1.2: 创建异常体系
**优先级**: P0
**预计时间**: 20min
**依赖**: Task 1.1

**目标**：
- 实现 `backend/exceptions.py`
- 定义异常层次结构（plan.md 第 4.1 节）

**验收标准**：
- [ ] 定义 `AIFileManagerError` 基类
- [ ] 定义 `ConfigError`, `ScannerError`, `SearchError` 子类
- [ ] 每个异常类包含 `message` 和 `details` 属性
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_exceptions.py`
   - `test_base_exception_creation()`
   - `test_config_error_inheritance()`
   - `test_exception_message_format()`
2. **GREEN**: 实现 `exceptions.py`
3. **REFACTOR**: 优化异常类设计
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 1.3: 创建日志配置模块
**优先级**: P0
**预计时间**: 30min
**依赖**: Task 1.2

**目标**：
- 实现 `backend/utils.py` 中的日志配置
- 按照 plan.md 第 7.4 节的日志策略

**验收标准**：
- [ ] 配置日志级别（DEBUG, INFO, WARNING, ERROR）
- [ ] 开发环境：控制台 + 文件输出
- [ ] 生产环境：文件输出到 `~/Library/Logs/AIFileManager/app.log`
- [ ] 日志格式包含时间戳、级别、模块名、消息
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_utils.py`
   - `test_setup_logging_dev_mode()`
   - `test_setup_logging_prod_mode()`
   - `test_log_file_creation()`
2. **GREEN**: 实现日志配置函数
3. **REFACTOR**: 优化日志配置
4. **COVERAGE**: 确认 80%+ 覆盖

---

## 阶段 2: 后端核心模块

### Task 2.1: 实现配置管理模块
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 1.2

**目标**：
- 实现 `backend/config.py`
- 支持配置读写、验证、默认值

**验收标准**：
- [ ] 加载默认配置（4 个类别：rules, agents, skills, commands）
- [ ] 从 `~/.aifilemanager/config.json` 加载自定义配置
- [ ] 保存配置到 `~/.aifilemanager/config.json`
- [ ] 自动创建配置目录（如不存在）
- [ ] 验证配置 schema（类别名称、路径格式）
- [ ] 添加/删除类别
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_config.py`
   - `test_load_default_config()`
   - `test_load_custom_config()`
   - `test_save_config()`
   - `test_validate_config_schema()`
   - `test_add_category()`
   - `test_remove_category()`
2. **GREEN**: 实现 `config.py` 的 `Config` 类
3. **REFACTOR**: 优化配置验证逻辑
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 2.2: 实现文件扫描模块
**优先级**: P0
**预计时间**: 1.5h
**依赖**: Task 2.1

**目标**：
- 实现 `backend/scanner.py`
- 递归扫描目录，提取文件元数据

**验收标准**：
- [ ] 扫描指定目录下所有文件
- [ ] 支持递归扫描子目录
- [ ] 提取文件元数据：路径、名称、大小、修改时间
- [ ] 处理权限错误（PermissionError）
- [ ] 缓存扫描结果
- [ ] 使用后台线程异步执行，不阻塞主线程
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_scanner.py`
   - `test_scan_directory()`
   - `test_scan_recursive()`
   - `test_scan_empty_directory()`
   - `test_scan_permission_denied()`
   - `test_extract_file_metadata()`
2. **GREEN**: 实现 `scanner.py` 的 `FileScanner` 类
3. **REFACTOR**: 优化扫描性能（使用 `os.scandir`）
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 2.3: 实现搜索引擎模块
**优先级**: P0
**预计时间**: 1.5h
**依赖**: Task 2.2

**目标**：
- 实现 `backend/searcher.py`
- 支持文件名和内容搜索

**验收标准**：
- [ ] 文件名模糊匹配（大小写不敏感）
- [ ] 文件内容全文搜索
- [ ] 限制搜索文件大小（< 10MB）
- [ ] 异步执行，不阻塞 UI
- [ ] 返回搜索结果（路径、匹配内容、行号）
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_searcher.py`
   - `test_search_by_filename()`
   - `test_search_by_content()`
   - `test_search_case_insensitive()`
   - `test_search_large_file_skip()`
   - `test_search_empty_query()`
   - `test_search_async_execution()` - 验证后台线程执行
2. **GREEN**: 实现 `searcher.py` 的 `FileSearcher` 类（使用 threading.Thread）
3. **REFACTOR**: 优化搜索算法（使用生成器）
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 2.4: 实现文件监控模块
**优先级**: P1
**预计时间**: 1h
**依赖**: Task 2.2
**说明**: 此模块为增强功能，API 层可独立工作

**目标**：
- 实现 `backend/watcher.py`
- 使用 watchdog 监控文件变化

**验收标准**：
- [ ] 监控指定目录的文件变化
- [ ] 触发增量更新（新增、修改、删除）
- [ ] 支持启动/停止监控
- [ ] 处理监控异常
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_watcher.py`
   - `test_start_watcher()`
   - `test_stop_watcher()`
   - `test_file_created_event()`
   - `test_file_modified_event()`
   - `test_file_deleted_event()`
2. **GREEN**: 实现 `watcher.py` 的 `FileWatcher` 类
3. **REFACTOR**: 优化事件处理逻辑
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 2.5: 实现工具函数模块
**优先级**: P0
**预计时间**: 30min
**依赖**: Task 1.3

**目标**：
- 实现 `backend/utils.py` 中的工具函数
- 提供路径验证、文件大小格式化等通用功能

**验收标准**：
- [ ] 路径验证函数（防止路径遍历攻击）
- [ ] 文件大小格式化函数（bytes → KB/MB/GB）
- [ ] 路径展开函数（支持 `~` 展开）
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_utils.py`
   - `test_validate_path_safe()`
   - `test_validate_path_traversal_attack()`
   - `test_format_file_size()`
   - `test_expand_path()`
2. **GREEN**: 实现工具函数
3. **REFACTOR**: 优化函数实现
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 2.6: 实现 API 层
**优先级**: P0
**预计时间**: 2h
**依赖**: Task 2.1, 2.2, 2.3, 2.5（Task 2.4 为软依赖，watcher 不可用时优雅降级）

**目标**：
- 实现 `backend/api.py`
- 暴露前端调用的 API 方法
- **重要**：watcher 为可选增强功能，启动失败时应继续运行（仅禁用自动更新）

**验收标准**：
- [ ] `get_categories()`: 获取所有类别
- [ ] `get_files(category)`: 获取指定类别的文件列表
- [ ] `search_files(query)`: 搜索文件
- [ ] `refresh_files()`: 刷新文件索引
- [ ] `add_category(name, path)`: 添加类别
- [ ] `remove_category(name)`: 删除类别
- [ ] `get_system_status()`: 获取系统状态（watcher 是否可用）
- [ ] 统一错误响应格式
- [ ] 优雅降级：watcher 启动失败时继续运行
- [ ] 通过 mypy 类型检查

**TDD 步骤**：
1. **RED**: 编写 `tests/test_api.py`
   - `test_get_categories()`
   - `test_get_files()`
   - `test_search_files()`
   - `test_refresh_files()`
   - `test_add_category()`
   - `test_remove_category()`
   - `test_get_system_status()`
   - `test_api_error_handling()`
   - `test_watcher_graceful_degradation()`
2. **GREEN**: 实现 `api.py` 的 `API` 类
3. **REFACTOR**: 优化 API 响应格式
4. **COVERAGE**: 确认 80%+ 覆盖

---

### Task 2.7: 实现应用入口
**优先级**: P0
**预计时间**: 30min
**依赖**: Task 2.6

**目标**：
- 实现 `backend/main.py`
- 初始化 PyWebView 窗口

**验收标准**：
- [ ] 创建 PyWebView 窗口
- [ ] 加载前端页面（`frontend/dist/index.html`）
- [ ] 暴露 API 实例
- [ ] 配置窗口属性（标题、大小、图标）

**TDD 步骤**：
- 无需单元测试（集成测试覆盖）

---

## 阶段 3: 前端开发

### Task 3.1: 实现 API 调用封装
**优先级**: P0
**预计时间**: 30min
**依赖**: Task 2.6

**目标**：
- 实现 `frontend/src/hooks/useApi.js`
- 封装 `window.pywebview.api` 调用

**验收标准**：
- [ ] 封装所有后端 API 方法
- [ ] 统一错误处理
- [ ] 返回 Promise

**TDD 步骤**：
- 前端测试可选（手动测试）

---

### Task 3.2: 实现侧边栏组件
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 3.1

**目标**：
- 实现 `frontend/src/components/Sidebar.jsx`
- 显示类别列表

**验收标准**：
- [ ] 显示所有类别
- [ ] 高亮当前选中类别
- [ ] 点击切换类别
- [ ] 显示类别文件数量

**TDD 步骤**：
- 前端测试可选（手动测试）

---

### Task 3.3: 实现搜索栏组件
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 3.1

**目标**：
- 实现 `frontend/src/components/SearchBar.jsx`
- 支持搜索输入和防抖

**验收标准**：
- [ ] 搜索输入框
- [ ] 300ms 防抖
- [ ] 刷新按钮
- [ ] 加载状态显示

**TDD 步骤**：
- 前端测试可选（手动测试）

---

### Task 3.4: 实现文件列表组件
**优先级**: P0
**预计时间**: 1.5h
**依赖**: Task 3.1

**目标**：
- 实现 `frontend/src/components/FileList.jsx`
- 显示文件表格

**验收标准**：
- [ ] 表格显示：文件名、路径、大小、修改时间
- [ ] 支持排序（按名称、大小、时间）
- [ ] 空状态提示

**TDD 步骤**：
- 前端测试可选（手动测试）

---

### Task 3.5: 实现类别管理组件
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 3.1

**目标**：
- 实现 `frontend/src/components/CategoryManager.jsx`
- 提供类别的增删改界面

**验收标准**：
- [ ] 对话框显示所有类别及其路径
- [ ] 添加新类别（输入名称和路径）
- [ ] 删除现有类别（带确认提示）
- [ ] 表单验证（名称和路径不能为空）
- [ ] 调用 `add_category()` 和 `remove_category()` API
- [ ] 操作成功后刷新类别列表

**TDD 步骤**：
- 前端测试可选（手动测试）

---

### Task 3.6: 实现根组件
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 3.2, 3.3, 3.4, 3.5

**目标**：
- 实现 `frontend/src/App.jsx`
- 组合所有组件

**验收标准**：
- [ ] 布局：侧边栏 + 主区域
- [ ] 全局状态管理（categories, files, selectedCategory）
- [ ] 集成 CategoryManager 组件（通过按钮触发）
- [ ] 错误边界处理
- [ ] 加载状态显示

**TDD 步骤**：
- 前端测试可选（手动测试）

---

## 阶段 4: 集成与测试

### Task 4.1: 集成测试
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 2.7, 3.6

**目标**：
- 编写集成测试
- 测试前后端通信

**验收标准**：
- [ ] 测试 API 调用流程
- [ ] 测试错误处理
- [ ] 测试数据格式

**TDD 步骤**：
1. 编写 `tests/test_integration.py`
2. 运行测试确认通过

---

### Task 4.2: 性能测试
**优先级**: P1
**预计时间**: 1h
**依赖**: Task 4.1

**目标**：
- 验证性能指标
- 优化性能瓶颈

**验收标准**：
- [ ] 扫描 1000 文件 < 2s
- [ ] 搜索响应 < 1s
- [ ] 内存占用 < 200MB（使用 psutil 测量）
- [ ] CPU 空闲时 < 5%（使用 psutil 测量）

**TDD 步骤**：
1. 编写性能测试脚本（`tests/test_performance.py`）
   - 使用 `time.perf_counter()` 测量扫描和搜索时间
   - 使用 `psutil.Process().memory_info().rss` 测量内存
   - 使用 `psutil.cpu_percent()` 测量 CPU 占用
2. 运行测试并记录结果
3. 优化性能瓶颈

---

### Task 4.3: E2E 测试
**优先级**: P2
**预计时间**: 1h
**依赖**: Task 4.1

**目标**：
- 编写端到端测试
- 测试完整用户流程

**验收标准**：
- [ ] 测试应用启动
- [ ] 测试文件列表显示
- [ ] 测试搜索功能
- [ ] 测试刷新功能

**TDD 步骤**：
1. 编写 E2E 测试脚本（可选）
2. 手动测试所有功能

---

## 阶段 5: 打包与部署

### Task 5.1: 配置打包
**优先级**: P0
**预计时间**: 1h
**依赖**: Task 4.3

**目标**：
- 配置 PyInstaller
- 打包成 macOS .app

**验收标准**：
- [ ] 创建 `build.spec` 文件
- [ ] 配置打包参数（--onefile, --windowed）
- [ ] 包含前端构建产物
- [ ] 添加应用图标

**TDD 步骤**：
- 无需测试（手动验证）

---

### Task 5.2: 测试打包应用
**优先级**: P0
**预计时间**: 30min
**依赖**: Task 5.1

**目标**：
- 测试打包后的应用
- 验证功能完整性

**验收标准**：
- [ ] .app 可以独立运行
- [ ] 所有功能正常
- [ ] 无报错和崩溃

**TDD 步骤**：
- 手动测试所有功能

---

## 任务优先级说明

- **P0**: 核心功能，必须完成
- **P1**: 重要功能，建议完成
- **P2**: 可选功能，时间允许时完成

## 预计总时间

- 阶段 1: 1.3h
- 阶段 2: 8h
- 阶段 3: 6h（Task 3.5 类别管理 + Task 3.6 根组件）
- 阶段 4: 3h
- 阶段 5: 1.5h

**总计**: ~19.8h（约 2-3 个工作日）

---

**任务清单制定人**: AI Assistant
**审核人**: zhuyu
**开始日期**: 2026-03-09
