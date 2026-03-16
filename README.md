# ระบบค่ามือหมอนวด

ระบบจัดการค่ามือหมอนวดที่ใช้ Google Sheets เป็นฐานข้อมูล

## การตั้งค่า Google Sheets

1. สร้าง Google Sheet ใหม่
2. ใน Sheet สร้างคอลัมน์: id, therapistId, serviceName, duration, price, fee, spa, date, createdAt
3. สร้าง Google Apps Script:
   - เข้าไปที่ Extensions > Apps Script
   - แทนที่โค้ดด้วย:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSheet();
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
    return ContentService
      .createTextOutput(JSON.stringify({success: true}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({error: error.message}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSheet();
    const data = sheet.getDataRange().getValues();
    const headers = data[0];
    const rows = data.slice(1).map(row => {
      const obj = {};
      headers.forEach((header, i) => obj[header] = row[i]);
      return obj;
    });
    return ContentService
      .createTextOutput(JSON.stringify(rows))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({error: error.message}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Deploy as web app และคัดลอก URL
5. แทนที่ `API_URL` ใน `index.html` ด้วย URL ที่ได้

## การใช้งาน

- เปิด `index.html` ในเบราว์เซอร์
- เพิ่มหมอนวดใหม่ (เก็บใน localStorage ของเครื่อง)
- บันทึกงานนวด (ส่งไป Google Sheets)
- รายงานจะโหลดข้อมูลจาก Google Sheets

**ข้อจำกัด:** ข้อมูลหมอนวดยังเก็บเฉพาะเครื่อง ถ้าต้องการแชร์หมอนวดด้วย ต้องเพิ่มใน Google Sheets

## การ deploy

อัปโหลด `index.html` ไปยังเว็บโฮสติ้งใดก็ได้