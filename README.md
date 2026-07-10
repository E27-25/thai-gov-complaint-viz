# 🇹🇭 Thai Government Complaint Data — Visualization & ML

วิเคราะห์และสร้าง Dashboard จาก **ข้อมูลเปิดภาครัฐไทย (Thai Open Government Data)** 2 ชุด
เกี่ยวกับ *เรื่องร้องเรียน/ร้องทุกข์* — ดึงข้อมูลจาก API จริง แล้วทำ **EDA · Interactive Dashboard · แผนที่ · Unsupervised Clustering · Predictive Model**

<p align="center">
  <img src="assets/pps_map.png" width="49%" alt="แผนที่เบาะแสยาเสพติดรายจังหวัด"/>
  <img src="assets/ocpb_map.png" width="49%" alt="แผนที่เรื่องร้องทุกข์ต่อประชากร กทม."/>
</p>

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-Interactive-3F4F75?logo=plotly&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-KMeans%20%7C%20Ridge-F7931E?logo=scikitlearn&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-SARIMA-1f77b4)

---

## 📦 Datasets

| Folder | หน่วยงาน | ข้อมูล | ขนาด | โมเดล |
|--------|----------|--------|------|-------|
| [`ปปส/`](ปปส/) | สำนักงาน ป.ป.ส. (ONCB) | เบาะแส/ร้องเรียนยาเสพติดรายเดือน รายจังหวัด (2560–2567) | 7,286 แถว · 144,117 เบาะแส | **SARIMA** time-series forecast |
| [`สคบ/`](สคบ/) | สำนักงาน สคบ. (OCPB) | เรื่องร้องทุกข์ผู้บริโภค กทม. รายเขต (2569) | 50 เขต · 8,461 เรื่อง | **Ridge** regression |

แต่ละโฟลเดอร์มี: notebook (`*_viz.ipynb`) · dashboard แบบ interactive (`*_viz.html`) · ข้อมูลดิบ (`.csv`) · ขอบเขตแผนที่ (`.geojson`) · README + API docs

---

## 🔦 ปปส — เบาะแสยาเสพติด (ONCB)

> ดึงจาก ONCB API สาธารณะ · รายเดือน 96 เดือน · แยก **5 กลุ่มเป้าหมาย × 3 ผลตรวจสอบ** ต่อจังหวัดต่อเดือน

<p align="center">
  <img src="assets/pps_forecast.png" width="88%" alt="SARIMA forecast — actual vs predicted"/>
</p>

**Predictive model — SARIMA(1,1,1)(1,1,1)₁₂:** พยากรณ์เบาะแสรายเดือน กันข้อมูล 12 เดือนสุดท้ายไว้ทดสอบ →
**MAE 244 · RMSE 295 · MAPE 13.4%** แล้วพยากรณ์ล่วงหน้าถึงปี 2568 พร้อมช่วงความเชื่อมั่น 95%

<p align="center">
  <img src="assets/pps_clusters.png" width="49%" alt="K-Means clustering ของจังหวัด"/>
  <img src="assets/pps_timeseries.png" width="49%" alt="Time series เบาะแสรายเดือน"/>
</p>

**K-Means (k=5, 12 features):** จัดกลุ่มจังหวัดตามโปรไฟล์การตรวจสอบเบาะแส — แยกได้ตั้งแต่กลุ่มปริมาณสูงแต่พบพฤติการณ์ต่ำ (18%) ไปจนถึงกลุ่มพบพฤติการณ์สูง (50%) · [รายละเอียด →](ปปส/)

---

## 🛒 สคบ — เรื่องร้องทุกข์ผู้บริโภค กทม. (OCPB)

> ดึงจาก public Tableau dashboard ของ สคบ. · 50 เขต · จัดกลุ่มเป็น 6 กลุ่มเขตตามการแบ่งของ กทม.

<p align="center">
  <img src="assets/ocpb_regression.png" width="60%" alt="Ridge regression predicted vs actual"/>
</p>

**Predictive model — Ridge Regression (5-fold CV):** ทำนายจำนวนเรื่องร้องทุกข์รายเขตจาก **ประชากร + สัดส่วนประเภท + กลุ่มเขต** →
**R² 0.45 · MAE 42** (ประชากรเป็น feature หลัก, corr ≈ 0.68)

<p align="center">
  <img src="assets/ocpb_treemap.png" width="49%" alt="Treemap กลุ่มเขต → เขต"/>
  <img src="assets/ocpb_clusters.png" width="49%" alt="K-Means clustering ของเขต"/>
</p>

**K-Means (k=4, 7 features):** จัดกลุ่ม 50 เขตตามโปรไฟล์ประเภทเรื่องร้องทุกข์ + ปริมาณ · [รายละเอียด →](สคบ/)

---

## 🗺️ Pipeline (เหมือนกันทั้ง 2 ชุด)

```
API / Dashboard  →  ดึงข้อมูล (CSV)  →  EDA + Treemap + แผนที่ Choropleth
                                     →  K-Means (StandardScaler → KMeans → PCA 2D)
                                     →  Predictive model (SARIMA / Ridge)
                                     →  Export dashboard (self-contained HTML)
```

## ▶️ วิธีใช้งาน

```bash
pip install pandas plotly scikit-learn statsmodels
# เปิด notebook ในแต่ละโฟลเดอร์แล้ว Run all — จะได้ dashboard HTML
jupyter notebook ปปส/pps_viz.ipynb
jupyter notebook สคบ/ocpb_viz.ipynb
```

รันบน **Google Colab** ได้เลย (เซลล์แรก `!pip install ...` ให้อยู่แล้ว) — notebook จะโหลด geojson จากไฟล์ในเครื่องหรือดึงจาก URL อัตโนมัติ
หรือเปิด `*_viz.html` ในเบราว์เซอร์เพื่อดู dashboard แบบ interactive ได้ทันที (ลองกดปุ่ม 🚨 / 🛒 มุมขวาล่างดู 👀)

## 🛠️ Tech stack

**pandas** · **Plotly** (treemap, choropleth, scatter, time series) · **scikit-learn** (K-Means, PCA, Ridge, StandardScaler) · **statsmodels** (SARIMA)

## 📚 Data sources

- **ปปส:** [gdcatalog.go.th/dataset/gdpublish-complain-04](https://gdcatalog.go.th/dataset/gdpublish-complain-04) · API: [data.oncb.go.th](https://data.oncb.go.th/complain_021)
- **สคบ:** [gdcatalog.go.th/dataset/gdpublish-opendata-03](https://gdcatalog.go.th/dataset/gdpublish-opendata-03) · [OCPB Connect dashboard](https://ocpbconnect.ocpb.go.th/Report/Detail?report_id=7EF99779-95B5-4541-A374-378B1CD11140)
- **ขอบเขตแผนที่:** provinces — [chingchai/OpenGISData-Thailand](https://github.com/chingchai/OpenGISData-Thailand) · Bangkok districts — [pcrete/gsvloader-demo](https://github.com/pcrete/gsvloader-demo)

ดู API documentation แบบละเอียดของแต่ละชุดได้ในไฟล์ README ของแต่ละโฟลเดอร์

> ⚠️ ข้อมูลเป็นสถิติสรุประดับพื้นที่จากข้อมูลเปิดภาครัฐ ใช้เพื่อการศึกษา/วิเคราะห์เท่านั้น
