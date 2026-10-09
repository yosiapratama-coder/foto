# Our Little Booth — Website Kenangan

Website galeri foto bergaya scrapbook untuk menyimpan kenangan photo booth berdua. Pengunjung tidak perlu login untuk melihat atau menambahkan foto.

## Penting: agar foto terlihat oleh semua orang
File HTML saja tidak bisa menyimpan unggahan secara bersama-sama. Website ini memakai **Firebase Storage + Cloud Firestore** sebagai penyimpanan online. Setelah konfigurasi Firebase dan hosting dilakukan, siapa pun yang memiliki tautan dapat melihat dan mengunggah foto tanpa login.

## 1. Buat proyek Firebase
1. Buka https://console.firebase.google.com/ lalu buat project baru.
2. Tambahkan aplikasi Web (`</>`), lalu salin konfigurasi Firebase.
3. Di **Build → Firestore Database**, buat database.
4. Di **Build → Storage**, siapkan bucket.
5. Salin `index.html` dan ganti nilai `firebaseConfig` dengan konfigurasi proyek Anda.

## 2. Aturan database dan storage
Untuk uji coba galeri publik, di Firestore Rules dapat digunakan aturan berikut:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /memories/{memoryId} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['url','caption','createdAt','storagePath'])
                    && request.resource.data.caption is string
                    && request.resource.data.caption.size() <= 120
                    && request.resource.data.url is string
                    && request.resource.data.storagePath is string;
      allow update, delete: if false;
    }
  }
}
```

Untuk Storage Rules, aturan demo yang mengizinkan pengunjung membaca dan mengunggah gambar hingga 8 MB:

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /memories/{fileName} {
      allow read: if true;
      allow write: if request.resource != null
                   && request.resource.size < 8 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
    }
  }
}
```

**Peringatan:** aturan ini sengaja membuka upload untuk publik, sehingga orang lain juga bisa mengunggah banyak gambar dan menambah biaya penyimpanan/bandwidth. Gunakan hanya untuk proyek yang memang ingin terbuka. Untuk situs publik jangka panjang, tambahkan App Check, pembatasan upload/rate limit melalui backend, moderasi, dan pemantauan kuota. Jangan simpan foto pribadi/sensitif. Pengunjung yang memiliki akses dapat menyimpan atau membagikan foto.

## 3. Jalankan dan publikasikan
Cara termudah:
- Untuk mencoba: buka folder melalui VS Code dan jalankan dengan ekstensi Live Server.
- Untuk online: unggah `index.html` ke Netlify Drop (https://app.netlify.com/drop) atau gunakan Firebase Hosting. Untuk Firebase Hosting, ikuti https://firebase.google.com/docs/hosting/quickstart.

## Cara pakai
- Klik **Tambahkan Foto** untuk memilih satu atau beberapa foto.
- Klik **Tulis Kenangan** untuk menyiapkan caption, lalu pilih foto.
- Foto maksimal 8 MB per file.
- Tanpa konfigurasi Firebase, website berjalan dalam mode pratinjau lokal: foto hanya terlihat pada perangkat tersebut. Setelah Firebase dikonfigurasi, unggahan tersimpan online dan galeri menjadi bersama.

## Privasi
Galeri ini bersifat publik. Jangan unggah foto orang lain tanpa izin. Jika ingin galeri privat, diperlukan autentikasi atau mekanisme akses lain.
