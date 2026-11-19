# AI 综合工作台 · 项目展示与实操流程

面向 **跨境电商 AI 应用开发** 方向的作品集公开展示仓库。这里只放可公开内容：项目介绍、操作流程、技术亮点、界面截图与脱敏商品素材，**不含任何账号、口令、API Key 或私有部署地址**。

## 这是什么

一个统一入口下的 AI 内容生产工作台：文档被切成可信知识块，检索服务给出带引用的答案，套图、漫剧和创作画布把商品素材变成营销内容，底层训推平台负责数据集、微调和推理服务。

```text
                    ┌─ 商境：内容创作画布 / 任务队列
统一桌面 ─ 6 个应用 ├─ PicAgent：套图生图 / 爆品图反推
  登录与授权        ├─ ComicFlow：故事到漫剧视频
                    ├─ Chunk Lab：文档切分与人工审阅
                    ├─ Agentic Retrieval：可追溯 RAG 检索
                    └─ CPPT：数据集 → 微调 → 推理服务
```

## 在线浏览

打开 `index.html` 即可在本机浏览完整展示页；也可以直接看下面各项目的实操流程与技术亮点。

## 项目实操流程与亮点

| 项目 | 实操流程（摘要） | 技术亮点 |
| --- | --- | --- |
| XShipAI Desktop | 登录 → 看桌面图标 → 直接访问子应用被拦截 → 点图标授权 → 退出撤销 | Nginx `auth_request` 统一网关；HttpOnly Session；飞书 OAuth2 + PKCE |
| 商境 | 五菜单导览 → 商品录入 → 项目画布 → 看节点/连线/任务队列 → 模型中心 | 八类节点画布；端口类型校验；同参数幂等；提交/轮询失败分级 |
| PicAgent | 建方案 → 传商品图 → 一键精修 → 选 Amazon 平台 → 勾资料 → 生成结构建议 → 确认 → 生成类别提示词 → 画布 | 两阶段 LLM 编排 + 人工确认闸门；RAG 事实约束；提示词/来源快照；attempt 版本血缘 |
| Chunk Lab | 上传 → 解析 → 选策略 → 应用参数并生成预览 → 校正/合并/拆分 → Context 审核 → 向量化入库 → 检索测试 | 结构感知/语义切分；父子块；来源定位；Context 生命周期；pgvector 写入 |
| Agentic Retrieval | 输入查询 → 切 Dense/BM25/Hybrid/Rerank → 看 Context Pack 与 Citation → 跑评测 | 只读 adapter；服务端权威证据；显式失败态；OpenAPI 生成 TS SDK |
| ComicFlow | 建项目 → 输中文故事 → 看预计 Credit → 剧本/分镜/角色卡/画面/字幕 → 任务详情 → FFmpeg 导出 | Celery 异步长任务；阶段产物版本化；局部依赖失效；成本门禁 |
| CPPT | 数据集校验 → 训练表单 → 任务队列 → Best/Final Adapter → 推理服务 → 云 GPU 库存价格 | LLaMA-Factory YAML 配置即代码；单 GPU 租约；SSH 失败恢复；eval_loss 选 Best |

更详细的逐步操作脚本见 `PRESENTATION.md`，现场速查卡见 `QUICK-CARD.md`。

## 截图

- `screenshots/01-desktop-login.png` — 统一登录入口
- `screenshots/02-desktop.png` — 六应用桌面
- `screenshots/03-shangjing.png` — 商境工作台
- `screenshots/04-comicflow.png` — ComicFlow 创作工作台
- `screenshots/05-picagent.png` — PicAgent 从商品实拍到套图方案
- `screenshots/06-chunk-lab.png` — Chunk Lab 文档切分审阅
- `screenshots/07-retrieval.png` — 智能检索工作台
- `screenshots/08-cppt.png` — CPPT 系统概览

## 商品素材

`assets/products/` 中四张鼠标商品图用于演示「实拍原图 → 平台规则约束 → 套图/详情页素材」的输入侧，不作为任何品牌授权声明。其中 `mouse-white-bg.webp` 是白底商品图，其余三张为实拍/场景/细节图。

## 技术关键词

AI Agent 编排 · RAG · pgvector · LLM 结构化输出 · Human-in-the-loop · 幂等 · 版本血缘 · Provider 适配层 · 异步任务 · Docker · FastAPI · React · Next.js · Fastify · PostgreSQL · MinIO · Redis · Celery · Nginx auth_request

## 说明

- 展示页为静态作品集，不提供在线登录；演示环境地址与口令不在此公开。
- 项目能力边界以代码、测试和验收记录为准；Mock/待测能力与真实验收在 `PRESENTATION.md` 中明确区分。
- 商品图片仅用于技术演示与作品集说明。
