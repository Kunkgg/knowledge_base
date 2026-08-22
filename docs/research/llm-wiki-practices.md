# [Research] LLM Wiki 增量编译的现有实践

- 调研对象: Karpathy「LLM Wiki」模式(ingest / query / lint 三操作,raw sources / wiki / schema 三层)在社区中的落地实践
- 调研日期: 2026-08-22(模式原帖 gist 于 2026-04 发布,社区实践约 4 个多月)
- 对应 issue: #3
- 标注约定: **[实证]** = 有可核查来源(README/实测数据/一手报告);**[推断]** = 基于实证的合理推断(主要是对本项目的建议)

## 0. TL;DR

1. 该模式已有一批可运行的开源实现,成熟度最高的三个:**obsidian-wiki**(39 个 agent skill 的技能包)、**Klore / llm-wiki-compiler**(全自动编译 CLI)、**Commonplace**(理论化的 agent-operated KB)。社区实测(3 本书 155K 词、68 个章节级源 → 210 个概念页、4597 条交叉引用、约 12M token)证明增量编译可行、成本可控(三档模型约 $0.06/源)。
2. 决定产出质量的第一变量是**源粒度**:整本书当一个源文件编译出来是平庸摘要,按章节切分后才会出现跨源综合 [实证,HN 一手实验]。
3. 对本项目(约 80 篇源起步、中文为主、Claude Code 会话驱动 + 人工验收、Obsidian 兼容、repo 子目录):**手写 CLAUDE.md schema + 轻量 manifest 增量 + `_staging/` 验收队列 + 分层 lint(每批结构 lint / 每月语义 lint 采样)+ query 回填 `reports/`**,不引入向量检索。80 源规模下纯 index.md 导航足够。

## 1. 原始模式(Karpathy gist)

来源: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

- **三层**: raw sources(不可变,LLM 只读)/ wiki(LLM 全权维护的 markdown 目录)/ schema(CLAUDE.md / AGENTS.md,约定页面结构、命名、工作流;人机共同演化)
- **三操作**:
  - ingest: 读源 → 与人讨论要点 → 写 summary 页 → 更新 index 与相关 entity/concept 页 → 追加 log。一个源可能牵动 10-15 个页面
  - query: 先读 index → 导航到相关页 → 综合回答并带引用;**好答案回填为新页面**(不消失在聊天记录里)
  - lint: 周期性扫描矛盾、陈旧声明、孤儿页、被提及但缺页的概念、缺失交叉引用、可用 web 搜索补的数据缺口
- **两个导航文件**: index.md(内容导向目录,~100 源规模够用,无需 embedding);log.md(append-only 时间线,统一前缀如 `## [2026-04-02] ingest | 标题` 使其可 grep)
- **定位**: wiki 是「一次编译、持续维护」的持久复利产物,区别于 RAG 每次查询重新检索、重新发现知识

## 2. 现有开源实现盘点

| 项目 | 形态 | 核心做法 | 备注 |
|---|---|---|---|
| [obsidian-wiki](https://github.com/Ar9av/obsidian-wiki)(Ar9av) | pip 安装的 39 个 agent skill(MIT) | manifest 增量、provenance 标签、staged writes、lint/dedup/cross-link、分层检索、archive & rebuild | 对 gist 的最完整工程化;有 benchmark |
| [Klore / llm-wiki-compiler](https://github.com/vbarsoum1/llm-wiki-compiler)(vbarsoum1) | 全自动 CLI(OpenRouter 三档模型) | 7 步流水线 Extract→Brief→Tag→Build→Review→Index/Log→Overview;`ask --save` 回填 reports/;git 自动 commit;SessionStart hook 注入 index | 用 Director 模型替代人工验收(与本项目路线相反,可对照) |
| [Commonplace](https://zby.github.io/commonplace/)(zby) | 开源 agent-operated KB | 类型阶梯(原文→结构化声明携带可靠性信号)、/connect /validate /ingest 技能;评测过 15 个 agent memory 系统 | 理论最深;「谁来决定记什么比存储格式更根本」 |
| [fabswill 实操](https://fabswill.com/blog/building-a-second-brain-that-compounds-karpathy-obsidian-claude/)(Fabian Williams) | 4 个 Claude Code skill | `/ingest-transcript`(frontmatter `compiled:false`)、`/wiki-compile` 幂等扫描未编译源、`/wiki-lint`、SQLite FTS5 检索脚本 | 5 源 → 21 文章 60+ 交叉链接,<2 分钟/源;最贴近本项目的自建路线 |
| [qmd](https://github.com/tobi/qmd)(tobi,~29k stars) | 本地混合搜索引擎(CLI + MCP + Claude Code 插件) | BM25(SQLite FTS5)+ 向量(embeddinggemma-300M)+ LLM 重排;分层 context 描述 | 规模化检索组件;**默认 embedding 英文优化,中文需换 Qwen3-Embedding 并全量重嵌入(README 明示)** |
| [llm-context-base](https://github.com/asakin/llm-context-base)(asakin) | git 仓库模板 | 30 天「训练期」建立元数据标准;stale lint | 轻量起步模板思路 |
| [hyalo](https://github.com/ractive/hyalo)(ractive) | CLI 工具 | 按 frontmatter/内容 grep、移动文件不断链、自动修链接 | Obsidian 兼容仓库的辅助工具 |
| claude-obsidian(skillsllm)、atomic(kenforthewin)、llmdoc(tristanmatthias)、llm-knowledge-base(arturseo-geo)、Graphify、Acontext | 单点工具 | 分别覆盖:聊天记录提取、RAG+wiki 综合、代码文档蒸馏、带版本 schema、代码知识图、会话技能持久化 | 来自 HN Show HN 线索,未逐一深读 |
| 中文实践教程: [知乎](https://zhuanlan.zhihu.com/p/2025834526673216988)、[acmerfight gist](https://gist.github.com/acmerfight/1c26b29ef39c0acc20f2e6f1f84e025f)、[CSDN](https://gitcode.csdn.net/69d386ed0a2f6a37c59d5d39.html) | 教程 | 均为「Claude Code + markdown 目录 + Obsidian 查看」的手工实现 | 中文社区以教程复刻为主,尚未见成熟中文向工具 |

## 3. 模式清单(按 issue 子议题)

### 3.1 ingest 分批与人工验收 [实证]

- **一次一篇、先讨论后写入**(Karpathy 原文):人始终在环,ingest 是对话而非批处理。
- **Inbox / staging 验收队列**:mindstudio 教程让新笔记先进 `Inbox/`,人过目后再归位,一周后把常见问题固化进 prompt;obsidian-wiki 提供 `WIKI_STAGED_WRITES=true` → 产物进 `_staging/` 审查后再落库。
- **起步小批量**:先 10-20 篇源调 schema,别第一天灌 200 篇 PDF(mindstudio「Processing Too Much Too Fast」)。
- **增量追踪两种实现**:
  - manifest 账本(obsidian-wiki):`.manifest.json` 记录每个源的 path / 时间戳 / 产出了哪些页,下次运行算 delta,只处理新增或变更;
  - frontmatter 标记(fabswill):源文件 `compiled: false`,`/wiki-compile` 幂等扫描未编译者。
- **源粒度是质量第一变量**(vbarsoum1 HN 一手实验):3 本 Hormozi 书 155K 词;naive 做法(1 书 = 1 源文件)输出是 slop;改为 68 个章节级源后,产出 210 个概念页、4597 条交叉引用(平均 19.2 链接/页)、20+ 概念被跨书综合、还真找到了一处跨书矛盾。
- **全自动替代人审**:Klore 用 Director 模型(默认 Gemini 3.0 Flash Preview)做编排审查,把人从循环中拿掉——可行但放弃了人工把关;HN 亦有「写笔记本身是学习,全自动会弱化个人编码」的反对意见。本项目的「Claude Code 会话 + 人工验收」恰是两者中间路线。

### 3.2 query 答案回填 [实证]

- Karpathy:值得留的查询答案(比较、分析、发现的联系)回填为 wiki 新页,让探索也复利。
- Klore:`klore ask --save` 把答案存入 `reports/`,并作为后续综合的输入——「编译后的 wiki 是人的接口,也是 AI 的知识层」。
- fabswill:`knowledge/output/` 归档查询结果。
- obsidian-wiki `/wiki-query`:分层检索——先读 titles/tags/summaries(frontmatter 里的 1-2 句 summary),便宜通道答不了再打开正文;说 "quick answer" 强制 index-only 模式;**查询成本从 20 页到 2000 页基本持平**;答案带 wikilink 引用可溯源。
- 反面教材(同 benchmark):普通 agent 会把 index.md 当地图走,因为 index 链接到一切,于是「找到」了无意义的短路径(38 页 vault 上正确率 44%,obsidian-wiki 83%)→ index 要一行一链一行摘要,且**图查询需排除 index/log 等簿记文件**。

### 3.3 lint 清单设计 [实证]

- Karpathy 原始六项:矛盾、陈旧声明(被新源取代)、孤儿页、被提及但无页的概念、缺失交叉引用、可 web 搜索补齐的数据缺口。
- obsidian-wiki 在其上扩展(最有参考价值的清单):
  - 结构类:孤儿页、断链 wikilink、缺必填 frontmatter、manifest 与实际不一致
  - 语义类:矛盾、陈旧、`wiki-dedup` 识别同一概念的不同命名页("RSC" vs "React Server Components")并合并
  - 结构健康:`_insights.md` 图分析——hub 页、bridge 页(删掉会切分图的节点)、死端、tag 聚类内聚度、图 delta
  - provenance 漂移:每条声明标 `extracted`(默认)/`^[inferred]`(LLM 综合)/`^[ambiguous]`(源之间打架);lint 对「大部分是推断」的页面告警
  - cross-linker:ingest 后扫未链接提及,自动织入图
  - tag 分类账 `_meta/taxonomy.md`:受控词表 + 全库 tag 归一
  - trust ledger:记录「人审过的版本指纹」,CI 可校验
- mindstudio 的治理规则(适合小库):概念页 <100 词且 30 天未更新 → 合并入相关页或删除;`glossary.md` 固定术语映射防止同概念建两页;每几周跑一次「合并/补链/立 MOC」综合 pass。
- Klore lint:矛盾 + 断链。
- **成本控制**(HN 讨论):矛盾检测是 O(N²) 两两比对,全量不可行 → 随机/受限子集采样,多次运行长期覆盖大部分矛盾。

### 3.4 index.md / log.md 在不同规模下的结构演化 [实证]

- **~100 源 / 几百页**:单 index.md(每页一行:链接 + 一句话摘要 + 元数据,按类分组)+ append-only log.md,无需任何 embedding 基础设施(Karpathy 明示此规模够用)。
- **数百页**:obsidian-wiki 增加 `hot.md`(~500 词的最近活动语义快照,每次写入更新,新会话直接续上不用爬全库)+ `_meta/taxonomy.md` + 分层检索(frontmatter summary 先行)。
- **全自动线**(Klore):`overview.md` 由 Director 维护的活综述;index + 2 篇文章 ≈330 行,对比直接塞原始源的 3200+ 行,**上下文省约 90%**;SessionStart hook 每次会话自动注入 index。
- **数千页**:换本地混合检索(qmd:BM25 + 向量 + LLM 重排,全本地)或 obsidian-wiki 式分层检索。
- **token 量化**(单人自述,Ketan Chavan substack):5 层上下文栈把每会话前置上下文从 20K token 降到 280(71.5x);CLAUDE.md 保持稳定可吃到 prompt cache(「稳定不变的 schema 是免费上下文」)。
- log.md 通用做法:统一前缀(`## [日期] ingest | 标题`)使其可 `grep`;Karpathy 同款。

### 3.5 git / 成本 / 上下文工程 [实证]

- wiki = 纯 markdown + git:版本历史、diff 审查、回滚免费;Klore 每次 compile 自动 commit;obsidian-wiki 支持 archive(时间戳快照)与 rebuild(漂移太远时整体重建,不丢任何东西)。
- 成本参考:
  - 全量编译 155K 词 / 68 源 ≈ **12M token、10-15 分钟**(vbarsoum1,三档便宜模型)
  - Klore:**~$0.06/源**;57K 词的书 ~$0.50;增量 compile **~$0.05-0.10/源**
  - mindstudio:60 分钟视频转录 ~$0.10-0.30;20 页 PDF <$0.10;个人月均 $5-20
  - lint 全量两两比对会吃掉可观配额(r/ObsidianMD 讨论;原帖被反爬拦截,依搜索摘要,置信度中)

## 4. 常见坑 [实证]

1. **上下文限制**:长上下文 ≠ 好推理;HN 实测即使 1M 窗模型,200-300K 后质量开始下降;注意力成本 O(n²)。解法:编译 + 分层导航,而非硬塞原文。
2. **页面漂移 / 二阶信息**:wiki 复读 wiki(摘要的摘要)→ 信息退化、模型坍缩风险(HN 争论)。obsidian-wiki 的解法:provenance 标签区分 extracted/inferred/ambiguous + archive & rebuild。
3. **重复页面**:别名页("RSC" vs "React Server Components");中文场景更常见——缩写 / 英文原名 / 中文译名三套叫法,需术语表 + 定期 dedup。
4. **index 污染图推理**:index 链接到一切 → agent 把它当路由表,产出无意义路径(benchmark 中 44% vs 83%)。解法:索引克制、图分析排除簿记文件。
5. **成本失控**:全量 lint、全量重编译都是 token 黑洞;增量 manifest + 采样 lint 是标配。
6. **「这不就是 RAG 吗」**:差异在**写循环 + lint pass**——RAG 语料静态、每次重发现;wiki 持续综合、矛盾预先标记、交叉引用预先建立。Commonplace 补充:真正的分野是「谁决定记什么」,不是存储格式。
7. **复杂度坍塌**:提取脚本 + 插件 + MCP + 图工具层层叠加,超出单人维护能力;HN 有「反向压缩」(扩写而不增加可提取结构)失败模式命名。起步越简单越好。
8. **学习价值争议**:全自动编译可能弱化个人对内容的编码(「writing is thinking」)。人审验收环节同时是学习环节,不是纯成本。

## 5. 对本项目的具体建议 [推断]

前提:约 80 篇源(12 个 HTML 课程 + 61 篇中文 Obsidian 笔记 + 学习记录);ingest 由 Claude Code 会话驱动、人工验收;wiki 放在本 repo 子目录且需 Obsidian 可直接打开。

### R1 目录与 schema(先做)

```
knowledge_base/
  CLAUDE.md            # schema: 页面类型/frontmatter 必填项/链接规则/lint 规则
  wiki/
    raw/               # 源,不可变(HTML 课程切分后入此)
    sources/           # 每源一页: 摘要+要点+回链 raw
    concepts/          # 概念/主题页(主要产物)
    entities/          # 工具/人物/库
    reports/           # query 回填的分析页
    index.md  log.md
    glossary.md        # 中英术语对照(防重复页的关键)
    _staging/          # 人工验收队列
    .manifest.json     # 增量账本: path/mtime/pages produced
```

- Obsidian 直接把 vault 指向 `wiki/`(或 repo 根),wikilink 语法天然兼容;不动 61 篇原始笔记,复制进 `raw/` 保持不可变。
- **HTML 课程先按章节切分成独立源文件再入库**(vbarsoum 粒度结论,对本项目 12 门课是最可能决定成败的一步);课程级再出 1 页「课程综述」。
- frontmatter 必填:`title / category / tags / sources / created / updated / summary(1-2 句)/ language`;每条非原文声明可后缀 `^[inferred]`。
- 页面语言跟源(中文为主),概念页标题用中文、正文首个链接给英文原名;`glossary.md` 固定「缩写=英文全称=中文译名」映射。

### R2 ingest 分批与人工验收

- 每次会话 1-5 篇;产物先进 `_staging/`,人在 Obsidian 过目后移入正式目录(或直接 `git diff` 审)。
- 增量:维持 `.manifest.json`(或 fabswill 式 `compiled:false` frontmatter,二选一,manifest 更适合源多的场景)。
- `log.md` append-only,统一前缀 `## [YYYY-MM-DD] ingest | 标题`。
- **每次 compile 结束自动 git commit**——commit 历史即审计线,漂移可回滚;攒到「漂移太远」时做一次 archive+rebuild(obsidian-wiki 模式)。

### R3 lint 节奏(两层)

- **每批次(便宜,自动)**:断链、孤儿页、缺 frontmatter、manifest 与文件系统不一致、index 覆盖率。
- **每月或每 ~10 次 ingest(语义,采样)**:矛盾检测(随机抽取页面对,N² 全量不可行)、dedup 别名页、stub 清理(<100 词且 30 天未更新 → 合并或删)、cross-linker 补缺失交叉引用、glossary 审计。
- 图分析(index 覆盖率、hub/孤儿)排除 `index.md / log.md / .manifest.json / _staging/`。

### R4 query 回填

- 值得留的答案写 `reports/YYYY-MM-DD-slug.md`,frontmatter 列 touched 概念页并从那些页回链;下次 ingest 前让 agent 读相关 reports(Klore 的 reports-as-input 模式)。
- 检索:80 源规模纯 index.md 导航即可,**不引入向量库**。将来若上 qmd 做规模化检索,注意其默认 embedding 英文优化,中文必须切 Qwen3-Embedding 并全量重嵌入。

### R5 规模演化路线(触发式,不预建)

- <200 页:现状(index.md + log.md)足够。
- >200 页:加 `hot.md` 语义快照 + frontmatter summary 分层检索。
- >500 页:考虑 qmd / obsidian-wiki 式 tiered retrieval,或直接 pip 装 obsidian-wiki 接管。
- 全程保持纯 markdown + git,不锁进数据库(需要时可导出 graph.json / OKF)。

### R6 自建 vs 用现成工具

- 建议先**手写 schema 起步**(量小、中文、要人工验收;现成工具多为英文假设且自带复杂度坍塌风险),但**照抄 obsidian-wiki 的设计**:manifest 结构、staged writes、lint 清单、provenance 标签、dedup、taxonomy。
- Klore 可作为「全自动对照组」参考其 reports 回填与 overview.md 设计;不建议本项目引入 Director 全自动(放弃人工验收与学习收益)。

## 6. 来源

已核实(一手阅读,2026-08-22):

1. Karpathy 原始 gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
2. HN 主讨论(含 vbarsoum1 155K 词实验数据): https://news.ycombinator.com/item?id=47640875
3. obsidian-wiki(README + docs/architecture.md): https://github.com/Ar9av/obsidian-wiki
4. Klore / llm-wiki-compiler README: https://github.com/vbarsoum1/llm-wiki-compiler
5. qmd(overview,含 CJK 限制说明): https://github.com/tobi/qmd
6. fabswill 实操文章: https://fabswill.com/blog/building-a-second-brain-that-compounds-karpathy-obsidian-claude/
7. mindstudio 教程(Inbox 验收 / stub 规则 / 成本 / FAQ): https://www.mindstudio.ai/blog/how-to-build-llm-wiki-knowledge-base-obsidian-claude-code
8. Commonplace: https://zby.github.io/commonplace/
9. llm-context-base: https://github.com/asakin/llm-context-base
10. hyalo: https://github.com/ractive/hyalo
11. Ketan Chavan, 71.5x token 实测(单人自述数据): https://ketanchavan.substack.com/p/715x-fewer-tokens-how-karpathys-llm
12. 知乎实践教程(中文): https://zhuanlan.zhihu.com/p/2025834526673216988
13. Roan Monteiro, LLM Wiki + 开发者第二大脑: https://medium.com/@roanmonteiro/building-a-complete-personal-harness-llm-wiki-developers-second-brain-in-obsidian-d7b61c7398ff (部分预览)
14. HN 相关讨论: https://news.ycombinator.com/item?id=47899844 , 48586811, 47654076

未深读/受限(仅作线索):
- r/ObsidianMD 两帖(1uai1w2 使用反馈、1sx040s 成本质疑)——原帖反爬拦截,内容依搜索摘要
- particula.tech / atlan.com / kunalganglani.com 的对比分析文章
