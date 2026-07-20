# Software Architecture Docs

[![Deploy Documentation](https://github.com/mabudiman/software-architecture-docs/actions/workflows/deploy-docs.yml/badge.svg)](https://github.com/mabudiman/software-architecture-docs/actions/workflows/deploy-docs.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Kumpulan catatan arsitektur perangkat lunak yang saya susun sambil belajar dan terus saya perbarui. Ditulis dua bahasa (Indonesian + English) karena saya sendiri sering bolak-balik antara keduanya. Isinya mulai dari hal dasar kayak struktur folder dan Clean Architecture, sampai topik yang biasanya bikin pusing kayak *idempotency*, *Saga*, atau *Circuit Breaker*.

A set of software architecture notes I put together while learning — and keep updating. Bilingual (Indonesian + English) because I keep switching between the two. Covers the basics like folder structure and Clean Architecture, up to the stuff that usually trips people up: *idempotency*, *Saga*, *Circuit Breaker*.

---

## Live Site

**[https://projects.mabudiman.my.id/software-architecture-docs/](https://projects.mabudiman.my.id/software-architecture-docs/)**

---

## Daftar Bab / Table of Contents

| # | Bab / Chapter | Isi / Contents |
|---|---|---|
| 1 | [Foundation](https://projects.mabudiman.my.id/software-architecture-docs/architecture/01-foundation/) | Struktur folder, Clean Architecture, Architecture Decision Record (ADR) |
| 2 | [Application Patterns](https://projects.mabudiman.my.id/software-architecture-docs/architecture/02-application-patterns/) | Microservice, Async & Concurrency, Data Access, Cache, Service Bus, Idempotency |
| 3 | [Security](https://projects.mabudiman.my.id/software-architecture-docs/architecture/03-security/) | Authentication & Authorization, Secret Management, Encryption, Secure Coding |
| 4 | [Quality & Reliability](https://projects.mabudiman.my.id/software-architecture-docs/architecture/04-quality-reliability/) | Testing (Unit, Integration, E2E, BDD), Code Quality, Edge Case, Exception Handling |
| 5 | [Operations](https://projects.mabudiman.my.id/software-architecture-docs/architecture/05-operations/) | Logging, Health Check, APM, Audit Trail |
| 6 | [User Interface](https://projects.mabudiman.my.id/software-architecture-docs/architecture/06-user-interface/) | Component Library, Design System, Accessibility (a11y) |
| 7 | [Optional Features](https://projects.mabudiman.my.id/software-architecture-docs/architecture/07-optional-features/) | Feature Flags |

---

## Apa yang Membuat Dokumentasi Ini Berbeda? / What Makes This Different?

Saya sengaja hindari gaya "daftar aturan kering" yang biasanya ada di dokumentasi perusahaan. Tiap topik saya tulis dengan pendekatan yang sama supaya gampang dibaca: definisi dan analogi sederhana dulu, kenapa ini penting, contoh skenario nyata, glosarium istilah, anti-pattern kalau perlu, terus aturan mainnya (dengan label `[Mandatory]` atau `[Optional]`), dan diakhiri contoh struktur atau potongan kode.

I deliberately avoid the dry "list of rules" style you usually see in corporate docs. Each topic follows the same approach so it's easy to read: a simple definition and analogy first, why it matters, a real scenario, a glossary, anti-patterns when relevant, then the actual rules (labeled `[Mandatory]` or `[Optional]`), ending with an example structure or code snippet.

Intinya: kalau kamu belum pernah denger istilah tertentu, habis baca bagian itu kamu harusnya sudah paham — nggak perlu buka Google lagi.

The bottom line: if you've never heard a term before, after reading that section you should get it — no need to Google again.

---

## Teknologi / Tech Stack

- **[MkDocs](https://www.mkdocs.org/)** — generator situs dokumentasi berbasis Markdown
- **[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)** — tema dengan dukungan dark mode, search, dan navigasi modern
- **[GitHub Pages](https://pages.github.com/)** — hosting publik gratis
- **[GitHub Actions](https://github.com/features/actions)** — auto-deploy saat push ke `main`

---

## Menjalankan Lokal / Running Locally

```bash
# Clone repo
git clone https://github.com/mabudiman/software-architecture-docs.git
cd software-architecture-docs

# Install dependencies
pip install -r requirements.txt

# Jalankan dev server
mkdocs serve
# Buka http://127.0.0.1:8000
```

### Build Statis / Static Build

```bash
mkdocs build
# Output di folder site/
```

---

## Struktur Repository / Repository Structure

```text
[root]
├── .github/
│   └── workflows/
│       └── deploy-docs.yml      ← GitHub Actions auto-deploy
├── docs/
│   ├── index.md                 ← Home page
│   ├── architecture/            ← 7 bab dokumentasi
│   └── stylesheets/extra.css    ← Custom badge & callout styling
├── mkdocs.yml                   ← Konfigurasi site
├── requirements.txt             ← Python dependencies
└── README.md                    ← (Anda di sini)
```

---

## Kontribusi / Contributing

Dokumentasi ini hidup — terbuka untuk perbaikan, klarifikasi, dan tambahan contoh:

This documentation is alive — open to fixes, clarifications, and added examples:

1. Fork repository ini.
2. Buat branch baru: `git checkout -b improve/topic-name`.
3. Commit perubahan dengan pesan jelas.
4. Buka *Pull Request* dengan deskripsi singkat *apa* yang berubah dan *kenapa*.

Untuk perbaikan tipo / kalimat kecil, langsung kirim PR. Untuk perubahan besar (menambah topik baru, mengubah struktur), buka *Issue* dulu untuk diskusi.

---

## Lisensi / License

[MIT](LICENSE) — silakan gunakan, modifikasi, dan distribusikan secara bebas.
