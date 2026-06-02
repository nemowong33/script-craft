# Script Craft - 网文改编漫剧剧本与漫剧剧本原创工作区

> 两个功能：网文改编漫剧剧本、漫剧剧本原创。

## 项目简介

Script Craft 由主 Agent「钳多多」调度两个功能：

- **网文改编漫剧剧本**：读取小说章节，拆解剧情点，生成漫剧单集剧本。
- **漫剧剧本原创**：从创意、角色、世界观开始原创漫剧项目。

## 两个入口

| 入口 | 适用场景 | 工作区 |
|------|----------|--------|
| `/改编` | 把网文/小说改成漫剧剧本 | `adaptation/novel/`、`adaptation/plot-breakdown.md`、`adaptation/scripts/` |
| `/创作` | 从零原创漫剧剧本 | `original/` |

## 项目结构

```text
workspace/
├── adaptation/                    # 网文改编工作区
│   ├── novel/
│   │   ├── chapter-001.txt
│   │   └── ...
│   ├── plot-breakdown.md
│   └── scripts/
│       ├── episode-01.md
│       └── ...
├── original/                       # 漫剧剧本原创工作区
│   ├── plan.md
│   ├── characters.md
│   ├── outline.md
│   ├── quality-check.md
│   └── scripts/
│       ├── episode-01.md
│       └── ...
├── skills/
│   ├── adaptation-skill/              # 网文改编漫剧
│   └── original-skill/                # 漫剧剧本原创
└── agents/
    ├── adaptation-breakdown-aligner/  # 网文拆解检查
    ├── adaptation-script-aligner/     # 网文改编剧本检查
    └── original-aligner/              # 漫剧原创剧本检查
```

## 网文改编漫剧剧本

输入是小说章节，流程只使用 `adaptation-skill`。

| 指令 | 功能 |
|------|------|
| `/改编` | 开启网文改编流程 |
| `/扫描` | 扫描 `adaptation/novel/` 章节 |
| `/拆解` | 拆解 6 章小说并写入 `adaptation/plot-breakdown.md` |
| `/出稿` | 根据未用剧情点生成 `adaptation/scripts/episode-XX.md` |
| `/检查拆解` | 检查剧情拆解质量 |
| `/检查剧本` | 检查改编剧本一致性 |

## 漫剧剧本原创

输入是创意和设定，不读取 `adaptation/novel/`，不使用 `adaptation/plot-breakdown.md`。

| 指令 | 功能 |
|------|------|
| `/创作` | 开启漫剧剧本原创流程 |
| `/方案` | 生成 `original/plan.md` |
| `/角色` | 生成 `original/characters.md` |
| `/大纲` | 生成 `original/outline.md` |
| `/出稿 N` | 生成 `original/scripts/episode-XX.md` |
| `/自检` | 生成 `original/quality-check.md` |

`/出稿` 会根据当前功能路由：在 `/改编` 中生成改编剧本，在 `/创作` 中生成原创剧本。

## 边界原则

- 网文改编不写入 `original/`。
- 原创创作不读取小说章节，不做原文还原。
- `/创作` 只表示漫剧剧本原创。
- 用户只说“剧本”时，先确认是 `/改编` 还是 `/创作`。

## 质量门禁

- `adaptation-breakdown-aligner`：只检查网文拆解。
- `adaptation-script-aligner`：只检查网文改编漫剧剧本。
- `original-aligner`：检查漫剧剧本原创。
