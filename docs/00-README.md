# UniComm 项目规划

> UniComm 企业通信平台项目规划仓库

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
- `unicomm-web` - 前端应用（React + TypeScript）
- `unicomm-plugin` - 插件系统（未来）

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

**禁止开发：**
- IM
- Notify
- AI
- 桌面托盘
- 多端同步
- 插件系统

## 文档目录

```
docs/
├── README.md           # 本文件
├── 01-项目结构设计.md   # 项目目录结构方案
├── 02-技术栈方案.md     # 技术栈选型
├── 03-Memo需求文档.md   # Memo 功能需求
├── 04-数据库设计.md     # Memo 数据库设计
├── 05-API设计.md        # Memo API 设计
└── 06-前端目录结构.md   # 前端项目结构
```

## 开发规划

### Phase 1: 规划与设计
- [x] 创建 unicomm-planning 仓库
- [ ] 完成项目结构设计
- [ ] 完成技术栈方案
- [ ] 完成 Memo 需求文档
- [ ] 完成数据库设计
- [ ] 完成 API 设计
- [ ] 完成前端原型

### Phase 2: Memo 开发
- [ ] 搭建 unicomm-server
- [ ] 搭建 unicomm-web
- [ ] 实现 Memo CRUD
- [ ] 实现 Markdown 编辑器
- [ ] 实现分组/标签功能
- [ ] 实现搜索筛选

### Phase 3: 功能扩展
- [ ] Notify 模块
- [ ] IM 模块
- [ ] AI 集成
- [ ] 文件系统
- [ ] 插件系统
- [ ] 多端同步

## License

MIT