<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>背单词APP生成器 - 随机满分版</title>
    <script src="https://cdn.sheetjs.com/xlsx-0.19.3/package/dist/xlsx.full.min.js"></script>
    <style>
        body { font-family: "Segoe UI", Roboto, Helvetica, Arial, sans-serif; padding: 20px; background: #f0f2f5; max-width: 1000px; margin: 0 auto; color: #333; }
        h1 { color: #2E7D32; border-bottom: 3px solid #2E7D32; padding-bottom: 10px; display: inline-block; }
        .container { background: white; padding: 30px; border-radius: 16px; box-shadow: 0 4px 20px rgba(0,0,0,0.08); }
        
        .step { margin-bottom: 30px; padding: 20px; background: #f8f9fa; border-radius: 10px; border-left: 5px solid #ccc; }
        .step.active { border-left-color: #2E7D32; background: #fff; border: 1px solid #e0e0e0; border-left: 5px solid #2E7D32; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
        .step h2 { margin-top: 0; font-size: 18px; color: #444; }

        .btn { background: #2E7D32; color: white; border: none; padding: 12px 24px; border-radius: 8px; cursor: pointer; font-size: 16px; font-weight: bold; transition: 0.2s; display: inline-flex; align-items: center; gap: 8px; }
        .btn:hover { background: #1b5e20; transform: translateY(-1px); }
        .btn:active { transform: scale(0.98); }
        .btn-del { background: #ff5252; padding: 6px 12px; font-size: 12px; border-radius: 4px; }
        .btn-del:hover { background: #d32f2f; }

        input[type="file"] { margin-top: 10px; }
        
        .preview-box { max-height: 300px; overflow-y: auto; margin-top: 15px; border: 1px solid #eee; border-radius: 8px; }
        table { width: 100%; border-collapse: collapse; font-size: 13px; }
        th, td { padding: 10px; text-align: left; border-bottom: 1px solid #eee; }
        th { background-color: #f1f1f1; position: sticky; top: 0; font-weight: 600; }
        tr:hover { background-color: #f9f9f9; }
        td[contenteditable="true"]:focus { background: #fffdeb; outline: 2px solid #ffd54f; }

        .note { color: #666; font-size: 14px; margin-bottom: 10px; background: #e8f5e9; padding: 10px; border-radius: 6px; color: #1b5e20; }
    </style>
</head>
<body>

    <div class="container">
        <h1>🛠️ 单词本制作工具 <small style="font-size:14px; color:#888; font-weight:normal;">(随机乱序 + 满分100)</small></h1>
        
        <div class="step active">
            <h2>1. 上传 Excel 数据</h2>
            <div class="note">
                💡 <strong>Excel 表头要求：</strong><br>
                单词, 音标, 词根, 词性, 中文释义, 阿语释义, 复数, 阴性, 现在式, 词源, 搭配阿, 搭配中, 例句阿, 例句中
            </div>
            <input type="file" id="upload" accept=".xlsx, .xls, .csv" />
        </div>

        <div class="step" id="previewStep" style="display:none;">
            <h2>2. 数据预览 (可直接修改)</h2>
            <div class="preview-box">
                <table id="dataTable">
                    <thead>
                        <tr>
                            <th width="60">操作</th>
                            <th>单词</th>
                            <th>音标</th>
                            <th>词性</th>
                            <th>中文释义</th>
                            <th>阿语释义</th>
                        </tr>
                    </thead>
                    <tbody></tbody>
                </table>
            </div>
        </div>

        <div class="step" id="downloadStep" style="display:none;">
            <h2>3. 生成 App</h2>
            <div style="display:flex; gap:10px; align-items: center; background: #f1f8e9; padding: 15px; border-radius: 8px;">
                <span>文件名:</span>
                <input type="text" id="filename" value="我的阿拉伯语单词本.html" style="padding:10px; border:1px solid #ccc; border-radius:6px; width:250px;">
                <button class="btn" onclick="generateApp()">🚀 立即生成下载</button>
            </div>
        </div>
    </div>

    <script>
        let globalData = [];

        // 字段映射配置
        const fieldMap = {
            "单词": "word", "音标": "ipa", "词根": "root", "词性": "pos",
            "中文释义": "def_cn", "阿语释义": "def_ar", "复数": "plural",
            "阴性": "fem", "现在式": "present", "词源": "source",
            "搭配阿": "collo_ar", "搭配中": "collo_cn", "例句阿": "sent_ar", "例句中": "sent_cn"
        };

        document.getElementById('upload').addEventListener('change', handleFile);

        function handleFile(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
                const rawData = XLSX.utils.sheet_to_json(firstSheet);
                processData(rawData);
            };
            reader.readAsArrayBuffer(file);
        }

        function processData(rawData) {
            globalData = rawData.map((row, index) => {
                let newRow = { id: index + 1 }; // 这里的ID仅作参考，实际逻辑会重置
                for (let cnKey in fieldMap) newRow[fieldMap[cnKey]] = row[cnKey] || "";
                // 兼容英文表头
                for (let key in row) if (Object.values(fieldMap).includes(key)) newRow[key] = row[key];
                return newRow;
            });
            renderTable();
            document.getElementById('previewStep').style.display = 'block';
            document.getElementById('downloadStep').style.display = 'block';
            document.querySelectorAll('.step')[1].classList.add('active');
            document.querySelectorAll('.step')[2].classList.add('active');
        }

        function renderTable() {
            const tbody = document.querySelector('#dataTable tbody');
            tbody.innerHTML = '';
            globalData.forEach((item, index) => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td><button class="btn btn-del" onclick="deleteRow(${index})">删</button></td>
                    <td contenteditable="true" onblur="updateData(${index}, 'word', this.innerText)">${item.word}</td>
                    <td contenteditable="true" onblur="updateData(${index}, 'ipa', this.innerText)">${item.ipa}</td>
                    <td contenteditable="true" onblur="updateData(${index}, 'pos', this.innerText)">${item.pos}</td>
                    <td contenteditable="true" onblur="updateData(${index}, 'def_cn', this.innerText)">${item.def_cn}</td>
                    <td contenteditable="true" onblur="updateData(${index}, 'def_ar', this.innerText)">${item.def_ar}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function updateData(index, key, value) { globalData[index][key] = value; }
        function deleteRow(index) { if(confirm('确定删除?')) { globalData.splice(index, 1); renderTable(); } }

        // === 核心：生成 APP 代码 ===
        function generateApp() {
            // 模版字符串
            const appTemplate = `<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>阿拉伯语智能背词</title>
<link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Noto+Sans+SC:wght@400;500;700&display=swap" rel="stylesheet">
<style>
:root{--primary:#4CAF50;--danger:#F44336;--bg:#F2F3F5;--card-bg:#FFF;}
*{box-sizing:border-box;user-select:none;-webkit-tap-highlight-color:transparent;}
body{font-family:'Noto Sans SC',sans-serif;background:var(--bg);margin:0;display:flex;flex-direction:column;align-items:center;height:100vh;overflow:hidden;}
.header-bar{width:100%;padding:15px 20px;display:flex;justify-content:space-between;color:#666;font-size:14px;font-weight:bold;background:var(--bg);z-index:10;}
.main-scroll-area{flex:1;width:100%;max-width:500px;overflow-y:auto;padding:0 20px 100px 20px;display:flex;flex-direction:column;align-items:center;}
.card-inner{width:100%;background:var(--card-bg);border-radius:20px;box-shadow:0 4px 20px rgba(0,0,0,0.05);overflow:hidden;margin-bottom:20px;}
.word-header{display:flex;flex-direction:column;align-items:center;padding:25px 20px 15px;background:linear-gradient(to bottom,#fff,#fafafa);}
.pos-badge{font-size:12px;color:#2E7D32;background:#E8F5E9;padding:4px 12px;border-radius:12px;font-weight:700;margin-bottom:15px;}
.ar-word{font-family:'Amiri',serif;font-size:52px;font-weight:700;color:#000;text-align:center;line-height:1.4;}
.ipa{font-family:monospace;color:#666;font-size:16px;margin:5px 0 10px;}
.audio-icon{width:40px;height:40px;border-radius:50%;background:#f5f5f5;color:var(--primary);display:flex;align-items:center;justify-content:center;font-size:20px;cursor:pointer;}
.tags-container{display:flex;flex-wrap:wrap;justify-content:center;gap:8px;padding:10px 20px 20px;border-bottom:1px dashed #eee;}
.tag{font-size:12px;padding:4px 10px;border-radius:8px;background:#f7f7f7;color:#555;display:flex;align-items:center;}
.tag-val{font-family:'Amiri',serif;font-weight:bold;color:#333;margin-left:4px;}
.options-area{padding:20px;display:flex;flex-direction:column;gap:12px;}
.option-btn{width:100%;padding:18px 20px;background:#fff;border:2px solid #f0f0f0;border-radius:16px;font-size:17px;text-align:left;cursor:pointer;transition:0.2s;font-weight:500;}
.option-btn.correct{background:var(--primary);color:#fff;border-color:var(--primary);}
.option-btn.wrong{background:var(--danger);color:#fff;border-color:var(--danger);animation:shake 0.4s;}
.fixed-footer{position:fixed;bottom:0;left:0;width:100%;padding:15px 20px 25px;background:rgba(255,255,255,0.95);backdrop-filter:blur(10px);border-top:1px solid #eee;display:flex;justify-content:center;z-index:100;}
.footer-inner{width:100%;max-width:500px;display:flex;gap:15px;}
.action-btn{flex:1;padding:14px;border:none;border-radius:14px;font-size:16px;font-weight:bold;cursor:pointer;box-shadow:0 4px 10px rgba(0,0,0,0.05);}
.btn-show{background:#fff;border:2px solid #eee;color:#555;}
.btn-next{background:var(--primary);color:#fff;}
.modal-overlay{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.6);z-index:998;opacity:0;pointer-events:none;transition:opacity 0.3s;}
.modal-overlay.show{opacity:1;pointer-events:auto;}
.bottom-sheet{position:fixed;bottom:-100%;left:0;width:100%;background:#fff;border-radius:24px 24px 0 0;z-index:999;transition:bottom 0.4s cubic-bezier(0.19,1,0.22,1);display:flex;flex-direction:column;max-height:85vh;box-shadow:0 -10px 40px rgba(0,0,0,0.2);}
.bottom-sheet.show{bottom:0;}
.sheet-content{padding:20px;overflow-y:auto;flex:1;}
.sheet-footer{padding:15px 20px 25px;border-top:1px solid #eee;}
.detail-block{margin-bottom:20px;padding-bottom:15px;border-bottom:1px solid #f5f5f5;}
.detail-ar{font-family:'Amiri',serif;font-size:22px;direction:rtl;margin-bottom:5px;}
@keyframes shake{0%,100%{transform:translateX(0);}25%{transform:translateX(-5px);}75%{transform:translateX(5px);}}
</style>
</head>
<body>
<div class="header-bar"><span id="progress-text">1 / 10</span><span id="score-text">得分: 0</span></div>
<div class="main-scroll-area">
    <div class="card-inner">
        <div class="word-header">
            <div class="pos-badge" id="pos-display"></div>
            <div class="ar-word" id="word-display"></div>
            <div class="ipa" id="ipa-display"></div>
            <div class="audio-icon" onclick="speakCurrent()">🔊</div>
        </div>
        <div class="tags-container" id="tags-container"></div>
        <div class="options-area" id="options-container"></div>
    </div>
</div>
<div class="fixed-footer"><div class="footer-inner"><button class="action-btn btn-show" onclick="openSheet()">💡 偷看/详情</button><button class="action-btn btn-next" onclick="nextWord()">下一个 ➜</button></div></div>
<div class="modal-overlay" onclick="closeSheet()"></div>
<div class="bottom-sheet" id="detail-sheet">
    <div style="padding:15px;text-align:center;border-bottom:1px solid #eee;"><div style="width:40px;height:5px;background:#ddd;border-radius:5px;margin:0 auto 10px;"></div><div style="font-weight:bold;color:var(--primary);">单词详解</div></div>
    <div class="sheet-content">
        <div class="detail-block"><div style="font-size:12px;color:#999;">释义</div><div id="sheet-def-cn" style="font-size:18px;font-weight:bold;"></div><div id="sheet-def-ar" class="detail-ar" style="color:#666;font-size:18px;"></div></div>
        <div class="detail-block" id="sheet-collo-box"><div style="font-size:12px;color:#999;">搭配</div><div id="sheet-collo-ar" class="detail-ar"></div><div id="sheet-collo-cn" style="color:#444;"></div></div>
        <div class="detail-block" id="sheet-sent-box"><div style="font-size:12px;color:#999;">例句</div><div id="sheet-sent-ar" class="detail-ar"></div><div id="sheet-sent-cn" style="color:#444;"></div></div>
    </div>
    <div class="sheet-footer"><button class="action-btn btn-next" style="width:100%" onclick="nextWord()">懂了，下一个 ➜</button></div>
</div>
<script>
const vocabData = /*DATA_PLACEHOLDER*/;
let currentIndex = 0;
let score = 0;
let isAnswered = false;
let currentWordObj = null;
let pointsPerWord = 0;

// 洗牌算法
function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

document.addEventListener('DOMContentLoaded', () => {
    if (!vocabData || vocabData.length === 0) { alert('没有数据'); return; }
    // 1. 随机打乱单词顺序
    shuffleArray(vocabData);
    // 2. 计算单题分值 (总分100)
    pointsPerWord = 100 / vocabData.length;
    loadCard(currentIndex);
});

function loadCard(index) {
    closeSheet();
    setTimeout(() => {
        isAnswered = false;
        currentWordObj = vocabData[index];
        document.getElementById('progress-text').innerText = (index + 1) + " / " + vocabData.length;
        document.getElementById('pos-display').innerText = currentWordObj.pos;
        document.getElementById('word-display').innerText = currentWordObj.word;
        document.getElementById('ipa-display').innerText = currentWordObj.ipa;

        const tagsContainer = document.getElementById('tags-container');
        tagsContainer.innerHTML = '';
        const addTag = (l, v) => { if(v && v!=='-') tagsContainer.innerHTML += '<div class="tag"><span style="opacity:0.6;font-size:10px;margin-right:2px;">'+l+'</span><span class="tag-val">'+v+'</span></div>'; };
        addTag('根',currentWordObj.root); addTag('复',currentWordObj.plural); addTag('现',currentWordObj.present); addTag('阴',currentWordObj.fem); addTag('源',currentWordObj.source);

        generateOptions(currentWordObj);

        document.getElementById('sheet-def-cn').innerText = currentWordObj.def_cn;
        document.getElementById('sheet-def-ar').innerText = currentWordObj.def_ar || '';
        const setDetail = (idB, idA, idC, vA, vC) => {
            const box = document.getElementById(idB);
            if(!vA && !vC) box.style.display='none'; else { box.style.display='block'; document.getElementById(idA).innerText=vA||''; document.getElementById(idC).innerText=vC||''; }
        };
        setDetail('sheet-collo-box','sheet-collo-ar','sheet-collo-cn',currentWordObj.collo_ar,currentWordObj.collo_cn);
        setDetail('sheet-sent-box','sheet-sent-ar','sheet-sent-cn',currentWordObj.sent_ar,currentWordObj.sent_cn);
    }, 200);
}

function generateOptions(correctObj) {
    const container = document.getElementById('options-container');
    container.innerHTML = '';
    // 确保干扰项不包含当前词，且也是随机抽取的
    let distractors = vocabData.filter(item => item.id !== correctObj.id).map(item => item.def_cn);
    shuffleArray(distractors); 
    let options = distractors.slice(0, 3);
    options.push(correctObj.def_cn);
    shuffleArray(options);
    
    options.forEach(optText => {
        const btn = document.createElement('button');
        btn.className = 'option-btn';
        btn.innerText = optText;
        btn.onclick = () => checkAnswer(btn, optText === correctObj.def_cn);
        container.appendChild(btn);
    });
}

function checkAnswer(btn, isCorrect) {
    if (isAnswered) return;
    isAnswered = true;
    const allBtns = document.querySelectorAll('.option-btn');
    if (isCorrect) {
        btn.classList.add('correct');
        // 累加分数
        score += pointsPerWord;
        document.getElementById('score-text').innerText = "得分: " + Math.round(score);
        speakCurrent();
    } else {
        btn.classList.add('wrong');
        allBtns.forEach(b => { if(b.innerText===currentWordObj.def_cn) b.classList.add('correct'); });
        setTimeout(openSheet, 400);
    }
}

function nextWord() {
    if (currentIndex < vocabData.length - 1) { currentIndex++; loadCard(currentIndex); }
    else { alert("测试结束！最终得分: " + Math.round(score)); location.reload(); } // 结束后刷新重新开始
}

function speakCurrent() {
    if(!currentWordObj) return;
    const u = new SpeechSynthesisUtterance(currentWordObj.word);
    u.lang = 'ar-SA'; u.rate = 0.9;
    window.speechSynthesis.speak(u);
}

function openSheet(){document.querySelector('.modal-overlay').classList.add('show');document.getElementById('detail-sheet').classList.add('show');}
function closeSheet(){document.querySelector('.modal-overlay').classList.remove('show');document.getElementById('detail-sheet').classList.remove('show');}
<\/script></body></html>`;

            const jsonData = JSON.stringify(globalData);
            const finalHtml = appTemplate.replace('/*DATA_PLACEHOLDER*/', jsonData);
            
            const blob = new Blob([finalHtml], {type: "text/html"});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            const name = document.getElementById('filename').value || '单词本.html';
            a.href = url; a.download = name.endsWith('.html')?name:name+'.html';
            document.body.appendChild(a); a.click(); document.body.removeChild(a);
        }
    </script>
</body>
</html>
