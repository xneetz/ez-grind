# EZ GRIND

**EZ GRIND** adalah aplikasi productivity planner local-first untuk mengelola project Web3, task, wallet publik, deadline, event kalender, serta pemasukan dan pengeluaran dalam satu workspace.

Versi saat ini: **1.9.45**

> Data utama disimpan di SQLite pada server milik sendiri. EZ GRIND tidak meminta seed phrase atau private key. Masukkan alamat wallet publik saja.

## Fitur utama

### Project workspace
- Membuat, mengubah, membuka detail, dan menghapus project.
- Logo, banner, deskripsi, chain/network, kategori, status, warna, dan tautan sosial per project.
- Progress project dihitung otomatis dari status task di dalamnya.
- Board kartu, pencarian, filter kategori/status, sorting, dan tabel project desktop.
- Kategori dan status dapat dikustomisasi dari Settings.

### Task execution
- Task selalu terhubung ke project dan mewarisi chain/network project.
- Deadline, status, prioritas, tipe task, wallet, brief/instructions, result notes, final URL, dan proof URL.
- Lampiran hingga 5 file per task, maksimum 10 MB per file.
- Preview, unduh, memo lampiran, dan penghapusan file.
- Alert untuk deadline yang mendekat atau terlewat.

### Wallet library
- Menyimpan label, network, alamat publik, foto, dan warna wallet.
- Menghubungkan wallet ke task.
- Melihat task yang menggunakan wallet tertentu dan menyalin alamat publik.

### Earnings & profit
- Pencatatan income dan expense per project.
- Ringkasan income, expense, net profit, dan jumlah transaksi.
- Tampilan Today, Daily, Monthly, dan Yearly dengan drill-down kalender.
- Filter berdasarkan project serta rincian sumber profit.

### Kalender dan alert
- Event independen dengan tanggal, waktu, jenis, catatan, warna, dan gambar.
- Agenda harian dan indikator event pada kalender.
- Alert untuk event mendatang, event yang baru berakhir, dan deadline task.

### Backup, tampilan, dan mobile
- Export/import full backup ZIP berisi SQLite database dan semua attachment.
- Safety backup otomatis sebelum restore; lima backup terbaru disimpan.
- Export backup langsung ke Google Drive menggunakan OAuth Client ID milik pengguna.
- Light/dark theme, custom app logo, dan UI responsif.
- Mobile app shell dengan sticky header, hero slider, drawer More, modal compact, dan bottom navigation tetap terlihat.

## Tech stack

- **Runtime:** Node.js 22.5+
- **Backend:** native Node.js HTTP server
- **Database:** SQLite melalui modul bawaan `node:sqlite`
- **Frontend:** HTML, CSS, dan vanilla JavaScript
- **Dependency npm:** tidak ada

## Menjalankan di localhost

### Persyaratan

- Node.js **22.5 atau lebih baru**
- Git, jika source diambil dari GitHub

Cek versi:

```bash
node --version
git --version
```

### Dari GitHub

```bash
git clone git@github.com:xneetz/ez-grind.git
cd ez-grind
npm start
```

Atau, setelah mengekstrak ZIP:

```bash
cd ez-grind
npm start
```

Buka `http://localhost:3000`.

Tidak perlu menjalankan `npm install` karena aplikasi hanya memakai modul bawaan Node.js.

### Mengubah port

Linux/macOS:

```bash
PORT=8080 npm start
```

Windows PowerShell:

```powershell
$env:PORT=8080
npm start
```

Server sengaja hanya mendengarkan `127.0.0.1`, sehingga deployment publik harus menggunakan reverse proxy seperti Nginx.

## Database dan penyimpanan

Pada startup pertama, aplikasi otomatis membuat:

```text
data/
├── ezdeck.db
├── attachments/
└── safety-backups/
```

Tabel utama:

| Tabel | Isi |
|---|---|
| `projects` | Project, branding, chain, kategori, status, dan link sosial |
| `tasks` | Task, deadline, status, priority, brief, dan hasil |
| `wallets` | Label, alamat publik, network, dan foto wallet |
| `earnings` | Income/expense yang terhubung ke project |
| `calendar_events` | Event dan memo kalender |
| `task_attachments` | Metadata attachment; file fisik berada di `data/attachments/` |
| `project_categories` | Kategori project yang dapat disesuaikan |
| `project_statuses` | Status project yang dapat disesuaikan |
| `task_types` | Tipe task yang dapat disesuaikan |
| `app_settings` | Theme, logo, dan penanda inisialisasi |

Fresh install hanya membuat opsi starter kategori/status/tipe task satu kali. Tidak ada dummy project, task, wallet, event, atau transaksi. Opsi yang sudah dihapus tidak dibuat kembali saat aplikasi direstart.

Database, attachment, backup, dan file environment sengaja tidak dikirim ke GitHub melalui `.gitignore`.

## Backup dan restore

Di aplikasi, buka **Settings → Full Backup**:

- **Export Full Backup** membuat ZIP berisi `ezdeck.db`, attachment, dan manifest.
- **Import Backup** memvalidasi SQLite serta attachment sebelum mengganti workspace.
- Sebelum restore, aplikasi otomatis menyimpan safety backup dari workspace saat ini.

## Instalasi di VPS

Panduan produksi lengkap tersedia di [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md). Ringkasannya:

1. Siapkan Ubuntu 24.04, Node.js 22+, Nginx, dan Git.
2. Clone private repository menggunakan SSH deploy key.
3. Jalankan sebagai service `systemd` menggunakan user non-root.
4. Reverse proxy domain melalui Nginx ke `127.0.0.1:3000`.
5. Aktifkan HTTPS dengan Certbot.
6. Tambahkan autentikasi sebelum membuka aplikasi ke internet.
7. Backup direktori data secara rutin.

## Health check

```bash
curl http://127.0.0.1:3000/api/health
```

Respons normal:

```json
{"ok":true,"database":"sqlite"}
```

## Catatan keamanan

- Jangan pernah menyimpan seed phrase, private key, token, atau password wallet di EZ GRIND.
- Aplikasi belum memiliki login bawaan. Jangan expose port Node.js langsung ke internet.
- Gunakan HTTPS dan autentikasi seperti Nginx Basic Auth, VPN/Tailscale, atau Cloudflare Access.
- Batasi permission direktori data hanya untuk user service.

## Batasan

EZ GRIND adalah planner dan tracker. Aplikasi tidak melakukan transaksi blockchain, tidak menyimpan private key, dan tidak mempublikasikan konten otomatis ke platform eksternal.

---

Created by **XNEETZ**.
