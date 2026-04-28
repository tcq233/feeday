Centos7.6老版本安装conda

## 安装脚本
```
#!/bin/bash
# ==========================================
# install_conda_centos7.sh
# 适用于 CentOS 7 的 Miniconda 自动安装脚本（兼容 glibc 2.17）
# 作者: ChatGPT
# ==========================================

set -e

INSTALL_DIR="/root/miniconda3"
CONDA_SCRIPT="Miniconda3-py38_23.3.1-0-Linux-x86_64.sh"
DOWNLOAD_URL="https://repo.anaconda.com/miniconda/${CONDA_SCRIPT}"

echo "🐧 检测系统版本..."
if grep -q "release 7" /etc/centos-release 2>/dev/null; then
    echo "✅ 检测到 CentOS 7，使用兼容版 Miniconda (Python 3.8)"
else
    echo "⚠️ 未检测到 CentOS 7，将使用最新版 Miniconda 安装包"
    CONDA_SCRIPT="Miniconda3-latest-Linux-x86_64.sh"
    DOWNLOAD_URL="https://repo.anaconda.com/miniconda/${CONDA_SCRIPT}"
fi

echo "🐍 检查 Conda 是否已安装..."
if command -v conda >/dev/null 2>&1; then
    echo "✅ Conda 已安装，版本：$(conda -V)"
    exit 0
fi

echo "🌍 正在下载 Miniconda 安装脚本..."
cd /opt
curl -LO "$DOWNLOAD_URL"

echo "📦 安装 Miniconda 到: $INSTALL_DIR ..."
bash "$CONDA_SCRIPT" -b -p "$INSTALL_DIR"

# 激活 conda
source "$INSTALL_DIR/bin/activate"

# 配置环境变量
if ! grep -q "miniconda3/bin/activate" ~/.bashrc; then
    echo "source $INSTALL_DIR/bin/activate" >> ~/.bashrc
    echo "✅ 已将 Conda 自动加载配置写入 ~/.bashrc"
fi

# 设置清华镜像源
echo "🧭 配置 TUNA 清华镜像源..."
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --set show_channel_urls yes

# 验证安装
echo "🔍 验证 Conda 安装..."
conda -V
python -V

echo "🎉 安装完成！请执行以下命令激活环境："
echo "👉 source ~/.bashrc"
echo "👉 conda create -n py310 python=3.10 -y"
echo "👉 conda activate py310"
echo "source /root/miniconda3/bin/activate"
```

## 创建激活环境
```
source /root/miniconda3/bin/activate
conda create -n py310 python=3.10 -y
conda activate py310
```