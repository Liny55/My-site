<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>回声澜界</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent}
body{background:#070a10;color:#d8e2f0;font-family:-apple-system,"PingFang SC",sans-serif;height:100vh;height:100dvh;overflow:hidden;display:flex;align-items:center;justify-content:center}
#app{width:100%;max-width:720px;height:100vh;height:100dvh;background:#0d131e;display:flex;flex-direction:column;position:relative;overflow:hidden}
#hud{padding:10px 14px;border-bottom:1px solid #1a2230;font-size:12px;color:#5a6a80;display:flex;gap:12px;flex-wrap:wrap;flex-shrink:0}
#hud b{color:#8ab0d0;font-weight:normal}
#main{flex:1;padding:18px;overflow-y:auto;-webkit-overflow-scrolling:touch}
#btns{padding:10px 14px 18px;display:flex;flex-direction:column;gap:6px;flex-shrink:0;max-height:52vh;overflow-y:auto}
button{text-align:left;background:#141c29;color:#c8d4e4;border:1px solid #2a3648;padding:11px 14px;border-radius:8px;font-size:13.5px;cursor:pointer;font-family:inherit;line-height:1.5}
button:active{background:#1c2738}
button.primary{background:#1a2a40;border-color:#3e5470;color:#e0e8f0}
button.danger{color:#d0a080;border-color:#3a2a20}
button.gray{color:#6d7a8c;font-size:12px;text-align:center}
button:disabled{opacity:.35;cursor:default}
h2{font-size:17px;color:#7fb0d8;margin-bottom:10px}
p{font-size:14px;line-height:1.85;color:#c8d4e4;margin-bottom:9px}
.dim{color:#7a8798;font-size:12.5px}
.npc{color:#f0a0c0}
.me{color:#a0d0f0}
.echo{color:#a0c8a0;font-style:italic}
.changes{margin:14px 0;padding:10px 14px;background:#0a0f17;border-radius:8px;border:1px solid #1a2230}
.change{font-size:13px;padding:3px 0;display:flex;justify-content:space-between}
.change .up{color:#6ac080}
.change .down{color:#d08080}
.change .info{color:#8ab0d0}
.daytitle{font-size:12px;color:#5a6a80;margin-bottom:14px;letter-spacing:1px}
.warnbox{padding:10px 14px;background:#1a1210;border:1px solid #3a2a20;border-radius:8px;font-size:12.5px;color:#d0a080;margin-bottom:12px;line-height:1.6}
</style>
</head>
<body>
<div id="app">
  <div id="hud">
    <span>第 <b id="day">1</b> 天</span>
    <span>行动点 <b id="ap">2</b>/2</span>
    <span>线索 <b id="clue">0</b></span>
    <span>静默 <b id="sil">0</b></span>
    <span>议会 <b id="council">0</b></span>
  </div>
  <div id="main"></div>
  <div id="btns"></div>
</div>
<script>
const S = {
  day: 1, ap: 2,
  clues: [], silence: 0, council: 0,
  rel: { ayou: 0, qiyie: 0, wenjue: 0, libo: 0 },
  flags: {}, acted: {}, missed: [],
};

const $ = id => document.getElementById(id);
const main = $('main'), btns = $('btns');

function updateHUD() {
  $('day').textContent = S.day;
  $('ap').textContent = S.ap;
  $('clue').textContent = S.clues.length;
  $('sil').textContent = S.silence;
  $('council').textContent = S.council;
}
function addClue(id) { if (!S.clues.includes(id)) S.clues.push(id); }
function rel(w, n) { S.rel[w] = Math.max(-5, Math.min(5, S.rel[w] + n)); }
function sil(n) {
  S.silence = Math.max(0, S.silence + n);
  if (S.silence >= 10 && !S.flags.memory_lost) {
    S.flags.memory_lost = true;
    S.missed.push('你的记忆出现了一小块空白。');
  }
}
function council(n) { S.council = Math.max(0, Math.min(5, S.council + n)); }

function showFeedback(title, text, changes, btnText, onContinue) {
  main.innerHTML = `<h2>${title}</h2>` +
    text.map(p => `<p>${p}</p>`).join('') +
    (changes.length ? `<div class="changes">${changes.map(c =>
      `<div class="change"><span>${c.t}</span><span class="${c.type}">${c.v}</span></div>`
    ).join('')}</div>` : '');
  btns.innerHTML = '';
  const b = document.createElement('button');
  b.className = 'primary';
  b.textContent = btnText || '继续';
  b.onclick = onContinue;
  btns.appendChild(b);
}

const ACTIONS = {
  ayou: {
    label: '找阿柚',
    desc: '她在礼堂，铃铛又在手里转。',
    run() {
      const n = (S.acted.ayou || 0) + 1;
      S.acted.ayou = n;
      if (n === 1) {
        addClue('ayou_song'); rel('ayou', 1);
        return {
          title: '礼堂 · 阿柚',
          text: ['<span class="npc">阿柚</span>：“你就是新来的？雾廊晚上有歌声，千万别去。”',
            '<span class="dim">她说话时把铃铛转得很快，像在掩饰什么。</span>'],
          changes: [
            { t: '阿柚信任', v: '+1', type: 'up' },
            { t: '获得线索：雾廊歌声', v: '', type: 'info' },
          ],
        };
      }
      if (n === 2) {
        addClue('bell_hint'); rel('ayou', 1);
        return {
          title: '礼堂 · 阿柚',
          text: ['阿柚把铃铛递过来：“它最近总响。你听听看，是不是坏了？”',
            '<span class="dim">铃身有一道旧裂纹，里面像压着不属于市集的声音。</span>'],
          changes: [
            { t: '阿柚信任', v: '+1', type: 'up' },
            { t: '开启支线：流星铃', v: '', type: 'info' },
          ],
        };
      }
      if (n === 3) {
        rel('ayou', 1);
        return {
          title: '礼堂 · 阿柚',
          text: ['阿柚：“你最近总来。是不是……也听见那歌声了？”',
            '<span class="dim">她第一次没有用玩笑的语气。</span>'],
          changes: [{ t: '阿柚信任', v: '+1', type: 'up' }],
        };
      }
      rel('ayou', 1);
      return {
        title: '礼堂 · 阿柚',
        text: ['阿柚把零食推过来：“吃点东西。你脸色不太好。”'],
        changes: [{ t: '阿柚信任', v: '+1', type: 'up' }],
      };
    },
  },

  classroom: {
    label: '去潮音教室',
    desc: '黎伯在写字，闻玦在核对目录。',
    run() {
      const n = (S.acted.classroom || 0) + 1;
      S.acted.classroom = n;
      if (n === 1) {
        addClue('classroom_song'); rel('libo', 1);
        return {
          title: '潮音教室',
          text: ['黎伯在石板上写：“代价律——改变回响，必须付出等量代价。”',
            '<span class="dim">写到最后，笔停了很久。墙里好像有极轻的歌声。</span>'],
          changes: [
            { t: '黎伯信任', v: '+1', type: 'up' },
            { t: '获得线索：课堂异响', v: '', type: 'info' },
          ],
        };
      }
      if (n === 2) {
        addClue('catalog'); rel('wenjue', 1);
        return {
          title: '潮音教室 · 闻玦',
          text: ['闻玦把目录推过来：“缺页记录被改过。改得很小心，但还是留下了痕。”',
            '<span class="dim">他顿了顿：“我哥也查过这个。”</span>'],
          changes: [
            { t: '闻玦信任', v: '+1', type: 'up' },
            { t: '获得线索：目录篡改', v: '', type: 'info' },
          ],
        };
      }
      rel('wenjue', 1);
      return {
        title: '潮音教室',
        text: ['闻玦把一叠整理好的笔记放在你桌上：“顺手。”',
          '<span class="dim">笔记是新的，还有墨迹。他大概是刚抄完。</span>'],
        changes: [{ t: '闻玦信任', v: '+1', type: 'up' }],
      };
    },
  },

  library: {
    label: '去旧物图书馆',
    desc: '安静，适合查不想被别人知道的事。',
    run() {
      const n = (S.acted.library || 0) + 1;
      S.acted.library = n;
      if (n === 1) {
        addClue('catalog'); sil(1);
        return {
          title: '旧物图书馆',
          text: ['你翻开目录，缺页记录被改过。旁边有一行铅笔小字：“问闻玦。”',
            '<span class="dim">旧物在架子上低声回响，像是在提醒你什么。</span>'],
          changes: [
            { t: '获得线索：目录篡改', v: '', type: 'info' },
            { t: '静默值', v: '+1', type: 'down' },
          ],
        };
      }
      if (n === 2) {
        rel('wenjue', 1); addClue('wenjue_echo');
        return {
          title: '旧物图书馆 · 闻玦',
          text: ['闻玦在角落找到你：“我哥留下半段回声。里面只有一句话——不是被盗，是被藏。”',
            '<span class="dim">他把一张旧纸推到你面前，没有收回手。</span>'],
          changes: [
            { t: '闻玦信任', v: '+1', type: 'up' },
            { t: '获得线索：半段回声', v: '', type: 'info' },
          ],
        };
      }
      sil(1);
      return {
        title: '旧物图书馆',
        text: ['你在角落坐了整晚。旧物在你耳边轻轻回响，你听不清它们在说什么。'],
        changes: [{ t: '静默值', v: '+1', type: 'down' }],
      };
    },
  },

  fog: {
    label: '去雾廊',
    desc: '危险。白天也像夜里。',
    danger: true,
    available() {
      return (S.acted.classroom || 0) >= 1 || S.day >= 2;
    },
    run() {
      const n = (S.acted.fog || 0) + 1;
      S.acted.fog = n;
      if (n === 1 && !S.flags.fog_night_done) {
        S.flags.fog_night_done = true;
        addClue('fog_song'); addClue('qiyie_hint'); rel('qiyie', 1);
        return {
          title: '雾廊 · 夜遇',
          text: ['雾里走出一个人。是栖夜。',
            '<span class="npc">栖夜</span>：“第七段回声不是被盗，是被藏。藏它的人，不想让议会拿到。”',
            '<span class="me">你</span>：（他看着我，像在等我做一个决定。）'],
          changes: [
            { t: '栖夜信任', v: '+1', type: 'up' },
            { t: '获得线索：第七回声', v: '', type: 'info' },
          ],
        };
      }
      if (n === 2 && !S.flags.notice_full) {
        S.flags.notice_full = true; addClue('notice_full'); council(1);
        return {
          title: '雾廊 · 旧告示',
          text: ['半张告示贴在雾廊尽头。留影术下，缺失的字慢慢显出来。',
            '<span class="me">你</span>：（日期是二十年前。潮汐夜，禁入静默井。）'],
          changes: [
            { t: '获得线索：旧告示完整', v: '', type: 'info' },
            { t: '议会介入', v: '+1', type: 'down' },
          ],
        };
      }
      sil(2); rel('qiyie', -1);
      return {
        title: '雾廊',
        text: ['你在雾廊里走了很久，什么也没找到。静默值又高了一点。'],
        changes: [
          { t: '静默值', v: '+2', type: 'down' },
          { t: '栖夜怀疑', v: '-1', type: 'down' },
        ],
      };
    },
  },

  bridge: {
    label: '去回声桥',
    desc: '风很大，适合坐一会儿。',
    run() {
      const n = (S.acted.bridge || 0) + 1;
      S.acted.bridge = n;
      if (n === 1) {
        rel('qiyie', 1); addClue('qiyie_soft');
        return {
          title: '回声桥 · 观潮',
          text: ['雾海退下去，桥边的风带着咸味。栖夜在你旁边坐下。',
            '栖夜：“我没有家。学园收留我的时候，黎伯说，守井人不需要家。”',
            '<span class="dim">他顿了顿：“但有时候，我会觉得桥边比井边暖和。”</span>'],
          changes: [
            { t: '栖夜信任', v: '+1', type: 'up' },
            { t: '获得线索：栖夜的软处', v: '', type: 'info' },
          ],
        };
      }
      rel('ayou', 1);
      return {
        title: '回声桥',
        text: ['阿柚拉着你赛跑。桥很窄，风很大，她跑得比看起来快。',
          '<span class="dim">跑到一半她突然停下，说：“要是毕业了还能这样跑就好了。”</span>'],
        changes: [{ t: '阿柚信任', v: '+1', type: 'up' }],
      };
    },
  },

  market: {
    label: '去外岛市集',
    desc: '退潮才开门。用记忆和承诺做买卖。',
    available() { return S.day >= 3; },
    run() {
      if (S.flags.market_done) {
        return {
          title: '外岛市集',
          text: ['市集门半开着。商人看了你一眼，没有招呼。'],
          changes: [],
        };
      }
      S.flags.market_done = true;
      return {
        title: '外岛市集 · 交易',
        text: ['<span class="npc">市集商人</span>：“退潮才开门。你带了什么来换？”',
          '<span class="dim">他面前摆着旧回声、旧名字、旧承诺。</span>'],
        changes: [],
        isChoice: true,
      };
    },
  },
};
function renderHub() {
  updateHUD();
  const acts = Object.keys(ACTIONS).filter(k => {
    const a = ACTIONS[k];
    if (a.available && !a.available()) return false;
    if (k === 'market' && S.flags.market_done) return false;
    return true;
  });

  main.innerHTML = `<div class="daytitle">第 ${S.day} 天 · 行动点 ${S.ap}/2</div>` +
    `<h2>今天做什么？</h2>` +
    `<p class="dim">每天只有两个行动点。没做的事，世界不会等你。</p>` +
    `<div class="changes">` +
    Object.keys(S.rel).map(k => {
      const v = S.rel[k];
      const cls = v > 0 ? 'up' : v < 0 ? 'down' : 'info';
      return `<div class="change"><span>${k}</span><span class="${cls}">${v > 0 ? '+' : ''}${v}</span></div>`;
    }).join('') +
    `</div>`;

  btns.innerHTML = '';
  acts.forEach(k => {
    const a = ACTIONS[k];
    const b = document.createElement('button');
    b.innerHTML = `<b>${a.label}</b><br><span class="dim">${a.desc}</span>`;
    if (a.danger) b.className = 'danger';
    b.onclick = () => doAction(k);
    btns.appendChild(b);
  });
  const book = document.createElement('button');
  book.className = 'gray';
  book.textContent = '查看线索簿';
  book.onclick = showBook;
  btns.appendChild(book);
}

function doAction(key) {
  if (S.ap <= 0) return;
  S.ap--;
  updateHUD();
  const a = ACTIONS[key];
  if (key === 'market' && !S.flags.market_done) {
    S.flags.market_done = true;
    showMarketChoice();
    return;
  }
  const result = a.run();
  showFeedback(result.title, result.text, result.changes,
    S.ap <= 0 ? '结束这一天' : '继续',
    () => { S.ap <= 0 ? nightSettle() : renderHub(); });
}

function showMarketChoice() {
  main.innerHTML = `<h2>外岛市集 · 交易</h2>
    <p><span class="npc">市集商人</span>：“退潮才开门。你带了什么来换？”</p>
    <p class="dim">他面前摆着旧回声、旧名字、旧承诺。</p>
    <div class="warnbox">不可逆：交易一旦完成，代价会留在你身上。</div>`;
  btns.innerHTML = '';
  const opts = [
    { t: '用一段记忆换线索', f: () => {
      sil(3); addClue('market_echo'); council(1);
      showFeedback('市集交易', ['你走出市集时，忘了自己刚才想说什么。手里多了一小段回声。'],
        [{ t: '静默值', v: '+3', type: 'down' }, { t: '获得线索：市集回声', v: '', type: 'info' }, { t: '议会介入', v: '+1', type: 'down' }],
        S.ap <= 0 ? '结束这一天' : '继续',
        () => { S.ap <= 0 ? nightSettle() : renderHub(); });
    }},
    { t: '用承诺换', f: () => {
      S.flags.market_promise = true; addClue('market_echo'); rel('ayou', -1);
      showFeedback('市集交易', ['商人：“毕业前，帮我带一个人来。”', '<span class="dim">你点了头，才想起没问是谁。</span>'],
        [{ t: '获得线索：市集回声', v: '', type: 'info' }, { t: '阿柚怀疑', v: '-1', type: 'down' }],
        S.ap <= 0 ? '结束这一天' : '继续',
        () => { S.ap <= 0 ? nightSettle() : renderHub(); });
    }},
    { t: '不换', f: () => {
      showFeedback('市集交易', ['商人耸耸肩：“那就等你改主意。”'], [],
        S.ap <= 0 ? '结束这一天' : '继续',
        () => { S.ap <= 0 ? nightSettle() : renderHub(); });
    }},
  ];
  opts.forEach(o => {
    const b = document.createElement('button');
    b.textContent = o.t;
    b.onclick = o.f;
    btns.appendChild(b);
  });
}

function nightSettle() {
  const events = [];

  if (S.day === 1 && !S.acted.ayou) {
    events.push('你听见宿舍里阿柚在摆弄铃铛，声音很轻。她好像在等谁问一句，但没有人问。');
    rel('ayou', -1);
  }
  if (S.day === 2 && !S.flags.fog_night_done) {
    events.push('第二天早上，你听说昨晚院长在雾廊抓到一个人。没人说是谁。');
    S.flags.qiyie_caught = true; rel('qiyie', -2); council(1);
  }
  if (S.day === 3) {
    events.push('议会的人出现在学园门口。岑澜站在门内，没有开门。');
    council(1);
  }
  if (S.day === 4 && S.clues.length < 5) {
    events.push('你听见岑澜在走廊里问黎伯：“那个新生，查到了什么？”');
    rel('libo', -1);
  }
  if (S.day === 5) {
    events.push('雾廊的歌声停了。静默井的裂痕在夜里发光。');
  }

  if (events.length === 0) {
    events.push('夜色很静。你躺在床上，听见雾从窗缝里渗进来。');
  }

  main.innerHTML = `<h2>第 ${S.day} 天 · 夜晚</h2>` +
    events.map(e => `<p>${e}</p>`).join('') +
    (S.missed.length ? `<div class="warnbox">你错过了：${S.missed.join(' ')}</div>` : '');
  btns.innerHTML = '';
  const b = document.createElement('button');
  b.className = 'primary';

  if (S.day >= 5) {
    b.textContent = '走向静默井';
    b.onclick = finale;
  } else {
    b.textContent = `进入第 ${S.day + 1} 天`;
    b.onclick = () => {
      S.day++;
      S.ap = 2;
      S.missed = [];
      renderHub();
    };
  }
  btns.appendChild(b);
}

function finale() {
  if (S.clues.length < 4) {
    main.innerHTML = `<h2>静默井入口</h2>
      <p>井口低鸣。你觉得自己知道得还不够。</p>
      <p class="dim">至少再找几条线索，才能面对井下的东西。</p>`;
    btns.innerHTML = '';
    const b = document.createElement('button');
    b.textContent = '硬着头皮下去';
    b.onclick = finalChoice;
    btns.appendChild(b);
    return;
  }
  main.innerHTML = `<h2>终章 · 静默井</h2>
    <p>静默井的裂痕在夜里发光。栖夜站在井边，手里攥着那页失踪的回声。</p>
    <p><span class="npc">栖夜</span>：“你们不该跟来。”</p>
    <p><span class="me">你</span>：（现在要说什么？）</p>`;
  btns.innerHTML = '';
  [
    { t: '出示线索', f: () => {
      main.innerHTML += `<p>你把线索一条条摆出来。井边的雾随着每句话变薄。</p>
        <p><span class="npc">栖夜</span>：“……你查得比我想的深。”</p>`;
      finalChoice();
    }},
    { t: '问栖夜为什么藏', f: () => {
      rel('qiyie', 1);
      main.innerHTML += `<p>栖夜：“因为议会拿到它，学园就不再是学园了。”</p>`;
      finalChoice();
    }},
    { t: '看其他人的反应', f: () => {
      main.innerHTML += `<p>阿柚站在你身后，闻玦站在门口。没有人先开口。</p>`;
      finalChoice();
    }},
  ].forEach(o => {
    const b = document.createElement('button');
    b.textContent = o.t;
    b.onclick = o.f;
    btns.appendChild(b);
  });
}

function finalChoice() {
  main.innerHTML += `<div class="warnbox">不可逆：这是最后的选择。</div>`;
  btns.innerHTML = '';
  [
    { t: '公开真相', k: 'truth_public' },
    { t: '帮栖夜隐瞒', k: 'truth_hide' },
    { t: '联合三人修复回声', k: 'truth_repair' },
    { t: '和市集交易', k: 'truth_trade' },
  ].forEach(o => {
    const b = document.createElement('button');
    b.textContent = o.t;
    b.onclick = () => { S.flags[o.k] = true; ending(); };
    btns.appendChild(b);
  });
}

function ending() {
  const c = S.clues.length, a = S.rel.ayou, q = S.rel.qiyie, w = S.rel.wenjue;
  let id = 'wrong';

  const lifeEchoes =
    (S.acted.ayou >= 2 ? 1 : 0) +
    (S.acted.bridge >= 1 ? 1 : 0) +
    (S.acted.bridge >= 2 ? 1 : 0) +
    (S.acted.library >= 2 ? 1 : 0) +
    (S.acted.classroom >= 3 ? 1 : 0);

  const canSeven = lifeEchoes >= 3 && a >= 0 && q >= 0 && w >= 0 && c >= 5;

  if (c < 4 || S.flags.memory_lost) {
    id = S.flags.truth_repair ? 'memory' : 'wrong';
  } else if (S.flags.truth_repair) {
    if (S.silence >= 8) id = 'memory';
    else if (canSeven && S.council <= 2) id = 'seven';
    else if (a >= 1 && q >= 1 && w >= 1 && c >= 7 && S.council <= 2) id = 'repair';
    else if (S.council >= 3) id = 'council';
    else id = 'memory';
  } else if (S.flags.truth_public) {
    id = S.council >= 4 ? 'council' : (w >= 1 && S.council >= 2 ? 'tower' : 'council');
  } else if (S.flags.truth_hide) {
    id = (q >= 1 && S.council <= 2) ? 'silent' : 'alone';
  } else if (S.flags.truth_trade) {
    id = 'market';
  }

  const E = {
    seven: ['七音重聚',
      '你站在井边，没有交出任何东西。\n你想起宿舍里阿柚转铃铛的声音，想起桥边的风，想起赛跑时她笑得最大声，想起闻玦把笔记推过来时说的那句“顺手”。\n第七回声自己补全了。它缺的那一段，不是秘密，是活人的声音。\n裂痕慢慢合上。岑澜没有吹骨笛。黎伯没有写字。阿柚还在转铃铛。\n学园不是靠秘密守住的，是靠人声撑住的。'],
    repair: ['共鸣修复',
      '三人的回响叠在一起。阿柚的铃、闻玦的半段回声、栖夜手里的第七段，在井口织成一道很薄的光。裂痕没有消失，但它不再扩大。\n岑澜第一次在学生面前吹响骨笛。黎伯站在最后面，纸上写了一行字：“这次，我没有躲。”\n第七回声没有消失，它变成了学园新的守夜声。'],
    council: ['公开真相 · 议会接管',
      '你把证据交到议会手里。第二天，学园门口多了两座尖塔。\n岑澜交出钥匙，没有争辩。阿柚站在人群里，没有看你。\n真相公开了。但学园不再只属于学生。'],
    tower: ['议会尖塔',
      '闻玦把证据交上去。议会给了你一个位置，也给了学园一道锁。\n你走进尖塔那天，闻玦没有来送你。\n你告诉自己，这是最稳妥的选择。'],
    silent: ['沉默守井',
      '栖夜把第七回声放回井里。裂痕没有扩大，但也没有愈合。\n你们约定，毕业前谁也不提。阿柚照常聊八卦，闻玦照常核对目录。\n只有你知道，井下面还有一段声音在等。'],
    market: ['市集流亡',
      '退潮那晚，你和阿柚带着流星铃离开学园。市集收留了你们，也收走了回声。\n阿柚说：“至少这里没人问我们是谁。”\n你没告诉她，你偶尔还会听见井里的歌声。'],
    alone: ['守井人栖夜',
      '栖夜独自走进井里。裂痕合上了，但他没有再出来。\n黎伯在井边坐了一整夜。岑澜没有吹骨笛。\n阿柚问你：“他还会回来吗？”\n你说：“我不知道。”'],
    wrong: ['错误指认',
      '你掌握的东西还不够。指认在争论中变成了互相怀疑，裂痕扩大，学园被迫封校。\n议会接管那天，雾廊的歌声停了。\n没有人知道第七回声去了哪里。'],
    memory: ['记忆代价',
      '裂痕合上了。你修好了回声，却想不起是谁在井边拉住你的手。\n阿柚来看你，你觉得她眼熟，但叫不出名字。\n她说：“没关系，我记得就行。”'],
  };

  const e = E[id];
  main.innerHTML = `<h2>${e[0]}</h2>` +
    e[1].split('\n').map(p => `<p>${p}</p>`).join('') +
    `<p class="dim">线索 ${c} 条 · 静默值 ${S.silence} · 议会介入 ${S.council} · 生活回响 ${lifeEchoes}/5 · 阿柚 ${a} / 栖夜 ${q} / 闻玦 ${w}</p>`;
  btns.innerHTML = '';
  const b = document.createElement('button');
  b.className = 'primary';
  b.textContent = '重新开始';
  b.onclick = () => location.reload();
  btns.appendChild(b);
}

function showBook() {
  const list = S.clues.length ? S.clues.map(id => `<div class="change"><span>${id}</span></div>`).join('') : '<div class="dim">暂时没有线索。</div>';
  main.innerHTML = `<h2>线索簿</h2>${list}
    <div class="changes" style="margin-top:14px">
      ${Object.keys(S.rel).map(k => {
        const v = S.rel[k];
        const cls = v > 0 ? 'up' : v < 0 ? 'down' : 'info';
        return `<div class="change"><span>${k}</span><span class="${cls}">${v > 0 ? '+' : ''}${v}</span></div>`;
      }).join('')}
    </div>`;
  btns.innerHTML = '';
  const b = document.createElement('button');
  b.className = 'primary';
  b.textContent = '返回';
  b.onclick = renderHub;
  btns.appendChild(b);
}

updateHUD();
renderHub();
</script>
</body>
</html>
