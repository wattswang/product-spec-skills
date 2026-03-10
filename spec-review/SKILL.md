---
name: spec-review
description: "评审产品 Spec/PRD 的质量，输出多维度评分和改进建议。支持本地文件、Obsidian、Notion、Coding 链接、自然语言描述、图片等多源输入。触发词：评审 spec、review prd、评估需求、spec 评审。"
argument-hint: "<spec来源：文件路径 | Obsidian路径 | Notion URL | Coding链接 | 直接描述>"
user-invocable: true
---

# 产品 Spec 评审

评审产品需求文档（Spec/PRD）的质量，从完整性、可验证性、技术可行性、风险识别四个维度打分（A-F），输出结构化评审报告到 Obsidian。

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

### Step 1: 解析输入

根据输入类型获取 Spec 内容：

- **本地文件**：使用 `Read` 工具读取
- **Obsidian 笔记**：使用 `ToolSearch` 查找 `obsidian_read_note`，然后读取笔记内容
- **Notion 页面**：使用 `ToolSearch` 查找 `notion-fetch`，然后获取页面内容
- **Coding 链接**：从 URL 中提取 `project_name` 和 `issue_code`，使用 `ToolSearch` 查找 `coding` 相关工具，然后调用 `get_defect` 获取详情
- **图片**：使用 `Read` 工具查看图片内容（Claude 支持多模态）
- **Figma URL**：使用 `WebFetch` 获取设计信息
- **直接描述**：直接使用用户输入的文本作为 Spec 内容

### Step 2: 30秒门控

在深入评审前，先快速回答三个关键问题并展示给用户：

1. **KILL** — 这个功能应该存在吗？是否有更好的替代方案？
2. **SKIP** — 能否推迟3个月？推迟的风险是什么？
3. **SHRINK** — 最小可行范围（MVP）是什么？可以砍掉哪些内容？

如果任一答案建议 Kill 或 Skip，提醒用户并询问是否继续评审。

### Step 3: 多维度评审

对 Spec 进行四个维度的评审，每个维度打分 A-F：

#### 3.1 完整性（Completeness）

检查以下要素是否齐全：
- [ ] 问题陈述 / 背景
- [ ] 目标用户 / 用户画像
- [ ] 用户故事（As a...I want...so that...）
- [ ] 功能需求（含优先级）
- [ ] 非功能需求（性能、安全、可用性）
- [ ] 验收标准
- [ ] Non-Goals（明确不做什么）
- [ ] 技术约束 / 依赖
- [ ] 里程碑 / 时间线
- [ ] 成功指标（KPI）

#### 3.2 可验证性（Verifiability）

检查需求是否可度量：

```diff
# 模糊（BAD）
- 系统响应要快
- 界面要美观易用
- 支持大量用户

# 具体（GOOD）
+ 搜索接口 P95 响应时间 < 200ms
+ 遵循 Material Design 3 规范，Lighthouse 可访问性评分 >= 95
+ 支持 10,000 并发用户，单表数据量 1000万
```

标记所有模糊需求并给出具体化建议。

#### 3.3 技术可行性（Feasibility）

评估：
- 技术方案是否合理
- 外部依赖是否明确且可控
- 是否存在技术债务风险
- 数据模型是否清晰
- 接口定义是否完整

#### 3.4 风险识别（Risk Assessment）

识别：
- 遗漏的边界场景
- 安全隐患（认证、授权、数据泄露）
- 性能瓶颈
- 第三方依赖风险
- 需求冲突或歧义

### Step 4: 生成评审报告

汇总评审结果，计算综合评分。

**综合评分规则**：
- A（优秀）：所有维度 >= B，无 CRITICAL 问题
- B（良好）：无 D/F 维度，CRITICAL 问题 <= 1
- C（合格）：无 F 维度，CRITICAL 问题 <= 3
- D（需改进）：存在 F 维度或 CRITICAL 问题 > 3
- F（不合格）：多个 F 维度或存在根本性缺陷

### Step 5: 输出到 Obsidian

使用 `ToolSearch` 查找 `obsidian_update_note` 工具，将评审报告写入 Obsidian。

**输出路径**：`Specs/Reviews/{spec-name}-review.md`

**输出模板**：

```markdown
---
tags: [spec-review, {project-name}]
date: {YYYY-MM-DD}
spec-source: {来源类型和路径}
overall-grade: {A-F}
---

# {Spec名称} — 评审报告

## 概要

| 维度 | 评分 | 关键发现 |
|------|------|---------|
| 完整性 | {A-F} | {一句话总结} |
| 可验证性 | {A-F} | {一句话总结} |
| 技术可行性 | {A-F} | {一句话总结} |
| 风险识别 | {A-F} | {一句话总结} |
| **综合** | **{A-F}** | |

## 30秒门控

- **KILL**: {结论}
- **SKIP**: {结论}
- **SHRINK**: {MVP建议}

## 问题清单

### CRITICAL（必须修复）
1. {问题描述} → {改进建议}

### HIGH（强烈建议修复）
1. {问题描述} → {改进建议}

### MEDIUM（建议改进）
1. {问题描述} → {改进建议}

## 模糊需求具体化建议

| 原始需求 | 建议改写 |
|---------|---------|
| {模糊描述} | {具体可度量的描述} |

## 改进建议

1. {建议}
2. {建议}
```

## Rules

- **不问不审**：如果 Spec 内容不足以评审，先向用户询问补充信息
- **不做假设**：对不确定的内容标记为 `[待确认]`，不自行推断
- **客观评分**：严格按照检查项打分，不因内容量大就给高分
- **中文输出**：评审报告使用中文，技术术语保留英文
- **保存前确认**：在写入 Obsidian 前，先在对话中展示评审摘要
