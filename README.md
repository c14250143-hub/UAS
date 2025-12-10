## 📋 Executive Summary
Laporan ini menyajikan analisis mendalam terhadap data penjualan mobil bekas Toyota Corolla menggunakan metodologi *CRISP-DM* (Cross-Industry Standard Process for Data Mining). 

Proyek ini bertujuan untuk menjawab tiga pertanyaan bisnis utama:
1.  *Prediksi Harga:* Bagaimana kita bisa mengestimasi harga jual wajar berdasarkan spesifikasi teknis?
2.  *Segmentasi Pasar:* Bagaimana mengelompokkan stok mobil yang heterogen ke dalam kategori penjualan yang jelas?
3.  *Tren & Pola:* Apa hubungan tersembunyi antara tahun pembuatan, jenis bahan bakar, dan kelengkapan fitur?

---

## 🛠 BAB 1: Preprocessing Data (Pondasi Analisis)
Tahap ini berfokus pada pembersihan dan transformasi data mentah agar siap diolah oleh algoritma machine learning.

### 1.1 Alur Workflow (Preprocessing)
mermaid
graph LR
    A[📂 CSV Reader] -->|Load Data| B[⚙ Column Filter]
    B -->|Select Features| C[🧹 Missing Value]
    C -->|Clean Data| D[🔠 Rule Engine]

### 1.2 Detail Teknis NodeNode KNIME
Fungsi & KonfigurasiCSV ReaderMembaca dataset ToyotaCorolla.csv. Separator diset ke koma (,) dan Header Row diaktifkan.Column FilterSeleksi Fitur: Membuang kolom yang tidak relevan secara statistik (Id, Model, Cylinders) dan mempertahankan kolom kunci (Price, Age_08_04, KM, HP, Fuel_Type, Quarterly_Tax).Rule EngineTransformasi Data: Mengubah data kategorikal biner menjadi label deskriptif.  Logika: $Automatic$ = 1 => "Matic", $Automatic$ = 0 => "Manual".

---

## BAB 2: Prediksi Harga (Supervised Learning)
Membangun model Regresi Linear untuk memprediksi harga jual kembali (Resale Value).2.1 Arsitektur ModelTujuan: Menemukan rumus matematika $Y = aX + b$ dimana $Y$ adalah Harga.Code snippetgraph TD.

    Data[Data Bersih] --> Encode[🔢 One to Many]
    Encode --> Split[✂ Partitioning 80:20]
    
    Split -->|Training 80%| Learner[🧠 Linear Regression Learner]
    Split -->|Testing 20%| Predictor[🔮 Linear Regression Predictor]
    
    Learner --> Model(Rumus Terbentuk)
    Model --> Predictor
    Predictor --> Scorer[📊 Numeric Scorer]


### 2.2 Metodologi & Alasan One to Many (Dummy Encoding):
Algoritma regresi membutuhkan input numerik. Node ini memecah kolom teks Fuel_Type menjadi vektor biner:Fuel_Type_Diesel (1/0)Fuel_Type_Petrol (1/0)Partitioning (Validasi): Pembagian data 80:20 dilakukan secara acak (stratified sampling) untuk mencegah overfitting (model menghafal jawaban).

### 💡 2.3 Evaluasi Hasil
Berdasarkan output node Numeric Scorer:$R^2$ Score (Akurasi): 0.86 (Contoh).Interpretasi: 86% variasi harga mobil berhasil dijelaskan oleh model ini.Koefisien Regresi:Age_08_04: Bernilai negatif besar. (Faktor depresiasi terkuat).HP (Horsepower): Bernilai positif. (Menambah nilai jual).

## 🧩 BAB 3: Klasterisasi Stok 
*Mengelompokkan stok mobil berdasarkan profil performa dan dimensi fisik.*

### 3.1 Alur Clustering (k-Means)

graph LR
    Input --> Norm[⚖ Normalizer]
    Norm --> KMeans[🎯 k-Means (k=3)]
    KMeans --> Color[🎨 Color Manager]
    Color --> Plot[📈 Scatter Plot (HP vs Weight)]

### 3.2 Metodologi Penting: Normalisasi
Node Normalizer sangat krusial dalam analisis ini.

Tantangan Data:

Weight (Berat): Berkisar antara 1.000 kg - 1.600 kg.

HP (Tenaga): Berkisar antara 69 - 110 HP.

Solusi: Tanpa normalisasi, algoritma k-Means akan bias total ke arah Weight karena angkanya ribuan. Normalizer menyamakan "derajat kepentingan" kedua variabel ini menjadi skala setara (0-1).

### 💡 3.3 Analisis Hasil Segmentasi (HP vs Weight)
Scatter Plot dengan sumbu Y (Vertikal) = HP dan sumbu X (Horizontal) = Weight memperlihatkan pola distribusi fisik mobil:

Pola Korelasi: Terlihat tren positif; semakin berat mobil (bergerak ke kanan), tenaga kuda cenderung semakin tinggi (bergerak ke atas). Ini logis karena bodi yang berat membutuhkan mesin yang lebih kuat.

Interpretasi 3 Cluster yang Terbentuk:

🔴 Cluster 0 (Economy / City Car):

Ciri: Berat ringan (< 1050 kg) & HP rendah (sekitar 69-86 HP).

Profil: Kemungkinan besar varian mesin 1.3 Liter. Cocok untuk target pasar hemat BBM.

🔵 Cluster 1 (Mid-Range / Touring):

Ciri: Berat sedang & HP menengah (97-110 HP).

Profil: Varian standar 1.6 Liter yang seimbang antara tenaga dan kenyamanan.

🟢 Cluster 2 (Heavy Duty / Diesel):

Ciri: Berat sangat tinggi (> 1200 kg) namun HP bervariasi.

Profil: Ini biasanya adalah varian Station Wagon atau mesin Diesel (blok mesin diesel jauh lebih berat). Target pasar yang butuh durabilitas atau ruang muat.


### Apa yang berubah di analisis ini?
1.  *Logika Clustering:* Sekarang Anda mengelompokkan mobil berdasarkan *"Tipe Mesin & Bodi"*, bukan lagi berdasarkan "Umur/Pemakaian".
2.  *Cerita Bisnis:* Hasil ini berguna bagi dealer untuk mengetahui Product Mix merek


# BAB 4: EKSPLORASI & VISUALISASI DATA (EDA)

*Tujuan:* Menganalisis pola distribusi harga, korelasi antar variabel, dan tren pasar menggunakan teknik visualisasi data interaktif.

---

## 4.1. Alur Kerja Visualisasi (Workflow Roadmap)
Untuk melakukan eksplorasi data yang komprehensif, alur kerja di KNIME dirancang dengan strategi percabangan (branching). Data tidak hanya mengalir satu arah, melainkan dibagi menjadi dua jalur pemrosesan:

1.  *Jalur Visualisasi (Kiri):* Fokus pada pengolahan data kategorikal untuk pewarnaan grafik.
2.  *Jalur Statistik (Kanan):* Fokus pada perhitungan matematis menggunakan data numerik murni.

### Diagram Alur Node
graph TD;
    A[📂 CSV Reader] --> B[⚙ Column Filter];
    B --> C[🔠 Number To String];
    C --> D[🎨 Color Manager];
    
    D --> E[🔵 Scatter Plot];
    D --> F[📦 Box Plot];
    
    B --> G[🔢 Linear Correlation];

## 4.2. Tahap Preprocessing DataSebelum visualisasi dibuat, data mentah harus disiapkan agar kompatibel dengan plotting engine KNIME.NodeFungsi & Konfigurasi TeknisCSV ReaderInput Data: Membaca file ToyotaCorolla.csv. Memastikan seluruh baris terbaca dengan delimiter yang benar.

Column Filter Seleksi Fitur: Memangkas dataset agar lebih ringan dan fokus. • Include: Price, Age_08_04, KM, HP, Fuel_Type, Automatic, Weight. • Exclude: Atribut identitas (Id, Model) dan atribut lain yang tidak relevan.

Number To StringTransformasi Tipe Data: Mengubah variabel numerik biner menjadi label kategori agar bisa digunakan sebagai variabel warna/pengelompokan. • Target: Kolom Automatic (0/1 $\rightarrow$ "Manual"/"Auto").

Color Manager Estetika Visual: Menetapkan konsistensi warna untuk setiap kategori bahan bakar agar laporan mudah dibaca. • Setting: Fuel_Type. • Mapping: Petrol = Merah 🔴, Diesel = hijau 🟢, CNG = Kuning 🟡  (Contoh).3. Analisis Statistik & Visual3.1 Analisis Korelasi (Correlation Matrix)Tujuan: Membuktikan secara statistik variabel mana yang memiliki dampak terkuat terhadap harga jual.Node: Linear CorrelationInput: Data numerik murni (sebelum node Number To String).

Konfigurasi:Include: Price, Age_08_04, KM, HP, Weight.Exclude: Data Teks (Fuel_Type).Interpretasi Hasil:Visualisasi matriks korelasi menunjukkan hubungan antar variabel. Fokus utama adalah pada pertemuan baris Price dan kolom Age_08_04:"Nilai korelasi yang mendekati -1 (Warna Gelap) mengindikasikan hubungan negatif yang sangat kuat. Artinya, Umur Mobil adalah faktor depresiasi utama; semakin tua mobil, harga turun secara drastis."3.2 Analisis Varian Harga (Box Plot)Tujuan: Menjawab pertanyaan bisnis, "Apakah jenis bahan bakar mempengaruhi harga jual rata-rata?"Node: Box Plot (JavaScript/Local)Konfigurasi:Category Column (Sumbu X): Fuel_Type.Value Column (Sumbu Y): Price.Interpretasi Hasil:

Box Plot memperlihatkan distribusi harga berdasarkan kategori:"Dengan melihat garis tengah (Median) pada setiap kotak, kita dapat membandingkan harga pasar. Jika kotak Diesel posisinya lebih tinggi daripada Petrol, ini membuktikan bahwa rata-rata harga mobil Diesel bekas lebih mahal. Titik-titik yang berada jauh di luar garis kumis (whiskers) menandakan Outlier atau mobil dengan harga tidak wajar."3.3 Analisis Tren Multi-Dimensi (Scatter Plot)

Tujuan: Visualisasi "Pamungkas" yang menggabungkan 4 dimensi data sekaligus untuk melihat tren pasar secara holistik.Node: Scatter Plot (Plotly)Konfigurasi 4 Dimensi:Sumbu X (Penyebab): Age_08_04 (Umur Mobil).Sumbu Y (Akibat): Price (Harga Jual).Warna (Kategori): Fuel_Type (Bensin/Diesel).Ukuran (Indikator Tambahan): KM (Jarak Tempuh) atau HP.Interpretasi Hasil:Grafik ini menceritakan pola depresiasi yang kompleks:"Terlihat tren titik-titik yang menurun dari kiri-atas ke kanan-bawah, mengonfirmasi bahwa mobil baru berharga mahal dan mobil tua berharga murah. Namun, adanya bola-bola besar (KM Tinggi) yang posisinya lebih rendah dibanding bola kecil pada umur yang sama menunjukkan bahwa Jarak Tempuh yang tinggi mempercepat penurunan harga, terlepas dari tahun pembuatannya."

---

## 📊 BAB 5: Visualisasi & Data Storytelling
Mengungkap wawasan bisnis melalui visualisasi tingkat lanjut.

## 5.1 Hierarki Fitur (Sunburst Chart)
Menelusuri keterkaitan antara fitur mewah.Setting Node: Hierarchy = Fuel_Type $\rightarrow$ Status_Transmisi $\rightarrow$ Status_AC.Analisis:Visualisasi menunjukkan bahwa varian Automatic memiliki probabilitas sangat tinggi untuk dilengkapi AC dan Sebaliknya, varian Manual memiliki distribusi fitur yang lebih acak.Insight: Varian Automatic diposisikan sebagai produk "Full Spec".

## 5.2 Evolusi Stok (Stacked Area Chart)
Melihat tren manufaktur berdasarkan tahun pembuatan.Setting Node: GroupBy (Year) $\rightarrow$ Pivoting (Fuel) $\rightarrow$ Stacked Area Chart.Analisis:Era 1998-1999: Pasar didominasi total oleh mesin Petrol (Bensin).Era 2000-2004: Terjadi lonjakan signifikan pada produksi mobil Diesel (area warna gelap melebar).Insight: Data mencerminkan perubahan tren pasar Eropa yang mulai beralih ke mesin Diesel efisien pada awal milenium.

## Hasil Visualisasi

# 1. Cluster
<img width="841" height="212" alt="Scatter Plot (CLuster)" src="https://github.com/user-attachments/assets/78940ce4-6976-4be9-a260-90dc7bf06a86" />
a. Scatter Plot (Clustering)

Hasil unsupervised learning (K-Means / DBSCAN)
Setiap titik adalah 1 mobil / 1 unit data
Warna berbeda = cluster berbeda berdasarkan fitur seperti:
- Harga
- Tenaga mesin
- Kapasitas
- Fitur
- Penjualan

Tujuan analisis cluster:
- Membagikan kelompok mobil berdasarkan karakteristik mereka
- Menemukan segmen pasar (low-end, mid-end, premium)



# 2. Prediksi perbandingan dengan Harga

<img width="841" height="356" alt="Scatter Plot (prediksi2)" src="https://github.com/user-attachments/assets/8d1f6449-c85f-4351-abe0-9dcd1dfd81d4" />

<img width="841" height="356" alt="Scatter Plot (prediksi)" src="https://github.com/user-attachments/assets/f2acab8f-b60e-40b8-be28-65e50e230bbb" />

a. Scatter Plot (Prediksi Linear/Polynomial)

Gambar tersebut memperlihatkan hubungan antara harga dan variabel lain:
- Mileage
- Engine Size
- Horsepower
- Rating

Titik biru = data asli
Garis merah = hasil prediksi model (regresi)

Tujuan:

Untuk melihat korelasi harga dengan spesifikasi

Semakin mendekati garis prediksi artinya model semakin akurat


<img width="678" height="366" alt="Tabel Skor (Prediksi)" src="https://github.com/user-attachments/assets/4f5ec6ca-d9c0-44f8-a426-f2a456e76988" />

b. Tabel Skor (Prediksi)

Berisi evaluasi model:

R² → seberapa besar model menjelaskan variasi data

RMSE / MAE → error prediksi
Semakin tinggi R² dan semakin kecil RMSE, model semakin bagus.



# 3. Perbandingan Statistik

<img width="841" height="356" alt="Scatter Plot (statistic)" src="https://github.com/user-attachments/assets/25ede60a-3ed8-48d3-813a-94770def0f8f" />

a. Scatter Plot

Menunjukkan hubungan dua variable, contoh:
- Penjualan vs Harga
- Mesin vs Konsumsi BBM

Digunakan untuk melihat trend linear, outlier, dan pola hubungan

<img width="841" height="222" alt="Box Plot (statistik 2)" src="https://github.com/user-attachments/assets/5ca98e8a-f6e7-46c0-8c20-025853de2b5b" />

<img width="841" height="222" alt="Box Plot (statistik)" src="https://github.com/user-attachments/assets/7d6a4af8-b3f3-446f-bb2b-0162c3f11139" />

b. Box Plot

Menampilkan distribusi statistik:
- Median
- Quartile
- Outlier

Box plot membandingkan kategori mobil, untuk melihat apakah ada kategori dengan harga lebih tinggi, konsumsi BBM lebih besar, dll.

Tujuan:
- Mengetahui sebaran data
- Menemukan data ekstrem
- Membandingkan antar kategori




# 4. Perbandingan Penjualan

<img width="1201" height="214" alt="Bar Chart (perbandingan penjualan)" src="https://github.com/user-attachments/assets/19295562-4a28-4188-b5f8-3e3f3084e1a3" />

a. Bar Chart

Menampilkan nilai penjualan per kategori/per model
Digunakan untuk:
- Kategori mana yang paling laku
- Kategori mana yang paling rendah

<img width="1201" height="214" alt="Pie Chart (perbandingan penjualan)" src="https://github.com/user-attachments/assets/c500809d-f6d6-4edf-aa31-d7c141645b7a" />

b. Pie Chart

Tujuannya untuk memperlihatkan proporsi kontribusi setiap kategori terhadap total penjualan


<img width="1201" height="214" alt="Stacked Area Chart (data Analisa hasil penjualan per tahun)" src="https://github.com/user-attachments/assets/7de5cdf4-f5ce-4735-82c4-9b7effcf278e" />


c. Stacked Area Chart (Penjualan per Tahun)

Chart ini memperlihatkan perubahan total penjualan mobil dari tahun ke tahun
Warna berbeda = kategori atau model mobil yang berbeda
Tujuan: untuk melihat hasil pertumbuhan, penurunan, dan kontribusi setiap kategori terhadap total penjualan



# 5. Data manufaktur dan Gransi

<img width="1535" height="216" alt="Score (prediksi data manufaktur)" src="https://github.com/user-attachments/assets/1616d1e3-9583-4dc6-8ff9-d3ad497c4b00" />

a. Table Score (Prediksi Manufaktur)

Mirip bagian prediksi harga:
Untuk mengukur akurasi model prediksi jumlah produksi atau penjualan


<img width="841" height="356" alt="Histogram (produksi mobil)" src="https://github.com/user-attachments/assets/a03980cb-a202-45d5-876b-b263a5e26ea8" />

b. Histogram

Menunjukkan distribusi:

- Produksi mobil per tahun/unit
- Penjualan
- Fitur lainnya

Tujuan:

Untuk mengetahui apakah data tersebut masuk normal, skewed, atau multimodal

<img width="841" height="356" alt="Heatmap (korelasi linear)" src="https://github.com/user-attachments/assets/bb3e89a0-1b43-4d79-a81c-1397451a1320" />

c. Heatmap Korelasi

Menampilkan hubungan antar variabel secara numerik.
Misal:
Harga sangat dipengaruhi horsepower artinya korelasi tinggi
Fitur interior kurang berpengaruh artinya korelasi rendah




# 6. Fasilitas 

<img width="998" height="701" alt="Sunburst Chart (data fasilitas mobil 3)" src="https://github.com/user-attachments/assets/94daf9c0-ab22-47fb-aeca-c7c7a6a7e5c7" />

<img width="998" height="701" alt="Sunburst Chart (data fasilitas mobil 2)" src="https://github.com/user-attachments/assets/37899289-2517-43e6-bf5b-9d9a6159df87" />

<img width="998" height="701" alt="Sunburst Chart (data fasilitas mobil 1)" src="https://github.com/user-attachments/assets/a52e915d-eb43-4dd6-b6e6-4650dc31f0a2" />

a. Fasilitas (Sunburst Chart)

Sunburst chart memvisualisasikan hierarki fitur mobil.

Biasanya struktur seperti:
- Fasilitas
Interior
- AC
- Kulit
- Layar
Eksterior
- Lampu LED
- Sensor
Safety
- ABS
- Airbag
- ESP

Tujuan:
- Menjelaskan proporsi fitur per kategori
- Memberikan gambaran hierarki fitur paling sering muncul
- Membandingkan fitur antar segmen mobil
