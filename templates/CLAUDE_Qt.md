# Qt/QWidget 项目开发与交互规范

## 规范执行优先级
- 本文件中的规则专门针对 Qt/QWidget 跨平台开发项目。
- **谋定而后动**：在修改任何代码前，必须先分析项目现有的布局和已有代码风格，保持新老代码风格绝对一致。
- **最小修改原则**：只改动实现功能所需的必要代码行，严禁无故大面积重构或重写。

# 核心技术规范

## 1. C/C++ 内存与 Qt 对象树 (最高优先级)
- **Qt 自动释放机制**：充分利用 Qt 的对象树（Object Tree）父子关系。属于特定 `QWidget` 或 `QObject` 的子控件/子对象，在构造（`new`）时**必须指定 `parent` 指针**，确保父对象析构时自动释放子对象内存。
- **动态悬空指针防护**：对于无法指定 `parent` 或生命周期不确定的 `QObject` 裸指针，必须使用 `QPointer<T>` 进行包裹，防止对象被销毁后留下野指针。
- **非 QObject 资源管理**：不属于 Qt 对象树的纯 C/C++ 资源，必须严格遵守 RAII 原则，优先使用智能指针（`std::unique_ptr` 或 `std::shared_ptr`）进行闭环管理。
- **静态防御**：严禁使用 `strcpy`, `sprintf` 等易导致缓冲区溢出的不安全函数，必须替换为安全版本或优先使用 `QString::asprintf()`。

## 2. 信号与槽 (Signals & Slots)
- **现代语法**：必须使用 C++11 的成员函数指针语法（`connect(sender, &Sender::sig, receiver, &Receiver::slot)`），**坚决禁止**使用旧版的 `SIGNAL()` 和 `SLOT()` 宏，以便在编译期捕获错误。
- **跨线程安全**：多线程间传递信号时，必须明确指定连接类型。默认使用 `Qt::QueuedConnection`（队列连接），若需要同步阻塞则使用 `Qt::BlockingQueuedConnection`，严禁导致死锁。
- **lambda 表达式防护**：在使用连接捕获 lambda 表达式时，必须指定接收者上下文（Context `QObject`），例如 `connect(sender, &Sender::sig, this, [=](){ ... })`。严禁在没有上下文的情况下捕获 `this` 指针，防止对象析构后引发崩溃。

## 3. QWidget 界面、线程解耦与跨平台
- **UI 与逻辑彻底分离**：界面类（继承自 `QWidget/QDialog/QMainWindow`）应当只处理界面展示和简单的输入校验。
- **严禁阻塞主线程**：所有复杂的后台计算、大文件读写、网络请求、串口通信或耗时循环，**必须**移入独立的线程（使用 `QThread`、`QWorker` 模式或 `QtConcurrent`）处理，绝对不允许阻塞主 UI 线程导致界面卡死。
- **跨平台与多分辨率适配**：控件尺寸严禁硬编码固定像素。必须使用 Qt 的布局管理器（`QHBoxLayout`, `QVBoxLayout`, `QGridLayout`）配合 `QSizePolicy` 与 `Spacer` 来实现界面的弹性和自动缩放。
- **国际化 (i18n)**：界面上的所有硬编码中文字符串、提示信息，必须包裹在 `tr()` 函数中（例如 `tr("确认")`），以便后续生成 `.ts` 文件进行多国语言翻译。

# 构建系统规范 (CMake)

- **现代 CMake 风格**：必须使用基于目标（Target-based）的现代语法（如 `target_link_libraries`），严禁使用全局污染宏。
- **Qt 自动化工具**：必须正确开启 Qt 专属的元对象编译器、用户界面编译器和资源编译器开关：
  ```cmake
  set(CMAKE_AUTOMOC ON)
  set(CMAKE_AUTOUIC ON)
  set(CMAKE_AUTORCC ON)
  ```
- **依赖引入**：通过 `find_package(Qt5 COMPONENTS Widgets Core REQUIRED)` 优雅地引入依赖。
- **禁止构建污染**：严禁直接在源码根目录下执行 `cmake .`。必须在 `build/` 目录下进行离线构建（Out-of-source Build）。

# 团队协作与沟通

## 交互习惯
- **语言偏好**：与我（用户）交谈、解释代码、汇报进度时，全程使用清晰、专业的中文。
- **代码与 Git 规范**：代码内的注释、代码本身以及 Git 提交信息（Commit Message），除非特别要求，否则一律使用英文编写。
- **提交规范**：Git Commit 优先使用约定式提交规范（Conventional Commits，例如：`feat: ...`, `fix: ...`）。在提交前，必须确保本地编译和静态检查顺利通过。
