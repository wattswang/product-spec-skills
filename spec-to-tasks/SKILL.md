---
name: spec-to-tasks
description: "将产品 Spec/PRD 拆解为可执行的开发任务，支持 Epic → Story → Task 三层分解，输出到 Obsidian 并可同步创建 Coding 平台缺陷。触发词：拆解任务、spec 转任务、需求拆分、task breakdown。"
argument-hint: "<spec来源：文件路径 | Obsidian路径 | Notion URL | Coding链接 | 直接描述>"
user-invocable: true
---

# Spec 转开发任务

将产品需求文档（Spec/PRD）拆解为结构化的开发任务。采用 Epic → Story → Task 三层分解，每个 Task 包含具体的实现细节、验收标准和工作量估算。结果输出到 Obsidian，并可选同步到 Coding 平台。

## 输入

`$ARGUMENTS` 支持以下来源（自动识别）：

| 输入类型 | 示例 | 识别方式 |
|---------|------|---------|
| 本地文件 | `/path/to/spec.md` 或 `D:\docs\spec.md` | 包含路径分隔符且为文件扩展名 |
| Obsidian 笔记 | `obsidian://Specs/my-feature` 或 `ob:Specs/my-feature` | 以 `obsidian://` 或 `ob:` 开头 |
| Notion 页面 | `https://www.notion.so/...` | 包含 `notion.so` |
| Coding 链接 | `https://xxx.coding.net/p/xxx/defect/123` | 包含 `coding.net` |
| 图片/设计稿 | `/path/to/mockup.png` 或 Figma URL | 图片扩展名或 `figma.com` |
| 直接描述 | 任意自然语言文本 | 以上均不匹配时 |

如果 `$ARGUMENTS` 为空，询问用户提供 Spec 来源。

## 执行步骤

### Step 1: 获取 Spec 内容

根据输入类型获取 Spec 内容（方式同 spec-review 的 Step 1）：

- **本地文件**：使用 `Read` 工具读取
- **Obsidian 笔记**：使用 `ToolSearch` 查找 `obsidian_read_note`，然后读取
- **Notion 页面**：使用 `ToolSearch` 查找 `notion-fetch`，然后获取
- **Coding 链接**：提取 `project_name` 和 `issue_code`，使用 `get_defect` 获取
- **图片**：使用 `Read` 工具查看
- **直接描述**：直接使用文本

### Step 2: Spec 分析与确认

在拆解前，先向用户确认：

1. **目标项目**：这个 Spec 属于哪个 Coding 项目？（使用 `list_projects` 列出可选项目）
2. **目标迭代**：任务归属哪个迭代？（使用 `list_iterations` 列出进行中的迭代）
3. **技术栈**：前端/后端/全栈？使用的框架和语言？
4. **代码仓库**：是否有现有代码库？路径是什么？

如果用户提供了代码仓库路径，使用 `Glob` 和 `Grep` 扫描项目结构，了解现有模块和约定。

### Step 3: 三层拆解

#### 3.1 Epic 级别（EPIC-001）

识别 Spec 中的大功能模块，每个 Epic 包含：
- **编号**：EPIC-001, EPIC-002, ...
- **名称**：简短的模块名
- **目标**：该模块要解决的核心问题
- **验收标准**：模块级别的验收条件

#### 3.2 Story 级别（STORY-001）

将每个 Epic 拆解为用户故事：
- **编号**：STORY-001, STORY-002, ...
- **格式**：`作为 {角色}，我希望 {功能}，以便 {价值}`
- **所属 Epic**：关联的 Epic 编号
- **验收标准**：Story 级别的 Done 定义
- **优先级**：P0（必须）/ P1（重要）/ P2（有则更好）

#### 3.3 Task 级别（TASK-001）

将每个 Story 拆解为原子开发任务：
- **编号**：TASK-001, TASK-002, ...
- **标题**：动词开头的具体描述（实现xxx / 创建xxx / 添加xxx）
- **所属 Story**：关联的 Story 编号
- **类型**：后端 / 前端 / 数据库 / 配置 / 测试
- **详细描述**：
  - 需要修改/创建的文件路径（如有代码库）
  - 涉及的函数/类/接口名称
  - 具体实现要点
- **验收标准**：可直接验证的完成条件
- **Story Points**：1 / 2 / 3 / 5 / 8
- **依赖**：前置任务编号（如 `blocked by TASK-003`）

### Step 4: 依赖分析

分析所有 Task 之间的依赖关系：

1. **构建依赖图**：标记哪些任务必须串行执行
2. **识别并行组**：标记可以同时执行的任务组
3. **关键路径**：标出最长的串行路径，即最短完成时间
4. **输出可视化**：

```
Phase 1 (并行):
  ├── TASK-001: 数据库表设计
  └── TASK-002: API 接口定义

Phase 2 (并行, 依赖 Phase 1):
  ├── TASK-003: 后端实现 [blocked by TASK-001, TASK-002]
  └── TASK-004: 前端页面框架 [blocked by TASK-002]

Phase 3 (串行):
  └── TASK-005: 联调测试 [blocked by TASK-003, TASK-004]
```

### Step 5: 工作量汇总

| 统计项 | 数值 |
|-------|------|
| Epic 数量 | N |
| Story 数量 | N |
| Task 数量 | N |
| 总 Story Points | N |
| 关键路径 Points | N |
| 预计并行执行阶段数 | N |

### Step 6: 输出到 Obsidian

使用 `ToolSearch` 查找 `obsidian_update_note` 工具，将任务分解结果写入 Obsidian。

**输出路径**：`Specs/Tasks/{spec-name}-tasks.md`

**输出模板**：

```markdown
---
tags: [spec-tasks, {project-name}]
date: {YYYY-MM-DD}
spec-source: {来源类型和路径}
total-stories: {N}
total-tasks: {N}
total-points: {N}
---

# {Spec名称} — 任务分解

## 概要

| 统计项 | 数值 |
|-------|------|
| Epic | {N} |
| Story | {N} |
| Task | {N} |
| 总 SP | {N} |

## 依赖图

{Phase 可视化}

---

## EPIC-001: {名称}

> 目标：{目标描述}

### STORY-001: 作为{角色}，我希望{功能} [P0]

#### TASK-001: {任务标题} [后端] [SP: 3]
- **描述**：{详细描述}
- **文件**：`src/main/java/xxx/XxxController.java`
- **验收**：{验收标准}
- **依赖**：无

#### TASK-002: {任务标题} [前端] [SP: 2]
- **描述**：{详细描述}
- **验收**：{验收标准}
- **依赖**：blocked by TASK-001
```

### Step 7: 同步到 Coding（可选）

在 Obsidian 输出完成后，询问用户是否要将任务同步到 Coding 平台。

如果用户确认：

1. 使用 `ToolSearch` 查找 `coding` 相关工具
2. 使用 `list_projects` 确认目标项目
3. 使用 `list_iterations` 确认目标迭代
4. 使用 `list_assignees` 获取可分配的成员列表
5. 询问用户每个 Task 的指派人（可批量指派）
6. 使用 `create_defect` 逐个创建任务，参数映射：
   - `title` ← Task 标题（含编号前缀）
   - `description` ← Task 详细描述 + 验收标准 + 依赖关系
   - `priority` ← 根据 Story 优先级映射（P0→3紧急, P1→2高, P2→1中）
   - `assignee_ids` ← 用户选择的指派人
   - `iteration_id` ← 目标迭代 ID
7. 输出创建结果汇总（成功/失败数量及链接）

## Rules

- **先分析后拆解**：必须先理解 Spec 全貌再开始拆解，不要边读边拆
- **原子化**：每个 Task 应该是一个人在 1-2 天内可以完成的工作量（1-8 SP）
- **不遗漏**：Spec 中的每个需求点都必须映射到至少一个 Task
- **不添加**：不要添加 Spec 中未提及的功能需求（基础设施任务如数据库建表除外）
- **动词开头**：Task 标题必须以动词开头（实现、创建、添加、修改、配置、编写）
- **中文输出**：任务描述使用中文，代码相关术语保留英文
- **创建前确认**：在 Coding 上创建任务前，必须先展示完整任务列表并获得用户确认
- **幂等性**：如果 Coding 上已存在同名任务，跳过创建并提示用户
