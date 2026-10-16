---
title: "Modul Pertemuan 5 — Storage dan Managed Data Services"
author: "Muhaemin Sidiq, S.Pd., M.Pd."
course: "Cloud Computing"
code: "PTI 2802"
program: "S1 Pendidikan Teknologi Informasi"
faculty: "Fakultas Ilmu Terapan dan Sains"
institution: "Institut Pendidikan Indonesia"
semester: "Ganjil 2026/2027"
meeting: 5
version: "1.0"
date: "2026-10-16"
language: "id-ID"
format: "Markdown / UTF-8"
---

# MODUL PEMBELAJARAN DAN PRAKTIKUM CLOUD COMPUTING

## Pertemuan 5 — Storage dan Managed Data Services

**Dari aplikasi stateless menuju layanan data persisten, aman, dapat dipulihkan, dan siap dikembangkan sebagai proyek individu**

| Identitas | Keterangan |
|---|---|
| Mata kuliah | **Cloud Computing** |
| Kode | **PTI 2802** |
| Program studi | **S1 Pendidikan Teknologi Informasi** |
| Fakultas | **Fakultas Ilmu Terapan dan Sains** |
| Perguruan tinggi | **Institut Pendidikan Indonesia** |
| Periode | **Semester Ganjil 2026/2027** |
| Pertemuan | **5 dari 16** |
| Dosen dan penyusun | **Muhaemin Sidiq, S.Pd., M.Pd.** |
| Sistem operasi server | **Ubuntu Server 24.04 LTS** |
| Sumber daya server | **VPS 1 vCPU, RAM 1 GB, disk 20 GB, public IPv4** |
| Perangkat mahasiswa | **Windows, Linux, iPhone/iPad, Android** |
| Produk akhir | **M05: persistent data + pilihan managed service + verified backup/restore + GitHub evidence** |
| Versi | **1.0 — 16 Oktober 2026** |

> **Prinsip desain:** proyek M03 dan M04 diteruskan, bukan diulang atau diganti total. GitHub tetap menjadi *source of truth* untuk source code dan dokumentasi, sedangkan data aplikasi berada pada storage yang persisten dan **tidak masuk ke repository**. Praktikum utama tetap dapat diselesaikan tanpa layanan berbayar. Jalur managed PostgreSQL merupakan latihan *service consumption* tambahan yang valid apabila akun, konektivitas, lokasi, dan biaya mengizinkan.

---

## Panduan penggunaan modul

Modul terdiri dari **materi konseptual**, **laboratorium wajib (A: SQLite persisten pada VPS)**, **laboratorium terkelola (B: PostgreSQL managed)**, **backup–restore**, **pengujian lintas perangkat**, serta **asesmen milestone M05**. Semua perintah tanpa penanda khusus dijalankan pada **terminal Ubuntu VPS**, termasuk ketika mahasiswa mengaksesnya melalui Windows, Linux, iPhone, atau Android. Perintah klien diberi label OS secara eksplisit.

Penulisan `PUBLIC_IP`, `DOMAIN`, `GITHUB_USER`, `REPO`, `DATABASE_URL`, dan `<...>` adalah placeholder. **Jangan menjalankan placeholder mentah**. Tidak pernah mencantumkan password, private SSH key, atau token nyata di screenshot, issue, commit, laporan, dan percakapan kelas.

**Prasyarat sebelum memulai:** URL HTTPS dan domain hasil M04 idealnya sudah berfungsi. Aplikasi tetap dapat diuji melalui SSH dan localhost jika DNS/TLS belum tersedia, tetapi mahasiswa **tidak boleh menggunakan HTTP publik untuk pengujian yang membawa kredensial**.

---

# BAGIAN I — KEDUDUKAN, CAPAIAN, DAN FONDASI TEORI

## 1. Integrasi berkelanjutan M03 → M04 → M05

```mermaid
flowchart TD
    M03["M03 — Project Inception: Flask + Gunicorn + systemd + Caddy"]
    M04["M04 — DNS + HTTPS/TLS + Secure Endpoint"]
    M05["M05 — Data Persistence + Managed PostgreSQL + Recovery"]
    M06["M06 — Containerization"]
    M07["M07 — Infrastructure as Code"]
    M09["M09 — CI/CD"]
    M16["M16 — Final project and defense"]
    M03 --> M04 --> M05 --> M06 --> M07 --> M09 --> M16
```

**Kondisi masuk M05** yang diharapkan:

- VPS masih Ubuntu Server 24.04; kapasitas tetap 1 vCPU, 1 GB RAM, 20 GB disk.
- Akun `cloudstudent` dapat masuk menggunakan SSH key; root login dan password SSH sudah dikendalikan sesuai M03.
- Repository GitHub pribadi berisi aplikasi Flask dan dokumentasi arsitektur.
- Direktori deployment di VPS: `/srv/cc-m03`.
- Aplikasi berjalan melalui service `cc-m03.service`, Gunicorn pada `127.0.0.1:8000`, serta Caddy sebagai reverse proxy.
- M04 menghasilkan domain/subdomain dan HTTPS, bila telah tersedia.
- Port backend dan port database **tidak** dibuka ke internet.

Target evolusi:

```text
Sebelum M05:
Browser -> HTTPS/Caddy -> Gunicorn -> Flask -> respons tanpa data persisten

Sesudah M05 (Jalur A):
Browser -> HTTPS/Caddy -> Gunicorn -> Flask -> SQLAlchemy -> SQLite file persisten
                                                    |
                                            /var/lib/cc-m03/

Sesudah M05 (Jalur B):
Browser -> HTTPS/Caddy -> Gunicorn -> Flask -> SQLAlchemy
                                                    |
                                           PostgreSQL managed
                                                    |
                                        encrypted DB connection
```

**Catatan penting:** *persisten* tidak berarti otomatis *high availability*, *durable across provider failure*, *backup*, atau *managed*. Kualitas ini mempunyai pengertian serta mekanisme yang berbeda [1]–[3].

## 2. Capaian pembelajaran dan Sub-CPMK

Setelah mengikuti pembelajaran, mahasiswa mampu:

| Kode | Capaian terukur | Evidence |
|---|---|---|
| M05-1 | Menganalisis block/file/object storage serta perbedaan storage dan database service | tabel pemilihan, studi kasus |
| M05-2 | Menjelaskan data persistence, transaksi ACID, schema, constraints, indexing, connection lifecycle, dan migration | penjelasan & praktikum |
| M05-3 | Menerapkan database persisten tanpa memasang RDBMS berat pada VPS 1 GB | SQLite pada direktori data di luar source |
| M05-4 | Mengimplementasikan operasi CRUD terkontrol dan endpoint baca-saja | CLI dan GET `/api/tasks` |
| M05-5 | Membedakan **self-managed DB** dari **managed DB service** melalui PostgreSQL terkelola | koneksi PostgreSQL opsional/terarah |
| M05-6 | Mengamankan kredensial, koneksi, file, akses, dan data | pemeriksaan permission/secret |
| M05-7 | Membuat backup yang konsisten, melakukan pemeriksaan integritas, dan menguji pemulihan | backup file + verifikasi + restore drill |
| M05-8 | Mengobservasi latency, kegagalan DB, disk/ram, log, dan status readiness | diagnosis berlapis |
| M05-9 | Mendokumentasikan keputusan, hasil uji, dan risiko pada repository GitHub pribadi | ADR-0002, README, evidence, tag `m05` |

### Kriteria selesai (Definition of Done)

Mahasiswa menyelesaikan M05 apabila dapat menunjukkan aplikasi yang tetap memiliki data setelah **restart service**, tetap mengakses data melalui HTTPS yang telah tersedia, memiliki sumber kode dan dokumentasi di GitHub, serta memiliki **bukti backup dan restore yang diverifikasi**. Mahasiswa wajib menjelaskan apakah solusi yang digunakan adalah database lokal atau managed service, bukan menuliskan dua istilah itu secara bergantian.

## 3. Mengapa data persistence menjadi kompetensi utama?

Proses aplikasi berjalan dalam memori. Nilai yang hanya disimpan dalam list atau variabel Python lenyap ketika proses berhenti; data persisten memerlukan medium dan mekanisme yang sesuai. Pada cloud, compute dan storage dapat mempunyai lifecycle berbeda. Sebuah VPS dengan disk persisten menyimpan file di luar lifetime proses, tetapi disk bisa hilang akibat penghapusan VPS/volume, kerusakan, tindakan operator, atau incident provider.

Perbedakan:

| Istilah | Artinya | Contoh M05 |
|---|---|---|
| Volatile | hilang saat proses/reboot tertentu | objek Python di RAM |
| Persistent | tetap ada setelah proses restart | file SQLite di disk |
| Durable | data bertahan sesuai karakteristik penyimpanan dan failure model | redundansi/replication sesuai layanan |
| Available | dapat diakses sesuai target uptime | DB merespons request |
| Recoverable | dapat dipulihkan dari kegagalan | backup dan uji restore |
| Managed | operasi tertentu ditangani provider | provisioning, maintenance/patching DB sesuai SLA/fitur paket |

**Persistensi diuji, tidak diasumsikan.** Minimal mahasiswa menyimpan data, me-restart `cc-m03`, kemudian membaca ulang data yang sama. Bukti yang lebih kuat melibatkan backup dan restore terpisah.

## 4. Tiga model penyimpanan: block, file, object

Model-model ini berbeda pada antarmuka, metadata, granularitas, dan pola akses. Contoh EBS/EFS/S3 adalah implementasi provider, bukan definisi universal [2], [3].

| Kriteria | Block storage | File storage | Object storage |
|---|---|---|---|
| Unit akses | block/device | file dalam hierarchy | object + key + metadata |
| Interface umum | disk/volume, filesystem di atasnya | POSIX/NFS/SMB | HTTP/API SDK |
| Penggunaan khas | OS, database volume | shared directories, dokumen | unggahan, media, backup, archive |
| Contoh | EBS/volume VPS | EFS/shared NFS | S3/Blob/Object Storage |
| Sesuai untuk database SQLite aktif | ya, melalui filesystem lokal | hati-hati locking/network filesystem | **tidak** sebagai file database SQLite langsung |
| Pola scaling | kapasitas volume dan IOPS | shared access | object access melalui API |

### 4.1 Block storage pada VPS

Disk 20 GB yang ditampilkan oleh VPS biasanya diakses oleh OS sebagai block device yang berisi partisi dan filesystem. Observasi non-destruktif:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
findmnt -T /srv/cc-m03
df -h /
df -i /
```

**Jangan jalankan** `mkfs`, `fdisk`, `parted`, `wipefs`, atau perubahan mount/partition pada disk aktif hanya untuk praktikum ini. Tugas M05 adalah memahami penyimpanan pada VPS yang sedang melayani aplikasi, bukan menghapus partisi.

### 4.2 File storage dan permission

Linux mengelola file melalui owner, group, permission, dan mount. Contoh struktur data yang akan dibuat:

```text
/srv/cc-m03/             source code; dikelola Git
/var/lib/cc-m03/         data aplikasi persisten; tidak di-Git
/var/backups/cc-m03/     backup lokal sementara; salin keluar VPS
/etc/cc-m03/m05.env     konfigurasi rahasia; tidak di-Git
```

Mengapa tidak menyimpan SQLite di `/srv/cc-m03/data`? Secara teknis boleh jika dikelola hati-hati, tetapi memisahkan state dari release kode lebih aman untuk proses checkout, redeploy, backup, permission, dan workflow container/IaC pada pertemuan selanjutnya.

### 4.3 Object storage

Object storage cocok untuk berkas foto, lampiran, ekspor, atau backup yang diakses melalui API. Tidak sama dengan mount POSIX pada local filesystem. Diskusikan bucket, object key, metadata, IAM/presigned URL, encryption, lifecycle, versioning, dan biaya egress [2], [3].

> **Latihan pilihan layanan:** sebuah aplikasi penilaian akademik menyimpan transaksi hasil kuis pada database, berkas soal PDF pada object storage, dan cache per-request dalam RAM. Mahasiswa harus mampu memberi alasan mengapa ketiga jenis data ini tidak disimpan dengan cara yang sama.

## 5. Database services: lokal, self-managed, dan managed

| Alternatif | Komponen yang dioperasikan mahasiswa | Keuntungan | Risiko/konsekuensi |
|---|---|---|---|
| SQLite pada VPS | file, schema, query, backup, restore | paling ringan, tidak ada daemon database | satu host; backup dan durability ditanggung pengguna |
| PostgreSQL self-hosted pada VPS | engine DB, service, patch, tuning, backup, akses | kontrol tinggi | beban operasional/RAM tambahan, tidak tepat sebagai jalur dasar VPS 1 GB |
| Managed PostgreSQL | koneksi, schema, role, aplikasi, kebijakan data | provider mengelola sebagian operasi server DB | network, pricing/quota, region, availability, vendor responsibility, backup policy |

**“Database di cloud” tidak otomatis berarti “managed database”.** PostgreSQL yang dipasang sendiri pada VM tetap membutuhkan operasi/patching dan pemeliharaan oleh pemilik VM. Klasifikasi layanan cloud harus mempertimbangkan karakteristik model operasional [1], [4].

Layanan managed tidak menghapus tanggung jawab konsumen atas **query yang salah, kesalahan aplikasi, privilege berlebihan, data pribadi, atau pemulihan yang belum pernah diuji**.

## 6. Perancangan data relasional

Reference project memakai entitas `tasks` sebagai contoh minimal, yang boleh diadaptasi menjadi `books`, `equipment`, `events`, atau entitas relevan proyek mahasiswa.

```mermaid
classDiagram
    class Task {
       +Integer id PK
       +String title
       +Boolean done
       +DateTime created_at
    }
```

Ketentuan:

- **Primary key** mengidentifikasi record.
- **NOT NULL** mencegah nilai kosong pada kolom tertentu.
- **Data type** menjaga kesesuaian bentuk data.
- **Transaction** membuat satu unit operasi berhasil atau dibatalkan.
- **Index** mempercepat pola akses tertentu dengan biaya maintenance/storage.
- **Migration** adalah perubahan struktur antarversi skema, **bukan** sekadar `CREATE TABLE IF NOT EXISTS`.

### 6.1 ACID, consistency, dan concurrency

ACID adalah *Atomicity*, *Consistency*, *Isolation*, dan *Durability*. Pada tugas M05, operasi tulis dijalankan melalui transaction (`engine.begin()`). Untuk SQLite yang berjalan pada VPS tunggal, jangan mengasumsikan paralelisme tulis yang sama seperti layanan PostgreSQL terkelola. Lebih penting mengukur serta memahami beban aktual sebelum memilih arsitektur baru.

### 6.2 SQL injection dan query terparameterisasi

Penggabungan string mentah dari input pengguna ke perintah SQL dapat membuka injection. SQLAlchemy menyediakan SQL expression dan parameter binding. Reference project menggunakan SQLAlchemy Core dan membatasi perubahan record pada CLI SSH terautentikasi. **Tidak ada POST/DELETE API publik tanpa autentikasi**.

### 6.3 Relasi database dan object storage

Data terstruktur sebaiknya tidak otomatis disimpan sebagai JSON/teks file biasa tanpa alasan. Sebaliknya, mengirim file multimedia besar ke kolom database juga perlu dipertimbangkan terhadap object storage, backup size, dan retrieval pattern.

## 7. Connection lifecycle dan biaya sumber daya

Untuk satu VPS kecil, gunakan connection pool konservatif, yakni contoh **`pool_size=1` dan `max_overflow=1`**. `pool_pre_ping=True` dapat membantu mendeteksi koneksi pooled yang sudah terputus sebelum digunakan kembali [5]. Konfigurasi ini bukan ukuran performa universal; angka dipilih sebagai titik awal praktikum 1 vCPU/1 GB.

Managed PostgreSQL menambahkan hop jaringan, latensi, DNS, koneksi TLS, dan batas koneksi layanan. Gunakan connection string dari provider, bukan mengarang hostname/port. Bila provider menawarkan pooled/direct endpoint, pahami kapan pooling atau direct connection sesuai untuk aplikasi dan operasi migration [6].

## 8. Database schema lifecycle

Untuk bootstrap M05, `metadata.create_all()` akan membuat tabel yang belum ada. **Metode itu tidak mengubah struktur tabel lama dengan aman ketika model berubah** [7]. Pada proyek yang bertumbuh, gunakan migration tool seperti Alembic beserta review dan rencana rollback, bukan `drop_all()/create_all()` yang menghapus data.

Rencana:

```text
M05: initial schema + controlled data operations
M06: container + volume/data lifecycle
M07: IaC + secret configuration boundary
M09: CI/CD + migration gate
M12: backup/recovery + monitoring lebih lanjut
```

## 9. Backup, restore, RPO, dan RTO

| Istilah | Definisi operasional |
|---|---|
| Backup | salinan data yang dapat digunakan untuk pemulihan |
| Restore | proses mengembalikan data dari backup |
| RPO | jumlah kehilangan data maksimal yang dapat ditoleransi dalam satuan waktu |
| RTO | durasi pemulihan layanan maksimal yang dapat ditoleransi |
| Snapshot | representasi state pada titik waktu tertentu, dengan semantik spesifik provider |
| PITR | point-in-time recovery, jika didukung paket/provider |

Contoh simulasi: jika backup harian dilakukan setiap 24 jam dan tidak ada log/PITR, potensi kehilangan data dari backup terakhir dapat mendekati 24 jam. Ini bukan jaminan RPO nyata tanpa implementasi dan uji.

**Backup yang hanya berada di disk VPS yang sama tidak cukup untuk melindungi dari kehilangan VPS.** Praktikum memerlukan satu salinan ke perangkat mahasiswa atau storage terpisah sesuai kebijakan institusi. Untuk SQLite aktif, **jangan menggunakan `cp` mentah terhadap file database yang sedang berjalan**, khususnya bila terdapat journal/WAL. SQLite menyediakan Online Backup API yang didesain untuk backup konsisten [8], [9]; peran file `-wal` saat WAL aktif dijelaskan dalam dokumentasi SQLite [18].

## 10. Matriks pemilihan storage bagi proyek individu

| Kebutuhan proyek | Rekomendasi awal | Alasan |
|---|---|---|
| To-do/task kecil, single VPS | SQLite + backup | ringan dan bisa diuji |
| Data relasional multipengguna berkembang | PostgreSQL managed | concurrency dan operasional DB terpisah |
| Dokumen/foto/lampiran | object storage | API, metadata, lifecycle |
| File sharing beberapa VM | managed/shared file storage | shared filesystem semantics |
| Cache sementara | memory/cache service | sifat data boleh hilang, jika sesuai requirement |
| Data sensitif/identitas siswa | rancang privacy dan access control terlebih dahulu | bukan dataset terbuka untuk demo |

---

# BAGIAN II — STRATEGI LINTAS PERANGKAT DAN PRA-LAB

## 11. Alat pada empat platform

| Platform klien | SSH & terminal | Browser | Cara menjalankan praktikum |
|---|---|---|---|
| Windows 10/11 | Windows Terminal + OpenSSH | Edge/Chrome/Firefox | SSH VPS; Git lokal opsional |
| Linux | OpenSSH + Bash | Firefox/Chrome | SSH VPS; Git lokal opsional |
| iPhone/iPad | Termius atau SSH client setara | Safari | SSH VPS, `nano`, GitHub web, verifikasi HTTPS |
| Android | Termius; alternatif Termux | Chrome/Firefox | SSH VPS, `nano`, GitHub web, `curl` jika tersedia |

Semua command instalasi dependency Python, edit service, backup, CLI data, dan pengujian internal dijalankan **di VPS**, sehingga tidak memerlukan Python/SQLAlchemy lokal pada iPhone atau Android. Windows/Linux juga dapat mengembangkan source di PC lokal lalu melakukan push/pull melalui GitHub sebagaimana M03.

### 11.1 Windows: akses VPS

```powershell
ssh -V
ssh -i "$HOME\.ssh\pti2802_m03" cloudstudent@<PUBLIC_IP>
```

Jika alias SSH digunakan pada M03, jalankan alias tersebut. Jangan menyalin private key ke perangkat lain.

### 11.2 Linux: akses VPS

```bash
ssh -V
ssh -i ~/.ssh/pti2802_m03 cloudstudent@<PUBLIC_IP>
```

### 11.3 iPhone/iPad: Termius

1. Buka Termius atau aplikasi SSH yang telah digunakan pada M03.
2. Pilih host VPS dan pastikan alamat public IPv4 benar.
3. Pilih user `cloudstudent` dan private key yang tersimpan secara aman di perangkat.
4. Sambungkan ke port SSH yang benar (biasanya 22 bila tidak diubah).
5. Jalankan `whoami`, `pwd`, dan `sudo -v` untuk memeriksa koneksi.
6. Gunakan keyboard tambahan bila tersedia; paste kode panjang secara hati-hati. `nano` menyimpan dengan **Ctrl+O**, Enter, lalu keluar dengan **Ctrl+X**; pada aplikasi mobile tombol Ctrl dapat berada pada toolbar tambahan.
7. Gunakan Safari untuk menguji `https://DOMAIN/ready` dan `https://DOMAIN/api/tasks`.

### 11.4 Android: Termius atau Termux

**Termius:** gunakan host serta SSH key yang sudah dibuat pada M03, lalu jalankan perintah yang sama di shell VPS.

**Termux (opsional):**

```bash
pkg update
pkg install openssh curl git
ssh -i ~/.ssh/pti2802_m03 cloudstudent@<PUBLIC_IP>
```

Perintah `pkg` dijalankan **pada Termux lokal Android**, sedangkan `apt`, `sudo systemctl`, dan operasi `/srv/cc-m03` dijalankan **pada Ubuntu VPS setelah login SSH**. Jika Termux tidak dipakai, browser + Termius tetap cukup.

### 11.5 Verifikasi akses dan kondisi M04

Di VPS:

```bash
whoami
cat /etc/os-release | head -n 5
nproc
free -h
df -h /
systemctl status cc-m03 --no-pager
systemctl status caddy --no-pager
sudo ufw status verbose
sudo ss -lntp
curl -fsS http://127.0.0.1:8000/health
```

Uji HTTPS publik dari perangkat klien atau browser:

```text
https://DOMAIN/health
```

Jika M04 belum sempurna, kembangkan data secara internal terlebih dahulu. Jangan membuka port `5432`, `8000`, atau port SQLite apa pun ke internet. SQLite tidak mempunyai port server.

## 12. Preflight keselamatan VPS

**Sebelum mengubah source:**

1. Catat commit/tag M04 terakhir; pastikan perubahan lokal sudah disimpan.
2. Pastikan console/rescue provider VPS dapat diakses apabila SSH bermasalah.
3. Gunakan data latihan/sintetis saja.
4. Jangan membuat database PostgreSQL lokal pada jalur dasar VPS 1 GB.
5. Pastikan ruang disk cukup; jangan mengisi backup hingga disk penuh.
6. Jangan meletakkan koneksi database/secret di README, Issue, shell history, screenshot, atau Git.
7. Jangan mengubah firewall SSH dan port HTTPS kecuali benar-benar diperlukan.
8. **Jangan menghapus** `/srv/cc-m03` atau resource VPS dari M03/M04.

```bash
cd /srv/cc-m03
git status --short
git log -5 --oneline --decorate
git branch --show-current
```

Jika ada modifikasi belum di-commit, selesaikan/backup terlebih dahulu sebelum menerapkan starter M05. Buat branch integrasi:

```bash
git switch -c feat/m05-persistence
```

Bila branch sudah ada, gunakan `git switch feat/m05-persistence`. Pada workflow Windows/Linux, branch dapat dibuat di komputer lokal terlebih dahulu lalu dipush; pada workflow mobile, mahasiswa dapat mengerjakan branch di VPS dengan izin GitHub M03 yang sesuai.

---

# BAGIAN III — LABORATORIUM A: PERSISTENSI LOKAL YANG BENAR

## 13. Tujuan, batas, dan urutan praktikum

Jalur A **wajib** untuk semua mahasiswa. Tujuannya memberi evidence objektif bahwa penambahan database meningkatkan kemampuan aplikasi tanpa memerlukan layanan berbayar tambahan.

```text
A1. Audit M03/M04
A2. Pisahkan lokasi code dan state
A3. Integrasikan model SQLAlchemy
A4. Inisialisasi schema
A5. Input–read–update–delete via SSH CLI
A6. Ekspos GET read-only via HTTPS
A7. Restart service, uji persistensi
A8. Backup online, verifikasi, restore drill
A9. Commit code/doc/evidence yang telah disanitasi
```

## 14. Menyiapkan direktori state, backup, dan secret

**Jalankan pada VPS sebagai user `cloudstudent`.**

```bash
sudo install -d -o cloudstudent -g cloudstudent -m 0700 /var/lib/cc-m03
sudo install -d -o cloudstudent -g cloudstudent -m 0700 /var/backups/cc-m03
sudo install -d -o root -g cloudstudent -m 0750 /etc/cc-m03
sudo install -o root -g cloudstudent -m 0640 /dev/null /etc/cc-m03/m05.env
```

Isi file konfigurasi eksternal:

```bash
sudo nano /etc/cc-m03/m05.env
```

Untuk jalur SQLite lokal:

```dotenv
APP_ENV=development
M05_DATA_DIR=/var/lib/cc-m03
# Tidak ada DATABASE_URL untuk jalur SQLite.
```

Validasi permission **tanpa mencetak isinya**:

```bash
stat -c '%A %U:%G %n' /etc/cc-m03 /etc/cc-m03/m05.env
stat -c '%A %U:%G %n' /var/lib/cc-m03 /var/backups/cc-m03
```

Pada Ubuntu 24.04 yang memakai user standar M03, `cloudstudent` berhak membaca file ini melalui group tetapi pengguna lain tidak. **Jangan menyalin file tersebut ke GitHub**.

## 15. Menyiapkan Python environment dan dependency

```bash
cd /srv/cc-m03
python3 --version
.venv/bin/python --version
```

Jika `.venv` sudah ada dari M03, **jangan hapus**. Tambahkan pustaka M05 di environment yang sama:

```bash
.venv/bin/python -m pip install \
  'Flask>=3.1,<4' \
  'Gunicorn>=23,<25' \
  'SQLAlchemy>=2.0,<3' \
  'psycopg[binary]>=3.2,<4' \
  'python-dotenv>=1.0,<2'
```

Untuk release proyek, lebih baik menggunakan dependency lock/pin yang diuji pada lingkungan sebenarnya. Starter menyediakan `requirements.txt` dengan *bounded major version ranges* agar dapat disiapkan, kemudian **setelah lulus uji** catat versi aktual ke release lock:

```bash
.venv/bin/python -m pip freeze > requirements.lock.txt
.venv/bin/python -m pip check
```

Saat update berikutnya, gunakan file yang telah direview (atau strategi penguncian dependency yang disepakati dosen). Jangan menjalankan upgrade massal tanpa tes pada VPS produksi praktikum.

## 16. Struktur source pada M05

Tambahkan/perbarui artefak berikut pada repository hasil M03/M04, bukan membuat repository baru:

```text
/srv/cc-m03/
├── app.py                   # endpoint aplikasi (lanjutan M03)
├── db.py                    # abstraksi SQLite/PostgreSQL
├── templates/index.html
├── static/style.css
├── scripts/
│   ├── __init__.py
│   ├── data_cli.py
│   ├── backup_sqlite.py
│   ├── verify_sqlite_backup.py
│   └── migrate_sqlite_to_postgres.py
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── deployment.md
│   ├── adr/0002-data-persistence.md
│   └── evidence/m05/
├── requirements.txt
├── requirements.lock.txt
├── .env.example
├── .gitignore
└── README.md

/var/lib/cc-m03/app.sqlite3     # jangan di Git
/var/backups/cc-m03/...         # backup; jangan di Git
/etc/cc-m03/m05.env             # secrets; jangan di Git
```

> **Penggunaan paket starter:** file-file referensi terlampir dalam paket `source/`. Pada Windows/Linux salin perubahan ke checkout lokal lalu commit–push–pull. Pada iPhone/Android gunakan SSH + `nano` dan GitHub (atau fitur editor web GitHub) untuk mengadaptasi file. Jangan mengganti source proyek individu tanpa memeriksa bagian fitur dan route yang sudah dibuat pada M03/M04.

## 17. Implementasi `db.py`: satu antarmuka, dua backend

Fungsi penting:

- `database_url()` memilih SQLite lokal atau PostgreSQL terkelola melalui konfigurasi **di luar Git**.
- `engine()` membuat engine SQLAlchemy dan pool konservatif.
- `initialize_schema()` membuat tabel awal; **bukan migration engine**.
- `add_task`, `set_task_done`, `remove_task`, dan `list_tasks` memakai transaction/parameter binding.
- `health_db()` memverifikasi konektivitas DB.

Perhatikan path SQLite `sqlite:////var/lib/cc-m03/app.sqlite3`: **empat slash** berarti path absolut pada POSIX [10]. Source referensi lengkap berikut disertakan agar file Markdown mandiri, bukan hanya instruksi abstrak.


### 17.1 Kode lengkap `db.py`

```python
"""Storage layer for Cloud Computing M05: local SQLite or managed PostgreSQL."""
from __future__ import annotations

from datetime import datetime, timezone
from functools import lru_cache
import os
from pathlib import Path

from dotenv import load_dotenv
from sqlalchemy import (
    Boolean, Column, DateTime, Integer, MetaData, String, Table, create_engine,
    delete, insert, select, update, text,
)

# On the VPS this file is outside the Git repository and mode 0640.
load_dotenv(os.environ.get("M05_ENV_FILE", "/etc/cc-m03/m05.env"), override=False)

metadata = MetaData()
tasks = Table(
    "tasks", metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("title", String(160), nullable=False),
    Column("done", Boolean, nullable=False, default=False),
    Column("created_at", DateTime(timezone=True), nullable=False),
)


def database_url() -> str:
    uri = os.environ.get("DATABASE_URL", "").strip()
    if uri:
        if uri.startswith("postgres://"):
            uri = "postgresql+psycopg://" + uri[len("postgres://"):]
        elif uri.startswith("postgresql://"):
            uri = "postgresql+psycopg://" + uri[len("postgresql://"):]
        if not uri.startswith("postgresql+psycopg://"):
            raise ValueError("DATABASE_URL harus berupa URL PostgreSQL untuk jalur managed.")
        return uri
    data_dir = Path(os.environ.get("M05_DATA_DIR", str(Path(__file__).parent / "data"))).resolve()
    data_dir.mkdir(parents=True, exist_ok=True)
    return "sqlite:///" + data_dir.joinpath("app.sqlite3").as_posix()


@lru_cache(maxsize=1)
def engine():
    uri = database_url()
    kw = {"pool_pre_ping": True, "pool_size": 1, "max_overflow": 1,
          "pool_timeout": 10, "pool_recycle": 1800}
    if uri.startswith("sqlite:"):
        kw["connect_args"] = {"timeout": 15}
    return create_engine(uri, **kw)


def initialize_schema():
    # Initial schema only, not an ALTER / evolving migration system.
    metadata.create_all(engine())


def health_db() -> bool:
    with engine().connect() as conn:
        conn.execute(text("SELECT 1")).scalar_one()
    return True


def list_tasks():
    with engine().connect() as conn:
        rows = conn.execute(select(tasks).order_by(tasks.c.id)).mappings().all()
    return [dict(row) for row in rows]


def add_task(title: str):
    title = title.strip()
    if not (1 <= len(title) <= 160):
        raise ValueError("Judul harus 1–160 karakter.")
    with engine().begin() as conn:
        conn.execute(insert(tasks).values(
            title=title, done=False, created_at=datetime.now(timezone.utc)
        ))


def set_task_done(task_id: int, value: bool):
    with engine().begin() as conn:
        result = conn.execute(update(tasks).where(tasks.c.id == task_id).values(done=value))
    return result.rowcount > 0


def remove_task(task_id: int):
    with engine().begin() as conn:
        result = conn.execute(delete(tasks).where(tasks.c.id == task_id))
    return result.rowcount > 0
```


### 18. Aplikasi `app.py`: pisahkan liveness, readiness, dan public read API

```python
"""Read-only public endpoints; all data mutation is via authenticated VPS SSH CLI."""
from datetime import datetime, timezone
import logging
import os

from flask import Flask, jsonify, render_template
from sqlalchemy.exc import SQLAlchemyError
from db import health_db, list_tasks

app = Flask(__name__)
app.config["MAX_CONTENT_LENGTH"] = 32 * 1024  # No public upload interface
app.logger.setLevel(logging.INFO)


@app.get("/")
def index():
    return render_template("index.html", app_env=os.getenv("APP_ENV", "development"))


@app.get("/health")
def health():
    """Liveness: the HTTP application process is responding."""
    return jsonify(status="ok", service="cc-m03", milestone="M05",
                   timestamp=datetime.now(timezone.utc).isoformat())


@app.get("/ready")
def ready():
    """Readiness: database network/login/SQL SELECT 1 works."""
    try:
        health_db()
    except (SQLAlchemyError, OSError, ValueError):
        app.logger.exception("database readiness check failed")
        return jsonify(status="unavailable"), 503
    return jsonify(status="ready"), 200


@app.get("/api/info")
def info():
    return jsonify(course="Cloud Computing", code="PTI 2802",
                   milestone="M05", privacy="synthetic-data-only")


@app.get("/api/tasks")
def api_tasks():
    try:
        rows = list_tasks()
        serialized = [{"id": r["id"], "title": r["title"], "done": bool(r["done"]),
                       "created_at": r["created_at"].isoformat()} for r in rows]
    except (SQLAlchemyError, OSError, ValueError):
        app.logger.exception("list tasks failed")
        return jsonify(error="data_service_unavailable"), 503
    return jsonify(count=len(serialized), items=serialized)
```


### 19. Halaman responsif sederhana: `templates/index.html`

```html
<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="Cloud Computing PTI 2802: Storage & Managed Data Services">
  <title>Cloud Computing — M05 Data Services</title>
  <link rel="stylesheet" href="/static/style.css">
</head>
<body>
<main class="shell">
  <span class="badge">PTI 2802 · M05</span>
  <h1>Storage &amp; Managed Data Services</h1>
  <p>Data persisten dari VPS atau PostgreSQL terkelola. API baca publik; perubahan data melalui CLI SSH yang terautentikasi.</p>
  <nav>
    <a href="/health">/health</a>
    <a href="/ready">/ready</a>
    <a href="/api/tasks">/api/tasks</a>
  </nav>
  <p id="status" role="status" aria-live="polite">Mengambil data…</p>
  <ul id="tasks"></ul>
  <p class="small">Gunakan hanya data latihan/sintetis. Jangan gunakan informasi pribadi nyata.</p>
</main>
<script>
(async () => {
  const list = document.getElementById('tasks');
  const status = document.getElementById('status');
  try {
    const r = await fetch('/api/tasks', {cache:'no-store'});
    if (!r.ok) throw new Error('HTTP '+r.status);
    const data = await r.json();
    status.textContent = data.count + ' tugas tersimpan';
    data.items.forEach(item => {
      const li = document.createElement('li');
      li.textContent = (item.done ? '✓ ' : '○ ') + item.title;
      list.appendChild(li);
    });
  } catch (_) { status.textContent = 'Layanan data tidak tersedia; periksa /ready.'; }
})();
</script>
</body>
</html>
```


### 20. CSS halaman proyek: `static/style.css`

```css
:root{font-family:system-ui,-apple-system,"Segoe UI",sans-serif;color-scheme:dark;color:#ecf3fc;background:#0a1626}
*{box-sizing:border-box}body{margin:0;min-height:100vh;display:grid;place-items:center;background:radial-gradient(ellipse at top right,#165780,transparent 60%),#081522}
.shell{width:min(780px,calc(100% - 2rem));padding:clamp(1.25rem,4vw,3rem);border:1px solid #47657d;border-radius:1.5rem;background:#10263c;box-shadow:0 15px 45px #0005}
h1{font-size:clamp(1.8rem,5vw,3rem);line-height:1.15}p{line-height:1.6;color:#c1d0e0}.badge{background:#1a658e;border-radius:999px;padding:.35rem .7rem;font-weight:700;font-size:.8rem}
nav{display:flex;flex-wrap:wrap;gap:.6rem}nav a{padding:.55rem .75rem;background:#17638e;color:white;text-decoration:none;border-radius:.5rem}nav a:hover,nav a:focus{background:#237db3}
li{padding:.5rem 0;border-bottom:1px solid #33526b;overflow-wrap:anywhere}.small{font-size:.8rem}
```


### 21. CLI data melalui SSH terautentikasi: `scripts/data_cli.py`

```python
"""Use inside SSH; public internet never receives write endpoints."""
import argparse
from db import initialize_schema, list_tasks, add_task, set_task_done, remove_task, database_url


def main():
    p = argparse.ArgumentParser(description="PTI 2802 — M05 data administration")
    subs = p.add_subparsers(dest="cmd", required=True)
    subs.add_parser("init", help="Create initial schema (not structural migration)")
    subs.add_parser("list")
    a = subs.add_parser("add"); a.add_argument("title")
    d = subs.add_parser("done"); d.add_argument("id", type=int)
    u = subs.add_parser("undone"); u.add_argument("id", type=int)
    r = subs.add_parser("remove"); r.add_argument("id", type=int)
    args = p.parse_args()
    if args.cmd == "init":
        initialize_schema(); print("Initial schema ready")
    elif args.cmd == "list":
        for row in list_tasks():
            print(f'{row["id"]}: [{"x" if row["done"] else " "}] {row["title"]}')
    elif args.cmd == "add":
        add_task(args.title); print("Created")
    elif args.cmd in ("done", "undone"):
        ok = set_task_done(args.id, args.cmd == "done")
        print("Updated" if ok else "No such ID")
    elif args.cmd == "remove":
        ok = remove_task(args.id); print("Removed" if ok else "No such ID")


if __name__ == "__main__":
    main()
```


### 22. Menghindari commit data/secret: `.gitignore` dan `.env.example`

```gitignore
.venv/
__pycache__/
*.py[cod]
.env
.env.*
!.env.example
data/
*.sqlite3
*.sqlite3-shm
*.sqlite3-wal
backups/
*.log
.DS_Store
Thumbs.db
```


### 22.1 Template konfigurasi (bukan credential nyata)

```dotenv
# Do NOT store real credentials here.
APP_ENV=development
M05_DATA_DIR=/var/lib/cc-m03
# DATABASE_URL='postgresql+psycopg://USER:REDACTED@HOST/DB?sslmode=require'
```


## 23. Integrasi dengan `systemd`: service tidak boleh menulis ke source

Pada M03 `cc-m03.service` menjalankan Gunicorn. M05 menambah **state directory** yang writable, tetapi mempertahankan jalur Caddy dan Gunicorn yang sudah ada. Lakukan audit dahulu:

```bash
sudo systemctl cat cc-m03
sudo cp -a /etc/systemd/system/cc-m03.service \
  /etc/systemd/system/cc-m03.service.pre-m05
```

Edit:

```bash
sudo nano /etc/systemd/system/cc-m03.service
```

Konfigurasi rekomendasi untuk proyek referensi; adaptasi jika unit M03 sudah mempunyai pengaturan tambahan yang benar:

```ini
[Unit]
Description=PTI 2802 M05 Cloud Project
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=cloudstudent
Group=cloudstudent
WorkingDirectory=/srv/cc-m03

Environment=PYTHONUNBUFFERED=1
Environment=M05_ENV_FILE=/etc/cc-m03/m05.env

ExecStart=/srv/cc-m03/.venv/bin/gunicorn --workers 1 --threads 2 --bind 127.0.0.1:8000 --timeout 30 --access-logfile - --error-logfile - app:app

Restart=on-failure
RestartSec=3

NoNewPrivileges=true
PrivateTmp=true
ProtectHome=true
ProtectSystem=strict
StateDirectory=cc-m03
StateDirectoryMode=0700
ReadWritePaths=/var/lib/cc-m03

[Install]
WantedBy=multi-user.target
```

Penjelasan security boundary:

- `User=cloudstudent`: proses aplikasi tidak berjalan sebagai root.
- `WorkingDirectory=/srv/cc-m03`: direktori source tetap sama seperti M03.
- `Environment=M05_ENV_FILE=...`: menunjukkan lokasi file rahasia eksternal; **tidak memuat password pada unit file**.
- `StateDirectory=cc-m03`: systemd menyiapkan lokasi state untuk service [11].
- `ProtectSystem=strict`: filesystem sistem dibatasi read-only untuk proses yang relevan, dengan pengecualian writable state.
- `ReadWritePaths=/var/lib/cc-m03`: menegaskan direktori tempat SQLite dapat menulis.
- `127.0.0.1:8000`: backend tidak dipublikasikan langsung [12].

**Jangan mengubah `/etc/caddy/Caddyfile` dari M04** bila `reverse_proxy 127.0.0.1:8000` sudah benar. HTTPS, domain, sertifikat, dan aturan firewall provider tetap dipakai sebagaimana M04.

Validasi sebelum restart:

```bash
sudo systemd-analyze verify /etc/systemd/system/cc-m03.service
sudo systemctl daemon-reload
```

Jika tidak ada error konfigurasi serius, lanjutkan inisialisasi skema di langkah berikutnya. Simpan session SSH aktif sebelum mengganti service.

### 23.1 Uji unit cepat sebelum mengubah service

Starter menyertakan pengujian `unittest` berbasis **database SQLite sementara**, sehingga tidak memodifikasi data VPS produksi. Setelah menyalin folder `tests/`, jalankan:

```bash
cd /srv/cc-m03
.venv/bin/python -m unittest discover -s tests -v
```

Unit test memeriksa init, CRUD, backup, dan validasi backup. Ini **bukan pengganti** uji HTTP Flask/Gunicorn/Caddy serta uji PostgreSQL managed pada provider yang sebenarnya.

## 24. Inisialisasi skema, pengujian CRUD, dan persistensi

Syarat: source dari starter M05 telah diintegrasikan dan dependency sudah tersedia.

```bash
cd /srv/cc-m03
.venv/bin/python -m scripts.data_cli init
.venv/bin/python -m scripts.data_cli add "Peminjaman perangkat laboratorium — contoh sintetis"
.venv/bin/python -m scripts.data_cli add "Persiapan demonstrasi cloud M05"
.venv/bin/python -m scripts.data_cli list
```

Contoh output (ID bergantung pada DB aktual):

```text
Initial schema ready
Created
Created
1: [ ] Peminjaman perangkat laboratorium — contoh sintetis
2: [ ] Persiapan demonstrasi cloud M05
```

Ubah satu record dan hapus satu record **hanya dari dataset simulasi**, bukan data nyata:

```bash
.venv/bin/python -m scripts.data_cli done 1
.venv/bin/python -m scripts.data_cli undone 1
.venv/bin/python -m scripts.data_cli done 1
.venv/bin/python -m scripts.data_cli remove 2
.venv/bin/python -m scripts.data_cli list
```

### 24.1 Periksa lokasi dan permission database

```bash
ls -lah /var/lib/cc-m03
stat -c '%A %U:%G %s %n' /var/lib/cc-m03/app.sqlite3
```

Perhatikan file `app.sqlite3` berada di luar repository. Jangan commit file `.sqlite3`, `-wal`, `-shm` ataupun `.env`.

### 24.2 Restart service dan uji bahwa data tidak hilang

```bash
sudo systemctl restart cc-m03
sudo systemctl status cc-m03 --no-pager
curl -fsS http://127.0.0.1:8000/health
curl -fsS http://127.0.0.1:8000/ready
curl -fsS http://127.0.0.1:8000/api/tasks
```

Kemudian:

```bash
.venv/bin/python -m scripts.data_cli list
```

Data yang telah ditulis **harus tetap ada**. Restart aplikasi bukan restore database; persistensi terjadi karena data berada di storage di luar memory proses. Diskusi: bagaimana hasilnya jika mahasiswa menyimpan tugas pada list Python global tanpa database?

## 25. Menguji tiga lapisan health

Ketiga pemeriksaan **berbeda makna**:

| Endpoint | Lapisan | Expected | Kesimpulan |
|---|---|---|---|
| `127.0.0.1:8000/health` | Flask/Gunicorn (liveness) | HTTP 200 | proses aplikasi dapat merespons |
| `127.0.0.1:8000/ready` | koneksi DB | HTTP 200 / 503 | query `SELECT 1` berhasil/gagal |
| `https://DOMAIN/api/tasks` | DNS → TLS → Caddy → Gunicorn → DB | HTTP 200 dengan JSON | alur aplikasi publik bekerja |

Lakukan di VPS:

```bash
curl -i http://127.0.0.1:8000/health
curl -i http://127.0.0.1:8000/ready
curl -i http://127.0.0.1:8000/api/tasks
curl -i http://127.0.0.1/api/tasks
```

Dari klien publik:

```text
https://DOMAIN/
https://DOMAIN/health
https://DOMAIN/ready
https://DOMAIN/api/tasks
```

Aplikasi tidak mempunyai endpoint tulis publik. Data hanya dapat diubah melalui CLI server yang memerlukan SSH sah. Strategi ini sengaja digunakan sebagai baseline pendidikan, **bukan klaim bahwa aplikasi sudah memiliki autentikasi pengguna lengkap**. Pada proyek individu dengan kebutuhan CRUD via browser, desain login/authorization, CSRF, validasi, dan rate limiting harus dipenuhi sebelum membuka mutasi ke internet.

## 26. Verifikasi khusus Windows, Linux, iPhone, dan Android

### 26.1 Windows PowerShell

```powershell
$Domain = "<DOMAIN>"
curl.exe -I "https://$Domain/"
curl.exe "https://$Domain/health"
curl.exe "https://$Domain/ready"
curl.exe "https://$Domain/api/tasks"
```

Jika ingin melihat JSON dengan PowerShell:

```powershell
Invoke-RestMethod -Uri "https://$Domain/api/tasks" -Method Get
```

`curl.exe` digunakan eksplisit agar tidak tertukar dengan alias PowerShell lama.

### 26.2 Linux client

```bash
DOMAIN="<DOMAIN>"
curl -fsS "https://$DOMAIN/health"
curl -fsS "https://$DOMAIN/ready"
curl -fsS "https://$DOMAIN/api/tasks"
```

Jika `jq` tersedia:

```bash
curl -fsS "https://$DOMAIN/api/tasks" | jq .
```

### 26.3 iPhone/iPad

1. Buka Safari; kunjungi `https://DOMAIN/`.
2. Buka `https://DOMAIN/api/tasks`; periksa JSON pada browser.
3. Masuk ke VPS melalui SSH menggunakan Termius atau klien yang setara.
4. Jalankan `.venv/bin/python -m scripts.data_cli add "Uji sintetis dari iPhone"` di `/srv/cc-m03`.
5. Refresh Safari; record baru harus muncul.
6. Lakukan `sudo systemctl restart cc-m03` dari sesi SSH.
7. Refresh Safari lagi; data harus tetap ada.
8. Simpan bukti yang disanitasi.

### 26.4 Android

1. Gunakan browser untuk membuka `https://DOMAIN/` dan `/api/tasks`.
2. Gunakan Termius (atau Termux + SSH) untuk masuk ke VPS.
3. Jalankan `.venv/bin/python -m scripts.data_cli add "Uji sintetis dari Android"`.
4. Lakukan `list`, lalu browser refresh.
5. Reboot/restart **service**, bukan VPS secara serampangan, dan ulangi pengecekan.
6. Simpan log/evidence; jangan mengirim secret ke aplikasi catatan publik.

Perangkat yang berbeda **tidak berarti empat database yang berbeda**. Seluruh perangkat berinteraksi dengan satu aplikasi di VPS dan backend data yang sama.

---

# BAGIAN IV — BACKUP ONLINE DAN UJI PEMULIHAN WAJIB

## 27. Mengapa `cp app.sqlite3 backup.sqlite3` tidak cukup?

Ketika SQLite sedang digunakan, file journal/WAL dan transaksi berjalan dapat memengaruhi konsistensi hasil copy. SQLite menjelaskan Online Backup API untuk salinan database yang konsisten tanpa mengharuskan database berhenti total selama proses backup [8], [9].

Starter mengimplementasikan `sqlite3.Connection.backup()` dan `PRAGMA integrity_check` lalu menyetel permission backup `0600`.

### 27.1 Kode `scripts/backup_sqlite.py`

#### Backup online

```python
"""Consistent online SQLite backup, never raw cp of a busy WAL DB."""
from datetime import datetime, timezone
from pathlib import Path
import os
import sqlite3
import sys
from contextlib import closing

from db import database_url


def main():
    uri = database_url()
    if not uri.startswith("sqlite:////"):
        raise SystemExit("Backup SQLite hanya berlaku pada jalur SQLite lokal.")
    source = Path(uri.removeprefix("sqlite:///"))
    if not source.exists():
        raise SystemExit("Database belum ada; jalankan init terlebih dahulu.")
    dest_dir = Path(sys.argv[1] if len(sys.argv) > 1 else "/var/backups/cc-m03")
    dest_dir.mkdir(parents=True, exist_ok=True, mode=0o700)
    dest = dest_dir / ("m05-" + datetime.now(timezone.utc).strftime("%Y%m%dT%H%M%SZ") + ".sqlite3")
    with closing(sqlite3.connect(source)) as src, closing(sqlite3.connect(dest)) as target:
        src.backup(target, pages=100)
        status = target.execute("PRAGMA integrity_check").fetchone()[0]
    os.chmod(dest, 0o600)
    if status != "ok":
        dest.unlink(missing_ok=True)
        raise SystemExit("Backup corrupt: " + status)
    print(f"Backup OK: {dest}")


if __name__ == "__main__":
    main()
```

#### Verifikasi integritas

```python
"""Read-only integrity check and row count on a backup artifact."""
import sqlite3
import sys
from pathlib import Path


def main():
    if len(sys.argv) != 2:
        raise SystemExit("Usage: python -m scripts.verify_sqlite_backup PATH")
    p = Path(sys.argv[1]).resolve(strict=True)
    con = sqlite3.connect(f"file:{p}?mode=ro", uri=True)
    try:
        result = con.execute("PRAGMA integrity_check").fetchone()[0]
        count = con.execute("SELECT COUNT(*) FROM tasks").fetchone()[0]
        if result != "ok":
            raise SystemExit("INVALID BACKUP: " + result)
        print(f"VALID BACKUP: integrity={result}; tasks={count}; file={p}")
    finally:
        con.close()


if __name__ == "__main__":
    main()
```


## 28. Jalankan backup yang konsisten

```bash
cd /srv/cc-m03
.venv/bin/python -m scripts.backup_sqlite /var/backups/cc-m03
```

Contoh output:

```text
Backup OK: /var/backups/cc-m03/m05-YYYYMMDDTHHMMSSZ.sqlite3
```

Periksa file tanpa mengekspos data:

```bash
ls -lh /var/backups/cc-m03/
stat -c '%A %U:%G %s %n' /var/backups/cc-m03/*.sqlite3
```

File backup mengandung data aplikasi. Perhatikan bahwa backup konsisten bukan jaminan backup bebas kesalahan logika aplikasi (misalnya data sudah salah sebelum dibackup).

## 29. Verifikasi integritas backup

Gunakan **nama file yang benar-benar dibuat**; jangan gunakan placeholder sebagai command:

```bash
BACKUP=$(find /var/backups/cc-m03 -maxdepth 1 -type f -name 'm05-*.sqlite3' \
  | sort | tail -n 1)
printf 'File backup terpilih: %s\n' "$BACKUP"
.venv/bin/python -m scripts.verify_sqlite_backup "$BACKUP"
```

Expected contoh:

```text
VALID BACKUP: integrity=ok; tasks=3; file=/var/backups/cc-m03/...
```

Catat jumlah record agar dapat dibandingkan saat restore. Jangan menganggap hasil `integrity_check=ok` saja telah membuktikan seluruh RPO/RTO; masih perlu menguji restore dan kesesuaian data.

## 30. Uji pemulihan **tanpa menimpa database aktif**

Simulasi restore dilakukan ke direktori sementara terpisah, bukan langsung ke `/var/lib/cc-m03/app.sqlite3` yang sedang dipakai server.

```bash
cd /srv/cc-m03
BACKUP=$(find /var/backups/cc-m03 -maxdepth 1 -type f -name 'm05-*.sqlite3' \
  | sort | tail -n 1)
RESTORE_DIR=$(mktemp -d)
chmod 700 "$RESTORE_DIR"
cp -- "$BACKUP" "$RESTORE_DIR/app.sqlite3"
chmod 600 "$RESTORE_DIR/app.sqlite3"
```

**Paksa pembacaan database hasil restore**, bukan konfigurasi PostgreSQL terkelola yang mungkin telah dipasang kemudian:

```bash
M05_DATA_DIR="$RESTORE_DIR" DATABASE_URL='' \
  .venv/bin/python -m scripts.data_cli list
```

Bukti pemulihan yang harus dicatat:

1. backup file ditemukan;
2. `integrity_check=ok`;
3. jumlah record pada hasil restore sesuai expected;
4. sample record sintetis terbaca;
5. database asli tetap berfungsi;
6. durasi pemulihan dicatat sebagai data praktikum (bukan SLA).

Setelah menyimpan evidence yang diperlukan:

```bash
rm -f -- "$RESTORE_DIR/app.sqlite3"
rmdir -- "$RESTORE_DIR"
```

Perintah `rm` **hanya boleh dijalankan setelah memverifikasi** bahwa `RESTORE_DIR` adalah direktori sementara hasil `mktemp -d`, bukan direktori produksi.

### 30.1 Pemulihan ketika database produksi rusak — konsep operasional

Prosedur yang benar bersifat perubahan terkontrol:

```text
1. Umumkan maintenance / kurangi write.
2. Stop service penulis.
3. Backup/snapshot kondisi rusak untuk forensik.
4. Verifikasi backup target sebelum dipakai.
5. Restore ke lokasi data yang benar.
6. Atur owner/permission.
7. Start service.
8. Periksa /ready dan /api/tasks.
9. Catat insiden, durasi, dan kehilangan data.
```

Modul **tidak** menyuruh mahasiswa menimpa live DB dalam praktikum awal karena tindakan tersebut berisiko menghilangkan data proyek.

## 31. Salinan off-server: syarat backup yang lebih masuk akal

Backup yang tetap berada pada disk 20 GB **bisa hilang bersama VPS**. Minimal buat salinan kedua ke lokasi terpisah.

### 31.1 Windows PowerShell (dijalankan pada PC Windows)

```powershell
New-Item -ItemType Directory -Force -Path "$HOME\Cloud-M05-Backups"
scp -i "$HOME\.ssh\pti2802_m03" `
  "cloudstudent@<PUBLIC_IP>:/var/backups/cc-m03/m05-YYYYMMDDTHHMMSSZ.sqlite3" `
  "$HOME\Cloud-M05-Backups\"
```

Ganti nama file dengan nama hasil backup aktual. Bila konfigurasi SSH memakai port nonstandar, gunakan `scp -P PORT` sesuai kondisi yang sudah diuji.

### 31.2 Linux client

```bash
mkdir -p ~/cloud-m05-backups
chmod 700 ~/cloud-m05-backups
scp -i ~/.ssh/pti2802_m03 \
  cloudstudent@<PUBLIC_IP>:/var/backups/cc-m03/m05-YYYYMMDDTHHMMSSZ.sqlite3 \
  ~/cloud-m05-backups/
```

### 31.3 iPhone/iPad

Gunakan klien SSH/SFTP **yang mendukung pengunduhan file**, atau layanan object storage yang disediakan dosen; dukungan UI dan ketersediaan fitur dapat berbeda antar aplikasi/plan. Masuk ke `/var/backups/cc-m03/`, unduh file yang telah diverifikasi, simpan pada Files/lokasi terenkripsi yang sesuai, dan jangan mengunggah backup nyata ke repository GitHub. Jika tidak tersedia fitur SFTP di perangkat, demonstrasi backup melalui terminal VPS tetap dilakukan, sedangkan salinan off-server dapat dilakukan dari perangkat lain yang disediakan laboratorium.

### 31.4 Android

Jika menggunakan Termux lokal:

```bash
mkdir -p ~/cloud-m05-backups
scp -i ~/.ssh/pti2802_m03 \
  cloudstudent@<PUBLIC_IP>:/var/backups/cc-m03/m05-YYYYMMDDTHHMMSSZ.sqlite3 \
  ~/cloud-m05-backups/
```

Alternatif: klien SFTP untuk Android yang mendukung pengunduhan dan penyimpanan aman. Jangan mengambil screenshot data pribadi dari isi backup.

### 31.5 Verifikasi hasil transfer

Di VPS:

```bash
sha256sum "$BACKUP"
```

Di PC Windows:

```powershell
Get-FileHash -Algorithm SHA256 "<PATH_BACKUP_LOKAL>"
```

Di Linux/Termux:

```bash
sha256sum <PATH_BACKUP_LOKAL>
```

Hash dua file **harus sama**. Hash membuktikan kesamaan byte hasil transfer, tetapi **bukan** bukti backup dapat dipulihkan — itu dibuktikan oleh restore drill.

### 31.6 Kebijakan retensi minimum

Untuk praktikum, gunakan jumlah salinan serta retensi yang proporsional, misalnya:

- satu backup baseline M05;
- satu backup setelah transaksi perubahan;
- satu salinan di luar VPS;
- verifikasi restore minimal satu kali;
- penghapusan backup latihan sesuai kebijakan kelas setelah evidence aman.

Backup mungkin mengandung data sensitif apabila mahasiswa salah menggunakan dataset nyata; karena itu gunakan **data sintetis** dan jangan mengunggah backup sebagai file publik.

---

# BAGIAN V — LABORATORIUM B: MANAGED POSTGRESQL (LANJUTAN TERARAH)

## 32. Tujuan dan batas praktikum managed

Jalur ini direkomendasikan bagi mahasiswa yang mendapat akun/kuota provider yang memenuhi syarat. **Managed data service benar-benar dijalankan provider**, bukan memasang server PostgreSQL pada VPS yang sama.

Model pilihan:

```mermaid
flowchart LR
    U["Windows / Linux / iPhone / Android"]
    C["Caddy HTTPS"]
    A["Flask + SQLAlchemy on VPS"]
    DB["Managed PostgreSQL\nProvider data plane"]
    U -->|HTTPS| C
    C -->|loopback| A
    A -->|PostgreSQL over TLS| DB
```

Neon dipakai sebagai **contoh penyedia PostgreSQL terkelola**, bukan syarat eksklusif mata kuliah. Penyedia managed PostgreSQL lain dapat digunakan jika menyediakan koneksi PostgreSQL kompatibel, alamat endpoint yang dapat dijangkau VPS, konfigurasi TLS yang sesuai, dan kebijakan biaya aman. Dokumentasi Neon menyediakan connection string standar beserta pilihan pooled/direct [6].

**Jangan menjanjikan free tier permanen.** Ketersediaan region, batas penggunaan, fitur backup, dan harga dapat berubah. Verifikasi halaman resmi provider sebelum membuat proyek, jangan memasukkan metode pembayaran berbayar tanpa izin, dan pilih layanan lain jika tidak sesuai. VPS 1 GB tidak perlu memasang daemon PostgreSQL tambahan. Dokumentasi resmi Ubuntu mencantumkan 1 GB RAM sebagai minimum untuk cloud image Ubuntu Server 24.04, bukan sebagai spesifikasi ideal untuk banyak service berat [15].

## 33. Managed DB: pembagian tanggung jawab

| Lapisan | Tanggung jawab utama provider | Tanggung jawab utama mahasiswa |
|---|---|---|
| Physical infrastructure | hardware, storage infrastructure | memilih layanan sesuai kebutuhan |
| Managed DB process | provisioning/operasi engine sesuai produk | memilih database dan menguji koneksi |
| DB credentials | layanan credential mechanism | membuat serta menjaga role/password |
| Database schema | tidak menentukan schema proyek | schema, constraints, migration |
| SQL queries | tidak menulis query aplikasi | SQL parameterized, indexing, validasi |
| Data governance | kontrol layanan sesuai produk | klasifikasi data, privacy, retention |
| Recovery | fasilitas sesuai plan | memastikan backup dan restore benar-benar dapat dipakai |
| Cost | mencatat billing sesuai provider | quota, budget, lifecycle dan cleanup |

## 34. Langkah terstruktur melalui browser (semua perangkat)

1. Buka situs resmi provider yang dipilih (contoh: **https://neon.com/**).
2. Buat/login akun menggunakan metode yang disediakan provider.
3. Pilih proyek baru dengan nama misalnya `pti2802-m05-<nim>`; jangan memuat password atau NIM sensitif dalam data publik.
4. Pilih region yang diizinkan dan masuk akal terhadap lokasi VPS, data residency, biaya, dan latency.
5. Pilih konfigurasi **gratis/sandbox** jika benar-benar tersedia dan sesuai batas layanan saat itu.
6. Catat nama database dan role, tetapi **jangan menyalin password ke README, GitHub, atau chat**.
7. Buka menu **Connect/Connection details**; peroleh connection string resmi dari dashboard.
8. Jika provider mendukung pooled dan direct, pilih sesuai kebutuhan; untuk demo koneksi sederhana, direct dapat digunakan, sementara pooled berguna untuk banyak koneksi paralel. Untuk operasi migration/admin tertentu, ikuti rekomendasi direct connection pada provider [6].
9. Periksa komponen URL: `postgresql://ROLE:PASSWORD@HOST/DATABASE?...` dan parameter TLS seperti `sslmode=require`.
10. Jangan membuat record DNS database sendiri ke IP VPS; DB terkelola menggunakan hostname dan jaringan provider.

**Praktikum ini tidak meminta port PostgreSQL dibuka pada UFW VPS.** Koneksi aplikasi ke DB terkelola umumnya merupakan koneksi outbound dari VPS ke provider pada port yang ditentukan provider (sering TCP/5432). Jangan menambahkan inbound `5432/tcp` pada UFW/provider firewall VPS. Prinsip firewall host dan least privilege selaras dengan dokumentasi Ubuntu [16], [17].

## 35. Menyimpan connection string **hanya di VPS**

Pada VPS melalui SSH:

```bash
sudo nano /etc/cc-m03/m05.env
```

Contoh bentuk yang **tidak berisi rahasia nyata**:

```dotenv
APP_ENV=development
M05_DATA_DIR=/var/lib/cc-m03
DATABASE_URL='postgresql+psycopg://ROLE:REDACTED@HOST/DATABASE?sslmode=require'
```

**Isi nilai `DATABASE_URL` dengan URL asli dari provider, hanya pada file eksternal ini.** Jika URL provider diawali `postgresql://`, kode starter M05 akan memilih dialect SQLAlchemy `postgresql+psycopg://`. Jangan mengubah karakter/encoding password secara manual. Jika connection string memuat query `&channel_binding=require` atau parameter keamanan lain dari provider, pertahankan sesuai dokumentasi.

Perhatikan:

- `sslmode=require` mengenkripsi transport menurut konfigurasi libpq, tetapi detail verifikasi identitas server/certificate dapat berbeda berdasarkan trust store dan parameter. Pada proyek sensitif/produksi, evaluasi kebijakan `verify-full` serta CA trust chain yang didukung provider, alih-alih menganggap semua pengaturan TLS setara.
- Hostname provider harus divalidasi dari dashboard resmi.
- Password dan token harus diputar/dicabut jika bocor.
- Batas koneksi/suspend/resume provider dapat menyebabkan koneksi idle terputus; `pool_pre_ping` menangani sebagian kasus sebelum request [5].

Atur ulang permission tanpa memperlihatkan isi file:

```bash
sudo chown root:cloudstudent /etc/cc-m03/m05.env
sudo chmod 0640 /etc/cc-m03/m05.env
stat -c '%A %U:%G %n' /etc/cc-m03/m05.env
```

## 36. Verifikasi kemampuan VPS mencapai managed DB

Gunakan nama hostname dari provider (bukan password). Langkah berikut **tidak perlu menampilkan connection string utuh**:

```bash
getent ahostsv4 <DB_HOSTNAME>
```

Jika ingin mengecek port dengan tool yang tersedia:

```bash
sudo apt install -y netcat-openbsd
nc -vz <DB_HOSTNAME> 5432
```

Jika TCP gagal, evaluasi DNS, kebijakan outbound provider VPS, firewall jaringan, endpoint, IPv4/IPv6, status managed database, dan quota. Sukses TCP **bukan bukti autentikasi database**; uji selanjutnya dilakukan dengan query `SELECT 1`.

## 37. Inisialisasi schema managed

Karena kode `db.py` membaca `DATABASE_URL` dari `/etc/cc-m03/m05.env`, command yang sama dengan jalur SQLite akan diarahkan ke PostgreSQL terkelola:

```bash
cd /srv/cc-m03
.venv/bin/python -m scripts.data_cli init
```

Perhatikan **jangan menjalankan `drop_all()`**; `create_all()` hanya membuat initial schema yang belum ada. Uji:

```bash
.venv/bin/python -c 'from db import engine; print("Backend:", engine().dialect.name)'
.venv/bin/python -m scripts.data_cli list
.venv/bin/python -m scripts.data_cli add "Transaksi sintetis pada PostgreSQL terkelola"
.venv/bin/python -m scripts.data_cli list
```

Restart Gunicorn dan uji:

```bash
sudo systemctl restart cc-m03
curl -i http://127.0.0.1:8000/ready
curl -i http://127.0.0.1:8000/api/tasks
```

Lalu dari Windows/Linux/iPhone/Android buka:

```text
https://DOMAIN/ready
https://DOMAIN/api/tasks
```

Hasil `ready` HTTP 200 membuktikan *query connectivity pada saat itu*, bukan jaminan seluruh fitur database atau seluruh transaksi akan selalu berhasil.

## 38. Catatan migrasi data: mengganti URL tidak memindahkan record

**Poin sangat penting:** mengubah `DATABASE_URL` dari SQLite ke PostgreSQL **tidak memindahkan data lama secara otomatis**. Mahasiswa harus memilih salah satu:

- **Jalur demonstrasi kosong:** biarkan database PostgreSQL baru kosong lalu buat record sintetis baru. Ini cukup untuk demonstrasi managed DB.
- **Jalur transfer data (pengayaan):** ekspor record SQLite, lalu impor ke PostgreSQL terkelola melalui transaksi terkontrol; uji jumlah record dan data. Starter menyediakan script khusus untuk tabel demo `tasks`.

Sebelum mencoba transfer data, jalankan backup dan uji restore SQLite. Kemudian pastikan destination PostgreSQL masih **kosong**, karena script menolak menimpa/menduplikasi destination yang telah berisi task.

### 38.1 Script transfer M05 (hanya untuk entitas contoh `tasks`)

```python
"""M05-only one-time copy: SQLite to an EMPTY managed PostgreSQL tasks table.

This script does not preserve primary key values or support relational dependencies.
Do not use as a generalized production migration tool.
"""
from pathlib import Path
import os
import sqlite3
import sys

from sqlalchemy import insert, select, func
from db import tasks, metadata, engine, database_url


def main():
    if len(sys.argv) != 2:
        raise SystemExit("Usage: python -m scripts.migrate_sqlite_to_postgres /var/lib/cc-m03/app.sqlite3")
    if not database_url().startswith("postgresql+psycopg://"):
        raise SystemExit("Set secure DATABASE_URL to managed PostgreSQL before running.")
    src_path = Path(sys.argv[1]).resolve(strict=True)
    conn = sqlite3.connect(f"file:{src_path}?mode=ro", uri=True)
    try:
        result = conn.execute("PRAGMA integrity_check").fetchone()[0]
        if result != "ok":
            raise SystemExit("Source DB fails integrity check")
        rows = conn.execute("SELECT title, done, created_at FROM tasks ORDER BY id").fetchall()
    finally:
        conn.close()
    eng = engine()
    metadata.create_all(eng)
    with eng.begin() as pg:
        if pg.scalar(select(func.count()).select_from(tasks)):
            raise SystemExit("Destination has tasks; refuse to duplicate or overwrite.")
        for title, done, created_at in rows:
            from datetime import datetime
            dt = datetime.fromisoformat(created_at)
            pg.execute(insert(tasks).values(title=title, done=bool(done), created_at=dt))
    print(f"Copied {len(rows)} tasks to an empty managed PostgreSQL table.")


if __name__ == "__main__":
    main()
```


### 38.2 Langkah menjalankan transfer data

**Sebelum jalur B mengisi managed DB**, dan setelah backup SQLite lulus integritas:

```bash
cd /srv/cc-m03
.venv/bin/python -m scripts.migrate_sqlite_to_postgres \
  /var/lib/cc-m03/app.sqlite3
```

Lalu verifikasi:

```bash
.venv/bin/python -m scripts.data_cli list
curl -fsS http://127.0.0.1:8000/ready
```

Jika Anda sudah menjalankan langkah 37 dan memasukkan data ke PostgreSQL, gunakan database/branch managed baru yang masih kosong untuk transfer ini; **jangan menghapus data destination tanpa review**. Script ini sengaja dibatasi pada satu tabel pembelajaran dan **tidak mempertahankan ID asli**. Jangan menggunakannya untuk migrasi produksi dengan foreign key/dependensi eksternal. Jika source dan destination telah memiliki data, jangan menggabungkannya sembarangan; lakukan rencana migration yang ditinjau dosen.

## 39. Managed DB: backup, restorability, dan biaya

Managed service dapat menawarkan backup, retention, PITR, atau database branching, tetapi kemampuan serta durasi historinya **bergantung pada paket yang digunakan**. Verifikasi fitur aktual pada console resmi sebelum mengklaim sistem memiliki RPO tertentu. Neon telah memperbarui kapasitas Free plan pada Oktober 2026, yang menunjukkan mengapa angka kuota dalam modul harus dinilai berdasarkan tanggal, bukan dianggap permanen [13].

Praktikum pengayaan:

1. Periksa pada dashboard provider apakah backup/restore/PITR tersedia untuk akun yang digunakan.
2. Catat retention dan batas fitur, tanpa menyalin password.
3. Buat beberapa record sintetis di database pengujian.
4. Gunakan **branch/database uji terpisah** bila provider menyediakan restore target aman; jangan menimpa database utama.
5. Verifikasi data hasil restore dan dokumentasikan evidence.
6. Jika fitur tidak tersedia pada free plan, tulis **“tidak tersedia pada paket saat ini”**; jangan membuat klaim hasil restore palsu.

Untuk PostgreSQL, `pg_dump` dapat membuat ekspor logis database yang konsisten, tetapi **bukan pengganti tunggal strategi backup produksi** dan harus menggunakan client version kompatibel dengan server [14]. Jangan menempelkan connection URL berpassword langsung pada perintah yang terekam shell history atau terlihat dalam process list. Gunakan metode autentikasi aman serta credential store sesuai panduan provider apabila melakukan tugas lanjutan ini.

### 39.1 Checklist biaya dan lifecycle managed DB

- [ ] Paket/free quota benar-benar tersedia pada akun saat praktikum.
- [ ] Region/latency/pembatasan akses sesuai kebijakan kelas.
- [ ] Jumlah proyek/database/compute tidak melebihi batas.
- [ ] Tidak ada upgrade tier berbayar tanpa persetujuan.
- [ ] Credential rahasia tidak tersebar.
- [ ] Managed DB dihentikan/dihapus setelah proyek selesai menurut kebijakan dosen.
- [ ] Data latihan dihapus ketika retention tidak diperlukan.

---

# BAGIAN VI — TROUBLESHOOTING, OBSERVABILITAS, DAN KONTROL KUALITAS

## 40. Model diagnosis berlapis

```mermaid
flowchart TD
    X["Gejala: data tidak muncul / aplikasi error"]
    N{"Dapat resolve DNS + HTTPS?"}
    C{"Caddy merespons?"}
    A{"Gunicorn /health = 200?"}
    D{"/ready = 200?"}
    Q{"Query /api/tasks berhasil?"}
    R["Periksa DNS/TLS/provider firewall"]
    H["Periksa Caddy, listener, UFW"]
    P["Periksa systemd, app imports, Python packages"]
    B["Periksa DB path/credentials/TLS/network"]
    S["Periksa schema/query/serialization/data"]
    OK["Sistem sehat; uji data persisten/restore"]
    X --> N
    N -->|Tidak| R
    N -->|Ya| C
    C -->|Tidak| H
    C -->|Ya| A
    A -->|Tidak| P
    A -->|Ya| D
    D -->|Tidak| B
    D -->|Ya| Q
    Q -->|Tidak| S
    Q -->|Ya| OK
```

### 40.1 Matriks gejala → dugaan → verifikasi

| Gejala | Dugaan | Pemeriksaan pertama |
|---|---|---|
| `/health` 200 tetapi `/ready` 503 | DB tidak tersedia/credential salah | `journalctl -u cc-m03`; tes DB |
| `/ready` 200 tetapi `/api/tasks` 503 | tabel belum dibuat, query/serialization error | CLI `init`, log aplikasi |
| aplikasi 502 | Gunicorn mati atau gagal start | `systemctl status cc-m03` |
| halaman HTML tampil tetapi JSON gagal | rute/DB/data layer | `curl -i .../api/tasks` |
| SQLite `unable to open database file` | permission/path/state directory | `ls -ld /var/lib/cc-m03`; user service |
| SQLite `database is locked` | persaingan write atau transaksi terlalu lama | pool/worker/transaksi log |
| data hilang setelah restart | data tersimpan dalam RAM atau path tidak persisten | periksa `M05_DATA_DIR`, file SQLite |
| managed DB name resolution gagal | DNS/outbound/hostname salah | `getent ahostsv4 <HOST>` |
| managed DB connection timeout | service suspended, network, outbound port | `nc -vz <HOST> 5432`; status provider |
| managed DB authentication failed | role/password/connection string | dashboard provider; rotate secret bila perlu |
| TLS error | CA/trust/SSL parameter/tanggal OS | konfigurasi provider & waktu server |
| backup file ada, restore gagal | backup tidak valid/skema tidak cocok | integrity check, restore drill |
| disk hampir penuh | backup/log/cache menumpuk | `df -h`, `du`, `journalctl --disk-usage` |
| RAM tinggi | worker, package, DB lokal berlebih | `free -h`, `ps aux --sort=-%mem` |

## 41. Command troubleshooting VPS

### Status aplikasi dan reverse proxy

```bash
sudo systemctl status cc-m03 --no-pager
sudo systemctl status caddy --no-pager
sudo journalctl -u cc-m03 -n 80 --no-pager
sudo journalctl -u caddy -n 80 --no-pager
```

### Listening ports dan jaringan

```bash
sudo ss -lntp
sudo ufw status verbose
curl -i http://127.0.0.1:8000/health
curl -i http://127.0.0.1:8000/ready
curl -i http://127.0.0.1:8000/api/tasks
```

### Resource usage

```bash
free -h
df -h /
df -i /
ps aux --sort=-%mem | head -n 12
journalctl --disk-usage
```

Jangan secara refleks menjalankan `apt autoremove`, menghapus `/var/log`, `rm -rf`, atau mematikan service tanpa mengetahui dampak; tindakan pemulihan harus berbasis evidence.

### Log privacy

Log yang dibagikan di GitHub harus disanitasi. Hindari output konfigurasi full `DATABASE_URL`, query yang memuat data pribadi, header otorisasi, private key, dan isi file `/etc/cc-m03/m05.env`.

## 42. Menguji kegagalan database secara aman

Untuk menguji perbedaan liveness vs readiness **tanpa menonaktifkan produksi**:

1. Jalankan aplikasi versi pengujian pada environment/sandbox terpisah dengan `DATABASE_URL` yang tidak valid **hanya di proses uji**.
2. Pastikan `GET /health` tetap bisa merespons jika aplikasi berhasil start.
3. Periksa `GET /ready` memberi status 503 dan tidak membocorkan credential.
4. Periksa log internal secara terbatas.
5. Jangan sengaja mengubah password database produksi atau merusak routing server yang sedang dipakai bersama.

Jika tidak tersedia environment terpisah, gunakan **analisis skenario** dan log simulatif sebagai tugas; tidak perlu menimbulkan outage nyata.

## 43. Bukti persistensi dan reliability

Mahasiswa wajib membuktikan:

```text
INSERT via SSH
     ↓
GET /api/tasks shows record
     ↓
systemctl restart cc-m03
     ↓
GET /api/tasks still shows record
     ↓
SQLite online backup
     ↓
integrity check = ok
     ↓
restore to separate directory
     ↓
restored CLI SELECT shows expected records
```

Pada managed database, ganti bagian backup/restore dengan fasilitas provider yang benar-benar tersedia atau prosedur backup yang disetujui; jika tidak berhasil diuji, nyatakan keterbatasannya secara eksplisit.

---

# BAGIAN VII — REPOSITORY, DOKUMENTASI, EVIDENCE, DAN MILESTONE M05

## 44. Revisi architecture v0.3

Buat/perbarui `docs/architecture.md` dengan perubahan storage dan data boundary.

```mermaid
flowchart TB
    C["Client: Windows/Linux/iPhone/Android"]
    D["DNS + HTTPS / Caddy"]
    A["Gunicorn + Flask"]
    L["SQLAlchemy Core"]
    S["SQLite on VPS disk"]
    P["Managed PostgreSQL (alternative)"]
    BK["SQLite backup / external copy"]
    C --> D --> A --> L
    L -->|Local mode| S
    L -->|Managed mode over TLS| P
    S --> BK
```

Dokumentasikan apa yang berubah sejak M03/M04: sebelumnya tidak ada persistence; sekarang terdapat backend data, config/secret boundary, lokasi data, connection lifecycle, backup, dan recovery procedure.

## 45. ADR-0002 — Storage and Managed Data Decision

Mahasiswa wajib menyimpan ADR yang menjawab:

1. Kebutuhan data sebenarnya apa?
2. Mengapa memilih SQLite atau managed PostgreSQL?
3. Bagaimana keterbatasan VPS 1 GB memengaruhi keputusan?
4. Bagaimana schema dan constraint dibuat?
5. Bagaimana credential/permission dipisahkan dari source?
6. Bagaimana backup/restore diuji?
7. Berapa perkiraan biaya dan ketergantungan provider?
8. Kondisi apa yang memicu revisi keputusan pada M06/M07?

Contoh lengkap tersedia dalam starter `docs/adr/0002-data-persistence.md`.

## 46. Format dokumentasi data M05

Buat `docs/data-design.md`:

```markdown
# M05 Data Architecture and Recovery

## Business Entity
Contoh: tasks / perangkat / inventaris / kegiatan.

## Data Model
Tabel atribut, primary key, nullability, constraints.

## Storage Decision
SQLite lokal atau PostgreSQL managed, beserta alasannya.

## Data Flow
Client → HTTPS → API baca → Data layer → Database.

## Access and Security
SSH-only mutation; no anonymous write; data sintetis.

## Secret Management
File eksternal /etc/cc-m03/m05.env; never in Git.

## Persistence Test
Before/after service restart.

## Backup Plan
Source, destination, schedule, retention, off-server copy.

## Restore Test
Path, method, expected count, actual count, duration.

## Known Limitations
No public CRUD auth; single-host SQLite; current risks.

## Roadmap
M06 container volume; M07 IaC; M09 migrations; M12 recovery.
```

## 47. Evidence yang wajib diserahkan

Usulan folder:

```text
docs/evidence/m05/
├── 01-preflight.md
├── 02-storage-comparison.md
├── 03-schema.md
├── 04-cli-crud-sanitized.txt
├── 05-api-read-sanitized.json
├── 06-persistence-restart.md
├── 07-backup-integrity.md
├── 08-restore-test.md
├── 09-managed-db.md             # atau 'not attempted + rationale'
├── 10-resource-usage.md
└── 11-reflection.md
```

Jangan commit file SQLite berisi data, file backup, real `DATABASE_URL`, `.env`, private key, atau dump PostgreSQL dengan data sensitif. Untuk screenshot berikan penjelasan command/hasil tanpa credential. Bukti waktu dihitung dari aktivitas nyata, bukan angka rekaan.

## 48. Git workflow dan tag M05

Jika menggunakan workflow Windows/Linux, lakukan perubahan di checkout lokal, `git push` ke branch, lalu deploy dengan `git pull` pada VPS. Jika hanya memakai smartphone, lakukan pengembangan dengan GitHub web editor/SSH sesuai M03 lalu commit–push; dapat juga meminta peer review perubahan melalui Pull Request.

Di VPS:

```bash
cd /srv/cc-m03
git status --short
git diff --check
.venv/bin/python -m pip check
```

Periksa semua perubahan dan pastikan data/secret tidak ikut terbawa:

```bash
git status
git diff --stat
```

Karena `/var/lib/cc-m03` berada **di luar repository**, verifikasi melalui `git status`, review staging dan daftar tracked files. Gunakan juga:

```bash
git ls-files | grep -E '(\.env$|\.sqlite3$|\.sqlite3-wal$|\.sqlite3-shm$|private_key|id_ed25519)' \
  && echo 'PERIKSA: ada file mencurigakan' \
  || echo 'Tidak ditemukan pola file rahasia umum'
```

Commit bertahap menurut unit perubahan:

```bash
git add app.py db.py templates/ static/ scripts/ requirements.txt .gitignore .env.example
git commit -m "feat: add M05 persistent data layer"

git add docs/ README.md
git commit -m "docs: document M05 storage design and recovery evidence"
```

Push branch ke GitHub:

```bash
git push -u origin feat/m05-persistence
```

Setelah review dan merge sesuai kebijakan dosen, buat tag dari commit final milestone di `main`:

```bash
git switch main
git pull --ff-only
# Pastikan merge dari branch M05 sudah selesai dan semua evidence aman.
git status
git tag -a m05 -m "M05: persistent data and verified recovery"
git push origin m05
```

Buat GitHub Release `M05 — Storage and Managed Data Services`, cantumkan penjelasan batasan. **Jangan memberi label managed database pada proyek yang hanya memakai SQLite lokal.**

## 49. Nilai akademik dari project-specific adaptation

Reference implementation memakai entitas `tasks`. Untuk proyek individu, **adaptasi paling tidak satu entitas** sesuai problem statement. Contoh:

| Proyek | Entitas | Atribut yang perlu dipertimbangkan |
|---|---|---|
| Inventaris laboratorium | asset | id, kode, nama, kondisi |
| Bank soal | question | id, indikator, teks, jenis |
| Jadwal konsultasi | appointment | id, tanggal, pemohon, status |
| Peminjaman buku | loan | id, buku, tanggal, pengembalian |
| Pencatatan lingkungan | measurement | id, parameter, nilai, waktu |

Mahasiswa wajib menambah data dictionary dan memutuskan constraint. Tidak diperlukan data asli siswa, NIM yang mengidentifikasi orang, nilai pribadi, atau informasi kesehatan dalam dataset praktikum.

---

# BAGIAN VIII — RENCANA PEMBELAJARAN, ASESMEN, DAN EVALUASI

## 50. Alokasi kegiatan

Karena perkuliahan 2 SKS dan sebagian aktivitas membutuhkan eksplorasi individual, tugas dibagi menjadi pra-kelas, tatap muka/praktikum terbimbing, serta kerja terstruktur mandiri. Durasi sinkron konkret mengikuti kalender dan pengaturan akademik institusi.

| Fase | Aktivitas | Output |
|---|---|---|
| Pra-kelas | review M03/M04, cek SSH/HTTPS, telaah storage models | preflight evidence |
| Microlecture | block vs file vs object, persistence, database modes | peta konsep |
| Guided lab A | integrasi schema, CRUD SSH, API baca | aplikasi persisten |
| Guided lab B | restart, liveness/readiness, failure reasoning | evidence health |
| Individual lab | backup online, integrity test, restore drill | bukti recovery |
| Extension lab | managed PostgreSQL dan data migration terkendali | evidence jika tersedia |
| Post-lab | ADR-0002, README, architecture, GitHub release | milestone M05 |

Jika keterbatasan waktu, jalur A + backup/restore **lebih diprioritaskan** dibanding membuat akun managed DB tergesa-gesa tanpa pengamanan. Jalur managed dapat diteruskan sebagai tugas terstruktur.

## 51. Rubrik penilaian M05 (100 poin)

| Dimensi | Bobot | Bukti yang dinilai |
|---|---:|---|
| Analisis storage & pemilihan backend | 10 | block/file/object dan keputusan beralasan |
| Desain schema & constraint | 10 | data dictionary, primary key, validasi |
| Implementasi persistence | 20 | CRUD benar, data bertahan restart |
| Keamanan & secret hygiene | 15 | tidak ada API tulis anonim/secret Git; permission benar |
| Backup dan restore **teruji** | 20 | online backup, integrity, restore drill |
| Kemampuan diagnosis data service | 10 | `/health` vs `/ready`, log, troubleshooting |
| Git, ADR, arsitektur, dokumentasi | 10 | commit, ADR-0002, evidence, tag `m05` |
| Technical defense individual | 5 | mampu menjelaskan keputusan dan hasil uji |
| **Total** | **100** | |

Penggunaan managed PostgreSQL tidak otomatis mendapat nilai lebih tinggi daripada SQLite. Keberhasilan dinilai dari **kesesuaian keputusan dengan kebutuhan, keamanan, evidence, dan penguasaan individu**, sehingga tidak diskriminatif terhadap mahasiswa yang tidak memiliki layanan berbayar.

### 51.1 Deskriptor tingkat pencapaian

| Level | Ciri kinerja |
|---|---|
| 4 — Sangat Baik | aplikasi dan data benar; backup/restore lulus; keputusan beralasan; troubleshooting dan repositori teruji |
| 3 — Baik | aplikasi persisten dan evidence mayoritas lengkap; kekurangan minor pada dokumentasi atau justifikasi |
| 2 — Cukup | persistence berjalan tetapi backup/restore belum meyakinkan atau security perlu penguatan |
| 1 — Perlu Remediasi | data hilang, rahasia terbuka, aplikasi rusak, atau mahasiswa tidak dapat menjelaskan mekanisme |
| 0 — Tidak Ada Evidence | tugas tidak ada/tidak dapat diverifikasi |

## 52. Pertanyaan diagnostik/tes konsep

1. Mengapa data pada variabel Python tidak setara dengan data yang disimpan pada DB?
2. Jelaskan perbedaan block, file, dan object storage untuk tiga use case nyata.
3. Apa perbedaan database pada VPS sendiri dan database managed?
4. Mengapa SQLite cocok untuk baseline 1 vCPU/1 GB, tetapi tidak selalu cocok untuk write concurrency tinggi?
5. Mengapa connection string tidak boleh berada di repository GitHub?
6. Apa perbedaan primary key, unique constraint, dan index?
7. Apa konsekuensi SQL injection dan bagaimana parameter binding mencegahnya?
8. Mengapa `create_all()` tidak dapat dijadikan strategi perubahan schema jangka panjang?
9. Apa perbedaan `/health`, `/ready`, dan `/api/tasks`?
10. Mengapa `cp` file SQLite yang aktif tidak direkomendasikan sebagai satu-satunya prosedur backup?
11. Apa yang dibuktikan oleh `PRAGMA integrity_check`, dan apa yang **belum** dibuktikannya?
12. Mengapa backup yang hanya disimpan pada disk VPS belum cukup?
13. Apa yang terjadi jika mengganti `DATABASE_URL` ke PostgreSQL tanpa migrasi data?
14. Mengapa port 5432 VPS tidak perlu dibuka saat memakai remote managed PostgreSQL?
15. Jelaskan risiko jika aplikasi mengekspos HTTP POST/DELETE publik tanpa authentication dan authorization.
16. Apa risiko ketergantungan vendor DB dan bagaimana mengurangi vendor lock-in?
17. Apa hubungan RPO/RTO dengan jadwal backup dan strategi restore?
18. Mengapa database managed pun masih membutuhkan pengujian restore oleh konsumen?
19. Apa yang seharusnya dilakukan bila paket provider tidak menyediakan fitur PITR?
20. Bagaimana Anda menentukan bahwa proyek individu membutuhkan database relasional dibanding object storage?

## 53. Latihan kasus dan penalaran

### Kasus A — Data hilang setelah deployment

Sebuah aplikasi menyimpan daftar kegiatan dalam list Python global. Setelah `systemctl restart`, daftar kosong. Mahasiswa wajib menjelaskan akar masalah, mengusulkan model data, dan membuktikan perbaikannya melalui persistence test.

### Kasus B — “Managed” yang bukan managed

Mahasiswa memasang PostgreSQL sendiri pada VM yang sama dan menyebutnya managed DB. Mahasiswa lain memakai PostgreSQL yang dioperasikan provider. Bandingkan ownership patch, maintenance, backup, cost, risiko RAM, dan arsitektur jaringan.

### Kasus C — Backup palsu

Mahasiswa mengunggah `app.sqlite3` secara manual ke GitHub ketika database aktif dan mengklaim sistem sudah aman. Temukan kesalahan metodologis, keamanan, serta langkah pembuktian yang seharusnya dilakukan.

### Kasus D — Managed database unavailable

Setelah server managed suspend, `/health=200` tetapi `/ready=503`. Mahasiswa harus membuktikan bahwa HTTP process tetap hidup, mengisolasi masalah data service, serta menghindari membocorkan credential ke log publik.

### Kasus E — Storage VPS 20 GB hampir habis

Disk terisi backup, log, dan dependency. Mahasiswa harus menyusun langkah diagnosis (`df`, `du`, `journalctl`) dan strategi retention/off-server tanpa menghapus file sistem secara acak.

## 54. Technical defense dosen

Dosen memilih 4–6 tindakan langsung secara acak:

- Jalankan CLI dan tambahkan satu record sintetis.
- Buktikan record bertahan setelah restart `cc-m03`.
- Tunjukkan disk/data directory tanpa memperlihatkan credential.
- Buktikan `/health` berbeda dari `/ready`.
- Tunjukkan skema tabel dan satu constraint.
- Jalankan SQLite backup dan integrity verification.
- Pulihkan ke direktori sementara dan bandingkan row count.
- Tunjukkan `git status` bahwa DB/backup/secret tidak di-commit.
- Jelaskan perbedaan managed DB dan self-managed DB.
- Jelaskan kegagalan jika credential managed DB salah.

Evaluasi didasarkan pada demonstrasi nyata, bukan semata jumlah screenshot/commit.

## 55. Refleksi individu

Mahasiswa menuliskan maksimal satu halaman tentang:

- keputusan storage yang diambil dan alasan teknisnya;
- dua kendala nyata yang dialami;
- evidence pemecahan masalah;
- kekhawatiran terbesar terkait kehilangan data;
- bagaimana desain M05 menyiapkan volume/container pada M06;
- bagian mana yang akan diubah jika aplikasi melayani lebih banyak pengguna.

## 56. Persiapan Pertemuan 6 — Containerization

Pertemuan berikutnya akan mengemas aplikasi ke container. Diskusikan sejak M05:

```text
Application container lifecycle != persistent data lifecycle
```

Rencana M06:

1. kode aplikasi dibungkus container image;
2. data SQLite lokal tidak ikut tertanam di image;
3. volume/bind mount didefinisikan dengan owner dan permission sesuai;
4. config/secret di-inject pada runtime, tidak di-build ke image;
5. alternatif DB managed tetap berada di luar container;
6. deployment dan rollback tidak menghapus database tanpa rencana.

---

# BAGIAN IX — LEMBAR FINAL M05 DAN STANDAR PUBLIKASI

## 57. Checklist final M05

### A. Kontinuitas proyek

- [ ] Repository GitHub sama dengan M03/M04.
- [ ] Source Flask, Gunicorn, systemd, Caddy, HTTPS hasil M04 masih konsisten.
- [ ] Tidak ada pembuatan VPS baru tanpa alasan.

### B. Storage/data

- [ ] Memahami block/file/object storage.
- [ ] Menetapkan entitas proyek dan constraint yang sesuai.
- [ ] Database berada di lokasi persisten di luar source.
- [ ] Operasi CRUD yang sah berjalan.
- [ ] Tidak ada unauthenticated public write endpoint.
- [ ] Data tetap ada setelah service restart.

### C. Security

- [ ] File credential eksternal bukan bagian Git.
- [ ] Permission `root:cloudstudent 0640` untuk secret.
- [ ] Backend tetap bind `127.0.0.1:8000`.
- [ ] Tidak membuka inbound port database di VPS.
- [ ] Data praktikum menggunakan dataset sintetis.

### D. Backup dan recoverability

- [ ] Online backup sukses.
- [ ] Integrity check sukses.
- [ ] Restore drill ke path terpisah sukses.
- [ ] Bukti off-VPS copy atau prosedur yang terverifikasi tersedia.
- [ ] Menjelaskan RPO/RTO tanpa membuat klaim tidak teruji.

### E. Managed DB (bila mengikuti jalur B)

- [ ] Provider dan paket dipilih secara sadar.
- [ ] Connection string aman.
- [ ] Koneksi PostgreSQL berhasil.
- [ ] Schema dan query berhasil.
- [ ] Dapat menjelaskan bahwa data SQLite tidak otomatis berpindah.
- [ ] Backup/restore provider diuji bila benar-benar tersedia, atau keterbatasannya dinyatakan.

### F. GitHub dan asesmen

- [ ] Architecture v0.3 diperbarui.
- [ ] ADR-0002 tersedia.
- [ ] README dan deployment procedure diperbarui.
- [ ] Evidence disanitasi.
- [ ] Tag/release `m05` dibuat pada commit final.
- [ ] Mahasiswa siap menjelaskan hasilnya secara individual.

## 58. Catatan akademik dan batas keabsahan

Modul memakai reference implementation pendidikan yang telah diperiksa secara sintaksis dan diuji pada jalur database SQLite. Tidak diklaim bahwa seluruh variasi provider, OS klien, paket free tier, konfigurasi TLS managed, dan proyek individu telah diuji end-to-end pada layanan eksternal nyata. Dosen harus melakukan *pre-lab smoke test* untuk provider dan resource laboratorium sebelum mewajibkan layanan tertentu.

Pola Flask/Gunicorn/SQLAlchemy dipilih sebagai media pembelajaran, bukan rekomendasi otomatis untuk semua sistem produksi. Penggunaan nyata yang melayani data sensitif memerlukan autentikasi aplikasi, authorization, audit, privacy review, observability, backup retention, vulnerability management, migration governance, dan evaluasi kapasitas yang lebih lengkap.

---

# REFERENCES (IEEE)

[1] P. Mell and T. Grance, *The NIST Definition of Cloud Computing*, NIST Special Publication 800-145. Gaithersburg, MD, USA: National Institute of Standards and Technology, Sep. 2011, doi: 10.6028/NIST.SP.800-145. [Online]. Available: https://doi.org/10.6028/NIST.SP.800-145

[2] Amazon Web Services, “Block vs. File vs. Object Storage,” *AWS Cloud Storage Comparison*. [Online]. Available: https://aws.amazon.com/compare/the-difference-between-block-file-object-storage/. [Accessed: Oct. 16, 2026].

[3] Amazon Web Services, “When to Choose Amazon EFS,” *Amazon EFS*. [Online]. Available: https://aws.amazon.com/efs/when-to-choose-efs/. [Accessed: Oct. 16, 2026].

[4] E. Simmon, *Evaluation of Cloud Computing Services Based on NIST SP 800-145*, NIST Special Publication 500-322. Gaithersburg, MD, USA: National Institute of Standards and Technology, Feb. 2018, doi: 10.6028/NIST.SP.500-322.

[5] SQLAlchemy Project, “Connection Pooling,” *SQLAlchemy 2.0 Documentation*. [Online]. Available: https://docs.sqlalchemy.org/en/20/core/pooling.html. [Accessed: Oct. 16, 2026].

[6] Neon, “Connecting Neon to Your Stack,” *Neon Documentation*. [Online]. Available: https://neon.com/docs/get-started/connect-neon. [Accessed: Oct. 16, 2026].

[7] SQLAlchemy Project, “Describing Databases with MetaData,” *SQLAlchemy 2.0 Documentation*. [Online]. Available: https://docs.sqlalchemy.org/en/20/core/metadata.html. [Accessed: Oct. 16, 2026].

[8] SQLite Project, “SQLite Backup API,” *SQLite Documentation*. [Online]. Available: https://sqlite.org/backup.html. [Accessed: Oct. 16, 2026].

[9] Python Software Foundation, “sqlite3 — DB-API 2.0 Interface for SQLite Databases,” *Python 3.12 Documentation*. [Online]. Available: https://docs.python.org/3.12/library/sqlite3.html. [Accessed: Oct. 16, 2026].

[10] SQLAlchemy Project, “SQLite,” *SQLAlchemy 2.0 Documentation*. [Online]. Available: https://docs.sqlalchemy.org/en/20/dialects/sqlite.html. [Accessed: Oct. 16, 2026].

[11] systemd Project, “systemd.exec — Execution Environment Configuration,” *systemd Documentation*. [Online]. Available: https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html. [Accessed: Oct. 16, 2026].

[12] Pallets, “Gunicorn,” *Flask Documentation*. [Online]. Available: https://flask.palletsprojects.com/en/stable/deploying/gunicorn/. [Accessed: Oct. 16, 2026].

[13] Neon, “Neon Gives You 100 Projects for Free, with 1 GB of Postgres Storage Each,” *Neon Blog*, Oct. 2, 2026. [Online]. Available: https://neon.com/blog/neon-free-plan-1-gb-per-project. [Accessed: Oct. 16, 2026].

[14] PostgreSQL Global Development Group, “pg_dump — Export a PostgreSQL Database,” *PostgreSQL Documentation*. [Online]. Available: https://www.postgresql.org/docs/current/app-pgdump.html. [Accessed: Oct. 16, 2026].

[15] Canonical Ltd., “System Requirements,” *Ubuntu Server Documentation*, Ubuntu 24.04 LTS. [Online]. Available: https://ubuntu.com/server/docs/system-requirements/. [Accessed: Oct. 16, 2026].

[16] Canonical Ltd., “Security Suggestions,” *Ubuntu Server Documentation*. [Online]. Available: https://ubuntu.com/server/docs/explanation/security/security_suggestions/. [Accessed: Oct. 16, 2026].

[17] Canonical Ltd., “Firewall,” *Ubuntu Server Documentation*. [Online]. Available: https://ubuntu.com/server/docs/firewalls/. [Accessed: Oct. 16, 2026].

[18] SQLite Project, “Write-Ahead Logging,” *SQLite Documentation*. [Online]. Available: https://www.sqlite.org/wal.html. [Accessed: Oct. 16, 2026].

---

## Riwayat dokumen

| Versi | Tanggal | Perubahan | Penyusun |
|---|---|---|---|
| 1.0 | 16 Oktober 2026 | Modul M05 berbasis VPS Ubuntu 24.04; storage model, SQLite persistence, managed PostgreSQL, backup/restore, GitHub, lab lintas perangkat, rubrik, dan technical defense | Muhaemin Sidiq, S.Pd., M.Pd. |

**Akhir Modul — Pertemuan 5: Storage dan Managed Data Services.**
