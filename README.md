# Semantic Search & LLM Application - FAQ E-commerce

Individual assignment AI Engineering Bootcamp. Isinya prototype asisten CS e-commerce yang bisa nyari FAQ paling relevan pakai semantic search (Gemini Embeddings + FAISS), terus jawabannya disusun ulang sama LLM (Gemini) berdasarkan FAQ yang ketemu.

## Problem Statement

**Objective**
Tim customer service e-commerce kebanjiran pertanyaan yang itu-itu aja tiap hari (soal pesanan, pengiriman, pembayaran, refund, dll). Tujuannya bikin prototype asisten AI yang bisa:
1. nemuin FAQ yang paling nyambung sama pertanyaan pelanggan, walaupun pelanggan nanya pakai kata-kata yang beda dari FAQ (misal pakai bahasa santai atau sinonim), dan
2. ngasih jawaban yang natural pakai LLM, tapi tetap berdasarkan isi FAQ.

Prototype ini nantinya jadi dasar sebelum dikembangin jadi RAG system yang lebih lengkap.

**Knowledge base**
- Sumber: `data/ecommerce_faq_knowledge_base.csv` (dari mentor, tidak diubah sama sekali).
- Isi: 59 FAQ, 4 kolom: `faq_id`, `category`, `question`, `answer`.
- 8 kategori: Pemesanan (9), Pengiriman (9), Pembayaran (8), Pengembalian (8), Akun (7), Produk (7), Promo (6), Customer Service (5).
- Data bersih, ga ada missing value atau pertanyaan dobel. Jawabannya pendek (rata-rata ~17 kata), jadi 1 FAQ langsung dijadiin 1 unit retrievable tanpa chunking.

**Batasan & asumsi**
- Sistem cuma boleh jawab berdasarkan FAQ yang ada. Kalau infonya ga ada, harus bilang ga tahu dan arahin ke customer service, bukan ngarang.
- Bahasa yang dipakai bahasa Indonesia (sesuai FAQ).
- Knowledge base dianggap statis, kalau ada FAQ baru ya embedding & index-nya harus dibuat ulang.
- Ga ada riwayat percakapan (tiap pertanyaan berdiri sendiri).
- Pakai Gemini API free tier, jadi ada limit request per menit (makanya di notebook ada retry & jeda).

## Alur Sistem

```
ingest  ->  embed  ->  store  ->  retrieve  ->  generate
```

| Tahap | Yang dilakuin |
|---|---|
| Ingest | Load CSV pakai Pandas, cek struktur data, gabungin `category + question + answer` jadi satu teks per FAQ |
| Embed | Tiap teks diubah jadi vektor pakai `gemini-embedding-001` (task type `RETRIEVAL_DOCUMENT`), hasilnya shape `(59, 3072)` |
| Store | Vektor dinormalisasi lalu dimasukin ke FAISS `IndexFlatL2` (59 vektor), index disimpan ke `vector_store/faq_index.faiss` |
| Retrieve | Query di-embed (task type `RETRIEVAL_QUERY`), dicari top-3 terdekat di FAISS. Ditampilin distance L2 dan similarity (cosine, karena vektornya udah dinormalisasi: `similarity = 1 - distance/2`) |
| Generate | Hasil retrieval dimasukin ke prompt (Persona, Task, Context, Format), dikirim ke `gemini-2.5-flash` dengan temperature 0.3 |

Catatan: di starter notebook modelnya `text-embedding-004` dan `gemini-2.0-flash`, tapi dua model itu udah ga ada di daftar model API key aku, jadi aku ganti ke `gemini-embedding-001` dan `gemini-2.5-flash`.

**Prompt design**
Prompt dibagi jadi beberapa bagian:
- **Persona**: asisten CS e-commerce yang ramah dan sopan.
- **Task**: jawab cuma dari CONTEXT, jangan ngarang angka/biaya/durasi, abaikan FAQ yang ga nyambung, dan kalau ga ada infonya bilang terus terang + sarankan hubungi CS.
- **Context**: top-3 FAQ hasil semantic search (lengkap sama kategori & skor similarity).
- **Format**: 2-4 kalimat, boleh pakai poin kalau ada langkah-langkah.

## Hasil Pengujian

Diuji pakai 6 query. Detail lengkapnya (top-3 + jawaban LLM) ada di notebook section 8 dan di [`documentation/retrieval_test_results.md`](documentation/retrieval_test_results.md).

| # | Jenis | Query | Top-1 FAQ | Similarity | Hasil |
|---|---|---|---|---|---|
| 1 | Ada di FAQ | Bagaimana cara membatalkan pesanan? | Bagaimana cara membatalkan pesanan yang sudah dibuat? | 0.788 | Tepat |
| 2 | Ada di FAQ | Berapa lama proses refund setelah barang dikembalikan? | Berapa lama proses refund setelah pengembalian disetujui? | 0.814 | Tepat |
| 3 | Parafrase | duit udah kepotong tapi orderan masih dibilang belum lunas, gimana nih? | Kenapa pembayaran saya berhasil dipotong tapi status pesanan masih belum terbayar? | 0.793 | Tepat |
| 4 | Parafrase | aku ga inget kata sandi buat masuk, caranya reset gimana ya | Saya lupa password akun, bagaimana cara mengaturnya ulang? | 0.769 | Tepat |
| 5 | Parafrase | barang yang dateng beda sama yang aku order | Saya menerima barang yang salah, apa yang harus saya lakukan? | 0.801 | Tepat |
| 6 | Tidak ada di FAQ | Apakah bisa kirim barang ke luar negeri, misalnya ke Jepang? | Apakah bisa memilih jasa pengiriman sendiri saat checkout? | 0.693 | LLM bilang ga ada infonya |

**Analisis singkat**
- 5 dari 5 query yang ada jawabannya berhasil dapet FAQ yang benar di rank 1, termasuk 3 query parafrase. Query 3 contohnya hampir ga ada kata yang sama ("duit kepotong" vs "pembayaran dipotong", "belum lunas" vs "belum terbayar"), tapi tetap ketemu. Jadi semantic search-nya memang nangkep makna, bukan cuma keyword.
- Ada sedikit noise di rank bawah. Contoh query 1 ("membatalkan pesanan"), rank 2 nya malah FAQ pengembalian barang. Tapi LLM cuma pakai FAQ yang relevan, jadi jawabannya tetap benar.
- Di query 5 LLM ngegabungin 2 FAQ (cara komplain + batas 3 hari), jadi jawabannya lebih lengkap.
- Query 6 (pengiriman luar negeri) skornya paling rendah (0.693) dan ketiga hasilnya skornya mepet (0.693/0.693/0.689), tanda ga ada FAQ yang benar-benar cocok. LLM ga ngarang dan ngarahin ke CS sesuai instruksi prompt.
- Semua angka di jawaban LLM (3-7 hari kerja, 15-30 menit, maksimal 3 hari) aku cek ulang ke CSV dan sesuai.

## Keterbatasan

- **FAISS selalu ngembaliin top-k**, relevan atau nggak. Belum ada threshold similarity, jadi FAQ yang ga nyambung tetap masuk ke prompt. Penanganan "ga tahu" sepenuhnya bergantung ke LLM nurut sama prompt.
- **Rentang skor similarity sempit** (sekitar 0.65-0.82), selisih antara relevan dan ga relevan cuma ~0.07-0.1. Kalau mau pasang threshold harus hati-hati (dari tes ini kira-kira di 0.72).
- **Pertanyaan yang nanya 2 hal sekaligus** bisa aja cuma kejawab sebagian karena konteksnya cuma top-3.
- **Gaya bahasa LLM belum konsisten**, kadang manggil "Kak", kadang "kamu".
- **Rate limit free tier** (5 request/menit untuk gemini-2.5-flash), jadi testing banyak query agak lama.
- Jawaban LLM bisa beda sedikit tiap run karena temperature 0.3 (isi tetap ngikutin FAQ).
- Knowledge base cuma 59 FAQ, jadi pakai `IndexFlatL2` (exact search) masih oke. Kalau datanya udah ribuan baru perlu index yang lebih efisien.

## Struktur Project

```
Semantic_Search_Assignment/
├── data/
│   └── ecommerce_faq_knowledge_base.csv     # knowledge base asli (ga diubah)
├── notebook/
│   └── starter_notebook_semantic_search.ipynb
├── vector_store/
│   ├── faq_index.faiss                      # FAISS index hasil dari notebook
│   └── faq_metadata.csv                     # mapping urutan vektor -> FAQ
├── documentation/
│   └── retrieval_test_results.md
├── README.md
└── requirements.txt
```

## Cara Menjalankan

1. Clone repo ini lalu masuk ke foldernya.
2. Install library:
   ```bash
   pip install -r requirements.txt
   ```
3. Bikin API key di [Google AI Studio](https://aistudio.google.com/apikey), terus simpan sebagai environment variable `GEMINI_API_KEY` (notebook ngambil key dari situ, ga ditulis di kode).
   - Windows (Command Prompt), habis itu tutup & buka lagi CMD/Jupyter-nya:
     ```
     setx GEMINI_API_KEY "MASUKKAN_API_KEY_DI_SINI"
     ```
   - macOS:
     ```
     echo 'export GEMINI_API_KEY="MASUKKAN_API_KEY_DI_SINI"' >> ~/.zshrc
     source ~/.zshrc
     ```
4. Buka notebook dari folder `notebook/` (path data di notebook pakai `../data/...`):
   ```bash
   cd notebook
   jupyter notebook starter_notebook_semantic_search.ipynb
   ```
5. Run All. Notebook bakal bikin embedding, nyimpen index ke `vector_store/`, lalu jalanin semua tes query. Karena ada jeda buat rate limit, section 8 butuh kira-kira 1-2 menit.
