# Architecture Guidelines
### Panduan Arsitektur Perangkat Lunak / Software Architecture Guide

Selamat datang! Dokumentasi ini ditulis untuk **mengajar**, bukan sekadar menjadi daftar aturan. Tujuan utamanya: programmer yang tadinya *tidak tahu* sebuah konsep arsitektur, setelah membaca menjadi *paham* — apa itu, kapan dipakai, dan kenapa penting.

Welcome! This documentation is written to **teach**, not just to list rules. The main goal: a programmer who *didn't know* an architectural concept will *understand* it after reading — what it is, when to use it, and why it matters.

---

## Untuk Siapa Dokumentasi Ini? / Who Is This For?

- **Developer junior** yang baru kenal konsep seperti *microservice*, *idempotency*, atau *Circuit Breaker*.
- **Developer berpengalaman** yang ingin meninjau aturan & best practice.
- **Tech lead / arsitek** yang ingin memastikan tim mengikuti standar yang sama.

> Tidak masalah jika Anda belum pernah dengar istilah-istilah teknis di atas. Setiap bab dimulai dari nol dan menjelaskan istilahnya sebelum masuk ke aturan. / It's fine if you've never heard those terms. Every chapter starts from zero and explains terms before introducing rules.

---

## Cara Membaca Dokumentasi Ini / How to Read This

Setiap topik mengikuti struktur yang sama agar mudah diikuti:

Every topic follows the same structure for easy reading:

1. **Apa ini? / What is this?** — definisi sederhana + analogi
2. **Mengapa penting? / Why it matters?** — masalah yang dipecahkan
3. **Use Case** — cerita skenario konkret
4. **Istilah & Konsep / Glossary** — penjelasan istilah teknis dengan bahasa awam
5. **Anti-pattern** (jika relevan) — contoh salah dan kenapa
6. **Aturan / Rules** — daftar aturan dengan label
7. **Contoh Struktur / Example** — folder tree atau kode

### Legenda Label / Label Legend

<span class="badge badge-mandatory">Mandatory</span> Aturan **wajib** diterapkan di semua proyek. / **Required** in every project.

<span class="badge badge-optional">Optional</span> Aturan opsional — diterapkan sesuai kebutuhan tim & proyek. / Optional — apply based on team/project needs.

---

## Daftar Bab / Table of Contents

| # | Bab / Chapter | Isi / Contents |
|---|---|---|
| 1 | [Foundation](architecture/01-foundation.md) | Struktur folder dasar, Clean Architecture (layered + DDD), Architecture Decision Record (ADR) |
| 2 | [Application Patterns](architecture/02-application-patterns.md) | Microservice, async & concurrency, data access, caching, service bus & event-driven, idempotency |
| 3 | [Security](architecture/03-security.md) | Authentication & authorization, secret management, encryption, secure coding |
| 4 | [Quality & Reliability](architecture/04-quality-reliability.md) | Testing (unit, integration, E2E, BDD), code quality, edge case handling, exception handling |
| 5 | [Operations](architecture/05-operations.md) | Logging, health check, application performance monitoring (APM), audit trail |
| 6 | [User Interface](architecture/06-user-interface.md) | Standar komponen UI, design system, aksesibilitas (a11y) |
| 7 | [Optional Features](architecture/07-optional-features.md) | Feature flags / toggle |

---

## Rekomendasi Urutan Membaca / Suggested Reading Order

**Jika Anda baru / If you're new:**

1. Mulai dari **Foundation** untuk memahami struktur dasar proyek.
2. Lanjut ke **Application Patterns** — bagian terbesar yang membahas pola umum aplikasi modern.
3. **Security** — wajib dipahami sebelum kode masuk produksi.
4. **Quality & Reliability** — cara memastikan kode tetap bisa diandalkan.
5. **Operations** — cara memantau aplikasi setelah deploy.
6. **User Interface** & **Optional Features** — sesuai kebutuhan.

---

## Istilah Singkat / Quick Glossary

Beberapa istilah yang sering muncul di berbagai bab:

- **Bounded Context** — batas logis sebuah area bisnis dalam sistem (mis. "Pemesanan", "Pembayaran"). Setiap konteks punya bahasa & model sendiri.
- **Aggregate Root** — pintu masuk satu-satunya untuk mengubah sekumpulan data yang berkaitan (mis. `Order` adalah root, `OrderItem` di bawahnya).
- **Idempotency** — sifat operasi yang aman dijalankan berulang kali tanpa efek samping berbeda. Bayar 1x atau bayar 5x dengan request yang sama → hasilnya sama.
- **Circuit Breaker** — pola seperti sekring listrik: kalau layanan downstream gagal terus, "buka sirkuit" dan tolak request sementara agar tidak menumpuk.
- **p95 latency** — angka latency di mana 95% request lebih cepat dari nilai itu. Mengukur pengalaman *mayoritas* pengguna, bukan rata-rata.
- **RBAC / ABAC** — Role-Based / Attribute-Based Access Control. Cara mengatur siapa boleh melakukan apa.
- **CQRS** — Command Query Responsibility Segregation. Pisahkan model untuk *menulis* data dan *membaca* data.
- **Saga** — pola untuk mengelola transaksi yang menyebar di beberapa layanan, dengan langkah *kompensasi* jika ada yang gagal.

> Istilah-istilah ini dijelaskan lebih lengkap di bab masing-masing. / These terms are explained in detail in their respective chapters.

---

## Kontribusi / Contributing

Dokumentasi ini hidup — terbuka untuk perbaikan, klarifikasi, dan tambahan contoh. Buka *issue* atau *pull request* di repository.

This documentation is alive — open for fixes, clarifications, and additional examples. File an *issue* or *pull request* in the repository.
