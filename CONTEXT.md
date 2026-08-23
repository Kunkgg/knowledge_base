# Knowledge Base

个人知识站点的三层系统:不可变的源、LLM 维护的 Wiki、静态站点。本文件是全项目唯一术语表(纯词汇,不含实现细节)。

## Language

### 三层与三操作

**源**:
不可变的原始层——`teach_ai`、`Obsidian-Vault` 等知识原文;v1 以快照形式冻结在 `wiki/raw/`。
_Avoid_: 语料、原始数据

**Wiki**:
LLM 全权维护、Obsidian 兼容的 markdown 层(`wiki/`);一次编译、持续维护的复利产物。
_Avoid_: 知识库(与仓库名混淆)、第二大脑

**站点**:
Wiki 的渲染展示层,Quartz 生成纯静态 HTML,自托管于阿里云 ECS。

**ingest**:
读源、与人讨论要点、写页面并更新簿记件的编译操作;一个源可牵动多页。
_Avoid_: 导入、索引(与 index 混淆)

**query**:
基于 Wiki 导航回答问题、好答案回填为报告页的操作。
_Avoid_: 检索、RAG

**lint**:
周期性扫描矛盾、陈旧、孤儿页、断链、重复页的保健操作。

### 页面类型

**概念页**:
承载抽象概念或主题的页面(`concepts/`);判据:不可下载,只能被理解。
_Avoid_: 词条页、主题页

**实体页**:
承载可命名的工具/库/框架/人物的页面(`entities/`);判据:能下载、引用、关注其 release。

**课程页**:
一门课(≥3 讲)的综述页(`courses/`);讲**课教了什么**。
_Avoid_: 目录页、大纲页

**学习轨迹页**:
一个学习域的学习历程叙事页(`trails/`),吸收 MISSION/RESOURCES/learning-records;讲**我怎么学的**。
_Avoid_: 学习记录页(记录是源,轨迹页是综合)

**报告页**:
query 会话中值得留存的答案回填页(`reports/`)。

### 簿记件

**index**:
Wiki 的导航目录(`index.md`),一行一链一行摘要;agent 的地图,克制的索引。

**log**:
append-only 时间线(`log.md`),统一前缀可 grep。

**glossary**:
中英术语受控词表(`glossary.md`);dedup 的依据。

**manifest**:
增量账本(`.manifest.json`),记每源的路径映射、批次、hash 与产出页。
_Avoid_: 索引(与 index 混淆)

**staging**:
人工验收队列(`_staging/`),一切产物归位前的中转。
