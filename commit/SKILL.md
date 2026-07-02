---
name: commit
description: Use when user requests commit message generation, formatted commit log, or standardized Git/SVN commit text. Requires VCS diff/staged changes. Not for: executing commits, non-repo directories, general commit standard discussion, SVN commit execution.
---

# 生成标准化 Commit 提交信息

## 概述

遵循 Angular Conventional Commits 规范，适配 C++ 热仿真项目（ThermalDesktop、SVN 自动化提交），兼容 Git 标准。解决 AI 自动提交乱码、描述模糊、难以追溯问题。

## 核心价值

1. **快速阅读日志**：一眼区分 bugfix / 新功能 / 文档 / 重构
2. **代码评审高效**：不用打开文件就能知道修改内容
3. **问题追溯**：通过日志快速定位引入 bug 的提交
4. **自动化工具兼容**：AI 自动提交、CHANGELOG 可自动生成
5. **跨团队协作**：统一阅读习惯，杜绝模糊提交

## 执行步骤

### 步骤 1：检测版本控制系统

```bash
git rev-parse --git-dir 2>/dev/null
```

- 成功 → Git 仓库，走步骤 2
- 失败 → 检测 `svn info`，成功则走步骤 3
- 都不是 → 输出错误，停止

### 步骤 2：获取 Git 变更

```bash
git status                    # 变更文件列表
git diff --staged             # 已暂存变更（优先）
git diff                      # 补充未暂存变更
git log --oneline -5          # 参考最近 5 条提交风格
```

### 步骤 3：获取 SVN 变更

```bash
svn status                    # 变更文件列表
svn diff                      # 分析变更内容
svn log --limit 5             # 参考最近 5 条提交风格
```

### 步骤 4：确定提交类型（只选一个）

| 类型 | 何时选 | 适用项目场景 |
|------|--------|-------------|
| `fix` | 修复缺陷、BUG | ThermalDataReader 报错、CSV 解析异常、界面刷新失效 |
| `feat` | 新增功能 | CSV 过滤功能、Heatmap 新渲染逻辑、新增接口 |
| `docs` | 仅注释/文档/说明 | 统一目录代码注释、接口文档补充 |
| `refactor` | 重构代码，无功能变化 | 优化布局、代码整理、抽离工具类 |
| `style` | 格式调整，无逻辑修改 | 空格、换行、命名统一 |
| `perf` | 性能优化 | 渲染提速、读取大文件优化 |
| `chore` | 工程构建、SDK、脚本 | 更新 SDK、打包脚本、删除冗余文件 |

若无法确定 → 默认 `chore`。
若变更涉及多种类型 → 拆分为多个 commit，分别生成。
若有破坏性变更 → 类型后加 `!`（如 `feat!:`）。

### 步骤 5：写 Subject（标题，最重要）

**格式：** `迭代号: 类型(作用域): 中文简短描述`

SVN 项目带迭代号前缀（如 `D16:`），与现有日志风格统一。Git 项目可省略。

**约束：**
- 中文撰写（类型保持英文小写，作用域保持英文大写）
- ≤50 字符
- 无句号
- 动词开头：修复 / 新增 / 优化 / 统一 / 更新
- 不写模糊词

**作用域（精确模块）：**

| 作用域 | 对应模块 |
|--------|---------|
| `READER` | ThermalDataReader 读取模块 |
| `HEATMAP` | 云图绘图模块 |
| `COMMON_UTIL` | 公共工具类 |
| `CSV` | CSV 导入导出 |
| `UI` | 界面布局 |
| `SDK` | 底层 GCtSDK |
| `REACTOR` | 反应堆求解模块 |

**作用域自动识别：** 根据 `git diff` / `svn diff` 的变更文件路径自动推断 scope，不询问用户。路径含 `ThermalDataReader` / `Reader` → `READER`，含 `CSV` → `CSV`，以此类推。无法推断时省略作用域。

**示例：**
```
D16: fix(READER): 修复ThermalDataReader文档激活后刷新失效
D16: feat(CSV): CSV导入新增列过滤，自动排除非节点温度列
D16: docs(COMMON_UTIL): 统一全部19个工具文件中文注释
```

### 步骤 6：写 Body（详细描述，可选）

**满足以下任一条件 → 必须写 Body：**
- 变更原因不显而易见
- 改了多文件、底层逻辑变更、有兼容性影响
- 有破坏性变更

**格式：**
- 空一行与 Subject 分隔
- 中文撰写
- 说明改动原因（背景）、做了什么、边界情况
- 按条例序号编写：1. 2. 3. 逐条列出改动逻辑

**示例正文：**
```
1. 文档激活事件触发时，未调用DataReader单例刷新接口，导致工况切换后数据不更新
2. 修改RcDocReactor文档激活回调，增加GetInstance().Refresh()调用
3. 同步更新头文件与SDK接口声明，保证跨模块调用一致
```

极简变更（typo 修复、一行改动）→ 省略 Body。

### 步骤 7：写 Footer（可选）

空一行与 Body 分隔。关联工单或声明破坏性变更时写。

**格式：**
```
Bug-ID: D16-3792
Task: D16热数据读取优化
测试点：多工况切换、重复打开CSV、空文件导入
修改文件：Reactor文档、DataReader读写层、底层SDK定义
```

### 步骤 8：输出日志文本

只输出日志文本本身。禁止输出任何其他内容。

## 完整示例

### BUG 修复
```
D16: fix(READER): 修复ThermalDataReader文档激活后刷新失效

文档激活documentActivated回调缺少数据刷新逻辑，切换工况后云图数据不会更新。
1. 在RcDocReactor.cpp激活函数调用ThermalDataReader单例Refresh接口
2. 同步修改对应.h头文件、GCtSDK接口定义
3. AppConfig同步新增读取配置项适配刷新逻辑

修改文件：Reactor文档、DataReader读写层、底层SDK定义
Bug-ID: D16-3792
```

### 新增功能
```
D16: feat(CSV): CSV导入新增列过滤功能，自动过滤非节点温度列

导入大尺寸CSV文件时会读取无关冗余列，拖慢加载速度。
新增过滤规则：仅保留节点编号+温度字段，其余列自动跳过；
兼容旧版CSV文件，无过滤列时保持原有读取逻辑。
```

### 文档注释
```
D16: docs(COMMON_UTIL): 统一Common/Util目录全部19个文件中文注释

补齐缺失函数说明，修正翻译混乱注释，统一接口注释格式；
无逻辑代码变更，仅调整注释文本。
```

### SDK 工程更新
```
D16: chore(SDK): 更新GCtSDK底层依赖包

替换旧版SDK动态库与头文件，修复底层内存读取溢出问题；
同步grx工程配置，删除旧版冗余压缩包。
```

## 针对 SVN + AI 自动提交的规范

### 中文编码
- 自动化脚本强制 UTF-8 输出提交信息
- 文案全程中文简洁，不夹杂生僻特殊符号
- SVN 客户端开启日志 UTF-8 解析

### AI 自动提交强制模板
AI 生成提交信息时强制以下格式，禁止只输出「更新文件」「修复 bug」：
```
迭代号: 类型(作用域): 简短修改说明

1. 简述改动逻辑
2. 标注改动文件范围
```

## 禁止的糟糕提交写法

| 反面示例 | 问题 |
|---------|------|
| 更新SDK | 无意义模糊描述 |
| 修改文件 | 无意义模糊描述 |
| 修复问题 | 无分类，无法区分改动类型 |
| CSV增加过滤功能 | 缺少 feat/csv 标识 |
| 回退错误提交 | 无细节，无法追溯 |
| 超长无分段 | 没有换行，可读性差 |
| 中英文夹杂转义字符 | 乱码问题 |
| 一次性提交几十份无关文件 | 不区分多个改动 |

## 失败处理

| 失败场景 | 处理 |
|---------|------|
| 不在版本控制目录 | 输出：`错误：当前目录不是 Git 或 SVN 仓库` |
| 无变更 | 输出：`无待提交的变更` |
| 无法确定类型 | 分别生成多个 commit message，标注建议拆分为 N 个提交 |
| git/svn 命令不可用 | 输出：`错误：未找到 git/svn 命令` |
| diff 内容无法解析 | 跳过该文件，基于文件名和路径推断变更意图 |
| Subject 超过 50 字符 | 多余内容移入 Body，Subject 保留核心动词+对象 |

## 输出格式

**成功时：** 只输出日志文本，不带任何包装。

**失败时：** 输出一行错误原因。

**禁止输出：**
- × `git commit`、`svn commit` 等提交命令
- × 「以下是提交日志」等开场白
- × 「建议执行...」等后续建议
- × 任何形式的提交操作执行

## 禁止行为

- × 执行任何提交操作：`git commit`、`svn commit`、`svn ci`，或任何 VCS 的提交/推送命令
- × 写入任何 VCS 的提交模板文件：`.git/COMMIT_EDITMSG`、`svn-commit.tmp` 等
- × 输出英文 Subject 或 Body（类型和作用域除外）
- × 输出开场白 / 结束语 / 命令建议
- × 在非仓库目录下猜测变更
- × 修改任何 VCS 配置文件（`.git/config`、SVN properties 等）
- × 使用无意义模糊词（更新、修改、修复问题）
- × Subject 不含类型标识或模块范围
