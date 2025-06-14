## 💸 Website Pengatur Keuangan Pribadi

Website sederhana untuk mencatat keuangan berdasarkan periode (mingguan, bulanan, tahunan), menghitung total pengeluaran, sisa anggaran, dan menyimpannya dalam bentuk PDF.

### ✨ Fitur

* Pilih periode: mingguan, bulanan, tahunan.
* Input keperluan dan nominalnya (bisa total atau per hari/minggu/bulan).
* Input uang yang dimiliki dan batas saldo ATM.
* Otomatis menghitung pengeluaran dan sisa anggaran.
* Bisa download hasil dalam format PDF.
* Form otomatis reset setelah hitung.
* Responsif dan ringan.

---

### 🚀 Cara Instalasi dan Menjalankan

#### 1. **Clone repository**

```bash
git clone https://github.com/namauser/nama-repo.git
cd nama-repo
```

#### 2. **Install dependensi**

Pastikan sudah menginstal [Node.js](https://nodejs.org) dan [npm](https://www.npmjs.com/):

```bash
npm install
```

#### 3. **Jalankan aplikasi**

```bash
npm run dev
```

Aplikasi akan bisa diakses di `http://localhost:5173`

---

### 🧾 Cara Menggunakan

1. Pilih periode: mingguan, bulanan, atau tahunan.
2. Masukkan jumlah uang yang kamu miliki sekarang.
3. Masukkan batas saldo ATM (opsional).
4. Masukkan daftar keperluan:

   * Nama keperluan
   * Nominal (atau biarkan kosong untuk dibagi rata otomatis)
   * Satuan (total, perhari, perminggu, perbulan)
5. Tekan tombol **Hitung & Download PDF**.
6. File PDF akan langsung diunduh ke laptop kamu.
7. Form akan otomatis direset setelah proses selesai.

---

### 🛠 Teknologi yang Digunakan

* Vue 3 + Vite
* JavaScript
* [jsPDF](https://github.com/parallax/jsPDF) untuk membuat PDF
* HTML + CSS (scoped styling)

---
