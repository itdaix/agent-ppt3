# AI PPT Agent（三类资产驱动）

把历史 PPT 收成可检索的资产，之后一句话生成新演示：**先借鉴讲法，再补齐内容，再挑经过视觉确认的版式**。

## 目录里有什么

| 文件 | 作用 |
|---|---|
| [agents.md](agents.md) | **宪法主体**：角色、九个命令、三类资产、生成流程、QA 与完成标准 —— 规定**做什么**。**放进工作区当指令用** |
| [agents-appendix.md](agents-appendix.md) | **宪法附录**：结构与字段的唯一口径 —— 目录与命名、`registry.json` 字段、两份 front-matter、两层 INDEX 固定表头、状态矩阵、`work/` 规则、入库后强制自检、历史偏差清单 —— 钉死**长什么样** |
| [agents-issues.md](agents-issues.md) | **Issue 表**：已踩过的坑与定论（26 条）—— 规定**按哪个口径执行**。动手前先查，按「答案」执行 |
| [demo.html](demo.html) | **演示壳基准**：翻页放映型 HTML —— 1600×900 画布 + 缩略图侧栏 + 工具栏（页码 / 来源 / 放映）+ 放映模式 + 打印分页 + `?shot=N` 截图模式 |
| [demo.pptx](demo.pptx) | `demo.html` 的成品导出版，用来看交付形态长什么样 |

> ⚠️ **三份宪法文件不可分割**：必须同时放进工作区、同时读取，缺一份即视为宪法不完整，不得据此执行入库或生成。
> 冲突裁决顺序：**Issue 表 > 附录 > 主体**（后写的裁定优先）。

## 怎么开始用

1. **建工作区**：新建一个目录，把三份宪法 `agents.md`、`agents-appendix.md`、`agents-issues.md` 放进去（`demo.html` 可选，作壳基准）。这个目录同时也是资产库所在地。
2. **开一个会话**，以该目录为工作区（`agents.md` 会作为工作区指令自动加载）。
3. **丢素材入库**——文件或整个文件夹都行：
   ```text
   ingest "产品介绍.pptx"
   ingest "待入库"
   ```
   第一次入库会自动建好 `ppt-library/` 目录、登记表与索引。
4. **让它做演示**（三种模式只差「库内哪一侧允许参考」）：
   
   ```text
   do "2026 年产品路线图，12 页"
   do-content "…"      # 只用库里的内容，版式重新设计
   do-visual "…"       # 只借库里的版式，内容由你提供
   ```
5. **看它返回的 HTML**（左侧缩略图 + 全屏翻页放映）。**确认这一版之后**再说导出：
   
   ```text
   export
   ```
   按需要产出 HTML / PPTX / PDF。
6. **越用越厚**：资产落在 `ppt-library/`，一页一个 `slide-XXX.md`，全库入口是 `ppt-library/INDEX.md`；查一查用 `lint`，删一份用 `remove`。

## 命令一览

| 命令 | 中文 | 作用 |
|---|---|---|
| `ingest` | 入库 | 文件或文件夹入库；PPT 收全三类资产，非 PPT 自动只收内容 |
| `ingest-content` | 内容入库 | 只收内容资产 |
| `ingest-visual` | 视觉入库 | 只收版式资产（仅 `.ppt` / `.pptx`） |
| `do` | 制作 | 内容与版式都参考库内资产 |
| `do-content` | 内容制作 | 只用库内内容，版式自主设计 |
| `do-visual` | 视觉制作 | 只借库内版式，内容来自你的材料或外部可靠来源 |
| `lint` | 巡检 | 只报告索引 / 链接 / 孤岛页问题，加 `--fix` 才修 |
| `remove` | 移除 | 按 `ppt_id` 删除整份资产包 |
| `export` | 导出 | 把已确认的那一版导出为 HTML / PPTX / PDF |

## 换成你自己的壳

`demo.html` 既是示例也是基准。想用自己的壳：替换这个文件，或另存一个新名字 —— 壳**按形态识别，不按文件名**，必须保留的结构要点见 [agents-issues.md](agents-issues.md) 的 Issue 006（演示壳）/ Issue 007（杂志壳，长页滚动型）。

> 详细规则一律以 [agents.md](agents.md) 为准；**格式细则（目录 / 命名 / 字段 / 表头 / 状态 / 自检）以 [agents-appendix.md](agents-appendix.md) 为准**；踩过的坑先查 [agents-issues.md](agents-issues.md)。
