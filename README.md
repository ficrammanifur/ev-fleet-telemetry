# ⚡ EV Fleet Telemetry & Battery Intelligence Platform

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![Hardware](https://img.shields.io/badge/Hardware-ESP32%20%7C%20INA219%20%7C%20GPS%20NEO--6M-green.svg)](https://espressif.com/)
[![MLOps](https://img.shields.io/badge/MLOps-MLflow%20%7C%20Scikit--Learn-orange.svg)](https://mlflow.org/)
[![Dashboard](https://img.shields.io/badge/Dashboard-GitHub%20Pages%20Ready-teal.svg)](./docs/index.html)
[![License](https://img.shields.io/badge/License-MIT-purple.svg)](LICENSE)

Proyek *end-to-end* **EV Fleet Telemetry & Battery Intelligence** dirancang khusus untuk portofolio **AI Solution Engineer** di industri kendaraan listrik (*Electric Vehicle Fleet & Energy Intelligence*). Proyek ini menggabungkan perangkat keras IoT (*Edge MCU*), *data pipeline* relasional SQL, *Machine Learning* untuk diagnostik degradasi baterai & prediksi sisa jarak tempuh, *MLOps experiment tracking* (MLflow), serta *interactive static dashboard* yang siap dideploy langsung ke **GitHub Pages**.

---

## 📚 Dokumentasi Spesifik Berdasarkan Modul

Untuk mempermudah peninjauan teknis, dokumentasi mendalam telah dipisahkan per modul:
- 📟 **[Dokumentasi Hardware & Firmware IoT (`firmware/README.md`)](./firmware/README.md)**: Skema sirkuit daya 3S, tabel pinout lengkap ESP32, konfigurasi `TFT_eSPI`, algoritma SOC, dan panduan *flashing*.
- ⚙️ **[Dokumentasi Backend, ML & MLOps (`backend/README.md`)](./backend/README.md)**: Skema database SQLAlchemy, data cleaning MQTT, model Isolation Forest & Random Forest, formula matematika, dan integrasi MLflow.
- 📊 **[Dokumentasi Dashboard Tampilan & UI/UX (`docs/README.md`)](./docs/README.md)**: Panduan UI/UX Dark Mode, integrasi peta CartoDB Dark Matter Leaflet, visualisasi Chart.js dual-axis, dan langkah aktivasi GitHub Pages.

---

## 📑 Daftar Isi
1. [Arsitektur Sistem](#-arsitektur-sistem)
2. [Spesifikasi Hardware & Skema Wiring Pinout](#-spesifikasi-hardware--skema-wiring-pinout)
3. [Fitur Utama Sistem](#-fitur-utama-sistem)
4. [Struktur Folder Proyek](#-struktur-folder-proyek)
5. [Panduan Instalasi & Menjalankan Pipeline](#-panduan-instalasi--menjalankan-pipeline)
6. [MLOps & Pelacakan Model (MLflow)](#-mlops--pelacakan-model-mlflow)
7. [Deploy Dashboard ke GitHub Pages](#-deploy-dashboard-ke-github-pages)
8. [Spesifikasi Model Machine Learning](#-spesifikasi-model-machine-learning)

---

## 🏛️ Arsitektur Sistem

Sistem beroperasi melalui pipeline 4 lapis (*4-tier architecture*):

```
┌────────────────────────────────────────────────────────────────────────────────┐
│ 1. EDGE IoT HARDWARE (ESP32 On-board Telemetry Tracker)                        │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌───────────┐ │
│   │ 3S Li-ion    │ ──> │ INA219 (I2C) │     │ GPS NEO-6M   │     │ TFT_eSPI  │ │
│   │ Battery Pack │     │ (Volt, Amp,  │     │ (Lat, Lng,   │     │ On-board  │ │
│   │ 9.6V - 12.6V │     │  Power, SOC) │     │  Speed km/h) │     │ Display   │ │
│   └──────────────┘     └──────┬───────┘     └──────┬───────┘     └─────▲─────┘ │
│                               └──────────┬─────────┘                   │       │
│                                          ▼                             │       │
│                               WiFiManager Captive AP                   │       │
│                               PubSubClient MQTT Sender ────────────────┘       │
└──────────────────────────────────────────┬─────────────────────────────────────┘
                                           │ MQTT Topic: "ev/fleet/unit01/telemetry" (JSON)
                                           ▼
┌────────────────────────────────────────────────────────────────────────────────┐
│ 2. INGESTION & DATA CLEANING ENGINE (Python Service)                           │
│   • Subscribes via paho-mqtt                                                   │
│   • Outlier & sensor noise filtering (validasi tegangan 6-16V, arus, GPS fix)  │
│   • Audit operational logging ke file logs/operations.log                      │
│   • Relational Persistence via SQLAlchemy (SQLite / PostgreSQL)                │
└──────────────────────────────────────────┬─────────────────────────────────────┘
                                           │ SQL Tables: trips, telemetry_logs, system_logs
                                           ▼
┌────────────────────────────────────────────────────────────────────────────────┐
│ 3. ML & MLOps PIPELINE (Scikit-Learn + MLflow)                                 │
│   • Model A: Isolation Forest (Deteksi Anomali Voltage Sag & Overcurrent)      │
│   • Model B: Random Forest Regressor (Prediksi Sisa Jarak Tempuh / Range km)   │
│   • MLOps: Logging Parameter, Metrik (RMSE, R2), & Model Artifacts ke MLflow   │
│   • Export Intelligence ke JSON Statis: /docs/data/trip_data.json              │
└──────────────────────────────────────────┬─────────────────────────────────────┘
                                           │ Asynchronous Fetch (HTTP / Fetch API)
                                           ▼
┌────────────────────────────────────────────────────────────────────────────────┐
│ 4. OPERATIONAL DASHBOARD (GitHub Pages Ready)                                  │
│   • Dark Mode UI (Google Maps Live Route + Strava Ride Analytics)              │
│   • Leaflet.js dengan CartoDB Dark Matter Tiles & Custom Anomaly Markers       │
│   • Chart.js (Dual Axis: Tegangan vs Arus, Kecepatan vs Daya Motor)            │
│   • KPI Cards: Status Baterai, Prediksi Range ML, Rekomendasi Aksi Mekanik    │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔌 Spesifikasi Hardware & Skema Wiring Pinout

### Komponen Utama:
1. **Mikrokontroler**: ESP32 DevKit V1 (30 atau 38 pin)
2. **Sensor Arus & Tegangan**: Modul INA219 High-Side DC I2C
3. **Modul Navigasi**: GY-NEO6MV2 GPS dengan antena keramik
4. **Layar Tampilan**: TFT Display 240x320 SPI (Driver ST7789 atau ILI9341)
5. **Regulator Daya**: Buck Converter Step-Down (LM2596 atau Mini360) untuk menurunkan tegangan baterai 3S (9.6V–12.6V) menjadi stabil 5.0V ke pin VIN ESP32.

### Tabel Koneksi Pinout:

| Komponen Hardware | Pin Modul Sensor / Display | Pin ESP32 DevKit | Keterangan / Fungsi |
| :--- | :--- | :--- | :--- |
| **Baterai 3S Li-ion** | Kutub Positif (+) | Vin+ (INA219) & IN+ Buck | Sumber tegangan pack baterai (12.6V max) |
| | Kutub Negatif (-) | GND & IN- Buck | Ground bersama (*common ground*) |
| **Buck Converter** | OUT+ (5.0V) | **VIN (5V)** | Suplai daya utama ESP32 |
| | OUT- (GND) | **GND** | Ground bersama |
| **INA219 (I2C)** | VCC | **3V3** | Suplai logika digital modul sensor |
| | GND | **GND** | Ground referensi |
| | SDA | **GPIO 21** | Jalur I2C Data ESP32 |
| | SCL | **GPIO 22** | Jalur I2C Clock ESP32 |
| | Vin+ | Baterai (+) Positif | Sisi masuk beban tegangan tinggi |
| | Vin- | Beban / ESC Motor (+) | Sisi keluar menuju beban powertrain |
| **GPS GY-NEO6MV2** | VCC | **VIN (5V)** atau **3V3** | Suplai daya modul GPS |
| | GND | **GND** | Ground |
| | TX | **GPIO 16 (RX2)** | Hardware UART2 Receive pada ESP32 |
| | RX | **GPIO 17 (TX2)** | Hardware UART2 Transmit pada ESP32 |
| **TFT Display (SPI)** | VCC | **3V3** | Daya backlight & modul |
| | GND | **GND** | Ground |
| | CS (Chip Select) | **GPIO 5** | SPI Slave Select |
| | RESET (RST) | **GPIO 4** | Display Reset |
| | DC / A0 | **GPIO 2** | Data / Command Selector |
| | SDI / MOSI | **GPIO 23** | SPI Master Out Slave In |
| | SCK / CLK | **GPIO 18** | SPI Clock |
| | LED / BL | **3V3** | Backlight ON terus-menerus |

> [!IMPORTANT]
> Pastikan semua ground (**GND**) terhubung menjadi satu (*Common Ground*): Ground baterai 3S, Buck converter, ESP32, INA219, dan GPS. Jangan menghubungkan tegangan baterai 3S (>12V) langsung ke pin GPIO atau pin 3V3 ESP32!

---

## 🚀 Fitur Utama Sistem

1. **Captive Portal Zero-Hardcode (WiFiManager)**:
   - Jika ESP32 belum terhubung ke WiFi, perangkat otomatis memancarkan Access Point bernama **`EV-Tracker-Config`**.
   - Pengguna cukup membuka browser di smartphone/laptop ke IP `192.168.4.1` untuk memilih WiFi/hotspot dan memasukkan kata sandi tanpa perlu *flash ulang* kode.
2. **On-Board Realtime TFT UI**:
   - Menampilkan status koneksi WiFi, jumlah satelit GPS aktif, tegangan pack 3S, arus discharge, estimasi persentase SOC, dan kecepatan kendaraan.
   - Peringatan visual warna merah otomatis muncul jika tegangan pack turun < 9.9V (*critical low voltage*) atau terjadi lonjakan arus > 15A.
3. **Data Cleaning & Resilient Ingestion**:
   - Skrip Python Ingestor secara otomatis memfilter koordinat GPS tidak valid (koordinat *null island* 0.0, 0.0 atau lonjakan kecepatan tidak wajar > 150 km/h) serta *noise sensor* sebelum disimpan ke database SQL.
4. **Deteksi Anomali Baterai dengan Machine Learning**:
   - Model **Isolation Forest** mendeteksi *voltage sag* abnormal dan *overcurrent peak* saat kendaraan melakukan akselerasi tajam di jalan menanjak.
5. **Prediktor Sisa Jarak Tempuh (Remaining Range)**:
   - Model **Random Forest Regressor** memprediksi sisa jarak tempuh (km) berdasarkan efisiensi energi dinamis (*rolling Wh/km*), profil kecepatan, dan level SOC baterai.
6. **MLOps Tracking dengan MLflow**:
   - Mencatat seluruh parameter eksperimen, metrik evaluasi (RMSE, MAE, $R^2$), grafik efisiensi, dan artefak model `.pkl` ke direktori `mlruns/`.
7. **Static Dashboard Siap Deploy (GitHub Pages)**:
   - Berkas statis (`index.html`, `style.css`, `script.js`) membaca `data/trip_data.json` secara asinkron tanpa memerlukan hosting backend berbayar.

---

## 📂 Struktur Folder Proyek

```
ev-fleet-telemetry/
├── firmware/
│   ├── README.md                   # Dokumentasi Hardware, BOM, Pinout & Firmware
│   └── esp32_tracker/
│       └── esp32_tracker.ino        # Kode Arduino C++ ESP32 + WiFiManager + INA219 + GPS + TFT + MQTT
├── backend/
│   ├── README.md                   # Dokumentasi Backend, Data Cleaning, ML & MLOps
│   ├── __init__.py
│   ├── database.py                 # SQLAlchemy ORM (Tabel trips, telemetry_logs, system_logs)
│   ├── mqtt_ingestor.py            # Stream ingestion, filter noise, dan database writer
│   ├── ml_pipeline.py              # ML Anomaly Detection, Range Predictor & MLflow Tracking
│   └── simulate_trip.py            # Simulator perjalanan realistis rute Tangerang-Jakarta
├── docs/                           # GitHub Pages Root Folder
│   ├── README.md                   # Dokumentasi UI/UX Dashboard, Peta & Chart Telemetri
│   ├── index.html                  # Dashboard Dark Mode (Tailwind CSS, Leaflet, Chart.js)
│   ├── style.css                   # Custom styles, pulsing pins & animations
│   ├── script.js                   # Logic fetch JSON, render peta, render grafik & KPI
│   └── data/
│       └── trip_data.json          # Hasil ekspor pipeline ML untuk dashboard
├── logs/
│   └── operations.log              # Audit operational log file
├── mlruns/                         # Direktori tracking eksperimen & model MLflow
├── requirements.txt                # Dependensi pustaka Python
└── README.md                       # Dokumentasi komprehensif proyek
```

---

## 🛠️ Panduan Instalasi & Menjalankan Pipeline

### Langkah 1: Kloning Repositori & Persiapan Lingkungan
```bash
git clone https://github.com/<username>/ev-fleet-telemetry.git
cd ev-fleet-telemetry
```

Buat dan aktifkan *virtual environment* Python:
```bash
python -m venv venv

# Di Linux / macOS:
source venv/bin/activate

# Di Windows PowerShell:
.\venv\Scripts\Activate.ps1
```

Instal seluruh dependensi:
```bash
pip install -r requirements.txt
```

---

### Langkah 2: Menjalankan Trip Simulation & ML Pipeline
Jalankan skrip simulator berikut:
```bash
python backend/simulate_trip.py
```
Skrip ini akan:
1. Menginisialisasi basis data SQLite `ev_fleet.db` (dengan tabel `trips`, `telemetry_logs`, `system_logs`).
2. Menghasilkan ~120 data titik telemetri realistis motor listrik melintasi rute ICE BSD &rarr; Alam Sutera (Tangerang).
3. Menyuntikkan 2 anomali realistis:
   - **Anomali 1**: *Extreme voltage sag* (< 9.9V) saat akselerasi tanjakan dengan arus 17.8A.
   - **Anomali 2**: *Overcurrent surge* (> 15A) saat akselerasi penuh (*full throttle*).
4. Menjalankan `backend/ml_pipeline.py` secara otomatis untuk melatih model Isolation Forest dan Random Forest.
5. Mencatat metrik eksperimen ke **MLflow**.
6. Mengekspor file `/docs/data/trip_data.json` yang berisi ringkasan perjalanan, koordinat rute, titik anomali, dan seri waktu grafik.

---

### Langkah 3: Menjalankan MQTT Ingestor (Untuk Hardware Nyata)
Jika Anda menggunakan hardware ESP32 fisik:
```bash
python backend/mqtt_ingestor.py
```
Layanan ini akan mendengarkan broker MQTT pada topik `ev/fleet/+/telemetry`, menyaring derau (*noise*), dan menyimpan data bersih ke database secara *realtime*.

---

### Langkah 4: Membuka Dashboard Secara Lokal
Anda dapat membuka dashboard langsung menggunakan Python built-in HTTP server:
```bash
# Jalankan web server dari folder docs/
python -m http.server 8000 --directory docs
```
Buka browser favorit Anda di alamat: **`http://localhost:8000`**.

---

## 🔬 MLOps & Pelacakan Model (MLflow)

Untuk memantau metrik performa model, parameter eksperimen, dan artefak model yang tersimpan:

```bash
mlflow ui
```
Buka antarmuka MLflow di browser: **`http://localhost:5000`**.

Anda akan melihat:
- Eksperimen: **`EV_Battery_Telemetry_Intelligence`**
- Parameter yang tercatat: `battery_chemistry`, `iso_forest_contamination`, `rf_n_estimators`, `sample_count`
- Metrik evaluasi: `rmse_remaining_km`, `mae_remaining_km`, `r2_score`, `anomalies_detected`, `battery_health_score`
- Artefak model: `isolation_forest_model` dan `range_predictor_model`

---

## 🌐 Deploy Dashboard ke GitHub Pages

Folder `/docs` dalam proyek ini telah dikonfigurasi secara mandiri (*self-contained static app*) sehingga dapat langsung di-hosting gratis via GitHub Pages:

1. *Push* kode repositori ini ke GitHub:
   ```bash
   git add .
   git commit -m "feat: complete ev fleet telemetry and mlops pipeline"
   git push origin main
   ```
2. Buka repositori Anda di GitHub melalui browser.
3. Masuk ke menu **Settings** &rarr; **Pages** (di bilah menu sebelah kiri).
4. Pada bagian **Build and deployment**:
   - **Source**: Pilih `Deploy from a branch`
   - **Branch**: Pilih `main` dan direktori folder **`/docs`**
5. Klik **Save**.
6. Dalam hitungan 1–2 menit, dashboard operasional Anda sudah tayang secara publik di:
   `https://<username>.github.io/ev-fleet-telemetry/`

---

## 🧠 Spesifikasi Model Machine Learning

### 1. Deteksi Anomali Baterai (Isolation Forest)
- **Tujuan**: Mendeteksi penurunan tegangan mendadak (*voltage sag* abnormal) dan beban arus lebih (*overcurrent surge*) yang dapat mengindikasikan resistansi sel baterai meningkat atau kerusakan konektor.
- **Fitur Masukan**:
  - `voltage_v` (Voltase terminal pack 3S)
  - `current_a` (Arus pemakaian motor)
  - `power_w` (Daya sesaat)
  - `voltage_sag` (Selisih deviasi tegangan terhadap rata-rata bergerak 5 titik)
- **Parameter**: `contamination = 0.04`, `n_estimators = 100`.

### 2. Prediktor Sisa Jarak Tempuh (Random Forest Regressor)
- **Tujuan**: Memprediksi sisa jarak tempuh aman (*remaining range in km*) secara akurat dan dinamis menyesuaikan gaya berkendara.
- **Fitur Masukan**:
  - `soc_percent` (State of Charge %)
  - `voltage_v` (Tegangan baterai)
  - `speed_kmh` (Kecepatan berkendara)
  - `power_w` (Daya motor listrik)
  - `rolling_efficiency_wh_km` (Konsumsi energi kumulatif per kilometer)
- **Metrik Evaluasi**: $R^2 > 0.95$, $\text{RMSE} < 0.6 \text{ km}$.

---

## 👨‍💻 Kontribusi & Hak Cipta
Dibuat sebagai implementasi proyek portofolio profesional untuk posisi **AI Solution Engineer / IoT Fleet ML Specialist**. Lisensi di bawah [MIT License](LICENSE).
