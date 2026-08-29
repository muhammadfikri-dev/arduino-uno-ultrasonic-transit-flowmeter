# ⚡ Arduino Uno Clamp-On Ultrasonic Transit-Time Water Flowmeter

[![Lisensi: MIT](https://img.shields.io/badge/Lisensi-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: Arduino Uno](https://img.shields.io/badge/Platform-Arduino%20Uno%20|%20ATmega328P-blue.svg)](#)
[![Framework: Arduino IDE](https://img.shields.io/badge/Framework-Arduino%20IDE%202.0%2B-teal.svg)](https://www.arduino.cc/)
[![Status: Firmware Produksi](https://img.shields.io/badge/Status-Firmware%20Produksi-brightgreen.svg)](#)
[![Developer: Muhammad Fikri](https://img.shields.io/badge/Developer-Muhammad%20Fikri-blue.svg)](#)

Non-invasive water pipe flowmeter calculating nanosecond transit-time delta between upstream/downstream 1MHz ultrasonic transducers to measure flow velocity.

---

## 📊 Diagram Blok Arsitektur & Skema Alur Rangkaian

Visualisasi interaktif alur daya, akuisisi sinyal sensor, pemrosesan algoritma inti, dan aktuasi proteksi perangkat:

```mermaid
graph TD
    subgraph Building_Field ["🏢 Sensor Keamanan & Zona Gedung"]
        RADAR["Radar 24GHz / Laser Tripwire"] --> SENS_IN["Detektor Presensi & Intrusi"]
        PRESSURE["Sensor Tekanan Ruangan / Asap"] --> SENS_IN
        SENS_IN --> MCU["🧠 Arduino Uno (ATmega328P 16MHz)"]
        ESTOP["Manual Emergency Break-Glass"] -->|"Interupsi"| MCU
    end

    subgraph BMS_Automation ["🧠 Otomasi Gedung & Interlock FSM"]
        MCU -->|"Safety State Machine"| FSM["Logika Interlock Pintu & Evakuasi"]
        FSM -->|"BACnet / DALI Protocol"| PROTO["Protokol Komunikasi Building Hub"]
        FSM -->|"Security Zone Engine"| SEC["Manajemen Multi-Zona"]
    end

    subgraph Actuation_Safety ["🚨 Aktuator & Sistem Proteksi"]
        FSM -->|"Relay Output"| DAMPER["Motor Damper VAV & Magnetic Lock"]
        FSM -->|"Alarm Trigger"| SIREN["Siren Darurat & Lampu Strobo"]
        MCU -->|"Fieldbus"| SCADA_HUB["SCADA Gedung BACnet MS/TP"]
        MCU -->|"I2C"| DISP["Layar Status Operator"]
    end

    style MCU fill:#1565c0,stroke:#0d47a1,stroke-width:2px,color:#fff
    style FSM fill:#2e7d32,stroke:#1b5e20,stroke-width:2px,color:#fff
    style DAMPER fill:#bf360c,stroke:#870000,stroke-width:2px,color:#fff
```

---

## 📦 Daftar Komponen & Bahan Lengkap (Bill of Materials - BOM)

Berikut rincian spesifikasi komponen fisik dan modul yang dibutuhkan untuk membangun proyek ini:

| No | Nama Komponen / Modul | Estimasi Jumlah | Fungsi & Spesifikasi Teknis |
|:---|:---|:---|:---|
| 1 | **Arduino Uno R3 (ATmega328P)** | 1 Unit | Mikrokontroler 8-bit deterministik 16MHz |
| 2 | **Adaptor Daya DC 9V-12V 1A / USB 5V** | 1 Unit | Sumber daya listrik stabil dengan proteksi arus |
| 3 | **Modul RS485 MAX485 / Modul DALI Bus Transceiver** | 1 Unit | Komunikasi protokol gedung pintar BACnet / DALI |
| 4 | **Sensor Lingkungan & Keamanan (Laser ToF / Radar 24GHz / Pyro)** | 1-2 Unit | Pendeteksi presensi dan intrusi keamanan |
| 5 | **Modul Relay Daya Tinggi 4-Channel dengan Optocoupler** | 1 Unit | Pengendali damper HVAC, lampu darurat, dan pintu interlock |
| 6 | **Layar Status I2C LCD / OLED** | 1 Unit | Tampilan status zona gedung dan alarm bahaya |
| 7 | **Siren Alarm 12V & Tombol Manual Interlock** | 1 Set | Peringatan darurat dan override fisik |

---

## 🧠 Arsitektur Sistem & Fitur Utama

- **Deterministic Non-Blocking State Machine:** Memisahkan pemrosesan sinyal presisi tinggi dari task telemetri untuk mencegah *latency jitter*.
- **Digital Signal Processing (DSP) & Filtering:** Dilengkapi algoritma digital filtering terdedikasi untuk eliminasi derau sinyal analog.
- **Non-Volatile Storage (Internal EEPROM):** Parameter kalibrasi, *setpoint*, dan konfigurasi tersimpan secara persisten terhadap siklus pemadaman daya.
- **Hardware Failsafe & Emergency Interlock:** Perlindungan otomatis jika terjadi anomali tegangan, kelebihan beban arus, atau pemicuan tombol *Emergency Stop*.
- **Industrial Telemetry & Diagnostics:** Pelaporan status operasional secara real-time via Serial/JSON stream.

---

## 🔌 Skema Pinout & Koneksi Hardware

| Komponen / Sinyal | Pin (Arduino Uno) | Deskripsi Fungsi |
|:---|:---|:---|
| **Sensor Analog Input** | `Pin A0` | Jalur pembacaan sensor utama berpresisi tinggi |
| **Emergency Stop (E-Stop)** | `Pin 2 (INT0)` | Pemicu pengaman darurat hardware interrupt |
| **Actuator / Relay Utama** | `Pin 9 (PWM) / Pin 7` | Pengendali beban daya tinggi / relay aktuator |
| **Acoustic Alarm Buzzer** | `Pin 8` | Indikator peringatan audible saat terjadi anomali |
| **Status / Heartbeat LED** | `Pin 13` | Indikator status aktivitas sistem real-time |

---

## 🛠️ Panduan Perakitan Hardware (Langkah Demi Langkah)

1. **Persiapan Catu Daya:** Hubungkan catu daya utama ke jalur daya mikrokontroler. Pasang kapasitor *decoupling* 100nF di dekat pin VCC untuk meredam ripple switching.
2. **Pemasangan Sensor & Modul:** Sambungkan jalur sinyal sensor ke pin mikrokontroler yang telah ditentukan. Gunakan resistor pull-up 4.7kΩ pada jalur SDA/SCL jika menggunakan modul I2C.
3. **Pemasangan Aktuator:** Hubungkan modul relay / gate driver MOSFET ke pin kontrol output. Pasang dioda *flyback* (1N4007) pada beban induktif untuk mengeliminasi lonjakan tegangan balik (*back-EMF*).
4. **Pemasangan Tombol Emergency Stop:** Sambungkan tombol darurat ke pin interupsi eksternal dengan konfigurasi *Active-LOW* menggunakan resistor *pull-up*.
5. **Verifikasi Koneksi:** Lakukan pengecekan jalur ground bersama (*Common Ground*) pada seluruh modul sebelum menyalakan daya.

---

## 🚀 Panduan Kompilasi & Upload (Arduino IDE)

1. Buka **Arduino IDE 2.0+**.
2. Masuk ke menu **Tools > Board**:
   * Pilih **`Arduino Uno`**.
3. Pastikan dependensi pustaka terpasang via Library Manager:
   * `ArduinoJson`
   * `Wire` & `SPI`
   * `EEPROM`
4. Buka berkas [`arduino-uno-ultrasonic-transit-flowmeter.ino`](./arduino-uno-ultrasonic-transit-flowmeter.ino).
5. Klik tombol **Verify** (✓) kemudian **Upload** (➔).
6. Buka **Serial Monitor** pada baudrate **`115200`** untuk melihat streaming telemetri dan status operasional.

---

## 📄 Lisensi
Didistribusikan di bawah lisensi open-source **MIT License**. Dikembangkan oleh **Muhammad Fikri**.
