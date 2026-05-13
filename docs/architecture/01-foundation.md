# 1. Foundation
### Fondasi Arsitektur / Architectural Foundation

Bab ini membahas hal paling dasar dalam menyusun proyek perangkat lunak: bagaimana folder ditata, bagaimana kode dipisah ke dalam *lapisan*, dan bagaimana kita mencatat keputusan-keputusan penting sehingga tim yang datang kemudian tahu *kenapa* sebuah pilihan dibuat.

This chapter covers the most fundamental aspects of setting up a software project: how folders are organized, how code is separated into *layers*, and how we record important decisions so future team members understand *why* a choice was made.

---

## Struktur Folder Dasar / Basic Folder Structure

### Apa ini? / What is this?

**ID:** Struktur folder adalah cara kita menata file & folder di sebuah proyek agar konsisten dan mudah ditemukan. Bayangkan seperti tata letak rak di perpustakaan — buku sains di rak A, novel di rak B. Tanpa tata letak yang disepakati, setiap orang akan menaruh barang di tempat berbeda, dan menemukan sesuatu jadi mimpi buruk.

**EN:** A folder structure is how we organize files & folders in a project so they're consistent and easy to find. Think of it like a library's shelf layout — science books on shelf A, novels on shelf B. Without an agreed layout, everyone puts things in different places, and finding anything becomes a nightmare.

### Mengapa penting? / Why it matters?

- Developer baru bisa cepat memahami isi proyek.
- Tools (CI/CD, linter, test runner) bisa menemukan file di lokasi yang dapat diprediksi.
- Mengurangi konflik saat banyak orang bekerja di proyek yang sama.
- Memisahkan kode aplikasi, dokumentasi, infrastruktur, dan skrip → tidak tercampur dalam satu folder besar.

### Use Case

> **Skenario:** Tim Anda menerima developer baru. Hari pertama, ia perlu menjawab: "Di mana saya menambahkan endpoint API baru? Di mana spec test ditulis? Bagaimana cara men-deploy ke staging?"
>
> Dengan struktur folder standar, ia bisa langsung melihat `src/` untuk kode, `specs/` untuk spec test, `infra/` untuk konfigurasi deploy, dan `docs/` untuk dokumentasi. Tanpa struktur jelas, ia harus tanya senior berulang kali → menghambat onboarding.
{ .usecase }

### Istilah & Konsep / Glossary

- **`src/`** — singkatan *source*, berisi kode aplikasi utama (production code).
- **`infra/`** — singkatan *infrastructure*, berisi file konfigurasi server, container, Kubernetes manifest, Terraform, dll.
- **`specs/`** — folder untuk *specification* — biasanya berisi file `.feature` (Gherkin) untuk BDD test, atau dokumen spesifikasi fungsional.
- **`adr/`** — *Architecture Decision Records*, catatan keputusan arsitektur yang pernah diambil. Akan dibahas di bagian berikutnya.
- **`scripts/`** — skrip bantu (mis. backup, seeder data, migrasi manual).
- **`.github/`** — folder khusus GitHub berisi workflow CI/CD, template issue, dll.

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Setiap proyek menggunakan struktur folder dasar yang disepakati sebelum coding dimulai.

<span class="badge badge-mandatory">Mandatory</span> Folder spesifik (`src/`, `infra/`, `docs/`, `specs/`) tidak dicampur isinya.

### Contoh Struktur / Example

```text
[root]
├── .vscode/         ← konfigurasi editor (opsional, sebagian tim ignore)
├── .github/
│   ├── instructions/
│   ├── skills/
│   └── workflows/   ← CI/CD pipeline
├── docs/
│   ├── adr/         ← architecture decision records
│   └── api/         ← API spec (OpenAPI/Swagger)
├── src/             ← kode aplikasi
├── scripts/         ← skrip bantu
├── specs/           ← BDD spec / feature files
├── infra/           ← konfigurasi deploy & infra
├── docker-compose.yml
└── readme.md
```

---

## Clean Architecture

### Apa ini? / What is this?

**ID:** Clean Architecture adalah cara membagi kode menjadi *lapisan-lapisan* (layer) dengan aturan ketat: lapisan dalam tidak boleh tahu apa-apa tentang lapisan luar. Bayangkan seperti bawang — di tengah ada *core* bisnis (paling penting), dikelilingi lapisan-lapisan teknis (framework, database, UI).

**EN:** Clean Architecture is a way of dividing code into *layers* with strict rules: inner layers must not know anything about outer layers. Imagine an onion — the business *core* sits at the center (most important), surrounded by technical layers (framework, database, UI).

Lapisan utamanya / The main layers:

1. **Domain** (inti) — aturan & entitas bisnis murni, tanpa framework.
2. **Application** — orkestrasi *use case* (alur cerita aplikasi).
3. **Infrastructure** — implementasi teknis (database, HTTP client, queue).
4. **Presentation** — yang dilihat pengguna atau di-expose ke dunia luar (REST API, gRPC, UI).

Aturan emas: **arah ketergantungan selalu menuju ke dalam.** Presentation tahu Application, Application tahu Domain. Tapi Domain TIDAK tahu apa-apa di luar dirinya.

### Mengapa penting? / Why it matters?

- **Domain bisa diuji tanpa database, tanpa HTTP, tanpa framework** → unit test cepat & deterministik.
- **Mengganti teknologi tidak meledakkan seluruh kode.** Migrasi dari MySQL ke PostgreSQL? Cukup ganti adapter di Infrastructure — Domain tidak tersentuh.
- **Logika bisnis tidak "bocor" ke controller atau ke SQL query** → mudah dilacak & dirawat.

### Use Case

> **Skenario:** Aplikasi e-commerce Anda awalnya pakai REST API + MySQL. Setahun kemudian, mobile team minta dukungan gRPC + ada kebutuhan migrasi ke PostgreSQL.
>
> Dengan Clean Architecture: tambahkan adapter gRPC baru di Presentation, ganti driver database di Infrastructure. **Domain & Application tidak berubah** karena aturan bisnis tetap sama.
>
> Tanpa Clean Architecture (kode bisnis bercampur di controller & SQL): perubahan ini bisa jadi proyek 3 bulan dengan banyak bug, karena logika "diskon untuk member" tersebar di puluhan file controller.
{ .usecase }

### Istilah & Konsep / Glossary

- **Entity** — objek bisnis utama yang punya *identitas* (mis. `User` dengan ID, `Order` dengan order number). Dua entity berbeda meskipun isinya sama jika ID-nya beda.
- **Value Object** — objek yang ditentukan oleh *nilainya*, bukan identitas. Mis. `Money(amount=100, currency=USD)`. Dua value object dengan nilai sama = sama.
- **Aggregate Root** — entity utama yang menjadi pintu masuk untuk memodifikasi sekumpulan objek terkait. Mis. untuk mengubah `OrderItem`, harus melalui `Order` (aggregate root). Ini mencegah data inkonsisten.
- **Domain Event** — peristiwa penting di domain (mis. `OrderPlaced`, `PaymentReceived`). Layanan lain bisa "mendengarkan" event ini dan bereaksi.
- **Repository** — abstraksi untuk menyimpan & mengambil entity. Di Domain hanya didefinisikan *interface*-nya; implementasinya (pakai SQL, MongoDB, file) ada di Infrastructure.
- **Domain Service** — logika domain yang tidak pas masuk ke entity tertentu (mis. `TransferService` yang melibatkan dua `Account`).
- **DDD (Domain-Driven Design)** — pendekatan desain yang menempatkan model bisnis (Domain) sebagai pusat.
- **ArchUnit / NetArchTest** — library untuk menulis *test* yang memverifikasi aturan arsitektur (mis. "Domain tidak boleh mengimport package Infrastructure").

### Anti-pattern

<div class="antipattern" markdown>
**Yang sering salah:**

- ❌ Domain mengimport library ORM (mis. `from sqlalchemy import Column` di entity) → Domain tergantung framework.
- ❌ Controller menulis SQL langsung → logika bisnis bocor ke Presentation.
- ❌ Service di Application memanggil class konkret dari Infrastructure → dependency rule terbalik.
- ❌ Folder dibagi per *jenis file* (`controllers/`, `models/`, `services/`) → struktur teknis, bukan arsitektur. Untuk proyek besar, lebih baik dibagi per *fitur* atau per *layer*.

**Yang benar:** Domain hanya berisi class murni (POJO / POCO / dataclass). Akses database lewat *interface* Repository yang implementasinya ada di Infrastructure.
</div>

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Batas antar layer ketat: Presentation → Application → Domain → Infrastructure. Inner layer tidak boleh tahu outer layer.

<span class="badge badge-mandatory">Mandatory</span> Domain harus bebas dari dependency framework — hanya konstruk bahasa pemrograman murni.

<span class="badge badge-mandatory">Mandatory</span> Domain dimodelkan dengan: Entity, Value Object, Aggregate Root, Domain Event, Repository (interface), Domain Service.

<span class="badge badge-mandatory">Mandatory</span> Komunikasi antar layer melalui *interface*, bukan class konkret.

<span class="badge badge-mandatory">Mandatory</span> Logika domain TIDAK bocor ke Application atau Presentation.

<span class="badge badge-mandatory">Mandatory</span> Aturan layer divalidasi dengan architecture unit test (ArchUnit, NetArchTest, atau setara).

<span class="badge badge-mandatory">Mandatory</span> Struktur folder mencerminkan layer arsitektur, bukan jenis file.

<span class="badge badge-mandatory">Mandatory</span> Kode shared antar service diletakkan di `/src/shared/` — tidak ada duplikasi cross-cutting concern.

### Contoh Struktur / Example

```text
[root]
└── src/
    ├── shared/
    │   ├── contracts/      ← interface/DTO yang dipakai bersama
    │   └── common/         ← utilitas umum (logger wrapper, helper)
    └── service-name/
        ├── src/
        │   ├── presentation/    ← REST/gRPC controller
        │   ├── application/     ← use case, orchestrator
        │   ├── domain/          ← entity, value object, repo interface
        │   └── infrastructure/  ← DB, HTTP client, queue impl
        └── tests/
            ├── unit/           ← test domain & application (no I/O)
            ├── integration/    ← test dgn DB / queue nyata
            └── contract/       ← consumer-driven contract test
```

---

## Architecture Decision Record (ADR)

### Apa ini? / What is this?

**ID:** ADR adalah catatan singkat tentang keputusan arsitektur yang pernah diambil tim — apa keputusannya, mengapa, dan apa konsekuensinya. Mirip seperti "catatan rapat" tapi khusus untuk pilihan teknis penting.

**EN:** An ADR is a short note about an architectural decision the team has made — what was decided, why, and what the consequences are. Like meeting minutes, but specific to important technical choices.

### Mengapa penting? / Why it matters?

- **6 bulan kemudian, tidak ada yang ingat kenapa kita pilih RabbitMQ daripada Kafka.** Tanpa ADR, perdebatan terulang dan keputusan bisa dibalik tanpa alasan.
- **Onboarding lebih cepat** — anggota baru bisa baca ADR untuk memahami konteks historis.
- **Mencegah "argumen yang sama berulang kali"** — keputusan yang sudah dibuat punya catatan yang bisa dirujuk.

### Use Case

> **Skenario:** Setahun lalu tim memutuskan pakai *Outbox Pattern* untuk publikasi event. Hari ini ada developer baru yang melihat tabel `outbox` dan berpikir "ini overhead tidak perlu" dan ingin menghapusnya.
>
> Dengan ADR `0007-use-outbox-pattern.md`, dia bisa baca: "Outbox dipakai karena pernah terjadi event hilang saat broker down. Konsekuensi: write & publish jadi atomic. Alternatif yang ditolak: publish langsung (alasan: tidak transaksional)."
>
> Tanpa ADR: ia menghapus outbox, masalah lama muncul lagi, butuh post-mortem mahal.
{ .usecase }

### Istilah & Konsep / Glossary

- **Status ADR:**
    - *Proposed* — masih diusulkan, belum disetujui.
    - *Accepted* — sudah disepakati & diterapkan.
    - *Deprecated* — tidak relevan lagi tapi belum diganti.
    - *Superseded* — diganti dengan ADR lain yang lebih baru (mis. ADR 0007 superseded by 0015).
- **Context** — situasi/masalah yang melatari keputusan.
- **Consequences** — efek setelah keputusan diambil — yang baik dan yang buruk.

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Setiap keputusan arsitektur signifikan dicatat sebagai ADR.

<span class="badge badge-mandatory">Mandatory</span> Format ADR: **Title, Status, Context, Decision, Consequences**.

<span class="badge badge-mandatory">Mandatory</span> ADR disimpan di `/docs/adr/` dengan penomoran urut (`0001-...`, `0002-...`).

<span class="badge badge-mandatory">Mandatory</span> ADR yang sudah *Accepted* bersifat immutable — jika berubah, buat ADR baru yang meng-*supersede*, jangan edit yang lama.

<span class="badge badge-mandatory">Mandatory</span> ADR ditulis saat keputusan dibuat, BUKAN setelah keputusan berlaku berbulan-bulan.

### Contoh Struktur / Example

```markdown
# 0007. Use Outbox Pattern for Event Publishing

## Status
Accepted — 2025-03-14

## Context
Kami pernah mengalami event hilang saat message broker (RabbitMQ) sempat down 30 detik
saat order service melakukan commit transaksi. Hasilnya: order tersimpan di DB tapi
event `OrderPlaced` tidak terpublikasi → downstream service (inventory) tidak ter-update.

## Decision
Semua domain event ditulis ke tabel `outbox` dalam transaksi yang sama dengan
operasi bisnisnya. Background relay membaca tabel ini dan mempublikasikan ke broker.

## Consequences
+ Event tidak akan hilang meski broker down (akan di-retry).
+ Publish jadi *eventually consistent* (delay milidetik–detik).
- Tambahan kompleksitas: butuh relay process & monitoring tabel outbox.
- Tabel outbox tumbuh — perlu housekeeping.
```

---

## Ringkasan / Summary

Di bab ini kita belajar:

1. **Struktur folder dasar** memberi kerangka konsisten untuk seluruh proyek.
2. **Clean Architecture** memisahkan kode jadi layer dengan aturan ketat agar Domain bersih dari teknis.
3. **ADR** mencatat *mengapa* keputusan dibuat agar pengetahuan tidak hilang.

> **Selanjutnya:** [2. Application Patterns](02-application-patterns.md) — pola-pola umum untuk membangun aplikasi modern: microservice, async, caching, dan lainnya.
