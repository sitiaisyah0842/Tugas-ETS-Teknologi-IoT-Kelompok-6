# TUGAS ETS TEKNOLOGI IOT
Kelompok : 6

Anggota  : 
- Siti Aisyah       (2042231008)

- Shelma Nur Sabila (2042231022)

# IoT Distance Monitoring System (HC-SR04, ESP32-S3, Rust, ThingsBoard, OTA)

## Deskripsi Project

Proyek ini bertujuan untuk merancang dan membangun **sistem IoT monitoring jarak** berbasis **sensor HC-SR04** dan **mikrokontroler ESP32-S3**. Sistem ini menggunakan **bahasa pemrograman Rust** di atas **sistem operasi Ubuntu**, dengan integrasi ke **ThingsBoard IoT Platform** untuk visualisasi data secara real-time, serta mendukung fitur **Over-The-Air (OTA)** update untuk pembaruan firmware jarak jauh.

## Langkah-langkah Implementasi

### 1. Persiapan Hardware dan Software

1.1 Siapkan perangkat berikut:
   - ESP32-S3 DevKitC
   - Sensor ultrasonik HC-SR04
   - Kabel jumper dan breadboard
   - Laptop dengan sistem operasi Ubuntu
   - Koneksi Wi-Fi

1.2 Instal Rust dan dependensinya di Ubuntu:
   - sudo apt update && sudo apt install git curl clang cmake libudev-dev
   - curl https://sh.rustup.rs -sSf | sh
   - rustup default stable

1.3 Tambahkan target ESP32-S3 dan toolchain:
   - rustup target add xtensa-esp32s3-none-elf
   - cargo install espup ldproxy
   - espup install
   
1.4 Verifikasi instalasi:
   - cargo build
   - rustc --version
   - idf.py --version

### 2. Rancang Bangun Sistem
2.1 Diagram Wiring

- **VCC → 5V**  
- **GND → GND**  
- **Trig → GPIO 5**  
- **Echo → GPIO 18**  

Pastikan semua sambungan kabel sesuai dengan daftar di atas dan gunakan **level shifter** bila diperlukan untuk menjaga kompatibilitas tegangan antara **HC-SR04** dan **ESP32-S3**.

2.2 Diagram Arsitektur

- **HC-SR04**: Mengukur jarak dengan memancarkan gelombang ultrasonik dan menerima pantulannya.  
- **ESP32-S3**: Memproses waktu pantulan sinyal untuk menghitung jarak dan mengirimkan data melalui Wi-Fi.  
- **Wi-Fi Network**: Menjadi media komunikasi antara mikrokontroler dan platform IoT.  
- **ThingsBoard**: Menerima data jarak dari ESP32-S3 melalui protokol **MQTT** dan menampilkannya dalam bentuk dashboard visual.  
- **OTA (Over-The-Air)**: Memungkinkan pembaruan firmware secara jarak jauh tanpa memerlukan koneksi kabel atau reprogram manual.  

### 3. Implementasi Program Rust

Struktur proyek terdiri atas:
   - main.rs
   - cargo.toml
   - beserta file pendukung yang dapat dilihat di folder 'code'

Seluruh program diatas sudah saya lampirkan di github dengan format .txt

### 4. Setup Thingsboard
Daftar di thingsboard.cloud via login Google, buka Entities > Devices > Add New Device, beri nama “ESP32-S3” dengan credentials MQTT (refresh Client ID seperti ’pe8qc3sr584qzm78ka8f’, username ’shelmais’, password ’shelma118’). Test koneksi di terminal Ubuntu dengan "sudo apt install mosquitto-clients" dan "mosquitto_pub -d -q 1 -h mqtt.thingsboard.cloud-p 1883 -t v1/devices/me/telemetry -i "pe8qc3sr584qzm78ka8f" -u "shelmais" -P "shelma118" -m "{distance:10}"". Lalu buat dashboard dengan widget Time-series untuk telemetry “distance” guna visualisasi data jarak.

### 5. Integrasi MQTT
Edit main.rs untuk tambahkan WiFi menggunakan esp-wifi (ubah SSID dan password). MQTT dengan rumqttc dependencies di Cargo.toml (tambah rumqttc, serde-json). Publish data jarak sebagai JSON ke topic “v1/devices/me/telemetry” setiap 2 detik dengan esp_println logging. Integrasikan inisialisasi sensor dan WiFi sesuai diagram alir (Mulai > Inisialisasi Sensor/WiFi/OTA > HC-SR04 Mengukur > Data ke ThingsBoard). Jalankan cargo build dan espflash flash –monitor hingga sukses meski ada error iteratif.

### 6. Implementasi OTA
Buat file buildota.sh dengan cargo build –release, flash.sh dengan espflash flash target/xtensa-esp32s3-espidf/project-iot –monitor, dan partitions.csv untuk partisi OTA di root proyek. Integrasikan ESP-IDF OTA component via esp-idf-sys di Cargo.toml. Konfigurasi web server HTTP port 80 untuk handle update. Generate .bin dengan ./buildota.sh (chmod +x jika error). Lalu update firmware via OTA sesuai diagram alir (Tunggu Interval > Periksa OTA > Proses Update Firmware OTA > Restart Sistem) dan diagram arsitektur (Rust Embedded > Device IoT > MQTT Connectivity > ThingsBoard).

### 7. Pengujian Sistem
Uji fungsionalitas pembacaan sensor (akurasi >95% via serial monitor), koneksi MQTT/WiFi (update real-time di ThingsBoard), dan OTA (update .bin versi 2 tanpa kabel via ./flash.sh atau web server) selama 24 jam
sesuai diagram alir lengkap (Mulai > WiFi Terhubung? > Pengukuran > Kirim Data > Periksa OTA > Update Selesai > Restart). Catat error dan iterasi di tabel sambil monitor anomali.

## Diagram Sistem

<p align="center">
  <img src="wiring.jpg" alt="Wiring Diagram" width="300"><br>
  <em>Gambar 1.0 Diagram Wiring ESP32-S3 dan Sensor HC-SR04</em>
</p>

<p align="center">
  <img src="Diagram Alir Sistem.png" alt="Diagram Alir Sistem" width="200"><br>
  <em>Gambar 1.1 Diagram Alir Sistem</em>
</p>

<p align="center">
  <img src="Diagram Arsitektur.png" alt="Diagram Arsitektur" width="200"><br>
  <em>Gambar 1.2 Diagram Arsitektur</em>
</p>

## Pembahasan Hasil
### 5.1 Hasil Pengujian pada Visual Studio Code
<p align="center">
  <img src="gambar 1.jpeg" alt="Hasil .\flash.sh atau Run Program Rust pada Terminal" width="300"><br>
  <em>Gambar 5.0 Hasil .\flash.sh atau Run Program Rust pada Terminal</em>
</p>
Berdasarkan hasil pengujian proses esp.flash pada ESP32-S3 yang ditampilkan melalui terminal Visual Studio Code, terlihat bahwa sistem berhasil melakukan proses pembacaan data sensor dan pengunduhan firmware secara bertahap melalui mekanisme chunk-based transfer. Log menunjukkan bahwa perangkat mampu mempublikasikan dan menerima pesan MQTT secara berurutan untuk setiap potongan data firmware (chunk) dengan ukuran tertentu, serta mengirimkan data telemetry seperti jarak air dan waktu pembacaan ke ThingsBoard Cloud secara real-time. Proses pembaruan firmware ditandai dengan status DOWNLOADING, di mana setiap chunk diterima, diakumulasi, dan diverifikasi hingga seluruh data firmware selesai diunduh. Hasil ini menunjukkan bahwa fungsi flash dan OTA pada ESP32-S3 bekerja dengan baik, memiliki kemampuan komunikasi dua arah yang stabil melalui MQTT, serta dapat melakukan update firmware secara remote tanpa gangguan, yang menegaskan keandalan sistem dalam mendukung pemeliharaan perangkat IoT secara efisien dan aman.

### 5.2 Hasil Pengujian dalam Thingsboard
<p align="center">
  <img src="Gambar 2.jpeg" alt="OTA Update pada Thingsboard" width="300"><br>
  <em>Gambar 5.1 OTA Update pada Thingsboard</em>
</p>
Berdasarkan hasil pengujian pada fitur OTA (Over-The-Air) update di platform ThingsBoard Cloud, terlihat bahwa sistem berhasil mengelola dan menyimpan beberapa versi paket firmware yang telah diunggah dengan format file .bin. Terdapat empat entri firmware dengan judul dan versi berbeda mulai dari shelmais2.0 hingga shelmais3.0 yang menunjukkan proses pembaruan firmware berjalan secara bertahap dan terdokumentasi dengan baik melalui kolom waktu pembuatan (created time) dan version tag. Hal ini menandakan bahwa mekanisme OTA pada sistem berfungsi sesuai tujuan, yaitu memungkinkan pembaruan firmware perangkat ESP32-S3 dari jarak jauh tanpa intervensi fisik. Dengan status paket yang aktif serta keberhasilan unggahan beberapa versi firmware, dapat disimpulkan bahwa integrasi antara perangkat, ThingsBoard Cloud, dan sistem OTA berjalan dengan baik, stabil, serta mendukung peningkatan fleksibilitas dan efisiensi pemeliharaan sistem IoT secara real-time.

<p align="center">
  <img src="gambar 3.jpeg" alt="Device Profile pada Thingsboard" width="300"><br>
  <em>Gambar 5.2 Device Profile pada Thingsboard</em>
</p>
Gambar tersebut menunjukkan tampilan halaman Device Profiles pada platform ThingsBoard Cloud, di mana pengguna dengan nama akun Shelma Sabila telah membuat profil perangkat bernama "shelmais". Profil ini berfungsi sebagai template konfigurasi yang menentukan parameter komunikasi, aturan data, serta pengaturan dashboard untuk perangkat IoT yang terhubung. Status sistem di bagian atas menunjukkan kondisi "Active (Action required)", yang menandakan bahwa langganan masih aktif namun memerlukan tindakan tambahan, seperti verifikasi atau pembaruan konfigurasi. Dengan adanya device profile ini, setiap perangkat yang terhubung dapat menggunakan pengaturan yang sama secara konsisten, sehingga mempermudah proses integrasi, pemantauan, dan pengelolaan data sensor melalui ThingsBoard Cloud.

<p align="center">
  <img src="gambar 4.jpeg" alt="Device pada Thingsboard" width="300"><br>
  <em>Gambar 5.3 Device pada Thingsboard</em>
</p>
Gambar tersebut menampilkan tampilan halaman Devices pada platform ThingsBoard Cloud, di mana perangkat bernama “shelmais” sedang dipantau melalui tab Latest Telemetry. Dari data yang terlihat, perangkat ini memiliki beberapa parameter penting seperti current_fw_title dengan nilai Water Level Sensor, current_fw_version versi V1.0, fw_state dengan status UPDATING, serta realtime_clock yang menunjukkan waktu pembaruan terakhir yaitu 2025-10-14 12:37:14. Informasi tersebut menunjukkan bahwa perangkat “shelmais” berfungsi sebagai sensor ketinggian air (water level sensor) dan sedang menjalani proses pembaruan firmware secara aktif. Data telemetry yang tercatat juga menandakan bahwa koneksi antara perangkat fisik (misalnya ESP32 atau mikrokontroler lain) dan platform cloud berjalan dengan baik, memungkinkan ThingsBoard untuk menerima data real-time serta status sistem dari perangkat tersebut.

<p align="center">
  <img src="gambar 5.jpeg" alt="Hasil Pembacaan Data Tabel pada Dashboard Thingsboard" width="300"><br>
  <em>Gambar 5.4 Hasil Pembacaan Data Tabel pada Dashboard Thingsboard</em>
</p>
Gambar tersebut menunjukkan tampilan Timeseries Table pada dashboard ThingsBoard Cloud, yang menampilkan data pembacaan sensor secara real-time. Berdasarkan tabel, nilai Level yang terukur adalah 8 cm secara konsisten pada setiap pembacaan dengan interval waktu yang sangat rapat, sekitar 2–3 detik antara setiap data. Hal ini menunjukkan bahwa sensor level air bekerja secara stabil dan menghasilkan data yang konstan tanpa adanya fluktuasi signifikan selama periode pengamatan. Kolom realtime_clock yang sejalan dengan timestamp menandakan bahwa sinkronisasi waktu antara RTC (Real Time Clock) di perangkat dan waktu server cloud berjalan dengan baik. Dari hasil ini dapat disimpulkan bahwa sistem pengiriman data dari sensor ke ThingsBoard melalui jaringan MQTT berfungsi optimal, dengan latensi rendah dan keandalan tinggi dalam pelaporan level air secara kontinu.

<p align="center">
  <img src="gambar 6.jpeg" alt="Hasil Pembacaan Data Grafik pada Dashboard Thingsboard" width="300"><br>
  <em>Gambar 5.5 Hasil Pembacaan Data Grafik pada Dashboard Thingsboard</em>
</p>
Berdasarkan hasil pembacaan grafik level pada dashboard ThingsBoard Cloud, terlihat bahwa nilai level air berada pada kisaran 7 cm dengan rata-rata pembacaan yang stabil dan fluktuasi yang sangat kecil selama periode waktu 19:54:05 hingga 19:55:00. Garis grafik yang cenderung datar menunjukkan bahwa sensor ultrasonik berfungsi dengan baik dan mampu menjaga konsistensi pengukuran tanpa adanya gangguan signifikan seperti noise atau lonjakan data. Stabilitas ini mengindikasikan bahwa sistem pengukuran dan komunikasi data dari perangkat ke cloud berjalan normal, serta koneksi MQTT antara ESP32-S3 dan ThingsBoard bekerja dengan baik dalam mentransmisikan data level secara real-time.

#### 5.3 Latency Analysis IoT Data Streaming
<p align="center">
  <img src="Grafik Perbandingan Timestamp dan Realtime Clock.jpg" alt="Grafik Perbandingan Timestamp dan Realtime Clock" width="500"><br>
  <em>Gambar 1.3 Grafik Perbandingan Timestamp dan Realtime Clock</em>
</p>
Berdasarkan grafik yang memperlihatkan hubungan level terhadap timestamp dan realtime clock, tampak terdapat perbedaan waktu antara kedua sumbu (latency) di mana nilai level yang sama tidak berada pada titik waktu yang persis sama antara kedua plot tersebut. Dari grafik tersebut menunjukkan bahwa perbandingan antara Timestamp dan Realtime Clock terhadap nilai Level selama akhir September hingga pertengahan Oktober pada umumnya sangat konsisten, dengan kedua garis yang hampir berhimpitan sehingga latensi antara pencatatan waktu internal dan waktu nyata perangkat biasanya sangat kecil, pada rentang milidetik hingga beberapa detik saja. Namun, pada beberapa titik terdapat lonjakan atau spike di sekitar 1, 8, 12, dan 15 Oktober, yang menandakan momen terjadinya gangguan sinkronisasi atau keterlambatan pencatatan data. Pada fase awal periode pengamatan, latensi cenderung lebih tinggi namun membaik pada pertengahan dan akhir periode, dengan sesekali terjadi deviasi yang menyebabkan puncak data.

Latensi bisa terjadi akibat delay selama pengiriman data, pemrosesan oleh sistem, atau pelambatan komunikasi saat data dikirimkan dari sensor ke sistem pencatatan. Selain itu, kemungkinan adanya perbedaan antara waktu dari modul real-time clock (RTC) dengan waktu yang dicatat pada timestamp internal sistem, bisa memperburuk deviasi waktu, terutama saat proses sinkronisasi tidak berjalan mulus. Dari literatur, fluktuasi latensi sesaat juga pernah dilaporkan, umumnya rata-rata selisih waktu antara RTC dan waktu aktual masih dalam kisaran 1 hingga 3 detik, namun bisa lebih besar jika terjadi gangguan teknis atau rekonfigurasi perangkat. Kesimpulannya, sistem ini pada dasarnya cukup andal dan sinkron, tetapi tetap berpotensi mengalami spike latensi secara insidental, seiring adanya faktor eksternal maupun internal yang memengaruhi transfer serta sinkronisasi data.



## Kesimpulan
1. Proses ESP flash dan OTA update pada ESP32-S3 berhasil dilakukan secara bertahap melalui mekanisme chunk-based transfer, menunjukkan bahwa perangkat mampu berkomunikasi dua arah dengan ThingsBoard Cloud secara stabil melalui protokol MQTT dan mendukung pembaruan firmware jarak jauh tanpa gangguan.
2. ThingsBoard Cloud berfungsi optimal dalam mengelola, menyimpan, dan mendistribusikan beberapa versi firmware (.bin) secara terorganisir, menandakan sistem OTA mampu mendukung pembaruan perangkat dengan fleksibel dan efisien untuk pemeliharaan sistem IoT secara real-time.
3. Data dari sensor level air menunjukkan kestabilan pembacaan di kisaran 7–8 cm dengan interval waktu yang konsisten, serta sinkronisasi waktu antara realtime clock perangkat dan timestamp server berjalan baik, menandakan pengiriman data berlangsung dengan latensi rendah.
4. Perangkat dan device profile yang terhubung melalui ThingsBoard bekerja sesuai konfigurasi, menghasilkan data telemetry yang teratur serta grafik pengukuran yang stabil, membuktikan integrasi sistem antara ESP32-S3 dan platform cloud berjalan dengan baik dan andal.
5. Terdapat sedikit perbedaan waktu antara timestamp dan realtime clock akibat faktor komunikasi atau sinkronisasi, namun deviasi yang muncul masih dalam batas wajar (1–3 detik), menunjukkan sistem tetap handal dan responsif dalam pengiriman data sensor ke platform cloud.



## Saran
1. Perlu dilakukan sinkronisasi otomatis antara RTC perangkat dan server agar data memiliki waktu pencatatan yang akurat dan latensi dapat diminimalkan.
2. Disarankan untuk menambahkan enkripsi serta verifikasi integritas firmware guna memastikan proses pembaruan jarak jauh berjalan aman dan bebas dari korupsi data.
3. Perlu dilakukan uji performa sistem pada berbagai kondisi jaringan untuk memastikan stabilitas komunikasi MQTT dan ketahanan sistem dalam situasi nyata.
