# Test cases — Image Prompt Architect

Uji manual skill dengan prompt di bawah ini; pastikan keluaran mengikuti struktur pada `prompt-template.json`.

## T1 — Selfie malam
**Input**: "Buat ulang selfie depan jendela hotel pada malam hari, kamera depan smartphone, wajah avatar tetap, senyum kecil, rasio 4:5."
**Expected**: `camera_type` smartphone front; `flashlight_level` none; ambient hotel/city light; close selfie perspective; same avatar face; no invented strong flash.

## T2 — Smartphone back camera malam
**Input**: "Foto malam di tangga, diambil dari atas, back camera 3–4 meter, background gelap, full body, wajah avatar tetap."
**Expected**: strong harsh direct flash (unless explicitly adequate ambient); noticeable subject illumination and darker background; high angle; no face transfer.

## T3 — Daylight
**Input**: "Selfie di lapangan basket pantai siang, jersey putih dari image outfit reference, tangan di perut, high angle."
**Expected**: no flash; natural light; smartphone selfie; outfit transfer; high angle; coherent 9:16 or explicit user ratio.

## T4 — Explicit eye edit
**Input**: "Pakai avatar saya, ubah warna mata menjadi silver dan bentuknya phoenix eyes; identitas lain tetap sama."
**Expected**: apply explicit eye edits, preserve rest of avatar; negative prompt MUST NOT include silver eyes or phoenix eyes as undesired.

## T5 — Professional editorial flash
**Input**: "Photographer shot di lounge vintage, model avatar di depan mantel, kamera profesional 50 mm, strong harsh on-camera flash, cool muted grade."
**Expected**: professional camera; 50 mm; flash strong despite indoor/night default; grade cool/muted; reasonable recommended ISO/shutter/aperture labeled as suggestions.

## T6 — Small edit only
**Input**: "Hapus kacamata dan cahaya merah saja. Jangan ubah lainnya."
**Expected**: remove glasses and red color cast; preserve all other elements of prior prompt; include `no glasses, no red cast` only if consistent with request.

## T7 — JSON parseability
**Input**: "Extract prompt detail JSON dari gambar dan buat negative prompt."
**Expected**: one valid JSON object, English values, no trailing commas, no unescaped quotes, no pseudo-JSON commentary.
