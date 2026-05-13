# Software Architecture Docs

[![Deploy Documentation](https://github.com/mabudiman/software-architecture-docs/actions/workflows/deploy-docs.yml/badge.svg)](https://github.com/mabudiman/software-architecture-docs/actions/workflows/deploy-docs.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Dokumentasi arsitektur perangkat lunak yang **mengajar** — bukan sekadar daftar aturan. Ditulis bilingual (Bahasa Indonesia + English) untuk programmer di semua level: dari junior yang baru mengenal istilah seperti *idempotency*, *Saga*, atau *Circuit Breaker*, hingga tech lead yang ingin meninjau ulang best practice.

A software architecture documentation that **teaches** — not just lists rules. Written bilingually (Indonesian + English) for programmers at every level.

---

## 🌐 Live Site

**👉 [https://mabudiman.github.io/software-architecture-docs/](https://mabudiman.github.io/software-architecture-docs/)**

---

## 📚 Daftar Bab / Table of Contents

| # | Bab / Chapter | Isi / Contents |
|---|---|---|
| 1 | [Foundation](https://mabudiman.github.io/software-architecture-docs/architecture/01-foundation/) | Struktur folder, Clean Architecture, Architecture Decision Record (ADR) |
| 2 | [Application Patterns](https://mabudiman.github.io/software-architecture-docs/architecture/02-application-patterns/) | Microservice, Async & Concurrency, Data Access, Cache, Service Bus, Idempotency |
| 3 | [Security](https://mabudiman.github.io/software-architecture-docs/architecture/03-security/) | Authentication & Authorization, Secret Management, Encryption, Secure Coding |
| 4 | [Quality & Reliability](https://mabudiman.github.io/software-architecture-docs/architecture/04-quality-reliability/) | Testing (Unit, Integration, E2E, BDD), Code Quality, Edge Case, Exception Handling |
| 5 | [Operations](https://mabudiman.github.io/software-architecture-docs/architecture/05-operations/) | Logging, Health Check, APM, Audit Trail |
| 6 | [User Interface](https://mabudiman.github.io/software-architecture-docs/architecture/06-user-interface/) | Component Library, Design System, Accessibility (a11y) |
| 7 | [Optional Features](https://mabudiman.github.io/software-architecture-docs/architecture/07-optional-features/) | Feature Flags |

---

## ✨ Apa yang Membuat Dokumentasi Ini Berbeda? / What Makes This Different?

Setiap topik mengikuti format yang sama agar mudah diikuti:

Each topic follows the same format for easy reading:

1. **Apa ini? / What is this?** — definisi sederhana + analogi
2. **Mengapa penting? / Why it matters?** — masalah yang dipecahkan
3. **Use Case** — cerita skenario konkret end-to-end
4. **Istilah & Konsep / Glossary** — penjelasan istilah teknis dengan bahasa awam
5. **Anti-pattern** (jika relevan) — contoh salah dan kenapa
6. **Aturan / Rules** — daftar aturan dengan label `[Mandatory]` atau `[Optional]`
7. **Contoh Struktur / Example** — folder tree atau snippet kode

> **Tujuan utama:** Programmer yang tadinya *tidak tahu* sebuah konsep, setelah membaca menjadi *paham* — tanpa harus googling istilah.
>
> **Main goal:** A programmer who *didn't know* a concept will *understand* it after reading — without needing to google terms.

---

## 🛠️ Teknologi / Tech Stack

- **[MkDocs](https://www.mkdocs.org/)** — generator situs dokumentasi berbasis Markdown
- **[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)** — tema dengan dukungan dark mode, search, dan navigasi modern
- **[GitHub Pages](https://pages.github.com/)** — hosting publik gratis
- **[GitHub Actions](https://github.com/features/actions)** — auto-deploy saat push ke `main`

---

## 🚀 Menjalankan Lokal / Running Locally

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

## 📁 Struktur Repository / Repository Structure

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

## 🤝 Kontribusi / Contributing

Dokumentasi ini hidup — terbuka untuk perbaikan, klarifikasi, dan tambahan contoh:

This documentation is alive — open to fixes, clarifications, and added examples:

1. Fork repository ini.
2. Buat branch baru: `git checkout -b improve/topic-name`.
3. Commit perubahan dengan pesan jelas.
4. Buka *Pull Request* dengan deskripsi singkat *apa* yang berubah dan *kenapa*.

Untuk perbaikan tipo / kalimat kecil, langsung kirim PR. Untuk perubahan besar (menambah topik baru, mengubah struktur), buka *Issue* dulu untuk diskusi.

---

## 📄 Lisensi / License

[MIT](LICENSE) — silakan gunakan, modifikasi, dan distribusikan secara bebas.
