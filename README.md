# 🪙 KEYTOCOIN (KTC) — Post-Quantum ML-DSA-65 Wallet & Node

KEYTOCOIN adalah implementasi jaringan blockchain eksperimental yang mengintegrasikan standar kriptografi tahan kuantum (**NIST FIPS 204 / ML-DSA-65**) secara langsung di browser, didukung oleh backend Node.js yang ringan, sistem hashing SHA3-256, serta simulasi CPU Mining.

---

## 🚀 Fitur Utama

- **Post-Quantum Cryptography:** Menggunakan algoritma tanda tangan digital **ML-DSA-65** via `@noble/post-quantum` untuk mengamankan dompet dan transaksi dari ancaman komputasi kuantum.
- **Enkripsi Klien Lokal:** Kunci privat disimpan secara lokal di browser menggunakan enkripsi **AES-256-GCM** dengan derivasi kunci **PBKDF2-SHA3250 (310,000 iterasi)**.
- **SHA3-256 Address Derivation:** Alamat dompet diturunkan secara kriptografis dari publik key menggunakan fungsi hashing SHA3-256.
- **CPU Mining Simulation:** Fitur penambangan koin langsung dari panel dasbor node dengan batas suplai maksimum (Max Supply) sebesar 45,000,000 KTC.
- **Block Explorer & Mempool:** Pemantauan status jaringan secara real-time, tinggi blok (*block height*), transaksi tertunda (*mempool*), dan riwayat blok.

---

## 📂 Struktur Proyek

Pastikan struktur direktori proyek Anda tersusun seperti ini:

```text
ktc-node/
├── public/
│   └── index.html      # Antarmuka web frontend (HTML + CSS + ES Module)
├── server.js           # Server backend Node.js (Express)
├── config.js           # Konfigurasi parameter jaringan & port (Opsional)
├── package.json        # Dependensi dan skrip proyek Node.js
└── README.md           # Dokumentasi proyek
