# Skills

> 一套 Claude Code 代码质量工作流技能 — 从分析、编码、审查到提交，覆盖全流程。

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![Updated](https://img.shields.io/badge/updated-2026--07--02-blue)

## ✨ 项目亮点

- **全流程覆盖**：分析 → 编码 → 注释 → 审查 → 编码规整 → 提交 → 文档，一条线走完
- **中文优先**：面向中文开发者设计，技术名词保留英文，解释用自然中文
- **互相编排**：`code-writting` 作为编排器，自动串联分析、注释规范、编码检查
- **双 VCS 兼容**：提交信息生成同时支持 Git 和 SVN，适配企业环境
- **即装即用**：每个技能独立运行，也可组合使用，无需额外配置

## 📦 技能列表

### 编码工作流（核心链）

| 技能 | 用途 | 触发场景 |
|------|------|---------|
| `code-analysis` | 分析问题根因，给出 ≥2 种方案，不写代码 | 用户描述 Bug/问题现象，或意图模糊时默认路由 |
| `code-writting` | 编排器 — 串联分析、注释、编码，完成修改+验证 | 用户明确要求修复/实现/修改代码 |
| `code-comment` | 代码注释规范检查 | 写新函数/类、修改逻辑、代码审查、提交前检查 |
| `code-commit` | 标准化提交信息生成（Angular Conventional Commits） | 生成 Git/SVN 提交信息 |

### 质量保障

| 技能 | 用途 | 触发场景 |
|------|------|---------|
| `code-refactor` | 识别代码坏味道（长方法、重复代码、大类等），给出重构方案 | 重构代码、优化结构、消除技术债务 |
| `code-revirwer` | 中文代码审查，生成结构化审查报告 | 代码审查、code review、代码质量评估 |
| `file-encoding` | 批量统一编码为 UTF-8 with BOM + LF，支持 GBK/Latin-1 自动转换 | MSVC 编译乱码、跨平台换行符混乱 |

### 文档生成

| 技能 | 用途 | 触发场景 |
|------|------|---------|
| `readme-zh` | 中文 README 生成器，先分析项目再写文档 | 写 README、重写项目介绍、补全文档首页 |

## 🏗️ 工作流架构

```
code-analysis（只分析，不动代码）
       │
       ▼
code-writting（编排器：分析 → 修改 → 验证）
       │
       ├── 调用 code-analysis（或接受已有分析结果）
       ├── 调用 code-comment（每写一个文件检查注释）
       └── 调用 file-encoding（每改一个文件检查编码）
       │
       ▼
code-revirwer（代码审查，生成报告）  ←→  code-refactor（识别坏味道，给出重构方案）
       │
       ▼
code-commit（生成标准化提交信息）
```

**独立技能**：`readme-zh` 可随时调用，不依赖其他技能。

- **`code-analysis`** 和 **`code-writting`** 是分析/修复配对 — 先想清楚再动手
- **`code-comment`** 和 **`file-encoding`** 是辅助技能，由 `code-writting` 在修改过程中自动调用
- **`code-revirwer`** 审查改动质量，输出结构化报告
- **`code-refactor`** 识别代码坏味道并提供重构方案，可与 `code-revirwer` 配合使用
- **`code-commit`** 在所有修改完成后生成提交信息
- **`readme-zh`** 独立运行，生成项目文档

## 🚀 快速开始

### 安装

将本仓库克隆到 `.reasonix/skills/` 目录即可生效：

```bash
git clone https://github.com/PingWangWang/Skills.git ~/.reasonix/skills
```

Claude Code 启动时会自动加载该目录下的技能定义。

### 使用方式

在 Claude Code 会话中直接描述需求，匹配的技能会自动触发：

```
帮我分析一下这个崩溃的原因          → 触发 code-analysis
直接帮我修这个 Bug，不用确认        → 触发 code-writting
审查一下这段代码                    → 触发 code-revirwer
这段代码太长了，帮我重构一下        → 触发 code-refactor
生成一条提交信息                    → 触发 code-commit
这个文件编码有问题，帮我统一一下    → 触发 file-encoding
帮我写一份项目 README               → 触发 readme-zh
```

## 📁 项目结构

```
skills/
├── code-analysis/       # 问题分析（只出方案，不动代码）
│   └── SKILL.md
├── code-writting/       # 修复编排器（分析→修改→验证）
│   └── SKILL.md
├── code-comment/        # 代码注释规范
│   └── SKILL.md
├── code-revirwer/       # 中文代码审查
│   └── SKILL.md
├── code-refactor/       # 重构顾问（坏味道检测）
│   └── SKILL.md
├── code-commit/         # 标准化提交信息生成
│   └── SKILL.md
├── file-encoding/       # 文件编码统一（UTF-8 BOM + LF）
│   └── SKILL.md
├── readme-zh/           # 中文 README 生成器
│   └── SKILL.md
├── .reasonix/           # Claude Code 工作区元数据
├── .gitignore
├── LICENSE
└── README.md
```

## ❓ 适用场景

- **个人开发者**：统一编码习惯，减少随手写模糊提交、缺注释、编码混乱
- **AI 辅助开发团队**：让 AI 生成的代码符合团队的注释、编码和提交规范
- **C++/跨平台项目**：解决 MSVC 编码乱码、换行符不一致的工程问题
- **企业 SVN + Git 混用环境**：跨版本控制系统统一提交格式
- **开源项目维护者**：用 `code-revirwer` 审查 PR，用 `code-refactor` 治理代码坏味道，用 `readme-zh` 生成专业文档

## 🛠️ 自定义

每个技能都是独立的 `SKILL.md` 文件，可直接修改：

1. 编辑对应目录下的 `SKILL.md`
2. 重启 Claude Code 即可生效

技能文件格式遵循 Claude Code Skill 规范，包含 `description`（触发条件）、执行步骤、失败处理、输出格式和禁止行为。

## 🤝 贡献

欢迎 PR / Issue。提交前请确保：

- 新增技能目录名与 `name` 字段一致
- `description` 明确注明触发条件和不适用场景
- 包含完整的执行步骤、失败处理和禁止行为

## 📝 License

[MIT](./LICENSE) © 2026 PingWang
