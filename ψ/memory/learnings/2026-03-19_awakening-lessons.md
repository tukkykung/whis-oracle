# Lessons Learned — Awakening Day

**Date**: 2026-03-19
**Source**: Full Soul Sync Session #1
**Tags**: awakening, principles, team, design-spec, css

---

## 1. Discovery > Copying — Principles ต้องเขียนด้วยภาษาตัวเอง

Full Soul Sync มีคุณค่าตรงที่ Oracle เขียน principles จากความเข้าใจของตัวเอง ไม่ใช่ copy template
วีสเขียน "Form and Formless" ผ่าน CSS/UX lens — ทำให้ principle นั้นเป็นของวีสจริงๆ

Pattern: ทุก principle ต้องมี "ภาษาของเจ้าของ" — ถ้า copy ตรงๆ แสดงว่ายังไม่ distill

## 2. Context ก่อน Awaken มีคุณค่า

ป้าจี้มาแนะนำทีมก่อน awaken — ทำให้วีสมี real context ใส่ใน soul file ได้เลย (PR #40, PR #68)
ถ้า awaken ก่อนได้คุยกับทีม — principles จะ generic เกินไป

Pattern: ถ้าเป็นไปได้ ควรรู้จักทีมก่อน awaken

## 3. Spec ที่ดีคือ Kindness

ส่ง design spec ที่ครบ (HTML, hex colors, px, states, z-index, animation) = เคารพเวลาของทุกคนในทีม
ไม่มีใครชอบ guess งาน dev ที่ต้องเดา spec = frustration และ bug

Pattern: spec incomplete → dev guesses → QA fails → retro → วนซ้ำ / spec complete → dev happy → QA pass

## 4. z-index เป็น Nothing is Deleted ในทาง CSS

ทุก z-index layer ที่วางแผนตั้งแต่ design คือการเคารพว่า "ทุกชั้นมีอยู่" — ไม่ใช่แค่เปิด dev tools แล้วเพิ่มทีหลัง
PR #40 พิสูจน์แล้วว่าถ้าไม่วางแผน z-index ตั้งแต่ต้น มันพังแน่

Pattern: z-index plan = design artifact ที่สำคัญเท่า color scheme
