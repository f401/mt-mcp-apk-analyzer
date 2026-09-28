# MT MCP Skill · MT管理器 MCP 技能包

![MT Manager](https://img.shields.io/badge/MT%20Manager-26091198-2ea44f)
![Skill](https://img.shields.io/badge/Skill-SKILL.md-8957e5)
![Platform](https://img.shields.io/badge/Platform-Android-3ddc84)

> 面向 AI 助手（Agent Skills / `SKILL.md`）的 **MT管理器 MCP 服务（`mt_mcp`）** 技能：让大模型直接对 APK 进行逆向分析、编辑重打包，并管理设备文件。
>
> **适配 MT管理器 版本：`26091198`**

*An Agent Skill for **MT Manager**'s MCP server (`mt_mcp`) — APK reverse-engineering, patching & repacking, native (SO) analysis, and device file operations, driven directly by an AI assistant.*

## ✨ 功能特性

- 🔍 **APK 分析** —— DEX/Smali、资源表、SO 原生库、签名证书、敏感字符串、调用链追踪
- ✏️ **编辑重打包** —— Smali / AXML / 资源 / SO 补丁，构建并签名新 APK（不改动源文件）
- 🧩 **原生分析** —— ELF 摘要、导入/符号/重定位、反汇编、控制流图、交叉引用
- 📁 **文件管理** —— 读写 / 搜索 / 复制 / 移动 / 追加 / 字节补丁，与 APK 工作区条目互通
- ⚡ **渐进式加载** —— 核心 `SKILL.md` + 5 份专题参考，按需读取、不挤占上下文

## 📦 适配版本

| 组件 | 版本 | 说明 |
|------|------|------|
| MT管理器（含 `mt_mcp` 服务） | **`26091198`** | 技能按此版本 MCP 的实际行为编写并实测验证 |
| 本技能 | v1（2026-09-28） | 见 [更新记录](#-更新记录) |

> 若你的 MT管理器版本与本技能适配版本存在差异，部分工具行为可能不同；建议先让助手实测确认再执行修改类操作。

## 📂 仓库结构

```
.
├── README.md
└── mt-mcp-apk-analyzer/          # 技能本体
    ├── SKILL.md                  # 核心规则 + 全部工具索引
    └── references/
        ├── apk-analysis.md       # 搜索 / 读取 / 分析细则
        ├── apk-editing.md        # 编辑 / 构建 / 签名细则
        ├── native.md             # SO 原生库全流程
        ├── file-tools.md         # mt_file_* 文件工具细则
        └── troubleshooting.md    # 错误表 / 分页规则 / 排查流程
```

## 💬 使用示例

| 你说 | 它会 |
|------|------|
| “分析一下这个 APK 里有哪些支付相关的接口” | 打开工作区 → 定位搜索 → 输出结构化报告 |
| “把 VIP 校验跳过，重新打包一个签名版” | 定位代码 → 临时工作区改 Smali → 构建签名 APK |
| “看看 libxxx.so 里有哪些反调试逻辑” | native_inspect → 符号 / 字符串 / 反汇编分析 |
| “把这个 SO 里的某字符串改成另一个” | 定位字符串 → UTF-8 窗口补丁 → 重新构建 |
| “读取日志文件并搜索报错关键字” | 文件读取 → 逐行搜索 → 返回匹配与上下文（需在权限范围内） |

> 编辑类操作会先建**临时工作区**与隔离编辑会话，验证构建无误后才产出 APK；**源文件全程不被修改**。

## 🧰 工具覆盖

<details>
<summary>共 49 个 MCP 工具（按 MT管理器 26091198 的工具集）</summary>

- **工作区 / 读取（13）**：open、list_available_apks、list_workspaces、list、search、read_text、read_bytes、dex_outline_class、dex_xref、resource_read、resource_xref、read_signature、continue
- **原生（6）**：native_inspect、native_read_items、native_map_address、native_disassemble、native_function_cfg、native_xref
- **编辑 / 构建（10）**：edit_open、edit_text、delete、edit_resource、patch_bytes、native_patch_instructions、native_patch_string、edit_check、build、close
- **文件（20）**：access_policy、list、stat、read_text、read_bytes、edit_text、write_bytes、replace_text、patch_bytes、append_text、append_bytes、truncate_bytes、copy、move、delete、create_directory、delete_directory、search、search_text、copy_workspace_entry

</details>

## ✅ 验证说明

技能内容基于真实设备端到端验证（MT管理器 `26091198`）：

- **只读链路**：打开工作区 → 搜索与翻页 → 反汇编 → 签名读取；
- **编辑闭环**：`edit_open → edit_text → edit_check(passed) → build(sign=true) → 产物复核 → delete → close`；
- **行为校准**：按最新版实测修正了模糊匹配移除、`delete_target` 模式移除、`build` 的 `sign` 参数、反汇编地址前缀、`continue` 判定等差异点。

## ❓ 常见问题

- **技能没有触发？** 确认客户端已加载 `skills` 目录、MT管理器 的 MCP 服务已连接，且对话包含相关意图（分析 / 修改 APK、文件操作等），或直接点名技能。
- **报 `FILE_ACCESS_DENIED`？** 目标路径超出 MT管理器 MCP 的文件访问范围，请在 MT管理器 的 MCP 设置中调整（Home 目录默认可读写）。
- **会把原 APK 改坏吗？** 不会。源 APK 全程只读：编辑发生在隔离会话中，构建输出为新文件。
- **为什么编辑前要先“读一遍”？** 编辑采用 `targetVersion` 过期守卫：先读取获取版本，再携带版本提交修改，避免基于过期数据改动。

## 📝 更新记录

### v1 · 2026-09-28

- 适配 **MT管理器 `26091198`** 的 `mt_mcp` 行为，全面重写；
- 新增覆盖：`mt_apk_delete` / `mt_apk_list_workspaces` / `mt_apk_read_signature` / 全套 `mt_file_*`（20 个）；
- 结构升级：单文件 → `SKILL.md` + `references/` 五份专题文档（按需加载）；
- 修正：`open` 不再支持模糊匹配、`edit_text` 的 `delete_target` 模式移除、`build` 必填 `sign`、反汇编地址带 `0x` 前缀、`continue` 判定收紧等。
- 融合实践要点：省磁盘策略（纯分析默认临时工作区、用完即释放）、`limit=1` 快速统计、错误码格式说明。

## ⚠️ 免责声明

本项目仅供**学习与技术研究**使用，请遵守相关软件许可与服务条款，勿用于任何违法用途。
