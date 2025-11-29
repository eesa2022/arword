<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>Arabic Hub</title>
<style>
:root { --primary: #4CAF50; --bg: #F2F3F5; --card: #FFF; --danger: #F44336; --accent: #2196F3; }
* { box-sizing: border-box; -webkit-tap-highlight-color: transparent; user-select: none; }
body { font-family: -apple-system, sans-serif; background: var(--bg); margin: 0; height: 100vh; overflow: hidden; color: #333; }

/* === 视图切换系统 === */
.view { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; background: var(--bg); }
.view.active { display: flex; z-index: 10; }

/* === 首页 (Home) === */
.home-banner { padding: 40px 20px 20px; background: #fff; border-bottom: 1px solid #eee; margin-bottom: 20px; }
.home-title { font-size: 28px; font-weight: 800; color: var(--primary); margin: 0; }
.scroll-content { flex: 1; overflow-y: auto; padding: 20px; }

/* 课程选择卡片 */
.course-card {
    background: #fff; padding: 20px; border-radius: 16px; box-shadow: 0 4px 15px rgba(0,0,0,0.05);
    margin-bottom: 30px; border: 2px solid transparent; cursor: pointer; display: flex; justify-content: space-between; align-items: center;
}
.course-card.empty { border-color: var(--accent); background: #E3F2FD; }
.course-info h3 { margin: 0 0 5px 0; font-size: 16px; }
.course-info p { margin: 0; color: #888; font-size: 12px; }

/* 功能按钮网格 */
.grid-menu { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
.menu-btn {
    background: #fff; border-radius: 20px; padding: 25px 15px; text-align: center;
    box-shadow: 0 4px 15px rgba(0,0,0,0.05); transition: 0.2s; cursor: pointer;
    display: flex; flex-direction: column; align-items: center; justify-content: center; height: 150px;
}
.menu-btn:active { transform: scale(0.96); }
.menu-btn.disabled { opacity: 0.5; filter: grayscale(1); pointer-events: none; }
.icon { font-size: 32px; margin-bottom: 10px; }
.menu-title { font-weight: bold; font-size: 18px; margin-bottom: 5px; }
.menu-desc { font-size: 12px; color: #999; }

/* === 课程列表页 === */
.nav-bar { padding: 15px; background: #fff; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #eee; padding-top: max(15px, env(safe-area-inset-top)); }
.nav-title { font-weight: bold; font-size: 18px; }
.back-link { color: var(--accent); font-size: 16px; cursor: pointer; }
.book-item { background: #fff; padding: 15px; border-radius: 12px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; cursor: pointer; border-bottom: 1px solid #f0f0f0; }

/* === 速记模式 (Memo) === */
.memo-list { padding: 15px; }
.memo-row { background: #fff; border-radius: 12px; margin-bottom: 12px; display: flex; overflow: hidden; box-shadow: 0 2px 5px rgba(0,0,0,0.03); min-height: 100px; }
.memo-left { flex: 2; padding: 15px; border-right: 1px dashed #eee; display: flex; flex-direction: column; justify-content: center; position: relative; }
.memo-left:active { background: #f9f9f9; }
.memo-ar { font-family: "Traditional Arabic", serif; font-size: 26px; font-weight: bold; color: #000; margin-bottom: 5px; }
.memo-tags { display: flex; flex-wrap: wrap; gap: 4px; }
.mini-tag { font-size: 10px; background: #f0f2f5; color: #666; padding: 2px 5px; border-radius: 4px; }

.memo-right { flex: 1.2; background: #e0e0e0; display: flex; align-items: center; justify-content: center; text-align: center; cursor: pointer; padding: 10px; transition: 0.2s; position: relative; }
.memo-right .content { opacity: 0; font-size: 15px; font-weight: bold; transition: 0.2s; }
.memo-right.revealed { background: #fff; }
.memo-right.revealed .content { opacity: 1; }
.memo-hint { position: absolute; font-size: 20px; color: #aaa; }
.memo-right.revealed .memo-hint { opacity: 0; }

/* === 复习模式 (Quiz) - 沿用之前的样式 === */
.quiz-header-bar { width: 100%; padding: 10px 15px; display: flex; align-items: center; justify-content: space-between; background: var(--bg); padding-top: max(10px, env(safe-area-inset-top)); }
.card-inner { background: #fff; border-radius: 20px; box-shadow: 0 4px 20px rgba(0,0,0,0.05); overflow: hidden; margin-bottom: 20px; }
.word-header { padding: 20px; background: linear-gradient(to bottom, #fff, #fcfcfc); min-height: 180px; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; }
.ar-word-big { font-family: "Traditional Arabic", serif; font-size: 40px; font-weight: bold; margin: 10px 0; }
.audio-btn { width: 36px; height: 36px; border-radius: 50%; background: #f0f0f0; color: var(--primary); display: flex; justify-content: center; align-items: center; margin-top: 5px; }

/* 选项 */
.options-area { padding: 20px; display: flex; flex-direction: column; gap: 12px; }
.option-btn { padding: 16px; background: #fff; border: 2px solid #f0f0f0; border-radius: 16px; font-size: 16px; font-weight: 500; min-height: 60px; }
.option-btn.correct { background: var(--primary); color: #fff; border-color: var(--primary); }
.option-btn.wrong { background: var(--danger); color: #fff; border-color: var(--danger); }
.option-btn.ar-opt { font-family: "Traditional Arabic", serif; font-size: 22px; font-weight: bold; }

/* 连线 */
.match-container { padding: 15px; display: flex; justify-content: space-between; width: 100%; position: relative; min-height: 400px; }
.match-col { display: flex; flex-direction: column; gap: 15px; width: 45%; z-index: 2; }
.match-item { background: #fff; border: 2px solid #eee; border-radius: 12px; padding: 15px 5px; min-height: 80px; display: flex; align-items: center; justify-content: center; text-align: center; font-size: 15px; font-weight: 500; }
.match-item.ar { font-family: "Traditional Arabic", serif; font-size: 22px; }
.match-item.selected { border-color: #2196F3; background: #E3F2FD; }
.match-item.matched { border-color: var(--primary); background: #E8F5E9; opacity: 0.6; pointer-events: none; }
.match-svg { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 1; }
line { stroke-width: 3; stroke-linecap: round; transition: all 0.5s; }

/* 填空 */
.cloze-input { width: 100%; padding: 15px; font-size: 20px; border: 2px solid #ddd; border-radius: 12px; text-align: center; direction: rtl; font-family: "Traditional Arabic", serif; margin-bottom: 10px; }
.sentence-cloze { font-family: "Traditional Arabic", serif; font-size: 24px; line-height: 2.0; text-align: center; direction: rtl; padding: 10px; }
.cloze-gap { display: inline-block; min-width: 60px; border-bottom: 2px solid #999; color: transparent; margin: 0 5px; }
.cloze-gap.revealed { color: var(--danger); font-weight: bold; border-bottom: none; }

/* 底部 */
.fixed-footer { position: fixed; bottom: 0; left: 0; width: 100%; padding: 15px 20px; padding-bottom: max(20px, env(safe-area-inset-bottom)); background: rgba(255,255,255,0.95); border-top: 1px solid #eee; display: flex; gap: 15px; z-index: 100; }
.action-btn { flex: 1; padding: 14px; border: none; border-radius: 14px; font-size: 16px; font-weight: bold; }
.btn-next { background: var(--primary); color: #fff; } .btn-show { background: #fff; border: 2px solid #eee; color: #555; }

/* 弹窗 */
.modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 998; opacity: 0; pointer-events: none; transition: 0.3s; }
.modal-overlay.show { opacity: 1; pointer-events: auto; }
.bottom-sheet { position: fixed; bottom: -100%; left: 0; width: 100%; background: #fff; border-radius: 24px 24px 0 0; z-index: 999; transition: bottom 0.4s cubic-bezier(0.19,1,0.22,1); display: flex; flex-direction: column; max-height: 80vh; }
.bottom-sheet.show { bottom: 0; }
.sheet-content { padding: 25px; overflow-y: auto; flex: 1; }
.detail-block { margin-bottom: 20px; }
.detail-ar { font-family: "Traditional Arabic", serif; font-size: 24px; direction: rtl; }
.morph-tag { display: inline-block; font-size: 13px; background: #E3F2FD; color: #1565C0; padding: 3px 8px; border-radius: 4px; margin-right: 5px; margin-bottom: 5px; }
</style>
</head>
<body>

<div id="view-home" class="view active">
    <div class="home-banner">
        <div style="font-size:14px;color:#999;">Welcome Back</div>
        <h1 class="home-title">Arabic Hub</h1>
    </div>
    <div class="scroll-content">
        <div class="course-card empty" id="courseSelector" onclick="navTo('view-library')">
            <div class="course-info">
                <p>当前课程</p>
                <h3 id="currentCourseName">点击选择课程</h3>
            </div>
            <div style="font-size:20px; color:var(--accent);">➜</div>
        </div>

        <div class="grid-menu">
            <div class="menu-btn disabled" id="btnMemo" onclick="startMemo()">
                <div class="icon" style="color:var(--primary);">⚡️</div>
                <div class="menu-title">速记</div>
                <div class="menu-desc">列表快速过词</div>
            </div>
            <div class="menu-btn disabled" id="btnQuiz" onclick="startQuiz()">
                <div class="icon" style="color:var(--accent);">🧠</div>
                <div class="menu-title">复习</div>
                <div class="menu-desc">混合题型测验</div>
            </div>
        </div>
    </div>
</div>

<div id="view-library" class="view">
    <div class="nav-bar"><span class="back-link" onclick="navTo('view-home')">取消</span><span class="nav-title">选择课程</span><span style="width:30px;"></span></div>
    <div class="scroll-content" id="libraryList"></div>
</div>

<div id="view-memo" class="view">
    <div class="nav-bar"><span class="back-link" onclick="navTo('view-home')">← 结束</span><span class="nav-title">速记模式</span><span style="width:30px;"></span></div>
    <div class="scroll-content memo-list" id="memoContainer"></div>
</div>

<div id="view-quiz" class="view">
    <div class="quiz-header-bar">
        <span class="back-link" onclick="goHome()">← 退出</span>
        <span style="font-weight:bold;color:#666;"><span id="q-idx">1</span>/<span id="q-total">10</span></span>
        <span style="font-weight:bold;color:var(--accent);">Score: <span id="q-score">0</span></span>
    </div>
    <div class="scroll-content" style="display:flex; flex-direction:column;">
        <div class="card-inner" id="cardContainer">
            </div>
    </div>
    <div class="fixed-footer">
        <button class="action-btn btn-show" id="btnShow" onclick="openSheet()">💡 详情</button>
        <button class="action-btn btn-next" onclick="nextWord()">Next ➜</button>
    </div>
</div>

<div class="modal-overlay" onclick="closeSheet()"></div>
<div class="bottom-sheet" id="detail-sheet">
    <div style="padding:15px;text-align:center;border-bottom:1px solid #eee;"><div style="width:40px;height:5px;background:#ddd;border-radius:5px;margin:0 auto 10px;"></div><div style="font-weight:bold;color:var(--primary);">单词详解</div></div>
    <div class="sheet-content">
        <div style="text-align:center;margin-bottom:20px;"><div id="sheet-word" class="detail-ar" style="font-size:36px;font-weight:bold;"></div><div id="sheet-ipa" style="color:#888;"></div></div>
        <div class="detail-block" id="sheet-morph-box" style="display:none;"><div style="font-size:12px;color:#999;margin-bottom:5px;">词法信息</div><div id="sheet-morph-content"></div></div>
        <div class="detail-block"><div style="font-size:12px;color:#999;">释义</div><div id="sheet-def-cn" style="font-size:18px;font-weight:bold;"></div><div id="sheet-def-ar" class="detail-ar" style="color:#666;font-size:18px;"></div></div>
        <div class="detail-block" id="sheet-collo-box"><div style="font-size:12px;color:#999;">搭配</div><div id="sheet-collo-ar" class="detail-ar"></div><div id="sheet-collo-cn" style="color:#444;"></div></div>
        <div class="detail-block" id="sheet-sent-box"><div style="font-size:12px;color:#999;">例句</div><div id="sheet-sent-ar" class="detail-ar"></div><div id="sheet-sent-cn" style="color:#444;"></div></div>
        <button class="action-btn btn-next" style="width:100%; margin-top:10px;" onclick="nextWord()">继续 ➜</button>
    </div>
</div>

<script>
const LIBRARY = [{"title":"新编第一册1-4课","data":[{"id":1,"word":"هِيَ","ipa":"hiya","root":"ه","pos":"名词","def_cn":"她","def_ar":"ضَمِيرٌ لِلْمُؤَنَّثِ","plural":"","fem":"","present":"","source":"","collo_ar":"هِيَ طَالِبَةٌ","collo_cn":"她是学生","sent_ar":"هِيَ تَتَكَلَّمُ الْعَرَبِيَّةَ.","sent_cn":"她说阿拉伯语。"},{"id":2,"word":"نَبِيل","ipa":"Nabīl","root":"نبل","pos":"名词","def_cn":"纳比勒（人名）/高尚的","def_ar":"شَرِيفٌ","plural":"نِبَال ونُبَلَاءُ ","fem":"  نَبِيلَة","present":"","source":"","collo_ar":"أَخْلَاقٌ نَبِيلَةٌ","collo_cn":"高尚的品德","sent_ar":"نَبِيل صَدِيقِي.","sent_cn":"纳比勒是我的朋友。"},{"id":3,"word":"هِنْد","ipa":"Hind","root":"هند","pos":"名词","def_cn":"欣德（女名）","def_ar":"اِسْمُ عَلَمٍ مُؤَنَّثٍ","plural":" أَهْنُد وأَهْنَاد","fem":"","present":"","source":"","collo_ar":"السَّيِّدَةُ هِنْد","collo_cn":"欣德女士","sent_ar":"هِنْد تِلْمِيذَةٌ ذَكِيَّةٌ.","sent_cn":"欣德是个聪明的小学生。"},{"id":4,"word":"أَنَا","ipa":"’anā","root":"أنأ","pos":"名词","def_cn":"我","def_ar":"ضَمِيرُ الْمُتَكَلِّمِ","plural":"","fem":"","present":"","source":"","collo_ar":"أَنَا مِصْرِيٌّ","collo_cn":"我是埃及人","sent_ar":"أَنَا أُحِبُّ بَلَدِي.","sent_cn":"我爱我的国家。"},{"id":5,"word":"أَنْتِ","ipa":"’anti","root":"أنت","pos":"名词","def_cn":"你（阴性）","def_ar":"ضَمِيرُ الْمُخَاطَبَةِ","plural":"","fem":"","present":"","source":"","collo_ar":"أَنْتِ جَمِيلَةٌ","collo_cn":"你很漂亮","sent_ar":"هَلْ أَنْتِ طَالِبَةٌ؟","sent_cn":"你是女学生吗？"},{"id":6,"word":"أَهْلًا وسَهْلًا","ipa":"’ahlan wa-sahlan","root":"أهل","pos":"名词","def_cn":"欢迎","def_ar":"عِبَارَةُ تَرْحِيبٍ","plural":"","fem":"","present":"","source":"","collo_ar":"أَهْلًا وَسَهْلًا بِكُمْ","collo_cn":"欢迎大家","sent_ar":"أَهْلًا وَسَهْلًا يَا صَدِيقِي.","sent_cn":"欢迎你，我的朋友。"},{"id":7,"word":"هُوَ","ipa":"huwa","root":"هوه","pos":"名词","def_cn":"他","def_ar":"ضَمِيرٌ لِلْمُذَكَّرِ","plural":"","fem":"","present":"","source":"","collo_ar":"هُوَ مُهَنْدِسٌ","collo_cn":"他是工程师","sent_ar":"هُوَ يَعْمَلُ فِي الْمَكْتَبِ.","sent_cn":"他在办公室工作。"},{"id":8,"word":"أَمِين","ipa":"’amīn","root":"أمن","pos":"名词","def_cn":"忠实的，秘书/艾敏","def_ar":"مُخْلِصٌ","plural":"أُمَنَاءُ  ","fem":"أَمِينَةٌ","present":"","source":"","collo_ar":"أَمِينُ الْمَكْتَبَةِ","collo_cn":"图书管理员","sent_ar":"الْمُسْلِمُ صَادِقٌ وَأَمِينٌ.","sent_cn":"穆斯林是诚实守信的。"},{"id":9,"word":"أَنْتَ","ipa":"’anta","root":"أنت","pos":"名词","def_cn":"你（阳性）","def_ar":"ضَمِيرُ الْمُخَاطَبِ","plural":"","fem":" أَنْتِ","present":"","source":"","collo_ar":"أَنْتَ رَجُلٌ طَيِّبٌ","collo_cn":"你是个好人","sent_ar":"مِنْ أَيْنَ أَنْتَ؟","sent_cn":"你从哪儿来？"},{"id":10,"word":"أَنِت","ipa":"anti","root":"أنت","pos":"名词","def_cn":"你（阴性）","def_ar":"ضَمِيرُ الْمُخَاطَبِ","plural":"","fem":"","present":"","source":"","collo_ar":"أَنْتِ جَمِيلَةٌ","collo_cn":"你是个美女","sent_ar":"مِنْ أَيْنَ أَنْتِ؟","sent_cn":"你从哪儿来？"},{"id":11,"word":"أَمِينَة","ipa":"’Amīnah","root":"أمن","pos":"名词","def_cn":"艾米娜（人名）","def_ar":"اِسْمُ عَلَمٍ مُؤَنَّثٍ","plural":"","fem":"","present":"","source":"","collo_ar":"الْأُخْتُ أَمِينَة","collo_cn":"艾米娜姐妹","sent_ar":"أَمِينَة تَدْرُسُ الطِّبَّ.","sent_cn":"艾米娜正在学医。"},{"id":12,"word":"مَنْ","ipa":"man","root":"من","pos":"名词","def_cn":"谁","def_ar":"اِسْمُ اسْتِفْهَامٍ","plural":"","fem":"","present":"","source":"","collo_ar":"مَنْ هَذَا؟","collo_cn":"这是谁？","sent_ar":"مَنْ أَنْتَ؟","sent_cn":"你是谁？"},{"id":13,"word":"و","ipa":"wa","root":"و","pos":"虚词","def_cn":"和，同","def_ar":"حَرْفُ عَطْفٍ","plural":"","fem":"","present":"","source":"","collo_ar":"أَنَا وَأَنْتَ","collo_cn":"我和你","sent_ar":"جَاءَ أَحْمَدُ وَمُحَمَّدٌ.","sent_cn":"艾哈迈德和穆罕默德来了。"},{"id":14,"word":"سَمِير","ipa":"Samīr","root":"سمر","pos":"名词","def_cn":"萨米尔（人名）","def_ar":"اِسْمُ عَلَمٍ مُذَكَّرٍ","plural":"سُمَرَاءُ","fem":"","present":"","source":"","collo_ar":"الصَّدِيقُ سَمِير","collo_cn":"朋友萨米尔","sent_ar":"ذَهَبَ سَمِير إِلَى الْعَمَلِ.","sent_cn":"萨米尔上班去了。"},{"id":15,"word":"أَهْلًا بِكَ","ipa":"’ahlan bik","root":"أهل","pos":"名词","def_cn":"你好（回答欢迎）","def_ar":"رَدٌّ عَلَى التَّرْحِيبِ","plural":"","fem":"","present":"","source":"","collo_ar":"أَهْلًا بِكَ يَا أَخِي","collo_cn":"你好啊兄弟","sent_ar":"أَهْلًا بِكَ فِي بَلَدِنَا.","sent_cn":"欢迎来到我们要国家。"},{"id":16,"word":"إِلَى اللِّقَاءِ","ipa":"’ilā l-liqā’","root":"لقى","pos":"名词","def_cn":"再见","def_ar":"عِبَارَةُ وَدَاعٍ","plural":"","fem":"","present":"","source":"","collo_ar":"إِلَى اللِّقَاءِ غَدًا","collo_cn":"明天见","sent_ar":"سَأُسَافِرُ الْآنَ، إِلَى اللِّقَاءِ.","sent_cn":"我现在要走了，再见。"},{"id":17,"word":"مُهَنْدِس","ipa":"muhandis","root":"هندس","pos":"名词","def_cn":"工程师","def_ar":"تِقْنِيٌّ","plural":"مُهَنْدِسُونَ","fem":"مُهَنْدِسَةٌ","present":"","source":"","collo_ar":"مُهَنْدِسٌ مِعْمَارِيٌّ","collo_cn":"建筑师","sent_ar":"أَبِي يَعْمَلُ مُهَنْدِسًا.","sent_cn":"我父亲当工程师。"},{"id":18,"word":"طَبِيب","ipa":"ṭabīb","root":"طبب","pos":"名词","def_cn":"医生","def_ar":"دُكْتُورٌ، مُعَالِجٌ","plural":"أَطِبَّاءُ وأَطِبَّة","fem":"طَبِيبَة","present":"","source":"","collo_ar":"طَبِيبُ الْأَسْنَانِ","collo_cn":"牙医","sent_ar":"ذَهَبْتُ إِلَى الطَّبِيبِ.","sent_cn":"我去看了医生。"},{"id":19,"word":"لا","ipa":"lā","root":"لا","pos":"虚词","def_cn":"不","def_ar":"حَرْفُ نَفْيٍ","plural":"","fem":"","present":"","source":"","collo_ar":"لَا أَعْرِفُ","collo_cn":"我不知道","sent_ar":"هَلْ أَنْتَ مِصْرِيٌّ؟ لَا.","sent_cn":"你是埃及人吗？不。"},{"id":20,"word":"أَيْضًا","ipa":"’ayḍan","root":"أيض","pos":"名词","def_cn":"也","def_ar":"كَذَلِكَ","plural":"","fem":"","present":"","source":"","collo_ar":"أَنَا أَيْضًا","collo_cn":"我也是","sent_ar":"هُوَ طَالِبٌ وَأَنَا طَالِبٌ أَيْضًا.","sent_cn":"他是学生，我也是学生。"},{"id":21,"word":"نَعَمْ","ipa":"na‘am","root":"نعم","pos":"名词","def_cn":"是的","def_ar":"حَرْفُ إِيجَابٍ","plural":"","fem":"","present":"","source":"","collo_ar":"نَعَمْ، أَنَا هُنَا","collo_cn":"是的，我在这儿","sent_ar":"هَلْ فَهِمْتَ؟ نَعَمْ.","sent_cn":"明白了吗？是的。"},{"id":22,"word":"مُمَرِّض","ipa":"mumarriḍ","root":"مرض","pos":"名词","def_cn":"男护士","def_ar":"مُسَاعِدُ الطَّبِيبِ","plural":"مَرِّضُون","fem":"مُمَرِّضَة","present":"","source":"","collo_ar":"مُمَرِّضٌ مَاهِرٌ","collo_cn":"熟练的护士","sent_ar":"يُسَاعِدُ الْمُمَرِّضُ الْمَرْضَى.","sent_cn":"护士帮助病人。"},{"id":23,"word":"مُدَرِّس","ipa":"mudarris","root":"درس","pos":"名词","def_cn":"教师","def_ar":"مُعَلِّمٌ","plural":"مُدَرِّسُون ","fem":" مُدَرِّسَةٌ","present":"","source":"","collo_ar":"مُدَرِّسُ اللُّغَةِ","collo_cn":"语言教师","sent_ar":"الْمُدَرِّسُ يَشْرَحُ الدَّرْسَ.","sent_cn":"老师正在讲解课文。"},{"id":24,"word":"لَا شُكْرًا عَلَى وَاجِبٍ","ipa":"lā shukran ‘alā wājib","root":"شكر","pos":"名词","def_cn":"不用谢","def_ar":"عَفْوًا","plural":"","fem":"","present":"","source":"","collo_ar":"لَا شُكْرًا عَلَى وَاجِبٍ","collo_cn":"（这是我应该做的）不用谢","sent_ar":"شُكْرًا لَكَ. لَا شُكْرًا عَلَى وَاجِبٍ.","sent_cn":"谢谢你。不用谢。"}]},{"title":"新编第一册5-6课","data":[{"id":1,"word":"هَلْ","ipa":"hal","root":"هل","pos":"名词","def_cn":"吗？（疑问词）","def_ar":"حَرْفُ اسْتِفْهَامٍ","plural":"","fem":"","present":"","source":"","collo_ar":"هَلْ أَنْتَ بِخَيْرٍ؟","collo_cn":"你好吗？","sent_ar":"هَلْ ذَهَبْتَ إِلَى السُّوقِ؟","sent_cn":"你去市场了吗？"},{"id":2,"word":"شُكْرًا","ipa":"shukran","root":"شكر","pos":"名词","def_cn":"谢谢","def_ar":"كَلِمَةُ امْتِنَانٍ","plural":"","fem":"","present":"","source":"","collo_ar":"شُكْرًا جَزِيلًا","collo_cn":"非常感谢","sent_ar":"شُكْرًا عَلَى مُسَاعَدَتِكَ.","sent_cn":"谢谢你的帮助。"},{"id":3,"word":"هَذِهِ","ipa":"hadhihi","root":"ه","pos":"名词","def_cn":"这（阴性）","def_ar":"اِسْمُ إِشَارَةٍ","plural":"","fem":"","present":"","source":"","collo_ar":"هَذِهِ بِنْتٌ","collo_cn":"这是一个女孩","sent_ar":"هَذِهِ مَدْرَسَتِي.","sent_cn":"这是我的学校。"},{"id":4,"word":"صِينِيّ","ipa":"ṣīniyy","root":"صين","pos":"名词","def_cn":"中国人，中国的","def_ar":"مَنْسُوبٌ لِلصِّينِ","plural":"","fem":"صِينِيَّةٌ","present":"","source":"","collo_ar":"طَعَامٌ صِينِيٌّ","collo_cn":"中国菜","sent_ar":"أَنَا صِينِيٌّ.","sent_cn":"我是中国人。"},{"id":5,"word":"هَذا","ipa":"hādhā","root":"هذى","pos":"名词","def_cn":"这（阳性）","def_ar":"اِسْمُ إِشَارَةٍ","plural":"","fem":" هَذِهِ","present":"","source":"","collo_ar":"هَذَا كِتَابٌ","collo_cn":"这是一本书","sent_ar":"هَذَا صَدِيقِي أَحْمَد.","sent_cn":"这是我的朋友艾哈迈德。"},{"id":6,"word":"سُعُوْدِيّ","ipa":"su‘ūdiyy","root":"سعد","pos":"名词","def_cn":"沙特人","def_ar":"مَنْسُوبٌ لِلسُّعُودِيَّةِ","plural":"","fem":"سُعُوْدِيَّةٌ","present":"","source":"","collo_ar":"رِيَالٌ سُعُودِيٌّ","collo_cn":"沙特里亚尔","sent_ar":"هُوَ طَالِبٌ سُعُودِيٌّ.","sent_cn":"他是沙特学生。"},{"id":7,"word":"كَذَلِكَ","ipa":"kadhālika","root":"كذى","pos":"名词","def_cn":"同样，也","def_ar":"أَيْضًا","plural":"","fem":"","present":"","source":"","collo_ar":"وَأَنَا كَذَلِكَ","collo_cn":"我也一样","sent_ar":"هُوَ ذَكِيٌّ وَأَخُوهُ كَذَلِكَ.","sent_cn":"他很聪明，他兄弟也一样。"},{"id":8,"word":"مَاجِد","ipa":"Mājid","root":"مجد","pos":"名词","def_cn":"马吉德（人名）","def_ar":"اِسْمُ عَلَمٍ مُذَكَّرٍ","plural":"مَوَاجِدُ","fem":"مَاجِدَة","present":"","source":"","collo_ar":"السَّيِّدُ مَاجِد","collo_cn":"马吉德先生","sent_ar":"مَاجِد يُحِبُّ الرِّيَاضَةَ.","sent_cn":"马吉德喜欢运动。"},{"id":9,"word":"سَمِيحَة","ipa":"Samīḥah","root":"سمح","pos":"名词","def_cn":"萨米哈（人名）","def_ar":"اِسْمُ عَلَمٍ مُؤَنَّثٍ","plural":"","fem":"","present":"","source":"","collo_ar":"الْآنِسَةُ سَمِيحَة","collo_cn":"萨米哈小姐","sent_ar":"سَمِيحَة تَتَعَلَّمُ الطَّبْخَ.","sent_cn":"萨米哈正在学做饭。"},{"id":10,"word":"مَعَ السَّلَامَةِ","ipa":"ma‘a s-salāmah","root":"سلم","pos":"名词","def_cn":"再见（保重）","def_ar":"وَدَاعًا","plural":"","fem":"","present":"","source":"","collo_ar":"اذْهَبْ مَعَ السَّلَامَةِ","collo_cn":"慢走/再见","sent_ar":"مَعَ السَّلَامَةِ، أَرَاكَ قَرِيبًا.","sent_cn":"再见，回头见。"},{"id":11,"word":"مَسْرُور","ipa":"masrūr","root":"سرر","pos":"名词","def_cn":"高兴的","def_ar":"فَرِحٌ، سَعِيدٌ","plural":"مَسْرُورون","fem":"مَسْرُورَةٌ","present":"","source":"","collo_ar":"أَنَا مَسْرُورٌ بِلِقَائِكَ","collo_cn":"很高兴见到你","sent_ar":"كَانَ مَسْرُورًا بِالنَّجَاحِ.","sent_cn":"他为成功感到高兴。"},{"id":12,"word":"وَعَلِيْكُمْ السَّلَامُ","ipa":"wa-‘alaykumu s-salām","root":"سلم","pos":"名词","def_cn":"你好（回敬语）","def_ar":"رَدُّ السَّلَامِ","plural":"","fem":"","present":"","source":"","collo_ar":"وَعَلَيْكُمُ السَّلَامُ وَرَحْمَةُ اللهِ","collo_cn":"愿主赐你平安和吉庆","sent_ar":"قُلْتُ لَهُ السَّلَامُ عَلَيْكُمْ، فَرَدَّ: وَعَلَيْكُمُ السَّلَامُ.","sent_cn":"我对他说祝你平安，他回答：也祝你平安。"},{"id":13,"word":"فُرْصَةٌ سَعِيدَةٌ","ipa":"furṣah sa‘īdah","root":"فرص","pos":"名词","def_cn":"幸会","def_ar":"تَشَرَّفْنَا","plural":"","fem":"","present":"","source":"","collo_ar":"كَانَتْ فُرْصَةً سَعِيدَةً","collo_cn":"真是幸会","sent_ar":"فُرْصَةٌ سَعِيدَةٌ أَنْ أَرَاكَ.","sent_cn":"很高兴见到你。"},{"id":14,"word":"نَجْوَى","ipa":"Najwā","root":"نجو","pos":"名词","def_cn":"纳尔瓦（人名）","def_ar":"اِسْمُ عَلَمٍ مُؤَنَّثٍ","plural":"نَجَاوَى","fem":"","present":"","source":"","collo_ar":"صَدِيقَتِي نَجْوَى","collo_cn":"我的朋友纳尔瓦","sent_ar":"نَجْوَى تُحِبُّ الشِّعْرَ.","sent_cn":"纳尔瓦喜欢诗歌。"},{"id":15,"word":"مِصْرِيّ","ipa":"miṣriyy","root":"مصر","pos":"名词","def_cn":"埃及人","def_ar":"مَنْسُوبٌ لِمِصْرَ","plural":" مَصَارٍ و مَصَارِىُّ","fem":"","present":"","source":"","collo_ar":"مُتْحَفٌ مِصْرِيٌّ","collo_cn":"埃及博物馆","sent_ar":"هَلْ أَنْتَ مِصْرِيٌّ؟","sent_cn":"你是埃及人吗？"},{"id":16,"word":"سُورِيّ","ipa":"sūriyy","root":"سور","pos":"名词","def_cn":"叙利亚人","def_ar":"مَنْسُوبٌ لِسُورِيَا","plural":"","fem":" سُورِيَّةٌ","present":"","source":"","collo_ar":"مَطْعَمٌ سُورِيٌّ","collo_cn":"叙利亚餐厅","sent_ar":"جَارِي سُورِيٌّ.","sent_cn":"我的邻居是叙利亚人。"},{"id":17,"word":"جِنْسِيَّة","ipa":"jinsiyyah","root":"جنس","pos":"名词","def_cn":"国籍","def_ar":"انْتِمَاءٌ لِدَوْلَةٍ","plural":"","fem":"","present":"","source":"","collo_ar":"مَا جِنْسِيَّتُكَ؟","collo_cn":"你的国籍是什么？","sent_ar":"جِنْسِيَّتِي صِينِيَّةٌ.","sent_cn":"我的国籍是中国。"},{"id":18,"word":"عَامِر","ipa":"‘Āmir","root":"عمر","pos":"名词","def_cn":"阿米尔（人名）/繁荣的","def_ar":"اِسْمُ عَلَمٍ مُذَكَّرٍ","plural":"عُمَّار و عَوَامِرُ","fem":" عَامِرَة","present":"","source":"","collo_ar":"بَيْتٌ عَامِرٌ","collo_cn":"充满生气的家","sent_ar":"عَامِر طَالِبٌ نَشِيطٌ.","sent_cn":"阿米尔是个积极的学生。"},{"id":19,"word":"لِقَاء","ipa":"liqā’","root":"لقي","pos":"名词","def_cn":"会面","def_ar":"اجْتِمَاعٌ","plural":"لِقَاءَات","fem":"","present":"","source":"","collo_ar":"لِقَاءٌ تِلِفِزْيُونِيّ","collo_cn":"电视访谈","sent_ar":"سَعِدْتُ بِلِقَائِكَ.","sent_cn":"很高兴见到你。"},{"id":20,"word":"ب","ipa":"bi","root":"ب","pos":"虚词","def_cn":"在...，用...（介词）","def_ar":"حَرْفُ جَرٍّ","plural":"","fem":"复数 \n阴性 \n现在式 \n索引 \n\nبِ\n===在，着随，以，用，思意等借凭，随伴，空时，因原示表，词介\n\n（ب باء)\n===号符的“ 乙”“ 次其“、“ 二第” 示表，2 字数表代；母字个二第的语伯拉阿\n\nب/حرف جرّ\n===义含种几列下有，词介\n+ب/استعانة\n===：助借示表\n++ذَهَبَ بالسَيَّارَة\n===往前车驱已他\n++بِمَ أَكْتُبُ\n===؟ 写笔么什用我\n++يَتَكلَّمُ باللُغةِ العربيّةِ\n===语阿讲他\n++اِتَّصِلْ بِه بِالتِلفُونِ\n===！话电打他给请，系联他跟话电用请\n++...بدُون ... بغير ...بلا\n===…无．缺，用没，用不 \n++أَكَلَ بِلا مِلْعَقةٍ\n===吃羹调用（不）没他\n++قَامَ الشيخُ بِغير مساعدة\n===来起了站己自，扶人没人老\n++يَئِيُّ بِدُونِ مَرَض\n===吟呻病无他\n+ب/إلصاق\n===连紧示表\n++أمْسَكْتُ بِيَدِه\n===手的他了住抓我\n+ب/مصاحبة\n===随伴示表\n++إذْهَبْ بِسَلامٍ\n===！吧去地安平你，安平路一你祝\n++وَصَلُوا إلى بَكِينَ بِسَلَامٍ\n===京到安平已们他\n+ب/ظرفيّة\n===语状作\n++سَهَرَ بِالليل\n===眠未夜彻他，了夜熬他\n++أقامَ بشنغهاىَ ثلاثةَ أيّامٍ\n===天三了住海上在他\n++بِسُرْعَةٍ\n===地快飞，地速迅\n++بِجِدٍّ واجْتِهَادٍ\n===地奋勤，地力努苦刻\n++بِوُصُوحٍ\n===地楚清\n++بِاخْتِصَارٍ\n===之总之言简\n++بِالجُمْلَةِ\n===发批[经] ；话句一\n++لَقَدْ نَصَرَكُمْ اللهُ بِبَدْرٍ وأَنْتُمْ أَذِلٌَةٌ\n===们你了助援已确主真而，的力势无是们你，役之尔德巴[伊]\n+ب/بدل\n===换交示表\n++بَاعَه بِثَمَنٍ بَخْسٍٍ\n===了卖它把价低以他\n++صَاعًا بِصاعٍ\n===报一还报一，牙还牙以之\n+ب/تعدية\n===物及或过通示表\n++ذَهَبْتُ بِهِ إلى البيتِ\n===里家到带他把我\n++وَصَلْتُ إلى دِمَشْقَ مَارًّا بِطَهْرَانَ\n===兰黑德（过经）道取曾我\n+ب/قسم\n===誓发示表\n++باللهِ لَأَفْعَلَنَّ\n===的做会定一我，誓起拉安凭\n+ب/سببيّة\n===因原示表\n++نَالَ بِالدَرْسِ النَجَاحَ\n===格及试考他， 习学持坚于由\n++أُخِذَ بِذَنْبِهِ\n===罚惩了受而罪犯因他\n++(...بِمَا أنّ...فقد...(فإنّ\n===…以所… 为因、于鉴、于由\n+ب/توكيد وهي زائدة\n===：面方几列下在用要主（的加附是الباء 的时这）调强示表\n++ب\n===前词述的句定否的كان\n++مَاكَانَ المُجْتَهِدُ بِخَائِبٍ\n===的望失会不是者奋勤\n+ب\n===前词述的لا和ما的用ليس当或ليس\n++لَيْسَ زَيدٌ بِقَائِمٍ\n===来起站有没德宰\n++مَا هُوَ بِذَكِيٍّ ومُجْتَهِدٍ\n===奋勤不又明聪不既他\n++ مَا كلُّ هاوٍ لِلجَمِيلِ بِفَاعِلٍ ولاَ كلُّ فعّالٍ له بِمُتَمِّمٍ\n===的底到做会都人人是不也人的劲起得干始开而，的做去会都不人的事好做欢喜个每\n+ب\n===前语主的句叹惊的型أَفْعِلْ\n++أَجْمِلْ بِهَذا المَنْظَرِ\n===！美多色景\n+ب\n===前语主的كفى在\n++كَفَى بِالحقِّ تَصِيرًا\n===（释解词分作نصر。语主作，位地格主在处بالحقّ)! 矣足已者持支为作理真\n++كَفَى بِاللهِ شهيدًا\n===了够已人证作主真\n+ب\n===之前 ذات 和نفس ٠عين 在\n++(أَلَّفَ ورَاجَعَ محمّد قاسم بِنفسِهِ (أو بعينه أو بذاته)\n===阅校和写编自亲姆塞卡• 穆罕默德\n+ب\n===前之语起作حسبك在\n++بِحَسْبِكَ درهمٌ\n===了足满你够已；了花你够已汗尔迪一\n+ب\n===后之 إذا الفجائية在\n++خَرَجْتُ فإذا بِزيدٍ في الطريقِ\n===上路在也德宰觉发然突，门出我\n+ب\n===前语状的式形定否是者配支的语状在\n++فَمَا رَجَعَتْ بِخَائِبَةٍ\n===归而望失有没她\n+ب/قد تأتى بمعنى عن وعلى وإلى \n===عن ,على و الى作可也时有\n","present":"","source":"","collo_ar":"بِالْقَلَمِ","collo_cn":"用笔","sent_ar":"كَتَبْتُ بِالْقَلَمِ.","sent_cn":"我用笔写了字。"},{"id":21,"word":"السَّلَامُ عَلَيْكُم","ipa":"as-salāmu ‘alaykum","root":"سلم","pos":"名词","def_cn":"祝你平安（你好）","def_ar":"تَحِيَّةُ الْإِسْلَامِ","plural":"","fem":"","present":"","source":"","collo_ar":"السَّلَامُ عَلَيْكُمْ وَرَحْمَةُ اللهِ","collo_cn":"祝你平安和真主的慈悯","sent_ar":"دَخَلَ وَقَالَ: السَّلَامُ عَلَيْكُمْ.","sent_cn":"他进来说：祝你们平安。"},{"id":22,"word":"ذَلِكَ","ipa":"dhālika","root":"ذلك","pos":"名词","def_cn":"那个","def_ar":"اِسْمُ إِشَارَةٍ للْبَعِيدِ","plural":"  أولئِكَ","fem":"تِلْكَ","present":"","source":"","collo_ar":"ذَلِكَ الْكِتَابُ","collo_cn":"那本书","sent_ar":"ذَلِكَ بَيْتٌ جَمِيلٌ.","sent_cn":"那是一所漂亮的房子。"},{"id":23,"word":"عَفْوًا","ipa":"‘afwan","root":"عفو","pos":"名词","def_cn":"对不起，没关系","def_ar":"طَلَبُ الْمَغْفِرَةِ","plural":"","fem":"","present":"","source":"","collo_ar":"عَفْوًا يَا سَيِّدِي","collo_cn":"对不起先生","sent_ar":"شُكْرًا. عَفْوًا.","sent_cn":"谢谢。没关系（不用谢）。"},{"id":24,"word":"صَحِيفَة","ipa":"ṣaḥīfah","root":"صحف","pos":"名词","def_cn":"报纸","def_ar":"جَرِيدَةٌ","plural":" صَحَائِفُ وصُحُف","fem":"","present":"","source":"","collo_ar":"قِرَاءَةُ الصَّحِيفَةِ","collo_cn":"读报","sent_ar":"اشْتَرَى أَبِي صَحِيفَةَ الْيَوْمِ.","sent_cn":"爸爸买了今天的报纸。"},{"id":25,"word":"لَكَ","ipa":"laka","root":"ل","pos":"名词","def_cn":"属于你，你有","def_ar":"يَمْلِكُهُ الْمُخَاطَبُ","plural":"","fem":"","present":"","source":"","collo_ar":"هَذَا لَكَ","collo_cn":"这是给你的","sent_ar":"لَكَ مُسْتَقْبَلٌ بَاهِرٌ.","sent_cn":"你有锦绣前程。"},{"id":26,"word":"طَيِّب","ipa":"ṭayyib","root":"طيب","pos":"名词","def_cn":"好的，善良的","def_ar":"جَيِّدٌ","plural":"طَيِّبُونَ ","fem":"    طَيِّبَاتٌ   طَيِّبَةٌ  ","present":"","source":"","collo_ar":"رَجُلٌ طَيِّبٌ","collo_cn":"善良的人","sent_ar":"هَذَا طَعَامٌ طَيِّبٌ.","sent_cn":"这是美味的食物。"},{"id":27,"word":"جَدِيد","ipa":"jadīd","root":"جدد","pos":"名词","def_cn":"新的","def_ar":"حَدِيثٌ","plural":" جُدُد و جُدَد","fem":"","present":"","source":"","collo_ar":"ثَوْبٌ جَدِيدٌ","collo_cn":"新衣服","sent_ar":"لَدَيْنَا مُدَرِّسٌ جَدِيدٌ.","sent_cn":"我们有一位新老师。"},{"id":28,"word":"دَفْتَر","ipa":"daftar","root":"دفتر","pos":"名词","def_cn":"笔记本","def_ar":"كُرَّاسَةٌ","plural":" دَفَاتِرُ","fem":"","present":"","source":"","collo_ar":"دَفْتَرُ الْمُلَاحَظَاتِ","collo_cn":"记事本","sent_ar":"اكْتُبِ الدَّرْسَ فِي الدَّفْتَرِ.","sent_cn":"把课文写在笔记本上。"},{"id":29,"word":"لِمَنْ","ipa":"li-man","root":"من","pos":"名词","def_cn":"是谁的","def_ar":"مِلْكُ مَنْ","plural":"","fem":"","present":"","source":"","collo_ar":"لِمَنْ هَذَا الْقَلَمُ؟","collo_cn":"这支笔是谁的？","sent_ar":"الْحُكْمُ لِمَنْ؟","sent_cn":"裁决权归谁？"},{"id":30,"word":"تِلْكَ","ipa":"tilka","root":"تلك","pos":"名词","def_cn":"那个（阴性）","def_ar":"اِسْمُ إِشَارَةٍ لِلْبَعِيدَةِ","plural":"أُولائكَ","fem":" تَانِّكَ","present":"","source":"","collo_ar":"تِلْكَ شَجَرَةٌ","collo_cn":"那是一棵树","sent_ar":"تِلْكَ الْمَرْأَةُ مُعَلِّمَتِي.","sent_cn":"那位女士是我的老师。"},{"id":31,"word":"صَوْرَة","ipa":"ṣūrah","root":"صور","pos":"名词","def_cn":"图片，照片","def_ar":"رَسْمٌ، لَقْطَةٌ","plural":"","fem":"","present":"","source":"","collo_ar":"صُورَةٌ جَمِيلَةٌ","collo_cn":"美丽的照片","sent_ar":"عَلَّقْتُ الصُّورَةَ عَلَى الْجِدَارِ.","sent_cn":"我把照片挂在墙上。"},{"id":32,"word":"مُفِيد","ipa":"mufīd","root":"فيد","pos":"名词","def_cn":"有用的","def_ar":"نَافِعٌ","plural":"","fem":" مُفِيدَةٌ","present":"","source":"","collo_ar":"كِتَابٌ مُفِيدٌ","collo_cn":"有益的书","sent_ar":"الرِّيَاضَةُ مُفِيدَةٌ لِلْجِسْمِ.","sent_cn":"运动对身体有益。"},{"id":33,"word":"غُرْفَة","ipa":"ghurfah","root":"غرف","pos":"名词","def_cn":"房间","def_ar":"حُجْرَةٌ","plural":"غُرَفٌ","fem":"","present":"","source":"","collo_ar":"غُرْفَةُ الْجُلُوسِ","collo_cn":"起居室","sent_ar":"غُرْفَتِي نَظِيفَةٌ وَمُرَتَّبَةٌ.","sent_cn":"我的房间干净整洁。"},{"id":34,"word":"جَمِيل","ipa":"jamīl","root":"جمل","pos":"名词","def_cn":"美丽的","def_ar":"حَسَنٌ، وَسِيمٌ","plural":"جُمَلاَءُ ","fem":" جَمِيلَة","present":"","source":"","collo_ar":"مَنْظَرٌ جَمِيلٌ","collo_cn":"美景","sent_ar":"الْجَوُّ جَمِيلٌ الْيَوْمَ.","sent_cn":"今天天气很好。"},{"id":35,"word":"تَافِه","ipa":"tāfih","root":"تفه","pos":"名词","def_cn":"琐碎的，无聊的","def_ar":"لَا قِيمَةَ لَهُ","plural":"","fem":" تَافِهَةٌ","present":"","source":"","collo_ar":"أَمْرٌ تَافِهٌ","collo_cn":"琐事","sent_ar":"لَا تَتَحَدَّثْ فِي أُمُورٍ تَافِهَةٍ.","sent_cn":"不要谈论无聊的事情。"},{"id":36,"word":"خَرِيطَة","ipa":"kharīṭah","root":"خرط","pos":"名词","def_cn":"地图","def_ar":"رَسْمٌ لِلْأَرْضِ","plural":" خَرَائِطُ","fem":"","present":"","source":"","collo_ar":"خَرِيطَةُ الْعَالَمِ","collo_cn":"世界地图","sent_ar":"نَظَرْتُ إِلَى الْخَرِيطَةِ.","sent_cn":"我看了地图。"},{"id":37,"word":"قَلَم","ipa":"qalam","root":"قلم","pos":"名词","def_cn":"笔","def_ar":"أَدَاةُ كِتَابَةٍ","plural":"أَقْلَام و قِلَام","fem":"","present":"","source":"","collo_ar":"قَلَمُ رَصَاصٍ","collo_cn":"铅笔","sent_ar":"أَكْتُبُ بِالْقَلَمِ.","sent_cn":"我用笔写字。"},{"id":38,"word":"كِتَاب","ipa":"kitāb","root":"كتب","pos":"名词","def_cn":"书","def_ar":"مُؤَلَّفٌ","plural":"كُتُبٌ","fem":"","present":"","source":"","collo_ar":"قِرَاءَةُ كِتَابٍ","collo_cn":"读书","sent_ar":"هَذَا كِتَابُ اللُّغَةِ الْعَرَبِيَّةِ.","sent_cn":"这是阿拉伯语书。"},{"id":39,"word":"زَمِيل","ipa":"zamīl","root":"زمل","pos":"名词","def_cn":"同事，同学","def_ar":"رَفِيقٌ","plural":"زُمَلَاءُ ","fem":"زَمِيلَةٌ  ","present":"","source":"","collo_ar":"زَمِيلُ الدِّرَاسَةِ","collo_cn":"同学","sent_ar":"زَمِيلِي فِي الْعَمَلِ نَشِيطٌ.","sent_cn":"我同事工作很积极。"}]}];
let currentBookIndex = -1, currentData = [], canSpeak = false;
if ('speechSynthesis' in window) canSpeak = true;

// 全局变量 (复习用)
let currentIndex = 0, score = 0, isAnswered = false, currentWordObj = null, currentMode = 'std', pointsPerWord = 0;
let matchPairs = [], selectedLeft = null, selectedRight = null, matchedCount = 0;

function navTo(viewId) { document.querySelectorAll('.view').forEach(el => el.classList.remove('active')); document.getElementById(viewId).classList.add('active'); }
function speakText(text) { if(!canSpeak) return; try { const u = new SpeechSynthesisUtterance(text); u.lang = 'ar-SA'; u.rate = 0.9; window.speechSynthesis.speak(u); } catch(e){} }
function shuffle(arr) { return arr.sort(() => Math.random() - 0.5); }
function stripTashkeel(str) { if(!str) return ""; return str.replace(/[\u064B-\u065F\u0670\u06D6-\u06ED]/g, ''); }
function createFuzzyRegex(word) { const bare = stripTashkeel(word); const pattern = bare.split('').join('[\\u064B-\\u065F\\u0670]*'); return new RegExp(pattern, 'g'); }

// === 初始化 ===
document.addEventListener('DOMContentLoaded', () => { renderLibrary(); });

// === 库管理 ===
function renderLibrary() {
    const list = document.getElementById('libraryList'); list.innerHTML = '';
    LIBRARY.forEach((book, idx) => {
        const div = document.createElement('div'); div.className = 'book-item';
        div.innerHTML = `<div><strong>${book.title}</strong> (${book.data.length}词)</div><div style="color:#ccc">➜</div>`;
        div.onclick = () => selectCourse(idx); list.appendChild(div);
    });
}
function selectCourse(idx) {
    currentBookIndex = idx; currentData = LIBRARY[idx].data;
    const selector = document.getElementById('courseSelector'); selector.classList.remove('empty'); selector.style.borderColor = 'var(--primary)';
    document.getElementById('currentCourseName').innerText = LIBRARY[idx].title;
    document.getElementById('btnMemo').classList.remove('disabled'); document.getElementById('btnQuiz').classList.remove('disabled');
    navTo('view-home');
}

// === 速记模式 ===
function startMemo() {
    const container = document.getElementById('memoContainer'); container.innerHTML = '';
    currentData.forEach(item => {
        const el = document.createElement('div'); el.className = 'memo-row';
        let tagsHtml = '';
        if(item.plural) tagsHtml += `<span class="mini-tag">复:${item.plural}</span>`;
        if(item.fem) tagsHtml += `<span class="mini-tag">阴:${item.fem}</span>`;
        if(item.present) tagsHtml += `<span class="mini-tag">现:${item.present}</span>`;
        if(item.source) tagsHtml += `<span class="mini-tag">源:${item.source}</span>`;
        el.innerHTML = `<div class="memo-left" onclick="speakText('${item.word}')"><div class="memo-ar">${item.word}</div><div class="memo-tags">${tagsHtml}</div><div style="position:absolute; right:10px; top:50%; transform:translateY(-50%); opacity:0.1;">🔊</div></div><div class="memo-right" onclick="this.classList.toggle('revealed')"><div class="memo-hint">👁️</div><div class="content">${item.def_cn}</div></div>`;
        container.appendChild(el);
    });
    navTo('view-memo');
}

// === 复习模式 ===
function startQuiz() {
    // 每次复习打乱
    let quizData = JSON.parse(JSON.stringify(currentData)); shuffle(quizData);
    currentData = quizData; // 更新为乱序版
    currentIndex = 0; score = 0; pointsPerWord = 100 / currentData.length;
    navTo('view-quiz');
    loadCard(0);
}
function goHome() { if(confirm("确定退出？进度不保存")) navTo('view-home'); }

function loadCard(index) {
    closeSheet();
    setTimeout(() => {
        isAnswered = false; currentWordObj = currentData[index];
        document.getElementById('q-idx').innerText = (index + 1); document.getElementById('q-total').innerText = currentData.length;
        document.getElementById('q-score').innerText = Math.round(score);
        document.getElementById('btnShow').style.display = 'block';

        const roll = Math.random(); const hasSent = !!currentWordObj.sent_ar;
        if(roll < 0.15) currentMode = 'match';
        else if(roll < 0.35) { if(hasSent) currentMode = 'cloze'; else currentMode = 'std'; }
        else if(roll < 0.60) currentMode = 'rev';
        else currentMode = 'std';

        if(currentMode === 'match') prepareMatchMode(); else renderStandardMode();
        preloadSheet();
    }, 200);
}

// 连线题渲染
function prepareMatchMode() {
    let others = currentData.filter(i => i.id !== currentWordObj.id); shuffle(others);
    let subset = [currentWordObj, ...others.slice(0, 3)];
    matchPairs = subset.map(item => ({ id: item.id, ar: item.word, cn: item.def_cn })); matchedCount = 0;
    const card = document.getElementById('cardContainer');
    let left = [...matchPairs]; shuffle(left); let right = [...matchPairs]; shuffle(right);
    card.innerHTML = `<div class="word-header" style="min-height:auto; padding:15px; background:#f9f9f9;"><div class="pos-badge" style="background:#E3F2FD;color:#1976D2;font-size:12px;padding:3px 8px;border-radius:4px;font-weight:bold;">连线匹配</div><div style="font-size:12px;color:#999;">点击左边，再点右边</div></div><div class="match-container" id="matchArea"><svg class="match-svg" id="matchSvg"></svg><div class="match-col">${left.map(i => `<div class="match-item ar" onclick="handleMatchClick('left', ${i.id}, this)">${i.ar}</div>`).join('')}</div><div class="match-col">${right.map(i => `<div class="match-item" onclick="handleMatchClick('right', ${i.id}, this)">${i.cn}</div>`).join('')}</div></div>`;
    selectedLeft = null; selectedRight = null; document.getElementById('btnShow').style.display = 'none';
}
// 连线逻辑
function handleMatchClick(side, id, el) {
    if (el.classList.contains('matched')) return;
    document.querySelectorAll(`.match-col:nth-child(${side==='left'?2:3}) .match-item`).forEach(d => d.classList.remove('selected'));
    el.classList.add('selected');
    if (side === 'left') selectedLeft = { id, el }; else selectedRight = { id, el };
    if (selectedLeft && selectedRight) {
        if (selectedLeft.id === selectedRight.id) {
            drawConnection(selectedLeft.el, selectedRight.el, true);
            selectedLeft.el.classList.add('matched'); selectedLeft.el.classList.remove('selected'); selectedRight.el.classList.add('matched'); selectedRight.el.classList.remove('selected');
            const item = matchPairs.find(p => p.id === selectedLeft.id); speakText(item.ar);
            selectedLeft = null; selectedRight = null; matchedCount++;
            if (matchedCount === 4) { score += pointsPerWord; document.getElementById('q-score').innerText = Math.round(score); setTimeout(() => { alert("🎉 完美匹配！"); nextWord(); }, 500); }
        } else {
            drawConnection(selectedLeft.el, selectedRight.el, false);
            setTimeout(() => { selectedLeft.el.classList.remove('selected'); selectedRight.el.classList.remove('selected'); selectedLeft = null; selectedRight = null; }, 500);
        }
    }
}
function drawConnection(el1, el2, isCorrect) {
    const svg = document.getElementById('matchSvg'); const r1 = el1.getBoundingClientRect(); const r2 = el2.getBoundingClientRect(); const c = document.getElementById('matchArea').getBoundingClientRect();
    const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    line.setAttribute('x1', r1.right - c.left - 5); line.setAttribute('y1', r1.top + r1.height/2 - c.top);
    line.setAttribute('x2', r2.left - c.left + 5); line.setAttribute('y2', r2.top + r2.height/2 - c.top);
    line.setAttribute('stroke', isCorrect ? '#4CAF50' : '#F44336'); svg.appendChild(line);
    if (!isCorrect) setTimeout(() => line.remove(), 500);
}

// 普通/填空渲染
function renderStandardMode() {
    const card = document.getElementById('cardContainer');
    card.innerHTML = `<div id="stdContent"><div class="word-header" id="qHeader"></div><div class="options-area" id="qOpts"></div></div>`;
    const header = document.getElementById('qHeader'); const opts = document.getElementById('qOpts');
    const spk = canSpeak ? 'audio-btn' : 'audio-btn disabled';

    if(currentMode === 'std') {
        // 阿选汉：显示复数阴性等标签
        let tagsHtml = '';
        if(currentWordObj.plural) tagsHtml += `<span class="mini-tag">复:${currentWordObj.plural}</span>`;
        if(currentWordObj.fem) tagsHtml += `<span class="mini-tag">阴:${currentWordObj.fem}</span>`;
        if(currentWordObj.present) tagsHtml += `<span class="mini-tag">现:${currentWordObj.present}</span>`;
        header.innerHTML = `<div style="color:#888;font-size:12px;margin-bottom:10px;">${currentWordObj.pos}</div><div class="ar-word-big">${currentWordObj.word}</div><div style="font-family:monospace;color:#999;">${currentWordObj.ipa}</div><div style="display:flex;gap:5px;margin-top:5px;justify-content:center;">${tagsHtml}</div><div class="${spk}" onclick="speakText('${currentWordObj.word}')">🔊</div>`;
        createOptions(true, opts);
    } else if (currentMode === 'rev') {
        header.innerHTML = `<div style="color:#888;font-size:12px;margin-bottom:10px;">${currentWordObj.pos}</div><div style="font-size:24px;font-weight:bold;color:#333;margin:15px 0;">${currentWordObj.def_cn}</div><div style="color:#999;font-size:14px;">请选择对应的阿语单词</div>`;
        createOptions(false, opts);
    } else if (currentMode === 'cloze') {
        let sent = currentWordObj.sent_ar; const regex = createFuzzyRegex(currentWordObj.word); let isReplaced = false;
        const safeSent = sent.replace(regex, () => { isReplaced = true; return '<span class="cloze-gap">_____</span>'; });
        const finalSent = isReplaced ? safeSent : sent + ' <br><span class="cloze-gap">_____</span>';
        header.innerHTML = `<div style="color:#888;font-size:12px;margin-bottom:10px;">拼写填空</div><div class="sentence-cloze">${finalSent}</div><div style="font-weight:bold;margin-top:10px;color:#555;">${currentWordObj.def_cn}</div>`;
        opts.innerHTML = `<input id="clozeIn" class="cloze-input" placeholder="输入阿语..."><button class="q-btn" onclick="checkCloze()">提交</button>`;
    }
}

function createOptions(isChinese, container) {
    let list = [];
    if(isChinese) {
        let others = currentData.filter(i => i.id !== currentWordObj.id).map(i => i.def_cn); shuffle(others);
        list = others.slice(0, 3).map(txt => ({ txt, isCorrect: false })); list.push({ txt: currentWordObj.def_cn, isCorrect: true });
    } else {
        let others = currentData.filter(i => i.id !== currentWordObj.id).map(i => i.word); shuffle(others);
        list = others.slice(0, 3).map(txt => ({ txt, isCorrect: false })); list.push({ txt: currentWordObj.word, isCorrect: true });
    }
    shuffle(list);
    list.forEach(o => {
        container.innerHTML += `<button class="option-btn ${!isChinese?'ar-opt':''}" onclick="checkOption(this, ${o.isCorrect})">${o.txt}</button>`;
    });
}

function checkOption(btn, isCorrect) {
    if(isAnswered) return; isAnswered = true; const target = currentMode==='std'?currentWordObj.def_cn:currentWordObj.word;
    if(isCorrect) { btn.classList.add('correct'); score += pointsPerWord; document.getElementById('q-score').innerText = Math.round(score); speakText(currentWordObj.word); }
    else { btn.classList.add('wrong'); document.querySelectorAll('.option-btn').forEach(b => { if(b.innerText === target) b.classList.add('correct'); }); setTimeout(openSheet, 600); }
}
function checkCloze() {
    if(isAnswered) return; const input = document.getElementById('clozeIn'); const val = input.value.trim(); if(!val) return;
    isAnswered = true;
    const u = stripTashkeel(val); const t = stripTashkeel(currentWordObj.word);
    if(u === t) { input.style.borderColor = 'green'; score += pointsPerWord; document.getElementById('q-score').innerText = Math.round(score); speakText(currentWordObj.word); document.querySelectorAll('.cloze-gap').forEach(g => { g.innerText = currentWordObj.word; g.classList.add('revealed'); }); }
    else { input.style.borderColor = 'red'; document.querySelectorAll('.cloze-gap').forEach(g => { g.innerText = currentWordObj.word; g.classList.add('revealed'); }); setTimeout(openSheet, 800); }
}

function nextWord() {
    if(currentIndex < currentData.length - 1) { currentIndex++; loadCard(currentIndex); }
    else { alert('🎉 课程结束！最终得分: ' + Math.round(score)); navTo('view-home'); }
}

function preloadSheet() {
    document.getElementById('sheet-word').innerText = currentWordObj.word; document.getElementById('sheet-ipa').innerText = currentWordObj.ipa;
    document.getElementById('sheet-def-cn').innerText = currentWordObj.def_cn; document.getElementById('sheet-def-ar').innerText = currentWordObj.def_ar || '';
    
    const morphBox = document.getElementById('sheet-morph-box');
    let morphHTML = '';
    if(currentWordObj.plural) morphHTML += `<span class="morph-tag">复: ${currentWordObj.plural}</span>`;
    if(currentWordObj.fem) morphHTML += `<span class="morph-tag">阴: ${currentWordObj.fem}</span>`;
    if(currentWordObj.present) morphHTML += `<span class="morph-tag">现: ${currentWordObj.present}</span>`;
    if(currentWordObj.source) morphHTML += `<span class="morph-tag">源: ${currentWordObj.source}</span>`;
    if(morphHTML) { morphBox.style.display='block'; document.getElementById('sheet-morph-content').innerHTML = morphHTML; } else morphBox.style.display='none';

    const setBox = (id, ar, cn) => { const el = document.getElementById(id); if(!ar && !cn) el.style.display='none'; else { el.style.display='block'; el.querySelector('.detail-ar').innerText = ar || ''; el.querySelectorAll('div')[2].innerText = cn || ''; }}; 
    setBox('sheet-collo-box', currentWordObj.collo_ar, currentWordObj.collo_cn); setBox('sheet-sent-box', currentWordObj.sent_ar, currentWordObj.sent_cn);
}
function openSheet(){document.querySelector('.modal-overlay').classList.add('show');document.getElementById('detail-sheet').classList.add('show');}
function closeSheet(){document.querySelector('.modal-overlay').classList.remove('show');document.getElementById('detail-sheet').classList.remove('show');}
</script></body></html>
