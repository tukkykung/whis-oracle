# Design Spec — Brain Screen v2: Cosmic Universe
**Issue**: #74 | **วีส** | **Date**: 2026-03-19
**Status**: Draft v2 — supersedes v1 (radial graph ทิ้งแล้ว)
**Concept**: Universe / Cosmic — Oracle = Black Hole ตรงกลาง, Memory = Stars/Planets

---

## Concept

จักรวาลแห่งความจำ — Oracle คือ Black Hole ที่ดูดทุกความรู้เข้าหาตัวเอง
Memory nodes ลอยอยู่ในอวกาศ เก่า = ดาวริมขอบจักรวาล, ใหม่ = ดาวใกล้ศูนย์กลาง
Connections = light beams ที่ส่องระหว่างดาว ไม่ใช่ edge ธรรมดา

---

## Layout Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER (60px) — translucent, blur backdrop                      │
│  ○ [Oracle] ● Active  ░░░░░░░░░░ Token 68%  [filter chips]     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                  FULL-SCREEN COSMIC CANVAS                      │
│              (100vw × calc(100vh - 60px))                       │
│                                                                 │
│         ·  ·    ✦         ·       ✦    ·                       │
│      ·      [RECALL]          ·      [ARCHIVAL]·                │
│           ·    \   ·    ·   /    ·                              │
│    ·          [CORE]──●──[CORE]         ·                       │
│           ·    /   ╔═══╗   \    ·                              │
│      ·        /   ║ ☀ ║    \        ·                          │
│              /    ╚═══╝     \                                   │
│      [RECALL]  ←light beam→  [CORE]                             │
│         ·          ·              ·       ·                     │
│                  [particle field — always on]                   │
│                                                                 │
│  [DETAIL CARD — bottom right, slides up on click]               │
└─────────────────────────────────────────────────────────────────┘
```

**เปลี่ยนจาก v1**: ไม่มี left/right panel แยก — ทุกอย่างอยู่ใน cosmic canvas เต็มจอ
Detail = floating card ที่ pop up ตรงจอ ไม่ใช่ side panel

---

## Color System

| Element | Hex | ใช้กับ |
|---------|-----|--------|
| Deep space bg | `#02020f` | page background |
| Nebula overlay | `#0a0520` radial | center glow area |
| Oracle / Black Hole | `#ffffff` core + `#00ffd4` corona | center node |
| CORE stars | `#00ffd4` | newest memory (< 1 day) |
| RECALL stars | `#a78bfa` | recent memory (1-7 days) |
| ARCHIVAL stars | `#1e3a5f` dim | old memory (> 7 days) |
| Light beams | `rgba(167,139,250,0.3)` | connections |
| Particle dust | `rgba(255,255,255,0.4)` | bg particles |
| Star twinkle | `#fff` → `rgba(255,255,255,0)` | random stars |
| Nebula cloud | `#7c3aed` at 5% opacity | background haze |

---

## CSS Variables

```css
:root {
  --space-bg:        #02020f;
  --nebula-center:   radial-gradient(ellipse 60% 60% at 50% 50%,
                       rgba(124,58,237,0.08) 0%, transparent 70%);
  --color-oracle:    #00ffd4;
  --color-core:      #00ffd4;
  --color-recall:    #a78bfa;
  --color-archival:  #1e3a5f;
  --glow-core:       0 0 20px rgba(0,255,212,0.6),
                     0 0 60px rgba(0,255,212,0.2);
  --glow-recall:     0 0 12px rgba(167,139,250,0.5);
  --glow-archival:   0 0 6px rgba(30,58,95,0.4);
  --header-h:        60px;
}
```

---

## Node Design

### Oracle Node (Black Hole / Sun) — center
```
Size: 64px diameter
Shape: circle
Core: #02020f (dark center — black hole)
Corona: border 3px solid #00ffd4
Glow: 0 0 0 8px rgba(0,255,212,0.15),
      0 0 0 20px rgba(0,255,212,0.06),
      0 0 40px rgba(0,255,212,0.3)
Animation: corona-pulse 4s ease-in-out infinite
           rotate corona rings slowly 20s linear infinite
Label: Oracle name — Inter 600 12px, color #00ffd4, below node
```

### Memory Nodes (Stars/Planets)

| Tier | Diameter | Color | Glow | Shape |
|------|----------|-------|------|-------|
| CORE | 18-24px (random) | `#00ffd4` | `var(--glow-core)` | circle |
| RECALL | 12-18px | `#a78bfa` | `var(--glow-recall)` | circle |
| ARCHIVAL | 6-10px | `#1e3a5f` | `var(--glow-archival)` | circle |

Node size = random range ทำให้ดูเป็น star field จริง ไม่ uniform

---

## Connection Lines (Light Beams)

ไม่ใช่ SVG line ธรรมดา — ใช้ CSS linear-gradient beam:

```css
.light-beam {
  position: absolute;
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(167,139,250,0.4) 30%,
    rgba(167,139,250,0.6) 50%,
    rgba(167,139,250,0.4) 70%,
    transparent 100%
  );
  transform-origin: left center;
  /* rotate + width calculated from node positions via JS */
}
```

CORE → Oracle connections: `rgba(0,255,212,0.5)` brighter beam
ARCHIVAL connections: ไม่แสดง (too far, lost in space)

---

## Particle System

```
จำนวน: 120 particles
ขนาด: 1-3px, random
สี: rgba(255,255,255, random 0.2-0.8)
Animation: twinkle (opacity oscillate) — random duration 2-8s
           drift (translateX/Y ±20px) — random duration 15-40s
Spawn: random position ทั่วจอ
```

CSS-only approach (ไม่ต้อง canvas/WebGL):
```css
.particle {
  position: absolute;
  border-radius: 50%;
  animation: twinkle var(--dur) ease-in-out infinite,
             drift var(--drift-dur) ease-in-out infinite alternate;
}
@keyframes twinkle {
  0%, 100% { opacity: var(--min-op); }
  50%       { opacity: var(--max-op); }
}
@keyframes drift {
  from { transform: translate(0, 0); }
  to   { transform: translate(var(--dx), var(--dy)); }
}
```

---

## Z-Index Plan

| Layer | z-index | Element |
|-------|---------|---------|
| Deep space bg | 0 | `#02020f` full page |
| Nebula overlay | 1 | radial gradient haze |
| Particles | 2 | 120 star dust dots |
| Light beams | 5 | connection lines (SVG absolute) |
| Memory nodes | 10 | ARCHIVAL → RECALL → CORE (order) |
| Oracle node | 15 | center black hole / sun |
| Header | 30 | translucent top bar |
| Detail card | 40 | floating memory detail |
| Tooltip | 45 | node hover label |

---

## HTML Structure

```html
<div class="brain-screen">
  <!-- z:30 -->
  <header class="brain-header">
    <div class="oracle-id">
      <span class="oracle-corona-dot"></span>
      <span class="oracle-name">วีส</span>
      <span class="status-badge">● Active</span>
    </div>
    <div class="token-meter">
      <div class="token-fill" style="--pct: 68%"></div>
      <span>68%</span>
    </div>
    <div class="tier-filters">
      <button class="filter-chip active" data-tier="core">Core</button>
      <button class="filter-chip active" data-tier="recall">Recall</button>
      <button class="filter-chip active" data-tier="archival">Archival</button>
    </div>
  </header>

  <!-- Full cosmic canvas -->
  <div class="cosmic-canvas">                <!-- z:0-15, overflow: hidden -->
    <div class="nebula-bg"></div>            <!-- z:1 — radial gradient -->
    <div class="particle-field"></div>       <!-- z:2 — 120 particles via JS -->
    <svg class="beam-layer"></svg>           <!-- z:5 — light beams -->
    <div class="node-field">                 <!-- z:10-15 -->
      <!-- Oracle center -->
      <div class="oracle-node" id="oracle-center">
        <div class="corona-ring ring-1"></div>
        <div class="corona-ring ring-2"></div>
        <div class="oracle-core"></div>
      </div>
      <!-- Memory nodes — positioned absolute via JS -->
      <div class="memory-node" data-tier="core" data-id="..."></div>
      <!-- ... -->
    </div>
  </div>

  <!-- z:40 — hidden by default -->
  <div class="detail-card" id="detail-card">
    <div class="detail-tier-badge"></div>
    <h3 class="detail-title"></h3>
    <p class="detail-date"></p>
    <p class="detail-preview"></p>
    <button class="detail-close">✕</button>
  </div>
</div>
```

---

## Animation Specs

| Animation | Element | Property | Duration | Easing |
|-----------|---------|----------|----------|--------|
| corona-pulse | Oracle corona | box-shadow scale | 4s | ease-in-out infinite |
| corona-rotate | ring-1 | transform rotate | 20s | linear infinite |
| corona-rotate-rev | ring-2 | transform rotate | 30s | linear infinite reverse |
| star-twinkle | particles | opacity | 2-8s random | ease-in-out infinite |
| star-drift | particles | translateX/Y | 15-40s random | ease-in-out infinite alternate |
| node-float | memory nodes | translateY ±6px | 4-10s random | ease-in-out infinite alternate |
| node-appear | new node | scale 0→1 + opacity | 600ms | cubic-bezier(0.34,1.56,0.64,1) |
| beam-pulse | light beams | opacity 0.3→0.7 | 3s | ease-in-out infinite |
| detail-slidein | detail card | translateY 20px→0 + opacity | 250ms | ease-out |

---

## 3 States

### State 1: Idle
- Corona pulse slow (4s), particles twinkle, nodes float gently
- Light beams กระพริบ slow (opacity 0.3 → 0.6)
- No user interaction indicator

### State 2: Active Research (new memory being added)
- Oracle corona flares: glow radius doubles, faster pulse 1.5s
- New node spawns at edge → animate toward Oracle (translateX/Y to position) 1.2s ease-in
- Beam draws from new node to Oracle: strokeDashoffset 0 → length 800ms
- Particle density increases: 120 → 200 (add 80 temporarily)

### State 3: Memory Inspector (click node)
- Clicked node: scale 1.5, glow brighten 2x, border `2px solid #fff`
- Light beams from this node → stay bright, others dim to 0.1
- Other nodes: opacity 0.3
- Detail card: slides up from bottom-right, `translateY(20px) → (0)` 250ms
- Click outside or ✕ → reverse all

---

## Fit-to-Screen Strategy

ไม่ใช้ Cytoscape.js — ใช้ pure CSS absolute positioning + JS สำหรับ layout:

```js
// วาง Oracle ตรงกลาง canvas เสมอ
oracleNode.style.left = '50%'
oracleNode.style.top = '50%'

// กระจาย memory nodes เป็น concentric zones
// CORE: radius 120-180px จาก center
// RECALL: radius 200-280px
// ARCHIVAL: radius 320-420px
// angle = random, แต่ distribute ให้ไม่ overlap
```

Canvas = `position: relative`, nodes = `position: absolute`, transform: translate(-50%, -50%)
**ไม่มี scroll, ไม่มี zoom** — nodes scale ตาม canvas size

---

## Library Dependencies

| Library | Version | ใช้ทำ |
|---------|---------|-------|
| ไม่มี graph lib | — | ทุกอย่าง CSS + vanilla JS |
| (optional) anime.js | ^3.2 | particle/node animation smoother |

ไม่ต้อง Cytoscape.js แล้ว — cosmic layout ทำ custom ง่ายกว่า และ control glow/particle ได้เต็มที่

---

*วีส — Angel of Precision 🟦 | Issue #74 v2 | 2026-03-19*
*v1 (Radial Graph) superseded — Nothing is Deleted, archived in v1 file*
