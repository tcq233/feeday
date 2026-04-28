## 按字幕切割音频视频
```
# -*- coding: utf-8 -*-
"""
cut_by_srt_auto.py
- 自动 pip 安装依赖 (srt, chardet, tqdm)
- 自动匹配置顶文件夹里的媒体文件 (mp4/mp3/wav/mkv等)
- 找到对应同名 .srt 文件
- 按字幕时间切片，并输出到 【原文件名_clips】文件夹
"""

import os
import sys
import subprocess
import importlib.util
from pathlib import Path
import re
from datetime import timedelta

# ==== 自动安装依赖 ====
def pip_install(package):
    try:
        __import__(package)
    except ImportError:
        print(f"[INFO] 安装依赖: {package}")
        subprocess.check_call([sys.executable, "-m", "pip", "install", package])

for pkg in ["srt", "chardet", "tqdm"]:
    pip_install(pkg)

import srt
import chardet
from tqdm import tqdm

# ==== 工具函数 ====
def detect_encoding(p):
    raw = Path(p).read_bytes()
    enc = chardet.detect(raw).get("encoding") or "utf-8"
    try:
        raw.decode(enc)
    except Exception:
        enc = "utf-8"
    return enc

def td2ss(t: timedelta):
    return t.total_seconds()

def ss2tc(seconds: float):
    ms = int(round(seconds * 1000))
    h = ms // 3600000
    m = (ms % 3600000) // 60000
    s = (ms % 60000) // 1000
    ms = ms % 1000
    return f"{h:02d}:{m:02d}:{s:02d}.{ms:03d}"

def safe_name(text, keep=30):
    text = re.sub(r"[\\/:*?\"<>|]", "_", text.strip())
    text = re.sub(r"\s+", " ", text)
    return text[:keep] if text else "clip"

# ==== 主逻辑 ====
def main():
    base = Path(__file__).parent  # 当前脚本所在目录（置顶文件夹）
    # 找媒体文件
    media_files = list(base.glob("*.mp4")) + list(base.glob("*.mkv")) + list(base.glob("*.mp3")) + list(base.glob("*.wav"))
    if not media_files:
        print("[ERROR] 没找到媒体文件（支持 mp4/mkv/mp3/wav）")
        return
    media = media_files[0]  # 默认取第一个
    print(f"[INFO] 使用媒体文件: {media}")

    # 找 srt 文件（同名）
    srt_file = media.with_suffix(".srt")
    if not srt_file.exists():
        print(f"[ERROR] 没找到对应字幕文件: {srt_file}")
        return
    print(f"[INFO] 使用字幕文件: {srt_file}")

    # 输出目录
    outdir = base / f"{media.stem}_clips"
    outdir.mkdir(exist_ok=True)

    # 读取字幕
    enc = detect_encoding(srt_file)
    text = srt_file.read_text(encoding=enc, errors="ignore")
    subs = list(srt.parse(text))

    total = 0
    for i, sub in enumerate(tqdm(subs, desc="Cutting", unit="seg"), start=1):
        start_s = td2ss(sub.start)
        end_s = td2ss(sub.end)

        ss = ss2tc(start_s)
        to = ss2tc(end_s)

        snippet = sub.content.replace("\n", " ").strip()
        name_text = safe_name(snippet, keep=20)

        ext = ".mp4" if media.suffix.lower() in [".mp4", ".mkv"] else ".m4a"
        out_path = outdir / f"{i:03d}_{name_text}{ext}"

        cmd = [
            "ffmpeg", "-y", "-hide_banner", "-loglevel", "error",
            "-ss", ss, "-to", to, "-i", str(media),
            "-c:v", "libx264", "-preset", "veryfast", "-crf", "23",
            "-c:a", "aac", "-b:a", "128k",
            str(out_path)
        ]
        try:
            subprocess.run(cmd, check=True)
            total += 1
        except subprocess.CalledProcessError:
            print(f"[WARN] 第 {i} 段导出失败")

    print(f"[完成] 共导出 {total} 段 -> {outdir}")

if __name__ == "__main__":
    main()
```