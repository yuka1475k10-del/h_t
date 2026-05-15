<!DOCTYPE html>  
  
<html lang="ja">  
<head>  
  <meta charset="UTF-8"/>  
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover"/>  
  <meta name="theme-color" content="#5cb87a"/>  
  <meta name="apple-mobile-web-app-capable" content="yes"/>  
  <meta name="apple-mobile-web-app-status-bar-style" content="default"/>  
  <meta name="apple-mobile-web-app-title" content="ひとことふたこと"/>  
  <title>ひとことふたこと</title>  
  <link rel="manifest" href="manifest.json"/>  
  
  <!-- PWA icons (inline SVG as data URI) -->  
  
  <link rel="apple-touch-icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 512 512'%3E%3Crect width='512' height='512' rx='100' fill='%235cb87a'/%3E%3Ctext x='256' y='340' font-size='280' text-anchor='middle'%3E🌿%3C/text%3E%3C/svg%3E"/>  
  
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>  
  
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>  
  
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>  
  
  <link rel="preconnect" href="https://fonts.googleapis.com"/>  
  <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@600;700;800;900&family=Zen+Maru+Gothic:wght@500;700;900&display=swap" rel="stylesheet"/>  
  <style>  
    html, body { margin:0; padding:0; background:#f2faf4; -webkit-tap-highlight-color:transparent; }  
    * { box-sizing:border-box; }  
    /* safe area for iPhone notch */  
    body { padding-top: env(safe-area-inset-top); padding-bottom: env(safe-area-inset-bottom); }  
  </style>  
</head>  
<body>  
<div id="root"></div>  
  
<script type="text/babel">  
const { useState } = React;  
  
const C = {  
  bg:"#f2faf4", bgDeep:"#e4f5e8", surface:"#ffffff", surfaceG:"#f7fdf8",  
  green1:"#5cb87a", green2:"#3d9e5c", greenLight:"#a8dbb8", greenPale:"#d4f0dc", greenFaint:"#edfaf1",  
  pink1:"#f07ab0", pink2:"#e0508a", pinkLight:"#f9b8d4", pinkPale:"#fde4ef", pinkFaint:"#fff0f6",  
  blue:"#7ab3e0", bluePale:"#d6eaf8", blueFaint:"#eef6fd",  
  gold:"#c8960c", goldPale:"#fdf3d0", goldBorder:"#f0d060",  
  ink:"#2d3f30", inkMid:"#5a7060", inkLight:"#96b09a", border:"#c2e8cc", white:"#ffffff",  
};  
  
const NEGATIVE_KW=["つらい","辛い","悲しい","悲しく","苦しい","疲れた","嫌","きつい","むかつく","イライラ","怒り","不安","心配","落ち込","泣","ダメ","だめ","最悪","ひどい","しんどい","無理","やばい","消えたい","hate","sad","tired","awful","terrible","worst","depressed"];  
const ENCOURAGEMENTS=["今日もよくここまで来ましたね 🌿 ここに置いていった分だけ、すこし軽くなれますように","そんな気持ち、ちゃんと受け取りました 🌸 明日はまた、ちがう風が吹きますよ","言葉にできただけで、もう十分えらいです 🌙 今夜はゆっくり休んでね","ごみ箱に捨てた分だけ、心に余白ができました ☁️ その余白にいいことが入ってきますように","モヤモヤを外に出せた、それだけで今日はよくやりました 🕊️","しんどい日があるから、穏やかな日がよりやさしく感じられるんだと思います 🌤️","ぜんぶ吐き出してくれてありがとう。あなたの正直さがすきです 💚"];  
  
function isNegative(t){return t?NEGATIVE_KW.some(k=>t.includes(k)):false;}  
function getDateKey(d=new Date()){return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,"0")}-${String(d.getDate()).padStart(2,"0")}`;}  
function formatDateJP(key){const[y,m,d]=key.split("-");return `${y}年${parseInt(m)}月${parseInt(d)}日`;}  
function todayLabel(){const d=new Date(),days=["日","月","火","水","木","金","土"];return `${d.getMonth()+1}月${d.getDate()}日（${days[d.getDay()]}）`;}  
function randomItem(arr){return arr[Math.floor(Math.random()*arr.length)];}  
function loadEntries(){try{return JSON.parse(localStorage.getItem("hf2_entries")||"{}");}catch{return {};}}  
function saveEntries(e){localStorage.setItem("hf2_entries",JSON.stringify(e));}  
  
async function fetchWiseQuote(hitokoto,futakoto){  
  const diaryText=[hitokoto,futakoto].filter(Boolean).join("　/　");  
  const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,messages:[{role:"user",content:`以下は今日の日記です。\n「${diaryText}」\n\nこの日記を書いた人の気持ちや状況にぴったり寄り添う、世界の偉人・著名人の言葉をひとつ選んで教えてください。\n\n必ずJSON形式のみで返してください。前置きや説明文は不要です。\n{\n  "quote": "（名言の日本語訳）",\n  "author": "（人物名）",\n  "authorEn": "（人物名の英語表記）",\n  "era": "（生きた時代・分野、例：古代ギリシャの哲学者）",\n  "reason": "（この言葉を選んだ理由を40字以内でやさしく）"\n}`}]})});  
  const data=await res.json();  
  const text=data.content?.map(c=>c.text||"").join("")||"";  
  try{return JSON.parse(text.replace(/```json|```/g,"").trim());}catch{return null;}  
}  
  
async function fetchLetter(entries,periodLabel){  
  const lines=Object.entries(entries).sort(([a],[b])=>a.localeCompare(b)).map(([date,e])=>`${formatDateJP(date)}：${e.hitokoto}${e.futakoto?"／"+e.futakoto:""}`).join("\n");  
  if(!lines)return null;  
  const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,messages:[{role:"user",content:`以下は${periodLabel}分の日記です。\n\n${lines}\n\nこの日記を書いた人へ、世界の偉人・著名人のひとりになりきって、手紙を書いてください。\n\n必ずJSON形式のみで返してください。前置きや説明文は不要です。\n{\n  "author": "（手紙を書く偉人の名前）",\n  "authorEn": "（英語表記）",\n  "era": "（時代・分野）",\n  "greeting": "（書き出し、例：親愛なるきみへ）",\n  "body": "（手紙の本文。200字以内。その人の日記の内容を踏まえて、偉人らしい言葉で温かく語りかける）",\n  "quote": "（その偉人の実際の名言、日本語）",\n  "closing": "（結びの言葉、例：あなたの味方より）"\n}`}]})});  
  const data=await res.json();  
  const text=data.content?.map(c=>c.text||"").join("")||"";  
  try{return JSON.parse(text.replace(/```json|```/g,"").trim());}catch{return null;}  
}  
  
async function generateSummary(entries,periodLabel){  
  const lines=Object.entries(entries).sort(([a],[b])=>a.localeCompare(b)).map(([date,e])=>`${formatDateJP(date)}：${e.hitokoto}${e.futakoto?"　/　"+e.futakoto:""}`).join("\n");  
  if(!lines)return "この期間の記録がまだありません 🌱\nすこしずつ書いていきましょう。";  
  const res=await fetch("https://api.anthropic.com/v1/messages",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({model:"claude-sonnet-4-20250514",max_tokens:1000,messages:[{role:"user",content:`以下は日記の記録です。「わたしの${periodLabel}」というタイトルで、その人を温かく振り返るまとめを書いてください。\n200文字以内、やさしく穏やかな文体で。絵文字を2〜3個自然に使って。箇条書きにせず、ひとつながりの文章で。\n\n${lines}`}]})});  
  const data=await res.json();  
  return data.content?.map(c=>c.text||"").join("")||"";  
}  
  
function WaveDecor(){  
  return React.createElement("svg",{viewBox:"0 0 400 36",preserveAspectRatio:"none",style:{position:"absolute",bottom:0,left:0,right:0,width:"100%",height:36,display:"block"}},  
    React.createElement("path",{d:"M0,18 C80,36 160,0 240,18 C310,34 360,8 400,18 L400,36 L0,36 Z",fill:C.bg}));  
}  
  
function QuoteCard({quote,onClose}){  
  if(!quote)return null;  
  return (  
    <div style={{marginTop:16,padding:"20px 20px 16px",background:`linear-gradient(150deg,${C.goldPale},#fffef5)`,borderRadius:22,border:`2px solid ${C.goldBorder}`,animation:"popIn 0.5s cubic-bezier(.34,1.56,.64,1)",position:"relative",boxShadow:`0 6px 24px rgba(200,150,12,0.14)`}}>  
      <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:12}}>  
        <div style={{width:30,height:30,borderRadius:"50%",flexShrink:0,background:`linear-gradient(135deg,${C.gold},#e8b030)`,display:"flex",alignItems:"center",justifyContent:"center",fontSize:13,color:"#fff",fontWeight:900,boxShadow:"0 2px 8px rgba(200,150,12,0.35)"}}>✦</div>  
        <div>  
          <div style={{fontSize:10,fontWeight:800,color:C.gold,letterSpacing:1.5}}>TODAY'S WISDOM</div>  
          <div style={{fontSize:11,color:C.inkLight}}>今日の日記にぴったりの言葉</div>  
        </div>  
      </div>  
      <div style={{borderLeft:`3px solid ${C.goldBorder}`,paddingLeft:13,marginBottom:12}}>  
        <p style={{margin:0,fontSize:14,color:C.ink,lineHeight:2,fontWeight:600}}>「{quote.quote}」</p>  
      </div>  
      <div style={{fontSize:13,fontWeight:700,color:C.inkMid}}>― {quote.author}</div>  
      {quote.authorEn&&<div style={{fontSize:10,color:C.inkLight}}>{quote.authorEn}</div>}  
      {quote.era&&<div style={{fontSize:10,color:C.inkLight,marginTop:1}}>{quote.era}</div>}  
      {quote.reason&&<div style={{marginTop:10,padding:"7px 11px",background:"rgba(200,150,12,0.09)",borderRadius:10,fontSize:12,color:C.inkMid,lineHeight:1.7}}>💬 {quote.reason}</div>}  
      <button onClick={onClose} style={{position:"absolute",top:12,right:14,background:"none",border:"none",fontSize:17,cursor:"pointer",color:C.inkLight}}>×</button>  
    </div>  
  );  
}  
  
function HomeView({onSaved}){  
  const today=getDateKey(),existing=loadEntries()[today];  
  const[hitokoto,setHitokoto]=useState(existing?.hitokoto||"");  
  const[futakoto,setFutakoto]=useState(existing?.futakoto||"");  
  const[encourage,setEncourage]=useState(null);  
  const[quote,setQuote]=useState(existing?.quote||null);  
  const[saved,setSaved]=useState(!!existing);  
  const[quoteLoading,setQuoteLoading]=useState(false);  
  const[shakeH,setShakeH]=useState(false);  
  
  const handleSave=()=>{  
    if(!hitokoto.trim()){setShakeH(true);setTimeout(()=>setShakeH(false),500);return;}  
    const all=loadEntries();  
    all[today]={hitokoto:hitokoto.trim(),futakoto:futakoto.trim(),quote:null,savedAt:Date.now()};  
    saveEntries(all);setSaved(true);  
    if(isNegative(futakoto))setEncourage(randomItem(ENCOURAGEMENTS));  
    onSaved?.();  
    setQuoteLoading(true);  
    fetchWiseQuote(hitokoto.trim(),futakoto.trim()).then(wq=>{  
      if(wq){const latest=loadEntries();if(latest[today]){latest[today].quote=wq;saveEntries(latest);}setQuote(wq);}  
    }).finally(()=>setQuoteLoading(false));  
  };  
  
  const ta=(shake)=>({width:"100%",boxSizing:"border-box",border:`2.5px solid ${shake?C.pink1:C.border}`,borderRadius:18,padding:"13px 15px",fontSize:15,fontFamily:"inherit",background:C.surfaceG,resize:"none",outline:"none",color:C.ink,lineHeight:1.85,boxShadow:shake?`0 0 0 4px ${C.pinkPale}`:"0 2px 8px rgba(92,184,122,0.08)",animation:shake?"shake 0.4s":"none",transition:"border 0.2s,box-shadow 0.2s"});  
  
  return (  
    <div style={{paddingBottom:8}}>  
      <div style={{textAlign:"center",marginBottom:26}}>  
        <span style={{background:`linear-gradient(135deg,${C.greenPale},${C.pinkPale})`,color:C.green2,borderRadius:50,padding:"7px 24px",fontSize:13,fontWeight:800,letterSpacing:0.8,boxShadow:`0 3px 12px rgba(92,184,122,0.2)`,display:"inline-block"}}>{todayLabel()}</span>  
      </div>  
      <div style={{marginBottom:18}}>  
        <label style={{display:"flex",alignItems:"center",gap:6,fontWeight:800,fontSize:13,color:C.green2,marginBottom:8}}>  
          <span style={{fontSize:17}}>✏️</span>ひとことめ：今日のことば  
          <span style={{fontSize:11,color:C.green1,background:C.greenPale,borderRadius:50,padding:"2px 9px",fontWeight:700}}>必須</span>  
        </label>  
        <textarea value={hitokoto} onChange={e=>{setHitokoto(e.target.value);setSaved(false);setQuote(null);}} placeholder="今日はどんな一日でしたか？" rows={3} style={ta(shakeH)}/>  
      </div>  
      <div style={{marginBottom:26}}>  
        <label style={{display:"flex",alignItems:"center",gap:6,fontWeight:700,fontSize:13,color:C.inkMid,marginBottom:8}}>  
          <span style={{fontSize:15}}>🗑️</span>ふたことめ：感情のごみ箱  
          <span style={{fontSize:11,color:C.inkLight,background:C.bgDeep,borderRadius:50,padding:"2px 9px",fontWeight:600}}>任意</span>  
        </label>  
        <textarea value={futakoto} onChange={e=>{setFutakoto(e.target.value);setSaved(false);setQuote(null);}} placeholder="ぐちでもモヤモヤでも、ここに捨てちゃいましょう" rows={3} style={ta(false)}/>  
      </div>  
      <button onClick={handleSave} style={{width:"100%",padding:"15px",borderRadius:50,border:"none",background:saved?`linear-gradient(135deg,#a8e06a,${C.green1},#4aaa6a)`:`linear-gradient(135deg,${C.pinkLight},${C.pink1})`,color:C.white,fontSize:saved?17:15,fontWeight:900,cursor:"pointer",boxShadow:saved?"0 8px 28px rgba(92,184,122,0.55), 0 0 0 3px rgba(168,224,106,0.3)":"0 6px 22px rgba(240,122,176,0.28)",letterSpacing:saved?2:0.8,fontFamily:"inherit",transition:"all 0.4s cubic-bezier(.34,1.56,.64,1)",transform:saved?"scale(1.03)":"scale(1)"}}>  
        {saved?"✨ 今日も、ちゃんと残せた。":"今日の気持ちを記録する 🌸"}  
      </button>  
      {quoteLoading&&<div style={{display:"flex",alignItems:"center",gap:8,marginTop:12,padding:"10px 16px",background:C.goldPale,borderRadius:14,border:`1.5px solid ${C.goldBorder}`}}><div style={{fontSize:15,animation:"spin 2s linear infinite",display:"inline-block"}}>✦</div><p style={{margin:0,fontSize:12,color:C.gold,fontWeight:700}}>偉人の言葉を探しています…（他の画面も使えます）</p></div>}  
      {encourage&&<div style={{marginTop:16,padding:"16px 18px",background:`linear-gradient(135deg,${C.greenFaint},${C.blueFaint})`,borderRadius:20,border:`2px solid ${C.greenLight}`,animation:"popIn 0.4s cubic-bezier(.34,1.56,.64,1)",position:"relative"}}><div style={{fontSize:11,fontWeight:800,color:C.green1,marginBottom:6,letterSpacing:1}}>ひとこと、おつかれさま 🌿</div><p style={{margin:0,fontSize:14,color:C.inkMid,lineHeight:1.9}}>{encourage}</p><button onClick={()=>setEncourage(null)} style={{position:"absolute",top:10,right:13,background:"none",border:"none",fontSize:17,cursor:"pointer",color:C.inkLight}}>×</button></div>}  
      <QuoteCard quote={quote} onClose={()=>setQuote(null)}/>  
    </div>  
  );  
}  
  
function TrashModal({year,month,entries,onClose}){  
  const MN=["1月","2月","3月","4月","5月","6月","7月","8月","9月","10月","11月","12月"];  
  const prefix=`${year}-${String(month+1).padStart(2,"0")}`;  
  const list=Object.entries(entries).filter(([k,e])=>k.startsWith(prefix)&&e.futakoto).sort(([a],[b])=>a.localeCompare(b));  
  return (  
    <div style={{position:"fixed",inset:0,zIndex:100,background:"rgba(45,63,48,0.4)",backdropFilter:"blur(5px)",display:"flex",alignItems:"flex-end",justifyContent:"center",animation:"fadeUp 0.2s ease"}} onClick={onClose}>  
      <div onClick={e=>e.stopPropagation()} style={{width:"100%",maxWidth:440,background:C.white,borderRadius:"28px 28px 0 0",padding:"22px 20px 52px",maxHeight:"75vh",overflowY:"auto",boxShadow:"0 -8px 40px rgba(45,63,48,0.18)"}}>  
        <div style={{width:40,height:5,borderRadius:3,background:C.greenPale,margin:"0 auto 20px"}}/>  
        <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:20}}>  
          <div style={{display:"flex",alignItems:"center",gap:10}}><span style={{fontSize:26}}>🗑️</span><div><div style={{fontWeight:800,fontSize:15,color:C.ink}}>感情のごみ箱</div><div style={{fontSize:11,color:C.inkLight}}>{year}年 {MN[month]} の記録</div></div></div>  
          <button onClick={onClose} style={{background:C.bgDeep,border:"none",borderRadius:50,width:34,height:34,cursor:"pointer",fontSize:17,color:C.inkLight,fontWeight:700}}>×</button>  
        </div>  
        {list.length===0?<div style={{textAlign:"center",padding:"36px 0",color:C.inkLight,fontSize:14}}>この月のごみ箱は空っぽです 🌤️</div>:<div style={{display:"flex",flexDirection:"column",gap:12}}>{list.map(([key,e])=><div key={key} style={{padding:"14px 16px",background:`linear-gradient(135deg,${C.pinkFaint},${C.blueFaint})`,borderRadius:18,border:`2px solid ${C.pinkLight}`}}><div style={{fontSize:11,fontWeight:800,color:C.pink1,marginBottom:6}}>{formatDateJP(key)}</div><p style={{margin:0,fontSize:14,color:C.inkMid,lineHeight:1.85}}>{e.futakoto}</p></div>)}</div>}  
      </div>  
    </div>  
  );  
}  
  
function CalendarView(){  
  const today=new Date();  
  const[year,setYear]=useState(today.getFullYear());  
  const[month,setMonth]=useState(today.getMonth());  
  const[selected,setSelected]=useState(null);  
  const[showTrash,setShowTrash]=useState(false);  
  const entries=loadEntries();  
  const firstDay=new Date(year,month,1).getDay();  
  const daysInMonth=new Date(year,month+1,0).getDate();  
  const DN=["日","月","火","水","木","金","土"];  
  const MN=["1月","2月","3月","4月","5月","6月","7月","8月","9月","10月","11月","12月"];  
  const cells=[];  
  for(let i=0;i<firstDay;i++)cells.push(null);  
  for(let d=1;d<=daysInMonth;d++)cells.push(d);  
  const prev=()=>{if(month===0){setYear(y=>y-1);setMonth(11);}else setMonth(m=>m-1);setSelected(null);};  
  const next=()=>{if(month===11){setYear(y=>y+1);setMonth(0);}else setMonth(m=>m+1);setSelected(null);};  
  const todayKey=getDateKey();  
  const prefix=`${year}-${String(month+1).padStart(2,"0")}`;  
  const trashCount=Object.entries(entries).filter(([k,e])=>k.startsWith(prefix)&&e.futakoto).length;  
  
  return (  
    <div>  
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:22}}>  
        <button onClick={prev} style={{background:C.greenPale,border:"none",borderRadius:50,width:38,height:38,cursor:"pointer",fontSize:18,color:C.green2,fontWeight:800}}>‹</button>  
        <span style={{fontWeight:800,fontSize:16,color:C.ink}}>{year}年 {MN[month]}</span>  
        <div style={{display:"flex",alignItems:"center",gap:8}}>  
          <button onClick={()=>setShowTrash(true)} style={{display:"flex",alignItems:"center",gap:4,background:"transparent",border:`1.5px solid ${C.pinkLight}`,borderRadius:50,padding:"5px 11px",color:C.pinkLight,fontSize:11,fontWeight:700,cursor:"pointer",fontFamily:"inherit",transition:"all 0.2s",whiteSpace:"nowrap"}}>  
            <span style={{fontSize:11}}>🗑️</span>ゴミ箱  
            {trashCount>0&&<span style={{background:C.pinkLight,color:C.white,borderRadius:50,fontSize:10,fontWeight:800,padding:"1px 6px"}}>{trashCount}</span>}  
          </button>  
          <button onClick={next} style={{background:C.greenPale,border:"none",borderRadius:50,width:38,height:38,cursor:"pointer",fontSize:18,color:C.green2,fontWeight:800}}>›</button>  
        </div>  
      </div>  
      <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",gap:3,marginBottom:6}}>  
        {DN.map((d,i)=><div key={d} style={{textAlign:"center",fontSize:11,fontWeight:800,color:i===0?C.pink1:i===6?C.blue:C.inkLight,padding:"3px 0"}}>{d}</div>)}  
      </div>  
      <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",gap:4}}>  
        {cells.map((d,i)=>{  
          if(!d)return React.createElement("div",{key:`e${i}`});  
          const key=`${year}-${String(month+1).padStart(2,"0")}-${String(d).padStart(2,"0")}`;  
          const entry=entries[key],isToday=key===todayKey,isSel=key===selected;  
          return <button key={key} onClick={()=>entry&&setSelected(isSel?null:key)} style={{aspectRatio:"1",border:isToday?`2.5px solid ${C.pink1}`:isSel?`2.5px solid ${C.green1}`:"2px solid transparent",borderRadius:14,background:isSel?`linear-gradient(135deg,${C.greenPale},${C.pinkPale})`:entry?C.greenFaint:"transparent",cursor:entry?"pointer":"default",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",gap:1,padding:2,transition:"all 0.15s",fontFamily:"inherit"}}>  
            <span style={{fontSize:13,fontWeight:isToday?900:400,color:isToday?C.pink2:C.ink}}>{d}</span>  
            {entry&&<span style={{fontSize:7,color:C.green1}}>●</span>}  
          </button>;  
        })}  
      </div>  
      {selected&&entries[selected]&&<div style={{marginTop:20,padding:"18px 20px",background:C.white,borderRadius:22,border:`2px solid ${C.greenPale}`,boxShadow:`0 6px 24px rgba(92,184,122,0.12)`,animation:"popIn 0.3s ease"}}>  
        <div style={{fontSize:12,fontWeight:800,color:C.green1,marginBottom:10}}>{formatDateJP(selected)}</div>  
        <div style={{marginBottom:entries[selected].quote?12:0}}><div style={{fontSize:11,color:C.green2,fontWeight:800,marginBottom:4}}>ひとことめ</div><p style={{margin:0,fontSize:14,color:C.ink,lineHeight:1.85}}>{entries[selected].hitokoto}</p></div>  
        {entries[selected].quote&&<div style={{paddingTop:10,borderTop:`1.5px dashed ${C.border}`,marginTop:4}}><div style={{fontSize:11,color:C.gold,fontWeight:800,marginBottom:6}}>✦ 偉人の言葉</div><p style={{margin:"0 0 4px",fontSize:13,color:C.ink,fontStyle:"italic",lineHeight:1.8}}>「{entries[selected].quote.quote}」</p><p style={{margin:0,fontSize:11,color:C.inkLight}}>― {entries[selected].quote.author}</p></div>}  
      </div>}  
      {showTrash&&<TrashModal year={year} month={month} entries={entries} onClose={()=>setShowTrash(false)}/>}  
    </div>  
  );  
}  
  
function LetterModal({letter,onClose}){  
  const[open,setOpen]=useState(false);  
  useState(()=>{setTimeout(()=>setOpen(true),80);},[]);  
  return (  
    <div style={{position:"fixed",inset:0,zIndex:200,background:"rgba(30,50,35,0.55)",backdropFilter:"blur(6px)",display:"flex",alignItems:"center",justifyContent:"center",padding:"20px",animation:"fadeUp 0.25s ease"}} onClick={onClose}>  
      <div onClick={e=>e.stopPropagation()} style={{width:"100%",maxWidth:400,background:"linear-gradient(160deg,#fffef8,#fdf9ee)",borderRadius:24,overflow:"hidden",boxShadow:"0 24px 60px rgba(30,50,35,0.25)",animation:"popIn 0.45s cubic-bezier(.34,1.56,.64,1)",border:`1.5px solid ${C.goldBorder}`}}>  
        <div style={{background:`linear-gradient(135deg,${C.goldPale},#fef7d6)`,padding:"22px 24px 16px",borderBottom:`1.5px dashed ${C.goldBorder}`,position:"relative",textAlign:"center"}}>  
          <div style={{fontSize:open?48:32,transition:"font-size 0.5s cubic-bezier(.34,1.56,.64,1)",marginBottom:8,display:"inline-block",animation:open?"letterBounce 0.6s cubic-bezier(.34,1.56,.64,1)":"none"}}>💌</div>  
          <div style={{fontSize:10,fontWeight:800,color:C.gold,letterSpacing:2,marginBottom:2}}>LETTER FROM</div>  
          <div style={{fontSize:17,fontWeight:900,color:C.ink}}>{letter.author}</div>  
          {letter.authorEn&&<div style={{fontSize:11,color:C.inkLight,marginTop:1}}>{letter.authorEn}</div>}  
          {letter.era&&<div style={{fontSize:10,color:C.inkLight,marginTop:1}}>{letter.era}</div>}  
          <button onClick={onClose} style={{position:"absolute",top:14,right:16,background:"rgba(200,150,12,0.12)",border:"none",borderRadius:50,width:28,height:28,cursor:"pointer",fontSize:14,color:C.inkMid,fontWeight:700}}>×</button>  
        </div>  
        <div style={{padding:"22px 26px 28px",overflowY:"auto",maxHeight:"60vh"}}>  
          <p style={{margin:"0 0 14px",fontSize:13,color:C.inkMid,fontStyle:"italic",fontWeight:700}}>{letter.greeting}</p>  
          <p style={{margin:"0 0 18px",fontSize:14,color:C.ink,lineHeight:2,whiteSpace:"pre-wrap"}}>{letter.body}</p>  
          <div style={{background:`linear-gradient(135deg,${C.greenFaint},${C.blueFaint})`,borderRadius:14,padding:"13px 16px",borderLeft:`4px solid ${C.green1}`,marginBottom:18}}>  
            <p style={{margin:0,fontSize:13,color:C.ink,fontStyle:"italic",lineHeight:1.9,fontWeight:600}}>「{letter.quote}」</p>  
          </div>  
          <p style={{margin:0,fontSize:12,color:C.inkLight,textAlign:"right",fontStyle:"italic"}}>{letter.closing}</p>  
        </div>  
      </div>  
    </div>  
  );  
}  
  
function SummaryView(){  
  const[period,setPeriod]=useState("week");  
  const[summary,setSummary]=useState(null);  
  const[loading,setLoading]=useState(false);  
  const[letterLoading,setLetterLoading]=useState(false);  
  const[letter,setLetter]=useState(null);  
  const[showLetter,setShowLetter]=useState(false);  
  const[newLetter,setNewLetter]=useState(false);  
  const entries=loadEntries();  
  const PERIODS=[{key:"week",label:"1週間",title:"1週間"},{key:"month",label:"1ヶ月",title:"1ヶ月"},{key:"year",label:"1年",title:"1年"}];  
  const getFiltered=()=>{  
    const now=new Date(),cutoff=new Date();  
    if(period==="week")cutoff.setDate(now.getDate()-7);  
    else if(period==="month")cutoff.setMonth(now.getMonth()-1);  
    else cutoff.setFullYear(now.getFullYear()-1);  
    return Object.fromEntries(Object.entries(entries).filter(([k])=>k>=getDateKey(cutoff)));  
  };  
  const handleGenerate=async()=>{  
    setLoading(true);setSummary(null);  
    const filtered=getFiltered(),pObj=PERIODS.find(p=>p.key===period);  
    const text=await generateSummary(filtered,pObj.title);  
    setSummary({title:`わたしの${pObj.title}`,text,count:Object.keys(filtered).length});  
    setLoading(false);  
  };  
  const handleLetter=async()=>{  
    setLetterLoading(true);setLetter(null);setNewLetter(false);  
    const filtered=getFiltered(),pObj=PERIODS.find(p=>p.key===period);  
    const result=await fetchLetter(filtered,pObj.title);  
    setLetterLoading(false);  
    if(result){setLetter(result);setNewLetter(true);setShowLetter(true);}  
  };  
  const filtered=getFiltered();  
  const sortedEntries=Object.entries(filtered).sort(([a],[b])=>b.localeCompare(a));  
  const count=sortedEntries.length;  
  return (  
    <div style={{paddingBottom:8}}>  
      <div style={{display:"flex",gap:8,marginBottom:20}}>  
        {PERIODS.map(p=><button key={p.key} onClick={()=>{setPeriod(p.key);setSummary(null);setLetter(null);setNewLetter(false);}} style={{flex:1,padding:"11px 0",borderRadius:50,border:"2.5px solid",borderColor:period===p.key?C.green1:C.border,background:period===p.key?`linear-gradient(135deg,${C.green1},${C.green2})`:C.white,color:period===p.key?C.white:C.inkLight,fontWeight:800,fontSize:13,cursor:"pointer",fontFamily:"inherit",transition:"all 0.2s"}}>{p.label}</button>)}  
      </div>  
      <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:20}}>  
        <div style={{padding:"11px 14px",background:C.surfaceG,borderRadius:16,border:`2px solid ${C.border}`,flexShrink:0}}>  
          <div style={{fontSize:10,color:C.inkLight,fontWeight:700}}>記録</div>  
          <div style={{fontSize:20,fontWeight:900,color:C.green1,lineHeight:1.1}}>{count}<span style={{fontSize:10,fontWeight:500,color:C.inkLight,marginLeft:2}}>日</span></div>  
        </div>  
        <button onClick={handleLetter} disabled={letterLoading||count===0} style={{position:"relative",flexShrink:0,padding:"10px 14px",borderRadius:50,border:`2px solid ${C.goldBorder}`,background:letterLoading?C.bgDeep:`linear-gradient(135deg,${C.goldPale},#fef7d0)`,color:letterLoading?C.inkLight:C.gold,fontSize:13,fontWeight:800,cursor:(letterLoading||count===0)?"wait":"pointer",fontFamily:"inherit",transition:"all 0.3s",display:"flex",alignItems:"center",gap:5,whiteSpace:"nowrap",boxShadow:count>0?"0 3px 12px rgba(200,150,12,0.2)":"none",opacity:count===0?0.5:1}}>  
          <span style={{fontSize:16,animation:letterLoading?"spin 1.5s linear infinite":"none",display:"inline-block"}}>{letterLoading?"✦":"💌"}</span>  
          {letterLoading?"届いてる...":"偉人のレター"}  
          {newLetter&&!showLetter&&<span style={{position:"absolute",top:-4,right:-4,width:12,height:12,borderRadius:"50%",background:C.pink1,boxShadow:`0 0 0 2px ${C.white}`,animation:"pulse 1.5s ease-in-out infinite"}}/>}  
        </button>  
        <button onClick={handleGenerate} disabled={loading||count===0} style={{flex:1,padding:"12px 0",borderRadius:50,border:"none",background:loading||count===0?C.bgDeep:`linear-gradient(135deg,${C.pinkLight},${C.pink1})`,color:loading||count===0?C.inkLight:C.white,fontSize:13,fontWeight:800,cursor:(loading||count===0)?"wait":"pointer",boxShadow:(!loading&&count>0)?"0 4px 16px rgba(240,122,176,0.28)":"none",letterSpacing:0.3,fontFamily:"inherit",transition:"all 0.3s",opacity:count===0?0.5:1}}>  
          {loading?"✨ 生成中...":"AIまとめ 🌸"}  
        </button>  
      </div>  
      {(loading||letterLoading)&&<div style={{textAlign:"center",padding:"18px 0"}}><div style={{fontSize:30,animation:"float 2s ease-in-out infinite",display:"inline-block"}}>{letterLoading?"💌":"🌿"}</div><p style={{color:C.inkLight,fontSize:12,marginTop:8}}>{letterLoading?"偉人があなたに手紙を書いています...":"あなたの日記を読んでいます..."}</p></div>}  
      {summary&&<div style={{padding:"20px",marginBottom:16,background:`linear-gradient(150deg,${C.greenFaint},${C.pinkFaint})`,borderRadius:20,border:`2px solid ${C.greenLight}`,animation:"popIn 0.5s cubic-bezier(.34,1.56,.64,1)",position:"relative",overflow:"hidden"}}><div style={{position:"absolute",top:-24,right:-24,width:90,height:90,background:`radial-gradient(circle,${C.greenPale},transparent)`,borderRadius:"50%",opacity:0.7}}/><div style={{fontSize:10,fontWeight:800,color:C.green1,marginBottom:5,letterSpacing:1}}>AI SUMMARY · {summary.count}日分</div><h3 style={{margin:"0 0 8px",fontSize:16,color:C.ink,fontWeight:900}}>{summary.title}</h3><p style={{margin:0,fontSize:13,color:C.inkMid,lineHeight:2,whiteSpace:"pre-wrap",position:"relative"}}>{summary.text}</p></div>}  
      {count===0?<div style={{textAlign:"center",padding:"36px 0",color:C.inkLight,fontSize:14}}>この期間の記録がまだありません 🌱</div>:<div style={{display:"flex",flexDirection:"column",gap:10}}>  
        <div style={{fontSize:11,fontWeight:800,color:C.inkLight,letterSpacing:0.8,marginBottom:2}}>ひとことの記録（新しい順）</div>  
        {sortedEntries.map(([key,e],i)=><div key={key} style={{padding:"13px 16px",background:i%2===0?C.white:`linear-gradient(135deg,${C.greenFaint},${C.blueFaint}20)`,borderRadius:18,border:`1.5px solid ${i%2===0?C.border:C.greenPale}`,animation:`fadeUp 0.3s ease ${i*0.04}s both`}}>  
          <div style={{fontSize:11,fontWeight:800,color:C.green1,marginBottom:5}}>{formatDateJP(key)}</div>  
          <p style={{margin:0,fontSize:14,color:C.ink,lineHeight:1.8,fontWeight:600}}>{e.hitokoto}</p>  
        </div>)}  
      </div>}  
      {showLetter&&letter&&<LetterModal letter={letter} onClose={()=>{setShowLetter(false);setNewLetter(false);}}/>}  
    </div>  
  );  
}  
  
const VIEWS={HOME:"home",CAL:"calendar",SUM:"summary"};  
function App(){  
  const[view,setView]=useState(VIEWS.HOME);  
  const[tick,setTick]=useState(0);  
  return (  
    <>  
      <style>{`  
        @keyframes shake{0%,100%{transform:translateX(0)}20%{transform:translateX(-5px)}40%{transform:translateX(5px)}60%{transform:translateX(-3px)}80%{transform:translateX(3px)}}  
        @keyframes fadeUp{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:translateY(0)}}  
        @keyframes popIn{from{opacity:0;transform:scale(0.92)}to{opacity:1;transform:scale(1)}}  
        @keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-10px)}}  
        @keyframes spin{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}  
        @keyframes letterBounce{0%{transform:scale(0.5) translateY(10px);opacity:0}60%{transform:scale(1.2) translateY(-4px)}100%{transform:scale(1) translateY(0);opacity:1}}  
        @keyframes pulse{0%,100%{transform:scale(1);opacity:1}50%{transform:scale(1.3);opacity:0.7}}  
        textarea:focus{border-color:#5cb87a!important;box-shadow:0 0 0 4px #d4f0dc!important;}  
        button:hover:not(:disabled){opacity:0.88;transform:translateY(-2px);}  
        button:active:not(:disabled){transform:translateY(0);opacity:1;}  
      `}</style>  
  
```  
  <div style={{minHeight:"100vh",background:`linear-gradient(160deg,${C.bg} 0%,#e8f8ee 55%,#fce8f3 100%)`,fontFamily:"'Nunito','Zen Maru Gothic',sans-serif",paddingBottom:90}}>  
    <div style={{background:`linear-gradient(135deg,${C.green1} 0%,#4aaa6a 40%,${C.blue} 80%,${C.pink1} 100%)`,padding:"26px 22px 44px",position:"relative",overflow:"hidden"}}>  
      <div style={{position:"absolute",top:-50,right:-50,width:180,height:180,borderRadius:"50%",background:"rgba(255,255,255,0.09)"}}/>  
      <div style={{position:"absolute",top:15,right:35,width:90,height:90,borderRadius:"50%",background:"rgba(255,255,255,0.06)"}}/>  
      <h1 style={{margin:0,fontSize:26,fontWeight:900,color:C.white,letterSpacing:2,fontFamily:"'Nunito',sans-serif",textShadow:"0 2px 12px rgba(40,100,60,0.3)"}}>ひとことふたこと</h1>  
      <p style={{margin:"6px 0 0",fontSize:12,color:"rgba(255,255,255,0.88)",letterSpacing:0.8,fontWeight:700}}>ひとひ、ひとこと。つながる、つきひ。</p>  
      <WaveDecor/>  
    </div>  
    <div style={{maxWidth:440,margin:"0 auto",padding:"28px 18px 0"}}>  
      {view===VIEWS.HOME&&<HomeView key={tick} onSaved={()=>setTick(t=>t+1)}/>}  
      {view===VIEWS.CAL&&<CalendarView/>}  
      {view===VIEWS.SUM&&<SummaryView/>}  
    </div>  
    <nav style={{position:"fixed",bottom:0,left:0,right:0,background:"rgba(255,255,255,0.97)",backdropFilter:"blur(18px)",borderTop:`2px solid ${C.greenPale}`,display:"flex",padding:"8px 0 20px",boxShadow:`0 -4px 20px rgba(92,184,122,0.12)`}}>  
      {[{id:VIEWS.HOME,icon:"✏️",label:"今日"},{id:VIEWS.CAL,icon:"🗓",label:"カレンダー"},{id:VIEWS.SUM,icon:"🌸",label:"振り返り"}].map(tab=>(  
        <button key={tab.id} onClick={()=>setView(tab.id)} style={{flex:1,background:"none",border:"none",cursor:"pointer",display:"flex",flexDirection:"column",alignItems:"center",gap:3,padding:"6px 0",fontFamily:"inherit"}}>  
          <div style={{fontSize:22,filter:view===tab.id?"none":"grayscale(70%) opacity(0.45)",transform:view===tab.id?"scale(1.18)":"scale(1)",transition:"all 0.2s"}}>{tab.icon}</div>  
          <span style={{fontSize:10,fontWeight:view===tab.id?800:500,color:view===tab.id?C.green2:C.inkLight,letterSpacing:0.4}}>{tab.label}</span>  
          {view===tab.id&&<div style={{width:20,height:3,borderRadius:2,background:`linear-gradient(90deg,${C.green1},${C.pink1})`,marginTop:-1}}/>}  
        </button>  
      ))}  
    </nav>  
  </div>  
</>  
```  
  
);  
}  
  
ReactDOM.createRoot(document.getElementById(“root”)).render(React.createElement(App));  
</script>  
  
<!-- PWA Service Worker -->  
  
<script>  
if('serviceWorker' in navigator){  
  window.addEventListener('load',()=>{  
    // inline SW as blob  
    const swCode = `  
      const CACHE='hf-v1';  
      const ASSETS=['/'];  
      self.addEventListener('install',e=>e.waitUntil(caches.open(CACHE).then(c=>c.addAll(ASSETS))));  
      self.addEventListener('fetch',e=>{  
        if(e.request.url.includes('api.anthropic.com')||e.request.url.includes('fonts.googleapis')||e.request.url.includes('unpkg.com')){  
          return e.respondWith(fetch(e.request));  
        }  
        e.respondWith(caches.match(e.request).then(r=>r||fetch(e.request)));  
      });  
    `;  
    const blob=new Blob([swCode],{type:'application/javascript'});  
    const url=URL.createObjectURL(blob);  
    navigator.serviceWorker.register(url);  
  });  
}  
</script>  
  
</body>  
</html>  
