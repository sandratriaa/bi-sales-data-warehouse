# 📊 Proyek KBA — Kelompok 3
## Sistem Kecerdasan Bisnis & Analitik — Data Warehouse ERP Sales

---

## 👥 Anggota Tim
| Nama | NIM |
|------|-----|
| Rayhan Wira Pamungkas | 245150407111083 |
| Sandra Triana Nursyafri | 245150401111027 |
| Shal Aisya Deeba Callysta | 245150407111061 |
| Zaky Ahmady Santoso | 245150407111048 |

---

## 📁 Struktur Folder Proyek

```
bi-sales-data-warehouse/
│
├── kba_warehouse.db       ← Database SQLite (star schema, sudah bersih)
├── filter_data.py         ← Script ETL awal (filter + sampling dari data.db)
├── etl_fix.py             ← Script fix revenue + validasi kualitas data
├── kpi_calculator.py      ← Hitung 5 KPI + export CSV + visualisasi
├── datamining.py          ← RFM segmentation, forecasting, basket analysis
├── customers.csv          ← Data master pelanggan
├── items.csv              ← Data master produk
├── README.md              ← Panduan ini
│
├── kpi_output/            ← (auto-dibuat) hasil CSV dan chart KPI
│   ├── KPI-01.csv
│   ├── KPI-02.csv
│   ├── KPI-03.csv
│   ├── KPI-04.csv
│   ├── KPI-05.csv
│   └── dashboard_kpi.png
│
└── dashboard/             ← File Power BI (.pbix)
```

---

## 🗄️ Struktur Database (Star Schema)

### fact_sales (1.665.059 baris)
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| invoice_id | TEXT | ID unik transaksi |
| customer_id | INTEGER | FK → dim_customers |
| product_id | INTEGER | FK → dim_items |
| invoice_date | DATE | Tanggal transaksi |
| quantity | INTEGER | Jumlah unit |
| revenue | REAL | Pendapatan (qty × unit_price) |
| store_id | INTEGER | ID toko |

### dim_customers (500.000 baris)
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| customer_id | INTEGER | PK |
| first_name | TEXT | Nama depan |
| last_name | TEXT | Nama belakang |
| full_name | TEXT | Nama lengkap |
| email | TEXT | Email |
| phone | TEXT | Nomor HP |
| email_opt_in | INTEGER | 0/1 |

### dim_items (98 baris)
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| product_id | INTEGER | PK |
| product_name | TEXT | Nama produk |
| brand | TEXT | Merek |
| category | TEXT | Kategori |
| unit_price | REAL | Harga satuan |

### Index yang tersedia
- `idx_customer` → fact_sales(customer_id)
- `idx_product` → fact_sales(product_id)
- `idx_date` → fact_sales(invoice_date)
- `idx_store` → fact_sales(store_id)

---

## ⚙️ Setup Environment

### 1. Pastikan Python 3.10+ terinstall
```bash
python --version
```

### 2. (Opsional) Buat virtual environment
```bash
python -m venv venv
venv\Scripts\activate   # Windows
```

### 3. Install semua library
```bash
pip install pandas scikit-learn prophet mlxtend plotly matplotlib seaborn yellowbrick reportlab sqlalchemy
```

---

## 🚀 Urutan Menjalankan Script

### Step 1 — Fix & Validasi Database
> Jalankan ini PERTAMA KALI setelah dapat kba_warehouse.db

```bash
python etl_fix.py
```

**Yang dilakukan:**
- Memperbaiki revenue = 0 → dihitung dari quantity × unit_price
- Mengisi missing first_name/last_name dengan 'Unknown'
- Menambah kolom full_name
- Memastikan semua index tersedia
- Menghapus true duplicates
- Menampilkan laporan validasi kualitas data

---

### Step 2 — Hitung KPI
```bash
python kpi_calculator.py
```

**Output:**
- `kpi_output/KPI-01.csv` → Revenue bulanan
- `kpi_output/KPI-02.csv` → Revenue per kategori
- `kpi_output/KPI-03.csv` → Top 5 brand
- `kpi_output/KPI-04.csv` → Avg transaction value
- `kpi_output/KPI-05.csv` → Sales volume per produk
- `kpi_output/dashboard_kpi.png` → Visualisasi semua KPI

---

### Step 3 — Data Mining
```bash
python datamining.py
```

**Output:**
- `output_rfm_segmentation.png` → Pie chart segmen pelanggan
- `output_forecasting.png` → Prediksi tren 6 bulan ke depan
- `output_basket_analysis.png` → Top association rules
- Tabel baru di database: `rfm_segments`, `forecast_results`, `basket_rules`

---

### Step 4 — Dashboard Power BI
1. Buka Power BI Desktop
2. **Get Data** → **SQLite** (install driver dulu jika belum ada)
   - Alternatif: **Get Data** → **Python script** → load via pandas
3. Import tabel: `fact_sales`, `dim_customers`, `dim_items`
4. Buat relationship:
   - `fact_sales[customer_id]` → `dim_customers[customer_id]`
   - `fact_sales[product_id]` → `dim_items[product_id]`
5. Buat 5 measure DAX (lihat di bawah)
6. Tambahkan visualisasi dan slicer

### Measure DAX untuk Power BI
```dax
Total Revenue = SUM(fact_sales[revenue])

Avg Transaction Value = DIVIDE(SUM(fact_sales[revenue]), COUNT(fact_sales[invoice_id]))

Total Transaksi = COUNT(fact_sales[invoice_id])

Total Quantity = SUM(fact_sales[quantity])

Revenue YoY Growth = 
DIVIDE(
    [Total Revenue] - CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(fact_sales[invoice_date])),
    CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(fact_sales[invoice_date]))
)
```

---

## 📊 KPI yang Dilacak

| Kode | Nama KPI | Target |
|------|----------|--------|
| KPI-01 | Total Revenue Bulanan | Tersedia di dashboard |
| KPI-02 | Revenue per Kategori | Tersedia di dashboard |
| KPI-03 | Top 5 Brand by Revenue | Tersedia di dashboard |
| KPI-04 | Average Transaction Value | Tersedia di dashboard |
| KPI-05 | Sales Volume per Produk | Tersedia di dashboard |

---

## ✅ Kriteria Keberhasilan

| Kriteria | Target | Status |
|----------|--------|--------|
| ETL selesai | < 2 jam | ✓ |
| Semua KPI tersedia | 5 KPI | ✓ |
| Missing values | < 1% | ✓ 0% |
| Duplikat | < 0.1% | ✓ 0% |
| Segmentasi pelanggan | Cluster interpretable | ✓ 4 segmen |

---

## 🔗 Tautan Penting
- **Dataset Kaggle:** https://www.kaggle.com/datasets/ualex1/synthetic-erp-sales-dataset
- **Google Doc PRD:** [isi link PRD kalian]
- **Google Doc Arsitektur C4:** [isi link arsitektur kalian]
- **Dosen Pengampu:** Ir. Nanang Yudi Setiawan, S.T, M.KOM.
