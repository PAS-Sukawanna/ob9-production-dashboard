# จำกัดการเข้าถึงด้วย Cloudflare Access (ล็อกด้วยอีเมล)

เป้าหมาย: ให้เปิด dashboard ได้เฉพาะอีเมลที่คุณอนุญาต (ต้อง login ก่อนถึงจะเห็นหน้าเว็บ)
วิธีนี้ปลอดภัยสูงและ**ฟรี** (Cloudflare Zero Trust ฟรีสูงสุด 50 ผู้ใช้)

แนวคิด: ย้ายการโฮสต์จาก githack → **Cloudflare Pages** (ต่อกับ GitHub repo โดยตรง)
แล้วครอบด้วย **Cloudflare Access** เพื่อบังคับ login

---

## ขั้นตอน

### 1) สมัคร Cloudflare (ฟรี)
- ไปที่ https://dash.cloudflare.com/sign-up สมัครด้วยอีเมลของคุณ

### 2) สร้าง Cloudflare Pages ต่อกับ repo
1. Dashboard → **Workers & Pages** → **Create** → แท็บ **Pages** → **Connect to Git**
2. อนุญาต (Authorize) GitHub แล้วเลือก repo **`PAS-Sukawanna/ob9-production-dashboard`**
3. ตั้งค่า build:
   - **Framework preset:** `None`
   - **Build command:** เว้นว่าง
   - **Build output directory:** `personal/Claude AI/Claude AI IPad`
     *(โฟลเดอร์มีเว้นวรรคได้ ถ้ามีปัญหาให้ตั้งเป็น `/` แล้วเข้าที่ path ย่อยแทน)*
4. กด **Save and Deploy** → รอสักครู่จะได้ URL เช่น `https://ob9-xxxx.pages.dev`
   - ไฟล์หลักจะอยู่ที่ `.../stock_portfolio_dashboard.html`

> ทุกครั้งที่มี commit ใหม่บน repo Cloudflare Pages จะ deploy อัตโนมัติ (ได้เวอร์ชันใหม่เอง)

### 3) เปิด Zero Trust แล้วครอบด้วย Access
1. Dashboard → **Zero Trust** (ครั้งแรกให้ตั้งชื่อ team + เลือกแพลนฟรี)
2. **Access → Applications → Add an application → Self-hosted**
3. ตั้งค่า:
   - **Application name:** Dividend Dashboard
   - **Session duration:** ตามต้องการ (เช่น 24 ชม.)
   - **Application domain:** ใส่โดเมน `ob9-xxxx.pages.dev` (subdomain ที่ได้จากข้อ 2)
4. **Policies → Add a policy:**
   - **Policy name:** Allowed users
   - **Action:** Allow
   - **Include → Emails →** ใส่อีเมลที่อนุญาต (ของคุณ + คนที่คุณไว้ใจ) หรือใช้ **Emails ending in** สำหรับทั้งโดเมน
5. Save

### 4) เสร็จ — ทดสอบ
- เปิด `https://ob9-xxxx.pages.dev/stock_portfolio_dashboard.html`
- Cloudflare จะให้ยืนยันตัวตนก่อน (ส่งรหัส OTP ไปอีเมล หรือ login Google) — เฉพาะอีเมลในรายการเท่านั้นที่ผ่าน

---

## แนะนำเพิ่มเติม
- **ปิด githack:** หลังใช้ Cloudflare Pages แล้ว เลิกใช้/เลิกแชร์ลิงก์ githack (ลิงก์นั้นไม่มีการล็อก)
- **ทำ repo เป็น private ได้:** Cloudflare Pages deploy จาก private repo ได้ → ซ่อนซอร์สโค้ดด้วย
  (แต่ repo นี้มีโปรเจกต์อื่นรวมอยู่ การเปลี่ยนเป็น private จะกระทบทั้ง repo — พิจารณาตามเหมาะสม
  หรือแยก dashboard ไป repo ใหม่เฉพาะ)
- **ข้อมูลยังเป็นส่วนตัวเสมอ:** ข้อมูลพอร์ตจริงเก็บใน localStorage ต่อเครื่อง/ต่อผู้ใช้
  แต่ละคนที่ login เห็นเฉพาะข้อมูลที่ตัวเองนำเข้า (เริ่มจากข้อมูลตัวอย่าง)

## ทางเลือกที่เร็วกว่า (ปลอดภัยน้อยกว่า)
ถ้ายังไม่พร้อมตั้ง Cloudflare — บอกผมได้ ผมเพิ่ม **รหัสผ่านหน้าเว็บ** (password gate) ให้ในไฟล์
กันคนทั่วไปได้ระดับหนึ่ง (แต่คนเปิดซอร์สดูได้ จึงควรใช้คู่กับการไม่มีข้อมูลจริงในโค้ด ซึ่งตอนนี้ทำแล้ว)
