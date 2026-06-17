<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Meeting & Call Tracker</title>
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
  --bg:#fff;--bg2:#f5f5f3;--bg3:#efefec;
  --tx:#1a1a18;--tx2:#6b6b67;--tx3:#a0a09a;
  --br:rgba(0,0,0,0.12);--br2:rgba(0,0,0,0.22);
  --ib:#e6f1fb;--it:#0c447c;--ibr:rgba(24,95,165,0.3);
  --sb:#eaf3de;--st:#3b6d11;--sbr:rgba(63,109,17,0.3);
  --db:#fcebeb;--dt:#a32d2d;--dbr:rgba(163,45,45,0.3);
  --wb:#faeeda;--wt:#854f0b;--wbr:rgba(133,79,11,0.3);
  --r:8px;--rl:12px
}
@media(prefers-color-scheme:dark){:root{
  --bg:#1c1c1a;--bg2:#2a2a28;--bg3:#323230;
  --tx:#f0efe8;--tx2:#a8a8a2;--tx3:#6b6b67;
  --br:rgba(255,255,255,0.1);--br2:rgba(255,255,255,0.2);
  --ib:#042c53;--it:#b5d4f4;--ibr:rgba(55,138,221,0.3);
  --sb:#173404;--st:#c0dd97;--sbr:rgba(99,153,34,0.3);
  --db:#501313;--dt:#f7c1c1;--dbr:rgba(226,75,74,0.3);
  --wb:#412402;--wt:#fac775;--wbr:rgba(186,117,23,0.3)
}}
body{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;background:var(--bg3);color:var(--tx);min-height:100vh;padding:1rem}
input,select{width:100%;padding:7px 10px;border:0.5px solid var(--br2);border-radius:var(--r);font-size:14px;background:var(--bg);color:var(--tx);font-family:inherit;outline:none}
input:focus,select:focus{box-shadow:0 0 0 2px var(--ibr)}
button{font-family:inherit;cursor:pointer}
.card{background:var(--bg);border-radius:var(--rl);border:0.5px solid var(--br);padding:1.25rem;margin-bottom:12px}
.row{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
.sec-label{font-size:11px;font-weight:500;color:var(--tx2);text-transform:uppercase;letter-spacing:.05em;margin-bottom:6px;margin-top:12px;display:block}
.stat-box{background:var(--bg2);border-radius:var(--r);padding:10px 12px;flex:1;min-width:70px}
.stat-box .sl{font-size:11px;color:var(--tx2);margin-bottom:2px}
.stat-box .sv{font-size:18px;font-weight:500}
.badge{font-size:11px;padding:2px 8px;border-radius:20px;white-space:nowrap}
.mtg-card{background:var(--bg);border:0.5px solid var(--br);border-radius:10px;padding:9px 12px;margin-bottom:6px}
.form-box{background:var(--bg2);border-radius:var(--rl);padding:1rem;margin-bottom:16px;border:0.5px solid var(--br)}
.grid3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:10px;margin-bottom:10px}
.grid2{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px}
.lbl{font-size:12px;color:var(--tx2);margin-bottom:3px;display:block}
.summary-strip{background:var(--ib);border-radius:10px;padding:10px 14px;margin-bottom:16px;display:flex;flex-wrap:wrap;gap:16px;align-items:center}
table{width:100%;border-collapse:collapse;font-size:13px}
th,td{padding:6px 8px;border-bottom:0.5px solid var(--br);white-space:nowrap}
thead tr{background:var(--bg2)}
tr.expandable{cursor:pointer}
tr.expandable:hover{background:var(--bg2)}
.week-detail{border:0.5px solid var(--ibr);border-radius:var(--rl);padding:1rem 1.25rem;margin-bottom:14px}
.mini-grid{display:flex;flex-wrap:wrap;gap:6px}
.mini-box{background:var(--bg2);border-radius:8px;padding:6px 10px;flex:1 1 80px}
.mini-box .ml{font-size:10px;color:var(--tx2);margin-bottom:1px}
.mini-box .mv{font-size:15px;font-weight:500}
.pts-bar{margin-top:12px;padding:10px 14px;background:var(--ib);border-radius:8px;display:flex;align-items:center}
.hidden{display:none}
@media(max-width:600px){.grid3{grid-template-columns:1fr 1fr}.grid2{grid-template-columns:1fr}}
</style>
</head>
<body>
<div id="app"></div>
<script>
const MEETING_TYPES=["Account Management","Follow-up","AM Intro","RMS Set Up","RMS Discussions","Upsell","Other"];
const MEETING_FORMATS=["Online","In Person"];
const STATUSES=["Scheduled","Completed","Completed - In Person","No Show","Cancelled","Rescheduled"];
const UPSELL_TYPES=["RMS Opt Out","Integration","Automation","Integration and Automation","Other"];
const CALL_STATUSES=["Not Connected","Connected - Quality Call","Connected - Brief Check In","Connected - Meeting Booked","Connected - Other Discussion"];
const CALL_POINTS={"Connected - Quality Call":1,"Connected - Meeting Booked":1};
const STATUS_COLORS={
  "Scheduled":{bg:"var(--ib)",tx:"var(--it)"},
  "Completed":{bg:"var(--sb)",tx:"var(--st)"},
  "Completed - In Person":{bg:"#e1f5ee",tx:"#0f6e56"},
  "No Show":{bg:"var(--db)",tx:"var(--dt)"},
  "Cancelled":{bg:"var(--db)",tx:"var(--dt)"},
  "Rescheduled":{bg:"var(--wb)",tx:"var(--wt)"}
};
const CALL_COLORS={
  "Not Connected":{bg:"#f0f0ee",tx:"#6b6b67"},
  "Connected - Quality Call":{bg:"var(--sb)",tx:"var(--st)"},
  "Connected - Brief Check In":{bg:"var(--ib)",tx:"var(--it)"},
  "Connected - Meeting Booked":{bg:"var(--wb)",tx:"var(--wt)"},
  "Connected - Other Discussion":{bg:"#f3e6fb",tx:"#5c0c7c"}
};
const TODAY=new Date().toISOString().split("T")[0];

let state={
  tab:"daily",
  date:TODAY,
  meetings:load("mtg_data2"),
  calls:load("call_data"),
  upsells:load("upsell_data"),
  showMtgForm:false,showCallForm:false,showUpsellForm:false,
  editMtgId:null,editCallId:null,editUpsellId:null,
  expandedWeek:null,
  mtgForm:{contact:"",type:MEETING_TYPES[0],format:MEETING_FORMATS[0],status:"",outcome:"",upsell:false,upsellType:""},
  callForm:{contact:"",status:"",notes:""},
  upsellForm:{contact:"",upsellType:"",notes:""}
};

function load(k){try{return JSON.parse(localStorage.getItem(k)||"[]")}catch{return[]}}
function save(){
  localStorage.setItem("mtg_data2",JSON.stringify(state.meetings));
  localStorage.setItem("call_data",JSON.stringify(state.calls));
  localStorage.setItem("upsell_data",JSON.stringify(state.upsells));
}
function setState(patch){Object.assign(state,patch);save();render()}

function getWeekKey(d){
  const dt=new Date(d),mon=new Date(dt);
  mon.setDate(dt.getDate()-((dt.getDay()+6)%7));
  return mon.toISOString().split("T")[0];
}
function formatDate(d){
  return new Date(d+"T00:00:00").toLocaleDateString("en-US",{weekday:"short",month:"short",day:"numeric"});
}
function calcMtg(arr){
  const on=arr.filter(m=>m.format==="Online"),ip=arr.filter(m=>m.format==="In Person");
  const co=on.filter(m=>m.status==="Completed").length;
  const ci=ip.filter(m=>m.status==="Completed - In Person"||(m.status==="Completed"&&m.format==="In Person")).length;
  const tc=arr.filter(m=>m.status.startsWith("Completed")).length;
  const ups=arr.filter(m=>m.upsell&&m.upsellType).length;
  return{total:arr.length,co,ci,tc,ups,
    ns:arr.filter(m=>m.status==="No Show").length,
    canc:arr.filter(m=>m.status==="Cancelled").length,
    resch:arr.filter(m=>m.status==="Rescheduled").length,
    pts:co*1+ci*2+ups,
    sro:on.length?Math.round(co/on.length*100)+"%":"—",
    srip:ip.length?Math.round(ci/ip.length*100)+"%":"—",
    srt:arr.length?Math.round(tc/arr.length*100)+"%":"—"
  };
}
function calcCall(arr){
  const conn=arr.filter(c=>c.status!=="Not Connected");
  const q=arr.filter(c=>c.status==="Connected - Quality Call").length;
  const b=arr.filter(c=>c.status==="Connected - Meeting Booked").length;
  return{total:arr.length,conn:conn.length,
    nc:arr.filter(c=>c.status==="Not Connected").length,
    q,b,
    brief:arr.filter(c=>c.status==="Connected - Brief Check In").length,
    other:arr.filter(c=>c.status==="Connected - Other Discussion").length,
    pts:q+b,
    cr:arr.length?Math.round(conn.length/arr.length*100)+"%":"—"
  };
}

function btn(txt,onclick,style=""){
  return `<button onclick="${onclick}" style="padding:7px 14px;border-radius:var(--r);border:0.5px solid var(--br2);background:var(--bg);color:var(--tx);font-size:13px;cursor:pointer;${style}">${txt}</button>`;
}
function badge(txt,bg,tx){
  return `<span class="badge" style="background:${bg};color:${tx}">${txt}</span>`;
}
function statBox(label,val,color="var(--tx)"){
  return `<div class="stat-box"><div class="sl">${label}</div><div class="sv" style="color:${color}">${val}</div></div>`;
}
function miniBox(label,val,color="var(--tx)"){
  return `<div class="mini-box"><div class="ml">${label}</div><div class="mv" style="color:${color}">${val}</div></div>`;
}
function selectOpts(opts,val,placeholder=""){
  return (placeholder?`<option value="">${placeholder}</option>`:"")+opts.map(o=>`<option${o===val?" selected":""}>${o}</option>`).join("");
}

function renderWeekly(){
  const weeks={};
  state.meetings.forEach(m=>{const k=getWeekKey(m.date);if(!weeks[k])weeks[k]={mtgs:[],calls:[],ups:[]};weeks[k].mtgs.push(m);});
  state.calls.forEach(c=>{const k=getWeekKey(c.date);if(!weeks[k])weeks[k]={mtgs:[],calls:[],ups:[]};weeks[k].calls.push(c);});
  state.upsells.forEach(u=>{const k=getWeekKey(u.date);if(!weeks[k])weeks[k]={mtgs:[],calls:[],ups:[]};weeks[k].ups.push(u);});
  const data=Object.entries(weeks).sort((a,b)=>b[0].localeCompare(a[0])).map(([wk,{mtgs,calls:wc,ups}])=>{
    const end=new Date(wk+"T00:00:00");end.setDate(end.getDate()+6);
    return{wk,mtgs,calls:wc,ups,ms:calcMtg(mtgs),cs:calcCall(wc),sup:ups.length,wkEnd:end.toISOString().split("T")[0]};
  });
  if(!data.length) return`<div style="text-align:center;padding:2rem 0;color:var(--tx3);font-size:14px">No data logged yet.</div>`;

  const rows=data.map(w=>`
    <tr class="expandable" onclick="toggleWeek('${w.wk}')">
      <td style="color:var(--tx2)">${formatDate(w.wk)}</td>
      <td style="text-align:center">${w.ms.total}</td>
      <td style="text-align:center;color:var(--st)">${w.ms.tc}</td>
      <td style="text-align:center;color:var(--it)">${w.ms.srt}</td>
      <td style="text-align:center;color:var(--it)">${w.ms.pts}</td>
      <td style="text-align:center;color:var(--wt)">${w.ms.ups+w.sup}</td>
      <td style="text-align:center">${w.cs.total}</td>
      <td style="text-align:center;color:var(--st)">${w.cs.conn}</td>
      <td style="text-align:center;color:var(--it)">${w.cs.cr}</td>
      <td style="text-align:center;color:var(--it)">${w.cs.pts}</td>
      <td style="text-align:center;color:var(--it);font-weight:600">${w.ms.pts+w.cs.pts+w.sup}</td>
    </tr>
  `).join("");

  const details=data.map(w=>{
    if(state.expandedWeek!==w.wk) return "";
    const days=[...new Set([...w.mtgs.map(m=>m.date),...w.calls.map(c=>c.date),...w.ups.map(u=>u.date)])].sort();
    const dayRows=days.map(d=>{
      const dm=w.mtgs.filter(m=>m.date===d),dc=w.calls.filter(c=>c.date===d),du=w.ups.filter(u=>u.date===d);
      const ms=calcMtg(dm),cs=calcCall(dc);
      return`<div style="font-size:13px;padding:5px 0;border-bottom:0.5px solid var(--br);display:flex;gap:8px;flex-wrap:wrap;align-items:center">
        <span style="min-width:100px;color:var(--tx2);font-weight:500">${formatDate(d)}</span>
        <span style="color:var(--tx2)">Mtgs: ${ms.tc}/${ms.total}</span>
        <span style="color:var(--tx2)">· Calls: ${cs.conn}/${cs.total}</span>
        ${du.length?`<span style="color:var(--wt)">${du.length} deal${du.length!==1?"s":""}</span>`:""}
        <span style="margin-left:auto;color:var(--it);font-weight:500">${ms.pts+cs.pts+du.length} pts</span>
      </div>`;
    }).join("");
    return`<div class="week-detail">
      <div style="font-weight:500;font-size:15px;margin-bottom:12px">Week of ${formatDate(w.wk)} — ${formatDate(w.wkEnd)}</div>
      <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px">
        <div><span class="sec-label" style="margin-top:0">Meetings</span>
          <div class="mini-grid">
            ${miniBox("Show rate",w.ms.srt,"var(--it)")}${miniBox("Completed",w.ms.tc,"var(--st)")}
            ${miniBox("Online",w.ms.co)}${miniBox("In person",w.ms.ci)}
            ${miniBox("No shows",w.ms.ns,"var(--dt)")}${miniBox("Cancelled",w.ms.canc,"var(--dt)")}
            ${miniBox("Mtg pts",w.ms.pts,"var(--it)")}${miniBox("Upsells",w.ms.ups,"var(--wt)")}
          </div>
        </div>
        <div><span class="sec-label" style="margin-top:0">Calls</span>
          <div class="mini-grid">
            ${miniBox("Total",w.cs.total)}${miniBox("Connected",w.cs.conn,"var(--st)")}
            ${miniBox("Not connected",w.cs.nc,"var(--dt)")}${miniBox("Quality",w.cs.q,"var(--st)")}
            ${miniBox("Mtg booked",w.cs.b,"var(--wt)")}${miniBox("Brief",w.cs.brief)}
            ${miniBox("Other",w.cs.other)}${miniBox("Call pts",w.cs.pts,"var(--it)")}
          </div>
        </div>
      </div>
      ${w.sup?`<div style="margin-top:12px"><span class="sec-label" style="margin-top:0">Standalone deals</span>
        <div class="mini-grid">${miniBox("Total deals",w.sup,"var(--wt)")}${miniBox("Deal pts",w.sup,"var(--it)")}</div></div>`:""}
      <div class="pts-bar">
        <div style="font-size:13px;color:var(--tx2)">Total points this week
          <div style="font-size:11px;margin-top:2px">${w.ms.pts} mtg + ${w.cs.pts} calls + ${w.sup} deals</div>
        </div>
        <span style="font-size:22px;font-weight:600;color:var(--it);margin-left:auto">${w.ms.pts+w.cs.pts+w.sup} pts</span>
      </div>
      <div style="border-top:0.5px solid var(--br);padding-top:10px;margin-top:14px">
        <span class="sec-label" style="margin-top:0">Daily breakdown</span>
        ${dayRows}
      </div>
    </div>`;
  }).join("");

  return`
    <span class="sec-label">All weeks at a glance</span>
    <div style="overflow-x:auto;margin-bottom:20px;border:0.5px solid var(--br);border-radius:10px">
      <table>
        <thead>
          <tr><th style="text-align:left">Week</th><th colspan="2" style="text-align:center">Meetings</th><th style="text-align:center">Show Rate</th><th style="text-align:center">Mtg Pts</th><th style="text-align:center">Deals</th><th colspan="2" style="text-align:center">Calls</th><th style="text-align:center">Connect%</th><th style="text-align:center">Call Pts</th><th style="text-align:center">Total Pts</th></tr>
          <tr><th style="text-align:left"></th><th style="text-align:center">Booked</th><th style="text-align:center">Done</th><th></th><th></th><th></th><th style="text-align:center">Made</th><th style="text-align:center">Connected</th><th></th><th></th><th></th></tr>
        </thead>
        <tbody>${rows}</tbody>
      </table>
    </div>
    <span class="sec-label">Week detail — click a row above to expand</span>
    ${details}
  `;
}

function renderDaily(){
  const dm=state.meetings.filter(m=>m.date===state.date);
  const dc=state.calls.filter(c=>c.date===state.date);
  const du=state.upsells.filter(u=>u.date===state.date);
  const ms=calcMtg(dm),cs=calcCall(dc);
  const totalPts=ms.pts+cs.pts+du.length;
  const f=state.mtgForm,cf=state.callForm,uf=state.upsellForm;

  const mtgFormHtml=state.showMtgForm?`
    <div class="form-box">
      <div style="font-weight:500;font-size:13px;margin-bottom:10px;color:var(--tx2)">MEETING</div>
      <div style="margin-bottom:10px"><span class="lbl">Contact / company *</span><input value="${f.contact||""}" oninput="updateMtgForm('contact',this.value)" placeholder="Name or company"/></div>
      <div class="grid3">
        <div><span class="lbl">Meeting type</span><select onchange="updateMtgForm('type',this.value)">${selectOpts(MEETING_TYPES,f.type)}</select></div>
        <div><span class="lbl">Format</span><select onchange="updateMtgForm('format',this.value)">${selectOpts(MEETING_FORMATS,f.format)}</select></div>
        <div><span class="lbl">Status *</span><select onchange="updateMtgForm('status',this.value)">${selectOpts(STATUSES,f.status,"Select...")}</select></div>
      </div>
      <div style="margin-bottom:10px"><span class="lbl">Outcome / notes</span><input value="${f.outcome||""}" oninput="updateMtgForm('outcome',this.value)" placeholder="Brief notes..."/></div>
      <div style="margin-bottom:12px;display:flex;align-items:center;gap:12px">
        <label style="display:flex;align-items:center;gap:8px;cursor:pointer;font-size:14px">
          <input type="checkbox" ${f.upsell?"checked":""} onchange="updateMtgForm('upsell',this.checked)" style="width:16px;height:16px"/>
          Successful upsell?
        </label>
        ${f.upsell?`<select onchange="updateMtgForm('upsellType',this.value)" style="flex:1">${selectOpts(UPSELL_TYPES,f.upsellType,"Select upsell type...")}</select>`:""}
      </div>
      ${btn(state.editMtgId?"Update meeting":"Save meeting","saveMtg()","background:var(--sb);color:var(--st);border:0.5px solid var(--sbr)")}
    </div>`:""

  const callFormHtml=state.showCallForm?`
    <div class="form-box">
      <div style="font-weight:500;font-size:13px;margin-bottom:10px;color:var(--tx2)">CALL</div>
      <div class="grid2">
        <div><span class="lbl">Contact / company *</span><input value="${cf.contact||""}" oninput="updateCallForm('contact',this.value)" placeholder="Name or company"/></div>
        <div><span class="lbl">Status *</span><select onchange="updateCallForm('status',this.value)">${selectOpts(CALL_STATUSES,cf.status,"Select...")}</select></div>
      </div>
      <div style="margin-bottom:12px"><span class="lbl">Notes</span><input value="${cf.notes||""}" oninput="updateCallForm('notes',this.value)" placeholder="Optional notes..."/></div>
      ${btn(state.editCallId?"Update call":"Save call","saveCall()","background:var(--sb);color:var(--st);border:0.5px solid var(--sbr)")}
    </div>`:""

  const upsellFormHtml=state.showUpsellForm?`
    <div class="form-box" style="border-color:var(--wbr)">
      <div style="font-weight:500;font-size:13px;margin-bottom:10px;color:var(--wt)">STANDALONE DEAL</div>
      <div class="grid2">
        <div><span class="lbl">Contact / company *</span><input value="${uf.contact||""}" oninput="updateUpsellForm('contact',this.value)" placeholder="Name or company"/></div>
        <div><span class="lbl">Deal type *</span><select onchange="updateUpsellForm('upsellType',this.value)">${selectOpts(UPSELL_TYPES,uf.upsellType,"Select...")}</select></div>
      </div>
      <div style="margin-bottom:12px"><span class="lbl">Notes</span><input value="${uf.notes||""}" oninput="updateUpsellForm('notes',this.value)" placeholder="Optional notes..."/></div>
      ${btn(state.editUpsellId?"Update deal":"Save deal","saveUpsell()","background:var(--wb);color:var(--wt);border:0.5px solid var(--wbr)")}
    </div>`:""

  const summaryStrip=(dm.length||dc.length||du.length)?`
    <div class="summary-strip">
      <div style="font-size:13px"><span style="color:var(--tx2)">Meetings: </span><strong>${ms.tc}/${ms.total}</strong><span style="color:var(--tx2);margin-left:6px">show rate</span><strong style="margin-left:4px">${ms.srt}</strong><span style="color:var(--it);margin-left:8px;font-weight:500">${ms.pts} pts</span></div>
      <div style="font-size:13px"><span style="color:var(--tx2)">Calls: </span><strong>${cs.conn}/${cs.total}</strong><span style="color:var(--tx2);margin-left:6px">connected</span><span style="color:var(--it);margin-left:8px;font-weight:500">${cs.pts} pts</span></div>
      ${du.length?`<div style="font-size:13px"><span style="color:var(--tx2)">Deals: </span><strong>${du.length}</strong><span style="color:var(--it);margin-left:8px;font-weight:500">${du.length} pts</span></div>`:""}
      <div style="margin-left:auto;text-align:right"><div style="font-size:18px;font-weight:600;color:var(--it)">${totalPts} pts today</div><div style="font-size:11px;color:var(--tx2)">${ms.pts} mtg + ${cs.pts} calls + ${du.length} deals</div></div>
    </div>`:"";

  const mtgCards=dm.length?`
    <span class="sec-label">Meetings</span>
    <div class="row" style="margin-bottom:10px">
      ${statBox("Total booked",ms.total)}${statBox("Show rate",ms.srt,"var(--it)")}${statBox("Online ✓",ms.co,"var(--st)")}${statBox("In person ✓",ms.ci,"var(--st)")}${statBox("No show",ms.ns,"var(--dt)")}${statBox("Cancelled",ms.canc,"var(--dt)")}${statBox("Rescheduled",ms.resch,"var(--wt)")}${statBox("Upsells",ms.ups,"var(--wt)")}${statBox("Mtg pts",ms.pts,"var(--it)")}
    </div>
    ${dm.map(m=>`<div class="mtg-card">
      <div class="row">
        <span style="font-weight:500;font-size:14px;flex:1">${m.contact}</span>
        <span class="badge" style="background:var(--bg2);color:var(--tx2)">${m.type}</span>
        <span class="badge" style="background:var(--bg2);color:var(--tx2)">${m.format}</span>
        ${badge(m.status,(STATUS_COLORS[m.status]||{}).bg||"#eee",(STATUS_COLORS[m.status]||{}).tx||"#555")}
        ${btn("Edit",`editMtg('${m.id}')`,"")}
        ${btn("✕",`delMtg('${m.id}')`, "border-color:var(--dbr);color:var(--dt)")}
      </div>
      ${m.outcome?`<div style="font-size:12px;color:var(--tx2);margin-top:4px">${m.outcome}</div>`:""}
      ${m.upsell&&m.upsellType?`<div style="margin-top:4px">${badge("Upsell: "+m.upsellType,"var(--wb)","var(--wt)")}</div>`:""}
    </div>`).join("")}`:"";

  const callCards=dc.length?`
    <span class="sec-label">Calls</span>
    <div class="row" style="margin-bottom:10px">
      ${statBox("Total calls",cs.total)}${statBox("Connected",cs.conn,"var(--st)")}${statBox("Not connected",cs.nc,"var(--dt)")}${statBox("Quality calls",cs.q,"var(--st)")}${statBox("Mtg booked",cs.b,"var(--wt)")}${statBox("Connect %",cs.cr,"var(--it)")}${statBox("Call pts",cs.pts,"var(--it)")}
    </div>
    ${dc.map(c=>`<div class="mtg-card">
      <div class="row">
        <span style="font-weight:500;font-size:14px;flex:1">${c.contact}</span>
        ${badge(c.status,(CALL_COLORS[c.status]||{}).bg||"#eee",(CALL_COLORS[c.status]||{}).tx||"#555")}
        ${CALL_POINTS[c.status]?badge("+"+CALL_POINTS[c.status]+" pt","var(--ib)","var(--it)"):""}
        ${btn("Edit",`editCall('${c.id}')`,"")}
        ${btn("✕",`delCall('${c.id}')`, "border-color:var(--dbr);color:var(--dt)")}
      </div>
      ${c.notes?`<div style="font-size:12px;color:var(--tx2);margin-top:4px">${c.notes}</div>`:""}
    </div>`).join("")}`:"";

  const upsellCards=du.length?`
    <span class="sec-label">Standalone deals</span>
    ${du.map(u=>`<div class="mtg-card" style="border-color:var(--wbr)">
      <div class="row">
        <span style="font-weight:500;font-size:14px;flex:1">${u.contact}</span>
        ${badge(u.upsellType,"var(--wb)","var(--wt)")}
        ${badge("+1 pt","var(--ib)","var(--it)")}
        ${btn("Edit",`editUpsell('${u.id}')`,"")}
        ${btn("✕",`delUpsell('${u.id}')`, "border-color:var(--dbr);color:var(--dt)")}
      </div>
      ${u.notes?`<div style="font-size:12px;color:var(--tx2);margin-top:4px">${u.notes}</div>`:""}
    </div>`).join("")}`:"";

  const empty=(!dm.length&&!dc.length&&!du.length)?`<div style="text-align:center;padding:2rem 0;color:var(--tx3);font-size:14px">Nothing logged for this day yet.</div>`:"";

  return`
    <div class="row" style="margin-bottom:16px">
      <input type="date" value="${state.date}" onchange="setState({date:this.value})" style="width:auto"/>
      <span style="font-size:14px;color:var(--tx2)">${formatDate(state.date)}</span>
      <div style="margin-left:auto;display:flex;gap:8px;flex-wrap:wrap">
        ${btn(state.showMtgForm?"Cancel":"+ Add meeting","toggleForm('mtg')","" )}
        ${btn(state.showCallForm?"Cancel":"+ Log call","toggleForm('call')","background:var(--bg2)")}
        ${btn(state.showUpsellForm?"Cancel":"+ Add deal","toggleForm('upsell')","color:var(--wt);border-color:var(--wbr)")}
      </div>
    </div>
    ${mtgFormHtml}${callFormHtml}${upsellFormHtml}
    ${summaryStrip}${mtgCards}${callCards}${upsellCards}${empty}
  `;
}

function render(){
  const activeTab=(t)=>state.tab===t?"background:var(--ib);color:var(--it);border:0.5px solid var(--ibr);font-weight:500":"";
  document.getElementById("app").innerHTML=`
    <div style="max-width:760px;margin:0 auto">
      <div class="card"><h1 style="font-size:18px;font-weight:500;margin-bottom:4px">Meeting & Call Tracker</h1><p style="font-size:13px;color:var(--tx2)">Log meetings, calls and deals — track outcomes and weekly performance.</p></div>
      <div class="card">
        <div class="row" style="margin-bottom:20px">
          ${btn("Daily log","setState({tab:'daily'})",activeTab("daily"))}
          ${btn("Weekly summary","setState({tab:'weekly'})",activeTab("weekly"))}
          <div style="display:flex;gap:6px;margin-left:auto;flex-wrap:wrap">
            <label style="padding:7px 14px;border-radius:var(--r);border:0.5px solid var(--ibr);background:var(--bg);color:var(--it);font-size:13px;cursor:pointer;font-family:inherit">
              ↑ Import Calendar<input type="file" accept=".ics" onchange="importICS(event)" style="display:none"/>
            </label>
            ${btn("↓ CSV","exportCSV()","color:var(--st);border-color:var(--sbr)")}
            ${btn("↓ Backup","backupData()","color:var(--it);border-color:var(--ibr)")}
            <label style="padding:7px 14px;border-radius:var(--r);border:0.5px solid var(--wbr);background:var(--bg);color:var(--wt);font-size:13px;cursor:pointer;font-family:inherit">
              ↑ Restore<input type="file" accept=".json" onchange="restoreData(event)" style="display:none"/>
            </label>
          </div>
        </div>
        ${state.tab==="daily"?renderDaily():renderWeekly()}
      </div>
    </div>
  `;
}

// Form updaters
function updateMtgForm(k,v){state.mtgForm[k]=v;if(k==="upsell"&&!v)state.mtgForm.upsellType="";render();}
function updateCallForm(k,v){state.callForm[k]=v;render();}
function updateUpsellForm(k,v){state.upsellForm[k]=v;render();}
function toggleForm(t){
  if(t==="mtg"){state.showMtgForm=!state.showMtgForm;state.showCallForm=false;state.showUpsellForm=false;if(!state.showMtgForm){state.editMtgId=null;state.mtgForm={contact:"",type:MEETING_TYPES[0],format:MEETING_FORMATS[0],status:"",outcome:"",upsell:false,upsellType:""};}}
  if(t==="call"){state.showCallForm=!state.showCallForm;state.showMtgForm=false;state.showUpsellForm=false;if(!state.showCallForm){state.editCallId=null;state.callForm={contact:"",status:"",notes:""};}}
  if(t==="upsell"){state.showUpsellForm=!state.showUpsellForm;state.showMtgForm=false;state.showCallForm=false;if(!state.showUpsellForm){state.editUpsellId=null;state.upsellForm={contact:"",upsellType:"",notes:""};}}
  render();
}
function toggleWeek(wk){setState({expandedWeek:state.expandedWeek===wk?null:wk});}

// Save actions
function saveMtg(){
  const f=state.mtgForm;if(!f.contact||!f.status)return;
  if(state.editMtgId){state.meetings=state.meetings.map(m=>String(m.id)===String(state.editMtgId)?{...m,...f,date:state.date,id:m.id}:m);state.editMtgId=null;}
  else state.meetings.push({...f,date:state.date,id:Date.now()});
  state.mtgForm={contact:"",type:MEETING_TYPES[0],format:MEETING_FORMATS[0],status:"",outcome:"",upsell:false,upsellType:""};
  state.showMtgForm=false;setState({});
}
function saveCall(){
  const f=state.callForm;if(!f.contact||!f.status)return;
  if(state.editCallId){state.calls=state.calls.map(c=>c.id===state.editCallId?{...f,date:state.date,id:state.editCallId}:c);state.editCallId=null;}
  else state.calls.push({...f,date:state.date,id:Date.now()});
  state.callForm={contact:"",status:"",notes:""};state.showCallForm=false;setState({});
}
function saveUpsell(){
  const f=state.upsellForm;if(!f.contact||!f.upsellType)return;
  if(state.editUpsellId){state.upsells=state.upsells.map(u=>u.id===state.editUpsellId?{...f,date:state.date,id:state.editUpsellId}:u);state.editUpsellId=null;}
  else state.upsells.push({...f,date:state.date,id:Date.now()});
  state.upsellForm={contact:"",upsellType:"",notes:""};state.showUpsellForm=false;setState({});
}

// Edit
function editMtg(id){const m=state.meetings.find(m=>String(m.id)===String(id));if(!m)return;state.mtgForm={contact:m.contact,type:m.type,format:m.format,status:m.status,outcome:m.outcome,upsell:m.upsell||false,upsellType:m.upsellType||""};state.editMtgId=String(id);state.showMtgForm=true;state.showCallForm=false;state.showUpsellForm=false;render();}
function editCall(id){const c=state.calls.find(c=>c.id==id);if(!c)return;state.callForm={contact:c.contact,status:c.status,notes:c.notes};state.editCallId=id;state.showCallForm=true;state.showMtgForm=false;state.showUpsellForm=false;render();}
function editUpsell(id){const u=state.upsells.find(u=>u.id==id);if(!u)return;state.upsellForm={contact:u.contact,upsellType:u.upsellType,notes:u.notes};state.editUpsellId=id;state.showUpsellForm=true;state.showMtgForm=false;state.showCallForm=false;render();}

// Delete
function delMtg(id){setState({meetings:state.meetings.filter(m=>m.id!=id)});}
function delCall(id){setState({calls:state.calls.filter(c=>c.id!=id)});}
function delUpsell(id){setState({upsells:state.upsells.filter(u=>u.id!=id)});}

// Import/Export/Backup
function importICS(e){
  const file=e.target.files[0];if(!file)return;
  const reader=new FileReader();
  reader.onload=evt=>{
    const text=evt.target.result,events=[],blocks=text.split("BEGIN:VEVENT");
    blocks.slice(1).forEach(block=>{
      const get=key=>{const m=block.match(new RegExp(key+"[^:]*:([^\\r\\n]+)"));return m?m[1].trim():"";};
      const dtRaw=get("DTSTART");if(!dtRaw)return;
      const dateStr=dtRaw.replace(/T\d+Z?$/,"").replace(/(\d{4})(\d{2})(\d{2})/,"$1-$2-$3");
      const summary=get("SUMMARY").replace(/\\,/g,",").replace(/\\n/g," ");
      const desc=get("DESCRIPTION").replace(/\\,/g,",").replace(/\\n/g," ");
      const location=get("LOCATION");
      const format=location&&!location.toLowerCase().includes("http")&&!location.toLowerCase().includes("zoom")&&!location.toLowerCase().includes("meet")&&!location.toLowerCase().includes("teams")?"In Person":"Online";
      const s=(summary+" "+desc).toLowerCase();
      const type=s.includes("intro")?"AM Intro":s.includes("upsell")?"Upsell":s.includes("rms set")?"RMS Set Up":s.includes("rms")?"RMS Discussions":s.includes("follow")||s.includes("f/u")?"Follow-up":"Account Management";
      events.push({dateStr,summary,format,type});
    });
    const day=events.filter(ev=>ev.dateStr===state.date);
    if(!day.length){alert(`No events found for ${formatDate(state.date)}.`);return;}
    const existing=state.meetings.filter(m=>m.date===state.date);
    const newM=day.filter(ev=>!existing.some(m=>m.contact===ev.summary))
      .map(ev=>({id:Date.now()+Math.random(),date:state.date,contact:ev.summary,type:ev.type,format:ev.format,status:"Scheduled",outcome:"",upsell:false,upsellType:""}));
    if(!newM.length){alert("All events already logged.");return;}
    state.meetings.push(...newM);setState({});
    alert(`${newM.length} meeting${newM.length!==1?"s":""} imported.`);
  };
  reader.readAsText(file);e.target.value="";
}
function exportCSV(){
  const esc=v=>`"${(v||"").replace(/"/g,'""')}"`;
  const mH=["Date","Contact","Meeting Type","Format","Status","Outcome","Upsell","Upsell Type"];
  const mR=[...state.meetings].sort((a,b)=>a.date.localeCompare(b.date)).map(m=>[m.date,m.contact,m.type,m.format,m.status,m.outcome,m.upsell?"Yes":"No",m.upsellType||""].map(esc).join(","));
  const cH=["Date","Contact","Call Status","Notes"];
  const cR=[...state.calls].sort((a,b)=>a.date.localeCompare(b.date)).map(c=>[c.date,c.contact,c.status,c.notes].map(esc).join(","));
  const uH=["Date","Contact","Deal Type","Notes"];
  const uR=[...state.upsells].sort((a,b)=>a.date.localeCompare(b.date)).map(u=>[u.date,u.contact,u.upsellType,u.notes].map(esc).join(","));
  const csv=["MEETINGS",mH.join(","),...mR,"","CALLS",cH.join(","),...cR,"","STANDALONE DEALS",uH.join(","),...uR].join("\n");
  Object.assign(document.createElement("a"),{href:URL.createObjectURL(new Blob([csv],{type:"text/csv"})),download:`tracker_${TODAY}.csv`}).click();
}
function backupData(){
  Object.assign(document.createElement("a"),{href:URL.createObjectURL(new Blob([JSON.stringify({meetings:state.meetings,calls:state.calls,upsells:state.upsells},null,2)],{type:"application/json"})),download:`tracker-backup-${TODAY}.json`}).click();
}
function restoreData(e){
  const file=e.target.files[0];if(!file)return;
  const r=new FileReader();
  r.onload=evt=>{try{
    const d=JSON.parse(evt.target.result);
    if(d.meetings&&d.calls){state.meetings=d.meetings;state.calls=d.calls;if(d.upsells)state.upsells=d.upsells;setState({});alert("Restored!");}
    else if(Array.isArray(d)){state.meetings=d;setState({});}
    else alert("Invalid backup file.");
  }catch{alert("Could not read file.");}};
  r.readAsText(file);e.target.value="";
}

render();
</script>
</body>
</html>
