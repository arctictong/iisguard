# IISGuard v0.9.7-beta

ไฟล์: `iisguard-win-x64.zip` (Windows x64, ไฟล์เดียว + `wwwroot\fonts` + `scripts`, ไม่ต้องติดตั้ง .NET)

## เปลี่ยนอะไรในรุ่นนี้

ทดสอบบน **Windows Server 2012 R2** ด้วยบัญชี Administrator (IIS 8, URL Rewrite 2.1)

1. **ตัวตรวจ web.config เลือกไฟล์ของ sub-application แทน site หลัก** — บนเซิร์ฟเวอร์จริงที่มี application ซ้อน (เช่น `/elib/report/api`) โปรแกรมเลือก `web.config` ของตัวที่แก้ล่าสุด ซึ่งทำให้กฎมีผลเฉพาะ path ย่อย ตอนนี้ให้ site root (`path="/"`) มาก่อนเสมอ
2. **ถอนกฎแล้วเหลือ `<rewrite><rules /></rewrite>` ค้างในไฟล์** — ทำให้ `web.config` ไม่กลับสภาพเดิม ตอนนี้ลบ container ที่ว่างออกให้ (`rules` → `rewrite`, `add` → `ipSecurity` → `security`)
3. **ข้อความหลังติดตั้ง service บอกพอร์ตผิด** — เดิมพิมพ์ `8787` ตายตัว ตอนนี้อ่านจาก `appsettings.json` ที่ติดตั้งจริง

## ทดสอบบน IIS จริง (2012 R2, Administrator)

| ตรวจ | ผล |
|---|---|
| ตรวจ `web.config` ของ site อัตโนมัติ | ถูก — `C:\inetpub\wwwroot\web.config` (site root) |
| บล็อค IP ลง `web.config` จริง | ผ่าน — เขียนผ่าน URL Rewrite, XML valid |
| เว็บยังทำงานหลังเขียนกฎ | **HTTP 200** |
| ถอนกฎ | ผ่าน — ไม่มีกฎค้าง, ไม่มี container ว่าง, XML valid |
| ไฟล์หลังถอน | เนื้อหาเดิม (ต่างแค่ตัวพิมพ์ `utf-8`/`UTF-8` ใน header) |
| ติดตั้ง Windows Service (`install-service.ps1`) | ผ่าน — service **Running**, dashboard ตอบ 200, อ่าน log ได้ |
| ถอน service (`uninstall-service.ps1 -RemoveData`) | ผ่าน — service หาย, โฟลเดอร์ข้อมูลถูกลบ |

## ยังไม่ได้ทดสอบ

- Windows Server 2016/2019/2022 (ทดสอบ 2012 R2 กับ 2025)
- ARM, การบล็อคระดับ firewall, ตัวติดตั้ง MSI

## หมายเหตุ

- การทดสอบเขียนไฟล์จริงทำกับ `C:\inetpub\wwwroot\web.config` ซึ่งเป็น site ทดสอบ: สำรองไว้ที่ `C:\IISGuard\backup` และคืนค่าแล้ว
- โปรแกรมรันได้บน 2012 R2 โดยไม่ต้องมี ESU แต่ 2012 R2 หมดรอบซัพพอร์ตของ Microsoft แล้ว (ต.ค. 2023)
- .NET 8 หมดรอบซัพพอร์ต 10 พ.ย. 2026 — ถ้าต้องรองรับ 2012 ระยะยาว ควรพิจารณา build บน .NET Framework 4.8

## ประวัติ

- v0.9.5-beta: แก้ 500 เมื่ออ่าน log ไม่ได้, ใส่ scripts ใน zip, .ps1 เป็น UTF-8 BOM
- v0.9.4-beta: ฟอนต์ Noto Sans Thai ทั้งโปรแกรม
- v0.9.3-beta: แก้จำนวนคำขอ, หัวเรื่อง /setup, กล่องข้อความเมื่อรันแบบ redirect
- v0.9.2-beta: แก้ตัวหา log, ตัวตรวจ web.config, ข้อความแยกกรณีไม่มีสิทธิ์
- v0.9.1-beta: แก้ 3 จุดจากการทดสอบผู้ใช้ใหม่ครั้งแรก
- v0.9-beta: รุ่นทดสอบแรก
