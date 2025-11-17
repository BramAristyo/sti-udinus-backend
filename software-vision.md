# Software Vision Document
## 📘 Software Vision Document – Website Kerja Praktik STI

**Version:** 1.0  
**Date:** December, 2025  
**Author:** Bram Aristyo

---

## Executive Summary

**Website Kerja Praktik STI** adalah sistem informasi berbasis web yang dirancang untuk mendukung proses pengelolaan Kerja Praktik (KP) di Program Studi Sarjana Teknik Informatika, Universitas Dian Nuswantoro. Sistem ini hadir sebagai solusi atas proses manual yang selama ini memperlambat pengolahan data, menyulitkan pemantauan mahasiswa, dan menghambat koordinasi antara mahasiswa, dosen pembimbing, dan koordinator Kerja Praktik.

---

## 1. Vision & Mission

### 1.1 Vision Statement

> "Mewujudkan sistem Kerja Praktik yang efisien, terintegrasi dengan SSO, dan mudah digunakan, sehingga dapat mendukung proses bimbingan, pengajuan, dan sidang Kerja Praktik secara optimal dan memperkuat ekosistem layanan akademik Program Studi Teknik Informatika."

### 1.2 Mission Statements

1. **Digitalisasi Proses Kerja Praktik**

   * Menggantikan proses manual dengan sistem otomatis yang lebih cepat, aman, dan terdokumentasi.
   * Mengurangi beban administratif bagi koordinator dan dosen pembimbing.
   * Memfasilitasi proses pengajuan, bimbingan, dan sidang secara digital.

2. **Integrasi Ekosistem STI Apps**

   * Mendukung Single Sign-On (SSO) agar pengguna cukup login satu kali.
   * Sinkronisasi data mahasiswa, dosen, dan periode melalui API kampus (UDI/SIADIN).

3. **Kemudahan Akses & Penggunaan**

   * Menyediakan antarmuka modern yang intuitif dan responsif.
   * Memfasilitasi mahasiswa, dosen, dan koordinator melalui akses dari berbagai perangkat.
   * Menyediakan dashboard yang informatif untuk setiap role.

4. **Monitoring & Evaluasi yang Lebih Akurat**

   * Menyediakan data progres logbook, statistik bimbingan, dan laporan secara real-time.
   * Mempermudah pemantauan pengajuan, bimbingan, dan sidang melalui sistem tracking terpusat.
   * Menyediakan log aktivitas untuk audit dan evaluasi.

5. **Terintegrasi Dengan Data UDI**

   * Integrasi data mahasiswa yang mengambil mata kuliah Kerja Praktik secara real-time.
   * Integrasi data dosen pembimbing dari sistem UDI.
   * Eliminasi input data manual yang berulang.

---

## 2. Problem Statement

### 2.1 Current Challenges

#### Untuk Mahasiswa
- 📄 **Proses manual**: Pengajuan Kerja Praktik, pengisian logbook, dan pengajuan sidang dilakukan secara manual dan tidak terdokumentasi dengan baik.
- ❌ **Error pengajuan**: Sistem lama sering error saat pengajuan karena masalah kuota dosen yang tidak terkelola dengan baik.
- 🔍 **Tidak ada pusat informasi**: Data pengumuman, periode, dan informasi penting tersebar di berbagai medium.
- ⏳ **Progres sulit dipantau**: Tidak ada dashboard yang menampilkan status pengajuan, progress logbook, dan checklist Kerja Praktik.
- 📝 **Koordinasi dengan dosen sulit**: Komunikasi dan tracking bimbingan tidak terstruktur.
- 🏢 **Data penyelia tidak terorganisir**: Informasi penyelia dari perusahaan tidak tersimpan dengan rapi.

#### Untuk Dosen Pembimbing
- 📑 **Beban administratif tinggi**: Pengecekan logbook, pengajuan, dan sidang dilakukan manual dan memakan waktu.
- ❗ **Sulit memonitor banyak mahasiswa**: Tidak ada sistem tracking terpusat untuk memantau progres semua mahasiswa bimbingan.
- 📉 **Risiko kehilangan data**: Dokumentasi manual rawan tidak konsisten dan sulit diakses.

#### Untuk Koordinator
- 📊 **Manajemen data kompleks**: Pengelolaan data dosen, mahasiswa, kuota, dan periode memerlukan koordinasi manual.
- 🔄 **Input data berulang**: Harus input data dosen secara manual berulang kali karena tidak terintegrasi dengan UDI.
- 🔒 **Keterbatasan sistem lama**: Aplikasi lama hanya bisa menjalankan 1 periode, menyulitkan manajemen multi-periode.
- 📈 **Monitoring tidak real-time**: Sulit memantau distribusi mahasiswa, kuota dosen, dan progres bimbingan secara menyeluruh.
- ⚠️ **Kuota dosen bermasalah**: Sistem lama sering error dalam pengelolaan kuota dosen pembimbing.

### 2.2 Opportunity

Dengan sistem terintegrasi:
- ✅ Proses pengajuan, bimbingan, dan sidang menjadi optimal dan lebih cepat
- ✅ Data tersentralisasi dan terintegrasi dengan sistem akademik kampus
- ✅ Pengguna hanya login sekali (SSO)
- ✅ Koordinator dapat memantau seluruh proses Kerja Praktik dengan lebih mudah
- ✅ Dosen dapat memantau progres mahasiswa bimbingan secara real-time
- ✅ Mahasiswa dapat melihat progress dan checklist Kerja Praktik mereka
- ✅ Pengolahan laporan dan rekap menjadi lebih efisien
- ✅ Data penyelia tersimpan rapi dan terstruktur
- ✅ Sistem dapat menjalankan multiple periode sekaligus
- ✅ Tidak perlu input data dosen berulang (sinkronisasi otomatis dengan UDI)

---

## 3. Product Overview

### 3.1 Product Description

Website Kerja Praktik STI adalah platform terpusat untuk mengelola seluruh aktivitas Kerja Praktik, seperti:

- Manajemen periode ajaran dengan sinkronisasi dari UDI (support multiple periode)
- Manajemen data dosen dan kuota bimbingan (terintegrasi dengan UDI)
- Manajemen data mahasiswa dan plotting ke dosen pembimbing
- Pengajuan Kerja Praktik oleh mahasiswa (dengan/tanpa tempat magang)
- Manajemen logbook bimbingan
- Manajemen gelombang sidang KP
- Pengajuan dan monitoring sidang KP
- Dashboard informatif untuk setiap role
- Manajemen pengumuman
- Log aktivitas sistem
- Penyimpanan data penyelia perusahaan yang terstruktur
- Akses berbasis role (koordinator, dosen, mahasiswa)
- Integrasi Single Sign-On STI Apps

Sistem ini mendukung alur kerja yang lebih cepat, aman, dan terorganisir dari awal pengajuan hingga sidang Kerja Praktik.

### 3.2 Key Features

#### Untuk Mahasiswa:
* ✅ Login dengan email/password atau SSO Google
* ✅ Dashboard dengan checklist dan progress Kerja Praktik
* ✅ Melihat dan mengisi form pengajuan Kerja Praktik
* ✅ Memilih dosen pembimbing dari daftar yang tersedia
* ✅ Mengajukan KP dengan atau tanpa tempat magang
* ✅ Melihat status dan riwayat pengajuan
* ✅ Membuat dan mengupdate logbook bimbingan
* ✅ Mengisi link folder Kerja Praktik
* ✅ Melihat progress logbook per bab
* ✅ Mengajukan sidang KP (setelah logbook bab 5 di-ACC)
* ✅ Melengkapi data penyelia perusahaan sebelum sidang
* ✅ Melihat status dan draft pengajuan sidang
* ✅ Melihat pengumuman Kerja Praktik
* ✅ Mengubah password

#### Untuk Dosen Pembimbing:
* ✅ Login dengan email/password atau SSO Google
* ✅ Dashboard dengan statistik logbook mahasiswa bimbingan
* ✅ Melihat daftar logbook mahasiswa bimbingan dengan filter dan search
* ✅ Menerima atau meminta revisi logbook mahasiswa
* ✅ Melihat detail logbook dan informasi mahasiswa
* ✅ Melihat daftar pengajuan mahasiswa bimbingan
* ✅ Menerima atau menolak pengajuan Kerja Praktik
* ✅ Melihat jadwal sidang mahasiswa bimbingan
* ✅ Memberi nilai sidang mahasiswa bimbingan
* ✅ Melihat log aktivitas mahasiswa bimbingan
* ✅ Melihat pengumuman
* ✅ Upload tanda tangan digital untuk keperluan daftar nilai
* ✅ Mengubah password

#### Untuk Koordinator:
* ✅ Login dengan email/password
* ✅ Dashboard dengan statistik logbook, distribusi mahasiswa, dan informasi kuota
* ✅ Manajemen periode ajaran (CRUD, sinkronisasi dengan UDI, aktif/nonaktif, support multiple periode)
* ✅ Manajemen gelombang sidang (CRUD, aktif/nonaktif, multiple gelombang per periode)
* ✅ Manajemen pengumuman (CRUD dengan notifikasi)
* ✅ Manajemen data dosen (CRUD, sinkronisasi otomatis dengan UDI, filter)
* ✅ Manajemen kuota dosen (update kuota, alokasi per periode, statistik)
* ✅ Plotting mahasiswa ke dosen pembimbing (manual & batch)
* ✅ Manajemen data mahasiswa KP (CRUD, tambah via excel/manual, sinkronisasi dengan UDI)
* ✅ Melihat daftar pengajuan mahasiswa dengan filter, search, dan sort
* ✅ Menerima atau menolak pengajuan Kerja Praktik
* ✅ Rekap logbook per dosen dan mahasiswa
* ✅ Monitoring sidang KP (plotting jadwal, menerima/revisi)
* ✅ Memberi nilai sidang mahasiswa
* ✅ Download PDF daftar nilai mahasiswa untuk rekapitulasi
* ✅ Upload tanda tangan digital
* ✅ Melihat log aktivitas seluruh user dengan filter
* ✅ Mengubah password

---

## 4. Target Audience

### Mahasiswa 🎓
* **Goal**: Mengajukan Kerja Praktik, melakukan bimbingan dengan dosen, mengajukan sidang, dan menyelesaikan Kerja Praktik sesuai ketentuan.
* **Pain Points**: Error pengajuan karena kuota, proses manual, informasi tersebar, sulit memantau progress, koordinasi dengan dosen tidak terstruktur, data penyelia tidak terorganisir.

### Dosen Pembimbing 👨‍🏫
* **Goal**: Membimbing mahasiswa, memantau progress logbook, mengevaluasi pengajuan, dan memastikan kualitas Kerja Praktik.
* **Pain Points**: Beban administratif tinggi, sulit memonitor banyak mahasiswa, dokumentasi manual tidak terstruktur.

### Koordinator Kerja Praktik 🎯
* **Goal**: Mengelola seluruh proses Kerja Praktik secara terstruktur, efisien, dan terpantau, termasuk manajemen data, kuota, gelombang sidang, dan monitoring.
* **Pain Points**: Manajemen data kompleks, input data dosen berulang, sistem lama hanya support 1 periode, kuota dosen error, sinkronisasi data sulit, monitoring tidak real-time, beban koordinasi tinggi.

---

## 5. Business Goals

### 5.1 Short-term Goals (0–6 bulan)
- Implementasi CRUD periode dengan support multiple periode
- Implementasi CRUD gelombang sidang
- Implementasi CRUD dosen, mahasiswa, dan pengumuman
- Integrasi SSO dasar dengan STI Apps
- Dashboard untuk setiap role (koordinator, dosen, mahasiswa)
- Sistem pengajuan Kerja Praktik
- Sistem logbook bimbingan
- Sistem pengajuan sidang KP
- Sinkronisasi data dengan UDI/SIADIN (eliminasi input manual berulang)
- Penyimpanan data penyelia terstruktur

### 5.2 Medium-term Goals (6–12 bulan)
- Peningkatan UX/UI
- Export laporan dan rekap (PDF/Excel)
- Penambahan fitur notifikasi email
- Optimalisasi performa & keamanan
- Fitur advanced filtering dan search
- Analytics dan reporting yang lebih detail
- Sistem upload tanda tangan digital

### 5.3 Long-term Goals (1–2 tahun)
- Integrasi penuh dengan sistem akademik UDINUS
- Ekspansi fitur pelaporan & analitik
- Standardisasi Kerja Praktik dalam STI Apps
- Fitur mobile app untuk akses lebih mudah
- Automated workflow untuk approval process

---

## 6. Success Metrics

### 6.1 User Metrics

| Metric             | Target           |
| ------------------ | ---------------- |
| Mahasiswa aktif KP | 80% per semester |
| Dosen aktif        | 90% per semester |
| Kepuasan pengguna  | > 4.3/5          |
| Penggunaan SSO     | 100%             |
| Pengajuan digital  | 100%             |

### 6.2 Operational Metrics

| Metric           | Target    |
| ---------------- | --------- |
| Uptime           | > 99%     |
| API response     | < 500 ms  |
| Error rate       | < 0.5%    |
| Sinkronisasi UDI | < 5 menit |
| Pengajuan error  | 0%        |

### 6.3 Process Metrics

| Metric                    | Target   |
| ------------------------- | -------- |
| Waktu pengajuan KP        | < 1 hari |
| Waktu respon dosen        | < 3 hari |
| Akurasi data sinkronisasi | > 99%    |
| Kelengkapan logbook       | > 95%    |

---

## 7. Competitive Analysis

### 7.1 Comparisons

#### Sistem Lama Kerja Praktik
- ❌ Hanya support 1 periode aktif
- ❌ Error pengajuan karena kuota bermasalah
- ❌ Harus input data dosen berulang
- ❌ Tidak terintegrasi dengan UDI
- ❌ Data penyelia tidak terstruktur
- 🎯 Sistem baru: Multiple periode, kuota terkelola baik, integrasi UDI, data penyelia rapi

#### Sistem Internal Lain (TA, BK, Alumni)
- ❌ Tidak mendukung proses Kerja Praktik secara spesifik
- ❌ Tidak ada manajemen gelombang sidang
- ❌ Tidak ada penyimpanan data penyelia
- 🎯 Website Kerja Praktik fokus pada kebutuhan spesifik proses KP

#### Sistem Akademik Umum (LMS)
- ❌ Tidak optimal untuk proses bimbingan dan pengajuan Kerja Praktik
- ❌ Tidak ada manajemen kuota dosen dan plotting mahasiswa
- 🎯 Sistem ini fokus pada manajemen bimbingan, logbook, dan sidang Kerja Praktik

### 7.2 Unique Value Proposition

**"Sistem Kerja Praktik terintegrasi STI Apps dengan SSO, support multiple periode, sinkronisasi otomatis dengan UDI, dan fokus pada manajemen bimbingan, logbook, pengajuan, dan sidang secara efisien dan terstruktur."**

---

## 8. Technical Vision

### 8.1 Architecture Philosophy

- **Frontend**: Next.js + TypeScript  
- **Backend**: Laravel + REST API  
- **Database**: MySQL  
- **Styling**: Tailwind CSS  
- **Auth**: SSO STI Apps + Sanctum  
- **Deployment**: VPS
- **API Integration**: UDI/SIADIN untuk sinkronisasi data

### 8.2 Technology Stack

#### Backend
- Laravel Framework
- MySQL Database
- Laravel Sanctum untuk authentication
- RESTful API architecture
- Queue system untuk sinkronisasi data

#### Frontend
- Next.js Framework
- TypeScript
- Tailwind CSS
- Shadcn UI components
- Responsive design

#### Integration
- SSO STI Apps
- UDI/SIADIN API untuk sinkronisasi
- Google OAuth untuk SSO Google

### 8.3 Future Roadmap

#### Phase 1 (Current)
- CRUD periode dengan support multiple periode
- CRUD gelombang sidang
- CRUD dosen, mahasiswa, pengumuman
- Sistem pengajuan Kerja Praktik
- Sistem logbook bimbingan
- Sistem pengajuan sidang KP
- SSO dasar
- Sinkronisasi data dengan UDI
- Penyimpanan data penyelia terstruktur

#### Phase 2 (Next 6 months)
- Dashboard analytics yang lebih detail
- Notifikasi email
- Export PDF/Excel untuk laporan
- Advanced filtering dan search
- Upload tanda tangan digital
- Download PDF daftar nilai

#### Phase 3 (Innovation)
- AI-based recommendation untuk dosen pembimbing
- Smart progress tracking dengan predictive analytics
- Real-time collaboration tools
- Automated workflow untuk approval process
- Mobile app development

---

## 9. Risk Assessment

### 9.1 Technical Risks

| Risk                        | Level  | Mitigation                                          |
| --------------------------- | ------ | --------------------------------------------------- |
| Integrasi SSO gagal         | High   | Dokumentasi lengkap, testing, fallback mechanism    |
| Integrasi UDI gagal         | High   | Error handling, retry mechanism, manual sync option |
| Beban server meningkat      | Medium | Optimasi API, caching, queue system                 |
| Bug pada logbook            | Medium | Code review, unit testing, backup mechanism         |
| Data loss pada sinkronisasi | Medium | Backup system, validation, rollback mechanism       |
| Error pengajuan kuota       | High   | Validasi kuota real-time, transaction management    |

### 9.2 Business Risks

| Risk                   | Level  | Mitigation                                       |
| ---------------------- | ------ | ------------------------------------------------ |
| Adopsi pengguna rendah | Medium | Pelatihan & sosialisasi, user-friendly interface |
| Perubahan requirement  | Medium | Agile development, regular feedback              |
| Resistance to change   | Medium | Change management, training, support             |
| Data privacy concerns  | Low    | Security best practices, compliance              |

### 9.3 Operational Risks

| Risk                    | Level  | Mitigation                               |
| ----------------------- | ------ | ---------------------------------------- |
| Downtime during peak    | Medium | Load balancing, monitoring, auto-scaling |
| Data inconsistency      | Medium | Validation, transaction management       |
| Performance degradation | Low    | Performance monitoring, optimization     |

---

## 10. Stakeholders

- **Mahasiswa** (pengguna utama)  
- **Dosen Pembimbing** (pengguna utama)  
- **Koordinator Kerja Praktik** (operasional)  
- **Program Studi** (pengambil keputusan)  
- **Tim Pengembang STI Apps** (integrasi teknis)
- **Administrator Sistem** (maintenance & support)
- **Perusahaan/Penyelia** (data tersimpan dalam sistem)

---

## 11. Feature Summary

Berdasarkan dokumen fitur yang ada, sistem Kerja Praktik mencakup 11 modul utama:

1. **STI-KP01: Login** - Login dengan email/password dan SSO Google, perubahan password
2. **STI-KP02: Setting Periode Ajaran** - CRUD periode, sinkronisasi dengan UDI, support multiple periode
3. **STI-KP03: Setting Gelombang Sidang** - CRUD gelombang, aktif/nonaktif, multiple gelombang per periode
4. **STI-KP04: Manajemen Mahasiswa KP** - CRUD mahasiswa KP, tambah via excel/manual, plotting dospem
5. **STI-KP05: Manajemen Dosen Pembimbing KP** - CRUD dosen, sinkronisasi dengan UDI, kuota
6. **STI-KP06: Pengajuan KP** - Form pengajuan KP, approval oleh dosen/koordinator
7. **STI-KP07: Logbook Mahasiswa** - Manajemen logbook bimbingan, approval oleh dosen
8. **STI-KP08: Pengajuan Sidang KP** - Form pengajuan sidang, plotting jadwal, penilaian, TTD digital, download PDF nilai
9. **STI-KP09: Pengumuman** - CRUD pengumuman dengan notifikasi
10. **STI-KP10: Dashboard** - Dashboard informatif untuk koordinator, dosen, dan mahasiswa
11. **STI-KP11: Log Aktivitas** - Tracking aktivitas user untuk audit dan monitoring

---

## 12. Conclusion

Website Kerja Praktik STI hadir sebagai solusi digital yang efisien, modern, dan terintegrasi untuk mengelola seluruh proses Kerja Praktik dari pengajuan hingga sidang. Dengan mendukung SSO, sistem terpusat, interface modern, dan mengatasi masalah sistem lama, platform ini akan:

- 🚀 Mempercepat proses administrasi dan koordinasi  
- 📊 Memudahkan pemantauan progress mahasiswa dan bimbingan  
- 🔗 Mengintegrasikan layanan akademik secara menyeluruh  
- 🎓 Mendukung proses Kerja Praktik secara optimal dan terstruktur  
- 📝 Meningkatkan kualitas dokumentasi dan tracking  
- ⚡ Meningkatkan efisiensi kerja dosen dan koordinator  
- ✅ Menghilangkan error pengajuan karena kuota yang bermasalah
- 🔄 Eliminasi input data dosen berulang dengan integrasi UDI
- 📅 Support multiple periode sekaligus
- 📋 Penyimpanan data penyelia yang terstruktur

Sistem ini tidak hanya menggantikan proses manual dan memperbaiki kelemahan sistem lama, tetapi juga meningkatkan kualitas dan akuntabilitas proses Kerja Praktik di Program Studi Teknik Informatika.

---

**Document Version:** 1.0  
**Last Updated:** December 2025  
**Status:** Approved
