# FAST Stroke Screening

แอปคัดกรองโรคหลอดเลือดสมอง (Stroke) เบื้องต้นตามหลัก FAST — Face, Arm, Speech, Time

> ⚠️ **คำเตือน:** แอปนี้ใช้เพื่อ **ประเมินเบื้องต้นเท่านั้น** ไม่สามารถทดแทนการวินิจฉัยโดยแพทย์ได้ หากสงสัยว่าเป็น Stroke ให้โทร **1669** ทันที

## โครงสร้างไฟล์

```
fast-screening/
  index.html        หน้าหลัก — เลือกโหมดทดสอบ + ติดตามความคืบหน้า
  face-test.html    ทดสอบความสมมาตรใบหน้า (MediaPipe FaceMesh + Z-score)
  arm-test.html     ทดสอบแขนตก (Gyroscope drift detection)
  speech-test.html  ทดสอบการพูด (Web Speech API + VAD)
  result.html       รวมคะแนน FAST Score
```

## โหมดการใช้งาน

**Full FAST** — ทดสอบครบ 3 ขั้นตอนต่อเนื่อง: ใบหน้า → แขน → การพูด → ผลรวม

**Individual** — เลือกทดสอบทีละอัน ผลสะสมใน sessionStorage และคำนวณรวมเมื่อครบ 3 อัน

## การให้คะแนน

แต่ละ module ให้คะแนน 0-100 และเก็บใน `sessionStorage`:

| Module | วิธีวัด | Weight |
|--------|---------|--------|
| Face   | Z-score ความสมมาตรขณะยิ้ม | 35% |
| Arm    | องศา drift ของแขน (max ซ้าย/ขวา) | 35% |
| Speech | Word accuracy + speech rate + volume stability | 30% |

FAST Score รวม = weighted average → จัดระดับ:
- **70-100:** ปกติ
- **45-69:** เฝ้าระวัง
- **0-44:** ความเสี่ยงสูง (แนะนำพบแพทย์)

## Data Format

แต่ละ module เขียนผลลง sessionStorage:

```javascript
sessionStorage.setItem('fast_<module>', JSON.stringify({
  score: 0-100,
  riskLevel: 'ok' | 'warn' | 'bad',
  passed: boolean,
  breakdown: { /* module-specific */ }
}));
```

## Deploy บน GitHub Pages

1. Push โฟลเดอร์นี้ขึ้น repository
2. Settings → Pages → Source: Deploy from branch → main → / (root)
3. เข้าใช้งานที่ `https://<username>.github.io/<repo>/`

## ข้อจำกัดทางเทคนิค

- ทำงานบน browser เท่านั้น (HTML/JS ล้วน ไม่มี backend)
- Face ต้องใช้กล้องหน้า + MediaPipe FaceMesh (โหลดจาก CDN)
- Arm ต้องใช้ Gyroscope/Accelerometer (มือถือเท่านั้น, iOS ต้องขออนุญาต)
- Speech ต้องใช้ไมโครโฟน + Web Speech API (Chrome/Safari)
- ค่า threshold ยังเป็นค่าเริ่มต้น **ต้อง calibrate กับข้อมูลผู้ใช้จริงก่อนใช้งานจริง**

## สถานะ

Prototype สำหรับทดสอบ logic — ยังไม่ผ่านการ validate ทางคลินิก
ค่า threshold ทั้งหมดต้อง calibrate กับข้อมูลคนปกติและผู้ป่วยจริงก่อนนำไปใช้
