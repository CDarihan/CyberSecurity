
# **Güvenli Linux Sunucu Yapılandırması + Saldırı & Savunma Simülasyonu** #


Bu proje, gerçek dünyadaki bir siber güvenlik senaryosunu simüle ederek:

* **Ubuntu Server** üzerinde bir sunucu yapılandırıyor,
* **SSH servisini güvenli hale getiriyor**,
* Güvenlik açıklarını test etmek için **Kali Linux üzerinden saldırı gerçekleştiriyor**,
* Saldırıları tespit edip otomatik olarak engellemek için **Fail2Ban yapılandırıyor**,
* Tüm süreci loglar, ekran görüntüleri ve yapılandırma dosyaları ile belgeliyor.


## 🎯 **Projenin Amacı**

✔ SSH’nin varsayılan zayıf ayarlarını güçlendirmek
✔ Brute‑force saldırılarını analiz etmek
✔ Loglardan saldırı izlerini çıkarmak
✔ Fail2Ban kullanarak saldırgan IP’leri otomatik banlamak
✔ Tüm yapılandırmayı dokümante etmek

---

## 🌐 **Ağ Yapılandırması**

Her iki makine de aynı NAT ağına bağlandı:

| Makine        | IP Adresi   | Rol       |
| ------------- | ----------- | --------- |
| Kali Linux    | `10.0.2.15` | Saldırgan |
| Ubuntu Server | `10.0.2.4`  | Hedef     |

Bağlantı doğrulaması:

```bash
ping 10.0.2.4
ping 10.0.2.15
```

---

## 🔐 **1. SSH Servisinin Kurulumu ve Doğrulanması**

SSH durumu kontrol edildi:

```
sudo systemctl status ssh 
```

Eğer servis aktif değilse aşağıdaki komutla aktif edildi:

```
sudo systemctl enable --now ssh
```

---

## 🔧 **2. SSH Güvenlik Arttırma**

### ✔ SSH Portunun Değiştirilmesi (22 → 2299)

SSH yapılandırması düzenlendi:

```bash
sudo nano /etc/ssh/sshd_config
```

Değiştirilen kritik ayarlar:

```
Port 2299    # Varsayılan SSH Portu 22'dir, portu değiştirerek saldırganların amaçlarına ulaşmasını zorlaştırıyoruz.
PermitRootLogin no  # root kullanıcı adıyla yapılan BruteForce denemelerinin başarılı olma ihtimali daha yüksek olduğundan root girişini kapatıyoruz.
PasswordAuthentication yes/no (her ikisi de test edildi)   # Ubuntu Serverımıza serverın şifresiyle girilip girilmeyeceğini kararlaştırıyoruz.
PubKeyAuthentication no  # Kaliden giriş için key oluşturulduğundan Public Key doğrulamasını aktifleştiriyoruz böylece kullanıcı giriş yaparken tekrar parola girmek zorunda kalmıyor.

```

Sözdizimi kontrolü:

```bash
sudo sshd -t
```

SSH servisi yeniden başlatıldı:

```bash
sudo systemctl restart ssh
```

---

## 🔑 **3. Kali → Ubuntu SSH Bağlantı Testleri**

Bağlantı şu şekilde test edildi:

```bash
ssh -p 2299 ural@10.0.2.4
```

Toplanan veriler:

* Yanlış şifre giriş denemeleri
* Başarılı bağlantılar
* Loglarda brute‑force benzeri davranışlar

Tüm veriler `/logs` dizinine eklendi.

---

## 📄 **4. Log Analizi**

Authentication logları incelendi:

```bash
sudo tail -f /var/log/auth.log
```

Tespit edilenler:

* Yanlış kullanıcı adı denemeleri
* Yanlış şifre denemeleri
* Aynı IP’den gelen ardışık denemeler
* Başarılı girişler
* Saldırı davranışı

---

## ⚔️ **5. Hydra ile Brute‑Force Saldırısı**

Örnek bir şifre listesi oluşturuldu:

```
123456
admin
root
ural

```

Hydra saldırısı:

```bash
hydra -l ural -P şifreler.txt -s 2299 ssh://10.0.2.4 -t 4 -V
```

Sonuç:

* Hydra tüm şifreleri tek tek denedi
* Doğru şifreyi buldu
* SSH’nın şifre tabanlı yapılandırmasının riskleri gözlemlendi

---

## 🛡 **6. Fail2Ban ile Otomatik IP Engelleme**

Kurulum:

```bash
sudo apt install fail2ban
```

Jail yapılandırması:

```
[sshd]
enabled = true
port = 2299
logpath = /var/log/auth.log
maxretry = 4
findtime = 1m # 1 dakika 
bantime = 600 # 10 dakika
```

Fail2Ban aktifleştirme:

```bash
sudo systemctl enable --now fail2ban
```

Ban listesini görüntüleme:

```bash
sudo fail2ban-client status sshd
```

Saldırılar tekrarlanınca IP otomatik olarak banlandı.

---

## 📊 **Sağlanan Güvenlik Kazanımları**

| Güvenlik Katmanı                  | Durum |
| --------------------------------- | ----- |
| Varsayılan SSH port değişimi      | ✔     |
| Root girişinin kapatılması        | ✔     |
| Brute‑force saldırı tespiti       | ✔     |
| Fail2Ban ile otomatik IP banlama  | ✔     |
| Log tabanlı izleme                | ✔     |
| Key‑based authentication testleri | ✔     |
| Saldırı simülasyonu               | ✔     |

---

# ÖNEMLİ: Yukarıda bahsedilen "Ban listesi görünümü" gibi ifadelerin örnekleri için diğer klasörlerdeki görüntülere bakmayı unutmayınız! #
