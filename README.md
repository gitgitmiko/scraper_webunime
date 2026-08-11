# Scraper WEBUNIME

GitHub Actions yang menjalankan sync katalog **WEBUNIME** (LK21 + film Indonesia kconaz + Samehadaku + jadwal rilis) sekali sehari, lalu push JSON ke repo [gitgitmiko/WEBUNIME](https://github.com/gitgitmiko/WEBUNIME) — TV (`public/data/`) dan HP (`public/data/mobile/`).

Website dan app Android TV membaca data dari:

`https://raw.githubusercontent.com/gitgitmiko/WEBUNIME/main/public/data/`

Script scrape tetap di repo WEBUNIME (`npm run sync:catalog`). Repo ini hanya berisi workflow.

Output Samehadaku tambahan:

- `anime-latest.json`, `anime.json`, `anime-movies.json`
- `anime-schedule.json` — jadwal rilis Senin–Minggu dari [samehadaku jadwal-rilis](https://v2.samehadaku.how/jadwal-rilis/) (uji manual: di WEBUNIME `npm run scrape:anime-schedule`)

## Yang perlu disiapkan (manual)

### 1. Buat repo GitHub untuk scraper

Contoh: `https://github.com/gitgitmiko/scraper_webunime` (kosong / tanpa README otomatis juga boleh).

### 2. Buat Personal Access Token (fine-grained)

1. Buka [GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/personal-access-tokens)
2. **Generate new token**
3. Token name: `webunime-scraper`
4. Expiration: sesuai kebutuhan (mis. 90 hari / 1 tahun)
5. Resource owner: akunmu
6. Repository access: **Only select repositories** → pilih **WEBUNIME**
7. Permissions → Repository permissions:
   - **Contents**: Read and write
8. Generate token → **salin token** (hanya tampil sekali)

### 3. Tambah secret di repo scraper

1. Buka repo scraper di GitHub
2. **Settings → Secrets and variables → Actions → New repository secret**
3. Name: `WEBUNIME_TOKEN`
4. Value: PAT dari langkah 2 → **Add secret**

Tanpa secret ini, workflow gagal saat checkout/push ke WEBUNIME.

### 4. Push project ini ke GitHub

```bash
cd "C:\Users\sjatm\OneDrive\Documents\Project\Scraper WEBUNIME"
git init -b main
git add .
git commit -m "Initial commit: GitHub Actions catalog sync"
git remote add origin https://github.com/gitgitmiko/scraper_webunime.git
git push -u origin main
```

(Ganti URL remote jika nama reponya berbeda.)

## Cara uji (workflow_dispatch)

1. Buka repo scraper → tab **Actions**
2. Pilih workflow **Sync WEBUNIME Catalog**
3. **Run workflow** → branch `main` → **Run workflow**
4. Tunggu sampai hijau
5. Cek [WEBUNIME commits](https://github.com/gitgitmiko/WEBUNIME/commits/main) — harus ada commit `chore(data): sync katalog…` jika ada data baru
6. Buka / restart app TV untuk menarik JSON terbaru

## Jadwal otomatis

| Trigger | Waktu |
|--------|--------|
| Cron | `0 17 * * *` UTC ≈ **00:00 WIB** setiap hari |
| Manual | Actions → Run workflow |

## Alur kerja

1. Checkout `gitgitmiko/WEBUNIME` (branch `main`) pakai `WEBUNIME_TOKEN`
2. `npm ci` + install Playwright Chromium
3. `npm run sync:catalog` (LK21 film/series/horror + Samehadaku terbaru/movie/jadwal → TV + mobile)
4. Commit & push perubahan di `public/data/` termasuk `public/data/mobile/` (skip jika tidak ada perubahan)

## Troubleshooting

| Gejala | Kemungkinan | Tindakan |
|--------|-------------|----------|
| `WEBUNIME_TOKEN` / auth error | Secret belum diisi / PAT salah scope | Cek secret + Contents: Read and write di WEBUNIME |
| Samehadaku timeout / Cloudflare | IP runner GitHub diblokir | Cek log Actions; LK21 biasanya tetap jalan. Sync anime lokal: di WEBUNIME jalankan `npm run sync:catalog` |
| Timeout job | Scrape lama | Default timeout 90 menit; jalankan ulang manual |
| Tidak ada commit baru | Katalog sudah up-to-date | Normal — log akan bilang *No catalog changes* |

## Catatan

- Jangan commit PAT / `.env` ke git.
- Menit Actions dihitung dari akun GitHub Free (~2000 menit/bulan); 1× sehari biasanya cukup.
