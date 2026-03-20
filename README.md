# 📡 Tidlor Content Radar

> Keyword research · TikTok script · Content ideas · News radar  
> Powered by SerpAPI + Gemini + Google Apps Script

---

## ภาพรวม

Tidlor Content Radar คือระบบ content intelligence สำหรับทีม Marcom เงินติดล้อ ทำงาน 2 โหมดพร้อมกัน

| โหมด | วิธีทริกเกอร์ | ทำอะไร |
|---|---|---|
| **Auto** | Time trigger ทุกวัน 08:00 | loop keyword ทั้งหมดใน Sheet → บันทึก + ส่ง alert |
| **On-demand** | กดปุ่มบน frontend | research + generate real-time → แสดงผลทันที |

---

## โครงสร้างไฟล์

```
tidlor-content-radar/
├── index.html          ← frontend (GitHub Pages)
├── appscript.js        ← Google Apps Script (copy ไปใส่ใน Apps Script Editor)
└── README.md
```

---

## การติดตั้ง

### ขั้นที่ 1 — เตรียม API Keys

| Key | ได้จากไหน |
|---|---|
| `SERP_API_KEY` | [serpapi.com](https://serpapi.com) → Dashboard → API Key |
| `GEMINI_API_KEY` | [aistudio.google.com](https://aistudio.google.com) → Get API Key |
| `SHEET_ID` | URL ของ Google Sheet: `docs.google.com/spreadsheets/d/`**`SHEET_ID`**`/edit` |
| `LINE_NOTIFY_TOKEN` | [notify-bot.line.me](https://notify-bot.line.me) → Generate token (optional) |

---

### ขั้นที่ 2 — ตั้งค่า Google Apps Script

1. เปิด [script.google.com](https://script.google.com) → **New project**
2. ลบ code เดิมออก แล้ว **paste code จาก `appscript.js`** ทั้งหมด
3. แก้ไขค่าใน `CONFIG`:

```js
const CONFIG = {
  SERP_API_KEY:      "YOUR_SERP_API_KEY",
  GEMINI_API_KEY:    "YOUR_GEMINI_API_KEY",
  SHEET_ID:          "YOUR_GOOGLE_SHEET_ID",
  LINE_NOTIFY_TOKEN: null,   // ใส่ token ถ้าต้องการ Line alert
  NOTIFY_EMAIL:      null,   // ใส่ email ถ้าต้องการ Email alert
};
```

4. **Deploy เป็น Web App**
   - กด **Deploy** → **New deployment**
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - กด **Deploy** → **Copy Web App URL**

5. **รัน `setupSheet()`** (ครั้งเดียว)
   - เลือกฟังก์ชัน `setupSheet` → กด ▶ Run
   - สร้าง tab Keywords พร้อม sample data อัตโนมัติ

6. **รัน `setupDailyTrigger()`** (ครั้งเดียว)
   - เลือกฟังก์ชัน `setupDailyTrigger` → กด ▶ Run
   - ตั้ง auto trigger ทุกวัน 08:00 Bangkok time

---

### ขั้นที่ 3 — ตั้งค่า Frontend

เปิดไฟล์ `index.html` ใส่ Apps Script URL ที่ได้จากขั้นที่ 2 ในช่อง config  
หรือจะ hardcode ลงในไฟล์เลยก็ได้:

```js
let appsScriptUrl = "https://script.google.com/macros/s/YOUR_ID/exec";
```

---

### ขั้นที่ 4 — Deploy บน GitHub Pages

```bash
# 1. สร้าง repo ใหม่บน GitHub
# 2. push ไฟล์ขึ้นไป
git init
git add index.html README.md
git commit -m "init: Tidlor Content Radar"
git remote add origin https://github.com/YOUR_USERNAME/tidlor-content-radar.git
git push -u origin main

# 3. เปิด GitHub Pages
# Settings → Pages → Source: main branch → Save
# URL จะเป็น: https://YOUR_USERNAME.github.io/tidlor-content-radar/
```

---

## การใช้งาน

### Frontend (GitHub Pages)

| ปุ่ม | action | ทำอะไร |
|---|---|---|
| ▶ **Run** | `full` | research + TikTok script + AEO + บันทึก Sheet |
| 💡 **Ideas** | `ideas` | สร้าง content ideas 6 ชิ้น (Gen Z 3 + Gen Y 3) พร้อม BOT check |
| 📰 **News** | `news` | ดึงข่าวการเงิน + seasonal angles |

### Google Sheet Tabs

| Tab | เก็บอะไร |
|---|---|
| `Keywords` | keyword list — เพิ่ม/ลด/เปิดปิดได้ตลอด |
| `ContentRadar` | TikTok script + AEO ทุก run |
| `ContentIdeas` | 6 ideas + BOT check ทุก run |
| `NewsRadar` | ข่าว + seasonal angles ทุกวัน |

### เพิ่ม Keyword ใน Auto Mode

เปิด Google Sheet → tab **Keywords** → เพิ่ม row:

| Keyword | Audience | Active | Note |
|---|---|---|---|
| เงินกู้ฉุกเฉิน | gen_z | TRUE | new keyword |

ระบบจะดึง keyword นี้อัตโนมัติใน run วันถัดไป

---

## BOT Compliance

ระบบฝัง BOT rules ไว้ใน prompt ของ Gemini อัตโนมัติ

**วลีที่ใช้ได้** เช่น `สมัครง่าย`, `อนุมัติไว รับเงินทันที`, `วงเงินสูง ดอกเบี้ยต่ำ`

**วลีที่ห้ามใช้** เช่น `กู้ง่าย`, `ติดบูโรก็กู้ได้`, `ไม่เช็คบูโร`

แต่ละ content idea มี `BOT Check: PASS / FAIL` แสดงบน frontend และใน Sheet

**Remark** จะถูกใส่อัตโนมัติตาม product ที่ Gemini เลือก เช่น  
`กู้เท่าที่จำเป็น และชำระคืนไหว อัตราดอกเบี้ย 12-24% ต่อปี`

---

## โครงสร้าง Apps Script

```
doGet()                     ← entry point รับ HTTP request
├── action=research         → runResearch()
├── action=full             → runResearch() + generateTikTokContent() + saveContentToSheet()
├── action=ideas            → runResearch() + generateContentIdeas() + saveIdeasToSheet()
└── action=news             → runNewsRadar()

runDailyDigest()            ← entry point สำหรับ time trigger
├── runNewsRadar()
├── loop keywords
│   ├── runResearch()
│   ├── generateTikTokContent()
│   ├── generateContentIdeas()
│   ├── saveContentToSheet()
│   └── saveIdeasToSheet()
└── sendAlert()             → Line Notify + Email

callGemini()                ← shared Gemini helper
getOrCreateSheet()          ← shared Sheet helper
```

---

## Troubleshooting

**`script load error` บน frontend**  
→ ตรวจสอบว่า Deploy ใหม่แล้วและตั้ง Who has access = **Anyone**

**`No supported Gemini model found`**  
→ ตรวจสอบ `GEMINI_API_KEY` ว่าถูกต้องและยังใช้งานได้

**Trigger ไม่ทำงาน**  
→ Apps Script Editor → ไอคอนนาฬิกา (Triggers) → ตรวจสอบว่ามี `runDailyDigest` อยู่  
→ ถ้าไม่มี รัน `setupDailyTrigger()` ใหม่อีกครั้ง

**ข้อมูลไม่บันทึกลง Sheet**  
→ ตรวจสอบ `SHEET_ID` ว่าถูกต้อง และ Google account ที่ deploy มีสิทธิ์แก้ไข Sheet นั้น

---

## Dependencies

- [SerpAPI](https://serpapi.com) — Google Search data
- [Google Gemini API](https://aistudio.google.com) — content generation
- [Google Apps Script](https://script.google.com) — backend + scheduler
- [Google News RSS](https://news.google.com/rss) — ฟรี ไม่ต้องใช้ API key
- [GitHub Pages](https://pages.github.com) — frontend hosting ฟรี
