
检查 imgur 图片在多个镜像站是否可用

- https://shiquda.link/load-imgur-images/
```
import requests
import time

# 测试超时（秒）
TIMEOUT = 10

# 测试的镜像站模式
MIRRORS = {
    "StackImgur":      "https://i.stack.imgur.com/{id}",
    "ImgurPics":       "https://imgur.pics/{id}",
    "ImgVue":          "https://imgvue.com/images/{id}",
    "RisuAI":          "https://risuai.com/imgur/{id}",  # 备选
}

def check_url(url):
    """HEAD 方式检查 URL 是否可访问"""
    try:
        r = requests.head(url, timeout=TIMEOUT)
        return r.status_code in (200, 301, 302)
    except:
        return False


def test_imgur_id(img_id, ext="jpg"):
    """检测单个 imgur ID 的可用镜像"""
    print(f"\n===== 测试 ID: {img_id} =====")

    results = {}

    # 拼接文件名
    full = f"{img_id}.{ext}"

    for name, pattern in MIRRORS.items():
        url = pattern.format(id=full)

        ok = check_url(url)
        results[name] = ok

        status = "✔ 可用" if ok else "✘ 不可用"
        print(f"{name:12s} → {status}  ({url})")

        time.sleep(0.2)

    return results


if __name__ == "__main__":
    # 示例 ID，可换成真实 ID
    test_list = [
        "qHxM2",  # 你提供的
        "9xndJ",
        "ciycE",
        "AFFHn",
    ]

    for img_id in test_list:
        test_imgur_id(img_id)
```

## 数据集下载测试
```
# -*- coding: utf-8 -*-
"""
逐行下载 ORIGINAL 和 PS 图片，并按指定格式重命名：
Original: {id}_{imgurID}_original.ext
PS:       {originalID}_{id_variant}_{imgurID}_ps.ext
"""

import os
import csv
import requests
import time

# ===== 代理域名 =====
PROXY_DOMAIN = "https://img.noobzone.ru/getimg.php?url="

def proxy_url(url):
    return f"{PROXY_DOMAIN}{url}"

# ===== 输入文件路径 =====
ORIGINAL_TSV = "/home/data/originals.tsv"
PS_TSV = "/home/data/photoshops.tsv"

# ===== 输出目录 =====
SAVE_ROOT = "/home/data/output"
ORIG_DIR = os.path.join(SAVE_ROOT, "original")
PS_DIR = os.path.join(SAVE_ROOT, "photoshop")
FAILED_LOG = os.path.join(SAVE_ROOT, "failed.txt")

os.makedirs(ORIG_DIR, exist_ok=True)
os.makedirs(PS_DIR, exist_ok=True)

def extract_imgur_id(url: str):
    """从 imgur 链接提取图片 ID"""
    return url.split("/")[-1].split(".")[0]


def download(line_no, url, save_path, retry=3):
    """下载函数（带代理 + 打印行号）"""
    for attempt in range(1, retry + 1):
        try:
            purl = proxy_url(url)
            print(f"\n==== 行 {line_no} (尝试 {attempt}/{retry}) ====")
            print(f"原始 URL: {url}")
            print(f"代理 URL: {purl}")

            r = requests.get(purl, timeout=20, stream=True)
            print(f"状态码: {r.status_code}")

            if r.status_code == 200:
                with open(save_path, "wb") as f:
                    for chunk in r.iter_content(8192):
                        f.write(chunk)
                print(f"✔ 成功 → {save_path}")
                return True
            else:
                print(f"❌ 错误状态码 {r.status_code}")
                err = f"{url}\tstatus {r.status_code}\n"

        except Exception as e:
            print(f"💥 出错: {e}")
            err = f"{url}\terror {e}\n"

        time.sleep(1)

    with open(FAILED_LOG, "a", encoding="utf-8") as f:
        f.write(err)

    print("❌ 三次尝试失败！")
    return False


# ========== 逐行交替处理 ==========

def read_tsv(path):
    data = []
    with open(path, "r", encoding="utf-8") as f:
        reader = csv.DictReader(f, delimiter="\t")
        for row in reader:
            data.append(row)
    return data


print("📄 正在读取 TSV…")
orig_rows = read_tsv(ORIGINAL_TSV)
ps_rows = read_tsv(PS_TSV)

max_len = max(len(orig_rows), len(ps_rows))

print(f"原图 {len(orig_rows)} 行，PS {len(ps_rows)} 行")
print("\n🚀 开始逐行下载…")

for i in range(max_len):
    line_no = i + 1

    # ===== ORIGINAL =====
    if i < len(orig_rows):
        row = orig_rows[i]
        id_or = row.get("id")
        url_or = row.get("url")
        ext_or = (row.get("end") or "jpg").lstrip(".")

        imgurID_or = extract_imgur_id(url_or)

        filename_or = f"{id_or}_{imgurID_or}_original.{ext_or}"
        save_or = os.path.join(ORIG_DIR, filename_or)

        download(line_no, url_or, save_or)

    # ===== PS =====
    if i < len(ps_rows):
        row = ps_rows[i]
        id_variant = row.get("id_variant") or row.get("id")
        id_original = row.get("original")
        url_ps = row.get("url")
        ext_ps = (row.get("end") or "jpg").lstrip(".")

        imgurID_ps = extract_imgur_id(url_ps)

        # 命名格式：
        # 10092l_c69axf4_0_xIJ4z_ps.png
        filename_ps = f"{id_original}_{id_variant}_{imgurID_ps}_ps.{ext_ps}"
        save_ps = os.path.join(PS_DIR, filename_ps)

        download(line_no, url_ps, save_ps)

print("\n🎉 完成！")
print(f"📁 输出目录: {SAVE_ROOT}")
print(f"📝 失败日志: {FAILED_LOG}")
```