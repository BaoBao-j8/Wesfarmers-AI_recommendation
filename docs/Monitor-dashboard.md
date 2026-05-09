import { useState, useEffect, useRef, useCallback } from "react";
import { LineChart, Line, AreaChart, Area, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Legend } from "recharts";
import { LayoutDashboard, ShoppingCart, Brain, Palette, MessageSquare, Users, ChevronRight, TrendingUp, TrendingDown, AlertTriangle, CheckCircle, RefreshCw, Settings, LogOut, Bell, Search, ArrowUpRight, Zap, Package, DollarSign, Activity, Shield, BarChart2, ChevronDown, Send, User, Bot, Star, Filter, MoreHorizontal, Plus, Eye, Cpu, Globe, Lock } from "lucide-react";

const BRANDS = [
  { id: "bunnings", name: "Bunnings", color: "#E8221E", bg: "#FFF0F0", short: "BUN" },
  { id: "kmart", name: "Kmart", color: "#E8221E", bg: "#FFF0F0", short: "KMT" },
  { id: "target", name: "Target", color: "#C8102E", bg: "#FFF0F2", short: "TGT" },
  { id: "officeworks", name: "Officeworks", color: "#5B2D8E", bg: "#F5F0FF", short: "OFW" },
  { id: "catch", name: "Catch", color: "#FF6B00", bg: "#FFF4EC", short: "CTH" },
  { id: "priceline", name: "Priceline", color: "#E91E8C", bg: "#FFF0F8", short: "PRL" },
  { id: "apihealth", name: "API Health", color: "#00897B", bg: "#F0FAFA", short: "APH" },
];

const G = {
  navy: "#0A2240",
  blue: "#005DAA",
  green: "#00A651",
  glass: "rgba(255,255,255,0.72)",
  glassBorder: "rgba(255,255,255,0.9)",
  glassDark: "rgba(255,255,255,0.45)",
  bg: "linear-gradient(135deg, #e8f4fd 0%, #f0faf4 40%, #eaf0f8 100%)",
  navBg: "rgba(10,34,64,0.96)",
  text: "#0A2240",
  textMid: "#3A5275",
  textLight: "#7A94B0",
  border: "rgba(0,93,170,0.12)",
  shadow: "0 8px 32px rgba(10,34,64,0.10), 0 1.5px 4px rgba(10,34,64,0.07)",
  shadowSm: "0 2px 12px rgba(10,34,64,0.08)",
};

const glass = {
  background: G.glass,
  backdropFilter: "blur(20px)",
  WebkitBackdropFilter: "blur(20px)",
  border: `1px solid ${G.glassBorder}`,
  borderRadius: 20,
  boxShadow: G.shadow,
};

const glassCard = { ...glass, padding: "1.25rem 1.5rem" };

function rand(min, max) { return Math.floor(Math.random() * (max - min + 1)) + min; }
function fmtM(n) { return n >= 1000 ? `$${(n/1000).toFixed(1)}B` : `$${n}M`; }
function fmtK(n) { return n >= 1000 ? `${(n/1000).toFixed(1)}k` : n; }

function genRevData() {
  return Array.from({length:12}, (_,i) => ({
    month: ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"][i],
    Bunnings: rand(580,720), Kmart: rand(420,540), Officeworks: rand(180,260),
    Catch: rand(90,150), Priceline: rand(120,180), Target: rand(200,280), APIHealth: rand(60,110),
  }));
}

function genForecastData(brand) {
  const base = { bunnings:580, kmart:420, target:200, officeworks:220, catch:120, priceline:140, apihealth:80 }[brand] || 300;
  const days = [];
  let v = base;
  for(let i=0;i<90;i++){
    v = v + rand(-8,12);
    days.push({ day: i+1, actual: i<30?v:null, forecast: i>=25?v+rand(-5,15):null, lower: i>=25?v-20:null, upper: i>=25?v+30:null });
  }
  return days;
}

function genVendorData() {
  return [
    { vendor:"Anthropic Claude", cost:12400, perf:97, calls:48200, brand:"All brands", color:"#7C3AED" },
    { vendor:"OpenAI GPT-4o", cost:18700, perf:94, calls:92100, brand:"Bunnings, Kmart", color:"#059669" },
    { vendor:"Google Vertex", cost:9200, perf:91, calls:31400, brand:"Officeworks", color:"#2563EB" },
    { vendor:"Internal Model", cost:2100, perf:88, calls:156000, brand:"All brands", color:"#D97706" },
  ];
}

const INVENTORY_ALERTS = [
  { brand:"Bunnings", item:"42mm PVC Pipe 6m", stock:12, reorder:200, urgency:"critical", aiQty:250 },
  { brand:"Kmart", item:"Kids Desk Lamp", stock:34, reorder:150, urgency:"high", aiQty:180 },
  { brand:"Officeworks", item:"A4 Copy Paper Ream", stock:88, reorder:500, urgency:"medium", aiQty:600 },
  { brand:"Catch", item:"USB-C Hub 7-Port", stock:21, reorder:80, urgency:"high", aiQty:100 },
  { brand:"Priceline", item:"SPF50+ Sunscreen 200ml", stock:45, reorder:300, urgency:"medium", aiQty:350 },
];

const PERSONAS = [
  { name:"The Weekend DIYer", size:"2.3M", brands:"Bunnings, Kmart, Catch", lift:"+34%", desc:"35–55 homeowners, high spend Q1/Q3, project-driven" },
  { name:"The Family Organiser", size:"1.8M", brands:"Kmart, Target, Officeworks", lift:"+28%", desc:"25–45, school-year peaks, value-sensitive, loyalty-driven" },
  { name:"The Health Conscious", size:"890K", brands:"Priceline, API Health", lift:"+41%", desc:"30–60, repeat health purchases, subscription-ready" },
  { name:"The Home Office Pro", size:"1.1M", brands:"Officeworks, Kmart, Catch", lift:"+31%", desc:"WFH professionals, tech-forward, premium buyers" },
];

const PRODUCTS = [
  { id:1, brand:"Officeworks", name:"ErgoDesk Pro Adjustable", price:349, img:"🖥️", match:96 },
  { id:2, brand:"Kmart", name:"Home Office Chair Deluxe", price:89, img:"🪑", match:92 },
  { id:3, brand:"Catch", name:"USB-C 7-Port Hub", price:49, img:"🔌", match:88 },
  { id:4, brand:"Officeworks", name:"Dual Monitor Stand", price:79, img:"🖥️", match:85 },
  { id:5, brand:"Kmart", name:"Desk Organiser Set", price:29, img:"📦", match:81 },
  { id:6, brand:"Catch", name:"Webcam HD 1080p", price:65, img:"📷", match:79 },
];

// ─── LAYOUT ───────────────────────────────────────────────────────────────────

function Sidebar({ page, setPage, role, setRole }) {
  const navItems = [
    { id:"command", label:"Command Centre", icon:LayoutDashboard, roles:["exec"] },
    { id:"demand", label:"Demand AI", icon:TrendingUp, roles:["exec","operator"] },
    { id:"orchestration", label:"Model Orchestration", icon:Cpu, roles:["exec","operator"] },
    { id:"product", label:"Product Design AI", icon:Palette, roles:["operator"] },
    { id:"chat", label:"Shopping Agent", icon:MessageSquare, roles:["customer"] },
    { id:"personalisation", label:"Personalisation", icon:Users, roles:["exec","operator"] },
  ];

  const visible = navItems.filter(n => n.roles.includes(role));

  return (
    <div style={{ width:240, minWidth:240, height:"100vh", background:G.navBg, display:"flex", flexDirection:"column", position:"sticky", top:0, zIndex:50 }}>
      <div style={{ padding:"1.5rem 1.25rem 1rem", borderBottom:"1px solid rgba(255,255,255,0.08)" }}>
        <div style={{ display:"flex", alignItems:"center", gap:10, marginBottom:16 }}>
          <div style={{ width:36, height:36, borderRadius:10, background:`linear-gradient(135deg, ${G.green}, ${G.blue})`, display:"flex", alignItems:"center", justifyContent:"center" }}>
            <Zap size={18} color="#fff" />
          </div>
          <div>
            <div style={{ color:"#fff", fontWeight:700, fontSize:15, letterSpacing:"-0.3px" }}>OneIntelligence</div>
            <div style={{ color:"rgba(255,255,255,0.45)", fontSize:10, letterSpacing:"0.1em", textTransform:"uppercase" }}>Wesfarmers AI Platform</div>
          </div>
        </div>
        <div style={{ background:"rgba(255,255,255,0.07)", borderRadius:10, padding:"6px 8px" }}>
          <div style={{ fontSize:10, color:"rgba(255,255,255,0.4)", marginBottom:4, textTransform:"uppercase", letterSpacing:"0.08em" }}>Viewing as</div>
          <select value={role} onChange={e=>{ setRole(e.target.value); setPage(e.target.value==="customer"?"chat":"command"); }}
            style={{ width:"100%", background:"transparent", border:"none", color:"#fff", fontSize:13, fontWeight:500, cursor:"pointer", outline:"none" }}>
            <option value="exec" style={{background:G.navy}}>Executive / C-Suite</option>
            <option value="operator" style={{background:G.navy}}>Brand Operator</option>
            <option value="customer" style={{background:G.navy}}>Customer</option>
          </select>
        </div>
      </div>
      <nav style={{ flex:1, padding:"0.75rem 0.75rem", display:"flex", flexDirection:"column", gap:2 }}>
        {visible.map(item => {
          const Icon = item.icon;
          const active = page === item.id;
          return (
            <button key={item.id} onClick={()=>setPage(item.id)}
              style={{ display:"flex", alignItems:"center", gap:10, padding:"9px 12px", borderRadius:10, border:"none", cursor:"pointer", transition:"all 0.15s",
                background: active ? `linear-gradient(135deg, ${G.blue}22, ${G.green}18)` : "transparent",
                borderLeft: active ? `3px solid ${G.green}` : "3px solid transparent",
                color: active ? "#fff" : "rgba(255,255,255,0.55)" }}>
              <Icon size={16} />
              <span style={{ fontSize:13, fontWeight: active?500:400 }}>{item.label}</span>
            </button>
          );
        })}
      </nav>
      <div style={{ padding:"1rem 1.25rem", borderTop:"1px solid rgba(255,255,255,0.08)" }}>
        <div style={{ display:"flex", alignItems:"center", gap:8, padding:"8px 10px", borderRadius:10, background:"rgba(255,255,255,0.06)" }}>
          <div style={{ width:28, height:28, borderRadius:"50%", background:`linear-gradient(135deg, ${G.green}, ${G.blue})`, display:"flex", alignItems:"center", justifyContent:"center" }}>
            <User size={13} color="#fff" />
          </div>
          <div style={{ flex:1 }}>
            <div style={{ color:"#fff", fontSize:12, fontWeight:500 }}>Rob Scott</div>
            <div style={{ color:"rgba(255,255,255,0.4)", fontSize:10 }}>Group MD</div>
          </div>
          <Settings size={13} color="rgba(255,255,255,0.35)" style={{cursor:"pointer"}} />
        </div>
      </div>
    </div>
  );
}

function TopBar({ title, subtitle }) {
  return (
    <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", marginBottom:"1.5rem" }}>
      <div>
        <h1 style={{ margin:0, fontSize:22, fontWeight:700, color:G.navy, letterSpacing:"-0.5px" }}>{title}</h1>
        {subtitle && <p style={{ margin:"2px 0 0", fontSize:13, color:G.textMid }}>{subtitle}</p>}
      </div>
      <div style={{ display:"flex", alignItems:"center", gap:10 }}>
        <div style={{ ...glassCard, padding:"7px 14px", display:"flex", alignItems:"center", gap:7, cursor:"pointer" }}>
          <Search size={13} color={G.textLight} />
          <span style={{ fontSize:13, color:G.textLight }}>Search platform...</span>
        </div>
        <div style={{ position:"relative" }}>
          <div style={{ ...glass, padding:"8px", borderRadius:12, cursor:"pointer" }}>
            <Bell size={16} color={G.navy} />
          </div>
          <div style={{ position:"absolute", top:5, right:5, width:7, height:7, borderRadius:"50%", background:G.green, border:"1.5px solid #fff" }}/>
        </div>
      </div>
    </div>
  );
}

function KpiTile({ label, value, delta, positive, icon: Icon, color }) {
  return (
    <div style={{ ...glassCard, flex:1, minWidth:0 }}>
      <div style={{ display:"flex", alignItems:"flex-start", justifyContent:"space-between", marginBottom:10 }}>
        <div style={{ width:36, height:36, borderRadius:10, background:`${color}18`, display:"flex", alignItems:"center", justifyContent:"center" }}>
          <Icon size={17} color={color} />
        </div>
        <div style={{ display:"flex", alignItems:"center", gap:4, fontSize:12, fontWeight:500,
          color: positive ? G.green : "#E8221E",
          background: positive ? "#E8F8F0" : "#FFF0F0", padding:"3px 8px", borderRadius:20 }}>
          {positive ? <TrendingUp size={11}/> : <TrendingDown size={11}/>}
          {delta}
        </div>
      </div>
      <div style={{ fontSize:26, fontWeight:700, color:G.navy, letterSpacing:"-1px", marginBottom:3 }}>{value}</div>
      <div style={{ fontSize:12, color:G.textLight }}>{label}</div>
    </div>
  );
}

// ─── PAGES ───────────────────────────────────────────────────────────────────

function CommandCentre() {
  const [revData, setRevData] = useState(genRevData());
  const [kpis, setKpis] = useState({ rev:2840, tx:1247, inv:87, models:312 });
  const [activeBrand, setActiveBrand] = useState(null);

  useEffect(()=>{
    const t = setInterval(()=>{
      setKpis(k => ({ rev: k.rev + rand(-5,12), tx: k.tx + rand(-3,8), inv: Math.max(80,Math.min(98, k.inv + rand(-1,1))), models: k.models + rand(-2,5) }));
    }, 2500);
    return ()=>clearInterval(t);
  },[]);

  return (
    <div>
      <TopBar title="Command Centre" subtitle="Real-time group intelligence across all 7 brands · Live" />
      <div style={{ display:"flex", gap:12, marginBottom:"1.25rem" }}>
        <KpiTile label="Group Revenue (YTD)" value={fmtM(kpis.rev)} delta="+8.3%" positive icon={DollarSign} color={G.blue} />
        <KpiTile label="Transactions Today" value={`${fmtK(kpis.tx)}k`} delta="+5.1%" positive icon={ShoppingCart} color={G.green} />
        <KpiTile label="Inventory Health" value={`${kpis.inv}%`} delta="+2.1%" positive icon={Package} color="#F59E0B" />
        <KpiTile label="AI Model Calls/hr" value={`${fmtK(kpis.models)}k`} delta="+12.4%" positive icon={Brain} color={G.blue} />
      </div>

      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12, marginBottom:"1.25rem" }}>
        <div style={{ ...glassCard }}>
          <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", marginBottom:14 }}>
            <div style={{ fontSize:14, fontWeight:600, color:G.navy }}>Cross-Brand Revenue Trend</div>
            <div style={{ fontSize:12, color:G.green, background:"#E8F8F0", padding:"3px 10px", borderRadius:20 }}>Live</div>
          </div>
          <ResponsiveContainer width="100%" height={200}>
            <AreaChart data={revData} margin={{top:0,right:0,left:-20,bottom:0}}>
              <defs>
                {BRANDS.slice(0,4).map(b=>(
                  <linearGradient key={b.id} id={`g_${b.id}`} x1="0" y1="0" x2="0" y2="1">
                    <stop offset="5%" stopColor={b.color} stopOpacity={0.15}/>
                    <stop offset="95%" stopColor={b.color} stopOpacity={0}/>
                  </linearGradient>
                ))}
              </defs>
              <CartesianGrid strokeDasharray="3 3" stroke={G.border} />
              <XAxis dataKey="month" tick={{fontSize:10, fill:G.textLight}} axisLine={false} tickLine={false}/>
              <YAxis tick={{fontSize:10, fill:G.textLight}} axisLine={false} tickLine={false}/>
              <Tooltip contentStyle={{...glass, fontSize:12, border:`1px solid ${G.border}`}} />
              {BRANDS.slice(0,4).map(b=>(
                <Area key={b.id} type="monotone" dataKey={b.name} stroke={b.color} strokeWidth={2} fill={`url(#g_${b.id})`} dot={false}/>
              ))}
            </AreaChart>
          </ResponsiveContainer>
        </div>

        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14, fontWeight:600, color:G.navy, marginBottom:14 }}>Brand Performance Cards</div>
          <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr 1fr", gap:8 }}>
            {BRANDS.map(b => (
              <div key={b.id} onClick={()=>setActiveBrand(activeBrand===b.id?null:b.id)}
                style={{ padding:"10px 12px", borderRadius:12, border:`1.5px solid ${activeBrand===b.id ? b.color : G.border}`,
                  background: activeBrand===b.id ? `${b.color}10` : "rgba(255,255,255,0.5)", cursor:"pointer", transition:"all 0.15s" }}>
                <div style={{ width:28, height:28, borderRadius:8, background:b.color, display:"flex", alignItems:"center", justifyContent:"center",
                  color:"#fff", fontSize:9, fontWeight:700, marginBottom:6 }}>{b.short}</div>
                <div style={{ fontSize:11, fontWeight:600, color:G.navy }}>{b.name}</div>
                <div style={{ fontSize:13, fontWeight:700, color:b.color, marginTop:2 }}>{fmtM(rand(80,720))}</div>
                <div style={{ fontSize:10, color:G.green }}>+{rand(3,15)}% ↑</div>
              </div>
            ))}
          </div>
        </div>
      </div>

      <div style={{ ...glassCard }}>
        <div style={{ fontSize:14, fontWeight:600, color:G.navy, marginBottom:14 }}>AI Platform Activity — All Brands</div>
        <ResponsiveContainer width="100%" height={160}>
          <BarChart data={BRANDS.map(b=>({name:b.name, calls:rand(20,160), cost:rand(5,40)}))}>
            <CartesianGrid strokeDasharray="3 3" stroke={G.border}/>
            <XAxis dataKey="name" tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false}/>
            <YAxis tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false}/>
            <Tooltip contentStyle={{...glass,fontSize:12}} />
            <Bar dataKey="calls" name="AI Calls (k)" radius={[6,6,0,0]}
              fill="url(#barGrad)" />
            <defs>
              <linearGradient id="barGrad" x1="0" y1="0" x2="0" y2="1">
                <stop offset="0%" stopColor={G.blue} stopOpacity={0.9}/>
                <stop offset="100%" stopColor={G.green} stopOpacity={0.7}/>
              </linearGradient>
            </defs>
          </BarChart>
        </ResponsiveContainer>
      </div>
    </div>
  );
}

function DemandAI() {
  const [brand, setBrand] = useState("bunnings");
  const [horizon, setHorizon] = useState(90);
  const [approved, setApproved] = useState({});
  const forecastData = genForecastData(brand).slice(0, horizon);
  const bObj = BRANDS.find(b=>b.id===brand);

  return (
    <div>
      <TopBar title="Demand AI & Inventory" subtitle="AI-powered demand forecasting and stock management" />
      <div style={{ display:"flex", gap:12, marginBottom:"1.25rem" }}>
        <div style={{ ...glassCard, flex:1 }}>
          <div style={{ fontSize:12, color:G.textLight, marginBottom:6 }}>Select brand</div>
          <div style={{ display:"flex", gap:8, flexWrap:"wrap" }}>
            {BRANDS.map(b=>(
              <button key={b.id} onClick={()=>setBrand(b.id)}
                style={{ padding:"6px 14px", borderRadius:20, border:`1.5px solid ${brand===b.id?b.color:G.border}`,
                  background: brand===b.id?`${b.color}15`:"rgba(255,255,255,0.5)", color:brand===b.id?b.color:G.textMid,
                  fontSize:12, fontWeight:500, cursor:"pointer", transition:"all 0.15s" }}>{b.name}</button>
            ))}
          </div>
        </div>
        <div style={{ ...glassCard, display:"flex", gap:8, alignItems:"center" }}>
          {[30,60,90].map(h=>(
            <button key={h} onClick={()=>setHorizon(h)}
              style={{ padding:"6px 16px", borderRadius:20, border:`1px solid ${horizon===h?G.blue:G.border}`,
                background: horizon===h?G.blue:"transparent", color:horizon===h?"#fff":G.textMid,
                fontSize:12, cursor:"pointer", transition:"all 0.15s" }}>{h}d</button>
          ))}
        </div>
      </div>

      <div style={{ ...glassCard, marginBottom:"1.25rem" }}>
        <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", marginBottom:14 }}>
          <div>
            <div style={{ fontSize:14, fontWeight:600, color:G.navy }}>Demand Forecast — {bObj?.name}</div>
            <div style={{ fontSize:12, color:G.textLight }}>Next {horizon} days · AI confidence: {rand(88,96)}%</div>
          </div>
          <div style={{ display:"flex", gap:6 }}>
            <div style={{ fontSize:11, display:"flex", alignItems:"center", gap:4, color:G.blue }}><div style={{width:16,height:2,background:G.blue,borderRadius:2}}/> Actual</div>
            <div style={{ fontSize:11, display:"flex", alignItems:"center", gap:4, color:bObj?.color }}><div style={{width:16,height:2,background:bObj?.color,borderRadius:2,borderTop:"2px dashed "+bObj?.color}}/> Forecast</div>
          </div>
        </div>
        <ResponsiveContainer width="100%" height={220}>
          <AreaChart data={forecastData} margin={{top:0,right:0,left:-15,bottom:0}}>
            <defs>
              <linearGradient id="actualGrad" x1="0" y1="0" x2="0" y2="1">
                <stop offset="5%" stopColor={G.blue} stopOpacity={0.2}/><stop offset="95%" stopColor={G.blue} stopOpacity={0}/>
              </linearGradient>
              <linearGradient id="fcGrad" x1="0" y1="0" x2="0" y2="1">
                <stop offset="5%" stopColor={bObj?.color} stopOpacity={0.15}/><stop offset="95%" stopColor={bObj?.color} stopOpacity={0}/>
              </linearGradient>
            </defs>
            <CartesianGrid strokeDasharray="3 3" stroke={G.border}/>
            <XAxis dataKey="day" tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false} interval={9} tickFormatter={v=>`D${v}`}/>
            <YAxis tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false}/>
            <Tooltip contentStyle={{...glass,fontSize:12}} formatter={(v)=>[`$${Math.round(v)}k`]}/>
            <Area type="monotone" dataKey="actual" stroke={G.blue} strokeWidth={2} fill="url(#actualGrad)" dot={false} name="Actual"/>
            <Area type="monotone" dataKey="forecast" stroke={bObj?.color} strokeWidth={2} strokeDasharray="5 3" fill="url(#fcGrad)" dot={false} name="Forecast"/>
          </AreaChart>
        </ResponsiveContainer>
      </div>

      <div style={{ ...glassCard }}>
        <div style={{ fontSize:14, fontWeight:600, color:G.navy, marginBottom:4 }}>AI Inventory Alerts</div>
        <div style={{ fontSize:12, color:G.textLight, marginBottom:14 }}>AI-generated reorder recommendations requiring approval</div>
        <div style={{ display:"flex", flexDirection:"column", gap:8 }}>
          {INVENTORY_ALERTS.map((a,i) => {
            const urgColor = a.urgency==="critical"?"#E8221E":a.urgency==="high"?"#F59E0B":"#00A651";
            const isApproved = approved[i];
            return (
              <div key={i} style={{ display:"flex", alignItems:"center", gap:12, padding:"10px 14px", borderRadius:12,
                border:`1px solid ${isApproved?G.green+"44":urgColor+"33"}`, background:isApproved?"#E8F8F0":"rgba(255,255,255,0.5)" }}>
                <div style={{ width:8,height:8,borderRadius:"50%",background:urgColor,flexShrink:0}}/>
                <div style={{ flex:1, minWidth:0 }}>
                  <div style={{ fontSize:13, fontWeight:500, color:G.navy }}>{a.item}</div>
                  <div style={{ fontSize:11, color:G.textLight }}>{a.brand} · Current stock: {a.stock} units</div>
                </div>
                <div style={{ textAlign:"center", minWidth:80 }}>
                  <div style={{ fontSize:11, color:G.textLight }}>AI Recommends</div>
                  <div style={{ fontSize:14, fontWeight:700, color:G.blue }}>{a.aiQty} units</div>
                </div>
                <div style={{ display:"flex", gap:6 }}>
                  {!isApproved ? (
                    <>
                      <button onClick={()=>setApproved(p=>({...p,[i]:true}))}
                        style={{ padding:"5px 14px", borderRadius:8, border:`1px solid ${G.green}`, background:`${G.green}15`, color:G.green, fontSize:12, cursor:"pointer", fontWeight:500 }}>Approve</button>
                      <button style={{ padding:"5px 14px", borderRadius:8, border:`1px solid ${G.border}`, background:"transparent", color:G.textLight, fontSize:12, cursor:"pointer" }}>Adjust</button>
                    </>
                  ) : (
                    <div style={{ display:"flex", alignItems:"center", gap:5, color:G.green, fontSize:12, fontWeight:500 }}>
                      <CheckCircle size={14}/> Approved
                    </div>
                  )}
                </div>
              </div>
            );
          })}
        </div>
      </div>
    </div>
  );
}

function ModelOrchestration() {
  const [vendors] = useState(genVendorData());
  const [logs] = useState(Array.from({length:8},(_,i)=>({
    ts: `${String(14-i).padStart(2,"0")}:${String(rand(0,59)).padStart(2,"0")}:${String(rand(0,59)).padStart(2,"0")}`,
    brand: BRANDS[i%7].name, vendor:vendors[i%4].vendor.split(" ")[0], tokens:rand(800,4200), ms:rand(320,1800), status:"200 OK"
  })));

  return (
    <div>
      <TopBar title="Model Orchestration Hub" subtitle="AI vendor routing, cost tracking, and governance" />
      <div style={{ display:"grid", gridTemplateColumns:"repeat(4,1fr)", gap:12, marginBottom:"1.25rem" }}>
        {vendors.map((v,i)=>(
          <div key={i} style={{ ...glassCard }}>
            <div style={{ display:"flex", alignItems:"center", gap:8, marginBottom:10 }}>
              <div style={{ width:32,height:32,borderRadius:10,background:`${v.color}20`,display:"flex",alignItems:"center",justifyContent:"center" }}>
                <Globe size={15} color={v.color}/>
              </div>
              <div style={{ fontSize:12,fontWeight:600,color:G.navy,lineHeight:1.3 }}>{v.vendor}</div>
            </div>
            <div style={{ display:"flex", justifyContent:"space-between", marginBottom:6 }}>
              <div>
                <div style={{ fontSize:10,color:G.textLight }}>Monthly Cost</div>
                <div style={{ fontSize:16,fontWeight:700,color:G.navy }}>${v.cost.toLocaleString()}</div>
              </div>
              <div style={{ textAlign:"right" }}>
                <div style={{ fontSize:10,color:G.textLight }}>Perf Score</div>
                <div style={{ fontSize:16,fontWeight:700,color:v.color }}>{v.perf}%</div>
              </div>
            </div>
            <div style={{ fontSize:11,color:G.textLight }}>{(v.calls/1000).toFixed(0)}k calls · {v.brand}</div>
            <div style={{ marginTop:8,height:4,background:`${v.color}20`,borderRadius:4,overflow:"hidden" }}>
              <div style={{ width:`${v.perf}%`,height:"100%",background:v.color,borderRadius:4 }}/>
            </div>
          </div>
        ))}
      </div>

      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12, marginBottom:"1.25rem" }}>
        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:14 }}>Cost vs Performance</div>
          <ResponsiveContainer width="100%" height={200}>
            <BarChart data={vendors} margin={{top:0,right:0,left:-20,bottom:0}}>
              <CartesianGrid strokeDasharray="3 3" stroke={G.border}/>
              <XAxis dataKey="vendor" tick={{fontSize:9,fill:G.textLight}} axisLine={false} tickLine={false} tickFormatter={v=>v.split(" ")[0]}/>
              <YAxis tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false}/>
              <Tooltip contentStyle={{...glass,fontSize:12}}/>
              <Bar dataKey="cost" name="Cost ($)" radius={[6,6,0,0]} fill={G.blue} opacity={0.8}/>
              <Bar dataKey="perf" name="Perf (%)" radius={[6,6,0,0]} fill={G.green} opacity={0.8}/>
            </BarChart>
          </ResponsiveContainer>
        </div>

        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:4 }}>Routing Rules</div>
          <div style={{ fontSize:12,color:G.textLight,marginBottom:12 }}>Active routing logic per use case</div>
          {[
            { useCase:"Customer chat", primary:"Anthropic Claude", fallback:"OpenAI GPT-4o", active:true },
            { useCase:"Demand forecasting", primary:"Internal Model", fallback:"Google Vertex", active:true },
            { useCase:"Product copy gen", primary:"OpenAI GPT-4o", fallback:"Anthropic Claude", active:true },
            { useCase:"Health AI triage", primary:"Anthropic Claude", fallback:"Internal Model", active:false },
          ].map((r,i)=>(
            <div key={i} style={{ display:"flex",alignItems:"center",gap:10,padding:"8px 10px",borderRadius:10,background:"rgba(255,255,255,0.5)",marginBottom:6 }}>
              <div style={{ width:8,height:8,borderRadius:"50%",background:r.active?G.green:"#ccc",flexShrink:0}}/>
              <div style={{ flex:1 }}>
                <div style={{ fontSize:12,fontWeight:500,color:G.navy }}>{r.useCase}</div>
                <div style={{ fontSize:10,color:G.textLight }}>{r.primary} → {r.fallback}</div>
              </div>
              <div style={{ fontSize:10,padding:"2px 8px",borderRadius:10,background:r.active?"#E8F8F0":"#F5F5F5",color:r.active?G.green:"#999" }}>
                {r.active?"Active":"Inactive"}
              </div>
            </div>
          ))}
        </div>
      </div>

      <div style={{ ...glassCard }}>
        <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:14 }}>AI Usage Log — Live</div>
        <div style={{ fontFamily:"'SF Mono', monospace" }}>
          <div style={{ display:"grid",gridTemplateColumns:"90px 100px 120px 80px 60px 70px",gap:8,padding:"0 0 8px",borderBottom:`1px solid ${G.border}`,marginBottom:6 }}>
            {["Timestamp","Brand","Vendor","Tokens","Latency","Status"].map(h=>(
              <div key={h} style={{ fontSize:10,fontWeight:600,color:G.textLight,textTransform:"uppercase",letterSpacing:"0.06em" }}>{h}</div>
            ))}
          </div>
          {logs.map((l,i)=>(
            <div key={i} style={{ display:"grid",gridTemplateColumns:"90px 100px 120px 80px 60px 70px",gap:8,padding:"6px 0",borderBottom:`1px solid ${G.border}22` }}>
              <div style={{ fontSize:11,color:G.textLight }}>{l.ts}</div>
              <div style={{ fontSize:11,color:G.navy,fontWeight:500 }}>{l.brand}</div>
              <div style={{ fontSize:11,color:G.textMid }}>{l.vendor}</div>
              <div style={{ fontSize:11,color:G.textMid }}>{l.tokens.toLocaleString()}</div>
              <div style={{ fontSize:11,color:l.ms>1200?"#F59E0B":G.textMid }}>{l.ms}ms</div>
              <div style={{ fontSize:10,color:G.green,fontWeight:500 }}>{l.status}</div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

function ProductDesignAI() {
  const [brand, setBrand] = useState("kmart");
  const [brief, setBrief] = useState("");
  const [loading, setLoading] = useState(false);
  const [concepts, setConcepts] = useState(null);
  const [saved, setSaved] = useState([]);
  const bObj = BRANDS.find(b=>b.id===brand);

  const generate = async () => {
    if (!brief.trim()) return;
    setLoading(true); setConcepts(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method:"POST", headers:{"Content-Type":"application/json"},
        body: JSON.stringify({
          model:"claude-sonnet-4-20250514", max_tokens:1000,
          system:`You are the Wesfarmers AI Product Design Studio for ${bObj?.name}. Generate exactly 3 distinct product concept variants based on a brief. Respond ONLY with valid JSON, no markdown, no backticks:
{"concepts":[{"name":"product name (max 6 words)","tagline":"compelling tagline (max 10 words)","description":"2 sentence product description","targetCustomer":"target segment","pricePoint":"price range","keyFeature":"standout feature","emoji":"one relevant emoji"}]}`,
          messages:[{role:"user",content:`${bObj?.name} product brief: ${brief}`}]
        })
      });
      const data = await res.json();
      const raw = data.content?.map(b=>b.text||"").join("") || "";
      const parsed = JSON.parse(raw.replace(/```json|```/g,"").trim());
      setConcepts(parsed.concepts);
    } catch(e) {
      setConcepts([{name:"Generation error",tagline:"Please try again",description:e.message,targetCustomer:"-",pricePoint:"-",keyFeature:"-",emoji:"⚠️"}]);
    }
    setLoading(false);
  };

  return (
    <div>
      <TopBar title="AI Product Design Studio" subtitle="Generate and iterate product concepts with generative AI" />
      <div style={{ display:"grid", gridTemplateColumns:"1fr 2fr", gap:12, marginBottom:"1.25rem" }}>
        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:14 }}>Design Brief</div>
          <div style={{ marginBottom:10 }}>
            <div style={{ fontSize:12,color:G.textLight,marginBottom:6 }}>Brand</div>
            <div style={{ display:"flex",gap:6,flexWrap:"wrap" }}>
              {BRANDS.slice(0,5).map(b=>(
                <button key={b.id} onClick={()=>setBrand(b.id)}
                  style={{ padding:"5px 12px",borderRadius:20,border:`1.5px solid ${brand===b.id?b.color:G.border}`,
                    background:brand===b.id?`${b.color}15`:"transparent",color:brand===b.id?b.color:G.textMid,fontSize:11,cursor:"pointer",transition:"all 0.15s" }}>{b.name}</button>
              ))}
            </div>
          </div>
          <div style={{ marginBottom:12 }}>
            <div style={{ fontSize:12,color:G.textLight,marginBottom:6 }}>Product brief</div>
            <textarea value={brief} onChange={e=>setBrief(e.target.value)}
              placeholder={`Describe the product concept for ${bObj?.name}...\n\ne.g. "Affordable ergonomic desk chair for remote workers under $120"`}
              style={{ width:"100%",boxSizing:"border-box",minHeight:120,padding:"10px 12px",borderRadius:12,border:`1px solid ${G.border}`,
                background:"rgba(255,255,255,0.6)",fontSize:13,color:G.navy,resize:"vertical",outline:"none",lineHeight:1.5 }}/>
          </div>
          <button onClick={generate} disabled={loading||!brief.trim()}
            style={{ width:"100%",padding:"10px",borderRadius:12,border:"none",
              background: loading||!brief.trim() ? "#ccc" : `linear-gradient(135deg,${G.blue},${G.green})`,
              color:"#fff",fontSize:14,fontWeight:600,cursor:loading||!brief.trim()?"not-allowed":"pointer",transition:"all 0.2s" }}>
            {loading ? "Generating concepts..." : "✦ Generate with AI"}
          </button>
          {saved.length > 0 && (
            <div style={{ marginTop:14 }}>
              <div style={{ fontSize:12,color:G.textLight,marginBottom:6 }}>Saved to pipeline ({saved.length})</div>
              {saved.map((s,i)=>(
                <div key={i} style={{ fontSize:12,color:G.textMid,padding:"4px 0",borderBottom:`1px solid ${G.border}` }}>{s}</div>
              ))}
            </div>
          )}
        </div>

        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:14 }}>Generated Concepts</div>
          {!concepts && !loading && (
            <div style={{ display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",height:280,color:G.textLight }}>
              <Palette size={36} style={{opacity:0.3,marginBottom:12}}/>
              <div style={{ fontSize:14 }}>Enter a brief and click Generate</div>
              <div style={{ fontSize:12,marginTop:4 }}>AI will generate 3 distinct product concepts</div>
            </div>
          )}
          {loading && (
            <div style={{ display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",height:280,color:G.textLight }}>
              <div style={{ width:40,height:40,border:`3px solid ${G.border}`,borderTopColor:G.blue,borderRadius:"50%",animation:"spin 0.8s linear infinite",marginBottom:12}}/>
              <div style={{ fontSize:14 }}>Generating concepts with Claude AI...</div>
            </div>
          )}
          {concepts && (
            <div style={{ display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:12 }}>
              {concepts.map((c,i)=>(
                <div key={i} style={{ border:`1.5px solid ${G.border}`,borderRadius:16,padding:"14px",background:"rgba(255,255,255,0.6)" }}>
                  <div style={{ fontSize:32,marginBottom:8,textAlign:"center" }}>{c.emoji}</div>
                  <div style={{ fontSize:13,fontWeight:700,color:G.navy,marginBottom:4,lineHeight:1.3 }}>{c.name}</div>
                  <div style={{ fontSize:11,color:bObj?.color,fontWeight:500,marginBottom:8 }}>{c.tagline}</div>
                  <div style={{ fontSize:11,color:G.textMid,lineHeight:1.5,marginBottom:8 }}>{c.description}</div>
                  <div style={{ display:"flex",flexDirection:"column",gap:4,marginBottom:10 }}>
                    <div style={{ fontSize:10,color:G.textLight }}>Target: <span style={{color:G.textMid}}>{c.targetCustomer}</span></div>
                    <div style={{ fontSize:10,color:G.textLight }}>Price: <span style={{color:G.navy,fontWeight:600}}>{c.pricePoint}</span></div>
                    <div style={{ fontSize:10,color:G.textLight }}>Key: <span style={{color:G.textMid}}>{c.keyFeature}</span></div>
                  </div>
                  <button onClick={()=>setSaved(p=>[...p,c.name])}
                    style={{ width:"100%",padding:"6px",borderRadius:8,border:`1px solid ${G.green}`,background:`${G.green}15`,
                      color:G.green,fontSize:11,cursor:"pointer",fontWeight:500 }}>Save to Pipeline</button>
                </div>
              ))}
            </div>
          )}
        </div>
      </div>
      <style>{`@keyframes spin{to{transform:rotate(360deg)}}`}</style>
    </div>
  );
}

function ShoppingAgent() {
  const [messages, setMessages] = useState([
    { role:"ai", text:"Hi Sarah! 👋 I'm your OneIntelligence shopping assistant. Based on your recent purchases at Officeworks and Kmart, I can see you're a home office enthusiast. What are you looking for today?", time:"Now" }
  ]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [showProducts, setShowProducts] = useState(false);
  const messagesEndRef = useRef(null);

  useEffect(()=>{ messagesEndRef.current?.scrollIntoView({behavior:"smooth"}); }, [messages]);

  const send = async () => {
    if (!input.trim() || loading) return;
    const userMsg = input.trim();
    setInput("");
    setMessages(p=>[...p,{role:"user",text:userMsg,time:"Now"}]);
    setLoading(true);

    try {
      const res = await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-20250514",max_tokens:400,
          system:`You are the OneIntelligence AI shopping assistant for Wesfarmers — you know the customer's cross-brand purchase history across Bunnings, Kmart, Target, Officeworks, Catch, and Priceline. You are helpful, warm, and specific. Keep responses under 80 words. Reference specific Wesfarmers brands when recommending. If the customer asks about home office, electronics, DIY, furniture, or health, mention relevant Wesfarmers brands. End with a natural follow-up question.`,
          messages:[...messages.filter(m=>m.role!=="products").map(m=>({role:m.role==="ai"?"assistant":"user",content:m.text})),{role:"user",content:userMsg}]
        })
      });
      const data = await res.json();
      const text = data.content?.map(b=>b.text||"").join("") || "Let me help you with that!";
      setMessages(p=>[...p,{role:"ai",text,time:"Now"}]);
      if (userMsg.toLowerCase().includes("home office") || userMsg.toLowerCase().includes("$500") || userMsg.toLowerCase().includes("desk") || userMsg.toLowerCase().includes("setup")) {
        setTimeout(()=>{ setMessages(p=>[...p,{role:"products"}]); setShowProducts(true); },800);
      }
    } catch(e) {
      setMessages(p=>[...p,{role:"ai",text:"I'm having trouble connecting. Please try again!",time:"Now"}]);
    }
    setLoading(false);
  };

  return (
    <div style={{ maxWidth:700, margin:"0 auto" }}>
      <TopBar title="AI Shopping Agent" subtitle="Cross-brand conversational commerce · OnePass personalised" />
      <div style={{ ...glassCard, display:"flex", flexDirection:"column", height:520 }}>
        <div style={{ display:"flex",alignItems:"center",gap:10,paddingBottom:14,borderBottom:`1px solid ${G.border}`,marginBottom:14 }}>
          <div style={{ width:38,height:38,borderRadius:"50%",background:`linear-gradient(135deg,${G.blue},${G.green})`,display:"flex",alignItems:"center",justifyContent:"center" }}>
            <Bot size={18} color="#fff"/>
          </div>
          <div>
            <div style={{ fontSize:14,fontWeight:600,color:G.navy }}>OneIntelligence Assistant</div>
            <div style={{ fontSize:11,color:G.green,display:"flex",alignItems:"center",gap:4 }}>
              <div style={{ width:6,height:6,borderRadius:"50%",background:G.green }}/> Online · Knows your history across 7 brands
            </div>
          </div>
          <div style={{ marginLeft:"auto",fontSize:11,padding:"3px 10px",borderRadius:20,background:"#E8F8F0",color:G.green,border:`1px solid ${G.green}33` }}>
            Sarah T. · OnePass Gold
          </div>
        </div>

        <div style={{ flex:1,overflowY:"auto",display:"flex",flexDirection:"column",gap:12,paddingRight:4 }}>
          {messages.map((m,i)=>{
            if(m.role==="products") return (
              <div key={i} style={{ display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:8 }}>
                {PRODUCTS.slice(0,6).map((p,j)=>(
                  <div key={j} style={{ border:`1px solid ${G.border}`,borderRadius:12,padding:"10px",background:"rgba(255,255,255,0.7)",cursor:"pointer",transition:"all 0.15s" }}>
                    <div style={{ fontSize:24,textAlign:"center",marginBottom:6 }}>{p.img}</div>
                    <div style={{ fontSize:10,fontWeight:600,color:BRANDS.find(b=>b.name===p.brand)?.color||G.blue }}>{p.brand}</div>
                    <div style={{ fontSize:11,fontWeight:600,color:G.navy,lineHeight:1.3,marginBottom:4 }}>{p.name}</div>
                    <div style={{ display:"flex",alignItems:"center",justifyContent:"space-between" }}>
                      <div style={{ fontSize:13,fontWeight:700,color:G.navy }}>${p.price}</div>
                      <div style={{ fontSize:10,color:G.green }}>{p.match}% match</div>
                    </div>
                  </div>
                ))}
              </div>
            );
            return (
              <div key={i} style={{ display:"flex",justifyContent:m.role==="user"?"flex-end":"flex-start",gap:8,alignItems:"flex-end" }}>
                {m.role==="ai" && (
                  <div style={{ width:28,height:28,borderRadius:"50%",background:`linear-gradient(135deg,${G.blue},${G.green})`,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0 }}>
                    <Bot size={13} color="#fff"/>
                  </div>
                )}
                <div style={{ maxWidth:"72%",padding:"10px 14px",borderRadius:m.role==="user"?"18px 18px 4px 18px":"18px 18px 18px 4px",
                  background:m.role==="user"?`linear-gradient(135deg,${G.blue},${G.blue}dd)`:"rgba(255,255,255,0.8)",
                  color:m.role==="user"?"#fff":G.navy,fontSize:13,lineHeight:1.5,
                  border:m.role==="user"?"none":`1px solid ${G.border}`,boxShadow:G.shadowSm }}>
                  {m.text}
                </div>
              </div>
            );
          })}
          {loading && (
            <div style={{ display:"flex",gap:8,alignItems:"flex-end" }}>
              <div style={{ width:28,height:28,borderRadius:"50%",background:`linear-gradient(135deg,${G.blue},${G.green})`,display:"flex",alignItems:"center",justifyContent:"center" }}>
                <Bot size={13} color="#fff"/>
              </div>
              <div style={{ padding:"10px 14px",borderRadius:"18px 18px 18px 4px",background:"rgba(255,255,255,0.8)",border:`1px solid ${G.border}` }}>
                <div style={{ display:"flex",gap:4,alignItems:"center" }}>
                  {[0,1,2].map(i=><div key={i} style={{ width:6,height:6,borderRadius:"50%",background:G.blue,animation:`bounce 1s ${i*0.2}s infinite` }}/>)}
                </div>
              </div>
            </div>
          )}
          <div ref={messagesEndRef}/>
        </div>

        <div style={{ borderTop:`1px solid ${G.border}`,paddingTop:12,marginTop:8,display:"flex",gap:8,alignItems:"center" }}>
          <input value={input} onChange={e=>setInput(e.target.value)} onKeyDown={e=>e.key==="Enter"&&send()}
            placeholder="Ask about products, compare across brands, get recommendations..."
            style={{ flex:1,padding:"10px 14px",borderRadius:12,border:`1px solid ${G.border}`,background:"rgba(255,255,255,0.7)",fontSize:13,color:G.navy,outline:"none" }}/>
          <button onClick={send} disabled={loading||!input.trim()}
            style={{ width:40,height:40,borderRadius:12,border:"none",background:input.trim()?`linear-gradient(135deg,${G.blue},${G.green})`:"#e0e0e0",
              display:"flex",alignItems:"center",justifyContent:"center",cursor:input.trim()?"pointer":"not-allowed",transition:"all 0.2s",flexShrink:0 }}>
            <Send size={16} color={input.trim()?"#fff":"#aaa"}/>
          </button>
        </div>
      </div>
      <style>{`@keyframes bounce{0%,60%,100%{transform:translateY(0)}30%{transform:translateY(-5px)}}`}</style>
    </div>
  );
}

function Personalisation() {
  const [activeSegment, setActiveSegment] = useState(0);
  return (
    <div>
      <TopBar title="Cross-Brand Personalisation" subtitle="AI-generated customer segments and personalisation rules" />
      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:12, marginBottom:"1.25rem" }}>
        {PERSONAS.map((p,i)=>(
          <div key={i} onClick={()=>setActiveSegment(i)}
            style={{ ...glassCard, cursor:"pointer", border:`1.5px solid ${activeSegment===i?G.blue:G.border}`,
              background:activeSegment===i?`${G.blue}08`:G.glass, transition:"all 0.15s" }}>
            <div style={{ display:"flex",alignItems:"flex-start",justifyContent:"space-between",marginBottom:10 }}>
              <div>
                <div style={{ fontSize:14,fontWeight:700,color:G.navy }}>{p.name}</div>
                <div style={{ fontSize:11,color:G.textLight,marginTop:2 }}>{p.brands}</div>
              </div>
              <div style={{ fontSize:18,fontWeight:800,color:G.green }}>{p.lift}</div>
            </div>
            <div style={{ fontSize:12,color:G.textMid,marginBottom:10 }}>{p.desc}</div>
            <div style={{ display:"flex",alignItems:"center",justifyContent:"space-between" }}>
              <div style={{ fontSize:13,fontWeight:700,color:G.navy }}>{p.size} customers</div>
              <div style={{ fontSize:11,padding:"3px 10px",borderRadius:20,background:"#E8F8F0",color:G.green }}>Active</div>
            </div>
          </div>
        ))}
      </div>

      <div style={{ display:"grid", gridTemplateColumns:"2fr 1fr", gap:12 }}>
        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:14 }}>Personalisation Lift — {PERSONAS[activeSegment].name}</div>
          <ResponsiveContainer width="100%" height={220}>
            <AreaChart data={Array.from({length:12},(_,i)=>({
              month:["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"][i],
              baseline:rand(60,80), personalised:rand(85,110)
            }))}>
              <defs>
                <linearGradient id="baseGrad" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="5%" stopColor="#94A3B8" stopOpacity={0.2}/><stop offset="95%" stopColor="#94A3B8" stopOpacity={0}/>
                </linearGradient>
                <linearGradient id="persGrad" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="5%" stopColor={G.green} stopOpacity={0.25}/><stop offset="95%" stopColor={G.green} stopOpacity={0}/>
                </linearGradient>
              </defs>
              <CartesianGrid strokeDasharray="3 3" stroke={G.border}/>
              <XAxis dataKey="month" tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false}/>
              <YAxis tick={{fontSize:10,fill:G.textLight}} axisLine={false} tickLine={false}/>
              <Tooltip contentStyle={{...glass,fontSize:12}}/>
              <Area type="monotone" dataKey="baseline" stroke="#94A3B8" strokeWidth={2} fill="url(#baseGrad)" name="Baseline"/>
              <Area type="monotone" dataKey="personalised" stroke={G.green} strokeWidth={2.5} fill="url(#persGrad)" name="AI Personalised"/>
            </AreaChart>
          </ResponsiveContainer>
        </div>

        <div style={{ ...glassCard }}>
          <div style={{ fontSize:14,fontWeight:600,color:G.navy,marginBottom:12 }}>Active Rules</div>
          {[
            { rule:"Cross-sell Kmart after Bunnings purchase", status:"active", lift:"+18%" },
            { rule:"Health AI upsell post-Priceline visit", status:"active", lift:"+24%" },
            { rule:"Office bundle for WFH segment", status:"active", lift:"+31%" },
            { rule:"Seasonal promotion suppression", status:"paused", lift:"-" },
          ].map((r,i)=>(
            <div key={i} style={{ padding:"8px 10px",borderRadius:10,background:"rgba(255,255,255,0.5)",marginBottom:6,
              border:`1px solid ${r.status==="active"?G.green+"33":G.border}` }}>
              <div style={{ display:"flex",alignItems:"center",justifyContent:"space-between" }}>
                <div style={{ fontSize:11,color:G.navy,fontWeight:500,flex:1,marginRight:8,lineHeight:1.4 }}>{r.rule}</div>
                <div style={{ fontSize:11,fontWeight:700,color:r.status==="active"?G.green:"#aaa",flexShrink:0 }}>{r.lift}</div>
              </div>
            </div>
          ))}
          <div style={{ marginTop:12,padding:"10px",background:"#E8F8F0",borderRadius:12,border:`1px solid ${G.green}33` }}>
            <div style={{ fontSize:11,color:G.green,fontWeight:600 }}>AI Governance</div>
            <div style={{ fontSize:11,color:G.textMid,marginTop:2 }}>All rules reviewed by AI Ethics CoE · Last audit: 2 days ago</div>
          </div>
        </div>
      </div>
    </div>
  );
}

// ─── ROOT APP ────────────────────────────────────────────────────────────────

export default function App() {
  const [page, setPage] = useState("command");
  const [role, setRole] = useState("exec");

  const pages = { command: CommandCentre, demand: DemandAI, orchestration: ModelOrchestration, product: ProductDesignAI, chat: ShoppingAgent, personalisation: Personalisation };
  const PageComponent = pages[page] || CommandCentre;

  return (
    <div style={{ display:"flex", minHeight:"100vh", background:G.bg, fontFamily:"system-ui, -apple-system, sans-serif" }}>
      <Sidebar page={page} setPage={setPage} role={role} setRole={setRole}/>
      <main style={{ flex:1, overflowY:"auto", padding:"2rem 2rem 3rem" }}>
        <PageComponent />
      </main>
    </div>
  );
}
