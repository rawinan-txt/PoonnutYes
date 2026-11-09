# POONNUTYES 🏸 — นัดตีแบดง่าย ๆ

เว็บนัดตีแบดมินตัน: สร้างนัด → แชร์ลิงก์ → เพื่อนลงชื่อโดยไม่ต้อง Login → เจ้าของจัดการด้วยรหัสลับ + ติดตามการจ่ายเงินแบบ Manual
Stack: React + Vite + TypeScript, Tailwind, Lucide, Cloudflare Workers + D1 (Free Tier ทั้งหมด, ไม่ต้องใช้บัตรเครดิต)

## ติดตั้ง (Windows / PowerShell)
ต้องมี Node.js 20+ และ Git
```powershell
npm install
```

## รัน Local
```powershell
npx wrangler d1 migrations apply poonnutyes-db --local   # หรือ npm run db:migrate:local
# เทอร์มินัล 1 (API + D1 local)
npm run dev:api
# เทอร์มินัล 2 (หน้าเว็บ) เปิด http://localhost:5173
npm run dev
```
ทดสอบ API อัตโนมัติ (ขณะ dev:api ทำงาน): `npm run test:smoke -- http://localhost:8787`

## Build
```powershell
npm run build
```

## Deploy ขึ้น Cloudflare (ต้องทำเอง: login Cloudflare)
1. `npx wrangler login` (เปิดเบราว์เซอร์ให้ล็อกอินบัญชี Cloudflare ฟรี)
2. `npx wrangler d1 create poonnutyes-db` → คัดลอก `database_id` ที่ได้ไปใส่ใน `wrangler.toml`
3. `npm run db:migrate:remote`
4. `npm run deploy`
5. ได้ URL แบบ `https://poonnutyes.<your-subdomain>.workers.dev` ส่งให้เพื่อนได้เลย

(ถ้าต้องการใช้ GitHub: `git init`, `git add .`, `git commit`, แล้ว push ไป repo ของคุณ)

## ความปลอดภัยที่ทำไว้
- รหัสลับเจ้าของ: PBKDF2-SHA256 + salt, ไม่ส่งกลับ API; ตรวจสิทธิ์ทุก Admin endpoint ที่ Backend (header `x-admin-secret`)
- Registration token สุ่ม 192 บิต เก็บเฉพาะ SHA-256 hash; ยกเลิกชื่อได้ด้วย token ของตัวเองเท่านั้น
- ลงชื่อแบบ atomic (INSERT…SELECT เช็กจำนวนและสถานะในคำสั่งเดียว) + Unique index ชื่อเล่นต่อนัด (ไม่สนตัวพิมพ์)
- Prepared statements ทั้งหมด, validate input, React escape ข้อความ (ป้องกัน XSS), ตรวจ URL แผนที่ให้เป็น http(s)
- CORS: same-origin เท่านั้น (เว็บและ API อยู่ใน Worker เดียวกัน)
- Rate limit: แบบ best-effort ในหน่วยความจำของ isolate (ไม่ใช่ global) — ถ้าต้องการเข้มกว่านี้ใช้ Cloudflare WAF Rate Limiting Rules
- เวลา: วันที่/เวลาเก็บเป็นเวลาไทย (Asia/Bangkok), แสดงผลด้วย timezone ไทย

## โครงสร้าง
`worker/index.ts` API · `migrations/` schema · `src/` frontend · `scripts/smoke.mjs` ทดสอบ API
