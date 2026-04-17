import { useState, useEffect } from "react";

// ─── AI PROFILE BUILDER ───────────────────────────────────────────────────────
const buildProfile = (name) => `You are a warm, trauma-informed journaling companion for ${name}.
They practice IFS (Internal Family Systems) parts work and somatic therapy.
Core principles: VALIDATE FIRST always. Soften shame gently — never bypass.
Trauma-informed: no urgency, no "should", no pushing. Brief and warm.
When you notice a part present, name it with gentle curiosity. Notice somatic language.
When generating a 'Recap', analyze patterns across history using these buckets:
1. Emotions: The dominant emotional narrative.
2. Threads: Connections to past stories/entries.
3. Body Map: Where tension or sensation has been held.
4. Quiet Murmurs: Naming parts present beneath the surface.`;

// ─── CONSTANTS & THEMES ───────────────────────────────────────────────────────
const PC=['#C4522A','#D4842A','#C8952A','#7A4A2A','#8B6B3A','#4A7A3A','#7A2A2A','#5A3A7A','#2A5A7A','#8B4A5A'];
const PE=['🍂','🍁','💧','🦊','🍄','🌻','🕯️','🐿️','🪵','🌾','🌿','🍃','🦉','🐚','✨','🌙','🪨','🌸','🦋','🧸'];
const MOODS=['😔','😕','😐','🙂','😊'];
const LEVELS=[{min:0,name:'Seedling',icon:'🌱'},{min:50,name:'Sprout',icon:'🌿'},{min:150,name:'Sapling',icon:'🍃'},{min:350,name:'Grove',icon:'🌲'},{min:700,name:'Old Growth',icon:'🌳'}];
const PTS={journal:20,checkin:5,part:15,interact:10,habit:10,idea:5,rel:8};
const HEMOJIS=['🌱','🍂','🌊','🧘','📚','💧','🏃','✍️','🎨','🎵','🌞','🌙','🍵','💪','🧠','🌻','🐾'];
const STARS=[{x:23,y:12,r:.8},{x:67,y:8,r:1.2},{x:112,y:22,r:.6},{x:156,y:5,r:1},{x:198,y:18,r:.8},{x:240,y:11,r:1.4},{x:284,y:24,r:.7},{x:325,y:7,r:1.1},{x:15,y:52,r:1},{x:87,y:38,r:.7},{x:134,y:58,r:1.3},{x:178,y:44,r:.8},{x:267,y:41,r:.6},{x:310,y:62,r:1.2},{x:30,y:82,r:.8},{x:119,y:88,r:.6},{x:207,y:85,r:.7},{x:251,y:78,r:1.3},{x:296,y:92,r:.8},{x:189,y:118,r:.8},{x:278,y:118,r:.6},{x:322,y:108,r:1.2}];

const THEMES = {
  'autumn-light': { name: 'Autumn Light', icon: '🍂', bg:'#F5E8D0', bgD:'#EDD9B8', card:'#FDF4E3', cardD:'#F0E0C0', rust:'#C4522A', amber:'#D4842A', forest:'#4A7A3A', burg:'#7A2A2A', umber:'#3D1C0C', muted:'#A07850', border:'#D8C4A0', textInv: '#FDF4E3' },
  'autumn-dark':  { name: 'Autumn Dark', icon: '🪵', bg:'#1A0F0A', bgD:'#120A07', card:'#2A1810', cardD:'#20120C', rust:'#D4842A', amber:'#C8952A', forest:'#5C8A4A', burg:'#8B3A3A', umber:'#FDF4E3', muted:'#A07850', border:'#4A2A1A', textInv: '#1A0F0A' },
  'constellation-light': { name: 'Constellation Light', icon: '✨', bg:'#E8EDF4', bgD:'#D0D9E8', card:'#F4F7FB', cardD:'#E2E8F0', rust:'#4A6A9A', amber:'#5A7AA8', forest:'#4A8A7A', burg:'#6A4A8A', umber:'#1C243D', muted:'#788AA0', border:'#C0D0E0', textInv: '#F4F7FB' },
  'constellation-dark':  { name: 'Constellation Dark', icon: '🌌', bg:'#0A0E17', bgD:'#05070B', card:'#151A28', cardD:'#0F131D', rust:'#7A9AC8', amber:'#5A7AA8', forest:'#5C9A8A', burg:'#8A6AA8', umber:'#F4F7FB', muted:'#788AA0', border:'#2A344A', textInv: '#0A0E17' }
};

const uid=()=>Date.now().toString(36)+Math.random().toString(36).slice(2,5);
const greet=()=>{const h=new Date().getHours();return h<12?'Good morning':h<17?'Good afternoon':'Good evening';};
const fmtD=iso=>new Date(iso).toLocaleDateString('en-US',{month:'short',day:'numeric'});
const todayK=()=>new Date().toISOString().slice(0,10);
const getLv=p=>[...LEVELS].reverse().find(l=>p>=l.min)||LEVELS[0];
const nextLv=p=>LEVELS.find(l=>l.min>p);

function getThemes(entries) {
  const counts = {};
  entries.forEach(e => {
    if(e.type === 'journal' && e.reflection?.parts_noticed) {
      e.reflection.parts_noticed.forEach(p => { if(p) counts[p] = (counts[p]||0)+1; });
    }
    if(e.type === 'quick') {
      const m = ['Struggling', 'Low', 'Neutral', 'Good', 'Joyful'][e.mood] || 'Check-in';
      counts[m] = (counts[m]||0)+1;
    }
    if(e.type === 'idea' && e.tag) {
      counts[e.tag] = (counts[e.tag]||0)+1;
    }
  });
  return Object.entries(counts).sort((a,b)=>b[1]-a[1]).slice(0,8);
}

async function callAI(sys,prompt,max=800){
  const r=await fetch('https://api.anthropic.com/v1/messages',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({model:'claude-sonnet-4-20250514',max_tokens:max,system:sys,messages:[{role:'user',content:prompt}]})});
  const d=await r.json();
  return d.content?.find(c=>c.type==='text')?.text||'';
}

// ─── CONFETTI COMPONENT ───────────────────────────────────────────────────────
function Confetti({ active, colors }) {
  if (!active) return null;
  return (
    <div style={{ position: 'fixed', inset: 0, pointerEvents: 'none', zIndex: 9999, overflow: 'hidden' }}>
      {[...Array(60)].map((_, i) => {
        const left = Math.random() * 100 + '%';
        const animDuration = Math.random() * 1.5 + 1.5 + 's';
        const animDelay = Math.random() * 0.2 + 's';
        const color = colors[Math.floor(Math.random() * colors.length)];
        const isCircle = Math.random() > 0.5;
        return (
          <div key={i} style={{ position: 'absolute', left, top: '-5%', width: 10, height: 10, backgroundColor: color, borderRadius: isCircle ? '50%' : '2px', animation: `fallConfetti ${animDuration} linear ${animDelay} forwards` }} />
        );
      })}
      <style>{`@keyframes fallConfetti { 0% { transform: translateY(0) rotate(0deg); opacity: 1; } 100% { transform: translateY(110vh) rotate(720deg); opacity: 0; } }`}</style>
    </div>
  );
}

// ─── MAIN APP ─────────────────────────────────────────────────────────────────
export default function InnerKeep() {
// ── Drive & Setup States ──
const [token, setToken] = useState(null);
const [status, setStatus] = useState('Not signed in');
const [statusVisible, setStatusVisible] = useState(false);
const [fileId, setFileId] = useState(null);
const [isLoaded, setIsLoaded] = useState(false);

// ── User & Settings States ──
const[userName,setUserName]=useState('');
const[tempName,setTempName]=useState(''); 
const[readwiseToken,setReadwiseToken]=useState('');
const[rwInput,setRwInput]=useState(null); 
const[dailyQuote,setDailyQuote]=useState(null);
const[theme,setTheme]=useState('autumn-light');
const[usePrompts,setUsePrompts]=useState(true);
  
  // ── A11y & Config ──
  const[hideReadwise,setHideReadwise]=useState(false);
  const[hideSearch,setHideSearch]=useState(false);
  const[hideThemes,setHideThemes]=useState(false);
  const[textScale,setTextScale]=useState(1.0); 
  const[fontStyle,setFontStyle]=useState('default');
  const[isLocked,setIsLocked]=useState(false);

  // ── App Data States ──
  const[view,setView]=useState('home');
  const[entries,setEntries]=useState([]);
  const[parts,setParts]=useState([]);
  const[habits,setHabits]=useState([]);
  const[ideas,setIdeas]=useState([]);
  const[trash,setTrash]=useState([]); 
  const[points,setPoints]=useState(0);
  
  // ── UI States ──
  const[ptFlash,setPtFlash]=useState(null);
  const[toast,setToast]=useState(null);
  const[showMenu,setShowMenu]=useState(false);
  const[searchQuery,setSearchQuery]=useState('');
  const[showHistory,setShowHistory]=useState(false);
  const[showConfetti,setShowConfetti]=useState(false);
  const[confettiColors,setConfettiColors]=useState(PC);
  
  // journal & quick
  const[jText,setJText]=useState('');
  const[activeEntryId,setActiveEntryId]=useState(null);
  const[aiRes,setAiRes]=useState(null);
  const[aiLoad,setAiLoad]=useState(false);
  const[pprompt,setPprompt]=useState('');
  const[ppLoad,setPpLoad]=useState(false);
  const[nudge,setNudge]=useState('');
  const[milestoneReached,setMilestoneReached]=useState(false);
  const[mood,setMood]=useState(2);
  const[energy,setEnergy]=useState(1);
  const[dump,setDump]=useState('');
  const[win,setWin]=useState('');
  const[qSaved,setQSaved]=useState(false);

  // Recap Logic
  const[recapRes,setRecapRes]=useState(null);
  const[recapLoad,setRecapLoad]=useState(false);
  
  // parts & world
  const[selPart,setSelPart]=useState(null);
  const[ep,setEp]=useState(null);
  const[showRoleHelp,setShowRoleHelp]=useState(false); 
  const[addRelMode,setAddRelMode]=useState(false);
  const[newRel,setNewRel]=useState({partId:'',type:'bonded',note:''});
  const[relRef,setRelRef]=useState('');
  const[relLoad,setRelLoad]=useState(false);
  const[iMsg,setIMsg]=useState('');
  const[iLoad,setILoad]=useState(false);
  const[iReply,setIReply]=useState('');
  
  // habits & ideas
  const[nh,setNh]=useState({name:'',emoji:'🌱',frequency:'daily'});
  const[addH,setAddH]=useState(false);
  const[ni,setNi]=useState({title:'',content:'',tag:''});
  const[addI,setAddI]=useState(false);
  const[iFilter,setIFilter]=useState('');
  const[expIdea,setExpIdea]=useState(null);
  const[resIdea,setResIdea]=useState(null);

  // ─── TIMEOUT LOGIC (15 MIN) ───────────────────────────────────────────────
  useEffect(() => {
    let timeoutId;
    const resetTimer = () => {
      clearTimeout(timeoutId);
      if (token && isLoaded && userName) {
        timeoutId = setTimeout(() => setIsLocked(true), 15 * 60 * 1000);
      }
    };
    window.addEventListener('mousemove', resetTimer);
    window.addEventListener('keydown', resetTimer);
    resetTimer();
    return () => {
      window.removeEventListener('mousemove', resetTimer);
      window.removeEventListener('keydown', resetTimer);
      clearTimeout(timeoutId);
    };
  }, [token, isLoaded, userName]);

  // ─── ACCESSIBILITY & STYLING LOGIC ──────────────────────────────────────────
  const C = THEMES[theme] || THEMES['autumn-light'];
  const sz = (baseSize) => Math.round(baseSize * textScale);
  const famBody = fontStyle === 'dyslexic' ? "'Comic Sans MS', 'OpenDyslexic', sans-serif" : "'Nunito', sans-serif";
  const famHead = fontStyle === 'dyslexic' ? famBody : "'Playfair Display', serif";

  const RELS={
    protective:{label:'Protective of',color:C.forest,icon:'🛡️',dash:'none'},
    tension:{label:'In tension with',color:C.burg,icon:'⚡',dash:'6,4'},
    bonded:{label:'Bonded with',color:C.rust,icon:'🤝',dash:'none'},
    triggered:{label:'Triggered by',color:C.amber,icon:'🔔',dash:'3,4'},
  };

  const sc=(x={})=>({background:C.card,borderRadius:18,padding:'17px 15px',marginBottom:12,boxShadow:'0 2px 14px rgba(0,0,0,.08)',border:`1px solid ${C.border}`,width:'100%',boxSizing:'border-box',...x});
  const sb=(x={})=>({background:C.rust,color:C.textInv,border:'none',borderRadius:11,padding:'10px 18px',fontFamily:famBody,fontWeight:800,fontSize:sz(13),cursor:'pointer',boxSizing:'border-box',...x});
  const sbo=(x={})=>({background:'transparent',color:C.rust,border:`1.5px solid ${C.rust}`,borderRadius:11,padding:'8px 14px',fontFamily:famBody,fontWeight:700,fontSize:sz(12),cursor:'pointer',boxSizing:'border-box',...x});
  const si=(x={})=>({width:'100%',padding:'10px 13px',borderRadius:10,border:`1px solid ${C.border}`,fontFamily:famBody,fontSize:16,background:C.bg,color:C.umber,outline:'none',boxSizing:'border-box',...x});
  const sta=(x={})=>({...si(),resize:'none',minHeight:80,lineHeight:1.75,...x});
  const slbl={fontSize:sz(10),fontWeight:800,color:C.muted,textTransform:'uppercase',letterSpacing:'.1em',marginBottom:6,display:'block',textAlign:'left',fontFamily:famBody};
  const H1={fontFamily:famHead,fontSize:sz(23),color:C.umber,fontWeight:700,margin:'0 0 3px',textAlign:'left'};
  const H2={fontFamily:famHead,fontSize:sz(17),color:C.umber,fontWeight:700,margin:'0 0 7px',textAlign:'left'};
  const H3={fontFamily:famHead,fontSize:sz(14),color:C.umber,fontWeight:600,margin:'0 0 5px',textAlign:'left'};
  const sub={fontSize:sz(13),color:C.muted,margin:0,lineHeight:1.5,textAlign:'left',fontFamily:famBody};
  const stag=(col=C.rust,x={})=>({fontSize:sz(10),fontWeight:800,padding:'3px 10px',borderRadius:20,background:`${col}1A`,color:col,border:`1px solid ${col}33`,display:'inline-block',fontFamily:famBody,...x});

  // IFS Part Card Styling
  const getGenderSkin = (gender, color) => {
    if (gender === 'female') return { borderRadius: 30, border: `3px solid transparent`, background: `linear-gradient(${C.card}, ${C.card}) padding-box, linear-gradient(135deg, ${color}, ${color}33) border-box` };
    if (gender === 'male') return { borderRadius: 14, border: `2px solid ${color}88`, boxShadow: `inset 0 4px 12px ${color}15` };
    if (gender === 'neutral') return { borderRadius: 20, border: `2px solid ${color}`, animation: `shimmer 3s infinite` };
    return { borderRadius: 18, border: `2px solid ${color}44` };
  };

  // ─── READWISE ─────────────────────────────────────────────────────────
  async function fetchReadwise(apiToken) {
    if (!apiToken) return;
    try {
      const res = await fetch('https://readwise.io/api/v2/highlights/', { headers: { Authorization: `Token ${apiToken}` } });
      const data = await res.json();
      if (data.results && data.results.length > 0) {
        const random = data.results[Math.floor(Math.random() * data.results.length)];
        setDailyQuote({ text: random.text, author: random.author, title: random.book_title });
      }
    } catch(err) { console.log("Readwise error:", err); }
  }

  // ─── COMPOST BIN LOGIC ───────────────────────────────────────────────────────
  const cleanupTrash = (currentTrash) => {
    const thirtyDaysAgo = Date.now() - (30 * 24 * 60 * 60 * 1000);
    return currentTrash.filter(t => t.deletedAt > thirtyDaysAgo);
  };

  const moveToTrash = (type, item) => {
    setTrash([{ id: uid(), type, data: item, deletedAt: Date.now() }, ...trash]);
  };

  const restoreFromTrash = (trashId) => {
    const item = trash.find(t => t.id === trashId);
    if (!item) return;
    if (item.type === 'entry') setEntries([item.data, ...entries]);
    if (item.type === 'part') setParts([item.data, ...parts]);
    if (item.type === 'habit') setHabits([item.data, ...habits]);
    if (item.type === 'idea') setIdeas([item.data, ...ideas]);
    setTrash(trash.filter(t => t.id !== trashId));
    showToast('🌱 Restored from Compost');
  };

  const emptyTrash = () => { if(window.confirm('Empty the compost bin forever?')) setTrash([]); };

  // ─── GOOGLE DRIVE LOGIC ──────────────────────────────────────────────────────
  const loadFromDrive = async (accessToken) => {
    setStatus('Loading backup...');
    try {
      const searchRes = await fetch("https://www.googleapis.com/drive/v3/files?q=name='inner-keep_Backup.json'", { headers: { Authorization: `Bearer ${accessToken}` } });
      const searchData = await searchRes.json();
      if (searchData.files && searchData.files.length > 0) {
        const id = searchData.files[0].id;
        setFileId(id);
        const fileRes = await fetch(`https://www.googleapis.com/drive/v3/files/${id}?alt=media`, { headers: { Authorization: `Bearer ${accessToken}` } });
        const backup = await fileRes.json();
        if (backup.userName) setUserName(backup.userName);
        if (backup.theme) setTheme(backup.theme);
        if (backup.hideReadwise !== undefined) setHideReadwise(backup.hideReadwise);
        if (backup.hideSearch !== undefined) setHideSearch(backup.hideSearch);
        if (backup.hideThemes !== undefined) setHideThemes(backup.hideThemes);
        if (backup.textScale) setTextScale(backup.textScale);
        if (backup.fontStyle) setFontStyle(backup.fontStyle);
        if (backup.usePrompts !== undefined) setUsePrompts(backup.usePrompts);
        if (backup.readwiseToken) { setReadwiseToken(backup.readwiseToken); fetchReadwise(backup.readwiseToken); }
        if (backup.entries) setEntries(backup.entries);
        if (backup.parts) setParts(backup.parts);
        if (backup.habits) setHabits(backup.habits);
        if (backup.ideas) setIdeas(backup.ideas);
        if (backup.trash) setTrash(cleanupTrash(backup.trash));
        if (backup.points) setPoints(backup.points);
        setStatus('Data loaded');
      } else { setStatus('Ready'); }
    } catch (err) { setStatus('Error loading'); }
    setIsLoaded(true); 
  };

  const saveToDrive = async () => {
    if (!token || !isLoaded || !userName) return;
    setStatus('Saving...');
    const backupData = { userName, theme, hideReadwise, hideSearch, hideThemes, textScale, fontStyle, usePrompts, readwiseToken, entries, parts, habits, ideas, trash, points };
    const fileContent = JSON.stringify(backupData, null, 2);
    const metadata = { name: 'inner-keep_Backup.json', mimeType: 'application/json' };
    const file = new Blob([fileContent], { type: 'application/json' });
    const formData = new FormData();
    formData.append('metadata', new Blob([JSON.stringify(metadata)], { type: 'application/json' }));
    formData.append('file', file);
    const url = fileId ? `https://www.googleapis.com/upload/drive/v3/files/${fileId}?uploadType=multipart` : 'https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart';
  
    try {
      const res = await fetch(url, { method: fileId ? 'PATCH' : 'POST', headers: { Authorization: `Bearer ${token}` }, body: formData });
      const data = await res.json();
      if (!fileId && data.id) setFileId(data.id);
      setStatus('Saved');
    } catch (err) { setStatus('Error saving'); }
  };

  const wipeAllData = async () => {
    const confirmText = window.prompt("🚨 DANGER ZONE 🚨\n\nThis will permanently delete your entire journal, all parts, and history from your Google Drive. This CANNOT be undone.\n\nTo confirm, type the word DELETE below:");
    
    if (confirmText === "DELETE") {
      if (fileId && token) {
        setStatus('Deleting...');
        try {
          await fetch(`https://www.googleapis.com/drive/v3/files/${fileId}`, {
            method: 'DELETE',
            headers: { Authorization: `Bearer ${token}` }
          });
        } catch (err) { 
          setStatus('Error deleting data'); 
          return; 
        }
      }
      handleSignOut();
      alert("Your keep has been successfully and permanently deleted.");
    } else if (confirmText !== null) {
      alert("Deletion cancelled. The text did not exactly match 'DELETE'.");
    }
  };

  useEffect(() => {
    const initTimer = setTimeout(() => {
      if (window.google) {
        window.googleClient = window.google.accounts.oauth2.initTokenClient({
          client_id: "509543907742-q2pf00mpqrlrm0m3rbhjcjr21eopp8v6.apps.googleusercontent.com",
          scope: 'https://www.googleapis.com/auth/drive.file',
          callback: (response) => { setToken(response.access_token); loadFromDrive(response.access_token); },
        });
      }
    }, 500);
    return () => clearTimeout(initTimer);
  }, []);

  useEffect(() => {
    if (!isLoaded || !token || !userName) return;
    const timeout = setTimeout(() => saveToDrive(), 2000);
    return () => clearTimeout(timeout);
  }, [userName, theme, hideReadwise, hideSearch, hideThemes, textScale, fontStyle, usePrompts, readwiseToken, entries, parts, habits, ideas, trash, points, token, isLoaded]); 

  useEffect(() => {
    if (status === 'Not signed in' || status === 'Ready') {
      setStatusVisible(false);
      return;
    }
    setStatusVisible(true);
    if (status === 'Saved' || status === 'Data loaded' || status.includes('Error')) {
      const timer = setTimeout(() => setStatusVisible(false), 5000);
      return () => clearTimeout(timer);
    }
  }, [status]);

  // ─── JOURNAL AUTOSAVE & NUDGES ────────────────────────────────────────────
  useEffect(() => {
    if (view !== 'journal' || !jText.trim()) return;
    const saveTimeout = setTimeout(() => {
      const currentId = activeEntryId || uid();
      if (!activeEntryId) setActiveEntryId(currentId);
      setEntries(prev => {
        const filtered = prev.filter(e => e.id !== currentId);
        return [{ id: currentId, date: new Date().toISOString(), type: 'journal', text: jText, reflection: aiRes }, ...filtered];
      });
    }, 5000);

    let nudgeTimeout;
    if (usePrompts) {
      nudgeTimeout = setTimeout(async () => {
        const prompt = await callAI(buildProfile(userName), `The user is journaling. Last sentence: "${jText.split('.').pop()}". Ask one gentle somatic or Rosebud-style nudge. No preamble.`, 100);
        setNudge(prompt);
      }, 12000);
    }

    const wordCount = jText.trim().split(/\s+/).length;
    if (wordCount >= 250 && !milestoneReached) {
      setMilestoneReached(true);
      earn(20, 'Nourished with 1 Drop');
      fireConfetti();
    }

    return () => { clearTimeout(saveTimeout); clearTimeout(nudgeTimeout); };
  }, [jText]);

  // ─── APP LOGIC ──────────────────────────────────────────────────────────────
  const PROFILE = userName ? buildProfile(userName) : '';

  function fireConfetti(colors = PC){ setConfettiColors(colors); setShowConfetti(true); setTimeout(()=>setShowConfetti(false), 2800); }
  function earn(amt,label){ setPoints(p => p + amt); setPtFlash({amt,label}); setTimeout(()=>setPtFlash(null),2200); }
  function showToast(m){setToast(m);setTimeout(()=>setToast(null),2200);}
  function handleSignOut() { setToken(null); setIsLoaded(false); setFileId(null); setStatus('Not signed in'); setUserName(''); setEntries([]); setParts([]); setHabits([]); setIdeas([]); setTrash([]); setPoints(0); setJText(''); setActiveEntryId(null); setView('home'); }

  async function reflect(){
    if(!jText.trim()||aiLoad)return;
    setAiLoad(true);setAiRes(null);
    try{
      const ctx=parts.length?`\nKnown parts: ${parts.map(p=>`"${p.name}" (${p.role||'part'})`).join(', ')}`:'';
      const raw=await callAI(PROFILE+ctx,`Journal entry:\n\n${jText}\n\nRespond ONLY with valid JSON:\n{"validation":"...","parts_noticed":["..."],"somatic":"...","gentle_question":"..."}`);
      setAiRes(JSON.parse(raw.replace(/```json|```/g,'').trim()));
    }catch{setAiRes({validation:"Something felt off — try again when ready 🍂",parts_noticed:[],somatic:'',gentle_question:'What do you need right now?'});}
    setAiLoad(false);
  }

  function deleteEntry(id){ const e=entries.find(x=>x.id===id); if(e){ moveToTrash('entry', e); setEntries(entries.filter(x=>x.id!==id)); } }

  function saveQuick(){
    setEntries([{id:uid(),date:new Date().toISOString(),type:'quick',mood,energy,dump,win},...entries]);
    earn(PTS.checkin,'Check-in'); fireConfetti(); setDump('');setWin('');setMood(2);setEnergy(1);setQSaved(true);setTimeout(()=>setQSaved(false),2000);
  }

  async function generateRecap() {
    setRecapLoad(true);
    try {
      const history = entries.slice(0, 10).map(e => e.text).join("\n---\n");
      const res = await callAI(buildProfile(userName), `Based on these entries, provide a 'Recap' in JSON format with keys: emotions (string), threads (string), bodyMap (string), quietMurmurs (string). History:\n${history}`);
      setRecapRes(JSON.parse(res.replace(/```json|```/g, '').trim()));
    } catch { showToast("The Keep is quiet right now."); }
    setRecapLoad(false);
  }

  async function getPartPrompt(part){
    setPpLoad(true);setPprompt('');
    try{ 
      let guidance = "What does this part most need you to hear today?";
      if(part.role === 'exile') guidance = "Shadow Work: In what ways have you felt the need to hide or suppress yourself to feel safe?";
      if(part.role === 'firefighter') guidance = "Shadow Work: What triggered this impulse? What pain are you trying to protect us from feeling?";
      if(part.role === 'manager') guidance = "Shadow Work: What is your role, and what are you afraid would happen if you didn't perform it?";
      setPprompt(await callAI(PROFILE,`My inner part "${part.name}" (${part.role||'part'}).\nUsing this guidance: "${guidance}"\nGive me one gentle journal prompt (1-2 sentences, no preamble, no quotes):`,200)); 
    }
    catch{ setPprompt(`What does ${part.name} most need you to hear today?`); }
    setPpLoad(false);
  }

  const generateShadowPrompt = async () => {
    setILoad(true);
    try {
      const promptText = await callAI(
        PROFILE, 
        `The user is interacting with their part named "${selPart.nickname || selPart.name}". Generate one deep-inquiry question based on IFS and Shadow Work principles to help them understand this part's protective role or hidden burden.`, 
        150
      );
      setIMsg(promptText);
    } catch { setIMsg("What does this part need you to know?"); }
    setILoad(false);
  };

  function upsertPart(p){
    setParts(parts.find(x=>x.id===p.id)?parts.map(x=>x.id===p.id?p:x):[p,...parts]);
    if(!parts.find(x=>x.id===p.id))earn(PTS.part,'New part discovered');
    setSelPart(p);setEp(null);setView('partDetail');
  }

  function deletePart(id){ const p=parts.find(x=>x.id===id); if(p){ moveToTrash('part', p); setParts(parts.filter(x=>x.id!==id).map(x=>({...x,relationships:(x.relationships||[]).filter(r=>r.partId!==id)}))); setSelPart(null); setView('parts'); } }

  function addRelationship(){
    if(!newRel.partId||!selPart)return;
    const updated=parts.map(p=>p.id!==selPart.id?p:{...p,relationships:[...(p.relationships||[]).filter(r=>r.partId!==newRel.partId),{partId:newRel.partId,type:newRel.type,note:newRel.note}]});
    setParts(updated); const up=updated.find(p=>p.id===selPart.id); setSelPart(up);
    const other=parts.find(p=>p.id===newRel.partId);
    if(other){
      setRelLoad(true);setRelRef('');
      callAI(PROFILE,`I've noticed my part "${up.name}" ${RELS[newRel.type]?.label} "${other.name}".\nOne reflection about this (1-2 sentences):`,200).then(t=>{setRelRef(t);setRelLoad(false);}).catch(()=>setRelLoad(false));
    }
    earn(PTS.rel,'Relationship mapped'); setAddRelMode(false); setNewRel({partId:'',type:'bonded',note:''});
  }

  function removeRel(toId){
    const updated=parts.map(p=>p.id!==selPart.id?p:{...p,relationships:(p.relationships||[]).filter(r=>r.partId!==toId)});
    setParts(updated); setSelPart(updated.find(p=>p.id===selPart.id));
  }

  function toggleHabit(id) {
    const today = todayK();
    setHabits(habits.map(h => {
      if (h.id !== id) return h;
      const current = h.completions[today];
      let nextState;
      if (!current) nextState = 'done';
      else if (current === 'done') nextState = 'honored';
      else nextState = null;

      const c = { ...h.completions };
      if (nextState) c[today] = nextState; else delete c[today];

      let streak = 0;
      if (!h.frequency || h.frequency === 'daily') {
        let d = new Date();
        while(true) { const k = d.toISOString().slice(0,10); if (c[k]) { streak++; d.setDate(d.getDate()-1); } else break; }
      } else { streak = Object.keys(c).length; }

      return { ...h, completions: c, streak };
    }));
    
    const h = habits.find(x => x.id === id);
    if (h && !h.completions[today]) { earn(PTS.habit, `${h.emoji} Win logged`); fireConfetti(); }
  }

  function deleteHabit(id){ const h=habits.find(x=>x.id===id); if(h){ moveToTrash('habit', h); setHabits(habits.filter(x=>x.id!==id)); } }

  function addIdea(){
    if(!ni.title.trim())return;
    setIdeas([{id:uid(),title:ni.title,content:ni.content,tag:ni.tag,date:new Date().toISOString(),pinned:false},...ideas]); 
    earn(PTS.idea,'Idea captured');setNi({title:'',content:'',tag:''});setAddI(false);showToast('✨ Captured!');
  }
  
  function togglePin(id){setIdeas(ideas.map(i=>i.id===id?{...i,pinned:!i.pinned}:i));}

  function resolveIdea(id, type) {
    const idea = ideas.find(i => i.id === id);
    if(idea) {
      moveToTrash('idea', idea);
      setIdeas(ideas.filter(i => i.id !== id));
      if (expIdea === id) setExpIdea(null);
      setResIdea(null);
      earn(5, 'Space Cleared');
      if (type === 'done') fireConfetti(); 
      else fireConfetti([C.muted, C.border, C.bgD]); 
    }
  }

  // ─── LOGIN, ONBOARDING & TIMEOUT VIEWS ───────────────────────────────────────
  if (isLocked) {
    return (
      <div style={{ background: C.bg, minHeight: '100vh', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', fontFamily: famBody }}>
        <style>{`body { background: #E0E4E8; margin: 0; }`}</style>
        <div style={{ fontSize: 52, marginBottom: 12 }}>🔒</div>
        <h2 style={{ fontFamily: famHead, fontSize: 24, color: C.umber, fontWeight: 700, margin: '0 0 8px' }}>Keep Locked</h2>
        <p style={{ color: C.muted, marginBottom: 24, fontSize: 14 }}>Hidden for your privacy after 15 minutes of inactivity.</p>
        <button onClick={() => setIsLocked(false)} style={sb({ padding: '14px 24px', fontSize: 15 })}>Unlock Keep</button>
      </div>
    );
  }

  if (!token) {
    return (
      <div style={{ background: C.bg, minHeight: '100vh', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', fontFamily: famBody }}>
        <style>{`@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&family=Nunito:wght@400;500;700;800&display=swap'); body { background: #E0E4E8; margin: 0; }`}</style>
        <div style={{ fontSize: 52, marginBottom: 12 }}>{THEMES[theme].icon}</div>
        <h1 style={{ fontFamily: famHead, fontSize: 32, color: C.umber, fontWeight: 700, margin: '0 0 8px' }}>inner-keep</h1>
        <p style={{ fontSize: 14, color: C.muted, margin: '0 0 24px', textAlign: 'center' }}>Securely connect your journal to Google Drive.</p>
        <button onClick={() => window.googleClient?.requestAccessToken()} style={sb({ padding: '14px 24px', fontSize: 15 })}>Sign in to Sync</button>
      </div>
    );
  }

  if (token && !isLoaded) {
    return (
      <div style={{ background: C.bg, minHeight: '100vh', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', fontFamily: famBody }}>
        <style>{`body { background: #E0E4E8; margin: 0; }`}</style>
        <div style={{ fontSize: 40, animation: 'pulse 1.5s infinite' }}>{THEMES[theme].icon}</div>
        <p style={{ marginTop: 16, color: C.umber, fontWeight: 700 }}>Opening your keep...</p>
        <style>{`@keyframes pulse { 0%, 100% { opacity: 1; transform: scale(1); } 50% { opacity: 0.5; transform: scale(1.1); } }`}</style>
      </div>
    );
  }

  if (isLoaded && !userName) {
    return (
      <div style={{ background: C.bg, minHeight: '100vh', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', padding: 20, fontFamily: famBody }}>
        <style>{`body { background: #E0E4E8; margin: 0; }`}</style>
        <div style={sc({ maxWidth: 400, width: '100%', padding: '30px 24px', textAlign: 'center' })}>
          <div style={{ fontSize: 40, marginBottom: 16 }}>🌱</div>
          <h2 style={{ ...H2, fontSize: 20, marginBottom: 8, textAlign: 'center' }}>Welcome to inner-keep</h2>
          <p style={{ ...sub, marginBottom: 24, fontSize: 14, textAlign: 'center' }}>To begin mapping your inner world, what would you like to be called?</p>
          <input value={tempName} onChange={e=>setTempName(e.target.value)} placeholder="Your name..." style={si({ marginBottom: 16, textAlign: 'center', padding: '12px' })} autoFocus />
          <button onClick={() => { if(tempName.trim()) setUserName(tempName.trim()); }} disabled={!tempName.trim()} style={sb({ width: '100%', padding: '12px', opacity: !tempName.trim() ? .5 : 1 })}>Enter your keep 🍂</button>
        </div>
      </div>
    );
  }

  // ─── CORE VIEW COMPONENTS ───────────────────────────────────────────────────
  
  const renderHome = () => {
    const lv=getLv(points),nl=nextLv(points);
    const pct=nl?Math.round(((points-lv.min)/(nl.min-lv.min))*100):100;
    const doneT=habits.filter(h=>h.completions[todayK()]).length;
    const searchResults = searchQuery ? entries.filter(e => (e.text || '').toLowerCase().includes(searchQuery.toLowerCase()) || (e.dump || '').toLowerCase().includes(searchQuery.toLowerCase())) : [];

    // Sacred Space Logic
    const recapProgress = entries.length % 7;
    const entriesNeeded = recapProgress === 0 && entries.length > 0 ? 0 : 7 - recapProgress;
    const isRecapReady = entriesNeeded === 0;

    return(
      <div style={{padding:'26px 15px 0'}}>
        <div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-start',marginBottom:16}}>
          <div style={{flex: 1}}>
            <p style={{...sub,fontSize:sz(11),marginBottom:2}}>{new Date().toLocaleDateString('en-US',{weekday:'long',month:'long',day:'numeric'})}</p>
            <h1 style={H1}>{greet()}, {userName ? userName.split(' ')[0] : ''} {THEMES[theme].icon}</h1>
            <p style={sub}>How are you and your parts today?</p>
          </div>
          <button onClick={()=>setShowMenu(true)} style={{background:C.cardD,border:`1px solid ${C.border}`,color:C.umber,borderRadius:11,width:36,height:36,cursor:'pointer',display:'flex',alignItems:'center',justifyContent:'center',fontSize:18,flexShrink:0, fontWeight: 800}}>
            {userName ? userName[0].toUpperCase() : '🍂'}
          </button>
        </div>

        <div style={{background:C.card, border:`1px solid ${C.border}`, borderRadius:18,padding:'15px 17px',marginBottom:12,boxShadow:'0 4px 20px rgba(0,0,0,.08)', width: '100%', boxSizing: 'border-box'}}>
          <div style={{display:'flex',alignItems:'center',justifyContent:'space-between',marginBottom:8}}>
            <div style={{display:'flex',alignItems:'center',gap:9}}>
              <span style={{fontSize:26}}>{lv.icon}</span>
              <div><p style={{fontFamily:famHead,fontSize:sz(13),color:C.umber,fontWeight:700,margin:'0 0 1px'}}>{lv.name}</p><p style={{fontSize:sz(11),color:C.muted,margin:0,fontWeight:800,fontFamily:famBody}}>{points} drops 💧</p></div>
            </div>
          </div>
          <div style={{background:C.bg,borderRadius:8,height:6,overflow:'hidden'}}><div style={{width:`${pct}%`,height:'100%',background:`linear-gradient(90deg,${C.forest},${C.rust})`,borderRadius:8}}/></div>
          <div style={{display:'flex',gap:12,marginTop:10,flexWrap:'wrap'}}>
            {[['📓',entries.length,'entries'],['🌱',parts.length,'parts'],['✅',doneT,'today'],['💡',ideas.length,'ideas']].map(([icon,n,label])=>(<div key={label} style={{display:'flex',alignItems:'center',gap:3}}><span style={{fontSize:sz(13)}}>{icon}</span><span style={{fontSize:sz(12),color:C.umber,fontWeight:800,fontFamily:famBody}}>{n}</span><span style={{fontSize:sz(10),color:C.muted,fontFamily:famBody}}>{label}</span></div>))}
          </div>
        </div>

        <div style={sc({ borderLeft: `4px solid ${isRecapReady ? C.rust : C.border}`, background: isRecapReady ? `linear-gradient(135deg, ${C.card}, ${C.bg})` : C.card })}>
          <h3 style={H3}>Sacred Space ✨</h3>
          <p style={{ ...sub, marginBottom: 10 }}>{isRecapReady ? "The Keep is ready to speak." : `${entriesNeeded} more entries until your keep unlocks.`}</p>
          <button onClick={() => { if(isRecapReady) generateRecap(); }} disabled={!isRecapReady || recapLoad} style={sb({ width: '100%', opacity: isRecapReady ? 1 : 0.4 })}>
            {recapLoad ? "Listening..." : "Unlock Recap"}
          </button>
          {recapRes && (
            <div style={{ marginTop: 15, padding: 12, background: C.bg, borderRadius: 12 }}>
              {[['Emotions', recapRes.emotions], ['Threads', recapRes.threads], ['Body Map', recapRes.bodyMap], ['Quiet Murmurs', recapRes.quietMurmurs]].map(([label, text]) => (
                <div key={label} style={{ marginBottom: 10 }}>
                  <span style={slbl}>{label}</span>
                  <p style={{ ...sub, color: C.umber }}>{text}</p>
                </div>
              ))}
            </div>
          )}
        </div>

        {!hideSearch && (
          <div style={sc()}>
            <h3 style={H3}>Search Memories 🔍</h3>
            <input value={searchQuery} onChange={e=>setSearchQuery(e.target.value)} placeholder="Search journals & check-ins..." style={si({marginBottom: searchQuery ? 12 : 0})} />
            {searchQuery && (
              <div style={{ display: 'flex', flexDirection: 'column', gap: 8 }}>
                {searchResults.length === 0 && <p style={{...sub, fontSize: sz(12)}}>No matches found.</p>}
                {searchResults.map(e => (
                  <div key={e.id} style={{ padding: '10px', background: C.bg, borderRadius: 10, border: `1px solid ${C.border}`, boxSizing: 'border-box' }}>
                    <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: 4 }}>
                      <span style={{ fontSize: sz(10), color: C.muted, fontWeight: 800, fontFamily: famBody }}>{fmtD(e.date)}</span>
                      <span style={stag(e.type==='quick'?C.forest:C.rust, {fontSize: sz(8), padding: '2px 6px'})}>{e.type.toUpperCase()}</span>
                    </div>
                    <p style={{ fontSize: sz(13), color: C.umber, margin: 0, lineHeight: 1.5, overflow: 'hidden', display: '-webkit-box', WebkitLineClamp: 3, WebkitBoxOrient: 'vertical', fontFamily: famBody }}>{e.text || e.dump}</p>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {!hideReadwise && (
          readwiseToken ? (
            dailyQuote && (
              <div style={sc({ borderLeft: `3px solid ${C.amber}`, background: `linear-gradient(135deg, ${C.card}, ${C.bg})` })}>
                <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: 6 }}><span style={slbl}>Daily Insight</span><span style={{ fontSize: 14 }}>📚</span></div>
                <p style={{ fontSize: sz(13), color: C.umber, fontStyle: 'italic', lineHeight: 1.6, margin: '0 0 6px', fontFamily: famBody }}>"{dailyQuote.text}"</p>
                <p style={{ fontSize: sz(10), color: C.muted, margin: 0, fontWeight: 800, fontFamily: famBody }}>— {dailyQuote.author} {dailyQuote.title && `(${dailyQuote.title})`}</p>
              </div>
            )
          ) : (
            <div style={sc({ padding: '12px 14px' })}>
              <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                <div><h3 style={{...H3, margin: 0, fontSize: sz(13)}}>Connect Readwise 📚</h3><p style={{fontSize: sz(11), color: C.muted, margin: '2px 0 0', fontFamily: famBody}}>Sync highlights to inspire you.</p></div>
                <button onClick={() => setRwInput(rwInput === null ? '' : null)} style={sbo({ padding: '4px 10px', fontSize: sz(11) })}>{rwInput === null ? 'Add Token' : 'Cancel'}</button>
              </div>
              {rwInput !== null && (
                <div style={{ marginTop: 10, display: 'flex', gap: 6 }}>
                  <input value={rwInput} onChange={e=>setRwInput(e.target.value)} placeholder="Access Token..." style={si({ padding: '6px 10px', minHeight: 'auto' })}/>
                  <button onClick={() => { setReadwiseToken(rwInput); fetchReadwise(rwInput); setRwInput(null); }} style={sb({ padding: '6px 12px', fontSize: sz(11) })}>Save</button>
                </div>
              )}
            </div>
          )
        )}

        {!hideThemes && (() => {
          const themes = getThemes(entries);
          if (themes.length === 0) return null;
          return (
            <div style={sc({ padding: '13px 15px' })}>
              <span style={slbl}>Recurring Themes</span>
              <div style={{ display: 'flex', gap: 6, flexWrap: 'wrap', marginTop: 8 }}>
                {themes.map(([w, count]) => (<span key={w} style={stag(C.forest, { fontSize: sz(11) })}>{w} <span style={{ opacity: 0.6, fontSize: sz(9), marginLeft: 2 }}>{count}</span></span>))}
              </div>
            </div>
          );
        })()}

        <div style={sc()}>
          <h3 style={H3}>Quick Check-In <span style={stag(C.forest,{marginLeft:5,fontSize:sz(9)})}>+{PTS.checkin} drops 💧</span></h3>
          <div style={{marginBottom:12,marginTop:9}}>
            <span style={slbl}>Mood</span>
            <div style={{display:'flex',gap:8,justifyContent:'space-between'}}>{MOODS.map((m,i)=>(<button key={i} onClick={()=>setMood(i)} style={{fontSize:24,cursor:'pointer',background:mood===i?`${C.rust}22`:'transparent',border:`2px solid ${mood===i?C.rust:'transparent'}`,borderRadius:10,padding:'3px 7px',transform:mood===i?'scale(1.18)':'scale(1)',transition:'all .15s',boxSizing:'border-box'}}>{m}</button>))}</div>
          </div>
          <div style={{marginBottom:11}}>
            <span style={slbl}>Energy</span>
            <div style={{display:'flex',gap:6}}>{[['🪫','Low'],['⚡','Some'],['🔥','High']].map(([icon,label],i)=>(<button key={i} onClick={()=>setEnergy(i)} style={{flex:1,padding:'6px 4px',borderRadius:9,border:`1px solid ${energy===i?C.rust:C.border}`,background:energy===i?`${C.rust}1A`:'transparent',cursor:'pointer',fontFamily:famBody,fontSize:sz(11),fontWeight:800,color:energy===i?C.rust:C.muted,boxSizing:'border-box'}}>{icon} {label}</button>))}</div>
          </div>
          <div style={{marginBottom:10}}><span style={slbl}>What's on your mind?</span><textarea value={dump} onChange={e=>setDump(e.target.value)} placeholder="Brain dump, no filter…" style={sta({height:80})}/></div>
          <div style={{marginBottom:12}}><span style={slbl}>One tiny win 💧</span><input value={win} onChange={e=>setWin(e.target.value)} placeholder="Even breathing counts" style={si({minHeight:'auto'})}/></div>
          <button onClick={saveQuick} style={sb({width:'100%'})}>{qSaved?'✓ Saved! 🍂':'Save Check-In'}</button>
        </div>

        {habits.length>0&&(
          <div style={sc({padding:'13px 14px'})}>
            <div style={{display:'flex',justifyContent:'space-between',alignItems:'center',marginBottom:8}}><h3 style={{...H3,margin:0}}>Today's Habits</h3><button onClick={()=>setView('habits')} style={{fontSize:sz(11),color:C.rust,fontWeight:800,background:'none',border:'none',cursor:'pointer',fontFamily:famBody}}>See all →</button></div>
            {habits.slice(0,4).map(h=>{
              const d=h.completions[todayK()];
              const isHonored = d === 'honored';
              const isDone = d === 'done';
              const bgCol = isHonored ? C.amber : C.forest;
              return(<div key={h.id} style={{display:'flex',alignItems:'center',gap:8,padding:'7px 9px',borderRadius:10,background:d?`${bgCol}15`:C.bg,marginBottom:5,border:`1px solid ${d?bgCol:C.border}`,boxSizing:'border-box'}}><button onClick={()=>toggleHabit(h.id)} style={{width:30,height:30,borderRadius:9,border:`2px solid ${d?bgCol:C.border}`,background:d?bgCol:'transparent',cursor:'pointer',display:'flex',alignItems:'center',justifyContent:'center',padding:0,boxSizing:'border-box',flexShrink:0}}>{isDone&&<span style={{color:C.bg,fontSize:15,fontWeight:900,fontFamily:famBody}}>✓</span>}{isHonored&&<span style={{color:C.bg,fontSize:14,fontWeight:900,fontFamily:famBody}}>🤍</span>}</button><span style={{fontSize:18}}>{h.emoji}</span><span style={{flex:1,fontSize:sz(13),fontWeight:700,color:d?bgCol:C.umber,textDecoration:isDone?'line-through':'none',textAlign:'left',fontFamily:famBody}}>{h.name}</span>{h.streak>0&&<span style={{fontSize:sz(10),color:C.amber,fontWeight:800,fontFamily:famBody}}>{(!h.frequency||h.frequency==='daily')?'🔥':'⭐'}{h.streak}</span>}</div>);
            })}
          </div>
        )}
      </div>
    );
  };

  const renderJournal = () => {
    const journalEntries = entries.filter(e => e.type === 'journal');
    const words = jText.trim() ? jText.trim().split(/\s+/).length : 0;
    const pct = Math.min(100, (words / 250) * 100);

    return(
      <div style={{padding:'26px 15px 0'}}>
        <div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-start',marginBottom:14}}>
          <div style={{flex: 1}}>
            <h1 style={H1}>Journal ✍️</h1>
            <p style={sub}>Your words are safe here. <span style={stag(C.rust,{marginLeft:3})}>+{PTS.journal} drops 💧</span></p>
          </div>
          <button onClick={() => { setJText(''); setActiveEntryId(null); setMilestoneReached(false); setNudge(''); setShowHistory(!showHistory); }} style={sbo({ padding: '6px 12px', fontSize: sz(11), background: showHistory ? `${C.rust}1A` : 'transparent' })}>{showHistory ? 'Write New ✍️' : 'Review Past 🕰️'}</button>
        </div>

        {!showHistory ? (
          <>
            {parts.length>0&&(
              <div style={sc({background:`linear-gradient(135deg,${C.bg},${C.bgD})`})}>
                <h3 style={H3}>Write with a part</h3><p style={{...sub,fontSize:sz(11),marginBottom:8}}>Tap a part to receive a prompt.</p>
                <div style={{display:'flex',gap:6,flexWrap:'wrap',marginBottom:pprompt||ppLoad?10:0}}>{parts.map(p=>(<button key={p.id} onClick={()=>getPartPrompt(p)} style={{display:'flex',alignItems:'center',gap:5,padding:'5px 10px',borderRadius:20,background:`${p.color}1A`,border:`1px solid ${p.color}55`,cursor:'pointer',fontFamily:famBody,fontSize:sz(12),fontWeight:800,color:p.color,boxSizing:'border-box'}}>{p.emoji} {p.name}</button>))}</div>
                {ppLoad&&<p style={{...sub,fontStyle:'italic',fontSize:sz(13)}}>Listening…</p>}
                {pprompt&&!ppLoad&&(<div style={{padding:'11px 13px',background:C.card,borderRadius:10,border:`1px solid ${C.border}`,boxSizing:'border-box'}}><p style={{fontSize:sz(14),color:C.umber,fontStyle:'italic',lineHeight:1.75,margin:'0 0 9px',textAlign:'left',fontFamily:famBody}}>"{pprompt}"</p><button onClick={()=>{setJText(jText?jText+'\n\n'+pprompt:pprompt);setPprompt('');}} style={sb({fontSize:sz(12),padding:'7px 12px'})}>Use prompt</button></div>)}
              </div>
            )}
            
            <div style={sc()}>
              <div style={{ position: 'relative' }}>
                <textarea value={jText} onChange={e=>setJText(e.target.value)} placeholder="What's alive in you right now? Begin anywhere…" style={sta({minHeight:320, paddingBottom:40})}/>
                <div style={{ position: 'absolute', bottom: 12, left: 12, right: 12 }}>
                  <div style={{ height: 4, background: `${C.border}44`, borderRadius: 2, overflow: 'hidden' }}><div style={{ width: `${pct}%`, height: '100%', background: C.rust }} /></div>
                  <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: 4 }}>
                    <span style={{ fontSize: 10, color: C.muted, fontWeight: 800 }}>{words}/250 words</span>
                    <span style={{ fontSize: 10, color: C.rust, fontWeight: 800 }}>Autosaving to Drive...</span>
                  </div>
                </div>
              </div>
              {nudge && <div style={{ marginTop: 12, padding: 12, background: `${C.rust}11`, borderRadius: 12, borderLeft: `3px solid ${C.rust}` }}><p style={{ ...sub, fontStyle: 'italic', color: C.umber }}>{nudge}</p></div>}
              
              <div style={{display:'flex',gap:8,marginTop:10}}>
                <button onClick={reflect} disabled={!jText.trim()||aiLoad} style={sb({flex:1,opacity:(!jText.trim()||aiLoad)?.5:1})}>{aiLoad?'Reflecting…':'Reflect with AI'}</button>
              </div>
            </div>
            {aiRes&&(<div style={sc({background:`linear-gradient(135deg,${C.bgD},${C.bg})`})}><div style={{display:'flex',alignItems:'center',gap:8,marginBottom:12}}><span style={{fontSize:22}}>🍂</span><h2 style={{...H2, margin: 0}}>Reflection</h2></div>{aiRes.validation&&<p style={{fontSize:sz(14),color:C.umber,lineHeight:1.8,marginBottom:10,textAlign:'left',fontFamily:famBody}}>{aiRes.validation}</p>}{aiRes.somatic&&(<div style={{padding:'9px 12px',background:`${C.forest}1A`,borderRadius:10,marginBottom:10,borderLeft:`3px solid ${C.forest}`,boxSizing:'border-box'}}><span style={{fontSize:sz(9),fontWeight:800,color:C.forest,display:'block',marginBottom:2,letterSpacing:'.08em',textAlign:'left',fontFamily:famBody}}>BODY MAP</span><p style={{fontSize:sz(13),color:C.umber,margin:0,lineHeight:1.65,textAlign:'left',fontFamily:famBody}}>{aiRes.somatic}</p></div>)}{aiRes.parts_noticed?.filter(Boolean).length>0&&(<div style={{marginBottom:10}}><span style={{fontSize:sz(9),fontWeight:800,color:C.rust,display:'block',marginBottom:5,letterSpacing:'.08em',textAlign:'left',fontFamily:famBody}}>QUIET MURMURS</span><div style={{display:'flex',flexWrap:'wrap',gap:5}}>{aiRes.parts_noticed.filter(Boolean).map((p,i)=><span key={i} style={stag(C.rust,{padding:'3px 10px',fontSize:sz(11)})}>{p}</span>)}</div></div>)}{aiRes.gentle_question&&(<div style={{padding:'10px 13px',background:C.card,borderRadius:10,borderLeft:`3px solid ${C.rust}`,boxSizing:'border-box'}}><p style={{fontSize:sz(14),color:C.umber,margin:0,fontStyle:'italic',lineHeight:1.8,textAlign:'left',fontFamily:famBody}}>{aiRes.gentle_question}</p></div>)}</div>)}
          </>
        ) : (
          <div style={{ marginTop: 16 }}>
            {journalEntries.length === 0 ? ( <div style={{ textAlign: 'center', padding: '40px 20px' }}><span style={{ fontSize: 40 }}>📓</span><p style={{ ...sub, marginTop: 10, textAlign: 'center' }}>No past entries yet. Your story begins today.</p></div> ) : (
              journalEntries.map(e => (
                <div key={e.id} style={sc({ marginBottom: 16, position: 'relative' })}>
                  <button onClick={()=>deleteEntry(e.id)} style={{position:'absolute',top:12,right:12,background:'none',border:'none',color:C.muted,cursor:'pointer',fontSize:18}}>×</button>
                  <p style={{ fontSize: sz(11), color: C.muted, fontWeight: 800, marginBottom: 8, letterSpacing: '.05em', textAlign:'left', fontFamily:famBody }}>{new Date(e.date).toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: 'numeric', year: 'numeric' })}</p>
                  <p style={{ fontSize: sz(14), color: C.umber, lineHeight: 1.7, whiteSpace: 'pre-wrap', margin: '0 0 12px', textAlign:'left', fontFamily:famBody }}>{e.text}</p>
                  {e.reflection && (
                    <div style={{ padding: '10px 12px', background: `${C.rust}1A`, borderRadius: 10, borderLeft: `3px solid ${C.rust}`, boxSizing: 'border-box' }}>
                      <span style={{ fontSize: sz(9), fontWeight: 800, color: C.rust, display: 'block', marginBottom: 4, letterSpacing: '.08em', textAlign:'left', fontFamily:famBody }}>AI REFLECTION</span>
                      {e.reflection.validation && <p style={{ fontSize: sz(13), color: C.umber, fontStyle: 'italic', margin: '0 0 6px', lineHeight: 1.6, textAlign:'left', fontFamily:famBody }}>{e.reflection.validation}</p>}
                      {e.reflection.parts_noticed?.length > 0 && (<div style={{ display: 'flex', gap: 4, flexWrap: 'wrap', marginTop: 4 }}>{e.reflection.parts_noticed.filter(Boolean).map((p,i) => <span key={i} style={stag(C.rust, { fontSize: sz(9), padding: '2px 8px' })}>{p}</span>)}</div>)}
                      {e.reflection.gentle_question && <p style={{ fontSize: sz(13), color: C.umber, fontStyle: 'italic', margin: '6px 0 0', lineHeight: 1.6, textAlign:'left', fontFamily:famBody, fontWeight: 700 }}>{e.reflection.gentle_question}</p>}
                    </div>
                  )}
                </div>
              ))
            )}
          </div>
        )}
      </div>
    );
  };

  const renderPartsList = () => {
    // Constellation setup
    const n=parts.length,cx=180,cy=142,r=n<=1?0:n<=3?86:n<=6?104:118;
    const pos=parts.map((_,i)=>({x:n===1?cx:cx+r*Math.cos((2*Math.PI*i/n)-Math.PI/2),y:n===1?cy:cy+r*Math.sin((2*Math.PI*i/n)-Math.PI/2)}));
    const lines=[];parts.forEach((part,i)=>(part.relationships||[]).forEach(rel=>{const j=parts.findIndex(p=>p.id===rel.partId);if(j>-1&&j!==i)lines.push({x1:pos[i].x,y1:pos[i].y,x2:pos[j].x,y2:pos[j].y,type:rel.type});}));

    return(
      <div style={{padding:'26px 15px 0'}}>
        <div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-start',marginBottom:15}}>
          <div style={{flex: 1}}><h1 style={H1}>My World 🌱</h1><p style={sub}>{parts.length} part{parts.length!==1?'s':''} discovered</p></div>
          <button onClick={()=>{setEp({id:uid(),name:'',nickname:'',gender:'neutral',emoji:PE[0],image:null,color:PC[0],role:'',age:'',coreFear:'',positiveIntent:'',vibe:'',description:'',origin:'',bodyLocation:'',trustLevel:3,relationships:[]});setView('partEditor');}} style={sb({padding:'8px 12px',fontSize:sz(12)})}>+ New</button>
        </div>

        {parts.length > 0 && (
          <div style={sc({padding:0,overflow:'hidden',background:C.bgD, marginBottom:16})}>
             <svg width="100%" viewBox="0 0 360 288" style={{display:'block'}}>
               <defs><radialGradient id="nb" cx="50%" cy="50%" r="60%"><stop offset="0%" stopColor={C.rust} stopOpacity=".15"/><stop offset="100%" stopColor={C.bgD} stopOpacity="1"/></radialGradient></defs>
               <rect width="360" height="288" fill="url(#nb)" rx="16"/>
               {STARS.map((s,i)=><circle key={i} cx={s.x} cy={s.y} r={s.r} fill={C.umber} opacity={.1+i%4*.1}/>)}
               {lines.map((l,i)=>{const rt=RELS[l.type]||RELS.bonded;return <line key={i} x1={l.x1} y1={l.y1} x2={l.x2} y2={l.y2} stroke={rt.color} strokeWidth={1.5} strokeOpacity={.55} strokeDasharray={rt.dash==='none'?undefined:rt.dash}/>;})}
               {parts.map((part,i)=>(
                 <g key={part.id} style={{cursor:'pointer'}} onClick={()=>{setSelPart(part);setPprompt('');setIReply('');setIMsg('');setView('partDetail');}}>
                   <circle cx={pos[i].x} cy={pos[i].y} r={21} fill={part.color} opacity={.9}/>
                   <circle cx={pos[i].x} cy={pos[i].y} r={25} fill="none" stroke={part.color} strokeWidth={1} opacity={.35}/>
                   <circle cx={pos[i].x} cy={pos[i].y} r={30} fill="none" stroke={part.color} strokeWidth={.5} opacity={.15}/>
                   {part.image ? (
                     <g clipPath={`url(#clip-${i})`}>
                       <clipPath id={`clip-${i}`}><circle cx={pos[i].x} cy={pos[i].y} r={21} /></clipPath>
                       <image href={part.image} x={pos[i].x-21} y={pos[i].y-21} height={42} width={42} preserveAspectRatio="xMidYMid slice" />
                     </g>
                   ) : (
                     <text x={pos[i].x} y={pos[i].y+1} textAnchor="middle" dominantBaseline="middle" fontSize={14}>{part.emoji}</text>
                   )}
                   <text x={pos[i].x} y={pos[i].y+36} textAnchor="middle" fontSize={10} fill={C.umber} fontFamily={famBody} fontWeight="800">{(part.nickname || part.name).length>13?(part.nickname || part.name).slice(0,11)+'…':(part.nickname || part.name)}</text>
                 </g>
               ))}
             </svg>
          </div>
        )}

        {parts.length===0?(<div style={sc({textAlign:'center',padding:'42px 20px'})}><div style={{fontSize:50,marginBottom:10}}>🌿</div><p style={{...sub,marginBottom:18,textAlign:'center'}}>Name and get to know the different parts of you here.</p><button onClick={()=>{setEp({id:uid(),name:'',nickname:'',gender:'neutral',emoji:PE[0],image:null,color:PC[0],role:'',age:'',coreFear:'',positiveIntent:'',vibe:'',description:'',origin:'',bodyLocation:'',trustLevel:3,relationships:[]});setView('partEditor');}} style={sb()}>Meet a part</button></div>)
        :(<div style={{display:'grid',gridTemplateColumns:'1fr 1fr',gap:10,boxSizing:'border-box'}}>{parts.map(p=>{
          const skin = getGenderSkin(p.gender, p.color);
          return (<button key={p.id} onClick={()=>{setSelPart(p);setPprompt('');setIReply('');setIMsg('');setView('partDetail');}} style={{background:C.card, ...skin, padding:14, cursor:'pointer', textAlign:'left', boxSizing:'border-box', position:'relative'}}>
            <div style={{width:50,height:50,borderRadius:'50%',background:`${p.color}25`,display:'flex',alignItems:'center',justifyContent:'center',fontSize:27,marginBottom:8,border:`1px solid ${p.color}44`,overflow:'hidden'}}>
              {p.image ? <img src={p.image} style={{width:'100%',height:'100%',objectFit:'cover'}} alt="symbol" /> : p.emoji}
            </div>
            <p style={{...H3,fontSize:sz(13),marginBottom:2}}>{p.name}</p>
            {p.nickname&&<p style={{...sub,fontSize:sz(10),color:p.color,fontWeight:800}}>{p.nickname}</p>}
            {p.role&&<span style={stag(p.color, {fontSize:sz(8), marginTop:4})}>{p.role.toUpperCase()}</span>}
          </button>);
        })}</div>)}
      </div>
    );
  };

  const renderPartDetail = () => {
    const cp=parts.find(p=>p.id===selPart?.id)||selPart; 
    if(!cp) return <div style={{padding:20, textAlign:'center'}}><p style={sub}>Part moved to compost.</p><button onClick={()=>setView('parts')} style={sbo({marginTop:10})}>Go Back</button></div>;
    const others=parts.filter(p=>p.id!==cp.id);
    const skin = getGenderSkin(cp.gender, cp.color);
    return(
      <div>
        <div style={{...skin, borderTop: 'none', borderLeft: 'none', borderRight: 'none', borderBottom: `2px solid ${cp.color}66`, borderRadius:'0 0 26px 26px', padding:'28px 15px 20px', marginBottom:12, width:'100%', boxSizing:'border-box'}}>
          <button onClick={()=>setView('parts')} style={sbo({fontSize:sz(11),padding:'4px 10px',marginBottom:12, borderColor: cp.color, color: cp.color})}>← Parts</button>
          <div style={{display:'flex',alignItems:'center',gap:12}}>
            <div style={{width:66,height:66,borderRadius:'50%',background:`${cp.color}35`,display:'flex',alignItems:'center',justifyContent:'center',fontSize:35,border:`2px solid ${cp.color}66`, flexShrink:0, overflow:'hidden'}}>
              {cp.image ? <img src={cp.image} style={{width:'100%',height:'100%',objectFit:'cover'}} alt="symbol" /> : cp.emoji}
            </div>
            <div style={{flex: 1}}>
              <h1 style={{...H1, margin:0}}>{cp.name}</h1>
              {cp.nickname && <p style={{...sub, fontSize:sz(13), color: cp.color, fontWeight:800, margin:'2px 0 0'}}>"{cp.nickname}"</p>}
              <div style={{display:'flex', gap:6, flexWrap:'wrap', marginTop:6}}>
                {cp.role && <span style={stag(cp.color)}>{cp.role.toUpperCase()}</span>}
              </div>
            </div>
          </div>
        </div>

        <div style={{padding:'0 15px',boxSizing:'border-box'}}>
          <div style={{display:'grid', gridTemplateColumns:'1fr 1fr', gap:10, marginBottom:12}}>
            {cp.age && <div style={sc({marginBottom:0, padding:'12px 14px'})}><span style={slbl}>Age / Maturity</span><p style={{fontSize:sz(13), color:C.umber, margin:0, fontFamily:famBody, fontWeight:700}}>{cp.age}</p></div>}
            {cp.bodyLocation && <div style={sc({marginBottom:0, padding:'12px 14px'})}><span style={slbl}>Body Location</span><p style={{fontSize:sz(13), color:C.umber, margin:0, fontFamily:famBody, fontWeight:700}}>{cp.bodyLocation}</p></div>}
          </div>

          {(cp.coreFear || cp.positiveIntent || cp.description || cp.origin || cp.vibe) && (
            <div style={sc()}>
              {cp.description && <div style={{marginBottom:12}}><span style={slbl}>About</span><p style={{fontSize:sz(14),color:C.umber,lineHeight:1.8,margin:0,textAlign:'left',fontFamily:famBody}}>{cp.description}</p></div>}
              {cp.positiveIntent && <div style={{marginBottom:12}}><span style={slbl}>Positive Intent / Needs</span><p style={{fontSize:sz(14),color:C.umber,lineHeight:1.8,margin:0,textAlign:'left',fontFamily:famBody}}>{cp.positiveIntent}</p></div>}
              {cp.coreFear && <div style={{marginBottom:12}}><span style={slbl}>Core Fear</span><p style={{fontSize:sz(14),color:C.umber,lineHeight:1.8,margin:0,textAlign:'left',fontFamily:famBody}}>{cp.coreFear}</p></div>}
              {cp.origin && <div style={{marginBottom:12}}><span style={slbl}>Origin Story</span><p style={{fontSize:sz(14),color:C.umber,lineHeight:1.8,margin:0,textAlign:'left',fontFamily:famBody}}>{cp.origin}</p></div>}
              {cp.vibe && <div><span style={slbl}>The Vibe</span><p style={{fontSize:sz(14),color:C.umber,lineHeight:1.8,margin:0,textAlign:'left',fontFamily:famBody}}>{cp.vibe}</p></div>}
            </div>
          )}
          
          <div style={sc({background:`${cp.color}0d`,border:`1px solid ${cp.color}33`})}>
            <span style={slbl}>Shadow Work Inquiry <span style={stag(cp.color,{marginLeft:4,fontSize:sz(9)})}>+{PTS.interact} drops 💧</span></span>
            <p style={{ ...sub, marginBottom: 12 }}>Sit with this part. What does it need you to know?</p>

            {iMsg ? (
              <div style={{ background: `${C.rust}11`, padding: 12, borderRadius: 12, marginBottom: 12, borderLeft: `3px solid ${C.rust}` }}>
                <p style={{ ...sub, color: C.umber, fontStyle: 'italic' }}>{iMsg}</p>
              </div>
            ) : (
              <button onClick={generateShadowPrompt} disabled={iLoad} style={sb({ width: '100%', marginBottom: 12, background: cp.color })}>
                {iLoad ? 'Listening...' : 'Ask a Question'}
              </button>
            )}

            <textarea 
              value={iReply} 
              onChange={e => setIReply(e.target.value)} 
              placeholder="Journal your response to this part..." 
              style={sta({ minHeight: 100, marginBottom: 10, background:`${cp.color}0a`, border:`1px solid ${cp.color}33` })} 
            />
            <button onClick={() => { 
              if (!iReply.trim()) return;
              setEntries([{ id: uid(), date: new Date().toISOString(), type: 'journal', text: `Interaction with ${cp.name}:\nPrompt: ${iMsg || 'Direct interaction'}\nResponse: ${iReply}` }, ...entries]);
              earn(PTS.interact, 'Part Interaction');
              setIReply(''); setIMsg('');
              showToast('Insight saved to journal.');
            }} style={sb({ width: '100%', opacity: !iReply.trim() ? .5 : 1 })}>Save Insight</button>
          </div>

          <div style={sc()}>
            <div style={{display:'flex',justifyContent:'space-between',alignItems:'center',marginBottom:10}}><span style={slbl}>Relationships</span>{others.length>0&&<button onClick={()=>{setAddRelMode(!addRelMode);setRelRef('');}} style={sbo({fontSize:sz(11),padding:'4px 10px'})}>{addRelMode?'Cancel':'+ Add'}</button>}</div>
            {addRelMode&&(
              <div style={{padding:'12px',background:C.bg,borderRadius:12,marginBottom:11,border:`1px solid ${C.border}`,boxSizing:'border-box'}}>
                <select value={newRel.partId} onChange={e=>setNewRel({...newRel,partId:e.target.value})} style={si({marginBottom:8, minHeight:'auto'})}><option value="">Choose a part…</option>{others.map(p=><option key={p.id} value={p.id}>{p.emoji} {p.name}</option>)}</select>
                <div style={{display:'flex',flexWrap:'wrap',gap:5,marginBottom:8}}>{Object.entries(RELS).map(([k,v])=>(<button key={k} onClick={()=>setNewRel({...newRel,type:k})} style={{fontSize:sz(11),padding:'4px 9px',borderRadius:20,cursor:'pointer',fontFamily:famBody,fontWeight:800,border:`1px solid ${v.color}`,background:newRel.type===k?v.color:'transparent',color:newRel.type===k?C.textInv:v.color,boxSizing:'border-box'}}>{v.icon} {v.label}</button>))}</div>
                <input value={newRel.note} onChange={e=>setNewRel({...newRel,note:e.target.value})} placeholder="Optional note…" style={si({marginBottom:8, minHeight:'auto'})}/>
                <button onClick={addRelationship} disabled={!newRel.partId} style={sb({fontSize:sz(12),opacity:!newRel.partId?.5:1})}>Add relationship</button>
              </div>
            )}
            {relLoad&&<p style={{...sub,fontStyle:'italic',fontSize:sz(13),marginBottom:7}}>Reflecting…</p>}
            {relRef&&!relLoad&&(<div style={{padding:'9px 12px',background:`${C.amber}1A`,borderRadius:10,marginBottom:10,border:`1px solid ${C.amber}44`,boxSizing:'border-box'}}><p style={{fontSize:sz(13),color:C.umber,fontStyle:'italic',lineHeight:1.7,margin:0,textAlign:'left',fontFamily:famBody}}>{relRef}</p></div>)}
            {(cp.relationships||[]).length===0&&!addRelMode&&<p style={{...sub,fontSize:sz(13)}}>No relationships mapped yet.</p>}
            {(cp.relationships||[]).map(rel=>{const other=parts.find(p=>p.id===rel.partId);if(!other)return null;const rt=RELS[rel.type]||RELS.bonded;return(<div key={rel.partId} style={{display:'flex',alignItems:'flex-start',gap:8,padding:'8px 10px',background:C.bg,borderRadius:10,marginBottom:5,border:`1px solid ${C.border}`,boxSizing:'border-box'}}><span style={{fontSize:16,marginTop:1}}>{rt.icon}</span><div style={{flex:1}}><p style={{fontSize:sz(13),fontWeight:800,color:C.umber,margin:'0 0 1px',textAlign:'left',fontFamily:famBody}}>{rt.label} {other.emoji} {other.name}</p>{rel.note&&<p style={{fontSize:sz(11),color:C.muted,margin:0,textAlign:'left',fontFamily:famBody}}>{rel.note}</p>}</div><button onClick={()=>removeRel(rel.partId)} style={{background:'none',border:'none',color:C.muted,cursor:'pointer',fontSize:17,padding:0,lineHeight:1}}>×</button></div>);})}
          </div>

          <div style={{display:'flex',gap:7,paddingBottom:14}}><button onClick={()=>{setEp({...cp});setView('partEditor');}} style={sb({flex:1,fontSize:sz(12)})}>Edit</button><button onClick={()=>deletePart(cp.id)} style={{...sbo({fontSize:sz(12)}),color:C.burg,borderColor:C.burg}}>Delete</button></div>
        </div>
      </div>
    );
  };

  const renderPartEditor = () => {
    if(!ep) return null; 
    const set=f=>setEp(prev=>({...prev,...f}));
    return(
      <div style={{padding:'26px 15px 0'}}>
        <button onClick={()=>setView(selPart?'partDetail':'parts')} style={sbo({fontSize:sz(11),padding:'4px 10px',marginBottom:14})}>← Back</button>
        <h1 style={{...H1,marginBottom:3}}>{ep.name?`Editing ${ep.name}`:'New Part'}</h1>
        
        <div style={sc()}>
          <span style={slbl}>Core Identity</span>
          <input value={ep.name} onChange={e=>set({name:e.target.value})} placeholder="Name (e.g. The Inner Critic)" style={si({marginBottom:8, minHeight:'auto'})}/>
          <input value={ep.nickname} onChange={e=>set({nickname:e.target.value})} placeholder="Nickname (e.g. John)" style={si({marginBottom:12, minHeight:'auto'})}/>
          
          <span style={slbl}>Gender Energy</span>
          <div style={{display:'flex', gap:6, marginBottom:16}}>
            {[{id:'female', l:'♀ Female'}, {id:'male', l:'♂ Male'}, {id:'neutral', l:'✨ Neutral'}].map(g=>(
              <button key={g.id} onClick={()=>set({gender:g.id})} style={{flex:1, padding:'6px', borderRadius:8, border:`2px solid ${ep.gender===g.id?C.rust:C.border}`, background:ep.gender===g.id?`${C.rust}1A`:'transparent', color:ep.gender===g.id?C.rust:C.muted, fontSize:sz(11), fontWeight:800, fontFamily:famBody, cursor:'pointer'}}>{g.l}</button>
            ))}
          </div>

          <span style={slbl}>Symbol (Icon or Upload Image)</span>
          <div style={{display:'flex', alignItems:'center', gap:10, marginBottom:12}}>
            <label style={{...sbo({padding:'6px 12px'}), cursor:'pointer'}}>
              Upload Photo
              <input type="file" accept="image/*" style={{display:'none'}} onChange={e => {
                const file = e.target.files[0];
                if(file){ const r=new FileReader(); r.onload=()=>set({image:r.result}); r.readAsDataURL(file); }
              }} />
            </label>
            {ep.image && <button onClick={()=>set({image:null})} style={{...sbo({borderColor:C.burg, color:C.burg, padding:'6px 12px'})}}>Remove Photo</button>}
          </div>

          {!ep.image && (
            <div style={{display:'flex',flexWrap:'wrap',gap:4,marginBottom:12}}>
              {PE.map(e=><button key={e} onClick={()=>set({emoji:e})} style={{fontSize:20,padding:'4px 6px',borderRadius:7,cursor:'pointer',border:'2px solid',borderColor:ep.emoji===e?C.rust:'transparent',background:ep.emoji===e?`${C.rust}1A`:'transparent',boxSizing:'border-box'}}>{e}</button>)}
            </div>
          )}
          <span style={slbl}>Color</span><div style={{display:'flex',gap:6,flexWrap:'wrap'}}>{PC.map(c=><button key={c} onClick={()=>set({color:c})} style={{width:28,height:28,borderRadius:'50%',background:c,border:'3px solid',borderColor:ep.color===c?C.umber:'transparent',cursor:'pointer',padding:0,boxSizing:'border-box'}}/>)}</div>
        </div>

        <div style={sc()}>
          <div style={{display:'flex', justifyContent:'space-between', alignItems:'center', marginBottom:6}}>
            <span style={{...slbl, marginBottom:0}}>IFS Role</span>
            <button onClick={()=>setShowRoleHelp(!showRoleHelp)} style={{background:'none',border:'none',color:C.rust,fontSize:sz(11),fontWeight:800,cursor:'pointer'}}>What is this?</button>
          </div>
          {showRoleHelp && (
            <div style={{padding:'10px', background:`${C.rust}1A`, borderRadius:10, marginBottom:12}}>
              <p style={{fontSize:sz(11), color:C.umber, margin:'0 0 6px', fontFamily:famBody}}><strong>Managers:</strong> Proactive protectors (planners, critics) preventing pain.</p>
              <p style={{fontSize:sz(11), color:C.umber, margin:'0 0 6px', fontFamily:famBody}}><strong>Firefighters:</strong> Reactive protectors (distractors, impulsivity) putting out pain.</p>
              <p style={{fontSize:sz(11), color:C.umber, margin:0, fontFamily:famBody}}><strong>Exiles:</strong> The wounded ones carrying past pain or shame.</p>
            </div>
          )}
          <select value={ep.role} onChange={e=>set({role:e.target.value})} style={si({marginBottom:12, minHeight:'auto'})}>
            <option value="">Unassigned...</option><option value="manager">Manager</option><option value="firefighter">Firefighter</option><option value="exile">Exile</option>
          </select>
          <span style={slbl}>Age / Maturity</span>
          <input value={ep.age} onChange={e=>set({age:e.target.value})} placeholder="e.g. 5 years old, Ageless" style={si({marginBottom:0, minHeight:'auto'})}/>
        </div>

        <div style={sc()}>
          <span style={slbl}>Deeper Understanding (Optional)</span>
          <textarea value={ep.coreFear} onChange={e=>set({coreFear:e.target.value})} placeholder="Core Fear (What happens if it stops working?)" style={sta({height:80, marginBottom:8})}/>
          <textarea value={ep.positiveIntent} onChange={e=>set({positiveIntent:e.target.value})} placeholder="Positive Intent (What does it need/want for you?)" style={sta({height:80, marginBottom:8})}/>
          <textarea value={ep.vibe} onChange={e=>set({vibe:e.target.value})} placeholder="The Vibe (Appearance, tone of voice, energy)" style={sta({height:80, marginBottom:8})}/>
          <textarea value={ep.description} onChange={e=>set({description:e.target.value})} placeholder="General Description..." style={sta({height:80, marginBottom:8})}/>
          <textarea value={ep.origin} onChange={e=>set({origin:e.target.value})} placeholder="Origin Story (When did it form?)" style={sta({height:80, marginBottom:8})}/>
          <input value={ep.bodyLocation} onChange={e=>set({bodyLocation:e.target.value})} placeholder="Body Location (e.g. tight chest)" style={si({minHeight:'auto'})}/>
        </div>

        <button onClick={()=>upsertPart(ep)} disabled={!ep.name.trim()} style={sb({width:'100%',marginBottom:20,fontSize:sz(14),padding:'12px',opacity:!ep.name.trim()?.5:1})}>Save</button>
      </div>
    );
  };

  const renderHabits = () => {
    const today=todayK(),done=habits.filter(h=>h.completions[today]).length;
    return(
      <div style={{padding:'26px 15px 0'}}>
        <div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-start',marginBottom:14}}>
          <div style={{flex: 1}}><h1 style={H1}>Habits 🌾</h1><p style={sub}>{done}/{habits.length} done today <span style={stag(C.forest,{marginLeft:4,fontSize:sz(9)})}>+{PTS.habit} drops 💧</span></p></div>
          <button onClick={()=>setAddH(true)} style={sb({padding:'8px 12px',fontSize:sz(12)})}>+ New</button>
        </div>
        
        {habits.length===0&&!addH&&(<div style={sc({textAlign:'center',padding:'42px 20px'})}><div style={{fontSize:50,marginBottom:10}}>🌾</div><h3 style={{...H3, textAlign:'center'}}>Nurture Your Routines</h3><p style={{...sub,marginBottom:18,textAlign:'center'}}>Tap once to mark completed (✓).<br/>Tap twice to honor needs (🤍) when you rest.<br/>Both build your streak. Zero perfectionism.</p><button onClick={()=>setAddH(true)} style={sb()}>Plant a habit</button></div>)}

        {addH&&(<div style={sc({border:`1px solid ${C.rust}`})}><h3 style={H3}>New habit</h3><div style={{display:'flex',flexWrap:'wrap',gap:4,marginBottom:10}}>{HEMOJIS.map(e=><button key={e} onClick={()=>setNh({...nh,emoji:e})} style={{fontSize:19,padding:'4px 5px',borderRadius:7,cursor:'pointer',border:'2px solid',borderColor:nh.emoji===e?C.rust:'transparent',background:nh.emoji===e?`${C.rust}1A`:'transparent',boxSizing:'border-box'}}>{e}</button>)}</div><div style={{display:'flex',gap:6,marginBottom:9}}><input value={nh.name} onChange={e=>setNh({...nh,name:e.target.value})} placeholder="e.g. Morning pages" style={{...si(),marginBottom:0, minHeight:'auto'}}/><select value={nh.frequency||'daily'} onChange={e=>setNh({...nh,frequency:e.target.value})} style={{...si(),width:'auto',marginBottom:0,padding:'10px', minHeight:'auto'}}><option value="daily">Daily</option><option value="weekly">Weekly</option><option value="monthly">Monthly</option></select></div><div style={{display:'flex',gap:7}}><button onClick={()=>{if(!nh.name.trim())return; setHabits([{id:uid(),name:nh.name,emoji:nh.emoji,frequency:nh.frequency,completions:{},streak:0},...habits]); setNh({name:'',emoji:'🌱',frequency:'daily'});setAddH(false);}} style={sb({flex:1})}>Add</button><button onClick={()=>setAddH(false)} style={sbo()}>Cancel</button></div></div>)}
        
        {habits.map(h=>{
          const d=h.completions[today];
          const isHonored = d === 'honored';
          const isDone = d === 'done';
          const bgCol = isHonored ? C.amber : C.forest;
          const last7=[...Array(7)].map((_,i)=>{const d=new Date();d.setDate(d.getDate()-i);return d.toISOString().slice(0,10);}).reverse();
          return(<div key={h.id} style={sc({border:`1px solid ${d?bgCol:C.border}`,background:d?`${bgCol}15`:C.card, position:'relative'})}>
            <button onClick={()=>deleteHabit(h.id)} style={{position:'absolute',top:10,right:10,background:'none',border:'none',color:C.muted,cursor:'pointer',fontSize:18}}>×</button>
            <div style={{display:'flex',alignItems:'center',gap:9,marginBottom:8}}><button onClick={()=>toggleHabit(h.id)} style={{width:30,height:30,borderRadius:9,border:`2px solid ${d?bgCol:C.border}`,background:d?bgCol:'transparent',cursor:'pointer',display:'flex',alignItems:'center',justifyContent:'center',padding:0,boxSizing:'border-box',flexShrink:0}}>{isDone&&<span style={{color:C.bg,fontSize:15,fontWeight:900,fontFamily:famBody}}>✓</span>}{isHonored&&<span style={{color:C.bg,fontSize:14,fontWeight:900,fontFamily:famBody}}>🤍</span>}</button><span style={{fontSize:21}}>{h.emoji}</span><div style={{flex:1, paddingRight:20}}><div style={{display:'flex',alignItems:'center',gap:6}}><p style={{fontSize:sz(13),fontWeight:800,color:d?bgCol:C.umber,margin:0,textDecoration:isDone?'line-through':'none',textAlign:'left',fontFamily:famBody}}>{h.name}</p><span style={stag(C.muted,{fontSize:sz(8),padding:'1px 5px'})}>{h.frequency||'daily'}</span></div></div>{(!h.frequency||h.frequency==='daily')&&h.streak>0&&<span style={{fontSize:sz(10),color:C.amber,fontWeight:800,fontFamily:famBody}}>🔥{h.streak}</span>}{(h.frequency&&h.frequency!=='daily')&&h.streak>0&&<span style={{fontSize:sz(10),color:C.amber,fontWeight:800,fontFamily:famBody}}>⭐{h.streak}</span>}</div><div style={{display:'flex',gap:3,paddingTop:5,borderTop:`1px solid ${C.border}`}}>{last7.map(k=>{const val=h.completions[k];const fill=val==='honored'?C.amber:val==='done'?C.forest:`${C.forest}33`;return <div key={k} style={{flex:1,height:6,borderRadius:4,background:fill}}/>;})}</div></div>);
        })}
      </div>
    );
  };

  const renderIdeas = () => {
    const filtered=ideas.filter(i=>!iFilter||i.title.toLowerCase().includes(iFilter.toLowerCase())||i.content?.toLowerCase().includes(iFilter.toLowerCase())||i.tag?.toLowerCase().includes(iFilter.toLowerCase()));
    const pinned=filtered.filter(i=>i.pinned),rest=filtered.filter(i=>!i.pinned);
    const allTags=[...new Set(ideas.map(i=>i.tag).filter(Boolean))];
    
    function IdeaCard(idea){
      const exp=expIdea===idea.id;
      const isRes = resIdea === idea.id;
      return(
        <div key={idea.id} style={sc({padding:'11px 13px',marginBottom:8})}>
          {!isRes ? (
            <div style={{display:'flex',alignItems:'flex-start',gap:7}}>
              <div style={{flex:1,cursor:'pointer'}} onClick={()=>setExpIdea(exp?null:idea.id)}>
                <p style={{fontSize:sz(14),fontWeight:800,color:C.umber,margin:'0 0 2px',lineHeight:1.4,textAlign:'left',fontFamily:famBody}}>{idea.title}</p>
                {idea.tag&&<span style={stag(C.amber,{fontSize:sz(9)})}>{idea.tag}</span>}
                {idea.content&&!exp&&<p style={{fontSize:sz(12),color:C.muted,margin:'3px 0 0',overflow:'hidden',textOverflow:'ellipsis',whiteSpace:'nowrap',textAlign:'left',fontFamily:famBody}}>{idea.content}</p>}
                {idea.content&&exp&&<p style={{fontSize:sz(13),color:C.umber,margin:'6px 0 0',lineHeight:1.7,textAlign:'left',fontFamily:famBody}}>{idea.content}</p>}
              </div>
              <div style={{display:'flex',gap:5,flexShrink:0}}>
                <button onClick={()=>togglePin(idea.id)} style={{background:'none',border:'none',cursor:'pointer',fontSize:14,opacity:idea.pinned?1:.3,padding:0}}>📌</button>
                <button onClick={()=>setResIdea(idea.id)} style={{background:'none',border:'none',color:C.muted,cursor:'pointer',fontSize:17,padding:0,lineHeight:1}}>×</button>
              </div>
            </div>
          ) : (
            <div>
              <p style={{fontSize:sz(12), fontWeight:800, color:C.umber, margin:'0 0 8px', textAlign:'center'}}>Resolve this idea?</p>
              <div style={{display:'flex', gap:6}}>
                <button onClick={()=>resolveIdea(idea.id, 'done')} style={{flex:1, padding:'6px', borderRadius:8, border:`1px solid ${C.forest}`, background:`${C.forest}1A`, color:C.forest, fontSize:sz(11), fontWeight:800, cursor:'pointer'}}>Done! 🎊</button>
                <button onClick={()=>resolveIdea(idea.id, 'archive')} style={{flex:1, padding:'6px', borderRadius:8, border:`1px solid ${C.amber}`, background:`${C.amber}1A`, color:C.amber, fontSize:sz(11), fontWeight:800, cursor:'pointer'}}>Not for now 🍂</button>
                <button onClick={()=>setResIdea(null)} style={{padding:'6px', background:'none', border:'none', color:C.muted, cursor:'pointer'}}>Cancel</button>
              </div>
            </div>
          )}
          {!isRes && <p style={{fontSize:sz(9),color:C.muted,margin:'5px 0 0',textAlign:'left',fontFamily:famBody}}>{fmtD(idea.date)}</p>}
        </div>
      );
    }

    return(
      <div style={{padding:'26px 15px 0'}}>
        <div style={{display:'flex',justifyContent:'space-between',alignItems:'flex-start',marginBottom:14}}><div style={{flex: 1}}><h1 style={H1}>Idea Garden 💡</h1><p style={sub}>{ideas.length} idea{ideas.length!==1?'s':''} growing</p></div><button onClick={()=>setAddI(true)} style={sb({padding:'8px 12px',fontSize:sz(12)})}>+ Capture</button></div>
        {ideas.length>0&&<input value={iFilter} onChange={e=>setIFilter(e.target.value)} placeholder="🔍 Search ideas…" style={si({marginBottom:8, minHeight:'auto'})}/>}
        {allTags.length>0&&<div style={{display:'flex',gap:5,flexWrap:'wrap',marginBottom:11,boxSizing:'border-box'}}><button onClick={()=>setIFilter('')} style={stag(iFilter===''?C.rust:C.muted,{cursor:'pointer',padding:'4px 10px',fontSize:sz(11),boxSizing:'border-box'})}>All</button>{allTags.map(t=><button key={t} onClick={()=>setIFilter(iFilter===t?'':t)} style={stag(iFilter===t?C.rust:C.muted,{cursor:'pointer',padding:'4px 10px',fontSize:sz(11),boxSizing:'border-box'})}>{t}</button>)}</div>}
        {addI&&(<div style={sc({border:`1px solid ${C.amber}`})}><h3 style={H3}>New idea ✨</h3><input value={ni.title} onChange={e=>setNi({...ni,title:e.target.value})} placeholder="What's the idea?" style={si({marginBottom:7, minHeight:'auto'})}/><textarea value={ni.content} onChange={e=>setNi({...ni,content:e.target.value})} placeholder="Expand on it… (optional)" style={sta({height:80,marginBottom:7})}/><input value={ni.tag} onChange={e=>setNi({...ni,tag:e.target.value})} placeholder="Tag: work, personal…" style={si({marginBottom:9, minHeight:'auto'})}/><div style={{display:'flex',gap:7}}><button onClick={addIdea} disabled={!ni.title.trim()} style={sb({flex:1,opacity:!ni.title.trim()?.5:1})}>Capture</button><button onClick={()=>setAddI(false)} style={sbo()}>Cancel</button></div></div>)}
        {ideas.length===0&&!addI?(<div style={sc({textAlign:'center',padding:'42px 20px'})}><div style={{fontSize:50,marginBottom:10}}>💡</div><p style={{...sub,marginBottom:18,textAlign:'center'}}>A place for ideas that flash through your mind before they disappear.</p><button onClick={()=>setAddI(true)} style={sb()}>Capture your first idea</button></div>)
        :(<>{pinned.length>0&&<div style={{marginBottom:4}}><span style={slbl}>📌 Pinned</span>{pinned.map(i=>IdeaCard(i))}</div>}{rest.length>0&&<div>{pinned.length>0&&rest.length>0&&<span style={slbl}>All ideas</span>}{rest.map(i=>IdeaCard(i))}</div>}</>)}
      </div>
    );
  };

  const renderSettings = () => {
    return (
      <div style={{padding:'26px 15px 0'}}>
        <button onClick={()=>setView('home')} style={sbo({fontSize:sz(11),padding:'4px 10px',marginBottom:14})}>← Home</button>
        <h1 style={H1}>Settings & Compost</h1>
        
        <div style={sc()}>
          <h3 style={H3}>Compost Bin 🗑️</h3>
          <p style={{...sub, fontSize:sz(11), marginBottom:12}}>Items deleted within the last 30 days are kept safely here.</p>
          {trash.length === 0 ? <p style={{...sub, fontSize:sz(12), fontStyle:'italic'}}>The compost bin is empty.</p> : (
            <div>
              {trash.map(t => (
                <div key={t.id} style={{display:'flex', justifyContent:'space-between', alignItems:'center', padding:'8px', borderBottom:`1px solid ${C.border}`}}>
                  <div style={{overflow:'hidden'}}>
                    <span style={stag(C.muted, {fontSize:sz(8)})}>{t.type.toUpperCase()}</span>
                    <p style={{fontSize:sz(12), color:C.umber, margin:'2px 0 0', whiteSpace:'nowrap', textOverflow:'ellipsis', overflow:'hidden'}}>{t.data.name || t.data.title || t.data.text || 'Item'}</p>
                  </div>
                  <button onClick={()=>restoreFromTrash(t.id)} style={sbo({padding:'4px 8px', fontSize:sz(10)})}>Restore</button>
                </div>
              ))}
              <button onClick={emptyTrash} style={{...sbo({fontSize:sz(11), color:C.burg, borderColor:C.burg}), marginTop:12, width:'100%'}}>Empty Compost Forever</button>
            </div>
          )}
        </div>
      </div>
    );
  };

  const NAV=[{id:'home',icon:'🏠',label:'Home'},{id:'journal',icon:'✍️',label:'Journal'},{id:'parts',icon:'🌱',label:'Parts'},{id:'habits',icon:'🌾',label:'Habits'},{id:'ideas',icon:'💡',label:'Ideas'}];

  return(
    <div style={{fontFamily:famBody,background:C.bg,minHeight:'100vh',maxWidth:440,margin:'0 auto',paddingBottom:90,position:'relative',width:'100%',boxSizing:'border-box',overflowX:'hidden', boxShadow:'0 0 40px rgba(0,0,0,0.1)'}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&family=Nunito:wght@400;500;700;800&display=swap');
        body { background: #E0E4E8; margin: 0; }
        * { box-sizing: border-box; margin: 0; }
        textarea { resize: none; }
        input:focus, textarea:focus, select:focus { border-color: ${C.rust} !important; box-shadow: 0 0 0 3px ${C.rust}33; }
        ::-webkit-scrollbar { width: 3px; }
        ::-webkit-scrollbar-thumb { background: ${C.rust}44; border-radius: 2px; }
        button:active { opacity: .82; transform: scale(.97); }
        select { appearance: none; }
        @keyframes shimmer { 0% { box-shadow: 0 0 4px currentColor; } 50% { box-shadow: 0 0 16px currentColor, inset 0 0 8px currentColor; } 100% { box-shadow: 0 0 4px currentColor; } }
      `}</style>

      {/* Confetti Explosion Layer */}
      <Confetti active={showConfetti} colors={confettiColors} />

      {/* Sync Status Pill */}
      {statusVisible && (
        <div style={{position:'fixed', top: 16, right: 'calc(50% - 200px)', background: C.card, border: `1px solid ${C.border}`, borderRadius: 20, padding: '5px 8px 5px 12px', fontSize: sz(11), fontWeight: 800, color: status === 'Saved' ? C.forest : status === 'Saving...' ? C.amber : C.muted, zIndex: 100, display: 'flex', alignItems: 'center', gap: 6, boxShadow: '0 2px 10px rgba(0,0,0,.08)', fontFamily: famBody}}>
          <div style={{width: 6, height: 6, borderRadius: '50%', background: status === 'Saved' ? C.forest : status === 'Saving...' ? C.amber : C.muted}}/>
          {status}
          <button onClick={() => setStatusVisible(false)} style={{background: 'none', border: 'none', color: C.muted, cursor: 'pointer', fontSize: 16, lineHeight: 1, padding: '0 0 2px 4px', display: 'flex'}} aria-label="Close status">×</button>
        </div>
      )}

      {/* Profile & Settings Menu Overlay */}
      {showMenu&&(
        <div style={{position:'fixed',inset:0,background:'rgba(0,0,0,.6)',zIndex:400,display:'flex',alignItems:'flex-end', justifyContent:'center'}} onClick={()=>setShowMenu(false)}>
          <div style={{width:'100%',maxWidth:440,background:C.card,borderRadius:'24px 24px 0 0',padding:'24px 20px 36px', borderTop:`1px solid ${C.border}`,boxSizing:'border-box',maxHeight:'85vh',overflowY:'auto'}} onClick={e=>e.stopPropagation()}>
            <div style={{display:'flex',alignItems:'center',gap:13,marginBottom:20}}>
              <div style={{width:48,height:48,borderRadius:'50%',background:`linear-gradient(135deg,${C.rust},${C.amber})`,display:'flex',alignItems:'center',justifyContent:'center',fontSize:22,color:'#FFF',fontWeight:800}}>{userName ? userName[0].toUpperCase() : '🍂'}</div>
              <div><p style={{fontFamily:famHead,fontSize:sz(16),fontWeight:700,color:C.umber,margin:'0 0 2px',textAlign:'left'}}>{userName || 'Journaler'}</p><p style={{fontSize:sz(11),color:C.muted,margin:0,textAlign:'left',fontFamily:famBody}}>{points} drops 💧</p></div>
            </div>
            
            <div style={{marginBottom: 20}}>
              <span style={slbl}>Color Theme</span>
              <div style={{display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 8, marginTop: 8}}>
                {Object.entries(THEMES).map(([k, t]) => (
                  <button key={k} onClick={() => setTheme(k)} style={{padding: '10px', borderRadius: 12, border: `2px solid ${theme === k ? C.rust : C.border}`, background: theme === k ? `${C.rust}1A` : C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800, display: 'flex', alignItems: 'center', gap: 6 }}>
                    <span style={{fontSize: 16}}>{t.icon}</span> {t.name}
                  </button>
                ))}
              </div>
            </div>

            <div style={{marginBottom: 20}}>
              <span style={slbl}>Text Size</span>
              <div style={{display: 'flex', alignItems: 'center', gap: 12, marginTop: 8, padding: '10px', background: C.bg, borderRadius: 12, border: `1px solid ${C.border}`}}>
                <span style={{fontSize: 12, fontWeight: 800, color: C.umber, fontFamily: famBody}}>A</span>
                <input type="range" min="0.9" max="1.5" step="0.05" value={textScale} onChange={(e) => setTextScale(parseFloat(e.target.value))} style={{flex: 1, accentColor: C.rust}} />
                <span style={{fontSize: 22, fontWeight: 800, color: C.umber, fontFamily: famBody}}>A</span>
              </div>
              <div style={{marginTop: 12}}>
                <button onClick={() => setFontStyle(fontStyle === 'default' ? 'dyslexic' : 'default')} style={{width:'100%', padding: '10px', borderRadius: 12, border: `2px solid ${fontStyle === 'dyslexic' ? C.rust : C.border}`, background: fontStyle === 'dyslexic' ? `${C.rust}1A` : C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800 }}>
                  {fontStyle === 'dyslexic' ? 'Default Font' : 'Dyslexia-Friendly Font'}
                </button>
              </div>
            </div>

            <div style={{marginBottom: 24}}>
              <span style={slbl}>Customize Layout</span>
              <div style={{marginTop: 8, display: 'flex', flexDirection:'column', gap:8}}>
                <button onClick={() => setUsePrompts(!usePrompts)} style={{width:'100%', padding: '10px', borderRadius: 12, border: `2px solid ${usePrompts ? C.rust : C.border}`, background: usePrompts ? `${C.rust}1A` : C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800 }}>
                  {usePrompts ? 'Journaling Prompts: ON' : 'Journaling Prompts: OFF'}
                </button>
                <button onClick={() => setHideSearch(!hideSearch)} style={{width:'100%', padding: '10px', borderRadius: 12, border: `2px solid ${hideSearch ? C.rust : C.border}`, background: hideSearch ? `${C.rust}1A` : C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800 }}>
                  {hideSearch ? 'Show Search Widget' : 'Hide Search Widget'}
                </button>
                <button onClick={() => setHideThemes(!hideThemes)} style={{width:'100%', padding: '10px', borderRadius: 12, border: `2px solid ${hideThemes ? C.rust : C.border}`, background: hideThemes ? `${C.rust}1A` : C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800 }}>
                  {hideThemes ? 'Show Themes Widget' : 'Hide Themes Widget'}
                </button>
                <button onClick={() => setHideReadwise(!hideReadwise)} style={{width:'100%', padding: '10px', borderRadius: 12, border: `2px solid ${hideReadwise ? C.rust : C.border}`, background: hideReadwise ? `${C.rust}1A` : C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800 }}>
                  {hideReadwise ? 'Show Readwise Widget' : 'Hide Readwise Widget'}
                </button>
                <button onClick={() => { setShowMenu(false); setView('settings'); window.scrollTo(0,0); }} style={{width:'100%', padding: '10px', borderRadius: 12, border: `2px solid ${C.border}`, background: C.bg, color: C.umber, cursor: 'pointer', fontFamily: famBody, fontSize: sz(12), fontWeight: 800 }}>
                  Open Compost Bin 🗑️ ({trash.length})
                </button>
              </div>
            </div>

            <button onClick={()=>{setShowMenu(false); handleSignOut();}} style={sbo({width:'100%',padding:'12px',fontSize:sz(13), marginBottom: 12})}>Sign Out of Google Drive</button>
            
            <button onClick={()=>{setShowMenu(false); wipeAllData();}} style={{width:'100%', padding:'12px', fontSize:sz(13), background:'transparent', border:`1.5px solid ${C.burg}`, color:C.burg, borderRadius:11, fontFamily:famBody, fontWeight:700, cursor:'pointer', boxSizing:'border-box'}}>
              🚨 Delete All My Data Permanently
            </button>
          </div>
        </div>
      )}

      {ptFlash&&(<div style={{position:'fixed',top:14,left:'50%',transform:'translateX(-50%)',background:`linear-gradient(135deg,${C.umber},${C.burg})`,borderRadius:22,padding:'7px 16px',zIndex:300,boxShadow:'0 4px 20px rgba(0,0,0,.35)',display:'flex',alignItems:'center',gap:7,whiteSpace:'nowrap'}}><span style={{fontSize:15}}>💧</span><span style={{fontSize:sz(13),fontWeight:800,color:'#FFF',fontFamily:famBody}}>+{ptFlash.amt}</span><span style={{fontSize:sz(11),color:C.amber,fontFamily:famBody}}>{ptFlash.label}</span></div>)}
      {toast&&(<div style={{position:'fixed',top:14,left:'50%',transform:'translateX(-50%)',background:C.umber,borderRadius:22,padding:'7px 16px',zIndex:300,boxShadow:'0 4px 20px rgba(0,0,0,.25)',whiteSpace:'nowrap'}}><span style={{fontSize:sz(13),fontWeight:800,color:'#FFF',fontFamily:famBody}}>{toast}</span></div>)}

      {/* App Views Rendered Safely as Functions */}
      {view==='home'&&renderHome()}
      {view==='journal'&&renderJournal()}
      {view==='parts'&&renderPartsList()}
      {view==='partDetail'&&renderPartDetail()}
      {view==='partEditor'&&renderPartEditor()}
      {view==='habits'&&renderHabits()}
      {view==='ideas'&&renderIdeas()}
      {view==='settings'&&renderSettings()}

      {/* Bottom Nav */}
      <div style={{position:'fixed',bottom:0,left:'50%',transform:'translateX(-50%)',width:'100%',maxWidth:440,background:C.card,borderTop:`1px solid ${C.border}`,display:'flex',padding:'8px 0 calc(18px + env(safe-area-inset-bottom))',zIndex:100,boxSizing:'border-box'}}>
        {NAV.map(({id,icon,label})=>(
          <button key={id} onClick={()=>{setView(id); window.scrollTo(0,0);}} style={{flex:1,background:'none',border:'none',cursor:'pointer',padding:'2px 0',display:'flex',flexDirection:'column',alignItems:'center',gap:2,boxSizing:'border-box'}}>
            <span style={{fontSize:19,opacity:view===id?1:.28}}>{icon}</span>
            <span style={{fontSize:sz(8),fontFamily:famBody,fontWeight:800,letterSpacing:'.06em',color:view===id?C.rust:C.muted,textTransform:'uppercase'}}>{label}</span>
            {view===id&&<div style={{width:3,height:3,borderRadius:'50%',background:C.rust}}/>}
          </button>
        ))}
      </div>
    </div>
  );
}
