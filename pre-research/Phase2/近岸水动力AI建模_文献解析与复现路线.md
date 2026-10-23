# 面向 Delft3D 的"自然语言智能建模系统"——文献解析与复现路线

> 目标：对标清华 HydroCraft（Tudaji, Tian & Wu, 2026, J. Hydrology 677:135801），
> 建一个"用户一句自然语言 → 自动完成研究区解析、网格、地形、潮汐/风边界、配置文件、运行、率定辅助、结果可视化"的近岸水动力建模系统。

---

## 第一部分：文献逐篇解析

### 1. HydroCraft（清华田富强团队）——总架构范本 ★★★★★（逐节点模仿）

- **出处**：Tudaji M., Tian F., Wu X. (2026). *HydroCraft: an efficient online platform for hydrological modeling integrated with a large language model agent.* Journal of Hydrology, 677, 135801. DOI: 10.1016/j.jhydrol.2026.135801
- **平台**：https://www.hydrocraft.hydr.tsinghua.edu.cn/ （需注册）
- **它做了什么**：
  1. 网页平台把建模全流程（流域提取 → 数据自动生成 → 参数率定 → 结果可视化）搬进浏览器；
  2. 内置 3 个水文过程模型，自带全球降水/气温/NDVI/LAI/土壤数据集——**用户不需要准备任何数据**；
  3. 核心创新：集成 LLM Agent（基于 ChatGLM/GLM-4），Agent 理解自然语言需求后**必须按规定的 JSON 格式输出结构化参数**，再驱动平台各功能模块；
  4. 自适应流域划分方法（adaptive watershed delineation）。
- **抄什么**：
  - 平台定位：浏览器内全流程、注册即用、按"流域"建案例库；
  - **对话协议**：LLM → 严格 JSON Schema → 平台执行。这是全文最值得逐字复现的设计；
  - "数据内置"思想：全球潮汐/地形/气象数据打包进平台，用户只给两样东西——地名（或地图框选）+ 时间范围。
- **不照搬**：水文模型的流域划分算法（近岸问题是网格+开边界，逻辑不同）；他们模型是概念性水文模型，我们没有全流程率定的历史包袱，但率定模块仍要保留。

### 2. emmashie/model-agent（PNNL，ROMS 自动建模智能体）——代码级复现模板 ★★★★★（直接 clone 逐文件读）

- **出处**：github.com/emmashie/model-agent（开源，配套 LLM 驱动 ROMS 建模工作流）
- **它做了什么**：5 个子智能体分工：
  - `llm_grid_agent.py` 网格生成（LLM 解析研究区/模拟目标 → 智能参数建议 → 下载 SRTM15+ 水深 → 生成格点 → 平滑 → 可视化）
  - `llm_init_agent.py` 初始场（GLORYS 温盐流插值到网格）
  - `llm_boundary_agent.py` 边界（自动检测开边界 + GLORYS 插值）
  - `llm_forcing_agent.py` 表面强迫（ERA5）
  - `llm_complete_agent.py` 串联全流程
  - 输入如："Create a ROMS setup for Chesapeake Bay from lat 36.5-39.5, lon -77.5 to -75.5, initialized for January 1, 2024"，约 1–2 分钟产出可运行模型。
- **抄什么**：
  - **五智能体拆分方式**（幻我们对应：网格 agent / 地形 agent / 开边界 agent / 边界强迫 agent / 总控 agent）；
  - **"有科学依据的参数建议 + 用户 Y/n 确认"交互模式**（例："次中尺度目标 → 建议 ~1km 分辨率、50 层、theta_s=7，是否采用？"）；
  - **地名知识表**（代码里硬编码了 Chesapeake Bay 等区域坐标，可替换为渤海湾/湄洲湾/长江口等国内近岸地名表）；
  - **LLM 失效回退机制**（API 不可用时回退到正则/表单解析，系统仍可用）；
  - 仓库结构（agents/ examples/ tests/ docs/）。
- **不照搬**：ROMS 的坐标系统（s 坐标）、GLORYS 初始场逻辑（近岸 Delft3D 常用地形直接起算+冷启动水位场）。

### 3. LLM + SWMM 灌溉调度（Comput. Electron. Agric. 2026）——MCP 集成模式 ★★★★

- **出处**：*Closed-loop autonomous scheduling of multi-level irrigation canal-gate systems using an LLM-agent framework*，2026，Computers and Electronics in Agriculture（S0168169926010422）
- **它做了什么**：物理仿真器（SWMM）通过 **MCP 协议**封装成 LLM 可调用的工具，布置在 Dify 平台上；LLM 决策、模型仿真回传水力反馈、MQTT 操控设备；对比 4 个 LLM（GPT-5/DeepSeek-R1/Qwen3/Kimi-k2）× 3 种提示策略（DI/CoT/CA）。
- **抄什么**：
  - **"把 Delft3D 包成 MCP server"** 的集成模式——比自制 function-calling 框架更标准化、更易接各家 LLM；
  - **约束增强提示（Constraint-Augmented prompting）**:把物理硬约束写进系统提示，缓解 LLM 对数值不敏感的问题（我们对应：Courant 数上限、dt 由网格尺度自动反算，写在提示里）；
  - 底座模型结论:Qwen3-32B 性能/成本均衡最优——支持国产模型路线。

### 4. Yan et al. 2026（GRL）六级自主性框架——论文定位与方法学骨架 ★★★★

- **出处**：Yan S., Chen M. et al. (2026). *AI Agent for Hydrologic Modeling: Definition, Development, and Application.* Geophysical Research Letters. DOI: 10.1029/2025GL119814（成稿源头工程 AQUAH agent，实现 Level-4 自主性）
- **它做了什么**：定义水文建模 AI agent 自主性六级框架（L1=知识问答/单步操作 → … → L4=闭环自主建模（含率定，AQUAH 已达到）→ 更高级=全自动排障与研究级任务）；agent 集成自然语言查询、数据检索、模型配置。
- **抄什么**：
  - **论文的方法学骨架**："定义分级框架 → 在框架中定位我们的系统 → 做出 L3–L4 → 用案例证明"。审稿人友好；
  - 精读后把六级定义搬进你们的方案书，作为研发里程碑的分档标准（当前目标 L3–L4）。
- **注意**：正文被 Wiley 拦截（403），需要在学校 IP/pass 中精读 PDF 的六条定义原文，避免我这里概括有偏差。

### 5. HydroSuite-AI（Pursnani, Ramirez, Sermet, Demir）——LLM 驱动已有科学库的方式 ★★★★

- **出处**：2024 EarthArXiv 预印本（EarthArXiv object 8121，被引 10+），2026 正式发表：*HydroSuite-AI: a web-based LLM environment for hydrological code generation and execution for the hydrosuite open-source ecosystem*（University of Iowa 机构库可查）
- **它做了什么**：LLM 助手集成三个开源水文 web 库（HydroLang、HydroCompute、HydroRTC），实现三类能力：
  1. **事实性问答**（带检索支持）；
  2. **代码片段生成**（针对库 API）；
  3. **在线执行环境**（用户直接跑生成的代码）。
- **抄什么**：
  - "LLM 生成调用既有库的代码 → 沙盒执行 → 返回结果"这条轻量路线——我们做 MVP 阶段可以不做完整平台，先做 **Jupyter/CLI 环境里 LLM 生成 HYDROLIB-core / pyDelft3D-FLOW 调用代码并执行**；
  - 把模型库的 API 文档挂进上下文的工程处理方式。
- **不照搬**：它偏"代码生成"路线，对所有用户仍有编程门槛；HydroCraft 的"结构化表单+JSON"路线对小白更友好——末端产品以 HydroCraft 型体验为准，MVP 用 HydroSuite 型实现。

### 6. HydroLLM（Kizilkaya, Sermet, Demir）——评测集构建方法 ★★★

- **出处**：Kizilkaya D., Sermet Y., Demir I. (2025). *Towards HydroLLM: approaches for building a domain-specific language model for hydrology.* Journal of Hydroinformatics, 27(10), ~1652.（另有 *Toward HydroLLM: a benchmark dataset*，Environmental Data Science, 2025）
- **它做了什么**：从教材+论文抽取约 8,800 条水文 QA 对，四种题型（多选/判断/填空/开放），微调 Qwen/Llama 系列，发现领域微调有效但大模型存在过拟合/缩放问题。
- **抄什么**：
  - **造"水动力建模评测集"的流水线**（用 LLM 从 Delft3D 手册+tutorials+你们组经验文档生成 QA 对，再人工校验）；
  - 选底座模型的评测方法（先考试再上岗）。
- **配套**：arXiv 2512.03672（评测主流 LLM 的水利科学知识水平）——选型阶段直接读它的结论省时间。

### 7. HydroAIM（Grounding LLMs in hydrologic modelling, 2026）——Grounding 概念补充 ★★

- **它做了什么**：LLM 智能体化水文时间序列预测建模，强调"把 LLM 锚定（grounding）在模型真实输入/输出空间上"。
- **抄什么**：文章口径——LLM 的每个参数输出必须能落到模型配置文件的对应字段上，禁止悬空输出。

### 8. Deltares 工具链（非论文，是复现的现实地基）★★★★★

| 工具 | 作用 | 获取 |
|---|---|---|
| **HydroMT** | Deltares 官方"模型自动构建"Python 框架：YAML 声明数据源+处理步骤 → 完整水动力模型 | github.com/Deltares/hydromt |
| **HydroMT-Delft3D FM 插件** | 直接生成/修改 D-Flow FM 模型 | deltares.github.io/hydromt_delft3dfm |
| **HYDROLIB-core** | 官方 Python 读写 D-Flow FM 输入文件（.mdu 及全部 netcdf/forcing 文件） | github.com/Deltares/HYDROLIB-core |
| **pyDelft3D-FLOW**（JulesBlm） | 读写经典 Delft3D4-FLOW 输入（.mdf 主控 + attribute 文件） | github.com/JulesBlm/pyDelft3D-FLOW |
| **数据源** | GEBCO/SRTM15+（地形）、TPXO9/FES2014（潮汐调和常数）、ERA5（风、气压、热通量）、GRDC（径流）、GLORYS（温盐，用于温盐模块或初始场） | 各自 API/Thredds |
| **校准工具** | PEST(+Python 绑定) / DDS / SCE-UA / DREAM | 开源 |
| **学习材料** | Delft3D-FLOW/WAVE User Manual；官方 tutorials；Deltares "Build and adjust Delft3D FM 1D2D models with Python tools" 免费课程（3.5h，讲 HYDROLIB-core+HydroMT） | content.oss.deltares.nl |

---

## 第二部分：复现路径（按里程碑排）

### Sprint 0（第 1–2 周）：两个决策 + 选型考试

1. **模型路线决策**：主线 D-Flow FM（有 HydroMT/HYDROLIB-core 全开源 Python 栈，agent 最好操纵）；兼容线 Delft3D4（pyDelft3D-FLOW 写 mdf，先只支持规则矩形网格，曲网放后期）。
2. **底座模型考试**：用 HydroLLM 方法造 100–200 道水动力建模题（参数含义/异常日志/边界设置），考 GLM-4、Qwen(32B/Max)、DeepSeek、GPT，综合 accuracy+成本+API 稳定性（Du参考 SWMM 论文结论：Qwen3-32B 大概率胜出）。
3. **精读 3 篇**:HydroCraft 正文（JSON 协议全文）、model-agent 源码（agents/ 下 5 个文件）、HydroSuite-AI 预印本。产出方案书。

### Sprint 1–2（第 3–8 周）：工具层（无 LLM，纯确定性代码）

克隆 model-agent 作结构模板，建 `delft3d_agent_tools` 仓库，实现 9 个原子工具：

| # | 工具 | 输入 | 输出 | 技术 |
|---|---|---|---|---|
| T1 | 区位解析 | 地名/bbox | lon/lat 范围 | geopandas + 地名表 + 地图框选 API |
| T2 | 网格生成 | bbox, 分辨率 | 网格文件 | HydroMT-Delft3DFM（D-Flow FM）/ 简版矩形网格（Delft3D4） |
| T3 | 地形获取插值 | bbox, 网格 | 水深文件 | GEBCO/SRTM15+ API → 插值 → 干湿单元 |
| T4 | 潮汐开边界 | 开边界段位置 + 时段 | 分潮边界文件 | pyTMD/FES2014 调和分析（最关键、最省事的一项） |
| T5 | 气象强迫 | bbox + 时段 | 风场文件 | ERA5（ cdsapi） |
| T6 | 径流边界 | 河流列表 + 时段 | 径流文件 | GRDC/用户提供数据标准化 |
| T7 | 配置文件生成 | 上述全部 | .mdu 全套（或 .mdf） | HYDROLIB-core / pyDelft3D-FLOW |
| T8 | 运行监控 | 模型目录 | 运行状态/错误信息 | 调 dimr/delft3d4（Docker），解析 .dia 日志 |
| T9 | 验证率定 | 观测站数据 + 结果 | NSE/RMSE/潮差相位误差 + 敏感性/率定（Manning、底摩擦） | xarray+matplotlib+scipy.optimize/PEST |

- **验收标准（不接 LLM）**：CLI 一条命令输入参数（研究区+时段+分辨率）→ 全自动产出可运行、算完的潮汐模型。
- 参照 model-agent 的 tests/ 写单元测试（无 LLM 的 basic 级测试模式直接照抄其目录结构）。

### Sprint 3–4（第 9–16 周）：Agent 层

1. **JSON Schema 对话协议**（照抄 HydroCraft）：

```json
{
  "task": "tidal_circulation",
  "region": {"name": "湄洲湾", "bbox": [118.9, 24.9, 119.3, 25.3]},
  "grid":  {"type": "unstructured", "res_m": {"min": 150, "max": 400}},
  "bathymetry": {"source": "GEBCO2024", "min_depth_m": 5},
  "open_boundary": {"type": "tidal", "constituents": ["M2","S2","N2","K1","O1","P1","K2","Q1"], "source": "TPXO9"},
  "forcing": {"wind": {"source": "ERA5", "on": true}, "rivers": [{"name": "...", "q_m3s": null}]},
  "time": {"start": "2024-06-01T00:00", "end": "2024-06-16T00:00", "dt_s": 60},
  "physics": {"roughness": {"type": "Manning", "value": 0.025}, "salinity_temp": false},
  "outputs": {"interval_min": 10, "variables": ["water_level","velocity"], "stations": [{"lon":119.1,"lat":25.1}]}
}
```

2. **系统提示词模块**：角色（近岸水动力建模工程师）+ 参数知识表（Courant 上限、网格-步长匹配规则、糙率典型值、常见 .dia 错误对照表）+ Constraint-Augmented 约束段（照 SWMM 论文）。
3. **交互设计**（照 model-agent）：多轮补全缺失信息 → "有理由的参数建议 + Y/n 确认" → 生成 JSON → 校验（schema + 物理规则双关）→ 进工具链。校验不过的东西**永远不进工具**（防幻觉）。
4. **自纠错闭环**：运行失败 → 解析 .dia → LLM 结合错误知识库诊断 → 改参数 → 重跑（上限 3–5 轮）→ 全失败则转人工。
5. **可选**：把 T1–T9 封装成 **MCP server**（照 SWMM 论文），在 Dify/自家前端接 GLM/Qwen。
- **验收**：一句自然语言（"帮我建一个湄洲湾 2024 年 6 月大潮期潮流模型，网格 200 米"）→ 聊天里拿到模型 → 跑通 → 看到图。

### Sprint 5–6（第 17–24 周）：Web 平台（HydroCraft 化）

- 前端：Vue/Next.js + 地图框选 + 对话主界面 + 任务进度 + 结果画板；
- 后端：FastAPI + Celery/Redis 任务队列；Docker 隔离每个模型运行；用户上传实测潮位/海图地形进"标准化接入器"（先做潮位站表和 .dep 两种格式）；
- 账号系统 + 案例库（历史案例一键复用）；
- 部署：Linux 服务器 + D-Flow FM 开源（或源码编译 Delft3D4）。

### Sprint 7–8（第 25–32 周）：验证与论文

**验证设计（发论文核心）**：
1. 基准案例:Delft3D 官方 tutorials 2–3 个 + 你们组已人工建好并有观测资料的近岸/河口案例 2–3 个（"盲测"）；
2. 指标：模型质量（NSE/RMSE/潮差与相位误差 vs 人工模型）+ **效率（人工建模型要多少小时天 vs AI 全流程分钟级）** + 用户实验（10–15 名新手做任务成功率）；
3. 按 Yan 六级框架声明系统当前自主等级（L3–L4），指出下一级路径。

**论文打包**（两篇）：
- 论文 A（方法）：*A natural-language agent system for intelligent Delft3D nearshore model construction* → Environmental Modelling & Software / Ocean Modelling / J. Hydrology；
- 论文 B（评测）：HydroDyn-LLM 问答基准数据集（用 Sprint 0 的考题扩到 ~2000 条）→ Environmental Data Science / J. Hydroinformatics。

---

## 第三部分：需要负责人现在拍板的事项

1. 主打 D-Flow FM 还是 Delft3D4（建议:双线,FM 为主）；
2. LLM 部署方式：公有 API（GLM/Qwen）还是本地部署（数据敏感？有 GPU 吗？）；
3. 数据许可证：TPXO 可要申请授权，FES2014 公开、Copernicus 需注册——按课题海域核对；
4. 算力：建模平台服务器配置（每个模型一个容器，先估算同时 3–5 用户）；
5. 基准案例的数据可得性（内部已经有的观测站潮位/流速记录先清点）；
6. 团队分工：1 人工具层（Python+HYDROLIB）、1 人 Agent/提示词、1 人前端平台、1 人案例验证+率定；可 2 人起步（工具+agent 合并）。

## 关键风险对策

| 风险 | 对策 |
|---|---|
| LLM 幻觉乱填参数 | 数值全部由确定性代码按物理规则算；LLM 只选类型/确认/传值；schema 校验不过就拒绝执行 |
| 免费地形/潮汐分辨率不够内湾精度 | 预留用户数据上传接口（海图/实测站文件），平台负责标准化 |
| Delft3D4 曲线网格自动化太难 | MVP 只做矩形/非结构网格；曲网借助 RGFGRID 参数化放 M3 |
| 软件 Windows 依赖 | D-Flow FM/HydroMT 全跨平台；Delft3D4 用官方 Linux 开源版编译 |
