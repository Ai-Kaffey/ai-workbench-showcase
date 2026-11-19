# 每个项目实操流程与技术亮点

> 这是公开版实操说明：只描述操作思路和技术实现，不公开任何私有地址、账号、口令或密钥。
> 配套截图在 `screenshots/steps/`，每个步骤都有对应截图；商品图素材在 `assets/products/`，作为各项目通用输入。

## 配套素材说明

四张鼠标商品图（`assets/products/`）是**各项目通用的输入素材**：

- `mouse-white-bg.jpg`：白底主图，可作为 Amazon 等平台首图输入；
- `mouse-black-detail.jpg`：外观细节，可用于详情图、图像理解数据说明；
- `mouse-packaging-dark.jpg`：包装实拍，可用于场景图与商品事实核对；
- `mouse-pink-lifestyle.jpg`：生活方式与配色参考，可用于风格卡与视觉约束演示。

图片已去除第三方水印；仅作技术演示，不代表品牌授权。

## 1. XShipAI Desktop — 统一入口与鉴权

**流程**
1. 打开统一桌面，用演示账号登录。
2. 新标签页直接访问任一子应用端口：被重定向回登录页。
3. 回到桌面，点击图标后才授权进入具体应用。
4. 退出登录后再次刷新子应用：会话失效。

截图：`steps/01-desktop-01-login.png`、`steps/02-desktop-02-apps.png`。

**亮点**
- Nginx `auth_request` 把身份认证收敛到一个服务，子应用只信任网关注入身份。
- 8 小时 HttpOnly Session；点击图标才逐应用授权；退出撤销全部授权。
- 飞书登录用用户 access token 查群成员身份，失败一律拒绝。

## 2. 商境 — 电商内容创作画布

**流程**
1. 顶部菜单依次浏览工作台、项目、商品、素材、系统。
2. 商品页查看手动/截图/链接三类录入入口。
3. 项目页进入画布，观察八类节点和类型化连线。
4. 底部任务队列查看运行中/成功/失败、耗时和错误原因。
5. 系统 → 模型中心查看模型目录与执行能力配置。

截图：`steps/03-shangjing-01-workbench.png` → `steps/03-shangjing-04-system.png`。

**亮点**
- 八类节点：商品输入、图片输入、提示词、生图、改图、合成、视频、输出。
- 同编号与参数幂等；提交失败与轮询失败按是否有上游编号分级处理。
- 动态渠道密钥 AES-256-GCM 加密，浏览器只见末四位。
- 拖拽/连线时冻结轮询与节点重建，避免画布丢帧和节点隐藏。

## 3. PicAgent — 电商套图生图

**流程**
1. 新建方案，上传 1–6 张商品实拍图。
2. 选已发布底图模板，执行白底精修，观察历史版本。
3. 在产品与设计简报中选择目标平台（如 Amazon），系统展示平台规则。
4. 勾选商品资料 Chunk，补充真实商品事实。
5. 生成结构建议 → 审核主图分配、详情图数量、风格卡 → 确认。
6. 生成类别提示词 → 进入画布查看任务卡与提示词映射。

输入素材示例：创建方案后上传 `mouse-white-bg.jpg` / `mouse-black-detail.jpg` 作为商品图，再做白底精修和平台规划。

截图：`steps/05-picagent-01-home.png` → `steps/05-picagent-05-amazon-platform.png`。

**亮点**
- 两阶段 LLM 编排：结构计划 → 分类提示词，中间设人工确认闸门。
- 平台规则来自 `platform_profiles`，服务端二次校验，且不覆盖商品事实。
- RAG Chunk 显式勾选，提示词/来源版本冻结成不可变快照。
- 任务生成用 attempt + version 血缘，失败重试不覆盖历史。

## 4. Chunk Lab — 文档切分与审阅

**流程**
1. 上传 Markdown/TXT/PDF 等文档，执行解析。
2. 选择 Sentence/Token/Structure-Aware/Semantic Boundary 策略。
3. 点击「应用参数并生成预览」，查看 Chunk 与来源定位。
4. 确认成 Draft 后做文本校正、拆分、合并、排除、撤销。
5. 对需要背景信息的 Chunk 生成 Context 并审核确认。
6. 向量化入库，执行检索测试并核对 Recall。

输入素材示例：把商品图对应的说明文档/参数表作为待审阅文档上传。

截图：`steps/06-chunk-lab-01-home.png` → `steps/06-chunk-lab-05-preview.png`。

**亮点**
- 结构感知切分保护表格、图片说明、警示语等完整语义单元。
- 父子块与可追溯 Source Segment，支持页面区域双向高亮。
- Context 生命周期确保只有 confirmed/human_modified 进入 embeddingText。
- 真实 PDF 验收：23 页 / 257 blocks / 55 chunks，38 条成功入库，Recall@5=1/1。

## 5. Agentic Retrieval — 可追溯检索

**流程**
1. 输入与商品资料相关的查询。
2. 依次切换 Dense、BM25、Hybrid、Hybrid+Rerank 四种 Profile。
3. 对比召回结果、Latency 和 Citation。
4. 展开 Context Pack，核对真正送进回答模型的证据。
5. 运行评测任务并查看结果。

输入素材示例：用「USB-C 无线鼠标」这类与商品图一致的事实内容进行查询。

截图：`steps/07-retrieval-01-home.png` → `steps/07-retrieval-04-profile.png`。

**亮点**
- 只读 adapter 访问 Chunk Lab 发布的 pgvector 数据，不重复建索引或向量化。
- 答案只基于服务端构造的 Context Pack，引用由服务端权威给定。
- 外部依赖失败返回明确状态，不静默切换到 fixture。
- OpenAPI 生成 TypeScript SDK，浏览器端不含检索逻辑。

## 6. ComicFlow — 故事到漫剧

**流程**
1. 创建项目，输入中文故事或创意。
2. 查看预计 Credit 后提交（演示中只讲解，不触发付费任务）。
3. 浏览剧本、分镜、角色卡、画面、声音字幕和时间轴阶段。
4. 在任务详情查看阶段、镜头、进度和错误原因。
5. 讲解释出 MP4、封面和 SRT 的流程。

输入素材示例：故事中指定「展示白底鼠标、细节特写、包装开箱」等镜头，对应商品图素材。

截图：`steps/04-comicflow-01-home.png` → `steps/04-comicflow-08-export.png`。

**亮点**
- 30–90 秒竖屏漫剧的 6–12 镜头结构化流水线。
- 中间产物可见、可编辑、可版本化，局部修改只失效下游。
- Celery + Redis 异步长任务，PostgreSQL 持久化业务状态。
- FFmpeg 服务端合成，签名下载地址，避免对象桶公开。

## 7. CPPT — 训推平台

**流程**
1. 数据集页查看 JSONL 上传与结构化校验报告。
2. 训练页查看 LoRA/QLoRA 参数、数据集绑定与任务队列。
3. 模型管理页查看 base、Best、Final Adapter 与归档恢复。
4. 推理服务页查看服务列表、SSE 调试入口和监控指标。
5. 云 GPU 页查看实时库存与按量价格，不启动真实任务。

输入素材示例：将鼠标商品图与标注整理为图文理解数据集，演示数据校验与模型产物管理。

截图：`steps/08-cppt-01-overview.png` → `steps/08-cppt-05-inference.png`。

**亮点**
- 表单配置解析为受限 YAML，不执行任意 Shell 命令。
- 单 GPU 租约互斥，训练与推理不能同时抢卡。
- Best/Final Adapter 按 eval_loss/train_loss 规则选择并显式警告。
- SSH 失败有限重试、非幂等命令不盲重放、资源失败冷却与健康回退。
