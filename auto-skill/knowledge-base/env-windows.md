---
skill_id: "ENV_WINDOWS"
category: "Core"
tags: ["Windows", "Python", "CLI"]
complexity: "Medium"
---

## 🔧 Windows 环境执行经验 (Python 解释器)
**日期：** 2026-04-07
**情境：** 在 Windows 系统上执行 Python 脚本时，直接使用 `python` 经常会遇到 "Python was not found" 并跳转 Microsoft Store 的报错。
**最佳实践：**
- **必须使用** `py` 启动器而不是 `python` 命令。例如：`py .\script.py`
- 不要依赖系统 PATH 中的 `python`，因为 Windows 商店的空执行档优先级很高。在使用 `run_command` 运行 Python 脚本前，必须记忆此条规则。
