# agent-prompt

OpenCode 与 Codex 共用的全局系统提示词。Windows 与 WSL 读取同一份文件，只在 `AGENTS.md` 中维护。

## 包含内容

- 对话人设与硬性规则（语言、准确性、格式）
- 项目事实与跨项目代码质量约定
- Lint / format 默认协议（项目自身配置优先）
- 提示词交付约定、Code First
- 本机工具清单
- 共享 Skill 规则
- OpenCodeReview（OCR）审查策略

## 读取关系

| 环境 | 工具 | 路径 | 方式 |
| --- | --- | --- | --- |
| Windows | OpenCode | `%USERPROFILE%\.config\opencode\AGENTS.md` | 符号链接 |
| Windows | Codex | `%USERPROFILE%\.codex\AGENTS.md` | 符号链接 |
| WSL | OpenCode | `~/.config/opencode/AGENTS.md` | 符号链接 |
| WSL | Codex | `~/.codex/AGENTS.md` | 符号链接 |

Windows 与 WSL 的所有入口最终指向：

- Windows：`%USERPROFILE%\.agents\AGENTS.md`
- WSL：`/mnt/c/Users/<WindowsUser>/.agents/AGENTS.md`

本机实际用户名为 `lenovo`，即 `C:\Users\lenovo\.agents\AGENTS.md`。

## 首次安装

把本仓库的 `AGENTS.md` 放到 Windows 用户目录：

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.agents" | Out-Null
Copy-Item .\AGENTS.md "$env:USERPROFILE\.agents\AGENTS.md" -Force
```

创建 Windows 符号链接（Developer Mode 打开时不需要管理员权限；没有权限就用 `-ItemType HardLink`）：

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.config\opencode" | Out-Null
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex" | Out-Null

Remove-Item "$env:USERPROFILE\.config\opencode\AGENTS.md" -Force -ErrorAction SilentlyContinue
Remove-Item "$env:USERPROFILE\.codex\AGENTS.md" -Force -ErrorAction SilentlyContinue

New-Item -ItemType SymbolicLink `
  -Path "$env:USERPROFILE\.config\opencode\AGENTS.md" `
  -Target "$env:USERPROFILE\.agents\AGENTS.md" | Out-Null

New-Item -ItemType SymbolicLink `
  -Path "$env:USERPROFILE\.codex\AGENTS.md" `
  -Target "$env:USERPROFILE\.agents\AGENTS.md" | Out-Null
```

在 WSL 中创建符号链接（把 `<WindowsUser>` 换成实际的 Windows 用户名）：

```bash
mkdir -p ~/.codex ~/.config/opencode
ln -sfn /mnt/c/Users/<WindowsUser>/.agents/AGENTS.md ~/.codex/AGENTS.md
ln -sfn /mnt/c/Users/<WindowsUser>/.agents/AGENTS.md ~/.config/opencode/AGENTS.md
```

## 更新

只修改本仓库的 `AGENTS.md`，然后同步到 Windows 本机：

```powershell
Copy-Item .\AGENTS.md "$env:USERPROFILE\.agents\AGENTS.md" -Force
```

WSL 不需要再次复制，符号链接会直接读到 Windows 侧的新内容。

> 如果用“先删除再重命名”的方式保存文件，链接可能断开。检查：
> `Get-Item "$env:USERPROFILE\.config\opencode\AGENTS.md" | Select-Object LinkType, Target`
> 输出为空或指向自身，就删掉重建。

## 说明

这里维护的是可维护的全局指令层（`AGENTS.md`）。OpenCode 和 Codex 各自编译进程序的底层系统提示词仍然保留；整体替换底层提示词会导致工具调用等内置行为失效。
