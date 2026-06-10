# UniComm 项目规划

> UniComm 企业级桌面协作应用

## 项目定位

**UniComm 不是普通 Web 管理后台，而是企业级桌面协作应用。**

核心特点：
- 🖥️ 长期运行的后台应用
- 📌 系统托盘常驻
- 🔌 WebSocket 保持连接
- 📴 支持离线工作
- 💾 本地数据缓存
- 🖱️ 桌面原生能力

## 项目概述

UniComm 是面向企业的模块化通信平台，支持：

- 📝 **Memo** - 备忘录与轻量共享系统
- 💬 **IM** - 即时通讯
- 🔔 **Notify** - 通知服务
- 🤖 **AI** - AI 助手集成
- 📁 **File** - 文件系统
- 🔌 **Plugin** - 插件系统

## 项目结构

### 规划仓库（当前）
- `unicomm-planning` - 项目规划、需求文档、技术方案

### 正式项目（后续）
- `unicomm-server` - 后端服务（Spring Boot 4 + Java 21）
- `unicomm-desktop` - 桌面应用（Tauri 2 + React + TypeScript）
- `unicomm-plugin` - 插件系统（未来）

## 技术栈

### 桌面端（unicomm-desktop）
- **容器**：Tauri 2
- **UI**：React + TypeScript
- **构建**：Vite
- **状态**：Zustand + TanStack Query
- **样式**：TailwindCSS
- **字体**：阿里巴巴普惠体 3.0 / Alibaba Sans JP，字体文件内置到桌面端
- **语言**：中文 / 日文

### 后端（unicomm-server）
- **框架**：Spring Boot 4 + Java 21
- **数据访问**：Spring JDBC / JdbcTemplate
- **缓存**：Redis
- **实时**：WebSocket
- **认证**：Sa-Token

## 当前阶段

### 第一阶段：Memo 模块

**定位：** 员工备忘录与轻量共享系统

**功能范围：**
- 新建/编辑/删除/查看 Memo
- 添加相关人，相关人按只读/可编辑权限查看或协作编辑共享 Memo
- 相关人搜索当前使用测试人员数据，未来通过统一人员适配器接入真实公司人员 API
- Milkdown Crepe 可视化 Markdown 编辑与 MD 源码视图
- 分组筛选
- 状态筛选
- 收藏/置顶
- 搜索筛选
- 快速 Memo 小窗口，仅用于快捷新增 Memo

**桌面端能力：**
- 当前 Windows 用户识别，包含 username 和企业 domain
- 后端基于 Sa-Token 进行会话认证，Memo 权限以后端 Token 用户为准
- 测试阶段使用 mock 人员源完成认证与相关人搜索，不接入真实人员 API
- 用户快照、认证审计、设备信任和 Token 刷新已接入；首次设备自动绑定，已绑定用户的新设备验证码测试阶段写入后端日志，邮件发送保留 TODO
- 设备信息识别，Rust 返回字段与前端统一使用 camelCase
- 系统托盘常驻
- 关闭窗口后后台运行
- 主窗口使用应用内自定义标题栏，避免 Windows 原生标题栏重复显示
- 全局快捷键唤出主界面
- 全局快捷键唤出快速 Memo 小窗口
- 设置页支持修改快捷键
- 设置页支持中文/日文切换
- 首次启动根据系统语言自动选择默认界面语言，未支持语言回退中文
- 前端内置阿里巴巴普惠体 3.0 / Alibaba Sans JP 常用 UI 字重
- WebSocket 实时同步（握手校验 Sa-Token，Memo 变更事件按创建者和相关人连接级推送）
- Tauri IPC 使用 `ipc.localhost` 内部通道调用桌面能力，发布前需要配置 CSP 并检查 capabilities
- 本地缓存支持离线（后续增强）
- 桌面通知（未来，不在当前主导航展示）

**禁止开发：**
- IM（未来）
- Notify（未来）
- AI（未来）
- 插件系统（未来）

## 文档目录

```
docs/
├── 00-README.md             # 本文件
├── 01-项目结构设计.md       # 项目目录结构、桌面端能力
├── 02-技术栈方案.md         # 技术栈选型（Tauri 2 + React）
├── 03-Memo需求文档.md       # Memo 功能需求
├── 04-数据库设计.md         # Memo 数据库设计
├── 05-API设计.md            # Memo API 设计
├── 06-桌面端目录结构.md       # 桌面端项目结构
└── 07-Memo原型/            # Memo HTML 原型
    └── index.html
```

## 开发规划

### Phase 1: 规划与设计
- [x] 创建 unicomm-planning 仓库
- [x] 完成项目结构设计（桌面端定位）
- [x] 完成技术栈方案（Tauri 2）
- [x] 完成 Memo 需求文档
- [x] 完成数据库设计
- [x] 完成 API 设计
- [x] 完成前端原型

### Phase 2: Desktop 搭建
- [x] 搭建 unicomm-desktop 项目骨架
- [x] 配置 Tauri 2 + React + Vite
- [x] 实现系统托盘
- [x] 实现窗口管理
- [x] 实现后台运行（关闭窗口隐藏，不退出）
- [x] 实现主窗口应用内自定义标题栏
- [x] 实现主界面全局快捷键
- [x] 实现快速 Memo 全局快捷键
- [x] 实现设置页快捷键配置
- [x] 实现中文/日文语言切换
- [x] 配置并内置阿里巴巴普惠体 3.0 / Alibaba Sans JP 字体文件
- [x] 配置 WebSocket 连接

### Phase 3: Memo 开发
- [x] 搭建 unicomm-server
- [x] 实现 Memo CRUD API
- [x] 实现 Memo 前端页面
- [x] 实现快速 Memo 新增入口
- [x] 接入 Milkdown Crepe 可视化 Markdown 编辑器
- [x] 实现 MD 源码视图切换
- [x] 实现分组筛选
- [x] 实现状态筛选
- [x] 实现创建者、相关人 view/edit 权限校验
- [x] 实现测试人员源下的相关人搜索
- [ ] 完善搜索体验（实时搜索、关键词高亮）
- [x] 抽象人员适配器，为未来真实公司人员 API 接入做准备

### Phase 4: 功能扩展（未来）
- [ ] 接入真实公司人员 API
- [ ] Notify 模块
- [ ] IM 模块
- [ ] AI 集成
- [ ] 文件系统
- [ ] 插件系统

## License

MIT
