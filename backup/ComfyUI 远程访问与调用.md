```
pip install flask websocket-client requests
```

```
import json
import uuid
import websocket
import urllib.request
import urllib.parse
import random
import threading
import time
import requests
import io
from flask import Flask, request, jsonify, Response

app = Flask(__name__)

# ================= 基础配置 =================
COMFY_ADDR = "127.0.0.1:8188"
CLIENT_ID = str(uuid.uuid4())
gpu_lock = threading.Lock()

def queue_prompt(prompt_workflow):
    p = {"prompt": prompt_workflow, "client_id": CLIENT_ID}
    data = json.dumps(p).encode('utf-8')
    req = urllib.request.Request(f"http://{COMFY_ADDR}/prompt", data=data)
    return json.loads(urllib.request.urlopen(req).read())

def upload_image_to_comfyui(file_bytes, filename):
    """直接发送字节流到 ComfyUI"""
    url = f"http://{COMFY_ADDR}/upload/image"
    files = {'image': (filename, file_bytes)}
    res = requests.post(url, files=files)
    return res.json()['name']

# ================= 前端页面路由 =================
@app.route('/')
def index():
    return '''
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
        <title>Z-Image AI 创作中心</title>
        <style>
            body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f4f4f9; padding: 15px; margin: 0; display: flex; justify-content: center; }
            .container { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); width: 100%; max-width: 600px; box-sizing: border-box; }
            h2 { text-align: center; margin-top: 0; color: #333; }
            .tabs { display: flex; margin-bottom: 20px; border-bottom: 2px solid #eee; }
            .tab { flex: 1; text-align: center; padding: 10px; cursor: pointer; font-weight: bold; color: #888; transition: 0.3s; }
            .tab.active { color: #007bff; border-bottom: 2px solid #007bff; }
            .input-group { margin-bottom: 15px; text-align: left; }
            .input-group label { display: block; margin-bottom: 5px; font-size: 14px; color: #555; }
            input[type="text"], input[type="file"] { width: 100%; padding: 12px; font-size: 15px; border: 1px solid #ccc; border-radius: 8px; box-sizing: border-box; }
            button { width: 100%; padding: 14px; font-size: 16px; background-color: #007bff; color: white; border: none; border-radius: 8px; cursor: pointer; transition: 0.3s; font-weight: bold; }
            #progressContainer { display: none; margin-top: 15px; }
            .progress-bar-bg { width: 100%; background-color: #e9ecef; border-radius: 8px; overflow: hidden; height: 12px; }
            .progress-bar-fill { height: 100%; background-color: #28a745; width: 0%; transition: width 0.2s; }
            #status { margin: 15px 0 5px 0; font-weight: bold; color: #555; text-align: center; font-size: 14px;}
            #timeInfo { font-size: 12px; color: #999; text-align: center; margin-bottom: 10px; display: none; }
            #resultImg { width: 100%; border-radius: 8px; margin-top: 10px; display: none; box-shadow: 0 2px 8px rgba(0,0,0,0.2); }
        </style>
    </head>
    <body>
        <div class="container">
            <h2>✨ AI 创作中心</h2>
            <div class="tabs">
                <div class="tab active" onclick="switchMode('t2i')" id="tab-t2i">文生图</div>
                <div class="tab" onclick="switchMode('i2i')" id="tab-i2i">图改图</div>
            </div>
            <form id="genForm">
                <div class="input-group" id="file-upload-group" style="display: none;">
                    <label>上传参考图：</label>
                    <input type="file" id="uploadImage" accept="image/*">
                </div>
                <div class="input-group">
                    <label id="prompt-label">画面描述：</label>
                    <input type="text" id="prompt" value="a beautiful girl, cinematic lighting, 8k">
                </div>
                <button type="button" id="genBtn" onclick="generate()">立即生成</button>
            </form>
            <p id="status"></p>
            <div id="timeInfo"></div>
            <div id="progressContainer">
                <div class="progress-bar-bg"><div class="progress-bar-fill" id="progressFill"></div></div>
            </div>
            <img id="resultImg" src="">
        </div>

        <script>
            let currentMode = 't2i';
            function switchMode(mode) {
                currentMode = mode;
                document.getElementById('tab-t2i').className = mode === 't2i' ? 'tab active' : 'tab';
                document.getElementById('tab-i2i').className = mode === 'i2i' ? 'tab active' : 'tab';
                document.getElementById('file-upload-group').style.display = mode === 'i2i' ? 'block' : 'none';
                document.getElementById('prompt').value = mode === 'i2i' ? '猫换成狗' : 'a beautiful girl, cinematic lighting, 8k';
            }

            async function generate() {
                const btn = document.getElementById('genBtn');
                const status = document.getElementById('status');
                const timeInfo = document.getElementById('timeInfo');
                const img = document.getElementById('resultImg');
                const progressFill = document.getElementById('progressFill');
                const progressContainer = document.getElementById('progressContainer');
                
                btn.disabled = true;
                status.innerText = "🚀 正在连接 AI 服务器...";
                timeInfo.style.display = "none";
                img.style.display = "none";
                progressContainer.style.display = "block";
                progressFill.style.width = "0%";

                const formData = new FormData();
                formData.append('mode', currentMode);
                formData.append('prompt', document.getElementById('prompt').value);
                if(currentMode === 'i2i') formData.append('image', document.getElementById('uploadImage').files[0]);

                try {
                    const response = await fetch('/generate', { method: 'POST', body: formData });
                    const reader = response.body.getReader();
                    const decoder = new TextDecoder("utf-8");
                    let buffer = "";
                    let startTime = 0;

                    while (true) {
                        const { done, value } = await reader.read();
                        if (done) break;
                        buffer += decoder.decode(value, { stream: true });
                        let lines = buffer.split('\\n');
                        buffer = lines.pop(); 
                        
                        for (let line of lines) {
                            if (!line.trim()) continue;
                            const data = JSON.parse(line);
                            if (data.status === "progress") {
                                if (startTime === 0) startTime = Date.now();
                                let percent = Math.round((data.value / data.max) * 100);
                                progressFill.style.width = percent + "%";
                                status.innerText = `🔄 正在绘制: ${percent}%`;
                            } else if (data.status === "success") {
                                status.innerText = "🎉 生成完毕";
                                timeInfo.innerText = `模型计算耗时: ${data.elapsed_time}s`;
                                timeInfo.style.display = "block";
                                img.src = data.image_url + "?t=" + new Date().getTime(); 
                                img.style.display = "block";
                                progressContainer.style.display = "none";
                            }
                        }
                    }
                } catch (e) { status.innerText = "❌ 出错啦"; }
                finally { btn.disabled = false; }
            }
        </script>
    </body>
    </html>
# ====3个 '''

@app.route('/api/image/<filename>')
def get_image(filename):
    url = f"http://{COMFY_ADDR}/view?filename={urllib.parse.quote(filename)}"
    try:
        with urllib.request.urlopen(url) as response:
            return Response(response.read(), mimetype='image/png')
    except: return "Image Not Found", 404

# ================= 后端逻辑优化 =================
@app.route('/generate', methods=['POST'])
def generate():
    # 【修复核心】在主函数中预先提取所有 request 数据
    mode = request.form.get("mode", "t2i")
    prompt = request.form.get("prompt", "")
    image_bytes = None
    image_filename = None

    if mode == 'i2i' and 'image' in request.files:
        file = request.files['image']
        image_bytes = file.read() # 将文件读入内存
        image_filename = file.filename

    def stream_generation(m, p, img_b, img_n):
        with gpu_lock:
            try:
                start_clock = time.time()
                # --- 工作流配置 ---
                if m == 'i2i':
                    comfy_name = upload_image_to_comfyui(img_b, img_n)
                    with open("FireRed-Image-Edit.json", "r", encoding="utf-8") as f:
                        wf = json.load(f)
                    wf["41"]["inputs"]["image"] = comfy_name
                    wf["68"]["inputs"]["prompt"] = p
                    wf["65"]["inputs"]["seed"] = random.randint(10**10, 10**15)
                    node_id = "9"
                else:
                    with open("Z-Image-GGUF.json", "r", encoding="utf-8") as f:
                        wf = json.load(f)
                    wf["8"]["inputs"]["text"] = p
                    wf["10"]["inputs"]["seed"] = random.randint(10**10, 10**15)
                    node_id = "18"

                # --- 任务推送 ---
                ws = websocket.WebSocket()
                ws.connect(f"ws://{COMFY_ADDR}/ws?clientId={CLIENT_ID}")
                prompt_id = queue_prompt(wf)['prompt_id']

                while True:
                    out = ws.recv()
                    if isinstance(out, str):
                        msg = json.loads(out)
                        if msg['type'] == 'progress':
                            yield json.dumps({"status": "progress", "value": msg['data']['value'], "max": msg['data']['max']}) + "\n"
                        elif msg['type'] == 'executing' and msg['data']['node'] is None and msg['data']['prompt_id'] == prompt_id:
                            break
                ws.close()

                # --- 耗时计算与结果 ---
                elapsed = round(time.time() - start_clock, 2)
                with urllib.request.urlopen(f"http://{COMFY_ADDR}/history/{prompt_id}") as r:
                    hist = json.loads(r.read())
                
                fname = hist[prompt_id]['outputs'][node_id]['images'][0]['filename']
                yield json.dumps({"status": "success", "image_url": f"/api/image/{fname}", "elapsed_time": elapsed}) + "\n"
                
            except Exception as e:
                yield json.dumps({"status": "error", "message": str(e)}) + "\n"

    # 将提取出的数据传给生成器
    return Response(stream_generation(mode, prompt, image_bytes, image_filename), mimetype='application/x-ndjson')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

