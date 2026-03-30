# Reconsiliation-Project

Proyek otomatisasi settlement dan rekonsiliasi transaksi untuk Youtap Indonesia dengan integrasi Google BigQuery.

## 📋 Deskripsi

Script ini melakukan proses settlement harian dengan:
- Mengambil data transaksi dari berbagai sumber (OVO, ShopeePay, LinkAja, Kredivo, Indodana, dll)
- Matching transaksi dengan data issuer/bank
- Menghitung MDR (Merchant Discount Rate), voucher, dan amount to transfer
- Menangani logic weekend dan hari libur nasional
- Upload hasil settlement ke BigQuery

## 🚀 Fitur

- **Multi-Source Reconciliation**: Integrasi dengan 15+ sumber data payment provider
- **Match/Unmatch Engine**: Pemisahan transaksi matched dan unmatched
- **Holiday Logic**: Penanganan transaksi weekend dan hari libur nasional
- **Voucher Processing**: Perhitungan voucher MCD_VOUCHER, YTI_VOUCHER, SNAP
- **OPS Balance Integration**: Integrasi dengan balance report
- **BigQuery Upload**: Upload otomatis hasil settlement ke BigQuery

## 📦 Dependencies

Install dependencies dengan menjalankan:

```bash
pip install -r requirements.txt
```

### Requirements (`requirements.txt`):
- `google-cloud-bigquery`
- `google-cloud-bigquery-storage`
- `pyarrow`
- `db-dtypes`
- `pandas`
- `numpy`
- `holidays`

## ⚙️ Konfigurasi

### Prerequisites

1. **Google Cloud Authentication**
   - Pastikan Anda telah menginstall Google Cloud SDK
   - Login dengan: `gcloud auth application-default login`
   - Atau set environment variable `GOOGLE_APPLICATION_CREDENTIALS` ke path service account JSON

2. **BigQuery Access**
   - Project ID: `youtap-indonesia-bi`
   - Pastikan memiliki akses ke dataset dan tabel yang diperlukan

### Dataset BigQuery yang Digunakan

| Dataset | Tabel | Deskripsi |
|---------|-------|-----------|
| `summary` | `merchants` | Data merchant dengan MDR rate |
| `summary` | `transactions` | Data transaksi summary |
| `summary` | `settlement_report` | Output settlement report |
| `summary` | `merchant_bank_accounts_v` | Data rekening bank merchant |
| `datawarehouses` | `yti_settlement_report_hourly` | Data transaksi YTI hourly |
| `datawarehouses` | `yti_settlement_mcd_report_hourly` | Data MCD settlement |
| `datawarehouses` | `recon_*` | Berbagai tabel rekonsiliasi |
| `datawarehouses` | `ops_balance_report` | OPS balance report |

## 🏃 Cara Menjalankan

```bash
python main.py
```

Script akan:
1. Menarik data dari BigQuery (6 query utama)
2. Memproses settlement dengan engine V15.8
3. Menghitung carry over, voucher, MDR, dan amount to transfer
4. Upload hasil ke BigQuery dengan replace data lama

## 📊 Output

Output disimpan di tabel BigQuery `summary.settlement_report` dengan kolom:

| Kolom | Deskripsi |
|-------|-----------|
| `BALANCE_DATE` | Tanggal settlement |
| `ACCOUNT_ID` | ID Akun Merchant |
| `MERCH_NAME` | Nama Merchant |
| `CARRY_OVER_BALANCE` | Saldo carry over dari periode sebelumnya |
| `TOTAL_GROSS_TRANSACTION` | Total transaksi kotor (matched) |
| `TOTAL_GROSS_BALANCE` | Total balance kotor |
| `GROSS_DIFFERENCE` | Selisih gross |
| `FIRST_TRX` | Nilai transaksi pertama |
| `TRX_00_AMT` | Transaksi pukul 00:00-00:01 |
| `AMOUNT_VOUCHER` | Total voucher |
| `AMOUNT_FINAL` | Amount final setelah penyesuaian |
| `BALANCE_FINAL` | Balance final |
| `TOTAL_MDR` | Total MDR |
| `TOTAL_NET_TRX` | Total transaksi net |
| `AMOUNT_TO_TRANSFER` | Amount yang akan ditransfer |
| `CONFIRM_TO_TRANSFER` | Status konfirmasi (CONFIRM / NOT MATCH / MINIMUM NOT MET) |
| `BANK_*` | Informasi rekening bank merchant |
| `TOTAL_*_NOT_MATCH` | Metrik transaksi yang tidak match |

## 🔧 Logic Settlement

### Status CONFIRM_TO_TRANSFER

- **CONFIRM**: Balance dan amount match (selisih ≤ 1)
- **NOT MATCH**: Selisih balance dan amount > 1
- **MINIMUM NOT MET**: Net transaction < 20.140 (minimum transfer)

### Holiday Carry Logic

Transaksi yang terjadi pada:
- Weekend (Sabtu-Minggu)
- Hari libur nasional

Akan di-carry ke hari kerja berikutnya.

### MDR Rate

MDR rate ditentukan berdasarkan:
- Account ID tertentu (rate khusus)
- Referral code acquisition
- Default: 0.7%

## 📅 Hari Libur Nasional 2026

Script menggunakan daftar hari libur manual untuk tahun 2026:
- 1 Januari (Tahun Baru)
- 16 Januari
- 17 Februari (Imlek)
- 19, 21, 22 Maret
- 3 April
- 1 Mei (Hari Buruh)
- 14, 27, 31 Mei
- 1 Juni
- 17 Agustus (HUT RI)
- 25 Desember (Natal)

## 🗂️ Struktur File

```
Reconsiliation-Project/
├── main.py                 # Script utama settlement engine
├── requirements.txt        # Python dependencies
└── README.md              # Dokumentasi (file ini)
```

## 🔐 Keamanan

- Jangan commit credentials Google Cloud ke repository
- Gunakan environment variables untuk sensitive configuration
- Pastikan service account memiliki permission minimal yang diperlukan

## 📝 Version History

- **v15.8**: GitHub Actions + BigQuery Storage API support
- **v15.7**: Integrated Unmatched Engine
- Fix bug weekend handling
- Fix bug Okasan Juice (duplicate account + transaction ID)

## 👤 Author

**Youtap BI Team**

Repository: https://github.com/youtapbi/Reconsiliation-Project

## 📄 License

Internal use only - Youtap Indonesia
