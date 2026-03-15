# ARIS 领域适配指南

本文档说明如何将 ARIS (Auto-claude-code-research-in-sleep) 适配到机器学习以外的其他研究领域，如 3D 打印、生物学、材料科学等。

## 核心思路

ARIS 的工作流分为三个部分：

| 部分 | 说明 | 适配难度 |
|------|------|----------|
| Idea/Review 部分 | 想法发现、审稿循环、文献调研 | 低 - 通用流程 |
| Experiment 部分 | 实验部署、运行、监控 | **高 - 需完全重写** |
| Paper 部分 | 论文写作、图表生成、模板 | 中 - 替换模板 |

**结论**：最核心的改动在 `run-experiment`，其他 SKILL 大多是通用流程。

---

## 需要修改的 SKILL

### 1. run-experiment（必须修改）

**现状**：只支持 GPU + Python + SSH + Screen

**适配方向**：

| 领域 | 需要添加的支持 |
|------|---------------|
| 3D 打印 | 3D 打印机 API (OctoPrint)、切片软件 (CuraEngine)、G-code 上传 |
| 生物学 | 显微镜控制、PCR 仪、离心机、流式细胞仪等设备接口 |
| 材料科学 | 材料合成设备、热处理炉、XRD/SEM 设备 |
| 机器人 | ROS 部署、硬件在环仿真 |

**示例：3D 打印实验流程**
```python
# 1. 检查打印机状态
curl http://octoprint/api/printer

# 2. 切片 STL → G-code
./CuraEngine -s settings.ini -o output.gcode input.stl

# 3. 上传并开始打印
curl -X POST http://octoprint/api/files/local/output.gcode -d "{"command":"select"}'
curl -X POST http://octoprint/api/job -d '{"command":"start"}'

# 4. 监控打印状态
curl http://octoprint/api/job
```

### 2. paper-write（必须修改）

**现状**：只有 ICLR/NeurIPS/ICML 模板

**需要替换的内容**：

1. **模板文件** - 替换为你的目标期刊模板
2. **Section 结构** - 不同领域有不同结构
   - ML 论文：Intro → Related Work → Method → Experiments → Conclusion
   - 生物论文：Intro → Methods → Results → Discussion → Conclusion
   - 材料论文：Intro → Experimental → Results → Discussion → Conclusion

**新增模板建议**：

| 领域 | 目标期刊/会议 |
|------|--------------|
| 3D 打印 | Additive Manufacturing, ASME Journal |
| 生物学 | Nature, Science, Cell, PNAS |
| 材料 | Advanced Materials, Nature Materials |
| 机器人 | ICRA, IROS, RSS |

### 3. paper-figure（可能需要修改）

**现状**：生成 matplotlib 图表（折线图、柱状图、热力图等）

**适配方向**：

| 领域 | 可能需要的新图表类型 |
|------|-------------------|
| 3D 打印 | STL 模型渲染图、打印参数曲线、层析图 |
| 生物学 | 胶图、流式细胞术图、显微图像、基因表达热图 |
| 材料 | XRD 图谱、SEM/TEM 图像、应力-应变曲线 |

### 4. research-lit（建议修改）

**现状**：搜索 arXiv、Semantic Scholar、Google Scholar

**适配方向**：

| 领域 | 建议的数据源 |
|------|-------------|
| 3D 打印 | IEEE Xplore, Springer Materials, Web of Science |
| 生物学 | PubMed, NCBI, UniProt, BioRxiv |
| 化学 | CAS, SciFinder, Reaxys |

---

## 需要新增的 SKILL

### 设备控制类（根据领域选择）

```
skills/
├── printer-control/           # 3D 打印：打印机 API 控制
│   └── SKILL.md
├── lab-equipment/           # 生物：实验室设备控制
│   └── SKILL.md
├── microscopy-control/       # 生物：显微镜/成像设备
│   └── SKILL.md
└── simulation-control/      # 通用：仿真软件控制
    └── SKILL.md
```

### 数据处理类

```
skills/
├── cad-import/              # 3D 打印：CAD 文件处理
│   └── SKILL.md
├── genome-analysis/         # 生物：基因组数据分析
│   └── SKILL.md
├── bio-image-analysis/      # 生物：生物图像处理
│   └── SKILL.md
└── material-property/       # 材料：材料性能数据库
    └── SKILL.md
```

---

## 适配检查清单

### 第一步：修改 run-experiment
- [ ] 确定实验运行方式（本地/远程）
- [ ] 确定设备控制方式（API/SSH/Serial）
- [ ] 添加设备状态检查逻辑
- [ ] 添加实验部署命令
- [ ] 添加结果收集逻辑

### 第二步：修改 paper-write
- [ ] 获取目标期刊的 LaTeX 模板
- [ ] 替换 `templates/` 目录下的模板
- [ ] 调整 section 结构
- [ ] 修改页面限制、引用格式等

### 第三步：修改 paper-figure（可选）
- [ ] 添加领域特定的图表生成逻辑
- [ ] 配置新的颜色方案/样式

### 第四步：修改 research-lit（可选）
- [ ] 添加领域专属的文献数据库
- [ ] 调整文献搜索关键词

---

## 最小改动示例

假设你要适配到 **3D 打印研究**，最少需要以下改动：

### 1. 修改 run-experiment

将 SSH+GPU 改为 3D 打印机控制：

```yaml
# 原始：GPU 实验
- ssh <server> nvidia-smi
- CUDA_VISIBLE_DEVICES=<gpu_id> python train.py

# 修改后：3D 打印实验
- curl http://<printer>/api/printer  # 检查状态
- ./CuraEngine -s print_settings.ini -o model.gcode model.stl
- curl -X POST http://<printer>/api/files/local/model.gcode -d '{"command":"select"}'
- curl -X POST http://<printer>/api/job -d '{"command":"start"}'
```

### 2. 替换 paper-write 模板

```
# 原始
templates/iclr2026.tex
templates/neurips2025.tex
templates/icml2025.tex

# 新增
templates/additive_manufacturing.tex  # Additive Manufacturing 期刊
templates/asme_journal.tex            # ASME 期刊
```

### 3. 可选：新增 printer-control

```yaml
# skills/printer-control/SKILL.md
---
name: printer-control
description: Control 3D printers via OctoPrint API
---

# Printer Control

## Constants

- PRINTER_URL: OctoPrint server URL
- API_KEY: OctoPrint API key

## Workflow

### 1. Check Status
curl http://PRINTER_URL/api/printer

### 2. Upload and Print
curl -X POST http://PRINTER_URL/api/files/local/model.gcode
curl -X POST http://PRINTER_URL/api/job -d '{"command":"start"}'

### 3. Monitor
curl http://PRINTER_URL/api/job
```

---

## 总结

| 改动级别 | 需要修改的内容 | 工作量 |
|----------|---------------|--------|
| 最小 | run-experiment + paper-write 模板 | 1-2 天 |
| 中等 | 上述 + paper-figure 图表类型 | 1 周 |
| 完整 | 上述 + 新增多个领域专用 SKILL | 2-4 周 |

**核心原则**：先让实验能跑起来，再让论文能写出来。
