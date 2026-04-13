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
import requests  # 必须 pip install requests
from flask import Flask, request, jsonify, Response

app = Flask(__name__)

# ================= 基础配置 =================
COMFY_ADDR = "127.0.0.1:8188"  # ComfyUI 的地址
CLIENT_ID = str(uuid.uuid4())
gpu_lock = threading.Lock()    # 显存保护锁

def queue_prompt(prompt_workflow):
    p = {"prompt": prompt_workflow, "client_id": CLIENT_ID}
    data = json.dumps(p).encode('utf-8')
    req = urllib.request.Request(f"http://{COMFY_ADDR}/prompt", data=data)
    return json.loads(urllib.request.urlopen(req).read())

def upload_image_to_comfyui(file_stream, filename):
    """将用户上传的图片转发给 ComfyUI 的后台"""
    url = f"http://{COMFY_ADDR}/upload/image"
    files = {'image': (filename, file_stream)}
    res = requests.post(url, files=files)
    return res.json()['name'] # 返回 ComfyUI 重命名后的文件名

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
            
            /* 选项卡样式 */
            .tabs { display: flex; margin-bottom: 20px; border-bottom: 2px solid #eee; }
            .tab { flex: 1; text-align: center; padding: 10px; cursor: pointer; font-weight: bold; color: #888; transition: 0.3s; }
            .tab.active { color: #007bff; border-bottom: 2px solid #007bff; }
            
            .input-group { margin-bottom: 15px; text-align: left; }
            .input-group label { display: block; margin-bottom: 5px; font-size: 14px; color: #555; }
            input[type="text"], input[type="file"] { width: 100%; padding: 12px; font-size: 15px; border: 1px solid #ccc; border-radius: 8px; box-sizing: border-box; }
            
            button { width: 100%; padding: 14px; font-size: 16px; background-color: #007bff; color: white; border: none; border-radius: 8px; cursor: pointer; transition: 0.3s; font-weight: bold; }
            button:hover { background-color: #0056b3; }
            button:disabled { background-color: #aaa; cursor: not-allowed; }
            
            #status { margin: 15px 0; font-weight: bold; color: #555; text-align: center; font-size: 14px;}
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
                    <input type="text" id="prompt" placeholder="例如：a beautiful cyberpunk city..." value="a beautiful girl, cinematic lighting, 8k">
                </div>

                <button type="button" id="genBtn" onclick="generate()">立即生成</button>
            </form>
            
            <p id="status"></p>
            <img id="resultImg" src="">
        </div>

        <script>
            let currentMode = 't2i';

            function switchMode(mode) {
                currentMode = mode;
                document.getElementById('tab-t2i').className = mode === 't2i' ? 'tab active' : 'tab';
                document.getElementById('tab-i2i').className = mode === 'i2i' ? 'tab active' : 'tab';
                
                document.getElementById('file-upload-group').style.display = mode === 'i2i' ? 'block' : 'none';
                
                if(mode === 'i2i') {
                    document.getElementById('prompt-label').innerText = '修改指令：';
                    document.getElementById('prompt').value = '猫换成狗';
                    document.getElementById('prompt').placeholder = '描述你要怎么改...';
                } else {
                    document.getElementById('prompt-label').innerText = '画面描述：';
                    document.getElementById('prompt').value = 'a beautiful girl, cinematic lighting, 8k';
                }
            }

            async function generate() {
                const btn = document.getElementById('genBtn');
                const status = document.getElementById('status');
                const img = document.getElementById('resultImg');
                const prompt = document.getElementById('prompt').value;
                const fileInput = document.getElementById('uploadImage');
                
                if(!prompt) { alert("请输入提示词！"); return; }
                if(currentMode === 'i2i' && fileInput.files.length === 0) {
                    alert("请上传一张参考图！"); return;
                }

                btn.disabled = true;
                btn.innerText = "生成中...";
                status.innerText = "🚀 正在连接 AI 服务器排队...";
                img.style.display = "none";

                // 使用 FormData 发送文本和文件
                const formData = new FormData();
                formData.append('mode', currentMode);
                formData.append('prompt', prompt);
                if(currentMode === 'i2i') {
                    formData.append('image', fileInput.files[0]);
                }

                try {
                    const response = await fetch('/generate', {
                        method: 'POST',
                        body: formData // 注意：使用 FormData 不要手动设置 Content-Type 头
                    });
                    const data = await response.json();
                    
                    if(data.status === "success") {
                        status.innerText = "🎉 生成成功！";
                        // 直接使用 Flask 代理返回的相对路径
                        img.src = data.image_url + "?t=" + new Date().getTime(); 
                        img.style.display = "block";
                    } else {
                        status.innerText = "❌ 生成失败: " + (data.message || "未知错误");
                    }
                } catch (error) {
                    status.innerText = "❌ 网络请求错误，请确认服务已启动。";
                } finally {
                    btn.disabled = false;
                    btn.innerText = "立即生成";
                }
            }
        </script>
    </body>
    </html>
    '''

# ================= 新增：图片代理路由 =================
# 解决手机端无法访问 127.0.0.1 的问题
@app.route('/api/image/<filename>')
def get_image(filename):
    url = f"http://{COMFY_ADDR}/view?filename={urllib.parse.quote(filename)}"
    req = urllib.request.Request(url)
    try:
        with urllib.request.urlopen(req) as response:
            return Response(response.read(), mimetype='image/png')
    except Exception as e:
        return str(e), 404

# ================= 后端 API 路由 =================
@app.route('/generate', methods=['POST'])
def generate():
    # 接收 FormData 数据
    mode = request.form.get("mode", "t2i")
    user_prompt = request.form.get("prompt", "")
    
    with gpu_lock:
        try:
            if mode == 'i2i':
                # --- 图生图逻辑 ---
                if 'image' not in request.files:
                    return jsonify({"status": "error", "message": "未收到上传的图片"}), 400
                
                # 1. 将图片转存到 ComfyUI
                upload_file = request.files['image']
                comfy_filename = upload_image_to_comfyui(upload_file.stream, upload_file.filename)

                # 2. 读取图改图 JSON
                with open("FireRed-Image-Edit.json", "r", encoding="utf-8") as f:
                    workflow = json.load(f)

                # 3. 根据你的 FireRed-Image-Edit.json 精准修改参数
                workflow["41"]["inputs"]["image"] = comfy_filename # 节点 41：加载图像
                workflow["68"]["inputs"]["prompt"] = user_prompt     # 节点 68：正向修改指令
                workflow["65"]["inputs"]["seed"] = random.randint(10**10, 10**15) # 节点 65：KSampler 种子
                save_node_id = "9" # 你的图改图 JSON 最终保存节点是 9

            else:
                # --- 文生图逻辑 ---
                with open("Z-Image-GGUF.json", "r", encoding="utf-8") as f:
                    workflow = json.load(f)

                workflow["8"]["inputs"]["text"] = user_prompt
                workflow["10"]["inputs"]["seed"] = random.randint(10**10, 10**15)
                save_node_id = "18" # 文生图 JSON 最终保存节点是 18

            # --- 公共执行逻辑 ---
            ws = websocket.WebSocket()
            ws.connect(f"ws://{COMFY_ADDR}/ws?clientId={CLIENT_ID}")
            
            prompt_res = queue_prompt(workflow)
            prompt_id = prompt_res['prompt_id']

            while True:
                out = ws.recv()
                if isinstance(out, str):
                    message = json.loads(out)
                    if message['type'] == 'executing':
                        data = message['data']
                        if data['node'] is None and data['prompt_id'] == prompt_id:
                            break
                else:
                    continue
            ws.close()

            # 获取最终图片文件名
            with urllib.request.urlopen(f"http://{COMFY_ADDR}/history/{prompt_id}") as r:
                history = json.loads(r.read())
            
            file_info = history[prompt_id]['outputs'][save_node_id]['images'][0]
            file_name = file_info['filename']

            # 关键修改：返回代理路由的相对路径！
            return jsonify({
                "status": "success",
                "image_url": f"/api/image/{file_name}"
            })
            
        except Exception as e:
            print(f"生成出错: {e}")
            return jsonify({"status": "error", "message": str(e)}), 500

if __name__ == '__main__':
    print("🚀 Flask 服务器已启动！请在浏览器打开: http://127.0.0.1:5000")
    app.run(host='0.0.0.0', port=5000)
```