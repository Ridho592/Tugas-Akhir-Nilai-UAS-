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

### Input
- **`prev_gray_frame`**: Frame sebelumnya dalam format grayscale.
- **`curr_gray_frame`**: Frame saat ini dalam format grayscale.
- **`p0`**: Koordinat fitur yang dilacak pada frame sebelumnya.

### Output
- **`p1`**: Lokasi fitur pada frame saat ini.
- **`state`**: Status pelacakan (berhasil atau tidak) untuk setiap fitur.
- **`err`**: Kesalahan estimasi untuk setiap fitur.

## 2. Memperbarui dan Menggambar Lintasan
