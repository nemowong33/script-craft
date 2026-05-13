# TOOLS

> 本文件是"本 workspace 的工具使用约定"，不控制工具可见性，只约定 Agent 如何使用它们。

## 文件系统（必用）
- **Read**：读取 `novel/chapter-XXX.txt`、`plot-breakdown.md`、`scripts/Episode-XX.md`、skill 内部文档（`adapt-method.md`、`output-style.md`、templates、examples）。
- **Write / Edit**：
  - 创建、追加、修改 `plot-breakdown.md`。
  - 写入 `scripts/Episode-XX.md`。
  - **所有写操作都必须发生在对应 aligner 返回 PASS 之后**。
- **Glob / LS**：扫描 `novel/` 目录，识别已上传章节（服务于 `/扫描` 指令）。

## Skill 调用（核心）
- **webtoon-skill**：唯一的"改编执行引擎"。
  - 调用时机：类型确定、剧情拆解、单集剧本、内容修订。
  - 调用方式：`调用 webtoon-skill`。
- 所有生成式内容（拆解、剧本、修订）**都必须**经由 `webtoon-skill`，主 Agent 不得绕过它直接生成内容。

## Sub-Agent 调用（强制质量门禁）
- **breakdown-aligner**：剧情拆解质量校验（8 维度）。
  - 触发：每完成 1 批次（6 章）拆解后自动触发。
  - 手动触发：`/检查拆解`。
- **webtoon-aligner**：单集剧本一致性校验（11 维度）。
  - 触发：每批次剧本创作完成后自动触发，或用户提出设定调整/约束确立类指令时触发。
  - 手动触发：`/检查剧本`。

## 质量门禁止损约定
- 同一批次若被 `breakdown-aligner` 或 `webtoon-aligner` **连续 FAIL 2 次**，主 Agent 必须停止无限自动返工，转入“人工确认模式”。
- 若连续两轮以上反馈核心问题高度重复，主 Agent 必须判定为“重复门禁反馈”，停止继续整版重写。
- 进入“人工确认模式”后，主 Agent 必须向用户提供：当前最优稿、卡点原因、继续微调/确认落盘/推翻重做三种路径。
- 用户明确确认的版本可作为“用户确认版”落盘，用于止损，不得再围绕同一轮反馈无限循环。

## 禁忌
- ❌ 绕过 aligner 直接写盘（除非已进入人工确认模式且用户明确授权落盘）。
- ❌ 使用 webtoon-skill 之外的"自我生成"代替改编。
- ❌ 删除或覆盖已落盘的单集剧本（除非经过 webtoon-aligner 重新 PASS，或用户在人工确认模式下明确要求重做）。
