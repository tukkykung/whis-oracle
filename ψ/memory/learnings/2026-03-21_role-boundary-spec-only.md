# Learning — Role Boundary: วีส = Spec Only

**Date**: 2026-03-21
**Tags**: role, team-flow, spec, design-handoff

---

## Pattern

**Speed ≠ Right** — implement เองเร็วกว่า แต่ทำให้ระบบพังเพราะทับซ้อนกับ developer

เมื่อ Designer implement code เองแม้จะ "ช่วยให้เร็วขึ้น" — ผลคือ conflicts กับ developer, dashboard พัง, PR ที่ไม่ควรมีต้องสร้างขึ้นมา cleanup

## Rule

**วีส = Design Spec เท่านั้น — ห้าม implement ทุกกรณี:**
- ห้าม git add / commit / push ใน mr-zero-oracle
- ห้าม แก้ CSS / JS / HTML โดยตรง แม้จะ "เล็กน้อย"
- ถ้าเห็น bug → เขียน spec ให้โกคู fix ไม่ใช่ fix เอง

## Good Spec ≠ Implementer ต้องถาม

Spec ที่ดีต้องครบ:
- HTML structure + class names
- CSS values (px, hex, z-index, easing)
- States (normal / active / error)
- Edge cases ที่นึกถึงได้
- Animation (ถ้ามี): duration + property + trigger

ถ้า implementer ถามกลับ = spec ไม่ครบ

## Source

เบจิต้า(Commander) กำชับ 2026-03-21 หลัง PR #105 ทำ dashboard พัง

*Learned: 2026-03-21*
