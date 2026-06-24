# C3-RowRace

import { useState, useEffect, useRef } from "react";

// ─── Theme (projenin tema dosyasından birebir) ────────────────────────────────
const C = {
  bg: "#0A0E1A",
  surface: "#111827",
  surfaceLight: "#1C2537",
  border: "#1E3A5F",
  primary: "#00E5FF",
  primaryDim: "#0099AA",
  primaryGlow: "rgba(0,229,255,0.15)",
  accent: "#F97316",
  accentDim: "rgba(249,115,22,0.2)",
  success: "#22C55E",
  warning: "#F59E0B",
  danger: "#EF4444",
  ghost: "#A855F7",
  textPrimary: "#F1F5F9",
  textSecondary: "#94A3B8",
  textMuted: "#475569",
  raceColors: ["#00E5FF","#F97316","#22C55E","#F59E0B","#EC4899","#A855F7"],
};

// ─── Yardımcı formatlar ───────────────────────────────────────────────────────
const formatTime = (ms) => {
  const s = Math.floor(ms / 1000);
  const m = Math.floor(s / 60);
  const sec = s % 60;
  const ds = Math.floor((ms % 1000) / 100);
  return `${String(m).padStart(2,"0")}:${String(sec).padStart(2,"0")}.${ds}`;
};
const formatSplit = (ms500) => {
  if (!ms500 || ms500 === 0) return "--:--";
  const s = Math.floor(ms500 / 1000);
  const m = Math.floor(s / 60);
  const sec = s % 60;
  return `${m}:${String(sec).padStart(2,"0")}`;
};

// ─── Simüle edilmiş cihaz verisi ─────────────────────────────────────────────
const useSimulatedRower = (active) => {
  const [data, setData] = useState({ distance: 0, spm: 0, watts: 0, split: 0, hr: 0 });
  const ref = useRef({ distance: 0, t: 0 });

  useEffect(() => {
    if (!active) { setData({ distance: 0, spm: 0, watts: 0, split: 0, hr: 0 }); return; }
    const id = setInterval(() => {
      ref.current.t += 0.1;
      const spm = 22 + Math.sin(ref.current.t * 0.3) * 3 + (Math.random() - 0.5);
      const watts = 180 + Math.sin(ref.current.t * 0.2) * 40 + (Math.random() - 0.5) * 20;
      const split = 110000 / watts; // ms per 500m approx
      ref.current.distance += (500 / (split / 100)) / 10;
      setData({
        distance: Math.floor(ref.current.distance),
        spm: Math.round(spm),
        watts: Math.round(watts),
        split: Math.round(split),
        hr: Math.round(148 + Math.sin(ref.current.t * 0.1) * 8),
      });
    }, 100);
    return () => clearInterval(id);
  }, [active]);

  const reset = () => { ref.current = { distance: 0, t: 0 }; };
  return { data, reset };
};

// ─── SCREENS ──────────────────────────────────────────────────────────────────

// HOME
function HomeScreen({ navigate, connected, setConnected }) {
  const [showBle, setShowBle] = useState(false);
  const [scanning, setScanning] = useState(false);
  const fakeDevices = ["PM5 Concept2 #A3F1", "PM5 Concept2 #B820"];

  const startScan = () => { setShowBle(true); setScanning(true); setTimeout(() => setScanning(false), 1800); };

  const menuItems = [
    { icon: "🏁", title: "Canlı Yarış", sub: "Diğer kürekçilerle gerçek zamanlı yar", color: C.primary, screen: "lobby", needsConn: true },
    { icon: "📊", title: "Antrenman", sub: "Tekil performans takibi", color: C.accent, screen: "training", needsConn: true },
    { icon: "👻", title: "Hayalet Rakip", sub: "Kendi rekoruna karşı yar", color: C.ghost, screen: "ghost", needsConn: true },
    { icon: "🏆", title: "Liderlik Tablosu", sub: "En iyi sonuçlar", color: C.warning, screen: "leaderboard", needsConn: false },
  ];

  return (
    <div style={s.screen}>
      {/* Header */}
      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start", padding:"20px 20px 12px" }}>
        <div>
          <div style={{ fontSize:28, fontWeight:900, color:C.primary, letterSpacing:4 }}>ERG RACE</div>
          <div style={{ fontSize:12, color:C.textSecondary, letterSpacing:1, marginTop:2 }}>Kürek. Yar. Kazan.</div>
        </div>
        <button
          onClick={connected ? () => setConnected(false) : startScan}
          style={{
            border: `1px solid ${connected ? C.success : C.textMuted}`,
            background:"transparent", borderRadius:999, padding:"6px 14px", cursor:"pointer",
          }}
        >
          <span style={{ fontSize:11, color: connected ? C.success : C.textMuted, fontWeight:600 }}>
            {connected ? "● PM5 #A3F1" : "○ Bağlı değil"}
          </span>
        </button>
      </div>

      {/* Menu */}
      <div style={{ padding:"0 16px", display:"flex", flexDirection:"column", gap:10 }}>
        {menuItems.map((m) => {
          const disabled = m.needsConn && !connected;
          return (
            <button
              key={m.title}
              onClick={() => !disabled && navigate(m.screen)}
              disabled={disabled}
              style={{
                display:"flex", alignItems:"center", background:C.surface,
                border:`1px solid ${C.border}`, borderRadius:16, padding:14,
                opacity: disabled ? 0.45 : 1, cursor: disabled ? "not-allowed" : "pointer",
                transition:"opacity 0.2s",
              }}
            >
              <div style={{ width:48, height:48, borderRadius:12, background:m.color+"22",
                display:"flex", alignItems:"center", justifyContent:"center", fontSize:22, flexShrink:0 }}>
                {m.icon}
              </div>
              <div style={{ flex:1, textAlign:"left", marginLeft:14 }}>
                <div style={{ color: disabled ? C.textMuted : C.textPrimary, fontWeight:600, fontSize:15 }}>{m.title}</div>
                <div style={{ color:C.textSecondary, fontSize:12, marginTop:2 }}>{m.sub}</div>
              </div>
              <span style={{ color: disabled ? C.textMuted : m.color, fontSize:24, fontWeight:700 }}>›</span>
            </button>
          );
        })}
      </div>

      {!connected && (
        <div style={{ padding:"20px 16px", textAlign:"center" }}>
          <div style={{ color:C.textMuted, fontSize:12, marginBottom:12 }}>Özellikler için önce PM5 cihazınıza bağlanın</div>
          <button onClick={startScan} style={{ background:C.primary, color:C.bg, border:"none", borderRadius:999, padding:"10px 28px", fontWeight:700, fontSize:14, cursor:"pointer" }}>
            Cihaz Bul
          </button>
        </div>
      )}

      {/* BLE Modal */}
      {showBle && (
        <div style={s.modalOverlay}>
          <div style={s.modalSheet}>
            <div style={s.modalTitle}>Cihaz Seç</div>
            {scanning && (
              <div style={{ display:"flex", alignItems:"center", gap:8, marginBottom:12 }}>
                <div style={s.spinner} />
                <span style={{ color:C.textSecondary, fontSize:13 }}>FTMS cihazları taranıyor...</span>
              </div>
            )}
            {!scanning && fakeDevices.map(d => (
              <button key={d} onClick={() => { setConnected(true); setShowBle(false); }}
                style={{ display:"block", width:"100%", textAlign:"left", background:"transparent",
                  border:"none", borderBottom:`1px solid ${C.border}`, padding:"14px 0", cursor:"pointer" }}>
                <div style={{ color:C.textPrimary, fontWeight:500, fontSize:15 }}>{d}</div>
                <div style={{ color:C.textMuted, fontSize:11, marginTop:2 }}>uuid:00{d.slice(-4)}</div>
              </button>
            ))}
            <button onClick={() => setShowBle(false)}
              style={{ marginTop:14, width:"100%", background:"transparent", border:"none", color:C.danger, fontSize:15, padding:12, cursor:"pointer" }}>
              İptal
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

// TRAINING
function TrainingScreen({ navigate }) {
  const distances = [500, 1000, 2000, 5000];
  const [target, setTarget] = useState(2000);
  const [running, setRunning] = useState(false);
  const [elapsed, setElapsed] = useState(0);
  const [showSummary, setShowSummary] = useState(false);
  const [showDistPicker, setShowDistPicker] = useState(false);
  const timerRef = useRef(null);
  const startRef = useRef(0);
  const { data, reset } = useSimulatedRower(running);

  const start = () => {
    reset();
    setElapsed(0);
    setRunning(true);
    startRef.current = Date.now();
    timerRef.current = setInterval(() => setElapsed(Date.now() - startRef.current), 100);
  };
  const stop = () => {
    clearInterval(timerRef.current);
    setRunning(false);
    setShowSummary(true);
  };
  const progress = Math.min(data.distance / target, 1);

  return (
    <div style={s.screen}>
      <div style={s.header}>
        <button onClick={() => navigate("home")} style={s.backBtn}>‹ Geri</button>
        <div style={s.headerTitle}>Antrenman</div>
        <button onClick={() => !running && setShowDistPicker(true)}
          style={{ background:C.surfaceLight, border:`1px solid ${C.border}`, borderRadius:999,
            padding:"6px 14px", color:C.primary, fontSize:12, fontWeight:600, cursor:"pointer" }}>
          {target}m
        </button>
      </div>

      {/* Main metric */}
      <div style={{ textAlign:"center", padding:"24px 20px 12px" }}>
        <div style={{ fontSize:72, fontWeight:900, color:C.primary, fontFamily:"monospace", lineHeight:1 }}>
          {formatTime(elapsed)}
        </div>
        <div style={{ color:C.textMuted, fontSize:12, marginTop:4, letterSpacing:2 }}>SÜRE</div>
      </div>

      {/* Progress bar */}
      <div style={{ padding:"0 20px", marginBottom:16 }}>
        <div style={{ display:"flex", justifyContent:"space-between", marginBottom:6 }}>
          <span style={{ color:C.textSecondary, fontSize:12 }}>Mesafe</span>
          <span style={{ color:C.textPrimary, fontSize:12, fontWeight:600 }}>{data.distance}m / {target}m</span>
        </div>
        <div style={{ height:8, background:C.surfaceLight, borderRadius:999, overflow:"hidden" }}>
          <div style={{ height:"100%", width:`${progress*100}%`, background:`linear-gradient(90deg, ${C.primaryDim}, ${C.primary})`,
            borderRadius:999, transition:"width 0.3s", boxShadow:`0 0 10px ${C.primary}66` }} />
        </div>
      </div>

      {/* Stats grid */}
      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:10, padding:"0 16px" }}>
        {[
          { label:"SPL / dk", value: running ? data.spm : "--", color: C.primary },
          { label:"Güç (W)", value: running ? data.watts : "--", color: C.accent },
          { label:"500m Split", value: running ? formatSplit(data.split) : "--:--", color: C.warning },
          { label:"Kalp Atışı", value: running ? data.hr : "--", color: C.danger },
        ].map(stat => (
          <div key={stat.label} style={{ background:C.surface, border:`1px solid ${C.border}`, borderRadius:14, padding:16 }}>
            <div style={{ color:C.textMuted, fontSize:11, letterSpacing:1 }}>{stat.label}</div>
            <div style={{ color:stat.color, fontSize:32, fontWeight:900, fontFamily:"monospace", marginTop:4 }}>{stat.value}</div>
          </div>
        ))}
      </div>

      {/* Start/Stop button */}
      <div style={{ padding:20 }}>
        {!running ? (
          <button onClick={start} style={{ width:"100%", background:C.primary, border:"none",
            borderRadius:999, padding:"16px 0", color:C.bg, fontWeight:700, fontSize:16, cursor:"pointer",
            boxShadow:`0 0 24px ${C.primary}44` }}>
            BAŞLAT
          </button>
        ) : (
          <button onClick={stop} style={{ width:"100%", background:"transparent", border:`2px solid ${C.danger}`,
            borderRadius:999, padding:"16px 0", color:C.danger, fontWeight:700, fontSize:16, cursor:"pointer" }}>
            DURDUR
          </button>
        )}
      </div>

      {/* Distance picker */}
      {showDistPicker && (
        <div style={s.modalOverlay}>
          <div style={s.modalSheet}>
            <div style={s.modalTitle}>Hedef Mesafe</div>
            {distances.map(d => (
              <button key={d} onClick={() => { setTarget(d); setShowDistPicker(false); }}
                style={{ display:"block", width:"100%", textAlign:"left", background: d === target ? C.primaryGlow : "transparent",
                  border:"none", borderBottom:`1px solid ${C.border}`, padding:"16px 8px", cursor:"pointer",
                  color: d === target ? C.primary : C.textPrimary, fontSize:16, fontWeight: d===target?700:400 }}>
                {d}m
              </button>
            ))}
          </div>
        </div>
      )}

      {/* Summary modal */}
      {showSummary && (
        <div style={s.modalOverlay}>
          <div style={s.modalSheet}>
            <div style={{ textAlign:"center", paddingBottom:16 }}>
              <div style={{ fontSize:40 }}>🏅</div>
              <div style={s.modalTitle}>Antrenman Tamamlandı</div>
            </div>
            {[
              ["Süre", formatTime(elapsed)],
              ["Mesafe", `${data.distance} m`],
              ["Ort. Split", formatSplit(data.split)],
              ["Ort. Güç", `${data.watts} W`],
              ["SPL Ortalaması", `${data.spm} /dk`],
            ].map(([k,v]) => (
              <div key={k} style={{ display:"flex", justifyContent:"space-between",
                borderBottom:`1px solid ${C.border}`, padding:"12px 4px" }}>
                <span style={{ color:C.textSecondary, fontSize:14 }}>{k}</span>
                <span style={{ color:C.textPrimary, fontWeight:700, fontFamily:"monospace" }}>{v}</span>
              </div>
            ))}
            <button onClick={() => { setShowSummary(false); setElapsed(0); }}
              style={{ marginTop:20, width:"100%", background:C.primary, border:"none",
                borderRadius:999, padding:"14px 0", color:C.bg, fontWeight:700, fontSize:15, cursor:"pointer" }}>
              Tamam
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

// RACE LOBBY
function LobbyScreen({ navigate }) {
  const rooms = [
    { id:"r1", name:"Ahmet'in Odası", distance:2000, participants:[{name:"Ahmet"},{name:"Zeynep"}], maxParticipants:6, status:"waiting" },
    { id:"r2", name:"Hızlı Kürek 1k", distance:1000, participants:[{name:"Mert"}], maxParticipants:4, status:"waiting" },
    { id:"r3", name:"Sprint 500", distance:500, participants:[{name:"Ali"},{name:"Selin"},{name:"Can"}], maxParticipants:4, status:"racing" },
  ];
  const [showCreate, setShowCreate] = useState(false);
  const [roomName, setRoomName] = useState("");
  const [dist, setDist] = useState(2000);

  return (
    <div style={s.screen}>
      <div style={s.header}>
        <button onClick={() => navigate("home")} style={s.backBtn}>‹ Geri</button>
        <div style={s.headerTitle}>Canlı Yarış</div>
        <button onClick={() => setShowCreate(true)}
          style={{ background:C.primary, border:"none", borderRadius:999, padding:"6px 16px",
            color:C.bg, fontWeight:700, fontSize:12, cursor:"pointer" }}>+ Oda</button>
      </div>

      <div style={{ padding:"8px 16px", color:C.textMuted, fontSize:12, letterSpacing:1 }}>AKTİF ODALAR</div>

      <div style={{ padding:"0 16px", display:"flex", flexDirection:"column", gap:10 }}>
        {rooms.map(room => (
          <button key={room.id}
            onClick={() => room.status === "waiting" && navigate("race")}
            style={{ background:C.surface, border:`1px solid ${C.border}`, borderRadius:16,
              padding:16, textAlign:"left", cursor: room.status==="racing" ? "not-allowed" : "pointer",
              opacity: room.status==="racing" ? 0.6 : 1 }}>
            <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center" }}>
              <span style={{ color:C.textPrimary, fontWeight:600, fontSize:15 }}>{room.name}</span>
              <span style={{ fontSize:11, fontWeight:700, padding:"3px 10px", borderRadius:999,
                background: room.status==="racing" ? C.danger+"22" : C.success+"22",
                color: room.status==="racing" ? C.danger : C.success }}>
                {room.status==="racing" ? "YAŞIYOR" : "BEKLİYOR"}
              </span>
            </div>
            <div style={{ display:"flex", gap:16, marginTop:10 }}>
              <span style={{ color:C.primary, fontSize:13, fontWeight:700 }}>{room.distance}m</span>
              <span style={{ color:C.textSecondary, fontSize:13 }}>
                👤 {room.participants.length}/{room.maxParticipants} — {room.participants.map(p=>p.name).join(", ")}
              </span>
            </div>
          </button>
        ))}
      </div>

      {showCreate && (
        <div style={s.modalOverlay}>
          <div style={s.modalSheet}>
            <div style={s.modalTitle}>Oda Oluştur</div>
            <input placeholder="Oda adı..." value={roomName} onChange={e => setRoomName(e.target.value)}
              style={{ width:"100%", background:C.surfaceLight, border:`1px solid ${C.border}`,
                borderRadius:10, padding:12, color:C.textPrimary, fontSize:15, marginBottom:16, boxSizing:"border-box" }} />
            <div style={{ display:"flex", gap:8, marginBottom:20, flexWrap:"wrap" }}>
              {[500,1000,2000,5000].map(d => (
                <button key={d} onClick={() => setDist(d)}
                  style={{ flex:1, minWidth:60, background: d===dist ? C.primary : C.surfaceLight,
                    border:`1px solid ${d===dist ? C.primary : C.border}`, borderRadius:8, padding:"10px 0",
                    color: d===dist ? C.bg : C.textSecondary, fontWeight:700, cursor:"pointer" }}>
                  {d}m
                </button>
              ))}
            </div>
            <button onClick={() => { setShowCreate(false); navigate("race"); }}
              style={{ width:"100%", background:C.primary, border:"none", borderRadius:999,
                padding:"14px 0", color:C.bg, fontWeight:700, fontSize:15, cursor:"pointer" }}>
              Oluştur
            </button>
            <button onClick={() => setShowCreate(false)}
              style={{ marginTop:10, width:"100%", background:"transparent", border:"none",
                color:C.danger, fontSize:14, padding:10, cursor:"pointer" }}>
              İptal
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

// RACE SCREEN
function RaceScreen({ navigate }) {
  const [phase, setPhase] = useState("countdown"); // countdown | racing | finished
  const [countdown, setCountdown] = useState(5);
  const [elapsed, setElapsed] = useState(0);
  const timerRef = useRef(null);
  const startRef = useRef(0);
  const { data, reset } = useSimulatedRower(phase === "racing");
  const targetDist = 2000;

  const [opponents] = useState([
    { id:"o1", name:"Ahmet", color:C.raceColors[1], speed:1.04 },
    { id:"o2", name:"Zeynep", color:C.raceColors[2], speed:0.97 },
    { id:"o3", name:"Mert", color:C.raceColors[3], speed:1.01 },
  ]);
  const [opponentDists, setOpponentDists] = useState([0,0,0]);

  useEffect(() => {
    if (phase !== "countdown") return;
    const id = setInterval(() => {
      setCountdown(c => {
        if (c <= 1) {
          clearInterval(id);
          setPhase("racing");
          reset();
          startRef.current = Date.now();
          timerRef.current = setInterval(() => setElapsed(Date.now() - startRef.current), 100);
          return 0;
        }
        return c - 1;
      });
    }, 1000);
    return () => clearInterval(id);
  }, [phase]);

  useEffect(() => {
    if (phase !== "racing") return;
    const id = setInterval(() => {
      setOpponentDists(prev => prev.map((d, i) => {
        const newD = d + opponents[i].speed * 1.65;
        return newD;
      }));
    }, 100);
    return () => clearInterval(id);
  }, [phase]);

  useEffect(() => {
    if (phase === "racing" && data.distance >= targetDist) {
      clearInterval(timerRef.current);
      setPhase("finished");
    }
  }, [data.distance, phase]);

  const allRacers = [
    { id:"me", name:"Sen", color:C.raceColors[0], dist: data.distance },
    ...opponents.map((o,i) => ({ id:o.id, name:o.name, color:o.color, dist: opponentDists[i] })),
  ].sort((a,b) => b.dist - a.dist);

  return (
    <div style={s.screen}>
      {/* Countdown overlay */}
      {phase === "countdown" && (
        <div style={{ position:"absolute", inset:0, zIndex:50, background:"rgba(10,14,26,0.92)",
          display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center" }}>
          <div style={{ color:C.textSecondary, fontSize:13, letterSpacing:3, marginBottom:16 }}>HAZIR OL</div>
          <div style={{ fontSize:120, fontWeight:900, color:C.primary, fontFamily:"monospace",
            textShadow:`0 0 60px ${C.primary}` }}>
            {countdown}
          </div>
        </div>
      )}

      <div style={{ padding:"16px 16px 8px", display:"flex", alignItems:"center", justifyContent:"space-between" }}>
        <div style={{ color:C.textSecondary, fontSize:13 }}>2000m Yarışı</div>
        <div style={{ color:C.primary, fontFamily:"monospace", fontWeight:700, fontSize:20 }}>
          {formatTime(elapsed)}
        </div>
      </div>

      {/* Track visualization */}
      <div style={{ padding:"4px 16px 16px" }}>
        {allRacers.map((r, idx) => {
          const prog = Math.min(r.dist / targetDist, 1);
          return (
            <div key={r.id} style={{ marginBottom:10 }}>
              <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                <span style={{ color: r.id==="me" ? C.textPrimary : C.textSecondary, fontSize:13,
                  fontWeight: r.id==="me"?700:400 }}>
                  {idx+1}. {r.name}
                </span>
                <span style={{ color:r.color, fontFamily:"monospace", fontSize:12 }}>{Math.floor(r.dist)}m</span>
              </div>
              <div style={{ height:10, background:C.surfaceLight, borderRadius:999, overflow:"hidden",
                border:`1px solid ${r.id==="me" ? r.color+"44" : "transparent"}` }}>
                <div style={{ height:"100%", width:`${prog*100}%`, background:r.color,
                  borderRadius:999, transition:"width 0.2s",
                  boxShadow: r.id==="me" ? `0 0 8px ${r.color}` : "none" }} />
              </div>
            </div>
          );
        })}
      </div>

      {/* My stats */}
      <div style={{ padding:"0 16px", display:"grid", gridTemplateColumns:"repeat(3,1fr)", gap:8 }}>
        {[
          { label:"SPL", value: phase==="racing" ? data.spm : "--" },
          { label:"WATT", value: phase==="racing" ? data.watts : "--" },
          { label:"SPLIT", value: phase==="racing" ? formatSplit(data.split) : "--:--" },
        ].map(stat => (
          <div key={stat.label} style={{ background:C.surface, border:`1px solid ${C.border}`,
            borderRadius:12, padding:"12px 10px", textAlign:"center" }}>
            <div style={{ color:C.textMuted, fontSize:10, letterSpacing:1 }}>{stat.label}</div>
            <div style={{ color:C.primary, fontWeight:900, fontFamily:"monospace", fontSize:22, marginTop:2 }}>{stat.value}</div>
          </div>
        ))}
      </div>

      {/* Finished */}
      {phase === "finished" && (
        <div style={s.modalOverlay}>
          <div style={s.modalSheet}>
            <div style={{ textAlign:"center" }}>
              <div style={{ fontSize:48 }}>🥇</div>
              <div style={{ color:C.primary, fontSize:28, fontWeight:900, marginTop:8 }}>Yarış Bitti!</div>
              <div style={{ color:C.textSecondary, fontSize:14, marginTop:4 }}>Süren: {formatTime(elapsed)}</div>
            </div>
            <div style={{ marginTop:20, display:"flex", flexDirection:"column", gap:8 }}>
              {allRacers.map((r,i) => (
                <div key={r.id} style={{ display:"flex", alignItems:"center", gap:12,
                  padding:"10px 8px", borderBottom:`1px solid ${C.border}` }}>
                  <span style={{ fontSize:18, width:28 }}>{["🥇","🥈","🥉","4️⃣"][i]}</span>
                  <span style={{ color: r.id==="me"?C.primary:C.textPrimary, fontWeight: r.id==="me"?700:400, flex:1 }}>{r.name}</span>
                  <span style={{ color:C.textMuted, fontFamily:"monospace", fontSize:13 }}>{Math.floor(r.dist)}m</span>
                </div>
              ))}
            </div>
            <button onClick={() => navigate("home")}
              style={{ marginTop:20, width:"100%", background:C.primary, border:"none",
                borderRadius:999, padding:"14px 0", color:C.bg, fontWeight:700, fontSize:15, cursor:"pointer" }}>
              Ana Menü
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

// GHOST SCREEN
function GhostScreen({ navigate }) {
  const ghosts = [
    { id:1, finishTime:398000, dist:2000, date:"23 Haz", avgWatts:195, avgSplit:99500 },
    { id:2, finishTime:412000, dist:2000, date:"21 Haz", avgWatts:182, avgSplit:103000 },
    { id:3, finishTime:185000, dist:1000, date:"20 Haz", avgWatts:201, avgSplit:92500 },
  ];
  const [selected, setSelected] = useState(null);
  const [phase, setPhase] = useState("select"); // select | racing | finished
  const [elapsed, setElapsed] = useState(0);
  const timerRef = useRef(null);
  const startRef = useRef(0);
  const { data, reset } = useSimulatedRower(phase === "racing");

  const [ghostDist, setGhostDist] = useState(0);
  const ghostRef = useRef(null);

  const startRace = () => {
    reset();
    setElapsed(0);
    setGhostDist(0);
    setPhase("racing");
    startRef.current = Date.now();
    timerRef.current = setInterval(() => {
      setElapsed(Date.now() - startRef.current);
    }, 100);
    ghostRef.current = setInterval(() => {
      setGhostDist(d => d + (selected.dist / (selected.finishTime / 100)));
    }, 100);
  };

  useEffect(() => {
    if (phase === "racing" && data.distance >= (selected?.dist || 9999)) {
      clearInterval(timerRef.current);
      clearInterval(ghostRef.current);
      setPhase("finished");
    }
  }, [data.distance, phase]);

  const targetDist = selected?.dist || 2000;
  const myProg = Math.min(data.distance / targetDist, 1);
  const ghostProg = Math.min(ghostDist / targetDist, 1);
  const myAhead = data.distance - ghostDist;

  return (
    <div style={s.screen}>
      <div style={s.header}>
        <button onClick={() => phase==="select" ? navigate("home") : setPhase("select")} style={s.backBtn}>‹ Geri</button>
        <div style={s.headerTitle}>Hayalet Rakip</div>
        <div />
      </div>

      {phase === "select" && (
        <div style={{ padding:"0 16px" }}>
          <div style={{ color:C.textMuted, fontSize:11, letterSpacing:1, marginBottom:10 }}>KAYDEDİLMİŞ ANTRENMANLAR</div>
          {ghosts.map(g => (
            <button key={g.id}
              onClick={() => setSelected(g)}
              style={{ display:"block", width:"100%", background: selected?.id===g.id ? C.ghost+"15" : C.surface,
                border:`1px solid ${selected?.id===g.id ? C.ghost : C.border}`, borderRadius:14,
                padding:16, marginBottom:10, textAlign:"left", cursor:"pointer" }}>
              <div style={{ display:"flex", justifyContent:"space-between" }}>
                <span style={{ color:C.textPrimary, fontWeight:600 }}>{g.dist}m</span>
                <span style={{ color:C.ghost, fontFamily:"monospace", fontWeight:700 }}>{formatTime(g.finishTime)}</span>
              </div>
              <div style={{ display:"flex", gap:16, marginTop:8 }}>
                <span style={{ color:C.textMuted, fontSize:12 }}>{g.date}</span>
                <span style={{ color:C.accent, fontSize:12 }}>{g.avgWatts}W</span>
                <span style={{ color:C.warning, fontSize:12 }}>{formatSplit(g.avgSplit)} /500m</span>
              </div>
            </button>
          ))}
          <button onClick={startRace} disabled={!selected}
            style={{ width:"100%", background: selected ? C.ghost : C.surfaceLight,
              border:"none", borderRadius:999, padding:"16px 0", color: selected ? "#fff" : C.textMuted,
              fontWeight:700, fontSize:15, cursor: selected ? "pointer" : "not-allowed", marginTop:4,
              boxShadow: selected ? `0 0 24px ${C.ghost}44` : "none" }}>
            {selected ? `${selected.dist}m Yarışını Başlat` : "Bir Antrenman Seç"}
          </button>
        </div>
      )}

      {(phase === "racing" || phase === "finished") && (
        <div>
          <div style={{ textAlign:"center", padding:"16px 0 8px" }}>
            <div style={{ fontSize:56, fontWeight:900, color:C.primary, fontFamily:"monospace" }}>{formatTime(elapsed)}</div>
          </div>

          <div style={{ padding:"0 16px" }}>
            {/* Me */}
            <div style={{ marginBottom:10 }}>
              <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                <span style={{ color:C.primary, fontWeight:700, fontSize:13 }}>Sen</span>
                <span style={{ color:C.primary, fontFamily:"monospace", fontSize:12 }}>{Math.floor(data.distance)}m</span>
              </div>
              <div style={{ height:12, background:C.surfaceLight, borderRadius:999, overflow:"hidden" }}>
                <div style={{ height:"100%", width:`${myProg*100}%`, background:C.primary,
                  borderRadius:999, boxShadow:`0 0 8px ${C.primary}` }} />
              </div>
            </div>
            {/* Ghost */}
            <div style={{ marginBottom:16 }}>
              <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                <span style={{ color:C.ghost, fontWeight:700, fontSize:13 }}>👻 Hayalet</span>
                <span style={{ color:C.ghost, fontFamily:"monospace", fontSize:12 }}>{Math.floor(ghostDist)}m</span>
              </div>
              <div style={{ height:12, background:C.surfaceLight, borderRadius:999, overflow:"hidden" }}>
                <div style={{ height:"100%", width:`${ghostProg*100}%`, background:C.ghost,
                  borderRadius:999, boxShadow:`0 0 8px ${C.ghost}66` }} />
              </div>
            </div>

            {/* Delta */}
            <div style={{ background:C.surface, border:`1px solid ${C.border}`, borderRadius:12, padding:14, textAlign:"center", marginBottom:16 }}>
              <div style={{ color:C.textMuted, fontSize:11, letterSpacing:1 }}>HAYALETE GÖRE</div>
              <div style={{ fontSize:28, fontWeight:900, fontFamily:"monospace", marginTop:4,
                color: myAhead >= 0 ? C.success : C.danger }}>
                {myAhead >= 0 ? "+" : ""}{Math.floor(myAhead)}m
              </div>
            </div>

            <div style={{ display:"grid", gridTemplateColumns:"repeat(3,1fr)", gap:8 }}>
              {[
                { label:"SPL", value: phase==="racing" ? data.spm : "--" },
                { label:"WATT", value: phase==="racing" ? data.watts : "--" },
                { label:"SPLIT", value: phase==="racing" ? formatSplit(data.split) : "--:--" },
              ].map(stat => (
                <div key={stat.label} style={{ background:C.surface, border:`1px solid ${C.border}`,
                  borderRadius:12, padding:"12px 10px", textAlign:"center" }}>
                  <div style={{ color:C.textMuted, fontSize:10, letterSpacing:1 }}>{stat.label}</div>
                  <div style={{ color:C.primary, fontWeight:900, fontFamily:"monospace", fontSize:22, marginTop:2 }}>{stat.value}</div>
                </div>
              ))}
            </div>
          </div>
        </div>
      )}

      {phase === "finished" && (
        <div style={s.modalOverlay}>
          <div style={s.modalSheet}>
            <div style={{ textAlign:"center" }}>
              <div style={{ fontSize:48 }}>{myAhead >= 0 ? "🏆" : "😤"}</div>
              <div style={{ color: myAhead>=0?C.success:C.danger, fontSize:24, fontWeight:900, marginTop:8 }}>
                {myAhead >= 0 ? "Hayaleti Geçtin!" : "Hayalet Kazandı!"}
              </div>
              <div style={{ color:C.textSecondary, fontSize:14, marginTop:4 }}>Süren: {formatTime(elapsed)}</div>
              <div style={{ color: myAhead>=0?C.success:C.danger, fontSize:14, fontWeight:700 }}>
                {myAhead>=0?"+":""}{Math.floor(myAhead)}m fark
              </div>
            </div>
            <button onClick={() => { setPhase("select"); setSelected(null); }}
              style={{ marginTop:24, width:"100%", background:C.ghost, border:"none",
                borderRadius:999, padding:"14px 0", color:"#fff", fontWeight:700, fontSize:15, cursor:"pointer" }}>
              Tekrar Dene
            </button>
            <button onClick={() => navigate("home")}
              style={{ marginTop:10, width:"100%", background:"transparent", border:"none",
                color:C.textSecondary, fontSize:14, padding:10, cursor:"pointer" }}>
              Ana Menü
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

// LEADERBOARD
function LeaderboardScreen({ navigate }) {
  const [distFilter, setDistFilter] = useState(2000);
  const distances = [0,500,1000,2000,5000];
  const distLabels = ["Tümü","500m","1000m","2000m","5000m"];

  const allEntries = [
    { rank:1, name:"Ali Yılmaz", dist:2000, time:376000, split:94000, watts:218, spm:26, date:"23 Haz" },
    { rank:2, name:"Zeynep Kaya", dist:2000, time:389000, split:97250, watts:204, spm:24, date:"22 Haz" },
    { rank:3, name:"Mert Demir", dist:2000, time:398000, split:99500, watts:195, spm:25, date:"23 Haz" },
    { rank:4, name:"Selin Ak", dist:2000, time:411000, split:102750, watts:185, spm:23, date:"21 Haz" },
    { rank:5, name:"Can Şahin", dist:2000, time:426000, split:106500, watts:174, spm:22, date:"20 Haz" },
    { rank:1, name:"Ali Yılmaz", dist:1000, time:175000, split:87500, watts:235, spm:27, date:"22 Haz" },
    { rank:2, name:"Mert Demir", dist:1000, time:181000, split:90500, watts:221, spm:26, date:"21 Haz" },
    { rank:1, name:"Zeynep Kaya", dist:500, time:84000, split:84000, watts:252, spm:30, date:"23 Haz" },
    { rank:1, name:"Can Şahin", dist:5000, time:1050000, split:105000, watts:172, spm:22, date:"19 Haz" },
  ];

  const filtered = distFilter === 0 ? allEntries : allEntries.filter(e => e.dist === distFilter);
  const rankEmoji = (r) => r===1?"🥇":r===2?"🥈":r===3?"🥉":r;
  const rankColor = (r) => r===1?"#FFD700":r===2?"#C0C0C0":r===3?"#CD7F32":C.textMuted;

  return (
    <div style={s.screen}>
      <div style={s.header}>
        <button onClick={() => navigate("home")} style={s.backBtn}>‹ Geri</button>
        <div style={s.headerTitle}>Liderlik Tablosu</div>
        <div />
      </div>

      {/* Distance filter */}
      <div style={{ display:"flex", gap:6, padding:"4px 16px 12px", overflowX:"auto" }}>
        {distances.map((d,i) => (
          <button key={d} onClick={() => setDistFilter(d)}
            style={{ flexShrink:0, background: d===distFilter ? C.primary : C.surfaceLight,
              border:`1px solid ${d===distFilter ? C.primary : C.border}`, borderRadius:999,
              padding:"7px 14px", color: d===distFilter ? C.bg : C.textSecondary,
              fontWeight: d===distFilter ? 700 : 400, fontSize:13, cursor:"pointer" }}>
            {distLabels[i]}
          </button>
        ))}
      </div>

      <div style={{ overflowY:"auto", flex:1, padding:"0 16px" }}>
        {filtered.map((e,idx) => (
          <div key={idx} style={{ display:"flex", alignItems:"center", gap:12,
            background:C.surface, border:`1px solid ${C.border}`, borderRadius:14,
            padding:"14px 14px", marginBottom:8 }}>
            <div style={{ width:32, textAlign:"center", fontSize: e.rank<=3?22:14, color:rankColor(e.rank), fontWeight:700 }}>
              {rankEmoji(e.rank)}
            </div>
            <div style={{ flex:1 }}>
              <div style={{ color:C.textPrimary, fontWeight:600, fontSize:14 }}>{e.name}</div>
              <div style={{ display:"flex", gap:10, marginTop:4 }}>
                <span style={{ color:C.accent, fontSize:12 }}>{e.watts}W</span>
                <span style={{ color:C.warning, fontSize:12 }}>{formatSplit(e.split)}</span>
                <span style={{ color:C.textMuted, fontSize:12 }}>{e.spm} spl</span>
              </div>
            </div>
            <div style={{ textAlign:"right" }}>
              <div style={{ color:C.primary, fontFamily:"monospace", fontWeight:700, fontSize:15 }}>{formatTime(e.time)}</div>
              <div style={{ color:C.textMuted, fontSize:11, marginTop:2 }}>{e.dist}m · {e.date}</div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

// ─── SHARED STYLES ────────────────────────────────────────────────────────────
const s = {
  screen: {
    display:"flex", flexDirection:"column", height:"100%",
    background:C.bg, color:C.textPrimary, fontFamily:"system-ui, -apple-system, sans-serif",
    overflowY:"auto",
  },
  header: {
    display:"flex", alignItems:"center", justifyContent:"space-between",
    padding:"16px 16px 8px", borderBottom:`1px solid ${C.border}`,
  },
  headerTitle: { color:C.textPrimary, fontWeight:700, fontSize:17 },
  backBtn: { background:"transparent", border:"none", color:C.primary, fontSize:16, cursor:"pointer", padding:"4px 0" },
  modalOverlay: {
    position:"absolute", inset:0, background:"rgba(0,0,0,0.75)",
    display:"flex", flexDirection:"column", justifyContent:"flex-end", zIndex:100,
  },
  modalSheet: {
    background:C.surface, borderTopLeftRadius:24, borderTopRightRadius:24,
    padding:24, maxHeight:"80%", overflowY:"auto",
  },
  modalTitle: { color:C.textPrimary, fontSize:20, fontWeight:700, marginBottom:16 },
  spinner: {
    width:18, height:18, border:`2px solid ${C.border}`, borderTopColor:C.primary,
    borderRadius:"50%", animation:"spin 0.8s linear infinite",
  },
};

// ─── APP ROOT ─────────────────────────────────────────────────────────────────
export default function App() {
  const [screen, setScreen] = useState("home");
  const [connected, setConnected] = useState(false);

  const navigate = (to) => setScreen(to);

  const screens = {
    home: <HomeScreen navigate={navigate} connected={connected} setConnected={setConnected} />,
    training: <TrainingScreen navigate={navigate} />,
    lobby: <LobbyScreen navigate={navigate} />,
    race: <RaceScreen navigate={navigate} />,
    ghost: <GhostScreen navigate={navigate} />,
    leaderboard: <LeaderboardScreen navigate={navigate} />,
  };

  return (
    <div style={{
      display:"flex", justifyContent:"center", alignItems:"center",
      minHeight:"100vh", background:"#050810", fontFamily:"system-ui",
    }}>
      <style>{`
        @keyframes spin { to { transform: rotate(360deg); } }
        * { margin: 0; padding: 0; box-sizing: border-box; }
        button { font-family: inherit; }
        input { font-family: inherit; outline: none; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: #0A0E1A; }
        ::-webkit-scrollbar-thumb { background: #1E3A5F; border-radius: 2px; }
      `}</style>

      {/* Phone frame */}
      <div style={{
        width:390, height:844,
        background:"#0A0E1A",
        borderRadius:52,
        border:"10px solid #1a1a2e",
        boxShadow:"0 0 80px rgba(0,229,255,0.08), 0 40px 80px rgba(0,0,0,0.8), inset 0 0 0 1px #2a2a4a",
        overflow:"hidden", position:"relative", display:"flex", flexDirection:"column",
      }}>
        {/* Notch */}
        <div style={{
          position:"absolute", top:0, left:"50%", transform:"translateX(-50%)",
          width:120, height:34, background:"#1a1a2e", borderRadius:"0 0 20px 20px", zIndex:200,
        }} />

        {/* Status bar */}
        <div style={{
          height:44, background:C.bg, display:"flex", alignItems:"flex-end",
          justifyContent:"space-between", padding:"0 24px 6px", zIndex:150, flexShrink:0,
        }}>
          <span style={{ color:C.textPrimary, fontSize:13, fontWeight:600 }}>9:41</span>
          <span style={{ color:C.textPrimary, fontSize:11 }}>▲▲▲ ◾ 🔋</span>
        </div>

        {/* Screen content */}
        <div style={{ flex:1, overflow:"hidden", position:"relative" }}>
          {screens[screen]}
        </div>

        {/* Home indicator */}
        <div style={{
          height:32, background:C.bg, display:"flex", alignItems:"center", justifyContent:"center", flexShrink:0,
        }}>
          <div style={{ width:120, height:5, background:C.border, borderRadius:3 }} />
        </div>
      </div>

      {/* Screen nav tabs below phone */}
      <div style={{
        position:"fixed", bottom:24,
        display:"flex", gap:6, background:"rgba(17,24,39,0.95)",
        border:"1px solid #1E3A5F", borderRadius:999, padding:"8px 12px",
        backdropFilter:"blur(10px)",
      }}>
        {[
          { id:"home", label:"🏠 Anasayfa" },
          { id:"training", label:"📊 Antrenman" },
          { id:"lobby", label:"🏁 Yarış Lobisi" },
          { id:"race", label:"⚡ Yarış" },
          { id:"ghost", label:"👻 Hayalet" },
          { id:"leaderboard", label:"🏆 Tablo" },
        ].map(tab => (
          <button key={tab.id} onClick={() => navigate(tab.id)}
            style={{
              background: screen===tab.id ? C.primary : "transparent",
              border: screen===tab.id ? "none" : `1px solid ${C.border}`,
              borderRadius:999, padding:"6px 12px",
              color: screen===tab.id ? C.bg : C.textSecondary,
              fontWeight: screen===tab.id ? 700 : 400,
              fontSize:12, cursor:"pointer", whiteSpace:"nowrap",
            }}>
            {tab.label}
          </button>
        ))}
      </div>
    </div>
  );
}
