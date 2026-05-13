# 7. Optional Features
### Fitur Opsional / Optional Features

Bab ini berisi fitur yang **tidak wajib** tapi sangat bermanfaat untuk tim yang menginginkan keleluasaan rilis & eksperimen.

This chapter covers features that are **not mandatory** but very useful for teams who want flexibility in releasing & experimenting.

---

## Feature Flag (Feature Toggle)

### Apa ini? / What is this?

**ID:** *Feature flag* (atau *feature toggle*) adalah saklar di kode yang menentukan apakah suatu fitur **aktif** atau **tidak aktif**. Bedanya dengan `if (config)` biasa: flag bisa diubah **saat aplikasi berjalan**, tanpa redeploy, dan bisa diatur per user/tenant/percentage traffic.

**EN:** A *feature flag* (or *feature toggle*) is a switch in the code that determines whether a feature is **on** or **off**. Unlike a regular `if (config)`, a flag can be changed **at runtime** without redeploy, and can target specific users / tenants / a percentage of traffic.

Analogi: seperti saklar lampu kamar. Tukang listrik (developer) memasang kabel & saklar, tapi keputusan kapan menyalakan ada di tangan penghuni (product manager / user) — tanpa perlu memanggil tukang listrik tiap nyala-mati.

### Mengapa penting? / Why it matters?

- **Pisahkan deploy dari release.** Kode baru bisa di-deploy ke production di belakang flag *off*, lalu di-aktifkan terpisah saat tim marketing siap.
- **A/B testing & gradual rollout.** Aktifkan fitur untuk 1% user dulu, monitor metrics. Jika oke, naikkan ke 10%, 50%, 100%.
- **Kill switch.** Saat fitur baru ternyata bermasalah, matikan flag dalam hitungan detik — tanpa rollback deploy yang lama.
- **Per-tenant rollout.** Customer A pakai fitur lama, Customer B pakai fitur baru — pengaturan beda.

### Use Case

> **Skenario A — Gradual rollout:** Tim Anda membuat algoritma checkout baru. Risiko: kalau ada bug, semua transaksi terganggu.
>
> Dengan feature flag: deploy kode baru → flag *off* untuk semua. Aktifkan untuk 1% user → monitor 24 jam. Tidak ada error → naikkan ke 10% → 50% → 100%. Bug ditemukan di 1%? Matikan flag, tidak perlu rollback.
>
> **Skenario B — Demo customer:** Customer enterprise C ingin trial fitur "Multi-warehouse". Tim Anda belum siap rilis untuk semua. Solusi: aktifkan flag hanya untuk tenant C. Customer lain tidak terdampak.
>
> **Skenario C — Trunk-based development:** Tim ingin merge kode setiap hari ke `main`, tapi fitur baru butuh 3 minggu selesai. Solusi: develop di belakang flag *off*. Kode ter-integrate terus, tapi user belum melihatnya.
{ .usecase }

### Istilah & Konsep / Glossary

- **Feature Flag / Toggle** — saklar runtime untuk fitur.
- **Release Toggle** — flag jangka pendek untuk pemisahan deploy vs release.
- **Experiment Toggle** — flag untuk A/B test (statistik membandingkan dua varian).
- **Ops Toggle** — flag operasional, mis. kill switch untuk matikan fitur saat insiden.
- **Permission Toggle** — flag jangka panjang per user/tenant (mis. "feature premium").
- **Gradual Rollout / Canary Release** — aktifkan ke porsi kecil user dulu sebelum semua.
- **Targeting** — aturan kapan flag aktif: per user ID, tenant, persentase traffic, atribut (negara, role).
- **Flag Owner** — penanggung jawab flag tertentu — tahu kapan boleh dihapus.
- **Flag Expiry** — tanggal kapan flag direncanakan dihapus (setelah fitur stabil & 100% aktif).
- **Flag Management Service** — sistem terpusat untuk kelola flag (mis. LaunchDarkly, Unleash, ConfigCat, atau buat sendiri).

### Anti-pattern

<div class="antipattern" markdown>
- ❌ **Flag yang menjadi permanen** — tidak pernah dihapus. Setelah 2 tahun, codebase penuh `if (flag.isEnabled)` yang membingungkan.
- ❌ **Flag tanpa owner** — tidak ada yang tahu boleh-tidaknya dihapus.
- ❌ **Pakai flag untuk konfigurasi statis** (mis. `if (flag.isProduction)`) — pakai config biasa, bukan feature flag system.
- ❌ **Logika flag tersebar di mana-mana** — sebaiknya disentralisasi di satu service / wrapper.
- ❌ **Flag berlapis-lapis** (`if A && B && !C`) → matriks state meledak, sulit di-test.
</div>

### Aturan / Rules

<span class="badge badge-optional">Optional</span> Pakai feature flag untuk memisahkan **deploy** dari **release**.

<span class="badge badge-optional">Optional</span> Flag dapat dikelola **saat runtime** tanpa redeploy.

<span class="badge badge-optional">Optional</span> Setiap flag punya **owner** & **rencana expiry** — flag bukan permanen.

<span class="badge badge-optional">Optional</span> Hapus flag setelah fitur **fully released & stable**.

<span class="badge badge-optional">Optional</span> Dukung **targeting**: per user, per tenant, per persentase traffic.

### Contoh Implementasi / Example

```python
# Tanpa feature flag (kaku, butuh redeploy untuk ubah perilaku)
def calculate_price(order):
    return new_pricing_algorithm(order)  # langsung pakai

# Dengan feature flag (fleksibel)
def calculate_price(order):
    if feature_flags.is_enabled("new-pricing-v2", user=order.user):
        return new_pricing_algorithm(order)
    return legacy_pricing_algorithm(order)
```

**Lifecycle flag yang sehat:**

```text
Phase 1: Flag dibuat → off untuk semua
Phase 2: Aktifkan untuk internal team   (1 hari)
Phase 3: Beta users (5%)                (3 hari)
Phase 4: 50% traffic                    (3 hari)
Phase 5: 100% traffic                   (1 minggu monitoring)
Phase 6: HAPUS flag dari kode + tutup tiket
```

---

## Ringkasan / Summary

Feature flag adalah alat ampuh untuk **mengendalikan release secara halus**, tapi punya biaya: kompleksitas codebase. Aturan emasnya — **selalu rencanakan tanggal hapus flag** sejak ia dibuat.

> **Selesai!** Anda sudah melewati semua bab. Jika ingin meninjau ulang istilah-istilah penting, buka kembali [Home](../index.md).
