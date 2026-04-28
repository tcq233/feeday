HF Mirror —— Hugging Face 下载全攻略（国内可用）

> AI 开发绕不过一个问题：如何从 Hugging Face 稳定下载模型或数据集。  
> 在网络通畅的情况下，任何方式都可以；但在国内环境中，**断点续传、多线程下载** 显得尤为重要。  
> 否则动辄断线重来、速度龟速，简直浪费生命！  
> 基于此，本文整合了官方方法与第三方工具，进行横向对比与实践说明。

---

### 官方与镜像资源

- 🇨🇳 镜像站：[https://hf-mirror.com](https://hf-mirror.com/)
- 教程原文：[https://padeoe.com/huggingface-large-models-downloader/](https://padeoe.com/huggingface-large-models-downloader/)
- 代码示例：[https://gist.github.com/padeoe/697678ab8e528b85a2a7bddafea1fa4f](https://gist.github.com/padeoe/697678ab8e528b85a2a7bddafea1fa4f)

---

### 下载方式对比表

| 方法类别 | 示例 / 链接 | 推荐程度 | 优点 | 缺点 |
|-----------|--------------|------------|--------|--------|
| 基于URL | [浏览器网页下载](https://padeoe.com/huggingface-large-models-downloader/#1.-%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BD%91%E9%A1%B5%E4%B8%8B%E8%BD%BD) | ⭐⭐⭐ | 通用性好 | 手动麻烦 / 无多线程 |
| 多线程下载器 | [多线程下载器](https://padeoe.com/huggingface-large-models-downloader/#2.-%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B8%8B%E8%BD%BD%E5%99%A8) | ⭐⭐⭐⭐ | 通用性好 / 可断点续传 | 需要人工操作 |
| CLI 工具 | [git clone 命令](https://padeoe.com/huggingface-large-models-downloader/#3.-Git-clone) | ⭐⭐ | 简单易用 | 无断点续传 / 冗余文件 / 无多线程 |
| 专用 CLI 工具 | [huggingface-cli + hf_transfer](https://padeoe.com/huggingface-large-models-downloader/#4.-huggingface-cli%2Bhf_transfer) | ⭐⭐⭐ | 官方下载工具链，功能最全 | 无进度条 / 容错性低 |
| huggingface-cli | [官方 huggingface-cli 工具](https://padeoe.com/huggingface-large-models-downloader/#4.1-huggingface-cli) | ⭐⭐⭐⭐⭐ | 官方下载工具，稳定可靠 | 不支持多线程 |
| Python 方法 | [snapshot_download](https://padeoe.com/huggingface-large-models-downloader/#5.-snapshot_download) | ⭐⭐⭐ | 官方支持，功能全 | 脚本复杂 / 无多线程 |
| from_pretrained | [from_pretrained](https://padeoe.com/huggingface-large-models-downloader/#6.-from_pretrained) | ⭐ | 官方支持，简单 | 不方便存储 / 功能不全 |
| hf_hub_download | [hf_hub_download](https://padeoe.com/huggingface-large-models-downloader/#6.-hf_hub_download) | ⭐ | 官方支持 | 不支持全量下载 / 无多线程 |

---

### 安装依赖

打开 PowerShell 安装依赖
```
pip install -U "huggingface_hub[cli]"
```

如失败卸载安装稳定版本
```
pip uninstall -y huggingface_hub
pip install huggingface_hub==0.25.2 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 验证安装依赖
```
dir "D:\Python313\Scripts" | findstr huggingface
```
一般输出
```
-a----         2025/11/1  ?? 11:54         108371 huggingface-cli.exe
```

### 下载数据集
```
$env:HF_ENDPOINT = "https://hf-mirror.com"
& "D:\Python313\Scripts\huggingface-cli.exe" download --repo-type dataset --resume-download datxy/file --local-dir D:\hf2\datxy_file --local-dir-use-symlinks False
```

### 一键下载脚本

```
@echo off
title 🤗 Hugging Face Mirror Downloader (hf-mirror.com)
chcp 65001 >nul
color 0A

echo.
echo ============================================
echo   🤗 Hugging Face Mirror Downloader
echo   Supports model / dataset auto download
echo   Mirror: https://hf-mirror.com
echo ============================================
echo.

:: 1. Set mirror environment variable (temporary)
set HF_ENDPOINT=https://hf-mirror.com

:: 2. Ask for repo type (model or dataset)
set /p REPO_TYPE=Enter repo type [model/dataset] (default: dataset): 
if "%REPO_TYPE%"=="" set REPO_TYPE=dataset

:: 3. Ask for repo ID (username/repo_name)
set /p REPO_ID=Enter Hugging Face repo ID (e.g. datxy/file): 

:: 4. Ask for local save path
set /p TARGET_DIR=Enter local save path (default: D:\hf_downloads\%REPO_ID%): 
if "%TARGET_DIR%"=="" set TARGET_DIR=D:\hf_downloads\%REPO_ID%

:: 5. Check if huggingface-cli exists
if not exist "D:\Python313\Scripts\huggingface-cli.exe" (
    echo ❌ huggingface-cli.exe not found!
    echo Please make sure huggingface_hub 0.25.2 is installed.
    echo Try:
    echo pip install huggingface_hub==0.25.2 -i https://pypi.tuna.tsinghua.edu.cn/simple
    pause
    exit /b
)

:: 6. Start download
echo.
echo 🚀 Starting download: %REPO_ID%
echo Target directory: %TARGET_DIR%
echo -------------------------------------
"D:\Python313\Scripts\huggingface-cli.exe" download --repo-type %REPO_TYPE% --resume-download %REPO_ID% --local-dir "%TARGET_DIR%" --local-dir-use-symlinks False
echo -------------------------------------
echo ✅ Download completed!
echo.

pause
exit /b
```