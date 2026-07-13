---
name: build-verify-dt
description: 编译验证 GcThermalDesktop — 执行 UpdateSDK → BuildDebug → BuildRelease，并修复编译错误
---

# build-verify-dt — GcThermalDesktop 编译验证

## 流程

### 1. 执行 UpdateSDK
```bash
cd "d:/Code/ThermalDesktop/trunk/Projects/D16_ThermalDesktop/GcThermalDesktop"
powershell -NoProfile -ExecutionPolicy Bypass -File ./UpdateSDK.ps1
```
使用 `run_in_background: true` 异步执行。

### 2. 执行 BuildDebug
```bash
cd "d:/Code/ThermalDesktop/trunk/Projects/D16_ThermalDesktop/GcThermalDesktop"
powershell -NoProfile -ExecutionPolicy Bypass -File ./BuildDebug.ps1
```
使用 `run_in_background: true` 异步执行。

### 3. 执行 BuildRelease
```bash
cd "d:/Code/ThermalDesktop/trunk/Projects/D16_ThermalDesktop/GcThermalDesktop"
powershell -NoProfile -ExecutionPolicy Bypass -File ./BuildRelease.ps1
```
使用 `run_in_background: true` 异步执行。

### 4. 修复编译错误

#### 错误收集
- 从每个构建步骤的输出中提取编译错误
- 错误来源：MSBuild 输出（`error CSxxxx`、`error LNKxxxx` 等）
- 使用 `Grep` / `Glob` 定位错误源文件

#### 修复策略
- 对于每次修复，创建不超过 4 个待办项
- 每项修复完成后重新编译验证
- TypeScript/C# 类型错误：检查类型定义、导入路径、null 安全
- 链接错误：检查库引用、依赖路径、平台目标
- NuGet/包引用错误：检查 SDK 版本兼容性、包源
- 迭代直到编译通过或达到 token 预算上限

### 5. 成功标准
- BuildDebug.ps1 以 exit code 0 完成
- BuildRelease.ps1 以 exit code 0 完成
- 输出目录存在编译产物
