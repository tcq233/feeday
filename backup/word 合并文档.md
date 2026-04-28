## 文档转换docx后合并
```
import os
import sys
import time
import shutil
import subprocess
from pathlib import Path

# ========== 自动安装依赖 ==========
def install(package_name, import_name=None):
    try:
        __import__(import_name or package_name)
    except ImportError:
        print(f"⚙️ 正在安装依赖: {package_name} ...")
        subprocess.check_call([sys.executable, "-m", "pip", "install", package_name])

install("python-docx", "docx")
install("docxcompose", "docxcompose")
try:
    __import__("win32com.client")
except ImportError:
    try:
        install("pywin32", "win32com")
    except Exception:
        pass

from docx import Document
from docxcompose.composer import Composer

# ========== 配置 ==========
input_dir = r"D:\doc"            # 待合并目录（含 .doc / .docx）
output_file = r"D:\merged.docx"  # 输出（必须 .docx）

# ========== 定位 soffice.exe ==========
def find_soffice_exe():
    candidates = [
        r"C:\Program Files\LibreOffice\program\soffice.exe",
        r"C:\Program Files (x86)\LibreOffice\program\soffice.exe",
    ]
    # 也支持便携版/自定义安装：在常见盘符搜一层
    for drive in ["C:", "D:", "E:"]:
        p = Path(drive + r"\LibreOffice\program\soffice.exe")
        if p.exists():
            candidates.insert(0, str(p))
    for p in candidates:
        if os.path.exists(p):
            return p
    # PATH 中查找
    w = shutil.which("soffice")
    return w

def has_word():
    try:
        import win32com.client  # noqa
        return True
    except Exception:
        return False

# ========== 转换函数 ==========
def convert_doc_to_docx_with_soffice(soffice_path, doc_path):
    outdir = os.path.dirname(doc_path)
    print(f"🔄 LibreOffice 转换: {doc_path}")
    try:
        subprocess.run(
            [soffice_path, "--headless", "--convert-to", "docx", "--outdir", outdir, doc_path],
            check=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=True
        )
        new_path = doc_path + "x"  # xxx.doc -> xxx.docx
        for _ in range(30):
            if os.path.exists(new_path):
                break
            time.sleep(0.1)
        if os.path.exists(new_path):
            print(f"✅ 转换成功: {new_path}")
            return new_path
        print(f"❌ 转换失败: 未找到 {new_path}")
    except subprocess.CalledProcessError as e:
        print("❌ LibreOffice 转换出错：")
        print(e.stdout or str(e))
    return None

def convert_doc_to_docx_with_word(doc_path):
    print(f"🔄 Word 转换: {doc_path}")
    try:
        import win32com.client as win32
        word = win32.gencache.EnsureDispatch("Word.Application")
        word.Visible = False
        doc = None
        new_path = doc_path + "x"
        try:
            doc = word.Documents.Open(doc_path)
            doc.SaveAs(new_path, FileFormat=16)  # 16 = wdFormatXMLDocument (.docx)
        finally:
            if doc is not None:
                doc.Close(False)
            word.Quit()
        if os.path.exists(new_path):
            print(f"✅ 转换成功: {new_path}")
            return new_path
        else:
            print(f"❌ 转换失败: 未找到 {new_path}")
    except Exception as e:
        print(f"❌ Word 转换出错: {e}")
    return None

def convert_doc_to_docx(doc_path):
    soffice = find_soffice_exe()
    if soffice and os.path.exists(soffice):
        p = convert_doc_to_docx_with_soffice(soffice, doc_path)
        if p:
            return p
    if has_word():
        p = convert_doc_to_docx_with_word(doc_path)
        if p:
            return p
    print(f"⚠️ 无法转换（缺少 LibreOffice 或 Word）：{doc_path}")
    return None

# ========== 合并 ==========
def merge_docs(input_dir, output_file):
    if not output_file.lower().endswith(".docx"):
        base, _ = os.path.splitext(output_file)
        output_file = base + ".docx"
        print(f"ℹ️ 输出强制为 .docx：{output_file}")

    names = [f for f in os.listdir(input_dir)
             if f.lower().endswith((".doc", ".docx")) and not f.startswith("~$")]
    names.sort()
    if not names:
        print("⚠️ 目录中没有 .doc / .docx 文件。")
        return

    # 统一准备成 .docx 列表
    docx_paths = []
    for name in names:
        p = os.path.join(input_dir, name)
        if name.lower().endswith(".docx"):
            docx_paths.append(p)
        else:
            newp = convert_doc_to_docx(p)
            if newp:
                docx_paths.append(newp)

    docx_paths = [p for p in docx_paths if os.path.exists(p)]
    if not docx_paths:
        print("⚠️ 没有可合并的 .docx（请先安装 LibreOffice 或 Word）。")
        return

    print(f"🧩 母版：{os.path.basename(docx_paths[0])}")
    master = Document(docx_paths[0])
    composer = Composer(master)

    for i, p in enumerate(docx_paths[1:], start=2):
        print(f"📄 追加第 {i} 个：{os.path.basename(p)}")
        doc = Document(p)
        if i <= len(docx_paths):
            doc.add_page_break()
        composer.append(doc)

    abs_inputs = set(map(os.path.abspath, docx_paths))
    out = output_file
    if os.path.abspath(out) in abs_inputs:
        out = os.path.join(os.path.dirname(out), "_merged_output.docx")
        print(f"ℹ️ 输出与输入冲突，改为：{out}")

    composer.save(out)
    print(f"\n✅ 合并完成：{out}")

if __name__ == "__main__":
    merge_docs(input_dir, output_file)

```