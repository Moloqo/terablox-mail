# Terablox Mail

Webmail single-file untuk `@terablox.store`, di-host di GitHub Pages:
**https://moloqo.github.io/terablox-mail/**

Satu file `index.html` — buka langsung di browser, tanpa build, tanpa install.

## Dua halaman (bukan lagi satu layar pilih mode)

- **Halaman utama `/` — Tamu (publik):** pengunjung langsung dapat alamat
  sementara acak `tmp-xxxxxxxxxx@terablox.store` dan hanya bisa **baca**
  email yang masuk ke alamat itu saja. Tidak bisa mengirim, tidak bisa
  melihat inbox, alamat, atau pengaturan pemilik. Tidak ada lagi tombol
  login Pemilik di halaman ini. **Maksimal 3 alamat sementara** — alamat
  TIDAK dibuat di browser, tapi diminta ke Worker (`POST
  /api/public/create-address`) yang membuat + mencatatnya di Cloudflare KV
  per IP pengunjung (maks 3 per IP, selamanya). Alamat ke-2 dan ke-3 dibuat lewat
  tombol "Ganti alamat", alamat ke-3 ditolak server; kedua alamat bisa
  dipilih lewat chip.
- **`/admin` — Pemilik:** login HANYA dengan kata sandi Worker (alamat
  Worker sudah diatur di dalam file, jadi tidak perlu dimasukkan lagi)
  — file `admin.html`, salinan dari `index.html` dengan mode boot Pemilik.
  Akses penuh: semua alamat `@terablox.store`, kirim email,
  ganti alamat inbox ala temp-mail (chip alamat di atas Kotak Masuk),
  kelola alamat.

## Cara kerja

- Email dikirim/diterima lewat [Resend](https://resend.com) (free tier),
  domain `terablox.store` dengan MX catch-all — alamat apa pun langsung aktif.
- `index.html` bicara ke Cloudflare Worker (`worker.js`, tidak ada di repo
  ini — di-deploy terpisah di Cloudflare) yang meneruskan request ke API
  Resend. API key Resend dan kata sandi proxy hanya tersimpan sebagai secret
  di Worker, **tidak pernah** tertulis di file ini.
- Rute publik Worker (`/api/public/*`) hanya melayani alamat `tmp-` acak,
  hanya baca, dan difilter di server per alamat penerima. Kill-switch:
  set `PUBLIC_ENABLED=false` di Worker lalu Deploy.
- Alamat Worker publik diatur di dalam `index.html` (`PUBLIC_WORKER_URL`).
- **Pencatatan server (KV, wajib supaya batas 3 alamat ditegakkan server):**
  di Cloudflare dashboard → Workers & Pages → KV → Create a namespace
  (mis. `terablox-guest`), lalu di Settings Worker → Variables → KV
  Namespace Bindings → Add binding: Variable name `GUEST_KV`, pilih
  namespace tadi → Deploy. Tanpa binding ini, pembuatan alamat tetap jalan
  tapi tidak tercatat (yang berlaku hanya batas di browser). Catatan KV:
  key `guest-ip:<IP>` → `{addresses:[...], updated_at}`, tanpa TTL (selamanya).

## File lain

- `PANDUAN.txt` — panduan pemasangan dari nol (di folder terablox-mail-proxy,
  tidak ikut di-deploy ke Pages).
- `worker.js` — kode Cloudflare Worker v3 (di-deploy manual di Cloudflare).
