# Design Spec — Brain Screen v3: JARVIS Cosmic
**Issue**: #74 | **วีส** | **Date**: 2026-03-19
**Status**: Concept v3 — รอ เบจิต้า approve ก่อน implement

---

## Concept: Cosmic Space + JARVIS HUD

ไม่ต้องเลือก — สองแนวนี้ merge ได้สมบูรณ์:

- **Background + Oracle node** = Cosmic (Gargantua black hole, deep space, nebula) — ตาม boss spec
- **Memory nodes** = Holographic JARVIS spheres — ใช้ threejs-holographic-material
- **Layout engine** = 3d-force-graph — physics-based auto-arrange
- **HUD overlay** = JARVIS scanlines + data readouts (CSS layer บน Three.js canvas)
- **Glow** = UnrealBloomPass post-processing

ผล: Oracle เป็น Black Hole อยู่ตรงกลาง, memory nodes เป็น holographic data crystals โคจรรอบ
เหมือน JARVIS กำลัง visualize brain ของ Oracle ผ่านกล้อง deep space

---

## Stack

| Layer | Library | ใช้ทำ |
|-------|---------|-------|
| 1 | **3d-force-graph** (MIT, 5k+ ⭐) | brain layout — physics auto-arrange |
| 2 | **threejs-holographic-material** (MIT, 183 ⭐) | JARVIS skin บน memory nodes |
| 3 | **oseiskar/black-hole** GLSL | Oracle node — Gargantua shader |
| 4 | **three-nebula** | deep space particle bg + nebula |
| 5 | **UnrealBloomPass** (@react-three/postprocessing) | neon glow ทุก element |
| 6 | CSS HUD overlay | scanlines, data readouts, header |

---

## Color System (Cosmic + JARVIS merged)

| Element | Hex | ที่มา |
|---------|-----|------|
| Void black bg | `#000510` | JARVIS palette (เข้มกว่า Cosmic v2) |
| Oracle corona | `#00ffd4` → `#00D4FF` | เปลี่ยน teal → JARVIS cyan |
| Memory nodes — active | `#00FFFF` | JARVIS active cyan |
| Memory nodes — recall | `#0088CC` | JARVIS edge blue |
| Memory nodes — archival | `#003355` | dim JARVIS |
| Holographic shimmer | `#00D4FF` scanline | threejs-holographic-material default |
| Nebula haze | `#0044aa` at 4% | blue-tinted nebula (ไม่ pink อีกแล้ว) |
| Bloom glow | `#00FFFF` | UnrealBloomPass color |
| HUD text | `#00D4FF` | Orbitron font |
| HUD scanline | `rgba(0,212,255,0.03)` | CSS overlay |

---

## Typography (updated)

```css
/* HUD / headers / Oracle name */
font-family: 'Orbitron', sans-serif;    /* JARVIS feel */
font-weight: 600;

/* Data values / coordinates / memory IDs */
font-family: 'IBM Plex Mono', monospace;

/* Body labels */
font-family: 'Space Grotesk', sans-serif;
```

```html
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;900&family=IBM+Plex+Mono&family=Space+Grotesk:wght@400;600&display=swap" rel="stylesheet">
```

---

## Layout Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ HUD HEADER (60px) — Orbitron font, scanline texture             │
│  [▣] ORACLE SYSTEM  ●ONLINE  [████░░] TOKEN 68%  [CORE][RCLL]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   THREE.js CANVAS (position:fixed, inset:0)                     │
│                                                                 │
│   ·  ✦  ·   [◈ holographic node]   ·      ✦       ·            │
│      ·         \  constellation  /       ·                      │
│  ✦    [◈]─ ─ ─[◈]─ ─[☯ ORACLE]─ ─[◈]─ ─ ─[◈]     ✦          │
│      ·         /   Gargantua BH   \      ·                      │
│   ·  ·    [◈ holographic node]   ✦     [◈]     ·               │
│      nebula haze (blue-tinted)                                  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ CSS HUD SCANLINES OVERLAY (position:fixed, pointer-none)│   │
│  │ subtle horizontal lines + corner brackets               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                            ┌────────────────┐  │
│                                            │ DETAIL CARD    │  │
│                                            │ Orbitron style │  │
│                                            └────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Memory Node Design (Holographic Sphere)

```jsx
import { HolographicMaterial } from 'threejs-holographic-material'

function MemoryNode({ node }) {
  return (
    <mesh>
      <sphereGeometry args={[nodeSize(node.importance), 16, 16]} />
      <HolographicMaterial
        fresnelOpacity={0.8}
        fresnelAmount={0.6}
        scanlineSize={8}
        signalSpeed={0.5}
        holographicColor={recencyColor(node.age)}  // cyan→blue→dim
        enableBlinking={node.tier === 'archival'}   // เก่า = blink
        blinkFresnelOnly={true}
        side={THREE.FrontSide}
      />
    </mesh>
  )
}
```

### Node size / color by tier

| Tier | Size | HolographicColor | Blink |
|------|------|-----------------|-------|
| CORE (< 1d) | 0.8-1.2 | `#00FFFF` | ไม่ |
| RECALL (1-7d) | 0.5-0.8 | `#0088CC` | ไม่ |
| ARCHIVAL (> 7d) | 0.3-0.5 | `#003355` | ✅ slow |

---

## 3d-force-graph Config

```jsx
import ForceGraph3D from '3d-force-graph'

const Graph = ForceGraph3D()
  .graphData(memoryGraphData)
  .backgroundColor('#000510')
  .nodeThreeObject(node => {
    // ← return holographic mesh
    return <MemoryNode node={node} />
  })
  .linkColor(() => '#0088CC')
  .linkOpacity(0.4)
  .linkWidth(0.5)
  .enableNodeDrag(false)      // ไม่ drag — fit-to-screen
  .enableNavigationControls(false)  // ห้าม zoom/pan
  .d3Force('charge', d3.forceManyBody().strength(-80))
  .d3Force('link', d3.forceLink().distance(60))
```

---

## CSS HUD Overlay

```css
/* Scanlines — วางทับ canvas ทั้งจอ, pointer-events:none */
.hud-scanlines {
  position: fixed;
  inset: 0;
  z-index: 5;
  pointer-events: none;
  background: repeating-linear-gradient(
    to bottom,
    transparent 0px,
    transparent 3px,
    rgba(0, 212, 255, 0.025) 3px,
    rgba(0, 212, 255, 0.025) 4px
  );
}

/* Corner brackets — JARVIS frame */
.hud-corner {
  position: fixed;
  width: 40px;
  height: 40px;
  z-index: 6;
  pointer-events: none;
}
.hud-corner::before,
.hud-corner::after {
  content: '';
  position: absolute;
  background: #00D4FF;
}
.hud-corner::before { width: 2px; height: 20px; }
.hud-corner::after  { width: 20px; height: 2px; }

.hud-corner.tl { top: 8px; left: 8px; }
.hud-corner.tl::before { top: 0; left: 0; }
.hud-corner.tl::after  { top: 0; left: 0; }

.hud-corner.tr { top: 8px; right: 8px; transform: scaleX(-1); }
.hud-corner.bl { bottom: 8px; left: 8px; transform: scaleY(-1); }
.hud-corner.br { bottom: 8px; right: 8px; transform: scale(-1); }

/* Detail card — JARVIS style */
.detail-card {
  background: rgba(0, 8, 24, 0.85);
  border: 1px solid rgba(0, 212, 255, 0.4);
  border-radius: 4px;
  box-shadow: 0 0 20px rgba(0, 212, 255, 0.15),
              inset 0 0 20px rgba(0, 212, 255, 0.05);
  font-family: 'Orbitron', sans-serif;
  color: #00D4FF;
}
.detail-card::before {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    to bottom,
    transparent 0px, transparent 3px,
    rgba(0,212,255,0.02) 3px, rgba(0,212,255,0.02) 4px
  );
  border-radius: 4px;
  pointer-events: none;
}
```

---

## Post-Processing (Bloom)

```jsx
import { EffectComposer, Bloom } from '@react-three/postprocessing'

<EffectComposer>
  <Bloom
    intensity={1.2}
    luminanceThreshold={0.1}   /* glow ทุก element สว่างกว่า threshold */
    luminanceSmoothing={0.9}
    mipmapBlur
  />
</EffectComposer>
```

---

## Z-Index Plan

| Layer | z-index | Element |
|-------|---------|---------|
| THREE.js canvas | 0 | full-screen scene |
| CSS scanlines overlay | 5 | pointer-events:none |
| CSS corner brackets | 6 | pointer-events:none |
| Header | 30 | HUD top bar |
| Detail card | 40 | memory detail |
| Tooltip | 45 | node hover |

---

## Animation Additions (JARVIS)

| Animation | Element | ลักษณะ |
|-----------|---------|--------|
| holographic-blink | archival nodes | HolographicMaterial built-in |
| holographic-scanline | all nodes | HolographicMaterial built-in |
| data-scroll | detail card text | text scrolls up 20px on appear |
| corner-flicker | HUD corners | opacity 1→0.6→1, 4s random |
| header-glitch | Oracle name | skewX(2deg) flash 50ms, rare (30s interval) |
| bloom-pulse | on memory access | Bloom intensity spike 1.2→2.5 0.3s |

(+ animations ทั้งหมดจาก Cosmic v2 ยังคงอยู่)

---

## Files สำหรับโกคู (เพิ่มจาก v2)

| ไฟล์ | Action |
|------|--------|
| `package.json` | เพิ่ม `threejs-holographic-material`, `3d-force-graph`, `@react-three/postprocessing` |
| `src/components/MemoryNode.jsx` | holographic sphere |
| `src/components/HUDOverlay.jsx` | scanlines + corners + header |
| `src/components/DetailCard.jsx` | JARVIS-styled card |
| `src/components/OracleNode.jsx` | oseiskar Gargantua shader (ไม่เปลี่ยน) |
| `src/BrainScreen.jsx` | wire ทุกอย่างเข้า 3d-force-graph |

---

## Concept Comparison (ให้ เบจิต้า เลือก)

| | v2 Cosmic | **v3 JARVIS Cosmic** |
|-|-----------|---------------------|
| Visual style | Dark space, organic | Dark space + HUD overlay |
| Memory nodes | Custom CSS stars | Holographic JARVIS spheres |
| Layout | Custom concentric JS | 3d-force-graph physics |
| Post-processing | ไม่มี | UnrealBloomPass |
| HUD | Glass header only | Scanlines + corner brackets |
| "มีชีวิต" level | ★★★★ | ★★★★★ |
| Dev complexity | medium | medium-high |

วีสแนะนำ **v3** — Gargantua Oracle ยังอยู่ครบ แต่ memory nodes กลายเป็น JARVIS holographic data crystals ทำให้รู้สึกเหมือน AI กำลัง visualize brain ตัวเองจริงๆ

---

*วีส — Angel of Precision 🟦 | Issue #74 v3 | 2026-03-19*
*Research credit: บุลมา (อ้ายฮู้) rounds 1-4*
*Nothing is Deleted: v1(radial), v2-draft, v2-cosmic-final, Gargantua patch — archived ทั้งหมด*
