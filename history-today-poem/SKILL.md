---
name: history-today-poem
description: "当用户要求制作/补做「历史上的今天·一首古诗」公众号草稿，或排查这条链路时使用。"
---

# 历史上的今天 · 一首古诗（明天稿 · 精简版）

## 一句话分工

Agent 做内容（算配额、联网查史料、写作、选诗、写背景、打标签）→ `history_post.py` 做投递（校验、排版、出图、上传、建草稿、写台账）。脚本不查史料、不选诗、不编任何文字；只跑脚本而没有当日 JSON 会直接报「找不到当日数据」。

**每天出 3 篇（3 个事件、3 首诗），进草稿箱，绝不自动发布。** 三篇分别落盘为 `data/history/<发布日>.json`、`<发布日>-2.json`、`<发布日>-3.json`（一篇一个文件），三份写完后跑一次脚本，会自动全部处理（一天多篇：`<date>-2.json` → `output/history/<date>/2/`，互不覆盖）。

正文只有两块文字：**事件经过（summary）** 和 **诗的背景（background）**。不写「历史意义或影响」，不写诗的「赏析」和「名词解释」。

## 边界（先读）

- 项目根目录 `ROOT`：与本 skill 配套的 **`app/`** 目录。LightVela 云端为 `~/history-today-poem/app`。所有命令都在该目录下执行，模块是平铺 import 的，换个目录调用会 `ModuleNotFoundError`。
- （仅原机 Windows）旧项目 `C:/Users/Lin/WorkBuddy/2026-11-09-17-49-03/daily-poem-news`、旧 skill `daily-poem-news` 与 `history-today-poem-cloud`、以及「今日十闻」链路：**一律不读不改，原样保留**。本 skill 只在 `app/` 下工作。
- 本 skill 是这条链路的**现行版本**。旧 skill 里的 07:00、「今天」、`link` / `poem_note`、「一首旧诗」栏目名、每天 1 篇等描述都已过时，不要照着做。

## 日期与版面口径（最容易错）

- 定时任务每天 **17:30（北京时间）** 跑一次，一次产出**明天发布**的 3 篇稿子。「历史上的今天」的日子 = **明天**（TARGET），选题、数据文件名、日期行全按 TARGET。
- **文章标题 = 正文标题行 = 封面主标题 =「历史上的今天 | {事件标题}」**，三处一字不差，**标题里不出现事件发生日期**。由脚本生成，JSON 里不写 `title` 字段。
- **`event.date` 仍必填**，是事件实际发生的完整公历日期，**月日必须与发布日一致**；脚本强校验，不一致直接报错不出稿。它进台账和日志，不进标题。只写年份、或日期含糊不清的事件，换题。
- **中国古代稿可以用农历**：史料只记农历时，换算成公历填 `event.date`（月日仍须等于 TARGET），并填 `event.lunar_md`（如「三月十五」）供脚本核对换算；`event.date_label`（如「清乾隆三十年三月十五」）只用于日志与台账。农历只改表述，不改「历史上的今天」按公历月日的口径。
- **正文顶部只有一行日期**：`公历 · 星期 · 农历 · 节气`，例如「2026年11月21日 · 星期六 · 农历十月初七 · 立冬后第15天」。农历排在星期后面，由脚本用 cnlunar 换算；cnlunar 不可用时自动省略农历与节气，不编造。
- **正文没有「历史上的今天 · 一首古诗」这行栏目名**，已经取消；栏目名只保留在封面底部。
- **字号口径（一套）**：正文 16px；「历史上的今天」红色加粗 19px，其后「| 具体事件」墨色加粗 19px；诗题《…》墨色加粗 19px；小节名「一首古诗」红色加粗 19px；正文第一段开头的日期 19px（只放大段首那一个日期，其余正文不动）。顶部日期行是唯一的淡灰小字（13px）。
- 不传 `--date` 时，`history_post.py` 自动用**北京时间今天 + 1 天**（固定 UTC+8，不依赖运行机器时区，也不需要 tzdata）。Agent 侧算日期同样用北京时间，别用本机时区。
- 只有手工补做/重做某日才传 `--date YYYY-MM-DD`，那时它代表那个发布日。

## 铁律

1. `WX_PUBLISH_MODE` 恒为 `draft`，绝不自动发布；发布前必须人工复核。不改 `.env` 里的公众号账号与配图配置。
2. **真实性高于文采**。不编造史实、年份、人名、数字、引语，不编造诗题、作者、朝代、诗句、典故、字词训诂和诗人背景。至少 2~3 个来源交叉核对；不用存疑、传说、单一来源的说法；不碰政治敏感、定性争议与主权表述有争议的内容。有对不上的地方就换题，仍不行就中止告警，**宁可今天不出稿**。中国近现代**非政治性**题材（铁路、报刊、大学、考古、工程、人物生卒）正常选。
3. 古诗异文按权威通行本**整首定稿**，不混拼、不改写、不拼接；逐句原文必须核实。
4. **不解释为什么选这首诗，也不写诗与历史事件的连接或比附**：JSON 里没有 `link`、`poem_note`，正文里没有「为什么是这首诗」小节。诗的部分只讲诗本身（见 `references/writing-rules.md`）。
5. 文章标题、正文标题行、封面主标题统一为「**历史上的今天 | {事件标题}**」，由脚本生成，JSON 里**不写 `title` 字段**；`event.date` 必填但不进标题。
6. 正文只有两块文字：**事件经过**（`event.summary`）和**诗的背景**（`poem.background`，小节名固定为「**背景**」）。**不写「历史意义或影响」「赏析」「名词解释」三块**。文末最后一句固定为「**本文由人工策划选题，AI辅助内容生成，再由人工审核润色完成。**」，由脚本生成，不用写进 JSON。
7. **一个发布日建 3 篇草稿（3 个事件）。** 每篇产物在 `output/history/<日期>/`（第二、三篇在子目录 `2/`、`3/`），`result.json` 里存 `draft_media_id`。动手前先看是否已有 `draft_media_id`；确实要重做，脚本会更新那一份、不新建重复稿。不静默产生重复稿、不擅自删草稿。**三篇不得重复同一事件或同一首诗。**
8. **选题按框架走**：先跑 `python history_post.py --quota` 读台账算本周配额，再读避重黑名单。**每天 3 篇、每周约 15 篇中国 + 6 篇国际（保持 5:2）**，按周不按天；**优先选中国相关、有意义但大众不太关注的小事件**；读者新鲜度优先；6 个题材池与 5 种气质相邻两天不重复；**单纯的「某人生日/忌日」事件一票否决**（重大贡献人物整周年诞辰/忌日除外，见 `references/selection-framework.md` 附加硬门槛；详见 `references/selection-framework.md`）；5 项各 0~2 分，**低于 6 分换题，宁可不出稿**。详见 `references/selection-framework.md`。
9. **标题优先用大众熟知的符号或俗称**，别拿生僻的原名当标题：写「米老鼠第一次亮相」，不要写「《威利号汽船》首映」；写「第一封电子邮件发出」，不要写「ARPANET 节点间传输实验」。符号要准确、能对应到这件事实，不能为了通俗而写错。
10. 文字平实、口语化、去 AI 味，**宁短勿长**；正文只写事件经过与诗的背景两块，写完能删就删。详见 `references/writing-rules.md`。

## 命令速查（在 ROOT 下执行）

LightVela 云端只有 `python3`，没有 `python`；下表命令一律用 `python3` 执行。

| 目的 | 命令 |
| --- | --- |
| 看最近 14 天台账与本周配额缺口（stdout 只输出 JSON） | `python history_post.py --quota` |
| 按意境词检索候选古诗（stdout 只输出 JSON，`has_notes` 表示有现成赏析资料） | `python history_post.py --suggest "苍凉,江月,故垒,兴亡" --limit 12` |
| 本地预览，不碰公众号（目标日＝明天） | `python history_post.py --dry-run` |
| 出图 + 进公众号草稿箱（目标日＝明天，不发布） | `python history_post.py` |
| 补做 / 重做指定发布日 | `python history_post.py --date 2026-11-13 --dry-run` |
| 用指定数据文件自检 | `python history_post.py --date 2026-11-13 --file data/history/_sample.json --dry-run` |

脚本默认读 `<发布日>.json`、`<发布日>-2.json`、`<发布日>-3.json` 三份数据（一天 3 篇；`--file` 可指定一个或多个文件，逗号分隔）。产物在 `output/history/<发布日>/`（第二、三篇在子目录 `2/`、`3/`）：`preview.html`、`wechat.html`、`cover.jpg`、`inline.jpg`、`article.json`、`result.json`、`run.log`。正式投递成功后脚本写 `data/selection-log.json` 台账（一天一行）；`--dry-run` 不写台账。

## 数据格式（基准 `data/history/_sample.json`）

写入 `data/history/<发布日>.json`。段落内换行在 JSON 里写成 `\n`。

| 字段 | 说明 |
| --- | --- |
| `date` | 发布日 `YYYY-MM-DD`（明天）；脚本会强制对齐 |
| `column` | 可选，默认「历史上的今天 · 一首古诗」，只用于封面底部，不进正文 |
| `selection` | **必填**选题标签：`region` 国内/国外、`pool` 6 池之一、`mood` 5 气质之一、`device` 8 笔法之一、`score` 5 项合计 ≥6，可选 `scores` 明细。见 `references/selection-framework.md` |
| `event.date` | **事件实际发生的完整公历日期** `YYYY-MM-DD`，**必填**，月日必须等于发布日；不进标题 |
| `event.date_label` / `event.lunar_md` | 可选，中国古代农历稿用：前者是农历/年号文字（进日志台账），后者是农历月日，脚本核对换算 |
| `event.title` | 事件标题，平实陈述，**必填**，进标题行；**优先用大众熟知的符号/俗称** |
| `event.summary` | 事件经过，200 字上下，只写核实过的事实，**必填** |
| `event.sources` | `[{"name","url"}]` 2~4 条；第一条 http 链接用作「阅读原文」，文末最多显示 6 行 |
| `poem.title` / `poem.author` | **必填** |
| `poem.dynasty` / `poem.tags` | 朝代、3~6 个具体取景词（配图取景用），可选 |
| `poem.lines` | 逐句原文字符串数组，**必填**，不得编造 |
| `poem.background` | 诗人写诗时的处境与缘起，对应正文「背景」，**必填** |
| `digest` | 摘要 ≤120 字，缺省取 `event.summary` 第一段 |

**一天 3 篇的文件命名**：`data/history/<发布日>.json`（第一篇）、`<发布日>-2.json`（第二篇）、`<发布日>-3.json`（第三篇）。三份都写完再跑脚本，一次全部投递；只写了一篇就只投一篇，不凑数、不降门槛。

正文不写「历史意义或影响」「赏析」「名词解释」，JSON 里也不要这三个字段（`event.impact`、`poem.interpretation`、`poem.words`）。旧格式里的 `event.year` 也已不再使用（日期统一走 `event.date`）；`title`、`link`、`poem_note` 都不要写。日期行、标题、封面、文末落款都由脚本生成。

`validate()` 查必填字段、`event.date` 格式、`event.lunar_md` 换算、`selection` 标签与分数、`poem.lines` 类型；`run()` 另校验事件日期与发布日的月日一致。**不做史实与诗句真伪校验**——核实责任在 Agent 这一侧。

## 运行前检查

1. `.env` 存在且 `WX_APPID`、`WX_APPSECRET` 非空。不满足时只回报一句「公众号密钥尚未配置，今日跳过」，不运行程序、不报错、不重试。
2. `python -c "from config import CFG; print(CFG.wx_ready, CFG.wx_publish_mode, CFG.image_provider)"`，确认 `draft`。不要把密钥打进日志。
3. `python history_post.py --quota` 能出 JSON；`data/selection-log.json` 存在且可读。
4. `data/history/<发布日>.json` 存在且 JSON 合法，字段齐全、`selection.score ≥ 6`、`event.date` 月日与发布日一致再跑。
5. 中文字体与 cnlunar 可用（封面永远用 Pillow 本地绘制）；微信 API 与图库网络可达；公众号后台 IP 白名单包含本机出口 IP。

## 失败速查

| 现象 | 处理 |
| --- | --- |
| `找不到当日数据：...` | 先按 SOP 检索写作落盘，再跑脚本 |
| `当日数据缺少必填字段：...` | 补齐 `selection`、`event.date/title/summary`、`poem.title/author/lines/background` |
| `event.date 必须填事件实际发生的完整公历日期` | 只写年份或格式不对；查清具体月日再填 |
| `event.lunar_md ... 换算出的农历 ... 对不上` | 农历换算错了：核实史料与换算依据，改 `lunar_md` 或改 `date`；不要删字段绕过 |
| `事件日期 ... 与目标发布日 ... 的月日不一致` | 选题错了：换一件当天发生的事，或改回正确的发布日；不要改校验代码绕过 |
| `选题打分 ... 低于 6 分门槛` | 按框架换题，宁可今天不出稿；不要为凑分改打分 |
| `selection.xxx 必须是其中之一` | 标签越界，按 `selection-framework.md` 的枚举改 |
| `40164` | 出口 IP 不在公众号白名单；取 `run.log` 里的 IP 去后台加白名单（可填 /24 网段）。重试无用，别做无谓重跑 |
| `40007` | 封面误用临时素材；封面必须走 `material/add_material` 拿永久 media_id |
| `找不到中文字体` | 在 `.env` 设 `FONT_PATH` 指向中文字体 |
| 日期行没出现农历 | cnlunar 缺失或当天换算失败；云端修复：`pip3 install cnlunar --break-system-packages`（PEP 668 环境必须带该参数），装完重跑自检。不要手写农历 |
| | 正文配图上传失败 | 代码会降级为无图正文并记日志，不阻断草稿，不必重跑整篇 |
| 改稿报 40007 invalid media_id | 原草稿多半已在后台被删：把 output/history/<日>/<槽>/result.json 移走再跑一次，脚本建新草稿；旧草稿若还在草稿箱，提醒用户手动删 |

数据 JSON 支持三个换图/换封面开关：`image_variant`（同一检索词候选往后取）、`image_query`（覆盖 Unsplash 检索词）、`cover_style`（指定封面风格，如 A_夜观星河）。坑：① 诗句连带词（杨柳/春风）会把山水诗误判进花木池，判定结果看日志 `[配图] … 主题=…`；② 夜月主题夜景图平均亮度天然低，阈值已放宽到 40，全偏暗时兜底选最亮一张。
| 草稿箱出现重复稿 | 用 `draft/batchget` 回查，必要时 `draft/delete`；**先问过用户再删** |
| 其他报错 | 读 `run.log` 定位，能修就修一次后重跑；仍失败如实回报，不伪造成功 |

## 配图源（2026-11 更新）

- 链路（stock 模式）：Unsplash → GA纳速云镜像（Pexels 免 key，中文词检索） → Pexels → Pixabay → Picsum 风景池 → 本地水墨，逐级回退。
- GA 纳速云镜像（https://ga.nasuyun.com/s/photos/<中文词>）是 Pexels 的镜像站，搜索页从云端可达、无需 key；代码用 poem.tags 的中文取景词搜索，从页面抽 Pexels photo id 后直连 images.pexels.com CDN 下载。镜像站失效时自动落到下一个源，不算事故。
- Unsplash 候选池已扩到 30 张，起点按发布日轮换，同主题不再天天同一张；Pexels / Pixabay 同样是 30 张 + 日期轮换。
- Pexels 需 `PEXELS_API_KEY`、Pixabay 需 `PIXABAY_API_KEY`（均免费申请，填在 `.env`）；未填时自动跳到下一个源，不报错。Openverse / Wikimedia 在 LightVela 云端被网络代理挡掉，不可用。
- **Pixabay 的 `pixabay.com/get/*` 热链在云端必被限速（429）**：代码改从 `previewURL` 推导 `cdn.pixabay.com` 正式 CDN 大图（`_150.` → `_1280.`），不限速。搜索 API 正常，别改回热链下载。
- **图源主题判定看诗的原文而非 tags**：`artwork.py` 的 `_poem_theme` 扫诗题+诗句+tags。注意诗句里的「杨柳」「春风」这类连带词曾把山水诗误判进「花木」池换出樱花图——已改为只有出现明显花木意象（花/梅/荷/桃/李/杏/梨/蕊/芳/菲）才判花木。改 tags 换图不起作用时，先查 `_poem_theme` 的判定结果（run 日志里 `[配图] … 主题=…`），再决定是调 tags 还是修判定。

- 定时任务：每天 **17:30（北京时间）** 触发一次投递，产出**3 篇**明天发布的稿子。另有配套的每日简报任务（17:40，检查当日运行状态/出稿情况/近 7 天台账并回报用户，job 名「古诗任务每日简报」）；两个任务的投递目标均为 origin,bot-chat。调整调度时间或启停请用定时任务工具，不要改代码。

## 详细规则

- 选题框架（四门槛、中国 5 : 国际 2 配额、读者新鲜度、题材池与气质轮换、5 项打分、台账）→ `references/selection-framework.md`
- 写作要求（事件经过、诗的背景、标题符号化、真实性底线）→ `references/writing-rules.md`
- 完整 SOP（算配额、选题、核对、落盘、投递、回报、踩过的坑）→ `references/workflow.md`
- **用户指定选题优先于配额，但事件日期先核再承诺**：用户明确给定事件/配诗时，先联网核实事件的确切月日是否等于 TARGET；不一致时不落盘、不改校验，向用户说明并给出选项（改做月日吻合的发布日，或换同日其他事件）。跨源日期冲突时以原文报道日期与权威多源共识为准，不用单一杂志文章的日期硬套校验。
- **已投草稿可更新不重复**：重跑 `--date <发布日> --file <改过的 json>` 会沿用原 `draft_media_id` 更新同一份草稿（先 dry-run 验证再正式跑）；换诗时两篇互换 poem 块即可，`poem.tags` 按新文章题材重写取景词。跑完在回报里注明「已进草稿箱，未自动发布」。
- 从用户提供的预览 HTML 抽内容时，只取事件经过、诗、背景、来源四块文字转成 JSON，HTML 的字号、栏目名等排版由脚本生成，不要照搬。
- 封面风格与改稿交付细节 → `references/cover-design.md`
