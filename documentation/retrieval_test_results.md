# Hasil Uji Retrieval & Jawaban LLM

Setup: `gemini-embedding-001` (3072 dimensi) + FAISS `IndexFlatL2` (vektor dinormalisasi), top_k = 3, LLM `gemini-2.5-flash` temperature 0.3.
Similarity = cosine similarity (`1 - distance/2`), makin tinggi makin mirip.

Output di bawah diambil dari hasil run notebook section 8. Kalimat jawaban LLM bisa sedikit beda kalau di-run ulang.

---

## 1. [Ada di FAQ] "Bagaimana cara membatalkan pesanan?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.788 | Pemesanan | Bagaimana cara membatalkan pesanan yang sudah dibuat? |
| 2 | 0.719 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? |
| 3 | 0.708 | Pemesanan | Kenapa pesanan saya otomatis dibatalkan sistem? |

**Jawaban LLM:**
> Halo! Untuk membatalkan pesanan, kamu bisa melakukannya lewat halaman 'Pesanan Saya'. Pembatalan hanya bisa dilakukan selama status pesananmu masih 'Diproses' ya. Kalau statusnya sudah 'Dikirim', pesanan tidak bisa dibatalkan.

**Catatan:** rank 1 tepat. Rank 2 kurang nyambung (pengembalian barang, bukan pembatalan), mungkin karena "membatalkan" dan "mengembalikan" dianggap mirip maknanya. LLM cuma pakai rank 1, jawaban sesuai FAQ.

---

## 2. [Ada di FAQ] "Berapa lama proses refund setelah barang dikembalikan?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.814 | Pengembalian | Berapa lama proses refund setelah pengembalian disetujui? |
| 2 | 0.732 | Pengembalian | Ke mana dana refund akan dikembalikan? |
| 3 | 0.721 | Pengembalian | Bagaimana prosedur pengembalian barang yang tidak sesuai? |

**Jawaban LLM:**
> Halo Kak! Untuk proses refund setelah barang dikembalikan, dananya biasanya akan diproses dalam 3-7 hari kerja setelah barang diterima dan diverifikasi oleh penjual. Semoga informasinya membantu ya!

**Catatan:** skor tertinggi dari semua tes. Ketiga hasil satu kategori dan masih nyambung. Angka 3-7 hari kerja sesuai FAQ.

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

**Catatan:** ini tes parafrase paling "berat" karena hampir semua katanya beda (duit/pembayaran, kepotong/dipotong, orderan/pesanan, belum lunas/belum terbayar) plus bahasa gaul. Tetap ketemu di rank 1.

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

**Catatan:** "ga inget kata sandi buat masuk" berhasil dicocokin ke "lupa password akun". Rank 2 juga masih relevan (masalah login).

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

**Catatan:** LLM ngegabungin rank 1 (komplain + foto) dan rank 2 (maksimal 3 hari), jadi jawabannya lebih lengkap dari satu FAQ aja.

---

## 6. [Tidak ada di FAQ] "Apakah bisa kirim barang ke luar negeri, misalnya ke Jepang?"

| Rank | Similarity | Kategori | FAQ |
|---|---|---|---|
| 1 | 0.693 | Pengiriman | Apakah bisa memilih jasa pengiriman sendiri saat checkout? |
| 2 | 0.693 | Pengiriman | Berapa lama estimasi waktu pengiriman barang? |
| 3 | 0.689 | Pengiriman | Apakah tersedia opsi pengiriman same-day? |

**Jawaban LLM:**
> Halo! Untuk saat ini, aku belum punya informasi apakah bisa kirim barang ke luar negeri seperti Jepang. Mungkin kamu bisa langsung menghubungi customer service kami ya, biar bisa dibantu lebih lanjut.

**Catatan:** retrieval tetap ngasih 3 FAQ (kategori Pengiriman, jadi topiknya kena) tapi ga ada yang bahas pengiriman internasional. Skornya paling rendah dan hampir sama rata, beda sama query lain yang rank 1 nya jauh di atas rank 2. LLM ga ngarang dan ngaku ga punya infonya.

---

## Rangkuman

| Jenis | Jumlah query | Retrieval rank-1 tepat | Jawaban LLM sesuai |
|---|---|---|---|
| Ada di FAQ | 2 | 2/2 | 2/2 |
| Parafrase | 3 | 3/3 | 3/3 |
| Tidak ada di FAQ | 1 | - | 1/1 (menolak dengan benar) |

- Skor top-1 query yang relevan: 0.769 - 0.814. Query yang ga ada jawabannya: 0.693.
- Selisih skor rank 1 vs rank 2 di query relevan sekitar 0.05-0.08, sedangkan di query 6 cuma 0.0001. Ini bisa jadi sinyal tambahan buat deteksi "ga ada jawaban".
- Ide pengembangan: pasang threshold similarity (sekitar 0.72) sebelum context dikirim ke LLM, biar ga cuma ngandelin prompt.
