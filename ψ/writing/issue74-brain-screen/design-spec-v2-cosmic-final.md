# Design Spec — Brain Screen v2 Final: Cosmic Universe
**Issue**: #74 | **วีส** | **Date**: 2026-03-19
**Status**: Final Draft — รอ เบจิต้า approve
**Supersedes**: design-spec-v2.md (upgraded with บุลมา research round 2)

**Primary Reference**: Vestige AI Dashboard — memory=star, connections=constellation, pulse on access

---

## Concept (refined)

ไม่ใช่แค่ "space background" — เป็น **Constellation Memory Map**
แต่ละ memory domain = constellation cluster (เหมือน Orion, Cassiopeia)
Access frequency = star pulse intensity
Recency = color temperature (blue=ใหม่, dim red=เก่า)
Forgetting = opacity fade เมื่อเวลาผ่าน

Oracle = **Black Hole / Pulsar** ตรงกลาง ที่ทุก constellation โคจรรอบ

---

## Stack ที่เปลี่ยนจาก v2 draft

| เดิม (v2 draft) | ใหม่ (final) | เหตุผล |
|-----------------|-------------|--------|
| Pure CSS + Vanilla JS | **React Three Fiber + three-nebula** | 3D space, nebula effect จริง |
| CSS particles (120 divs) | **three-nebula** particle system | performance + visual quality |
| SVG light beams | **3d-force-graph** edges | force-directed constellation |
| Cytoscape.js (ถูกทิ้งตั้งแต่ v2) | — | — |

```
React Three Fiber    — 3D canvas, camera, scene
3d-force-graph       — constellation node-link layout (force-directed 3D)
three-nebula         — nebula particle bg + warp effects
@react-three/drei    — OrbitControls, Stars, Billboard labels
```

---

## Layout Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER (60px) — glass blur, position: fixed, z: 30             │
│  [●] วีส  ■ Active  ████░░ Token 68%  [Core][Recall][Archive]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│              THREE.js CANVAS — full screen                      │
│         (position: fixed, inset: 0, z: 0)                       │
│                                                                 │
│   ·  ✦  ·    [cluster: learnings]    ·       ✦     ·           │
│      ·              ★ ─ ─ ─ ★             ·                    │
│  ✦       ★ ─ ─ ─ ─ ─ ★ ─ ─ ─ ─ ─ ★            ·               │
│               ─ ─ [☯ ORACLE] ─ ─                               │
│  ·     ★ ─ ─ ─ ─ ─ ★  pulse ─ ★ ─ ─ ─ ★       ✦              │
│         [cluster: retros]   [cluster: feedback]                 │
│   ✦  ·     ·       nebula drift bg          ·       ·  ✦       │
│                                                                 │
│                              ┌─────────────────────┐           │
│                              │ DETAIL CARD (z: 40) │           │
│                              │ slides up on click  │           │
│                              └─────────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Color System (updated from บุลมา research)

| Element | Hex | เหตุผล |
|---------|-----|--------|
| Void black bg | `#050813` | Vestige reference — deeper กว่า #02020f |
| Oracle pulsar | `#ffffff` core + `#00ffd4` corona | ยังเหมือนเดิม |
| Stars — ใหม่ (< 1d) | `#7ab8ff` (cool blue) | color temp = recency |
| Stars — recent (1-7d) | `#a78bfa` (violet) | mid-age |
| Stars — aging (7-30d) | `#ff9d7a` (warm orange) | older |
| Stars — dim (> 30d) | `#5a3a2a` (dim red ember) | fading memory |
| Constellation edges | `#5a4ff2` at 40% | violet glow = บุลมา palette |
| Nebula haze | `#ff7ad9` at 3-6% | nebula pink = บุลมา palette |
| Particle stars | `rgba(255,255,255,0.3-0.9)` random | background star field |
| Glass header | `rgba(5,8,19,0.7)` + backdrop-blur 20px | |

---

## Typography (updated)

```css
/* Headings / Oracle name / Labels */
font-family: 'Space Grotesk', sans-serif;
font-weight: 600;

/* Data / coordinates / memory IDs / token meter */
font-family: 'IBM Plex Mono', monospace;
font-weight: 400;
```

Google Fonts import:
```html
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;600&family=IBM+Plex+Mono&display=swap" rel="stylesheet">
```

---

## Memory Node Design System (from บุลมา)

```
size         = importance score (6-28px)
color temp   = recency (blue ใหม่ → dim red เก่า)
glow         = access frequency (high access = brighter halo)
halo thick   = connection count (many connections = thick ring)
opacity      = decay (freshness fade over time, min 20%)
cluster      = knowledge domain (constellation grouping)
```

### Glow Spec per tier

```css
/* High access, recent */
.node-active {
  filter: drop-shadow(0 0 8px #7ab8ff) drop-shadow(0 0 20px rgba(122,184,255,0.4));
}
/* Medium */
.node-recall {
  filter: drop-shadow(0 0 5px #a78bfa) drop-shadow(0 0 12px rgba(167,139,250,0.3));
}
/* Low access / fading */
.node-archival {
  filter: drop-shadow(0 0 3px #5a3a2a);
  opacity: 0.4;
}
```

---

## Constellation Clustering

Memory nodes จัด group ตาม type เป็น constellation clusters:

| Cluster / Constellation | Memory type | สี edges |
|------------------------|-------------|----------|
| The Oracle (center) | oracle node | `#00ffd4` |
| Andromeda | learnings | `#7ab8ff` |
| Cassiopeia | retrospectives | `#a78bfa` |
| Orion | feedback | `#5a4ff2` |
| Lyra | user memories | `#ff9d7a` |
| Vega | project memories | `#c4b5fd` |

Edges ภายใน cluster = solid lines
Edges ข้าม cluster = dashed, dim

---

## Z-Index Plan

| Layer | z-index | Element |
|-------|---------|---------|
| THREE.js canvas | 0 | full-screen 3D scene |
| HTML overlay — particles fallback | 2 | (ถ้า WebGL ไม่รองรับ) |
| Header | 30 | glass nav bar |
| Detail card | 40 | memory detail float |
| Tooltip | 45 | node hover |
| Warp overlay | 50 | enter transition flash |

---

## HTML Shell (React component structure)

```jsx
// BrainScreen.jsx
<div className="brain-screen">                {/* position: fixed, inset: 0 */}

  {/* Three.js Scene */}
  <Canvas className="cosmic-canvas">          {/* z: 0, fill viewport */}
    <NebulaBg />                              {/* three-nebula particles */}
    <StarField count={2000} />               {/* @react-three/drei Stars */}
    <ForceGraph3D                             {/* 3d-force-graph */}
      nodeVal={n => n.importance}
      nodeColor={n => recencyColor(n.age)}
      linkColor={() => '#5a4ff2'}
      linkOpacity={0.4}
      onNodeClick={handleNodeClick}
    />
    <OracleNode />                            {/* custom pulsar mesh */}
    <OrbitControls enableZoom={false}         {/* pan only — ห้าม zoom */}
                   enableRotate={false} />
  </Canvas>

  {/* HTML Overlays */}
  <header className="brain-header">           {/* z: 30 */}
    <OracleIdentity />
    <TokenMeter value={68} />
    <TierFilters />
  </header>

  <DetailCard node={selectedNode} />          {/* z: 40 */}
</div>
```

---

## Animation Specs

| Animation | Element | Duration | Easing | Trigger |
|-----------|---------|----------|--------|---------|
| Warp enter | full screen flash white→dark | 800ms | ease-out | page load |
| Nebula drift | particle bg | continuous | perlin noise | always |
| Star twinkle | bg stars | 2-8s random | ease-in-out | always |
| Node float | memory nodes | 4-12s random | ease-in-out alternate | always |
| Node breathe | glow pulse | 3-6s random | ease-in-out | always |
| Pulse on access | node glow flare | 300ms flash | ease-out | on read/write event |
| Orbit spin | constellation cluster | 120s | linear infinite | always |
| Node appear | scale 0→1 + opacity | 600ms | spring cubic-bezier | on new memory |
| Beam draw | constellation edge | 500ms strokeDash | ease | on new connection |
| Detail slide-up | card translateY 20→0 | 250ms | ease-out | click node |

---

## Fit-to-Screen Strategy

THREE.js canvas = `position: fixed; inset: 0` → always full viewport
`OrbitControls` — pan OK, **zoom disabled** (`enableZoom: false`)
Camera z จะ set ให้เห็น constellation ทั้งหมดพอดี auto (fitView บน graph data)
Header = `position: fixed; top: 0` overlay บน canvas

ผล: ไม่มี scroll, ไม่มี zoom, responsive ทุกขนาดจอโดย THREE.js จัดการ viewport เอง

---

## Fallback (no WebGL)

ถ้า `WebGLRenderer` ไม่รองรับ → แสดง 2D CSS version (v2 draft approach)
detect: `THREE.WebGLRenderer` try-catch → set `webglSupported` flag

---

*วีส — Angel of Precision 🟦 | Issue #74 v2-final | 2026-03-19*
*Research credit: บุลมา (อ้ายฮู้) — Vestige ref + three-nebula + Space Grotesk*
*Nothing is Deleted: v1 (radial) + v2 draft archived, ไม่ถูกลบ*
