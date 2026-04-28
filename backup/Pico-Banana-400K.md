苹果发布的香蕉图像编辑数据集
- [https://github.com/apple/pico-banana-400k](https://github.com/apple/pico-banana-400k)

逐行读取 JSONL 文件，下载 open_image 和 output_image。
命名格式：
000001_open_image.png
000001_output_image.png

增强功能：
- ✅ 从指定 JSONL 行号开始（断点续传）
- ✅ 编号与行号同步（例如从第 98233 行开始 → 文件名从 098233 开始）
- ✅ 任一下载失败则两者都不保留
- ✅ 记录所有错误日志

## 预览数据集图片
![open_image_input_url](https://c1.staticflickr.com/8/7404/9423051591_cb1bf5c5e1_o.jpg)
![output_image](https://ml-site.cdn-apple.com/datasets/pico-banana-300k/nb/images/positive-edit/1.png)

```
{"open_image_input_url": "https://c1.staticflickr.com/8/7404/9423051591_cb1bf5c5e1_o.jpg", 
"text": "Remove the red flag and its white pole from the upper right of the image, seamlessly extending the clear blue sky, the sandy dune with its subtle texture, and the wooden fence to fill the void, ensuring the lighting, color, and natural grain of the background are perfectly matched for a realistic and unblemished result.", 
"output_image": "images/positive-edit/1.png", 
"edit_type": "Remove an existing object",
 "summarized_text": "Flag removed; extend sky, dune, and fence seamlessly."}
```

## 下载脚本
```
import os
import sys
import json
import subprocess

# ===== 自动安装依赖 =====
required_packages = ["requests", "tqdm"]
for pkg in required_packages:
    try:
        __import__(pkg)
    except ImportError:
        print(f"📦 未找到 {pkg}，正在安装...")
        subprocess.check_call([sys.executable, "-m", "pip", "install", pkg, "-i", "https://pypi.tuna.tsinghua.edu.cn/simple"])

import requests
from tqdm import tqdm


# ===== 配置区 =====
JSONL_FILE = r"E:\Pico-Banana-400K\sft.jsonl"   # JSONL 文件路径
BASE_URL = "https://ml-site.cdn-apple.com/datasets/pico-banana-300k/nb/"  # output_image 前缀
SAVE_DIR = r"E:\Pico-Banana-400K\sft"                                                       # 保存目录
FAILED_LOG = os.path.join(SAVE_DIR, "failed_downloads.txt")               # 错误日志路径
START_LINE = 98233  # ←★★ 从第几行开始（自动跳过之前的）

os.makedirs(SAVE_DIR, exist_ok=True)


# ===== 下载函数 =====
def safe_download(url, path):
    """下载文件（成功返回 True，失败返回 False）"""
    try:
        if os.path.exists(path) and os.path.getsize(path) > 0:
            print(f"⏭️ 已存在：{os.path.basename(path)}，跳过")
            return True

        print(f"⬇️ 正在下载：{url}")
        resp = requests.get(url, stream=True, timeout=30)
        if resp.status_code == 200:
            with open(path, "wb") as f:
                for chunk in resp.iter_content(8192):
                    f.write(chunk)
            print(f"✅ 下载成功：{os.path.basename(path)}")
            return True
        else:
            print(f"❌ 下载失败（状态码 {resp.status_code}）：{url}")
            with open(FAILED_LOG, "a", encoding="utf-8") as log:
                log.write(f"{url}\t状态码 {resp.status_code}\n")
            return False
    except Exception as e:
        print(f"⚠️ 下载出错：{url}\n{e}")
        with open(FAILED_LOG, "a", encoding="utf-8") as log:
            log.write(f"{url}\t错误 {e}\n")
        return False


# ===== 主逻辑 =====
total = 0
skipped = 0

with open(JSONL_FILE, "r", encoding="utf-8") as f:
    for line_no, line in enumerate(tqdm(f, desc="Processing JSONL"), start=1):
        if line_no < START_LINE:
            continue  # 跳过前面未到起点的行

        line = line.strip()
        if not line:
            continue

        try:
            item = json.loads(line)
        except json.JSONDecodeError as e:
            print(f"⚠️ JSON 解析错误（第 {line_no} 行）：{e}")
            continue

        # 支持字段兼容
        open_url = item.get("open_image_input_url") or item.get("open_image")
        output_path = item.get("output_image") or item.get("output_image_path")

        if not open_url or not output_path:
            print(f"⚠️ 第 {line_no} 行缺字段，跳过")
            continue

        # 拼接 output_image URL
        if output_path.startswith("http"):
            full_output_url = output_path
        else:
            full_output_url = BASE_URL.rstrip("/") + "/" + output_path.lstrip("/")

        # === 生成文件名 ===
        prefix = f"{line_no:06d}"  # 与 JSONL 行号对应
        open_save = os.path.join(SAVE_DIR, f"{prefix}_open_image.png")
        output_save = os.path.join(SAVE_DIR, f"{prefix}_output_image.png")

        # ---- 下载 open_image ----
        ok_open = safe_download(open_url, open_save)
        if not ok_open:
            skipped += 1
            print(f"🚫 第 {line_no} 行 open_image 下载失败，跳过该条记录")
            continue

        # ---- 下载 output_image ----
        ok_out = safe_download(full_output_url, output_save)
        if not ok_out:
            skipped += 1
            # 删除已下载的 open_image
            if os.path.exists(open_save):
                os.remove(open_save)
                print(f"🗑️ 已删除：{os.path.basename(open_save)}（因为 output_image 下载失败）")
            continue

        total += 1

print(f"\n✅ 下载完成，共成功 {total} 对，跳过 {skipped} 对。")
print(f"📄 错误日志：{FAILED_LOG}")
print(f"▶ 从第 {START_LINE} 行开始处理 JSONL")

```

