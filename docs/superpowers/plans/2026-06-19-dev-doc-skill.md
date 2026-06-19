# dev-doc Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个通用的技术方案写作 skill（`dev-doc`），借鉴 garden-skills 的 harness 工程思想，落地 6 层 harness + 重量分级 + 规范域动态发现。

**Architecture:** 纯 Markdown skill，无 scripts（不跑 PoC，工具栈是 Agent 原生能力）。由 SKILL.md（入口+流程）+ references/（配方/清单/指引）组成。运行时在用户项目生成 `dev-doc/` 目录承载任务产物和规范缓存。

**Tech Stack:** Markdown, YAML frontmatter, mermaid 语法

**Spec:** `docs/superpowers/specs/2026-06-19-dev-doc-skill-design.md`

---

## 项目特殊性说明（影响计划写法）

这个项目**没有可执行代码**，产物全是 Markdown。因此：
- 不能用传统的 TDD 循环（写失败测试 → 实现 → 通过）
- 替换为：**写文件 → 结构自检 → 端到端走查验证**
- "测试通过"等价于"文件结构完整 + 内容覆盖 spec 要求 + 端到端走查逻辑自洽"

每个任务的验证步骤会明确列出"检查什么"。

---

## 文件结构总览

实现完成后，skill 目录长这样：

```
dev-doc/                              ← skill 本体
├── SKILL.md                          ← 入口：6 层机制 + 6 Phase 流程 + 重量分级
├── references/
│   ├── recipes/
│   │   ├── _backend-domain.md        ← 后端领域决策点（共享层）
│   │   ├── generic-medium.md         ← 通用骨架配方
│   │   ├── multi-task.md             ← 多任务场景配方
│   │   └── greenfield.md             ← 新项目配方
│   ├── anti-ai-signs.md              ← 反方案 AI 味清单（10 条）
│   ├── mermaid-checklist.md          ← 图表规范 + 自检清单
│   └── convention-extraction.md      ← 规范域发现指引
├── README.md                         ← 给人看的说明文档
└── manifest.json                     ← skill 元信息（name/version/compat）
```

**文件职责边界**：
- `SKILL.md` —— Agent 激活即读，只放流程和核心约束，**不放细节**（细节都在 references）
- `references/recipes/*` —— Phase 2/4 加载，定义章节结构和领域决策点
- `references/anti-ai-signs.md` —— Phase 3/5 加载，终审对照清单
- `references/mermaid-checklist.md` —— Phase 4 写图表章节时加载
- `references/convention-extraction.md` —— Phase 1 扫描时加载

---

## Task 1: 项目骨架 + manifest

**Files:**
- Create: `manifest.json`
- Create: `README.md`

- [ ] **Step 1: 创建 manifest.json**

```json
{
  "name": "dev-doc",
  "version": "0.1.0",
  "category": "Documentation / Engineering",
  "description": "通用技术方案写作 skill。借鉴 garden-skills 的 harness 工程思想，落地 6 层机制（上下文管理/工具系统/执行编排/状态记忆/评估观测/约束恢复）+ 重量分级（轻/中/重）+ 规范域动态发现。支持需求驱动扫描代码库、3 个 checkpoint、反 AI 味清单。产物为 Markdown 技术方案文档。",
  "homepage": "",
  "compat": [
    "claude-code",
    "codex-cli",
    "gemini-cli",
    "opencode"
  ]
}
```

- [ ] **Step 2: 创建 README.md**

内容要点：
- 一句话定位（通用技术方案写作 skill）
- 解决的痛点（4 条：硬编码/无 checkpoint/不扫库/质量差）
- 核心特性（6 Phase + 重量分级 + 规范域发现 + 反 AI 味清单）
- 目录结构说明（skill 本体 + 用户项目运行时产物）
- 快速开始（如何激活、产物在哪）
- 配方说明（4 个配方各适用什么场景）
- 设计文档链接（指向 specs/）

- [ ] **Step 3: 结构自检**

检查：
- `manifest.json` 的 `name` 字段 = `dev-doc`（和目录名一致）
- `manifest.json` 是合法 JSON（`python3 -c "import json; json.load(open('manifest.json'))"`）
- `README.md` 存在且非空

- [ ] **Step 4: Commit**

```bash
git add manifest.json README.md
git commit -m "feat: add skill manifest and readme skeleton"
```

---

## Task 2: 反 AI 味清单（anti-ai-signs.md）

先做这个是因为它独立、内容明确，是后续配方和 SKILL.md 引用的基础。

**Files:**
- Create: `references/anti-ai-signs.md`

- [ ] **Step 1: 写 anti-ai-signs.md**

内容必须包含 spec 4.2 节的 10 条，分两部分：

**通用部分（所有方案适用，6 条）**：
1. 空话架构 —— 反模式描述 + 正确做法
2. 无 trade-off —— 反模式描述 + 正确做法
3. 放之四海皆准 —— 反模式描述 + 正确做法
4. 无数据量 —— 反模式描述 + 正确做法（强调单次查询命中量）
5. 无风险/回滚 —— 反模式描述 + 正确做法
6. 过度设计 —— 反模式描述 + 正确做法（强调重量分级）

**后端服务特有（4 条）**：
7. DDL 无估算 —— 反模式 + 正确（索引结合数据量和查询模式）
8. 状态机不完整 —— 反模式 + 正确（触发条件 + 副作用 + 守卫条件）
9. 接口契约模糊 —— 反模式 + 正确（错误码/幂等/版本）
10. 测试方案空洞 —— 反模式 + 正确（具体测试类/覆盖场景/数据准备）

每条用统一格式：
```
### N. <名称>
**反模式**：<具体表现>
**正确做法**：<应该怎么写>
**自检问题**：<Agent 终审时问自己的话>
```

文档末尾加"使用说明"：终审时逐条对照，任何一条命中就标出并要求修正。

- [ ] **Step 2: 内容自检**

对照 spec 4.2 节，确认 10 条全覆盖、无遗漏。每条都有"反模式 + 正确做法 + 自检问题"三要素。

- [ ] **Step 3: Commit**

```bash
git add references/anti-ai-signs.md
git commit -m "feat: add anti-ai-signs checklist (10 items)"
```

---

## Task 3: mermaid 图表规范（mermaid-checklist.md）

**Files:**
- Create: `references/mermaid-checklist.md`

- [ ] **Step 1: 写 mermaid-checklist.md**

内容包含：
- **图表类型选择**：架构图用 flowchart/C4Context，时序图用 sequenceDiagram，状态机用 stateDiagram-v2，ER 图用 erDiagram
- **自检清单**（继承初版第 98 行规则）：
  - 节点标签引号匹配（`[xxx"` 是错的）
  - subgraph 名不加多余引号
  - 箭头标签 `|"xxx"|` 内不必要的引号
  - 节点文本中的 `~` 符号会被解析为删除线，用"至"或"—"替代
  - 特殊字符（括号/分号/引号）在节点文本中的转义
- **强制规则**：所有架构图/流程图/关系图必须用 mermaid，禁止 ASCII/text 画图
- **示例**：每种图类型给一个最小正确示例

- [ ] **Step 2: 内容自检**

确认：4 种图类型都有示例、自检清单覆盖初版规则、强制规则明确。

- [ ] **Step 3: Commit**

```bash
git add references/mermaid-checklist.md
git commit -m "feat: add mermaid checklist and diagram conventions"
```

---

## Task 4: 后端领域决策点（_backend-domain.md）

这是共享层，被三个配方引用，必须先于配方完成。

**Files:**
- Create: `references/recipes/_backend-domain.md`

- [ ] **Step 1: 写 _backend-domain.md**

内容（继承初版精华 + spec 第 1 节的设计）：

**文件头**：说明这是共享层，被 generic-medium/multi-task/greenfield 三个配方引用，不独立使用。下划线前缀表示"被引用层"。

**必备决策点**（每个决策点写：什么时候要想 + 想不清楚怎么办）：
- **数据量估算** —— 必须结合查询条件估算单次命中量（不是表总行数）。量级影响：分页策略/批量大小/是否异步。拿不准找用户确认具体数据量。
- **并发与一致性** —— 是否需要锁？乐观锁还是分布式锁？锁的粒度？防超卖/防重复？
- **事务边界** —— 跨表/跨服务操作，事务怎么保证？是否引入消息最终一致？
- **幂等性** —— 接口是否需要幂等？幂等键怎么设计？重试场景？
- **状态设计** —— 用枚举，code 类型与数据库列类型一致。每个状态流转写触发条件 + 副作用。

**必备章节模板**（在通用骨架基础上，后端方案必须有）：
- 数据模型（DDL + 索引 + 约束）—— DDL 必须标注遵循团队规范（Druid 等校验规则，但具体规则走动态发现）
- 接口契约（路径/方法/请求响应/错误码/幂等）
- 状态机（枚举 + 流转图 + 流转条件）
- 改动清单（删/改/新增表格）
- 数据库变更（DDL + 迁移 + 回滚）

**反 AI 味要点**（本领域特有，呼应 anti-ai-signs.md 第 7-10 条）：
- ❌ 只写"使用 Redis 缓存"不写：命中率预估/淘汰策略/key 过期/缓存击穿处理
- ❌ DDL 没有估算单表数据量就加索引
- ❌ 状态机只画图不写流转的触发条件和副作用
- ❌ 接口只写"返回成功/失败"不写错误码和幂等性

**工时评估参考**（继承初版第 82-93 行）：
| 类型 | 工时 | 适用场景 |
| 基础设施 | 2-3h | 模型、实体类、枚举、DTO 一次性搭建 |
| 简单接口 | 1-1.5h | 单表 CRUD、字段修改、状态切换 |
| 中等接口 | 2h | 多步骤校验、批量操作、并发控制（锁）、树结构计算 |
| 复杂接口 | 3-4h | 跨模块改动、复杂业务编排、多表事务 |
| 改动现有方法 | 1h | 在现有方法中追加逻辑 |
| 测试 | 实现工时 ~40% | 单元测试 + 集成测试 |
| 联调 | 3-4h | 和前端联调自测 |

- [ ] **Step 2: 内容自检**

确认：5 个必备决策点都有、必备章节覆盖后端方案核心、反 AI 味要点和 anti-ai-signs.md 第 7-10 条呼应、工时表完整搬自初版。

- [ ] **Step 3: Commit**

```bash
git add references/recipes/_backend-domain.md
git commit -m "feat: add backend domain decision points (shared layer)"
```

---

## Task 5: 通用骨架配方（generic-medium.md）

**Files:**
- Create: `references/recipes/generic-medium.md`

- [ ] **Step 1: 写 generic-medium.md**

内容（spec 1.4 节的 9 节结构 + 引用 _backend-domain.md）：

**适用场景**：新模块、技改、重构、对外提供接口（spec 3.1 节的"中/重"方案默认用此配方）。

**文档结构**（9 节 + 实现速查）：
1. 需求背景 —— 问题描述/现状分析/存在问题/改进必要性
2. 需求说明 —— 核心需求/具体要求/业务场景
3. 技术方案
   - 整体设计思路（架构图用 mermaid）
   - 数据模型（引用 _backend-domain.md 的 DDL 规范）
   - 接口契约（引用 _backend-domain.md）
   - 状态机（引用 _backend-domain.md）
   - 关键流程（核心算法/流程说明）
4. 改动点（表格：删除/改造/新增/接口变更）
5. 数据库变更（表结构/索引/迁移/回滚）
6. 测试方案（引用团队测试规范，按层策略）
7. 上线方案（灰度/应急/回滚；监控指标仅在需求明确或影响重大时写）
8. 风险评估（表格：技术/业务/数据/性能风险 + 缓解措施）
9. 工时评估（按接口粒度，引用 _backend-domain.md 工时表）
10. 实现速查（索引：DDL 在 3.2、接口在 3.3、状态机在 3.4、改动在 4、工时在 9）

**每个章节的写作指引**：用 1-2 句话说明"这章该写什么、不该写什么"。

**章节深浅表**（引用 spec 3.4 节的重量分级映射，标注深写/正常/薄写/跳过的具体含义）。

- [ ] **Step 2: 内容自检**

确认：9 节结构完整、实现速查节正确索引其他章节、引用了 _backend-domain.md、章节深浅表存在。

- [ ] **Step 3: Commit**

```bash
git add references/recipes/generic-medium.md
git commit -m "feat: add generic-medium recipe (9-section skeleton)"
```

---

## Task 6: 多任务场景配方（multi-task.md）

**Files:**
- Create: `references/recipes/multi-task.md`

- [ ] **Step 1: 写 multi-task.md**

**适用场景**：多个小优化合在一个需求里。

**核心差异**（和 generic-medium 的区别）：章节按子任务重组，不是按技术维度。每个子任务是一个 mini 方案。

**文档结构**：
1. 需求背景（整体说明，为什么把这些小优化合在一起）
2. 子任务清单（表格：子任务名/影响范围/预估工时/优先级）
3. 逐个子任务：
   - 子任务 N
     - 改动描述
     - 数据模型/接口契约（如有，引用 _backend-domain.md）
     - 改动点
     - 测试要点
4. 整体数据库变更（汇总所有子任务的 DDL）
5. 整体上线方案（是否一次性上线、依赖关系、回滚）
6. 整体风险评估（子任务间的相互影响）
7. 整体工时评估（引用 _backend-domain.md 工时表，汇总）

**关键约束**：
- 子任务之间如果有依赖/冲突，必须在"整体风险评估"里点明
- 每个子任务都要能独立回滚（不能因为一个子任务失败导致整体回滚）

- [ ] **Step 2: 内容自检**

确认：章节按子任务重组（不是 9 节平铺）、引用 _backend-domain.md、子任务间依赖/回滚约束明确。

- [ ] **Step 3: Commit**

```bash
git add references/recipes/multi-task.md
git commit -m "feat: add multi-task recipe (subtask-organized structure)"
```

---

## Task 7: 新项目配方（greenfield.md）

**Files:**
- Create: `references/recipes/greenfield.md`

- [ ] **Step 1: 写 greenfield.md**

**适用场景**：新项目（重量判定为"重"）。

**核心差异**（和 generic-medium 的区别）：无"改动点"章节（全是新建），重点在架构选型和里程碑规划。

**文档结构**：
1. 项目背景与目标（业务背景/项目定位/成功标准）
2. 架构设计
   - 整体架构（mermaid 架构图，必须有）
   - 技术选型（每个选型：选项对比 + 选择理由 + trade-off）
   - 模块职责划分
   - 数据流（mermaid 时序图/数据流图）
3. 工程结构（包结构/分层规范/命名规范 —— 引用动态发现的 conventions/project-structure）
4. 数据模型（核心表设计，引用 _backend-domain.md）
5. 接口契约（核心接口，引用 _backend-domain.md）
6. 部署架构（环境规划/依赖中间件/容量预估）
7. 里程碑规划（分阶段交付计划，每阶段产出物）
8. 风险评估（技术风险/依赖风险/容量风险）
9. 工时评估（按里程碑，引用 _backend-domain.md 工时表）

**关键约束**：
- "技术选型"每项必须有选项对比 + trade-off（对应 anti-ai-signs.md 第 2 条）
- 容量预估必须有数据依据（对应 anti-ai-signs.md 第 4 条）
- 无"改动点"和"数据库变更（迁移/回滚）"章节（新建项目）

- [ ] **Step 2: 内容自检**

确认：无"改动点"章节、有里程碑规划、技术选型要求 trade-off、引用 _backend-domain.md。

- [ ] **Step 3: Commit**

```bash
git add references/recipes/greenfield.md
git commit -m "feat: add greenfield recipe (new project, milestone-driven)"
```

---

## Task 8: 规范域发现指引（convention-extraction.md）

**Files:**
- Create: `references/convention-extraction.md`

- [ ] **Step 1: 写 convention-extraction.md**

内容（spec 2.4 节）：

**目的**：Phase 1 阶段 B 深度分析代码时，教 Agent 怎么从代码里发现团队工程约定。

**流程**：
1. 对每个被读的代码区域，提取"这里体现了什么工程约定"
2. 聚类成规范域（不预设清单）
3. 写入 `dev-doc/conventions/<domain>.md`（带时间戳 + 来源文件 + 置信度）
4. 集中展示给用户确认/修正

**规范域文件的精确模板**（spec 2.4 节的结构）：
```
# 规范域：<domain 名称>
## 提取时间
## 来源文件
## 推断的约定
## 置信度（可信 / 待确认）
## 用户确认（已确认 / 已修正）
```

**置信度规则**：
- 可信：多个文件一致体现
- 待确认：单文件/少量推测

**常见规范域提示**（只是提示，实际域由扫描结果决定）：
命名规范 / 分层架构 / 接口规范 / SQL 与数据模型 / 日志规范 / 异常处理 / 配置管理

**示范样例**（从初版 Druid 规则改编，告诉 Agent "团队约定长这样"）：
展示一个完整的 `sql.md` 规范域文件示例，内容是 Druid 解析器校验规则（主键格式/PRIMARY KEY 独立声明/int 带长度/索引名 ≤30 字符），让 Agent 知道提取出来的规范该详尽到什么程度。

**关键纪律**：
- 提取的是"现状推测"，不是真规范 —— 必须用户确认
- 不越界提取（只提阶段 A 圈定范围内的约定）
- 规范域之间不重叠（聚类时要合并相似的）

- [ ] **Step 2: 内容自检**

确认：流程清晰、规范域文件模板完整、置信度两档、示范样例具体（Druid 规则）、纪律明确。

- [ ] **Step 3: Commit**

```bash
git add references/convention-extraction.md
git commit -m "feat: add convention-extraction guide with Druid example"
```

---

## Task 9: SKILL.md —— Phase 0 + 前置 + 目录约定

SKILL.md 是核心，内容多，拆成 3 个任务（Task 9-11）分批写、分批 commit。

**Files:**
- Create: `SKILL.md`

- [ ] **Step 1: 写 SKILL.md 的 frontmatter + 开篇 + 前置步骤 + Phase 0**

**frontmatter**：
```yaml
---
name: dev-doc
description: 通用技术方案写作 skill。按 6 Phase 流程生成技术方案文档：需求归一化 → 需求驱动扫描代码库 + 规范域动态发现 → 提纲 checkpoint → 核心设计 checkpoint（轻方案跳过）→ 展开章节 → 终审 checkpoint。支持重量分级（轻/中/重）自适应 checkpoint 数和章节深浅。产物为 Markdown 文档。用户要"写技术方案/技术设计/方案文档"时使用。
---
```

**开篇**：
- 一句话定位
- 核心机制概览（6 Phase + 重量分级 + 规范域发现）
- **`.dev-doc` 副作用告知**：会创建 `dev-doc/` 目录，建议团队共享（进 git）

**目录约定**（spec 1.2 节）：
```
dev-doc/
├── conventions/                  ← 项目级，跨任务累积
└── tasks/<feature>/              ← 需求级，按任务隔离
    ├── requirement.md
    ├── plan.md
    └── <feature>技术方案.md
```

**前置步骤：确定任务**：
- 用户提供 feature 名
- 检查 `dev-doc/tasks/<feature>/` 是否存在
- 不存在 → 创建新任务；存在 → 询问继续还是新建

**Phase 0：需求归一化**：
- 写 `requirement.md`（结构见 spec 2.1 节：需求来源/需求复述/业务流程/待确认项/数据量线索）
- 需求质量自检（4 项，spec 2.2 节）
- 低质量 → 触发需求对齐节点（mermaid 流程图 + 待确认项 → 用户确认）
- 高质量 → 直接进 Phase 1

- [ ] **Step 2: 内容自检**

确认：frontmatter 的 name = dev-doc、副作用告知明确、目录约定和 spec 1.2 一致、Phase 0 流程和 spec 2.1/2.2 一致。

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "feat: add SKILL.md frontmatter, preamble, phase 0 (requirement normalization)"
```

---

## Task 10: SKILL.md —— Phase 1 + Phase 2

**Files:**
- Modify: `SKILL.md`（追加 Phase 1 和 Phase 2）

- [ ] **Step 1: 追加 Phase 1**

**Phase 1：需求驱动扫描 + 规范域动态发现**

两阶段扫描（spec 2.3 节）：
- 阶段 A：边界确定（轻量定位）—— 从 requirement.md 提取"要碰什么"，用 grep/glob 只定位不深读，产出"影响范围"写入 plan.md
- 阶段 B：深度分析（只读边界内）—— 严禁越界

**核心纪律**（写进 SKILL.md，强调）：
- 阶段 A 必须先于阶段 B
- 阶段 B 严禁越界
- 范围确认搭车 Phase 2 提纲 checkpoint，不加独立 checkpoint

规范域发现（spec 2.4 节）：
- 加载 `references/convention-extraction.md` 学习发现方法
- 对阶段 B 读到的代码提取工程约定 → 聚类成规范域 → 写入 `dev-doc/conventions/`
- 集中展示给用户确认/修正（置信度两档：可信/待确认）

- [ ] **Step 2: 追加 Phase 2**

**Phase 2：提纲 checkpoint**

**判定重量**（spec 3.1 节的 5 维度表 + 判定规则）：
- 触碰模块数 / 新建基础设施 / 数据模型变更 / 接口契约变更 / 影响半径
- 任一"重"→ 重；否则任一"中"→ 中；全"轻"→ 轻
- 依据写入 plan.md 的"重量判定依据"

**选择配方**（根据需求类型）：
- 新模块/技改/重构/对外接口 → generic-medium
- 多个小优化合一个需求 → multi-task
- 新项目 → greenfield

**提纲 checkpoint 内容**（用 AskUserQuestion，每项独立确认，禁止打包 yes/no）：
- 影响范围（搭车确认）
- 重量判定（轻/中/重 + 依据 + 后续流程）
- 章节规划（哪些深写/薄写，spec 3.4 节的深浅表）
- 关键设计方向（1-2 句话，不展开）

**重量分支**：
- 轻 → 跳过 Phase 3，直接 Phase 4
- 中 → Phase 3 验"改动核心设计"
- 重 → Phase 3 验"全局架构骨架"

**plan.md 完整结构**（spec 2.6 节）：在此阶段产出 plan.md，包含任务元信息/影响范围/现状分析/重量判定/章节规划+规范域关联/关键决策（Phase 3 补）。

- [ ] **Step 3: 内容自检**

确认：两阶段扫描纪律明确、规范域发现引用了 convention-extraction.md、重量判定 5 维度表完整、提纲 checkpoint 的 4 项确认内容明确、配方选择逻辑清晰。

- [ ] **Step 4: Commit**

```bash
git add SKILL.md
git commit -m "feat: add phase 1 (scanning + convention discovery) and phase 2 (outline checkpoint)"
```

---

## Task 11: SKILL.md —— Phase 3/4/5 + 约束 + reference 加载总表

**Files:**
- Modify: `SKILL.md`（追加 Phase 3-5 和全局约束）

- [ ] **Step 1: 追加 Phase 3**

**Phase 3：核心设计/架构 checkpoint（轻方案跳过）**

加载 `references/anti-ai-signs.md`。

中方案验收内容：
- 改动的核心接口/数据模型/状态机 + 关键决策
- 每个关键决策带"考虑过 X，因为 Y 没选"

重方案验收内容：
- 全局架构图（mermaid）+ 模块职责 + 核心选型理由

SubAgent 三视角评审（可行性/风险/成本）：
- 加载 `references/anti-ai-signs.md` 作为评审依据
- 评审结果**必须以问题清单形式**给出（不是通过/不通过）
- 对话里给用户，**不落盘**

用户确认后，决策写入 plan.md 的"关键决策"区块。

- [ ] **Step 2: 追加 Phase 4**

**Phase 4：展开章节**

核心机制：**按章节按域加载**。
- 每写一章，先查 plan.md 的"章节规划 + 规范域关联"表
- 加载对应的 `dev-doc/conventions/<domain>.md`
- 写图表章节时加载 `references/mermaid-checklist.md`

写作约束：
- 所有章节必须回看 plan.md 的"关键决策"区块，保证不漂移
- 章节深浅按 plan.md 的章节规划表（深写/正常/薄写/跳过）
- 数据模型/接口契约引用 `_backend-domain.md` 的规范

- [ ] **Step 3: 追加 Phase 5**

**Phase 5：终审 checkpoint**

SubAgent 多视角终审（逻辑自洽/风险完整/可实现）：
- 加载 `references/anti-ai-signs.md`
- 评审结果以问题清单形式给，不落盘

终审对照：
- 反 AI 味清单逐条核对
- requirement.md 的待确认项是否都已闭环
- 数据量线索是否都体现在方案里

- [ ] **Step 4: 追加全局约束**

**最小切片修复**（spec 4.3 节）：
- 任何 checkpoint 用户提修改或终审暴露问题 → 只改出问题的部分
- 修复前**必须先声明影响范围**（"影响 X、Y 章节，Z 不动"）
- 禁止静默大面积重写

**reference 加载总表**（spec 2.5 节）：一张表说明每个 Phase 加载哪个 reference。

**常见规范域提示**（spec 2.4 节）：命名/分层/接口/SQL/日志/异常/配置，明确这只是提示。

- [ ] **Step 5: 内容自检**

确认：Phase 3 的中/重验收内容区分清楚、SubAgent 评审输出问题清单且不落盘、Phase 4 的按域加载机制清晰、Phase 5 的终审对照完整、最小切片修复纪律明确、reference 加载总表存在。

- [ ] **Step 6: Commit**

```bash
git add SKILL.md
git commit -m "feat: add phase 3-5, minimal-slice recovery, reference loading table"
```

---

## Task 12: 端到端走查验证

不写新文件，用一个虚拟需求走查整个 SKILL.md，验证流程逻辑自洽、无遗漏、无矛盾。

**Files:**
- 无（验证任务）

- [ ] **Step 1: 虚拟需求走查 —— 轻方案**

虚拟需求："给 voucher 表加一个 expire_time 字段，并在查询接口返回"。

走查：
- Phase 0：需求清晰（加字段 + 查询返回），高质量，跳过对齐节点
- Phase 1：扫描 voucher 相关代码，发现 SQL 规范（Druid 规则）
- Phase 2：重量判定 = 轻（触碰 1 模块、不新建基础设施、加字段、改现有接口、单点影响）。提纲：深写数据模型/接口/改动/工时，薄写其他
- Phase 3：跳过（轻方案）
- Phase 4：按 plan.md 章节表，加载 sql.md 规范，写章节
- Phase 5：终审，对照反 AI 味清单

**验证点**：流程能跑通、轻方案确实跳过 Phase 3、checkpoint 数 = 2。

- [ ] **Step 2: 虚拟需求走查 —— 重方案**

虚拟需求："新建一个优惠券发放子系统，支持规则引擎、库存管理、发放记录"。

走查：
- Phase 0：需求中等清晰，触发对齐节点画业务流程图
- Phase 1：扫描范围较大，发现多个规范域
- Phase 2：重量判定 = 重（触碰多模块、新建基础设施、新表、新接口、跨模块）。选 greenfield 配方
- Phase 3：验全局架构骨架，开 SubAgent 三视角评审
- Phase 4：展开 9 节，按域加载规范
- Phase 5：终审

**验证点**：重方案走 3 个 checkpoint、greenfield 配方适用、Phase 3 验全局架构。

- [ ] **Step 3: 检查 SKILL.md 与所有 reference 的引用一致性**

逐项检查：
- SKILL.md 提到的 reference 文件都存在（recipes/_backend-domain.md / generic-medium.md / multi-task.md / greenfield.md / anti-ai-signs.md / mermaid-checklist.md / convention-extraction.md）
- 配方里引用 _backend-domain.md 的地方，_backend-domain.md 确实有对应内容
- anti-ai-signs.md 的 10 条在 SKILL.md 的 Phase 3/5 都被引用
- plan.md / requirement.md 的结构在 SKILL.md 和 references 里描述一致

- [ ] **Step 4: 修复发现的问题（如有）**

走查中发现的任何矛盾、遗漏、引用断裂，就地修复。

- [ ] **Step 5: Commit（如有修复）**

```bash
git add -A
git commit -m "fix: end-to-end walkthrough fixes"
```

（如无修复，跳过此步）

---

## Task 13: README 完善 + 最终结构验证

**Files:**
- Modify: `README.md`

- [ ] **Step 1: 完善 README.md**

Task 1 写的是骨架，现在补全：
- 快速开始（激活方式、产物路径）
- 6 Phase 流程图（mermaid，给人看的高层概览）
- 重量分级说明（轻/中/重对应什么需求）
- 配方选择指南（什么场景用哪个配方）
- 运行时目录结构（dev-doc/conventions + dev-doc/tasks）
- 设计文档链接
- 致谢（garden-skills）

- [ ] **Step 2: 最终目录结构验证**

检查实际产物：
```
dev-doc/
├── SKILL.md                          ✓
├── manifest.json                     ✓
├── README.md                         ✓
└── references/
    ├── recipes/
    │   ├── _backend-domain.md        ✓
    │   ├── generic-medium.md         ✓
    │   ├── multi-task.md             ✓
    │   └── greenfield.md             ✓
    ├── anti-ai-signs.md              ✓
    ├── mermaid-checklist.md          ✓
    └── convention-extraction.md      ✓
```

- [ ] **Step 3: manifest.json 与目录一致性验证**

`python3 -c "import json; m=json.load(open('manifest.json')); print(m['name'])"` → 应输出 `dev-doc`

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: complete README with quickstart and flow diagram"
```

---

## Self-Review

### Spec coverage 检查

| Spec 要求 | 对应 Task |
|---|---|
| manifest + README 骨架 | Task 1 |
| 反 AI 味清单 10 条 | Task 2 |
| mermaid 规范 | Task 3 |
| 后端领域决策点（共享层）| Task 4 |
| 通用骨架配方（9 节）| Task 5 |
| 多任务配方 | Task 6 |
| 新项目配方 | Task 7 |
| 规范域发现指引 | Task 8 |
| SKILL.md Phase 0 + 目录约定 | Task 9 |
| SKILL.md Phase 1 + 2 | Task 10 |
| SKILL.md Phase 3/4/5 + 约束 | Task 11 |
| 端到端验证 | Task 12 |
| README 完善 | Task 13 |

✓ 所有 spec 要求都有对应 Task。

### Placeholder scan

✓ 无 TBD/TODO。每个步骤都写了具体内容（文件结构、章节要点、验证点）。

### Type consistency

✓ 检查关键命名一致性：
- 目录名 `dev-doc` = manifest name = SKILL.md frontmatter name
- 规范域缓存路径 `dev-doc/conventions/` 在 SKILL.md/convention-extraction.md/spec 中一致
- 任务产物路径 `dev-doc/tasks/<feature>/` 一致
- 配方文件名在 SKILL.md 和实际创建的文件一致

---

## 执行选择

Plan complete and saved to `docs/superpowers/plans/2026-06-19-dev-doc-skill.md`.
