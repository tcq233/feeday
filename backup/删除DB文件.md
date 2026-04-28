递归删除指定目录下所有 .db 文件，并生成日志（UTF-8）。
支持命令行参数：python delDB.py [目标路径]

```
import os
import sys
import time
from pathlib import Path

# ========== 默认配置 ==========
DEFAULT_ROOT = Path(r"C:\test")  # 默认扫描目录
LOG_FILE = Path(__file__).with_name("deleted_db_list.txt")  # 日志自动放脚本目录
SHOW_PRINT = True           # True：打印每个删除项
DRY_RUN = False             # True：只预览不删除（安全模式）
# ==============================

def delete_db_files(root: Path):
    """递归删除 .db 文件并记录日志"""
    count = 0
    total_size = 0
    start_time = time.time()

    with open(LOG_FILE, "w", encoding="utf-8") as log:
        log.write("Deleted .db files list\n")
        log.write(f"Root: {root}\n")
        log.write(f"Time: {time.strftime('%Y-%m-%d %H:%M:%S')}\n\n")

        for path in root.rglob("*.db"):
            try:
                size = path.stat().st_size
                total_size += size
                count += 1
                log.write(f"{path}\t{size} bytes\n")

                if SHOW_PRINT:
                    print(f"🗑️ 删除: {path} ({size} bytes)")

                if not DRY_RUN:
                    path.unlink()
            except Exception as e:
                log.write(f"❌ 无法删除: {path} -> {e}\n")
                if SHOW_PRINT:
                    print(f"❌ 无法删除: {path} -> {e}")

        elapsed = time.time() - start_time
        log.write(f"\n共删除 {count} 个文件，总大小 {total_size/1024:.2f} KB，用时 {elapsed:.2f} 秒。\n")

    print("\n========== 任务完成 ==========")
    print(f"📜 删除日志: {LOG_FILE}")
    print(f"📦 共删除: {count} 个文件, 总大小: {total_size/1024:.2f} KB")

if __name__ == "__main__":
    # 支持命令行参数：python delDB.py [路径]
    if len(sys.argv) > 1:
        ROOT_DIR = Path(sys.argv[1])
    else:
        ROOT_DIR = DEFAULT_ROOT

    if not ROOT_DIR.exists():
        print(f"❌ 路径不存在: {ROOT_DIR}")
        sys.exit(1)

    delete_db_files(ROOT_DIR)
```