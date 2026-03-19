# Visual Spec — Organic Neural Sphere
**Issue**: #74 | **PR**: #79 (redesign) | **วีส** | **Date**: 2026-03-19
**Problem**: Fibonacci lattice → geometric, uniform rotation, uniform firing → ไม่มีชีวิต
**Goal**: สมองจริงๆ — organic structure, cascade firing, depth layers, JARVIS amber/gold

---

## 1. Color Palette

| Element | Hex | Opacity | หมายเหตุ |
|---------|-----|---------|----------|
| Background | `#050208` | 100% | ดำ warm-tinted (ไม่ cool blue) |
| Node — hub (core) | `#FFB300` | 100% | amber อบอุ่น, สว่างสุด |
| Node — interneuron | `#FF8C00` | 90% | amber mid |
| Node — peripheral | `#CC5500` | 70% | amber dim |
| Node — firing flash | `#FFFFFF` → `#FFD700` | 100%→0% | white flash decay to gold |
| Node — refractory | `#3D1F00` | 50% | หลัง fire — dim dark amber |
| Edge — passive | `#FF8C00` | 5% | แทบมองไม่เห็น |
| Edge — active | `#FFD700` | 60%→0% | สว่างแล้ว fade |
| Edge — cascade | `#FFFFFF` | 80%→0% | white beam วิ่งตาม edge |
| Bloom | `#FF8C00` | — | UnrealBloomPass color tint |
| Background stars | `#FF8C00` | 15-40% | warm amber star field |
| Point light | `#FF9500` | — | at origin, pulsing |

**หลักการ**: Cold edges (passive) → Warm edges (active) → White flash (firing peak)
ไม่ใช้ cyan/blue เลย — ทุกอย่าง amber/gold/white เท่านั้น เพื่อให้ coherent กับ JARVIS brain

---

## 2. Structure — Organic Node Layout

### ปัญหาของ Fibonacci Lattice
Fibonacci lattice กระจาย node สม่ำเสมอ 100% → ดูเหมือน golf ball ไม่ใช่สมอง

### แก้: Noise-Displaced Sphere + Regional Clustering

```js
// แทนที่ Fibonacci lattice ด้วย:
function organicBrainLayout(nodeCount) {
  const positions = []

  // 1. เริ่มจาก Fibonacci base (ยังใช้อยู่)
  const base = fibonacciSphere(nodeCount)

  // 2. Displace ด้วย 3D Simplex Noise
  base.forEach(pos => {
    const noise = simplex3(pos.x * 1.5, pos.y * 1.5, pos.z * 1.5)
    const displacement = noise * 0.25  // ±25% of radius
    positions.push(pos.multiplyScalar(1 + displacement))
  })

  // 3. Regional bias — สมองมี lobe ที่ dense กว่ากัน
  // ด้านบน (y > 0.5r) = frontal lobe: node dense ขึ้น 30%
  // ด้านหลัง (z < -0.5r) = occipital: node เบาบาง
  // กลาง = limbic: node ใหญ่ขึ้น (hub neurons)

  return positions
}
```

### Node Tiers (3 ชั้นตาม depth)

```
CORE (r = 0-60px)     — hub neurons, ขนาดใหญ่ 6-10px, เชื่อมต่อมาก, active ตลอด
MID  (r = 60-120px)   — interneurons, ขนาด 3-5px, fire บ่อย
OUTER (r = 120-160px) — peripheral, ขนาด 1-3px, fire นานๆ ครั้ง
```

### Node Sizes (varied, ไม่ uniform)

```js
// ขนาด node = log(connectionCount) * baseSizeForTier
// hub neuron (connections > 8) → size 8-12px
// interneuron (connections 3-7) → size 3-6px
// peripheral (connections 1-2) → size 1-3px
// ผล: สมองมีดาวใหญ่-เล็กปน ไม่ใช่ทุกจุดเท่ากัน
```

---

## 3. Animation Behavior — Neural Firing

### ปัญหาของ PR #79
- Particle sparks travel t=0→1 ทุก edge พร้อมกัน → loop สม่ำเสมอ ดูเหมือน machine ไม่ใช่ brain
- Rotation 0.12 rad/s ตายตัว → mechanical

### แก้: Cascade Firing System

#### State Machine ต่อ Node

```
RESTING → FIRING → PEAK → REFRACTORY → RESTING
  90%      10ms    50ms      200ms
```

```js
class Neuron {
  constructor() {
    this.state = 'resting'  // resting | firing | peak | refractory
    this.energy = 0         // 0-1
    this.lastFired = -Infinity
    this.threshold = 0.6 + Math.random() * 0.3  // ต่างกันแต่ละ node
    this.refractoryPeriod = 200 + Math.random() * 300  // 200-500ms
  }

  // รับ signal จาก neighbor
  receivePulse(strength) {
    if (this.state === 'refractory') return  // กำลัง recover ไม่รับ signal
    this.energy = Math.min(1, this.energy + strength)
    if (this.energy >= this.threshold) this.fire()
  }

  fire() {
    this.state = 'firing'
    // ส่ง signal ต่อไป neighbor ด้วย delay
    this.neighbors.forEach((n, i) => {
      setTimeout(() => n.receivePulse(0.4 + Math.random() * 0.3),
                 i * 15 + Math.random() * 30)  // stagger delay
    })
    setTimeout(() => { this.state = 'refractory'; this.energy = 0 },
               this.refractoryPeriod)
    setTimeout(() => { this.state = 'resting' },
               this.refractoryPeriod + 100)
  }
}
```

#### Spontaneous Firing (Theta Rhythm)

สมองไม่เงียบสนิท — มี spontaneous firing ตลอดเวลา

```js
// Theta wave: burst ทุก ~125ms (8Hz)
// แต่ randomize เพื่อไม่ให้ดู mechanical
function thetaBurst() {
  const burstSize = 1 + Math.floor(Math.random() * 3)  // 1-3 neurons fire
  const candidates = neurons.filter(n => n.state === 'resting')
  // bias ต่อ hub neurons (fire บ่อยกว่า)
  const weighted = candidates.sort(() => Math.random() - 0.5)
  weighted.slice(0, burstSize).forEach(n => n.fire())

  // ช่วงถัดไป: random 100-200ms (ไม่ใช่ 125ms ตายตัว)
  setTimeout(thetaBurst, 100 + Math.random() * 100)
}
```

#### Visual Response ต่อ State

```js
// ใน animation loop
neurons.forEach(neuron => {
  const mesh = neuron.mesh
  switch(neuron.state) {
    case 'resting':
      mesh.material.color.set(neuron.baseColor)  // amber dim
      mesh.scale.setScalar(neuron.baseSize)
      break
    case 'firing':
      mesh.material.color.set('#FFFFFF')          // white flash
      mesh.scale.setScalar(neuron.baseSize * 2.5) // scale up
      break
    case 'peak':
      // decay from white → gold
      mesh.material.color.lerpColors(white, gold, peakProgress)
      mesh.scale.setScalar(neuron.baseSize * (1 + (1-peakProgress) * 1.5))
      break
    case 'refractory':
      mesh.material.color.set('#3D1F00')          // dark amber
      mesh.scale.setScalar(neuron.baseSize * 0.8) // slightly smaller
      break
  }
})
```

---

## 4. Edge Animation — Active Pulse (แทน Particle Spark)

### ปัญหาของ PR #79
Particle sparks (t=0→1) ทุก edge → uniform → ดูเหมือน LED matrix

### แก้: Signal Pulse บน Edge เฉพาะที่ active

```js
// แสดง edge เมื่อมี signal เท่านั้น
// Passive edges: opacity 3-5% (แทบมองไม่เห็น background structure)
// Active edge: animate pulse วิ่งจาก source → target

class EdgePulse {
  constructor(source, target) {
    this.progress = 0      // 0 = source, 1 = target
    this.speed = 0.8 + Math.random() * 0.4  // ความเร็วต่างกัน
    this.active = true
  }

  update(delta) {
    this.progress += delta * this.speed
    if (this.progress >= 1) {
      this.active = false
      this.target.receivePulse(0.35)  // deliver signal เมื่อถึงปลายทาง
    }
  }
}

// Render: วาด gradient line จาก source → current position
// สีที่ head: white → gold → transparent
// ความยาว tail: ~20% of edge length
```

---

## 5. Rotation — Organic, ไม่ Mechanical

### ปัญหา: 0.12 rad/s ตายตัว = clock hand
### แก้: Perlin noise rotation

```js
let rotX = 0, rotY = 0
let velX = 0, velY = 0
let noiseT = 0

function updateRotation(delta) {
  noiseT += delta * 0.05

  // target velocity จาก noise (ช้า-เร็ว-หยุด-กลับ)
  const targetVelY = simplex2(noiseT, 0) * 0.08          // max 0.08 rad/s
  const targetVelX = simplex2(noiseT + 100, 0) * 0.03   // x-axis หมุนน้อยกว่า

  // smooth lerp ไปหา target (เหมือน brain drift ไม่ใช่ motor)
  velY = lerp(velY, targetVelY, delta * 0.5)
  velX = lerp(velX, targetVelX, delta * 0.5)

  rotY += velY * delta
  rotX += velX * delta

  sphere.rotation.set(rotX, rotY, 0)
}
```

ผล: หมุนช้าๆ แล้วหยุด แล้วเปลี่ยนทิศ แล้วเร็วขึ้น — เหมือน brain floating in liquid

---

## 6. Depth & Layering

### Visual depth โดยไม่ต้องเพิ่ม complexity

```js
// Node ที่อยู่ไกลกว่า (ฝั่งหลัง sphere) → dim ลง
// Node ที่หันหน้ามาหา camera → สว่าง
const cameraDir = camera.position.clone().normalize()
nodes.forEach(node => {
  const nodeFacing = node.position.clone().normalize()
  const dot = nodeFacing.dot(cameraDir)  // -1 (behind) → 1 (front)
  const depthFactor = (dot + 1) / 2      // 0 → 1

  // mix base opacity กับ depth
  node.material.opacity = node.baseOpacity * (0.3 + depthFactor * 0.7)
})

// Edge ที่ cross จาก back → front ให้ fade ตาม depth ด้วย
```

### Layer ที่มองเห็นได้ชัด

```
Front face: nodes สว่าง 100%, edges เห็นชัด
Mid sphere: nodes 60%, edges 희微
Back face: nodes 20-30%, edges 5% (silhouette)
Core (inside): hub neurons สว่างกว่า peripheral เสมอ
```

---

## 7. JARVIS Feel — ความแตกต่างจาก Generic Neural Viz

| Generic | JARVIS Brain |
|---------|-------------|
| ทุก node ขนาดเท่ากัน | Hub neurons ใหญ่กว่าชัดเจน |
| Fire สม่ำเสมอ | Cascade burst + quiet period |
| Edge ทุกเส้นเท่ากัน | Active edge เท่านั้นที่เห็นชัด |
| Rotation ตายตัว | Organic drift |
| Node color ตาม category | Color ตาม state (firing/resting/refractory) |
| Perfect sphere | Noise-displaced, organic surface |

### JARVIS Brain Signature Effects

1. **Firing cascade visible**: เห็น signal วิ่งจาก hub → neighbor → neighbor ชัดเจน
2. **Activity hotspot**: มี region ที่ active มากกว่า region อื่น (เหมือน brain processing)
3. **Quiet → burst → quiet**: มีช่วงเงียบ แล้ว cascade ระเบิดออก แล้วเงียบอีก
4. **Hub neurons dominant**: 3-5 node ใหญ่เด่น ดึงสายตา (Degree centrality visible)

---

## 8. Implementation Notes สำหรับโกคู

### ไฟล์ที่ต้องแก้: `src/public/brain.html`

| Component | เปลี่ยนจาก | เปลี่ยนเป็น |
|-----------|-----------|------------|
| Node layout | `fibonacciSphere()` | `fibonacciSphere()` + simplex noise displacement |
| Node size | uniform per category | `log(connections) * tierBase` |
| Node color | static per category | dynamic per firing state |
| Firing | particle t=0→1 every edge | cascade state machine + spontaneous theta burst |
| Edge render | LineSegments all visible | passive 3% opacity + active pulse only |
| Rotation | `sphere.rotation.y += 0.12 * delta` | perlin noise velocity |
| Depth | ไม่มี | dot product camera → node opacity |

### Library เพิ่ม

```html
<!-- Simplex Noise (ไม่มี dependency ใหม่ใหญ่ — tiny lib) -->
<script src="https://cdn.jsdelivr.net/npm/simplex-noise@4/dist/cjs/simplex-noise.js"></script>
```

ทุกอย่างอื่นใช้ Three.js เดิมที่มีอยู่แล้ว — ไม่ต้องเพิ่ม library ใหญ่

---

## 9. Animation Timeline (1 รอบตัวอย่าง)

```
t=0s    sphere appear (scale 0→1, 800ms spring)
t=0.5s  first spontaneous fire — 1 hub neuron
t=0.6s  cascade: hub → 3 neighbors (15ms stagger)
t=0.7s  neighbors → their neighbors: 8 nodes active
t=0.9s  cascade fades, nodes enter refractory
t=1.0s  quiet period — only passive edges visible (3%)
t=1.1s  theta rhythm: 2 random neurons fire
...
t=3-5s  next big cascade (random interval 3-8s)
```

---

*วีส — Angel of Precision 🟦 | Issue #74 / PR #79 redesign | 2026-03-19*
