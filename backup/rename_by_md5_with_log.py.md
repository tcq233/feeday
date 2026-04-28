以文件 MD5 作为文件名（保留扩展名），递归处理子文件夹，
    并生成一个 md5_rename_map.csv 记录原始文件名与改后文件名及MD5

```
import os
import csv
import hashlib
from pathlib import Path

# ======== 可修改配置 ========
ROOT_DIR = Path(r"C:\test\md")  # 要处理的目录
OUTPUT_CSV = ROOT_DIR / "md5_rename_map1025.csv"    # 输出映射表路径
DRY_RUN = False  # True 仅预览不执行重命名
# ===========================


def md5_of_file(file_path, chunk_size=8192):
    """计算文件 MD5"""
    md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        while chunk := f.read(chunk_size):
            md5.update(chunk)
    return md5.hexdigest()


def rename_file_to_md5(file_path: Path, writer):
    """重命名单个文件为md5，并写入CSV"""
    try:
        md5_val = md5_of_file(file_path)
        new_name = f"{md5_val}{file_path.suffix.lower()}"
        new_path = file_path.with_name(new_name)

        if new_path == file_path:
            # 已经是 md5 命名
            return

        if new_path.exists():
            print(f"⚠️ 已存在相同MD5文件，跳过：{file_path}")
            return

        if DRY_RUN:
            print(f"[DRY] {file_path} -> {new_path}")
        else:
            file_path.rename(new_path)
            print(f"✅ {file_path} -> {new_path}")

        # 写入映射表
        writer.writerow([str(file_path), str(new_path), md5_val])

    except Exception as e:
        print(f"❌ {file_path} 计算或重命名失败：{e}")


def main():
    with open(OUTPUT_CSV, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(["original_path", "new_path", "md5"])

        for file in ROOT_DIR.rglob("*"):
            if file.is_file():
                rename_file_to_md5(file, writer)

    print(f"\n📄 映射表已生成：{OUTPUT_CSV}")


if __name__ == "__main__":
    main()
```