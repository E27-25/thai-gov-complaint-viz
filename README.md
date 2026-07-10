<div align="center">

# 🇹🇭 Thai Government Complaint Data — Visualization & ML

**วิเคราะห์ข้อมูลเปิดภาครัฐไทย 2 ชุด เกี่ยวกับเรื่องร้องเรียน/ร้องทุกข์**
ดึงข้อมูลจาก API จริง → EDA → แผนที่ → Clustering → Predictive Model → Interactive Dashboard

<br>

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-Interactive-3F4F75?logo=plotly&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-KMeans%20%7C%20Ridge-F7931E?logo=scikitlearn&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-SARIMA-1f77b4)
![Open Data](https://img.shields.io/badge/data-Thai%20Open%20Gov-2a9d8f)

<br>

<img src="assets/pps_map.png" width="48%" alt="แผนที่เบาะแสยาเสพติดรายจังหวัด"/>
&nbsp;
<img src="assets/ocpb_map.png" width="48%" alt="แผนที่เรื่องร้องทุกข์ต่อประชากร กทม."/>

<sub>ซ้าย: เบาะแสยาเสพติดรายจังหวัด (ปปส.) · ขวา: เรื่องร้องทุกข์ผู้บริโภคต่อประชากร รายเขต กทม. (สคบ.)</sub>

</div>

---

## ✨ ไฮไลต์

- 🔌 **ดึงข้อมูลจาก API รัฐจริง** — ONCB (ป.ป.ส.) แบบ query ตรง + OCPB (สคบ.) จาก Tableau dashboard
- 🗺️ **แผนที่ Choropleth** — ระดับจังหวัด (ทั้งประเทศ) และระดับเขต (กทม. ต่อประชากร)
- 🧩 **Unsupervised** — K-Means + PCA จัดกลุ่มพื้นที่ตามโปรไฟล์
- 🔮 **Predictive** — SARIMA time-series forecast + Ridge regression (แสดง *predicted vs actual*)
- 📊 **Interactive dashboard** — export เป็น HTML ไฟล์เดียว เปิดได้ทันที (มี easter egg 🥚)
- ♻️ **Self-updating** — notebook ปปส. auto-fetch ข้อมูลล่าสุดจาก API เอง

---

## 📦 สองชุดข้อมูล

| | 🔦 [`ปปส/`](ปปส/) — ONCB | 🛒 [`สคบ/`](สคบ/) — OCPB |
|---|---|---|
| **หน่วยงาน** | สนง.ป้องกันและปราบปรามยาเสพติด | สนง.คณะกรรมการคุ้มครองผู้บริโภค |
| **ข้อมูล** | เบาะแส/ร้องเรียนยาเสพติด | เรื่องร้องทุกข์ผู้บริโภค กทม. |
| **ช่วงเวลา** | รายเดือน 2560–2569 (114 เดือน) | สแนปช็อตปี 2569 |
| **ขอบเขต** | 77 จังหวัด · 10 ภาค | 50 เขต · 6 กลุ่มเขต |
| **ปริมาณ** | 8,669 แถว · 194,795 เบาะแส | 50 เขต · 8,461 เรื่อง |
| **โมเดล** | **SARIMA** forecast · MAPE 18.8% | **Ridge** regression · R² 0.45 |

<div align="center"><sub>แต่ละโฟลเดอร์: <code>*_viz.ipynb</code> · <code>*_viz.html</code> · <code>.csv</code> · <code>.geojson</code> · README + API docs</sub></div>

---

## 🔦 ปปส — เบาะแสยาเสพติด (ONCB)

> รายเดือน 114 เดือน · แต่ละจังหวัด/เดือน แยกเป็น **matrix 5 กลุ่มเป้าหมาย × 3 ผลตรวจสอบ**

### 🔮 SARIMA Forecast — Actual vs Predicted

<div align="center">
<img src="assets/pps_forecast.png" width="90%" alt="SARIMA forecast"/>
</div>

พยากรณ์เบาะแสรายเดือน · กันข้อมูล 12 เดือนสุดท้ายไว้ทดสอบ → **MAE 610 · RMSE 691 · MAPE 18.8%**
แล้วพยากรณ์ล่วงหน้า 12 เดือน พร้อมช่วงความเชื่อมั่น 95% — เห็นชัดว่าเบาะแส **พุ่งสูงขึ้นมากในปี 2568–2569** (จาก ~1,500 → ~3,700 เรื่อง/เดือน)

### 🗺️ แผนที่ · Treemap · K-Means

<div align="center">
<img src="assets/pps_treemap.png" width="32%" alt="treemap ภาค→จังหวัด"/>
<img src="assets/pps_clusters.png" width="32%" alt="K-Means clusters"/>
<img src="assets/pps_timeseries.png" width="32%" alt="time series"/>
</div>

**K-Means (k=5 · 12 features):** จัดกลุ่มจังหวัดตามโปรไฟล์การตรวจสอบ — ตั้งแต่กลุ่ม *ปริมาณสูง·พบพฤติการณ์ต่ำ 20%* จนถึงกลุ่ม *พบพฤติการณ์สูง 52%*

<div align="right"><b><a href="ปปส/">อ่านรายละเอียด + API docs →</a></b></div>

---

## 🛒 สคบ — เรื่องร้องทุกข์ผู้บริโภค กทม. (OCPB)

> 50 เขต · จัดกลุ่มเป็น **6 กลุ่มเขต** ตามการแบ่งของ กทม. · มีข้อมูลประชากรจาก geojson สำมะโน

### 🔮 Ridge Regression — Predicted vs Actual

<div align="center">
<img src="assets/ocpb_regression.png" width="62%" alt="Ridge regression"/>
</div>

ทำนายจำนวนเรื่องร้องทุกข์รายเขต จาก **ประชากร + สัดส่วนประเภท + กลุ่มเขต** (5-fold CV) → **R² 0.45 · MAE 42**
ประชากรเป็นตัวขับหลัก (corr ≈ 0.68) · จตุจักรเป็น outlier ชัดเจน

### 🗺️ แผนที่ · Treemap · K-Means · สถานะ

<div align="center">
<img src="assets/ocpb_treemap.png" width="32%" alt="treemap กลุ่มเขต→เขต"/>
<img src="assets/ocpb_clusters.png" width="32%" alt="K-Means clusters"/>
<img src="assets/ocpb_status.png" width="32%" alt="donut สถานะ"/>
</div>

**K-Means (k=4 · 7 features):** จัดกลุ่ม 50 เขตตามโปรไฟล์ประเภทเรื่องร้องทุกข์ + ปริมาณ + ความกระจุกตัว

<div align="right"><b><a href="สคบ/">อ่านรายละเอียด + API docs →</a></b></div>

---

## 🔁 Pipeline (เหมือนกันทั้ง 2 ชุด)

```mermaid
flowchart LR
    A[API / Dashboard] --> B[ดึงข้อมูล CSV]
    B --> C[EDA + Treemap]
    B --> D[แผนที่ Choropleth]
    B --> E[K-Means → PCA 2D]
    B --> F[Predictive Model<br/>SARIMA / Ridge]
    C & D & E & F --> G[Export HTML Dashboard]
```

---

## ▶️ วิธีใช้งาน

```bash
pip install pandas plotly scikit-learn statsmodels

# เปิด notebook แล้ว Run all — ได้ dashboard HTML
jupyter notebook ปปส/pps_viz.ipynb     # หรือ  สคบ/ocpb_viz.ipynb
```

- 🟢 **Google Colab:** เปิดแล้ว *Run all* ได้เลย — เซลล์แรกติดตั้ง dependency และ notebook ปปส. **auto-fetch ข้อมูลล่าสุดจาก API เอง** (ไม่ต้องอัปโหลดไฟล์)
- 🌐 **ดู dashboard ทันที:** เปิดไฟล์ [`ปปส/pps_viz.html`](ปปส/pps_viz.html) หรือ [`สคบ/ocpb_viz.html`](สคบ/ocpb_viz.html) ในเบราว์เซอร์ (ลองกดปุ่มลอย 🚨 / 🛒 มุมขวาล่าง 👀)

---

## 📁 โครงสร้างโปรเจกต์

```
thai-gov-complaint-viz/
├── README.md
├── assets/                     # 12 รูปผลลัพธ์ (แผนที่ · โมเดล · cluster)
├── ปปส/                        # ONCB — เบาะแสยาเสพติด
│   ├── pps_viz.ipynb           # notebook 22 เซลล์ (self-updating)
│   ├── pps_viz.html            # interactive dashboard
│   ├── oncb_verify_monthly.csv # ข้อมูลรายเดือน 2560–2569
│   ├── oncb_complaint_04.csv   # ข้อมูลรายปี (map ภาค)
│   ├── th_provinces.geojson    # ขอบเขต 77 จังหวัด
│   └── README.md               # + API documentation
└── สคบ/                        # OCPB — เรื่องร้องทุกข์ผู้บริโภค
    ├── ocpb_viz.ipynb          # notebook 22 เซลล์
    ├── ocpb_viz.html           # interactive dashboard
    ├── ocpb_complaint_bkk.csv  # 50 เขต
    ├── bkk_districts.geojson   # ขอบเขตเขต + ประชากร
    └── README.md               # + API documentation
```

---

## 🧰 Tech stack

| หมวด | เครื่องมือ |
|------|-----------|
| Data | **pandas** · REST API (urllib) |
| Viz | **Plotly** — treemap · choropleth · scatter · time series · donut |
| Unsupervised | **scikit-learn** — StandardScaler → KMeans → PCA |
| Predictive | **statsmodels** SARIMA · **scikit-learn** Ridge |
| Maps | GeoJSON choropleth (province & Bangkok-district boundaries) |

---

## 📚 แหล่งข้อมูล

| ชุด | Data catalog | API |
|-----|-------------|-----|
| ปปส | [gdpublish-complain-04](https://gdcatalog.go.th/dataset/gdpublish-complain-04) | [data.oncb.go.th](https://data.oncb.go.th/complain_021) |
| สคบ | [gdpublish-opendata-03](https://gdcatalog.go.th/dataset/gdpublish-opendata-03) | [OCPB Connect](https://ocpbconnect.ocpb.go.th/Report/Detail?report_id=7EF99779-95B5-4541-A374-378B1CD11140) |

**ขอบเขตแผนที่:** [chingchai/OpenGISData-Thailand](https://github.com/chingchai/OpenGISData-Thailand) (จังหวัด) · [pcrete/gsvloader-demo](https://github.com/pcrete/gsvloader-demo) (เขต กทม.)

<div align="center">
<br>
<sub>⚠️ ข้อมูลเป็นสถิติสรุประดับพื้นที่จากข้อมูลเปิดภาครัฐ · ใช้เพื่อการศึกษา/วิเคราะห์เท่านั้น</sub>
</div>
