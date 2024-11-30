# Tugas Akhir / Nilai UAS 
---
 
## Fitur Utama
### 1. **Fungsi Utama**
- `image_resize`: Mengubah ukuran gambar berdasarkan lebar atau tinggi yang diberikan.
- `add_text`: Menambahkan teks ke dalam gambar pada posisi tertentu dengan font, ukuran, warna, dan ketebalan yang diberikan.

### 2. **Argumen Masukan**
Menggunakan `argparse` untuk memproses argumen dari command line:
- `--video`: Jalur ke file video yang akan diproses. Jika tidak diberikan, akan menggunakan kamera.
- `--max_corners`: Jumlah maksimum sudut (corner) yang dapat dilacak. Default adalah **100**.

### 3. **Parameter Deteksi dan Pelacakan**
- **`feature_params`**  
  Parameter untuk mendeteksi fitur menggunakan `cv2.goodFeaturesToTrack`, seperti kualitas fitur, jarak antar fitur, dll.
- **`lk_params`**  
  Parameter untuk algoritma Lucas-Kanade Optical Flow, termasuk ukuran jendela, tingkat piramida maksimum, dan kriteria terminasi.
- **`termination`**  
  Kriteria untuk menghentikan iterasi Lucas-Kanade (bisa berbasis jumlah iterasi atau nilai epsilon).

---

## Proses Utama (While Loop)

### 1. **Membaca Frame dari Kamera atau Video**
- Membaca frame dari video atau kamera.
- Jika frame tidak dapat dibaca, menghasilkan error.
- Ukuran frame diubah menggunakan `image_resize` agar lebih mudah ditampilkan.

### 2. **Konversi Frame ke Grayscale**
- Konversi frame ke grayscale diperlukan karena Optical Flow hanya bekerja dengan gambar hitam-putih.

### 3. **Pelacakan Optical Flow**
Tahapan proses pelacakan:
- **Tahap 1:** Jika ada jalur pelacakan (*tracks*), algoritma Lucas-Kanade menghitung pergerakan fitur dari frame sebelumnya (`prev_gray_frame`) ke frame saat ini (`curr_gray_frame`).
- **Tahap 2:** Melakukan validasi menggunakan metode *forward-backward consistency check* untuk memastikan fitur valid.
- **Tahap 3:** Memperbarui jalur pelacakan (*tracks*) dan menggambar lintasan pada frame output.

### 4. **Penambahan Fitur Baru**
Setiap 100 frame, algoritma mendeteksi ulang fitur baru menggunakan `cv2.goodFeaturesToTrack`, memastikan pelacakan tetap optimal.

### 5. **Tampilan Output**
Menampilkan berbagai hasil:
- Frame asli.
- Frame grayscale.
- Masking fitur yang ditandai.
- Frame output dengan lintasan pelacakan.

### 6. **Kontrol Keyboard**
- Tekan `q` untuk keluar.
- Tekan `r` untuk mereset semua lintasan pelacakan.

---
Bagian kode yang berfungsi untuk **mendeteksi pergerakan** yang menggunakan algoritma **Lucas-Kanade** adalah sebagai berikut : 
## 1. Deteksi Optical Flow (Pergerakan Fitur)
Menggunakan `cv2.calcOpticalFlowPyrLK` untuk menghitung pergerakan fitur antar frame.

```python
p1, state, err = cv2.calcOpticalFlowPyrLK(prev_gray_frame, curr_gray_frame, p0, None, **lk_params)
p0r, state, err = cv2.calcOpticalFlowPyrLK(curr_gray_frame, prev_gray_frame, p1, None, **lk_params)
```

### Input
- **`prev_gray_frame`**: Frame sebelumnya dalam format grayscale.
- **`curr_gray_frame`**: Frame saat ini dalam format grayscale.
- **`p0`**: Koordinat fitur yang dilacak pada frame sebelumnya.
### Output
- **`p1`**: Lokasi fitur pada frame saat ini.
- **`state`**: Status pelacakan (berhasil atau tidak) untuk setiap fitur.
- **`err`**: Kesalahan estimasi untuk setiap fitur.

## 2. Memperbarui dan Menggambar Lintasan
Menambahkan koordinat terbaru fitur ke lintasan dan menggambar lintasan tersebut.

```python
new_tracks = []
for tr, (x, y), good_flag in zip(tracks, p1.reshape(-1, 2), good):
    if not good_flag:
        continue
    tr.append((x, y))
    if len(tr) > track_length:
        del tr[0]
    new_tracks.append(tr)
tracks = new_tracks
cv2.polylines(output, [np.int32(tr) for tr in tracks], False, [0, 0, 255], 2)
```

### Update lintasan fitur
- Menambahkan koordinat terbaru (x, y) ke lintasan tr jika fitur valid.
- Menghapus koordinat lama jika panjang lintasan melebihi batas (track_length).
- Menyimpan lintasan fitur valid ke dalam new_tracks.
### Gambar lintasan
- Menggunakan `cv2.polylines` untuk menggambar lintasan fitur pada frame output menggunakan koordinat dalam `tracks`.

## 3. Deteksi Fitur Baru
Setiap 100 frame, fitur baru dideteksi menggunakan Shi-Tomasi Corner Detection `cv2.goodFeaturesToTrack`

```python
mask = 255 * np.ones(curr_gray_frame.shape, dtype=np.uint8)
for x, y in [np.int32(tr[-1]) for tr in tracks]:
    cv2.circle(mask, (x, y), 5, 0, -1)
p = cv2.goodFeaturesToTrack(prev_gray_frame, mask=None, **feature_params)
```

### Masking
- Area dekat fitur yang sudah dilacak dihindari agar fitur baru tidak terdeteksi terlalu dekat dengan yang lama.
---
**Gambar Hasil Percobaan :**
