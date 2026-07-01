---
name: file-encoding-standardization
description: 统一代码文件编码与换行符规范 — UTF-8 with BOM + LF，确保跨平台兼容和 MSVC 编码识别
---

# 统一代码文件编码与换行符规范

**Why:**
确保所有源码文件使用 **UTF-8 with BOM** 编码和 **LF** 换行符，解决以下问题：

1. **跨平台兼容性**：Windows/Linux/macOS 换行符处理不同，LF 确保一致性。
2. **MSVC 编码识别**：无 BOM 的 UTF-8 文件可能被 MSVC 误解析为本地编码（如 GBK），导致中文注释乱码或编译错误。BOM 强制指定编码。
3. **Git 差异洁净**：混合换行符导致 diff 出现无意义变更，增加审查负担。

**Scope:**

- **适用文件**：`.cpp`, `.h`, `.hpp`, `.c`, `.cc`, `.cxx`, `.hh`, `.hxx` 等源码文件。
- **扩展适用**：`.py`, `.js`, `.ts`, `.java`, `.rs`, `.go` 等文本文件（编码统一为 UTF-8，换行符为 LF；BOM 仅 C/C++ 场景必需）。
- **排除文件**：二进制文件（图片、音频）、第三方库文件、`build/` / `vendor/` / `node_modules/` 下自动生成的文件。

---

## Trigger conditions（满足任一即触发）

1. **文件保存时**：编辑器保存源码文件后自动检测编码与换行符。
2. **批量转换命令**：用户要求标准化编码或换行符。
3. **跨平台协作**：接收来自不同平台的代码（PR 检出、项目clone）。
4. **构建失败**：MSVC 编译因编码问题报错（中文注释乱码等）。

---

## How to apply

### 1. 检查文件合规性

```
BOM 检测：    文件前 3 字节是否为 EF BB BF
换行符检测：  是否含 0D 0A（CRLF），理想状态仅 0A（LF）
编码检测：    能否以 UTF-8 解码，无解码错误
```

### 2. 自动转换流程

```
读取文件 → 检测编码 → 检测换行符 → 转换 → 验证 → 输出结果
```

### 3. 跨平台转换命令

**Windows (PowerShell)：**

```powershell
# 批量转换 .cpp/.h 文件为 UTF-8 with BOM + LF
Get-ChildItem -Recurse -Include *.cpp, *.h | ForEach-Object {
    $content = Get-Content -Raw -Path $_.FullName
    # 移除 BOM（如有）再写入带 BOM + LF 的 UTF-8
    $utf8Bom = New-Object System.Text.UTF8Encoding $true
    [System.IO.File]::WriteAllText($_.FullName, $content, $utf8Bom)
    Write-Host "已修复: $($_.FullName)"
}
```

```powershell
# 仅检测不合规文件（不修改）
Get-ChildItem -Recurse -Include *.cpp, *.h | ForEach-Object {
    $bytes = [System.IO.File]::ReadAllBytes($_.FullName)
    $hasBom = $bytes[0] -eq 0xEF -and $bytes[1] -eq 0xBB -and $bytes[2] -eq 0xBF
    $hasCrlf = [System.Text.Encoding]::UTF8.GetString($bytes) -match "`r`n"
    if (-not $hasBom -or $hasCrlf) {
        Write-Host "不合规: $($_.FullName) (BOM=$hasBom, CRLF=$hasCrlf)"
    }
}
```

**Linux/macOS (Python)：**

```python
# batch_convert_encoding.py
import os, sys

def convert_file(path):
    with open(path, 'rb') as f:
        raw = f.read()
    
    # 解码，去除 BOM
    if raw[:3] == b'\xef\xbb\xbf':
        content = raw[3:].decode('utf-8')
    else:
        content = raw.decode('utf-8')
    
    # 转换换行符
    content = content.replace('\r\n', '\n').replace('\r', '\n')
    
    # 写出 UTF-8 with BOM + LF
    with open(path, 'wb') as f:
        f.write(b'\xef\xbb\xbf' + content.encode('utf-8'))
    
    return True

# 批量处理
target_exts = ('.cpp', '.h', '.hpp', '.c', '.cc', '.cxx')
for root, dirs, files in os.walk('.'):
    dirs[:] = [d for d in dirs if d not in ('build', 'vendor', 'node_modules', '.git')]
    for f in files:
        if f.endswith(target_exts):
            convert_file(os.path.join(root, f))
```

**Git 预提交 Hook（.git/hooks/pre-commit）：**

```bash
#!/bin/sh
# 检查即将提交的文件是否为 UTF-8 with BOM + LF
for file in $(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(cpp|h|hpp|c|cc)$'); do
    if [ -f "$file" ]; then
        # 检查 BOM
        if [ "$(head -c 3 "$file")" != "$(printf '\xef\xbb\xbf')" ]; then
            echo "错误: $file 缺少 UTF-8 BOM"
            exit 1
        fi
        # 检查 CRLF
        if grep -rl $'\r' "$file" > /dev/null 2>&1; then
            echo "错误: $file 含有 CRLF 换行符"
            exit 1
        fi
    fi
done
```

---

## 快速参考

| 场景 | 检查方法 | 修复方法 |
|------|----------|----------|
| 检查 BOM | `head -c 3 file | xxd` 或 Hex 编辑器 | 写入 `EF BB BF` 前缀 |
| 检测 CRLF | `file file.cpp` 或 `grep -rl $'\r'` | 替换为 LF |
| 批量修复（Win） | 上方 PowerShell 脚本 | 同上脚本 |
| 批量修复（Linux） | 上方 Python 脚本 | 同上脚本 |
| Git 提交阻止 | `pre-commit` hook | 修复后重新 stage |
| VS Code 查看 | 状态栏右下角的编码/换行符指示 | 点击切换或 `>Change End Of Line Sequence` |

---

## 常见错误

| 错误 | 表现 | 修正 |
|------|------|------|
| BOM 重复 | 反复转换后文件头出现多个 BOM | 转换前应移除现有 BOM 再写入 |
| 二进制损坏 | 对 `.png`/`.exe` 执行编码转换 | 按扩展名过滤 + 检测前 4 字节魔数 |
| 只改换行符不改编码 | 换行符 LF 但编码是 GBK | 两步都要执行，先转编码再转换行符 |
| 误改第三方库 | `vendor/` 代码被转换，产生大量 diff | 排除目录必须在脚本中硬编码 |
| 提交时才发现 | CI 失败才发现编码不合规 | 配置 pre-commit hook 提前拦截 |

---

## Verification（自我检查清单）

- [ ] 文件头 3 字节为 `EF BB BF`
- [ ] 无 `0D 0A`（CRLF），仅含 `0A`（LF）
- [ ] 中文注释在 MSVC 编译无乱码
- [ ] 批量转换后 diff 仅涉及目标文件（无误改）
- [ ] 排除目录未受影响（build/vendor/node_modules）
- [ ] pre-commit hook 生效：违规提交被阻止
- [ ] 转换前后功能测试通过（编码变更不改变语义）

---

## Output rules

1. **成功转换**：输出 `已修复 [文件名]：UTF-8 with BOM + LF`
2. **文件合规**：静默通过，不输出。
3. **错误处理**：
   - 无法访问 → `无法处理：[路径]（权限不足）`
   - 二进制误触 → `检测到二进制文件，跳过：[路径]`
   - 解码失败 → `编码异常：[路径]（非 UTF-8）`
4. **统计结束**：批量任务完成后输出 `完成：共处理 N 个文件`

---

## 备注

- **BOM 争议**：尽管部分场景反对 BOM，在跨平台 C/C++ 编译与 MSVC 兼容性权衡下，此为必要折中。仅 C/C++ 场景强制 BOM；其他语言用不带 BOM 的 UTF-8。
- **性能优化**：缓存文件哈希值，仅对修改过的文件执行检查，避免重复扫描。
- **推荐集成**：pre-commit hook + CI lint 阶段双重保障。
