<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Meeting & Call Tracker</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --bg-primary: #ffffff; --bg-secondary: #f5f5f3; --bg-tertiary: #efefec;
    --text-primary: #1a1a18; --text-secondary: #6b6b67; --text-tertiary: #a0a09a;
    --border: rgba(0,0,0,0.12); --border-strong: rgba(0,0,0,0.22);
    --info-bg: #e6f1fb; --info-text: #0c447c; --info-border: rgba(24,95,165,0.3);
    --success-bg: #eaf3de; --success-text: #3b6d11; --success-border: rgba(63,109,17,0.3);
    --danger-bg: #fcebeb; --danger-text: #a32d2d; --danger-border: rgba(163,45,45,0.3);
    --warn-bg: #faeeda; --warn-text: #854f0b; --warn-border: rgba(133,79,11,0.3);
    --radius-md: 8px; --radius-lg: 12px;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg-primary: #1c1c1a; --bg-secondary: #2a2a28; --bg-tertiary: #323230;
      --text-primary: #f0efe8; --text-secondary: #a8a8a2; --text-tertiary: #6b6b67;
      --border: rgba(255,255,255,0.1); --border-strong: rgba(255,255,255,0.2);
      --info-bg: #042c53; --info-text: #b5d4f4; --info-border: rgba(55,138,221,0.3);
      --success-bg: #173404; --success-text: #c0dd97; --success-border: rgba(99,153,34,0.3);
      --danger-bg: #501313; --danger-text: #f7c1c1; --danger-border: rgba(226,75,74,0.3);
      --warn-bg: #412402; --warn-text: #fac775; --warn-border: rgba(186,117,23,0.3);
    }
  }
  body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; background: var(--bg-tertiary); color: var(--text-primary); min-height: 100vh; padding: 1rem; }
  input, select { width: 100%; padding: 7px 10px; border: 0.5px solid var(--border-strong); border-radius: var(--radius-md); font-size: 14px; background: var(--bg-primary); color: var(--text-primary); font-family: inherit; outline: none; }
  input:focus, select:focus { box-shadow: 0 0 0 2px var(--info-border); }
  button { font-family: inherit; cursor: pointer; }
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel" data-presets="react">
const { useState, useEffect } = React;

const MEETING_TYPES = ["Account Management","Follow-up","AM Intro","RMS Set Up","RMS Discussions","Upsell","Other"];
const MEETING_FORMATS = ["Online","In Person"];
const STATUSES = ["Scheduled","Completed","Completed - In Person","No Show","Cancelled","Rescheduled"];
const UPSELL_TYPES = ["RMS Opt Out","Integration","Automation","Integration and Automation","Other"];
const CALL_STATUSES = ["Not Connected","Connected - Quality Call","Connected - Brief Check In","Connected - Meeting Booked","Connected - Other Discussion"];
const CALL_POINTS = { "Connected - Quality Call":1, "Connected - Meeting Booked":1 };

const STATUS_COLORS = {
  "Scheduled":             { bg:"var(--info-bg)",    text:"var(--info-text)" },
  "Completed":             { bg:"var(--success-bg)", text:"var(--success-text)" },
  "Completed - In Person": { bg:"#e1f5ee",           text:"#0f6e56" },
  "No Show":               { bg:"var(--danger-bg)",  text:"var(--danger-text)" },
  "Cancelled":             { bg:"var(--danger-bg)",  text:"var(--danger-text)" },
  "Rescheduled":           { bg:"var(--warn-bg)",    text:"var(--warn-text)" },
};
const CALL_COLORS = {
  "Not Connected":                { bg:"#f0f0ee", text:"#6b6b67" },
  "Connected - Quality Call":     { bg:"var(--success-bg)", text:"var(--success-text)" },
  "Connected - Brief Check In":   { bg:"var(--info-bg)",    text:"var(--info-text)" },
  "Connected - Meeting Booked":   { bg:"var(--warn-bg)",    text:"var(--warn-text)" },
  "Connected - Other Discussion": { bg:"#f3e6fb", text:"#5c0c7c" },
};

const TODAY = new Date().toISOString().split("T")[0];
const EMPTY_MTG    = { contact:"", type:MEETING_TYPES[0], format:MEETING_FORMATS[0], status:"", outcome:"", upsell:false, upsellType:"" };
const EMPTY_CALL   = { contact:"", status:"", notes:"" };
const EMPTY_UPSELL = { contact:"", upsellType:"", notes:"" };
const secLabel = { fontSize:11, fontWeight:500, color:"var(--text-secondary)", textTransform:"uppercase", letterSpacing:"0.05em", marginBottom:6, marginTop:12, display:"block" };

function getWeekKey(d) {
  const dt=new Date(d), mon=new Date(dt);
  mon.setDate(dt.getDate()-((dt.getDay()+6)%7));
  return mon.toISOString().split("T")[0];
}
function formatDate(d) {
  return new Date(d+"T00:00:00").toLocaleDateString("en-US",{weekday:"short",month:"short",day:"numeric"});
}
function calcMtgStats(arr) {
  const online=arr.filter(m=>m.format==="Online");
  const inPerson=arr.filter(m=>m.format==="In Person");
  const compOnline=online.filter(m=>m.status==="Completed").length;
  const compInPerson=inPerson.filter(m=>m.status==="Completed - In Person"||(m.status==="Completed"&&m.format==="In Person")).length;
  const totalComp=arr.filter(m=>m.status.startsWith("Completed")).length;
  const upsells=arr.filter(m=>m.upsell&&m.upsellType).length;
  return {
    total:arr.length, compOnline, compInPerson, totalComp, upsells,
    ns:arr.filter(m=>m.status==="No Show").length,
    canc:arr.filter(m=>m.status==="Cancelled").length,
    resch:arr.filter(m=>m.status==="Rescheduled").length,
    points:compOnline*1+compInPerson*2+upsells,
    showRateOnline:online.length?Math.round((compOnline/online.length)*100)+"%":"—",
    showRateInPerson:inPerson.length?Math.round((compInPerson/inPerson.length)*100)+"%":"—",
    showRateTotal:arr.length?Math.round((totalComp/arr.length)*100)+"%":"—",
  };
}
function calcCallStats(arr) {
  const connected=arr.filter(c=>c.status!=="Not Connected");
  const quality=arr.filter(c=>c.status==="Connected - Quality Call").length;
  const booked=arr.filter(c=>c.status==="Connected - Meeting Booked").length;
  return {
    total:arr.length, connected:connected.length,
    notConnected:arr.filter(c=>c.status==="Not Connected").length,
    quality, booked,
    brief:arr.filter(c=>c.status==="Connected - Brief Check In").length,
    other:arr.filter(c=>c.status==="Connected - Other Discussion").length,
    points:quality+booked,
    connectRate:arr.length?Math.round((connected.length/arr.length)*100)+"%":"—",
  };
}

function MiniStats({ items }) {
  return (
    <div style={{ display:"flex", flexWrap:"wrap", gap:6 }}>
      {items.map(([l,v,c])=>(
        <div key={l} style={{ background:"var(--bg-secondary)", borderRadius:8, padding:"6px 10px", flex:"1 1 80px" }}>
          <div style={{ fontSize:10, color:"var(--text-secondary)", marginBottom:1 }}>{l}</div>
          <div style={{ fontSize:15, fontWeight:500, color:c||"var(--text-primary)" }}>{v}</div>
        </div>
      ))}
    </div>
  );
}

function WeeklyView({ data=[] }) {
  const [expanded, setExpanded] = useState(null);
  const toggle = wk=>setExpanded(e=>e===wk?null:wk);
  const th = { fontSize:11, fontWeight:500, color:"var(--text-secondary)", padding:"6px 8px", textAlign:"center", borderBottom:"0.5px solid var(--border)", whiteSpace:"nowrap" };
  const td = (c) => ({ fontSize:13, padding:"7px 8px", borderBottom:"0.5px solid var(--border)", color:c||"var(--text-primary)", textAlign:"center", whiteSpace:"nowrap" });

  return (
    <>
      <span style={secLabel}>All weeks at a glance</span>
      <div style={{ overflowX:"auto", marginBottom:20, border:"0.5px solid var(--border)", borderRadius:10 }}>
        <table style={{ width:"100%", borderCollapse:"collapse", fontSize:13 }}>
          <thead>
            <tr style={{ background:"var(--bg-secondary)" }}>
              <th style={{...th,textAlign:"left"}}>Week</th>
              <th style={th} colSpan={2}>Meetings</th>
              <th style={th}>Show Rate</th>
              <th style={th}>Mtg Pts</th>
              <th style={th}>Deals</th>
              <th style={th} colSpan={2}>Calls</th>
              <th style={th}>Connect %</th>
              <th style={th}>Call Pts</th>
              <th style={th}>Total Pts</th>
            </tr>
            <tr style={{ background:"var(--bg-secondary)" }}>
              <th style={{...th,textAlign:"left"}}></th>
              <th style={th}>Booked</th><th style={th}>Done</th>
              <th style={th}></th><th style={th}></th><th style={th}></th>
              <th style={th}>Made</th><th style={th}>Connected</th>
              <th style={th}></th><th style={th}></th><th style={th}></th>
            </tr>
          </thead>
          <tbody>
            {data.map(w=>(
              <tr key={w.wk} onClick={()=>toggle(w.wk)} style={{ cursor:"pointer", background:expanded===w.wk?"var(--info-bg)":"transparent" }}>
                <td style={{...td("var(--text-secondary)"),textAlign:"left"}}>{formatDate(w.wk)}</td>
                <td style={td()}>{w.ms.total}</td>
                <td style={td("var(--success-text)")}>{w.ms.totalComp}</td>
                <td style={td("var(--info-text)")}>{w.ms.showRateTotal}</td>
                <td style={td("var(--info-text)")}>{w.ms.points}</td>
                <td style={td("var(--warn-text)")}>{w.ms.upsells+w.standaloneUpsellPts}</td>
                <td style={td()}>{w.cs.total}</td>
                <td style={td("var(--success-text)")}>{w.cs.connected}</td>
                <td style={td("var(--info-text)")}>{w.cs.connectRate}</td>
                <td style={td("var(--info-text)")}>{w.cs.points}</td>
                <td style={{...td("var(--info-text)"),fontWeight:600}}>{w.ms.points+w.cs.points+w.standaloneUpsellPts}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      <span style={secLabel}>Week detail — click a row above to expand</span>
      {data.map(w=>expanded===w.wk&&(
        <div key={w.wk} style={{ border:"0.5px solid var(--info-border)", borderRadius:12, padding:"1rem 1.25rem", marginBottom:14 }}>
          <div style={{ fontWeight:500, fontSize:15, marginBottom:12 }}>Week of {formatDate(w.wk)} — {formatDate(w.wkEnd)}</div>
          <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12 }}>
            <div>
              <span style={{...secLabel,marginTop:0}}>Meetings</span>
              <MiniStats items={[["Show rate",w.ms.showRateTotal,"var(--info-text)"],["Completed",w.ms.totalComp,"var(--success-text)"],["Online",w.ms.compOnline],["In person",w.ms.compInPerson],["No shows",w.ms.ns,"var(--danger-text)"],["Cancelled",w.ms.canc,"var(--danger-text)"],["Mtg pts",w.ms.points,"var(--info-text)"],["Upsells (mtg)",w.ms.upsells,"var(--warn-text)"]]} />
            </div>
            <div>
              <span style={{...secLabel,marginTop:0}}>Calls</span>
              <MiniStats items={[["Total calls",w.cs.total],["Connected",w.cs.connected,"var(--success-text)"],["Not connected",w.cs.notConnected,"var(--danger-text)"],["Quality calls",w.cs.quality,"var(--success-text)"],["Mtg booked",w.cs.booked,"var(--warn-text)"],["Brief check in",w.cs.brief],["Other",w.cs.other],["Call pts",w.cs.points,"var(--info-text)"]]} />
            </div>
          </div>
          {w.standaloneUpsellPts>0&&(
            <div style={{ marginTop:12 }}>
              <span style={{...secLabel,marginTop:0}}>Standalone deals</span>
              <MiniStats items={[["Total deals",w.standaloneUpsellPts,"var(--warn-text)"],["Deal pts",w.standaloneUpsellPts,"var(--info-text)"]]} />
            </div>
          )}
          <div style={{ marginTop:12, padding:"10px 14px", background:"var(--info-bg)", borderRadius:8, display:"flex", alignItems:"center" }}>
            <div style={{ fontSize:13, color:"var(--text-secondary)" }}>
              Total points this week
              <div style={{ fontSize:11, marginTop:2 }}>{w.ms.points} mtg + {w.cs.points} calls + {w.standaloneUpsellPts} deals</div>
            </div>
            <span style={{ fontSize:22, fontWeight:600, color:"var(--info-text)", marginLeft:"auto" }}>{w.ms.points+w.cs.points+w.standaloneUpsellPts} pts</span>
          </div>
          <div style={{ borderTop:"0.5px solid var(--border)", paddingTop:10, marginTop:14 }}>
            <span style={{...secLabel,marginTop:0}}>Daily breakdown</span>
            {[...new Set([...w.mtgs.map(m=>m.date),...w.calls.map(c=>c.date),...w.ups.map(u=>u.date)])].sort().map(d=>{
              const dm=w.mtgs.filter(m=>m.date===d); const dc=w.calls.filter(c=>c.date===d); const du=w.ups.filter(u=>u.date===d);
              const ms=calcMtgStats(dm); const cs=calcCallStats(dc);
              return (
                <div key={d} style={{ fontSize:13, padding:"5px 0", borderBottom:"0.5px solid var(--border)" }}>
                  <div style={{ display:"flex", gap:8, flexWrap:"wrap", alignItems:"center" }}>
                    <span style={{ minWidth:100, color:"var(--text-secondary)", fontWeight:500 }}>{formatDate(d)}</span>
                    <span style={{ color:"var(--text-secondary)" }}>Mtgs: {ms.totalComp}/{ms.total}</span>
                    <span style={{ color:"var(--text-secondary)" }}>·</span>
                    <span style={{ color:"var(--text-secondary)" }}>Calls: {cs.connected}/{cs.total}</span>
                    {du.length>0&&<span style={{ color:"var(--warn-text)" }}>{du.length} deal{du.length!==1?"s":""}</span>}
                    <span style={{ marginLeft:"auto", color:"var(--info-text)", fontWeight:500 }}>{ms.points+cs.points+du.length} pts</span>
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      ))}
    </>
  );
}

function App() {
  const [tab, setTab] = useState("daily");
  const [selectedDate, setSelectedDate] = useState(TODAY);
  const [meetings, setMeetings] = useState(()=>{ try{return JSON.parse(localStorage.getItem("mtg_data2")||"[]");}catch{return[];} });
  const [calls, setCalls] = useState(()=>{ try{return JSON.parse(localStorage.getItem("call_data")||"[]");}catch{return[];} });
  const [upsells, setUpsells] = useState(()=>{ try{return JSON.parse(localStorage.getItem("upsell_data")||"[]");}catch{return[];} });
  const [mtgForm, setMtgForm] = useState(EMPTY_MTG);
  const [callForm, setCallForm] = useState(EMPTY_CALL);
  const [upsellForm, setUpsellForm] = useState(EMPTY_UPSELL);
  const [editMtgId, setEditMtgId] = useState(null);
  const [editCallId, setEditCallId] = useState(null);
  const [editUpsellId, setEditUpsellId] = useState(null);
  const [showMtgForm, setShowMtgForm] = useState(false);
  const [showCallForm, setShowCallForm] = useState(false);
  const [showUpsellForm, setShowUpsellForm] = useState(false);

  useEffect(()=>{ try{localStorage.setItem("mtg_data2",JSON.stringify(meetings));}catch{} },[meetings]);
  useEffect(()=>{ try{localStorage.setItem("call_data",JSON.stringify(calls));}catch{} },[calls]);
  useEffect(()=>{ try{localStorage.setItem("upsell_data",JSON.stringify(upsells));}catch{} },[upsells]);

  const dayMtgs=meetings.filter(m=>m.date===selectedDate);
  const dayCalls=calls.filter(c=>c.date===selectedDate);
  const dayUpsells=upsells.filter(u=>u.date===selectedDate);

  function saveMtg() {
    if (!mtgForm.contact||!mtgForm.status) return;
    if (editMtgId){setMeetings(p=>p.map(m=>m.id===editMtgId?{...mtgForm,date:selectedDate,id:editMtgId}:m));setEditMtgId(null);}
    else setMeetings(p=>[...p,{...mtgForm,date:selectedDate,id:Date.now()}]);
    setMtgForm(EMPTY_MTG);setShowMtgForm(false);
  }
  function saveCall() {
    if (!callForm.contact||!callForm.status) return;
    if (editCallId){setCalls(p=>p.map(c=>c.id===editCallId?{...callForm,date:selectedDate,id:editCallId}:c));setEditCallId(null);}
    else setCalls(p=>[...p,{...callForm,date:selectedDate,id:Date.now()}]);
    setCallForm(EMPTY_CALL);setShowCallForm(false);
  }
  function saveUpsell() {
    if (!upsellForm.contact||!upsellForm.upsellType) return;
    if (editUpsellId){setUpsells(p=>p.map(u=>u.id===editUpsellId?{...upsellForm,date:selectedDate,id:editUpsellId}:u));setEditUpsellId(null);}
    else setUpsells(p=>[...p,{...upsellForm,date:selectedDate,id:Date.now()}]);
    setUpsellForm(EMPTY_UPSELL);setShowUpsellForm(false);
  }
  function startEditMtg(m){setMtgForm({contact:m.contact,type:m.type,format:m.format,status:m.status,outcome:m.outcome,upsell:m.upsell||false,upsellType:m.upsellType||""});setEditMtgId(m.id);setShowMtgForm(true);setShowCallForm(false);setShowUpsellForm(false);}
  function startEditCall(c){setCallForm({contact:c.contact,status:c.status,notes:c.notes});setEditCallId(c.id);setShowCallForm(true);setShowMtgForm(false);setShowUpsellForm(false);}
  function startEditUpsell(u){setUpsellForm({contact:u.contact,upsellType:u.upsellType,notes:u.notes});setEditUpsellId(u.id);setShowUpsellForm(true);setShowMtgForm(false);setShowCallForm(false);}

  function parseICS(text) {
    const events=[];
    const blocks=text.split("BEGIN:VEVENT");
    blocks.slice(1).forEach(block=>{
      const get=key=>{const m=block.match(new RegExp(key+"[^:]*:([^\\r\\n]+)"));return m?m[1].trim():"";};
      const dtRaw=get("DTSTART");if(!dtRaw)return;
      const dateStr=dtRaw.replace(/T\d+Z?$/,"").replace(/(\d{4})(\d{2})(\d{2})/,"$1-$2-$3");
      const summary=get("SUMMARY").replace(/\\,/g,",").replace(/\\n/g," ");
      const desc=get("DESCRIPTION").replace(/\\,/g,",").replace(/\\n/g," ");
      const location=get("LOCATION");
      const format=location&&!location.toLowerCase().includes("http")&&!location.toLowerCase().includes("zoom")&&!location.toLowerCase().includes("meet")&&!location.toLowerCase().includes("teams")?"In Person":"Online";
      const typeGuess=(()=>{const s=(summary+" "+desc).toLowerCase();
        if(s.includes("intro"))return "AM Intro";if(s.includes("upsell"))return "Upsell";
        if(s.includes("rms set"))return "RMS Set Up";if(s.includes("rms"))return "RMS Discussions";
        if(s.includes("follow")||s.includes("f/u"))return "Follow-up";return "Account Management";
      })();
      events.push({dateStr,summary,format,typeGuess});
    });
    return events;
  }
  function importICS(e) {
    const file=e.target.files[0];if(!file)return;
    const reader=new FileReader();
    reader.onload=evt=>{
      const events=parseICS(evt.target.result).filter(ev=>ev.dateStr===selectedDate);
      if(!events.length){alert(`No events found for ${formatDate(selectedDate)}.`);return;}
      const existing=meetings.filter(m=>m.date===selectedDate);
      const newM=events.filter(ev=>!existing.some(m=>m.contact===ev.summary))
        .map(ev=>({id:Date.now()+Math.random(),date:selectedDate,contact:ev.summary,type:ev.typeGuess,format:ev.format,status:"Scheduled",outcome:"",upsell:false,upsellType:""}));
      if(!newM.length){alert("All events already logged.");return;}
      setMeetings(prev=>[...prev,...newM]);
      alert(`${newM.length} meeting${newM.length!==1?"s":""} imported.`);
    };
    reader.readAsText(file);e.target.value="";
  }
  function exportCSV() {
    const esc=v=>`"${(v||"").replace(/"/g,'""')}"`;
    const mHdr=["Date","Contact","Meeting Type","Format","Status","Outcome","Upsell","Upsell Type"];
    const mRows=[...meetings].sort((a,b)=>a.date.localeCompare(b.date)).map(m=>[m.date,m.contact,m.type,m.format,m.status,m.outcome,m.upsell?"Yes":"No",m.upsellType||""].map(esc).join(","));
    const cHdr=["Date","Contact","Call Status","Notes"];
    const cRows=[...calls].sort((a,b)=>a.date.localeCompare(b.date)).map(c=>[c.date,c.contact,c.status,c.notes].map(esc).join(","));
    const uHdr=["Date","Contact","Deal Type","Notes"];
    const uRows=[...upsells].sort((a,b)=>a.date.localeCompare(b.date)).map(u=>[u.date,u.contact,u.upsellType,u.notes].map(esc).join(","));
    const csv=["MEETINGS",mHdr.join(","),...mRows,"","CALLS",cHdr.join(","),...cRows,"","STANDALONE DEALS",uHdr.join(","),...uRows].join("\n");
    Object.assign(document.createElement("a"),{href:URL.createObjectURL(new Blob([csv],{type:"text/csv"})),download:`tracker_${TODAY}.csv`}).click();
  }
  function backupData() {
    Object.assign(document.createElement("a"),{href:URL.createObjectURL(new Blob([JSON.stringify({meetings,calls,upsells},null,2)],{type:"application/json"})),download:`tracker-backup-${TODAY}.json`}).click();
  }
  function restoreData(e) {
    const file=e.target.files[0];if(!file)return;
    const r=new FileReader();
    r.onload=evt=>{try{const d=JSON.parse(evt.target.result);
      if(d.meetings&&d.calls){setMeetings(d.meetings);setCalls(d.calls);if(d.upsells)setUpsells(d.upsells);alert("Restored!");}
      else if(Array.isArray(d)){setMeetings(d);}
      else alert("Invalid backup file.");
    }catch{alert("Could not read file.");}};
    r.readAsText(file);e.target.value="";
  }
  function weeklyData() {
    const weeks={};
    meetings.forEach(m=>{const k=getWeekKey(m.date);if(!weeks[k])weeks[k]={mtgs:[],calls:[],ups:[]};weeks[k].mtgs.push(m);});
    calls.forEach(c=>{const k=getWeekKey(c.date);if(!weeks[k])weeks[k]={mtgs:[],calls:[],ups:[]};weeks[k].calls.push(c);});
    upsells.forEach(u=>{const k=getWeekKey(u.date);if(!weeks[k])weeks[k]={mtgs:[],calls:[],ups:[]};weeks[k].ups.push(u);});
    return Object.entries(weeks).sort((a,b)=>b[0].localeCompare(a[0])).map(([wk,{mtgs,calls:wc,ups}])=>{
      const end=new Date(wk+"T00:00:00");end.setDate(end.getDate()+6);
      return{wk,mtgs,calls:wc,ups,ms:calcMtgStats(mtgs),cs:calcCallStats(wc),standaloneUpsellPts:ups.length,wkEnd:end.toISOString().split("T")[0]};
    });
  }

  const inp={width:"100%",padding:"7px 10px",border:"0.5px solid var(--border-strong)",borderRadius:"var(--radius-md)",fontSize:14,background:"var(--bg-primary)",color:"var(--text-primary)"};
  const btn=(extra={})=>({padding:"7px 14px",borderRadius:"var(--radius-md)",border:"0.5px solid var(--border-strong)",background:"var(--bg-primary)",color:"var(--text-primary)",fontSize:13,cursor:"pointer",...extra});
  const lbl={fontSize:12,color:"var(--text-secondary)",marginBottom:3,display:"block"};
  const Stat=({label,value,color})=>(
    <div style={{background:"var(--bg-secondary)",borderRadius:"var(--radius-md)",padding:"10px 12px",flex:1,minWidth:70}}>
      <div style={{fontSize:11,color:"var(--text-secondary)",marginBottom:2}}>{label}</div>
      <div style={{fontSize:18,fontWeight:500,color:color||"var(--text-primary)"}}>{value}</div>
    </div>
  );
  const Badge=({label,colors})=>(
    <span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:colors?.bg||"#eee",color:colors?.text||"#555",whiteSpace:"nowrap"}}>{label}</span>
  );

  const ms=calcMtgStats(dayMtgs);
  const cs=calcCallStats(dayCalls);
  const totalDayPts=ms.points+cs.points+dayUpsells.length;

  return (
    <div style={{maxWidth:760,margin:"0 auto"}}>
      <div style={{background:"var(--bg-primary)",borderRadius:"var(--radius-lg)",border:"0.5px solid var(--border)",padding:"1.25rem",marginBottom:12}}>
        <h1 style={{fontSize:18,fontWeight:500,marginBottom:4}}>Meeting & Call Tracker</h1>
        <p style={{fontSize:13,color:"var(--text-secondary)"}}>Log meetings, calls and deals — track outcomes and weekly performance.</p>
      </div>
      <div style={{background:"var(--bg-primary)",borderRadius:"var(--radius-lg)",border:"0.5px solid var(--border)",padding:"1.25rem"}}>
        <div style={{display:"flex",gap:8,marginBottom:20,flexWrap:"wrap"}}>
          {["daily","weekly"].map(t=>(
            <button key={t} onClick={()=>setTab(t)} style={btn({background:tab===t?"var(--info-bg)":"var(--bg-primary)",color:tab===t?"var(--info-text)":"var(--text-primary)",border:tab===t?"0.5px solid var(--info-border)":"0.5px solid var(--border-strong)",fontWeight:tab===t?500:400})}>
              {t==="daily"?"Daily log":"Weekly summary"}
            </button>
          ))}
          <div style={{display:"flex",gap:6,marginLeft:"auto",flexWrap:"wrap"}}>
            <label style={{...btn({color:"var(--info-text)",border:"0.5px solid var(--info-border)",cursor:"pointer"}),display:"inline-block"}}>
              ↑ Import Calendar<input type="file" accept=".ics" onChange={importICS} style={{display:"none"}}/>
            </label>
            <button onClick={exportCSV} style={btn({color:"var(--success-text)",border:"0.5px solid var(--success-border)"})}>↓ CSV</button>
            <button onClick={backupData} style={btn({color:"var(--info-text)",border:"0.5px solid var(--info-border)"})}>↓ Backup</button>
            <label style={{...btn({color:"var(--warn-text)",border:"0.5px solid var(--warn-border)",cursor:"pointer"}),display:"inline-block"}}>
              ↑ Restore<input type="file" accept=".json" onChange={restoreData} style={{display:"none"}}/>
            </label>
          </div>
        </div>

        {tab==="daily"&&(
          <>
            <div style={{display:"flex",alignItems:"center",gap:12,marginBottom:16,flexWrap:"wrap"}}>
              <input type="date" value={selectedDate} onChange={e=>setSelectedDate(e.target.value)} style={{...inp,width:"auto"}}/>
              <span style={{fontSize:14,color:"var(--text-secondary)"}}>{formatDate(selectedDate)}</span>
              <div style={{marginLeft:"auto",display:"flex",gap:8,flexWrap:"wrap"}}>
                <button onClick={()=>{setMtgForm(EMPTY_MTG);setEditMtgId(null);setShowMtgForm(s=>!s);setShowCallForm(false);setShowUpsellForm(false);}} style={btn()}>
                  {showMtgForm?"Cancel":"+ Add meeting"}
                </button>
                <button onClick={()=>{setCallForm(EMPTY_CALL);setEditCallId(null);setShowCallForm(s=>!s);setShowMtgForm(false);setShowUpsellForm(false);}} style={btn({background:"var(--bg-secondary)"})}>
                  {showCallForm?"Cancel":"+ Log call"}
                </button>
                <button onClick={()=>{setUpsellForm(EMPTY_UPSELL);setEditUpsellId(null);setShowUpsellForm(s=>!s);setShowMtgForm(false);setShowCallForm(false);}} style={btn({color:"var(--warn-text)",border:"0.5px solid var(--warn-border)"})}>
                  {showUpsellForm?"Cancel":"+ Add deal"}
                </button>
              </div>
            </div>

            {showMtgForm&&(
              <div style={{background:"var(--bg-secondary)",borderRadius:"var(--radius-lg)",padding:"1rem",marginBottom:16,border:"0.5px solid var(--border)"}}>
                <div style={{fontWeight:500,fontSize:13,marginBottom:10,color:"var(--text-secondary)"}}>MEETING</div>
                <div style={{marginBottom:10}}><span style={lbl}>Contact / company *</span><input value={mtgForm.contact} onChange={e=>setMtgForm(f=>({...f,contact:e.target.value}))} placeholder="Name or company" style={inp}/></div>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:10,marginBottom:10}}>
                  {[["Meeting type","type",MEETING_TYPES],["Format","format",MEETING_FORMATS],["Status *","status",STATUSES]].map(([label,key,opts])=>(
                    <div key={key}><span style={lbl}>{label}</span>
                      <select value={mtgForm[key]} onChange={e=>setMtgForm(f=>({...f,[key]:e.target.value}))} style={inp}>
                        {key==="status"&&<option value="">Select...</option>}
                        {opts.map(o=><option key={o}>{o}</option>)}
                      </select>
                    </div>
                  ))}
                </div>
                <div style={{marginBottom:10}}><span style={lbl}>Outcome / notes</span><input value={mtgForm.outcome} onChange={e=>setMtgForm(f=>({...f,outcome:e.target.value}))} placeholder="Brief notes..." style={inp}/></div>
                <div style={{marginBottom:12,display:"flex",alignItems:"center",gap:12}}>
                  <label style={{display:"flex",alignItems:"center",gap:8,cursor:"pointer",fontSize:14}}>
                    <input type="checkbox" checked={mtgForm.upsell} onChange={e=>setMtgForm(f=>({...f,upsell:e.target.checked,upsellType:""}))} style={{width:16,height:16}}/>
                    Successful upsell?
                  </label>
                  {mtgForm.upsell&&(
                    <select value={mtgForm.upsellType} onChange={e=>setMtgForm(f=>({...f,upsellType:e.target.value}))} style={{...inp,flex:1}}>
                      <option value="">Select upsell type...</option>
                      {UPSELL_TYPES.map(u=><option key={u}>{u}</option>)}
                    </select>
                  )}
                </div>
                <button onClick={saveMtg} style={btn({background:"var(--success-bg)",color:"var(--success-text)",border:"0.5px solid var(--success-border)"})}>
                  {editMtgId?"Update meeting":"Save meeting"}
                </button>
              </div>
            )}

            {showCallForm&&(
              <div style={{background:"var(--bg-secondary)",borderRadius:"var(--radius-lg)",padding:"1rem",marginBottom:16,border:"0.5px solid var(--border)"}}>
                <div style={{fontWeight:500,fontSize:13,marginBottom:10,color:"var(--text-secondary)"}}>CALL</div>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
                  <div><span style={lbl}>Contact / company *</span><input value={callForm.contact} onChange={e=>setCallForm(f=>({...f,contact:e.target.value}))} placeholder="Name or company" style={inp}/></div>
                  <div><span style={lbl}>Status *</span>
                    <select value={callForm.status} onChange={e=>setCallForm(f=>({...f,status:e.target.value}))} style={inp}>
                      <option value="">Select...</option>
                      {CALL_STATUSES.map(s=><option key={s}>{s}</option>)}
                    </select>
                  </div>
                </div>
                <div style={{marginBottom:12}}><span style={lbl}>Notes</span><input value={callForm.notes} onChange={e=>setCallForm(f=>({...f,notes:e.target.value}))} placeholder="Optional notes..." style={inp}/></div>
                <button onClick={saveCall} style={btn({background:"var(--success-bg)",color:"var(--success-text)",border:"0.5px solid var(--success-border)"})}>
                  {editCallId?"Update call":"Save call"}
                </button>
              </div>
            )}

            {showUpsellForm&&(
              <div style={{background:"var(--bg-secondary)",borderRadius:"var(--radius-lg)",padding:"1rem",marginBottom:16,border:"0.5px solid var(--warn-border)"}}>
                <div style={{fontWeight:500,fontSize:13,marginBottom:10,color:"var(--warn-text)"}}>STANDALONE DEAL</div>
                <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:10}}>
                  <div><span style={lbl}>Contact / company *</span><input value={upsellForm.contact} onChange={e=>setUpsellForm(f=>({...f,contact:e.target.value}))} placeholder="Name or company" style={inp}/></div>
                  <div><span style={lbl}>Deal type *</span>
                    <select value={upsellForm.upsellType} onChange={e=>setUpsellForm(f=>({...f,upsellType:e.target.value}))} style={inp}>
                      <option value="">Select...</option>
                      {UPSELL_TYPES.map(u=><option key={u}>{u}</option>)}
                    </select>
                  </div>
                </div>
                <div style={{marginBottom:12}}><span style={lbl}>Notes</span><input value={upsellForm.notes} onChange={e=>setUpsellForm(f=>({...f,notes:e.target.value}))} placeholder="Optional notes..." style={inp}/></div>
                <button onClick={saveUpsell} style={btn({background:"var(--warn-bg)",color:"var(--warn-text)",border:"0.5px solid var(--warn-border)"})}>
                  {editUpsellId?"Update deal":"Save deal"}
                </button>
              </div>
            )}

            {(dayMtgs.length>0||dayCalls.length>0||dayUpsells.length>0)?(
              <>
                <div style={{background:"var(--info-bg)",borderRadius:10,padding:"10px 14px",marginBottom:16,display:"flex",flexWrap:"wrap",gap:16,alignItems:"center"}}>
                  <div style={{fontSize:13}}>
                    <span style={{color:"var(--text-secondary)"}}>Meetings: </span>
                    <span style={{fontWeight:500}}>{ms.totalComp}/{ms.total}</span>
                    <span style={{color:"var(--text-secondary)",marginLeft:6}}>show rate</span>
                    <span style={{fontWeight:500,marginLeft:4}}>{ms.showRateTotal}</span>
                    <span style={{color:"var(--info-text)",marginLeft:8,fontWeight:500}}>{ms.points} pts</span>
                  </div>
                  <div style={{fontSize:13}}>
                    <span style={{color:"var(--text-secondary)"}}>Calls: </span>
                    <span style={{fontWeight:500}}>{cs.connected}/{cs.total}</span>
                    <span style={{color:"var(--text-secondary)",marginLeft:6}}>connected</span>
                    <span style={{color:"var(--info-text)",marginLeft:8,fontWeight:500}}>{cs.points} pts</span>
                  </div>
                  {dayUpsells.length>0&&(
                    <div style={{fontSize:13}}>
                      <span style={{color:"var(--text-secondary)"}}>Deals: </span>
                      <span style={{fontWeight:500}}>{dayUpsells.length}</span>
                      <span style={{color:"var(--info-text)",marginLeft:8,fontWeight:500}}>{dayUpsells.length} pts</span>
                    </div>
                  )}
                  <div style={{marginLeft:"auto",textAlign:"right"}}>
                    <div style={{fontSize:18,fontWeight:600,color:"var(--info-text)"}}>{totalDayPts} pts today</div>
                    <div style={{fontSize:11,color:"var(--text-secondary)"}}>{ms.points} mtg + {cs.points} calls + {dayUpsells.length} deals</div>
                  </div>
                </div>

                {dayMtgs.length>0&&(
                  <>
                    <span style={secLabel}>Meetings</span>
                    <div style={{display:"flex",gap:8,flexWrap:"wrap",marginBottom:10}}>
                      <Stat label="Total booked" value={ms.total}/>
                      <Stat label="Show rate" value={ms.showRateTotal} color="var(--info-text)"/>
                      <Stat label="Online ✓" value={ms.compOnline} color="var(--success-text)"/>
                      <Stat label="In person ✓" value={ms.compInPerson} color="var(--success-text)"/>
                      <Stat label="No show" value={ms.ns} color="var(--danger-text)"/>
                      <Stat label="Cancelled" value={ms.canc} color="var(--danger-text)"/>
                      <Stat label="Rescheduled" value={ms.resch} color="var(--warn-text)"/>
                      <Stat label="Upsells" value={ms.upsells} color="var(--warn-text)"/>
                      <Stat label="Mtg pts" value={ms.points} color="var(--info-text)"/>
                    </div>
                    <div style={{display:"flex",flexDirection:"column",gap:6,marginBottom:16}}>
                      {dayMtgs.map(m=>(
                        <div key={m.id} style={{background:"var(--bg-primary)",border:"0.5px solid var(--border)",borderRadius:10,padding:"9px 12px"}}>
                          <div style={{display:"flex",alignItems:"center",gap:8,flexWrap:"wrap"}}>
                            <span style={{fontWeight:500,fontSize:14,flex:1}}>{m.contact}</span>
                            <span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:"var(--bg-secondary)",color:"var(--text-secondary)"}}>{m.type}</span>
                            <span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:"var(--bg-secondary)",color:"var(--text-secondary)"}}>{m.format}</span>
                            <Badge label={m.status} colors={STATUS_COLORS[m.status]}/>
                            <button onClick={()=>startEditMtg(m)} style={{fontSize:12,padding:"3px 8px",borderRadius:6,border:"0.5px solid var(--border-strong)",background:"transparent",cursor:"pointer",color:"var(--text-secondary)"}}>Edit</button>
                            <button onClick={()=>setMeetings(p=>p.filter(x=>x.id!==m.id))} style={{fontSize:12,padding:"3px 8px",borderRadius:6,border:"0.5px solid var(--danger-border)",background:"transparent",cursor:"pointer",color:"var(--danger-text)"}}>✕</button>
                          </div>
                          {m.outcome&&<div style={{fontSize:12,color:"var(--text-secondary)",marginTop:4}}>{m.outcome}</div>}
                          {m.upsell&&m.upsellType&&<div style={{marginTop:4}}><span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:"var(--warn-bg)",color:"var(--warn-text)",fontWeight:500}}>Upsell: {m.upsellType}</span></div>}
                        </div>
                      ))}
                    </div>
                  </>
                )}

                {dayCalls.length>0&&(
                  <>
                    <span style={secLabel}>Calls</span>
                    <div style={{display:"flex",gap:8,flexWrap:"wrap",marginBottom:10}}>
                      <Stat label="Total calls" value={cs.total}/>
                      <Stat label="Connected" value={cs.connected} color="var(--success-text)"/>
                      <Stat label="Not connected" value={cs.notConnected} color="var(--danger-text)"/>
                      <Stat label="Quality calls" value={cs.quality} color="var(--success-text)"/>
                      <Stat label="Mtg booked" value={cs.booked} color="var(--warn-text)"/>
                      <Stat label="Connect %" value={cs.connectRate} color="var(--info-text)"/>
                      <Stat label="Call pts" value={cs.points} color="var(--info-text)"/>
                    </div>
                    <div style={{display:"flex",flexDirection:"column",gap:6,marginBottom:16}}>
                      {dayCalls.map(c=>(
                        <div key={c.id} style={{background:"var(--bg-primary)",border:"0.5px solid var(--border)",borderRadius:10,padding:"9px 12px"}}>
                          <div style={{display:"flex",alignItems:"center",gap:8,flexWrap:"wrap"}}>
                            <span style={{fontWeight:500,fontSize:14,flex:1}}>{c.contact}</span>
                            <Badge label={c.status} colors={CALL_COLORS[c.status]}/>
                            {CALL_POINTS[c.status]&&<span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:"var(--info-bg)",color:"var(--info-text)",fontWeight:500}}>+{CALL_POINTS[c.status]} pt</span>}
                            <button onClick={()=>startEditCall(c)} style={{fontSize:12,padding:"3px 8px",borderRadius:6,border:"0.5px solid var(--border-strong)",background:"transparent",cursor:"pointer",color:"var(--text-secondary)"}}>Edit</button>
                            <button onClick={()=>setCalls(p=>p.filter(x=>x.id!==c.id))} style={{fontSize:12,padding:"3px 8px",borderRadius:6,border:"0.5px solid var(--danger-border)",background:"transparent",cursor:"pointer",color:"var(--danger-text)"}}>✕</button>
                          </div>
                          {c.notes&&<div style={{fontSize:12,color:"var(--text-secondary)",marginTop:4}}>{c.notes}</div>}
                        </div>
                      ))}
                    </div>
                  </>
                )}

                {dayUpsells.length>0&&(
                  <>
                    <span style={secLabel}>Standalone deals</span>
                    <div style={{display:"flex",flexDirection:"column",gap:6}}>
                      {dayUpsells.map(u=>(
                        <div key={u.id} style={{background:"var(--bg-primary)",border:"0.5px solid var(--warn-border)",borderRadius:10,padding:"9px 12px"}}>
                          <div style={{display:"flex",alignItems:"center",gap:8,flexWrap:"wrap"}}>
                            <span style={{fontWeight:500,fontSize:14,flex:1}}>{u.contact}</span>
                            <span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:"var(--warn-bg)",color:"var(--warn-text)",fontWeight:500}}>{u.upsellType}</span>
                            <span style={{fontSize:11,padding:"2px 8px",borderRadius:20,background:"var(--info-bg)",color:"var(--info-text)",fontWeight:500}}>+1 pt</span>
                            <button onClick={()=>startEditUpsell(u)} style={{fontSize:12,padding:"3px 8px",borderRadius:6,border:"0.5px solid var(--border-strong)",background:"transparent",cursor:"pointer",color:"var(--text-secondary)"}}>Edit</button>
                            <button onClick={()=>setUpsells(p=>p.filter(x=>x.id!==u.id))} style={{fontSize:12,padding:"3px 8px",borderRadius:6,border:"0.5px solid var(--danger-border)",background:"transparent",cursor:"pointer",color:"var(--danger-text)"}}>✕</button>
                          </div>
                          {u.notes&&<div style={{fontSize:12,color:"var(--text-secondary)",marginTop:4}}>{u.notes}</div>}
                        </div>
                      ))}
                    </div>
                  </>
                )}
              </>
            ):(
              <div style={{textAlign:"center",padding:"2rem 0",color:"var(--text-tertiary)",fontSize:14}}>Nothing logged for this day yet.</div>
            )}
          </>
        )}

        {tab==="weekly"&&(
          weeklyData().length===0
            ?<div style={{textAlign:"center",padding:"2rem 0",color:"var(--text-tertiary)",fontSize:14}}>No data logged yet.</div>
            :<WeeklyView data={weeklyData()}/>
        )}
      </div>
    </div>
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(<App/>);
</script>
</body>
</html>
