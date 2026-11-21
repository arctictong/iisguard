# IISGuard v0.9.5-beta

ไฟล์: `iisguard-win-x64.zip` (Windows x64, ไฟล์เดียว + `wwwroot\fonts` + `scripts`, ไม่ต้องติดตั้ง .NET)

## เปลี่ยนอะไรในรุ่นนี้

รอบนี้ทดสอบบน **Windows Server 2012 R2** จริง (IIS 8, URL Rewrite 2.1)

1. **`/api/setup/status` ตอบ 500 เมื่ออ่านโฟลเดอร์ log ไม่ได้** — บัญชีผู้ใช้ทั่วไปบน 2012 R2 ถูกปฏิเสธสิทธิ์ที่ `C:\inetpub\logs\LogFiles\W3SVC1` ทำให้ endpoint โยน `UnauthorizedAccessException` ตอนนี้ตอบ 200 พร้อม `logReadable=false` และหน้า `/setup` ขึ้น "ไม่มีสิทธิ์อ่านโฟลเดอร์นี้" (`/api/status` ก็กันแบบเดียวกัน)
2. **zip ไม่มี `scripts\install-service.ps1`** ทั้งที่ README บอกให้รัน — เพิ่ม `scripts\install-service.ps1` และ `uninstall-service.ps1` เข้าแพ็กเกจแล้ว
3. **สคริปต์ PowerShell parse ไม่ผ่านบน 2012 R2** — ไฟล์เป็น UTF-8 ไม่มี BOM ทำให้ PowerShell 4.0/5.1 อ่านข้อความไทยผิดแล้ว syntax error ตอนนี้บันทึกเป็น UTF-8 with BOM (install/uninstall/publish)
4. **ค่าเริ่มต้น `-Source` ของ install-service** ชี้ `dist/iisguard` ซึ่งไม่มีใน zip ตอนนี้เดาจากตำแหน่งสคริปต์ (โฟลเดอร์ที่มี `iisguard-app.exe`)

## ทดสอบบน Windows Server 2012 R2 (IIS 8)

| ตรวจ | ผล |
|---|---|
| แอปรันด้วย .NET 8 self-contained | ผ่าน (ไม่ต้องติดตั้ง .NET, ใช้ VC++ ที่มีอยู่) |
| อ่าน log (324 บรรทัดสังเคราะห์) | ผ่าน 320 events |
| `/api/setup/status` เมื่อ log อ่านได้ | 200 · `logReadable=true` |
| `/api/setup/status` เมื่อ log ถูกปฏิเสธสิทธิ์ | **200 · `logReadable=false`** (เดิม 500) |
| findings + จำนวนคำขอ | 3 เรื่อง · 230/90 คำขอ ตรงกัน |
| เขียนกฎบล็อคลง web.config (สำเนา) | ผ่าน — URL Rewrite, XML valid, มี backup |
| ถอนกฎ | ผ่าน — กฎหาย, XML valid |
| ฟอนต์ Noto Sans Thai | 200 `font/woff2` |
| `install-service.ps1` แบบไม่ใช่ admin | parse ผ่าน และแจ้ง "ต้องรันแบบ Administrator" ถูกต้อง |

## ยังไม่ได้ทดสอบบน 2012 R2

- เขียน `web.config` ของ **site จริง** และติดตั้ง Windows Service — บัญชีทดสอบเป็น standard user (ต้องรันเป็น Administrator)
- อ่านรายชื่อเว็บจาก `applicationHost.config` (สิทธิ์เดียวกัน)

## หมายเหตุ

- โปรแกรมรันได้บน 2012 R2 โดยไม่ต้องมี ESU; แต่ 2012 R2 หมดรอบซัพพอร์ตของ Microsoft แล้ว (ต.ค. 2023)
- .NET 8 หมดรอบซัพพอร์ต 10 พ.ย. 2026 — ถ้าต้องรองรับ 2012 ระยะยาว ควรพิจารณา build บน .NET Framework 4.8

## ประวัติ

- v0.9.4-beta: ฟอนต์ Noto Sans Thai ทั้งโปรแกรม
- v0.9.3-beta: แก้จำนวนคำขอ, หัวเรื่อง /setup, กล่องข้อความเมื่อรันแบบ redirect
- v0.9.2-beta: แก้ตัวหา log, ตัวตรวจ web.config, ข้อความแยกกรณีไม่มีสิทธิ์
- v0.9.1-beta: แก้ 3 จุดจากการทดสอบผู้ใช้ใหม่ครั้งแรก
- v0.9-beta: รุ่นทดสอบแรก
