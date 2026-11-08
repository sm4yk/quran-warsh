<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="theme-color" content="#0b3b34">
<title>القرآن الكريم - محمد سايد - ورش</title>
<style>
:root{
  --bg:#0b3b34; --surface:#125047; --line:#1f6b60; --text:#eaf4f0; --muted:#9ec4ba; --accent:#d4a94f; --accent-ink:#2a1f05;
  color-scheme: dark;
}
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--text);font-family:"Amiri","Noto Naskh Arabic","Traditional Arabic",serif;display:flex;flex-direction:column;
  padding-top:env(safe-area-inset-top,0px)}
header{padding:18px 20px 10px}
h1{margin:0;font-size:1.7rem;font-weight:700;line-height:1.3}
.sub{margin:2px 0 0;color:var(--muted);font-size:1rem}
.search{margin:12px 20px 8px;position:relative}
.search input{width:100%;padding:12px 16px;border-radius:999px;border:1px solid var(--line);background:var(--surface);color:var(--text);font:inherit;font-size:1.05rem;outline:none}
.search input:focus-visible{border-color:var(--accent);box-shadow:0 0 0 2px rgba(212,169,79,.35)}
.search input::placeholder{color:var(--muted)}
main{flex:1;overflow-y:auto;padding:4px 12px 150px;-webkit-overflow-scrolling:touch}
.row{display:flex;align-items:center;gap:14px;width:100%;padding:12px 10px;background:none;border:0;border-bottom:1px solid var(--line);color:inherit;font:inherit;font-size:1.25rem;text-align:right;cursor:pointer}
.row .n{flex:none;width:2.4rem;height:2.4rem;border:1px solid var(--line);border-radius:50%;display:grid;place-items:center;font-size:.9rem;color:var(--muted);font-family:system-ui,sans-serif}
.row .t{flex:1}
.row.on{background:var(--surface)}
.row.on .n{background:var(--accent);border-color:var(--accent);color:var(--accent-ink);font-weight:700}
.row:focus-visible{outline:2px solid var(--accent);outline-offset:-2px}
.empty{padding:30px 20px;color:var(--muted);text-align:center}
.player{position:fixed;inset:auto 0 0 0;background:var(--surface);border-top:1px solid var(--line);padding:12px 18px calc(12px + env(safe-area-inset-bottom,0px))}
.now{display:flex;justify-content:space-between;align-items:baseline;gap:10px}
.now b{font-size:1.3rem}
.msg{font-size:.9rem;color:var(--muted);font-family:system-ui,sans-serif}
.msg.err{color:#ffb4a8}
.seek{display:flex;align-items:center;gap:10px;margin:8px 0 6px;direction:ltr;font:.8rem system-ui,sans-serif;color:var(--muted)}
.seek input{flex:1;accent-color:var(--accent);height:24px}
.ctrl{display:flex;justify-content:center;align-items:center;gap:22px;direction:ltr}
.ctrl button{background:none;border:0;color:var(--text);width:46px;height:46px;border-radius:50%;display:grid;place-items:center;cursor:pointer}
.ctrl button:focus-visible{outline:2px solid var(--accent)}
.ctrl svg{width:26px;height:26px;fill:currentColor}
.ctrl .play{width:60px;height:60px;background:var(--accent);color:var(--accent-ink)}
.ctrl .play svg{width:30px;height:30px}
.ctrl .rep[aria-pressed="true"]{color:var(--accent)}
.ctrl .rep[aria-pressed="false"]{opacity:.55}
</style>
</head>
<body>
<header>
  <h1>القرآن الكريم</h1>
  <p class="sub">بصوت القارئ محمد سايد، برواية ورش عن نافع</p>
</header>
<div class="search"><input id="q" type="search" placeholder="ابحث عن سورة" aria-label="ابحث عن سورة" autocomplete="off"></div>
<main id="list" aria-label="قائمة السور"></main>

<div class="player" role="region" aria-label="المشغّل">
  <div class="now"><b id="title">اختر سورة للاستماع</b><span id="msg" class="msg"></span></div>
  <div class="seek"><span id="cur">0:00</span><input id="bar" type="range" min="0" max="100" value="0" step="0.1" aria-label="موضع التشغيل"><span id="dur">0:00</span></div>
  <div class="ctrl">
    <button id="rep" class="rep" aria-pressed="false" aria-label="تكرار السورة"><svg viewBox="0 0 24 24"><path d="M7 7h10v3l4-4-4-4v3H5v6h2V7zm10 10H7v-3l-4 4 4 4v-3h12v-6h-2v4z"/></svg></button>
    <button id="prev" aria-label="السورة السابقة"><svg viewBox="0 0 24 24"><path d="M6 6h2v12H6V6zm3.5 6l8.5 6V6l-8.5 6z"/></svg></button>
    <button id="play" class="play" aria-label="تشغيل"><svg id="ico" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg></button>
    <button id="next" aria-label="السورة التالية"><svg viewBox="0 0 24 24"><path d="M6 18l8.5-6L6 6v12zM16 6v12h2V6h-2z"/></svg></button>
    <span style="width:46px"></span>
  </div>
</div>
<audio id="a" preload="none"></audio>

<script>
const NAMES="الفاتحة,البقرة,آل عمران,النساء,المائدة,الأنعام,الأعراف,الأنفال,التوبة,يونس,هود,يوسف,الرعد,إبراهيم,الحجر,النحل,الإسراء,الكهف,مريم,طه,الأنبياء,الحج,المؤمنون,النور,الفرقان,الشعراء,النمل,القصص,العنكبوت,الروم,لقمان,السجدة,الأحزاب,سبأ,فاطر,يس,الصافات,ص,الزمر,غافر,فصلت,الشورى,الزخرف,الدخان,الجاثية,الأحقاف,محمد,الفتح,الحجرات,ق,الذاريات,الطور,النجم,القمر,الرحمن,الواقعة,الحديد,المجادلة,الحشر,الممتحنة,الصف,الجمعة,المنافقون,التغابن,الطلاق,التحريم,الملك,القلم,الحاقة,المعارج,نوح,الجن,المزمل,المدثر,القيامة,الإنسان,المرسلات,النبأ,النازعات,عبس,التكوير,الانفطار,المطففين,الانشقاق,البروج,الطارق,الأعلى,الغاشية,الفجر,البلد,الشمس,الليل,الضحى,الشرح,التين,العلق,القدر,البينة,الزلزلة,العاديات,القارعة,التكاثر,العصر,الهمزة,الفيل,قريش,الماعون,الكوثر,الكافرون,النصر,المسد,الإخلاص,الفلق,الناس".split(",");
const BASE="https://server16.mp3quran.net/m_sayed/Rewayat-Warsh-A-n-Nafi/";
const $=id=>document.getElementById(id);
const a=$("a"),list=$("list"),msg=$("msg");
let cur=0,repeat=false;

let resolved=null,tries=0;
const urls=n=>{
  const p=String(n).padStart(3,"0"),out=[];
  [BASE,resolved].forEach(b=>{if(b){[b+p+".mp3",b+n+".mp3"].forEach(u=>{if(!out.includes(u))out.push(u)})}});
  return out;
};
async function resolveServer(){
  try{
    const r=await fetch("https://www.mp3quran.net/api/v3/reciters?language=ar");
    const j=await r.json();
    const rec=j.reciters.find(x=>norm(x.name).includes("محمد سايد"));
    const m=rec&&(rec.moshaf.find(x=>/ورش/.test(x.name))||rec.moshaf[0]);
    resolved=m&&m.server?(m.server.endsWith("/")?m.server:m.server+"/"):false;
  }catch(e){resolved=false}
}
const fmt=s=>{if(!isFinite(s))return"0:00";s=Math.floor(s);const h=Math.floor(s/3600),m=Math.floor(s%3600/60),r=s%60;
  return (h?h+":"+String(m).padStart(2,"0"):m)+":"+String(r).padStart(2,"0")};
const norm=s=>s.replace(/[\u064B-\u0652\u0640]/g,"").replace(/[أإآ]/g,"ا").replace(/ى/g,"ي").replace(/ة/g,"ه");

function render(filter=""){
  const f=norm(filter.trim());
  list.innerHTML="";
  let c=0;
  NAMES.forEach((name,i)=>{
    const n=i+1;
    if(f&&!norm(name).includes(f)&&String(n)!==f)return;
    c++;
    const b=document.createElement("button");
    b.className="row"+(n===cur?" on":"");
    b.dataset.n=n;
    b.innerHTML='<span class="n">'+n+'</span><span class="t">سورة '+name+'</span>';
    b.onclick=()=>load(n,true);
    list.appendChild(b);
  });
  if(!c)list.innerHTML='<div class="empty">لا توجد سورة بهذا الاسم</div>';
}
function mark(){
  list.querySelectorAll(".row").forEach(r=>r.classList.toggle("on",+r.dataset.n===cur));
}
function load(n,play){
  if(n<1)n=114; if(n>114)n=1;
  cur=n;
  tries=0;
  a.src=urls(n)[0];
  $("title").textContent="سورة "+NAMES[n-1];
  msg.textContent="جارٍ التحميل"; msg.className="msg";
  $("bar").value=0;$("cur").textContent="0:00";$("dur").textContent="0:00";
  mark();
  try{localStorage.setItem("last",n)}catch(e){}
  if("mediaSession"in navigator){
    navigator.mediaSession.metadata=new MediaMetadata({title:"سورة "+NAMES[n-1],artist:"محمد سايد",album:"رواية ورش عن نافع"});
  }
  if(play)a.play().catch(()=>{});
}
function setIcon(){
  $("ico").innerHTML=a.paused?'<path d="M8 5v14l11-7z"/>':'<path d="M6 5h4v14H6zm8 0h4v14h-4z"/>';
  $("play").setAttribute("aria-label",a.paused?"تشغيل":"إيقاف مؤقت");
}
$("play").onclick=()=>{
  if(!cur){load(1,true);return}
  a.paused?a.play().catch(()=>{}):a.pause();
};
$("next").onclick=()=>load((cur||0)+1,true);
$("prev").onclick=()=>load((cur||2)-1,true);
$("rep").onclick=e=>{repeat=!repeat;e.currentTarget.setAttribute("aria-pressed",repeat)};
$("bar").oninput=e=>{if(a.duration)a.currentTime=a.duration*e.target.value/100};
a.onplay=a.onpause=setIcon;
a.onloadedmetadata=()=>{$("dur").textContent=fmt(a.duration)};
a.oncanplay=()=>{msg.textContent="";};
a.onwaiting=()=>{msg.textContent="جارٍ التحميل";msg.className="msg"};
a.onplaying=()=>{msg.textContent=""};
a.ontimeupdate=()=>{
  $("cur").textContent=fmt(a.currentTime);
  if(a.duration)$("bar").value=a.currentTime/a.duration*100;
};
a.onended=()=>{repeat?(a.currentTime=0,a.play()):load(cur+1,true)};
a.onerror=async()=>{
  const n=cur;
  tries++;
  let list=urls(n);
  if(tries>=list.length&&resolved===null){
    msg.textContent="جارٍ البحث عن مصدر آخر";msg.className="msg";
    await resolveServer();
    if(cur!==n)return;
    list=urls(n);
  }
  if(tries<list.length){a.src=list[tries];a.play().catch(()=>{});return}
  const code=a.error?a.error.code:"?";
  msg.textContent="تعذّر التشغيل (رمز "+code+"). تأكد من الإنترنت ومن إذن الاتصال للتطبيق.";
  msg.className="msg err";setIcon();
};
if("mediaSession"in navigator){
  navigator.mediaSession.setActionHandler("play",()=>a.play());
  navigator.mediaSession.setActionHandler("pause",()=>a.pause());
  navigator.mediaSession.setActionHandler("nexttrack",()=>load(cur+1,true));
  navigator.mediaSession.setActionHandler("previoustrack",()=>load(cur-1,true));
}
$("q").oninput=e=>render(e.target.value);
render();
try{
  const last=+localStorage.getItem("last");
  if(last>=1&&last<=114){load(last,false);list.querySelector(".row.on")?.scrollIntoView({block:"center"})}
}catch(e){}
</script>
</body>
</html>
