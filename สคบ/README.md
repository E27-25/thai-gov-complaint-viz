# 🛒 สคบ — เรื่องร้องทุกข์ผู้บริโภค กทม. (OCPB)

สถิติ**เรื่องร้องทุกข์ผู้บริโภค**ในเขตกรุงเทพมหานคร จากสำนักงานคณะกรรมการคุ้มครองผู้บริโภค (สคบ. / OCPB)

| | |
|---|---|
| **ช่วงเวลา** | ปี 2569 (สแนปช็อตปีปัจจุบัน) |
| **ขอบเขต** | 50 เขต · 6 กลุ่มเขต (กทม.) |
| **ปริมาณ** | 8,461 เรื่อง (ยุติ 5,200 · กำลังดำเนินการ 3,261) |

<p align="center">
  <img src="../assets/ocpb_map.png" width="46%"/>
  <img src="../assets/ocpb_regression.png" width="52%"/>
</p>

---

## 🔌 API — วิธีดึงข้อมูล

**Data catalog:** https://gdcatalog.go.th/dataset/gdpublish-opendata-03
**Dashboard:** https://ocpbconnect.ocpb.go.th/Report/Detail?report_id=7EF99779-95B5-4541-A374-378B1CD11140
**เอกสาร API (PDF):** [คู่มือ Web Service ComplaintBangkok](https://ocpb.gdcatalog.go.th/dataset/10b080fa-7f7e-43b3-8f17-32a8edacd88f/resource/b5ff7dde-d734-4c1f-9584-0a06b57755f6/download/untitled.pdf)

### Official Web Service — `ComplaintBangkok`

```
GET https://ocpbconnect.ocpb.go.th/api/Complaint/ComplaintBangkok
```

| Header | Required | Type | คำอธิบาย |
|--------|----------|------|----------|
| `apiKey` | ✅ | String | API Key |
| `password` | ✅ | String | รหัสผ่าน |

| Query param | Type | คำอธิบาย |
|-------------|------|----------|
| `yearStart` | Int | ปีเริ่มต้น |
| `yearEnd` | Int | ปีสิ้นสุด |

<details><summary><b>📋 Response schema</b></summary>

| Field | Type | คำอธิบาย |
|-------|------|----------|
| `complaintTotal` | Int | จำนวนเรื่องทั้งหมด |
| `complaintInprogressTotal` | Int | กำลังดำเนินการ |
| `complaintTerminateTotal` | Int | ยุติแล้ว |
| `complaintTumbon[]` | Array | แยกตาม ปี · ประเภท · สถานะ · เขต · แขวง |
| `…​.year / .count` | Int | ปี / จำนวนเรื่อง |
| `…​.categoryName` | String | ประเภทการร้องทุกข์ |
| `…​.statusName` | String | สถานะ |
| `…​.ampher / .tumbon` | String | เขต / แขวง |

</details>

### ⚠️ หมายเหตุการเข้าถึงข้อมูล

Web Service ทางการต้องใช้ **`apiKey` + `password`** ซึ่งไม่ได้เผยแพร่สาธารณะ
ข้อมูลในโปรเจกต์นี้จึงดึงจาก **public Tableau dashboard** ของ สคบ. (ocpbconnect) แทน โดยมีข้อจำกัด:

- เป็น**สแนปช็อตปีปัจจุบัน (2569)** เท่านั้น — ไม่มีข้อมูลย้อนหลัง (จึงไม่มี time series)
- dashboard **ไม่เปิดเผยชื่อ 4 หมวดประเภท** (แสดงเป็น หมวด 1–4)
- ระดับ**แขวง**และ**ประเภทตามปี**ต้องใช้ Web Service API ที่มี credential

> หากมี `apiKey`+`password` สามารถขยายเป็น time series ย้อนหลัง + ชื่อประเภทจริง + ระดับแขวงได้

---

## 📁 ไฟล์ในโฟลเดอร์

| ไฟล์ | คำอธิบาย |
|------|----------|
| `ocpb_viz.ipynb` | Notebook หลัก (22 เซลล์) — EDA · แผนที่ · K-Means · Ridge · **self-contained** |
| `ocpb_viz.html` | Dashboard แบบ interactive |
| `ocpb_complaint_bkk.csv` | 50 เขต · 4 หมวด · กลุ่มเขต · ยอดรวม |
| `bkk_districts.geojson` | ขอบเขตเขต กทม. (50 เขต + ข้อมูลประชากร) |

## ♻️ Self-contained

เซลล์โหลดข้อมูล **ฝังสแนปช็อต 50 เขตไว้ในตัว notebook** และจะ *สร้างไฟล์ CSV ให้อัตโนมัติถ้าไม่พบ* ส่วน geojson ก็ดึงจาก URL เอง —
เปิดใน Google Colab แล้ว *Run all* ได้เลยโดยไม่ต้องอัปโหลดไฟล์ใด ๆ
(ต่างจาก ปปส. ที่ auto-fetch จาก API ได้ — สคบ. เป็นสแนปช็อตปีเดียวเพราะ dashboard สาธารณะไม่มี API ย้อนหลัง จึงฝังข้อมูลไว้แทน)

## 📊 สิ่งที่อยู่ใน Notebook

1. **Treemap** — กลุ่มเขต (6) → เขต (50)
2. **แผนที่ Choropleth** — เรื่องร้องทุกข์ต่อประชากร 10,000 คน
3. **องค์ประกอบประเภท** แยกตามกลุ่มเขต + **โดนัทสถานะ**
4. **Top 15 เขต**
5. **K-Means (k=4)** + PCA scatter + cluster profiles
6. **🔮 Ridge regression** — ทำนายจำนวนเรื่องร้องทุกข์ (ด้านล่าง)

## 🔮 Predictive Model — Ridge Regression

ทำนาย*จำนวนเรื่องร้องทุกข์*รายเขต จาก **ประชากร + สัดส่วนประเภท (4 หมวด) + กลุ่มเขต (one-hot)**

- **ประเมิน:** 5-fold cross-validation → **R² 0.45 · MAE 42**
- ประชากรมี correlation กับจำนวนเรื่อง ≈ **0.68** และเป็น feature ที่มีน้ำหนักมากที่สุด
- ส่วนที่เหลืออธิบายไม่ได้ = ปัจจัยเชิงเศรษฐกิจ/พาณิชย์ที่ไม่มีในข้อมูลเปิด (จตุจักรเป็น outlier ชัดเจน)

**K-Means 7 features:** สัดส่วนประเภท 4 + log_volume + top_share (ความกระจุกตัว) + entropy (ความหลากหลาย)

<p align="center">
  <img src="../assets/ocpb_treemap.png" width="32%"/>
  <img src="../assets/ocpb_composition.png" width="32%"/>
  <img src="../assets/ocpb_clusters.png" width="32%"/>
</p>

## 🗺️ BKK 6 กลุ่มเขต

`กรุงเทพกลาง` · `กรุงเทพใต้` · `กรุงเทพเหนือ` · `กรุงเทพตะวันออก` · `ธนบุรีเหนือ` · `ธนบุรีใต้` (แบ่งตามการจัดกลุ่มเขตของ กทม.)
