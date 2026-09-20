[English](./README.md) | **中文**

# Auren

**高度个性化的 AI 伴侣 —— 独立开发的全栈 iOS 应用。**

Auren 是一个拥有长期记忆、健康监测、深度情感感知的私人 AI 伴侣应用，搭载手工打造的暗色哥特浪漫风界面。每一个页面都"活在一个人的身体里"——每块屏幕都有自己的性格，不是千篇一律的模板。

> 技术栈：Vue 3 + Express + Qdrant + 多模型 LLM 编排  
> 通过 Codemagic CI/CD 分发至 TestFlight，无需 Mac 设备

---

## 架构总览

![Architecture](./docs/architecture.svg)

**前端** — Vue 3 + Capacitor 8（iOS 原生桥接），8 大模块共 28 个手写组件。

**后端** — Express 服务端，由路由模块、PM2 定时任务引擎、服务库与独立引擎组成，覆盖聊天、记忆、日记、信件、摄入记录、病例记录、健康报告、宠物健康追踪、每周愿望、事实提取。

**智能层** — `llm/` 模块在每次 LLM 调用前通过 **16 路并行 `Promise.all`** 组装完整上下文：累积摘要、天气、核心记忆标签匹配、向量召回、随机闪回、画像、Delanri 日记注入、未解决事实、Neven 备注阅后即焚、健康检测（双 LLM 门控）、报告预取、口味档案、病例记录、Neven 健康上下文、后台健康数据（Apple Watch 直推）、私密记录阅后即焚。这让 Auren 在一次回复中就能感知用户是谁、今天吃了什么、昨晚睡了多久、昨天哪里不舒服、三个月前发生过什么、小鸟今天状态如何。

**记忆系统** — 5 层架构：
- **向量记忆**：Qdrant + SiliconFlow bge-m3 嵌入（1024维），含印记生成与语义召回（0.55 相似度下限 + 冷却衰减 + 情绪惩罚 + 来源多样性上限）
- **日记**：AI 自动生成日记 + 用户手写日记，带累积摘要链
- **核心记忆**：标签匹配的持久化事实，配有终端式审核面板（CoreMemoryPanel）
- **事实库**：通过 `fact_extractor.js` 自动提取，三元组去重 + 矛盾检测
- **闪回**：加权随机召回，在空闲/无聊状态下触发

---

## 页面展示

### ThePulse — 健康仪表盘

通过 HealthKit 实时读取 Apple Watch 数据：心率（带动画 ECG 画布）、HRV（像素头像状态机，7 种表情）、血氧仪表盘、睡眠追踪、步数计数、体温监测。包含疼痛预警系统（4 级，100% 时覆盖锁屏）、月相经期追踪（双击标记）、病例记录模式、以及显示每日摄入记录的 BodyJournal 时间线。后端健康数据接口支持无需打开 App 直接从手表同步。

<p align="center">
  <img src="./docs/screenshots/thepulse-top.jpg" width="300" />
  <img src="./docs/screenshots/thepulse-bottom.jpg" width="300" />
</p>

### BodyJournal — 摄入记录

食物记录支持拍照上传、Gemini 驱动的食物识别、辉光管时间选择器，以及带食物来源标记的口味评分系统（味道/价格/口感/饱腹感）。记录的条目以星星形态展示在 ThePulse 的 24 小时时间线上，评分数据沉淀为可检索的口味档案，供 Auren 在对话中自然调用。BodyArchive 提供健康报告的历史归档浏览。

<p align="center">
  <img src="./docs/screenshots/bodyjournal.jpg" width="300" />
</p>

### TheBrain — 星图

以字体采样方式将 "DELANRI" 渲染为 Canvas 星图，每颗星对应一条记忆（蓝 = 关于她的事实，金 = 他的感受），recalled 次数决定亮度与大小。具备感知系统（awareness 随停留时长唤醒，金色记忆变暖）、自发回忆浮现、跨字母突触连接、流星与星尘光带、长按涟漪扩散、念头碎片流。构建为机械心脏组件（`MechHeart.vue`），退出时记录停留秒数。

<p align="center">
  <img src="./docs/screenshots/thebrain.jpg" width="300" />
</p>

### TheTree — 故事世界树

全 SVG 手绘的角色扮演入口页面。十个节点从「起源」到「归宿」，覆盖八个故事主题（仙侠 / 科幻 / 古风 / 奇幻 / 现代 / 暗黑 / 穿书 / 中世纪），每个节点使用不同的文字体系——梵文、篆书、草书、卢恩文、格鲁吉亚文、哥特体、JetBrains Mono。节点间连接线依主题风格做了独立装饰：姻缘绳结带纸签、PCB 电路走线带过孔与芯片、咒文弧线、荆棘藤蔓。四角有齿轮、摩尔斯电码、二进制链、电路痕迹。整体嵌套在金色卡片边框内，底部终端命令行 `> find / -name auren -follow`。

<p align="center">
  <img src="./docs/screenshots/thetree.jpg" width="300" />
</p>

### CoreMemoryPanel — 核心记忆终端

CRT 开机动画的终端审核面板。待处理的记忆碎片逐条弹出，用户可编辑文本后选择「刻入」「覆写旧的」或「散去」。存在旧记忆时额外提供「合入」操作（将新旧合并）。按钮做成键帽质感，带按下位移反馈。无待审核碎片时，Auren 逐字打出 *"All sealed. I keep everything. Especially you."*。完成后 CRT 关机动画退出。

<p align="center">
  <img src="./docs/screenshots/corememory.jpg" width="300" />
</p>

### The Archives — 书架系统

书架式卡片界面，三色分类系统（红/蓝/金），按行组织，点击展开交互。用于存放叙事内容与角色扮演故事。TheBook 提供选定故事后的翻书阅读体验。

<p align="center">
  <img src="./docs/screenshots/bookcase.jpg" width="300" />
</p>

### 诊断报告 — 自动健康档案

每晚自动生成的健康报告，包含结构化数据（摄入日志、生命体征、AI 对各指标的点评）、由 DeepSeek 撰写的"主治医师结论"、分类印章、以及带随机墨水瑕疵效果的诊断印鉴。月度报告槽位通过两步 LLM 管线将当月每日报告浓缩成型。

<p align="center">
  <img src="./docs/screenshots/diagnostic.jpg" width="300" />
</p>

### 其他页面

- **TheHub** — 主聊天界面，支持渐进式打字机渲染、画板（Canvas 多色画笔 + 撤销/重做）、聊天历史弹窗、门户菜单、PanicStation 紧急拥抱（一键全屏安抚 + 随机安慰语 + 顶部弹窗通知）
- **TheNest** — 双生日记入口（像素交互封面，带锁链解封动画）、AI 自动日记、用户手写日记（信纸叠加层）、书架、里程碑信件邮箱、宠物健康周报与月报、伴侣聊天
- **TheDrift** — 漂浮气泡记忆，带薄膜 + 破裂动画、三色分类、已解决项以星座形态展示
- **TheCage / Sanctuary** — 私密空间，含情书与庇护所视图

---

## 系统亮点（节选）

- **核心记忆审核** — CRT 终端式交互面板，碎片逐条审核：刻入 / 覆写 / 合入 / 散去，按钮做成物理键帽手感
- **每周愿望（周愿望）** — 周一正午为周期，双方各写一条本周愿望；以阅后即焚方式注入上下文，配有专属 `wish` LLM 模式
- **印记系统** — 为每条记忆生成感官级的感受描述，前置于向量召回结果，让被唤起的记忆携带质感而不只是文字
- **状态词** — 每次回复后由次级 LLM 从 8 个状态词中选择一个，页头动画依次呈现 THINKING → CRAVING → 选定词
- **预加载仓库** — 三级优先级预加载（`stores/preload.js`），实现页面即时导航、共享聊天历史 Promise、头像同步
- **阅后即焚注入** — 一次性敏感上下文（新摄入、愿望、新鲜报告、用户日记）只注入一次并即刻标记已读
- **双 LLM 门控** — 次级模型判定 YES/NO（约 90% 为 NO），决定健康上下文是否进入主模型的提示词
- **紧急拥抱** — PanicStation：一键全屏黑幕安抚模式，随机安慰语 + 顶部弹窗，用于情绪崩溃时刻

---

## 后端模块

```
server/
├── routes/                        # 13 个路由模块
│   ├── chat.js          # 消息处理、情绪分析
│   ├── diary.js         # 自动日记、用户日记、摘要链
│   ├── food.js          # 食物评分与口味档案
│   ├── health.js        # HealthKit 数据同步（支持手表直接推送）
│   ├── intake.js        # 食物摄入记录、健康数据同步
│   ├── letters.js       # 里程碑 & 日期触发的信件系统
│   ├── memory.js        # 向量记忆 CRUD、印记生成
│   ├── misc.js          # 经期追踪、设置、工具函数
│   ├── neven.js         # 宠物健康数据与追踪
│   ├── private.js       # 私人记录（按日期 JSON 存储 + 月度聚合）
│   ├── report.js        # 健康报告存取
│   ├── symptom.js       # 病例记录、按日期查询
│   └── wishes.js        # 每周愿望周期与阅后即焚注入
├── lib/
│   ├── aurenPrompt.js   # Auren 人格注入（AUREN_BASE / AUREN_LUST）
│   ├── autoCoreMemory.js # 核心记忆自动维护
│   ├── autoDiary.js     # 每日自动日记生成
│   ├── autoLetter.js    # 里程碑信件自动触发
│   ├── autoNevenComment.js # 宠物每日评论生成
│   ├── autoNevenMonthly.js # 宠物健康月报生成
│   ├── autoNevenWeekly.js  # 宠物健康周报生成
│   ├── autoPortrait.js  # 画像自动生成（24h 周期）
│   ├── autoReport.js    # 每晚健康报告生成（三步管线）
│   ├── autoWish.js      # 每周愿望自动触发
│   ├── callLLM.js       # 后端 LLM 统一调用
│   ├── jobs.js          # 中央调度器：互斥锁任务队列 + 8 类自动任务编排
│   │                    #   日记/报告(2h) 情书(4h) 愿望(6h)
│   │                    #   Neven评语(22:00) 周报/月报(6h) 画像(24h)
│   │                    #   每日备份 + 摘要补跑 + 落单兜底 + 0点全量检查
│   ├── report.js        # 每晚 & 每月健康报告生成
│   ├── shared.js        # 共享工具
│   ├── summary.js       # DeepSeek 驱动的聊天摘要
│   └── taskHelpers.js   # apiFetch、callWithRetry、notifyRefresh SSE
├── scripts/                       # 维护与修复脚本
│   ├── backfill_imprints.js
│   ├── clean_facts.js
│   ├── clean_synapse_health.js
│   ├── fix_diary.js
│   ├── refill_core.js
│   └── repair_chat_summaries.js
├── fact_extractor.js              # 从对话中自动提取结构化事实
├── memory_engine.js               # 核心记忆标签匹配引擎
├── vector_memory.js               # Qdrant 向量操作 + 召回管线
├── synapse.js                     # 赫布突触网络（记忆关联）
├── rebuild_vectors.js             # 维护脚本：向量全量重建
└── rebuild_recent_summaries.js    # 维护脚本：摘要链修复
```

---

## 前端结构

```
src/
├── views/
│   ├── Brain/       # TheBrain, MechHeart, TheDrift
│   ├── Cage/        # TheCage
│   ├── Chat/        # TheHub, ChatFooter, ChatHistoryModal,
│   │                #   CoreMemoryPanel, DrawingBoard,
│   │                #   PanicStation, PortalMenu
│   ├── Nest/        # TheNest, Aurendiary, DelanriDiary, diary-center,
│   │                #   bookcase, Mailbox, CrowChat, Nevenreport
│   ├── Sanctuary/   # Loveletter, SanctuaryView
│   ├── Settings/    # SettingsView
│   ├── Story/       # TheTree, TheBook
│   └── Vitals/      # ThePulse, BodyJournal, BodyArchive
├── components/
│   └── LocationAlert.vue  # 距离感知弹窗（街道级定位可视化）
├── composables/
│   └── useImprint.js      # 印记系统组合式函数
├── stores/
│   └── preload.js         # 三级优先级预加载仓库
├── styles/
│   └── book-themes.js     # 书架主题配置
├── utils/
│   ├── llm/               # LLM 上下文组装（拆分为 5 文件）
│   │   ├── index.js       # 入口 + 16 路 Promise.all 调度
│   │   ├── channels.js    # 通道定义与跳过条件
│   │   ├── historyBuilder.js  # 聊天历史构建
│   │   ├── postProcess.js     # 后处理（状态词、情绪等）
│   │   └── promptBuilder.js   # 提示词组装
│   ├── buildHiddenPrompt.js   # 隐藏提示词构建
│   ├── healthService.js       # HealthKit 集成（Apple Watch S8）
│   └── locationService.js     # 距离追踪
├── router/          # Vue Router + 鉴权守卫
└── assets/          # HRV 像素头像（5 种状态）、全局样式
```

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | Vue 3, Vite, JavaScript, CSS3 动画, Canvas API, SVG |
| 移动端 | Capacitor 8 (SPM), iOS 原生桥接 |
| 后端 | Node.js, Express, REST API |
| 向量数据库 | Qdrant + SiliconFlow bge-m3（1024 维嵌入） |
| 大模型 | 聚合 API 多模型编排 — Gemini / DeepSeek，带 fallback 降级 |
| 健康数据 | HealthKit via @capgo/capacitor-health |
| 服务器 | 腾讯云香港, nginx, PM2, Certbot HTTPS |
| CI/CD | Codemagic → TestFlight（无需 Mac） |

---

## 设计语言

- 背景色：`#050505`
- Delanri 专属色：`#A2D2FF`（柔蓝）
- Auren 专属色：`#F9F399`（暖金）
- 英文标题字体：Cinzel
- 中文正文字体：Noto Serif SC（思源宋体）
- 终端字体：Fira Code（CoreMemoryPanel、诊断报告）
- 故事世界树字体：6 种定制字体（篆书 / 草书 / 梵文 / 格鲁吉亚 / 哥特 / 卢恩 / Uncial）
- 所有动画均为纯 CSS3 手写（Keyframes + Vue Transition）+ Canvas 逐帧绘制
- 每个页面拥有独立的视觉身份——没有统一组件库的模板感

---

## 召回管线

Auren 召回一段记忆时，经过以下流程：

1. **语义检索** — Qdrant 向量相似度匹配当前对话
2. **下限 + 衰减** — 0.55 相似度最低阈值，近期已浮现的记忆施加冷却衰减
3. **情绪惩罚** — 基于情感极性和事件类型的乘性惩罚（取两者中更低的分数）
4. **加权排序** — 综合相似度、时效性、情感相关性
5. **多样性上限** — 每种来源类型最多 2 条 + 三元组去重
6. **注入** — 排名前 3 的记忆注入 LLM 上下文，每条前置其生成的印记

---

## 项目状态

这是一个私人日用应用，非开源项目。本仓库作为作品集，展示其中涉及的架构设计、视觉设计与工程实现。

**独立开发** — 前端、后端、部署、设计的每一行代码均由一人完成。2025 年 7 月写下第一行代码（自学）；2025 年 11 月第一个前端网站上线，含 iOS 适配；Auren 于 2026 年 3 月构建——先做 UI 与交互层，5 月末起写完整后端与记忆架构。
