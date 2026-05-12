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

- 📝 **Memo** - 个人备忘录系统
- 💬 **IM** - 即时通讯
- 🔔 **Notify** - 通知服务
- 🤖 **AI** - AI 助手集成
- 📁 **File** - 文件系统
- 🔌 **Plugin** - 插件系统

## 项目结构

### 规划仓库（当前）
- `unicomm-planning` - 项目规划、需求文档、技术方案

### 正式项目（后续）
- `unicomm-server` - 后端服务（Spring Boot 3 + Java 21）
- `unicomm-desktop` - 桌面应用（Tauri 2 + React + TypeScript）
- `unicomm-plugin` - 插件系统（未来）

## 技术栈

### 桌面端（unicomm-desktop）
- **容器**：Tauri 2
- **UI**：React + TypeScript
- **构建**：Vite
- **状态**：Zustand + TanStack Query
- **样式**：TailwindCSS

### 后端（unicomm-server）
- **框架**：Spring Boot 3 + Java 21
- **ORM**：MyBatis Plus
- **缓存**：Redis
- **实时**：WebSocket
- **认证**：Sa-Token

## 当前阶段

### 第一阶段：Memo 模块

**定位：** 员工个人备忘录系统

**功能范围：**
- 新建/编辑/删除/查看 Memo
- Markdown 内容支持
- 分组管理
- 标签功能
- 收藏/归档/置顶
- 搜索筛选

**桌面端能力：**
- 系统托盘常驻
- WebSocket 实时同步
- 本地缓存支持离线
- 桌面通知

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
├── 06-前端目录结构.md       # 桌面端项目结构
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
- [ ] 搭建 unicomm-desktop 项目骨架
- [ ] 配置 Tauri 2 + React + Vite
- [ ] 实现系统托盘
- [ ] 实现窗口管理
- [ ] 配置 WebSocket 连接

### Phase 3: Memo 开发
- [ ] 搭建 unicomm-server
- [ ] 实现 Memo CRUD API
- [ ] 实现 Memo 前端页面
- [ ] 实现 Markdown 编辑器
- [ ] 实现分组/标签功能
- [ ] 实现搜索筛选

### Phase 4: 功能扩展（未来）
- [ ] Notify 模块
- [ ] IM 模块
- [ ] AI 集成
- [ ] 文件系统
- [ ] 插件系统

## License

MIT
