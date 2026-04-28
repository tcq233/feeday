## 模型定价
- [在线生成](https://console.volcengine.com/ark/region:ark+cn-beijing/model/detail?Id=doubao-seededit-3-0-i2i)
- [使用量查看](https://console.volcengine.com/ark/region:ark+cn-beijing/openManagement?LLM=%7B%7D&OpenTokenDrawer=false&tab=ComputerVision)

模型名称 | 定价元/张
-- | --
doubao-seedream-4.0 | 0.2
doubao-seedream-3.0-t2i | 0.259
doubao-seededit-3.0-i2i | 0.3


## 豆包图像识别

```
import os
import sys
import subprocess

# ===== 安装依赖 =====
def install(package):
    subprocess.check_call([sys.executable, "-m", "pip", "install", "--upgrade", package])

install("openai>=1.0")

from openai import OpenAI

# 初始化 Ark 客户端（建议方式：从环境变量读取）
client = OpenAI(
    base_url="https://ark.cn-beijing.volces.com/api/v3",
    api_key="xxxx",
)

# 调用模型
response = client.chat.completions.create(
    model="doubao-1-5-vision-pro-32k-250115",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://ark-project.tos-cn-beijing.ivolces.com/images/view.jpeg"
                    },
                },
                {"type": "text", "text": "这是哪里？"},
            ],
        }
    ],
)

print(response.choices[0].message)
```

## 豆包图像修改

```
import os
import sys
import subprocess

# ===== 自动安装依赖 =====
def install(package):
    subprocess.check_call([sys.executable, "-m", "pip", "install", "--upgrade", package])

install("volcengine-python-sdk[ark]")

# ===== 导入 SDK =====
from volcenginesdkarkruntime import Ark

# 初始化 Ark 客户端
client = Ark(
    base_url="https://ark.cn-beijing.volces.com/api/v3",
    api_key="xxxx",
)

# 调用图片生成接口
imagesResponse = client.images.generate(
    model="doubao-seededit-3-0-i2i-250628",
    prompt="改成爱心形状的泡泡",
    image="https://ark-project.tos-cn-beijing.volces.com/doc_image/seededit_i2i.jpeg",
    seed=123,
    guidance_scale=5.5,
    size="adaptive",
    watermark=True
)

# 打印返回图片地址
print(imagesResponse.data[0].url)
```

## 豆包改图下载本地

```
import os
import sys
import subprocess
import requests

# ===== 自动安装依赖 =====
def install(package):
    subprocess.check_call([sys.executable, "-m", "pip", "install", "--upgrade", package])

try:
    from volcenginesdkarkruntime import Ark
except ImportError:
    install("volcengine-python-sdk[ark]")
    from volcenginesdkarkruntime import Ark

# ===== 初始化 Ark 客户端 =====
client = Ark(
    base_url="https://ark.cn-beijing.volces.com/api/v3",
    api_key="xxx",  # 建议改成 os.getenv("ARK_API_KEY")
)

# ===== 配置 =====
TXT_FILE = r"c:\prompts.txt"   # 提示词文件
SAVE_DIR = r"C:\doubao\i2i"   # 下载保存目录
os.makedirs(SAVE_DIR, exist_ok=True)

# ===== 读取提示词文件并逐行调用 =====
with open(TXT_FILE, "r", encoding="utf-8") as f:
    prompts = [line.strip() for line in f if line.strip()]

for idx, prompt in enumerate(prompts, start=1):
    try:
        imagesResponse = client.images.generate(
            model="doubao-seededit-3-0-i2i-250628",
            prompt=prompt,  # 每行作为提示词
            image="https://ark-project.tos-cn-beijing.volces.com/doc_image/seededit_i2i.jpeg",
            seed=123,
            guidance_scale=5.5,
            size="adaptive",
            watermark=True
        )
        url = imagesResponse.data[0].url
        print(f"[{idx}] 提示词: {prompt} -> 图片地址: {url}")

        # ===== 下载图片 =====
        resp = requests.get(url, timeout=60)
        if resp.status_code == 200:
            # 文件名：序号_前10个字符提示词.jpg
            safe_prompt = "".join(c for c in prompt if c.isalnum())[:10]
            filename = os.path.join(SAVE_DIR, f"{idx:03d}_{safe_prompt}.jpg")
            with open(filename, "wb") as f:
                f.write(resp.content)
            print(f"    已保存到: {filename}")
        else:
            print(f"    下载失败: HTTP {resp.status_code}")

    except Exception as e:
        print(f"[{idx}] 提示词: {prompt} -> 生成失败: {e}")
```

## 本地图片批量修改

```
import os
import sys
import subprocess
import base64
import mimetypes
import requests
import re
from time import sleep

# ===== 自动安装依赖 =====
def install(package):
    subprocess.check_call([sys.executable, "-m", "pip", "install", "--upgrade", package])

try:
    from volcenginesdkarkruntime import Ark
except ImportError:
    install("volcengine-python-sdk[ark]")
    from volcenginesdkarkruntime import Ark

# ===== 工具函数 =====
def to_data_url(img_path: str) -> str:
    """把本地图片转成 data URL: data:image/xxx;base64,xxxx"""
    with open(img_path, "rb") as f:
        b64 = base64.b64encode(f.read()).decode("utf-8")
    mime, _ = mimetypes.guess_type(img_path)
    if not mime:
        mime = "image/jpeg"
    return f"data:{mime};base64,{b64}"

def download(url: str, out_path: str, timeout: int = 60, retry: int = 3) -> bool:
    for i in range(1, retry + 1):
        try:
            r = requests.get(url, timeout=timeout)
            if r.status_code == 200:
                with open(out_path, "wb") as f:
                    f.write(r.content)
                return True
            else:
                print(f"    [下载] HTTP {r.status_code}（{i}/{retry}）")
        except Exception as e:
            print(f"    [下载] 异常：{e}（{i}/{retry}）")
        sleep(1)
    return False

def natural_key(s: str):
    """自然排序 key，确保 2.jpg < 10.jpg"""
    return [int(text) if text.isdigit() else text.lower()
            for text in re.split(r'(\d+)', s)]

# ===== 初始化 Ark 客户端 =====
client = Ark(
    base_url="https://ark.cn-beijing.volces.com/api/v3",
    api_key=os.getenv("ARK_API_KEY", "xxx"),  # 推荐改用环境变量
)

# ===== 路径配置 =====
IMG_DIR  = r"C:\doubao\188"     # 输入原图目录
SAVE_DIR = r"C:\doubao\i2i"    # 输出目录
TXT_FILE = r"C:\prompts.txt"   # 提示词文件
os.makedirs(SAVE_DIR, exist_ok=True)

# ===== 读取提示词 =====
with open(TXT_FILE, "r", encoding="utf-8") as f:
    prompts = [line.strip() for line in f if line.strip()]
if not prompts:
    raise RuntimeError("prompts.txt 为空：请至少提供一行提示词。")

# ===== 收集并自然排序图片 =====
images = [fn for fn in os.listdir(IMG_DIR) if fn.lower().endswith((".jpg", ".jpeg", ".png", ".webp", ".bmp"))]
images.sort(key=natural_key)
if not images:
    raise RuntimeError(f"在目录中未找到图片：{IMG_DIR}")

# 提示词策略
if len(prompts) == 1:
    def pick_prompt(_idx): return prompts[0]
elif len(prompts) == len(images):
    def pick_prompt(idx): return prompts[idx]
else:
    print(f"[警告] 图片数({len(images)}) 与提示词行数({len(prompts)})不匹配，将对所有图片使用第一行提示词。")
    def pick_prompt(_idx): return prompts[0]

# ===== 逐张处理 =====
ok_count = 0
for i, img_name in enumerate(images):
    src = os.path.join(IMG_DIR, img_name)
    name, _ = os.path.splitext(img_name)
    dst = os.path.join(SAVE_DIR, f"{name}-i2i.jpg")
    prompt = pick_prompt(i)

    try:
        image_data_url = to_data_url(src)

        resp = client.images.generate(
            model="doubao-seededit-3-0-i2i-250628",
            prompt=prompt,
            image=image_data_url,   # data URL
            seed=123,
            guidance_scale=5.5,
            size="adaptive",
            watermark=True,
        )

        out_url = resp.data[0].url
        print(f"[{i+1}] {img_name} | 提示词: {prompt} -> {out_url}")

        if download(out_url, dst):
            print(f"    已保存: {dst}")
            ok_count += 1
        else:
            print(f"    [失败] 下载未成功: {dst}")

    except Exception as e:
        print(f"[{i+1}] {img_name} -> 生成失败: {e}")

print(f"\n完成：成功生成并保存 {ok_count}/{len(images)} 张，输出目录：{SAVE_DIR}")
```