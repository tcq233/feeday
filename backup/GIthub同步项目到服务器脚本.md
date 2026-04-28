同步代码项目脚本
```
#!/bin/bash
# ==========================================
# auto_git_update.sh
# 每天自动更新多个 Git 仓库到指定目录
# 作者: ChatGPT
# ==========================================

# ===== 基本配置 =====
BASE_DIR="/home/git"          # 所有仓库存放的目录
LOG_FILE="/home/git_update.log"

# ===== 仓库列表（可添加多个） =====
GIT_URLS=(
  "https://github.com/tcq20256/feeday.git"
  "https://github.com/tcq20256/tcq20256.github.io.git"
)

# ===== 函数：更新或克隆仓库 =====
update_repo() {
  local GIT_URL="$1"
  local REPO_NAME=$(basename "$GIT_URL" .git)
  local REPO_DIR="${BASE_DIR}/${REPO_NAME}"

  echo "----------------------------------------" | tee -a "$LOG_FILE"
  echo "🕒 $(date '+%F %T') 开始处理仓库：$REPO_NAME" | tee -a "$LOG_FILE"

  # 确保目录存在
  mkdir -p "$BASE_DIR"

  # 克隆或更新
  if [ ! -d "$REPO_DIR/.git" ]; then
    echo "📦 第一次 clone 仓库：$GIT_URL" | tee -a "$LOG_FILE"
    git clone "$GIT_URL" "$REPO_DIR" >> "$LOG_FILE" 2>&1
  else
    echo "🔄 更新仓库：$REPO_NAME" | tee -a "$LOG_FILE"
    cd "$REPO_DIR" || { echo "❌ 无法进入目录：$REPO_DIR" | tee -a "$LOG_FILE"; return; }
    git reset --hard HEAD >> "$LOG_FILE" 2>&1
    git pull >> "$LOG_FILE" 2>&1
  fi

  if [ $? -eq 0 ]; then
    echo "✅ 完成更新：$REPO_NAME" | tee -a "$LOG_FILE"
  else
    echo "⚠️ 更新失败：$REPO_NAME" | tee -a "$LOG_FILE"
  fi
}

# ===== 主逻辑：循环处理每个仓库 =====
echo "========================================" | tee -a "$LOG_FILE"
echo "🚀 启动自动 Git 更新任务 $(date '+%F %T')" | tee -a "$LOG_FILE"

for url in "${GIT_URLS[@]}"; do
  update_repo "$url"
done

echo "✅ 所有仓库处理完毕 $(date '+%F %T')" | tee -a "$LOG_FILE"
echo "========================================" | tee -a "$LOG_FILE"
```