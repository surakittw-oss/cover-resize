# Image Tool

เครื่องมือย่อภาพและจัดการ cover web สำหรับทีมเว็บ — ประมวลผลในเบราว์เซอร์ ไม่อัปโหลดไฟล์ขึ้น server

Single-file HTML + Vanilla JS · Google OAuth (Internal Workspace) · ไม่มี backend

---

## Features

### Cover Mode
ปรับขนาดภาพปกเว็บทีละภาพ พร้อมตัวเลือกปรับละเอียด

- **Preset 6 ขนาด** — Web Cover (1200×628), Facebook (1200×630), Twitter 16:9, IG Square/Portrait/Story
- **Custom size** + ปุ่ม swap สลับกว้าง/สูง พร้อมแสดง aspect ratio
- **3 โหมดเติมพื้นที่** — พื้นสี / เบลอ / ครอป
- **Smart Crop** — ใช้ saliency algorithm (edge + saturation + skin-tone detection) หาจุดสนใจอัตโนมัติ
- **Focal Point Crop** — คลิกบนภาพเพื่อปักจุดสำคัญ
- **Manual Crop** — ลากกรอบ + handle 4 มุม ล็อค aspect ratio
- **Drag-to-place** สำหรับโหมดพื้นสี/เบลอ — ย้ายและปรับขนาดรูปในกรอบได้
- **Auto/Manual compress** — ปรับ JPG quality อัตโนมัติให้ใกล้เคียง target file size

### Batch Mode
ย่อภาพหลายไฟล์พร้อมกัน เหมาะกับภาพประกอบบทความ

- **Drag หลายไฟล์พร้อมกัน** ประมวลผล parallel ทันที
- **Default 850×567** ปรับได้
- **Smart Crop / พื้นขาว / เบลอ BG** — เลือกครั้งเดียว apply ทุกภาพ
- **Apply to all** — เปลี่ยน setting ใดๆ → reprocess ทุกภาพอัตโนมัติ
- **Per-image actions** — ดาวน์โหลดทีละไฟล์ หรือลบออกจาก batch
- **Download All as ZIP** ใน 1 คลิก

### Security
- **Google OAuth (Internal Workspace)** — เฉพาะอีเมล `@thestandard.co`
- **Auto session restore** ผ่าน localStorage
- **Auto sign-out** เมื่อ token หมดอายุ
- **JWT validation ฝั่ง client** เป็น defense in depth

### UX
- โทน **Apple-minimal** ขาวดำ + accent CI สีแดง `#e62227`
- ฟอนต์ **Prompt** (Thai-friendly)
- **Responsive** ครบทุกอุปกรณ์ (320px → 1280px+)
- **Touch-friendly** drag/resize handles ขยายอัตโนมัติบน mobile
- **iOS safe-area** รองรับ iPhone notch
- **Dark mode** ของ system bar (theme-color)

---

## Tech Stack

| Layer | Used |
|---|---|
| Markup | HTML5 |
| Styling | Vanilla CSS (CSS Variables, Grid, Flexbox) |
| Scripting | Vanilla JavaScript (ES2020) |
| Image processing | HTML5 Canvas API |
| Compression | `canvas.toBlob()` + binary search quality |
| Smart crop | Saliency map + integral image (sliding window) |
| ZIP | [JSZip](https://stuk.github.io/jszip/) (CDN) |
| Auth | [Google Identity Services](https://developers.google.com/identity/gsi/web) |
| Fonts | [Prompt](https://fonts.google.com/specimen/Prompt) (Google Fonts) |

**ไม่มี build step** — เปิด HTML ตรงๆ ก็ใช้ได้ ไม่ต้อง npm install

---

## Quick Start

### Local Development

```bash
# clone หรือ download cover-tool.html
cd path/to/folder

# ต้องเปิดผ่าน local server เพราะ Google OAuth ไม่ทำงานกับ file://
python3 -m http.server 8000
# หรือ
npx serve
```

เปิด http://localhost:8000 ในเบราว์เซอร์

> **สำคัญ:** ต้อง add `http://localhost:8000` ใน Authorized JavaScript origins ของ OAuth Client ก่อน (ดู [Google OAuth Setup](#google-oauth-setup))

---

## Deploy

### Vercel (แนะนำ — ใช้ได้ฟรี)

```bash
# สร้าง vercel.json ในโฟลเดอร์
echo '{"cleanUrls": true}' > vercel.json

# deploy
npx vercel --prod
```

หลัง deploy ได้ URL เช่น `https://image-tool.vercel.app` → กลับไป Google Cloud Console เพิ่ม URL นี้ใน Authorized JavaScript origins

### Netlify

```bash
# drag & drop ไฟล์ cover-tool.html เข้า netlify.com/drop
# หรือ
npx netlify deploy --prod --dir .
```

### GitHub Pages

1. push ไฟล์ขึ้น repo
2. Settings → Pages → Source: `main` branch → root
3. รอ ~1 นาที จะได้ URL `https://username.github.io/repo-name`

### Internal server

ใส่ไฟล์ใน folder ของ web server (Nginx, Apache, IIS) ได้เลย — ไม่มี dependency

---

## Google OAuth Setup

ขั้นตอน setup ครั้งเดียวสำหรับ admin Workspace

### 1. สร้าง Project ใน Google Cloud Console

- ไปที่ https://console.cloud.google.com
- คลิก project dropdown ด้านบน → **New Project**
- ตั้งชื่อ เช่น `image-tool-thestandard` → Create

### 2. Configure OAuth Consent Screen

- เมนูซ้าย → **APIs & Services** → **OAuth consent screen**
- เลือก **Internal** (จำเป็น — Google จะ block user ที่ไม่ใช่ Workspace member ให้เอง)
- ใส่ข้อมูล:
  - App name: `Image Tool`
  - User support email: อีเมล admin
  - Developer contact: อีเมล admin
- Save & Continue (skip Scopes, Test users)

### 3. สร้าง OAuth Client ID

- เมนูซ้าย → **Credentials** → **+ Create Credentials** → **OAuth client ID**
- Application type: **Web application**
- Name: `Image Tool Web`
- **Authorized JavaScript origins** ใส่ URL ทั้งหมดที่จะเปิดใช้:
  ```
  http://localhost:8000
  https://image-tool.thestandard.co
  https://your-vercel-domain.vercel.app
  ```
- **Authorized redirect URIs** ไม่ต้องใส่
- Create → copy **Client ID**

### 4. ใส่ Client ID ในโค้ด

แก้ไฟล์ `cover-tool.html` ที่บรรทัด:

```js
const GOOGLE_CLIENT_ID = 'YOUR_CLIENT_ID.apps.googleusercontent.com';
const ALLOWED_DOMAIN = 'thestandard.co';
```

---

## Architecture

### File Structure
```
cover-tool.html      # ทั้งหมดอยู่ในไฟล์เดียว
└── README.md        # ไฟล์นี้
```

### Code Sections

```
<head>
  ├── meta tags + viewport + theme-color
  ├── Google Fonts (Prompt)
  ├── JSZip CDN
  ├── Google Identity Services CDN
  └── <style> (CSS variables → all components)

<body>
  ├── <nav> + user badge
  ├── <div #login-screen>           # OAuth gate
  ├── <div #app-container>
  │   ├── <div .tabs>               # Cover / Batch
  │   ├── <div #view-cover>
  │   │   ├── hero + dropzone
  │   │   └── workspace
  │   │       ├── preview / crop editor / place editor
  │   │       └── controls panel
  │   └── <div #view-batch>
  │       ├── hero + dropzone (multiple)
  │       └── workspace
  │           ├── toolbar + summary
  │           └── grid of cards
  └── <footer>

<script>  # Auth (~200 lines)
  ├── JWT decode + storage
  ├── Google Identity Services init
  └── Sign-in/out flow

<script>  # App (~1500 lines)
  ├── Cover state + render pipeline
  ├── Smart crop (saliency)
  ├── Crop editor (focal/manual)
  ├── Place editor (drag/resize)
  ├── Batch state + parallel processing
  └── Tab switching
```

### State Management

แยก 2 state objects:

```js
state = {           // Cover tool
  img, origFile, outSize,
  fitMode, cropMode,
  bgColor, blurStrength,
  placement, cropRect, focalPoint,
  compressMode, targetKB, manualQuality,
  finalBlob, finalQuality,
}

batchState = {      // Batch tool
  items: [{id, file, img, blob, status, ...}],
  outSize, fitMode, targetKB,
  nextId,
}

auth = {            // Google OAuth
  user: {name, email, picture, exp},
  token,
}
```

### Coordinate Systems
- **Image px** — พิกัดในรูปต้นฉบับ (ใช้กับ `cropRect`, `focalPoint`)
- **Output px** — พิกัดในกรอบผลลัพธ์ เช่น 1200×628 (ใช้กับ `placement`)
- **Display px** — พิกัดบนหน้าจอ (แปลงผ่าน `displayScale`)

แปลงไป-มาด้วย scale factor ที่คำนวณจาก `getBoundingClientRect()` หลัง render

---

## Browser Support

| Browser | Status |
|---|---|
| Chrome 90+ | ✅ Full support |
| Safari 15+ | ✅ Full support |
| Firefox 100+ | ✅ Full support (backdrop-filter อาจไม่มีใน <103) |
| Edge 90+ | ✅ Full support |
| iOS Safari 15+ | ✅ Full support (touch + drag tested) |
| Chrome Android | ✅ Full support |

ฟีเจอร์ที่ใช้:
- `canvas.toBlob()`, `imageSmoothingQuality`
- `Promise.all()` for batch parallel
- CSS `aspect-ratio`, `min()`, `env(safe-area-inset-*)`
- Touch Events API
- localStorage

---

## Performance Notes

| Operation | Speed |
|---|---|
| Smart crop saliency | ~50ms (รูปขนาดไหนก็เท่ากัน — ย่อเป็น 64px ก่อน) |
| Auto compress (binary search) | 7 iterations × ~50ms encode = ~350ms |
| Batch 10 รูป (parallel) | ~3-5 วินาที (ขึ้นกับ CPU) |
| Place editor drag | 60fps (DOM update ทันที, re-encode debounce 120ms) |

**ทุก operation อยู่ใน main thread** — ถ้าโหลดเกิน 50 รูปอาจทำให้ UI หน่วง ในอนาคตอาจย้ายไป Web Worker ถ้าจำเป็น

---

## Security Notes

ระบบนี้เป็น **client-side only** — security มี 2 ระดับ:

### ระดับที่กันได้
✅ User ทั่วไปที่ไม่ใช่ `@thestandard.co` เปิดใช้ไม่ได้ (Google block ตั้งแต่ login)
✅ Token หมดอายุ → auto sign out
✅ ไฟล์ภาพไม่ออกจากเครื่อง — ทุกอย่างประมวลผลในเบราว์เซอร์

### ข้อจำกัด
⚠ **Bypass ได้ถ้าจริงจัง** — เพราะโค้ดทุกอย่างอยู่ใน browser คนที่มีความรู้สามารถเปิด DevTools แก้ JS ได้
⚠ **Client ID เปิดเผย** — ใครก็เห็น (เป็นเรื่องปกติของ OAuth client-side)
⚠ **ไม่มี server-side validation** — ถ้าต้องการ security ระดับ enterprise ควรย้ายไป architecture ที่มี backend

ระดับการป้องกันนี้เหมาะกับ tool ภายในองค์กรที่ต้องการ **กันคนนอกหลงเข้ามา** ไม่ใช่ปกป้องข้อมูลลับ

---

## Troubleshooting

**`Sign in with Google` ไม่ขึ้น**
- ตรวจว่าเปิดผ่าน `http://` หรือ `https://` ไม่ใช่ `file://`
- ตรวจ Authorized JavaScript origins ใน Google Cloud Console
- เปิด DevTools Console ดู error

**Login แล้วไม่เข้า — ขึ้น "เฉพาะอีเมล @thestandard.co"**
- ตรวจว่า Workspace ของอีเมลที่ใช้ login เป็นของ `thestandard.co` จริง
- ลองออกจาก Google account อื่นในเบราว์เซอร์ก่อน

**iOS zoom เวลา focus input**
- แก้แล้ว — input ทุกตัวมี `font-size: 16px` บน mobile

**Drag ไม่ลื่นบน touch device**
- ตรวจว่า browser รองรับ `touch-action: none` (Safari 13+)

**Smart crop ครอปไม่ตรงจุด**
- เปลี่ยนเป็นโหมด "จุดโฟกัส" หรือ "ลากปรับ" เพื่อกำหนดเอง

**ZIP ดาวน์โหลดไม่ได้**
- ตรวจว่า JSZip CDN โหลดสำเร็จ (DevTools → Network)
- ถ้า block อาจต้อง self-host JSZip

---

## Roadmap (ถ้าจะพัฒนาต่อ)

- [ ] Web Worker สำหรับ batch ขนาดใหญ่ (50+ รูป)
- [ ] รองรับ WebP / AVIF output
- [ ] Sharpen filter ตอน downscale (กันรูปฟุ้ง)
- [ ] Drag handle ขอบ (4 ขอบ ไม่ใช่แค่ 4 มุม)
- [ ] History / undo
- [ ] Preset save/load (JSON export)
- [ ] Per-image edit ใน batch (เปิด crop editor ทีละรูป)

---

## License

Internal use only — The Standard

---

## Credits

- Smart crop algorithm inspired by [smartcrop.js](https://github.com/jwagner/smartcrop.js) (heuristic version)
- UI design inspired by Apple Human Interface Guidelines
- Built with assistance from Claude (Anthropic)
