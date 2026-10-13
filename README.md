# Vehicle Sales Pricing Analysis

> Analisis selisih harga jual terhadap nilai pasar (MMR) pada data penjualan kendaraan bekas untuk mendukung keputusan pricing dan inventory.

![Python](https://img.shields.io/badge/Python-3.9+-blue)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## Daftar Isi

1. [Business Problem](#-business-problem)
2. [Background Problem](#-background-problem)
3. [Business Context](#-business-context)
4. [Dataset](#-dataset)
5. [Metodologi](#-metodologi)
6. [Insight](#-insight)
7. [Strategic Recommendation](#-strategic-recommendation)
8. [Limitasi](#-limitasi)
9. [Struktur Repository](#-struktur-repository)
10. [Cara Menjalankan](#-cara-menjalankan)

---

## 🎯 Business Problem

**Industri:** Otomotif (jual-beli kendaraan bekas)
**Stakeholder:** Pricing Manager, Inventory Manager, Pemilik Dealer

**Problem statement:**

> Faktor apa (merek, umur kendaraan, kondisi, odometer, negara bagian, dan waktu penjualan) yang paling memengaruhi selisih antara harga jual dan nilai pasar (MMR), dan segmen kendaraan mana yang paling sering terjual di bawah nilai pasarnya sehingga margin dealer tergerus?

---

## Background Problem

1. **Fakta industri:** Pasar kendaraan bekas besar dan harganya berfluktuasi mengikuti kondisi ekonomi, suku bunga, dan pasokan unit. *(Tambahkan 1 sumber berita/laporan terbaru di sini.)*
2. **Masalah umum:** Penentuan harga sering bergantung pada intuisi, sehingga unit terjual terlalu murah atau terlalu lama tertahan di stok.
3. **Kaitan dengan dataset:** Data transaksi memuat harga jual dan nilai pasar (MMR), sehingga pola penyimpangan harga dapat dianalisis.
4. **Risiko jika dibiarkan:** Margin tergerus, modal tertahan di stok lama, dan keputusan pembelian stok kurang tepat.

---

## 💼 Business Context

- **Tujuan bisnis:** Menaikkan margin dan mempercepat perputaran stok.
- **Kontribusi project:** Menunjukkan segmen yang cenderung dijual di bawah pasar, serta waktu dan wilayah dengan harga jual terbaik.
- **Pengambil keputusan:** Pricing Manager dan Inventory Manager.

| Tujuan Bisnis | Dampak Analisis |
|---|---|
| Revenue | Naik jika harga mendekati atau di atas nilai pasar |
| Cost Efficiency | Turun jika stok tidak menumpuk dan salah beli berkurang |
| Customer Growth | Naik jika harga kompetitif untuk segmen yang dicari pembeli |

---

## Dataset

- **Sumber:** [Kaggle – Vehicle Sales Data (`syedanwarafridi/vehicle-sales-data`)](https://www.kaggle.com/datasets/syedanwarafridi/vehicle-sales-data)
- **File:** `car_prices.csv`
- **Ukuran:** ± 558 ribu baris, 16 kolom

| Kolom | Deskripsi |
|---|---|
| `year` | Tahun produksi kendaraan |
| `make`, `model`, `trim` | Merek, model, dan varian |
| `body` | Tipe bodi (sedan, SUV, dll.) |
| `transmission` | Jenis transmisi |
| `vin` | Nomor identifikasi kendaraan |
| `state` | Negara bagian lokasi penjualan |
| `condition` | Skor kondisi kendaraan |
| `odometer` | Jarak tempuh |
| `color`, `interior` | Warna eksterior dan interior |
| `seller` | Penjual |
| `mmr` | Manheim Market Report, estimasi nilai pasar |
| `sellingprice` | Harga jual |
| `saledate` | Tanggal penjualan |

---

## Metodologi

1. **Data cleaning**
   - Cek missing value, duplikat, dan outlier (harga/odometer tidak wajar)
   - Ubah `saledate` ke format tanggal
2. **Feature engineering**
   - `price_gap = sellingprice - mmr`
   - `gap_pct = (sellingprice - mmr) / mmr`
   - `vehicle_age`, `sale_month`, `sale_year`
3. **Exploratory Data Analysis**
   - Distribusi harga, odometer, dan kondisi
   - Volume penjualan per merek, body, dan state
4. **Analisis segmen**
   - `gap_pct` per merek/model, body, state, seller, dan bulan
   - Korelasi antara kondisi, odometer, dan selisih harga

```python
import pandas as pd

df = pd.read_csv("data/car_prices.csv")

# Cleaning dasar
df = df.drop_duplicates()
df["saledate"] = pd.to_datetime(df["saledate"], errors="coerce", utc=True)
df = df.dropna(subset=["mmr", "sellingprice", "saledate"])
df = df[df["mmr"] > 0]

# Feature engineering
df["price_gap"] = df["sellingprice"] - df["mmr"]
df["gap_pct"] = df["price_gap"] / df["mmr"]
df["sale_year"] = df["saledate"].dt.year
df["sale_month"] = df["saledate"].dt.month
df["vehicle_age"] = df["sale_year"] - df["year"]

# Contoh: rata-rata gap per merek (minimal 500 transaksi)
by_make = (
    df.groupby("make")
      .agg(n=("vin", "count"), avg_gap_pct=("gap_pct", "mean"))
      .query("n >= 500")
      .sort_values("avg_gap_pct")
)
print(by_make.head(10))
```

---

## Insight

> Bagian ini diisi setelah analisis dijalankan. Gunakan angka asli dari hasil Anda.

**Template insight:**

- **Insight 1:** Kendaraan `[body/merek]` dengan odometer di atas `[X]` mil menyumbang `[Y]%` unit yang terjual di bawah MMR.
- **Insight 2:** Harga jual rata-rata tertinggi terjadi pada bulan `[Z]`, dengan `gap_pct` sebesar `[N]%`.
- **Insight 3:** State `[A]` konsisten menghasilkan `gap_pct` lebih tinggi dibanding `[B]`.

**Visualisasi (tambahkan gambar di folder `images/`):**

| Visual | Tujuan |
|---|---|
| Histogram `gap_pct` | Melihat sebaran selisih harga terhadap MMR |
| Bar chart `gap_pct` per merek | Mengidentifikasi merek yang sering terjual di bawah pasar |
| Line chart `gap_pct` per bulan | Mengidentifikasi pola musiman |
| Scatter `odometer` vs `sellingprice` | Melihat pengaruh jarak tempuh terhadap harga |

---

## Strategic Recommendation

> Sesuaikan dengan temuan analisis Anda.

| No | What | Why | How |
|---|---|---|---|
| 1 | Terapkan batas harga minimum berbasis MMR untuk segmen yang sering rugi | Mencegah penjualan di bawah nilai pasar | Hitung batas bawah per segmen dari data historis |
| 2 | Jadwalkan penjualan pada bulan dengan harga terbaik | Harga jual lebih tinggi pada periode tertentu | Prioritaskan unit bernilai tinggi ke bulan dengan `gap_pct` terbaik |
| 3 | Prioritaskan pembelian stok dengan margin dan perputaran baik | Modal tidak tertahan di unit sulit laku | Ranking segmen berdasarkan `gap_pct` dan volume |
| 4 | Evaluasi seller/kanal dengan kinerja harga rendah | Mengurangi kebocoran margin | Review berkala berdasarkan `gap_pct` |

**Prioritas:** urutkan berdasarkan potensi dampak, kemudahan implementasi, dan kecepatan hasil.

---

## Limitasi

- Data berbasis pasar Amerika Serikat (kolom `state`), sehingga insight belum tentu berlaku untuk pasar Indonesia.
- `mmr` adalah estimasi pihak ketiga, bukan harga pasti. Selisih positif atau negatif belum tentu berarti untung atau rugi.
- Tidak ada data biaya akuisisi unit, jadi margin sebenarnya tidak dapat dihitung, hanya selisih terhadap MMR.
- Tidak ada data pelanggan, jadi analisis segmen pembeli tidak dapat dilakukan.

---

## Struktur Repository

```
vehicle-sales-pricing-analysis/
├── data/
│   └── car_prices.csv        # tidak di-commit jika terlalu besar (lihat .gitignore)
├── notebooks/
│   └── analysis.ipynb
├── images/                   # visualisasi untuk README
├── requirements.txt
└── README.md
```

---

## Cara Menjalankan

```bash
# 1. Clone repository
git clone https://github.com/<username>/vehicle-sales-pricing-analysis.git
cd vehicle-sales-pricing-analysis

# 2. Install dependency
pip install -r requirements.txt

# 3. Unduh dataset dari Kaggle ke folder data/
kaggle datasets download -d syedanwarafridi/vehicle-sales-data -p data/ --unzip

# 4. Jalankan notebook
jupyter notebook notebooks/analysis.ipynb
```

**`requirements.txt` (contoh):**

```
pandas
numpy
matplotlib
seaborn
jupyter
kaggle
```
