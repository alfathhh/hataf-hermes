---
name: data-analysis
description: Analisis data dengan Python (pandas, matplotlib, seaborn). Load data, clean, explore, visualize, kasih insight. Output = code yang jalan + chart + summary. Gak ngarang angka — semua dari data.
version: 1.0.0
metadata:
  hermes:
    tags: [data, analysis, pandas, python, visualization, chart, statistics]
    category: development
    requires_toolsets: [terminal, core]
---

# Data Analysis

Analisis data pake Python. Load → clean → explore → visualize → insight. Semua angka DARI data.

## When to Use

- Analisis CSV/Excel/JSON
- Bikin chart/visualisasi
- Cari tren/pattern
- Bersihin data berantakan
- Hitung statistik

## Prerequisites

```bash
pip install pandas matplotlib seaborn openpyxl
```

## Procedure

### 1. Load → 2. Clean → 3. Explore → 4. Visualize → 5. Insight

## Chart type cheatsheet

| Mau nunjukin | Chart |
|---|---|
| Perbandingan | Bar |
| Tren waktu | Line |
| Distribusi | Histogram/boxplot |
| Korelasi | Scatter |
| Proporsi | Pie (max 6 segment) |
| Multi-variable | Heatmap |

## Pitfalls

1. Insight dari data yang belum di-clean
2. Correlation ≠ causation
3. Halu angka (SEMUA dari pandas, gak ngarang)
4. Chart tanpa label
5. Over-interpret small data
6. Pie chart >6 kategori

## Verification

1. Data clean? 2. Angka dari data? 3. Chart punya label? 4. Caveat data quality? 5. Code jalan? 6. Correlation bukan causation?
