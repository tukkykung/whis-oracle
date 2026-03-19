# Design Spec — Brain Screen v1
**Issue**: #74 | **วีส** | **Date**: 2026-03-19
**Status**: Draft — รอ เบจิต้า approve concept

---

## Concept ที่เลือก: Memory Map (Radial) + 3-Panel Layout

เหตุผล: Oracle เป็นศูนย์กลาง, radial ทำให้ fit-to-screen ง่ายที่สุด, อ่านง่าย ไม่มี edge clutter

---

## Layout Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER (60px)                                                   │
│  🟦 [Oracle Name]  ●  [Status: Active]  ▓▓▓▓░░ Token 68%       │
├────────────────┬────────────────────────────┬───────────────────┤
│ MEMORY LAYERS  │                            │ DETAIL PANEL      │
│ (280px)        │    GRAPH CANVAS            │ (320px)           │
│                │    (flex remaining)        │                   │
│ ● CORE         │                            │ [Node selected]   │
│ ● RECALL       │     [Radial Map]           │ Type: learning    │
│ ● ARCHIVAL     │                            │ Date: 2026-03-19  │
│                │                            │ Content preview   │
│ [Legend]       │                            │                   │
└────────────────┴────────────────────────────┴───────────────────┘
```

**Viewport**: 100vw × 100vh — ห้าม scroll ทุก panel

---

## Color System

| Element | Hex | ใช้กับ |
|---------|-----|--------|
| Background | `#080810` | page bg |
| Surface | `#0d0d1a` | panel bg |
| Border | `#1a1a2e` | panel border |
| Active / CORE | `#00ffd4` | oracle node, active memory |
| RECALL | `#3b82f6` | recent memory nodes |
| ARCHIVAL | `#8b5cf6` | old memory nodes |
| Text primary | `#e2e8f0` | headings |
| Text muted | `#64748b` | metadata |
| Token meter | `#00ffd4` → `#f59e0b` → `#ef4444` | gradient by % |

---

## Typography

- Headings: **Inter** 600
- Code/IDs: **JetBrains Mono** 400
- Body: Inter 400

---

## Radial Graph Spec (Graph Canvas)

### Node System

| Node Type | Size | Color | Label |
|-----------|------|-------|-------|
| Oracle (center) | 48px diameter | `#00ffd4` glow | Oracle name |
| CORE memory | 20px | `#00ffd4` 100% opacity | type tag |
| RECALL memory | 14px | `#3b82f6` 80% opacity | type tag |
| ARCHIVAL memory | 10px | `#8b5cf6` 40% opacity | — |

### Edge System

- Color: `rgba(0, 255, 212, 0.15)` — subtle teal
- Width: 1px (CORE), 0.5px (RECALL/ARCHIVAL)
- Style: curved bezier

### Radial Layout (Cytoscape.js — `cose-bilkent` layout)

```js
{
  name: 'cose-bilkent',
  idealEdgeLength: 80,
  nodeRepulsion: 8000,
  gravity: 0.4,
  fit: true,          // ← fit-to-screen key
  padding: 40
}
```

---

## Z-Index Plan

| Layer | z-index | Element |
|-------|---------|---------|
| Background | 0 | page bg, subtle grid |
| Graph canvas | 10 | Cytoscape container |
| Panels | 20 | Left + Right panels |
| Header | 30 | Top bar |
| Tooltip | 40 | Node hover tooltip |
| Modal | 50 | Memory detail expand |

---

## 3 States / Mockups

### State 1: Idle
- Oracle node: pulse animation `0 0 0 8px rgba(0,255,212,0.1)` → fade → repeat 3s
- Nodes: breathing opacity (95% → 80% → 95%) 4s ease-in-out
- No edge animation

### State 2: Active Research
- Oracle node: faster pulse 1.5s, brighter glow
- New nodes appear: `scale(0) → scale(1)` 300ms ease-out
- Edges draw in: strokeDashoffset animation 500ms
- RECALL nodes float upward 2px subtle

### State 3: Memory Inspector (click node)
- Clicked node: scale 1.4x, ring highlight `2px solid #00ffd4`
- Right panel slides in: `translateX(320px) → translateX(0)` 250ms ease-out
- Other nodes: opacity 40%
- Edges from selected node: highlight full opacity

---

## HTML Structure

```html
<div class="brain-screen">                         <!-- 100vw 100vh, overflow: hidden -->
  <header class="brain-header">                    <!-- h: 60px, z-index: 30 -->
    <div class="oracle-identity">
      <span class="oracle-dot"></span>              <!-- 8px circle, #00ffd4 pulse -->
      <span class="oracle-name">วีส</span>
      <span class="oracle-status">Active</span>
    </div>
    <div class="token-meter">                       <!-- width: 180px -->
      <div class="token-bar"></div>                 <!-- width: var(--token-pct) -->
      <span class="token-label">Token 68%</span>
    </div>
  </header>

  <div class="brain-body">                          <!-- display: flex, h: calc(100vh - 60px) -->
    <aside class="memory-layers">                   <!-- w: 280px, overflow-y: auto -->
      <div class="layer-group" data-tier="core">...</div>
      <div class="layer-group" data-tier="recall">...</div>
      <div class="layer-group" data-tier="archival">...</div>
    </aside>

    <main class="graph-canvas">                     <!-- flex: 1 -->
      <div id="cy"></div>                           <!-- Cytoscape mount, w: 100%, h: 100% -->
    </main>

    <aside class="detail-panel">                    <!-- w: 320px, hidden by default -->
      <div class="node-detail">...</div>
    </aside>
  </div>
</div>
```

---

## CSS Variables (Design Tokens)

```css
:root {
  --bg-page:      #080810;
  --bg-surface:   #0d0d1a;
  --bg-border:    #1a1a2e;
  --color-core:   #00ffd4;
  --color-recall: #3b82f6;
  --color-archive:#8b5cf6;
  --text-primary: #e2e8f0;
  --text-muted:   #64748b;
  --header-h:     60px;
  --panel-left:   280px;
  --panel-right:  320px;
}
```

---

## Animation Specs

| Animation | Property | Duration | Easing | Trigger |
|-----------|----------|----------|--------|---------|
| Oracle pulse (idle) | box-shadow | 3s | ease-in-out | always |
| Node appear | transform scale | 300ms | ease-out | on data load |
| Edge draw | strokeDashoffset | 500ms | ease | on data load |
| Detail panel slide | transform X | 250ms | ease-out | click node |
| Token meter fill | width | 1s | ease | on mount |

---

## Library

**Cytoscape.js** (MIT)
- `cytoscape-cose-bilkent` — force-directed layout
- Built-in event: `cy.on('tap', 'node', fn)` สำหรับ Memory Inspector state

---

*วีส — Angel of Precision 🟦 | Issue #74 | 2026-03-19*
