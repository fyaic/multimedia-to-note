# 问题记录：Agent 频繁删除脚本行为分析

**记录日期**: 2025-12-24
**记录人**: Trae AI Assistant

## ⚠️ 现象描述 (Phenomenon)
在执行 "生成笔记并同步到 Obsidian" 的任务时，基于Gemini-3-Pro 的 Agent 表现出一种重复的行为模式：
1.  接收任务（如同步 `Note_A.md`）。
2.  编写一个**一次性**脚本 `sync_note.py`，其中硬编码了 `Note_A.md` 的文件名。
3.  运行该脚本完成同步。
4.  **立即删除**该脚本。
5.  当用户要求同步 `Note_B.md` 时，Agent 重新编写一个新的 `sync_note_v2.py`，再次硬编码并重复上述过程。

## 原因分析 (Root Cause Analysis)

### 1. 模型训练偏好 (Training Bias)
*   **"不留痕迹"原则**: 大语言模型 (LLM) 在代码解释器 (Code Interpreter) 环境下的训练数据通常鼓励“清理现场”。在临时沙箱环境中，为了避免磁盘空间不足或文件混乱，模型倾向于在任务完成后删除中间产物。
*   **缺乏长期记忆视角**: Agent 默认将每个 Turn 视为独立的任务闭环，而不是一个长期项目的一部分。因此，它优先考虑“完成当前指令并清理”，而不是“构建可复用的基础设施”。

### 2. 交互模式 (Interaction Mode)
*   **一次性交付**: Agent 倾向于交付“结果”（同步完成的状态），而不是“工具”（同步脚本）。
*   **硬编码简单性**: 编写一个硬编码文件名的脚本比编写一个带参数解析 (argparse) 和错误处理的通用脚本要简单、快速，且出错概率更低（对于单次任务而言）。

## 解决方案与改进 (Resolution)

### 1. 固化工具 (Tool Consolidation)
*   **行动**: 创建 `universal_sync_obsidian.py`。
*   **改进**: 脚本支持命令行参数 `python universal_sync_obsidian.py <file_path>`。
*   **结果**: 不再需要为每个文件重新生成代码，只需调用现有工具。

### 2. 工作流标准化 (Workflow Standardization)
*   建立明确的 `Workflow_Audio_Video_Agent.md` 文档，定义脚本为“基础设施”而非“临时产物”。
*   在未来的交互中，优先检查是否存在 `universal_sync_obsidian.py`，如果存在则直接调用。

## 📉 结论 (Conclusion)
这种“写完即删”的行为虽然在单次交互中保持了环境整洁，但在连续工作流中极度低效。通过建立持久化的工具库 (Utility Scripts) 和明确的工作流文档，已纠正此行为，提升了 Agent 作为长期协作伙伴的稳定性。
