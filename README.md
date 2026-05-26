# Datathon 2026 — Round 1

Phân tích dữ liệu e-commerce thời trang Việt Nam và dự báo doanh thu.

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
│   └── raw/                             # Dữ liệu thô — KHÔNG chỉnh sửa
│
├── notebooks/
│   ├── eda.ipynb                        # Exploratory Data Analysis
│   ├── Data_Storytelling.ipynb          # Phân tích Customer Lifecycle
│   └── Revenue_Forecast.ipynb           # Mô hình dự báo doanh thu
│
├── models/
│   ├── xgb_revenue.pkl                  # XGBoost (trained on 2012–2022)
│   └── sarima_revenue.pkl               # SARIMA(1,1,1)(1,1,1,7)
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

### 3. `Revenue_Forecast.ipynb` — Dự Báo Doanh Thu
Dự báo doanh thu hàng ngày giai đoạn **01/01/2023 – 01/07/2024** (548 ngày).

| Mô hình | Mô tả |
|---------|-------|
| Baseline | Naive Seasonal — cùng ngày năm trước |
| SARIMA(1,1,1)(1,1,1,7) | Thống kê — bắt xu hướng + mùa vụ tuần |
| XGBoost | ML — 40+ features: lag, rolling, Fourier, calendar |
| **Ensemble** | Trung bình có trọng số (inverse-RMSE) của SARIMA + XGBoost |

**Train/Validation split:** Train 2012–2021 · Validation 2022 · Forecast 2023–2024

---

## Kết Quả Dự Báo

File: `outputs/predictions/revenue_forecast_2023_2024.csv`

| Cột | Mô tả |
|-----|-------|
| `Date` | Ngày (2023-01-01 → 2024-07-01) |
| `Revenue_XGBoost` | Dự báo từ mô hình XGBoost |
| `Revenue_SARIMA` | Dự báo từ mô hình SARIMA |
| `Revenue_Ensemble` | Dự báo ensemble (kết quả chính) |

---

## Môi Trường

**Python:** 3.11  
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
