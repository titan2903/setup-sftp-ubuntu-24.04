# Instal dan Konfigurasi Server SFTP Ubuntu 24.04

## 1. **Memperbarui Paket dan Menginstal OpenSSH Server**
   Perintah pertama yang dijalankan adalah memperbarui paket di sistem dan menginstal `openssh-server` yang diperlukan untuk menjalankan server SSH.

   ```
   sudo apt update
   ```
   - **Penjelasan**: Perintah ini digunakan untuk memperbarui daftar repositori paket di sistem.

   ```
   sudo apt install openssh-server -y
   ```
   - **Penjelasan**: Menginstal paket `openssh-server` yang menyediakan layanan SSH yang diperlukan untuk menghubungkan ke server melalui protokol SSH (termasuk SFTP).

## 2. **Memulai dan Mengaktifkan Layanan SSH**
   Setelah menginstal `openssh-server`, kita perlu memulai dan mengaktifkan layanan SSH agar dapat dijalankan saat sistem reboot.

   ```
   sudo systemctl start ssh
   ```
   - **Penjelasan**: Memulai layanan SSH agar dapat digunakan segera.

   ```
   sudo systemctl enable ssh
   ```
   - **Penjelasan**: Mengaktifkan layanan SSH agar secara otomatis berjalan setiap kali sistem di-reboot.

## 3. **Memeriksa Status Layanan SSH**
   Memeriksa apakah layanan SSH berjalan dengan baik.

   ```
   sudo systemctl status ssh
   ```
   - **Penjelasan**: Menampilkan status layanan SSH. Perintah ini memastikan bahwa layanan SSH berjalan tanpa masalah.

## 4. **Membuat Grup dan Pengguna untuk SFTP**
   - Membuat grup khusus `sftp_users` dan pengguna `sftpuser` yang hanya akan menggunakan SFTP.

   ```
   sudo groupadd sftp_users
   ```
   - **Penjelasan**: Membuat grup `sftp_users` yang akan digunakan untuk membatasi akses hanya untuk pengguna yang berada dalam grup ini.

   ```
   sudo useradd -m -G sftp_users -s /usr/sbin/nologin sftpuser
   ```
   - **Penjelasan**: Membuat pengguna baru `sftpuser` yang akan masuk dalam grup `sftp_users`. Opsi `-s /usr/sbin/nologin` memastikan pengguna ini tidak dapat login melalui terminal biasa, hanya dapat menggunakan SFTP.

## 5. Menambahkan Password untuk Pengguna `sftpuser`
   Setelah membuat pengguna sftpuser, Anda perlu menambahkan password agar pengguna bisa melakukan autentikasi saat menggunakan SFTP:

   ```
   sudo passwd sftpuser
   ```

   - **Penjelasan**: Perintah ini akan meminta Anda untuk memasukkan password baru untuk pengguna sftpuser. Password ini digunakan saat pengguna ingin login dan mengakses server melalui SFTP.

## 6. **Menambahkan Konfigurasi di File sshd_config**
   Langkah ini mengedit file konfigurasi SSH untuk menentukan akses yang dibatasi bagi grup `sftp_users` dan hanya mengizinkan akses SFTP, bukan akses shell.

   - Buka file konfigurasi SSH `sshd_config` untuk diubah:
     ```
     sudo nano /etc/ssh/sshd_config
     ```

   - **Tambahkan konfigurasi berikut di bagian akhir file:**
     ```
     Match Group sftp_users
     ChrootDirectory %h
     ForceCommand internal-sftp
     AllowTcpForwarding no
     X11Forwarding no
     ```
     - **Penjelasan**:
       - `Match Group sftp_users`: Bagian ini akan menerapkan pengaturan untuk grup `sftp_users`.
       - `ChrootDirectory %h`: Membatasi pengguna ke direktori home mereka sendiri sehingga mereka tidak dapat mengakses direktori lain di server.
       - `ForceCommand internal-sftp`: Memaksa pengguna untuk menggunakan SFTP dan tidak dapat menjalankan perintah lain seperti SSH shell.
       - `AllowTcpForwarding no`: Menonaktifkan pemajuan TCP untuk meningkatkan keamanan.
       - `X11Forwarding no`: Menonaktifkan pemajuan X11 untuk mencegah akses grafis jarak jauh yang tidak diperlukan.

## 7. **Restart Layanan SSH**
   Setelah mengubah file konfigurasi, Anda perlu merestart layanan SSH agar perubahan diterapkan.

   ```
   sudo systemctl restart ssh
   ```
   - **Penjelasan**: Merestart layanan SSH untuk menerapkan perubahan yang telah dilakukan pada file konfigurasi.

## 8. **Menyiapkan Direktori dan Izin Pengguna SFTP**
   - Mengatur izin pada direktori pengguna SFTP dan membuat direktori `uploads` untuk file yang dapat diakses melalui SFTP.

   ```
   sudo chown root:root /home/sftpuser
   ```
   - **Penjelasan**: Mengubah kepemilikan direktori home pengguna `sftpuser` menjadi milik root. Hal ini diperlukan karena direktori home pengguna SFTP harus dimiliki oleh root agar kebijakan `ChrootDirectory` dapat diterapkan dengan benar.

   ```
   sudo chmod 755 /home/sftpuser
   ```
   - **Penjelasan**: Memberikan izin baca, tulis, dan eksekusi untuk pemilik (root), serta izin baca dan eksekusi untuk grup dan pengguna lain.

   ```
   sudo mkdir /home/sftpuser/uploads
   ```
   - **Penjelasan**: Membuat direktori `uploads` di dalam direktori home pengguna `sftpuser` untuk menyimpan file yang dapat diunggah melalui SFTP.

   ```
   sudo chown sftpuser:sftp_users /home/sftpuser/uploads
   ```
   - **Penjelasan**: Mengubah kepemilikan direktori `uploads` menjadi milik pengguna `sftpuser` dan grup `sftp_users`, sehingga pengguna dapat mengakses dan mengunggah file ke dalamnya.

---

Dengan mengikuti langkah-langkah di atas, Anda telah berhasil mengonfigurasi SFTP di Ubuntu 24.04 dan membatasi akses pengguna `sftpuser` hanya untuk SFTP, dengan kemampuan untuk mengunggah file ke direktori yang ditentukan. Pastikan untuk memeriksa setiap langkah dengan teliti untuk menghindari kesalahan konfigurasi.