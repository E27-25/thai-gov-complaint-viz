# 🔦 ปปส — เบาะแส/ร้องเรียนยาเสพติด (ONCB)

การตรวจสอบ**เบาะแส/ร้องเรียนยาเสพติด**รายเดือน แยกตามจังหวัด จากสำนักงานคณะกรรมการป้องกันและปราบปรามยาเสพติด (ป.ป.ส. / ONCB)

| | |
|---|---|
| **ช่วงเวลา** | ปีงบประมาณ 2560–2569 (114 เดือน · ล่าสุด มิ.ย. 2569) |
| **ขอบเขต** | 77 จังหวัด · 10 ภาค ปปส. |
| **ปริมาณ** | 8,669 แถว · 194,795 เบาะแส |
| **อัตราพบพฤติการณ์** | 33% ของที่ตรวจสอบ |

<p align="center">
  <img src="../assets/pps_map.png" width="46%"/>
  <img src="../assets/pps_forecast.png" width="52%"/>
</p>

### 🔎 ข้อค้นพบสำคัญ

- 📈 **เบาะแสพุ่งขึ้น ~2.5 เท่า** — เฉลี่ยจาก **1,288 → 3,278 เรื่อง/เดือน** (2560 → 2569) เร่งตัวแรงในปี 2568–2569
- 🏙️ **กระจุกที่เมืองใหญ่** — กรุงเทพฯ 26,035 · ปทุมธานี 5,766 · ชลบุรี 5,654 · ขอนแก่น 5,485 · สมุทรปราการ 5,356
- 🔍 **ตรวจแล้วพบพฤติการณ์ 33%** ของเบาะแสที่ตรวจสอบทั้งหมด
- 🧩 **K-Means แยก 5 กลุ่ม** — ตั้งแต่กลุ่ม *ปริมาณสูง·พบพฤติการณ์ต่ำ 20%* ถึงกลุ่ม *พบพฤติการณ์สูง 52%*

---

## 🔌 API — วิธีดึงข้อมูล

**Data catalog:** https://gdcatalog.go.th/dataset/gdpublish-complain-04
**API portal:** https://data.oncb.go.th/complain_021

ข้อมูลดึงจาก ONCB API สาธารณะ (ไม่ต้อง auth) 2 endpoint:

### 1) รายเดือน (ใช้เป็นข้อมูลหลัก) — `complainVerify`

```
GET https://dataapi.oncb.go.th/suppress/complainVerify/{NEWS_MONTH_CODE}/{NEWS_YEAR}
```

| Query param | Type | คำอธิบาย |
|-------------|------|----------|
| `NEWS_MONTH_CODE` | String | รหัสเดือน `01`–`12` |
| `NEWS_YEAR` | Integer | ปี พ.ศ. (2560–2569) |

คืนค่า JSON `{"data":[ {…per province…} ]}` — โครงสร้างเป็น **matrix 5 กลุ่มเป้าหมาย × 3 ผลตรวจสอบ**:

<details><summary><b>📋 Data dictionary (คลิกเพื่อดูทั้งหมด)</b></summary>

| Field | คำอธิบาย |
|-------|----------|
| `PROV_ID` / `PROV_NAME` | รหัส / ชื่อจังหวัด |
| `NEWS_MONTH_CODE` / `NEWS_YEAR` | เดือน / ปี พ.ศ. |
| `COMPLAIN_ALL` | ร้องเรียนทั้งหมด (เรื่อง) |
| `ALL1` / `ALL2` / `ALL3` | ผลตรวจสอบ: พบพฤติการณ์ / ไม่พบ / พิสูจน์ทราบไม่ได้ |
| `ALL123` | ตรวจสอบทั้งหมด |
| **กลุ่มเป้าหมาย 5 กลุ่ม** (แต่ละกลุ่มมี suffix `1`/`2`/`3`/`123` = ผลตรวจสอบ) | |
| `FOUNDS`, `F1`–`F123` | **กลุ่ม 1:** มีตัวตน · พบพฤติการณ์ |
| `NOTFOUNDS`, `NF1`–`NF123` | **กลุ่ม 2:** มีตัวตน · ไม่พบพฤติการณ์ |
| `UNKNOW`, `U1`–`U123` | **กลุ่ม 3:** ไม่มีตัวตนใน ทร.14 |
| `PLACE`, `P1`–`P123` | **กลุ่ม 4:** กลุ่มสถานที่ |
| `AREA`, `A1`–`A123` | **กลุ่ม 5:** กลุ่มพื้นที่ |

> `COMPLAIN_ALL` = `FOUNDS + NOTFOUNDS + UNKNOW + PLACE + AREA` (แยกตามกลุ่มเป้าหมาย) ≈ `ALL1 + ALL2 + ALL3` (แยกตามผลตรวจสอบ)

</details>

### 2) รายปี — `complain` (ใช้ map จังหวัด → ภาค ปปส.)

```
GET https://dataapi.oncb.go.th/suppress/complain/{year}     # year = 2558–2566
```
ให้ `REG_ONCB` / `REG_NAME` (ภาค ปปส. 1–9 + กทม.) สำหรับสร้าง treemap ลำดับชั้น ภาค → จังหวัด

### ตัวอย่างดึงข้อมูลทั้งหมด (Python)

```python
import urllib.request, json, csv
rows = []
for year in range(2560, 2570):   # 2569 มีถึงเดือน 06 (มิ.ย. 2026)
    for m in range(1, 13):
        url = f"https://dataapi.oncb.go.th/suppress/complainVerify/{m:02d}/{year}"
        with urllib.request.urlopen(url, timeout=30) as r:
            rows += json.load(r).get("data", [])
# -> บันทึกเป็น oncb_verify_monthly.csv (encoding='utf-8-sig')
```

---

## 📁 ไฟล์ในโฟลเดอร์

| ไฟล์ | คำอธิบาย |
|------|----------|
| `pps_viz.ipynb` | Notebook หลัก (22 เซลล์) — EDA · แผนที่ · K-Means · SARIMA · **auto-fetch ข้อมูลล่าสุด** |
| `pps_viz.html` | Dashboard แบบ interactive (เปิดในเบราว์เซอร์ได้เลย) |
| `oncb_verify_monthly.csv` | ข้อมูลรายเดือน 8,669 แถว (2560–2569) |
| `oncb_complaint_04.csv` | ข้อมูลรายปี (สำหรับ map ภาค) |
| `th_provinces.geojson` | ขอบเขตจังหวัด (77 จังหวัด, มี `pro_th`) |

## ♻️ Self-updating

เซลล์โหลดข้อมูลจะ **auto-fetch จาก ONCB API ให้อัตโนมัติถ้าไม่พบไฟล์ CSV** (ดึงตั้งแต่ 2560 ถึงปีปัจจุบัน) —
เปิดใน Google Colab แล้ว *Run all* ได้เลยโดยไม่ต้องอัปโหลดไฟล์ และจะได้ข้อมูลล่าสุดเสมอ
หัวข้อกราฟทั้งหมดคำนวณช่วงปีจากข้อมูลจริง (`{y_min}–{y_max}`) จึงไม่ล้าสมัยเมื่อมีข้อมูลใหม่

## 📊 สิ่งที่อยู่ใน Notebook

1. **Treemap** — ภาค ปปส. → จังหวัด (จำนวนเบาะแส)
2. **แผนที่ Choropleth** — เบาะแสรายจังหวัด (สีเข้ม = มาก)
3. **Time series** — เบาะแสรายเดือน + ผลตรวจสอบ 3 แบบ
4. **Top 15 จังหวัด**
5. **K-Means (k=5)** + PCA scatter + cluster profiles
6. **🔮 SARIMA forecast** — พยากรณ์รายเดือน (ด้านล่าง)

<p align="center">
  <img src="../assets/pps_treemap.png" width="49%" alt="treemap ภาค→จังหวัด"/>
  <img src="../assets/pps_timeseries.png" width="49%" alt="time series เบาะแสรายเดือน"/>
</p>
<div align="center"><sub>Treemap ภาค ปปส. → จังหวัด · Time series เบาะแสรายเดือน + ผลตรวจสอบ 3 แบบ</sub></div>

## 🔮 Predictive Model — SARIMA

`SARIMA(1,1,1)(1,1,1)₁₂` พยากรณ์จำนวนเบาะแสรายเดือนทั้งประเทศ

- **Train/test:** กันข้อมูล 12 เดือนสุดท้ายไว้ทดสอบ
- **ผล (holdout 12 เดือน):** MAE **610** · RMSE **691** · MAPE **18.8%**
- refit ข้อมูลทั้งหมด → พยากรณ์ล่วงหน้า 12 เดือน พร้อมช่วงความเชื่อมั่น 95% (เบาะแสพุ่งสูงในปี 2568–2569)

**K-Means 12 features:** สัดส่วนกลุ่มเป้าหมาย 5 + สัดส่วนผลตรวจสอบ 3 + found_rate + verify_rate + log_volume + trend

<p align="center">
  <img src="../assets/pps_clusters.png" width="49%"/>
  <img src="../assets/pps_top.png" width="49%"/>
</p>
