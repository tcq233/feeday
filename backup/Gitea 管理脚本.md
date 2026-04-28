# 部署 Gitea
```
#!/bin/bash
# Gitea 管理脚本：安装前先卸载旧的
set -e

GITEA_VERSION="1.22.3"                          # 要安装的版本
GITEA_BIN="/usr/local/bin/gitea"
GITEA_USER="git"
GITEA_HOME="/var/lib/gitea"
GITEA_CONF="/etc/gitea"
SERVICE_FILE="/etc/systemd/system/gitea.service"

uninstall_gitea() {
    echo "⚠️ 卸载旧版 Gitea ..."
    systemctl stop gitea >/dev/null 2>&1 || true
    systemctl disable gitea >/dev/null 2>&1 || true
    rm -f $SERVICE_FILE
    rm -f $GITEA_BIN
    systemctl daemon-reload
    echo "✅ 卸载完成（数据目录 $GITEA_HOME 和配置 $GITEA_CONF 保留）"
}

install_gitea() {
    echo "📥 开始安装 Gitea v${GITEA_VERSION} ..."
    # 卸载旧版本
    uninstall_gitea

    # 创建用户和目录
    id -u $GITEA_USER &>/dev/null || useradd -r -m -d $GITEA_HOME -s /bin/bash $GITEA_USER
    mkdir -p $GITEA_HOME/{custom,data,log} $GITEA_CONF
    chown -R $GITEA_USER:$GITEA_USER $GITEA_HOME $GITEA_CONF

    # 下载二进制
    wget -O $GITEA_BIN "https://dl.gitea.com/gitea/${GITEA_VERSION}/gitea-${GITEA_VERSION}-linux-amd64"
    chmod +x $GITEA_BIN

    # 写 systemd 服务
    cat > $SERVICE_FILE <<EOF
[Unit]
Description=Gitea
After=syslog.target
After=network.target
Requires=network.target

[Service]
RestartSec=2s
Type=simple
User=$GITEA_USER
Group=$GITEA_USER
WorkingDirectory=$GITEA_HOME
ExecStart=$GITEA_BIN web --config $GITEA_CONF/app.ini
Restart=always
Environment=USER=$GITEA_USER HOME=$GITEA_HOME GITEA_WORK_DIR=$GITEA_HOME

[Install]
WantedBy=multi-user.target
EOF

    systemctl daemon-reload
    systemctl enable gitea
    systemctl start gitea
    echo "✅ 安装完成！访问 http://你的IP:3000 初始化"
}

restart_gitea() {
    echo "🔄 正在重启 Gitea ..."
    systemctl restart gitea
    echo "✅ 已重启"
}

stop_gitea() {
    echo "⏹️ 正在停止 Gitea ..."
    systemctl stop gitea
    echo "✅ 已停止"
}

menu() {
    echo "===== Gitea 管理 ====="
    echo "1) 安装最新 Gitea（会自动卸载旧版）"
    echo "2) 卸载 Gitea"
    echo "3) 重启 Gitea"
    echo "4) 停止 Gitea"
    echo "======================"
    read -p "请选择操作: " choice

    case $choice in
        1) install_gitea ;;
        2) uninstall_gitea ;;
        3) restart_gitea ;;
        4) stop_gitea ;;
        *) echo "无效选择" ;;
    esac
}

menu
```