# Patch Spec — Oracle Node: Gargantua Black Hole
**Issue**: #74 | **วีส** | **Date**: 2026-03-19
**Target**: OracleNode component / `.oracle-node` element
**Type**: CSS animation patch — ไม่เปลี่ยน layout หรือ stack

Reference: Gargantua (Interstellar) — accretion disk, gravitational lensing, asymmetric corona

---

## HTML Structure (replace Oracle node inner HTML)

```html
<div class="oracle-node">

  <!-- Layer 1: Gravitational lensing ring — แสงโค้งรอบขอบ black hole -->
  <div class="grav-lens"></div>

  <!-- Layer 2: Accretion disk — จานหมุนรอบ, สว่างบน มืดล่าง -->
  <div class="accretion-disk">
    <div class="disk-inner"></div>
  </div>

  <!-- Layer 3: Black hole core — ความมืดสมบูรณ์ -->
  <div class="bh-core"></div>

  <!-- Layer 4: Corona top — สว่างด้านบน (Doppler blueshift) -->
  <div class="corona-top"></div>

  <!-- Layer 5: Particle stream — ถูกดูดเข้า -->
  <div class="particle-stream">
    <span class="infalling-particle" style="--angle:  20deg; --delay: 0s;    --r: 60px"></span>
    <span class="infalling-particle" style="--angle: 110deg; --delay: 0.4s;  --r: 52px"></span>
    <span class="infalling-particle" style="--angle: 200deg; --delay: 0.9s;  --r: 68px"></span>
    <span class="infalling-particle" style="--angle: 310deg; --delay: 1.5s;  --r: 48px"></span>
    <span class="infalling-particle" style="--angle:  70deg; --delay: 2.1s;  --r: 72px"></span>
    <span class="infalling-particle" style="--angle: 260deg; --delay: 0.7s;  --r: 56px"></span>
  </div>

  <!-- Layer 6: Pulse ring — pulsing energy -->
  <div class="pulse-ring pulse-ring-1"></div>
  <div class="pulse-ring pulse-ring-2"></div>

</div>
```

---

## CSS

```css
/* ─── Oracle Node Container ─── */
.oracle-node {
  position: absolute;
  width: 120px;
  height: 120px;
  transform: translate(-50%, -50%);
  /* perspective สำหรับ disk tilt */
  perspective: 400px;
}

/* ─── Layer 1: Gravitational Lensing ─── */
/* วงแสงที่โค้งงอรอบขอบ black hole */
.grav-lens {
  position: absolute;
  inset: -12px;                        /* ใหญ่กว่า core 12px โดยรอบ */
  border-radius: 50%;
  background: transparent;
  box-shadow:
    0 0 0 2px rgba(255, 200, 100, 0.15),   /* photon ring ชั้น 1 */
    0 0 0 4px rgba(255, 180, 60, 0.08),    /* photon ring ชั้น 2 */
    0 0 30px 8px rgba(255, 160, 40, 0.12); /* diffuse lens glow */
  animation: lens-breathe 6s ease-in-out infinite;
}

@keyframes lens-breathe {
  0%, 100% {
    box-shadow:
      0 0 0 2px rgba(255,200,100,0.15),
      0 0 0 4px rgba(255,180,60,0.08),
      0 0 30px 8px rgba(255,160,40,0.12);
  }
  50% {
    box-shadow:
      0 0 0 3px rgba(255,220,120,0.25),
      0 0 0 6px rgba(255,190,80,0.12),
      0 0 50px 14px rgba(255,160,40,0.18);
  }
}

/* ─── Layer 2: Accretion Disk ─── */
/* จานหมุน — tilt perspective, สว่างบน มืดล่าง */
.accretion-disk {
  position: absolute;
  inset: -20px;
  border-radius: 50%;
  /* Tilt เหมือน Gargantua — disk เอียง ~15deg */
  transform: rotateX(72deg) rotateZ(0deg);
  animation: disk-spin 8s linear infinite;
  /* สว่างบน (Doppler blueshift) มืดล่าง (redshift) */
  background: conic-gradient(
    from 0deg,
    rgba(255, 140, 0,  0.0)   0deg,
    rgba(255, 200, 80, 0.6)  60deg,   /* top — bright, blueshifted */
    rgba(255, 240,160, 0.9)  90deg,   /* peak brightness */
    rgba(255, 200, 80, 0.6) 120deg,
    rgba(255, 100, 20, 0.3) 180deg,   /* side */
    rgba(180,  50,  0, 0.15) 240deg,  /* bottom — dim, redshifted */
    rgba(100,  20,  0, 0.05) 300deg,
    rgba(255, 140,  0, 0.0) 360deg
  );
}

/* disk inner — punch out center เพื่อให้เห็น core */
.disk-inner {
  position: absolute;
  inset: 18px;
  border-radius: 50%;
  background: transparent;
  /* mask center ออก */
  box-shadow: inset 0 0 0 100px #050813;
}

@keyframes disk-spin {
  from { transform: rotateX(72deg) rotateZ(0deg); }
  to   { transform: rotateX(72deg) rotateZ(360deg); }
}

/* ─── Layer 3: Black Hole Core ─── */
/* ความมืดสมบูรณ์ — ไม่มีแสงออกมา */
.bh-core {
  position: absolute;
  inset: 20px;
  border-radius: 50%;
  background: radial-gradient(
    circle at 38% 38%,     /* offset center — gravitational lensing asymmetry */
    #000000 40%,
    #050813 100%
  );
  /* subtle rim เพื่อให้เห็นขอบ */
  box-shadow:
    0 0 0 1px rgba(255,160,40,0.2),
    inset 0 0 20px rgba(0,0,0,1);
}

/* ─── Layer 4: Corona Top ─── */
/* สว่างด้านบน อย่าง Interstellar */
.corona-top {
  position: absolute;
  left: 50%;
  top: -8px;
  width: 80px;
  height: 40px;
  transform: translateX(-50%);
  border-radius: 50% 50% 0 0;
  background: radial-gradient(
    ellipse at 50% 100%,
    rgba(255, 220, 120, 0.9) 0%,
    rgba(255, 160,  60, 0.5) 40%,
    transparent 70%
  );
  filter: blur(4px);
  animation: corona-pulse 3s ease-in-out infinite;
}

@keyframes corona-pulse {
  0%, 100% { opacity: 0.7; transform: translateX(-50%) scaleX(1)   scaleY(1);   }
  50%       { opacity: 1.0; transform: translateX(-50%) scaleX(1.2) scaleY(1.3); }
}

/* ─── Layer 5: Infalling Particles ─── */
/* particle ถูกดูดเข้า — spiral infall */
.infalling-particle {
  position: absolute;
  width: 3px;
  height: 3px;
  border-radius: 50%;
  background: rgba(255, 200, 100, 0.9);
  top: 50%;
  left: 50%;
  /* เริ่มที่ระยะ --r จาก center, มุม --angle */
  transform:
    rotate(var(--angle))
    translateX(var(--r))
    translate(-50%, -50%);
  animation: infall var(--fall-dur, 2.5s) var(--delay) ease-in infinite;
  box-shadow: 0 0 4px rgba(255,200,100,0.8);
}

@keyframes infall {
  0% {
    transform:
      rotate(var(--angle))
      translateX(var(--r))
      translate(-50%, -50%);
    opacity: 0.9;
    width: 3px; height: 3px;
  }
  70% {
    opacity: 0.6;
    width: 2px; height: 2px;
  }
  100% {
    /* spiral เข้าหา center — rotate เพิ่ม + radius หด */
    transform:
      rotate(calc(var(--angle) + 540deg))  /* หมุนก่อนดูด */
      translateX(8px)                       /* เหลือระยะ 8px ก่อนหาย */
      translate(-50%, -50%);
    opacity: 0;
    width: 1px; height: 1px;
  }
}

/* ─── Layer 6: Pulse Rings ─── */
/* energy burst ออกมาเป็นระยะ */
.pulse-ring {
  position: absolute;
  inset: 10px;
  border-radius: 50%;
  border: 1px solid rgba(255, 180, 60, 0.6);
  animation: pulse-out 3s ease-out infinite;
}
.pulse-ring-2 {
  animation-delay: 1.5s;  /* offset ครึ่งรอบ */
  border-color: rgba(0, 255, 212, 0.4);
}

@keyframes pulse-out {
  0% {
    transform: scale(1);
    opacity: 0.7;
  }
  100% {
    transform: scale(2.5);
    opacity: 0;
  }
}
```

---

## Z-Index ภายใน `.oracle-node`

| Layer | z-index | Element |
|-------|---------|---------|
| Pulse rings (ด้านหลังสุด) | 1 | `.pulse-ring` |
| Grav lens | 2 | `.grav-lens` |
| Accretion disk | 3 | `.accretion-disk` |
| Black hole core | 4 | `.bh-core` |
| Corona top | 5 | `.corona-top` |
| Infalling particles | 6 | `.infalling-particle` |

---

## Animation Summary

| Animation | Duration | ลักษณะ |
|-----------|----------|--------|
| `lens-breathe` | 6s infinite | photon ring สว่าง-หรี่ |
| `disk-spin` | 8s linear | จานหมุนรอบต่อเนื่อง |
| `corona-pulse` | 3s infinite | corona บนโต-เล็ก พร้อม glow |
| `infall` | 2.5s infinite (staggered) | particle spiral เข้า black hole |
| `pulse-out` | 3s infinite (2 rings offset) | energy ring แผ่ออก fade |

---

## Performance Note

ทุก animation ใช้ `transform` + `opacity` เท่านั้น — GPU accelerated, ไม่ reflow
`will-change: transform, opacity` ให้ `.accretion-disk`, `.infalling-particle`

---

*วีส — Angel of Precision 🟦 | Patch for Issue #74 | 2026-03-19*
