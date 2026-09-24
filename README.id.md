[English](README.md) | **Bahasa Indonesia** | [日本語](README.ja.md)

# Memperbaiki Mapping Tombol F-Key di Linux (`hid_apple`)

Beberapa keyboard, seperti Rexus M84X, menggunakan firmware yang kompatibel dengan Apple. Linux mendeteksinya sebagai keyboard Apple dan menanganinya dengan modul kernel `hid_apple`, sehingga F1–F12 secara default berfungsi sebagai tombol media, bukan function key standar.

Panduan ini mengatur parameter `fnmode` dari modul `hid_apple` agar F1–F12 secara default berfungsi sebagai function key standar.

## Daftar isi

- [Gejala](#gejala)
- [Penyebab](#penyebab)
- [Periksa status saat ini](#periksa-status-saat-ini)
- [Coba sementara](#coba-sementara)
- [Buat perubahan permanen](#buat-perubahan-permanen)
- [Verifikasi hasil](#verifikasi-hasil)
- [Pemecahan masalah](#pemecahan-masalah)
- [Catatan](#catatan)
- [Kontribusi](#kontribusi)

## Gejala

- Menekan F4 (atau F-key lainnya) memicu aksi media atau fungsi khusus, bukan menghasilkan input F4.
- F1–F12 hanya berfungsi sebagai function key standar saat `Fn` ditekan.
- Masalah terjadi pada beberapa keyboard non-Apple yang menggunakan firmware kompatibel Apple.

## Penyebab

Linux menangani keyboard tersebut melalui modul kernel `hid_apple`. Parameter `fnmode` mengatur perilaku baris function key.

| Nilai | Perilaku |
| --- | --- |
| `0` | `Fn` dinonaktifkan; F1–F12 selalu berfungsi sebagai function key standar dan fungsi media tidak tersedia |
| `1` | Tombol media secara default; F-key hanya saat `Fn` ditekan |
| `2` | F-key secara default; fungsi media saat `Fn` ditekan (perilaku yang digunakan dalam panduan ini) |

## Periksa status saat ini

Jalankan perintah berikut untuk memeriksa apakah `hid_apple` aktif:

```bash
lsmod | grep hid_apple
dmesg | grep -i apple
```

Jika `lsmod` menampilkan baris `hid_apple`, lanjutkan ke bagian berikutnya. Jika tidak menampilkan apa pun, keyboard Anda tidak ditangani oleh `hid_apple` dan panduan ini tidak berlaku.

## Coba sementara

Untuk langsung mencoba perilakunya tanpa reboot, muat ulang modul dengan `fnmode=2`:

```bash
sudo rmmod hid_apple
sudo modprobe hid_apple fnmode=2
```

> [!NOTE]
> Perubahan ini akan hilang setelah reboot kecuali Anda menyelesaikan pengaturan permanen di bawah ini.

> [!TIP]
> Memuat ulang modul dapat membuat keyboard tidak merespons untuk sesaat. Untuk menghindarinya, atur nilainya secara langsung tanpa memuat ulang modul:
>
> ```bash
> echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode
> ```

Uji F4 atau function key lainnya sebelum melanjutkan ke pengaturan permanen.

## Buat perubahan permanen

> [!WARNING]
> Konfigurasi ini berlaku secara global untuk setiap keyboard yang menggunakan modul `hid_apple`.

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

#### Arch Linux, Manjaro, EndeavourOS, atau CachyOS

```bash
sudo mkinitcpio -P
sudo reboot
```

## Verifikasi hasil

Setelah reboot, tekan F4 dan pastikan tombol tersebut menghasilkan input F4, bukan memicu aksi media atau fungsi khusus.

Anda juga dapat memeriksa nilai `fnmode` yang aktif:

```bash
cat /sys/module/hid_apple/parameters/fnmode
```

Output yang diharapkan:

```text
2
```

## Pemecahan masalah

### `fnmode` bukan `2` setelah reboot

- **Masalah:** Perintah verifikasi tidak menghasilkan `2`.
- **Penyebab:** Konfigurasi permanen modul belum diterapkan.
- **Perbaikan:** Pastikan `/etc/modprobe.d/hid_apple.conf` berisi `options hid_apple fnmode=2`. Kemudian bangun ulang initramfs menggunakan perintah yang sesuai dengan distribusi Anda dan lakukan reboot kembali.

### `fnmode` sudah `2`, tetapi F-key masih berperilaku tidak sesuai harapan

- **Masalah:** Nilainya sudah benar, tetapi baris F-key tidak berperilaku sesuai harapan.
- **Penyebab:** Beberapa keyboard memiliki sakelar Fn Lock fisik atau kombinasi tombol `Fn` yang juga mengubah cara kerja baris F-key.
- **Perbaikan:** Periksa panduan keyboard Anda untuk cara mengaktifkan atau menonaktifkan Fn Lock, lalu coba ubah pengaturannya.

## Catatan

> [!TIP]
> `/etc/modprobe.d/hid_apple.conf` akan hilang jika sistem operasi diinstal ulang. Sebaiknya buat cadangan file ini.

## Kontribusi

Kontribusi dapat diajukan melalui GitHub Issues atau pull request.
