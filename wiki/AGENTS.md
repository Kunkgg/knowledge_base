# Wiki Schema

Wiki 层(`wiki/`)的信息架构约定:页面类型、目录、命名、frontmatter、链接与簿记件。术语定义见根 `CONTEXT.md`;ingest/query/lint 的运行细则见「Ingest 运行手册」。

决议来源:[Wayfinder #5 · Wiki 信息架构与 Schema 约定](https://github.com/Kunkgg/knowledge_base/issues/5) · [#6 · Ingest 工作流](https://github.com/Kunkgg/knowledge_base/issues/6)。

## 铁律

1. **源不可变**:`raw/` **入库后只读**——此后任何操作不改写、不删除、不移动 raw/ 文件;唯一合法的写时刻是入库动作本身(拷贝快照、切分产物首写),由 ingest 会话执行。
2. **先验收后归位**:一切新页/改页先进 `_staging/`,人工过目后移入正式目录(query 回填的 reports 页除外,见运行手册)。
3. **中文正文 + 英文术语**:正文行文中文,术语保留英文原文、不硬译;文件名不受此条约束(见「命名约定」,主名从众)。

## 目录结构

```
wiki/
  AGENTS.md          # 本文件(schema)
  raw/               # 源快照,不可变,按源仓库镜像
    teach-ai/           # lessons/ · reference/ · learning-records/ · MISSION · RESOURCES
    obsidian-vault/     # 30_notes/yazi/… · 30_notes/ai/极客时间_AI原生工作流/…
    collected-html/     # 第三源留门(v1 为空,文件到位后启用)
  concepts/          # 概念页
  entities/          # 实体页
  courses/           # 课程页
  trails/            # 学习轨迹页
  reports/           # 报告页
  assets/            # 展示资产(png 等),embed 引用,不作知识源 ingest
  index.md           # 簿记:导航目录
  log.md             # 簿记:append-only 时间线
  glossary.md        # 簿记:中英术语受控词表
  _staging/          # 人工验收队列
  .manifest.json     # 增量账本
```

- `raw/` 子目录按**源仓库**分组,镜像仓库内相对路径原样;批次(B1–B4)只是 manifest 的字段,不刻进目录。第三源到位即新增子目录,无代码假设 54 源。
- HTML 源的章节/词条切分产物也落 `raw/`(同源目录下);切分格式与规则归源→Wiki 映射策略决议。
- **排除约定**:
  - 图分析与导航统计(Obsidian graph 过滤、lint 图规则):排除 `raw/`、`_staging/`、三簿记件、`.manifest.json`;
  - 站点构建(Quartz):排除 `raw/`、`_staging/`、`.manifest.json`(`assets/` 随站点发布)。

## 页面类型学

| type | 目录 | 触发者 | 建页阈值 |
|---|---|---|---|
| `concept` 概念页 | `concepts/` | ingest · query · lint | 抽象概念/主题:在 ≥2 个源出现,或单源以整节展开 |
| `entity` 实体页 | `entities/` | ingest | 工具/库/框架/人物:可下载、可引用、可关注其 release |
| `course` 课程页 | `courses/` | 该课批次 ingest 收尾时 | 一门课 ≥3 讲的综述(v1:AI 工程十二讲、yazi 九讲);单篇源不建 |
| `trail` 学习轨迹页 | `trails/` | ingest(叙事性源) | 每个学习域 1 页,吸收 MISSION/RESOURCES/learning-records 叙事 |
| `report` 报告页 | `reports/` | query 会话回填 | 值得留的分析/比较/联系,文件名 `YYYY-MM-DD-slug.md` |

- **概念 vs 实体**:能「下载、引用、关注 release」的是 entity,其余是 concept(RAG、HITL、superstep 是概念;yazi、LangGraph、Chroma 是实体)。
- **course vs trail**:course 讲**课教了什么**;trail 讲**我怎么学的**(时间线叙事)。trail 中的非显然结论**拆进**概念/实体页,叙事页不留孤立知识点;记录原文留 raw/ 可回链。
- **不设的页型**:
  - MOC——index 分组 + course 页已承担导航;规模化阈值到再立(见地图雾区「wiki 规模化导航策略」);
  - 每源一页的源卡片——概念/实体页 frontmatter `sources:` 直链 `raw/` 文件,manifest 已记「源→产出页」账;
  - learning-records 逐份建页——聚进 trail,避免 22 个碎片 stub。

## frontmatter(所有正式页面必填)

```yaml
type: concept            # concept | entity | course | trail | report
title: RAG               # 默认与文件名一致,Quartz 显示标题
aliases: [Retrieval-Augmented Generation]   # 英文全称/缩写,dedup 依据
tags: []
summary: ""              # 1–2 句;分层检索与 index 摘要的来源
sources: ["[[raw/teach-ai/lessons/0004-rag-mvp.html]]"]
created: 2026-08-23
updated: 2026-08-23
privacy: yellow          # 仅 🟡 页标 yellow;🟢 缺省不写
```

- 相对研究骨架(R1)的裁剪:无 `language`(全库统一中文正文+英文术语,单页标注无信息量)、无 `category`(被 `type` 取代)。
- `privacy` 对齐源清单 manifest 的私密度标记:🟡 个人语境(自托管 OK,公开切换前逐篇复议),🔴 绝不进 wiki(`teach_ai/.env` 类,永不落盘)。
- provenance 不进 frontmatter,行内标记:推断/综合 `^[inferred]`,源间冲突 `^[ambiguous]`(extracted 缺省不标)。

## 命名约定

- **主名从众**:文件名 = 输入 `[[` 时的第一直觉。圈内通用缩写直接用缩写(`RAG.md`、`HITL.md`);通用中文词用中文(`检查点.md`)。
- entity 用英文原名(`LangGraph.md`、`yazi.md`);course/trail 用中文名;report 用 `YYYY-MM-DD-slug.md`。
- 中文文件名经 Quartz 生成 percent-encoded URL,自托管接受(选型已定,优先照顾 Obsidian 主界面)。

## 链接与正文

- 一律 wikilink `[[…]]`,不用 markdown link(Quartz 原生解析)。
- 正文首次提及重要概念/实体时链接对应页;**允许暂时断链**,由 lint 报「被提及但缺页」后补建。
- embed `![[…]]` 仅用于 `assets/` 资产与正文片段引用。

## 簿记件

- **index.md**:agent 的地图,按类型分组(课程 → 轨迹 → 概念 → 实体 → 报告),一行一链一行摘要。克制是特性——index 链接到一切会污染图推理:

  ```markdown
  ## 课程
  - [[AI 工程十二讲]] — 从 API 调用到 LangGraph HITL 的完整主线
  ```

- **log.md**:append-only,统一前缀可 grep。op ∈ `ingest | query | lint | schema`(schema 记 schema 本身的修订,含本文件):

  ```markdown
  ## [2026-08-23] ingest | B1 teach-ai lessons
  ```

- **glossary.md**:受控词表,ingest 新增术语必更,dedup lint 扫它防「缩写/原名/译名」三套叫法建三个页:

  ```markdown
  | 中文 | English | 缩写 | 页 |
  |---|---|---|---|
  | 检索增强生成 | Retrieval-Augmented Generation | RAG | [[RAG]] |
  ```

## .manifest.json(骨架)

```json
{
  "sources": [
    {
      "origin_path": "~/Repos/teach-ai/lessons/0004-rag-mvp.html",
      "raw_path": "wiki/raw/teach-ai/lessons/0004-rag-mvp.html",
      "batch": "B1",
      "hash": "…",
      "ingested_at": "2026-08-23",
      "pages": ["concepts/RAG.md", "courses/AI 工程十二讲.md"]
    }
  ]
}
```

增量判定与验收流转见「Ingest 运行手册」。

## Ingest 运行手册

启动:人说「ingest <批次|文件>」→ 读 `.manifest.json` 算 delta(未 ingest / hash 变更的源)。

1. **逐源处理**:源快照拷入 `raw/` 同源目录(🔴 拒绝落盘;🟡 记 `privacy: yellow`)→ HTML 按源→Wiki 映射策略切分、产物落同源目录 → 编译页面草稿**全部进 `_staging/`**(含 index/log/glossary 簿记草稿)。
2. **调 schema 期**(B1 前 3 源):每源完成即停,人过目纠偏后继续。
3. **批尾验收**:人过目 `_staging/` + `git diff` → OK 后 agent 归位全部文件、回写 manifest(`ingested_at` / `pages`)、追加 log → 一次 commit `wiki: ingest <批> · <n> sources`;驳回则 agent 改后复审。**manifest 只在归位时回写**——commit 了才算 ingest 完成,delta 语义干净。
4. **结构 lint(层 1,会话收尾)**,清单:
   - 断链 wikilink(被提及但缺页)
   - 孤儿页(无入链)
   - 缺必填 frontmatter
   - manifest ↔ 文件系统不一致
   - index 覆盖率(正式页是否每页一行)

   小问题当场修,大问题记票。

### query 回填

综合答案值得留(比较/分析/发现的联系)→ **提议** → 人点头 → 写 `reports/YYYY-MM-DD-slug.md`(**免 staging**;frontmatter `sources:` 回链触及的概念页)+ log 记 `query`。

### 语义 lint(层 2,独立会话)

触发:每月 或 每积累 ~10 次 ingest,先到先跑。清单:

- 矛盾检测:随机采样页面对(O(N²) 不全量)
- dedup 别名页(缩写/原名/译名三套叫法)
- stub 清理:<100 词且 30 天未更新 → 合并或删
- cross-link 补缺(未链接的提及)
- glossary 审计

改动照走 `_staging/` 验收。

### 脚本化留门

一切状态显式化于 manifest(账本)+ log(时间线)+ git(审计),无会话内存活的隐藏状态——本手册即未来脚本的规格底稿;脚本触发与无人值守边界待会话驱动稳定后再议(地图雾区)。
