# old-tablet-smart-home
old-tablet-smart-home
# Samsung T110 & Raspberry Pi ile Tapo Akıllı Ev Kontrol Paneli ve NVR Sistemi

Bu proje, elinizde bulunan eski bir **Samsung Galaxy Tab 3 Lite (SM-T110)** tableti, **Raspberry Pi (Ubuntu 20.04)** ve **100 GB USB Sabit Disk** kullanarak profesyonel bir akıllı ev kontrol paneline ve kamera kayıt cihazına (NVR) dönüştürme rehberidir.

Sistem sayesinde evinizdeki tüm **TP-Link Tapo** cihazlarını dokunmatik ekrandan kontrol edebilir, kameralarınızı canlı izleyebilir ve görüntüleri 7/24 USB diskinize kaydedebilirsiniz.

---

## 🛠️ İhtiyacımız Olanlar (Gereksinimler)

### Donanım
* **Akıllı Ev Beyni:** Raspberry Pi (Üzerinde Ubuntu 20.04 ve Docker kurulu)
* **Kontrol Paneli:** Samsung Galaxy Tab 3 Lite (SM-T110) - *LineageOS yüklenmiş olmalıdır.*
* **Depolama (NVR):** 100 GB USB Harddisk (Linux uyumluluğu için `ext4` formatında)
* **Ağ:** Tapo cihazları, Raspberry Pi ve Tablet aynı Wi-Fi ağına bağlı olmalıdır.

### Yazılım
* **Docker & Docker Compose** (Ubuntu üzerinde servisleri çalıştırmak için)
* **Home Assistant (Docker)** (Akıllı ev merkezi)
* **Frigate NVR (Docker)** (Yapay zeka destekli kamera kayıt yazılımı)
* **Fully Kiosk Browser APK** (Tableti kiosk moduna kilitlemek için)

---

## 📁 Proje Klasör Yapısı

GitHub reponuzda dosyaları şu şekilde organize edeceğiz:

```text
tapo-old-tablet-kiosk/
├── README.md               # Bu rehber dosya
├── docker-compose.yml      # Sistemleri tek tıkla ayağa kaldıran kod
└── config/
    └── frigate.yml         # Tapo kamera kayıt ayarları şablonu
```

---

## 🚀 Kurulum Adımları

### 1. Adım: USB Harddiski Ubuntu'ya Kalıcı Olarak Bağlama (Mount)
100 GB diskinizin Pi her açıldığında otomatik olarak tanınması gerekir.
1. Diskinizi Pi'ye takın ve terminalde `sudo blkid` yazarak diskinizin **UUID** kodunu bulun.
2. Klasör oluşturun: `sudo mkdir -p /mnt/usb-depo`
3. `/etc/fstab` dosyasını açın (`sudo nano /etc/fstab`) ve en alta şu satırı ekleyin (Kendi UUID'nizi yazın):
   ```text
   UUID=SİZİN-DİSKİN-UUID-KODU /mnt/usb-depo ext4 defaults,nofail 0 2
   ```
4. Kaydedip çıkın ve `sudo mount -a` komutuyla diski bağlayın.

### 2. Adım: Docker ve Sistemlerin Kurulumu
Proje klasörünü oluşturun ve içine `docker-compose.yml` dosyasını ekleyin:

```yaml
version: '3.8'
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    privileged: true
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./config/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro

  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable
    privileged: true
    restart: unless-stopped
    shm_size: "64mb"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./config/frigate.yml:/config/config.yml
      - /mnt/usb-depo:/media/frigate
    ports:
      - "5000:5000"
      - "8554:8554"
```

Terminalde bu klasörün içine girip sistemi başlatın:
```bash
docker compose up -d
```

### 3. Adım: Tapo Kameralarını NVR'a Bağlama (config/frigate.yml)
Tapo uygulamasından kameranız için oluşturduğunuz **Kamera Hesabı** bilgilerini ve kameranın yerel IP adresini `config/frigate.yml` dosyasına şu şekilde girin:

```yaml
mqtt:
  enabled: False # İsteğe bağlı, basit kurulum için kapalı

cameras:
  tapo_kamera_1:
    ffmpeg:
      inputs:
        - path: rtsp://kullanici_adi:sifre@KAMERA_IP_ADRESI:554/stream1
          roles:
            - detect
            - record
    detect:
      enabled: True
    record:
      enabled: True
      retain:
        days: 7 # 7 gün sonra eski kayıtları otomatik siler
```

### 4. Adım: Home Assistant Arayüzü ve Tablet Ayarları
1. Bilgisayarınızdan `http://RASPBERRY_PI_IP:8123` adresine giderek Home Assistant ilk kurulumunu yapın.
2. **Ayarlar > Cihazlar ve Hizmetler** bölümünden **Tapo** entegrasyonunu kurun ve tüm cihazlarınızı ekleyin.
3. Tabletinize **Fully Kiosk Browser APK** indirin ve kurun.
4. Başlangıç URL'si olarak `http://RASPBERRY_PI_IP:8123` yazın ve uygulamayı **Kiosk Modu**na alın.

---
## 💡 Önemli İpuçları & Güvenlik
* **Batarya Şişmesini Önleme:** Tablet sürekli şarjda kalacağı için, tabletin bağlı olduğu adaptörü bir **Tapo Akıllı Prize** takın. Home Assistant otomasyonu ile tablet şarjı %80 olunca prizi kapatıp, %20 olunca açılacak şekilde ayarlayın.
* **Hareket Algılama ile Ekran Uykusu:** Fully Kiosk ayarlarından ön kamerayı açarak, bir insan yaklaştığında tablet ekranının uyanmasını sağlayabilirsiniz.

Bu proje tamamen açık kaynaklıdır, geliştirmelere ve katkılara açıktır!
