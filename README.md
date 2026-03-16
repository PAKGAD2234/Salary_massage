# ระบบค่ามือหมอนวด

ระบบจัดการค่ามือหมอนวดที่ใช้ Google Sheets เป็นฐานข้อมูล

## การตั้งค่า Google Sheets

1. สร้าง Google Sheet ใหม่
2. สร้าง 2 sheet:
   - `massage_records` (columns: id, therapistId, serviceName, duration, price, fee, spa, date, createdAt)
   - `therapists` (columns: id, name, type, createdAt)
3. สร้าง Google Apps Script:
   - Extensions > Apps Script
   - วางโค้ดนี้ลงใน `Code.gs`:

```javascript
const SS_ID = "1pfequ86KFD2Ah6_74L0JAG_eCL3ykVq1PBxDfkGd2mc";

function doPost(e) {
  try {
    // รับข้อมูลแบบ form-url-encoded (จากหน้าเว็บ)
    const data = e.parameter;

    const ss = SpreadsheetApp.openById(SS_ID);

    // เพิ่มรายการนวด
    if (data.type === "record") {
      const sheet = ss.getSheetByName("massage_records");
      sheet.appendRow([
        data.id,
        data.therapistId,
        data.serviceName,
        data.duration,
        data.price,
        data.fee,
        data.spa,
        data.date,
        data.createdAt
      ]);
    }

    // เพิ่มหมอนวด
    if (data.type === "therapist") {
      const sheet = ss.getSheetByName("therapists");
      sheet.appendRow([
        data.id,
        data.name,
        data.therapistType, // ใช้ therapistType แทน type
        data.createdAt
      ]);
    }

    return ContentService
      .createTextOutput(JSON.stringify({status:"success"}))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({status:"error", message: error.message}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet() {
  const ss = SpreadsheetApp.openById(SS_ID);

  const recSheet = ss.getSheetByName("massage_records");
  const therSheet = ss.getSheetByName("therapists");

  const recData = recSheet.getDataRange().getValues();
  const therData = therSheet.getDataRange().getValues();

  recData.shift();
  therData.shift();

  const records = recData.map(r => ({
    id: r[0],
    therapistId: r[1],
    serviceName: r[2],
    duration: r[3],
    price: r[4],
    fee: r[5],
    spa: r[6],
    date: Utilities.formatDate(new Date(r[7]), "Asia/Bangkok", "yyyy-MM-dd"),
    createdAt: r[8]
  }));

  const therapists = therData.map(t => ({
    id: t[0],
    name: t[1],
    type: t[2],
    createdAt: t[3]
  }));

  return ContentService
    .createTextOutput(JSON.stringify({ records, therapists }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. Deploy as web app:
   - Execute as: Me
   - Who has access: Anyone
5. คัดลอก URL และแทนที่ใน `index.html` (บรรทัดที่มี `const API_URL = ...`)

## การใช้งาน

- เปิด `index.html` ในเบราว์เซอร์
- เพิ่มหมอนวดใหม่ (บันทึกใน Google Sheets)
- บันทึกงานนวด (บันทึกใน Google Sheets)
- รายงานจะโหลดข้อมูลจาก Google Sheets

## ฟีเจอร์

- ✅ ข้อมูลใช้ร่วมกันจากทุกเครื่อง (ผ่าน Google Sheets)
- ✅ เพิ่ม/ลบหมอนวด
- ✅ บันทึกงานนวดและคำนวณค่ามืออัตโนมัติ
- ✅ รายงานรายเดือนและรายวัน
- ✅ UI ที่สวยงามและใช้งานง่าย

## การ deploy

อัปโหลด `index.html` ไปยังเว็บโฮสติ้งใดก็ได้