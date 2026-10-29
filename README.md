# Comparative Evaluation of Fine-Tuned mBERT and mDeBERTa Cross-Encoders for Reranking on Indonesian MIRACL

Kode, log eksperimen, dan hasil mini research mata kuliah **Advanced Information Retrieval** (Fasilkom UI, 2026).

**Penulis:** Juan Carlos Tepanus Pardosi (NPM 2506712092)

Penelitian ini membandingkan dua *pointwise cross-encoder* multilingual, yaitu **mBERT** (`bert-base-multilingual-cased`) dan **mDeBERTa-v3** (`microsoft/mdeberta-v3-base`). Keduanya di-*fine-tune* untuk me-*rerank* kandidat **BM25** pada subset bahasa Indonesia dari **MIRACL** (Dev resmi, 960 kueri). Data, hyperparameter, dan seed untuk kedua model dibuat sama.

---

## Ringkasan hasil

**Eksperimen 1: perbandingan utama** (Dev, k = 100, rata-rata ± simpangan baku dari 3 seed)

| Sistem | nDCG@10 | MRR@10 | MAP@100 | Recall@100 |
|---|---|---|---|---|
| BM25 (tanpa rerank) | 0,4486 | 0,5306 | 0,3893 | 0,9041 |
| BM25 + mBERT-FT | 0,5947 ± 0,0056 | 0,6956 ± 0,0058 | 0,5159 ± 0,0051 | 0,9041 |
| BM25 + mDeBERTa-FT | 0,6035 ± 0,0060 | 0,7053 ± 0,0064 | 0,5301 ± 0,0061 | 0,9041 |

**Temuan utama**

- **RQ1, peningkatan atas BM25:**
  - mBERT +0,146 nDCG@10, mDeBERTa +0,155 nDCG@10.
  - Keduanya signifikan (Wilcoxon per kueri, koreksi Holm, p < 1e-35).
- **RQ2, mBERT vs mDeBERTa:**
  - Efektivitas: mDeBERTa unggul +0,0088 nDCG@10 (95% CI [−0,0003; +0,0179], p = 0,18). Selisih ini tidak signifikan dan di bawah ambang 0,01.
  - Efisiensi: mDeBERTa membutuhkan latency **2,8×** lebih lama (501 vs 177 ms/kueri pada k = 100), memori GPU 1,8× lebih besar, dan waktu training 2,1× lebih lama.
- **RQ3, top-k ∈ {20, 50, 100}:**
  - nDCG@10 naik dari k = 20 ke k = 50, lalu melandai. Untuk mBERT, k = 50 ≈ k = 100.
  - Latency naik hampir linear terhadap k.
- **RQ4, ukuran data latih (25/50/100%, deskriptif, 1 seed):**
  - Kedua model terus naik.
  - Kurva mDeBERTa lebih landai: +0,032 dari 25% ke 100%, dibanding +0,046 untuk mBERT.

Ringkasan otomatis ada di [`results/summary.md`](results/summary.md), grafik di [`figures/`](figures/).

---

## Struktur repositori

```
├── notebooks/
│   ├── 01_data_bm25.ipynb            # topics & qrels, split 90/10, BM25 top-100, hard negative 1:3, teks passage
│   ├── 02_train_eval_exp1_LOG.ipynb  # fine-tuning Eksp. 1 (100% data, seed 42/100/2026) + pengukuran efisiensi
│   ├── 02_train_eval_exp3_LOG.ipynb  # fine-tuning Eksp. 3 (25% & 50% data, seed 42)
│   ├── 03_topk_efficiency.ipynb      # Eksp. 2: variasi top-k & trade-off akurasi vs latency
│   └── 04_analysis.ipynb             # uji signifikansi, learning curve, truncation, analisis kualitatif
├── data/
│   ├── splits.json                   # qid untuk train (3.664) / val (407) / dev (960) / subset 25% & 50%
│   └── train_pairs.parquet           # 44.814 pasangan latih (qid, docid, label, sumber negatif, flag subset)
├── scores/                           # skor cross-encoder untuk top-100 BM25 Dev, satu file per run (10 run)
├── results/                          # metrik per run (JSON), metrik per kueri (CSV), tabel ringkasan, uji signifikansi
└── figures/                          # grafik Eksp. 2, Eksp. 3, dan analisis overlap leksikal
```

> Notebook `02_train_eval_exp1_LOG` dan `02_train_eval_exp3_LOG` adalah **notebook yang sama**, dijalankan dua kali dengan nilai `EXPERIMENT = 'exp1'` dan `EXPERIMENT = 'exp3'`. Keduanya disimpan beserta output-nya sebagai log run.

Setiap file `results/<model>_frac<persen>_seed<seed>.json` berisi hyperparameter, epoch terbaik, metrik validasi dan Dev, waktu training, versi library, serta riwayat loss per 200 langkah.

---

## Cara menjalankan ulang

Semua notebook dijalankan di **Google Colab**. Data perantara disimpan di Google Drive pada `BASE_DIR = '/content/drive/MyDrive/MIRACL_Reranking'`. Ubah variabel ini di sel konfigurasi bila perlu.

| Urutan | Notebook | Runtime | Waktu training (dari log) |
|---|---|---|---|
| 1 | `01_data_bm25` | CPU (butuh Java 21 untuk Pyserini) | – |
| 2 | `02_train_eval` dengan `EXPERIMENT='exp1'` | GPU **NVIDIA L4** | ± 4 jam (mBERT ± 25 menit/run, mDeBERTa ± 53 menit/run) |
| 3 | `02_train_eval` dengan `EXPERIMENT='exp3'` | GPU L4 | ± 1,25 jam |
| 4 | `03_topk_efficiency` | CPU | – |
| 5 | `04_analysis` | CPU | – |

Untuk mengulang **hanya analisis** (notebook 03–04) tanpa GPU:
1. Jalankan notebook 01 untuk membuat ulang qrels, run BM25, dan teks passage.
2. Salin isi folder `scores/` dan `results/` dari repositori ini ke `BASE_DIR` di Drive.

File besar berikut **tidak disertakan** karena bisa dibuat ulang oleh notebook 01: `passages.parquet` (± 40 MB), `run.bm25.{train,dev}.top100.txt`, `qrels.{train,dev}.tsv`, dan `topics.parquet`. Checkpoint model juga tidak disimpan. Setiap run hanya menyimpan skor dan metrik.

---

## Konfigurasi eksperimen

| Komponen | Nilai |
|---|---|
| First-stage | BM25, prebuilt index Pyserini `miracl-v1.0-id` (k1 = 0,9, b = 0,4), top-100 |
| Data latih | Train MIRACL-id: 90% training (3.664 kueri), 10% validasi (407 kueri), split per kueri, seed 42 |
| Pasangan latih | Positif = `rel=1`. Negatif = `rel=0` dari qrels, lalu passage tak berlabel dari top-30 BM25. Rasio 1:3 (aktual 1:2,99) |
| Model | `bert-base-multilingual-cased` (177,9 M param), `microsoft/mdeberta-v3-base` (278,8 M param) |
| Training | Pointwise BCE, AdamW, lr 2e-5, warmup 0,1, weight decay 0,01, batch 16, 3 epoch, max length 256, BF16, tanpa tuning |
| Pemilihan checkpoint | nDCG@10 tertinggi pada validasi 10% |
| Seed | Eksp. 1: 42, 100, 2026. Eksp. 3: 42 |
| Evaluasi | nDCG@10 (utama), MRR@10, MAP@100, Recall@k (`pytrec_eval`) |
| Efisiensi | Latency per kueri termasuk tokenisasi, batch inferensi 32, 20 kueri warmup dibuang (940 kueri diukur), puncak memori GPU |
| Signifikansi | Uji per kueri (n = 960, rata-rata 3 seed). Shapiro-Wilk, lalu paired t-test atau Wilcoxon. 95% CI (t & bootstrap), koreksi Holm. "Lebih efektif" = p < 0,05 dan Δ ≥ 0,01 |

---

## Lingkungan eksperimen

| Komponen | Versi / spesifikasi |
|---|---|
| Platform | Google Colab, Linux 6.6 (glibc 2.39), Python 3.13 |
| GPU (notebook 02) | NVIDIA L4, 23 GB VRAM, driver 580.82.07, CUDA 13.0 |
| CPU / RAM | Intel Xeon @ 2,20 GHz. RAM 52 GB (runtime L4), 12 GB (runtime CPU) |
| Library | torch 2.11.0, transformers 5.20.2, sentencepiece 0.2.2, numpy 2.1.3, pandas 2.2.3, scipy 1.16.3, pyarrow 23.0.1, matplotlib 3.10.0 |
| IR toolkit | Pyserini 2.4.0 (Anserini/Lucene, OpenJDK 21), pytrec-eval-terrier 0.5.11 |

Daftar paket ada di [`requirements.txt`](requirements.txt).

---

## Data & referensi

- **MIRACL:** Zhang, X., Thakur, N., Ogundepo, O., Kamalloo, E., Alfonso-Hermelo, D., Li, X., Liu, Q., Rezagholizadeh, M., & Lin, J. (2023). MIRACL: A multilingual retrieval dataset covering 18 diverse languages. *TACL, 11*, 1114–1131. https://doi.org/10.1162/tacl_a_00595. Data asli: https://huggingface.co/datasets/miracl/miracl
- **Pyserini:** https://github.com/castorini/pyserini
- **mBERT:** Devlin et al. (2019). **mDeBERTa-v3:** He, Gao, & Chen (2023).

Dataset dan korpus Wikipedia tunduk pada lisensinya masing-masing (lihat halaman dataset MIRACL). Lisensi di bawah ini hanya berlaku untuk kode dan hasil di repositori ini.

## Lisensi

[MIT](LICENSE)
