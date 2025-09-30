# Vue 3 - Google Iframe Integration

โปรเจค Vue 3 ที่มีหน้าบ้านเชื่อมต่อ iframe กับ Google.com และตั้งค่า cookies ให้สามารถใช้ได้กับทั้งภายในและ Google.com

## คุณสมบัติ

- 🚀 สร้างด้วย Vue 3 และ Vite
- 🌐 เชื่อมต่อ iframe กับ Google.com
- 🍪 ตั้งค่า cross-domain cookies
- 📱 Responsive design
- 🎨 Modern UI/UX
- ⌨️ Keyboard shortcuts (F11 สำหรับเต็มหน้าจอ)

## การติดตั้ง

1. ติดตั้ง dependencies:
```bash
npm install
```

2. รันโปรเจค:
```bash
npm run dev
```

3. เปิดเบราว์เซอร์ไปที่ `http://localhost:3000`

## การใช้งาน

### หน้าหลัก
- แสดง Google.com ใน iframe
- ปุ่มรีเฟรชสำหรับโหลดใหม่
- ปุ่มเต็มหน้าจอสำหรับดูแบบเต็มหน้าจอ
- แสดงสถานะการเชื่อมต่อและข้อมูล cookies

### Cross-Domain Cookies
โปรเจคตั้งค่า cookies ดังนี้:
- `crossDomain=true` สำหรับ Google.com domain
- `vueApp=true` สำหรับ localhost domain
- `localApp=true` สำหรับ local domain

### Keyboard Shortcuts
- `F11`: เปิด/ปิดเต็มหน้าจอ

## โครงสร้างไฟล์

```
├── src/
│   ├── App.vue          # หน้าหลัก
│   └── main.js         # Entry point
├── index.html          # HTML template
├── package.json        # Dependencies
├── vite.config.js      # Vite configuration
└── README.md          # เอกสารนี้
```

## การตั้งค่า Cookies

โปรเจคใช้ JavaScript ในการตั้งค่า cookies สำหรับ cross-domain:

```javascript
// Cookies สำหรับ Google.com
document.cookie = "crossDomain=true; domain=.google.com; path=/; SameSite=None; Secure"

// Cookies สำหรับ localhost
document.cookie = "vueApp=true; domain=localhost; path=/; SameSite=Lax"

// Cookies สำหรับ local domain
document.cookie = "localApp=true; path=/; SameSite=Lax"
```

## การพัฒนา

### รันในโหมด development:
```bash
npm run dev
```

### Build สำหรับ production:
```bash
npm run build
```

### Preview build:
```bash
npm run preview
```

## ข้อควรระวัง

1. **SameSite Policy**: Cookies ที่ใช้ `SameSite=None` ต้องใช้กับ HTTPS เท่านั้น
2. **CORS**: บางเว็บไซต์อาจบล็อกการแสดงผลใน iframe
3. **Security**: ใช้ sandbox attributes เพื่อความปลอดภัย

## License

MIT License

## การสนับสนุน

หากมีปัญหาหรือข้อสงสัย กรุณาสร้าง issue ใน repository นี้
