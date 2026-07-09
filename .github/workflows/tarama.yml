// TS EDGE PRO Trend Scanner v3 — GitHub Actions sürümü
// Veri: Yahoo Finance (gecikmeli, halka açık). Bağımlılık yok, Node 18+ yeterli.
// Çıktı: docs/data.json

const fs = require("fs");

const BIST30 = ["AKBNK","ARCLK","ASELS","ASTOR","BIMAS","BRSAN","EKGYO","ENKAI","EREGL","FROTO","GARAN","GUBRF","HEKTS","ISCTR","KCHOL","KONTR","KOZAL","KRDMD","OYAKC","PETKM","PGSUS","SAHOL","SASA","SISE","TCELL","THYAO","TOASO","TUPRS","ULKER","YKBNK"];
const SYMBOLS = [
  ...BIST30.map(s => ({ sym: s + ".IS", name: s, tip: "BIST" })),
  { sym: "GC=F", name: "Altın (Ons)", tip: "Emtia" },
  { sym: "SI=F", name: "Gümüş", tip: "Emtia" },
  { sym: "BZ=F", name: "Brent", tip: "Emtia" },
  { sym: "CL=F", name: "WTI", tip: "Emtia" },
];

// ── Yahoo chart API ─────────────────────────────────────────
async function fetchYahoo(sym, interval, range) {
  const url = `https://query1.finance.yahoo.com/v8/finance/chart/${encodeURIComponent(sym)}?interval=${interval}&range=${range}`;
  const res = await fetch(url, { headers: { "User-Agent": "Mozilla/5.0" } });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const j = await res.json();
  const r = j?.chart?.result?.[0];
  if (!r) throw new Error("veri yok");
  const q = r.indicators.quote[0];
  const bars = [];
  for (let i = 0; i < r.timestamp.length; i++) {
    if (q.close[i] == null) continue;
    bars.push({ t: r.timestamp[i], o: q.open[i], h: q.high[i], l: q.low[i], c: q.close[i], v: q.volume[i] ?? 0 });
  }
  return bars;
}

// 1 saatlik barlardan 4 saatlik barlar üret
function to4h(bars) {
  const out = [];
  for (let i = 0; i < bars.length; i += 4) {
    const g = bars.slice(i, i + 4);
    out.push({
      t: g[0].t,
      o: g[0].o,
      h: Math.max(...g.map(b => b.h)),
      l: Math.min(...g.map(b => b.l)),
      c: g[g.length - 1].c,
      v: g.reduce((s, b) => s + b.v, 0),
    });
  }
  return out;
}

// ── İndikatörler ────────────────────────────────────────────
const sma = (a, len, i) => {
  if (i + 1 < len) return null;
  let s = 0;
  for (let k = i - len + 1; k <= i; k++) s += a[k];
  return s / len;
};

function rsi(c, len = 14) {
  if (c.length < len + 1) return null;
  let g = 0, l = 0;
  for (let i = 1; i <= len; i++) { const d = c[i] - c[i-1]; d > 0 ? g += d : l -= d; }
  let ag = g / len, al = l / len;
  for (let i = len + 1; i < c.length; i++) {
    const d = c[i] - c[i-1];
    ag = (ag * (len-1) + Math.max(d,0)) / len;
    al = (al * (len-1) + Math.max(-d,0)) / len;
  }
  return al === 0 ? 100 : 100 - 100 / (1 + ag / al);
}

function adx(h, l, c, len = 14) {
  const n = c.length;
  if (n < len * 2 + 1) return null;
  const tr = [], pdm = [], ndm = [];
  for (let i = 1; i < n; i++) {
    tr.push(Math.max(h[i]-l[i], Math.abs(h[i]-c[i-1]), Math.abs(l[i]-c[i-1])));
    const up = h[i]-h[i-1], dn = l[i-1]-l[i];
    pdm.push(up > dn && up > 0 ? up : 0);
    ndm.push(dn > up && dn > 0 ? dn : 0);
  }
  let str = 0, spd = 0, snd = 0;
  for (let i = 0; i < len; i++) { str += tr[i]; spd += pdm[i]; snd += ndm[i]; }
  const dx = [];
  for (let i = len; i < tr.length; i++) {
    str = str - str/len + tr[i]; spd = spd - spd/len + pdm[i]; snd = snd - snd/len + ndm[i];
    const p = 100*spd/str, m = 100*snd/str;
    dx.push(p+m === 0 ? 0 : 100*Math.abs(p-m)/(p+m));
  }
  if (dx.length < len) return null;
  let a = dx.slice(0, len).reduce((x,y)=>x+y)/len;
  for (let i = len; i < dx.length; i++) a = (a*(len-1)+dx[i])/len;
  return a;
}

function fisher(h, l, len = 9) {
  const hl2 = h.map((v,i)=>(v+l[i])/2);
  const out = []; let val = 0, fish = 0;
  for (let i = 0; i < h.length; i++) {
    const from = Math.max(0, i-len+1);
    let hi = -Infinity, lo = Infinity;
    for (let k = from; k <= i; k++) { hi = Math.max(hi,hl2[k]); lo = Math.min(lo,hl2[k]); }
    val = 0.66*((hl2[i]-lo)/Math.max(hi-lo,1e-10)-0.5) + 0.67*val;
    val = Math.min(Math.max(val,-0.999),0.999);
    fish = 0.5*Math.log((1+val)/(1-val)) + 0.5*fish;
    out.push(fish);
  }
  return out;
}

// ── 100 puanlık skor ────────────────────────────────────────
function score(bars) {
  const c = bars.map(b=>b.c), h = bars.map(b=>b.h), l = bars.map(b=>b.l), v = bars.map(b=>b.v);
  const i = c.length - 1;
  if (i < 60) return { score: null };

  const s10 = sma(c,10,i), s50 = sma(c,50,i), s100 = sma(c,100,i), s200 = sma(c,200,i);
  const r = rsi(c.slice(-120));
  const a = adx(h,l,c);
  const volAvg = sma(v,20,i);
  const relVol = volAvg ? v[i]/volAvg : null;
  const f = fisher(h,l);
  const fTurn = f[i] > f[i-1] && f[i-1] <= f[i-2];

  let newBreak = false, brkIdx = -1;
  if (s100 !== null) {
    for (let k = i; k > i-3 && k > 100; k--) {
      const sp = sma(c,100,k-1), sc2 = sma(c,100,k);
      if (c[k-1] <= sp && c[k] > sc2) { newBreak = true; brkIdx = k; break; }
    }
  }
  let brkVol = false;
  if (newBreak && brkIdx > 20) {
    const va = sma(v,20,brkIdx);
    brkVol = va && v[brkIdx]/va > 1.3;
  }

  let sc = 0;
  if (s10 !== null && s50 !== null && s10 > s50) sc += 10;
  if (s100 !== null && s50 !== null && s100 > s50) sc += 10;
  if (s200 !== null && s100 !== null && s200 > s100) sc += 10;
  if (s50 !== null && c[i] > s50) sc += 10;
  if (fTurn) sc += 15;
  if (r !== null && r > 55) sc += 10;
  if (a !== null && a > 25) sc += 10;
  if (relVol !== null && relVol > 1.5) sc += 10;
  if (newBreak) sc += 10;
  if (brkVol) sc += 5;

  let atr = 0;
  const alen = Math.min(14, i);
  for (let k = i-alen+1; k <= i; k++)
    atr += Math.max(h[k]-l[k], Math.abs(h[k]-c[k-1]), Math.abs(l[k]-c[k-1]));
  atr /= alen;
  const risk = Math.min(atr/c[i]*100*10 + (s50 ? Math.abs(c[i]-s50)/c[i]*100*5 : 0), 100);

  return { score: sc, rsi: r, adx: a, relVol, risk, close: c[i], sma10: s10 };
}

// ── Ana akış ────────────────────────────────────────────────
async function main() {
  const rows = [];
  let usdtry = null;
  try {
    const fx = await fetchYahoo("USDTRY=X", "1h", "5d");
    usdtry = fx[fx.length-1].c;
  } catch {}

  for (const { sym, name, tip } of SYMBOLS) {
    try {
      const b1 = await fetchYahoo(sym, "1h", "1y");
      const b4 = to4h(b1);
      const r1 = score(b1), r4 = score(b4);
      const common = r1.score !== null && r4.score !== null
        ? 0.4*r1.score + 0.6*r4.score : (r4.score ?? r1.score);
      const near10 = r1.sma10 && Math.abs(r1.close-r1.sma10)/r1.sma10 < 0.01;
      const secondBuy = common >= 70 && near10 && r1.rsi > 45 && r1.rsi < 60;
      const warrant = common >= 75 && (r1.adx ?? 0) > 30 && (r1.relVol ?? 0) > 2;
      const sig = common >= 80 ? "GÜÇLÜ AL" : secondBuy ? "İKİNCİ ALIM" : common >= 50 ? "İZLE" : "—";
      const row = { name, sym, tip, s1: r1.score, s4: r4.score,
        common: Math.round(common*10)/10, sig, secondBuy, warrant,
        risk: Math.round((r4.risk ?? r1.risk ?? 0)),
        rsi: r1.rsi ? Math.round(r1.rsi) : null,
        adx: r1.adx ? Math.round(r1.adx) : null,
        relVol: r1.relVol ? Math.round(r1.relVol*100)/100 : null,
        close: Math.round(r1.close*100)/100 };
      if (sym === "GC=F" && usdtry) row.gram = Math.round(r1.close*usdtry/31.1035*100)/100;
      rows.push(row);
      console.log(`✓ ${name}: ortak ${row.common} — ${sig}`);
    } catch (e) {
      rows.push({ name, sym, tip, err: e.message });
      console.log(`✗ ${name}: ${e.message}`);
    }
    await new Promise(r => setTimeout(r, 400)); // nazik hız
  }

  rows.sort((a,b) => (b.common ?? -1) - (a.common ?? -1));
  const out = { updated: new Date().toISOString(), usdtry, rows };
  fs.mkdirSync("docs", { recursive: true });
  fs.writeFileSync("docs/data.json", JSON.stringify(out, null, 1));
  console.log(`\nTamamlandı: ${rows.length} sembol → docs/data.json`);
}

main().catch(e => { console.error(e); process.exit(1); });
