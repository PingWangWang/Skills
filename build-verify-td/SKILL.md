---
name: build-verify-td
description: Use when building Thermal Desktop projects (D16_ThermalDesktop) — compiling GcTdEntity and GcThermalDesktop, running UpdateSDK, fixing compilation errors. Trigger: user mentions TD building/building errors/GcTdEntity/GcThermalDesktop/build scripts
---

# Build & Verify — Thermal Desktop 编译流程

## 依赖要求

- 执行目录：`D16_ThermalDesktop` 项目根目录
- 脚本路径（相对项目根）：
  - `GcTdEntity\BuildDebug.ps1`
  - `GcTdEntity\BuildRelease.ps1`
  - `GcThermalDesktop\UpdateSDK.ps1`
  - `GcThermalDesktop\BuildDebug.ps1`
  - `GcThermalDesktop\BuildRelease.ps1`
- 环境：PowerShell、MSBuild（Visual Studio）、SVN 命令行

---

## 执行步骤

### 步骤 1：编译 GcTdEntity

按顺序执行以下脚本（均带 `-NoWait` 参数）：

```powershell
.\GcTdEntity\BuildDebug.ps1 -NoWait
.\GcTdEntity\BuildRelease.ps1 -NoWait
```

**说明**：
- `-NoWait` 使脚本跳过结尾的「按任意键退出」等待，适合自动化执行
- Debug 先于 Release，不可颠倒
- 脚本内部自动执行 SVN update + 版本号更新 + MSBuild 编译

### 步骤 2：更新 GcThermalDesktop SDK

```powershell
.\GcThermalDesktop\UpdateSDK.ps1 -NoWait
```

**说明**：将 `..\GcTdSDK` 的最新 SDK 拷贝到 `GcThermalDesktop\GcTdSDK`，确保 GcThermalDesktop 编译时使用最新的 SDK 头文件和库。

### 步骤 3：编译 GcThermalDesktop

按顺序执行以下脚本（均带 `-NoWait` 参数）：

```powershell
.\GcThermalDesktop\BuildDebug.ps1 -NoWait
.\GcThermalDesktop\BuildRelease.ps1 -NoWait
```

### 步骤 4：检查编译结果

- 每个脚本执行后检查其退出码（`$LASTEXITCODE`）
- 记录成功的步骤和失败的步骤
- 如有编译失败，进入步骤 5

### 步骤 5：修复编译错误

1. 读取 MSBuild 输出的错误信息，定位错误文件和行号
2. 逐一分析根因（语法错误、链接错误、类型不匹配、头文件缺失等）
3. 修复代码
4. 重新编译失败的脚本（单独执行对应脚本验证修复）
5. 重复直到所有编译通过

---

## 注意事项

| 要点 | 说明 |
|------|------|
| `-NoWait` | 所有脚本必须带此参数，否则脚本会等待按键阻塞执行 |
| 执行顺序 | GcTdEntity → UpdateSDK → GcThermalDesktop，不可打乱 |
| Debug → Release | 每个项目先 Debug 后 Release，不可颠倒 |
| SVN 依赖 | 脚本依赖 SVN 工作副本信息获取版本号，确保在工作副本内执行 |
| 编译日志 | 关注 MSBuild 输出的 error/warning，优先处理 error |
| 重复编译 | 若修复后重新编译某脚本，同样需要带 `-NoWait` |

---

## 失败处理

| 失败场景 | 处理方式 |
|---------|---------|
| SVN 更新失败 | 检查 SVN 工作副本状态，可能是冲突或网络问题 |
| MSBuild 未找到 | 检查 Visual Studio 安装，或通过 vswhere 定位 |
| GcTdEntity 编译失败 | 查看 MSBuild 错误输出，修复后重试单个脚本 |
| UpdateSDK 失败 | 检查 GcTdSDK 源目录是否存在 |
| GcThermalDesktop 编译失败 | 先确认 UpdateSDK 已成功执行，再查看编译错误 |
| 修复后仍有错误 | 逐条修复，每次只重编失败的项目+配置 |

---

## 快速参考

```powershell
# 完整编译流程（项目根目录执行）
.\GcTdEntity\BuildDebug.ps1 -NoWait
if ($LASTEXITCODE -ne 0) { Write-Host "GcTdEntity Debug 编译失败" -ForegroundColor Red }
.\GcTdEntity\BuildRelease.ps1 -NoWait
.\GcThermalDesktop\UpdateSDK.ps1 -NoWait
.\GcThermalDesktop\BuildDebug.ps1 -NoWait
.\GcThermalDesktop\BuildRelease.ps1 -NoWait
```

```powershell
# 单独重编译某项（修复后验证）
.\GcTdEntity\BuildDebug.ps1 -NoWait
.\GcThermalDesktop\BuildRelease.ps1 -NoWait # 替换为需重编的脚本
```

---

## 禁止行为

- × 跳过 `-NoWait` 参数（会导致脚本挂起等待按键）
- × 先 Release 后 Debug
- × 先 GcThermalDesktop 后 GcTdEntity
- × 跳过 UpdateSDK 直接编译 GcThermalDesktop
- × 批量无脑重跑所有脚本而不是先分析错误根因
