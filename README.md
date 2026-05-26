# Datathon 2026 — Round 1

---

## Tổng Quan Dataset

| Thông tin | Giá trị |
|-----------|---------|
| Giai đoạn | 2012-07-04 → 2022-12-31 (10 năm) |
| Tổng doanh thu | ~16.4 tỷ VND |
| Khách hàng | 121,930 |
| Đơn hàng | 646,945 |
| Sản phẩm | 2,412 |
| Danh mục | Streetwear · Casual · Outdoor · GenZ |

---

## Cấu Trúc Project

```
datathon-2026-round-1/
│
├── baseline.ipynb                       # Notebook gốc của ban tổ chức
├── submission.csv                       # File nộp bài cuối cùng
│
├── data/
│   └── raw/                             # Dữ liệu thô
│
├── notebooks/
│   ├── eda.ipynb                        # Exploratory Data Analysis
│   ├── Data_Storytelling.ipynb          # Phân tích Customer Lifecycle
│   └── Revenue_Forecast.ipynb           # Mô hình dự báo doanh thu
│
├── models/
│   ├── xgb_revenue.pkl                  # XGBoost Revenue (trained on 2012–2022)
│   ├── xgb_cogs.pkl                     # XGBoost COGS (trained on 2012–2022)
│   └── sarima_revenue.pkl               # SARIMA(1,1,1)(1,1,1,7) — tham khảo
│
└── outputs/
    ├── charts/
    │   ├── eda/                         # 12 biểu đồ EDA
    │   ├── lifecycle/                   # 8 biểu đồ Customer Lifecycle
    │   └── forecast/                    # 7 biểu đồ Revenue Forecast
    └── predictions/
        └── revenue_forecast_2023_2024.csv   # Dự báo 548 ngày
```

---

## Notebooks

### 1. `eda.ipynb` — Exploratory Data Analysis
Khám phá tổng quan toàn bộ dataset: phân phối, xu hướng, missing values, tương quan giữa các bảng.

### 2. `Data_Storytelling.ipynb` — Customer Lifecycle Analysis
Trả lời 2 câu hỏi phân tích:

**Q1 — Acquisition & First Purchase**
> *"Khách hàng đến từ đâu và hành trình từ đăng ký đến lần mua đầu tiên diễn ra như thế nào?"*
- Số khách hàng mới theo năm & kênh acquisition
- Demographics (giới tính, nhóm tuổi)
- Time-to-Convert: phân bổ ngày signup → mua lần đầu
- CLV theo kênh acquisition

**Q2 — Retention, CLV & Churn**
> *"Sau lần mua đầu, bao nhiêu % khách hàng quay lại? Ai có nguy cơ rời bỏ?"*
- Tỷ lệ repeat purchase & đóng góp doanh thu theo tầng mua hàng
- Cohort Retention Matrix (24 cohorts × 18 tháng)
- RFM Segmentation (Champions / Loyal / At Risk / Lost / ...)
- Churn risk analysis theo kênh & CLV distribution

### 3. `Revenue_Forecast.ipynb` — Dự Báo Doanh Thu & COGS
Dự báo **Revenue và COGS** hàng ngày giai đoạn **01/01/2023 – 01/07/2024** (548 ngày).

| Mô hình | MAPE (Validation 2022) | Vai trò |
|---------|------------------------|---------|
| Baseline (Naive Seasonal) | 27.9% | Benchmark |
| SARIMA(1,1,1)(1,1,1,7) | 119.6% | Tham khảo — kém ở forecast dài hạn |
| **XGBoost** | **19.6%** | **Model chính — Revenue & COGS** |

- **Features:** 47 features — lag (1–365 ngày), rolling stats (7/14/30/90d), Fourier terms (weekly + annual), calendar
- **Strategy:** XGBoost duy nhất; SARIMA loại bỏ khỏi final forecast vì MAPE >100% khi forecast 365 bước liên tiếp
- **Train/Validation split:** Train 2012–2021 · Validation 2022 · Forecast 2023–2024

---

## Kết Quả Dự Báo

**Submission:** `submission.csv` (root) — 548 ngày, format `Date, Revenue, COGS`

**Chi tiết:** `outputs/predictions/revenue_forecast_2023_2024.csv`

| Cột | Mô tả |
|-----|-------|
| `Date` | Ngày (2023-01-01 → 2024-07-01) |
| `Revenue` | Dự báo doanh thu — XGBoost (VND) |
| `COGS` | Dự báo giá vốn — XGBoost (VND) |

**Tóm tắt kết quả:**
- Revenue: ~1.0M – 10.1M VND/ngày (mean ~4.2M)
- COGS: ~0.85M – 9.3M VND/ngày

---

## Môi Trường

**Thư viện chính:**

```
pandas · numpy · matplotlib · seaborn
scikit-learn · xgboost · statsmodels
joblib
```

**Chạy notebook:**
```bash
# Từ thư mục notebooks/
python -m nbconvert --to notebook --execute --inplace <notebook>.ipynb
```

---

## Quan Hệ Giữa Các Bảng Dữ Liệu

```
customers ──┐
            ├──▶ orders ──▶ order_items ──▶ products
geography ──┘       │
                    ├──▶ payments
                    ├──▶ shipments
                    ├──▶ returns
                    └──▶ reviews

sales          (daily aggregated revenue — độc lập)
web_traffic    (daily web sessions — độc lập)
inventory      (monthly stock snapshots)
promotions     (linked to order_items via promo_id)
```
