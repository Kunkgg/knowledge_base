# 静态站点生成器选型研究

- **Issue**: [#2 [Research] 静态站点生成器选型](https://github.com/Kunkgg/knowledge_base/issues/2)
- **日期**: 2026-08-22
- **方法**: 关键事实均经一手来源核实(官方文档、GitHub 仓库源码/tag/issue/PR、npm registry、Pagefind 官方文档);仓库活跃度数据经 GitHub API 于 2026-08-22 抓取。未能核实的点集中在「未核实事项」一节,明确标出。

## 1. 需求回顾(选型权重)

来自 issue #2 与 issue #1(wayfinder 地图)的既定约束:

1. **wiki 层必须是 Obsidian 兼容 markdown**(用户本地直接用 Obsidian 浏览)→ 生成器必须原生吃下 Obsidian 语法,或在构建期做转换(转换层 = 额外维护成本与漂移风险)。
2. **中文正文 + 英文术语** → 全文搜索的中文分词是硬考察点。
3. **纯静态输出**,阿里云 ECS nginx 直接托管;先本地跑通。
4. **v1 底线:阅读 + 导航**;搜索若白送则要;**graph 视图只留门**。
5. 后续拓展:graph 视图、数据结构与算法可视化交互页。
6. wiki 位于 repo 的 `wiki/` 子目录,**站点项目不在内容根**。

## 2. 候选版本快照(2026-08-22,GitHub API)

| 项目 | 当前版本 | Stars | 最近 push | 维护方 |
|---|---|---|---|---|
| Quartz | v4.5.2(2025-09-21)/ **v5.0.0(2026-03-14,现主线**,默认分支 `v5`) | 13,063 | 2026-08-18 | jackyzha0(个人)+ 社区 |
| Astro + Starlight | Astro 7.x;@astrojs/starlight 0.41.7(2026-08-05) | 61,930 / 9,110 | 2026-08-22(当日) | Astro 团队(公司背景) |
| VitePress | 稳定 1.6.4;2.0.0-alpha.19 开发中 | 18,220 | 2026-08-22(当日) | Vite/Vue 团队(Evan You) |
| Eleventy | 3.1.6(2026-06-02);官方文档站已运行 v4.0.0 开发版 | 19,856 | 2026-08-19 | Zach Leatherman + 社区 |

> 注意:Quartz 的 GitHub Releases 页不可靠(latest 显示 2023 年的 v4.0.8),以 git tag 为准。Quartz 已于 2026-03 发布 v5 大版本:配置从 `quartz.config.ts` 改为 YAML(`quartz.config.default.yaml`),插件外部化为 npm 包(`@quartz-community/*`);v4 文档在 quartz.jzhao.net,v5 文档在 quartz.jzhao.xyz。

## 3. 对比矩阵

| 维度 | Quartz 4.5.2 / v5 | Astro Starlight | VitePress | Eleventy 3 |
|---|---|---|---|---|
| **Obsidian 语法** | **原生**(ObsidianFlavoredMarkdown 默认开启) | 非原生 | 非原生 | 非原生 |
| wikilink `[[...]]` | 全语法:别名、锚点、`^block-ref`、`![[embed]]` | 需社区 remark 插件(remark-wiki-link,2023-10 后未更新) | 需 markdown-it 插件(维护状况未核实) | 全 DIY |
| callout `> [!note]` | 原生 | `:::note` 自有语法,需机械转换 | `::: info` 容器语法,需转换 | DIY |
| frontmatter | 原生解析 Obsidian 字段 | 每页必须 `title`;Zod schema 校验,自定义字段需 `extend` | 支持 | data cascade,最宽松 |
| 中文内容渲染 | 可用;zh UI 翻译已有(#896);中文锚点历史 bug 已修(#409) | 可用;UI 内置 zh-CN 翻译 | 可用;官方文档本身有简体中文版 | 可用 |
| **全文搜索** | 内置 FlexSearch;**中文在 4.5.2 有实证缺陷(#2252)**,CJK 分词修复 PR(#2231)已并入 v5;v5 搜索改为外部插件(0.1.0,2026-07) | **内置 Pagefind(默认开启),官方文档确认中文分词支持** | 内置本地搜索 + Algolia DocSearch;**中文可用性未核实** | 无内置;标准方案 Pagefind |
| 纯静态输出 | `public/`,SPA 式路由但纯静态 | `dist/`(默认全预渲染) | `.vitepress/dist/` | `_site/` |
| 本地预览 | `npx quartz build --serve` | `astro dev` / `astro preview` | `vitepress dev` | `--serve`(watch + dev server) |
| **wiki/ 子目录构建** | `-d/--directory` 指定内容目录(v4 文档);社区常用 symlink `content -> ../wiki`;v5 待核实 | `docsLoader()` 固定 `src/content/docs/`;Astro 层 `glob()` loader 可指向文件系统任意路径 + `processedDirs`,但需自行接线 | `srcDir` 已文档化;指向项目根之外未核实 | **`--input` 一等公民,任意路径** |
| **graph 视图门** | **内置**(局部 + 全局图,d3 力导向,可配置) | 需自研组件(可行) | 需自研组件(可行) | DIY |
| 交互页(算法可视化)门 | 可写自定义组件,生态小 | **最强**(React/Vue/Svelte/Solid 岛屿) | 强(Vue 组件直接写进 markdown) | WebC / 任意技术,DIY |
| 维护活跃度 | 活跃,但单人维护 + 正处大版本切换期 | 非常活跃,组织背书 | 非常活跃,组织背书 | 活跃 |
| 定位 | 数字花园 / Obsidian 发布 | 文档站 | 文档站 | 通用 SSG,一切自组装 |

## 4. 关键事实与来源

### 4.1 Obsidian markdown 兼容

**Quartz**——四者中唯一把 Obsidian 当一等公民:

- 官方文档原话:"Quartz was originally designed as a tool to publish Obsidian vaults as websites … ships with the ObsidianFlavoredMarkdown plugin" + "support for frontmatter parsing with the same fields that Obsidian uses through the Frontmatter transformer plugin"(v4.5.2 `docs/features/Obsidian compatibility.md`)。
- wikilink 全语法支持:`[[Path]]`、`[[Path | 别名]]`、`[[Path#Anchor]]`、`[[Path#^block-ref]]`,以及全部 embed 形式 `![[image]]`、`![[file]]`、`![[file#Anchor]]`、`![[file#^block]]`(v4.5.2 `docs/features/wikilinks.md`)。
- callout 为原生特性(有专门文档页 `docs/features/callouts.md`;Quartz 官方文档自身大量使用 `> [!hint]`、`> [!note]`)。
- v5 默认配置的 `ignorePatterns` 直接含 `.obsidian`,佐证其定位仍是「发布 Obsidian vault」(v5 `quartz.config.default.yaml`)。
- 中文相关历史 bug:中文锚点 wikilink 无法识别(#409,已关闭 2024-07);简体中文 UI 翻译已合入(#896,2024-02)。

**Starlight**——语法不同构,存在转换成本:

- aside(等价物)用三冒号语法 `:::note` / `:::tip` / `:::caution` / `:::danger`,支持 `:::tip[自定义标题]`(**不是** Obsidian 的 `> [!note]`)(官方 authoring-content 指南)。
- frontmatter:「Every page must include at least a `title`」;schema 校验通过 `docsSchema()` + `extend` 扩展自定义字段(官方 Configuration Reference)→ LLM 生成的 Obsidian 风格 frontmatter(tags/aliases 等)需要扩展 schema 或预处理。
- wikilink 非原生:主流社区插件 remark-wiki-link 最新版 2.0.1 发布于 **2023-10-10**(npm registry),近三年未更新。

**VitePress**——同为文档站语法族:自定义容器 `::: info`(markdown-it 体系),与 Obsidian callout 不同构;wikilink 非原生,依赖 markdown-it 社区插件(本调研未核实其维护状况)。frontmatter 按页支持。

**Eleventy**——无任何 Obsidian 概念,wikilink/callout 全部自建(markdown-it 可配插件或自写),frontmatter 走 data cascade(对未知字段最宽容)。

### 4.2 全文搜索与中文(本次选型的决胜点)

**Starlight(以及 Eleventy 的标准方案)= Pagefind,中文支持为一手确认:**

- Starlight 默认启用 Pagefind:`pagefind` 配置项 default `true`,含语言等选项(官方 Configuration Reference;且注明 `prerender: false` 时不可用,印证纯静态定位)。
- Pagefind 官方多语言文档:按 `<html lang>` 自动分语言建索引;中文属「有 UI 翻译、无词干化」语言,但**支持无空格语言的分词**(segmentation),默认的 `npx pagefind` 即含此扩展版。原文示例:"on a page tagged as a `zh-` language, `每個月都` will be indexed as the words `每個`, `月`, and `都`";引号查询可精确匹配整串。
- 结论:**Pagefind 的中文搜索是四条路线中唯一被官方文档明确背书的**。

**Quartz = FlexSearch,中文是实证痛点,但有明确的修复轨迹:**

- v4.5.2 搜索组件源码(`quartz/components/scripts/search.inline.ts` @ tag v4.5.2):自定 encoder 仅按空白切分(`str.toLowerCase().split(/\s+/)`)+ FlexSearch `tokenize: "forward"`。中文整段无空格文本会变成超长 token,非前缀子串无法命中。
- 官方文档虽声称 "It properly tokenizes Chinese, Korean, and Japanese characters"(v4.5.2 `docs/features/full-text search.md`),但用户实测打脸:issue #2252(2025-11,Quartz 4.5.2)报告「人、无人机、飞」等单字/词搜索不到。
- 修复轨迹:#163(2022,首次报告 CJK 搜索失效)→ PR #2108「Update FlexSearch and Add Support for All Languages」(merged 2025-09-17,已含入 4.5.2)→ **PR #2231「improve search tokenization for CJK languages」(merged 2025-12-02,晚于 4.5.2 tag,应落入 v5)**→ #2252 于 2025-12-19 关闭。
- v5 的搜索已外部化为独立插件 `quartz-community/search`(GitHub / npm `@quartz-community/search` **0.1.0,2026-07-22 发布,极早期**)。
- 结论:v4.5.2 的中文搜索**不可依赖**;v5 大概率已含分词修复,但**没有实测证据**。

**VitePress**——内置本地搜索(MiniSearch)+ Algolia DocSearch 两选项;其官方搜索文档页在本轮调研中 404,**中文分词可用性未取得一手证据**,列为未核实。

**Eleventy**——无内置搜索(官方 CLI/文档无此特性),事实标准方案是构建后跑 Pagefind(同 Starlight 的已验证路线)。

### 4.3 纯静态输出与本地预览

四者全部输出纯静态 HTML,nginx 可直接托管,无服务端依赖:

- Quartz:`npx quartz build --serve` 本地热预览(官方明言 serve 仅限本地,生产见 hosting 文档);输出 `public/`(官方 build.md:`-o/--output`,默认 `public`)。
- Starlight/Astro:`astro build` → `dist/`;Starlight `prerender` 默认 `true`。
- VitePress:`outDir` 默认 `./.vitepress/dist`;另有实验性 `mpa` 模式可 0kb JS。
- Eleventy:`--output` 默认 `_site`;`--serve` 带 watch。

### 4.4 从 `wiki/` 子目录构建(内容不在站点项目根)

- **Quartz**:v4 官方 build 文档明确 `-d` / `--directory`:「the content folder. This is normally just `content`」→ 可直接 `npx quartz build -d ../wiki`;社区亦常用 symlink。v5 是否保留该 flag 未核实(见第 7 节)。
- **Starlight**:默认 `docsLoader()` 固定从 `src/content/docs/` 读取(官方 Configuration Reference);但 Astro 层的 `glob()` loader 官方原话「creates entries from directories of files from **anywhere on the filesystem**」,`base` 可指向任意目录;Starlight 另有 `markdown.processedDirs` 让额外目录享受其 markdown 管线。即:**可行,但属于自行接线的非常规路径**(自定义 collection + glob base 指向 `../wiki` + docsSchema),非开箱即用。
- **VitePress**:`srcDir` 官方文档化(「The directory where your markdown pages are stored, relative to project root」),指向项目根之外(`../wiki`)社区常见但**官方支持口径未核实**。
- **Eleventy**:`--input` 是一等公民 CLI 参数(官方 usage 文档:`npx @11ty/eleventy --input=. --output=_site`),任意路径无假设,**最自由**。

### 4.5 可扩展性(graph 门 / 交互页门)

- **Quartz**:graph view **内置**(局部图一跳邻域 + 全局图,d3 力导向参数可配:repelForce/linkDistance/depth 等;官方 `docs/features/graph view.md`)——issue #1 说「graph 一律留门」,Quartz 等于**门已经装好,只是不开也行**。另有 backlinks、popover 预览、explorer 等 wiki 导航件。自定义交互组件可写(preact/TS 组件体系),但生态最小。
- **Starlight/Astro**:岛屿架构,React/Vue/Svelte/Solid/Alpine 任选,做算法可视化交互页**最强**;graph 视图需自研(用 d3-force / sigma.js 包一个组件即可);另有组件覆盖与插件 API 两条官方扩展通道。
- **VitePress**:Vue 组件可直接写进 markdown,交互页很顺手;graph 同样需自研。
- **Eleventy**:任意模板语言 + WebC,一切皆可,一切皆需自建。

### 4.6 维护活跃度与成熟度

- 四个项目 2026-08 均活跃(见第 2 节 push 日期),无弃坑迹象。
- 结构性差异:Starlight/VitePress 有组织与全职团队背书;Quartz 是**单人主导**(jackyzha0)+ 社区,且正处于 v4→v5 大版本切换期(插件生态刚外部化,`@quartz-community/search` 一个月大);Eleventy 单维护者 + 社区,当前发布节奏健康。
- 另:Quartz **不是 npm 包**,使用方式是 clone/fork 模板仓库(社区对此有明确抱怨,#403)→ vendoring 结构需要设计(建议站点代码独立于 `wiki/`,见风险 4)。

## 5. 推荐

**主选:Quartz v5。备选/演进路线:Astro Starlight(仅当交互可视化成为主线时切换)。**

理由:本项目的不可妥协约束是「wiki 层保持 Obsidian 纯净、本地用 Obsidian 直接浏览」,这要求站点生成器原生消化 Obsidian 语法——Quartz 是四者中唯一做到的(wikilink 全语法、`> [!callout]`、Obsidian frontmatter、backlinks/explorer/popover 一整套 wiki 导航),而且 v1 不打算做的 graph view 它已经内置(「留门」变「送门」,不开即可);输出纯静态 `public/`,nginx 直接托管,本地 `--serve` 热预览,`-d` 指向 `../wiki` 即可从子目录构建。唯一实质风险是中文搜索:v4.5.2 被 #2252 实证有缺陷,v5 已并入 CJK 分词修复(#2231)但无实测,且搜索插件刚外部化(0.1.0)。该风险有双层缓解——v5 大概率已修;即便不行,**Pagefind 可以后处理任何静态 HTML 输出**(与 Starlight 同款、中文分词已被官方文档背书),无需更换框架。因此「Quartz + 搜索不行就挂 Pagefind 后处理」的组合,同时拿到 Obsidian 兼容、graph 门和中文搜索兜底。

Starlight 工程质量、搜索、交互页能力最强,但吃 Obsidian 语法需要转换层或依赖三年未更新的 remark-wiki-link,与「wiki 保持 Obsidian 纯净」直接冲突,故定位为演进路线而非 v1。VitePress 与 Eleventy:前者是文档站骨架(侧边栏导航模型)且中文搜索未核实,后者一切自组装,均与 wiki 形态错配,排除。

**进入 prototype 票的验收测试**:用真实中文笔记在 Quartz v5 跑搜索冒烟测试(单字、二字词、任意位置词组、引号精确匹配、英文术语);不过关则在 build 后接 Pagefind(`npx pagefind --site public`)再测一轮。

## 6. 风险与缓解

1. **Quartz v5 切换期动荡**(2026-03 发布,配置体系换 YAML,插件刚外部化且版本号 0.1.0)→ 起步即锁 tag;升级走官方 update 流程;不依赖新生态插件。v4.5.2 是退路但中文搜索已知有缺陷。
2. **中文搜索质量未实测**(文档声称与用户报告矛盾的历史前科)→ prototype 冒烟测试为硬性验收项;Pagefind 后处理作为不换框架的兜底。
3. **单人维护者 bus factor**(对比 Astro/Vite 团队)→ 缓解:内容本身是纯 markdown,wiki 与站点解耦后,迁移成本 = 换一个渲染器(Starlight + 转换层是现成退路)。
4. **Quartz 非 npm 包、模板即仓库**(#403)→ 结构决策(留给后续票):建议 `site/`(Quartz 副本)与 `wiki/`(纯内容)严格分离,`wiki/` 不掺任何站点配置。
5. **若走 Starlight 路线**的已知坑:remark-wiki-link 停更(2023-10);frontmatter schema 需 `extend`;内容需迁入 `src/content/docs/` 或走 glob loader 自行接线。

## 7. 未核实事项(诚实清单)

- VitePress 本地搜索的中文分词:文档页在调研中返回 404,未取得一手证据。
- VitePress `srcDir` 指向项目根之外(如 `../wiki`)的官方支持口径:文档仅说「relative to project root」。
- Quartz v5 是否保留 `-d/--directory` flag(v4 文档有;v5 默认配置与 CLI 未逐一核验)。
- Quartz callout 支持的具体类型清单(由 ObsidianFlavoredMarkdown 覆盖,判定为支持)。
- `@quartz-community/search` 0.1.0 的 CJK 实际效果(需 prototype 实测)。
- Starlight `docsSchema()` 对未知 Obsidian frontmatter 字段是报错还是忽略(只核实了 `extend` 机制存在)。
- npm 周下载量未采集(以 stars + push 活跃度代替)。

## 8. 来源

**Quartz**
- 仓库与 tag(GitHub API,2026-08-22):https://github.com/jackyzha0/quartz —— v4.5.2(2025-09-21)、v5.0.0(2026-03-14),stars 13,063,pushed 2026-08-18
- v4.5.2 仓库内文档:`docs/features/Obsidian compatibility.md`、`docs/features/wikilinks.md`、`docs/features/full-text search.md`、`docs/features/graph view.md`、`docs/build.md`、`docs/configuration.md`(经 GitHub contents API 读取)
- 搜索源码:`quartz/components/scripts/search.inline.ts` @ v4.5.2(空白切分 encoder + FlexSearch forward tokenize)
- v5 搜索插件文档与配置:`docs/plugins/Search.md`、`quartz.config.default.yaml` @ 分支 `v5`;https://github.com/quartz-community/search ;npm `@quartz-community/search` 0.1.0(2026-07-22)
- Issues/PR:#163(CJK 搜索失效,2022)、#2108(merged 2025-09-17)、#2231(merged 2025-12-02)、#2252(2025-11,4.5.2 中文搜索缺陷,2025-12-19 关闭)、#409(中文锚点)、#896(简中 i18n)、#403(非 npm 包需 fork)
- v4 文档站:https://quartz.jzhao.net ;v5 文档站:https://quartz.jzhao.xyz

**Astro / Starlight**
- https://starlight.astro.build/guides/authoring-content/ (aside `:::note` 语法、title 必填)
- https://starlight.astro.build/reference/configuration/ (pagefind 默认 true、docsLoader 固定 src/content/docs、processedDirs、plugins/components 扩展、zh-CN locale)
- https://docs.astro.build/en/reference/content-loader-reference/ (glob() loader「anywhere on the filesystem」、base 选项)
- 仓库数据(GitHub API):withastro/astro(61,930 stars)、withastro/starlight(@astrojs/starlight 0.41.7,2026-08-05)
- remark-wiki-link:npm registry(latest 2.0.1,2023-10-10)

**VitePress**
- https://vitepress.dev/reference/site-config (srcDir/outDir/mpa;站点版本标识 1.6.4 / 2.0.0-alpha.19)
- 仓库数据(GitHub API):vuejs/vitepress(18,220 stars,pushed 2026-08-22)

**Eleventy**
- https://www.11ty.dev/docs/usage/ (--input/--output/--serve/--incremental;文档站 generator 标识 v4.0.0)
- 仓库数据(GitHub API):11ty/eleventy(v3.1.6,2026-06-02;19,856 stars)

**Pagefind**
- https://pagefind.app/docs/multilingual/ (zh 分词与引号精确匹配、extended release 默认包含)
