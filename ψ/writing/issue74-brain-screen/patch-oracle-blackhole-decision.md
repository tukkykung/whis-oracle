# Design Decision — Black Hole Implementation
**Issue**: #74 | **วีส** | **Date**: 2026-03-19
**Input**: บุลมา research — 5 repos + 2 Shadertoy

---

## Decision: oseiskar/black-hole เป็น base + sirxemic เป็น visual reference

### เหตุผล

| Criteria | sirxemic | oseiskar | vlwkaos | josetxu |
|----------|----------|----------|---------|---------|
| License | MIT ✅ | MIT ✅ | GPL v3 ⚠️ | CodePen |
| Stack fit (R3F) | ❌ Pure WebGL | ✅ Three.js+GLSL | ✅ Three.js | ✅ CSS |
| Visual quality | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Integration effort | สูง (port WebGL→R3F) | ต่ำ (Three.js ตรง) | ปานกลาง | ต่ำมาก |
| Physics accuracy | wormhole+disk | Doppler+lensing ✅ | bloom only | ไม่มี |

**oseiskar** = Three.js + GLSL + MIT + Doppler shift + Schwarzschild geodesics
ตรงกับ Gargantua spec เราพอดี (Doppler asymmetry สว่างบน/มืดล่าง)
integrate กับ R3F ได้โดยใช้ `<shaderMaterial>` ตรงๆ

**sirxemic** = ใช้เป็น visual reference เท่านั้น — ดู GLSL shader logic แล้ว adapt
ไม่ fork ตรงๆ เพราะ pure WebGL ต้องเขียน R3F wrapper ยาว

**josetxu CSS** = fallback เมื่อ WebGL ไม่รองรับ (แทน patch CSS ที่วีสเขียนไว้)
ใช้แทน patch-oracle-node-gargantua.md ได้เลย — visual ดีกว่า spec วีสเขียนเอง

---

## Implementation Plan สำหรับโกคู

### Step 1: Fork oseiskar
```bash
git clone https://github.com/oseiskar/black-hole
# ดู src/shaders/ — gravitational_lensing.glsl, doppler.glsl
```

### Step 2: Extract shaders เข้า OracleNode component

```jsx
// OracleNode.jsx (R3F)
import { useRef } from 'react'
import { useFrame } from '@react-three/fiber'
import blackHoleVert from './shaders/blackhole.vert'
import blackHoleFrag from './shaders/blackhole.frag'  // ported from oseiskar

export function OracleNode({ position }) {
  const meshRef = useRef()

  useFrame(({ clock }) => {
    meshRef.current.material.uniforms.uTime.value = clock.getElapsedTime()
  })

  return (
    <mesh ref={meshRef} position={position}>
      <planeGeometry args={[2, 2]} />        {/* fullscreen quad for shader */}
      <shaderMaterial
        vertexShader={blackHoleVert}
        fragmentShader={blackHoleFrag}
        uniforms={{
          uTime:          { value: 0 },
          uAccretionColor:{ value: [1.0, 0.78, 0.31] },   /* #ffc84f warm */
          uDopplerShift:  { value: true },
          uLensStrength:  { value: 0.35 },
        }}
        transparent
      />
    </mesh>
  )
}
```

### Step 3: GLSL uniforms ที่ต้องปรับจาก oseiskar → Gargantua spec วีส

```glsl
// ปรับใน blackhole.frag (ported จาก oseiskar)

// Doppler asymmetry: บนสว่าง (blueshift), ล่างมืด (redshift)
// oseiskar มีอยู่แล้ว — แค่ปรับ disk_tilt = 72.0 degrees

uniform float uDiskTilt;    // 72.0 — Gargantua tilt
uniform float uLensStrength; // 0.35 — ความแรง gravitational lensing
uniform vec3  uAccretionColor; // warm amber #ffc84f

// Corona top — เพิ่ม fragment ด้านบน disk
float corona = smoothstep(0.6, 1.0, dot(normal, vec3(0.0, 1.0, 0.0)));
gl_FragColor.rgb += uAccretionColor * corona * 0.8;
```

### Step 4: Accretion disk animation (oseiskar มีให้แล้ว)

oseiskar compute Keplerian orbital velocity ให้เลย — disk หมุนตาม physics จริง
ไม่ต้องเขียน CSS animation `disk-spin` แล้ว shader จัดการ

### Step 5: Infalling particles → three-nebula

แทนที่ CSS `.infalling-particle` ด้วย three-nebula emitter:

```js
import { GPUParticleSystem } from 'three-nebula'

const emitter = new GPUParticleSystem({
  maxParticles: 200,
  containerCount: 1,
})
// spawn ที่ orbit radius ~60px, velocity toward (0,0,0)
// lifespan 2.5s, color: #ffc84f → transparent
// spiral: angular velocity + inward radial velocity
```

---

## CSS Fallback (WebGL ไม่รองรับ)

ใช้ **josetxu v2** (codepen.io/josetxu/pen/rNWgNeq) แทน patch CSS ที่วีสเขียนเอง
visual ดีกว่า patch เดิม — ไม่ต้องใช้ patch-oracle-node-gargantua.md

```js
// detect WebGL
const canvas = document.createElement('canvas')
const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl')
if (!gl) {
  // render <OracleNodeCSS /> แทน — josetxu CSS approach
}
```

---

## สรุปไฟล์ที่โกคูต้องทำ

| ไฟล์ | Action |
|------|--------|
| `src/components/OracleNode.jsx` | เขียนใหม่ — R3F + shaderMaterial |
| `src/shaders/blackhole.vert` | port จาก oseiskar |
| `src/shaders/blackhole.frag` | port + ปรับ uniforms ตาม spec |
| `src/components/OracleNodeCSS.jsx` | fallback — josetxu approach |
| `package.json` | เพิ่ม `three-nebula` สำหรับ infalling particles |

**ไม่ต้อง** patch-oracle-node-gargantua.md แล้ว — shader แทนทั้งหมด

---

*วีส — Angel of Precision 🟦 | Issue #74 | 2026-03-19*
*Research credit: บุลมา (อ้ายฮู้) round 3*
