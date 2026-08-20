# 评审版技术方案（review doc）Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为 dev-doc skill 增加评审版技术方案能力——主方案交付确认后生成面向技术评审的精简版，主方案变更后自动整篇重生成，任务级可关闭。

**Architecture:** 裁剪规则外置到新 reference `references/review-doc.md`（SKILL.md 只加挂载点不内联规则）。两个挂载点：Phase 5.3 交付 checkpoint 的"评审版"决策项（生成入口）+ 前置步骤「继续」路径的同步检测（变更后立即整篇重生成）。评审版是单向衍生品：只从主方案生成、不可手动编辑。验证走本地 evals 体系（`evals/` 已 gitignore）。

**Tech Stack:** Agent Skill（Markdown prompt 工程，Claude Code / Codex CLI / Gemini CLI / OpenCode 四端兼容，无代码可执行）；本地 eval 回归（fixture 副本 + subagent 跑 + grading.json 机检断言）。

## Global Constraints

- 产物命名固定：`dev-doc/tasks/<feature>/<feature>技术方案-评审版.md`（spec 原文，不得变体）
- 评审版单向衍生：以主方案为准、请勿直接编辑；同步一律**整篇重生成**，禁止章节级切片
- 关闭开关为**任务级**（plan.md 记「评审版：关闭」），不做项目级/全局配置
- 文档中文；路径、文件名、代码块内标识符英文
- commit 遵循 conventional commits，scope 用 `dev-doc`，例：`feat(dev-doc): ...`
- `evals/` 整体已 gitignore——Task 4/5 的产物不 commit、不入库
- spec（需求唯一事实源）：`docs/superpowers/specs/2026-08-20-review-doc-design.md`

---

### Task 1: 裁剪规则文件 `references/review-doc.md`

**Files:**
- Create: `references/review-doc.md`

**Interfaces:**
- Consumes: 主方案章节结构（generic-medium 9 节，见 `references/recipes/generic-medium.md`）
- Produces: 文件 `references/review-doc.md`，被 SKILL.md（Task 2）和 README（Task 3）以相对路径 `references/review-doc.md` 引用；产物文件名 `<feature>技术方案-评审版.md`、plan.md 状态字面量「评审版：未生成 / 已生成 / 关闭」由本任务确立，Task 2/4 沿用

- [ ] **Step 1: 写入完整文件内容**

创建 `references/review-doc.md`，内容如下（全文照写）：

````markdown
# 评审版技术方案生成规则

> **用途**：主方案通过 Phase 5 交付确认后，按本规则从主方案生成面向技术评审的精简版。首次生成与变更后的重新生成都走这份规则。
> **定位**：主方案给编码 AI 看契约，评审版给人讲决策。评审版是**单向衍生品**——只从主方案生成、永不手动编辑，主方案是唯一事实源。

## 产物与命名

- 路径：`dev-doc/tasks/<feature>/<feature>技术方案-评审版.md`
- 文件头（固定写法，放在标题下方）：

```markdown
> 本文件为《<feature>技术方案.md》的评审版衍生品，面向技术评审（产品 / 测试 / 前端 / 后端协作者）。
> 内容以主方案为准，请勿直接编辑本文件；主方案变更后本文件自动重生成。
```

## 前置条件

- 主方案已通过交付确认（plan.md 状态为「已完成」）。主方案不完整时**不生成**，提示用户先补全主方案（fail fast，不拿半成品转换）。

## 8 节裁剪表（按讲解顺序）

### 1. 需求
- **来源**：主方案 1 需求背景 + 2 需求说明，合并压缩
- **写**：一段话说清做什么、为什么（5 行以内）——评审要的是"为什么开会"
- **不写**：业务场景罗列、PRD 复述、范围细节

### 2. 整体设计思路
- **来源**：主方案 3.1 + 3.4
- **写**：架构图（直接复用主方案 3.1 的 mermaid）+ 设计主线 2-3 段；**状态机融入本节**：复用主方案 3.4 的 stateDiagram，每个流转一行触发条件
- **不写**：逐字段论证、模块职责长文、流转的副作用/守卫条件细节（主方案里有，评审不展开）

### 3. 数据模型
- **来源**：主方案 3.2（+ 涉及迁移时主方案第 5 节的 DML）
- **写**：DDL 原样保留（评审要拍板表结构），每个表一句话说明角色；**涉及存量数据迁移时附 DML/迁移语句**
- **不写**：索引逐个论证（索引定义留在 DDL 里即可）

### 4. 接口改动
- **来源**：主方案 3.3 + 第 4 节改动清单的「接口变更」行
- **写**：
  - **新增接口：完整 schema**（路径/方法/请求响应字段/错误码）——前端第一次见
  - **变更接口：只给 diff**——「变更前 → 变更后」对照表，老接口前端熟
  - 每个接口一行影响方（谁调用/谁对接）
- **不写**：幂等性等实现细节（除非该点需要评审拍板）
- diff 对照表写法：

| 字段 | 变更前 | 变更后 |
|---|---|---|
| status | int（0/1/2） | string（PENDING/USED/EXPIRED） |
| expireTime | （无） | 新增，datetime，ISO8601 |

### 5. 关键流程
- **来源**：主方案 3.5
- **写**：sequenceDiagram 原样复用 + 编号步骤文字（比主方案可再压缩）
- **不写**：算法伪代码级细节

### 6. 测试影响
- **来源**：主方案第 4 节改动点 + plan.md「影响范围」的上下游依赖
- **写**：① 本次变更要测什么（2-4 条核心场景）② **受影响的现有功能清单（回归范围）**——由「改了什么 × 谁在用」推导，如"改了 VoucherService 状态流转 → 现有券列表页状态展示、ExpireVoucherJob 需回归"
- **不写**：测试类名、数据准备方式（主方案第 6 节的事）
- **这是唯一允许增量推理（非摘录）的一节**：推导不出上下游时如实写"暂无已知影响面，欢迎评审补充"，禁止编造

### 7. 风险评估
- **来源**：主方案第 8 节
- **写**：只留「评审要拍板」的风险（数据迁移、兼容性、跨团队依赖），每条一行缓解措施
- **不写**：四类风险凑满、常规性能风险的模板化内容

### 8. 工时评估
- **来源**：主方案第 9 节
- **写**：表格原样照搬——工时是产品/管理直接消费的数字，不压缩

## 生成纪律（铁律）

1. **图直接复用**：架构图/时序图/状态图从主方案原样复制，不重画不简化（主方案已过 mermaid-checklist 自检）
2. **有则写无则跳过**：主方案没有状态机/存量迁移/接口变更，对应内容不硬凑空节
3. **只裁不造**：除第 6 节回归范围（明确允许推导）外，全部内容来自主方案摘录/压缩，禁止新增技术决策
4. **配方适配**：multi-task 的子任务依赖/上线顺序并入第 2 节或第 5 节；greenfield 以第 2 节为核心，无接口改动则跳过第 4 节
5. **禁止手改**：生成后用户要调措辞 → 改主方案再重生成评审版，不直接编辑评审版文件

## 生成后自检

| 检查项 | 标准 |
|---|---|
| 文件头标注 | 「以主方案为准」「请勿直接编辑」在 |
| 章节完整 | 8 节中有来源的都齐，无来源的没硬凑 |
| 接口写法 | 新增 = 完整 schema，变更 = diff 对照 |
| 图一致 | 与主方案一致，无重画走样 |
| 回归清单 | 第 6 节有受影响现有功能，或如实标「暂无已知」 |
````

- [ ] **Step 2: 结构走查**

Run: `grep -c '^###' /Users/zhuyuchen/ai/skill/dev-doc/references/review-doc.md && grep -n '评审版：' /Users/zhuyuchen/ai/skill/dev-doc/references/review-doc.md`
Expected: 8 节标题（`### 1. 需求` … `### 8. 工时评估`）；"评审版："字样出现在文件头模板附近

- [ ] **Step 3: Commit**

```bash
cd /Users/zhuyuchen/ai/skill/dev-doc
git add references/review-doc.md
git commit -m "feat(dev-doc): 新增评审版裁剪规则 references/review-doc.md"
```

---

### Task 2: SKILL.md 挂载点（6 处 edit）

**Files:**
- Modify: `SKILL.md`（行号基于当前 HEAD 39d7588，编辑时以锚文本为准）

**Interfaces:**
- Consumes: Task 1 的 `references/review-doc.md` 及其确立的命名/状态字面量
- Produces: SKILL.md 行为——Phase 5.3 评审版决策项、继续路径同步检测、plan.md 模板三态字段。Task 4 的 eval 断言依赖这些行为

- [ ] **Step 1: 核心机制列表加一条（约 17 行）**

old_string:
```
- **需求对齐节点**：PRD 质量低时自动生成业务流程图；待确认项先做事实/决策分诊（能自己查的不问用户），再标依赖分轮询问；交互类需求可征询用户后生成低保真原型辅助对齐（需求自带原型不可信——图文矛盾/无法读取——时同样触发）
```
new_string:
```
- **需求对齐节点**：PRD 质量低时自动生成业务流程图；待确认项先做事实/决策分诊（能自己查的不问用户），再标依赖分轮询问；交互类需求可征询用户后生成低保真原型辅助对齐（需求自带原型不可信——图文矛盾/无法读取——时同样触发）
- **评审版衍生**：主方案交付确认后生成面向技术评审的精简版（`<feature>技术方案-评审版.md`，8 节给人讲决策），主方案变更后自动整篇重生成；任务级可关闭（用户说"不要评审版"即关）
```

- [ ] **Step 2: 目录结构图加评审版文件（约 33-34 行）**

old_string:
```
            ├── prototype.html    # 低保真原型（仅交互类需求对齐时产出，一次性工具非交付物）
            └── <feature>技术方案.md  # 最终产物
```
new_string:
```
            ├── prototype.html    # 低保真原型（仅交互类需求对齐时产出，一次性工具非交付物）
            ├── <feature>技术方案.md  # 最终产物（面向编码 AI，含完整契约）
            └── <feature>技术方案-评审版.md  # 评审版（方案确认后生成，变更自动重生成；单向衍生不可手改）
```

- [ ] **Step 3: 前置步骤「继续」路径加同步检测（约 51 行）**

old_string:
```
       - 选「继续」→ 先读 `requirement.md` + `plan.md` 恢复上下文，按 plan.md 的"任务元信息-状态"和已落地的章节判断卡在哪个 Phase，从断点续做（不要从头重做）
```
new_string:
```
       - 选「继续」→ 先读 `requirement.md` + `plan.md` 恢复上下文，按 plan.md 的"任务元信息-状态"和已落地的章节判断卡在哪个 Phase，从断点续做（不要从头重做）
         - **评审版同步检测**：恢复上下文时检查任务目录是否存在 `*-技术方案-评审版.md`。存在且 plan.md 未记「评审版：关闭」→ 本次会话主方案任何变更落盘后，**立即按 [`references/review-doc.md`](references/review-doc.md) 整篇重生成**评审版并告知用户（变更已过确认，重生成是机械操作，不攒到 checkpoint）；plan.md 已记「评审版：关闭」→ 不生成；文件存在但 plan.md 无评审版记录（疑似用户手建）→ 询问用户采信现有文件还是按规则覆盖
```

- [ ] **Step 4: plan.md 模板任务元信息加评审版字段（约 283-286 行）**

old_string:
```
## 任务元信息
- 创建时间：YYYY-MM-DD
- 状态：进行中
- 配方：<generic-medium / multi-task / greenfield>
- 重量：<轻 / 中 / 重>
```
new_string:
```
## 任务元信息
- 创建时间：YYYY-MM-DD
- 状态：进行中
- 配方：<generic-medium / multi-task / greenfield>
- 重量：<轻 / 中 / 重>
- 评审版：<未生成 / 已生成 / 关闭>
```

- [ ] **Step 5: Phase 5.3 交付 checkpoint 加评审版决策项（约 458-463 行）**

old_string:
```
交付决策（逐项询问）：
- 通过 · 交付当前文档
- 局部修改 · 我会列出具体修哪里
- 先停一停 · 我要再看看

用户确认通过后，更新 `plan.md` 状态为"已完成"，任务结束。
```
new_string:
```
交付决策（逐项询问）：
- 通过 · 交付当前文档
- 局部修改 · 我会列出具体修哪里
- 先停一停 · 我要再看看

**评审版**（交付通过后独立询问；plan.md 已记「评审版：关闭」则跳过不问）：
- 生成（推荐）· 按 [`references/review-doc.md`](references/review-doc.md) 从主方案生成面向技术评审的精简版
- 不生成 · 在 plan.md 记「评审版：关闭」，本任务后续不再询问

用户也可在激活 skill 时或任意时点直接说"不要评审版"——等价于选"不生成"，立即记入 plan.md（**任务级**生效，换任务仍会询问）。

用户确认通过后：更新 `plan.md` 状态为"已完成"；若生成评审版，写 `<feature>技术方案-评审版.md`、plan.md 记「评审版：已生成」，并把评审版章节结构展示给用户过目（用户要调措辞 → 改主方案后重生成评审版，不直接编辑评审版）。任务结束。
```

- [ ] **Step 6: Reference 加载总表加一行（约 507 行，Phase 5 行之后）**

old_string:
```
| Phase 5 | 反 AI 味清单 + 终审视角 + mermaid 自检 | [`anti-ai-signs.md`](references/anti-ai-signs.md)、[`mermaid-checklist.md`](references/mermaid-checklist.md) |
```
new_string:
```
| Phase 5 | 反 AI 味清单 + 终审视角 + mermaid 自检 | [`anti-ai-signs.md`](references/anti-ai-signs.md)、[`mermaid-checklist.md`](references/mermaid-checklist.md) |
| Phase 5 交付后（评审版）| 评审版裁剪规则 | [`review-doc.md`](references/review-doc.md) |
```

- [ ] **Step 7: 走查**

Run: `grep -n 'review-doc\|评审版' /Users/zhuyuchen/ai/skill/dev-doc/SKILL.md | head -20`
Expected: 命中 ≥6 处——核心机制、目录结构图、继续路径同步检测、plan.md 模板、Phase 5.3 决策项、加载总表；且 `references/review-doc.md` 相对链接与 Task 1 文件路径一致

- [ ] **Step 8: Commit**

```bash
cd /Users/zhuyuchen/ai/skill/dev-doc
git add SKILL.md
git commit -m "feat(dev-doc): SKILL.md 挂载评审版生成与变更同步机制"
```

---

### Task 3: README 更新（2 处 edit）

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: Task 1/2 确立的行为与命名
- Produces: 无（纯文档）

- [ ] **Step 1: 核心特性列表加一条（约 20 行，需求对齐节点条目之后）**

old_string:
```
- **需求对齐节点**：PRD 质量低时自动生成业务流程图；待确认项先做事实/决策分诊（能自己查的不问用户），再标依赖分轮询问；交互类需求可征询用户后生成低保真原型辅助对齐（需求自带原型不可信——图文矛盾/无法读取——时同样触发）
```
new_string:
```
- **需求对齐节点**：PRD 质量低时自动生成业务流程图；待确认项先做事实/决策分诊（能自己查的不问用户），再标依赖分轮询问；交互类需求可征询用户后生成低保真原型辅助对齐（需求自带原型不可信——图文矛盾/无法读取——时同样触发）
- **评审版衍生**：方案确认后一键生成面向技术评审的精简版（8 节：需求 / 整体设计思路 / 数据模型 / 接口改动 / 关键流程 / 测试影响 / 风险 / 工时），主方案变更自动整篇重生成，任务级可关闭（"不要评审版"即关）
```

- [ ] **Step 2: 目录结构图同步（约 42-43 行）**

old_string:
```
            ├── prototype.html    # 低保真原型（仅交互类需求对齐时产出，一次性工具非交付物）
            └── <feature>技术方案.md  # 最终产物
```
new_string:
```
            ├── prototype.html    # 低保真原型（仅交互类需求对齐时产出，一次性工具非交付物）
            ├── <feature>技术方案.md  # 最终产物（面向编码 AI）
            └── <feature>技术方案-评审版.md  # 评审版（确认后生成，变更自动重生成，单向衍生不可手改）
```

- [ ] **Step 3: 走查 + Commit**

Run: `grep -c '评审版' /Users/zhuyuchen/ai/skill/dev-doc/README.md`
Expected: ≥2 处命中

```bash
cd /Users/zhuyuchen/ai/skill/dev-doc
git add README.md
git commit -m "docs(dev-doc): README 补评审版衍生能力说明"
```

---

### Task 4: evals 扩展（本地资产，不入库）

**Files:**
- Modify: `evals/evals.json`（已 gitignore，**不 commit**）
- Test fixture: `evals/fixtures/demo-voucher-service/`（复用，不改）

**Interfaces:**
- Consumes: Task 1/2 的行为（文件名、plan.md 状态字面量、8 节结构）
- Produces: eval1 新增 2 条断言 + eval4（9 条断言）；Task 5 按此跑

- [ ] **Step 1: eval1 prompt 加关闭指令**

`evals/evals.json` 中 id=1 的 `prompt` 字段，在结尾"最终一次性交付。）"后追加一句，改为：

```
...最终一次性交付。另外：本任务不要生成评审版。）
```

（即在右括号前插入"另外：本任务不要生成评审版。"）

- [ ] **Step 2: eval1 断言与预期输出扩展**

id=1 的 `assertions` 数组末尾追加两条：

```json
"任务目录下不存在 *-技术方案-评审版.md（关闭入参生效，未生成评审版）",
"plan.md 任务元信息记了「评审版：关闭」"
```

`expected_output` 末尾追加一句：`用户声明不要评审版 → 不生成评审版，plan.md 记「评审版：关闭」。`

- [ ] **Step 3: 新增 eval4（生成 + 同步）**

`evals` 数组末尾（id=3 之后）追加：

```json
{
  "id": 4,
  "name": "review-doc-generate-and-sync",
  "prompt": "给 demo-voucher-service 加一个券过期自动退款能力：用户持有的券到期未使用时，按用户购券时的实付金额原路退回（不是券面额），需要新增一张退款记录表和退款查询接口，退款有 INITIATED/SUCCESS/FAILED 三个状态。项目在 evals/fixtures/demo-voucher-service/。（测试环境说明：本任务无人交互，视作用户已主动明确授权——所有 checkpoint 按你的推荐推进，不必停下等确认；交付确认时请生成评审版。交付并生成评审版后，还有一个变更：退款失败的原因码从 string 改为 int 枚举（1001 余额不足 / 1002 渠道超时 / 1009 未知），请同步更新所有产物。最终一次性交付。）",
  "expected_output": "主方案过终审交付后生成评审版（8 节裁剪：需求一段话 / 整体设计思路含状态机 / DDL / 接口改动新增全 schema / 关键流程 / 测试影响含回归清单 / 简化风险 / 工时照搬），文件头标注以主方案为准。变更指令后主方案与评审版同步更新（原因码 string→int 枚举），评审版整篇重生成而非手动编辑，plan.md 记「评审版：已生成」。",
  "files": ["evals/fixtures/demo-voucher-service/"],
  "assertions": [
    "任务目录下存在 <feature>技术方案-评审版.md",
    "评审版文件头含「以主方案为准」和「请勿直接编辑」标注",
    "评审版含 8 节结构：需求 / 整体设计思路 / 数据模型 / 接口改动 / 关键流程 / 测试影响 / 风险评估 / 工时评估",
    "评审版的数据模型节含退款记录表 DDL",
    "评审版接口改动节：新增的退款查询接口有完整 schema（请求与响应字段）",
    "状态机内容出现在「整体设计思路」节内，评审版无独立状态机章节",
    "「测试影响」节含受影响的现有功能清单（回归范围，如券详情 / ExpireVoucherJob 相关）",
    "变更后评审版已同步：退款失败原因码在评审版中体现为 int 枚举（1001/1002/1009），不再是 string",
    "plan.md 任务元信息记「评审版：已生成」"
  ]
}
```

- [ ] **Step 4: JSON 合法性走查**

Run: `python3 -c "import json; d=json.load(open('/Users/zhuyuchen/ai/skill/dev-doc/evals/evals.json')); print(len(d['evals']), [e['name'] for e in d['evals']])"`
Expected: `4 ['light-weight-single-point', 'low-quality-requirement-alignment', 'multi-task-recipe-and-relations', 'review-doc-generate-and-sync']`

（evals/ 已 gitignore，本任务无 commit）

---

### Task 5: 跑 eval 验证（本地，不入库）

**Files:**
- Test 运行目录：`../dev-doc-workspace/iteration-2/`（兄弟目录，产物不 commit）

**Interfaces:**
- Consumes: Task 4 的 eval1/eval4 定义、Task 1-3 的 skill 文件
- Produces: eval1 + eval4 的 with_skill 运行结果（timing.json / grading.json / outputs/）

- [ ] **Step 1: 准备 fixture 副本**

```bash
cd /Users/zhuyuchen/ai/skill
for e in "eval-1-light-weight-single-point" "eval-4-review-doc-generate-and-sync"; do
  mkdir -p "dev-doc-workspace/iteration-2/$e/with_skill/run-1/project"
  cp -R dev-doc/evals/fixtures/demo-voucher-service/. "dev-doc-workspace/iteration-2/$e/with_skill/run-1/project/"
done
```

（eval1 关闭路径和 eval4 生成/同步路径是新行为的两条验证线；eval2/eval3 不受影响可留待全量回归。本次只跑 with_skill——裸跑对照对新断言无意义：裸 agent 不知道评审版概念，eval4 必挂、eval1 的关闭断言会虚高。）

- [ ] **Step 2: 并行派 2 个 general-purpose subagent 跑 eval**

每个 subagent 的 prompt 要点（按既有跑法）：
1. 第一步 `Read` `/Users/zhuyuchen/ai/skill/dev-doc/SKILL.md`（带 skill）
2. 任务正文 = 对应 eval 的 `prompt` 字段原文
3. 工作目录 = 该 eval 的 `run-1/project/`
4. 完成后在通知里报告 duration_ms

- [ ] **Step 3: 机检断言，写 grading.json**

对照 eval 的 assertions 逐条检查 `run-1/project/dev-doc/tasks/<feature>/` 下产物，写 `run-1/grading.json`（字段名固定：`summary.pass_rate/passed/failed/total` + `expectations[].text/passed/evidence`），产物拷贝进 `run-1/outputs/`。

- [ ] **Step 4: 判定与回归**

- eval4：9/9 通过 → 功能验证 OK
- eval1：原 7 条不回归 + 新 2 条通过 → 关闭路径 OK
- 有失败 → 定位（规则文件表述歧义 / SKILL.md 挂载点遗漏）→ 修 Task 1-3 对应文件 → 重跑该 eval。**测试资产与修复都要如实汇报，不粉饰**

- [ ] **Step 5: 汇总**

向用户汇报：通过率、耗时、发现的问题与修复。提醒 iteration-2 结果可与 iteration-1 基线（21/21, 411s）对照。全量 4 条 eval 回归可另行安排（或在真实项目试用时顺带验证）。
