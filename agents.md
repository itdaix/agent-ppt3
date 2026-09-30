# AGENTS.md — 三类资产驱动的 AI PPT Agent

> **宪法构成（不可分割）**：本宪法由**三个文件**共同构成，文件名固定为 `agents.md`、`agents-appendix.md`、`agents-issues.md`；三者**必须同时交付、同时读取、缺一不可**。缺任一份即视为宪法不完整，**不得据此执行 `ingest*` / `do*` / `export` / `remove`**：
>
> | # | 文件名 | 作用 |
> |---|---|---|
> | ① | `agents.md` | 主体：原则、命令、流程——规定**做什么**（即本文） |
> | ② | [agents-appendix.md](agents-appendix.md) | 宪法附录：目录、命名、registry 字段、front-matter、两层 INDEX 表头、状态矩阵——钉死**长什么样**，与主体具有同等约束力 |
> | ③ | [agents-issues.md](agents-issues.md) | Issue 表：已确认的问题裁定与处理答案——规定**按哪个口径执行** |
>
> 冲突裁决顺序：**Issue 表 > 附录 > 主体**（后写的裁定优先），并应尽快把稳定裁定回写上位文件。
> 三份文件必须放在**同一目录**，用相对路径互链，可整体复制到任意目录或机器；交付与自检清单见附录 A10。

## 1. 系统目标与不可跳过的原则

### 命令入口

以下是用户与 Agent 交互的任务命令，不表示机器上已安装同名命令行程序。Agent 根据命令执行本文对应流程，不得将原始命令文本直接当作 shell 命令运行。中文名称作为同义入口保留。本文所有示例路径均**相对当前工作区**，文件名只是形式，替换为你自己的。

| 命令 | 中文别名 | 职责 | 完成边界 |
|---|---|---|---|
| `ingest` | 入库 | 指定文件或文件夹入库；PPT 完整入库，非 PPT 自动按 `ingest-content` 处理 | PPT 提取三类资产；非 PPT 仅提取内容资产并更新 Wiki 与索引 |
| `ingest-content` | 内容入库 | 指定 PPT 或文件夹，仅提取内容资产 | 完成去重、编号、Global Content、Content Asset、Wiki 与索引更新；视觉标记为不分析 |
| `ingest-visual` | 视觉入库 | 仅允许 PPT / PPTX，仅提取视觉资产 | 非 PPT 必须直接报错并拒绝入库；PPT 完成 Global Visual、Visual Asset、Wiki 与索引更新 |
| `do` | 制作 | 按需求检索内容与视觉资产并制作演示 | 返回通过 QA 的交互 HTML，等待用户确认 |
| `do-content` | 内容制作 | 仅使用库内内容资产；视觉不使用库内 Visual Asset | 以库内内容为基础完成演示，视觉由 Agent 自行设计，返回通过 QA 的交互 HTML |
| `do-visual` | 视觉制作 | 仅使用库内视觉资产；内容不使用库内 Content Asset | 以库内视觉为依据完成演示，内容来自用户材料或外部可靠来源，返回通过 QA 的交互 HTML |
| `export` | 导出 | 将已确认的 HTML 按要求交付 | 完成所选格式输出及对应最终 QA |
| `lint` | 巡检 | 检查索引、链接、孤岛页与登记一致性 | 默认只报告；显式 `--fix` 时修复可确定的问题 |
| `remove` | 移除 | 按 `ppt_id` 删除整份资产包 | 同步删除资产目录、登记与索引入口，清理相关链接和版本关系；编号不回收 |

#### ingest：完整入库

```text
ingest "产品介绍.pptx"
ingest "待入库"
```

指定单个文件或文件夹时，**只根据文件扩展名分流，不做 MIME、文件内容或其他类型猜测**。扩展名为 `.ppt` / `.pptx` 的文件按 PPT 处理；其他扩展名一律视为非 PPT，并由 `ingest` 自动路由为 `ingest-content`。PPT 的 Content Asset 必须以逐页截图 / 渲染结果为主要内容理解输入，结合多模态读取页面中的文字、图表、图片与其他可见信息；非 PPT 则直接读取文件可解析的正文、结构与元数据来抽取 Content Asset。非 PPT 文件内部如包含图片、图表、扫描页或其他需要视觉理解的嵌入内容，必须对这些内容使用多模态理解并将提取结果并入 Content Asset；这属于内容抽取，不因此建立 Visual Asset。文件夹批量入库时逐文件按扩展名路由：PPT 走完整 `ingest`，非 PPT 走 `ingest-content`；显式 `--recursive` 才递归子目录。忽略 PowerPoint 临时锁文件，不修改或删除用户原文件。首次入库时可自动初始化缺失的资产库目录、登记表和总 INDEX；已有登记表不得重置。

PPT 入库执行完整流程：文件哈希去重 → 程序分配 ppt_id → 保存源文件 → 提取 Global / Content / Visual Asset → 更新两层 INDEX 与实际链接 → 检查本次资产。非 PPT 自动进入内容入库流程，并将视觉状态标记为 `not_applicable`，而不是 `not_analyzed`，以明确该源文件不属于可执行 PPT 视觉资产抽取的类型。批量结果逐项报告实际路由、成功、重复、失败和未验证状态，不把部分成功宣称为全部完成。


#### ingest-content：仅内容资产入库

```text
ingest-content "产品介绍.pptx"
ingest-content "待入库"
```

仅建立内容侧资产，并与 `ingest` 使用完全相同的扩展名分流与内容抽取规则：`.ppt` / `.pptx` 必须逐页截图 / 渲染并结合多模态抽取页面可见内容；非 PPT 必须直接读取可解析的正文、结构与元数据，文件内部存在图片、图表、扫描页等视觉内容时再使用多模态抽取并并入 Content Asset。流程为：文件哈希去重 → 分配或复用 `ppt_id` → 按原扩展名保存源文件 → 内容抽取 → 建立 Global Content 与 Content Asset → 更新 Wiki、INDEX 与登记状态 → 内容检查。不得建立 Visual Asset。PPT 的视觉状态标记为 `not_analyzed`；非 PPT 的视觉状态标记为 `not_applicable`。

同一源文件此前若已通过 `ingest-visual` 入库，不得因哈希相同而跳过；应复用原 `ppt_id` 并补齐 Content Asset。不得重复创建资产目录。

#### ingest-visual：仅视觉资产入库

```text
ingest-visual "产品介绍.pptx"
ingest-visual "待入库"
```

`ingest-visual` **只允许扩展名为 `.ppt` / `.pptx` 的文件**，判断依据仅为文件扩展名。任何其他扩展名必须立即报错并拒绝该文件：**不得自动转为 `ingest-content`，不得尝试截图或渲染后冒充 Visual Asset，也不得创建伪视觉资产。** 文件夹批量执行时，非 PPT 项逐项报错；其他合法 PPT 可继续处理，并在最终结果中明确列出被拒绝文件。

对合法 PPT，仅建立视觉侧资产：文件哈希去重 → 分配或复用 `ppt_id` → 保存源文件 → 提取视觉所需原生结构 → 逐页渲染并保存标准 Preview → 建立 Global Visual 与逐页 Visual Asset → 更新 Wiki、两层 INDEX 与登记状态 → 视觉检查。不得形成 Content Knowledge；对应内容状态标记为 `not_analyzed`。

同一 PPT 源文件此前若已通过 `ingest-content` 入库，应复用原 `ppt_id` 并补齐 Visual Asset。两侧均完成后，该 PPT 的整体资产状态更新为 `complete`。

#### do：制作

```text
do 面向企业管理层的 AI 产品介绍，10 页
```

执行需求 → 全局资产检索 → Storyline → Slide Brief → 内容检索 → 强制视觉检索或复用 → 按需渲染 → HTML 设计与视觉、交互 QA。返回带左侧缩略图预览和全屏翻页能力的 HTML，状态停在 `awaiting_user_confirmation`。

即使需求中写了“制作 PPT / PDF”，本命令也必须先返回 HTML；不能提前转换。用户反馈修改时继续更新同一演示，并再次提交预览。

#### do-content：仅使用库内内容资产

```text
do-content 面向企业管理层的 AI 产品介绍，10 页
```

执行需求 → Global Content / Content Asset Retrieval → Storyline → Slide Brief → 内容补齐与来源核验 → Agent 自主视觉设计 → HTML 视觉与交互 QA。**不得检索、读取或复用库内 Visual Asset 作为设计依据。** 视觉可以由 Agent 根据当前内容和全局要求自行设计。其余预览、确认与导出规则与 `do` 相同。

#### do-visual：仅使用库内视觉资产

```text
do-visual 面向企业管理层的 AI 产品介绍，10 页
```

执行需求 → Storyline / Slide Brief → 从用户材料或外部可靠来源形成内容 → Visual Asset Retrieval → 按需渲染并确认视觉参考 → HTML 设计与视觉、交互 QA。**不得检索、读取或复用库内 Content Asset 作为内容来源。** 其余预览、确认与导出规则与 `do` 相同。

#### export：导出

```text
export pptx
export pdf
export html
export pptx,pdf
确认当前 HTML，export pptx,pdf
```

支持 `pptx`、`pdf`、`html`，其中 `ppt` 作为 `pptx` 的输入别名。仅对已明确确认的当前 HTML 版本执行；单独收到 `export` 不自动视为确认。用户可以在同一条消息中明确确认并指定格式，已有有效确认时无需重复确认。

多个演示或版本并存且目标不明确时，先明确目标，不能猜测导出对象。PPT 经 DOM / Render IR 与 Renderer 输出；PDF 从 HTML 直接导出；HTML 保留交互和所需资源。只生成用户要求的格式，并分别完成最终 QA。

#### lint：索引与关联巡检

```text
lint all
lint ppt_000003
lint all --fix
```

省略范围时默认 `all`。单份巡检除检查本目录，还要核对全库登记、总 INDEX 入口和指向该资产的跨目录链接，但不借机修改无关资产。

至少检查：

- 全库 INDEX 是否覆盖已入库 PPT，每份 INDEX 是否链接到 PRESENTATION.md 和全部逐页 Wiki；反向导航是否齐全。
- 内部链接是否有效，是否存在同名误链、目标缺失、错误页码、失效的标题锚点，以及跨 PPT 引用或版本关系不一致。
- 以全局 INDEX 为起点，沿实际 Markdown 内部链接计算可达性；无法访问到的资产 Markdown 文件标记为孤岛。区分完全无链接的页面与彼此互链但整体不可达的孤立分组，不能仅用入链数量判断。
- 登记表、ppt_id、目录名、源文件哈希、版本关系、原 PPT 实际页数及逐页 Wiki 是否一致；不存在的源文件、缺页、重复页与未完成入库分别报告。
- 逐页 Content / Visual 区块、全局资产、来源及必要状态字段是否缺失；结构检查不等同于事实核验或视觉 QA。

默认只读，输出包含文件、问题类别、具体证据、影响和修复建议的清单，并报告检查范围及未能验证的项目。登记表等辅助文件和临时输出不作为资产笔记参与孤岛统计；指向不存在文件的链接不算有效可达路径。 对分模式入库必须按登记状态判断完整性：与命令模式一致的 `content_status: not_analyzed`、`visual_status: not_analyzed` 或非 PPT 的 `visual_status: not_applicable` 都不是缺失错误；只有状态宣称已完成却缺少对应资产、或状态与实际文件不一致时才报告问题。

`--fix` 授权修复依据明确的问题，例如补齐已确认归属的索引入口和返回链接、将唯一可确定的错误相对路径改为正确目标。修复后重新巡检，并列出已修复与仍未解决的问题。

不得为消除孤岛而虚构语义关系；不得自动删除页面、重编号、覆盖源 PPT、改写内容事实或猜测版本关联。缺失的资产分析不能用空白占位文件冒充完成；归属不明或存在多个可能目标时保留问题并请求必要信息。

#### remove：移除整份资产

```text
remove ppt_000002
```

`remove` 仅接受明确的 `ppt_id`，用于删除该 ID 对应的整份资产包，不用于删除单页或仅删除 Content / Visual 的某一侧。执行前必须确认该 `ppt_id` 在 `registry.json` 中存在且与目标资产目录一致；目标不存在时报告不存在，不得猜测或按名称模糊匹配。

执行顺序：核对 `ppt_id` 与登记记录 → 检查全库对该资产的实际引用 → 删除该资产目录 → 从 `registry.json` 删除对应资产登记 → 从全局 `INDEX.md` 删除入口 → 清理其他 Wiki 中指向已删除资产的实际 Markdown 链接及已确认的版本关系 → 对受影响范围执行一致性巡检。`next_sequence` 不回退，已删除的 `ppt_id` 永不重新分配。

若其他资产的 `previous_version_ppt_id` 指向被删除资产，应移除该悬空关系或按已有、可核实的信息更新；不得猜测新的版本关系。跨 PPT 内容关联、视觉参考或替代建议若指向被删除资产，应删除失效链接及仅依赖该目标成立的关联说明，不得留下断链。删除操作不得影响与目标无关的资产。

`remove` 是显式破坏性操作，只处理用户明确指定的 `ppt_id`；不得由 `lint --fix`、去重、版本更新或其他流程隐式触发。

把每份历史 PPT 建成一个复合资产包，通过资产检索、内容规划和视觉参考，先生成内容可信、设计优良、可交互演示的 HTML。用户预览确认后，再按其选择交付 HTML、可编辑 PPTX 或由 HTML 直接生成的 PDF。

系统以三类资产组织知识和工作：

| 资产 | 粒度 | 回答的问题 | 主要复用价值 |
|---|---|---|---|
| Global Asset | 整份 PPT | 面向谁、讲什么、按什么顺序讲、如何论证、整套呈现如何统一？ | 主题、目录、storyline、推进与论证方式、全局视觉语言 |
| Content Asset | 每一页 | 这一页讲了什么，哪些内容可以可信地复用？ | 原文、观点、事实、数据、案例、方法和知识 |
| Visual Asset | 每一页 | 这一页如何有效表达，什么内容适合放进去？ | 表达意图、信息结构、布局、层级、风格、容量和适配经验 |

三类资产分别检索、分别记录来源，可以来自不同 PPT。内容与视觉解耦，但不应无理由拆开一个已经同时适合内容与视觉的历史页面。

### 1.1 强制原则

1. 优先检索同主题 Global Asset 和 Content Asset，再考虑相似主题或跨领域的结构经验。
2. 内容资产缺失时，从用户材料或外部可靠知识形成内容；不得编造事实、数字、案例或来源。
3. **在 `do` 与 `do-visual` 模式中，Visual Asset 是每页必经的核心环节。内容没有命中，绝不构成跳过视觉检索的理由。** 封面、目录、章节页和结尾页也不例外；`do-content` 是显式例外，不读取库内 Visual Asset，由 Agent 自主完成视觉设计。
4. 内容高度命中某页时，必须先评估该页视觉；适合则优先复用，不适合才另找主视觉参考。
5. Wiki 只能筛选视觉候选，必须查看对应的真实页面 Preview 后才能确认参考；Preview 不可用或身份不一致时，必须从原 PPT 重新渲染确认。
6. 禁止收到主题后直接生成 HTML 或 PPT。`do` 与 `do-visual` 禁止无库内视觉依据地自由拼装页面；`do-content` 允许不依赖库内 Visual Asset 自主设计，但仍必须经过 HTML 视觉与交互 QA。
7. 页面设计和视觉修复在 HTML 阶段完成；COM / PPTX Renderer 执行已确定的设计，不负责猜坐标或重新设计。
8. 必须先返回可打开的 HTML 供用户预览，并等待用户明确确认当前版本；确认前不得转换为 PPT 或 PDF。不能把“做一份 PPT”的初始请求或泛化的自主执行授权视为预览确认。
9. 确认后只生成用户要求的格式：HTML 保留交互；PPT 走 DOM / Render IR → COM / PPTX → 最终 PPT QA；PDF 从已确认的 HTML 直接导出并进行 PDF QA，不经过 PPT。

### 1.1.1 `do` 模式边界

- `do`：同时启用库内 Global / Content / Visual Asset，执行完整资产驱动流程。
- `do-content`：库内只启用内容侧资产；跳过库内视觉检索与复用，允许 Agent 自主完成视觉设计。
- `do-visual`：库内只启用视觉侧资产；跳过库内内容检索，内容必须来自用户材料或外部可靠来源。
- “只用”指禁止使用被关闭一侧的**库内资产**，不表示禁止完成该侧工作；三种模式最终都必须形成完整可预览的演示。

### 1.2 标准生成顺序

```text
需求
→ Global Asset Retrieval（优先同主题，并了解已有内容覆盖）
→ Storyline
→ Slide Brief × N
→ 每页 Content Retrieval（缺失则补充可靠内容）
→ Visual Asset Retrieval / Reuse Decision
→ 读取 Visual Preview、视觉确认；必要时从原 PPT 重新渲染
→ HTML 设计
→ HTML 视觉 QA / 修复
→ 返回交互 HTML（左侧缩略图预览 + 全屏翻页放映）
→ 等待用户明确确认当前 HTML 版本
→ 按用户要求分支交付
   ├─ HTML → 保留预览 / 放映交互 → 最终 HTML QA
   ├─ PPT → DOM bbox / Render IR → COM / PPTX → 最终 PPT QA
   └─ PDF → HTML 直接打印 / 导出 PDF → 最终 PDF QA
```

这是带质量门槛的顺序流程。后续发现内容数量、论证关系或设计发生实质变化时，返回对应阶段并重新验证，不得带着失效的 Brief 或参考决定继续输出。

## 2. Global Asset：整份 PPT 资产

每份 PPT 建立 `PRESENTATION.md`，保存完整的全局资产。它不是页标题列表，也不是若干单页摘要的拼接。

### 2.1 必须记录的内容

| 字段 | 要求 |
|---|---|
| 主题与目的 | 核心主题、核心主张、业务用途、希望受众理解或采取的行动 |
| 受众与情境 | 受众角色、已有知识、关注点、演讲或阅读场景 |
| 目录与章节 | 实际章节顺序、章节职责、对应页码范围 |
| Storyline | 用连贯叙述说明整份演示如何从起点推进至结论 |
| 章节推进 | 问题→方案、背景→洞察→行动、总分总、时间线等推进关系，以及章节间过渡 |
| 论证方式 | 观点如何由数据、案例、对比、机制、演示或方法支撑；记录关键证据所在页 |
| 开场与收尾 | 如何建立问题或注意力，如何总结、提出建议或引导行动 |
| 全局视觉语言 | 比例、字体层级、色彩角色、网格、边距、留白、图表与图标风格、章节节奏 |
| 复用边界 | 适用任务、可迁移结构、领域限定内容、不适用情境、需要替换的环节 |
| 来源与状态 | `ppt_id`、源文件、版本标识、分析日期、已观察事实与推断、置信度 |

建议结构：

```yaml
global_asset:
  ppt_id: ppt_000001
  theme: 产品介绍
  audience: 企业业务负责人
  presentation_goal: 理解产品价值并确定下一步试用行动
  core_message: 待从原文提取
  chapters:
    - title: 业务问题
      slide_range: [2, 4]
      role: 建立需求
  storyline: 问题与机会 → 产品定位 → 能力与场景 → 证据 → 行动
  progression_patterns: [problem_solution, claim_evidence_action]
  argumentation:
    - claim_slide: 5
      evidence_slides: [6, 7]
      method: case_and_data
  opening: 从受众的业务问题切入
  closing: 总结价值并提出下一步行动
  global_visual_language:
    aspect_ratio: '16:9'
    typography: 记录实际字体与层级
    colors: 记录背景、正文、强调色角色
    layout_rhythm: 记录信息页与过渡页的节奏
  reuse_for: [产品介绍, 方案说明]
  avoid_when: [需要完整学术推导]
```

示例仅说明资产格式，不能当作对任何实际 PPT 的分析结论。缺失字段标为未知，不得为补齐字段虚构内容。

### 2.2 全局复用规则

- 同主题且受众、目标匹配时，优先借鉴目录、storyline 和论证组织；具体事实仍需验证。
- 没有同主题资产时，检索相似任务和叙事方式。例如另一个产品的介绍可以贡献推进结构，但不能直接贡献当前产品的功能事实。
- 完全没有适合的 Global Asset 时，根据需求自行规划 storyline，并记录检索范围与未采用原因；逐页视觉检索仍然强制执行。
- 全局视觉语言可以来自另一份 PPT，但必须形成统一的新演示视觉规范，不能逐页随意切换风格。

## 3. Content Asset：每页内容资产

每个 `slide-XXX.md` 中独立设置 `Content Asset` 区块。原文、提炼知识与来源分别保存。

### 3.1 Source Content：忠实保存原始内容

尽可能提取原始标题、副标题、正文、bullet、图表标题与数据、表格、标签、图片 caption、有业务意义的页脚、Speaker Notes，以及可识别的流程步骤、产品名、客户名、功能名和指标名。

- 保留原文，不用摘要覆盖原文；标明正文、备注、图表等位置。
- 数据保留数值、单位、时间、口径、适用范围和原页引用。
- 对图片中的文字或图表识别结果标注识别方式及不确定性；不可读内容不得猜测。

### 3.2 Content Knowledge：可检索与可复用知识

```yaml
content_asset:
  content_topic: []
  content_summary: ''
  core_claims: []
  key_facts: []
  key_metrics: []
  cases: []
  methods: []
  entities: []
  products: []
  features: []
  business_concepts: []
  use_cases: []
  reusable_knowledge: []
  questions_this_slide_can_answer: []
  content_keywords: []
  reuse_boundaries: []
```

区分原页陈述、作者观点和分析者推断。客户专属承诺、个案结果、假设和预测，不得抽象成适用于所有场景的事实。

### 3.3 来源与时效

每个复用观点、数据或案例都应能追溯到具体来源。历史 PPT 是“原页曾这样写”的证据，不自动证明其今天仍然正确。

```yaml
source_refs:
  - source_id: src_001
    type: ppt
    ppt_id: ppt_000001
    slide_id: 12
    location: 图表及页脚引用
    original_citation: 原页注明的文献或链接；没有则标未提供
    used_for: [claim_01, metric_01]
freshness:
  status: needs_verification
  as_of: null
  verified_at: null
  verification_source: null
```

外部资料记录标题、发布机构、URL、发布日期（如可得）、访问或验证日期；用户材料记录文件及页码或段落位置。涉及价格、产品功能、市场数据、政策、组织信息等时效性内容，使用前核验。无法确认时标明不确定、删去该主张或保留待补缺口，不得伪装成已核实。

## 4. Visual Asset：每页视觉资产

每个 `slide-XXX.md` 中独立设置 `Visual Asset` 区块，继承并完整组织原 Page DNA 的视觉能力。视觉资产必须足够具体，能支持检索、适配和质量判断。

### 4.1 必须抽取的视觉维度

| 维度 | 必须回答的问题 |
|---|---|
| Semantic Role | 是封面、目录、总结、概念、产品、问题、方案、对比、流程、时间线、架构、指标、案例、引言、团队还是 CTA？可多标签 |
| 表达意图 | 解决什么表达任务？例如“以三个并列证据支撑一个主张”，不能只写“三栏” |
| 信息结构 | 主观点、支撑项、数据、图片、图表、说明、标签和 CTA 的数量与关系是什么？ |
| Layout | 主次区域、列数、比例、网格、对齐、阅读顺序、对称性、视觉中心、留白如何安排？ |
| 视觉层级 | 第一眼、第二眼、第三眼分别看到什么？标题、数字、正文、注释如何区分？ |
| 风格 | 极简、编辑式、企业、科技等风格，以及明暗、字体气质、卡片、圆角、边框、阴影、图像、图标、图表和装饰倾向 |
| 色彩角色 | 背景、主文字、次文字、强调色与辅助色，而不只是颜色列表 |
| 容量 | 标题长度、支撑项数量、每项行数、正文密度、图片与图表需求，注明语言和画布条件 |
| 适用 / 不适用 | 适合的任务和信息关系，以及长段落、超量条目、复杂表格等明确排除项 |
| 可变性 | 必须保留的构图、层级、留白节奏，与可变化的图标、照片、装饰、文案和有限条目调整 |
| 视觉摘要 | 能直接供 HTML 设计理解的自然语言描述 |
| 质量与经验 | 视觉优缺点、复用反馈、已知限制、替代参考页 |

容量是推荐范围，不是填满目标，也不是无限适配许可。超过容量优先精简、拆页或换参考，不能通过持续缩小字号硬塞。

并列、递进、因果、对比等关系必须区分。三个并列模块不自动适合三个连续步骤。

### 4.2 视觉结构示例

```yaml
visual_asset:
  semantic_roles: [feature_overview, three_key_points]
  expression_intent: 用一个核心观点统领三个同等级的支撑信息
  content_schema:
    primary_message: 1
    supporting_items: 3
    relation: parallel
    supporting_item_fields: [short_title, short_description, optional_icon]
  layout:
    family: title_plus_three_columns
    composition: top_title_bottom_three_equal_columns
    reading_order: left_to_right
    symmetry: high
    density: low
    focal_region: upper_left
  hierarchy:
    level_1: main_title
    level_2: item_titles
    level_3: supporting_text
    level_4: annotations
  visual_style:
    keywords: [minimal, corporate]
    whitespace: high
    decoration: low
    card_style: flat
  colors:
    background: '#FFFFFF'
    primary_text: '#161616'
    secondary_text: '#666666'
    accent: '#2458D3'
    supporting: []
  capacity:
    language: zh-CN
    title_chars_recommended: '8–24'
    item_count: {min: 3, ideal: 3, max: 4}
    supporting_text: 每项 1–3 行；需真实渲染验证
    body_density: low
    image_count: {ideal: 0}
  suitable_for: [三个能力, 三个优势, 三个并列证据]
  avoid_when: [连续步骤关系, 多个长段落, 复杂数据表]
  adaptability:
    preserve: [整体构图, 信息层级, 留白节奏, 视觉平衡]
    flexible: [图标, 装饰, 文案, 强调色]
    conditional: [条目数变化后必须重新检查容量和节奏]
  visual_summary: 白底强留白，顶部大标题，下方三个等宽模块，以短标题和简短说明构成。
  search_text: 三个能力 并列证据 三栏 极简 企业风
```

示例中的数值与颜色不作为全库硬规则。参考页是某次成品，原客户 logo、图片、承诺和装饰均不应被当作模板必需元素。

## 5. 资产入库与 Markdown Wiki

### 5.1 存储结构与真实来源

以下路径均相对当前工作区；`ppt-library/` 与 `work/` 由入库流程自动建立。

```text
ppt-library/
├── INDEX.md                       # 第一层：全库导航
├── ppt_000001/
│   ├── source.<原扩展名>           # 长期保留的原始源文件；如 source.pptx / source.pdf / source.docx
│   ├── INDEX.md                   # 第二层：本 PPT 的逐页导航
│   ├── PRESENTATION.md            # Global Asset
│   ├── previews/                  # Visual Asset 的持久化逐页预览，仅视觉已分析的 PPT 建立
│   │   ├── slide-001.png
│   │   └── ...
│   ├── slide-001.md               # Content Asset + Visual Asset
│   ├── slide-002.md
│   └── ...
└── ppt_000002/
    └── ...

work/<run-id>/                     # 临时渲染、候选图与 QA 工作文件
```

- 原始源文件是内容的 Source of Truth；对 PPT，原始 `.ppt` / `.pptx` 同时也是视觉 Source of Truth。Wiki 是提炼、导航和使用经验，不替代原文件。
- 默认不建立 Vector DB、Embedding 或本地向量索引。通过 AI 阅读 Markdown Wiki 进行分层导航、语义判断和候选重排。
- 长期按原扩展名保存源文件，并保存两层 INDEX、PRESENTATION.md 和对应资产 Wiki。
- 对完成 Visual Asset 分析的 PPT，必须长期保存入库时实际用于视觉分析的逐页标准 Preview，默认位于 `<ppt_id>/previews/slide-XXX.png`。Preview 是 Visual Asset 的持久化视觉表示，用于后续检索确认与设计参考，但不替代原始 PPT 作为视觉 Source of Truth。`ingest-content` 为内容理解产生的临时截图不保存为 Preview；非 PPT 不建立 Visual Preview。
- 使用 `ppt_id + slide_id + source_version` 定位原页。`slide_id` 明确为从 1 开始的实际页序号；源文件更新或重排时作为新文件分配新 ppt_id，重新生成对应 Wiki、索引和缓存；保留旧源文件和旧引用，禁止覆盖后静默错配。

### 5.1.1 唯一编号、目录命名、去重与版本管理

**文件夹名统一使用永久唯一的 `ppt_id`，原文件名和展示名称只作为元数据保存。** 本单库采用顺序编号：`ppt_000001`、`ppt_000002`……数字至少补齐六位，超过六位时自然扩展，不截断、不重置。不得再用主题、原文件名或展示名称作为资产目录名。

```text
ppt-library/
├── INDEX.md
├── registry.json                 # 程序维护的编号与文件登记表
├── ppt_000001/
│   ├── source.<原扩展名>
│   ├── INDEX.md
│   ├── PRESENTATION.md
│   ├── previews/
│   │   └── slide-001.png
│   └── slide-001.md
└── ppt_000002/
    ├── source.<原扩展名>
    ├── INDEX.md
    ├── PRESENTATION.md
    └── slide-001.md
```

`registry.json` 是入库程序的登记表，不替代 Markdown Wiki 或两层 INDEX，也不参与向量检索。至少保存单调递增的 `next_sequence`，以及各资产的 `ppt_id`、`original_filename`、`display_name`、`source_sha256`、`source_version`、`relative_path`、`ingested_at`、`status` 和可选的 `previous_version_ppt_id`。对应身份字段同步到 PRESENTATION.md；逐页记录所属 ppt_id 和源版本。 `status` 应区分整体状态，并至少另存 `content_status` 与 `visual_status`（按适用情况使用 `not_analyzed`、`not_applicable`、`processing`、`complete`、`failed`、`unverified`）；分模式入库时允许同一 `ppt_id` 后续补齐另一侧资产，两侧均完成后整体状态为 `complete`。

入库程序按以下顺序处理，不能让模型猜测下一个编号：

1. 对待入库文件计算完整文件的 SHA-256 哈希，并检查登记表。
2. 哈希相同表示文件字节内容完全相同，复用已有资产与 `ppt_id`，不分配新 ID；若当前命令请求的 Content 或 Visual 资产尚未完成，则在该资产目录中补齐对应侧，而不是以“重复”为由跳过。可补充原文件名别名或来源记录。哈希不同不代表语义必然不同，不得声称已完成语义去重。
3. 新文件在独占锁或等效事务保护下，重新检查哈希，从 `next_sequence` 领取编号并推进计数器，登记为处理中；目录已存在时不得覆盖，需先检查登记冲突。
4. 将源文件按原扩展名复制到该目录，例如 `.pptx` → `source.pptx`、`.pdf` → `source.pdf`、`.docx` → `source.docx`；不得强制改成 `.pptx`。校验哈希后完成对应资产分析和 Wiki 写入，再更新状态。登记表采用原子写入，防止中断损坏。
5. 并行入库必须共享同一分配锁，避免编号冲突和重复入库。失败记录保留，可在核实同一源文件后恢复原任务；删除或取消不回收编号，不按现有文件夹数量重新计数。
6. 登记表缺失或损坏时先根据已有资产核对恢复，并保留已用编号记录；无法证明编号可安全使用时停止分配，不覆盖现有目录。

重名与版本规则：

| 情况 | 处理 |
|---|---|
| 同名、文件哈希不同 | 分配不同 ppt_id，分别入库，绝不因重名覆盖 |
| 同名或异名、文件哈希相同 | 复用已有资产，不重复编号 |
| 已确认是已有 PPT 的新版 | 分配新 ppt_id，通过 previous_version_ppt_id 关联旧版，保留旧源文件、Wiki 和引用 |
| 仅文件名相同、版本关系不明确 | 当作独立资产；不能只凭名称自动认定版本关系 |
| 修改展示名称 | 只更新元数据与索引显示文本，ppt_id、目录和链接目标不变 |

源文件一经登记即视为不可变快照，不能用新版覆盖旧目录中的 `source.<原扩展名>`。`source_version` 记录该快照的明确版本标识，并由 `source_sha256` 核实文件身份；新版本关系来自用户说明或已核实的证据。

例如两份“产品介绍.pptx”可分别存于 `ppt_000001/` 和 `ppt_000002/`，在总 INDEX 中展示为“AI 产品介绍”和“制造业产品介绍”。更新版分配 `ppt_000003`，若已确认对应第一份，记录 `previous_version_ppt_id: ppt_000001`，并在 Wiki 中建立实际版本链接。

顺序编号的唯一性范围是当前资产库。跨库合并时检查 ID 冲突，重编号导入资产并同步修正登记表、版本关系和所有 Wiki 链接，禁止直接按同名目录覆盖。

### 5.2 入库流程

```text
原 PPTX + 可选的名称 / 行业 / 品牌 / 用途 / 受众说明
→ 文件哈希去重 → 程序分配唯一 ppt_id → 创建同名资产目录与登记记录
→ 提取逐页原生结构与原始内容
→ 逐页渲染，结合结构进行视觉分析，并将实际分析图保存为标准 Preview
→ 建立逐页 Content Asset 和 Visual Asset
→ 通读页面顺序与章节关系，建立 Global Asset
→ 建立每份 PPT INDEX
→ 更新全库 INDEX
→ 检查来源、链接、页码、Preview 与覆盖情况
→ 清理非资产性的临时图片；保留 Visual Asset 标准 Preview
```

完整 `ingest` 与 `ingest-visual` 必须把视觉分析时实际观察的逐页渲染图保存为标准 Preview，并与 `ppt_id + slide_id + source_version + source_sha256` 对应。`ingest-content` 产生的内容理解截图仍属于临时文件，任务结束后清理。生成时优先读取已有 Preview，只有 Preview 缺失、损坏、身份校验不一致或确有重新验证需要时才从原 PPT 重新渲染。

原生结构至少提取：slide size / aspect ratio、页码、master / layout、background、shape 数量与类型、bbox、z-order、文本、字体族 / 大小 / 字重 / 颜色、fill、line、transparency、image bbox / crop、chart / table / diagram 类型、group、alignment 和 rotation。

**不能仅凭 OOXML / COM Shape 数据判断视觉。** 页面分析必须结合实际渲染图和结构信息。无法渲染时标记 `visual_status: unverified`，不得声称已完成视觉理解，也不得把该页直接当作已验证参考。

### 5.3 两层 INDEX

第一层 `ppt-library/INDEX.md`：帮助检索 Global、Content 和 Visual，不只导航风格库。每份 PPT 至少列出：

- `ppt_id`、名称、页数、路径、版本、`PRESENTATION.md` 与本地 `INDEX.md` 链接。
- 主题、产品、业务、案例、行业、受众与用途。
- storyline / 推进模式摘要、论证特点、可复用的全局结构。
- 内容覆盖、时效状态、全局视觉风格、擅长表达的任务、常见布局与密度。

第二层 `<ppt_id>/INDEX.md`：链接本 PPT 的 Global Asset，并导航逐页双资产。

```markdown
| 页 / Wiki 链接 | 章节角色 | 内容主题 / 核心观点 | 内容时效 | 表达意图 | 信息结构 / Layout | 容量 | 风格 / 适用限制 |
|---|---|---|---|---|---|---|---|
| 12 / slide-012.md | 能力说明 | 某产品的三个能力 | 待核验 | 三点并列 | 1+3 / 三栏 | 每项短文 | 极简 / 不适合递进 |
```

INDEX 只保存导航摘要；详细判断读取 `PRESENTATION.md` 或 `slide-XXX.md`，最终视觉判断查看真实渲染页。禁止一次性加载整个知识库。

### 5.4 逐页 Wiki 的固定结构

```markdown
# Slide 012
## 身份与来源
ppt_id、slide_id、source_version、源文件、所属章节、分析状态
## Content Asset
### Source Content
### Content Knowledge
### 来源、时效与复用边界
## Visual Asset
### 表达意图与信息结构
### Layout、层级、风格与色彩
### 容量、适用 / 不适用场景与可变性
### 视觉摘要与质量判断
## 客观结构信息
## 使用经验与替代参考
```

人工反馈可持续写入使用经验，例如“适合三个同等级能力，不适合连续步骤；连续步骤参见 slide-043.md”。反馈不能覆盖原文或伪装成源 PPT 的事实。

### 5.5 Wiki 链接规范与 Obsidian 关系图

将整个 `ppt-library/` 文件夹作为 Obsidian 仓库打开。全局 `INDEX.md` 是本系统的导航主入口；Obsidian 关系图本身没有固定根节点，也不保证将此文件放在全图中心。查看全局 INDEX 的局部关系图并增加显示深度，可以围绕入口浏览资产关联。

所有导航关系必须写成可解析的标准 Markdown 内部链接，不能只写文件名、页码或目录文本。链接应放在正文或表格中，不放在代码块里充当实际导航。下列代码块只展示写法，生成 Wiki 时必须输出为真实链接。

**必须建立的导航链接：**

1. 全局 `INDEX.md` 链接到每份 PPT 的 `INDEX.md`；按前述索引要求，同时提供对应 `PRESENTATION.md` 入口。
2. 每份 PPT 的 `INDEX.md` 链接到自己的 `PRESENTATION.md` 和全部 `slide-XXX.md`，并提供返回全库 INDEX 的链接。
3. 每份 `PRESENTATION.md` 与每个 `slide-XXX.md` 提供返回所属 PPT INDEX 的链接；全局资产中的关键论证页、证据页等引用也应链接到对应 Slide Wiki。
4. 有实际依据的跨页内容关联、视觉参考、替代建议和使用经验，必须链接到具体目标 Wiki。不能只写“参见 Slide 38”，也不能仅因关键词相同就批量建立无意义关联。

**路径与显示名称：**

- 使用相对于当前 Markdown 文件的路径，明确链接到 `.md` 文件，不只链接到文件夹。
- 跨 PPT 链接必须包含目标目录；不得用含糊的 `[[INDEX]]` 或 `[[slide-001]]`，以免同名文件错配。
- 链接文字使用可读名称，例如“制造业方案 · 第 38 页”，便于分辨相同文件名的资产。
- 路径中的空格使用 `%20` 等标准 URL 编码；移动或重命名文件后检查并更新相关链接。

全局 `INDEX.md` 中的写法：

```markdown
[AI 产品介绍 · 页面索引](ppt_000001/INDEX.md)
[AI 产品介绍 · 全局资产](ppt_000001/PRESENTATION.md)
[制造业方案 · 页面索引](ppt_000002/INDEX.md)
```

`ppt_000001/INDEX.md` 中的写法：

```markdown
[返回全库索引](../INDEX.md)
[本 PPT 全局资产](PRESENTATION.md)
[第 1 页](slide-001.md)
[第 2 页](slide-002.md)
```

`ppt_000001/slide-012.md` 中的写法：

```markdown
[返回 AI 产品介绍索引](INDEX.md)
[所属全局资产](PRESENTATION.md)

相似视觉结构：[制造业方案 · 第 38 页](../ppt_000002/slide-038.md)
关联原因：两页均以三个同等级模块支撑一个主观点，可比较其容量与留白。
```

以上名称和页码仅为示例；只能为实际存在且关系已核实的文件生成链接。

**关系图粒度与入库检查：**

- 关系图中的笔记节点对应 Markdown 文件，连线来自内部链接，不会仅根据文件夹归属或语义相似自动产生。
- 每个 `PRESENTATION.md` 是一个全局资产节点；每个 `slide-XXX.md` 是一个逐页资产节点。同一文件中的 Content Asset 和 Visual Asset 不会自动拆成两个节点，仍可分别检索和使用。
- 不为图形更密集而复制内容、拆分无必要文件或添加无依据的交叉链接。
- 入库结束时检查：全库入口覆盖所有 PPT，每份索引覆盖全部已入库页，返回链接有效，跨 PPT 链接指向正确文件，无断链或同名误链。
- 本规范使用标准 Markdown 链接，不依赖 Obsidian 专用插件；Obsidian 是浏览和关系展示工具，不是资产检索或生成流程的必需运行环境。

## 6. 需求、Global Retrieval 与 Storyline

### 6.1 明确需求

记录 `presentation_goal`、主题、受众、核心信息、使用场景、预期页数或时长、语言、内容材料、品牌要求和交付要求。缺失项可以采用合理假设并注明，关键内容缺口必须明确。

### 6.2 Global Asset Retrieval

先读全库 INDEX，选择候选 PPT，再读其 PRESENTATION.md 与 INDEX：

1. 优先同主题且受众、目标匹配的全局资产，同时识别其内容覆盖。
2. 无合适命中时扩大到相似主题、相似演示任务和跨领域推进模式。
3. 综合比较目录职责、storyline、章节推进、论证方式、开场 / 收尾及视觉语言；不得只按产品关键词判断。
4. 记录选用资产、复用环节、调整理由和不采用的限制。无命中时明确标记，不伪造来源。

此阶段可按需读取关键内容页来验证大纲可行性；不能替代 Brief 形成后的逐页 Content Retrieval。

### 6.3 建立 Storyline 与全局视觉规范

输出：

```yaml
presentation_goal: ''
audience: ''
core_message: ''
tone: ''
slide_count: 0
global_asset_refs: []
story_arc: ''
chapters: []
opening_strategy: ''
argumentation_plan: []
closing_strategy: ''
global_visual_spec:
  slide_size: 默认 16:9；用户要求优先
  typography: ''
  color_roles: {}
  grid_and_margins: ''
  chart_and_icon_style: ''
  density_and_rhythm: ''
content_gaps: []
```

新 storyline 应服务当前需求，不能因为历史目录存在就全部照搬。全局视觉规范用于约束后续跨 PPT 的视觉复用。

## 7. Slide Brief 与每页 Content Retrieval

### 7.1 为每页建立 Brief

```yaml
slide_no: 4
chapter_role: 核心能力说明
purpose: 帮助受众理解三个同等级能力
key_message: 待由可靠内容支持的单一核心观点
content_requirements: [能力名称, 简短解释, 必要证据]
information_type: [concept, feature_overview]
information_relation: parallel
preferred_visual_expression: [three_parallel_points]
content_count: {primary_message: 1, supporting_items: 3}
visual_priority: 主标题高于支撑项，三项同等级
density: low
content: []
content_sources: []
unresolved_gaps: []
```

Brief 是检索输入和设计契约。初稿描述所需内容，逐页内容检索后补齐文案与证据，再更新实际数量、字数和密度，作为视觉匹配的最终输入。

### 7.2 内容检索与补充

按主题、具体主张、产品实体、问题和知识需求，从两层 INDEX 导航到 Content Asset：

- 同主题内容优先，判断事实、口径、适用范围及当前任务是否一致。
- 相似主题内容只复用确实通用的知识或方法，不能把其他产品功能、客户成果和服务承诺迁移成新主题事实。
- 无命中或覆盖不足时，使用用户材料和外部可靠来源，优先原始资料、官方材料及可追溯研究。
- 核验时效、解决冲突并记录来源；无法支撑的内容作为缺口处理。

内容高度命中表示核心主张、事实对象及信息结构有实质相似性，不能只因标题有同一关键词就认定。记录命中页及具体复用内容，交给下一阶段优先评估其视觉。

## 8. Visual Asset Retrieval / Reuse Decision

**本阶段对每一页强制执行，和内容是否命中无关。**

### 8.1 内容高度命中：先评估原页视觉

```text
Content 高度命中历史页
→ 读取该页 Visual Asset
→ 评估信息结构、容量、全局风格、视觉质量和可变性
├─ 适合 → 作为优先主参考候选
├─ 整体适合、局部不足 → 保留主参考，补充明确局部参考
└─ 不适合 → 记录原因，独立检索其他 Visual Asset
→ 按需渲染候选页并进行真实视觉确认
```

```yaml
visual_reuse_check:
  content_structure_fit: ''
  capacity_fit: ''
  current_deck_style_fit: ''
  visual_quality: ''
  adaptability: ''
  decision: reuse_candidate_or_search_other
  reasons: []
```

内容命中带来视觉评估优先级，不意味着无条件复制。真实渲染确认不通过时返回检索；通过后优先使用该页，不无理由替换成其他视觉。

### 8.2 独立视觉检索

无内容命中、内容来自外部，或原内容页视觉不适合时，均执行：

```text
最终 Slide Brief + Global Visual Spec
→ 全库 INDEX
→ 选择通常 2–5 套候选 PPT
→ 候选 PPT INDEX
→ 选择通常 5–10 个候选页
→ 按需读取 Visual Asset
→ 依据表达意图、信息关系、容量、布局、风格、可变性与排除项重排
→ 得到通常 1–3 个候选
→ 按需渲染和视觉确认
```

候选数量是导航建议，不是硬性配额。同页复用已满足要求时无需为凑数量额外搜索。

视觉检索不受内容主题限制。例如新内容讲三个产品能力，可参考制造业 PPT 中三个并列优势的视觉结构；原制造业内容不得随之混入新页面。

### 8.3 主参考、辅助参考与无命中处理

每页确认一个主视觉参考；辅助参考仅贡献指定局部：

```yaml
visual_references:
  primary:
    ppt_id: ppt_000001
    slide_id: 12
    source_version: v1
    use_for: [overall_composition, hierarchy, whitespace]
  secondary:
    - ppt_id: ppt_000002
      slide_id: 23
      source_version: v1
      use_for: [data_expression]
  adaptation_plan: 统一强调色与字体，替换原文，删除客户装饰
  rendered_visual_confirmed: false
```

禁止无约束拼接多个风格。参考结构可以适配，但必须说明保留什么、变化什么以及为何仍符合表达任务。

若找不到合格视觉参考，依次扩大跨主题搜索、调整内容密度或拆页，再重新检索。仍无合格参考或库不可访问时，明确报告缺少视觉资产并请求补充可读取的原 PPT；该页保持未完成状态。**不得静默降级为自由生成，不得把“搜过但无结果”视为视觉门槛通过。**

## 9. 按需渲染与参考确认

视觉确认优先直接读取候选资产已保存的 `<ppt_id>/previews/slide-XXX.png`。Preview 必须与 `ppt_id + slide_id + source_version + source_sha256` 对应，并且是该页 Visual Asset 入库分析时实际观察的渲染结果。只有 Preview 缺失、损坏、身份校验不一致、质量不足以完成判断，或确有重新验证需要时，才从对应原始 `source.ppt` / `source.pptx` 重新渲染真实页面；重新渲染确认有效后应同步更新标准 Preview。临时比较图和 QA 图仍在任务结束后清理。

视觉确认同时查看：参考图、Visual Asset、客观结构、当前 Slide Brief 和全局视觉规范。确认真实构图、层级、容量、留白、品质和可适配性。Wiki 描述匹配但实际页面不合适时，必须更换候选。

通过后才把以下材料送入 HTML 设计：

```text
已确认的主 / 辅参考图
+ 对应 Visual Asset
+ 已补齐内容与来源的 Slide Brief
+ Global Visual Spec
+ 明确的适配方案
```

## 10. HTML 设计、视觉 QA 与用户预览

### 10.1 HTML 设计

理解并迁移参考页的构图、信息层级、空间关系、色彩角色、字体层级、视觉密度、留白与节奏，使用新内容设计页面。不得机械复制原客户文案、logo、图片和无关装饰。

- 每页使用固定尺寸画布，默认 16:9；整套一致。
- 每个需转换为 PPT 对象的元素使用稳定且页内唯一的 `data-ppt-id`。
- 预览边框、阴影、缩放和导航属于预览 UI，不进入实际 slide 内容或导出几何信息。
- 禁止把整页做成一张背景图片来规避设计或可编辑性要求。

```html
<section class="slide" data-slide-id="4">
  <h1 data-ppt-id="title">当前页面标题</h1>
  <div data-ppt-id="feature-1">当前内容</div>
</section>
```

### 10.2 HTML 视觉 QA

逐页渲染截图并实际检查，同时查看整套页面节奏：

- 是否实现 Brief 的主张、信息关系与表达重点，内容及数据是否正确。
- 是否保留参考设计的有效构图、层级、留白和品质，而非仅套上相似颜色。
- 是否存在溢出、遮挡、拥挤、失衡、错误换行、过小字体、对比度不足或无意义装饰。
- 是否统一字体、色彩角色、边距、图表和图标语言，避免逐页风格拼盘。
- 当前文字数量、图片裁切与图表标注是否真实适合画布。

```text
HTML → 截图 → 视觉 QA
                 ├─ 失败 → 修 HTML / CSS 或返回参考匹配 → 再截图
                 └─ 通过 → 用户预览
```

不能以 DOM 没有溢出代替视觉判断。无渲染或视觉检查能力时标记未验证，不能声称通过 QA。

### 10.3 HTML 必须具备两种演示模式

HTML 既是所有格式的设计与预览母版，也可以作为最终交付格式。交互参照 PowerPoint 的普通预览和全屏放映体验，不要求复制编辑器功能或工具栏。

**普通预览模式：左侧缩略图，右侧当前页。**

- 左侧按页序显示整套幻灯片缩略图与页码，独立滚动；缩略图必须反映当前实际页面，不能用文字占位卡代替。
- 点击缩略图，右侧切换到对应幻灯片；选中项明显高亮，并随翻页自动滚动到可见位置。
- 右侧一次显示一张完整幻灯片，在可用区域等比例缩放、居中适配，保持固定画布与内容布局，不裁切、不拉伸。
- 提供上一页、下一页、当前页 / 总页数和进入全屏放映按钮。上下连续滚动的长网页不能替代此模式。
- 缩略图可以使用同源画布的缩放副本或当前成品生成的预览图；不得让副本的重复 ID、隐藏状态或缩放干扰主画布导出。交付所需缩略图属于成品 UI 资源，不作为历史 PPT 资产库的长期截图。

**全屏放映模式：只展示当前幻灯片。**

- 从当前页进入放映，隐藏左侧缩略图、普通工具栏、页面边框与其他预览 UI；幻灯片等比例适配屏幕，不同比例的剩余区域使用中性留边。
- 支持上一页 / 下一页按钮，以及方向键、PageUp / PageDown、Space 翻页；Home / End 跳到首尾页，Esc 退出全屏并回到当前页的普通预览。
- 提供简洁的切页效果，例如短时淡入或水平滑动；不得采用干扰内容阅读的复杂特效，并尊重减少动态效果的系统偏好。
- 首尾页不越界，默认不自动播放或循环。输入框正在接收输入时不拦截翻页按键。
- 全屏需由用户点击触发；浏览器不支持或拒绝全屏时，退化为占满当前窗口的放映模式，并保留明确的退出按钮。

两种模式共用同一套幻灯片内容和当前页状态，不另做一套可能与导出版不一致的页面。演示动画只属于查看体验，不得改变导出内容或造成半透明、位移中的页面被导出。

### 10.4 返回 HTML，等待用户确认

HTML 视觉 QA 通过后，必须实际返回可打开的 HTML 文件或预览链接；不能只给截图、描述或声称已提供预览。优先交付可直接打开的 HTML，必要资源随包提供，避免依赖本机临时路径。

本阶段将当前演示版本持久化并标记为 `awaiting_user_confirmation`，随后**结束本次执行，不保留运行中的子任务、Agent 或后台等待进程**。这里的“等待用户确认”是演示状态，不是常驻任务。到此停止格式转换，不能提前生成 PPT / PDF。

用户后续明确确认当前版本时，Agent 读取**同一演示**及其当前版本状态，从对应阶段继续处理；确认其仍为待确认的当前版本后，记录 `confirmed_design_version` 和要求的输出格式。无需保持此前的执行进程，也不得因此新建演示任务。若格式尚未明确，再询问需要 HTML、PPT、PDF 或多个格式；用户已明确指定格式时不重复询问。仅交付 HTML 时无需额外转换，但仍保留用户审阅和修改流程。

收到修改要求时，Agent 读取**同一演示**的当前状态并更新同一 HTML，从对应阶段继续处理，重新检查受影响页面及交互，再返回用户预览；返回后再次结束本次运行并保持 `awaiting_user_confirmation`。无需保持此前的执行进程，也不得因此新建演示任务。改动内容或视觉后，旧版本确认不自动适用于新版本；必须重新取得确认。确认后冻结版本，各格式都从该版本生成。

### 10.5 HTML 交互 QA

除逐页视觉 QA 外，实际验证缩略图内容与页序、点击定位、选中高亮、前后翻页、首尾边界、键盘导航、全屏进入 / 退出与回退、模式切换后当前页保持、窗口变化下等比例适配、字体和图片加载。不能只检查静态截图便声称交互通过。

检查预览、全屏和导出画布的内容一致性。导出时关闭动画并恢复稳定样式；预览缩放不能影响原始画布尺寸。

## 11. DOM bbox 与 Render IR

本节仅适用于用户确认 HTML 后要求输出 PPT 的分支。HTML 交付和 PDF 导出不要求先生成 Render IR，也不调用 COM。

### 11.1 从定稿 HTML 提取

等待字体、图片及布局稳定后，对所有 `[data-ppt-id]` 元素调用 `getBoundingClientRect()`。

使用独立、无预览缩放的导出画布逐页布局；不能直接测量预览中 `display: none` 的非当前页，也不能把左侧缩略图副本纳入提取。

坐标必须相对于实际 slide 画布：减去画布偏移，消除预览缩放，不直接使用视口坐标。保存画布尺寸、元素 bbox、层级与稳定 ID，避免父子容器重复绘制。

同时提取 computed font family、font size、font weight、颜色、背景、边框、圆角、文本对齐、透明度、图片路径、object-fit / crop，以及必要的文本段落与样式、旋转、分组和层叠顺序。

### 11.2 Render IR 是确定性的转换协议

HTML 不直接驱动 COM。先形成可检查、可复现的中间表示：

```json
{
  "schema_version": "1.0",
  "slide_id": 4,
  "design_version": "approved-v1",
  "size": {"width": 1920, "height": 1080, "unit": "px"},
  "elements": [
    {
      "id": "title",
      "type": "text",
      "bbox": [120, 80, 1000, 120],
      "z_order": 1,
      "text": "当前页面标题",
      "style": {"font_family": "Arial", "font_size_px": 64, "color": "#161616"}
    }
  ]
}
```

IR 应保存输出所需的文本、形状、线条、图片、图表、表格、分组、样式和资源信息；上例仅示范最小结构。坐标与单位必须明确，转换到 PPT 点单位时按目标画布等比例映射，不猜测尺寸。

导出前验证 ID 唯一、bbox 有效、资源存在、画布比例一致，以及样式是否被目标 Renderer 支持。无法直接表示的 CSS 效果须显式转换或局部降级并记录，不能默默丢失。

## 12. 按目标格式输出与最终 QA

### 12.1 Renderer 只执行设计

仅在当前 HTML 版本已获用户确认且用户要求 PPT 时执行。

使用 PowerPoint COM 或其他支持原生 shape 的 PPTX Renderer，把 Render IR 映射为 PowerPoint 对象：

| HTML / IR 元素 | PPT 输出 |
|---|---|
| text | TextBox，尽量保留段落和局部文字样式 |
| rect / shape | 原生 Shape |
| image | Picture，保留裁切关系 |
| line | 原生 Line |
| chart | 可实现时使用原生 Chart |
| table | 可实现时使用原生 Table |

优先保留可编辑性；不支持的复杂效果可以局部图像化，但必须记录范围、原因和可编辑性损失，禁止以整页栅格化替代正常转换。

COM 不决定布局、字号或卡片位置。转换中的字体映射、文本内边距、行距、裁切、层叠顺序和坐标映射必须明确，避免依赖 PowerPoint 默认设置改变设计。

### 12.2 最终 PPT QA

```text
PPTX → PowerPoint 原生渲染 → 临时逐页 PNG
→ 对照已确认的 HTML 版本进行视觉 QA
→ 修复 → 重新渲染受影响页面 → 复核全局一致性
```

逐页检查 text overflow、字体替换、换行变化、元素偏移、裁切差异、图表和表格差异，同时核对页数、顺序、内容、数据、来源及主要对象可编辑性。

失败按原因返回：

- 设计问题：修 HTML，重新走视觉 QA、必要的预览与 IR 提取。
- 转换问题：修 Renderer / mapping，并重新生成和渲染。
- 字体问题：修 font mapping，必要时回 HTML 校准并重新验证。

禁止在 COM 中临时随意重排来掩盖转换错误。无法执行 PowerPoint 原生渲染时，明确报告最终兼容性未验证；替代渲染可用于辅助检查，但不能冒充原生 QA 通过。

### 12.3 PDF：从已确认的 HTML 直接导出

```text
已确认 HTML → 稳定的打印布局 → 浏览器打印 / PDF 导出
→ PDF 逐页渲染 → 对照已确认 HTML 进行 QA → 交付 PDF
```

- 不先生成 PPT，不通过 PPT 转 PDF，不要求 DOM / Render IR 或 COM。
- 使用专用打印样式：隐藏缩略图、工具栏、按钮、预览背景和 UI 边框；显示全部幻灯片，而非只打印当前页。
- 取消预览缩放、固定定位、滚动容器裁切、动画与过渡；按页序将每张幻灯片放在独立 PDF 页面。
- 设置与幻灯片相同的页面尺寸和方向，默认沿用 16:9；零页边距、关闭浏览器页眉页脚、保留背景色与背景图，防止额外空白页或一页被拆成多页。
- 字体、图片和图表加载完成后再导出。尽量保留可选择文字、矢量内容及有效链接，不默认把整页截图拼成 PDF。
- PDF 不保留翻页动画和交互控件，导出其稳定的完整内容状态；翻页体验由 PDF 阅读器提供。
- QA 核对页数、顺序、页面尺寸、文字换行、字体、图片、背景、裁切、空白页和 UI 泄漏，逐页与已确认 HTML 比较。修打印样式或导出设置后重新导出；若需改动设计本身，回到 HTML QA 和用户确认。

### 12.4 HTML：保留可交互的最终成品

用户选择 HTML 时，交付已确认版本及全部必要资源，保留左侧缩略图预览、右侧当前页、翻页和全屏放映。不得只交付截图或移除交互的静态长页面。

在实际交付文件或链接上完成最终视觉与交互 QA，检查资源路径可用，不能只验证开发环境中的页面。优先自包含 HTML；无法内嵌的资源使用可随交付移动的相对路径并一起打包。

## 13. 决策记录、交付与完成标准

每份新演示保存全局资产采用记录；每页保存内容来源、视觉来源、复用决定及 QA 状态。记录可随项目存放，不向长期 Wiki 写入临时截图。

```yaml
slide_no: 4
global_asset_refs: []
content_sources: []
content_match: high_or_partial_or_none
same_slide_visual_evaluation:
  status: evaluated_or_not_applicable
  reason: ''
visual_references: {}
visual_confirmation: pending_or_pass_or_fail
html_qa: pending_or_pass_or_fail
preview_design_version: null
preview_status: awaiting_user_confirmation_or_changes_requested_or_confirmed
confirmed_design_version: null
requested_formats: []
html_interaction_qa: pending_or_pass_or_fail
render_ir_version: null
ppt_qa: not_requested_or_pending_or_pass_or_fail
pdf_qa: not_requested_or_pending_or_pass_or_fail
editability_limitations: []
```

交付前必须确认：

1. 已检索 Global Asset，并明确采用的结构或自行规划的原因。
2. 每页有 Brief，已执行 Content Retrieval；新增内容有可靠依据，历史内容有来源与时效处理。
3. 每个内容高度命中页都先评估了原页视觉，并记录复用或换参考的原因。
4. **每页都有经真实渲染确认的 Visual Asset 主参考；没有页面因内容无匹配而跳过视觉环节。**
5. HTML 逐页视觉与交互 QA 通过，已实际返回用户预览；转换 PPT / PDF 前用户已明确确认当前版本。
6. 按用户选择分支：PPT 的 DOM / IR 对应已确认设计且尽量原生可编辑；PDF 直接从已确认 HTML 导出；HTML 保留两种演示模式。
7. 所选格式均通过对应的最终 QA；未请求的格式无需生成，任何未验证项不得报告为通过。
8. 提供用户要求的 HTML / PPTX / PDF、必要资源和来源与 QA 摘要；按交付需要保留 IR，清理临时参考图和 QA 截图缓存，不删除成品依赖的缩略图资源。

## 14. 系统定义

> 先把整份 PPT 的讲述经验、逐页内容知识和逐页视觉经验分别建成可检索资产；生成时先借鉴全局讲法，再为每页补齐可信内容并选择经过视觉确认的参考，完成具备缩略图预览和全屏翻页能力的 HTML，通过 QA 后返回用户。用户明确确认后，按需保留 HTML、通过 DOM / Render IR 生成可编辑 PPT，或直接由 HTML 导出 PDF，并验证对应格式的最终成品。

## 15. Issue List

> 执行中暴露的问题与已确认的处理答案**已独立成文**：[Issue List](agents-issues.md)。**动手前先查该表**（每次动手都查，不等"遇到同类情况"才查），按「答案」执行，不要重复试错。
>
> 格式细则（目录与命名、registry 字段、两份 front-matter、两层 INDEX 固定表头、状态矩阵、work/ 命名与清理）见 [宪法附录](agents-appendix.md)。拆出原因见 [agents-issues.md](agents-issues.md) 中的 Issue 013：本文件体积超过工作区指令预算（65536 字节），Issue 表原位于文末，加载时会随尾部一起被截断而整段读不到。**新增 Issue 一律写进 [agents-issues.md](agents-issues.md)、新增格式规则一律写进 [agents-appendix.md](agents-appendix.md)，都不再追加到本文件末尾。**
