<div align="center">

# AI 汉字老师 · ai-character-teacher

**输入一个或多个汉字，自动输出带「逐笔动画 + 逐笔语音讲解 + 男女声可选」的 MP4 教学视频。**

把"人工录屏 + 配音"变成"一条命令、几秒钟"的自动化产出。

![Status](https://img.shields.io/badge/status-active-brightgreen)
![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)
![Vue](https://img.shields.io/badge/Vue-3-42b883?logo=vuedotjs&logoColor=white)
![FFmpeg](https://img.shields.io/badge/FFmpeg-required-007808?logo=ffmpeg&logoColor=white)
![No GPU](https://img.shields.io/badge/GPU-not%20required-success)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blueviolet)

[它解决什么问题](#它解决什么问题) · [功能特性](#功能特性) · [快速开始](#快速开始) · [技术栈](#技术栈) · [路线图](#路线图) · [数据来源与许可](#数据来源与许可)

</div>

---

## 它解决什么问题

汉字笔顺教学视频的传统做法是"人工录屏 + 配音"，成本高、无法按需生成。本项目把它变成一次命令、几秒钟的自动化产出：

- **渲染自研**：直接消费 [Make Me a Hanzi](https://github.com/skishore/makemeahanzi) 的矢量笔画数据（`strokes` 闭合轮廓 + `medians` 运笔路径），逐帧栅格化，可精确控制"任意进度 `t` 的笔画生长"，因此能输出任意帧率的视频；
- **语音按笔讲解**："第 1 笔，点。"逐笔朗读，男 / 女声可选，语速与动画同步调速；
- **零 GPU 出片**：逐帧渲染是纯矢量绘制，CPU 即可（1080×1080 / ss=2 实测约 20~26 ms/帧，随笔画数线性增长）；GPU 仅在神经 TTS 推理时需要。

## 功能特性

| 能力 | 说明 |
| --- | --- |
| 逐笔动画 | 精确控制每一笔任意进度的生长，写到哪停在哪，帧率与速度自由可调 |
| 逐笔语音 | 逐笔朗读讲解，男 / 女声可选，语速与动画同步 |
| 多字批量 | 一次输入一个或多个汉字，批量产出成片 |
| 纯动画模式 | `--no-audio` 只出画面，便于二次配音或制作互动练习素材 |

- **质量门禁**：两条渲染硬不变量——同笔揭示区域随进度**只增不减**（单调性）、收尾与完整轮廓 **IoU ≥ 0.95**（收敛性），由 `scripts/check_reveal.py` 把关；
- **依赖极简**：核心仅 `numpy + Pillow + 静态 ffmpeg`，安装快、跑得动；
- **测试覆盖**：166 项单元与回归用例（e2e 默认跳过，可单独开启）。

## 快速开始

```bash
# 1) 安装依赖
python -m venv .venv && . .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.lock

# 2) 准备数据（约 31MB，详见 character-data/raw/README.md）

# 3) 建索引 + 笔画名称表（首次执行一次即可）
python scripts/build_indexes.py
python scripts/build_stroke_names.py

# 4) 生成视频
python cli.py --text 永 --voice female --out out/yong.mp4
python cli.py --text 中国人 --voice male --speed 1.2 --out out/zgr.mp4
python cli.py --text 一 --no-audio --out out/yi.mp4        # 纯动画
```

常用查询：

```bash
python cli.py --list-voices        # 可用音色与 TTS 通道
python cli.py --stats              # 数据集统计（字数 / 笔画名称层级分布）
python cli.py --text 永 --dry-run  # 只解析不生成
```

## 技术栈

| 方向 | 选型 |
| --- | --- |
| 渲染 | 自研矢量渲染器（SVG 解析 / 笔顺生长 / 时间轴 / 遮罩合成） |
| 视频合成 | FFmpeg（帧流直送，不落盘） |
| 语音 | Edge-TTS（开发演示）/ CosyVoice（Apache-2.0，商用主线）/ 云 TTS 兜底 |
| 服务 | FastAPI + 队列 Worker（cpu-worker / gpu-worker） |
| 前端 | Vue 3 + Vite + Element Plus（学习页 + 后台） |
| 存储 | MySQL · Redis · MinIO |

## 路线图

- [x] **V1** 渲染引擎 + 数据层 + 语音 + 编排（命令行出片）
- [x] 笔画揭示门禁（单调性 / 收敛性）与 166 项测试
- [x] 合规说明、数据来源、性能基线文档
- [ ] **V2** FastAPI 服务化与任务编排、神经 TTS
- [ ] **V3** Web 学习页与后台、队列 Worker、Docker 编排
- [ ] **V4** 商业落地：AI 批改、机构批量定制

## 数据来源与许可

- **字形与笔顺**：Make Me a Hanzi（`graphics.txt`），图形数据为 **Arphic Public License** 衍生的楷体字形 → **须署名**，商用前须法务复核（画面右下角默认渲染署名）。
- **拼音**：**pypinyin（MIT）**；`dictionary.txt`（LGPL-3.0+）仅用于内部交叉校验，**不进产品数据**。
- **笔画名称**：cnchar-order（MIT），与 Make Me a Hanzi 交叉校验，笔画数一致率 **99.94%**。
- 本仓库**暂未添加开源许可证（LICENSE）**；对外开源 / 商用前请先确定本项目自身许可。

详见 [`docs/数据来源.md`](数据来源.md)、[`docs/合规说明.md`](合规说明.md)、[`docs/拼音来源校验.md`](拼音来源校验.md)。

## 文档

| 文档 | 内容 |
| --- | --- |
| [docs/合规说明.md](合规说明.md) | 数据 / 字体 / TTS 的授权结论与商用风险 |
| [docs/数据来源.md](数据来源.md) | 数据集溯源、版本、校验与加工流程 |
| [docs/拼音来源校验.md](拼音来源校验.md) | pypinyin ⇄ dictionary 交叉校验与多音字复核 |
| [docs/环境清单.md](环境清单.md) | 运行环境与依赖版本、ffmpeg 能力验证 |
| [docs/性能基线.md](性能基线.md) | 渲染 / TTS / 合成的实测耗时与体积 |
| [docs/V1_验收.md](V1_验收.md) | V1 验收证据与演示覆盖 |
| [docs/宣传文案.md](宣传文案.md) | 面向公众号 / 社媒的中文宣传稿 |

## 贡献

欢迎提交 Issue 与 Pull Request。提交前建议先跑：

```bash
python scripts/check_env.py        # 环境与依赖自检
python scripts/check_reveal.py     # 渲染门禁
python -m pytest tests/ -q         # 单元 + 回归
```

<div align="center">

**如果这个项目对你有帮助，欢迎点一个 Star 支持我们。**

</div>

---

## 🏢 关于我们

<p align="center">
  <a href="http://www.net188.net">
    <img src="http://www.net188.net/images/logo1.png" alt="Net188 Logo" width="200" />
  </a>
</p>

<p align="center">
  <strong>Net188 · 互联网技术服务</strong>
</p>

<p align="center">
  专注于跨平台应用开发、AI Agent 集成与大模型应用落地。<br/>
  提供从产品设计、开发实施到部署运维的全栈技术解决方案。
</p>

<p align="center">
  🌐 <a href="http://www.net188.net"><strong>www.net188.net</strong></a>
</p>

---
