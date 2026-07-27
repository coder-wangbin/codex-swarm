# Swarm

> CodeX 并行智能体集群 Skill —— 第一性原理 × 对抗式审查 × 防作弊护栏，三位一体的并行执行框架。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.txt)
[![CodeX Skill](https://img.shields.io/badge/CodeX-Skill-339cff)](https://github.com/openai/codex)

[English](README_en.md)

---

## 这是什么

`swarm` 是一个 CodeX Skill，让 AI 编程助手在面对复杂任务时自动启用并行分解策略。

它的核心思想源于三大支柱，并融合了 [leader](https://github.com/KKKKhazix/khazix-skills/tree/main/leader) skill 的结构化任务书方法论：

| 支柱 | 管什么 | 核心逻辑 |
|------|--------|---------|
| **第一性原理** | 生成 | 打断类比推理，回到根本事实重新推导——治本不治表 |
| **对抗式审查** | 验证 | 多 Agent 并发攻击，在上线前把系统搞崩——找到所有隐藏漏洞 |
| **防作弊护栏** | 质量 | 明确禁止 skip 测试、放宽断言、mock 核心、`\|\| true`——堵死 AI 偷懒的后门 |

三者构成**完整闭环**：根因分析 → 结构化任务书 → 并行实现（带防作弊）→ 对抗式审查 → 反向验证 → 修复根因 → 再验证。

## 触发条件（自动，无需手动调用）

Skill 在以下条件中 ≥2 条满足时**自动激活**：

- 任务涉及 ≥3 个独立文件或 ≥2 个独立模块
- 同时需要代码探索和代码实现
- 存在 ≥2 个无数据依赖的独立子任务
- 需要多维度验证（构建 + 测试 + Lint）
- 你说了「并行」「同时」「分头」「concurrent」
- 你说了「从第一性原理出发」「first principles」「根本原因」
- 你说了「对抗式审查」「adversarial」「攻击测试」
- 你说了「安全审查」「越权」「权限漏洞」「security audit」
- 你说了「代码审查」「code review」「审计」「审查一下」

**单独触发**：说「审查」「review」「有什么问题」「越权」「安全审查」会直接进入对抗式审查/安全审计模式。

## 它能做什么

### 1. 智能任务分解（含任务类型分流）

接到任务后先分类——能写验收命令的是执行型（硬指标），要答案本身的是探索型（证据目标）。再从第一性原理分析根因，输出基线验证和「替用户拍的板」清单，最后按因果关系分解。

### 2. 结构化 Agent 任务书

每个 spawn 的 Agent 收到的不再是模糊指令，而是包含边界/验收/防作弊/禁止顺手活/反向验证/进度标记的完整结构化任务书。

### 3. 防作弊护栏

每个 Worker Agent 明确禁止：skip 测试、放宽断言、mock 核心逻辑、删除测试、`|| true` 掩错、修改测试基线。违反即任务失败，零容忍。

### 4. 并行 Agent 编排

最多 12 个 Agent 并发执行（Worker ≤9 个，Explorer+Attacker ≤6 个），超限自动排队。写集合隔离保证安全。

### 5. 对抗式审查 + 反向验证

7 大攻击向量维度（含 Go 后端 Auth 专项），修复后必须执行反向验证——故意触发一次 bug 证明检查真的会报警（红→绿证据链）。

### 6. 安全审计（Auth/Permission 专项）

Go 后端专属：middleware 链审计、角色提升检测、路由级访问控制、JWT/Cookie 安全、RBAC 一致性、DB 级权限。

### 7. 全生命周期自动管理

Agent 创建→监控→收集→回收全自动。活性检测防卡死，完成即回收防泄漏。PROGRESS.md 断点续跑。

### 8. 定期审计

每 2-4 周一次全局对抗式审计——架构、依赖、代码质量、文档一致性。

## 安装

```bash
git clone https://github.com/coder-wangbin/codex-swarm.git ~/.codex/skills/swarm
# 重启 CodeX 生效
```

## 使用示例

### 示例 1：多模块功能开发（带防作弊）

```
你说：「给 skb 的知识库模块和权限模块各加一个操作审计日志」

模型自动：
1. Phase 0.5 基线验证 → 跑 go test 记录当前测试数/覆盖率
2. 第一性原理分析 → 审计日志的本质是「谁在什么时间对什么做了什么操作」
3. 分解：Worker-1 知识库审计，Worker-2 权限审计
4. 每个 Worker 收到结构化任务书：边界/验收/防作弊/反向验证
5. 并行执行 + 对抗式审查 + 反向验证（红→绿证据）
6. 集成验证，确认测试数未减少、覆盖率未下降
```

### 示例 2：Bug 根因修复

```
你说：「OpenAI 抓取器坏了，修一下」

模型行为：
❌ 无第一性原理：直接修抓取器 → 治标不治本
✅ 有第一性原理：发现底层流量路由机制有设计缺陷 → 重构路由层 → 一劳永逸
✅ 防作弊护栏：修复后不能 skip 任何现有测试
✅ 反向验证：故意触发路由异常 → 贴红输出 → 修复 → 贴绿输出
```

### 示例 3：安全审计

```
你说：「安全审查一下 dinghe 的 auth 模块有没有越权漏洞」

模型自动：
1. Phase 0.5 基线验证
2. Map 攻击面：middleware 链 → 路由组 → 角色常量 → JWT 配置
3. 并行 spawn 4 个 Attacker：middleware 链、路由 ACL、角色提升、JWT 安全
4. 收集所有发现，按 CRITICAL/HIGH/MEDIUM/LOW 排序
5. 呈现审查报告，等你确认后修复
6. 修复后反向验证每条 CRITICAL/HIGH
```

## 目录结构

```
swarm/
├── SKILL.md                          # Skill 主文件（三支柱 + 执行协议）
├── README.md                         # 本文档（中文）
├── README_en.md                      # English README
├── LICENSE.txt                       # MIT
├── agents/
│   └── openai.yaml                   # UI 元数据
├── assets/
│   └── icon.svg                      # Skill 图标
└── references/                       # 深入参考文档
    ├── lifecycle.md                  # Agent 生命周期状态机 + Pool 内部结构
    ├── patterns.md                   # 11 种分解模式 + 反模式
    ├── adversarial-review.md         # 7 类攻击向量分类学 + 多 Agent 攻击编排
    ├── auth-review.md                # Go 后端 Auth 安全审计（middleware/JWT/RBAC/SSO）
    └── task-brief.md                 # 结构化 Agent 任务书模板 + 防作弊 clause 目录
```

## 设计理念

这个 Skill 的设计本身也遵循第一性原理。

Skill 的本质是什么？不是给模型加功能，而是改变模型的**思考方式**。

大多数 Skill 告诉模型「做什么」——这个 Skill 告诉模型「怎么想」：
1. 先回到根本事实重新推导（第一性原理）
2. 再站在对面找漏洞（对抗式审查）
3. 最后用防作弊护栏堵死所有偷懒的后门

这四个思维习惯一旦内化，代码质量有质的飞跃——不限于任何具体领域。

融合了 [KKKKhazix/leader](https://github.com/KKKKhazix/khazix-skills/tree/main/leader) skill 的结构化任务书方法论。

这篇文章详细解释了前两个 Prompt 的实际效果：[Vibe Coding 两大基石](https://mp.weixin.qq.com/s/umPqTD_-IubbhXIgiS47eQ)

## License

MIT
