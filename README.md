# Swarm

> CodeX 并行智能体集群 Skill —— 基于第一性原理与对抗式审查的双基石并行执行框架。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.txt)
[![CodeX Skill](https://img.shields.io/badge/CodeX-Skill-339cff)](https://github.com/openai/codex)

[English](README_en.md)

---

## 这是什么

`swarm` 是一个 CodeX Skill，让 AI 编程助手在面对复杂任务时自动启用并行分解策略。

它的核心思想源于两个被社区称为「神级 Prompt」的技巧：

| 基石 | 管什么 | 核心逻辑 |
|------|--------|---------|
| **第一性原理** | 生成 | 打断类比推理，回到根本事实重新推导——治本不治表 |
| **对抗式审查** | 验证 | 多 Agent 并发攻击，在你上线之前把系统搞崩——找到所有隐藏的漏洞 |

两者构成**完整闭环**：第一性原理 → 根因分解 → 并行实现 → 对抗式审查 → 修复根因 → 再验证。

## 触发条件（自动，无需手动调用）

Skill 在以下条件中 ≥2 条满足时**自动激活**：

- 任务涉及 ≥3 个独立文件或 ≥2 个独立模块
- 同时需要代码探索和代码实现
- 存在 ≥2 个无数据依赖的独立子任务
- 需要多维度验证（构建 + 测试 + Lint）
- 你说了「并行」「同时」「分头」「concurrent」
- 你说了「从第一性原理出发」「first principles」「根本原因」
- 你说了「对抗式审查」「adversarial」「攻击测试」

**单独触发**：说「审查」「review」「有什么问题」会直接进入对抗式审查模式。

## 它能做什么

### 1. 智能任务分解

接到复杂任务后，先从第一性原理分析根因，再按因果关系分解，而非按文件结构机械拆分。

### 2. 并行 Agent 编排

最多 12 个 Agent 并发执行（Worker ≤9 个，Explorer ≤6 个），超限自动排队。写集合隔离保证安全。

### 3. 对抗式审查

6 大攻击向量维度（时间、数据、并发、资源、状态、安全），40+ 具体触发条件，多 Agent 并发攻击你的代码，在用户发现问题之前把漏洞全找出来。

### 4. 全生命周期自动管理

Agent 从创建到回收的全生命周期自动管理——活性检测防卡死、完成即回收防泄漏。

### 5. 定期审计

每 2-4 周一次全局对抗式审计——审查架构、依赖、代码质量、文档一致性。

## 安装

```bash
# 直接 clone
git clone https://github.com/coder-wangbin/codex-swarm.git ~/.codex/skills/swarm

# 重启 CodeX 生效
```

也可以使用 CodeX 自带的 skill-installer 安装（如果可用）：

```bash
# 通过 skill-installer
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo coder-wangbin/codex-swarm \
  --path /
```

## 使用示例

### 示例 1：多模块功能开发

```
你说：「给 skb 的知识库模块和权限模块各加一个操作审计日志」

模型自动：
1. 第一性原理分析 → 审计日志的本质是「谁在什么时间对什么做了什么操作」
2. 分解：Worker-1 知识库审计，Worker-2 权限审计
3. 两个 Worker 并行执行
4. 完成后自动进入对抗式审查——攻击 Agent 尝试绕过审计、伪造时间戳、触发并发写入
5. 修复发现的问题
6. 集成验证，所有 Agent 自动回收
```

### 示例 2：Bug 根因修复

```
你说：「OpenAI 抓取器坏了，修一下」

模型行为：
❌ 无第一性原理：直接修抓取器 → 治标不治本
✅ 有第一性原理：发现底层流量路由机制有设计缺陷 → 重构路由层 → 一劳永逸
✅ 修完后对抗式审查 → 确认其他信源不会再有同类问题
```

### 示例 3：对抗式审查

```
你说：「审查一下 pkg/logic/qa.go 有什么问题」

模型自动：
1. 生成攻击向量：时间异常、空数据、并发写入、缓存不一致、SQL 注入……
2. 并行 spawn 3-6 个攻击 Agent，按维度分组
3. 收集所有发现，按严重程度排序
4. 呈现审查报告，等你确认后修复
```

## 目录结构

```
swarm/
├── SKILL.md                          # Skill 主文件（触发规则 + 执行协议）
├── README.md                         # 本文档（中文）
├── README_en.md                      # English README
├── LICENSE.txt                       # MIT
├── agents/
│   └── openai.yaml                   # UI 元数据
├── assets/
│   └── icon.svg                      # Skill 图标
└── references/                       # 深入参考文档
    ├── lifecycle.md                  # Agent 生命周期状态机 + Pool 内部结构
    ├── patterns.md                   # 9 种分解模式 + 反模式（含对抗式审查）
    └── adversarial-review.md         # 攻击向量分类学 + 多 Agent 攻击编排
```

## 设计理念

这个 Skill 的设计本身也遵循第一性原理。

Skill 的本质是什么？不是给模型加功能，而是改变模型的**思考方式**。

大多数 Skill 告诉模型「做什么」——这个 Skill 告诉模型「怎么想」：先回到根本事实重新推导（第一性原理），再站在对面找漏洞（对抗式审查）。这两个思维习惯一旦内化，代码质量有质的飞跃——不限于任何具体领域。

这篇文章详细解释了这两个 Prompt 的实际效果：[Vibe Coding 两大基石](https://mp.weixin.qq.com/s/umPqTD_-IubbhXIgiS47eQ)

## License

MIT
