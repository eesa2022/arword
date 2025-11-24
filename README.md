<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>阿拉伯语智能背词</title>
<link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Noto+Sans+SC:wght@400;500;700&display=swap" rel="stylesheet">
<style>
:root {
    --primary: #4CAF50; --primary-dark: #388E3C; --danger: #F44336;
    --bg: #F2F3F5; --card-bg: #FFFFFF; --text-main: #333;
}
* { box-sizing: border-box; user-select: none; -webkit-tap-highlight-color: transparent; }
body {
    font-family: 'Noto Sans SC', sans-serif; background-color: var(--bg); margin: 0;
    display: flex; flex-direction: column; align-items: center;
    /* 关键：使用 dvh 适配手机浏览器地址栏 */
    height: 100vh; height: 100dvh; 
    overflow: hidden; padding: 0;
}

/* 顶部栏 */
.header-bar {
    width: 100%; padding: 15px 20px;
    display: flex; justify-content: space-between;
    color: #666; font-size: 14px; font-weight: bold;
    background: var(--bg); z-index: 10;
    padding-top: max(15px, env(safe-area-inset-top)); /* 适配刘海屏 */
}

/* 滚动区域 */
.main-scroll-area {
    flex: 1; width: 100%; max-width: 600px;
    overflow-y: auto; padding: 0 20px 120px 20px; /* 底部预留大空间给按钮 */
    display: flex; flex-direction: column; align-items: center;
}

.card-inner {
    width: 100%; background: var(--card-bg); border-radius: 20px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.05); overflow: hidden; margin-bottom: 20px;
}

/* 单词展示区 (动态高度) */
.word-header {
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    padding: 30px 20px 20px;
    background: linear-gradient(to bottom, #fff, #f9f9f9);
    min-height: 200px; /* 保证最小高度 */
    transition: all 0.3s;
}

.pos-badge {
    font-size: 12px; color: #2E7D32; background: #E8F5E9;
    padding: 4px 12px; border-radius: 12px; font-weight: 700; margin-bottom: 10px;
}

.ar-word {
    font-family: 'Amiri', serif;
    font-weight: 700; color: #000; text-align: center; line-height: 1.4;
    /* 关键：自适应字体大小 */
    font-size: clamp(32px, 10vw, 56px);
    margin: 5px 0;
}

.sentence-cloze {
    font-family: 'Amiri', serif; font-size: 26px; line-height: 1.8;
    text-align: center; direction: rtl; padding: 10px; color: #333;
}
.cloze-gap {
    display: inline-block; width: 60px; border-bottom: 2px solid var(--primary);
    margin: 0 5px; vertical-align: middle;
}

.ipa { font-family: monospace; color: #888; font-size: 16px; margin-top: 5px; }
.audio-icon { 
    width: 36px; height: 36px; border-radius: 50%; background: #f0f0f0; 
    color: var(--primary); display: flex; align-items: center; justify-content: center; 
    margin-top: 10px; cursor: pointer; 
}

/* 选项区 */
.options-area { padding: 20px; display: flex; flex-direction: column; gap: 12px; }
.option-btn {
    width: 100%; padding: 16px 20px; background: #fff;
    border: 2px solid #f0f0f0; border-radius: 16px;
    font-size: 16px; text-align: center; /* 选项居中更好看 */
    cursor: pointer; transition: 0.2s; font-weight: 500;
    display: flex; align-items: center; justify-content: center;
    min-height: 60px; /* 保证触摸面积 */
}
.option-btn.ar-opt { font-family: 'Amiri', serif; font-size: 24px; font-weight: bold; }

.option-btn.correct { background: var(--primary); color: #fff; border-color: var(--primary); }
.option-btn.wrong { background: var(--danger); color: #fff; border-color: var(--danger); animation: shake 0.4s; }

/* 底部固定栏 */
.fixed-footer {
    position: fixed; bottom: 0; left: 0; width: 100%;
    padding: 15px 20px; 
    padding-bottom: max(20px, env(safe-area-inset-bottom)); /* 适配 iPhone 黑条 */
    background: rgba(255,255,255,0.95); backdrop-filter: blur(10px);
    border-top: 1px solid #eee; display: flex; justify-content: center; z-index: 100;
}
.footer-inner { width: 100%; max-width: 600px; display: flex; gap: 15px; }
.action-btn {
    flex: 1; padding: 14px; border: none; border-radius: 14px;
    font-size: 16px; font-weight: bold; cursor: pointer;
    box-shadow: 0 4px 10px rgba(0,0,0,0.05);
}
.btn-show { background: #fff; border: 2px solid #eee; color: #555; }
.btn-next { background: var(--primary); color: white; }

/* 弹窗 */
.modal-overlay {
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0,0,0,0.6); z-index: 998;
    opacity: 0; pointer-events: none; transition: opacity 0.3s;
}
.modal-overlay.show { opacity: 1; pointer-events: auto; }
.bottom-sheet {
    position: fixed; bottom: -100%; left: 0; width: 100%;
    background: #fff; border-radius: 24px 24px 0 0; z-index: 999;
    transition: bottom 0.4s cubic-bezier(0.19,1,0.22,1);
    display: flex; flex-direction: column; max-height: 80vh;
    box-shadow: 0 -10px 40px rgba(0,0,0,0.2);
}
.bottom-sheet.show { bottom: 0; }
.sheet-content { padding: 25px 25px 40px 25px; overflow-y: auto; flex: 1; }
.detail-block { margin-bottom: 25px; padding-bottom: 15px; border-bottom: 1px solid #f5f5f5; }
.detail-ar { font-family: 'Amiri', serif; font-size: 24px; direction: rtl; margin-bottom: 8px; color: #000; }
@keyframes shake{0%,100%{transform:translateX(0);}25%{transform:translateX(-5px);}75%{transform:translateX(5px);}}
</style>
</head>
<body>
<div class="header-bar"><span id="progress-text">Loading...</span><span id="score-text">得分: 0</span></div>

<div class="main-scroll-area">
    <div class="card-inner">
        <div class="word-header" id="question-header"></div>
        
        <div class="options-area" id="options-container"></div>
    </div>
</div>

<div class="fixed-footer">
    <div class="footer-inner">
        <button class="action-btn btn-show" onclick="openSheet()">💡 详情</button>
        <button class="action-btn btn-next" onclick="nextWord()">下一个 ➜</button>
    </div>
</div>

<div class="modal-overlay" onclick="closeSheet()"></div>
<div class="bottom-sheet" id="detail-sheet">
    <div style="padding:15px;text-align:center;border-bottom:1px solid #eee;">
        <div style="width:40px;height:5px;background:#ddd;border-radius:5px;margin:0 auto 10px;"></div>
        <div style="font-weight:bold;color:var(--primary);">单词详解</div>
    </div>
    <div class="sheet-content">
        <div style="text-align:center; margin-bottom:20px;">
            <div id="sheet-word" style="font-family:'Amiri';font-size:36px;font-weight:bold;"></div>
            <div id="sheet-ipa" style="color:#888;"></div>
        </div>

        <div class="detail-block">
            <div style="font-size:12px;color:#999;text-transform:uppercase;">Definition</div>
            <div id="sheet-def-cn" style="font-size:18px;font-weight:bold;margin-bottom:5px;"></div>
            <div id="sheet-def-ar" class="detail-ar" style="font-size:20px;color:#555;"></div>
        </div>
        
        <div class="detail-block" id="sheet-collo-box">
            <div style="font-size:12px;color:#999;">常见搭配</div>
            <div id="sheet-collo-ar" class="detail-ar"></div>
            <div id="sheet-collo-cn" style="color:#444;"></div>
        </div>
        
        <div class="detail-block" id="sheet-sent-box">
            <div style="font-size:12px;color:#999;">例句</div>
            <div id="sheet-sent-ar" class="detail-ar"></div>
            <div id="sheet-sent-cn" style="color:#444;"></div>
        </div>
        
        <button class="action-btn btn-next" style="width:100%; margin-top:10px;" onclick="nextWord()">懂了，继续 ➜</button>
    </div>
</div>

<script>
const vocabData = [{"id":1,"word":"طَابِع","ipa":"[ṭābi']","root":"طبع","pos":"名词","def_cn":"邮票","def_ar":"مُلْصَقُ الْبَرِيدِ","plural":"طَوَابِع","fem":"","present":"","source":"","collo_ar":"جَمْعُ الطَّوَابِعِ","collo_cn":"集邮","sent_ar":"هِوَايَتِي جَمْعُ الطَّوَابِعِ.","sent_cn":"我的爱好是集邮。"},{"id":2,"word":"هِوَايَة","ipa":"[hiwāya]","root":"هوي","pos":"名词","def_cn":"业余爱好","def_ar":"عَمَلٌ مَحْبُوبٌ","plural":"هِوَايَات","fem":"","present":"","source":"","collo_ar":"هِوَايَةٌ مُفِيدَةٌ","collo_cn":"有益的爱好","sent_ar":"مَا هِيَ هِوَايَتُكَ الْمُفَضَّلَةُ؟","sent_cn":"你最喜欢的爱好是什么？"},{"id":3,"word":"هَوِيَ","ipa":"[hawiya]","root":"هوي","pos":"动词","def_cn":"喜欢，爱好","def_ar":"أَحَبَّ","plural":"","fem":"","present":"يَهْوَى","source":"هِوَايَةً","collo_ar":"يَهْوَى الْمُوسِيقَى","collo_cn":"爱好音乐","sent_ar":"يَهْوَى الرَّسْمَ مُنْذُ الصِّغَرِ.","sent_cn":"他从小就爱好画画。"},{"id":4,"word":"عَلِمَ","ipa":"['alima]","root":"علم","pos":"动词","def_cn":"知道，知晓","def_ar":"عَرَفَ","plural":"","fem":"","present":"يَعْلَمُ","source":"عِلْمًا","collo_ar":"عَلِمَ بِالْخَبَرِ","collo_cn":"得知消息","sent_ar":"عَلِمَ أَنَّ الْأَمْرَ خَطِيرٌ.","sent_cn":"他知道事情很严重。"},{"id":5,"word":"بِالضَّبْطِ","ipa":"[bi-ḍ-ḍabṭ]","root":"ضبط","pos":"副词","def_cn":"恰好，正是","def_ar":"تَمَامًا","plural":"","fem":"","present":"","source":"","collo_ar":"السَّاعَةُ الْخَامِسَةُ بِالضَّبْطِ","collo_cn":"正好五点","sent_ar":"هَذَا مَا أُرِيدُهُ بِالضَّبْطِ.","sent_cn":"这正是我想要的。"},{"id":6,"word":"جَازَ","ipa":"[jāza]","root":"جوز","pos":"动词","def_cn":"可以，允许","def_ar":"أُبِيحَ / أَمْكَنَ","plural":"","fem":"","present":"يَجُوزُ","source":"جَوَازًا","collo_ar":"يَجُوزُ لَكَ ذَلِكَ","collo_cn":"你可以那样做","sent_ar":"لَا يَجُوزُ التَّدْخِينُ هُنَا.","sent_cn":"这里不许吸烟。"},{"id":7,"word":"مُسْتَحِيل","ipa":"[mustaḥīl]","root":"حول","pos":"形容词","def_cn":"不可能的","def_ar":"غَيْرُ مُمْكِنٍ","plural":"","fem":"مُسْتَحِيلَة","present":"","source":"","collo_ar":"أَمْرٌ مُسْتَحِيلٌ","collo_cn":"不可能的事","sent_ar":"لَا شَيْءَ مُسْتَحِيلٌ.","sent_cn":"没有什么是不可能的。"},{"id":8,"word":"زَاوَلَ","ipa":"[zāwala]","root":"زول","pos":"动词","def_cn":"从事，实施","def_ar":"مَارَسَ","plural":"","fem":"","present":"يُزَاوِلُ","source":"مُزَاوَلَةً","collo_ar":"زَاوَلَ مِهْنَةً","collo_cn":"从事职业","sent_ar":"يُزَاوِلُ الرِّيَاضَةَ كُلَّ يَوْمٍ.","sent_cn":"他每天进行体育锻炼。"},{"id":9,"word":"آن / وَقْت","ipa":"[ān / waqt]","root":"أون / وقت","pos":"名词","def_cn":"时间，时期","def_ar":"زَمَن / حِين","plural":"آوِنَة / أَوْقَات","fem":"","present":"","source":"","collo_ar":"فِي آنٍ وَاحِدٍ","collo_cn":"同时","sent_ar":"لَيْسَ لَدَيَّ وَقْتٌ الْآنَ.","sent_cn":"我现在没时间。"},{"id":10,"word":"وَضَعَ","ipa":"[waḍa'a]","root":"وضع","pos":"动词","def_cn":"放，置","def_ar":"حَطَّ / جَعَلَ","plural":"","fem":"","present":"يَضَعُ","source":"وَضْعًا","collo_ar":"وَضَعَ الْخُطَّةَ","collo_cn":"制定计划","sent_ar":"وَضَعَ الْكِتَابَ عَلَى الْمَكْتَبِ.","sent_cn":"他把书放在桌子上。"},{"id":11,"word":"السُّودَان","ipa":"[as-sūdān]","root":"سود","pos":"地名","def_cn":"苏丹","def_ar":"دَوْلَةٌ عَرَبِيَّةٌ","plural":"","fem":"","present":"","source":"","collo_ar":"جُمْهُورِيَّةُ السُّودَانِ","collo_cn":"苏丹共和国","sent_ar":"سَافَرْتُ إِلَى السُّودَانِ.","sent_cn":"我去了苏丹。"},{"id":12,"word":"نَظَّفَ","ipa":"[naẓẓafa]","root":"نظف","pos":"动词","def_cn":"清洗，打扫","def_ar":"طَهَّرَ","plural":"","fem":"","present":"يُنَظِّفُ","source":"تَنْظِيفًا","collo_ar":"نَظَّفَ الْغُرْفَةَ","collo_cn":"打扫房间","sent_ar":"يُنَظِّفُ الطِّفْلُ أَسْنَانَهُ.","sent_cn":"孩子在刷牙。"},{"id":13,"word":"رَتَّبَ","ipa":"[rattaba]","root":"رتب","pos":"动词","def_cn":"整理，布置","def_ar":"نَظَّمَ","plural":"","fem":"","present":"يُرَتِّبُ","source":"تَرْتِيبًا","collo_ar":"رَتَّبَ السَّرِيرَ","collo_cn":"整理床铺","sent_ar":"رَتَّبَتِ الْأُمُّ الْمَنْزِلَ.","sent_cn":"母亲整理了房子。"},{"id":14,"word":"نَظَّمَ","ipa":"[naẓẓama]","root":"نظم","pos":"动词","def_cn":"组织，安排","def_ar":"رَتَّبَ / أَدَارَ","plural":"","fem":"","present":"يُنَظِّمُ","source":"تَنْظِيمًا","collo_ar":"نَظَّمَ وَقْتَهُ","collo_cn":"安排时间","sent_ar":"نَظَّمَتِ الْمَدْرَسَةُ رِحْلَةً.","sent_cn":"学校组织了一次旅行。"},{"id":15,"word":"ضَيَّعَ","ipa":"[ḍayya'a]","root":"ضيع","pos":"动词","def_cn":"浪费，错过","def_ar":"فَقَدَ / أَهْدَرَ","plural":"","fem":"","present":"يُضَيِّعُ","source":"تَضْيِيعًا","collo_ar":"ضَيَّعَ الْفُرْصَةَ","collo_cn":"错失良机","sent_ar":"لَا تُضَيِّعْ وَقْتَكَ فِي اللَّعِبِ.","sent_cn":"别把时间浪费在玩耍上。"},{"id":16,"word":"حَادِث","ipa":"[ḥādith]","root":"حدث","pos":"名词","def_cn":"事故，事件","def_ar":"وَاقِعَة","plural":"حَوَادِث","fem":"حَادِثَة","present":"","source":"","collo_ar":"حَادِثُ سَيَّارَةٍ","collo_cn":"车祸","sent_ar":"نَجَا مِنَ الْحَادِثِ بِأُعْجُوبَةٍ.","sent_cn":"他奇迹般地在事故中幸存。"},{"id":17,"word":"لَيْتَ","ipa":"[layta]","root":"-","pos":"虚词","def_cn":"但愿，希望","def_ar":"لِلتَّمَنِّي","plural":"","fem":"","present":"","source":"","collo_ar":"لَيْتَ الشَّبَابَ يَعُودُ","collo_cn":"但愿青春重来","sent_ar":"لَيْتَكَ كُنْتَ مَعَنَا.","sent_cn":"但愿你当时和我们在一起。"},{"id":18,"word":"مَتْحَف","ipa":"[matḥaf]","root":"تحف","pos":"名词","def_cn":"博物馆","def_ar":"مَكَانُ التُّحَفِ","plural":"مَتَاحِف","fem":"","present":"","source":"","collo_ar":"مَتْحَفُ التَّارِيخِ","collo_cn":"历史博物馆","sent_ar":"زُرْنَا الْمَتْحَفَ الْوَطَنِيَّ.","sent_cn":"我们参观了国家博物馆。"},{"id":19,"word":"إِمْكَانِيَّة","ipa":"[imkāniyya]","root":"مكن","pos":"名词","def_cn":"可能性","def_ar":"اِحْتِمَال","plural":"إِمْكَانِيَّات","fem":"","present":"","source":"","collo_ar":"عَدَمُ الْإِمْكَانِيَّةِ","collo_cn":"不可能性","sent_ar":"هُنَاكَ إِمْكَانِيَّةٌ لِلنَّجَاحِ.","sent_cn":"有成功的可能性。"},{"id":20,"word":"عَلَى كَيْفِكَ","ipa":"['alā kayfika]","root":"كيف","pos":"短语","def_cn":"随你便","def_ar":"كَمَا تُرِيدُ","plural":"","fem":"","present":"","source":"","collo_ar":"الْأَمْرُ عَلَى كَيْفِكَ","collo_cn":"事情随你意","sent_ar":"اِفْعَلْ مَا شِئْتَ، عَلَى كَيْفِكَ.","sent_cn":"做你想做的，随你便。"},{"id":21,"word":"مُبَاشَرَةً","ipa":"[mubāsharatan]","root":"بشر","pos":"副词","def_cn":"直接地","def_ar":"فَوْرًا / رَأْسًا","plural":"","fem":"","present":"","source":"","collo_ar":"ذَهَبَ مُبَاشَرَةً","collo_cn":"直接去","sent_ar":"سَأَتَّصِلُ بِكَ مُبَاشَرَةً.","sent_cn":"我会直接联系你。"},{"id":22,"word":"مَاكِيَاج","ipa":"[mākiyāj]","root":"-","pos":"名词","def_cn":"化妆","def_ar":"تَجْمِيل","plural":"","fem":"","present":"","source":"","collo_ar":"وَضْعُ الْمَاكِيَاجِ","collo_cn":"化妆","sent_ar":"تَضَعُ الْمَرْأَةُ الْمَاكِيَاجَ.","sent_cn":"女人正在化妆。"},{"id":23,"word":"أَشْرَفَ عَلَى","ipa":"[ashrafa 'alā]","root":"شرف","pos":"动词","def_cn":"主持，俯视","def_ar":"تَوَلَّى / أَطَلَّ","plural":"","fem":"","present":"يُشْرِفُ","source":"إِشْرَافًا","collo_ar":"أَشْرَفَ عَلَى الْبَحْثِ","collo_cn":"指导研究","sent_ar":"تُشْرِفُ النَّافِذَةُ عَلَى الْحَدِيقَةِ.","sent_cn":"窗户临着花园。"},{"id":24,"word":"نَتِيجَة","ipa":"[natīja]","root":"نتج","pos":"名词","def_cn":"结果，成绩","def_ar":"عَاقِبَة / ثَمَرَة","plural":"نَتَائِج","fem":"","present":"","source":"","collo_ar":"نَتِيجَةُ الِامْتِحَانِ","collo_cn":"考试成绩","sent_ar":"كَانَتِ النَّتِيجَةُ مُرْضِيَةً.","sent_cn":"结果令人满意。"},{"id":25,"word":"بَرَّرَ","ipa":"[barrara]","root":"برر","pos":"动词","def_cn":"辩护，解释","def_ar":"سَوَّغَ","plural":"","fem":"","present":"يُبَرِّرُ","source":"تَبْرِيرًا","collo_ar":"بَرَّرَ مَوْقِفَهُ","collo_cn":"辩解立场","sent_ar":"لَا تُبَرِّرْ أَخْطَاءَكَ.","sent_cn":"不要为你的错误辩解。"},{"id":26,"word":"حَسَدَ","ipa":"[ḥasada]","root":"حسد","pos":"动词","def_cn":"忌妒","def_ar":"تَمَنَّى زَوَالَ النِّعْمَةِ","plural":"","fem":"","present":"يَحْسُدُ","source":"حَسَدًا","collo_ar":"حَسَدَهُ عَلَى مَالِهِ","collo_cn":"嫉妒他的钱","sent_ar":"لَا تَحْسُدْ أَحَدًا عَلَى نِعْمَةٍ.","sent_cn":"不要嫉妒任何人的恩典。"},{"id":27,"word":"جَامَلَ","ipa":"[jāmala]","root":"جمل","pos":"动词","def_cn":"客套，奉承","def_ar":"أَحْسَنَ الْمُعَامَلَةَ","plural":"","fem":"","present":"يُجَامِلُ","source":"مُجَامَلَةً","collo_ar":"جَامَلَ الضَّيْفَ","collo_cn":"客套待客","sent_ar":"هُوَ يُجَامِلُ رَئِيسَهُ كَثِيرًا.","sent_cn":"他经常奉承他的老板。"},{"id":28,"word":"خَارِجَ","ipa":"[khārija]","root":"خرج","pos":"名词","def_cn":"在……外面","def_ar":"بَرَّا","plural":"","fem":"","present":"","source":"","collo_ar":"خَارِجَ الْبِلَادِ","collo_cn":"国外","sent_ar":"انْتَظِرْنِي خَارِجَ الْقَاعَةِ.","sent_cn":"在大厅外面等我。"},{"id":29,"word":"بَاعَ","ipa":"[bā'a]","root":"بيع","pos":"动词","def_cn":"卖，出售","def_ar":"تَاجَرَ","plural":"","fem":"","present":"يَبِيعُ","source":"بَيْعًا","collo_ar":"بَاعَ وَاشْتَرَى","collo_cn":"买卖","sent_ar":"بَاعَ التَّاجِرُ بِضَاعَتَهُ.","sent_cn":"商人卖掉了他的货物。"},{"id":30,"word":"سِعْر","ipa":"[si'r]","root":"سعر","pos":"名词","def_cn":"价格","def_ar":"ثَمَن","plural":"أَسْعَار","fem":"","present":"","source":"","collo_ar":"سِعْرٌ مُنَاسِبٌ","collo_cn":"合适的价格","sent_ar":"ارْتَفَعَتْ أَسْعَارُ الذَّهَبِ.","sent_cn":"金价上涨了。"},{"id":31,"word":"مَعْقُول","ipa":"[ma'qūl]","root":"عقل","pos":"形容词","def_cn":"合理的","def_ar":"مَقْبُول","plural":"","fem":"مَعْقُولَة","present":"","source":"","collo_ar":"كَلَامٌ مَعْقُولٌ","collo_cn":"合理的话","sent_ar":"هَذَا السِّعْرُ مَعْقُولٌ جِدًّا.","sent_cn":"这个价格非常合理。"},{"id":32,"word":"بِضَاعَة","ipa":"[biḍā'a]","root":"بضع","pos":"名词","def_cn":"货物","def_ar":"سِلْعَة","plural":"بَضَائِع","fem":"","present":"","source":"","collo_ar":"شَحْنُ الْبِضَاعَةِ","collo_cn":"运货","sent_ar":"الْبِضَاعَةُ الْمُبَاعَةُ لَا تُرَدُّ.","sent_cn":"售出商品概不退换。"},{"id":33,"word":"نَقْد","ipa":"[naqd]","root":"نقد","pos":"名词","def_cn":"现金，钱","def_ar":"مَال","plural":"نُقُود","fem":"","present":"","source":"","collo_ar":"دَفْعٌ نَقْدًا","collo_cn":"现金支付","sent_ar":"لَا أَحْمِلُ نُقُودًا كَافِيَةً.","sent_cn":"我没带够现金。"},{"id":34,"word":"اِفْتَكَرَ","ipa":"[iftakara]","root":"فكر","pos":"动词","def_cn":"想，认为","def_ar":"ظَنَّ / تَأَمَّلَ","plural":"","fem":"","present":"يَفْتَكِرُ","source":"اِفْتِكَارًا","collo_ar":"افْتَكَرَ فِي الْحَلِّ","collo_cn":"思考解决办法","sent_ar":"اِفْتَكَرْتُ أَنَّكَ نَائِمٌ.","sent_cn":"我以为你睡着了。"},{"id":35,"word":"عَلَى وَشْكِ","ipa":"['alā washki]","root":"وشك","pos":"短语","def_cn":"快要，几乎","def_ar":"قَرِيبٌ مِنْ","plural":"","fem":"","present":"","source":"","collo_ar":"عَلَى وَشْكِ الْانْتِهَاءِ","collo_cn":"快结束了","sent_ar":"الْقِطَارُ عَلَى وَشْكِ الْوُصُولِ.","sent_cn":"火车快到了。"},{"id":36,"word":"أَفْلَسَ","ipa":"[aflasa]","root":"فلس","pos":"动词","def_cn":"破产","def_ar":"خَسِرَ مَالَهُ","plural":"","fem":"","present":"يُفْلِسُ","source":"إِفْلَاسًا","collo_ar":"أَفْلَسَ التَّاجِرُ","collo_cn":"商人破产了","sent_ar":"أَعْلَنَتِ الشَّرِكَةُ إِفْلَاسَهَا.","sent_cn":"公司宣布破产。"},{"id":37,"word":"بَصِير","ipa":"[baṣīr]","root":"بصر","pos":"形容词","def_cn":"有远见的","def_ar":"ذُو بَصِيرَةٍ","plural":"","fem":"بَصِيرَة","present":"","source":"","collo_ar":"رَجُلٌ بَصِيرٌ","collo_cn":"有眼力的人","sent_ar":"كَانَ بَصِيرًا بِالْعَوَاقِبِ.","sent_cn":"他对后果很有预见性。"},{"id":38,"word":"أَغْلَقَ","ipa":"[aghlaqa]","root":"غلق","pos":"动词","def_cn":"关，关闭","def_ar":"سَدَّ / قَفَلَ","plural":"","fem":"","present":"يُغْلِقُ","source":"إِغْلَاقًا","collo_ar":"أَغْلَقَ الْبَابَ","collo_cn":"关门","sent_ar":"أَغْلَقَ الْمَحَلُّ أَبْوَابَهُ.","sent_cn":"商店关门了。"},{"id":39,"word":"غَالٍ","ipa":"[ghālin]","root":"غلو","pos":"形容词","def_cn":"贵的","def_ar":"مُرْتَفِعُ الثَّمَنِ","plural":"","fem":"غَالِيَة","present":"","source":"","collo_ar":"ثَمَنٌ غَالٍ","collo_cn":"昂贵的价格","sent_ar":"هَذِهِ السَّيَّارَةُ غَالِيَةٌ.","sent_cn":"这辆车很贵。"},{"id":40,"word":"رَخِيص","ipa":"[rakhīṣ]","root":"رخص","pos":"形容词","def_cn":"便宜的","def_ar":"قَلِيلُ الثَّمَنِ","plural":"","fem":"رَخِيصَة","present":"","source":"","collo_ar":"سِعْرٌ رَخِيصٌ","collo_cn":"便宜的价格","sent_ar":"اشْتَرَيْتُهُ بِثَمَنٍ رَخِيصٍ.","sent_cn":"我以低价买下了它。"},{"id":41,"word":"مَجْنُون","ipa":"[majnūn]","root":"جنن","pos":"形容词","def_cn":"发疯的","def_ar":"فَاقِدُ الْعَقْلِ","plural":"مَجَانِين","fem":"مَجْنُونَة","present":"","source":"","collo_ar":"حُبٌّ مَجْنُونٌ","collo_cn":"疯狂的爱","sent_ar":"هَلْ أَنْتَ مَجْنُونٌ؟","sent_cn":"你疯了吗？"},{"id":42,"word":"مُعْظَم","ipa":"[mu'ẓam]","root":"عظم","pos":"名词","def_cn":"大多数","def_ar":"أَكْثَر / غَالِبِيَّة","plural":"","fem":"","present":"","source":"","collo_ar":"مُعْظَمُ النَّاسِ","collo_cn":"大多数人","sent_ar":"قَرَأْتُ مُعْظَمَ الْكِتَابِ.","sent_cn":"我读了书的大部分。"},{"id":43,"word":"إِنْسَان","ipa":"[insān]","root":"أنس","pos":"名词","def_cn":"人，人类","def_ar":"بَشَر","plural":"نَاس","fem":"","present":"","source":"","collo_ar":"حُقُوقُ الْإِنْسَانِ","collo_cn":"人权","sent_ar":"الْإِنْسَانُ كَائِنٌ حَيٌّ.","sent_cn":"人是生物。"},{"id":44,"word":"مَتْجَر","ipa":"[matjar]","root":"تجر","pos":"名词","def_cn":"商店，商场","def_ar":"مَحَلٌّ تِجَارِيٌّ","plural":"مَتَاجِر","fem":"","present":"","source":"","collo_ar":"مَتْجَرُ الْمَلَابِسِ","collo_cn":"服装店","sent_ar":"ذَهَبَ إِلَى الْمَتْجَرِ لِلتَّسَوُّقِ.","sent_cn":"他去商场购物了。"},{"id":45,"word":"حُرِّيَّة","ipa":"[ḥurriyya]","root":"حرر","pos":"名词","def_cn":"自由","def_ar":"اِسْتِقْلَال","plural":"حُرِّيَّات","fem":"","present":"","source":"","collo_ar":"حُرِّيَّةُ الرَّأْيِ","collo_cn":"言论自由","sent_ar":"نَاضَلُوا مِنْ أَجْلِ الْحُرِّيَّةِ.","sent_cn":"他们为了自由而斗争。"},{"id":46,"word":"رَاحَة","ipa":"[rāḥa]","root":"روح","pos":"名词","def_cn":"休息，舒适","def_ar":"اِسْتِرَاحَة","plural":"","fem":"","present":"","source":"","collo_ar":"يَوْمُ الرَّاحَةِ","collo_cn":"休息日","sent_ar":"شَعَرَ بِالرَّاحَةِ بَعْدَ التَّعَبِ.","sent_cn":"劳累后他感到舒适。"},{"id":47,"word":"حُرّ","ipa":"[ḥurr]","root":"حرر","pos":"形容词","def_cn":"自由的","def_ar":"طَلِيق / مُسْتَقِلّ","plural":"أَحْرَار","fem":"حُرَّة","present":"","source":"","collo_ar":"رَجُلٌ حُرٌّ","collo_cn":"自由人","sent_ar":"أَنْتَ حُرٌّ فِي اخْتِيَارِكَ.","sent_cn":"你可以自由选择。"},{"id":48,"word":"وَقْتُ الْفَرَاغِ","ipa":"[waqtu-l-farāgh]","root":"وقت","pos":"短语","def_cn":"业余时间","def_ar":"وَقْتٌ خَالٍ","plural":"أَوْقَاتُ الْفَرَاغِ","fem":"","present":"","source":"","collo_ar":"فِي وَقْتِ الْفَرَاغِ","collo_cn":"在空闲时间","sent_ar":"أَقْرَأُ الرِّوَايَاتِ فِي وَقْتِ الْفَرَاغِ.","sent_cn":"我在业余时间读小说。"},{"id":49,"word":"طَبْع","ipa":"[ṭab']","root":"طبع","pos":"名词","def_cn":"个性，本性","def_ar":"سَجِيَّة / خُلُق","plural":"طِبَاع","fem":"","present":"","source":"","collo_ar":"حَادُّ الطَّبْعِ","collo_cn":"性格暴躁","sent_ar":"الطَّبْعُ يَغْلِبُ التَّطَبُّعَ.","sent_cn":"江山易改，本性难移。"},{"id":50,"word":"تَجَاذَبَ","ipa":"[tajādhaba]","root":"جذب","pos":"动词","def_cn":"聊天，互拉","def_ar":"تَبَادَلَ","plural":"","fem":"","present":"يَتَجَاذَبُ","source":"تَجَاذُبًا","collo_ar":"تَجَاذَبَ أَطْرَافَ الْحَدِيثِ","collo_cn":"闲聊","sent_ar":"جَلَسُوا يَتَجَاذَبُونَ أَطْرَافَ الْحَدِيثِ.","sent_cn":"他们坐在一起闲聊。"},{"id":51,"word":"يَدَوِيّ","ipa":"[yadawiyy]","root":"يدي","pos":"形容词","def_cn":"手工的","def_ar":"مَصْنُوعٌ بِالْيَدِ","plural":"","fem":"يَدَوِيَّة","present":"","source":"","collo_ar":"صُنْعٌ يَدَوِيٌّ","collo_cn":"手工制作","sent_ar":"هَذِهِ سَجَّادَةٌ يَدَوِيَّةٌ.","sent_cn":"这是一块手工地毯。"},{"id":52,"word":"زِرَاعَة","ipa":"[zirā'a]","root":"زرع","pos":"名词","def_cn":"农业","def_ar":"فَلَاحَة","plural":"","fem":"","present":"","source":"","collo_ar":"وِزَارَةُ الزِّرَاعَةِ","collo_cn":"农业部","sent_ar":"يَعْمَلُ الْفَلَّاحُ فِي الزِّرَاعَةِ.","sent_cn":"农民从事农业。"},{"id":53,"word":"مَكَان","ipa":"[makān]","root":"مكن","pos":"名词","def_cn":"地方，地点","def_ar":"مَوْضِع","plural":"أَمَاكِن","fem":"","present":"","source":"","collo_ar":"مَكَانٌ جَمِيلٌ","collo_cn":"美丽的地方","sent_ar":"هَذَا مَكَانٌ مُنَاسِبٌ لِلرَّاحَةِ.","sent_cn":"这是个休息的好地方。"},{"id":54,"word":"بَحَثَ","ipa":"[baḥatha]","root":"بحث","pos":"动词","def_cn":"找，寻找","def_ar":"فَتَّشَ عَنْ","plural":"","fem":"","present":"يَبْحَثُ","source":"بَحْثًا","collo_ar":"بَحَثَ عَنِ الْمِفْتَاحِ","collo_cn":"找钥匙","sent_ar":"يَبْحَثُ عَنْ عَمَلٍ جَدِيدٍ.","sent_cn":"他在找新工作。"},{"id":55,"word":"ذَات","ipa":"[dhāt]","root":"ذوت","pos":"名词","def_cn":"自身，物主","def_ar":"نَفْس / صَاحِبَة","plural":"ذَوَات","fem":"","present":"","source":"","collo_ar":"ذَاتُ قِيمَةٍ","collo_cn":"有价值的（东西）","sent_ar":"دَافَعَ عَنْ ذَاتِهِ.","sent_cn":"他为自己辩护。"},{"id":56,"word":"بِطَاقَة","ipa":"[biṭāqa]","root":"بطق","pos":"名词","def_cn":"卡片","def_ar":"وَرَقَةٌ صَغِيرَةٌ","plural":"بِطَاقَات","fem":"","present":"","source":"","collo_ar":"بِطَاقَةُ هُوِيَّةٍ","collo_cn":"身份证","sent_ar":"قَدَّمَ لِي بِطَاقَةَ دَعْوَةٍ.","sent_cn":"他给了我一张请以此帖。"},{"id":57,"word":"عُمْلَة","ipa":"['umla]","root":"عمل","pos":"名词","def_cn":"钱币，货币","def_ar":"نَقْد","plural":"عُمْلَات","fem":"","present":"","source":"","collo_ar":"عُمْلَةٌ صَعْبَةٌ","collo_cn":"硬通货","sent_ar":"جَمْعُ الْعُمْلَاتِ هِوَايَةٌ مُمْتِعَةٌ.","sent_cn":"收集钱币是有趣的爱好。"},{"id":58,"word":"الَّذِي","ipa":"[al-ladhī]","root":"-","pos":"关系代词","def_cn":"……者","def_ar":"اِسْمٌ مَوْصُولٌ","plural":"الَّذِينَ","fem":"الَّتِي","present":"","source":"","collo_ar":"الرَّجُلُ الَّذِي","collo_cn":"那个男人","sent_ar":"جَاءَ الطَّالِبُ الَّذِي نَجَحَ.","sent_cn":"那个成功的学生来了。"},{"id":59,"word":"مَثَّلَ","ipa":"[maththala]","root":"مثل","pos":"动词","def_cn":"代表，扮演","def_ar":"نَابَ عَنْ / لَعِبَ دَوْرًا","plural":"","fem":"","present":"يُمَثِّلُ","source":"تَمْثِيلاً","collo_ar":"مَثَّلَ بِلَادَهُ","collo_cn":"代表国家","sent_ar":"مَثَّلَ الْمُمَثِّلُ دَوْرًا رَئِيسِيًّا.","sent_cn":"演员扮演了主角。"},{"id":60,"word":"تَسَلُّقُ الْجِدَارِ","ipa":"[tasalluqu-l-jidār]","root":"سلق","pos":"短语","def_cn":"攀岩","def_ar":"رِيَاضَةُ التَّسَلُّقِ","plural":"","fem":"","present":"","source":"","collo_ar":"رِيَاضَةُ تَسَلُّقِ الْجِدَارِ","collo_cn":"攀岩运动","sent_ar":"تَسَلُّقُ الْجِدَارِ يَحْتَاجُ إِلَى قُوَّةٍ.","sent_cn":"攀岩需要力量。"},{"id":61,"word":"صَيْدُ الْأَسْمَاكِ","ipa":"[ṣaydu-l-asmāk]","root":"صيد","pos":"短语","def_cn":"钓鱼，捕鱼","def_ar":"إِمْسَاكُ السَّمَكِ","plural":"","fem":"","present":"","source":"","collo_ar":"أَدَوَاتُ صَيْدِ الْأَسْمَاكِ","collo_cn":"钓鱼工具","sent_ar":"ذَهَبُوا لِصَيْدِ الْأَسْمَاكِ فِي النَّهْرِ.","sent_cn":"他们去河里钓鱼了。"},{"id":62,"word":"مَا","ipa":"[mā]","root":"-","pos":"关系代词","def_cn":"……的（事物）","def_ar":"اِسْمٌ مَوْصُولٌ","plural":"","fem":"","present":"","source":"","collo_ar":"كُلُّ مَا تُرِيدُ","collo_cn":"你想要的一切","sent_ar":"اشْتَرِ مَا تَحْتَاجُ إِلَيْهِ.","sent_cn":"买你需要的东西。"},{"id":63,"word":"قِيمَة","ipa":"[qīma]","root":"قوم","pos":"名词","def_cn":"价值","def_ar":"قَدْر","plural":"قِيَم","fem":"","present":"","source":"","collo_ar":"قِيمَةٌ عَالِيَةٌ","collo_cn":"高价值","sent_ar":"لِهَذَا الْكِتَابِ قِيمَةٌ كَبِيرَةٌ.","sent_cn":"这本书价值很大。"},{"id":64,"word":"صَقَلَ","ipa":"[ṣaqala]","root":"صقل","pos":"动词","def_cn":"磨练，磨光","def_ar":"لَمَّعَ / هَذَّبَ","plural":"","fem":"","present":"يَصْقُلُ","source":"صَقْلاً","collo_ar":"صَقَلَ الْمَوْهِبَةَ","collo_cn":"磨练才干","sent_ar":"السَّفَرُ يَصْقُلُ مَوَاهِبَ الْإِنْسَانِ.","sent_cn":"旅行能磨练人的才干。"},{"id":65,"word":"عَزْم","ipa":"['azm]","root":"عزم","pos":"名词","def_cn":"决心","def_ar":"إِرَادَة / تَصْمِيم","plural":"عُزُوم","fem":"","present":"","source":"","collo_ar":"قُوَّةُ الْعَزْمِ","collo_cn":"意志力","sent_ar":"لَدَيْهِ عَزْمٌ قَوِيٌّ عَلَى النَّجَاحِ.","sent_cn":"他有成功的坚定决心。"},{"id":66,"word":"هَمّ","ipa":"[hamm]","root":"همم","pos":"名词","def_cn":"心思，忧虑","def_ar":"حُزْن / قَلَق","plural":"هُمُوم","fem":"","present":"","source":"","collo_ar":"يَحْمِلُ هَمًّا","collo_cn":"心怀忧虑","sent_ar":"أَزَالَ اللهُ هَمَّكَ.","sent_cn":"愿主消除你的忧虑。"},{"id":67,"word":"سَعَادَة","ipa":"[sa'āda]","root":"سعد","pos":"名词","def_cn":"幸福","def_ar":"فَرَح / سُرُور","plural":"","fem":"","present":"","source":"","collo_ar":"سَعَادَةُ الْأُسْرَةِ","collo_cn":"家庭幸福","sent_ar":"أَتَمَنَّى لَكُمُ السَّعَادَةَ الدَّائِمَةَ.","sent_cn":"我祝你们永远幸福。"},{"id":68,"word":"تَامّ","ipa":"[tāmm]","root":"تمم","pos":"形容词","def_cn":"完全的，圆满的","def_ar":"كَامِل","plural":"","fem":"تَامَّة","present":"","source":"","collo_ar":"شِفَاءٌ تَامٌّ","collo_cn":"完全康复","sent_ar":"لَدَيَّ ثِقَةٌ تَامَّةٌ بِكَ.","sent_cn":"我对你完全信任。"}];
let currentIndex = 0;
let score = 0;
let isAnswered = false;
let currentWordObj = null;
let currentMode = 'std'; // std(常规), rev(汉选阿), cloze(填空)
let pointsPerWord = 0;

function shuffle(arr) { return arr.sort(() => Math.random() - 0.5); }

document.addEventListener('DOMContentLoaded', () => {
    if (!vocabData || vocabData.length === 0) { alert('没有数据'); return; }
    shuffle(vocabData);
    pointsPerWord = 100 / vocabData.length;
    loadCard(currentIndex);
});

function loadCard(index) {
    closeSheet();
    setTimeout(() => {
        isAnswered = false;
        currentWordObj = vocabData[index];
        document.getElementById('progress-text').innerText = (index + 1) + " / " + vocabData.length;

        // 随机决定题型
        const roll = Math.random();
        // 如果有例句，30%概率出填空题；否则只在常规和汉选阿之间随机
        if (currentWordObj.sent_ar && roll > 0.7) {
            currentMode = 'cloze'; // 填空题
        } else if (roll > 0.35) {
            currentMode = 'rev';   // 汉选阿
        } else {
            currentMode = 'std';   // 阿选汉(经典)
        }

        renderQuestion();
        preloadSheet();
    }, 250);
}

function renderQuestion() {
    const header = document.getElementById('question-header');
    const optContainer = document.getElementById('options-container');
    header.innerHTML = '';
    optContainer.innerHTML = '';

    // 1. 生成问题头
    if (currentMode === 'std') {
        // [阿 -> 中] 显示大阿语
        header.innerHTML = `
            <div class="pos-badge">${currentWordObj.pos}</div>
            <div class="ar-word">${currentWordObj.word}</div>
            <div class="ipa">${currentWordObj.ipa}</div>
            <div class="audio-icon" onclick="speakText('${currentWordObj.word}')">🔊</div>
        `;
    } else if (currentMode === 'rev') {
        // [中 -> 阿] 显示中文，隐藏阿语
        header.innerHTML = `
            <div class="pos-badge">${currentWordObj.pos}</div>
            <div style="font-size:24px; font-weight:bold; color:#333; margin:10px 0;">${currentWordObj.def_cn}</div>
            <div style="color:#888; font-size:14px;">请选择对应的阿语单词</div>
        `;
    } else if (currentMode === 'cloze') {
        // [填空] 显示挖空的阿语例句
        // 简单处理：把单词去掉，替换为下划线。注意：需要处理前缀后缀可能比较复杂，这里做简单替换
        // 为了匹配更准，直接替换原句中的词。
        let sent = currentWordObj.sent_ar;
        // 尝试移除单词（不做太复杂的正则，简单替换）
        const blank = '<span class="cloze-gap"></span>';
        const safeSent = sent.replace(currentWordObj.word, blank); 
        
        header.innerHTML = `
            <div class="pos-badge">句子填空</div>
            <div class="sentence-cloze">${safeSent.includes('span') ? safeSent : sent.replace(/\S{3,}/, blank)}</div>
            <div style="font-size:14px; color:#666; margin-top:10px;">${currentWordObj.sent_cn}</div>
        `;
    }

    // 2. 生成选项
    let options = [];
    if (currentMode === 'std') {
        // 选项是中文
        let distractors = vocabData.filter(i => i.id !== currentWordObj.id).map(i => i.def_cn);
        shuffle(distractors);
        options = distractors.slice(0, 3).map(txt => ({ txt, isCorrect: false }));
        options.push({ txt: currentWordObj.def_cn, isCorrect: true });
    } else {
        // 选项是阿语 (Rev 和 Cloze 模式)
        let distractors = vocabData.filter(i => i.id !== currentWordObj.id).map(i => i.word);
        shuffle(distractors);
        options = distractors.slice(0, 3).map(txt => ({ txt, isCorrect: false }));
        options.push({ txt: currentWordObj.word, isCorrect: true });
    }
    
    shuffle(options);

    options.forEach(opt => {
        const btn = document.createElement('button');
        btn.className = 'option-btn' + (currentMode !== 'std' ? ' ar-opt' : '');
        // 如果是阿语选项，强制右对齐可能更好，但居中通用性强
        btn.innerText = opt.txt;
        btn.onclick = () => checkAnswer(btn, opt.isCorrect);
        optContainer.appendChild(btn);
    });
}

function checkAnswer(btn, isCorrect) {
    if (isAnswered) return;
    isAnswered = true;
    const allBtns = document.querySelectorAll('.option-btn');
    
    // 找出正确答案的文本
    const correctTxt = currentMode === 'std' ? currentWordObj.def_cn : currentWordObj.word;

    if (isCorrect) {
        btn.classList.add('correct');
        score += pointsPerWord;
        document.getElementById('score-text').innerText = "得分: " + Math.round(score);
        speakText(currentWordObj.word);
    } else {
        btn.classList.add('wrong');
        allBtns.forEach(b => {
            if (b.innerText === correctTxt) b.classList.add('correct');
        });
        setTimeout(openSheet, 500);
    }
}

function preloadSheet() {
    document.getElementById('sheet-word').innerText = currentWordObj.word;
    document.getElementById('sheet-ipa').innerText = currentWordObj.ipa;
    document.getElementById('sheet-def-cn').innerText = currentWordObj.def_cn;
    document.getElementById('sheet-def-ar').innerText = currentWordObj.def_ar || '';

    const setBox = (id, ar, cn) => {
        const el = document.getElementById(id);
        if(!ar && !cn) el.style.display='none';
        else {
            el.style.display='block';
            el.querySelector('.detail-ar').innerText = ar || '';
            el.querySelectorAll('div')[2].innerText = cn || ''; // 第三个div是中文
        }
    };
    setBox('sheet-collo-box', currentWordObj.collo_ar, currentWordObj.collo_cn);
    setBox('sheet-sent-box', currentWordObj.sent_ar, currentWordObj.sent_cn);
}

function nextWord() {
    if (currentIndex < vocabData.length - 1) {
        currentIndex++; loadCard(currentIndex);
    } else {
        alert("🎉 测试结束！最终得分: " + Math.round(score));
        location.reload();
    }
}

function speakText(text) {
    const u = new SpeechSynthesisUtterance(text);
    u.lang = 'ar-SA'; u.rate = 0.9;
    window.speechSynthesis.speak(u);
}

function openSheet(){document.querySelector('.modal-overlay').classList.add('show');document.getElementById('detail-sheet').classList.add('show');}
function closeSheet(){document.querySelector('.modal-overlay').classList.remove('show');document.getElementById('detail-sheet').classList.remove('show');}
</script>
</body>
</html>
