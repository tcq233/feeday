## 版本更新


- **2025/09/08** IndexTTS-2 发布（首个支持精确合成时长控制的自回归零样本文本转语音模型）  
- **2025/05/14** IndexTTS-1.5 发布（提升模型稳定性和英文表现）  
- **2025/03/25** IndexTTS-1.0 发布（开放权重和推理代码）  
- **2025/02/12** 论文提交至 arXiv，并发布 Demo 与测试集  

---

<div style="text-align:left">
  <a href='https://arxiv.org/abs/2506.21619'>
    <img src='https://img.shields.io/badge/ArXiv-2506.21619-red?logo=arxiv'/>
  </a>
  <br/>
  <a href='https://github.com/index-tts/index-tts'>
    <img src='https://img.shields.io/badge/GitHub-Code-orange?logo=github'/>
  </a>
  <a href='https://index-tts.github.io/index-tts2.github.io/'>
    <img src='https://img.shields.io/badge/GitHub-Demo-orange?logo=github'/>
  </a>
  <br/>
  <a href='https://huggingface.co/spaces/IndexTeam/IndexTTS-2-Demo'>
    <img src='https://img.shields.io/badge/HuggingFace-Demo-blue?logo=huggingface'/>
  </a>
  <a href='https://huggingface.co/IndexTeam/IndexTTS-2'>
    <img src='https://img.shields.io/badge/HuggingFace-Model-blue?logo=huggingface' />
  </a>
  <br/>
  <a href='https://modelscope.cn/studios/IndexTeam/IndexTTS-2-Demo'>
    <img src='https://img.shields.io/badge/ModelScope-Demo-purple?logo=modelscope'/>
  </>
  <a href='https://modelscope.cn/models/IndexTeam/IndexTTS-2'>
    <img src='https://img.shields.io/badge/ModelScope-Model-purple?logo=modelscope'/>
  </a>
</div>

---

## 模型下载

| **HuggingFace**                                          | **ModelScope** |
|----------------------------------------------------------|----------------------------------------------------------|
| [IndexTTS-2](https://huggingface.co/IndexTeam/IndexTTS-2) | [IndexTTS-2](https://modelscope.cn/models/IndexTeam/IndexTTS-2) |
| [IndexTTS-1.5](https://huggingface.co/IndexTeam/IndexTTS-1.5) | [IndexTTS-1.5](https://modelscope.cn/models/IndexTeam/IndexTTS-1.5) |
| [IndexTTS](https://huggingface.co/IndexTeam/Index-TTS) | [IndexTTS](https://modelscope.cn/models/IndexTeam/Index-TTS) |

---


##  社区支持

* QQ 群：553460296 (No.1) / 663272642 (No.4)
* Discord：[Join](https://discord.gg/uT32E7KDmy)
* Email：[indexspeech@bilibili.com](mailto:indexspeech@bilibili.com)
* 官方仓库：[https://github.com/index-tts/index-tts](https://github.com/index-tts/index-tts)

---

## 论文概览

IndexTTS2：情感表达和持续时间控制的自动回归零样本文本转语音的突破

[周思怡](https://arxiv.org/search/cs?searchtype=author&query=Zhou,+S)， [周义全](https://arxiv.org/search/cs?searchtype=author&query=Zhou,+Y)， [何毅](https://arxiv.org/search/cs?searchtype=author&query=He,+Y)， [周勋](https://arxiv.org/search/cs?searchtype=author&query=Zhou,+X)， [王金超](https://arxiv.org/search/cs?searchtype=author&query=Wang,+J)， [邓伟](https://arxiv.org/search/cs?searchtype=author&query=Deng,+W)， [舒景晨](https://arxiv.org/search/cs?searchtype=author&query=Shu,+J)

[v2] 2025 年 9 月 3 日星期三 10：46：35 UTC （1,632 KB）

现有的自回归大规模文本转语音（TTS）模型在语音自然性方面具有优势，但其逐个标记的生成机制使得合成语音的持续时间难以精确控制。这在需要严格视听同步的应用（例如视频配音）中成为一个重大限制。本文介绍了IndexTTS2，该方法提出了一种新颖、通用、自回归的语音时长控制模型友好方法。该方法支持两种生成模式：一种明确指定生成的标记数量以精确控制语音持续时间;另一种以自回归的方式自由生成语音，无需指定标记的数量，同时忠实地再现输入提示的韵律特征。此外，IndexTTS2 实现了情感表达和说话者身份之间的解开，实现了对音色和情感的独立控制。在零样本设置中，模型可以准确地重建目标音色（来自音色提示），同时完美再现指定的情感音调（来自风格提示）。为了提高高度情感表达中的语音清晰度，我们结合了GPT潜在表示，并设计了一种新颖的三阶段训练范式，以提高生成语音的稳定性。此外，为了降低情绪控制的门槛，我们通过微调Qwen3设计了一种基于文本描述的软指令机制，有效地引导了具有所需情感取向的语音生成。最后，在多个数据集上的实验结果表明，IndexTTS2在单词错误率、说话人相似性和情感保真度方面优于最先进的零样本TTS模型。

---


## 测试分数

GPT识别转换未核对

| 数据集              | 模型        | 说话人相似度 (SS↑) | 词错误率 WER(%)↓ | 自然度 SMOS↑ | 节奏 PMOS↑ | 整体质量 QMOS↑ |
|---------------------|-------------|--------------------|-----------------|--------------|------------|----------------|
| **LibriSpeech test-clean** | Ground Truth | 0.833 | 3.405 | 4.02±0.22 | 3.85±0.26 | 4.23±0.12 |
|                     | MaskGCT     | 0.790 | 7.759 | 4.12±0.09 | 3.98±0.11 | 4.19±0.19 |
|                     | F5-TTS      | 0.821 | 8.044 | 4.08±0.21 | 3.73±0.27 | 4.12±0.13 |
|                     | CosyVoice2  | 0.843 | 5.999 | 4.02±0.22 | 4.04±0.28 | 4.17±0.25 |
|                     | SparkTTS    | 0.756 | 8.843 | 4.06±0.20 | 3.94±0.21 | 4.15±0.16 |
|                     | IndexTTS    | 0.819 | 3.436 | 4.23±0.14 | 4.02±0.18 | 4.29±0.22 |
|                     | IndexTTS2   | 0.870 | 3.115 | 4.44±0.12 | 4.12±0.17 | 4.29±0.14 |
|                     | - GPT latent| **0.887** | 3.334 | **4.33±0.10** | 4.10±0.12 | 4.17±0.22 |
| **SeedTTS test-en** | Ground Truth | 0.820 | 1.897 | 4.21±0.19 | 4.06±0.25 | 4.40±0.15 |
|                     | MaskGCT     | 0.824 | 2.530 | 4.35±0.20 | 4.02±0.24 | 4.50±0.17 |
|                     | F5-TTS      | 0.803 | 1.937 | 4.44±0.14 | 4.06±0.21 | 4.40±0.12 |
|                     | CosyVoice2  | 0.794 | 3.277 | 4.42±0.26 | 3.96±0.24 | 4.52±0.15 |
|                     | SparkTTS    | 0.755 | 1.543 | 3.96±0.23 | 4.12±0.22 | 3.89±0.20 |
|                     | IndexTTS    | 0.808 | 1.844 | **4.67±0.16** | **4.52±0.14** | **4.67±0.19** |
|                     | IndexTTS2   | 0.860 | **1.521** | 4.42±0.19 | 4.40±0.13 | 4.48±0.15 |
|                     | - GPT latent| **0.879** | 1.616 | 4.40±0.22 | 4.31±0.17 | 4.42±0.20 |
| **SeedTTS test-zh** | Ground Truth | 0.776 | 1.254 | 3.81±0.24 | 4.04±0.28 | 4.21±0.26 |
|                     | MaskGCT     | 0.807 | 2.447 | 3.94±0.22 | 3.54±0.26 | 4.15±0.15 |
|                     | F5-TTS      | 0.844 | 1.514 | 4.19±0.21 | 3.88±0.23 | 4.38±0.16 |
|                     | CosyVoice2  | 0.846 | 1.451 | 4.12±0.25 | 4.33±0.19 | 4.31±0.21 |
|                     | SparkTTS    | 0.683 | 2.636 | 3.65±0.22 | 4.10±0.25 | 3.79±0.18 |
|                     | IndexTTS    | 0.781 | 1.097 | 4.10±0.09 | 3.73±0.23 | 4.33±0.20 |
|                     | IndexTTS2   | 0.865 | **1.008** | **4.44±0.17** | **4.46±0.11** | **4.54±0.08** |
|                     | - GPT latent| **0.890** | 1.261 | 4.44±0.13 | 4.33±0.15 | 4.48±0.17 |
| **AIShell-1 test**  | Ground Truth | 0.847 | 1.840 | 4.27±0.19 | 3.83±0.25 | 4.25±0.14 |
|                     | MaskGCT     | 0.598 | 4.930 | 3.92±0.03 | 2.67±0.08 | 3.67±0.07 |
|                     | F5-TTS      | 0.831 | 3.671 | 4.17±0.30 | 3.60±0.25 | 4.25±0.22 |
|                     | CosyVoice2  | 0.834 | 1.967 | 4.21±0.23 | 4.33±0.19 | 4.37±0.20 |
|                     | SparkTTS    | 0.593 | 1.743 | 3.48±0.22 | 3.96±0.16 | 3.79±0.20 |
|                     | IndexTTS    | 0.794 | 1.478 | 4.48±0.18 | **4.25±0.19** | 4.49±0.15 |
|                     | IndexTTS2   | 0.843 | **1.516** | **4.54±0.11** | 4.42±0.17 | **4.52±0.17** |
|                     | - GPT latent| **0.868** | 1.791 | 4.33±0.22 | 4.27±0.26 | 4.40±0.19 |




## 相关数据

| 名称 | 来源 | 类型 | 发布时间 | 用途 / 备注 |
|------|------|------|----------|-------------|
| DiDiSpeech | [GitHub](https://github.com/athena-team/DiDiSpeech) | 中文普通话语音 | 2021 | 部分语料被采样用于 SeedTTS test-zh 基准测试 |
| ESD (Emotional Speech Dataset) | [GitHub](https://github.com/HLTSingapore/ESD) | 多语言情感语音数据集 | 2020 | 提供 29 小时情感语音，用于增强情感建模 |
| Common Voice | [Mozilla Common Voice](https://commonvoice.mozilla.org/) | 多语言众包语音 | 2017 | 部分语料被采样用于 SeedTTS test-en 基准测试 |
| AISHELL-1 | [OpenSLR](https://www.openslr.org/33/) | 中文普通话语音 | 2017 | 随机抽取 1,000 条语音作为测试集 |
| LibriSpeech | [OpenSLR](https://www.openslr.org/12/) | 英语朗读语音 (有声书) | 2015 | 随机抽取 test-clean 子集，用于英语语音评测 |

---

##  部署方法

#### 1. 安装依赖

```bash
# 安装 uv (推荐的依赖管理工具)
pip install -U uv

# 克隆项目
git clone https://github.com/index-tts/index-tts.git && cd index-tts
git lfs install
git lfs pull

# 同步依赖
uv sync --all-extras
# 如果网络慢，可用国内镜像：
uv sync --all-extras --default-index "https://mirrors.aliyun.com/pypi/simple"
```

#### 2. 下载模型

```bash
# HuggingFace
uv tool install "huggingface-hub[cli,hf_xet]"
hf download IndexTeam/IndexTTS-2 --local-dir=checkpoints

# 或 ModelScope
uv tool install "modelscope"
modelscope download --model IndexTeam/IndexTTS-2 --local_dir checkpoints

# 或 直接用 uvx 方式临时调用
uvx modelscope download --model IndexTeam/IndexTTS-2 --local_dir checkpoints

```

#### 3. 检查 GPU 环境

```bash
uv run tools/gpu_check.py
```

#### 4. 启动 WebUI

```bash
uv run webui.py
# 浏览器访问 http://127.0.0.1:7860
```

#### 5. Python 调用示例

```python
from indextts.infer_v2 import IndexTTS2

tts = IndexTTS2(cfg_path="checkpoints/config.yaml", model_dir="checkpoints", use_fp16=True)
text = "Translate for me, what is a surprise!"
tts.infer(spk_audio_prompt='examples/voice_01.wav', text=text, output_path="gen.wav", verbose=True)
```

## 运行代码

```
# .vscode/preview.yml
autoOpen: true # 打开工作空间时是否自动开启所有应用的预览
apps:
  - port: 7860 # 应用的端口
    run: uv run webui.py
    root: ./index-tts # 应用的启动目录
    name: IndexTTS2  # 应用名称
    description: IndexTTS2 # 应用描述
    autoOpen: true # 打开工作空间时是否自动运行命令（优先级高于根级 autoOpen）
    autoPreview: true # 是否自动打开预览, 若无则默认为true
```

## 批量生成

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
IndexTTS2 批量生成（离线/脚本版）
- 自动安装依赖（含 modelscope / transformers / accelerate / WeTextProcessing / descript-audiotools）
- 读取 in/1.txt（每行一句）和 in/ 下全部参考音频（wav/mp3/flac/m4a/ogg）
- 做“音频 × 文本”笛卡尔积，生成到 out/
默认路径：
  --in_dir    /workspace/index-tts/in
  --out_dir   /workspace/index-tts/out
  --model_dir /workspace/index-tts/checkpoints
"""
import os, sys, time, subprocess, importlib
from pathlib import Path
import argparse

# -------------------- 依赖自动安装（含版本对齐 & 特殊回退） --------------------
PINNED_PKGS = {
    # 基础
    "numpy": "numpy>=1.24",
    "scipy": "scipy>=1.10",
    "soundfile": "soundfile>=0.12",
    "librosa": "librosa>=0.10",
    "einops": "einops>=0.6",
    "sentencepiece": "sentencepiece>=0.1.99",
    "safetensors": "safetensors>=0.4.2",
    "tqdm": "tqdm>=4.66",
    "packaging": "packaging>=23.2",
    "omegaconf": "omegaconf>=2.3.0",
    # 关键三件套
    "transformers": "transformers>=4.44.2",
    "accelerate": "accelerate>=0.26.0",
    "modelscope": "modelscope>=1.19.0",
    # DAC 依赖（导入名 audiotools；特殊逻辑见下）
    "descript-audiotools": "descript-audiotools",
    # 常见封装辅助（可选）
    "av": "av>=12.0.0",
    "ffmpeg-python": "ffmpeg-python>=0.2.0",
    # 中文文本正则化（提供 tn.chinese.normalizer）
    "WeTextProcessing": "WeTextProcessing",
}

IMPORT_NAME_MAP = {
    "descript-audiotools": "audiotools",
    "ffmpeg-python": "ffmpeg",
    "WeTextProcessing": "tn",   # 安装包名 WeTextProcessing，导入名 tn
}

ALIYUN = "https://mirrors.aliyun.com/pypi/simple/"
PYPI = "https://pypi.org/simple/"

def _import_name_of(spec_key: str) -> str:
    return IMPORT_NAME_MAP.get(spec_key, spec_key.split("[")[0].split("==")[0].split(">=")[0])

def pip_install(args_list):
    print("[PIP]", " ".join(map(str, args_list)))
    subprocess.check_call(args_list)

def install_general(spec_key: str, spec_value: str):
    """通用安装：默认索引 -> 阿里镜像"""
    mod = _import_name_of(spec_key)
    try:
        importlib.import_module(mod)
        return
    except Exception:
        pass
    try:
        pip_install([sys.executable, "-m", "pip", "install", "-U", spec_value])
        importlib.import_module(mod); return
    except Exception as e1:
        print(f"[WARN] install {spec_value} on default index failed: {e1}")
    try:
        pip_install([sys.executable, "-m", "pip", "install", "-U", spec_value, "-i", ALIYUN])
        importlib.import_module(mod); return
    except Exception as e2:
        print(f"[WARN] install {spec_value} on Aliyun failed: {e2}")
        raise

def install_descript_audiotools():
    """descript-audiotools 特殊处理：
       1) PyPI 最新；2) PyPI 0.7.2；3) 阿里 0.7.2
    """
    mod = "audiotools"
    try:
        importlib.import_module(mod); return
    except Exception:
        pass
    for spec, idx in [
        ("descript-audiotools", PYPI),
        ("descript-audiotools==0.7.2", PYPI),
        ("descript-audiotools==0.7.2", ALIYUN),
    ]:
        try:
            pip_install([sys.executable, "-m", "pip", "install", "-U", spec, "-i", idx])
            importlib.import_module(mod); return
        except Exception as e:
            print(f"[WARN] {spec} via {idx} failed: {e}")
    raise RuntimeError("Failed to install descript-audiotools")

def install_wetextprocessing():
    """WeTextProcessing 提供 tn.*：
       1) PyPI 最新；2) 阿里最新；失败则报错
    """
    try:
        importlib.import_module("tn"); return
    except Exception:
        pass
    for idx in [PYPI, ALIYUN]:
        try:
            pip_install([sys.executable, "-m", "pip", "install", "-U", "WeTextProcessing", "-i", idx])
            importlib.import_module("tn"); return
        except Exception as e:
            print(f"[WARN] WeTextProcessing via {idx} failed: {e}")
    raise RuntimeError("Failed to install WeTextProcessing (tn)")

def ensure_deps():
    for k, v in PINNED_PKGS.items():
        if k == "descript-audiotools":
            install_descript_audiotools()
        elif k == "WeTextProcessing":
            install_wetextprocessing()
        else:
            install_general(k, v)
    # 关键：立刻验证 accelerate
    try:
        import accelerate  # noqa: F401
    except Exception as e:
        print("[FATAL] accelerate 仍不可用：", e)
        print("请退出当前进程后重跑本脚本；或手动执行：")
        print("  python -m pip install -U 'transformers>=4.44.2' 'accelerate>=0.26.0' 'modelscope>=1.19.0'")
        sys.exit(1)

ensure_deps()

# -------------------- 业务逻辑 --------------------
SCRIPT_DIR = Path(__file__).resolve().parent
sys.path.append(str(SCRIPT_DIR))
sys.path.append(str(SCRIPT_DIR / "indextts"))

from indextts.infer_v2 import IndexTTS2  # noqa: E402

def find_prompt_audios(in_dir: Path):
    exts = ["*.wav", "*.mp3", "*.flac", "*.m4a", "*.ogg"]
    files = []
    for p in exts: files += list(in_dir.glob(p))
    return sorted([p for p in files if p.is_file()], key=lambda x: x.name.lower())

def read_lines(txt: Path):
    with txt.open("r", encoding="utf-8") as f:
        return [ln.strip() for ln in f if ln.strip()]

def safe_stem(p: Path) -> str:
    return p.stem.replace(" ", "_").replace("/", "_").replace("\\", "_")[:80]

def detect_fp16() -> bool:
    try:
        import torch
        return torch.cuda.is_available()
    except Exception:
        return False

def main():
    parser = argparse.ArgumentParser(description="IndexTTS2 批量生成（音频 × 文本）")
    parser.add_argument("--in_dir", type=str, default="/workspace/index-tts/in", help="输入目录（含 1.txt 与参考音频）")
    parser.add_argument("--out_dir", type=str, default="/workspace/index-tts/out", help="输出目录")
    parser.add_argument("--model_dir", type=str, default="/workspace/index-tts/checkpoints", help="模型目录（含 config.yaml 等）")
    # 生成参数
    parser.add_argument("--max_text_tokens_per_segment", type=int, default=120)
    parser.add_argument("--do_sample", action="store_true", default=True)
    parser.add_argument("--top_p", type=float, default=0.8)
    parser.add_argument("--top_k", type=int, default=30)
    parser.add_argument("--temperature", type=float, default=0.8)
    parser.add_argument("--num_beams", type=int, default=3)
    parser.add_argument("--repetition_penalty", type=float, default=10.0)
    parser.add_argument("--max_mel_tokens", type=int, default=1500)
    args = parser.parse_args()

    in_dir = Path(args.in_dir).resolve()
    out_dir = Path(args.out_dir).resolve()
    model_dir = Path(args.model_dir).resolve()

    if not in_dir.exists():
        print(f"[ERROR] 输入目录不存在：{in_dir}"); sys.exit(1)
    txt = in_dir / "1.txt"
    if not txt.exists():
        print(f"[ERROR] 缺少文本文件：{txt}"); sys.exit(1)
    if not model_dir.exists():
        print(f"[ERROR] 模型目录不存在：{model_dir}"); sys.exit(1)

    prompts = find_prompt_audios(in_dir)
    texts = read_lines(txt)
    if not prompts:
        print(f"[ERROR] {in_dir} 下未找到音频（支持 wav/mp3/flac/m4a/ogg）"); sys.exit(1)
    if not texts:
        print(f"[ERROR] {txt} 为空"); sys.exit(1)

    out_dir.mkdir(parents=True, exist_ok=True)

    use_fp16 = detect_fp16()
    print(f"[INFO] Loading IndexTTS2 ... (fp16={use_fp16})")
    tts = IndexTTS2(
        cfg_path=str(model_dir / "config.yaml"),
        model_dir=str(model_dir),
        use_fp16=use_fp16,
        use_cuda_kernel=False,
        use_deepspeed=False,
    )
    print(f"[INFO] Model loaded. version={tts.model_version or '1.0'}")

    total = len(prompts) * len(texts)
    idx = 0
    for pa in prompts:
        base = safe_stem(pa)
        for li, text in enumerate(texts, 1):
            idx += 1
            ts = int(time.time() * 1000)
            out_path = out_dir / f"{base}__L{li:03d}__{ts}.wav"
            print(f"[{idx}/{total}] {pa.name} × L{li} -> {out_path.name}")
            try:
                tts.infer(
                    spk_audio_prompt=str(pa),
                    text=text,
                    output_path=str(out_path),
                    emo_audio_prompt=None,
                    emo_alpha=0.65,
                    emo_vector=None,
                    use_emo_text=False,
                    emo_text=None,
                    use_random=False,
                    verbose=False,
                    max_text_tokens_per_segment=int(args.max_text_tokens_per_segment),
                    do_sample=bool(args.do_sample),
                    top_p=float(args.top_p),
                    top_k=int(args.top_k) if int(args.top_k) > 0 else None,
                    temperature=float(args.temperature),
                    length_penalty=0.0,
                    num_beams=int(args.num_beams),
                    repetition_penalty=float(args.repetition_penalty),
                    max_mel_tokens=int(args.max_mel_tokens),
                )
            except Exception as e:
                print(f"[ERROR] 生成失败：{e}")

    print(f"[DONE] 共生成 {total} 条音频，输出目录：{out_dir}")

if __name__ == "__main__":
    main()
```