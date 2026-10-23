# 近岸水动力模型自然语言智能构建系统 —— 完整方案合集
（文献解析 ＋ 复现路径 ＋ 智能体全景解析 ＋ 落地执行清单）

> **总目标**：对标清华 HydroCraft（Tudaji, Tian & Wu, 2026, *Journal of Hydrology* 677:135801），
> 建成"用户一句自然语言 → 自动完成研究区解析、网格生成、地形/潮汐/风场数据接入、配置文件生成、
> 模型运行、自动排错、率定辅助、结果可视化"的 **Delft3D 近岸水动力智能建模系统**。

---

# 第一部分　智能体与大模型基础（大白话版）

## 1.1 三个基础概念

| 概念 | 一句话解释 |
|---|---|
| **LLM 大语言模型** | "脑子"。GLM/Qwen/DeepSeek/GPT 这类模型，只负责"说话"，不会主动干活 |
| **Agent 智能体** | "会动手的员工"＝ LLM(大脑) ＋ 工具箱(手) ＋ 工作循环(自己决定先干什么后干什么) |
| **子智能体 Sub-agent** | "流水线工种"。把复杂任务拆给几个专职 agent，各管一段 |

另一个关键概念：**MCP 协议** ＝"万能插座"——把专业软件(Delft3D、SWMM)封装成标准接口，
任何大模型都能像插插头一样接上来调用，比自制 function-calling 框架更通用。

## 1.2 文献里出现的所有智能体系统（逐个解释）

### ① HydroCraft 的 LLM Agent（清华）——"智能客服＋翻译官"
- 用户在网页上说需求 → Agent 听懂、追问、把需求**翻译成规定格式的 JSON 参数单** → 递给平台后面的施工队(水文模型+自动化程序)执行。
- 核心分工原则：**LLM 只翻译、不施工**。参数由确定性程序校验，防止 AI 脑补数字。

### ② model-agent 五个子智能体（PNNL，ROMS）——"造船厂流水线"（代码级模板）

| 子智能体 | 干什么 | 比喻 |
|---|---|---|
| Grid Agent | 解析经纬度范围→建议分辨率→下载水深→生成网格 | 划图纸打地基的 |
| Init Agent | 把某时刻海洋温盐状态填进网格 | 备现场状态的 |
| Boundary Agent | 模型边缘接外部数据 | 接水接电的 |
| Forcing Agent | 下载风场/气象作外力 | 装鼓风机的 |
| Complete Agent | 按序串联全部 | 工地总指挥 |

- 通用干法：**听需求 → 有理由的建议让用户 Y/n 确认 → 调工具干活 → 失败报告**。
- 对应到 Delft3D 版本：**网格 agent / 地形 agent / 潮汐开边界 agent / 风场径流 agent / 总控 agent**。

### ③ AQUAH（Yan et al. 2026, GRL）——"能独立做课题的研究生"
自己查数据、自己配模型、自己跑、自己率定，全程基本不用人管。
其提出的**六级自主性框架**（给 AI 员工评职称）：
- **L1** 只能问一答一（查资料）；
- **L2** 会干单步活（叫它下数据才动）；
- **L3** 会自己排工序，但每步要人确认；
- **L4** 从话到成品自主闭环（含率定，AQUAH 已达到）；
- **L5/L6** 能自主安排实验/发现新问题（目前多属愿景）。
- **我们的定位：做到 L3–L4。**

### ④ HydroSuite-AI 助手——"考过证的编程陪练"
网页版 AI：不懂的能答，要算的**生成代码并当场跑**给你看。透明但仍有代码门槛 → 适合 MVP 阶段实现方式，不适合最终产品形态。

### ⑤ WaterGPT / HydroLLM——"只会答题的学霸"（知识型 AI）
用 ~8800 道水文题（多选/判断/填空/问答）微调大模型 → 领域专业选手。
对你们的价值：**用同样方法给 Delft3D 知识刷题、建评测集**，相当于先让 AI 考"Delft3D 从业资格证"。

### ⑥ 自主 GIS Agent（LLM+QGIS 文献）——"自己会用 ArcGIS 的绘图员"
一句需求 → 自己选工具、串流程、排错、出图。证明"AI 独立操作专业软件"路线已通。

### ⑦ SWMM 灌溉调度 Agent——"水厂调度台＋对讲机"（*Comput. Electron. Agric.* 2026）
- LLM 当调度总管决定开 63 道闸；**SWMM 水力模型当沙盘**，每个决定先在沙盘推演会不会漫堤再修正。
- 两个关键技术比喻：**MCP 万能插座**（把 Delft3D 做成标准接口给各家 LLM 用）；
  **约束增强提示 CA prompting**＝在 AI 操作规程里预先写死红线（Courant 上限、闸门开度上限）——专治 AI 对数字不敏感。

### ⑧ HydroAIM——"把 LLM 拴在模型上的系绳"
核心一个词 **grounding（锚定）**：AI 输出的每个参数必须能落到模型配置文件的对应字段，落不到的禁止输出。

## 1.3 文中出现的大模型（大脑本体）选购指南

| 名字 | 定位 | 文中角色 |
|---|---|---|
| **GLM / ChatGLM（智谱）** | 国产、清华系 | **HydroCraft 的引擎**——同款路线最顺 |
| **Qwen 通义千问（阿里）** | 国产全能、开源可自部署 | SWMM 论文实测 **Qwen3-32B 性价比最优**（32B=320 亿参数，一张卡可养） |
| **DeepSeek** | 国产推理强手 | 对比选手 |
| **Kimi-k2** | 国产长文本特长 | 对比选手 |
| **GPT / Claude** | 美国头部 | 无合规顾虑时的备选；ROMS 仓库用的 Claude 接口 |
| **Llama（Meta）** | 开源实验小白鼠 | HydroLLM 微调用 |

**一句话**：要合规→Qwen3-32B 自部署 或 GLM API（HydroCraft 同款）；无合规顾虑→Claude/GPT。

---

# 第二部分　文献逐篇解析（它做了什么 → 抄什么 → 不抄什么）

## 2.1 HydroCraft ★★★★★ 总架构范本
- **出处**：Tudaji M., Tian F., Wu X. (2026). *HydroCraft: an efficient online platform for hydrological modeling
  integrated with a large language model agent.* **Journal of Hydrology, 677, 135801**. DOI: 10.1016/j.jhydrol.2026.135801
- **平台**：https://www.hydrocraft.hydr.tsinghua.edu.cn/ （需注册）
- **做了什么**：①网页全流程（流域提取→数据生成→率定→可视化）；②内置 3 个过程模型+全球数据（用户零准备）；
  ③GLM-4 Agent 严格 JSON 输出驱动平台；④自适应流域划分。
- **抄**：浏览器全流程产品形态；**LLM→JSON Schema→平台执行**的对话协议（全文最值得逐字复现的部位）；"数据全部内置"思想。
- **不抄**：流域划分算法（近岸是网格+开边界，逻辑不同）；概念性水文模型本身。

## 2.2 model-agent（github.com/emmashie/model-agent）★★★★★ 代码模板
- 五工种智能体架构（见 1.2②）；自然语言→1–2 分钟出可运行 ROMS 模型。
- 数据管道：SRTM15+ 水深、GLORYS 温盐、ERA5 风场。
- **抄**：agents/ 目录结构；工种拆分；参数建议+确认交互；地名知识表（换成渤海湾/湄洲湾/长江口）；
  LLM 失效回退（正则/表单）；tests/ 的 basic 级测试模式。
- **不抄**：ROMS 的 s 坐标、GLORYS 初始场逻辑。

## 2.3 SWMM 灌溉 LLM 论文 ★★★★ 集成模式
- **出处**：*Closed-loop autonomous scheduling of multi-level irrigation canal-gate systems using an LLM-agent framework*,
  Computers and Electronics in Agriculture, 2026。
- **抄**：**Delft3D 包成 MCP server**；**CA 约束增强提示**；LLM 对比结论（Qwen3-32B 均衡最优）。

## 2.4 Yan et al. 2026（*GRL*, DOI 10.1029/2025GL119814）★★★★ 论文骨架
- 六级自主性框架 + AQUAH（L4）。**抄**：方法学骨架"定义框架→定位→实现→案例证明"。
- 注意：Wiley 正文需校内权限精读六级定义原文。

## 2.5 HydroSuite-AI（Pursnani et al.；2024 EarthArXiv / 2026 正式发表）★★★★
- LLM 助手集成 HydroLang/HydroCompute/HydroRTC：问答＋代码生成＋在线执行。
- **抄**：MVP 轻量路线（Jupyter 里 LLM 生成并执行 HYDROLIB-core 调用代码）；API 文档挂上下文的工程方式。
- **不抄**：作为最终形态（仍有编程门槛，最终走 HydroCraft 式表单+JSON）。

## 2.6 HydroLLM（Kizilkaya, Sermet, Demir; *J. Hydroinformatics* 27(10) 2025）★★★
- ~8800 条 QA 对流水线。**抄**：造水动力版评测集；"先考试再上岗"的底座选型法。
- 配套：arXiv 2512.03672（主流 LLM 水利知识评测）——选型可先读其结论。

## 2.7 HydroAIM（2026）★★ grounding 口径。

## 2.8 Deltares 工具链（地基，非论文）★★★★★

| 工具 | 作用 | 获取 |
|---|---|---|
| HydroMT | 官方模型自动构建框架（YAML 声明数据→生成模型） | github.com/Deltares/hydromt |
| HydroMT-Delft3D FM 插件 | 直接生成/修改 D-Flow FM 模型 | deltares.github.io/hydromt_delft3dfm |
| HYDROLIB-core | 官方 Python 读写 D-Flow FM 输入(.mdu 等) | github.com/Deltares/HYDROLIB-core |
| pyDelft3D-FLOW (JulesBlm) | 读写 Delft3D4 的 .mdf+attribute 文件 | github.com/JulesBlm/pyDelft3D-FLOW |
| 数据源 | GEBCO/SRTM15+ 地形、TPXO9/FES2014 潮汐、ERA5 气象、GRDC 径流、GLORYS 温盐 | 各自 API |
| 率定 | PEST(Python 绑定)/DDS/SCE-UA/DREAM | 开源 |
| 官方资料 | Delft3D-FLOW/WAVE 手册+tutorials；Deltares 免费 3.5h Python 建模课 | content.oss.deltares.nl |

---

# 第三部分　复现路径（8 个 Sprint，约 8 个月）

## Sprint 0（第 1–2 周）：两个决策＋选型考试
1. **模型路线**：主线 **D-Flow FM**（HydroMT/HYDROLIB-core 全开源栈，agent 最好操纵）；
   兼容线 **Delft3D4**（pyDelft3D-FLOW 写 mdf；先只支持矩形网格，曲网放后期）。
2. **底座考试**：用 HydroLLM 方法出 100–200 道水动力建模题，考 GLM/Qwen/DeepSeek 等（参考 SWMM 论文：Qwen3-32B 大概率胜出）。
3. **精读**：HydroCraft 全文（JSON 协议）、model-agent 源码（agents/ 五个文件）、HydroSuite-AI 预印本 → 产出方案书。

## Sprint 1–2（第 3–8 周）：工具层（无 LLM，纯确定性代码）
建 `delft3d_agent_tools` 仓库（目录结构照抄 model-agent），实现 9 个原子工具：

| # | 工具 | 输入→输出 | 技术 |
|---|---|---|---|
| T1 | 区位解析 | 地名/bbox → lon/lat 范围 | geopandas＋地名表＋地图框选 |
| T2 | 网格生成 | bbox,分辨率 → 网格 | HydroMT-Delft3DFM / 简版矩形网格 |
| T3 | 地形插值 | bbox,网格 → 水深 | GEBCO/SRTM15+ → 插值 → 干湿单元 |
| T4 | **潮汐开边界** ★ | 边界段+时段 → 分潮边界文件 | TPXO9/FES2014 调和分析（最关键） |
| T5 | 气象强迫 | bbox+时段 → 风场 | ERA5 (cdsapi) |
| T6 | 径流边界 | 河流列表+时段 → 径流文件 | GRDC/用户数据标准化 |
| T7 | 配置文件生成 | 全部 → .mdu 全套(或 .mdf) | HYDROLIB-core / pyDelft3D-FLOW |
| T8 | 运行监控 | 模型目录 → 状态/错误 | 调 dimr/delft3d4(Docker)，解析 .dia |
| T9 | 验证率定 | 观测+结果 → NSE/RMSE/潮差相位误差 | xarray+scipy+PEST/DDS |

- **验收（不接 LLM）**：CLI 一条命令 → 全自动产出可运行、算完的潮汐模型。

## Sprint 3–4（第 9–16 周）：Agent 层
1. **JSON Schema 对话协议**（照 HydroCraft），示例：
```json
{
  "task": "tidal_circulation",
  "region": {"name": "湄洲湾", "bbox": [118.9, 24.9, 119.3, 25.3]},
  "grid": {"type": "unstructured", "res_m": {"min": 150, "max": 400}},
  "bathymetry": {"source": "GEBCO2024", "min_depth_m": 5},
  "open_boundary": {"type": "tidal",
    "constituents": ["M2","S2","N2","K1","O1","P1","K2","Q1"], "source": "TPXO9"},
  "forcing": {"wind": {"source": "ERA5", "on": true},
    "rivers": [{"name": "...", "q_m3s": null}]},
  "time": {"start": "2024-06-01T00:00", "end": "2024-06-16T00:00", "dt_s": 60},
  "physics": {"roughness": {"type": "Manning", "value": 0.025}, "salinity_temp": false},
  "outputs": {"interval_min": 10, "variables": ["water_level","velocity"],
    "stations": [{"lon": 119.1, "lat": 25.1}]}
}
```
2. **系统提示词**：角色(近岸水动力建模工程师)＋参数知识表＋CA 红线约束段。
3. **交互**：多轮补全 → 参数建议 Y/n 确认 → JSON → **schema+物理规则双重校验**（校验不过永不进工具，防幻觉）。
4. **自纠错闭环**：运行失败→解析 .dia→LLM 结合错误知识库诊断→改参重跑（≤5 轮）→仍败转人工。
5. 可选：T1–T9 封装成 **MCP server** 接 Dify/自家前端。
- **验收**："帮我建一个湄洲湾 2024 年 6 月大潮期潮流模型，网格 200 米" → 聊天出模型 → 跑通出图。

## Sprint 5–6（第 17–24 周）：Web 平台（HydroCraft 化）
- 前端：Vue/Next.js＋地图框选＋对话主界面＋任务进度＋结果画板。
- 后端：FastAPI＋Celery/Redis 队列；Docker 隔离每次运行；用户数据上传标准化接入器
  （先支持：潮位站数据表、.dep 地形文件两种格式）。
- 账号系统＋案例库（一键复用历史案例）；Linux 服务器部署。

## Sprint 7–8（第 25–32 周）：验证与论文
**验证设计（论文核心）**：
1. 盲测基准：Delft3D 官方 tutorials 2–3 个 ＋ 你们组已人工建模且有观测的近岸/河口案例 2–3 个；
2. 指标：模型质量（NSE/RMSE/潮差与相位误差 vs 人工模型）＋**效率对比（人工天 vs AI 分钟）**＋用户实验（10–15 名新手任务成功率）；
3. 按 Yan 六级框架声明当前自主等级（L3–L4）及下一级路径。

**论文打包（两篇）**：
- **论文 A（方法）**：*A natural-language agent system for intelligent Delft3D nearshore model construction*
  → Environmental Modelling & Software / Ocean Modelling / J. Hydrology
- **论文 B（评测）**：水动力建模问答基准（Sprint 0 考题扩到 ~2000 条）
  → Environmental Data Science / J. Hydroinformatics

---

# 第四部分　现在就要拍板的 6 件事

1. **模型路线**：D-Flow FM 为主、Delft3D4 兼容（建议双线）；
2. **LLM 部署**：公有 API（GLM/Qwen）还是本地部署（数据敏感？有 GPU 否？）；
3. **数据许可**：TPXO 需申请授权，FES2014 公开，Copernicus 需注册——按课题海域核对；
4. **算力**：服务器配置（每模型一个容器，先按 3–5 并发用户估）；
5. **基准弹药**：清点组内已有观测潮位/流速数据和已发表建模案例；
6. **分工**：2 人起步（1 人工具层、1 人 Agent+提示词），前端后置到 Sprint 5；标准 4 人配置为
   工具层/Agent/前端平台/案例验证各 1 人。

# 第五部分　风险与对策

| 风险 | 对策 |
|---|---|
| LLM 幻觉乱填参数 | 数值全部由确定性代码按物理规则算（如 dt 由 Courant 上限反算）；LLM 只选类型/传值；schema 校验不过就拒绝执行 |
| 免费地形/潮汐分辨率不足内湾精度 | 预留用户上传接口（海图/实测站），平台负责标准化接入 |
| Delft3D4 曲线网格自动化难 | MVP 只做矩形/非结构网格；曲网参数化 RGFGRID 放后期 |
| 软件依赖 Windows | D-Flow FM/HydroMT 全跨平台；Delft3D4 有官方 Linux 开源版可编译 |
| 百度网盘式数据分发不稳定 | 建组内 NAS/服务器统一存放软件包与数据 |

---

# 第六部分　AI 训练策略（练什么、怎么练、怎么验收）

> 核心结论：本系统**不从零训练大模型**。HydroCraft 用的也是现成 GLM-4（API 调用）。
> 分四层武装 AI：①提示词工程（必做，零训练）→ ②RAG 检索（必做，零训练，建向量索引）→
> ③LoRA 微调（二期可选）→ ④从零预训练（不做）。Sprint 0–6 全程零训练，
> 提示词+RAG 先跑通并积累真实轨迹数据，数据攒够再进第三层。

## 6.1 训练目标：4 项能力 → 4 类数据

| 能力 | 触发场景 | 训练数据形态 | 目标条数 |
|---|---|---|---|
| A. 参数抽取 | "建湄洲湾潮流模型" → JSON 参数单 | (需求原文，标准 JSON) | 7000（含 1000 多轮） |
| B. 错误诊断 | .dia 日志 → 原因+修复 | (错误日志，诊断+修法) | 2500 |
| C. 参数建议 | 任务类型 → 参数+物理理由 | (任务，推荐+推理依据) | 1500 |
| D. 知识问答 | ".bcc 是干嘛的" | (问题，答案) | 9000 |
| **合计** | | | **≈20000，留出 1600 考试** |

## 6.2 数据来源与构造方法

| 来源 | 用途 |
|---|---|
| Delft3D-FLOW/WAVE User Manual（700+ 页） | D 主体；A 的参数含义 |
| 官方 tutorials（~10 案例）+ 示例 mdf/mdu | A/C 真实范例 |
| oss.deltares.nl 论坛/FAQ | B（ Kro错误金矿） |
| 组内人工建模历史资料（报告/配置文件/审稿意见） | C 专家经验 |
| Sprint 1–3 跑真案例的存档轨迹 | A/B/C 全部（**最有价值**） |

**构造要点（防 AI 编错）**：
- **A 类**：场景母件池（50 研究区 × 时段 × 精度 × 进程开关）合成 6000+ 需求措辞（故意口语化）；
  **标准 JSON 由工具链真跑一遍反推生成**（工具跑得通=JSON 是对的），杜绝"看起来对其实算不动"；
  另配 1000 条多轮对话版（追问补全→出 JSON）。
- **B 类**：造假真错误——故意构造非法配置（dt 超 Courant、糙率为负、边界不闭合…）真跑收集 .dia 真报错 300–500 条
  → 整理 60–80 种错字典 → LLM 写变体扩充到 2500 条。
- **C 类**：参数知识表(参数→典型值→理由)三列直接展开，量可少，此能力提示词+RAG 也能撑八成。
- **D 类**：HydroLLM 流水线——手册切 300 字小节 → 每节让大模型出 2–3 题（多选/判断/开放，附答案出处页码）→ 人工校验 10%。

## 6.3 数据格式（JSONL 样例）

```json
{"conversations": [
  {"from": "system", "value": "你是近岸水动力建模工程师...（系统提示词全文）"},
  {"from": "human", "value": "帮我建湄洲湾2024年6月大潮期潮流模型，网格200米，要风场"},
  {"from": "assistant", "value": "{\"task\":\"tidal_circulation\",\"region\":{\"name\":\"湄洲湾\",
    \"bbox\":[118.9,24.9,119.3,25.3]},\"grid\":{\"res_m\":200},
    \"open_boundary\":{\"type\":\"tidal\",\"constituents\":[\"M2\",\"S2\",\"N2\",\"K1\",\"O1\",\"P1\",\"K2\",\"Q1\"],\"source\":\"TPXO9\"},
    \"forcing\":{\"wind\":{\"source\":\"ERA5\",\"on\":true}},
    \"time\":{\"start\":\"2024-06-07T00:00\",\"end\":\"2024-06-12T00:00\",\"dt_s\":180},
    \"outputs\":{\"interval_min\":10,\"variables\":[\"water_level\",\"velocity\"]}}"}
]}
```

## 6.4 训练执行

| 项 | 设置 |
|---|---|
| 底座 | Qwen3-32B-Instruct（开源可自部署） |
| 框架 | LLaMA-Factory（或 ms-swift） |
| 方法 | LoRA：rank=32, alpha=64, dropout=0.05, target=all |
| 数据 | delft3d_sft_v1，cutoff_len 4096，template qwen3 |
| 超参 | lr 1e-4 cosine、warmup 3%、epoch=2（宁少勿多防背题）、有效 batch 16、bf16 |
| 硬件 | 1×A100/H20 80G（或 2×4090QLoRA；也可租云，单次几百元） |
| 时长 | 2 万条×2ep×4K ≈ 10–20 小时（含调试留 2–3 天） |
| 二期可选 | DPO 偏好训练：用（错误修复，正确修复）真对 2000 对，同 LoRA，数小时 |

**三条纪律**：
1. **防考密闭**：留出的 1600 条物理隔离、只读权限，永不进训练；
2. **版本对齐**：数据标注基于"Delft3D 4.07 + D-Flow FM 2026.01"，版本升级后重建；
3. **错标熔断**：任何 JSON 后被证明跑不通即从训练集删除并记录原因。

## 6.5 训后验收（硬指标）

| 指标 | 及格线 |
|---|---|
| JSON 格式合规率 | 100%（300 条留出全过 schema） |
| 参数抽取准确率 | ≥95%（对比工具反推 ground truth） |
| 建模闭环跑通率 | ≥90%（20 个全新研究区全流程） |
| 潮汐案例质量 | NSE(水位) ≥0.85（对照人工模型） |
| 错误诊断命中率 | ≥85%（200 条留出报错） |
| D 考试准确率 | ≥85%（封存 1000 题） |
| **必须打赢基线** | SFT 版 vs 纯提示词+RAG 版对比 = **论文 B 主图** |

## 6.6 训练时间线

- **Sprint 1–4**：零训练（提示词+RAG 版上线），同步存档每次真实运行的 (需求, JSON, 日志)；
- **Sprint 5–6**：数据冻结封版 → 按 6.1–6.4 配方启动 LoRA；
- **第 7 个月起**：SFT 模型接管 Agent 大脑；此后每季度用新增案例增量微调一版；
- **永远不做**：从零预训练、全量微调、亿级自标数据。

**成本**：1 张 GPU（买 4090 或租 A100）＋ 3–4 周数据工程 ＋ 10–20 小时算时 ＋ 2 周评测。
**定位**：训练是改良附件，不是主战场——主战场永远是工具链（T1–T9）和数据质量。

---

*文档版本：2026-10-23 完全体（含智能体全景解析、文献逐篇解析、8-Sprint 复现路径、AI 训练策略）*
