# v1 源清单 Manifest

> Wayfinder 票：[盘点并冻结 v1 源清单](https://github.com/Kunkgg/knowledge_base/issues/4) · 冻结日期 2026-08-23
> 供 ingest 流程消费：批内文件即分批单位，剔除项不进 `_staging/`。

## 总览

| 源 | 实测文件数 | 进 v1 | 剔除 |
|---|---|---|---|
| `~/Repos/teach_ai`（文档类） | 35 | **31** | 4 + 代码/基础设施 |
| `~/Repos/Obsidian-Vault`（md） | 60 | **23** | 37 |
| 合计 | — | **54** | — |

- Vault 实测 60 个 md（map 记 61，差异为近期清理/移动，以本清单为准）。
- 第三源（用户收集的优质 HTML）不在此机，不进 v1，留门（见 map）。

**私密度标记**：🟢 公开安全 · 🟡 个人语境（自托管 OK，公开切换前需逐篇复议）· 🔴 绝不进 wiki（密钥）。
**质量标记**：高 = 结构完整、经教学验证；中 = 真实内容但不完整；低 = 骨架/草稿；空 = 无正文。

## 分批 ingest 顺序

| 批 | 内容 | 文件数 | 量级 | 理由 |
|---|---|---|---|---|
| **B1** | teach_ai lessons + reference HTML | 14 | ~270KB HTML | 质量最高的主线知识；HTML 按章节切源是质量第一变量（见 issue #3 决议），单独成批控制切分风险 |
| **B2** | teach_ai md：learning-records + MISSION + RESOURCES | 17 | ~40KB | 决策与计划层，轻量；与 B1 同域，紧随其后便于交叉链接 |
| **B3** | vault yazi 全套 md | 22 | ~1160 行 | 独立完整课程，自成知识域 |
| **B4** | vault 极客时间 index.md（+ assets 图片） | 1 + 2 png | 35 行 | 收尾小批；图片作为 display assets 原样拷贝，不作知识源 ingest |

每批 ≈ 一个 ingest 会话；批内顺序即上表文件列举顺序。

## teach_ai 明细（31 进 / 其余剔除）

### B1 — lessons 与 reference（14，全 🟢）

| 文件 | 主题 | 质量 |
|---|---|---|
| `lessons/0001-ai-engineering-landscape.html` | AI 工程全景 | 高 |
| `lessons/0002-llm-api-first-call.html` | LLM API 调用 | 高 |
| `lessons/0003-vector-db-chunking.html` | Vector DB / Chunking | 高 |
| `lessons/0004-rag-mvp.html` | RAG MVP | 高 |
| `lessons/0005-langchain-lcel.html` | LangChain / LCEL | 高 |
| `lessons/0006-langchain-rag.html` | LangChain RAG | 高 |
| `lessons/0007-doc-loader-splitter.html` | Loader / Splitter | 高 |
| `lessons/0008-tool-calling.html` | Tool Calling | 高 |
| `lessons/0009-langgraph-hello.html` | LangGraph 入门 | 高 |
| `lessons/0010-langgraph-source-qa.html` | LangGraph 源码三问 | 高 |
| `lessons/0011-langgraph-memory.html` | Checkpointer / 记忆 | 高 |
| `lessons/0012-interrupt-hitl.html` | interrupt / HITL | 高 |
| `reference/0001-engineering-pitfalls.html` | AI 工程避坑（幽灵 Bug） | 高 |
| `reference/glossary.html` | 术语表（~48KB，词条密集） | 高 |

注：lessons 均为 cloze 教学格式 + 已验证；`glossary.html` 是 wiki 词条的黄金源，切源时按词条粒度。

### B2 — 记录与计划（17）

| 文件 | 主题 | 质量 | 私密 |
|---|---|---|---|
| `learning-records/0001` … `0015`（15 个） | 学习决策与非显然结论（API 组合、embedding 内部、superstep 语义等） | 高 | 🟢 |
| `MISSION.md` | 学习计划与成功标准 | 高 | 🟡 含年龄/在职状态/面试动机 |
| `RESOURCES.md` | 分级验证资源清单 | 高 | 🟢 |

### teach_ai 剔除项

| 项 | 理由 |
|---|---|
| `.env` | 🔴 API key，绝不进 wiki、不进任何 git 分支内容 |
| `NOTES.md` | 教学过程 session log，私密度高（个人背景、工作项目语境）；insight 已沉淀于 learning-records，留档不进 v1 |
| `lessons/exercise-0004-retrieve-cloze.html` | cloze 格式 demo，已被 0005+ 课程内建格式取代，非知识内容 |
| `src/` `tests/` `test_*.py` `chroma_db/` `pyproject.toml` `uv.lock` `README.md` `assets/*.css|js` | 代码与基础设施，非知识源 |

## Obsidian-Vault 明细（23 进 / 37 剔除）

### B3 — yazi 全套（22）

| 组 | 文件 | 质量 | 私密 |
|---|---|---|---|
| 导航 | `30_notes/yazi/index.md` | 高 | 🟢 |
| 计划 | `30_notes/yazi/MISSION.md`、`RESOURCES.md` | 高 | 🟢 |
| 课程 | `30_notes/yazi/lessons/0001`–`0009`（9 个，md 格式） | 高 | 🟢 |
| 记录 | `30_notes/yazi/learning-records/0001`–`0007`（7 个） | 高 | 🟢 |
| 速查 | `30_notes/yazi/reference/0001`–`0002`（2 个） | 高 | 🟢 |
| 教学笔记 | `30_notes/yazi/NOTES.md` | 高 | 🟡 大量本机环境细节（WSL 路径、版本），ingest 时作为事实源但需标注环境限定 |

### B4 — 极客时间 AI 原生工作流（1）

| 文件 | 主题 | 质量 | 私密 |
|---|---|---|---|
| `30_notes/ai/极客时间_AI原生工作流/index.md` | 传统开发 vs AI 开发流程、工具选择 | 中（真实内容但不完整） | 🟢 |
| 同目录 `assets/index/*.png`（2 张） | 流程图 | display asset 原样拷贝 | 🟢 |

### Vault 剔除项（37）

| 组 | 文件数 | 理由 |
|---|---|---|
| `20_daily/`（12） | 全部为空模板骨架（38–47 行模板，无正文） | 空模板 |
| `00_system/`（12） | `meta/` 4 个（naming/note_lifecycle/project/tags 规范）+ `templates/` 8 个 | PKM 系统规范与模板，非学习知识；公开前可复议是否作「方法论」页面（fog 外备注） |
| `10_inbox/`（3） | `面试准备.md`（空骨架）、`2026 · 成为父亲之前.md`、excalidraw 画板 | 私密 / 非知识 / 二进制画板 |
| `40_projects/whitebox_ciboard/`（3） | 工作项目重构文档 + AI 对话原始记录，涉公司内部系统 | 🟡 私密（工作敏感），v1 不进 |
| `60_mocs/`（3） | moc_database / moc_python / moc_system_design | 空文件 |
| `30_notes/ai/…/Claude_Code_功能介绍.md` | 仅标题骨架（36 行全是 heading，无正文） | 草稿 |
| `30_notes/ai/…/Claude_Code_实战演练.md` | 0 行 | 空 |
| `90_archived/tmp.md`（1）、`home.md`（1） | 空 / vault 仪表盘骨架 | 空模板，站点有自己的导航 |

## 公开切换前的复议清单（🟡 汇总）

1. `teach_ai/MISSION.md` — 年龄、在职状态、面试动机
2. `teach_ai` lessons 中以「可信看板」为载体的示例语境（B1 各课的项目背景段）
3. `30_notes/yazi/NOTES.md` — 本机环境细节
4. `40_projects/`（已剔除）与 `00_system/meta/`（已剔除，可复议转方法论页面）— 若转公开需整组重审
