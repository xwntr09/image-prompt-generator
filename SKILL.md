---
name: image-prompt-architect
description: Membuat, mengekstrak, merekonstruksi, dan merevisi prompt image generator dari gambar referensi atau instruksi teks. Gunakan saat pengguna meminta prompt JSON photorealistic, identity lock avatar, transfer pose/outfit/background, analisis angle/lighting/kamera, negative prompt, atau konversi prompt ke Leonardo AI. Skill ini menyusun prompt, bukan menghasilkan gambar.
---

# Image Prompt Architect

## Tujuan dan batasan

Hasilkan prompt gambar yang spesifik, visual, akurat terhadap referensi, dan konsisten dengan identitas avatar. Secara default hasilkan **JSON valid** dalam bahasa Inggris untuk isi prompt, dengan penjelasan singkat dalam bahasa pengguna hanya jika dibutuhkan. Jangan membuat gambar; bila pengguna meminta gambar jadi, serahkan ke kemampuan image generation yang tersedia di lingkungan tersebut.

Jangan menyatakan teks prompt dapat menjamin kesamaan wajah 100%; kestabilan identitas bergantung pada engine dan input referensi. Token `/preserve-face`, `/same-person`, `/facial-features-locked`, dan `/photorealistic` adalah petunjuk dalam prompt, **bukan perintah universal resmi** seluruh generator.

## Trigger / kapan digunakan

Gunakan untuk: "extract prompt from this image", "buat prompt JSON", "recreate image 1 pakai avatar image 2", "ganti outfit dari image 3", "edit pose/ekspresi/angle", "buat prompt Leonardo AI", atau "buat detail camera, lighting, negative prompt".

Jangan gunakan untuk: identifikasi nama orang nyata pada foto, ekstraksi data EXIF faktual yang tidak tersedia, atau menghasilkan file gambar langsung.

## Sumber masukan dan urutan prioritas

1. **Instruksi terbaru pengguna**: permintaan perubahan dan batasannya.
2. **Gambar avatar/identity reference**: sumber wajah dan karakter yang harus dipertahankan.
3. **Pose/composition reference**: tubuh, gesture, expression, perspektif, framing; **bukan identitas wajah** jika avatar terpisah tersedia.
4. **Outfit reference**: pakaian, detail material, warna, aksesori yang dipilih.
5. **Background/location reference**: geometri ruang, latar, suasana.
6. **Default skill**: digunakan hanya ketika sumber di atas tidak menentukan sesuatu.

Jika user melabeli gambar, ikuti label tersebut. Jangan menganggap "image 1" selalu avatar. Jika ada satu gambar dan permintaan "extract", gunakan gambar itu sebagai objek analisis, bukan otomatis sebagai avatar. Jangan mengarang visual dari gambar yang tidak disediakan atau tidak dapat dibaca. Jika detailnya tidak jelas, gunakan deskripsi netral alih-alih mengklaim kepastian.

**Prioritas konflik**: perubahan eksplisit terbaru mengalahkan default skill dan batasan yang lebih lama untuk atribut terkait. Pertahankan identitas lain yang tidak diminta berubah. Jika perintah terbaru saling bertentangan, terapkan instruksi paling spesifik; jika tetap mustahil ditentukan, jangan membuat asumsi besar—jelaskan konflik secara singkat atau pilih interpretasi konservatif.

## Workflow wajib

### 1 — Parse request

Tentukan mode:
- `EXTRACT`: uraikan image referensi tanpa memindahkan identitas.
- `RECREATE`: ciptakan komposisi referensi dengan avatar utama.
- `EDIT`: ubah hanya atribut yang diminta, pertahankan lainnya.
- `COMBINE`: ambil avatar, pose, outfit, background dari sumber berlainan.
- `CONVERT`: ubah format JSON menjadi prompt plain-text untuk platform target tanpa merusak detail utama.

Ekstrak daftar sumber, perubahan eksplisit, elemen wajib dipertahankan, aspek rasio, dan platform target bila disebut.

### 2 — Map references

Petakan setiap sumber ke salah satu peran: `avatar`, `pose`, `outfit`, `background`, `style/lighting`. Pisahkan **fakta yang terlihat** dari **pilihan kreatif**. Jangan mentransfer warna mata, wajah, warna kulit, atau tato model pose bila avatar berbeda. Jangan mengklaim mengetahui detail yang tertutup atau tidak terlihat.

### 3 — Lock avatar identity

Jika ada avatar utama, gunakan ketentuan berikut sebagai baseline yang dapat ditimpa instruksi terbaru:
- Adult woman; age appearance approximately 35 years old; height **180 cm**.
- Tall, curvy hourglass silhouette; long legs; established body proportions.
- Pertahankan wajah dari gambar avatar: face shape, eye shape/color, eyebrows, nose, lips, skin tone, unique natural features.
- Rambut, makeup, outfit, ekspresi, pose, dan aksesori boleh berubah **jika diminta**.
- **Warna mata dan bentuk mata tidak boleh diganti tanpa permintaan eksplisit.** Jika pengguna meminta silver eyes atau phoenix eyes, terapkan perubahan hanya pada atribut tersebut dan pertahankan wajah lainnya.
- Pertahankan skin tone asli avatar, bukan tone kulit model di pose reference. Jangan memaksakan deskripsi baseline kulit yang bertentangan dengan avatar yang terlihat.

Tulis instruksi identitas inti berikut saat ada avatar:

`/preserve-face /same-person /facial-features-locked /photorealistic`

`Use the exact same facial identity as the avatar reference. Preserve all facial features not explicitly requested to change. Never copy the pose-reference person's face.`

Jika tidak ada avatar atau identitas sebelumnya yang benar-benar tersedia, jangan berpura-pura dapat menjamin wajah yang sama; gunakan deskripsi karakter yang tersedia dan beri batasan secukupnya.

### 4 — Extract subject and scene

Catat:
- **Makeup style**: intensitas, complexion, eyes, brows, lips, skin finish.
- **Expression**: arah tatapan, bentuk senyum, karakter tatapan, ketegangan wajah.
- **Pose**: orientasi badan, letak tangan/kaki, distribusi berat, head tilt, torso tilt, interaksi dengan objek.
- **Outfits**: potongan, bahan, warna, panjang, fit, detail, aksesori.
- **Background**: ruang dan objek penting, kedalaman, waktu, suasana.
- **Photo style**: candid smartphone, selfie, editorial professional, studio, dll.
- **Color grading/vibe**: temperatur, saturasi, contrast, highlights, shadows.

Utamakan relasi spasial eksplisit (mis. "left elbow on railing, head turned back over right shoulder") daripada frasa generik ("elegant pose"). Jangan menambahkan orang, benda, tato, tahi lalat, kacamata, atau perhiasan yang tidak diminta.

### 5 — Determine lighting (rule engine)

Evaluasi dalam urutan ini:
1. Instruksi flash/lighting **eksplisit** dari pengguna mengalahkan default.
2. **Daylight**: default natural light, `flashlight_level: none`, kecuali user meminta flash.
3. **Night + smartphone FRONT camera / selfie**: default **NO FLASH**; manfaatkan ambient window/street/lamp light. Jika user khusus meminta flash, ikuti permintaannya.
4. **Night + smartphone BACK camera**: default **HARSH DIRECT FLASH**; `strong`. Namun jika ambient lighting jelas memadai atau user meminta no flash, gunakan no flash.
5. **Night + professional camera**: gunakan ambient/controlled light sesuai scene; direct on-camera flash hanya jika terlihat jelas atau diminta.
6. Jika waktu atau perangkat tak jelas, gambarkan efek pencahayaan yang terlihat; jangan mengarang flash.

Level flash: `none` (tanpa flash), `soft` (subtle fill), `medium` (noticeable direct flash), `strong` (harsh direct flash with hard shadows and specular highlights). Jangan mencampur `no flash` dengan `harsh direct flash` untuk foto yang sama.

### 6 — Determine camera and composition

Jelaskan camera type, viewpoint, camera angle, distance, framing/crop, character position, perspective, aspect ratio. Utamakan pilihan eksplisit atau framing referensi.

- **Smartphone selfie**: genuine front camera, typically ~24–28 mm equivalent; close arm's-length perspective; jangan mendeskripsikannya sebagai foto kamera profesional.
- **Smartphone back camera**: typically ~24–28 mm equivalent main camera; gunakan direct flash jika aturan langkah 5 terpenuhi.
- **Professional camera**: sebut full-frame atau APS-C hanya jika berguna; pilih mis. 35/50/85 mm sesuai field of view; shutter, ISO, aperture yang masuk akal.
- Angka ISO, shutter, aperture, focal length, dan jarak yang tidak tersedia secara faktual hanya **suggested generation settings**, bukan data EXIF hasil deteksi.
- Jika tinggi 180 cm perlu terlihat, gunakan full-body framing dengan perspektif wajar; foto close-up tak perlu mengarang tinggi yang tampak secara visual.

### 7 — Compose prompt

Gunakan **main `goal` ringkas** (ideal 45–110 kata) berisi: identitas/avatar, lokasi, pose, ekspresi, pakaian, komposisi, waktu, pencahayaan, gaya foto, dan kualitas. Hindari mengulang detail panjang yang sudah ada pada bagian lain.

Isi format sesuai [template JSON](./references/prompt-template.json). Default format:
- `main_prompt.commands`
- `main_prompt.goal`
- `main_prompt.rules`
- `main_prompt.detail_character.identity_lock`
- `main_prompt.detail_character.height`, `makeup_style`, `expression`, `pose`, `outfits`
- `main_prompt.detail_photo_style.photo_style`, `background`, `lighting`, `flashlight_level`, `colour_grading`, `vibe`
- `main_prompt.detail_camera.camera_type`, `camera_angle`, `camera_settings` (`frame_and_lens`, `ISO`, `shutter_speed`, `aperture`), `distance_between_subject_and_camera`, `character_position_on_frame`, `aspect_ratio`
- `main_prompt.negative_prompt`

Gunakan teks dalam bahasa Inggris pada kolom prompt untuk kompatibilitas luas. Isi key stabil berformat `snake_case`. Hindari nilai kabur seperti "perfect lighting", "beautiful pose", "cinematic camera" tanpa detail.

### 8 — Generate negative prompt

Buat negative prompt yang relevan, misalnya: `different facial identity, face swap artifacts, incorrect eye color, inconsistent proportions, deformed hands, extra fingers, duplicate limbs, plastic skin, excessive retouching, oversharpening, cartoon look, watermark, text`.

Tambahkan larangan spesifik sesuai user: `no glasses`, `no earrings`, `no mole`, `no red cast`, dll. **Jangan** memasukkan unsur negatif yang melarang fitur wajib yang justru diminta.

### 9 — Run QA before sending

Periksa:
- `goal` sesuai permintaan terbaru dan seluruh gambar punya peran jelas.
- Pose/model referensi tidak menggantikan wajah avatar.
- Identitas dan perubahan eksplisit tidak bertabrakan (khususnya mata dan warna kulit).
- Lighting cocok dengan siang/malam dan front/back/pro camera.
- Camera angle, crop, aspect ratio, jarak, dan bidang pandang tidak bertentangan.
- Tidak menyajikan setting sintetis sebagai EXIF faktual.
- JSON dapat diparse: double quotes, tanpa komentar/trailing commas/Markdown di dalam JSON.
- Negative prompt tidak meniadakan fitur yang diwajibkan.
- Prompt photorealistic: realistic skin texture, anatomical plausibility, natural lens perspective.

Jika gagal, perbaiki sendiri sebelum memberi respons.

## Output contracts

**Default — JSON:** satu blok kode `json` berisi satu objek JSON valid sesuai template. Jangan menambahkan penjelasan panjang bila pengguna hanya meminta prompt.

**Jika user minta "Leonardo AI" atau platform plain-text:** berikan `Main Prompt` natural-language dan `Negative Prompt` terpisah; pertahankan identity reference dan detail kamera penting. Jangan mengklaim slash commands dikenali secara native oleh platform.

**Jika user minta edit kecil:** ubah hanya atribut yang diminta, jangan mengganti pose, pakaian, background, lighting atau wajah yang sudah benar.

**Jika user meminta beberapa variasi:** beri 2–3 versi yang berbeda pada atribut yang diminta saja. Jangan mencampur variasi dalam satu JSON ambigu.

## Contoh skenario uji

Lihat [test cases](./references/test-cases.md) saat menguji perubahan skill. Gunakan paling sedikit skenario selfie malam tanpa flash, foto back camera malam dengan harsh direct flash, dan portrait siang tanpa flash. Selalu uji permintaan perubahan mata yang eksplisit serta transfer pose/outfit tanpa transfer wajah.
