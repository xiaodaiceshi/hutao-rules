# PROJECT_LESSONS.md

## P001: hutao-rules 经验教训

---

### 两次审查迭代

**日期：** 2026-06-12
**项目编号：** P001
**项目名称：** hutao-rules
**类型：** 优点
**关键词：** 审查迭代, 设计级, 质控级

#### 发生了什么

Codex Phase 1→4 执行完成后，Claude 进行了一轮质控审查，发现了 21 个旧文件未删除、BOM 头、deploy.yml 格式错误等 5 个问题。修复后又发现了 Phase 1 遗漏的 3 个文件（steam-domain、talkatone-domain、iptv-domain）和 banAd 命名不匹配。

#### 影响或价值

证明了两次审查迭代（设计级 + 质控级）的必要性——第一次看大方向，第二次逐文件验证细节，缺一不可。

#### 正确做法 / 后续复用方式

所有 Codex 批量执行后，Claude 应至少做两轮审查：先看结构合理性，再逐文件验证内容一致性。

---

### docs/inbox/ 一次性指令传递

**日期：** 2026-06-12
**项目编号：** P001
**项目名称：** hutao-rules
**类型：** 规则
**关键词：** 文档传递, 修复指令, inbox

#### 发生了什么

发现推送远端再 fetch 的方式对一次性修复指令过于笨重。改为 Claude 直接写入 Codex 侧本地 `docs/inbox/` 目录。`.gitignore` 已排除该目录，不会误提交。

#### 影响或价值

省去了 git add → commit → push → fetch 的流程，一次性指令直接从 Claude 到 Codex，用完即弃。

#### 正确做法 / 后续复用方式

规范文档走 git（需版本历史），修复指令走 inbox（一次性的）。

---
