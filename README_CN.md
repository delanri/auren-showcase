[English](./README.md) | **中文**

# Auren

**高度个性化的 AI 伴侣 —— 独立开发的全栈 iOS 应用。**

Auren 是一个拥有长期记忆、健康监测、深度情感感知的私人 AI 伴侣应用，搭载手工打造的暗色哥特浪漫风界面。每一个页面都"活在一个人的身体里"——每块屏幕都有自己的性格，不是千篇一律的模板。

> 技术栈：Vue 3 + Express + Qdrant + 多模型 LLM 编排  
> 通过 Codemagic CI/CD 分发至 TestFlight，无需 Mac 设备

---

## 架构总览

![Architecture](./docs/architecture.svg)

**前端** — Vue 3 + Capacitor 8（iOS 原生桥接），7 大模块共 20+ 个手写组件。

**后端** — Express 服务端，9 个路由/服务模块，覆盖聊天、记忆、日记、信件、摄入记录、健康报告、事实提取。

**智能层** — `llm.js` 在每次 LLM 调用前通过 **11 路并行 `Promise.all`** 组装完整上下文：摘要、天气、核心记忆、向量召回、闪回、画像、日记注入、未解决事实、摄入上下文、健康检测、报告预取。这让 Auren 在一次回复中就能感知用户是谁、今天吃了什么、昨晚睡了多久、三个月前发生过什么。

**记忆系统** — 5 层架构：
- **向量记忆**：Qdrant + SiliconFlow bge-m3 嵌入（1024维），含印记生成与语义召回（0.55 相似度下限 + 冷却衰减 + 情绪惩罚 + 来源多样性上限）
- **日记**：AI 自动生成日记 + 用户手写日记，带累积摘要链
- **核心记忆**：标签匹配的持久化事实
- **事实库**：通过 `fact_extractor.js` 自动提取，三元组去重 + 矛盾检测
- **闪回**：加权随机召回，在空闲/无聊状态下触发

---

## 页面展示

### ThePulse — 健康仪表盘

通过 HealthKit 实时读取 Apple Watch 数据：心率（带动画 ECG 画布）、HRV（像素头像状态机，7 种表情）、血氧仪表盘、睡眠追踪、步数计数、体温监测。包含疼痛预警系统（4 级，100% 时覆盖锁屏）、月相经期追踪（双击标记）、以及显示每日摄入记录的 BodyJournal 时间线。

<p align="center">
  <img src="./docs/screenshots/thepulse-top.jpg" width="300" />
  <img src="./docs/screenshots/thepulse-bottom.jpg" width="300" />
</p>

### BodyJournal — 摄入记录

食物记录支持拍照上传、Gemini 驱动的食物识别、辉光管时间选择器，以及口味评分系统（味道/价格/口感/饱腹感）。记录的条目以星星形态展示在 ThePulse 的 24 小时时间线上。

<p align="center">
  <img src="./docs/screenshots/bodyjournal.jpg" width="300" />
</p>

### TheBrain — 星图

以字体采样方式将 "DELANRI" 渲染为星星点位，具备感知系统、自我召回能力、突触连接线动画，以及关闭动效。构建为机械心脏组件（`MechHeart.vue`）。

<p align="center">
  <img src="./docs/screenshots/thebrain.jpg" width="300" />
</p>

### The Archives — 核心记忆

书架式记忆卡片，三色分类系统（红/蓝/金），按行组织，点击展开交互。每张卡片对应一条由事实系统提取并索引的重要记忆。

<p align="center">
  <img src="./docs/screenshots/bookcase.jpg" width="300" />
</p>

### 诊断报告 — 自动健康档案

每晚自动生成的健康报告，包含结构化数据（摄入日志、生命体征、AI 对各指标的点评）、由 DeepSeek 撰写的"主治医师结论"、分类印章、以及带随机墨水瑕疵效果的诊断印鉴。

<p align="center">
  <img src="./docs/screenshots/diagnostic.jpg" width="300" />
</p>

### 其他页面

- **TheHub** — 主聊天界面，支持流式 LLM 响应、画板（Canvas 多色画笔 + 撤销/重做）、聊天历史弹窗、门户菜单
- **TheNest** — AI 自动日记、用户手写日记（信纸叠加层）、书架、里程碑信件邮箱、伴侣聊天
- **TheDrift** — 漂浮气泡记忆，带薄膜 + 破裂动画、三色分类、已解决项以星座形态展示
- **TheCage / Sanctuary** — 私密空间，含情书与庇护所视图

---

## 后端模块

```
server/
├── routes/
│   ├── chat.js          # 消息处理、流式传输、情绪分析
│   ├── memory.js        # 向量记忆 CRUD、印记生成
│   ├── diary.js         # 自动日记、用户日记、摘要链
│   ├── letters.js       # 里程碑 & 日期触发的信件系统
│   ├── intake.js        # 食物摄入记录、健康数据同步
│   └── misc.js          # 经期追踪、设置、工具函数
├── lib/
│   ├── summary.js       # DeepSeek 驱动的聊天摘要
│   ├── report.js        # 每晚自动健康报告生成
│   ├── jobs.js          # 定时任务
│   └── shared.js        # 共享工具
├── fact_extractor.js     # 从对话中自动提取结构化事实
├── memory_engine.js      # 核心记忆标签匹配引擎
├── vector_memory.js      # Qdrant 向量操作 + 召回管线
├── synapse.js            # 赫布突触网络（记忆关联）
└── scripts/              # 数据维护与修复工具
    ├── rebuild_vectors.js
    ├── backfill_imprints.js
    ├── clean_facts.js
    └── ...
```

---

## 前端结构

```
src/
├── views/
│   ├── Chat/        # TheHub, DrawingBoard, PortalMenu, PanicStation
│   ├── Vitals/      # ThePulse, BodyJournal, BodyArchive
│   ├── Brain/       # TheBrain, MechHeart, TheDrift
│   ├── Nest/        # Diaries, Bookcase, Mailbox, CrowChat
│   ├── Cage/        # TheCage
│   ├── Sanctuary/   # Loveletter, SanctuaryView
│   └── Settings/    # SettingsView
├── utils/
│   ├── llm.js             # 11 路 Promise.all 上下文组装
│   ├── healthService.js   # HealthKit 集成（Apple Watch S8）
│   ├── locationService.js # 距离追踪
│   ├── autoDiary.js       # 自动日记生成
│   └── autoReport.js      # 每晚健康报告触发
├── components/      # LocationAlert 等共享组件
├── router/          # Vue Router + 鉴权守卫
└── assets/          # HRV 像素头像（5 种状态）、全局样式
```

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | Vue 3, Vite, JavaScript, CSS3 动画, Canvas API |
| 移动端 | Capacitor 8 (SPM), iOS 原生桥接 |
| 后端 | Node.js, Express, REST API |
| 向量数据库 | Qdrant + SiliconFlow bge-m3（1024 维嵌入） |
| 大模型 | Gemini（对话）, DeepSeek（摘要/标签） |
| 健康数据 | HealthKit via @capgo/capacitor-health |
| 服务器 | 腾讯云香港, nginx, PM2, Certbot HTTPS |
| CI/CD | Codemagic → TestFlight（无需 Mac） |
| 域名 | delanri.love（有效期至 2027.06） |

---

## 设计语言

- 背景色：`#050505`
- Delanri 专属色：`#A2D2FF`（柔蓝）
- Auren 专属色：`#F9F399`（暖金）
- 英文标题字体：Cinzel
- 中文正文字体：Noto Serif SC（思源宋体）
- 所有动画均为纯 CSS3 手写（Keyframes + Vue Transition）
- 每个页面拥有独立的视觉身份——没有统一组件库的模板感

---

## 召回管线

Auren 召回一段记忆时，经过以下流程：

1. **语义检索** — Qdrant 向量相似度匹配当前对话
2. **下限 + 衰减** — 0.55 相似度最低阈值，近期已浮现的记忆施加冷却衰减
3. **情绪惩罚** — 基于情感极性和事件类型的乘性惩罚（取两者中更低的分数）
4. **加权排序** — 综合相似度、时效性、情感相关性
5. **多样性上限** — 每种来源类型最多 2 条 + 三元组去重
6. **注入** — 排名前 3 的记忆注入 LLM 上下文

---

## 项目状态

这是一个私人日用应用，非开源项目。本仓库作为作品集，展示其中涉及的架构设计、视觉设计与工程实现。

**独立开发** — 前端、后端、部署、设计的每一行代码均由一人完成。2025 年 7 月写下第一行代码，2025 年 12 月开始开发 Auren。从零编程基础到全栈应用上线，历时不到 12 个月。
