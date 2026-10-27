# Check EDI Handle Inter

เครื่องมือตรวจสอบเอกสารขาเข้าเรือก่อนจัดทำใบขนสินค้าขาเข้า (Thailand ocean import) —
เทียบ **MANIFEST** (รายงานจากสายเรือ, .xls/.xlsx หรือ PDF "CARGO MANIFEST") กับ **ENTER**
(จดหมาย AMENDMENT ของ H.I.T. INTERCON / Handle Inter, .pdf) ทีละ B/L ย่อย แล้วสร้างรายงาน `EDI.xlsx`
แบบไฮไลต์สี บอกจุดที่ต้องแก้ก่อนยื่นใบขน

> **ข้อจำกัด:** หน้าเว็บรองรับเฉพาะ **ENTER ฟอร์ม AMENDMENT ของ Handle Inter** (รหัสฟอร์ม `[FM-CS-01-05]`)
> ถ้าเจอฟอร์มของเจ้าอื่นหน้าเว็บจะแจ้งว่าไม่ใช่ฟอร์มของ Handle Inter · ไฟล์ CNTRS ไม่นำมาเทียบ

## Check EDI Handle Inter บนเว็บ (ให้เพื่อนใช้ได้ ไม่ต้องติดตั้ง)

https://rattanao.github.io/Handle-inter/

1. ลากไฟล์ **MANIFEST** (.xls/.xlsx หรือ PDF) และ **ENTER ของ Handle Inter** มาวางพร้อมกัน
   (ตั้งชื่ออะไรก็ได้ — PDF ที่เป็น CARGO MANIFEST จะถูกจัดเป็น MANIFEST ให้เอง, PDF อื่นเป็น ENTER;
   ENTER หลายไฟล์ใส่พร้อมกันได้; ไฟล์ที่ชื่อมี `CNTRS` จะถูกข้าม)
2. ใส่ SHED NO. ที่แจ้ง (ถ้ามี) แล้วกด **ตรวจสอบ**
3. ดูผลบนหน้าเว็บ (ตารางสีเหมือน EDI.xlsx, ค้นหา/กรอง "แสดงเฉพาะที่ไม่ตรง") หรือกด **ดาวน์โหลด EDI.xlsx**

ไฟล์ถูกประมวลผลในเบราว์เซอร์ของผู้ใช้เท่านั้น ไม่มีการอัปโหลดไปที่ใด

โค้ดหน้าเว็บอยู่ที่ `docs/` (`index.html` + `core.js` — พอร์ตจาก `build_edi.py` layout D ส่วนฟอร์ม Handle,
ฐานเดียวกับ [Check EDI Sinotrans](https://rattanao.github.io/Sinotrans/) และ
[Check EDI AGN](https://rattanao.github.io/check-edi-AGN/))

## ฟอร์ม ENTER ที่รองรับ

หัวเอกสาร + ตาราง B/L หลายใบต่อหน้า (หลายหน้าได้):

```
H.I.T. INTERCON CO., LTD.                          AMENDMENT
VESSEL : SAWASDEE XXXX V.2609S     M B/L : HASLC01260000123
STATUS : *** เร่งเปิดตู้เข้าโกดังด่วน ***
B/L CHANGE NO.     MARKS & NUMBERS   QUANTITY   DESCRIPTION        CONSIGNEE          GROSS WEIGHT
HASLC01260000123   ABC-00001         3 WOODEN   STEEL SHEET        ABC (THAILAND)     5,320.000 KGS.
                   MADE IN CHINA     CASES      IN COIL            CO.,LTD.               2.301 CBM
Cont No : HALU1234567/20'GP
...
                   TOTAL 18 PACKAGES                                           12,437.740 KGS.
[FM-CS-01-05] Revision : 01/2555
```

- แยกคอลัมน์ตามตำแหน่ง: MARKS < 300 | QUANTITY < 375 | DESCRIPTION < 530 | CONSIGNEE < 700 | G.W./CBM ≥ 700
  (ขอบเลื่อนตามหัว `QUANTITY` ของแต่ละหน้า — บางไฟล์ทั้งหน้าเลื่อนซ้ายราว 6pt)
- เช็คฟอร์มทีละหน้าจากรหัส `[FM-CS-01-05]` + หัว `B/L CHANGE NO.` (ชื่อบริษัทมีแค่หน้าแรก)
- STATUS หัวเอกสารใช้กับทุก B/L; ข้อความไทยที่ฟอนต์ถอดเพี้ยน (`เรรงเปปดตตเขตาโกดดงดรวน`, `ลากตต`) แปลงกลับให้
- FreeText ที่ลูกค้าพิมพ์ทับ (STATUS/DG/REEFER/TRANSIT/NOTIFY) ใช้กับ B/L ที่อยู่ใต้ข้อความนั้น
- STATUS: `CY`/`CY/CY`/`ลากตู้` = CY, `LCL`/`ขน` = LCL, `CFS`/`เปิดตู้` = LCL/CFS

## สิ่งที่ตรวจ

CONSIGNEE · TAX ID vs NOTIFY · CONTAINER · STATUS · จำนวน/ชนิดหีบห่อ · G.W. · MEAS. · MARKS · DESCRIPTION ·
REEFER · DG · TRANSIT · SHED NO. · ประเทศปลายทาง · CARGO MOVEMENT — สีแดง ⚠ = ไม่ตรง, ส้ม = ต้องยืนยัน,
เหลือง = ไม่มี ENTER เทียบ, เขียว = ผ่าน · B/L ที่มีใน ENTER แต่ไม่มีใน MANIFEST = วิกฤต

## ใช้แบบ Python (ในเครื่อง)

```
pip install -r requirements.txt
python build_edi.py --manifest input/MANIFEST.xls --enter input/ENTER.pdf --outdir output [--shed 2813]
```

## ตรวจความตรงกันของหน้าเว็บกับ Python

2026-10-27: งาน SAWASDEE SPICA 2609S (8 B/L) ตรงกัน 252/252 ช่อง (ค่าและสี) ทั้งแบบไม่ใส่ SHED
และ `--shed 2813` · หน้าเว็บปฏิเสธ ENTER ของเจ้าอื่นทั้ง 19 ฟอร์มที่มีตัวอย่าง และอ่านหน้า Handle ใน ENTER
ชุดรวมของ hahm ได้ถูกต้อง

ไม่มีข้อมูลลูกค้าจริงใน repo นี้ (`.gitignore` กัน `input/`, `output/`, `*.xls*`, `*.pdf`)

Project by Ms. Rattana Mudlae — Position Import DOC.
