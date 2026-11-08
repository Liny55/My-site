function go(){
  const fn = SCENES[S.loc];
  if(!fn) return goto('hall');
  const d = fn();
  render(d.title, d.paras, d.actions);
}

const SCENES = {
  hall(){
    const acts = [];
    acts.push({t:'和阿柚说话', f:talkAyou});
    acts.push({t:'看桌上的共鸣器', f:resonator});
    acts.push({t:'看墙上的旧告示', f:notice});
    if(!S.flags.dorm_done && S.night >= 1) acts.push({t:'回宿舍夜谈', f:dormTalk, c:'special'});
    if(S.flags.dorm_done && !S.flags.bell_done) acts.push({t:'看看阿柚的流星铃', f:starBell, c:'special'});
    if(!S.flags.crush_done && S.flags.dorm_done) acts.push({t:'查看匿名纸条', f:anonNote, c:'special'});
    acts.push({t:'去潮音教室', f:()=>goto('classroom')});
    if(S.flags.classroom_done) acts.push({t:'去雾廊', f:()=>goto('fog')});
    if(S.flags.fog_night_done) acts.push({t:'去回声桥', f:()=>goto('bridge')});
    if(S.flags.fog_night_done) acts.push({t:'去旧物图书馆', f:()=>goto('library')});
    acts.push({t:'打开线索簿（Tab）', f:toggleBook, c:'gray'});
    return {
      title:'回声礼堂',
      paras:['穹顶很高，雾从窗缝里渗进来。远处有个女生朝你挥手。',
        '<span class="dim">阿柚站在桌边，桌上摆着几只共鸣器。墙上贴着一张发黄的告示。走廊尽头是潮音教室的方向。</span>'],
      actions:acts
    };
  },
  classroom(){
    if(!S.flags.class_started){ flag('class_started'); S.night = Math.max(S.night,1); updatePhase(); }
    const acts = [];
    if(!S.flags.classroom_done) acts.push({t:'靠近墙听', f:puzzleClass});
    if(!S.flags.libo_talked) acts.push({t:'课后问黎伯', f:liboTalk});
    acts.push({t:'找闻玦说话', f:wenjue});
    if(S.flags.libo_talked && !S.flags.libo_resolved) acts.push({t:'帮黎伯整理旧物', f:liboHelp, c:'special'});
    acts.push({t:'回礼堂', f:()=>goto('hall')});
    return {title:'潮音教室',
      paras:['黎伯在石板上写字，笔停了很久。墙里好像有极轻的歌声。',
        '<span class="dim">闻玦坐在角落，反复核对一本旧目录。</span>'],
      actions:acts};
  },
  library(){
    const acts = [];
    if(!S.flags.catalog_done) acts.push({t:'翻查图书馆目录', f:catalogCheck});
    if(!S.flags.lib_crush) acts.push({t:'在角落找座位', f:libraryCorner, c:'special'});
    acts.push({t:'回礼堂', f:()=>goto('hall')});
    return {title:'旧物图书馆',
      paras:['书架高到看不见顶。旧物在架子上轻轻回响，像在低声说话。',
        '<span class="dim">这里很安静，适合查一些不想被别人知道的事。</span>'],
      actions:acts};
  },
  fog(){
    if(S.flags.finale_done) return {title:'雾廊',paras:['雾廊已经安静了。故事结束了。'],actions:[{t:'重新开始',f:()=>location.reload()}]};
    const acts = [];
    if(!S.flags.fog_night_done){
      if(S.night < 2) acts.push({t:'（天还没黑）等到夜晚', f:()=>{S.night=2;updatePhase();go();}});
      else acts.push({t:'循声而去', f:fogNight});
    } else {
      if(S.flags.heard_fog_song && !S.flags.notice_done) acts.push({t:'查看雾廊尽头的半张告示', f:tornNotice, c:'special'});
      if(!S.flags.rumor_done) acts.push({t:'听墙角的八卦', f:fogRumor, c:'special'});
      if(!S.flags.sneak_done) acts.push({t:'用静默术潜行', f:sneak, c:'special'});
      acts.push({t:'走到静默井入口（终章）', f:finale, c:'warn'});
    }
    acts.push({t:'回礼堂', f:()=>goto('hall')});
    return {title:'雾廊',
      paras:['灯一盏盏灭了。墙里的歌声越来越近，像有人在很远的地方哼一首没写完的曲子。',
        '<span class="dim">雾廊很长，尽头通向静默井。两侧的墙上有旧痕。</span>'],
      actions:acts};
  },
  bridge(){
    const acts = [];
    if(!S.flags.tide_done) acts.push({t:'在桥边坐下看雾海', f:tideView, c:'special'});
    if(!S.flags.race_done) acts.push({t:'和朋友赛跑', f:race, c:'special'});
    if(!S.flags.bonfire_done) acts.push({t:'参加雾海篝火夜', f:bonfire, c:'special'});
    if(S.night === 1 || S.flags.tide_low) acts.push({t:'去外岛市集', f:()=>goto('island')});
    acts.push({t:'回礼堂', f:()=>goto('hall')});
    return {title:'回声桥',
      paras:['雾海退下去，桥边的风带着咸味。远处有一座小岛的轮廓。',
        '<span class="dim">桥很窄，两侧是雾。有人喜欢坐在这里吹风。</span>'],
      actions:acts};
  },
  island(){
    const acts = [];
    if(!S.flags.market_done) acts.push({t:'走向市集门', f:market, c:'special'});
    else acts.push({t:'看看市集', f:()=>render('外岛',['市集门半开着，商人正在打盹。'],[{t:'回桥',f:()=>goto('bridge')}])});
    acts.push({t:'回桥', f:()=>goto('bridge')});
    return {title:'外岛',
      paras:['退潮露出的浅滩上，一扇旧门立在雾里。门楣上挂着褪色的布幡。',
        '<span class="dim">门只在退潮时开。里面的人用记忆和承诺做买卖。</span>'],
      actions:acts};
  },
};

function talkAyou(){
  if(S.flags.ayou_done){
    if(S.flags.heard_fog_song){
      render('阿柚',['<span class="npc">阿柚</span>压低声音：“你昨晚去雾廊了？我就说那歌声不对劲。”'],[
        {t:'我听见了第七段回声', f:()=>{flag('told_ayou');rel('ayou','trust',1);clue('qiyie_hint');render('阿柚',['阿柚愣了一下，把零食推给你：“……那你小心点。”'],[{t:'回到礼堂',f:()=>goto('hall')}]);}},
        {t:'没什么，只是风声', f:()=>{rel('ayou','doubt',1);render('阿柚',['阿柚看了你一眼，没再问。'],[{t:'回到礼堂',f:()=>goto('hall')}]);}}
      ]);
    }else{
      render('阿柚',['<span class="npc">阿柚</span>：“礼堂的旧告示你看了吗？二十年前的日期，怪吓人的。”'],[{t:'回到礼堂',f:()=>goto('hall')}]);
    }
    return;
  }
  render('阿柚',['<span class="npc">阿柚</span>：“你就是新来的？我跟你讲，雾廊晚上有歌声，千万别去。”','<span class="me">你</span>：（她的眼睛很亮，像在说一件很刺激的事。）'],[
    {t:'我想去看看', f:()=>{flag('curious');rel('ayou','doubt',1);clue('ayou_song');flag('ayou_done');afterAyou();}},
    {t:'你陪我去吗', f:()=>{rel('ayou','trust',1);clue('ayou_song');flag('ayou_done');afterAyou();}},
    {t:'你桌上那个铃铛是什么', f:()=>{clue('bell_hint');rel('ayou','debt',1);flag('ayou_done');afterAyou();}}
  ]);
}
function afterAyou(){
  render('阿柚',['阿柚：“总之，晚上别乱跑。院长最近脾气不好。”','<span class="dim">她说完又低头摆弄那个铃铛，像在藏什么。</span>'],[{t:'回到礼堂',f:()=>goto('hall')}]);
}
function resonator(){
  if(S.flags.resonator){render('共鸣器',['你已经领了共鸣器：<b>'+S.flags.resonator+'</b>。它还没响过。'],[{t:'回到礼堂',f:()=>goto('hall')}]);return;}
  render('共鸣器',['桌上摆着三只共鸣器：音叉、骨笛、石铃。你只能选一只。'],[
    {t:'音叉（听感+）',f:()=>{flag('resonator','音叉');flag('attr_listen');render('共鸣器',['你拿起音叉。它很轻，像在等一个声音。'],[{t:'回到礼堂',f:()=>goto('hall')}]);}},
    {t:'骨笛（共情+）',f:()=>{flag('resonator','骨笛');flag('attr_empathy');render('共鸣器',['你拿起骨笛。院长也有一只，但她从不吹。'],[{t:'回到礼堂',f:()=>goto('hall')}]);}},
    {t:'石铃（缄默+）',f:()=>{flag('resonator','石铃');flag('attr_silence');render('共鸣器',['你拿起石铃。铃舌是空的，暂时不会响。'],[{t:'回到礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function notice(){
  if(S.flags.notice){render('旧告示',['旧告示：潮汐夜，勿留雾廊。落款日期是二十年前。'],[{t:'回到礼堂',f:()=>goto('hall')}]);return;}
  render('旧告示',['墙上贴着一张发黄的告示，字迹被雾浸过，只能看清一半。','“潮汐夜，勿留雾廊……”落款日期是二十年前。'],[
    {t:'记下来',f:()=>{flag('notice');clue('old_notice');render('旧告示',['你把告示抄进线索簿。'],[{t:'回到礼堂',f:()=>goto('hall')}]);}},
    {t:'先不管',f:()=>goto('hall')}
  ]);
}
function dormTalk(){
  flag('dorm_done');
  render('宿舍夜谈',['<span class="npc">阿柚</span>：“熄灯了还醒着？来聊会儿。”','<span class="dim">她把零食推过来。</span>'],[
    {t:'聊八卦',f:()=>{flag('dorm_gossip');clue('rumor_qiyie');rel('ayou','trust',1);render('宿舍夜谈',['阿柚压低声音：“有人说栖夜半夜在雾廊等人。你说，他是不是在谈恋爱？”','<span class="dim">她说完自己先笑了，但笑得很轻。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'聊家里',f:()=>{flag('dorm_family');rel('ayou','trust',1);render('宿舍夜谈',['阿柚：“我家在回声市集，开旧物铺的。别跟别人说啊。”','<span class="dim">她说完把铃铛转得很快。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'聊害怕的事',f:()=>{flag('dorm_fear');rel('ayou','trust',2);clue('ayou_fear');render('宿舍夜谈',['阿柚沉默了一会儿：“我怕被看出来。市集出身的人，在这里总像借来的。”','<span class="dim">你第一次见她这么安静。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'聊暗恋',f:()=>{flag('dorm_crush');rel('ayou','debt',1);flag('crush_ayou');render('宿舍夜谈',['阿柚：“你有没有觉得……闻玦那种人，越冷越让人想逗他？”','<span class="dim">她说完立刻补一句：“我随便说说的。”</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function starBell(){
  render('流星铃',['<span class="npc">阿柚</span>：“别碰那个铃。它……有点怪。”','<span class="me">你</span>：（铃身有一道旧裂纹，里面像压着声音。）'],[
    {t:'我帮你修',f:()=>{flag('repair_started');rel('ayou','trust',1);repairPuzzle();}},
    {t:'它哪来的？',f:()=>{flag('bell_origin');rel('ayou','doubt',1);clue('bell_hint');render('流星铃',['阿柚：“市集捡的。别问了。”','<span class="dim">她把铃往身后藏了藏。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'先不动它',f:()=>{flag('bell_done');render('流星铃',['你放下铃铛。它没有响。'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function repairPuzzle(){
  render('旧物修复：流星铃',['铃舌卡住了。选正确的零件组合。'],[
    {t:'细铜丝 + 旧铃舌',f:()=>{flag('bell_done');flag('bell_repaired');clue('bell_echo');render('流星铃',['铃修好了。它没有响，但你知道里面多了一小段不属于市集的声音。','<span class="npc">阿柚</span>：“……你听到了？”'],[
      {t:'交还给她',f:()=>{rel('ayou','trust',2);flag('bell_returned');render('流星铃',['阿柚把铃收进怀里，很久没说话。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
      {t:'先留着',f:()=>{rel('ayou','doubt',1);flag('bell_kept');render('流星铃',['阿柚看着你，没伸手。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
      {t:'交给院长',f:()=>{rel('ayou','doubt',3);flag('bell_reported');council(1);render('流星铃',['阿柚退后一步：“你拿去啊。”'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
    ]);}},
    {t:'市集胶 + 碎玻璃',f:()=>{flag('bell_done');flag('bell_broken');rel('ayou','doubt',1);render('流星铃',['铃裂得更厉害了。阿柚没说话。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'回响石片 + 铜丝',f:()=>{flag('bell_done');flag('bell_broken');rel('ayou','doubt',1);sil(1);render('流星铃',['铃响了一声，然后哑了。阿柚把铃抢了回去。'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function anonNote(){
  flag('crush_done');
  render('匿名纸条',['你在桌上发现一张折得很小的纸条。字迹很急，只写了一句：“今晚雾廊，别一个人去。”','<span class="dim">没有署名。</span>'],[
    {t:'回一张纸条：你是谁？',f:()=>{flag('note_replied');rel('ayou','awkward',1);render('匿名纸条',['第二天，桌上多了回信：“你猜。”','<span class="dim">字迹和昨天不一样，也许是故意的。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'先收起来',f:()=>{flag('note_kept');render('匿名纸条',['你把纸条收进口袋。它很轻，但让你一整天都不太专心。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'拿去问阿柚',f:()=>{rel('ayou','doubt',1);flag('note_asked');render('匿名纸条',['阿柚：“不是我写的。”她回答得很快，快得有点不自然。','<span class="dim">你不知道该不该信。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function liboTalk(){
  flag('libo_talked');
  render('黎伯',['黎伯在纸上写：“有些声音，听一次就够了。”','<span class="dim">他指了指自己的喉咙，又指了指地下。</span>'],[
    {t:'记下这条线索',f:()=>{clue('libo_voice');render('黎伯',['你把这句话抄进线索簿。'],[{t:'回教室',f:()=>goto('classroom')}]);}},
    {t:'先回教室',f:()=>goto('classroom')}
  ]);
}
function liboHelp(){
  render('黎伯的旧物',['旧物堆里有一段很旧的录音。声音像黎伯，但更年轻。','<span class="echo">录音：“井下面有东西在回应我。我不该叫它的名字。”</span>'],[
    {t:'当面问他',f:()=>{rel('libo','trust',1);flag('libo_confronted');render('黎伯',['黎伯在纸上写：“有些声音，是我自己弄丢的。不是别人拿走的。”','<span class="dim">他把纸折好，推给你。</span>'],[{t:'记下线索',f:()=>{clue('libo_voice');clue('old_recording');flag('libo_resolved');flag('libo_witness');goto('classroom');}}]);}},
    {t:'告诉岑澜',f:()=>{council(1);flag('libo_resolved');rel('cenlan','trust',1);render('黎伯',['岑澜听完，只说了一句：“我知道了。”'],[{t:'回教室',f:()=>goto('classroom')}]);}},
    {t:'自己留着',f:()=>{flag('libo_resolved');clue('libo_voice');clue('old_recording');render('黎伯',['你把录音收进口袋。黎伯没有察觉。'],[{t:'回教室',f:()=>goto('classroom')}]);}}
  ]);
}
function puzzleClass(){
  render('倾听拼图：课堂异响',['黎伯停下笔。哪一段才是教室里真正的声音来源？'],[
    {t:'粉笔划过石板的声音',f:()=>wrongListen()},
    {t:'窗外雾海的风声',f:()=>wrongListen()},
    {t:'墙内极轻的歌声，像被什么压住',f:()=>{clue('classroom_song');flag('classroom_done');flag('heard_classroom_song');render('倾听拼图',['你听见了。墙里有歌声，被什么压着。','<span class="echo">它好像在说：“第七段……”</span>'],[{t:'回教室',f:()=>goto('classroom')}]);}},
    {t:'后排同学翻书的声音',f:()=>wrongListen()}
  ]);
}
function wrongListen(){sil(1);render('倾听拼图',['你听错了。耳边只剩杂音。静默值上升了。'],[{t:'再试一次',f:puzzleClass},{t:'先回去',f:()=>goto('classroom')}]);}
function wenjue(){
  if(S.flags.wenjue_done){render('闻玦',['<span class="npc">闻玦</span>：“目录的事，你别再查了。”'],[{t:'回教室',f:()=>goto('classroom')}]);return;}
  render('闻玦',['<span class="npc">闻玦</span>：“你翻过目录了。”','<span class="me">你</span>：（缺页记录被改过。）'],[
    {t:'直接问他',f:()=>{rel('wenjue','doubt',1);wenjueAfter();}},
    {t:'先说我查到的',f:()=>{rel('wenjue','trust',1);flag('wenjue_shared');wenjueAfter();}},
    {t:'提他哥哥',f:()=>{rel('wenjue','doubt',2);flag('wenjue_brother');wenjueAfter();}}
  ]);
}
function wenjueAfter(){
  if(S.rel.wenjue.trust>=1||S.flags.wenjue_shared){
    render('闻玦',['闻玦沉默很久：“我哥留下半段回声。里面只有一句话。”','<span class="dim">他把一张旧纸推到你面前。</span>'],[
      {t:'我们一起查',f:()=>{rel('wenjue','trust',2);flag('wenjue_ally');flag('wenjue_done');clue('wenjue_echo');clue('catalog');render('闻玦',['闻玦：“不是被盗，是被藏。”'],[{t:'回教室',f:()=>goto('classroom')}]);}},
      {t:'你为什么不自己查？',f:()=>{flag('wenjue_done');clue('wenjue_echo');rel('wenjue','debt',1);render('闻玦',['闻玦：“因为我不确定，查下去会查到谁。”'],[{t:'回教室',f:()=>goto('classroom')}]);}}
    ]);
  }else{flag('wenjue_done');render('闻玦',['闻玦合上目录：“我没什么好说的。”'],[{t:'回教室',f:()=>goto('classroom')}]);}
}
function catalogCheck(){flag('catalog_done');clue('catalog');sil(1);render('图书馆目录',['你翻开目录。缺页记录被改过，改得很小心，但还是留下了痕。','<span class="dim">旁边有一行铅笔小字：“问闻玦。”</span>'],[{t:'回图书馆',f:()=>goto('library')}]);}
function libraryCorner(){
  flag('lib_crush');
  render('图书馆角落',['你在角落坐下。过了一会儿，闻玦把一叠整理好的笔记放在你桌上，没说话。','<span class="dim">笔记是新的，还有墨迹。他大概是刚抄完。</span>'],[
    {t:'叫住他道谢',f:()=>{rel('wenjue','trust',1);rel('wenjue','awkward',1);render('图书馆角落',['闻玦停了一下：“顺手。”'],[{t:'回图书馆',f:()=>goto('library')}]);}},
    {t:'什么都不说',f:()=>{rel('wenjue','debt',1);render('图书馆角落',['闻玦走远了。笔记还留着。'],[{t:'回图书馆',f:()=>goto('library')}]);}}
  ]);
}
function fogNight(){
  flag('fog_night_done');
  render('雾廊夜遇',['<span class="dim">歌声里有一句模糊的话：“第七段……别交给议会。”</span>','雾里走出一个人。是栖夜。','<span class="npc">栖夜</span>：“你不该来这里。”'],[
    {t:'你听见了吗？',f:()=>{rel('qiyie','trust',1);clue('qiyie_hint');clue('fog_song');flag('heard_fog_song');fogTalk();}},
    {t:'我要告诉院长',f:()=>{rel('qiyie','doubt',2);flag('reported');clue('fog_song');flag('heard_fog_song');council(1);fogTalk();}},
    {t:'你在这里做什么？',f:()=>{rel('qiyie','doubt',1);clue('fog_song');flag('heard_fog_song');fogTalk();}}
  ]);
}
function fogTalk(){
  render('栖夜',['<span class="npc">栖夜</span>：“第七段回声不是被盗，是被藏。藏它的人，不想让议会拿到。”','<span class="me">你</span>：（他看着我，像在等我做一个决定。）'],[
    {t:'我帮你保密',f:()=>{rel('qiyie','trust',1);flag('qiyie_ally');render('栖夜',['栖夜看了你很久，点了点头。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'我需要想想',f:()=>{render('栖夜',['栖夜：“想清楚再来。”他转身走进雾里。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'我要上报',f:()=>{rel('qiyie','doubt',2);flag('reported');council(1);render('栖夜',['栖夜没有拦你。他只是站在原地，看着你走。'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function tornNotice(){
  flag('notice_done');
  render('旧告示',['半张告示贴在雾廊尽头。缺失的字慢慢显出来。','<span class="me">你</span>：（日期是二十年前。潮汐夜，禁入静默井。）'],[
    {t:'上报院长',f:()=>{rel('cenlan','trust',1);flag('council');council(1);clue('notice_full');render('旧告示',['岑澜看完，只说：“我知道了。”'],[{t:'回雾廊',f:()=>goto('fog')}]);}},
    {t:'先找栖夜',f:()=>{rel('qiyie','trust',1);flag('qiyie_ally');clue('notice_full');render('旧告示',['栖夜看完，低声说：“你比我想的会查。”'],[{t:'回雾廊',f:()=>goto('fog')}]);}},
    {t:'自己留着',f:()=>{clue('notice_full');render('旧告示',['你把告示折好，收进口袋。'],[{t:'回雾廊',f:()=>goto('fog')}]);}}
  ]);
}
function fogRumor(){
  flag('rumor_done');
  render('墙角的八卦',['两个高年级学生躲在雾廊拐角说话。你假装路过。','<span class="dim">“听说岑澜和黎伯年轻时是一对？”</span>','<span class="dim">“别瞎说，是因为那件事才闹翻的。”</span>'],[
    {t:'凑近听',f:()=>{clue('rumor_cenlan');flag('rumor_heard');render('墙角的八卦',['“那件事”到底是什么，没人往下说。','<span class="dim">他们看见你，立刻散了。</span>'],[{t:'回雾廊',f:()=>goto('fog')}]);}},
    {t:'直接问他们',f:()=>{rel('cenlan','doubt',1);flag('rumor_asked');render('墙角的八卦',['“没什么没什么。”他们跑得很快。','<span class="dim">看来这里的传闻不太欢迎被追问。</span>'],[{t:'回雾廊',f:()=>goto('fog')}]);}},
    {t:'走开',f:()=>{flag('rumor_ignored');goto('fog');}}
  ]);
}
function sneak(){
  flag('sneak_done');sil(2);rel('cenlan','doubt',1);
  render('静默术潜行',['你按住自己的回响，慢慢往雾廊深处走。脚步声消失了，但脑子开始发空。','<span class="dim">你在墙上发现了一道很浅的划痕，像是有人在这里等过谁。</span>'],[
    {t:'记下划痕',f:()=>{clue('notice_full');render('静默术潜行',['你把划痕抄进线索簿。静默值又高了一点。'],[{t:'回雾廊',f:()=>goto('fog')}]);}},
    {t:'赶紧退回去',f:()=>goto('fog')}
  ]);
}
function tideView(){
  flag('tide_done');
  render('雾海观潮',['雾海退下去，桥边的风带着咸味。有人和你一起坐下。'],[
    {t:'聊梦想',f:()=>{rel('ayou','trust',1);render('雾海观潮',['阿柚：“我想开一家比我家更大的旧物铺。到时候你来，我给你打折。”'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'聊家里',f:()=>{rel('qiyie','trust',1);flag('qiyie_soft');clue('qiyie_soft');render('雾海观潮',['栖夜：“我没有家。学园收留我的时候，黎伯说，守井人不需要家。”','<span class="dim">他顿了顿：“但有时候，我会觉得桥边比井边暖和。”</span>'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'问对方怕不怕毕业',f:()=>{rel('wenjue','trust',1);render('雾海观潮',['闻玦：“怕。毕业意味着要回议会，回那个不许提我哥的地方。”'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'什么都不说',f:()=>{flag('tide_silent');render('雾海观潮',['你们坐了很久，谁也没说话。风把雾吹开又合上。'],[{t:'回桥',f:()=>goto('bridge')}]);}}
  ]);
}
function race(){
  flag('race_done');
  render('回声桥赛跑',['有人提议赛跑。桥很窄，风很大，但没人想先认输。'],[
    {t:'全力跑',f:()=>{flag('race_won');rel('ayou','trust',1);render('回声桥赛跑',['你赢了。阿柚喘着气笑：“你跑得比看起来快。”'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'留着力气',f:()=>{flag('race_slow');rel('qiyie','trust',1);render('回声桥赛跑',['你落在后面。跑到一半，栖夜伸手拉了你一把。','<span class="dim">他什么都没说。</span>'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'中途停下',f:()=>{flag('race_stopped');rel('wenjue','doubt',1);render('回声桥赛跑',['你停下来。闻玦回头看了一眼，继续跑。'],[{t:'回桥',f:()=>goto('bridge')}]);}}
  ]);
}
function bonfire(){
  flag('bonfire_done');council(1);
  render('雾海篝火夜',['全班围在火边。有人唱歌，有人讲鬼故事，火光把每个人的脸照得很近。','<span class="dim">有人提到“第七段”，又很快停住。</span>'],[
    {t:'唱歌',f:()=>{flag('bonfire_sing');rel('ayou','trust',1);render('雾海篝火夜',['你跑调了。阿柚笑得最大声，但最后也跟着唱。'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'讲鬼故事',f:()=>{flag('bonfire_scare');rel('qiyie','doubt',1);render('雾海篝火夜',['你讲了一个关于静默井的故事。火光暗了一瞬，没人说话。','<span class="dim">栖夜看着火，很久没有动。</span>'],[{t:'回礼堂',f:()=>goto('hall')}]);}},
    {t:'安静坐着',f:()=>{flag('bonfire_silent');rel('wenjue','trust',1);render('雾海篝火夜',['你没说话。闻玦也没说。火光照着你们两个人的影子。'],[{t:'回礼堂',f:()=>goto('hall')}]);}}
  ]);
}
function market(){
  if(S.night===2&&!S.flags.tide_low){render('市集门',['门关着。退潮已经过了。'],[{t:'回桥',f:()=>goto('bridge')}]);return;}
  render('市集商人',['<span class="npc">市集商人</span>：“退潮才开门。你带了什么来换？”','<span class="dim">他面前摆着旧回声、旧名字、旧承诺。</span>'],[
    {t:'用一段记忆换线索',f:()=>{flag('market_done');flag('market_memory');sil(2);clue('market_echo');render('市集商人',['你走出市集时，忘了自己刚才想说什么。手里多了一小段回声。'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'用承诺换',f:()=>{flag('market_done');flag('market_promise');rel('ayou','debt',1);clue('market_echo');render('市集商人',['商人：“毕业前，帮我带一个人来。”','<span class="dim">你点了头，才想起没问是谁。</span>'],[{t:'回桥',f:()=>goto('bridge')}]);}},
    {t:'不换',f:()=>{flag('market_done');flag('market_refused');render('市集商人',['商人耸耸肩：“那就等你改主意。”'],[{t:'回桥',f:()=>goto('bridge')}]);}}
  ]);
}
function finale(){
  if(S.clues.length<3){render('静默井入口',['井口低鸣。你觉得自己知道得还不够。','<span class="dim">至少再找几条线索，才能面对井下的东西。</span>'],[{t:'回雾廊',f:()=>goto('fog')}]);return;}
  if(S.flags.finale_done){ending();return;}
  render('终章',['静默井的裂痕在夜里发光。栖夜站在井边，手里攥着那页失踪的回声。','<span class="npc">栖夜</span>：“你们不该跟来。”','<span class="me">你</span>：（现在要说什么？）'],[
    {t:'出示线索',f:()=>{render('终章',['你把线索一条条摆出来。井边的雾随着每句话变薄。','<span class="npc">栖夜</span>：“……你查得比我想的深。”'],[{t:'进入最后的选择',f:finalChoice}]);}},
    {t:'问栖夜为什么藏',f:()=>{rel('qiyie','trust',1);render('终章',['栖夜：“因为议会拿到它，学园就不再是学园了。”'],[{t:'进入最后的选择',f:finalChoice}]);}},
    {t:'看其他人的反应',f:()=>{render('终章',['阿柚站在你身后，闻玦站在门口。没有人先开口。'],[{t:'进入最后的选择',f:finalChoice}]);}}
  ]);
}
function finalChoice(){
  render('终章',['裂痕在扩大。你必须决定第七回声的命运。','<span class="me">你</span>：（最后的选择。）'],[
    {t:'公开真相',f:()=>{flag('truth_public');flag('finale_done');ending();}},
    {t:'帮栖夜隐瞒',f:()=>{flag('truth_hide');flag('finale_done');ending();}},
    {t:'联合三人修复回声',f:()=>{flag('truth_repair');flag('finale_done');ending();}},
    {t:'和市集交易',f:()=>{flag('truth_trade');flag('finale_done');ending();}}
  ]);
}
function ending(){
  const c=S.clues.length,tA=S.rel.ayou.trust,tW=S.rel.wenjue.trust,tQ=S.rel.qiyie.trust;
  let id='wrong_accusation';
  if(c<5||S.memoryLost){id=(S.flags.truth_repair&&S.memoryLost)?'memory':'wrong_accusation';}
  else if(S.flags.truth_repair){
    if(S.silence>=8&&S.memoryLost)id='memory';
    else if(tA>=1&&tW>=1&&tQ>=1&&c>=8&&S.council<=2)id='repair';
    else if(S.council>=3)id='council';
    else id='memory';
  }else if(S.flags.truth_public){id=S.council>=4?'council':(tW>=1&&S.council>=2?'tower':'council');}
  else if(S.flags.truth_hide){id=(tQ>=1&&S.council<=2)?'silent':'alone';}
  else if(S.flags.truth_trade){id='market';}
  S.ending=id;
  const E={
    repair:['共鸣修复','三人的回响叠在一起。阿柚的铃、闻玦的半段回声、栖夜手里的第七段，在井口织成一道很薄的光。裂痕没有消失，但它不再扩大。\n岑澜第一次在学生面前吹响骨笛。黎伯站在最后面，纸上写了一行字：“这次，我没有躲。”\n第七回声没有消失，它变成了学园新的守夜声。'],
    council:['公开真相 · 议会接管','你把证据交到议会手里。第二天，学园门口多了两座尖塔。\n岑澜交出钥匙，没有争辩。阿柚站在人群里，没有看你。\n真相公开了。但学园不再只属于学生。'],
    tower:['议会尖塔','闻玦把证据交上去。议会给了你一个位置，也给了学园一道锁。\n你走进尖塔那天，闻玦没有来送你。\n你告诉自己，这是最稳妥的选择。'],
    silent:['沉默守井','栖夜把第七回声放回井里。裂痕没有扩大，但也没有愈合。\n你们约定，毕业前谁也不提。阿柚照常聊八卦，闻玦照常核对目录。\n只有你知道，井下面还有一段声音在等。'],
    market:['市集流亡','退潮那晚，你和阿柚带着流星铃离开学园。市集收留了你们，也收走了回声。\n阿柚说：“至少这里没人问我们是谁。”\n你没告诉她，你偶尔还会听见井里的歌声。'],
    alone:['守井人栖夜','栖夜独自走进井里。裂痕合上了，但他没有再出来。\n黎伯在井边坐了一整夜。岑澜没有吹骨笛。\n阿柚问你：“他还会回来吗？”\n你说：“我不知道。”'],
    wrong_accusation:['错误指认','你掌握的东西还不够。指认在争论中变成了互相怀疑，裂痕扩大，学园被迫封校。\n议会接管那天，雾廊的歌声停了。\n没有人知道第七回声去了哪里。'],
    memory:['记忆代价','裂痕合上了。你修好了回声，却想不起是谁在井边拉住你的手。\n阿柚来看你，你觉得她眼熟，但叫不出名字。\n她说：“没关系，我记得就行。”']
  };
  const e=E[id];
  render(e[0],e[1].split('\n').map(p=>'<p>'+p+'</p>').join('')+
    '<p class="dim">线索 '+c+' 条 · 静默值 '+S.silence+' · 议会介入 '+S.council+
    ' · 阿柚 信'+S.rel.ayou.trust+'/疑'+S.rel.ayou.doubt+
    ' · 栖夜 信'+S.rel.qiyie.trust+
    ' · 闻玦 信'+S.rel.wenjue.trust+'</p>',
    [{t:'重新开始',f:()=>location.reload()}]);
}
function toggleBook(){
  const b=$('book');
  if(b.classList.contains('show')){b.classList.remove('show');return;}
  b.innerHTML='<h2>线索簿</h2><h3>世界观摘要</h3>'+
    WORLD.map(t=>'<div class="item">'+t+'</div>').join('')+
    '<h3>已获线索</h3>'+(S.clues.length?S.clues.map(id=>'<div class="item">'+(CLUES[id]||id)+'</div>').join(''):'<div class="item" style="color:#5a6a80">暂时没有线索。</div>')+
    '<h3>关系</h3>'+Object.keys(S.rel).map(k=>{const r=S.rel[k];return '<div class="item">'+k+'：信任'+r.trust+' / 怀疑'+r.doubt+' / 亏欠'+r.debt+' / 尴尬'+r.awkward+'</div>';}).join('')+
    '<div class="close">关闭（Tab）</div>';
  b.classList.add('show');
  b.querySelector('.close').onclick=()=>b.classList.remove('show');
}
document.addEventListener('keydown',e=>{if(e.key==='Tab'){e.preventDefault();toggleBook();}});
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
function startGame(){
  S.started=true;
  updatePhase();
  $('cc').textContent=S.clues.length;
  $('sv').textContent=S.silence;
  $('cv').textContent=S.council;
  goto('hall');
}
showPro();
</script>
</body>
</html>
