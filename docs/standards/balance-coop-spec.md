# Claude + Codex 协同维护规范（平衡版）

## 设计目标

本规范适用于：

* 单人维护仓库
* Claude 与 Codex 协同工作
* 规则库维护
* 配置同步
* 批量修复
* 文档治理

目标：

* 保持流程简单
* 避免 Git 冲突
* 保证任务可追溯
* 避免过度工程化

---

# Agent 分工

## Claude（Review Agent）

职责：

* 制定规范
* 拆解任务
* 编写任务文档
* 审查执行结果
* 输出 Review 结论
* 必要时进行小规模修复

允许修改：

```text
docs/
rules/
```

允许场景：

* 规范维护
* Review 修复
* 示例实现
* Hotfix

默认不承担：

```text
大规模批量修改
重复性执行任务
```

---

## Codex（Execution Agent）

职责：

* 根据任务文档执行修改
* 批量修改文件
* 自动化修复
* 编写测试
* 实施规范要求

允许修改：

```text
src/
tests/
config/
rules/
```

限制：

* 不修改规范文档
* 不修改 Review 结论
* 不擅自扩大需求范围
* 不改变架构设计

---

# 分支模型

长期保留：

```text
main
claude/review
codex/automate
```

说明：

```text
main            生产基线
claude/review   Claude 工作分支
codex/automate  Codex 工作分支
```

不要求为每个任务创建新分支。

---

# 同步规则

唯一可信基线：

```text
main
```

同步方式：

```text
main → claude/review
main → codex/automate

claude/review → main
codex/automate → main
```

允许：

```bash
git rebase main
git merge main
```

禁止：

```text
claude/review → codex/automate
codex/automate → claude/review
```

两个 Agent 不直接同步。

所有同步必须经过：

```text
main
```

---

# 文档结构

```text
docs/
├─ standards/
├─ tasks/
├─ reviews/
├─ questions/
└─ archive/
```

---

# 任务流程

## 阶段一：Claude 创建任务

创建：

```text
docs/tasks/Pxxx.md
```

必须包含：

```text
任务目标
修改范围
禁止修改项
验收标准
测试要求
风险说明
```

提交示例：

```text
[P003] add task specification
```

---

## 阶段二：Codex 执行

Codex 从最新 main 同步：

```bash
git checkout codex/automate
git fetch
git rebase main
```

读取：

```text
docs/tasks/Pxxx.md
```

实施修改。

提交示例：

```text
[P003] implement rule updates
```

完成后：

```text
codex/automate → main
```

---

## 阶段三：Claude Review

Claude 同步最新 main：

```bash
git checkout claude/review
git fetch
git rebase main
```

创建：

```text
docs/reviews/Pxxx-review.md
```

Review 结果必须为：

```text
PASS
```

或：

```text
FAIL
```

---

## PASS 规则

PASS 必须满足：

```text
需求完成
测试通过
无额外副作用
修改范围符合要求
```

Review 示例：

```text
Result: PASS
```

完成后归档：

```text
docs/archive/Pxxx.md
docs/archive/Pxxx-review.md
```

---

## FAIL 规则

FAIL 必须包含：

```text
发现的问题
修复建议
影响范围
```

示例：

```text
Result: FAIL

Issue 1:
缺少路径测试

Issue 2:
配置未同步
```

FAIL 后：

```text
Codex 继续修复
```

无需创建新工作分支。

---

# 歧义处理

Codex 遇到以下情况必须暂停：

```text
规范不完整
需求冲突
实现路径不明确
可能影响架构
需要额外决策
```

创建：

```text
docs/questions/Qxxx.md
```

示例：

```text
Q005.md
```

内容：

```text
问题描述
涉及文件
可能方案
需要 Claude 决策项
```

禁止自行猜测需求。

---

# 提交规范

任务：

```text
[P003] implement rule updates
```

修复：

```text
[P003] fix review issues
```

Review：

```text
[P003] review pass
[P003] review fail
```

规范：

```text
[P003] add task specification
```

问题：

```text
[Q005] request clarification
```

---

# Review 优先原则

Claude 的 Review 结论优先级最高。

当出现：

```text
任务文档
Codex 实现
Review 结论
```

冲突时：

```text
Review > Task > Implementation
```

执行 Agent 必须遵循最新 Review。

---

# 核心原则

1. Claude 负责规范和质量控制。
2. Codex 负责执行和自动化修改。
3. 所有同步通过 main 完成。
4. Agent 之间禁止直接同步分支。
5. Review 必须给出 PASS 或 FAIL。
6. Codex 遇到歧义必须暂停。
7. 文档必须可追溯。
8. 保持流程简单，避免不必要的临时分支。
