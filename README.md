# Analisis dan Prediksi Customer Satisfaction Terhadap Karakteristik Pengiriman dan Ulasan pada E-Commerce

Proyek Data Science · Kelompok Apa (56)

| NIM | NAMA | Kelas Praktikum | Kelas Teori |
| --- | --- | --- | --- |  
| 123240074 | Selgy Pradista Risqi | IF-I | IF-E |
| 123240082 | Muhammad Zidane Zulfikar | IF-C | IF-E |
| 123240099 | Asiil Nasywa Rabbani | IF-G | IF-E |
| 123240229 | Riesky Tanto Firmansyah | IF-F | IF-E |

Proyek ini menganalisis hubungan ketepatan pengiriman dengan skor ulasan pelanggan dan memprediksi risiko keterlambatan pesanan di marketplace Olist. Proses eksplorasi dan percobaan dilakukan di branch master. Branch ini berisi notebook final yang sudah dirapikan mengikuti alur CRISP-DM. Notebook utama mengikuti alur CRISP-DM dari Business Understanding sampai Evaluation. [Buka notebook](olist_crisp_dm.ipynb)

## Business Understanding

Pengelola marketplace, tim operasional, dan mitra logistik dapat memakai hasil analisis untuk memahami pengalaman pelanggan dan memprioritaskan peninjauan pesanan berisiko. Proyek menjawab dua pertanyaan.

1. Bagaimana perbedaan skor ulasan antara pesanan yang tiba tepat waktu dan pesanan yang terlambat? Skor 1–5 digunakan sebagai indikator kepuasan pelanggan.
2. Setelah pesanan disetujui dan tenggat penjual tersedia, seberapa baik Random Forest dan XGBoost dapat mendeteksi pesanan yang akan terlambat?

Satu observasi mewakili satu pesanan. Pesanan diberi label terlambat (`is_late = 1`) jika tanggal tiba melewati tanggal estimasi. Pesanan yang tiba pada atau sebelum tanggal estimasi diberi label tepat waktu (`is_late = 0`).

Kriteria keberhasilan model adalah **F1 kelas terlambat pada data uji yang lebih tinggi daripada baseline kelas mayoritas**. Precision, recall, accuracy, dan confusion matrix melengkapi evaluasi agar kesalahan pada kelas minoritas terlihat.

## Data Understanding

Dataset publik Olist mencakup 99.441 pesanan dari Brasil pada periode 2016–2018. Notebook memeriksa kualitas data dan memakai tiga tabel CSV berikut. [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

| File                              | Peran                                                                                         |
| --------------------------------- | --------------------------------------------------------------------------------------------- |
| `olist_orders_dataset.csv`        | Status pesanan, waktu pembelian dan persetujuan, serta tanggal estimasi dan penerimaan aktual |
| `olist_order_reviews_dataset.csv` | Skor ulasan yang dirata-ratakan per pesanan sebelum dibandingkan antarstatus pengiriman       |
| `olist_order_items_dataset.csv`   | Tenggat pengiriman penjual yang diringkas ke tenggat paling awal per pesanan                  |

Eksplorasi dimulai dari distribusi status pesanan. Dari 96.478 pesanan berstatus `delivered`, 96.470 memiliki tanggal yang diperlukan untuk membandingkan durasi estimasi dan aktual. Sebanyak **6.534 pesanan (6,77%)** tiba terlambat. Notebook kemudian melihat sebaran jumlah hari keterlambatan dan membandingkan skor ulasan.

Di antara pesanan dengan ulasan, rata-rata skornya **4,29** untuk pesanan tepat waktu dan **2,27** untuk pesanan terlambat. Perbandingan ini menggambarkan hubungan yang teramati pada data Olist. Laporan juga mencatat bahwa ulasan didominasi skor 5 dan cakupan geografis data tidak merata. Kedua hal tersebut membatasi generalisasi temuan ke wilayah, periode, atau marketplace lain.

## Data Preparation dan Modeling

Notebook menyiapkan 96.456 pesanan dengan informasi tanggal dan tenggat yang lengkap. Fitur model dihitung berdasarkan informasi yang tersedia sebelum barang tiba.

| Fitur                     | Definisi                                                                               |
| ------------------------- | -------------------------------------------------------------------------------------- |
| `estimated_delivery_days` | Hari penuh dari pembelian hingga estimasi penerimaan                                   |
| `approval_time`           | Jam dari pembelian hingga persetujuan                                                  |
| `seller_deadline_gap`     | Hari penuh dari pembelian hingga tenggat pengiriman paling awal di antara item pesanan |

Tanggal tiba aktual digunakan untuk membentuk label. Skor ulasan digunakan untuk analisis deskriptif. Data dibagi secara acak menjadi 80% data latih dan 20% data uji dengan `random_state=42`. Random Forest dan XGBoost memberi bobot lebih pada kelas terlambat. Pemilihan parameter menggunakan Grid Search dengan validasi silang tiga fold pada data latih.

## Evaluation

| Model                    | F1 terlambat | Precision terlambat | Recall terlambat | Accuracy |
| ------------------------ | -----------: | ------------------: | ---------------: | -------: |
| Baseline kelas mayoritas |        0,000 |                0,0% |             0,0% |    93,3% |
| Random Forest            |        0,155 |                8,7% |            70,3% |    48,7% |
| XGBoost                  |        0,155 |                8,7% |            68,5% |    50,2% |

Angka tersebut berasal dari output data uji yang tersimpan di notebook. Kedua model melampaui F1 baseline, tetapi hanya sekitar 9 dari 100 pesanan yang ditandai terlambat benar-benar terlambat. Hasil ini mengukur kinerja pada pembagian acak data historis Olist.
