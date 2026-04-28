文本模型批量生成文本测试

## ChatGPT 

第三方代理接口

-  [https://openkey.cloud](https://openkey.cloud/register?aff=22CVF)

### 执行脚本
```
from openai import OpenAI
import time
import csv
import os
from datetime import datetime

# 初始化客户端
client = OpenAI(
    api_key="sk-x",
    base_url="https://openkey.cloud/v1"
)

primary_classes = [
    "案件案例", "博客文章", "个人日记", "观点", "广告文案", "技术文档",
    "评论", "散文", "社交媒体帖子", "诗歌", "小说片段", "新闻报道", "学术论文摘要"
]

secondary_classes = [
    "AI", "动物", "情感", "公益", "购物", "古代文明", "交通", "教育", "近代战争", "经济",
    "科幻", "科技", "科普", "历史", "旅行", "美食", "母婴", "奇幻", "气候变化", "三农",
    "社会问题", "摄影", "生活", "时尚", "时政", "体育", "文化", "武器", "校园", "医疗",
    "艺术", "音乐", "影视", "游戏", "娱乐", "育儿", "职场", "植物", "商业"
]

styles = ["正式", "叙事", "情感化", "科普"]

category_map = {
    "案件案例": "Case Study",
    "博客文章": "Blog Article",
    "个人日记": "Personal Diary",
    "观点": "Opinion",
    "广告文案": "Advertising Copy",
    "技术文档": "Technical Document",
    "评论": "Review",
    "散文": "Essay",
    "社交媒体帖子": "Social Media Post",
    "诗歌": "Poetry",
    "小说片段": "Fiction Excerpt",
    "新闻报道": "News Report",
    "学术论文摘要": "Academic Abstract",
    "AI": "Artificial Intelligence",
    "动物": "Animals",
    "情感": "Emotion",
    "公益": "Public Welfare",
    "购物": "Shopping",
    "古代文明": "Ancient Civilization",
    "交通": "Transportation",
    "教育": "Education",
    "近代战争": "Modern War",
    "经济": "Economics",
    "科幻": "Science Fiction",
    "科技": "Technology",
    "科普": "Popular Science",
    "历史": "History",
    "旅行": "Travel",
    "美食": "Cuisine",
    "母婴": "Mother and Baby",
    "奇幻": "Fantasy",
    "气候变化": "Climate Change",
    "三农": "Agriculture and Rural Affairs",
    "社会问题": "Social Issues",
    "摄影": "Photography",
    "生活": "Lifestyle",
    "时尚": "Fashion",
    "时政": "Current Politics",
    "体育": "Sports",
    "文化": "Culture",
    "武器": "Weapons",
    "校园": "Campus",
    "医疗": "Medical",
    "艺术": "Art",
    "音乐": "Music",
    "影视": "Film and TV",
    "游戏": "Gaming",
    "娱乐": "Entertainment",
    "育儿": "Parenting",
    "职场": "Workplace",
    "植物": "Plants",
    "商业": "Business",
    "正式": "Formal",
    "叙事": "Narrative",
    "情感化": "Emotional",
    "科普": "Popular Science"
}

def char_count(text: str) -> int:
    return len(text)

def generate_text(primary, secondary, style, max_retries=3):
    primary_en = category_map.get(primary, primary)
    secondary_en = category_map.get(secondary, secondary)
    style_en = category_map.get(style, style)

    prompt = (
        f"Please write a coherent, well-structured English text with at least 250 characters and preferably no more than 350 characters about the following:\n"
        f"Primary category: {primary_en}\n"
        f"Secondary category: {secondary_en}\n"
        f"Writing style: {style_en}\n"
        f"Important: The entire text must be in English without any Chinese characters or words."
    )
    text = ""
    for attempt in range(1, max_retries + 1):
        try:
            response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=[{"role": "user", "content": prompt}],
                temperature=0.7,
                max_tokens=900
            )
            text = response.choices[0].message.content.strip()
            char_num = char_count(text)
            if char_num >= 250:
                return text
            else:
                print(f"Retry {attempt} for {primary}-{secondary}-{style}, char count {char_num} < 250")
                time.sleep(1)
        except Exception as e:
            print(f"Error for {primary}-{secondary}-{style}: {e}")
            time.sleep(2)
    print(f"Max retries reached for {primary}-{secondary}-{style}, returning last result")
    return text

def write_to_csv_with_timestamp(base_name, rows, batch_size, output_dir="D:/data/output"):
    os.makedirs(output_dir, exist_ok=True)
    now_str = datetime.now().strftime("%Y%m%d%H%M")
    filename = f"{base_name}_{now_str}_{batch_size}.csv"
    full_path = os.path.join(output_dir, filename)
    with open(full_path, 'w', newline='', encoding='utf-8-sig') as f:
        writer = csv.writer(f)
        writer.writerow(["编号", "一级类", "二级类", "风格", "内容", "字符数"])
        writer.writerows(rows)
    print(f"Saved batch of {len(rows)} records to {full_path}")

def main():
    total_tasks = len(primary_classes) * len(secondary_classes) * len(styles)
    task_counter = 0
    batch_size = 5  # 生成5条保存成表
    buffer = []
    base_name = "generated_texts"
    output_dir = r"C:\test"  # 你需要的输出目录，请修改为你想要的路径

    for primary in primary_classes:
        for secondary in secondary_classes:
            for style in styles:
                task_counter += 1
                print(f"\n[{task_counter}/{total_tasks}] Generating: {primary} - {secondary} - {style}\n")
                content = generate_text(primary, secondary, style)
                char_num = char_count(content)
                print(f"Content ({char_num} chars):\n")
                print(content)
                print("\n" + "="*80 + "\n")

                buffer.append([task_counter, primary, secondary, style, content, char_num])

                if len(buffer) >= batch_size:
                    write_to_csv_with_timestamp(base_name, buffer, batch_size, output_dir=output_dir)
                    buffer.clear()

                time.sleep(1)  # 限流防封禁

    if buffer:
        write_to_csv_with_timestamp(base_name, buffer, len(buffer), output_dir=output_dir)

    print("All done!")

if __name__ == "__main__":
    main()
```
### 输出结果
```
[354/2028] Generating: 个人日记 - 科幻 - 叙事
Generated chars: 400
Full content:
October 12, 2147

Today, I stumbled upon an ancient device in the ruins of an old library—an old smartphone. Its screen flickered to life, revealing images of a world long gone. I felt a surge of nostalgia for a time when humans thrived on connection, not just data. As I scrolled through its apps, I wondered what stories lay hidden in its memory, waiting to bridge the gap between past and present.
```

## DeepSeek
- [https://platform.deepseek.com/api_keys](https://platform.deepseek.com/api_keys)
### 运行脚本
```
from openai import OpenAI
import time
import csv
import os
from datetime import datetime

# 初始化客户端，替换成 DeepSeek 的 base_url 和 api_key
client = OpenAI(
    api_key="sk-x",  # 这里换成你在 DeepSeek 申请的 API Key
    base_url="https://api.deepseek.com"    # DeepSeek API 地址，带/v1也可以
)

primary_classes = [
    "案件案例", "博客文章", "个人日记", "观点", "广告文案", "技术文档",
    "评论", "散文", "社交媒体帖子", "诗歌", "小说片段", "新闻报道", "学术论文摘要"
]

secondary_classes = [
    "AI", "动物", "情感", "公益", "购物", "古代文明", "交通", "教育", "近代战争", "经济",
    "科幻", "科技", "科普", "历史", "旅行", "美食", "母婴", "奇幻", "气候变化", "三农",
    "社会问题", "摄影", "生活", "时尚", "时政", "体育", "文化", "武器", "校园", "医疗",
    "艺术", "音乐", "影视", "游戏", "娱乐", "育儿", "职场", "植物", "商业"
]

styles = ["正式", "叙事", "情感化", "科普"]

def generate_text(primary: str, secondary: str, style: str, max_retries=3) -> str:
    prompt = (
        f"Please write an English text about the following topic.\n"
        f"The text must be coherent and well-structured,\n"
        f"with at least 200 characters. Avoid making the text too long.\n\n"
        f"Primary category: {primary}\n"
        f"Secondary category: {secondary}\n"
        f"Writing style: {style}"
    )
    text = ""
    for attempt in range(1, max_retries + 1):
        try:
            response = client.chat.completions.create(
                model="deepseek-chat",
                messages=[{"role": "user", "content": prompt}],
                temperature=0.7,
                max_tokens=500  # 允许稍长文本，模型自动控制长度
            )
            text = response.choices[0].message.content.strip()
            length = len(text)
            if length >= 200:
                return text.replace('\n', ' ')
            else:
                print(f"Retry {attempt} for {primary} - {secondary} - {style}: char count {length} < 200")
                time.sleep(1)
        except Exception as e:
            print(f"Error on {primary} - {secondary} - {style}: {e}")
            time.sleep(2)

    print(f"Max retries reached for {primary} - {secondary} - {style}. Returning last result.")
    if text:
        return text.replace('\n', ' ')
    return ""

def save_batch_to_csv(rows, batch_num, base_name="deepseek_output", output_dir="output"):
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
    filename = f"{base_name}_{timestamp}_batch{batch_num}.csv"
    filepath = os.path.join(output_dir, filename)
    with open(filepath, mode='w', encoding='utf-8-sig', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(["编号", "一级类", "二级类", "风格", "内容", "字符数"])
        writer.writerows(rows)
    print(f"Saved batch {batch_num} with {len(rows)} records to {filepath}")

def main():
    total_tasks = len(primary_classes) * len(secondary_classes) * len(styles)
    task_counter = 0
    batch_size = 5  # 生成5条保存成表
    batch_data = []
    batch_number = 1
    base_name = "deepseek_output"
    output_dir = r"c:\test"  # 修改成你想保存的路径

    for primary in primary_classes:
        for secondary in secondary_classes:
            for style in styles:
                task_counter += 1
                print(f"\n[{task_counter}/{total_tasks}] Generating: {primary} - {secondary} - {style}\n")
                content = generate_text(primary, secondary, style)
                length = len(content)
                print(f"Content ({length} characters):\n")
                print(content)
                print("\n" + "="*100 + "\n")

                batch_data.append([task_counter, primary, secondary, style, content, length])

                if len(batch_data) >= batch_size:
                    save_batch_to_csv(batch_data, batch_number, base_name, output_dir)
                    batch_data.clear()
                    batch_number += 1

                time.sleep(1)  # 避免请求过快被限流

    if batch_data:
        save_batch_to_csv(batch_data, batch_number, base_name, output_dir)

    print("All tasks completed.")

if __name__ == "__main__":
    main()
```
### 输出结果
```
[15/2028] Generating: 案件案例 - 公益 - 情感化

Content (827 characters):

**A Beacon of Hope: The Power of Compassion in Legal Cases**    In the midst of cold courtrooms and rigid laws, some cases shine as reminders of humanity’s warmth. Take the story of an elderly woman evicted unfairly—her plight moved strangers to crowdfund her legal fees. Or the pro bono lawyers who fought for a child’s right to education against all odds. These stories aren’t just about justice; they’re about hearts uniting to lift others up.    Every such case whispers a truth: the law is stronger when wrapped in kindness. Behind every docket number is a life, and behind every verdict, a chance to heal. Let’s celebrate these unsung heroes—the donors, volunteers, and advocates—who turn legal battles into triumphs of empathy. Because justice, when paired with love, doesn’t just win—it transforms.    (Characters: 598)
```
## Kimi 

-  [https://platform.moonshot.cn/console/api-keys](https://platform.moonshot.cn/console/api-keys)

### 执行脚本
```
from openai import OpenAI
import time
import csv
import os
from datetime import datetime

# === 模型选择 ===
USE_MODEL = "moonshot"

# === 初始化客户端（Moonshot 中文）===
client = OpenAI(
    api_key="sk-xxx",  # ← 替换为你的 API Key
    base_url="https://api.moonshot.cn/v1"
)



model_name = "kimi-k2-0711-preview"
system_prompt = (
    "你是 Kimi，由 Moonshot AI 提供的人工智能助手。你擅长中文内容创作，能够根据给定的分类和风格，"
    "生成真实、连贯且结构良好的中文文本，内容不少于200字。"
)

# === 分类定义 ===
primary_classes = [
    "案件案例", "博客文章", "个人日记", "观点", "广告文案", "技术文档",
    "评论", "散文", "社交媒体帖子", "诗歌", "小说片段", "新闻报道", "学术论文摘要"
]

secondary_classes = [
    "AI", "动物", "情感", "公益", "购物", "古代文明", "交通", "教育", "近代战争", "经济",
    "科幻", "科技", "科普", "历史", "旅行", "美食", "母婴", "奇幻", "气候变化", "三农",
    "社会问题", "摄影", "生活", "时尚", "时政", "体育", "文化", "武器", "校园", "医疗",
    "艺术", "音乐", "影视", "游戏", "娱乐", "育儿", "职场", "植物", "商业"
]

styles = ["正式", "叙事", "情感化", "科普"]

# === 内容生成函数 ===
def generate_text(primary: str, secondary: str, style: str, max_retries=3) -> str:
    prompt = (
        f"请根据以下要求，撰写一段结构清晰、通顺连贯的中文内容。\n"
        f"内容长度应不少于200字，不要过长。\n\n"
        f"一级分类：{primary}\n"
        f"二级分类：{secondary}\n"
        f"写作风格：{style}"
    )
    text = ""
    for attempt in range(1, max_retries + 1):
        try:
            messages = [{"role": "system", "content": system_prompt},
                        {"role": "user", "content": prompt}]

            response = client.chat.completions.create(
                model=model_name,
                messages=messages,
                temperature=0.7,
                max_tokens=500,
                timeout=60  # ⏰ 防止长时间卡住
            )
            text = response.choices[0].message.content.strip()
            if len(text) >= 200:
                return text.replace('\n', ' ')
            else:
                print(f"⚠️ Retry {attempt}: 内容太短（{len(text)} 字符）")
                time.sleep(1)
        except Exception as e:
            print(f"❌ 错误：{primary}-{secondary}-{style} 第 {attempt} 次尝试失败：{e}")
            time.sleep(2)

    # 最终失败处理
    fail_msg = "生成失败：内容为空"
    with open("error_log.txt", "a", encoding="utf-8") as f:
        f.write(f"[失败] {primary}-{secondary}-{style}\n")
    return fail_msg

# === 保存 CSV ===
def save_batch_to_csv(rows, batch_num, base_name="output", output_dir="output"):
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
    filename = f"{base_name}_{timestamp}_batch{batch_num}.csv"
    filepath = os.path.join(output_dir, filename)
    with open(filepath, mode='w', encoding='utf-8-sig', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(["编号", "一级类", "二级类", "风格", "内容", "字符数"])
        writer.writerows(rows)
    print(f"✅ 保存第 {batch_num} 批，共 {len(rows)} 条 → {filepath}")

# === 主函数 ===
def main():
    total_tasks = len(primary_classes) * len(secondary_classes) * len(styles)  # = 2028
    task_counter = 0
    batch_data = []
    batch_size = 52
    batch_number = 1
    output_dir = r"C:\test"  # ← 替换为你的输出目录

    for primary in primary_classes:
        for secondary in secondary_classes:
            for style in styles:
                task_counter += 1
                print(f"\n[{task_counter}/{total_tasks}] ⏳ 正在生成：{primary} - {secondary} - {style}")
                content = generate_text(primary, secondary, style)
                length = len(content)
                print(f"→ 内容长度: {length} 字符")
                print("内容：")
                print(content)
                print("=" * 100)

                batch_data.append([task_counter, primary, secondary, style, content, length])

                if len(batch_data) >= batch_size:
                    save_batch_to_csv(batch_data, batch_number, base_name="moonshot", output_dir=output_dir)
                    batch_data.clear()
                    batch_number += 1
                time.sleep(1)

    if batch_data:
        save_batch_to_csv(batch_data, batch_number, base_name="moonshot", output_dir=output_dir)

    print("\n🎉 所有 2028 条中文内容生成完毕！")

if __name__ == "__main__":
    main()
```

### 输出结果
```
[37/2028] ⏳ 正在生成：案件案例 - 经济 - 正式
→ 内容长度: 534 字符
内容：
案例名称：金辉集团财务造假案  案件背景：2022年3月，经证监会稽查总队立案，上市公司金辉集团（代码：603877）被指控在2019至2021年间，通过虚增海外工程收入、提前确认未完工项目利润及利用关联方循环交易等方式，累计虚增营业收入人民币24.6亿元，虚增净 利润7.8亿元，导致年度报告存在重大虚假记载。  调查与审理：证监会联合财政部、公安部成立专案组，调取金辉集团及其上下游企业 的电子账套、银行流水及海运提单。经比对，发现其海外项目完工率被系统篡改，且多家注册于英属维京群岛的壳公司与实际控制人存在隐蔽股权关系。2023年1月，案件移送上海市人民检察院第三分院审查起诉。同年12月，上海市第三中级人民法院以违规披露重要信息罪 、操纵证券市场罪两罪并罚，判处金辉集团罚金人民币3亿元；对时任董事长赵某判处有期徒刑七年，并处罚金2000万元；对财务总监等 五名直接责任人员分别判处三至五年不等有期徒刑。  典型意义：本案系注册制下首例以“全链条证据穿透”认定财务造假的标杆案件。判决书首次确认，上市公司通过“海外项目”跨境舞弊同样适用《刑法》第一百六十一条，为后续同类案件审理提供了明确裁判标准，并推动证监会修订《上市公司现场检查办法》，强化了对异常境外收入的监管。
====================================================================================================
```

## 腾讯混元

-  [https://console.cloud.tencent.com/cam/capi](https://console.cloud.tencent.com/cam/capi)

### 执行脚本
```
# -*- coding: utf-8 -*-
import time
import csv
import os
import json
from datetime import datetime
from tencentcloud.common import credential
from tencentcloud.common.profile.client_profile import ClientProfile
from tencentcloud.common.profile.http_profile import HttpProfile
from tencentcloud.hunyuan.v20230901 import hunyuan_client, models

# pip install tencentcloud-sdk-python

# === 模型 & 提示词 ===
system_prompt = (
    "你是由腾讯云混元大模型提供的中文文本生成助手，擅长撰写结构清晰、真实、连贯的内容，风格多样，符合指定语体要求。"
)

# === 分类定义 ===
primary_classes = [
    "案件案例", "博客文章", "个人日记", "观点", "广告文案", "技术文档",
    "评论", "散文", "社交媒体帖子", "诗歌", "小说片段", "新闻报道", "学术论文摘要"
]

secondary_classes = [
    "AI", "动物", "情感", "公益", "购物", "古代文明", "交通", "教育", "近代战争", "经济",
    "科幻", "科技", "科普", "历史", "旅行", "美食", "母婴", "奇幻", "气候变化", "三农",
    "社会问题", "摄影", "生活", "时尚", "时政", "体育", "文化", "武器", "校园", "医疗",
    "艺术", "音乐", "影视", "游戏", "娱乐", "育儿", "职场", "植物", "商业"
]

styles = ["正式", "叙事", "情感化", "科普"]

# === 初始化腾讯混元客户端 ===
cred = credential.Credential("xxx", "xxx")  # ← 替换为你的密钥
httpProfile = HttpProfile()
httpProfile.endpoint = "hunyuan.tencentcloudapi.com"
clientProfile = ClientProfile()
clientProfile.httpProfile = httpProfile
client = hunyuan_client.HunyuanClient(cred, "", clientProfile)

# === 内容生成函数 ===
def generate_text(primary: str, secondary: str, style: str, max_retries=3) -> str:
    prompt = (
        f"请根据以下要求撰写一段中文内容，结构清晰、语义连贯。\n"
        f"内容长度不少于200字，避免冗长。\n\n"
        f"一级分类：{primary}\n"
        f"二级分类：{secondary}\n"
        f"写作风格：{style}"
    )

    for attempt in range(1, max_retries + 1):
        try:
            req = models.ChatCompletionsRequest()
            params = {
                "Model": "hunyuan-turbo",
                "Messages": [
                    {"Role": "system", "Content": system_prompt},
                    {"Role": "user", "Content": prompt}
                ],
                "Temperature": 0.7,
                "TopP": 0.8,
                "MaxTokens": 500
            }
            req.from_json_string(json.dumps(params))
            resp = client.ChatCompletions(req)
            text = resp.Choices[0].Message.Content.strip()
            if len(text) >= 200:
                return text.replace('\n', ' ')
            else:
                print(f"⚠️ Retry {attempt}: 内容太短（{len(text)} 字符）")
                time.sleep(1)
        except Exception as e:
            print(f"❌ 错误：{primary}-{secondary}-{style} 第 {attempt} 次尝试失败：{e}")
            time.sleep(2)

    with open("error_log.txt", "a", encoding="utf-8") as f:
        f.write(f"[失败] {primary}-{secondary}-{style}\n")
    return "生成失败：内容为空"

# === 保存 CSV ===
def save_batch_to_csv(rows, batch_num, base_name="output", output_dir="output"):
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d%H%M%S")
    filename = f"{base_name}_{timestamp}_batch{batch_num}.csv"
    filepath = os.path.join(output_dir, filename)
    with open(filepath, mode='w', encoding='utf-8-sig', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(["编号", "一级类", "二级类", "风格", "内容", "字符数"])
        writer.writerows(rows)
    print(f"✅ 保存第 {batch_num} 批，共 {len(rows)} 条 → {filepath}")

# === 主函数 ===
def main():
    total_tasks = len(primary_classes) * len(secondary_classes) * len(styles)  # = 2028
    task_counter = 0
    batch_data = []
    batch_size = 52
    batch_number = 1
    output_dir = r"C:\test"  # ← 替换为你自己的保存目录

    for primary in primary_classes:
        for secondary in secondary_classes:
            for style in styles:
                task_counter += 1
                print(f"\n[{task_counter}/{total_tasks}] ⏳ 正在生成：{primary} - {secondary} - {style}")
                content = generate_text(primary, secondary, style)
                length = len(content)
                print(f"→ 内容长度: {length} 字符")
                print("内容：")
                print(content)
                print("=" * 100)

                batch_data.append([task_counter, primary, secondary, style, content, length])

                if len(batch_data) >= batch_size:
                    save_batch_to_csv(batch_data, batch_number, base_name="hunyuan", output_dir=output_dir)
                    batch_data.clear()
                    batch_number += 1
                time.sleep(1)

    if batch_data:
        save_batch_to_csv(batch_data, batch_number, base_name="hunyuan", output_dir=output_dir)

    print("\n🎉 所有 2028 条中文内容生成完毕！")

if __name__ == "__main__":
    main()

```
### 输出结果
```
[111/2028] ⏳ 正在生成：案件案例 - 武器 - 情感化
⚠️ Retry 1: 内容太短（177 字符）
→ 内容长度: 413 字符
内容：
《致命的武器》  在那个看似平静的小镇上，发生了一件令人痛心疾首的案件。受害者是一位无辜的路人，而这一切的罪恶源头，竟然是一把被恶意使用的武器。  那是一个阴霾密布的午后，街道上行人寥寥。犯罪嫌疑人不知从何处掏出了一把寒光闪闪的匕首，在毫无征兆的情况下，冲向了一位正匆匆赶路的女士。女士惊恐的眼神中充满了绝望，她试图躲避，可那把匕首就像恶魔的獠牙，无情地刺向她。周围的人都被这突如其来的一幕吓呆了，短暂的惊愕之后，才有人反应过来报警。  当警察迅速赶到现场时，受害者已经倒在血泊之中，生命垂危。那把冰冷的匕首，此时成为了罪恶的象征。它本是一种防身的工具，却在坏人的手中变成了伤害他人生命、破坏家庭幸福的凶器。这起案件让我们深刻地意识到，武器的危险性一旦失控，将会带来多么巨大的灾难。它不仅伤害了一个人的身体，更是给无数亲人的心中留下了永远无法磨灭的伤痛。我们渴望社会的安全与和谐，希望这样的悲剧不再因为被恶意使用的武器而发生。
```