---
name: skill-compliance-checker
description: 当用户要求检查一个SKILL.md是否符合《Skill不完全指南》和《Skill进阶指南》的标准时触发。读取指定skill的SKILL.md，逐项对照指南要求输出合规性报告。
version: 1.0.0
author: Hermes Agent
created: 2026-07-10
updated: 2026-07-10
license: MIT
platforms: [windows, linux, macos]
allowed-tools: [read_file, terminal, execute_code, write_file]
trigger: ["检查skill", "skill合规", "检查SKILL", "合规性报告", "check skill", "skill compliance"]
tags: [skill, compliance, review, quality, checker]
max-tokens: 8000
metadata:
  hermes:
    tags: [skill, compliance, review, quality, checker]
    related_skills: [hermes-agent-skill-authoring]
---

# Skill 合规性检查器

## 适用场景

用户输入一个 Skill 名称（本地或 Hermes 已安装的 skill），本 Skill 读取其 `SKILL.md`，对照两份指南逐项检查，输出完整的合规性报告。

**注意：** 本 Skill 只能检查 `skill_view()` 能读取到的 skill（即已安装的 skill 或仓库内的 skill）。如果要检查一个文件路径下的 SKILL.md，请先通过 `read_file` 读取内容再执行。

## 参考指南

本检查器参考以下两份指南中的标准：

1. **《Skill不完全指南》** — 涵盖元数据字段、正文结构、渐进式披露、设计原则、常见错误和安全要点
2. **《Skill进阶指南》** — 涵盖编写范式（流水线型/检查清单型/多方案选择型/API集成型）、进阶技法（多Agent协作/思维蒸馏/模式组合/管道）

---

## 执行流程

### 阶段一：加载技能

1. 用用户提供的 skill 名称调用 `skill_view(name='<skill-name>')` 读取该 skill 的 SKILL.md 内容
2. 如果 skill_view 返回失败（skill 不存在），提示用户可用 skill 列表
3. 提取以下信息：
   - `frontmatter` — YAML 元数据（从 SKILL.md 第一组 `---` 中提取）
   - `body` — SKILL.md 正文（元数据之后的内容）
   - `total_chars` — 文件总字符数
   - `linked_files` — skill 关联的脚本、引用、资产文件列表

**检查点：** 确认 SKILL.md 内容成功加载，记录文件大小和结构概览。

### 阶段二：逐项检查

对 SKILL.md 逐项检查以下维度的合规性。每个维度输出 **通过/⚠️警告/❌失败**。

#### 检查维度 A — 元数据（来自《Skill不完全指南》）

| # | 检查项 | 标准 | 检查方法 |
|---|--------|------|---------|
| A1 | name 存在 | 必填，英文小写+连字符，不含空格 | 检查 frontmatter 的 `name` 字段 |
| A2 | description 存在 | 必填，50-150字场景描述而非功能名 | 检查 `description` 字段长度和内容 |
| A3 | created/updated | 推荐有创建和更新日期 | 检查 `created` 和 `updated` 字段 |
| A4 | version | 推荐有语义化版本号 | 检查 `version` 字段 |
| A5 | author | 推荐有作者信息 | 检查 `author` 字段 |
| A6 | tags | 推荐有标签数组 | 检查 `tags` 字段 |
| A7 | allowed-tools | 推荐声明允许的工具列表 | 检查 `allowed-tools` 字段 |
| A8 | trigger | 推荐有显式触发词 | 检查 `trigger` 字段 |
| A9 | max-tokens | 推荐限制 token 消耗 | 检查 `max-tokens` 字段 |
| A10 | description 质量 | 写场景不写功能名，50-150字 | 检查 description 是否包含场景触发词 |

#### 检查维度 B — 正文结构（来自《Skill不完全指南》）

| # | 检查项 | 标准 | 检查方法 |
|---|--------|------|---------|
| B1 | 用祈使句 | 用"读取用户提供的 Excel 文件"，不用"你应该读取" | 检查是否出现"你应该""建议你"等 |
| B2 | 步骤化 | 拆成第一步、第二步、第三步 | 检查是否有编号列表或 `## 阶段` 分段 |
| B3 | 有判断条件 | "如果完成率低于 60%，标记为风险项" | 检查是否有"如果""当""若"等条件词 |
| B4 | 有边界声明 | 声明适用范围和限制 | 检查是否有"不超过""仅限""边界条件"等 |
| B5 | 长度合理 | 正文 500-2000 字，不超过 3000 字 | 检查去掉 frontmatter 后的 body 长度 |
| B6 | 输入输出明确 | 字段名称、类型和含义都写清楚 | 检查是否包含输入/输出描述 |
| B7 | description 不重复正文 | 元数据的 description 不与正文大量重复 | 检查 description 是否只是正文的摘要 |

#### 检查维度 C — 渐进式披露（来自《Skill不完全指南》）

| # | 检查项 | 标准 | 检查方法 |
|---|--------|------|---------|
| C1 | 引用 references/ | 如果有 references/，需在正文中显式引用 | 检查引用路径是否在正文中出现 |
| C2 | 引用 scripts/ | 如果有 scripts/，需在正文中说明如何调用 | 检查脚本调用说明 |
| C3 | 引用 assets/ | 如果有 assets/，需在正文中引用 | 检查资产文件引用 |
| C4 | 路径用正斜杠 | 用 `scripts/extract.py` 而非 `scripts\extract.py` | 检查是否出现反斜杠路径 |

#### 检查维度 D — 设计原则与常见错误（来自《Skill不完全指南》）

| # | 检查项 | 标准 | 检查方法 |
|---|--------|------|---------|
| D1 | 先确认再动手 | 关键决策点停下来等确认 | 检查是否有"确认""询问用户"等词 |
| D2 | 边做边存 | 每阶段产出保存到文件 | 检查是否有保存/写入文件的步骤 |
| D3 | 一个 Skill 只做一件事 | 职责清晰，不臃肿 | 检查 body 是否在一个主题内 |
| D4 | 给选择不给答案 | 生成方案让用户选 | 检查是否有多方案输出 |
| D5 | 无时效性信息 | 不写"今年第二季度"等会过期内容 | 检查是否包含具体年份/季度 |
| D6 | no allowed-tools 绕过确认 | 允许的工具权限是否合理 | 检查 allowed-tools 声明是否合理 |

#### 检查维度 E — 安全要点（来自《Skill不完全指南》）

| # | 检查项 | 标准 | 检查方法 |
|---|--------|------|---------|
| E1 | 不看来源 | 是否有开发者/来源信息 | 检查 author 字段 |
| E2 | 敏感信息 | 是否有密码/API密钥写在 SKILL.md 中 | 检查是否出现 key/secret/password 等 |
| E3 | 权限声明合理 | allowed-tools 不过多 | 检查工具列表是否合理 |

#### 检查维度 F — 编写范式（来自《Skill进阶指南》）

| # | 检查项 | 标准 | 检查方法 |
|---|--------|------|---------|
| F1 | 范式识别 | 识别此 Skill 属于哪种/哪些范式 | 分析 body 结构判断范式类型 |
| F2 | 流水线型 — 检查点 | 如果使用流水线型，每阶段有检查点 | 检查是否有"检查点"/"确认"阶段边界 |
| F3 | 检查清单型 — 条目可执行 | 如果使用检查清单型，条目具体到可执行 | 检查检查项是否具体 |
| F4 | 多方案型 — 至少3个方案 | 如果使用多方案型，至少生成3个方案 | 检查方案数量 |
| F5 | API集成型 — 异常处理 | 如果使用API集成型，有异常处理逻辑 | 检查是否有错误处理描述 |
| F6 | 范式组合合理 | 如果组合多个范式，阶段划分清晰 | 检查阶段间是否通过文件/数据传递 |

---

### 阶段三：生成合规性报告

生成完整的 Markdown 合规性报告，包含：

1. **技能概览** — 名称、总字符数、文件结构
2. **合规总分** — 通过的检查项 / 总检查项（只统计适用的项）
3. **各维度明细** — 每个维度的检查结果列表
4. **问题汇总** — 所有失败和警告项汇总
5. **改进建议** — 按优先级排序的改进建议
6. **范式分析** — 识别到的编写范式组合分析

报告保存到 `D:/hermes/skill-compliance-report-<skill-name>.md`

**检查点：** 输出报告路径，列出关键发现摘要。

## 边界条件

- 如果 skill 不存在于本地，列出 `skills_list()` 的结果提示用户可用的技能
- 如果 SKILL.md 格式异常（如无法解析 frontmatter），输出"格式异常"并展示原始内容前 500 字符
- 如果 skill 文件超过 50KB，提示文件较大可能包含大量引用，检查重点放在元数据和正文结构
- 如果 skill 没有 `allowed-tools` 字段，标注为"未声明"而非"失败"
- 范式识别为 AI 推测，在报告中标注"推测"而非定论

## 错误处理

| 场景 | 处理方式 |
|------|---------|
| skill 不存在 | 提示用户可用 skills_list() 查看已安装技能 |
| SKILL.md 无 frontmatter | 报告"格式严重异常"，无法完成大部分元数据检查 |
| 文件读取失败 | 提示文件路径或权限问题 |
| 正文过长 >100KB | 仅检查前 50KB + 最后 10KB，标注"部分检查" |

## 输出示例

```markdown
# Skill 合规性报告: sales-data-analysis

## 概览
- 名称: sales-data-analysis
- 总字符数: 5,107
- 文件结构: SKILL.md + references/ (2) + scripts/ (1)

## 合规总分: 22/26 (84.6%)

## 各维度明细

### A. 元数据 (8/10 ✅)
- [✅] A1: name 存在
- [⚠️] A10: description 可优化 — 建议加入更多触发场景词

### B. 正文结构 (5/6 ✅)
...

### C. 渐进式披露 (3/4 ⚠️)
...

## 范式分析
- 识别范式: 流水线型
- 检查点: ✅ 有阶段边界检查点
- 阶段数量: 3

## 改进建议 (按优先级)
1. [高] 添加 created/updated 日期字段
2. [中] 优化 description 提高触发准确率
...
```

## 参考资料

- 指南文本文件：`D:/hermes/skill-guide-1-text.txt`（《Skill不完全指南》）
- 指南文本文件：`D:/hermes/skill-guide-2-text.txt`（《Skill进阶指南》）
