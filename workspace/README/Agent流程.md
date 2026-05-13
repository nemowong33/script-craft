# Agent 工作流程

> 本文档描述在 OpenClaw workspace 下，主 Agent（钳多多）+ webtoon-skill + 两个 aligner 的完整协同流程。

## 0. Session 起手仪式

```
[OpenClaw 启动]
    │
    ▼
注入 .openclaw/IDENTITY.md + SOUL.md + USER.md + TOOLS.md + AGENTS.md
    │
    ▼
主 Agent 显示 chandor ASCII 艺术（见 IDENTITY.md）
    │
    ▼
主 Agent 输出欢迎语 + 使用提示
    │
    ▼
执行 [项目状态检测与路由]
    ├─ 无 plot-breakdown.md ────────► 进入 [类型确定阶段]
    ├─ 有 plot-breakdown.md，无剧情点 ► 进入 [剧情拆解阶段]
    └─ 有 plot-breakdown.md，有剧情点 ► 根据"未用"剧情数量决定：
                                         ├─ 未用 > 0：可进 [剧情拆解] 或 [单集剧本]
                                         └─ 未用 = 0：提示先 /拆解
```

## 1. 类型确定阶段

```
用户输入：（任意触发，如打招呼）
    │
    ▼
主 Agent：
    1. 输出 Q1 / Q2 收集小说名 + 类型
    │
    ▼
用户回答 小说名 + 类型
    │
    ▼
主 Agent：调用 webtoon-skill
    │
    ▼
webtoon-skill：
    1. 读取 templates/plot-breakdown-template.md
    2. 按模板生成基础结构（含小说名 + 类型）
    │
    ▼
主 Agent：Write plot-breakdown.md
    │
    ▼
主 Agent：通知用户 → 引导 /扫描 或上传章节 → 再 /拆解
```

## 2. 剧情拆解阶段（`/拆解`）

```
用户：/拆解
    │
    ▼
主 Agent：
    1. Glob novel/*.txt 识别章节
    2. 对比 plot-breakdown.md 已拆解的章节
    3. 确定下一批次 6 章范围 [X, X+5]
    │
    ▼
    ┌─ 如无未拆解章节 → 提示用户上传下 6 章 → 结束
    │
    ▼（有未拆解章节）
主 Agent：
    1. Read plot-breakdown.md
    2. Read 第 X~X+5 章小说原文
    │
    ▼
主 Agent：调用 webtoon-skill
    │
    ▼
webtoon-skill：
    1. 读 adapt-method.md + output-style.md + templates + examples
    2. 按方法论生成剧情点列表（【剧情n】格式，含分集标注，状态：未用）
    3. 返回候选拆解内容（不落盘）
    │
    ▼
主 Agent：调用 breakdown-aligner（自动触发）
    │
    ▼
breakdown-aligner：
    1. 读 adapt-method.md + 小说原文 + 候选拆解
    2. 执行 8 维度检查：
       ├─ 冲突强度评估
       ├─ 情绪钩子识别准确性
       ├─ 冲突密度达标性
       ├─ 分集标注合理性
       ├─ 压缩策略正确性
       ├─ 剧情点描述规范性
       ├─ 原文还原准确性
       └─ 类型特性符合度
    3. 输出 PASS 或 FAIL + 问题清单
    │
    ▼
    ┌─ PASS ────► 主 Agent：Edit plot-breakdown.md（追加批次）
    │               │
    │               ▼
    │            主 Agent：通知用户 → 引导 /拆解 或 /出稿
    │
    └─ FAIL ────► 主 Agent：调用 webtoon-skill（修订模式）
                     │
                     ▼
                   webtoon-skill：根据 FAIL 反馈局部修改
                     │
                     ▼
                   主 Agent：再次调用 breakdown-aligner  ←┐
                     │                                    │
                     ▼                                    │
                   （PASS / FAIL 循环直到 PASS）─────────┘
```

## 3. 单集剧本创作阶段（`/出稿`）

```
用户：/出稿
    │
    ▼
主 Agent：
    1. Read plot-breakdown.md
    2. 识别所有"状态：未用"的剧情点
    3. 按集数连续性确定本批次创作范围
    │
    ▼
    ┌─ 如无未用剧情 → 提示用户先 /拆解 → 结束
    │
    ▼（有未用剧情）
主 Agent：
    1. 确定本批次覆盖的集数范围 [N1, N2]
    2. Read 对应章节的小说原文
    │
    ▼
主 Agent：调用 webtoon-skill
    │
    ▼
webtoon-skill：
    1. 读 adapt-method.md + output-style.md + templates/出稿-template.md + examples/出稿-example.md
    2. 读 plot-breakdown.md（本批次剧情点）
    3. 读小说原文对应章节
    4. 批量创作第 N1 - N2 集剧本
       - 每集 500-800 字
       - 起承转钩结构
       - 视觉符号（※△【】）
       - 结尾必【卡黑】
    5. 返回候选剧本（不落盘）
    │
    ▼
主 Agent：调用 webtoon-aligner（自动触发）
    │
    ▼
webtoon-aligner：
    1. 读 plot-breakdown.md + adapt-method.md + output-style.md
    2. 读 scripts/Episode-[N1-1].md（如 N1 > 1，检查跨集连贯）
    3. 逐集执行 11 维度检查：
       ├─ 剧情点还原一致性
       ├─ 剧情点使用一致性
       ├─ 跨集连贯性（仅 N1 且 N1 > 1）
       ├─ 节奏控制一致性
       ├─ 视觉化风格一致性
       ├─ 人物行为一致性
       ├─ 时间线逻辑一致性
       ├─ 格式规范一致性
       ├─ 悬念设置一致性
       ├─ 类型特性一致性
       └─ 改编禁忌检查
    4. 如含第 20 集 → 额外检查付费转化强度
    5. 输出 PASS 或 FAIL + 问题清单
    │
    ▼
    ┌─ PASS ────► 主 Agent：
    │               1. Write scripts/Episode-N1.md ~ Episode-N2.md
    │               2. Edit plot-breakdown.md：将本批次剧情状态改为"已用"
    │               3. 通知用户 → 引导 /出稿 或 /拆解 或 /进度
    │
    └─ FAIL ────► 主 Agent：调用 webtoon-skill（修订模式）
                     │
                     ▼
                   webtoon-skill：根据 FAIL 反馈精准修改对应集数
                     │
                     ▼
                   主 Agent：再次调用 webtoon-aligner  ←┐
                     │                                  │
                     ▼                                  │
                   （PASS / FAIL 循环直到 PASS）───────┘
```

## 4. 内容修订流程（任意阶段触发）

```
用户提出修改意见
    │
    ▼
主 Agent：
    1. 识别意图类型：
       ├─ 设定调整语（推翻/改设定/重写/调整/...）
       ├─ 约束确立语（必须/不能/统一/固定/...）
       └─ 普通修改意见
    │
    ▼
主 Agent：调用 webtoon-skill（修订模式）
    │
    ▼
webtoon-skill：
    1. 读被改文档当前版本
    2. 读相关基准文档
    3. 局部修改（避免过度）
    │
    ▼
    ┌─ 修改涉及单集剧本 → 调用 webtoon-aligner
    │       │
    │       ▼
    │     PASS → 写入；FAIL → 再修
    │
    ├─ 修改涉及剧情拆解 → 更新 plot-breakdown.md
    │       │
    │       ▼
    │     提醒用户：受影响的剧本（如已创作）可能需要重新创作
    │
    └─ 修改涉及其他 → 直接写入
    │
    ▼
主 Agent：通知用户修改影响范围 → 引导回主流程下一步
```

## 5. 辅助指令流程

### `/扫描`
```
Glob novel/*.txt → 列出所有章节 → 与 plot-breakdown.md 已拆解部分对比 → 报告"已有 / 未拆"状态
```

### `/进度`
```
Read plot-breakdown.md + LS scripts/
    │
    ▼
输出：
- 小说名称 / 类型
- 已拆 X 批次，共 X 个剧情点
- 已用 X 个，未用 X 个
- 已完成 X 集剧本
- 当前阶段 + 下一步建议指令
```

### `/帮助`
```
列出所有 / 指令 + 每个指令的用法 + 工作流程图
```

### `/检查拆解`
```
主 Agent：读指定批次的剧情点 → 调用 breakdown-aligner 手动检查 → 输出结果
```

### `/检查剧本`
```
主 Agent：读所有已创作剧本 → 调用 webtoon-aligner 全量检查 → 输出结果
```

## 6. 异常处理

| 场景 | 主 Agent 应对 |
|------|---------------|
| aligner 连续 3 次 FAIL 仍未 PASS | 向用户汇报问题清单，询问是否人工介入（例如调整小说类型、补充上下文）。 |
| `novel/` 中章节命名不规范 | 在 `/扫描` 时报告异常文件名并建议修正。 |
| `plot-breakdown.md` 被手工编辑后格式损坏 | 在读取时校验结构；异常时提示用户确认。 |
| 单集字数 > 800 或 < 500 | webtoon-aligner 维度 4 / 11 触发 FAIL，进入修订循环。 |
| 用户要求跳过 aligner | 拒绝；SOUL.md 边界中已声明"不跳过质量门禁"。 |
