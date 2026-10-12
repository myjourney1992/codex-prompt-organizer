---
title: Codex Engineering Prompt Playbook
aliases:
  - Codex 工程 Prompt 手册
  - Codex Prompt 模板库
tags:
  - AI
  - Codex
  - Agentic-Engineering
  - Prompt
  - Software-Engineering
version: v1.0
created: 2026-09-30
updated: 2026-09-30
status: active
---

# Codex Engineering Prompt Playbook

> 目标：把 Codex 从“代码生成器”升级成可委派工程任务的 Software Engineering Agent。
>
> 核心原则：**中文描述意图 + 英文保留工程实体 + 明确 Goal / Context / Constraints / Done Criteria / Verification。**

---

## 0. 快速索引

| 场景 | Codex 角色 | 第一动作 | 核心产物 |
|---|---|---|---|
| 陌生代码理解 | Investigator | Trace | 系统地图 / 调用链 |
| Bug 调试 | Debugger | Reproduce / Evidence | Root Cause + Fix |
| Refactor / Migration | Migration Engineer | Inventory | 分阶段迁移方案 |
| 测试设计 | Test Engineer | Risk Analysis | 高价值 Regression Tests |
| Code Review | Reviewer | Diff + Callers | 高价值 Findings |
| 架构研究 | Staff Engineer | Understand Constraints | Options + Trade-offs |
| 新功能开发 | Feature Engineer | Understand Existing Pattern | 最小一致实现 |
| 性能排查 | Performance Engineer | Measure First | Bottleneck + Evidence |
| 最终验收 | Release Reviewer | Verify | Delivery Report |

---

# 1. Codex 的正确心智模型

不要把 Codex 当成：

> 会写代码的 ChatGPT。

更准确的模型是：

```text
Goal
 ↓
Understand
 ↓
Investigate
 ↓
Plan
 ↓
Implement
 ↓
Verify
 ↓
Review
 ↓
Done
```

一个高质量 Codex 任务通常包含：

```text
Goal
Context
Scope
Constraints
Done Criteria
Verification
```

## 1.1 中文还是英文？

默认使用：

```text
中文：需求、业务规则、限制、验收标准、分析要求
英文：类名、函数名、文件路径、API、字段、命令、日志、技术专有名词
```

示例：

```text
检查 reconnect 以后 websocket 是否重复注册 listener。

重点查看：
- MessageRepository
- SocketManager
- reconnect()
- StreamSubscription

不要先加 dedup workaround，先确认 root cause。
```

不需要为了 Codex 强行把整段任务翻译成英文。

---

# 2. 通用任务模板

适用于绝大多数中大型任务。

```text
# 目标

我要解决：

【任务目标】

# 背景

当前情况：

【业务背景 / Bug 表现 / 当前实现】

相关模块：

【模块、文件、类；不知道可以不填】

# 任务范围

重点关注：

【需要调查或修改的范围】

不要处理：

【明确排除范围】

# 约束

- 尽量保持现有 public API 不变
- 不做与本任务无关的重构
- 优先复用现有抽象
- 不新增依赖，除非确有必要
- 不确定时先调查现有实现，不要猜测

# 工作方式

先理解现有实现。

如果是复杂问题：
先调查 → 找证据 → 给出判断 → 再实施。

不要为了完成任务而掩盖症状。

# 完成标准

只有满足以下条件才认为完成：

【Done Criteria】

# 验证

完成修改后：

- 运行相关测试
- 运行静态检查
- 检查最终 git diff
- 检查是否产生 regression
- 删除无关修改

最后告诉我：

1. 你发现了什么
2. root cause / 核心设计是什么
3. 修改了什么
4. 为什么这样修改
5. 做了哪些验证
6. 是否还有风险
```

---

# 3. 模板 01：陌生代码库 / 模块理解

## 适用场景

- 新接手项目
- 接手别人模块
- 阅读老代码
- 理解开源项目
- 理解 IM / UGC / 支付 / 登录等完整链路
- 想知道“这段业务到底怎么跑起来的”

## Prompt

```text
# 任务：理解代码路径

我需要理解：

【例如：聊天消息发送流程】

不要修改代码。

请从真实代码出发，完整 Trace：

【例如】

UI
→ State Management
→ Service / Domain
→ Repository
→ SDK / API
→ WebSocket
→ Cache
→ UI Update

对于每一层说明：

- 对应文件
- 核心类 / 方法
- 这一层负责什么
- 输入是什么
- 输出是什么
- 状态在哪里维护
- async boundary 在哪里
- error 在哪里处理

最后给我：

1. 完整调用链
2. 核心数据流
3. 关键状态变化
4. 最值得先看的 5 个文件
5. 模块之间的 ownership
6. 可疑 coupling
7. 潜在风险

不要只根据文件名总结，必须实际阅读相关实现。
```

## 推荐连续追问

### 第二轮：反向路径

```text
现在继续 Trace 对应的反向流程：

【例如：消息接收流程】

仍然不要修改代码。

重点说明：
- event 从哪里进入
- identity 在哪里建立
- state 在哪里更新
- UI 如何得到通知
```

### 第三轮：交叉比较

```text
现在比较发送和接收两条路径。

重点找：
- 重复责任
- ownership 不清晰的位置
- identity 建立位置
- state merge 位置
- 可能产生 race condition 的 async boundary
```

---

# 4. 模板 02：Bug Debugging

## 核心原则

**Evidence First，不要 Patch First。**

错误模式：

```text
症状
→ 猜
→ 改
```

推荐模式：

```text
症状
→ Trace
→ Hypothesis
→ Evidence
→ Reproduce
→ Root Cause
→ Fix
→ Regression Test
```

## Prompt

```text
# 任务：Bug 调查

问题：

【描述现象】

期望行为：

【正确行为】

已知触发条件：

【如果知道就写，不知道留空】

先不要修改代码。

第一阶段只调查。

请：

1. 找出所有可能导致这个现象的执行路径
2. Trace 相关数据流和状态变化
3. 找出 candidate root causes
4. 为每个假设寻找代码证据
5. 排除没有证据的猜测
6. 尝试设计最小复现方式

先告诉我：

- 最可能的 root cause
- 证据是什么
- 如何验证
- 为什么其他可能性较低

确认根因后，再实施最小修复。

修复要求：

- 不通过隐藏症状解决问题
- 不使用大范围 defensive workaround，除非确有必要
- 尽量不改变无关行为
- 增加 regression test

完成后运行：

【例如 flutter test】
【例如 flutter analyze】

最后再 review 一次自己的修改，特别检查：

- regression
- lifecycle
- concurrency / race condition
- error handling
- edge cases
```

## IM 重复消息示例

```text
调查聊天消息偶发重复的问题。

现象：
发送一条消息后，本地 optimistic message 已显示，
websocket 返回 server message 后偶尔出现两条。

期望：
server acknowledgement 完成后列表中只能保留一条逻辑消息。

先不要修改代码。

重点检查：
- 所有可以 insert message 的入口
- clientId / messageId identity semantics
- optimistic message
- websocket acknowledgement
- local cache
- history merge
- reconnect
- pagination

先找到 root cause，并提供代码证据。

不要直接增加一个全局 Set 做 dedup，除非根因证明这确实是正确方案。

确认根因后：
1. 实施最小修复
2. 增加 regression test
3. 运行相关测试
4. 运行 flutter analyze
5. 检查 reconnect / pagination / ordering 是否 regression
```

---

# 5. 模板 03：Refactor / Migration

## 核心原则

```text
Inventory
→ Risk Classification
→ Pilot
→ Verify
→ Extract Pattern
→ Scale
→ Cleanup
```

不要：

```text
设计新架构
+
一次迁移全项目
+
一起修所有问题
```

## Prompt

```text
# 任务：重构 / 迁移

目标：

【例如：将 Provider 逐步迁移到 Riverpod】

不要立即进行全量修改。

第一阶段先做 Inventory。

请找出：

- 当前实现涉及哪些模块
- shared state
- feature-local state
- dependency injection
- side effects
- lifecycle-sensitive state
- tests
- 与其他模块的 coupling

按照迁移风险分为：

低风险
中风险
高风险

然后提出迁移顺序。

优先选择一个低风险但具有代表性的 Vertical Slice 作为 Pilot。

先只迁移 Pilot。

要求：

- behavior unchanged
- 尽可能保持 public API
- 测试保持通过
- 不同时进行无关架构重构

Pilot 完成后：

分析这次迁移中形成的可复用 Pattern。

只有 Pattern 得到验证后，再继续迁移其他模块。
```

## Pilot 完成后的追问

```text
Review 这次 Pilot migration。

请总结：

1. 哪些步骤是可复用 Pattern
2. 哪些部分是 feature-specific
3. 哪些地方存在 migration risk
4. 测试覆盖是否足够
5. 下一批最适合迁移哪些模块

不要继续修改代码。
```

## 批量迁移阶段

```text
按照已经验证过的 Pilot Pattern，
迁移：

【模块列表】

要求：
- 不重新设计 migration pattern
- 每完成一个 logical group 后运行测试
- 遇到结构差异时停下来分析原因，不要强行套模板
- 最终检查是否遗留旧 Provider / adapter / dead code
```

---

# 6. 模板 04：测试设计

## 核心原则

测试的目标不是 coverage 数字，而是：

> 如果这里发生 regression，什么行为真的会伤害用户？

## Prompt

```text
# 任务：测试分析

先不要写测试。

请作为 Test Engineer 分析：

【目标模块】

目标不是提高 coverage 数字。

目标是找到：

“如果这里发生 regression，最可能真正影响用户的行为。”

重点分析：

- state transition
- async behavior
- error recovery
- lifecycle
- retry
- concurrency
- cache
- idempotency
- boundary conditions
- network failure
- duplicate events
- out-of-order events

按照：

高风险
中风险
低风险

进行分类。

告诉我为什么这些场景值得测试。

然后只为最高价值的场景增加测试。

优先测试 observable behavior。

避免测试：

- private implementation detail
- trivial getter/setter
- 无意义 mock interaction
```

## 测试完成后的自查

```text
Review 你刚刚增加的测试。

告诉我：

1. 每个测试具体能抓住什么 regression
2. 哪些重要行为仍然没有覆盖
3. 哪些测试过度依赖 implementation detail
4. 哪些 mock 让测试产生 false confidence
5. 有没有 flaky risk

如果存在低价值测试，删除或改写。
```

---

# 7. 模板 05：Code Review

## 核心原则

不要只审 diff 的表面。

应该结合：

```text
diff
+
callers
+
affected state
+
tests
+
related modules
```

## Prompt

```text
# 任务：Code Review

Review 当前 diff。

不要只看 diff。

同时检查：

- callers
- related state
- affected tests
- related modules

优先寻找真正会产生影响的问题。

严重级别：

P0
数据损坏、安全问题、严重 correctness 问题

P1
Crash、race condition、核心行为错误

P2
高概率 regression、lifecycle、performance 问题

P3
有明确后果的 maintainability 问题

不要报告：

- 纯格式问题
- 无关命名偏好
- 没有现实 execution path 的理论问题

对于每一个 finding 给出：

位置：
问题：
触发场景：
实际影响：
为什么现有测试可能发现不了：
最小修复：

如果没有重要问题，直接告诉我没有发现高价值 finding。

不要为了显得有输出而制造问题。
```

## Flutter Review 追加项

```text
Flutter 额外关注：

- BuildContext lifecycle
- mounted
- async gap
- StreamSubscription dispose
- Riverpod lifecycle
- duplicate listeners
- rebuild scope
- state mutation
- navigation lifecycle
- isolate / async ordering
```

## iOS Review 追加项

```text
iOS / Swift 额外关注：

- retain cycle
- reference ownership
- MainActor
- actor isolation
- async cancellation
- Task lifecycle
- thread safety
- UI lifecycle
- NotificationCenter observer lifecycle
- delegate ownership
```

---

# 8. 模板 06：架构研究

## 核心原则

不要先问：

> 什么架构最好？

而是先问：

> 当前架构到底有什么真实问题？

## Prompt

```text
# 任务：架构分析

我正在考虑：

【架构想法】

先不要实现。

首先调查当前代码。

请画出当前：

- responsibilities
- module boundaries
- data flow
- state ownership
- dependencies
- side effects

然后分析当前架构真正存在的问题。

不要因为我提出了某个方案，就默认这个方案是正确的。

给出三个方向：

A：最小演进方案
B：更彻底的架构调整
C：保持当前设计

分别分析：

- 能解决什么问题
- 不能解决什么
- migration cost
- complexity
- affected modules
- risks
- long-term maintenance

然后再站在一个 skeptical Staff Engineer 的角度攻击这些方案：

- 是否 over-engineering
- 是否引入不必要 abstraction
- 是否增加认知成本
- 是否真的解决当前痛点
- 当前架构有没有其实更好的地方

最终寻找：

“用最小改动获得大部分收益”的方案。

不要为了追求架构形式漂亮而增加抽象。
```

## IM 三层架构专用示例

```text
我正在考虑把 IM Web / Flutter 实现拆成三层：

1. SDK Driver
2. Data Normalization
3. Business Orchestration

先不要实现。

先调查当前 IM 代码中：

- SDK integration
- models
- normalization
- cache
- conversation state
- message state
- business rules
- UI state
- error handling
- reconnect

分别由谁负责。

然后评估三层设计是否真的解决现有问题。

给出：

A：最小演进
B：三层完整拆分
C：保持当前结构

重点分析：
- ownership 是否更清晰
- 是否减少 SDK coupling
- 是否方便未来更换 SDK
- 是否增加 abstraction overhead
- Flutter / Web 是否能共享概念模型
- 迁移成本
- 测试复杂度

最后寻找最小可验证架构实验，不要直接全量重构。
```

---

# 9. 模板 07：新功能开发

## Prompt

```text
# 任务：实现新功能

目标：

【功能目标】

用户行为：

【用户如何触发、看到什么结果】

先不要立刻写代码。

先调查仓库中最相似的现有功能，找出：

- UI pattern
- state-management pattern
- data layer pattern
- error handling
- analytics / logging
- tests

告诉我你准备复用哪些现有 Pattern。

然后实施最小一致方案。

约束：

- 不创建第二套架构
- 优先复用现有 component / service / abstraction
- 不修改与本功能无关的代码
- 新 abstraction 必须有明确复用价值

Done Criteria：

【验收条件】

完成后：

1. 跑相关测试
2. 跑 static analysis
3. 检查 UI / state / error path
4. review git diff
5. 告诉我剩余风险
```

---

# 10. 模板 08：性能排查

## 核心原则

**Measure First。**

不要：

```text
感觉这里慢
→ 加缓存
```

应该：

```text
现象
→ Measure
→ Locate Bottleneck
→ Hypothesis
→ Experiment
→ Compare
```

## Prompt

```text
# 任务：性能调查

现象：

【例如：聊天列表首次打开明显卡顿】

不要先优化代码。

先定位性能瓶颈。

请调查：

- critical path
- repeated work
- unnecessary rebuild / render
- I/O
- serialization
- cache
- synchronous blocking
- large allocations
- repeated network calls
- expensive loops
- unnecessary database queries

如果项目已有 profiling / benchmark 工具，优先使用。

先给我：

1. 当前性能路径
2. candidate bottlenecks
3. 每个判断的证据
4. 如何量化
5. 最小实验方案

只有得到测量证据后再修改。

修改后必须比较：

Before
vs
After

并说明：
- 提升来自哪里
- 是否用 memory 换 performance
- 是否增加 complexity
- 是否存在 regression
```

---

# 11. 最终验收 Prompt

建议所有重要任务最后执行一次。

```text
现在不要继续增加功能。

对当前最终结果进行一次交付前检查。

检查：

1. 原始需求是否完全满足
2. 是否有遗漏的 edge case
3. 是否改变了任务范围之外的行为
4. 是否存在不必要的复杂度
5. 是否存在重复代码
6. error handling 是否正确
7. lifecycle / concurrency 是否安全
8. tests 是否真正覆盖 regression
9. 是否有无关修改
10. 是否有可以删除的临时代码

运行相关：

- tests
- static analysis
- build / typecheck（如果适用）

然后检查最终 git diff。

如果发现问题，直接修复并重新验证。

最后给我一个简洁交付报告：

Root cause / Goal：
Implementation：
Files changed：
Verification：
Remaining risks：
```

---

# 12. Flutter 专用附加指令

可以附加到 Debug / Review / Feature / Refactor Prompt 后。

```text
这是 Flutter 项目。

额外关注：

- Widget rebuild scope
- BuildContext lifecycle
- mounted
- async gap
- Stream / StreamSubscription
- Timer lifecycle
- AnimationController lifecycle
- Riverpod Provider lifecycle
- autoDispose
- ref.watch / ref.read 使用
- duplicate listeners
- state mutation
- isolate
- navigation lifecycle
- pagination
- optimistic update
- network reconnect

验证优先使用项目现有命令。

常见命令：

flutter pub get
flutter analyze
flutter test
dart run build_runner build --delete-conflicting-outputs

不要修改 generated files。
```

---

# 13. iOS / Swift 专用附加指令

```text
这是 iOS / Swift 项目。

额外关注：

- ARC / retain cycle
- weak / unowned ownership
- MainActor
- actor isolation
- thread safety
- Task cancellation
- structured concurrency
- async / await lifecycle
- NotificationCenter observer
- Combine subscription
- delegate ownership
- UIViewController lifecycle
- SwiftUI state ownership
- @State / @StateObject / @ObservedObject
- memory pressure
- background / foreground lifecycle

优先复用项目现有 concurrency 和 architecture pattern。
```

---

# 14. Web 专用附加指令

```text
这是 Web 项目。

额外关注：

- component lifecycle
- state ownership
- stale closure
- async race
- request cancellation
- duplicate request
- cache invalidation
- hydration
- SSR / CSR boundary
- event listener cleanup
- memory leak
- bundle impact
- rendering frequency
- API error path
- browser compatibility

不要为了局部功能引入新的全局状态方案。
```

---

# 15. AGENTS.md 推荐模板

原则：**短、稳定、项目级。**

不要把所有知识都塞进 AGENTS.md。

```markdown
# Project

【项目一句话介绍】

# Architecture

【核心目录 / 依赖方向】

例如：

Feature-first architecture:

lib/features/<feature>/
- data
- domain
- presentation

UI must not access API clients directly.

# Commands

Install:
【命令】

Analyze:
【命令】

Test:
【命令】

Build:
【命令】

# Rules

- Preserve existing public APIs unless the task requires changes.
- Do not edit generated files.
- Prefer existing abstractions before creating new ones.
- Keep business logic outside UI components.
- Do not introduce dependencies unless necessary.
- Avoid unrelated refactors.

# Verification

For implementation tasks:

- run affected tests
- run static analysis
- inspect final git diff
- fix regressions caused by the change

# Communication

与我沟通时默认使用中文。

代码、变量名、类名、API、命令、日志和技术专有名词保持原文。
不要为了中文表达而翻译代码实体。
```

---

# 16. Goal / Completion Condition 模板

适合：

- flaky test
- 偶现 Bug
- 性能问题
- 大型迁移
- 复杂 Debug
- 结果需要多轮实验才能判断的任务

```text
Goal：

【最终目标】

Completion Condition：

只有满足以下条件才停止：

- 【条件 1】
- 【条件 2】
- 【条件 3】
- tests pass
- static analysis pass
- final diff has no unrelated changes

中间过程可以自行决定。

不要因为第一次实现看起来可行就停止。
```

## Bug 示例

```text
Goal：

复现并彻底解决聊天消息偶发重复问题。

Completion Condition：

- 存在可重复或明确可解释的触发路径
- root cause 有代码证据
- regression test 能覆盖该问题
- fix 后 regression test 通过
- 原有 message tests 通过
- reconnect / pagination 行为没有 regression
```

---

# 17. 调查模式 Prompt

当你不知道问题在哪里时，优先使用。

```text
先不要修改代码。

把自己当作 investigator。

目标不是快速给方案，而是理解事实。

请：

1. Trace 真实执行路径
2. 找到 state ownership
3. 找到 side effects
4. 找 async boundary
5. 找 error path
6. 找测试覆盖
7. 用代码证据验证你的判断

明确区分：

事实：
推测：
尚未确认：

不要把推测写成事实。
```

---

# 18. 自我反驳 Prompt

适合架构、重构、复杂修复。

```text
现在不要继续实施。

站在一个 skeptical Staff Engineer 的角度，
尝试推翻你刚才提出的方案。

重点寻找：

- hidden assumptions
- over-engineering
- unnecessary abstraction
- regression risk
- migration cost
- operational complexity
- test blind spot
- simpler alternative

如果原方案仍然成立，解释原因。

如果存在更小、更安全的方案，提出替代方案。
```

---

# 19. Diff 清理 Prompt

```text
检查当前 git diff。

目标：

只保留完成本任务真正需要的修改。

请找出：

- unrelated changes
- debug code
- temporary logging
- duplicate abstraction
- unnecessary rename
- formatting noise
- dead code
- commented-out code
- accidental generated-file changes

能安全删除的直接删除。

然后重新运行相关验证。
```

---

# 20. 高效工作流：复杂任务三段式

## 第一段：Understand

```text
先调查，不要修改代码。

Trace 相关实现，并说明：

- current behavior
- ownership
- data flow
- state transition
- async boundary
- likely risks

所有重要判断都要有代码证据。
```

## 第二段：Implement

```text
基于前面的调查结果，
实施最小正确修改。

保持任务范围之外行为不变。

优先复用现有 Pattern。
```

## 第三段：Verify

```text
验证当前实现。

运行：
- affected tests
- static analysis
- 适用的 build / typecheck

然后 review final diff：

- regression
- unnecessary complexity
- unrelated changes
- lifecycle
- concurrency
- error handling

发现问题直接修复并重新验证。
```

---

# 21. 什么任务不需要复杂模板？

以下任务可以直接做：

```text
修改文案
简单 UI 调整
明确的小型 API 替换
单文件低风险修改
格式化
简单脚本
```

可以直接：

```text
实现这个修改：

【需求】

要求：
- 保持现有 Pattern
- 不改无关代码
- 完成后运行相关检查
```

不要所有任务都套十段模板。

---

# 22. Codex 委派决策表

接到任务后先问自己：

## Q1：我知道问题在哪里吗？

不知道：

```text
Investigation / Trace
```

知道：

```text
进入 Q2
```

## Q2：这是 Bug 还是 Feature？

Bug：

```text
Evidence-first Debug
```

Feature：

```text
Find Existing Pattern → Implement
```

## Q3：修改范围大吗？

大：

```text
Inventory → Pilot → Scale
```

小：

```text
Implement → Verify
```

## Q4：风险高吗？

高：

```text
更多 Evidence
Regression Test
Self Review
```

低：

```text
快速实现 + Verification
```

---

# 23. 我与 Codex 的职责边界

## 我负责

```text
Product Judgment
Architecture Decision
Risk Acceptance
Scope
Trade-off
Done Criteria
Final Review
```

## Codex 负责

```text
Repo Exploration
Trace
Investigation
Implementation
Migration
Tests
Static Analysis
Diff Review
Documentation
Repetitive Engineering Work
```

目标不是：

> 让 Codex 替我思考。

而是：

> 让我把注意力集中到高价值判断上。

---

# 24. 最重要的 10 条原则

1. **先理解，再修改。**
2. **复杂 Bug 先找证据，不要直接 Patch。**
3. **要求 Root Cause，不只要求症状消失。**
4. **重构先 Pilot，再 Scale。**
5. **测试关注 Regression Risk，不追 Coverage 数字。**
6. **Review 要看 execution path，不要堆风格意见。**
7. **架构先证明问题，再设计 abstraction。**
8. **告诉 Codex 什么叫 Done，而不是规定每一步怎么走。**
9. **所有重要修改必须进入真实 Verification Loop。**
10. **最终一定检查 git diff。**

---

# 25. 日常最小口诀

开始任务时问：

> **这是什么类型的工程问题？**

过程中问：

> **它的判断有代码证据吗？**

结束时问：

> **它真的验证了吗？**

---

# 26. 后续版本计划

## v1.1

- Flutter IM Debug 专用模板
- Flutter Riverpod Migration 模板
- UGC / Draft / Publish 状态流模板
- iOS Crash / Memory / Concurrency 模板
- Web IM 架构模板

## v1.2

- PR Review Workflow
- GitHub Issue → Codex Task 模板
- Codex + Claude Code 分工
- Codex + Cursor 分工
- Worktree / Branch 并行开发工作流

## v2

- Agentic Engineering 完整工作流
- Spec → Plan → Build → Verify → Review → Ship
- 可复用 Skills
- 项目级 Context Engineering
- Multi-agent Workflow

---

# Personal Notes

## 我当前最值得练习的能力

- [ ] 不直接让 Codex 写，先学会让它调查
- [ ] 给 Bug 定义可验证的 Root Cause
- [ ] 给任务定义 Done Criteria
- [ ] 学会用 Vertical Slice 做 Pilot
- [ ] Review Codex 的判断，而不是逐行替它写代码
- [ ] 让测试成为 Codex 的反馈系统
- [ ] 建立自己的高价值 Prompt / Pattern 库

## 我的实践记录

### 任务

【】

### 使用模板

【】

### Codex 表现

【】

### 哪个 Prompt 最有效

【】

### 出现的问题

【】

### 下次如何改进

【】
