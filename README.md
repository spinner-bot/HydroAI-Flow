<img src="assets/hero.svg" alt="HydroAI Flow — AI-assisted Hydrodynamic Modeling &amp; Simulation" width="100%">

# HydroAI Flow

AI-assisted Hydrodynamic Modeling & Simulation。

本项目探索使用 AI 代理模型加速水动力数值模拟中的重复计算与参数率定，缩短“调整参数 → 运行模拟 → 对比结果”的反馈周期。

> 当前阶段：前期调研、问题定义与可行性验证。

## 项目目标

项目暂不预设具体研究对象和模型路线。首先通过前期调研明确研究场景、目标变量、数据条件、传统模拟流程和可验证的 baseline，再进入模型实现。

当前围绕三条主线推进：

| 主线 | 关注内容 |
| --- | --- |
| 数值模拟 | Delft3D 算例、输入参数、输出变量与计算瓶颈 |
| 代理模型 | AI 替代的计算环节、输入输出形式与误差边界 |
| 参数率定 | 历史数据、目标函数与自动化调参流程 |

## 仓库结构

```text
.
├── assets/                  # README、演示和文档使用的静态资源
├── configs/                 # 实验与训练配置模板
├── data/                    # 数据分层目录（默认不提交原始数据）
│   ├── raw/                 # 原始数据，只读保存
│   ├── interim/             # 清洗和转换中的中间数据
│   └── processed/           # 可供实验使用的数据集
├── docs/
│   ├── design/              # 已确认的设计方案
│   ├── req/                 # 需求与问题定义
│   ├── spec/                # 开发与设计规范
│   └── decisions/           # 重要技术决策记录
├── experiments/             # 实验方案、运行记录与结果说明
├── meeting/                 # 组会材料与讨论记录
├── notebooks/               # 探索性分析和可复现实验笔记本
├── pre-research/            # 前期调研：文献、领域、数据与工具调研
├── scripts/                 # 数据处理、评估和辅助脚本
├── src/                     # 项目源码
└── tests/                   # 自动化测试
```

## 推荐工作顺序

1. 在 [`pre-research/`](pre-research/) 收集并整理外部资料、已有方案、数据源和工具信息。
2. 在 [`meeting/`](meeting/) 讨论并确认研究对象、预测变量、数据范围和评价指标。
3. 将已确认内容沉淀到 [`docs/req/`](docs/req/)，并在 [`docs/decisions/`](docs/decisions/) 记录关键取舍。
4. 设计 baseline 和数据切分方案，在 [`experiments/`](experiments/) 中留下可复现的实验记录。
5. 再开始实现 [`src/`](src/)、[`scripts/`](scripts/) 和 [`tests/`](tests/) 中的代码。

## 开发约定

开始前请阅读 [`docs/spec/`](docs/spec/)。前端或可视化实现遵循“先设计、后编码”；数据、实验结果和本地环境文件按 `.gitignore` 规则管理。

项目尚未形成稳定 API 或模型版本。研究结论、数据来源和实验结果应优先记录在文档中，再进入代码实现。
