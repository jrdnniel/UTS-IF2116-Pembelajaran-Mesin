# Milestone 1 (UTS) IF2116 Pembelajaran Mesin — Proyek B01: Karakteristik Lokasi dan Kejadian Kecelakaan

Repositori ini memuat seluruh kode eksperimen, dataset, laporan akademik, tabel metrik evaluasi, serta artefak visualisasi untuk **Milestone 1 (Ujian Tengah Semester)** mata kuliah **IF2116 Pembelajaran Mesin**.

## 1. Identitas Proyek dan Anggota Kelompok

| Atribut | Keterangan |
| :--- | :--- |
| **Project ID** | `B01` |
| **Nama Kelompok** | Kelompok B01 |
| **Topik Masalah** | Karakteristik Lokasi dan Kejadian Kecelakaan Lalu Lintas |
| **Jenis Masalah Machine Learning** | *Classical Machine Learning — Unsupervised Clustering* dan Reduksi Dimensi *Principal Component Analysis (PCA)* |
| **Cakupan Pengerjaan** | Pertemuan 2 (*Data Understanding & EDA*) hingga Pertemuan 6 (*PCA, Pemilihan K Optimal, & Klasterisasi*) |
| **Tautan Repository GitHub** | [https://github.com/jrdnniel/UTS-IF2116-Pembelajaran-Mesin](https://www.google.com/url?sa=E&source=gmail&q=https://github.com/jrdnniel/UTS-IF2116-Pembelajaran-Mesin) |

### Daftar Anggota Kelompok

| No | Nama Lengkap | NIM |
| :-: | :--- | :--- |
| 1 | **Kevin Rorista Kaban** | `231112119` |
| 2 | **Fadillah Rasyd** | `231112885` |
| 3 | **Jordanniel Sihombing** | `231112048` |

## 2. Deskripsi Masalah dan Tujuan Eksperimen

### Latar Belakang dan Rumusan Masalah
Kecelakaan lalu lintas jalan raya dipengaruhi oleh interaksi multivariat antara kondisi geospasial (perkotaan vs pedesaan), geometri jalan (batas kecepatan, kelas jalan, keberadaan persimpangan), kondisi lingkungan (pencahayaan malam hari, cuaca hujan/kabut, permukaan jalan basah/es), serta siklus waktu kejadian. Evaluasi agregat tunggal sering kali menyamarkan perbedaan mendasar antara insiden komuter berkecepatan rendah di pusat kota dengan kecelakaan fatal berkecepatan tinggi di koridor arteri antarkota maupun insiden tabrakan beruntun multi-korban.

Proyek **B01** ini merumuskan masalah bagaimana mengelompokkan (*clustering*) **101.525 peristiwa kecelakaan lalu lintas** di Inggris Raya sepanjang tahun 2025 ke dalam segmen-segmen karakteristik lokasi dan kejadian yang homogen secara objektif, terukur dengan metrik evaluasi internal, dan **bebas dari kebocoran data (*data leakage*)**.

### Tujuan dan Metodologi Utama
1. **Audit Kualitas Data & Penanganan Kode Sentinel:** Mengidentifikasi nilai hilang eksplisit (`NaN` pada 2 baris koordinat) dan nilai hilang implisit berkode sentinel (`-1`, `9`, dan `99` pada standar pelaporan kepolisian Inggris STATS19), membuang 6 kolom historis yang memiliki `>85%` kode sentinel, serta menghilangkan redundansi koordinat OSGR terhadap WGS84 (`r > 0,999`).
2. **Pencegahan Data Leakage melalui Pipeline Terkunci:** Membagi data bersih (`101.523` baris) menjadi **80% Training Set (`81.218` baris)** dan **20% Testing Set (`20.305` baris)** dengan `seed = 42` sebelum ekstraksi parameter statistik apa pun. Seluruh imputasi median/modus, penskalaan `RobustScaler` berbasis IQR, pengkodean `OneHotEncoder` (35 dimensi), serta reduksi dimensi `PCA` (8 komponen utama = **85,64% varians**) di-fit **eksklusif pada Training Set**.
3. **Rekayasa Fitur Domain (*Feature Engineering*):** Membentuk 4 fitur baru yang relevan dengan domain keselamatan transportasi, yaitu `rasio_korban_per_kendaraan`, `indeks_bahaya_lingkungan` (skor komposit `0–4`), serta pasangan siklikal waktu 24 jam `siklus_jam_sin` dan `siklus_jam_cos`.
4. **Komparasi Baseline, Model Kandidat, dan Validasi 5-Fold Ensemble:** Membandingkan **Baseline K-Means (Tanpa PCA, 35D)** terhadap **K-Means + PCA (8D)**, **Bisecting K-Means + PCA (8D)**, **Ward Hierarchical + PCA (8D)**, dan **Consensus Ensemble Clustering (5 Seeds) + PCA (8D)** menggunakan metrik *Silhouette Score*, *Davies-Bouldin Index (DBI)*, *Calinski-Harabasz Index (CHI)*, dan *Within-Cluster Inertia (SSE)*.

### Ringkasan Hasil Eksperimen Utama (`K = 4`)

| Model Klasterisasi (`K = 4`) | Ruang Fitur | Train Silhouette | Test Silhouette | Train DBI | Test DBI | Train CHI | Test CHI | Waktu Fit |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1. Baseline K-Means** | Raw 35D | `0,1226` | `0,1251` | `2,2544` | `2,2574` | `11.645,5` | `3.514,2` | `0,2284 s` |
| **2. K-Means + PCA (Terbaik)** | PCA 8D (85,64%) | **`0,3434`** | **`0,3522`** | **`1,2882`** | **`1,3041`** | `13.301,3` | `3.997,2` | **`0,0818 s`** |
| **3. Bisecting K-Means + PCA** | PCA 8D (85,64%) | `0,2180` | `0,2223` | `1,3002` | `1,7258` | **`23.467,9`** | **`5.874,4`** | `0,1416 s` |
| **4. Ward Hierarchical + PCA** | PCA 8D (85,64%) | `0,2230` | `0,2199` | `1,6701` | `1,6874` | `15.969,2` | `4.981,7` | `0,3122 s` |
| **5. Consensus Ensemble (5-Fold CV)** | PCA 8D (5 Seeds) | `0,3168 ± 0,0592` | `0,3485` | `1,6859 ± 0,3221` | `1,3190` | `12.840,1` | `3.912,6` | `0,3360 s` |
| *Sanity Check: Klaster Acak* | PCA 8D | `-0,0038` | `-0,0040` | `96,4120` | `98,6453` | `1,0` | `1,0` | `0,0010 s` |

## 3. Sumber Dataset

- **Nama Dataset:** *UK Department for Transport (DfT) Road Safety Data — Collisions 2025 (STATS19)*
- **Nama Berkas Lokal di Repository:** `dft-road-casualty-statistics-collision-2025.csv` (`101.525` baris, `44` kolom, ukuran `19,35 MB`)
- **Hash SHA-256:** `e699c19bdb82e47e449b6f3381ac87631865f6a31cef118c9955ba80ca870e2b`
- **Tautan Portal Resmi:** [https://www.gov.uk/government/statistical-data-sets/road-safety-open-data](https://www.gov.uk/government/statistical-data-sets/road-safety-open-data) dan [https://www.gov.uk/government/collections/road-accidents-and-safety-statistics](https://www.gov.uk/government/collections/road-accidents-and-safety-statistics)
- **Lisensi:** [Open Government Licence v3.0 (OGL v3.0)](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/)

## 4. Struktur Direktori Repository

```text
UTS-IF2116-Pembelajaran-Mesin/
├── README.md                                                     # Dokumentasi utama repository (berkas ini)
├── UTS_B01.ipynb                                                 # Jupyter Notebook eksperimen utama (Komponen 1 s.d. 10)
├── Laporan_UTS_Milestone1_B01_Karakteristik_Kecelakaan.pdf       # Laporan UTS Milestone 1 dalam format PDF siap periksa
├── Laporan_UTS_Milestone1_B01_Karakteristik_Kecelakaan.docx      # Laporan UTS Milestone 1 dalam format Microsoft Word (.docx)
├── dft-road-casualty-statistics-collision-2025.csv               # Dataset mentah resmi DfT STATS19 Collision 2025
└── artefak_milestone1_b01/                                       # Direktori luaran otomatis dari eksekusi notebook
    ├── model_pipeline_milestone1_b01.pkl                         # Objek parameter preprocessor, PCA 8D, dan centroid klaster
    ├── Artefak_Lengkap_Milestone1_B01.zip                        # Paket arsip seluruh tabel CSV dan grafik PNG
    ├── tabel_eksperimen/                                         # 10 tabel CSV hasil audit data, CV, dan evaluasi model
    │   ├── 01_audit_kualitas_44_kolom.csv
    │   ├── 02_statistik_deskriptif_numerik.csv
    │   ├── 03_temuan_eda_dan_implikasi.csv
    │   ├── 04_ringkasan_split_dan_pipeline.csv
    │   ├── 05_optimasi_jumlah_klaster_K2_K8.csv
    │   ├── 06_perbandingan_baseline_vs_kandidat.csv
    │   ├── 07_detail_5fold_cross_validation.csv
    │   ├── 08_ringkasan_stabilitas_ensemble_cv.csv
    │   ├── 09_profil_fisik_4_klaster_optimal.csv
    │   └── 10_bedah_kasus_observasi_uji.csv
    └── visualisasi/                                              # 4 kanvas grafik PNG resolusi tinggi
        ├── eda_1_kualitas_kecepatan_dan_waktu.png
        ├── eda_2_korelasi_spasial_dan_bahaya.png
        ├── eval_3_perbandingan_model_cv_dan_pca.png
        └── profil_4_peta_spasial_dan_heatmap_klaster.png
```

## 5. Petunjuk Penggunaan dan Reproduksi Eksperimen

Notebook `UTS_B01.ipynb` dirancang mandiri (*self-contained*) dengan `seed_acak = 42` sehingga seluruh angka, tabel, dan grafik di dalam laporan dapat direproduksi secara identik.

### Opsi A — Menjalankan secara Lokal (Windows / Linux / macOS)

1. **Kloning repository ke komputer lokal:**
   ```bash
   git clone https://github.com/jrdnniel/UTS-IF2116-Pembelajaran-Mesin.git
   cd UTS-IF2116-Pembelajaran-Mesin
   ```
2. **Pasang pustaka Python yang dibutuhkan (Python 3.10+):**
   ```bash
   pip install numpy pandas scipy matplotlib seaborn jupyter
   ```
3. **Buka dan jalankan seluruh sel pada Jupyter Notebook:**
   ```bash
   jupyter notebook UTS_B01.ipynb
   ```
   Di dalam antarmuka Jupyter atau VS Code, pilih menu **Kernel > Restart & Run All** (atau **Run All**). Fungsi `muat_berkas_kecelakaan` pada notebook akan mendeteksi berkas `dft-road-casualty-statistics-collision-2025.csv` di direktori kerja secara otomatis dan memperbarui seluruh tabel serta grafik di dalam folder `artefak_milestone1_b01/`.

### Opsi B — Menjalankan di Google Colab

1. Buka [Google Colab](https://colab.research.google.com/) dan pilih tab **GitHub**, lalu masukkan URL repository `https://github.com/jrdnniel/UTS-IF2116-Pembelajaran-Mesin` dan buka berkas `UTS_B01.ipynb` (atau unggah berkas `UTS_B01.ipynb` secara langsung).
2. Unggah berkas `dft-road-casualty-statistics-collision-2025.csv` ke panel **Files** (`/content/`) di Google Colab.
3. Pilih menu **Runtime > Run all**. Seluruh tahapan eksperimen akan dieksekusi dari awal hingga menghasilkan arsip `artefak_milestone1_b01/Artefak_Lengkap_Milestone1_B01.zip`.

## 6. Referensi Utama

1. **Anderson, T. K. (2009).** *Kernel density estimation and K-means clustering to profile road accident hotspots.* Accident Analysis & Prevention, 41(3), 359–364. [https://doi.org/10.1016/j.aap.2008.12.014](https://doi.org/10.1016/j.aap.2008.12.014)
2. **Pedregosa, F., et al. (2011).** *Scikit-learn: Machine Learning in Python.* Journal of Machine Learning Research (JMLR), 12(85), 2825–2830. [https://jmlr.org/papers/v12/pedregosa11a.html](https://jmlr.org/papers/v12/pedregosa11a.html)
3. **Virtanen, P., et al. (2020).** *SciPy 1.0: Fundamental algorithms for scientific computing in Python.* Nature Methods, 17(3), 261–272. [https://doi.org/10.1038/s41592-019-0686-2](https://doi.org/10.1038/s41592-019-0686-2)
4. **Arthur, D., & Vassilvitskii, S. (2007).** *k-means++: The advantages of careful seeding.* Proceedings of the 18th ACM-SIAM Symposium on Discrete Algorithms (SODA '07), 1027–1035. [https://research.google/pubs/k-means-the-advantages-of-careful-seeding/](https://research.google/pubs/k-means-the-advantages-of-careful-seeding/)
5. **Fred, A. L. N., & Jain, A. K. (2005).** *Combining multiple clusterings using evidence accumulation.* IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI), 27(6), 835–850. [https://doi.org/10.1109/TPAMI.2005.113](https://doi.org/10.1109/TPAMI.2005.113)
6. **Rousseeuw, P. J. (1987).** *Silhouettes: A graphical aid to the interpretation and validation of cluster analysis.* Journal of Computational and Applied Mathematics, 20, 53–65. [https://doi.org/10.1016/0377-0427(87)90125-7](https://doi.org/10.1016/0377-0427(87)90125-7)
7. **Davies, D. L., & Bouldin, D. W. (1979).** *A cluster separation measure.* IEEE Transactions on Pattern Analysis and Machine Intelligence, PAMI-1(2), 224–227. [https://doi.org/10.1109/TPAMI.1979.4766909](https://doi.org/10.1109/TPAMI.1979.4766909)
8. **Department for Transport, United Kingdom (2025).** *Road Safety Data — Collisions 2025 (STATS19 Specification).* Gov.uk Official Statistics. [https://www.gov.uk/government/statistical-data-sets/road-safety-open-data](https://www.gov.uk/government/statistical-data-sets/road-safety-open-data)
