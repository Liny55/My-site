<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>回声澜界</title>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{background:#070a10;color:#d8e2f0;font-family:"Microsoft YaHei",sans-serif;height:100vh;overflow:hidden;display:flex;align-items:center;justify-content:center}
#app{width:min(880px,94vw);height:min(640px,92vh);background:#0d131e;border:1px solid #26303f;border-radius:12px;display:flex;flex-direction:column;overflow:hidden;position:relative}
#top{padding:12px 20px;border-bottom:1px solid #1a2230;font-size:13px;color:#5a6a80;display:flex;gap:20px;flex-wrap:wrap}
#top b{color:#8ab0d0;font-weight:normal}
#main{flex:1;padding:28px 32px;overflow-y:auto}
#main h2{font-size:20px;color:#7fb0d8;margin-bottom:16px;letter-spacing:1px}
#main p{font-size:16px;line-height:1.9;color:#c8d4e4;margin-bottom:12px}
#main .dim{color:#7a8798;font-size:14px}
#btns{padding:16px 32px 24px;display:flex;flex-direction:column;gap:8px}
button{text-align:left;background:#141c29;color:#c8d4e4;border:1px solid #2a3648;padding:11px 16px;border-radius:6px;font-size:15px;cursor:pointer;font-family:inherit}
button:hover{background:#1c2738;border-color:#3e5470;color:#fff}
button.gray{color:#6d7a8c;font-size:13px;text-align:center}
#overlay{position:absolute;inset:0;background:rgba(4,6,12,.94);display:none;align-items:center;justify-content:center;padding:40px;text-align:center;cursor:pointer;z-index:10}
#overlay.show{display:flex}
#overlay .txt{font-size:20px;line-height:1.9;max-width:640px;color:#d8e2f0}
#overlay .skip{position:absolute;top:20px;right:24px;font-size:13px;color:#8a96a8;border:1px solid #2a3342;padding:6px 14px;border-radius:4px;background:transparent;cursor:pointer}
#overlay .hint{position:absolute;bottom:28px;font-size:13px;color:#5a6a80}
#toast{position:absolute;bottom:20px;left:50%;transform:translateX(-50%);background:#141c29;border:1px solid #2a3648;color:#8ab0d0;padding:8px 18px;border-radius:20px;font-size:13px;opacity:0;transition:.3s;pointer-events:none;z-index:20}
#toast.show{opacity:1}
#book{position:absolute;inset:0;background:rgba(4,6,12,.96);display:none;padding:32px;overflow-y:auto;z-index:15}
#book.show{display:block}
#book h2{font-size:19px;color:#7fb0d8;margin-bottom:14px}
#book h3{font-size:13px;color:#5a6a80;margin:16px 0 8px}
#book .item{font-size:14px;line-height:1.8;color:#c8d4e4;padding:5px 0;border-bottom:1px solid #1a2230}
#book .close{margin-top:20px;text-align:center;font-size:13px;color:#6d7a8c;cursor:pointer}
</style>
</head>
<body>
<div id="app">
  <div id="top">
    <span>场景：<b id="loc">回声礼堂</b></span>
    <span>线索：<b id="clueCount">0</b></span>
    <span>静默值：<b id="silence">0</b></span>
  </div>
  <div id="main"></div>
  <div id="btns"></div>
  <div id="overlay"><div class="txt" id="ovTxt"></div><div class="hint">点击继续</div><button class="skip" id="ovSkip">跳过</button></div>
  <div id="book"></div>
  <div id="toast"></div>
</div>
<script>
const S={flags:{},clues:[],silence:0,rel:{ayou:0,qiyie:0,wenjue:0},loc:'回声礼堂',done:{}};
const CLUES={
 old_notice:'礼堂旧告示：潮汐夜，勿留雾廊。落款二十年前。',
 ayou_song:'阿柚提到雾廊晚上有歌声，劝你别去。',
 classroom_song:'潮音教室的墙里，有被压住的歌声。',
 fog_song:'雾廊歌声：不属于任何人的回响，像在找倾听者。',
 bell_hint:'阿柚的流星铃里，藏着一小段陌生回响。',
 qiyie_hint:'栖夜藏了第七回声，他在守一个秘密。',
 libo_voice:'黎伯的失声，与静默井有关。',
 notice_full:'旧告示完整内容：二十年前潮汐夜，禁入静默井。'
};
const WORLD=[
 '澜界由浮岛与雾海组成，回声桥只在潮汐时显现。',
 '魔法靠倾听、引导、留下回响，不靠念咒。',
 '三条铁律：不能伪造；施法留痕；改变必付代价。',
 '缄默议会管理拾音人，守井人看守静默井。',
 '澜序学园地下有七口静默井，第七口封着危险回响。',
 '二十年前井边事故：一名拾音人失踪，一名守井人失声。',
 '今年第七回声失踪，雾廊夜里有歌声，议会等着接管。'
];
const $=id=>document.getElementById(id);
const main=$('main'),btns=$('btns');
function toast(t){const e=$('toast');e.textContent=t;e.classList.add('show');setTimeout(()=>e.classList.remove('show'),1800);}
function clue(id){if(!S.clues.includes(id)){S.clues.push(id);$('clueCount').textContent=S.clues.length;toast('获得线索：'+(CLUES[id]||id));}}
function flag(k,v=true){S.flags[k]=v;}
function sil(n){S.silence+=n;$('silence').textContent=S.silence;}
function rel(w,n){S.rel[w]+=n;}
function render(html){main.innerHTML=html;}
function options(arr){btns.innerHTML='';arr.forEach(o=>{const b=document.createElement('button');b.textContent=o.t;b.onclick=o.f;btns.appendChild(b);});}
function goScene(name,fn){S.loc=name;$('loc').textContent=name;fn();}
// 线索簿
document.addEventListener('keydown',e=>{if(e.key==='Tab'){e.preventDefault();toggleBook();}});
function toggleBook(){
 const b=$('book');
 if(b.classList.contains('show')){b.classList.remove('show');return;}
 b.innerHTML='<h2>线索簿</h2><h3>世界观摘要</h3>'+
  WORLD.map(t=>'<div class="item">'+t+'</div>').join('')+
  '<h3>已获线索</h3>'+(S.clues.length?S.clues.map(id=>'<div class="item">'+(CLUES[id]||id)+'</div>').join(''):'<div class="item" style="color:#5a6a80">暂时没有线索。</div>')+
  '<div class="close">关闭（Tab）</div>';
 b.classList.add('show');
 b.querySelector('.close').onclick=()=>b.classList.remove('show');
}
// 前情提要
const PRO=[
 '澜界没有大陆，只有漂浮在雾海上的岛屿。岛与岛之间，回声桥只在潮汐时显现。',
 '万物都会留下回响——声音、记忆、情绪的残片。魔法师不念咒，他们倾听回响、引导回响、留下回响。',
 '三条铁律：不能伪造回响；施法必留痕迹；改变回响，必付代价。代价可以是体力、声音、记忆，甚至一段关系。',
 '缄默议会管理所有拾音人。守井人看守静默井，封印雾海深处的旧回响。',
 '澜序学园建在最大的浮岛上。地下有七口静默井，第七口，封着一段不该被唤醒的回声。',
 '二十年前，井边出过一次事故。一名拾音人失踪，一名守井人失声。报告被封存，学园照常上课。',
 '今年开学前，井裂了。第七段回声失踪，雾廊夜里传来不属于任何人的歌声。议会盯着学园，等着接管的理由。',
 '而你，今天入学。你领到一只共鸣器，它还没响过。你还没决定：要听多少，说多少，瞒多少。'
];
let pi=0;
function showPro(){const o=$('overlay');o.classList.add('show');$('ovTxt').textContent=PRO[pi];}
$('overlay').onclick=e=>{if(e.target.id==='ovSkip')return;pi++;if(pi>=PRO.length){$('overlay').classList.remove('show');startGame();}else showPro();};
$('ovSkip').onclick=e=>{e.stopPropagation();$('overlay').classList.remove('show');startGame();};
// 场景
function hall(){
 goScene('回声礼堂',()=>{
  render('<h2>回声礼堂</h2><p>穹顶很高，雾从窗缝里渗进来。远处有个女生朝你挥手。</p><p class="dim">阿柚站在桌边，桌上摆着几只共鸣器。墙上贴着一张发黄的告示。</p>');
  const opts=[];
  opts.push({t:'和阿柚说话',f:talkAyou});
  opts.push({t:'看桌上的共鸣器',f:resonator});
  opts.push({t:'看墙上的旧告示',f:notice});
  opts.push({t:'去潮音教室',f:classroom});
  if(S.flags.classroom_done) opts.push({t:'去雾廊（夜）',f:fog});
  opts.push({t:'打开线索簿（Tab）',f:toggleBook});
  options(opts);
 });
}
function talkAyou(){
 if(S.flags.ayou_done){
  if(S.flags.heard_fog){
   render('<p>阿柚压低声音：“你昨晚去雾廊了？我就说那歌声不对劲。”</p>');
   options([
    {t:'我听见了第七段回声',f:()=>{flag('told_ayou');rel('ayou',1);clue('qiyie_hint');render('<p>阿柚愣了一下，把零食推给你：“……那你小心点。”</p>');options([{t:'回到礼堂',f:hall}]);}},
    {t:'没什么，只是风声',f:()=>{rel('ayou',-1);render('<p>阿柚看了你一眼，没再问。</p>');options([{t:'回到礼堂',f:hall}]);}}
   ]);
  }else{
   render('<p>阿柚：“礼堂的旧告示你看了吗？二十年前的日期，怪吓人的。”</p>');
   options([{t:'回到礼堂',f:hall}]);
  }
  return;
 }
 render('<p><b>阿柚</b>：“你就是新来的？我跟你讲，雾廊晚上有歌声，千万别去。”</p>');
 options([
  {t:'我想去看看',f:()=>{flag('curious');rel('ayou',-1);clue('ayou_song');flag('ayou_done');afterAyou();}},
  {t:'你陪我去吗',f:()=>{rel('ayou',1);clue('ayou_song');flag('ayou_done');afterAyou();}},
  {t:'你桌上那个铃铛是什么',f:()=>{clue('bell_hint');rel('ayou',1);flag('ayou_done');afterAyou();}}
 ]);
}
function afterAyou(){
 render('<p>阿柚：“总之，晚上别乱跑。院长最近脾气不好。”</p><p class="dim">她说完又低头摆弄那个铃铛，像在藏什么。</p>');
 options([{t:'回到礼堂',f:hall}]);
}
function resonator(){
 if(S.flags.resonator){
  render('<p>你已经领了共鸣器：<b>'+S.flags.resonator+'</b>。它还没响过。</p>');
  options([{t:'回到礼堂',f:hall}]);
  return;
 }
 render('<p>桌上摆着三只共鸣器：音叉、骨笛、石铃。你只能选一只。</p>');
 options([
  {t:'音叉',f:()=>{flag('resonator','音叉');render('<p>你拿起音叉。它很轻，像在等一个声音。</p>');options([{t:'回到礼堂',f:hall}]);}},
  {t:'骨笛',f:()=>{flag('resonator','骨笛');render('<p>你拿起骨笛。院长也有一只，但她从不吹。</p>');options([{t:'回到礼堂',f:hall}]);}},
  {t:'石铃',f:()=>{flag('resonator','石铃');render('<p>你拿起石铃。铃舌是空的，暂时不会响。</p>');options([{t:'回到礼堂',f:hall}]);}}
 ]);
}
function notice(){
 if(S.flags.notice){
  render('<p>旧告示：潮汐夜，勿留雾廊。落款日期是二十年前。</p>');
  options([{t:'回到礼堂',f:hall}]);
  return;
 }
 render('<p>墙上贴着一张发黄的告示，字迹被雾浸过，只能看清一半。</p><p>“潮汐夜，勿留雾廊……”落款日期是二十年前。</p>');
 options([
  {t:'记下来',f:()=>{flag('notice');clue('old_notice');render('<p>你把告示抄进线索簿。</p>');options([{t:'回到礼堂',f:hall}]);}},
  {t:'先不管',f:hall}
 ]);
}
function classroom(){
 goScene('潮音教室',()=>{
  render('<h2>潮音教室</h2><p>黎伯在石板上写字，笔停了很久。墙里好像有极轻的歌声。</p>');
  const opts=[];
  opts.push({t:'靠近墙听',f:puzzleClass});
  if(!S.flags.libo_talked){opts.push({t:'课后问黎伯',f:liboTalk});}
  opts.push({t:'回礼堂',f:hall});
  options(opts);
 });
}
function liboTalk(){
 flag('libo_talked');
 render('<p>黎伯在纸上写：“有些声音，听一次就够了。”他指了指自己的喉咙，又指了指地下。</p>');
 clue('libo_voice');
 options([{t:'回教室',f:classroom}]);
}
function puzzleClass(){
 render('<p>你靠近墙。要听哪一段回响？</p>');
 options([
  {t:'粉笔划过石板的声音',f:()=>wrongListen()},
  {t:'窗外雾海的风声',f:()=>wrongListen()},
  {t:'墙内极轻的歌声，像被什么压住',f:()=>{clue('classroom_song');flag('classroom_done');render('<p>你听见了。墙里有歌声，被什么压着。它好像在说：“第七段……”</p>');options([{t:'回礼堂',f:hall}]);}},
  {t:'后排同学翻书的声音',f:()=>wrongListen()}
 ]);
}
function wrongListen(){
 sil(1);
 render('<p>你听错了。耳边只剩杂音。静默值上升了。</p>');
 options([{t:'再试一次',f:puzzleClass},{t:'回礼堂',f:hall}]);
}
function fog(){
 goScene('雾廊',()=>{
  if(S.flags.finale){
   render('<h2>雾廊</h2><p>雾廊已经安静了。故事结束了。</p>');
   options([{t:'查看结局',f:ending}]);
   return;
  }
  render('<h2>雾廊</h2><p>灯一盏盏灭了。墙里的歌声越来越近，像有人在很远的地方哼一首没写完的曲子。</p>');
  options([{t:'循声而去',f:fogListen},{t:'先回礼堂',f:hall}]);
 });
}
function fogListen(){
 render('<p>歌声里有一句模糊的话：“第七段……别交给议会。”</p><p>雾里走出一个人。是栖夜。</p><p><b>栖夜</b>：“你不该来这里。”</p>');
 options([
  {t:'你听见了吗？',f:()=>{rel('qiyie',1);clue('qiyie_hint');fogTalk();}},
  {t:'我要告诉院长',f:()=>{flag('reported');rel('qiyie',-2);fogTalk();}},
  {t:'你在这里做什么？',f:()=>{rel('qiyie',-1);fogTalk();}}
 ]);
}
function fogTalk(){
 clue('fog_song');
 render('<p><b>栖夜</b>：“第七段回声不是被盗，是被藏。藏它的人，不想让议会拿到。”</p><p>他看着我，像在等我做一个决定。</p>');
 options([
  {t:'公开真相',f:()=>{flag('truth_public');ending();}},
  {t:'帮栖夜隐瞒',f:()=>{flag('truth_hide');ending();}},
  {t:'联合三人修复回声',f:()=>{flag('truth_repair');ending();}},
  {t:'和市集交易',f:()=>{flag('truth_trade');ending();}}
 ]);
}
function ending(){
 flag('finale');
 let id='wrong_accusation';
 const c=S.clues.length;
 if(c>=3){
  if(S.flags.truth_repair){id=(S.rel.ayou>=1&&S.rel.qiyie>=1&&c>=5)?'repair':'memory';}
  else if(S.flags.truth_public)id='council';
  else if(S.flags.truth_hide)id=S.rel.qiyie>=1?'silent':'alone';
  else if(S.flags.truth_trade)id='market';
 }
 const E={
  repair:['共鸣修复','三人的回响叠在一起，裂痕慢慢合拢。第七回声没有消失，它变成了学园新的守夜声。岑澜第一次在学生面前吹响骨笛。'],
  council:['公开真相 · 议会接管','真相公开了。议会第二天就进驻学园，岑澜交出钥匙。你说了真话，但学园不再只属于学生。'],
  silent:['沉默守井','栖夜把回声放回井里。裂痕没有扩大，但也没有愈合。你们约定，毕业前谁也不提。'],
  market:['市集流亡','退潮那晚，你和阿柚带着流星铃离开学园。市集收留了你们，也收走了回声。'],
  alone:['守井人栖夜','栖夜独自走进井里。裂痕合上了，但他没有再出来。'],
  wrong_accusation:['错误指认','你掌握的东西还不够。指认在争论中变成了互相怀疑，裂痕扩大，学园被迫封校。'],
  memory:['记忆代价','裂痕合上了。你修好了回声，却想不起是谁在井边拉住你的手。']
 };
 const e=E[id];
 render('<h2>'+e[0]+'</h2><p>'+e[1]+'</p><p class="dim">你共获得 '+c+' 条线索，静默值 '+S.silence+'。</p>');
 options([{t:'重新开始',f:()=>location.reload()}]);
}
function startGame(){
 if(!S.flags.resonator){toast('提示：先去桌上选共鸣器');}
 hall();
}
// 启动
showPro();
</script>
</body>
</html>
