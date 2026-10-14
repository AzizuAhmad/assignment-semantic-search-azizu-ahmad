# Semantic Search & LLM Application - FAQ E-commerce

Repository ini berisi tugas individu AI Engineering Bootcamp. Pada tugas ini dibuat prototype asisten customer service e-commerce yang mencari FAQ paling relevan menggunakan semantic search (Gemini Embeddings dan FAISS). Setelah itu jawaban disusun oleh LLM (Gemini) berdasarkan FAQ yang ditemukan.

## Problem Statement

### Objective

Tim customer service e-commerce setiap hari menerima banyak pertanyaan yang berulang, misalnya tentang pesanan, pengiriman, pembayaran, dan pengembalian barang. Karena itu pada assignment ini dibuat prototype asisten AI yang bisa:

1. Menemukan FAQ yang paling sesuai dengan pertanyaan pelanggan, meskipun pelanggan bertanya dengan kata-kata yang berbeda dari FAQ.
2. Memberikan jawaban yang natural menggunakan LLM, tetapi isinya tetap mengikuti FAQ.

Prototype ini nantinya akan menjadi dasar untuk membangun RAG system yang lebih lengkap.

### Knowledge Base

- Sumber data: `data/ecommerce_faq_knowledge_base.csv`
- Jumlah data: 59 FAQ dengan 4 kolom, yaitu `faq_id`, `category`, `question`, dan `answer`.
- Terdapat 8 kategori: Pemesanan (9), Pengiriman (9), Pembayaran (8), Pengembalian (8), Akun (7), Produk (7), Promo (6), dan Customer Service (5).
- Tidak ada missing value maupun pertanyaan yang duplikat. Jawaban pada FAQ juga cukup pendek, rata-rata sekitar 17 kata, sehingga setiap FAQ langsung dijadikan satu unit retrievable tanpa perlu chunking.

### Batasan dan Asumsi

- Sistem hanya menjawab berdasarkan FAQ yang tersedia. Jika informasinya tidak ada, sistem harus menyampaikan bahwa informasi belum tersedia dan menyarankan pelanggan menghubungi customer service.
- Bahasa yang digunakan adalah bahasa Indonesia, sama seperti FAQ.
- Knowledge base dianggap tetap. Jika ada FAQ baru, embedding dan index perlu dibuat ulang.
- Setiap pertanyaan diproses secara terpisah dan tidak ada riwayat percakapan.
- API yang digunakan adalah Gemini API versi gratis yang punya batas jumlah request per menit, sehingga di notebook ditambahkan retry dan jeda waktu.

## Alur Sistem

```
ingest  ->  embed  ->  store  ->  retrieve  ->  generate
```

| Tahap | Proses |
|---|---|
| Ingest | Data CSV dimuat menggunakan Pandas, lalu dicek strukturnya. Kolom category, question, dan answer digabung menjadi satu teks untuk setiap FAQ. |
| Embed | Setiap teks diubah menjadi vektor menggunakan `gemini-embedding-001` dengan task type `RETRIEVAL_DOCUMENT`. Hasilnya berukuran `(59, 3072)`. |
| Store | Vektor dinormalisasi lalu disimpan ke FAISS `IndexFlatL2`. Total ada 59 vektor dan index-nya disimpan di `vector_store/faq_index.faiss`. |
| Retrieve | Query pelanggan diubah menjadi embedding dengan task type `RETRIEVAL_QUERY`, lalu dicari 3 FAQ terdekat di FAISS. Hasilnya ditampilkan bersama distance L2 dan similarity. Karena vektor sudah dinormalisasi, similarity dihitung dengan `1 - distance/2`. |
| Generate | Hasil retrieval dimasukkan ke dalam prompt, lalu dikirim ke `gemini-2.5-flash` dengan temperature 0.3. |

Di starter notebook model yang digunakan adalah `text-embedding-004` dan `gemini-2.0-flash`. Kedua model tersebut sudah tidak tersedia di API key yang saya gunakan, jadi diganti dengan `gemini-embedding-001` dan `gemini-2.5-flash`.

### Prompt Design

Prompt dibagi menjadi empat bagian:

- Persona: asisten customer service e-commerce yang ramah dan sopan.
- Task: menjawab hanya dari context, tidak mengarang angka, biaya, atau durasi, mengabaikan FAQ yang tidak berhubungan, dan jika informasinya tidak ada maka harus disampaikan dengan jujur serta menyarankan menghubungi customer service.
- Context: 3 FAQ teratas dari hasil semantic search beserta kategori dan nilai similarity-nya.
- Format: jawaban 2-4 kalimat, dan boleh memakai poin jika berisi langkah-langkah.

## Hasil Pengujian

Pengujian dilakukan dengan 6 query. Hasil lengkapnya bisa dilihat di notebook bagian 8 dan di [`documentation/retrieval_test_results.md`](documentation/retrieval_test_results.md).

| No | Jenis | Query | FAQ Rank 1 | Similarity | Hasil |
|---|---|---|---|---|---|
| 1 | Ada di FAQ | Bagaimana cara membatalkan pesanan? | Bagaimana cara membatalkan pesanan yang sudah dibuat? | 0.788 | Tepat |
| 2 | Ada di FAQ | Berapa lama proses refund setelah barang dikembalikan? | Berapa lama proses refund setelah pengembalian disetujui? | 0.814 | Tepat |
| 3 | Parafrase | duit udah kepotong tapi orderan masih dibilang belum lunas, gimana nih? | Kenapa pembayaran saya berhasil dipotong tapi status pesanan masih belum terbayar? | 0.793 | Tepat |
| 4 | Parafrase | aku ga inget kata sandi buat masuk, caranya reset gimana ya | Saya lupa password akun, bagaimana cara mengaturnya ulang? | 0.769 | Tepat |
| 5 | Parafrase | barang yang dateng beda sama yang aku order | Saya menerima barang yang salah, apa yang harus saya lakukan? | 0.801 | Tepat |
| 6 | Tidak ada di FAQ | Apakah bisa kirim barang ke luar negeri, misalnya ke Jepang? | Apakah bisa memilih jasa pengiriman sendiri saat checkout? | 0.693 | LLM menyampaikan informasi tidak tersedia |

### Analisis

- Kelima query yang jawabannya ada di FAQ berhasil mendapatkan FAQ yang benar di rank 1, termasuk 3 query parafrase. Pada query 3 hampir tidak ada kata yang sama dengan FAQ aslinya, misalnya "duit kepotong" dengan "pembayaran dipotong" dan "belum lunas" dengan "belum terbayar", tetapi FAQ yang tepat tetap ditemukan. Ini menunjukkan semantic search mencocokkan makna, bukan hanya kata kunci.
- Masih ada hasil yang kurang relevan di rank bawah. Pada query 1 tentang membatalkan pesanan, rank 2 justru FAQ tentang pengembalian barang. Namun LLM hanya memakai FAQ yang relevan sehingga jawabannya tetap benar.
- Pada query 5, LLM menggabungkan informasi dari 2 FAQ, yaitu cara mengajukan komplain dan batas waktu 3 hari, sehingga jawabannya lebih lengkap.
- Query 6 tentang pengiriman ke luar negeri memiliki skor paling rendah (0.693) dan ketiga hasilnya memiliki skor yang hampir sama (0.693, 0.693, 0.689). Ini menandakan tidak ada FAQ yang benar-benar cocok. LLM tidak mengarang jawaban dan mengarahkan pelanggan ke customer service sesuai instruksi di prompt.
- Angka yang muncul di jawaban LLM, seperti 3-7 hari kerja, 15-30 menit, dan maksimal 3 hari, sudah dicek kembali ke data FAQ dan hasilnya sesuai.

## Keterbatasan

- FAISS selalu mengembalikan top-k hasil walaupun hasilnya tidak relevan. Saat ini belum ada batas minimum similarity, sehingga FAQ yang tidak berhubungan tetap masuk ke prompt. Penanganan pertanyaan yang tidak ada jawabannya masih bergantung pada LLM yang mengikuti instruksi prompt.
- Rentang skor similarity cukup sempit, sekitar 0.65 sampai 0.82. Selisih antara hasil yang relevan dan tidak relevan hanya sekitar 0.07 sampai 0.1. Dari pengujian ini, batas minimum yang mungkin cocok berada di sekitar 0.72.
- Pertanyaan yang menanyakan dua hal sekaligus bisa saja hanya terjawab sebagian karena context yang diambil hanya 3 FAQ.
- Gaya bahasa LLM belum konsisten. Kadang memanggil pelanggan dengan "Kak" dan kadang dengan "kamu".
- Batas request Gemini API versi gratis adalah 5 request per menit untuk gemini-2.5-flash, sehingga pengujian dengan banyak query membutuhkan waktu lebih lama.
- Kalimat jawaban LLM bisa sedikit berbeda setiap kali dijalankan karena temperature 0.3, tetapi isinya tetap mengikuti FAQ.
- Jumlah FAQ hanya 59 sehingga `IndexFlatL2` masih cukup. Jika datanya sudah ribuan, sebaiknya menggunakan jenis index yang lebih efisien.

## Struktur Project

```
Semantic_Search_Assignment/
├── data/
│   └── ecommerce_faq_knowledge_base.csv
├── notebook/
│   └── starter_notebook_semantic_search.ipynb
├── vector_store/
│   ├── faq_index.faiss
│   └── faq_metadata.csv
├── documentation/
│   └── retrieval_test_results.md
├── README.md
└── requirements.txt
```

## Cara Menjalankan

1. Clone repository ini lalu masuk ke folder project.
2. Install library yang dibutuhkan:
   ```bash
   pip install -r requirements.txt
   ```
3. Buat API key di [Google AI Studio](https://aistudio.google.com/apikey), lalu simpan sebagai environment variable `GEMINI_API_KEY`. Notebook akan mengambil API key dari environment variable tersebut.
   - Windows (Command Prompt), setelah itu tutup dan buka kembali Command Prompt atau Jupyter:
     ```
     setx GEMINI_API_KEY "MASUKKAN_API_KEY_DI_SINI"
     ```
   - macOS:
     ```
     echo 'export GEMINI_API_KEY="MASUKKAN_API_KEY_DI_SINI"' >> ~/.zshrc
     source ~/.zshrc
     ```
4. Buka notebook dari dalam folder `notebook/`, karena path data di notebook menggunakan `../data/...`:
   ```bash
   cd notebook
   jupyter notebook starter_notebook_semantic_search.ipynb
   ```
5. Jalankan semua cell dengan Run All. Notebook akan membuat embedding, menyimpan index ke `vector_store/`, lalu menjalankan semua query pengujian. Karena ada jeda untuk batas request, bagian 8 membutuhkan waktu sekitar 1-2 menit.
