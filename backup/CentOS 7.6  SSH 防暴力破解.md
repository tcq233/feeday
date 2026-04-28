#  普通连接

```
ssh -p 22 root@192.168.1.1
```

#  密钥连接

- https://github.com/settings/ssh/new

```
ssh-keygen -t rsa -b 4096 -C "123@qq.com"
type C:\.ssh\id_rsa.pub

ssh -i <私钥文件路径> <用户名>@<服务器IP>

ssh -T git@github.com
git@github.com:tcq20256/feeday.git
```

# 封禁攻击的IP

```
#!/usr/bin/env bash
# setup_ssh_antibrute.sh
# CentOS 7.6：Fail2Ban SSH 防暴力破解（安装/配置 + 自愈修复 一体化）
# - 仅动 fail2ban，不改 sshd 端口/认证方式，不影响其他服务
# - firewalld 在跑：firewallcmd-ipset（运行时规则，非永久）；否则用 iptables-multiport
# - BAN/UNBAN 审计日志可自定义路径（默认 /home/lighthouse/bash/ssh-ban.log，或 BAN_LOG="__SCRIPT_DIR__"）
# - 检测到 socket 连接失败会自动执行修复流程（清理残留、重建 /run/fail2ban、恢复 SELinux 上下文、补齐 iptables）

set -euo pipefail

### ===== 可调参数（也可用环境变量覆盖）=====
BANTIME="${BANTIME:-3600}"     # 被封时长（秒）
FINDTIME="${FINDTIME:-600}"    # 观察窗口（秒）
MAXRETRY="${MAXRETRY:-5}"      # 失败次数阈值
MY_IP="${MY_IP:-}"             # 可选：你的出口白名单，如 1.2.3.4
DEFAULT_BAN_LOG="/home/lighthouse/bash/ssh-ban.log"

# 日志位置：支持 BAN_LOG="__SCRIPT_DIR__"
SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"
if [[ "${BAN_LOG:-}" == "__SCRIPT_DIR__" ]]; then
  BAN_LOG="${SCRIPT_DIR}/ssh-ban.log"
else
  BAN_LOG="${BAN_LOG:-$DEFAULT_BAN_LOG}"
fi

### ===== 小工具 =====
msg(){ echo -e "\033[1;32m[INFO]\033[0m $*"; }
warn(){ echo -e "\033[1;33m[WARN]\033[0m $*"; }
err(){ echo -e "\033[1;31m[ERR ]\033[0m $*"; }

require_root(){ [[ ${EUID:-$(id -u)} -eq 0 ]] || { err "请用 root 运行：sudo bash $0"; exit 1; }; }
file_put(){ # $1:path  $2:content
  local p="$1"; shift
  umask 022; cat >"$p" <<<"$*"
  chmod 0644 "$p"
}

### ===== 修复流程：清理残留 / 目录 / SELinux / 组件 =====
repair_fail2ban(){
  warn "触发自愈修复：清理残留并重建运行环境……"
  systemctl stop fail2ban || true
  pkill -9 -f fail2ban-server || true

  rm -rf /run/fail2ban /var/run/fail2ban
  install -d -m 755 -o root -g root /run/fail2ban
  ln -sfn /run/fail2ban /var/run/fail2ban

  # SELinux（若启用则恢复上下文，无副作用）
  if command -v selinuxenabled >/dev/null 2>&1 && selinuxenabled; then
    restorecon -Rv /run/fail2ban || true
  fi

  # 组件兜底：当前 banaction 可能用到 iptables
  yum install -y -q iptables iptables-services || true

  # 确保 fail2ban.conf 使用标准 socket 路径（仅修正缺失/异常情况）
  local conf="/etc/fail2ban/fail2ban.conf"
  if [[ -f "$conf" ]]; then
    grep -qE '^\s*socket\s*=\s*/var/run/fail2ban/fail2ban\.sock' "$conf" || \
      sed -ri 's|^\s*socket\s*=.*|socket = /var/run/fail2ban/fail2ban.sock|g' "$conf"
    grep -qE '^\s*pidfile\s*=\s*/var/run/fail2ban/fail2ban\.pid' "$conf" || \
      sed -ri 's|^\s*pidfile\s*=.*|pidfile = /var/run/fail2ban/fail2ban.pid|g' "$conf"
  fi

  systemctl restart fail2ban
  sleep 1
}

### ===== 主流程 =====
require_root

# 1) 安装依赖
if ! rpm -qa | grep -qiE '^epel-release'; then
  msg "安装 epel-release ..."
  yum install -y epel-release
fi
if ! rpm -qa | grep -qiE '^fail2ban(-server)?'; then
  msg "安装 fail2ban ..."
  yum install -y fail2ban
else
  msg "fail2ban 已安装"
fi

# 2) 检测 firewalld
FIREWALLD_ACTIVE=0
if systemctl is-active firewalld >/dev/null 2>&1; then
  FIREWALLD_ACTIVE=1
  msg "firewalld 运行中：banaction=firewallcmd-ipset（运行时规则，非永久）"
else
  warn "firewalld 未运行：banaction=iptables-multiport"
fi
BANACTION="iptables-multiport"
[[ $FIREWALLD_ACTIVE -eq 1 ]] && BANACTION="firewallcmd-ipset"

# 3) 自定义动作：log-ban（正确使用 <name>/<ip>/<port>/<failures>；printf 用 %%s；date 用 %%F %%T）
file_put /etc/fail2ban/action.d/log-ban.local \
'[Definition]
actionban   = /bin/sh -c '\''printf "%%s\tBAN\tjail=<name>\tip=<ip>\tport=<port>\tfailures=<failures>\tsrc=%(src)s\n" "$(date "+%%F %%T")" >> %(logfile)s'\''
actionunban = /bin/sh -c '\''printf "%%s\tUNBAN\tjail=<name>\tip=<ip>\n" "$(date "+%%F %%T")" >> %(logfile)s'\'''
chmod 0644 /etc/fail2ban/action.d/log-ban.local

# 4) 生成 jail.local（仅开启 sshd 监狱）
JAIL_LOCAL="/etc/fail2ban/jail.local"
if [[ -f "$JAIL_LOCAL" ]]; then
  cp -a "$JAIL_LOCAL" "${JAIL_LOCAL}.bak.$(date +%Y%m%d-%H%M%S)"
  msg "已备份原配置：${JAIL_LOCAL}.bak.*"
fi
IGNOREIP="127.0.0.1/8"
[[ -n "$MY_IP" ]] && IGNOREIP="$IGNOREIP $MY_IP"
file_put "$JAIL_LOCAL" \
"[DEFAULT]
bantime   = ${BANTIME}
findtime  = ${FINDTIME}
maxretry  = ${MAXRETRY}
backend   = auto
ignoreip  = ${IGNOREIP}
banaction = ${BANACTION}

[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/secure
action   = %(action_)s
           log-ban[logfile=${BAN_LOG}, src=/var/log/secure]
"
chmod 0644 "$JAIL_LOCAL"

# 5) 日志与 logrotate
mkdir -p "$(dirname -- "$BAN_LOG")"
touch "$BAN_LOG"
chmod 0640 "$BAN_LOG"
chown root:root "$BAN_LOG"
file_put /etc/logrotate.d/ssh-ban \
"${BAN_LOG} {
    daily
    rotate 14
    missingok
    notifempty
    compress
    create 0640 root root
}
"

# 6) 兜底运行目录
install -d -m 755 -o root -g root /run/fail2ban
ln -sfn /run/fail2ban /var/run/fail2ban

# 7) 语法测试 & 启动
msg "校验 fail2ban 配置语法 ..."
if ! fail2ban-client -t; then
  err "配置语法校验失败，请检查上方输出。"
  exit 1
fi

systemctl enable fail2ban >/dev/null 2>&1 || true
systemctl restart fail2ban || true
sleep 1

# 8) 健康检查；若 socket 不可达则自动修复一次
if ! fail2ban-client status >/dev/null 2>&1; then
  warn "检测到 fail2ban socket 不可访问，开始修复……"
  repair_fail2ban
fi

# 9) 展示状态与用法
echo
msg "Fail2Ban 总览："
if ! fail2ban-client status; then
  err "仍无法访问 fail2ban socket（/var/run/fail2ban/fail2ban.sock）。请执行：
  systemctl status fail2ban --no-pager -l
  journalctl -u fail2ban -b --no-pager | tail -n 100"
else
  echo
  cat <<USAGE
常用命令：
  # 查看 SSH 监狱详情
  fail2ban-client status sshd

  # 手动封禁/解封（也会写入 ${BAN_LOG}）
  fail2ban-client set sshd banip <IP>
  fail2ban-client set sshd unbanip <IP>

  # 实时查看审计日志
  tail -f ${BAN_LOG}

说明：
- 本脚本仅启动/重启 fail2ban，不修改 sshd 配置/端口，不影响其他服务。
- firewalld 场景使用运行时规则（非 --permanent）；主机重启后由 fail2ban 自动恢复拦截。
- 审计日志字段：时间、jail、ip、port、failures、src（来源日志路径）。
USAGE
fi
```


# 优化选择部署脚本

```
#!/usr/bin/env bash
# setup_ssh_antibrute.sh
# CentOS 7.6：Fail2Ban SSH 防暴力破解（安装/配置 + 自愈修复 + 交互菜单）
# - 仅影响 fail2ban；不修改 sshd 配置与端口，不影响其它服务
# - firewalld 在跑：banaction=firewallcmd-ipset（运行时规则，非永久）；未跑：iptables-multiport
# - BAN/UNBAN 审计日志可定制（默认 /home/lighthouse/bash/ssh-ban.log；或 BAN_LOG="__SCRIPT_DIR__" 落脚本同目录）
# - 自愈：socket 不可达时自动清理/修复/重启
# - 菜单：部署/重启/停止/查看运行状态/查看监狱/封禁/解封/看日志/卸载/退出

set -euo pipefail

##### 可调参数（也可用环境变量覆盖） #####
BANTIME="${BANTIME:-3600}"     # 封禁时长（秒）
FINDTIME="${FINDTIME:-600}"    # 观察窗口（秒）
MAXRETRY="${MAXRETRY:-5}"      # 失败次数阈值
MY_IP="${MY_IP:-}"             # 白名单（可空），例如 1.2.3.4
DEFAULT_BAN_LOG="/home/lighthouse/bash/ssh-ban.log"

# 日志位置：支持 BAN_LOG="__SCRIPT_DIR__"
SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"
if [[ "${BAN_LOG:-}" == "__SCRIPT_DIR__" ]]; then
  BAN_LOG="${SCRIPT_DIR}/ssh-ban.log"
else
  BAN_LOG="${BAN_LOG:-$DEFAULT_BAN_LOG}"
fi

##### 工具函数 #####
msg(){ echo -e "\033[1;32m[INFO]\033[0m $*"; }
warn(){ echo -e "\033[1;33m[WARN]\033[0m $*"; }
err(){ echo -e "\033[1;31m[ERR ]\033[0m $*"; }
require_root(){ [[ ${EUID:-$(id -u)} -eq 0 ]] || { err "请用 root 运行：sudo bash $0"; exit 1; }; }

##### 自愈修复：清理残留/重建目录/恢复 SELinux/补齐组件/重启 #####
repair_fail2ban() {
  warn "触发自愈修复：清理残留并重建运行环境……"
  systemctl stop fail2ban || true
  pkill -9 -f fail2ban-server || true

  rm -rf /run/fail2ban /var/run/fail2ban
  install -d -m 755 -o root -g root /run/fail2ban
  ln -sfn /run/fail2ban /var/run/fail2ban

  # SELinux（若启用则恢复上下文，无副作用）
  if command -v selinuxenabled >/dev/null 2>&1 && selinuxenabled; then
    restorecon -Rv /run/fail2ban || true
  fi

  # 兜底：iptables 组件
  yum install -y -q iptables iptables-services || true

  # 确保 fail2ban.conf 使用标准 socket/pid 路径
  local conf="/etc/fail2ban/fail2ban.conf"
  if [[ -f "$conf" ]]; then
    grep -qE '^\s*socket\s*=\s*/var/run/fail2ban/fail2ban\.sock' "$conf" || \
      sed -ri 's|^\s*socket\s*=.*|socket = /var/run/fail2ban/fail2ban.sock|g' "$conf"
    grep -qE '^\s*pidfile\s*=\s*/var/run/fail2ban/fail2ban\.pid' "$conf" || \
      sed -ri 's|^\s*pidfile\s*=.*|pidfile = /var/run/fail2ban/fail2ban.pid|g' "$conf"
  fi

  systemctl restart fail2ban || true
  sleep 1
}

##### 部署：安装/配置/测试/启动（不改 sshd） #####
deploy_fail2ban() {
  # 依赖
  if ! rpm -qa | grep -qiE '^epel-release'; then
    msg "安装 epel-release ..."
    yum install -y epel-release
  fi
  if ! rpm -qa | grep -qiE '^fail2ban(-server)?'; then
    msg "安装 fail2ban ..."
    yum install -y fail2ban
  else
    msg "fail2ban 已安装"
  fi

  # 检测 firewalld
  local FIREWALLD_ACTIVE=0
  if systemctl is-active firewalld >/dev/null 2>&1; then
    FIREWALLD_ACTIVE=1
    msg "firewalld 运行中：banaction=firewallcmd-ipset（运行时规则，非永久）"
  else
    warn "firewalld 未运行：banaction=iptables-multiport"
  fi
  local BANACTION="iptables-multiport"
  [[ $FIREWALLD_ACTIVE -eq 1 ]] && BANACTION="firewallcmd-ipset"

  # 自定义动作（占位符用 <name>/<ip>/<port>/<failures>；printf 用 %%s；date 用 %%F %%T）
  cat >/etc/fail2ban/action.d/log-ban.local <<'EOF'
[Definition]
# 说明：
#  - <name>     ：jail 名（如 sshd）
#  - <ip>       ：被封 IP
#  - <port>     ：端口
#  - <failures> ：失败次数
#  - %(logfile)s：在 jail 中通过 log-ban[logfile=...] 传入
#  - %(src)s    ：在 jail 中通过 log-ban[src=...] 传入（来源日志路径）
actionban   = /bin/sh -c 'printf "%%s\tBAN\tjail=<name>\tip=<ip>\tport=<port>\tfailures=<failures>\tsrc=%(src)s\n" "$(date "+%%F %%T")" >> %(logfile)s'
actionunban = /bin/sh -c 'printf "%%s\tUNBAN\tjail=<name>\tip=<ip>\n" "$(date "+%%F %%T")" >> %(logfile)s'
EOF
  chmod 0644 /etc/fail2ban/action.d/log-ban.local

  # 生成 jail.local（仅开启 sshd 监狱）
  local JAIL_LOCAL="/etc/fail2ban/jail.local"
  if [[ -f "$JAIL_LOCAL" ]]; then
    cp -a "$JAIL_LOCAL" "${JAIL_LOCAL}.bak.$(date +%Y%m%d-%H%M%S)"
    msg "已备份原配置：${JAIL_LOCAL}.bak.*"
  fi

  local IGNOREIP="127.0.0.1/8"
  [[ -n "$MY_IP" ]] && IGNOREIP="$IGNOREIP $MY_IP"

  cat >"$JAIL_LOCAL" <<EOF
[DEFAULT]
bantime   = ${BANTIME}
findtime  = ${FINDTIME}
maxretry  = ${MAXRETRY}
backend   = auto
ignoreip  = ${IGNOREIP}
banaction = ${BANACTION}

[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/secure
# 组合动作：真实封禁 + 写审计日志（src 指定来源日志路径）
action   = %(action_)s
           log-ban[logfile=${BAN_LOG}, src=/var/log/secure]
EOF
  chmod 0644 "$JAIL_LOCAL"

  # 日志与 logrotate
  mkdir -p "$(dirname -- "$BAN_LOG")"
  touch "$BAN_LOG"
  chmod 0640 "$BAN_LOG"
  chown root:root "$BAN_LOG"
  cat >/etc/logrotate.d/ssh-ban <<EOF
${BAN_LOG} {
    daily
    rotate 14
    missingok
    notifempty
    compress
    create 0640 root root
}
EOF
  chmod 0644 /etc/logrotate.d/ssh-ban

  # 运行目录兜底
  install -d -m 755 -o root -g root /run/fail2ban
  ln -sfn /run/fail2ban /var/run/fail2ban

  # 语法测试 & 启动
  msg "校验 fail2ban 配置语法 ..."
  fail2ban-client -t

  systemctl enable fail2ban >/dev/null 2>&1 || true
  systemctl restart fail2ban || true
  sleep 1

  # 健康检查：socket 不可达则自愈修复一次
  if ! fail2ban-client status >/dev/null 2>&1; then
    warn "检测到 fail2ban socket 不可访问，开始修复……"
    repair_fail2ban
  fi

  # 展示总览
  echo
  msg "Fail2Ban 总览："
  fail2ban-client status || true
}

##### 菜单动作函数 #####
show_service_status(){ systemctl status fail2ban --no-pager -l; }
show_jail(){ fail2ban-client status sshd; }
ban_ip(){ read -rp "请输入要封禁的 IP: " ip; [[ -n "${ip:-}" ]] && fail2ban-client set sshd banip "$ip"; }
unban_ip(){ read -rp "请输入要解封的 IP: " ip; [[ -n "${ip:-}" ]] && fail2ban-client set sshd unbanip "$ip"; }
tail_log(){ echo "[Ctrl+C 退出跟踪]"; tail -f "$BAN_LOG"; }

uninstall_fail2ban(){
  echo "[WARN] 卸载会移除 Fail2Ban 与相关配置，日志文件将保留：${BAN_LOG}"
  read -rp "确认卸载？(yes/no): " ans
  if [[ "$ans" == "yes" ]]; then
    systemctl stop fail2ban || true
    systemctl disable fail2ban || true
    yum remove -y fail2ban || true
    rm -f /etc/fail2ban/jail.local
    rm -f /etc/fail2ban/action.d/log-ban.local
    rm -f /etc/logrotate.d/ssh-ban
    echo "[INFO] 卸载完成，日志保留在 ${BAN_LOG}"
  else
    echo "[INFO] 已取消卸载"
  fi
}

##### 主流程：首次先部署一次，再进菜单 #####
require_root
deploy_fail2ban

# 交互菜单
while true; do
  echo "=============================="
  echo "  1) 部署脚本（重新安装/配置 Fail2Ban）"
  echo "  2) 重启服务"
  echo "  3) 停止服务"
  echo "  4) 查看运行状态"
  echo "  5) 查看 SSH 监狱详情"
  echo "  6) 手动封禁 IP"
  echo "  7) 手动解封 IP"
  echo "  8) 实时查看审计日志 (${BAN_LOG})"
  echo "  9) 卸载脚本（移除配置，保留日志）"
  echo " 10) 退出"
  echo "=============================="
  read -rp "请选择操作 [1-10]: " choice
  case "$choice" in
    1) deploy_fail2ban ;;
    2) systemctl restart fail2ban; echo "[INFO] Fail2Ban 已重启" ;;
    3) systemctl stop fail2ban; echo "[INFO] Fail2Ban 已停止" ;;
    4) show_service_status ;;
    5) show_jail ;;
    6) ban_ip ;;
    7) unban_ip ;;
    8) tail_log ;;
    9) uninstall_fail2ban ;;
    10) echo "退出."; break ;;
    *) echo "无效选择，请输入 1-10." ;;
  esac
done
 ```