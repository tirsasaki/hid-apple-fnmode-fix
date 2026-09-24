[English](README.md) | **Bahasa Indonesia** | [日本語](README.ja.md)

# Memperbaiki Mapping Tombol F-Key di Linux (`hid_apple`)

Beberapa keyboard pihak ketiga, termasuk Rexus M84X dan keyboard 75% lainnya, menggunakan firmware yang kompatibel dengan Apple. Di Linux, hal ini dapat menyebabkan F1–F12 berfungsi sebagai tombol multimedia, bukan function key standar.

Panduan ini mengatur modul kernel `hid_apple` agar F1–F12 berfungsi sebagai function key standar secara default.

<!-- ADDED: Daftar isi untuk memudahkan navigasi. -->
## Daftar isi

- [Gejala](#gejala)
- [Penyebab](#penyebab)
- [Periksa status saat ini](#periksa-status-saat-ini)
- [Coba sementara](#coba-sementara)
- [Buat perubahan permanen](#buat-perubahan-permanen)
- [Verifikasi hasil](#verifikasi-hasil)
- [Pemecahan masalah](#pemecahan-masalah)
- [Telah diuji pada](#telah-diuji-pada)
- [Catatan](#catatan)
- [Lisensi dan kontribusi](#lisensi-dan-kontribusi)

## Gejala

- Menekan F4 malah membuka browser, bukan menghasilkan input F4.
- F1–F12 menghasilkan aksi media atau fungsi khusus, bukan input function key standar.
- Masalah terjadi pada beberapa keyboard non-Apple yang menggunakan firmware kompatibel Apple.

## Penyebab

Linux menangani keyboard tersebut melalui modul kernel `hid_apple`. Parameter `fnmode` mengatur perilaku baris function key.

| Nilai | Perilaku |
| --- | --- |
| `0` | Menonaktifkan F-key sepenuhnya; hanya fungsi media yang tersedia |
| `1` | F-key aktif hanya saat tombol `Fn` ditekan |
| `2` | F-key aktif secara default—perilaku yang diinginkan dalam panduan ini |

## Periksa status saat ini

Jalankan perintah berikut untuk memeriksa apakah `hid_apple` aktif:

```bash
lsmod | grep hid_apple
dmesg | grep -i apple
```

Jika modul aktif, lanjutkan ke bagian berikutnya.

## Coba sementara

Untuk langsung mencoba perilakunya tanpa reboot, muat ulang modul dengan `fnmode=2`:

```bash
sudo rmmod hid_apple
sudo modprobe hid_apple fnmode=2
```

> [!NOTE]
> Perubahan ini bersifat sementara. Setelah reboot, konfigurasi dalam `/etc/modprobe.d/hid_apple.conf`, setelah file tersebut dibuat, akan berlaku.

Uji F4 atau function key lainnya sebelum melanjutkan ke pengaturan permanen.

## Buat perubahan permanen

### 1. Buat file konfigurasi

Buka `/etc/modprobe.d/hid_apple.conf`:

```bash
sudo nano /etc/modprobe.d/hid_apple.conf
```

Tambahkan baris berikut:

```ini
options hid_apple fnmode=2
```

Simpan file, lalu tutup editor.

### 2. Bangun ulang initramfs dan reboot

Gunakan perintah yang sesuai dengan distribusi Anda.

#### Debian, Ubuntu, atau Linux Mint

```bash
sudo update-initramfs -u
sudo reboot
```

#### Fedora, RHEL, atau CentOS Stream

```bash
sudo dracut --force
sudo reboot
```

#### Arch Linux, Manjaro, atau EndeavourOS

```bash
sudo mkinitcpio -P
sudo reboot
```

## Verifikasi hasil

Setelah reboot, tekan F4 dan pastikan tombol tersebut menghasilkan input F4, bukan membuka browser.

Anda juga dapat memeriksa nilai `fnmode` yang aktif:

```bash
cat /sys/module/hid_apple/parameters/fnmode
```

Output yang diharapkan:

```text
2
```

<!-- ADDED: Panduan pemecahan masalah disusun dari langkah konfigurasi permanen dan verifikasi yang sudah ada; tidak ada metode perbaikan baru yang ditambahkan. -->
## Pemecahan masalah

### `fnmode` bukan `2` setelah reboot

- **Masalah:** Perintah verifikasi tidak menghasilkan `2`.
- **Penyebab:** Konfigurasi permanen modul belum diterapkan.
- **Perbaikan:** Pastikan `/etc/modprobe.d/hid_apple.conf` berisi `options hid_apple fnmode=2`. Kemudian bangun ulang initramfs menggunakan perintah yang sesuai dengan distribusi Anda dan lakukan reboot kembali.

## Telah diuji pada

| Distribusi | Kernel | Keyboard |
| --- | --- | --- |
| Debian 13 | `<isi di sini>` | `<isi di sini>` |
| Fedora 44 | `<isi di sini>` | `<isi di sini>` |
| Arch Linux | `<isi di sini>` | `<isi di sini>` |

## Catatan

> [!TIP]
> `/etc/modprobe.d/hid_apple.conf` akan hilang jika sistem operasi diinstal ulang. Sebaiknya buat cadangan file ini.

> [!WARNING]
> Konfigurasi ini berlaku secara global untuk setiap keyboard yang menggunakan modul `hid_apple`.

<!-- ADDED: README asli tidak menetapkan lisensi atau proses kontribusi. -->
## Lisensi dan kontribusi

Repository ini belum menetapkan lisensi. Tambahkan file `LICENSE` untuk menjelaskan bagaimana orang lain boleh menggunakan, memodifikasi, dan mendistribusikan proyek ini.

Kontribusi dapat diajukan melalui GitHub Issues atau pull request.
