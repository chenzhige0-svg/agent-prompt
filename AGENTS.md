# 全局系统提示词（OpenCode + Codex 共用）

> 本文件是唯一维护入口。Windows 与 WSL 的 OpenCode、Codex 都读取这里。

## 共享 Skill 规则

- 用户自建的 skill 统一写入 `~/.agents/skills/<skill-id>/SKILL.md`（Windows 为 `C:\Users\ZhiGe\.agents\skills\<skill-id>\SKILL.md`）。
- 不要把自建 skill 写入 `~/.codex/skills` 或 `~/.config/opencode/skills`。
- 目录名即 skill ID，使用小写 kebab-case。SKILL.md 必须包含 `name` 和 `description`，`description` 写清触发场景。
- 平台差异写在 skill 内部，分 “WSL” 与 “Windows” 两节说明。
- 新建或修改后，告诉用户文件路径。

## 跨项目代码质量约定

- **先遵循项目约定。** 修改代码前读取适用的仓库指令和已有 lint、格式化、类型检查配置。沿用项目的语言、工具和检查入口。只有任务涉及工程规范时，才新增或调整门禁。
- **报错提供可执行的修复指引。** 对可静态判断的项目规则，在现有检查中输出文件位置和稳定的规则编号，说明违规点、约束原因和具体改法。引用该项目真实存在的 API、路径或命令。让规则在触发时提供细节，常规语法和风格问题沿用工具原生诊断。
- **统一风格，减少注释。** 同一文件类型沿用项目选定的格式化器和配置，格式化限于本次变更相关文件。用清晰的命名和控制流表达意图。删掉复述代码、重复文档和充当分隔线的注释，保留解释设计取舍、兼容性、非显然边界的短注释，以及工具指令和有调用价值的文档。
- **在边界验证，用类型表达约束。** 对外部配置、请求和序列化数据，先检查结构、必填字段、可空值和集合大小，再构造具体类型。互斥状态用语言支持的判别联合、枚举或变体表示，各分支只携带所需字段，并尽量用类型检查验证分支穷尽。保持现有公开接口和数据格式兼容。
- **修复根因并复跑。** 不用新增抑制注释、宽泛类型、未经验证的断言或放宽规则来绕过诊断。按项目现有方式处理历史例外。新增检查时用违规、合规、绕过样例验证。对本次改动运行相关 lint、类型检查和测试，说明已验证的范围及阻塞项。

## Lint / format 协议（默认入口）

- 优先使用项目自身的配置与脚本；以下命令仅在项目没有约定时作为默认值使用。命令不存在时跳过并说明，不要临时引入新工具链。
- 改完 Python：`ruff check --fix <files>`，然后 `ruff format <files>`
- Python 类型检查：`pyright`（不要再用 mypy，遗留项目除外）
- 改完 Shell：`shellcheck <files>`；需要格式化时 `shfmt -w -i 2 -ci -sr <files>`
- 改完 JS/TS：优先 `npx --no-install eslint --fix <files>`（项目无本地依赖时用 `eslint --fix`），再 `prettier --write <files>`
- 改完 CSS/Markdown/YAML/JSON：`prettier --write <files>`
- 改完 C/C++：`clang-format -i <files>`；提交前再跑 `clang-tidy -p build <files>`
- 改完 Go：`gofmt -w <files>`；提交前 `golangci-lint run ./...`
- 改完 Rust：`rustfmt <files>`；提交前 `cargo clippy --all-targets -- -D warnings`
- 提交前：`pre-commit run`（只查暂存区）
- 大检查：`pre-commit run --all-files --hook-stage pre-push`
- 禁止 `git commit --no-verify`，除非用户明确要求
- 只格式化本次改动的文件，不要顺手重排无关代码

配置模板与说明：`~/.config/dev-toolbox/`
