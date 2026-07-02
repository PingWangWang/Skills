---
name: build-verify-entity
description: GcTDEntity 编译验证 — 先 Debug 后 Release，全部 NoWait 异步执行，修复编译错误
---

# Build Verify Entity — GcTDEntity 编译验证

## 触发条件

修改 GcTdEntity 项目的 `.h` / `.cpp` / `.ps1` / `.sln` / `.vcxproj` / `.props` 文件，或增删文件导致项目结构变化后，执行本 skill。

## 执行步骤

### 第 1 步：Debug 编译

```powershell
pwsh -NoProfile -Command "& { .\BuildDebug.ps1 -NoWait; exit $LASTEXITCODE }"
```

- 工作目录：`GcTdEntity/`（项目根目录）
- 使用 `-NoWait` 参数异步执行，不等待按键
- 检查 exit code：
  - **0 → 成功**，继续下一步
  - **非 0 → 失败**，进入修复流程

### 第 2 步：Release 编译

```powershell
pwsh -NoProfile -Command "& { .\BuildRelease.ps1 -NoWait; exit $LASTEXITCODE }"
```

- 同上，使用 `-NoWait` 异步执行
- 等待完成并检查结果

### 第 3 步：修复编译错误

如果 Debug 或 Release 任意一个编译失败：

1. **仔细阅读编译输出**，定位所有错误信息（文件名、行号、错误代码、错误描述）
2. **逐一修复**：
   - 头文件引用/路径问题
   - 语法错误
   - 类型不匹配
   - 链接错误
   - 警告视为错误的项目需要处理 WarningsAsErrors
3. **重新从第 1 步开始**，直到 Debug 和 Release 均编译通过

### 第 4 步：确认

编译全部通过后，报告结果：

```
[Build Verify Entity]
- Debug:   PASS
- Release: PASS
- 修复轮次: N 轮
```

## 注意事项

- 使用 `-NoWait` 参数避免脚本暂停等待按键
- 两个构建脚本都会执行 SVN update + 更新版本号，这是正常行为
- 如果编译时间过长，可以并行运行两个构建，但输出可能会交错
- Debug 和 Release 的配置差异可能导致 Release 特有的错误（如 `#ifdef NDEBUG` 块中的代码）
- 本 skill 仅作用于 GcTdEntity 项目，不涉及其他子项目
