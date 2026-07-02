---
name: file-encoding
description: 批量统一源码文件编码为 UTF-8 with BOM + LF 换行符，支持 GBK/Latin-1 等非 UTF-8 文件自动转换。适用：MSVC 编译乱码、跨平台协作换行符混乱、用户明确要求标准化编码。不适用：二进制文件、第三方库目录、仅检查不修复的场景
---

# 统一代码文件编码与换行符规范

## 依赖要求

| 平台 | 依赖 | 检查命令 |
|------|------|---------|
| Windows | PowerShell 5.0+（系统自带） | `powershell -Command "echo ok"` |
| Linux/macOS | Python 3.6+（系统自带） | `python3 --version` |
| 通用（检测用） | `file` 命令 或 `xxd` | `file --version` |
| Git Hook | Bash（Git 自带） | 无需额外安装 |

**前置条件**：当前目录是项目根目录，排除目录已确认（`build/`、`vendor/`、`node_modules/`、`.git/`）。

**适用文件扩展名**：
- C/C++ 必检：`.cpp`, `.h`, `.hpp`, `.c`, `.cc`, `.cxx`, `.hh`, `.hxx`
- 扩展适用：`.py`, `.js`, `.ts`, `.java`, `.rs`, `.go`（UTF-8 但不强制 BOM）

---

## 执行步骤

### 步骤 1：确认平台和模式

询问用户：
- **检测模式**：只报告不合规文件，不修改
- **修复模式**：转换不合规文件

默认先检测，让用户确认后再修复。

### 步骤 2：扫描目标文件

```bash
# 扫描所有目标文件（排除 build/vendor/node_modules/.git）
find . -type f \( -name "*.cpp" -o -name "*.h" -o -name "*.hpp" -o -name "*.c" \) \
  ! -path "*/build/*" ! -path "*/vendor/*" ! -path "*/node_modules/*" ! -path "*/.git/*"
```

### 步骤 3：逐文件检测

对每个文件检查三项：

| 检查项 | 方法 | 合规标准 |
|--------|------|---------|
| 编码 | 尝试 UTF-8 解码；失败则检测 GBK/Latin-1/Shift-JIS | UTF-8 无解码错误 |
| BOM | 读前 3 字节 | `EF BB BF` |
| 换行符 | 检查 `0D 0A` | 只有 `0A`（LF），无 `0D 0A` |

记录不合规文件列表：路径 + 编码 + BOM 状态 + CRLF 状态。

### 步骤 4：检测模式 → 输出报告

```
不合规文件列表：
- src/main.cpp (编码=GBK, BOM=否, CRLF=是)
- src/utils.h (编码=UTF-8, BOM=否, CRLF=否)
- src/legacy.cc (编码=Latin-1, BOM=否, CRLF=是)

共 N 个文件不合规。执行修复？
```

### 步骤 5：修复模式 → 转换文件

**Windows (PowerShell)**：
```powershell
# 检测并转换：自动处理 UTF-8/GBK/Latin-1 等编码 → UTF-8 with BOM + LF
$sourceEncodings = @(
    [System.Text.Encoding]::UTF8,
    [System.Text.Encoding]::GetEncoding(936),   # GBK
    [System.Text.Encoding]::GetEncoding(28591), # Latin-1 (ISO-8859-1)
    [System.Text.Encoding]::GetEncoding(932)    # Shift-JIS
)

function Convert-FileEncoding {
    param([string]$Path)
    $raw = [System.IO.File]::ReadAllBytes($Path)
    $decoded = $null
    $detectedEnc = "unknown"

    foreach ($enc in $sourceEncodings) {
        try {
            $decoded = $enc.GetString($raw)
            # 验证：重新编码后字节一致（排除非精确匹配的编码）
            $detectedEnc = $enc.EncodingName
            break
        } catch { continue }
    }

    if ($null -eq $decoded) {
        Write-Host "编码异常: $Path (无法识别编码，跳过)"
        return
    }

    # 转 LF 换行符
    $decoded = $decoded -replace "`r`n", "`n" -replace "`r", "`n"

    # 写入 UTF-8 with BOM
    $utf8Bom = New-Object System.Text.UTF8Encoding $true
    [System.IO.File]::WriteAllText($Path, $decoded, $utf8Bom)
    Write-Host "已修复: $Path ($detectedEnc → UTF-8 BOM + LF)"
}
```

**Linux/macOS (Python)**：
```python
import os

# 支持的源编码列表，按优先级尝试
SOURCE_ENCODINGS = ['utf-8', 'gbk', 'gb2312', 'gb18030', 'latin-1', 'shift-jis', 'cp932']

def detect_and_convert(path):
    with open(path, 'rb') as f:
        raw = f.read()

    decoded = None
    detected_enc = None

    for enc in SOURCE_ENCODINGS:
        try:
            decoded = raw.decode(enc)
            detected_enc = enc
            break
        except (UnicodeDecodeError, LookupError):
            continue

    if decoded is None:
        print(f"编码异常: {path} (无法识别编码，跳过)")
        return False

    # 移除已有 BOM
    if decoded.startswith('﻿'):
        decoded = decoded[1:]

    # 转 LF 换行符
    decoded = decoded.replace('\r\n', '\n').replace('\r', '\n')

    with open(path, 'wb') as f:
        f.write(b'\xef\xbb\xbf' + decoded.encode('utf-8'))

    print(f"已修复: {path} ({detected_enc} → UTF-8 BOM + LF)")
    return True
```

### 步骤 6：验证转换结果

对每个转换后的文件：读头 3 字节确认为 `EF BB BF`，搜索 `\r\n` 确认为 0 结果，UTF-8 解码确认无错误。

---

## 失败处理

| 失败场景 | 表现 | 处理 |
|---------|------|------|
| 二进制文件误入 | 含 `.png`/`.exe` 等非文本文件 | 检查文件前 4 字节魔数。是二进制 → 输出「检测到二进制文件，跳过：[路径]」，不修改 |
| BOM 重复 | 多次转换后文件头出现多个 `EF BB BF` | 转换前先移除现有 BOM，再写入新 BOM |
| 权限不足 | 文件只读或无写权限 | 输出「无法处理：[路径]（权限不足）」，跳过继续 |
| 非 UTF-8 编码 | 文件为 GBK/Latin-1/Shift-JIS 等 | 自动检测源编码并转换为 UTF-8 with BOM；无法识别时输出「编码异常：[路径]（无法识别编码，跳过）」 |
| 误改第三方库 | `vendor/` 代码被转换产生 diff | 扫描阶段必须排除目录。若已误改，用 `git checkout -- vendor/` 回滚 |
| Python 不可用 (Linux) | `python3` 命令不存在 | 输出「错误：未找到 python3，需 Python 3.6+」 |
| PowerShell 不可用 (Win) | PowerShell 版本过低 | 输出「错误：需要 PowerShell 5.0+」 |
| Git hook 拦截提交 | pre-commit 检查失败 | 先执行步骤 5 修复，`git add` 重新 stage，再提交 |

---

## 输出格式

**检测模式**：
```
不合规：src/main.cpp (编码=GBK, 缺少 BOM, 含 CRLF)
合规：src/utils.h
...
完成：扫描 N 个文件，M 个不合规
```

**修复模式**：
```
已修复：src/main.cpp → UTF-8 with BOM + LF
跳过：vendor/lib.cpp（第三方目录）
...
完成：共处理 N 个文件，修复 M 个，跳过 K 个
```

**批量任务结束**：必须输出统计：`完成：共处理 N 个文件`

---

## 禁止行为

- × 对二进制文件执行编码转换
- × 修改 `build/`、`vendor/`、`node_modules/`、`.git/` 下的文件
- × 仅转换行符不转编码（或反之）— 必须两步都检查
- × 非 C/C++ 项目强制 BOM（其他语言 UTF-8 无 BOM 即可）
- × 转换后不验证结果就声称完成
