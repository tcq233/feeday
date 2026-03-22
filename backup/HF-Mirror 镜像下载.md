## 安装依赖

```
pip install -U huggingface_hub
```

- Linux/macOS (临时生效):
```
export HF_ENDPOINT=https://hf-mirror.com
```
- Windows PowerShell (临时生效):
```
$env:HF_ENDPOINT = "https://hf-mirror.com"
```


## 下载模型文件
```
import os
from huggingface_hub import snapshot_download

# 1. 设置环境变量，指向国内镜像站
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"

# 2. 定义模型 ID 和保存路径
model_id = "HauhauCS/Qwen3.5-9B-Uncensored-HauhauCS-Aggressive"
local_dir = "./Qwen3.5-9B-Uncensored"

print(f"开始从镜像站下载模型: {model_id} ...")

# 3. 执行下载
snapshot_download(
    repo_id=model_id,
    local_dir=local_dir,
    local_dir_use_symlinks=False, # 建议设为 False，直接下载真实文件而非软链接
    ignore_patterns=["*.msgpack", "*.h5", "*.ot"], # 可选：忽略不需要的格式（如 Pytorch 用户忽略 Rust/TensorFlow 文件）
    resume_download=True
)

print("下载完成！模型存放在:", local_dir)
```

## 下载单个模型文件
```
import os
from huggingface_hub import hf_hub_download

# 1. 设置环境变量，强制走国内镜像站
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"

# 2. 定义模型 ID 和保存路径
model_id = "HauhauCS/Qwen3.5-9B-Uncensored-HauhauCS-Aggressive"
local_dir = "E:/Models/Qwen3.5-9B-Uncensored"

# 3. 精确指定要下载的文件（单文件模型 + 视觉编码器）
files_to_download = [
    "Qwen3.5-9B-Uncensored-HauhauCS-Aggressive-Q6_K.gguf",
    "mmproj-Qwen3.5-9B-Uncensored-HauhauCS-Aggressive-BF16.gguf"
]

for filename in files_to_download:
    print(f"\n开始从镜像站下载: {filename} ...")
    try:
        # 新版 huggingface_hub 默认自带断点续传和非软链接特性
        file_path = hf_hub_download(
            repo_id=model_id,
            filename=filename,
            local_dir=local_dir
        )
        print(f"✅ 下载成功！文件已存放在: {file_path}")
    except Exception as e:
        print(f"❌ 下载报错: {e}")
```