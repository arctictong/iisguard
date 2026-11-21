# IISGuard

อ่าน IIS log บนเครื่องคุณ ให้คะแนน bot / crawler / scraper แล้วบอกว่าเรื่องไหนควรสนใจ — และบล็อคได้เมื่อคุณสั่ง

> **สถานะ: รุ่นทดสอบ (beta)** — ค่าเริ่มต้นเป็นโหมดเฝ้าดู (dry-run) ไม่เขียนไฟล์ใด ๆ

## ดาวน์โหลด

ไฟล์ zip อยู่ใน [Releases](../../releases/latest) — Windows x64, ไฟล์เดียว ไม่ต้องติดตั้ง .NET

## เริ่มใช้ 3 ขั้น

1. แตก zip ไว้ที่ใดก็ได้บนเครื่องที่รัน IIS แล้วดับเบิลคลิก `iisguard-app.exe`
2. เปิดเบราว์เซอร์ที่ `http://127.0.0.1:8787`
3. หน้าเริ่มใช้งานจะหาโฟลเดอร์ log ให้เอง ถ้าไม่พบให้ชี้ path ตามที่โปรแกรมบอก แล้วเลือกโหมด **เฝ้าดู**

รายละเอียดเพิ่มเติมอยู่ในไฟล์ `README-อ่านก่อนใช้งาน.txt` ใน zip

## ต้องมี

| | |
|---|---|
| OS | Windows Server 2016 หรือใหม่กว่า (x64) |
| IIS | เปิด W3C logging |
| สิทธิ์ | อ่านโฟลเดอร์ log — การบล็อคจริงต้องเขียน `web.config` ของ site ได้ |
| ทางเลือก | ถ้า `ipSecurity` ถูกล็อค โปรแกรมจะใช้ URL Rewrite แทน |

## ความปลอดภัยเป็นค่าเริ่มต้น

- โหมดแรกคือเฝ้าดู — ไม่แก้ config
- ปฏิเสธการบล็อค IP วงใน / loopback / ช่วง Googlebot
- หลังเขียนกฎ: อ่านไฟล์กลับยืนยัน และถ้าตั้ง `VerifyUrl` แล้วเว็บตอบ 5xx จะย้อน backup ให้เอง
- กฎมีอายุ 1 ชม. → 24 ชม. → 7 วัน แล้วถอนเอง
- ไม่มี telemetry — dashboard เปิดเฉพาะ `127.0.0.1`

## ทดสอบกับอะไรแล้ว

- Windows Server 2025 + IIS 10 บน VM เว็บจริง: บล็อค IP ใน LAN ได้ 403 จริง ถอนคืนได้ เว็บไม่ล่ม
- log ตัวอย่าง 151,555 คำขอ/วัน: 53 การตัดสิน (ควรบล็อค 2, เฝ้าดู 10, ข้อมูล 41)
- กรณี `ipSecurity` ถูกล็อค: สลับไป URL Rewrite อัตโนมัติ XML ยังถูกต้อง

**ยังไม่ได้ทดสอบ:** IIS ต่ำกว่า 10, ARM, การบล็อคระดับ firewall, ตัวติดตั้ง MSI

## ช่วยทดสอบ

เปิด issue ตามแบบฟอร์มที่เตรียมไว้:

- [แจ้งบั๊ก](../../issues/new?template=bug_report.yml)
- [ใช้งานไม่เข้าใจ / ข้อความกำกวม](../../issues/new?template=usability.yml)
- [ตัดสินผิด (false positive / false negative)](../../issues/new?template=detection.yml)
- [ขอฟีเจอร์](../../issues/new?template=feature_request.yml)

อย่าแนบ `web.config` ทั้งไฟล์ถ้ามีข้อมูลลูกค้า — ตัดเฉพาะส่วนที่ใส่กฎ

## โครงสร้าง

```
src/IISGuard.Core     parser, normalize, tailer, detection engine
src/IISGuard.Cli      analyze / replay
src/IISGuard.App      service + SQLite + dashboard
src/IISGuard.Tests    regression tests
```

## สร้างเอง

```powershell
dotnet build IISGuard.sln -c Release
dotnet run --project src/IISGuard.Tests -c Release
```

## สัญญาอนุญาต

MIT
