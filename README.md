# Santanic Script Hub

Website katalog script dengan dark mode developer. Frontend di-host di GitHub Pages; metadata, autentikasi, dan file menggunakan Supabase Free. Pengunjung bisa mencari, melihat detail, menyalin link, dan mengunduh file. Hanya akun yang memiliki `app_metadata.role = admin` yang boleh mengunggah.

## Struktur

```text
santanic-script-hub/
├── index.html       # halaman website
├── style.css        # desain responsif
├── app.js           # katalog, pencarian, auth, upload, download
├── config.js        # Supabase URL + public anon/publishable key
├── supabase.sql     # tabel, RLS, storage policies, RPC download count
└── README.md
```

## 1. Buat proyek Supabase

1. Buat proyek di https://supabase.com/ dan simpan password database di tempat aman.
2. Buka **SQL Editor**, tempel seluruh isi `supabase.sql`, lalu jalankan.
3. Buka **Project Settings → API** (atau bagian API Keys di dashboard terbaru). Salin **Project URL** dan **anon/public key** atau **publishable key**. Jangan pernah menggunakan `service_role` atau secret key di browser.
4. Buka `config.js` dan ganti placeholder:

```js
window.SANTANIC_CONFIG = {
  SUPABASE_URL: "https://PROJECT-REF.supabase.co",
  SUPABASE_ANON_KEY: "PUBLIC_ANON_OR_PUBLISHABLE_KEY"
};
```

Public anon/publishable key memang dipakai frontend; keamanan data ditegakkan oleh Row Level Security (RLS) dan Storage policies, bukan dengan menyembunyikan key tersebut.

## 2. Buat akun admin dan beri peran admin

1. Di Supabase Dashboard, buka **Authentication → Users → Add user** dan buat user email/password milikmu. Jangan sediakan pendaftaran publik di website ini.
2. Buka SQL Editor. Ganti `ADMIN_USER_UUID` dengan UUID user yang tampil di daftar Authentication → Users, lalu jalankan:

```sql
update auth.users
set raw_app_meta_data = coalesce(raw_app_meta_data, '{}'::jsonb) || '{"role":"admin"}'::jsonb
where id = 'ADMIN_USER_UUID';
```

3. Keluar dan login ulang di website agar token baru memuat role admin.
4. Jangan menaruh service role key di `config.js`, GitHub, atau browser. Role harus berada di `app_metadata`, bukan `user_metadata`, karena user tidak boleh bisa menetapkan role admin sendiri.

## 3. Deploy ke GitHub Pages

1. Buat repository GitHub baru, misalnya `santanic-script-hub`.
2. Upload `index.html`, `style.css`, `app.js`, `config.js`, `supabase.sql`, dan `README.md` ke root repository.
3. Buka **Settings → Pages**. Pilih **Deploy from a branch**, branch `main`, folder `/ (root)`, lalu Save.
4. Tunggu proses deploy. Website akan tersedia di URL GitHub Pages yang ditampilkan GitHub.
5. Pastikan nilai Supabase di `config.js` sudah benar sebelum upload ke repository.

## 4. Menggunakan website

- Klik **Admin**, login menggunakan akun yang sudah diberi `app_metadata.role = admin`.
- Isi nama, deskripsi, kategori, bahasa/runtime, pilih file, lalu upload.
- Pengunjung dapat mencari script, memfilter kategori, membuka detail, menyalin link detail, dan mengunduh file.
- File maksimum 20 MB. Jenis executable umum diblokir oleh validasi frontend, tetapi validasi browser bukan pengamanan penuh. Upload hanya file yang memang layak dibagikan.

## Catatan keamanan dan operasional

- Bucket `scripts` dibuat **public** supaya pengunjung anonim dapat mengunduh. Artinya siapa pun yang mengetahui URL file dapat mengaksesnya; jangan pernah unggah file privat, token, password, konfigurasi rahasia, atau data pribadi.
- Supabase RLS dan Storage policies membatasi operasi upload dan perubahan metadata ke admin. Jangan menonaktifkan RLS.
- Supabase Free memiliki kuota dan ketentuan yang dapat berubah. Pantau penggunaan storage, bandwidth/egress, dan database di dashboard Supabase.
- Jangan mengunggah file berbahaya atau script yang melanggar hukum. Pengguna sebaiknya meninjau kode sebelum menjalankannya.
- Untuk menghapus file, saat ini hapus metadata/file melalui Supabase Dashboard (Storage → `scripts`, dan Table Editor → `scripts`). Jangan menghapus metadata saja jika file masih ada, atau sebaliknya.
- Hitungan download adalah perkiraan (bisa terpengaruh pemblokir browser, klik berulang, atau download yang dibatalkan), bukan analitik audit.
- Jika menggunakan Content Security Policy ketat, izinkan script dari `cdn.jsdelivr.net` dan koneksi ke domain proyek Supabase.


## Perbaikan download dan penghitung unduhan

- Frontend mengambil file dari Supabase Storage sebagai Blob lalu memicu unduhan dengan `download` memakai nama file asli yang diunggah (contoh `SC PUSHKONTAK.zip`), bukan nama path internal yang memakai UUID.
- Penghitung bertambah setelah file berhasil diambil dari Storage. Ini menghitung unduhan yang berhasil disiapkan browser, bukan bukti bahwa pengguna sudah membuka file.
- Setelah deploy file frontend terbaru, jika hitungan tetap 0, buka Supabase Dashboard → SQL Editor lalu jalankan ulang bagian `increment_script_downloads` di bagian paling bawah `supabase.sql`. Pastikan tidak ada error SQL.
- Setelah mengubah file `app.js`, deploy ulang seluruh isi folder proyek. Bersihkan cache / hard refresh jika versi lama masih tampil.
