---
name: data-analysis
description: Analisis data dengan Python (pandas, matplotlib, seaborn). Load → clean → explore → visualize → insight. Semua angka DARI data, gak ngarang.
version: 2.0.0
metadata:
  hermes:
    tags: [data, analysis, pandas, python, visualization, chart, statistics]
    category: development
    requires_toolsets: [terminal, core]
---

# Data Analysis

## KAPAN PAKAI

```
IF user kasih CSV/Excel/JSON + minta analisis → PAKAI
IF user minta chart/visualisasi dari data → PAKAI
IF user minta "cari tren/pattern" dari data → PAKAI
IF user minta bersihin data → PAKAI
IF user minta hitung statistik dari data → PAKAI
IF user gak punya data (minta opini/coding) → JANGAN PAKAI
```

---

## PROCEDURE (ikuti exact, JANGAN skip)

### Step 1: Load data

```python
import pandas as pd

# CSV
df = pd.read_csv("path/to/file.csv")

# Excel
df = pd.read_excel("path/to/file.xlsx")

# JSON
df = pd.read_json("path/to/file.json")

# Quick look
print(f"Shape: {df.shape}")
print(f"Columns: {df.columns.tolist()}")
print(df.head())
print(df.dtypes)
```

### Step 2: Clean data

```python
# Cek missing
print(df.isnull().sum())

# Cek duplicates
print(f"Duplicates: {df.duplicated().sum()}")

# Decision tree cleaning:
# IF numeric column has NaN → df['col'].fillna(df['col'].median())
# IF categorical column has NaN → df['col'].fillna('Unknown')
# IF duplicate rows → df.drop_duplicates(inplace=True)
# IF date column as string → df['date'] = pd.to_datetime(df['date'])
# IF price as string "Rp 1.250.000" → parse to int
```

### Step 3: Explore

```python
# Statistik dasar
print(df.describe())

# Distribusi per kategori
print(df['category'].value_counts())

# Korelasi (numeric only)
print(df.select_dtypes(include='number').corr())
```

### Step 4: Visualize

```
PILIH chart berdasarkan tujuan:

IF mau nunjukin perbandingan → Bar chart
IF mau nunjukin tren waktu → Line chart
IF mau nunjukin distribusi → Histogram / Boxplot
IF mau nunjukin korelasi 2 variable → Scatter plot
IF mau nunjukin proporsi (max 6 segment) → Pie chart
IF mau nunjukin multi-variable → Heatmap

RULE: Setiap chart HARUS punya title, xlabel, ylabel
RULE: Simpan ke file (plt.savefig), jangan cuma plt.show()
```

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Contoh: line chart
plt.figure(figsize=(10, 6))
plt.plot(df['date'], df['revenue'], marker='o')
plt.title('Revenue Trend 2024-2026')
plt.xlabel('Date')
plt.ylabel('Revenue (Rp)')
plt.grid(True)
plt.tight_layout()
plt.savefig('/tmp/revenue_trend.png')
print("Chart saved: /tmp/revenue_trend.png")
```

### Step 5: Insight (dari data, BUKAN ngarang)

```
RULE: Setiap insight HARUS ada angka pendukung dari pandas output
RULE: Correlation ≠ causation (selalu caveat)
RULE: Small sample = low confidence (flag ini)
DO NOT: invent numbers yang gak ada di data
DO NOT: over-interpret (3 data points bukan "tren signifikan")
```

---

## OUTPUT TEMPLATE

```markdown
## 📊 Data Analysis: [Nama Dataset]

**Source**: [path/file]
**Rows**: [N] | **Columns**: [N]
**Period**: [date range kalau ada]

### Data Quality
- Missing values: [N] di kolom [X, Y]
- Duplicates: [N] (removed/kept)
- Clean rows used: [N]

### Key Findings

1. **[Finding 1]**: [angka spesifik dari data]
   - Metric: [value]
   - Insight: [1 kalimat interpretasi]

2. **[Finding 2]**: [angka spesifik]
   - Metric: [value]
   - Insight: [1 kalimat]

3. **[Finding 3]**: [angka spesifik]
   - Metric: [value]
   - Insight: [1 kalimat]

### Visualisasi
📁 Chart saved: [path]
[deskripsi singkat chart]

### Caveat
- [Limitasi data: sample size, missing values, time range]
- [Correlation ≠ causation]
- [Angka bisa berubah dengan data baru]
```

---

## CONTOH OUTPUT

```markdown
## 📊 Data Analysis: Sales Q1 2026

**Source**: ~/data/sales_q1_2026.csv
**Rows**: 1,247 | **Columns**: 8
**Period**: Jan 2026 - Mar 2026

### Data Quality
- Missing values: 23 di kolom `region` (filled: "Unknown")
- Duplicates: 5 (removed)
- Clean rows used: 1,242

### Key Findings

1. **Revenue total Q1**: Rp 4.7 Miliar
   - Rata-rata per transaksi: Rp 3.8 juta
   - Insight: Naik 12% dari Q1 2025 (Rp 4.2M)

2. **Top product category**: Electronics (38% revenue)
   - 2nd: Fashion (24%)
   - 3rd: F&B (18%)
   - Insight: Electronics dominasi karena promo Harbolnas carry-over

3. **Region performance**: Jakarta contribute 45% total revenue
   - Surabaya: 18%
   - Bandung: 12%
   - Insight: Konsentrasi tinggi di Jawa — ekspansi luar Jawa potential

### Visualisasi
📁 Chart saved: /tmp/sales_q1_breakdown.png
Bar chart: Revenue per category + monthly trend line

### Caveat
- Data cuma Q1 (3 bulan) — terlalu pendek untuk prediksi annual
- Region "Unknown" (23 rows) gak diinclude di regional analysis
- Correlation revenue × marketing spend = 0.72 (moderate, bukan causation)
```

---

## DECISION TREE: Chart Selection

```
IF data = [category, value] → Bar chart (horizontal kalau label panjang)
IF data = [date, value] → Line chart
IF data = [numeric, numeric] → Scatter plot
IF data = [category, count] dan ≤6 kategori → Pie chart
IF data = [category, count] dan >6 kategori → Bar chart (top 10)
IF data = correlation matrix → Heatmap (seaborn)
IF data = distribution 1 variable → Histogram + kde
IF data = compare distributions → Boxplot
```

---

## VERIFICATION

```
□ Data di-clean sebelum analysis?
□ SEMUA angka dari pandas output (bukan ngarang)?
□ Chart punya title + label axes?
□ Caveat data quality disebutkan?
□ Code bisa jalan (tested)?
□ Correlation gak di-claim sebagai causation?

IF ada □ TIDAK → fix
```
