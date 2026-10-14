# Hasil Uji Retrieval dan Jawaban LLM

Pengujian menggunakan `gemini-embedding-001` dengan dimensi 3072, FAISS `IndexFlatL2` dengan vektor yang sudah dinormalisasi, top_k = 3, dan LLM `gemini-2.5-flash` dengan temperature 0.3.
Nilai similarity yang ditampilkan adalah cosine similarity (`1 - distance/2`). Semakin tinggi nilainya, semakin mirip FAQ tersebut dengan query.

Hasil di bawah diambil dari notebook bagian 8. Kalimat jawaban LLM bisa sedikit berbeda jika notebook dijalankan ulang.

---

## 1. [Ada di FAQ] "Bagaimana cara membatalkan pesanan?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.788 | Pemesanan | Bagaimana cara membatalkan pesanan yang sudah dibuat? |
| 2 | 0.719 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? |
| 3 | 0.708 | Pemesanan | Kenapa pesanan saya otomatis dibatalkan sistem? |

**Jawaban LLM:**
> Halo! Untuk membatalkan pesanan, kamu bisa melakukannya lewat halaman 'Pesanan Saya'. Pembatalan hanya bisa dilakukan selama status pesananmu masih 'Diproses' ya. Kalau statusnya sudah 'Dikirim', pesanan tidak bisa dibatalkan.

**Catatan:** FAQ di rank 1 sudah tepat. FAQ di rank 2 kurang relevan karena membahas pengembalian barang, bukan pembatalan pesanan. Kemungkinan kata "membatalkan" dan "mengembalikan" dianggap memiliki makna yang mirip. LLM hanya memakai FAQ rank 1 sehingga jawabannya tetap sesuai.

---

## 2. [Ada di FAQ] "Berapa lama proses refund setelah barang dikembalikan?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.814 | Pengembalian | Berapa lama proses refund setelah pengembalian disetujui? |
| 2 | 0.732 | Pengembalian | Ke mana dana refund akan dikembalikan? |
| 3 | 0.721 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? |

**Jawaban LLM:**
> Halo Kak! Untuk proses refund setelah barang dikembalikan, dananya biasanya akan diproses dalam 3-7 hari kerja setelah barang diterima dan diverifikasi oleh penjual. Semoga informasinya membantu ya!

**Catatan:** Query ini mendapatkan skor tertinggi dari semua pengujian. Ketiga hasil berasal dari kategori yang sama dan masih berhubungan dengan pertanyaan. Angka 3-7 hari kerja pada jawaban sesuai dengan FAQ.

---

## 3. [Parafrase] "duit udah kepotong tapi orderan masih dibilang belum lunas, gimana nih?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.793 | Pembayaran | Kenapa pembayaran saya berhasil dipotong tapi status pesanan masih belum terbayar? |
| 2 | 0.721 | Pemesanan | Kenapa pesanan saya otomatis dibatalkan sistem? |
| 3 | 0.713 | Pembayaran | Berapa lama batas waktu pembayaran sebelum pesanan dibatalkan? |

**Jawaban LLM:**
> Hai Kak! Jangan khawatir, ini biasanya terjadi karena ada sedikit keterlambatan sinkronisasi di sistem pembayaran kami.
>
> Kakak bisa coba tunggu dulu sekitar 15-30 menit ya. Kalau setelah itu status pesanan masih belum berubah, mohon hubungi customer service kami untuk bantuan lebih lanjut.

**Catatan:** Ini adalah pengujian parafrase yang paling sulit karena hampir semua katanya berbeda dari FAQ, misalnya "duit" dengan "pembayaran", "orderan" dengan "pesanan", dan "belum lunas" dengan "belum terbayar". Selain itu query ditulis dengan bahasa tidak baku. FAQ yang tepat tetap ditemukan di rank 1.

---

## 4. [Parafrase] "aku ga inget kata sandi buat masuk, caranya reset gimana ya"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.769 | Akun | Saya lupa password akun, bagaimana cara mengaturnya ulang? |
| 2 | 0.715 | Akun | Kenapa akun saya tiba-tiba tidak bisa login? |
| 3 | 0.658 | Akun | Bagaimana cara mendaftar akun baru? |

**Jawaban LLM:**
> Halo! Kalau kamu lupa kata sandi, tenang saja, bisa direset kok. Caranya gampang:
> *   Klik 'Lupa Password' di halaman login.
> *   Masukkan email yang terdaftar.
> *   Ikuti instruksi reset password yang akan dikirimkan ke email kamu.
>
> Semoga membantu ya!

**Catatan:** Kalimat "ga inget kata sandi buat masuk" berhasil dicocokkan dengan FAQ "lupa password akun". FAQ di rank 2 juga masih relevan karena membahas masalah login.

---

## 5. [Parafrase] "barang yang dateng beda sama yang aku order"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.801 | Pengembalian | Saya menerima barang yang salah, apa yang harus saya lakukan? |
| 2 | 0.737 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? |
| 3 | 0.709 | Pengembalian | Apakah bisa menukar barang dengan ukuran/warna lain daripada refund? |

**Jawaban LLM:**
> Halo! Maaf ya atas ketidaknyamanannya. Kalau barang yang kamu terima beda dengan yang diorder, kamu bisa ajukan komplain 'Barang Tidak Sesuai' lewat menu Pesanan Saya.
>
> Caranya gampang kok:
> *   Pilih menu 'Pesanan Saya'.
> *   Ajukan komplain dengan memilih alasan 'Barang Tidak Sesuai'.
> *   Sertakan foto barang yang kamu terima sebagai bukti.
>
> Pengajuan ini bisa dilakukan maksimal 3 hari setelah barang diterima ya.

**Catatan:** LLM menggabungkan informasi dari FAQ rank 1 tentang cara mengajukan komplain dan FAQ rank 2 tentang batas waktu 3 hari. Hasilnya jawaban menjadi lebih lengkap dibanding hanya memakai satu FAQ.

---

## 6. [Tidak ada di FAQ] "Apakah bisa kirim barang ke luar negeri, misalnya ke Jepang?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.693 | Pengiriman | Apakah bisa memilih jasa pengiriman sendiri saat checkout? |
| 2 | 0.693 | Pengiriman | Berapa lama estimasi waktu pengiriman barang? |
| 3 | 0.689 | Pengiriman | Apakah tersedia opsi pengiriman same-day? |

**Jawaban LLM:**
> Halo! Untuk saat ini, aku belum punya informasi apakah bisa kirim barang ke luar negeri seperti Jepang. Mungkin kamu bisa langsung menghubungi customer service kami ya, biar bisa dibantu lebih lanjut.

**Catatan:** Retrieval tetap mengembalikan 3 FAQ dari kategori Pengiriman, tetapi tidak ada yang membahas pengiriman ke luar negeri. Skornya paling rendah dan ketiga hasil memiliki nilai yang hampir sama, berbeda dengan query lain yang skor rank 1 nya cukup jauh di atas rank 2. LLM tidak mengarang jawaban dan menyampaikan bahwa informasinya belum tersedia.

---

## Rangkuman

| Jenis | Jumlah query | Retrieval rank-1 tepat | Jawaban LLM sesuai |
|---|---|---|---|
| Ada di FAQ | 2 | 2/2 | 2/2 |
| Parafrase | 3 | 3/3 | 3/3 |
| Tidak ada di FAQ | 1 | - | 1/1 |

- Skor rank 1 untuk query yang jawabannya ada di FAQ berada di antara 0.769 sampai 0.814, sedangkan query yang tidak ada jawabannya hanya 0.693.
- Selisih skor rank 1 dan rank 2 pada query yang relevan sekitar 0.05 sampai 0.08, sedangkan pada query 6 hanya 0.0001. Selisih ini bisa dipakai sebagai tanda tambahan bahwa pertanyaan tidak ada di FAQ.
- Untuk pengembangan berikutnya, bisa ditambahkan batas minimum similarity sekitar 0.72 sebelum context dikirim ke LLM, sehingga sistem tidak hanya bergantung pada instruksi prompt.
