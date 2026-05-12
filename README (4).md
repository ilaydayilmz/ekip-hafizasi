# 🧠 Ekip Hafızası — AI Destekli Takım Yönetim Sistemi

> Takımınızın görev, deadline ve bildirimlerini otomatik yöneten, yapay zeka destekli akıllı asistan sistemi.

---

## 📌 Proje Hakkında

**Ekip Hafızası**, e-posta, takvim ve Telegram entegrasyonlarını birleştirerek takım içi görev takibini tamamen otomatikleştiren bir n8n + Gemini AI çözümüdür. Sistem; gelen e-postaları okur, görevleri çıkarır, deadline'ları takip eder, gecikmeleri tespit eder ve sabah brifing raporu gönderir.

---

## 🏗️ Sistem Mimarisi

```
Gmail (Gelen Postalar)          Telegram (Sesli Mesajlar)
        │                                │
        └──────────────┬─────────────────┘
                       ▼
[n8n — AI Asistan Workflow]
        │
        ├─► Duplicate Kontrol (Sheets)
        ├─► Gemini AI Analizi
        ├─► Google Sheets (Görev Kayıt)
        ├─► Google Calendar (Deadline Ekleme)
        └─► Telegram Bildirimi
                │
                ▼
[n8n — Gecikme Kontrol Workflow]
        │
        ├─► Geciken Görev Tespiti
        ├─► Gemini AI Brifing Üretimi
        └─► Gmail (Sabah Brifing Maili)
```

---

## ⚙️ Bileşenler ve İş Akışları

### 1. 🤖 AI Asistan Workflow (`Ekip_Hafızası_AI_Asistan.json`)

Her 15 dakikada bir çalışır ve gelen e-postaları işler.

| Adım | Node | Açıklama |
|------|------|----------|
| 1 | Every 15 Minutes | Zamanlayıcı tetikleyici |
| 2 | Get Unread Emails | Gmail'den okunmamış mailler alınır |
| 3 | Get Telegram Voice Messages | Telegram'dan sesli mesajlar alınır |
| 4 | Process One by One | Her kaynak öğesi ayrı ayrı işlenir |
| 5 | Generate Email Hash | Tekrar kontrolü için hash oluşturulur |
| 6 | Check for Duplicate | Sheets'te daha önce işlenmiş mi kontrol edilir |
| 7 | Is Duplicate? | Mükerrer kayıt filtresi |
| 8 | Get User Profile | Kullanıcı profili çekilir |
| 9 | Build Profile Context | AI için bağlam hazırlanır |
| 10 | Call Gemini AI | Görev, kişi, deadline, öncelik çıkarımı yapılır |
| 11 | Parse AI Response | AI yanıtı ayrıştırılır |
| 12 | Is Relevant? | Görev içerip içermediğine karar verilir |
| 13 | Save to Sheets | Görev Google Sheets'e kaydedilir |
| 14 | Has Deadline? | Deadline var mı kontrol edilir |
| 15 | Add to Calendar | Google Calendar'a deadline eklenir |
| 16 | Telegram Notification | Telegram'a onay mesajı gönderilir |
| 17 | Mark Email as Read | Mail okundu olarak işaretlenir |

---

### 2. ⏰ Gecikme Kontrol Workflow (`Ekip_Hafızası_Gecikme_Kontrol.json`)

Her hafta içi sabah **09:00**'da otomatik çalışır.

| Adım | Node | Açıklama |
|------|------|----------|
| 1 | Her Sabah 09:00 (Hafta İçi) | Cron tetikleyici (`0 9 * * 1-5`) |
| 2 | Sheets — Tüm Görevleri Al | Google Sheets'ten tüm görevler okunur |
| 3 | Geciken ve Yarın Bitecekleri Bul | Deadline geçmiş veya yarın biten görevler filtrelenir |
| 4 | Sorun Var Mı? | Sorunlu görev var mı kontrol edilir |
| 5 | Gemini — Brifing Metni Üret | AI ile özetlenmiş brifing metni oluşturulur |
| 6 | Brifing Metnini Al | AI yanıtı ayrıştırılır |
| 7 | Gmail — Brifing Mailini Gönder | HTML formatlı brifing maili gönderilir |

---

### 3. 🖥️ Web Arayüzü (`ekip-hafizasi.html`)

Takımın tüm görevlerini tek ekrandan izlemek için sade ve modern bir dashboard.

**Özellikler:**
- Sidebar navigasyon (Görevler, Takvim, Ekip, Ayarlar)
- Görev durumu kartları (Bekliyor / Devam Ediyor / Tamamlandı)
- Deadline ve öncelik gösterimi
- Ekip üyesi bazlı filtreleme
- Koyu tema (dark mode)

---

## 🎙️ Telegram Sesli Mesaj İşleme

Kullanıcı bota sesli mesaj gönderdiğinde sistem otomatik olarak devreye girer:

1. **Transkript** — Sesli mesaj metne dönüştürülür
2. **Görev Tespiti** — Gemini AI içeriği analiz eder; görev içerip içermediğine karar verir
3. **İlgili değilse** — Bot kullanıcıya daha net anlatması için örnek bir yönlendirme gönderir:
   > *"Sesli notu dinledim ama içinde herhangi bir görev tespit edemedim. Biraz daha net anlatmayı dener misin? Örnek: 'Ali'nin pazartesiye kadar raporu bitirmesi lazım'"*
4. **İlgiliyse** — Görev, kişi, deadline ve öncelik çıkarılır; Google Sheets ve Google Calendar'a kaydedilir

---

## 🔔 Telegram Bildirimleri

Yeni bir görev algılandığında (Gmail veya Telegram sesli mesajdan) Telegram'a otomatik mesaj gönderilir:

```
🎙️ Sesli Not İşlendi!
📝 Transkript: ...

✅ Tespit Edilen Görevler (1):
1. 🟢 [Görev adı]
   👤 [Kişi] | 📅 [Deadline]

Görevler Sheets ve Takvime kaydedildi.
```

Kullanıcı `/evet` veya `/hayir` komutuyla görevi onaylayabilir ya da reddedebilir.

---

## 📧 Sabah Brifing Maili

Geciken veya yarın son günü olan görevler tespit edildiğinde otomatik HTML mail gönderilir:

- ⚠️ **Geciken Görevler** — Kişi adı ve kaç gün geciktiği
- 📅 **Yarın Bitenler** — Kişi ve görev adı
- 🤖 **AI Analizi** — Gemini'nin özet değerlendirmesi

---

## 🛠️ Kullanılan Teknolojiler

| Teknoloji | Kullanım Amacı |
|-----------|----------------|
| [n8n](https://n8n.io) | Otomasyon iş akışları |
| Google Gemini AI | Görev çıkarımı ve brifing üretimi |
| Google Sheets | Görev veritabanı |
| Google Calendar | Deadline takibi |
| Gmail | E-posta okuma ve brifing gönderimi |
| Telegram Bot | Sesli mesaj algılama, anlık bildirim ve onay sistemi |
| HTML / CSS | Web dashboard arayüzü |

---

## 🚀 Kurulum

### Gereksinimler

- n8n (self-hosted veya cloud)
- Google Cloud hesabı (Sheets, Calendar, Gmail OAuth2)
- Telegram Bot Token
- Google Gemini API anahtarı

### Adımlar

1. **n8n'i kurun** ve çalıştırın.

2. **Google OAuth2 kimlik bilgilerini** n8n'e ekleyin:
   - Google Sheets OAuth2 API
   - Gmail OAuth2

3. **Telegram Bot** oluşturun ([BotFather](https://t.me/BotFather) üzerinden) ve token'ı n8n'e ekleyin.

4. **Gemini API anahtarını** alın ve n8n'de `httpQueryAuth` olarak kaydedin.

5. **Google Sheets** üzerinde görev tablosunu hazırlayın. Gerekli sütunlar:

   | Sütun | Açıklama |
   |-------|----------|
   | Görev | Görev adı |
   | Kişi | Sorumlu kişi |
   | Öncelik | Yüksek / Orta / Düşük |
   | Durum | Bekliyor / Devam Ediyor / Tamamlandı |
   | Deadline | `GG.AA.YYYY` formatında |
   | Kaynak | Görevin kaynağı (Gmail vb.) |

6. **Workflow JSON dosyalarını** n8n'e import edin:
   - `Ekip_Hafızası_AI_Asistan.json`
   - `Ekip_Hafızası_Gecikme_Kontrol.json`

7. Workflow'larda **Sheets URL'sini** kendi tablonuzun URL'si ile güncelleyin.

8. Her iki workflow'u da **aktif** hale getirin.

---

## 📁 Dosya Yapısı

```
ekip-hafizasi/
├── Ekip_Hafızası_AI_Asistan.json        # AI e-posta işleme workflow'u
├── Ekip_Hafızası_Gecikme_Kontrol.json   # Sabah brifing workflow'u
├── ekip-hafizasi.html                   # Web dashboard arayüzü
└── README.md                            # Bu dosya
```

---

## 🤝 Takım

**Takım 12** — Yapay Zeka Hackathon 2026

| İsim | Rol |
|------|-----|
| İlayda Yılmaz | Geliştirici |
| Larissa Fındık | Geliştirici |

---

## 📄 Lisans

Bu proje hackathon kapsamında geliştirilmiştir. Tüm haklar saklıdır.
