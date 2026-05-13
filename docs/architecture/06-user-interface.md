# 6. User Interface
### Standar Antarmuka Pengguna / User Interface Standard

Antarmuka adalah wajah aplikasi — bagian yang langsung dirasakan pengguna. Tanpa standar, UI cepat berantakan: setiap halaman terlihat berbeda, komponen serupa di-implement ulang dengan bug masing-masing, dan pengguna dengan disabilitas tidak bisa memakai aplikasi.

The user interface is the face of an application — what users directly experience. Without standards, a UI quickly becomes chaotic: pages look different from each other, similar components are reimplemented with their own bugs, and users with disabilities can't use the app.

---

## Komponen UI Standar / Standard UI Component

### Apa ini? / What is this?

**ID:** Sekumpulan komponen UI (tombol, input, modal, table, dll.) yang sudah dibangun, di-test, dan dirilis sebagai library bersama. Semua aplikasi/halaman wajib pakai komponen dari library ini, bukan bikin sendiri.

**EN:** A set of UI components (buttons, inputs, modals, tables, etc.) that are built, tested, and shipped as a shared library. Every app/page must use components from this library rather than rolling their own.

### Mengapa penting? / Why it matters?

- **Konsistensi visual** — tombol "Submit" di mana-mana terlihat sama.
- **Quality satu kali, dipakai semua** — fix bug di library → semua halaman dapat manfaat.
- **Velocity** — tim fitur tidak menghabiskan waktu memikirkan padding & warna tombol.
- **Aksesibilitas built-in** — sekali komponen aksesibel, semua pakai = aksesibel.

### Use Case

> **Skenario:** Tanpa komponen library, 5 tim membuat 5 versi tombol "Bayar" mereka sendiri. Hasilnya: warna sedikit beda (#1976D2 vs #1A77D5), ukuran beda, beberapa pakai loading spinner beberapa tidak. Pengguna bingung; brand inkonsisten.
>
> Dengan library: semua import `<PrimaryButton loading={isSubmitting}>Bayar</PrimaryButton>` → identik di mana-mana.
{ .usecase }

### Istilah & Konsep / Glossary

- **Component Library** — kumpulan komponen reusable (mis. internal "Acme UI Kit", atau open source: MUI, Ant Design, Shadcn).
- **Internal Package Registry** — server private untuk publikasi package (npm Enterprise, JFrog Artifactory, GitHub Packages).
- **Versioning** — library dirilis dengan versi (mis. v2.3.1) — aplikasi pin versi untuk stabilitas.
- **Migration Guide** — petunjuk pindah dari komponen lama ke yang baru, biasanya saat satu versi di-*deprecated*.
- **Deprecated** — masih ada, tapi tidak disarankan dipakai & akan dihapus di versi mendatang.

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Semua UI pakai komponen dari component library internal yang disetujui — JANGAN komponen custom one-off tanpa approval.

<span class="badge badge-mandatory">Mandatory</span> Component library diberi versi & dipublikasi ke internal package registry.

<span class="badge badge-mandatory">Mandatory</span> Komponen baru melalui design review sebelum masuk library.

<span class="badge badge-mandatory">Mandatory</span> Komponen yang di-deprecated diberi flag di library dengan migration guide ke pengganti.

<span class="badge badge-mandatory">Mandatory</span> Komponen di-test untuk: rendering, interaction state, responsiveness, accessibility.

---

## Design System

### Apa ini? / What is this?

**ID:** *Design system* lebih luas dari sekadar komponen — ia adalah keseluruhan sistem yang mengatur tampilan: warna, tipografi, spacing, ikonografi, motion, plus dokumentasi penggunaan & filosofi desain. Komponen UI adalah salah satu *output* design system.

**EN:** A *design system* is broader than just components — it's the entire system governing appearance: colors, typography, spacing, iconography, motion, plus usage documentation and design philosophy. UI components are one *output* of a design system.

### Mengapa penting? / Why it matters?

- **Single source of truth** untuk visual brand.
- **Designer & developer berbicara bahasa yang sama** lewat *design tokens*.
- Mengganti tema (mis. dukung dark mode, rebranding) jadi mudah jika token terdefinisi.

### Use Case

> **Skenario:** Brand memutuskan warna primer pindah dari biru ke hijau. Tanpa design system: 200 file CSS perlu diubah manual → minggu pekerjaan, kemungkinan miss.
>
> Dengan design system + design tokens: ubah `--color-primary: green;` di satu file → 200 halaman ter-update otomatis.
{ .usecase }

### Istilah & Konsep / Glossary

- **Design Tokens** — variabel desain bernama (mis. `color.primary.500`, `spacing.md`, `font.heading.lg`). Bukan nilai literal yang berserakan.
- **Storybook** — tool untuk men-*showcase* komponen di luar aplikasi, dengan contoh interaktif. Berfungsi sebagai "katalog komponen".
- **Light / Dark Mode** — dua skema warna; aplikasi modern wajib dukung keduanya.
- **Breakpoint** — titik di mana layout berubah (mis. mobile < 768px, tablet 768-1024px, desktop > 1024px).
- **Responsive Design** — UI yang menyesuaikan layout sesuai ukuran layar.
- **CSS Variable / Custom Property** — variabel di CSS (`--color-primary: #1976D2`) yang bisa diubah runtime.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ Warna hex hardcoded di tiap komponen: `color: #1976D2;` di mana-mana.
- ❌ `margin-top: 13px` — kenapa 13? Pakai token `spacing.sm` (8px) atau `spacing.md` (16px).
- ❌ Tidak ada Storybook → developer tidak tahu komponen yang ada → bikin sendiri.
</div>

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Semua komponen UI berasal dari design system yang disetujui — JANGAN ad-hoc styling.

<span class="badge badge-mandatory">Mandatory</span> Komponen didokumentasikan dengan contoh penggunaan di component library (mis. Storybook).

<span class="badge badge-mandatory">Mandatory</span> Design token (warna, spacing, tipografi) didefinisikan sebagai variabel — JANGAN nilai hardcoded.

<span class="badge badge-mandatory">Mandatory</span> Setiap komponen punya unit test untuk rendered output & interaction state.

<span class="badge badge-mandatory">Mandatory</span> Komponen mendukung light & dark mode.

<span class="badge badge-mandatory">Mandatory</span> Perilaku responsif didefinisikan per komponen untuk semua breakpoint yang didukung.

---

## Aksesibilitas / Accessibility (a11y)

### Apa ini? / What is this?

**ID:** Aksesibilitas (sering disingkat **a11y** — "a" + 11 huruf + "y") = praktik membuat aplikasi bisa dipakai semua orang, termasuk:

- Pengguna *screen reader* (tunanetra).
- Pengguna yang hanya bisa keyboard (tidak bisa mouse).
- Pengguna *color blind*.
- Pengguna dengan disabilitas motorik, kognitif, dll.

**EN:** Accessibility (often abbreviated **a11y**) = the practice of making apps usable by everyone, including users of screen readers, keyboard-only users, color blind users, and users with various disabilities.

### Mengapa penting? / Why it matters?

- **Etis & inklusif** — sebagian populasi punya disabilitas; aplikasi tidak boleh menutup pintu.
- **Legal** — banyak negara mewajibkan aksesibilitas untuk situs publik / pemerintah.
- **Bisnis** — jangkauan pengguna lebih luas.
- **SEO & UX umum** — banyak praktik a11y meningkatkan UX untuk semua orang (mis. label yang jelas).

### Use Case

> **Skenario:** Form login Anda tidak punya `<label>` untuk input email. Pengguna screen reader mendengar: "edit text, edit text" — tidak tahu mana untuk email, mana untuk password. Tidak bisa login.
>
> Dengan label eksplisit: screen reader berkata "Email, edit text. Password, edit text" → bisa diisi dengan benar.
{ .usecase }

### Istilah & Konsep / Glossary

- **WCAG (Web Content Accessibility Guidelines)** — standar a11y internasional dari W3C. Versi terbaru 2.1 / 2.2.
- **WCAG Level A / AA / AAA** — tingkat kepatuhan. **AA** = target umum aplikasi publik. **AAA** sangat ketat, biasanya hanya untuk konten khusus.
- **Screen Reader** — software yang membaca konten layar dengan suara: NVDA, JAWS, VoiceOver, TalkBack.
- **Semantic HTML** — pakai tag HTML sesuai maknanya: `<button>` untuk tombol (bukan `<div onClick>`), `<nav>`, `<main>`, `<article>`.
- **ARIA (Accessible Rich Internet Applications)** — atribut tambahan untuk memberi makna semantik di mana HTML asli tidak cukup (mis. `aria-label`, `role`).
- **alt text** — atribut `alt` di tag `<img>`. Berisi deskripsi gambar untuk pengguna yang tidak bisa melihat.
- **Decorative Image** — gambar yang hanya untuk hiasan (bukan informasi). Pakai `alt=""` agar screen reader skip.
- **Keyboard Navigation** — semua aksi (klik tombol, isi form, buka menu) dapat dilakukan dengan keyboard (Tab, Enter, Space, arrow keys).
- **Focus Indicator** — outline visual yang menunjukkan elemen mana yang sedang fokus keyboard.
- **Color Contrast Ratio** — perbandingan kecerahan teks vs background. WCAG AA: ≥ 4.5:1 untuk teks normal.
- **axe-core** — engine populer untuk automated a11y testing.

### Anti-pattern

<div class="antipattern" markdown>
- ❌ `<div onClick={...}>Klik</div>` — bukan keyboard-accessible. Pakai `<button>`.
- ❌ Warna sebagai satu-satunya indikator (mis. "field merah = wajib"). Tambahkan label/ikon.
- ❌ Input tanpa label, hanya placeholder.
- ❌ Image tanpa `alt` (atau `alt="image"`).
- ❌ Modal yang menjebak fokus tapi tidak bisa di-close dengan Esc.
- ❌ Teks abu muda di background putih (kontras 2:1 → tidak terbaca).
</div>

### Aturan / Rules

<span class="badge badge-mandatory">Mandatory</span> Semua UI memenuhi minimal WCAG 2.1 Level AA.

<span class="badge badge-mandatory">Mandatory</span> Setiap elemen interaktif dapat dinavigasi dengan keyboard.

<span class="badge badge-mandatory">Mandatory</span> Semua image punya `alt` deskriptif. Image dekoratif pakai `alt=""`.

<span class="badge badge-mandatory">Mandatory</span> Warna BUKAN satu-satunya cara menyampaikan informasi.

<span class="badge badge-mandatory">Mandatory</span> Form punya label eksplisit yang terhubung ke setiap input.

<span class="badge badge-mandatory">Mandatory</span> Automated a11y check (mis. axe-core) jalan sebagai bagian dari CI pipeline.

---

## Ringkasan / Summary

- **Component library** mencegah re-implementasi tombol & input yang sama berulang kali.
- **Design system + design tokens** = bahasa bersama designer & developer; mempermudah rebranding & dark mode.
- **Aksesibilitas** bukan opsional — wajib WCAG AA, kerjakan dari awal (lebih murah daripada retrofit).

> **Selanjutnya:** [7. Optional Features](07-optional-features.md) — fitur tambahan seperti feature flags.
