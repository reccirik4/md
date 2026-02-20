# 🚀 Google Play'de Uygulama Yayınlama - Eksiksiz Rehber (2026 Güncel)

> **Son Güncelleme:** Şubat 2026  
> **Kapsam:** Hesap açma → AdMob → Abonelik/Premium → Test → Yayın → Güncelleme  
> **Yöntem:** Her adımı tamamlayınca "Yaptım" de, bir sonraki adıma geçelim.  
> **Dil:** Tüm menü yolları 🇹🇷 Türkçe ve 🇬🇧 İngilizce olarak verilmiştir.

---

## 📋 İÇİNDEKİLER

1. [Ön Hazırlık Kontrol Listesi](#1--ön-hazırlık-kontrol-listesi)
2. [Play Console Türkçe ↔ İngilizce Menü Eşleşme Tablosu](#2--play-console-türkçe--i̇ngilizce-menü-eşleşme-tablosu)
3. [Google Play Console Hesabı Oluşturma](#3--google-play-console-hesabı-oluşturma)
4. [Kimlik Doğrulama ve Hesap Onayı](#4--kimlik-doğrulama-ve-hesap-onayı)
5. [AdMob Hesabı ve Reklam Entegrasyonu](#5--admob-hesabı-ve-reklam-entegrasyonu)
6. [Play Console'da Uygulama Oluşturma](#6--play-consoleda-uygulama-oluşturma)
7. [Store Listing / Mağaza Girişi Hazırlama](#7--store-listing--mağaza-girişi-hazırlama)
8. [Uygulama İçeriği Ayarları — Detaylı](#8--uygulama-i̇çeriği-ayarları--detaylı)
9. [Senaryo Bazlı Uygulama İçeriği Matrisi](#9--senaryo-bazlı-uygulama-i̇çeriği-matrisi)
10. [Senaryo Bazlı Veri Güvenliği Tıklama Rehberi](#10--senaryo-bazlı-veri-güvenliği-tıklama-rehberi)
11. [Monetization: Abonelik ve Premium Ürün Oluşturma](#11--monetization-abonelik-ve-premium-ürün-oluşturma)
12. [AAB Dosyası Hazırlama ve İmzalama](#12--aab-dosyası-hazırlama-ve-i̇mzalama)
13. [Internal Testing / Dahili Test](#13--internal-testing--dahili-test)
14. [Closed Testing / Kapalı Test — KRİTİK ADIM](#14--closed-testing--kapalı-test--kri̇ti̇k-adim)
15. [Tester Bulma Yöntemleri](#15--tester-bulma-yöntemleri)
16. [Production Erişim Başvurusu](#16--production-erişim-başvurusu)
17. [Production'a / Üretime Yayınlama](#17--productiona--üretime-yayınlama)
18. [Google İnceleme Süreci](#18--google-i̇nceleme-süreci)
19. [Yayın Sonrası: app-ads.txt ve AdMob Bağlama](#19--yayın-sonrası-app-adstxt-ve-admob-bağlama)
20. [Yayın Sonrası İzleme](#20--yayın-sonrası-i̇zleme)
21. [Güncelleme Yayınlama](#21--güncelleme-yayınlama)
22. [Sık Karşılaşılan Sorunlar ve Çözümleri](#22--sık-karşılaşılan-sorunlar-ve-çözümleri)
23. [Senaryo Bazlı Hızlı Yol Haritaları](#23--senaryo-bazlı-hızlı-yol-haritaları)
24. [Önemli Linkler](#24--önemli-linkler)

---

## 1. 📦 Ön Hazırlık Kontrol Listesi

### 🔲 Herkes İçin Zorunlu
- [ ] Google Hesabı (Gmail) — hesaba kalıcı bağlanır
- [ ] 25 USD ödeme için kredi/banka kartı (ön ödemeli kart kabul edilmez)
- [ ] Geçerli kimlik belgesi (nüfus cüzdanı / pasaport / ehliyet)
- [ ] Telefon numarası ve e-posta adresi (kullanıcılara gösterilecek)
- [ ] Google hesabında 2 Adımlı Doğrulama (2FA) aktif
- [ ] AAB dosyası (.aab formatı — APK kabul edilmiyor)
- [ ] Target API Level: Android 15 (API 35) veya üstü
- [ ] Uygulama ikonu: 512×512 px, PNG
- [ ] Feature Graphic / Öne Çıkan Grafik: 1024×500 px, JPEG veya 24-bit PNG
- [ ] Ekran görüntüleri: Min 2, max 8 (telefon). 320-3840 px, 16:9 oran
- [ ] Kısa açıklama: Max 80 karakter
- [ ] Uzun açıklama: Max 4000 karakter
- [ ] Gizlilik politikası URL'si (web sitenizde barındırılmalı)

### 🔲 Kurumsal Hesap İçin Ek
- [ ] D-U-N-S numarası (ücretsiz, onay 5-30 gün)
- [ ] Kurum yasal adı ve adresi (D-U-N-S ile birebir eşleşmeli)

### 🔲 Reklam (AdMob) Kullanacaklar İçin Ek
- [ ] AdMob hesabı (https://admob.google.com)
- [ ] AdMob App ID (`ca-app-pub-XXXX~YYYY`)
- [ ] Ad Unit ID'leri (banner, interstitial, rewarded vb.)
- [ ] app-ads.txt dosyası için geliştirici web sitesi
- [ ] UMP SDK entegrasyonu (GDPR rıza yönetimi)

### 🔲 Ücretli İçerik / Abonelik İçin Ek
- [ ] Google Play Billing Library (v7+ zorunlu, v8 önerilir)
- [ ] Merchant (Satıcı) hesabı / Ödeme profili
- [ ] Ürün ID'leri planlanmış (değiştirilemez!)
- [ ] Fiyatlandırma stratejisi (aylık/yıllık/lifetime)

### 🔲 Promo Video (Opsiyonel)
- [ ] YouTube'a yüklenmiş (unlisted/public), 30sn-2dk

> ✅ **Yaptım** → Adım 2'ye geç

---

## 2. 🗂️ Play Console Türkçe ↔ İngilizce Menü Eşleşme Tablosu

Play Console'u Türkçe'ye çevirmek için URL'ye `?hl=tr` ekle: `https://play.google.com/console?hl=tr`

### Ana Menü Yapısı

| 🇹🇷 Türkçe | 🇬🇧 English | Açıklama |
|---|---|---|
| **Tüm uygulamalar** | **All apps** | Uygulama listesi |
| **Uygulama oluştur** | **Create app** | Yeni uygulama |
| **Kontrol paneli** | **Dashboard** | Ana gösterge paneli |
| **Gelen kutusu** | **Inbox** | Bildirimler |
| **İstatistikler** | **Statistics** | Detaylı metrikler |
| **Yayınlamaya genel bakış** | **Publishing overview** | Yayın durumu |

### Sürüm / Release

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Sürüm → Üretim | Release → Production |
| Sürüm → Test etme → Dahili test | Release → Testing → Internal testing |
| Sürüm → Test etme → Kapalı test | Release → Testing → Closed testing |
| Sürüm → Test etme → Açık test | Release → Testing → Open testing |
| Sürüm → Erişim ve cihazlar | Release → Reach and devices |
| Sürümlere genel bakış | Releases overview |
| Yeni sürüm oluştur | Create new release |
| Sürümü incele | Review release |
| Üretime yayınlamaya başla | Start rollout to Production |
| Kapalı teste yayınlamaya başla | Start rollout to Closed testing |
| Dahili teste yayınlamaya başla | Start rollout to Internal testing |

### Kullanıcı Sayısını Artır / Grow Users

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Kullanıcı sayısını artır → Mağaza varlığı → Ana mağaza girişi | Grow users → Store presence → Main store listing |
| Kullanıcı sayısını artır → Mağaza varlığı → Mağaza ayarları | Grow users → Store presence → Store settings |
| Kullanıcı sayısını artır → Mağaza varlığı → Özel mağaza girişleri | Grow users → Store presence → Custom store listings |
| Kullanıcı sayısını artır → Mağaza performansı | Grow users → Store performance |

### Play ile Para Kazanma / Monetize with Play

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Play ile para kazanma → Ürünler → Abonelikler | Monetize with Play → Products → Subscriptions |
| Play ile para kazanma → Ürünler → Uygulama içi ürünler | Monetize with Play → Products → In-app products |
| Play ile para kazanma → Para kazanma kurulumu | Monetize with Play → Monetization setup |
| Play ile para kazanma → Finansal raporlar | Monetize with Play → Financial reports |
| Play ile para kazanma → Sipariş yönetimi | Monetize with Play → Order management |
| Abonelik oluştur | Create subscription |
| Temel plan ekle | Add base plan |
| Teklif ekle | Add offer |
| Fiyatları ayarla | Set prices |
| Etkinleştir | Activate |
| Ürün oluştur | Create product |

### Politika ve Programlar / Policy

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Politika ve programlar → Uygulama içeriği | Policy → App content |
| └ Gizlilik Politikası | └ Privacy Policy |
| └ Uygulama erişimi | └ App access |
| └ Reklamlar | └ Ads |
| └ İçerik derecelendirmesi | └ Content rating |
| └ Hedef kitle ve içerik | └ Target audience and content |
| └ Haber uygulaması | └ News app |
| └ Veri güvenliği | └ Data safety |
| └ Devlet uygulamaları | └ Government apps |
| └ Finansal özellikler | └ Financial features |
| └ Sağlık uygulamaları | └ Health apps |
| Başlat | Start |
| Yönet | Manage |
| Kaydet | Save |
| Gönder | Submit |

### Kalite / Quality

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Kalite → Android vitals | Quality → Android vitals |
| Kalite → Derecelendirmeler ve yorumlar | Quality → Ratings and reviews |

### Kurulum / Setup

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Kurulum → Gelişmiş ayarlar | Setup → Advanced settings |
| Kurulum → Uygulama imzalama | Setup → App signing |

### Sık Kullanılan Butonlar ve İfadeler

| 🇹🇷 Türkçe | 🇬🇧 English |
|---|---|
| Değişiklikleri kaydet | Save changes |
| İncele ve yayınla | Review and publish |
| Evet / Hayır | Yes / No |
| İleri / Sonraki | Next |
| Geri | Back |
| Başvur | Apply |
| Güncelle | Update |
| Sürüm notları | Release notes |
| Test kullanıcıları | Testers |
| E-posta listesi oluştur | Create email list |
| Katılım bağlantısını kopyala | Copy opt-in link |
| Üretime erişim için başvur | Apply for production access |

> ✅ **Yaptım, tabloyu inceledim** → Adım 3'e geç

---

## 3. 🔑 Google Play Console Hesabı Oluşturma

1. Aç: **https://play.google.com/console**
   - Türkçe için: `https://play.google.com/console?hl=tr`
2. Google hesabınla giriş yap
3. 🇹🇷 **"Başlayın"** / 🇬🇧 **"Get Started"** tıkla
4. Hesap türü seç:

| | Kişisel (Personal) | Kurumsal (Organization) |
|---|---|---|
| Kapalı test zorunlu mu? | ✅ EVET (12 tester, 14 gün) | ❌ HAYIR |
| D-U-N-S gerekli mi? | ❌ | ✅ EVET |

5. 🇹🇷 **Geliştirici Dağıtım Sözleşmesi'ni** / 🇬🇧 **Developer Distribution Agreement** kabul et
6. **25 USD** öde
7. Profil bilgilerini doldur (ad, e-posta, telefon, web sitesi)
8. Gönder

**⏳ Onay: 48 saate kadar**

> ✅ **Yaptım** → Adım 4'e geç

---

## 4. 🛡️ Kimlik Doğrulama ve Hesap Onayı

### Kişisel Hesaplar:
1. Kimlik belgesi yükle
2. Kimlikle aynı isimde kredi/banka kartı
3. Cihaz doğrulama: Play Store'dan **"Google Play Console"** uygulamasını indir → hesabınla giriş yap
4. Doğrulama tamamlanana kadar yayınlama yapılamaz

### Kurumsal Hesaplar:
1. D-U-N-S numarasını gir (9 haneli)
2. Google otomatik doğrular
3. Yasal ad ve adres birebir eşleşmeli

> ✅ **Yaptım** → Adım 5'e geç (AdMob varsa) veya Adım 6'ya geç (yoksa)

---

## 5. 📢 AdMob Hesabı ve Reklam Entegrasyonu

### 5.1 AdMob Hesabı
1. **https://admob.google.com** → kaydol/giriş
2. Hesap bilgileri + ödeme bilgileri ekle
3. Onay: genelde 24 saat

### 5.2 Uygulama Ekleme
1. 🇹🇷 Uygulamalar → Uygulama ekle / 🇬🇧 Apps → Add app
2. Platform: Android
3. 🇹🇷 "Yayınlanmamış" / 🇬🇧 "Unpublished" seç
4. Not al → **App ID:** `ca-app-pub-XXXX~YYYY`

### 5.3 Reklam Birimi (Ad Unit) Oluşturma

| Tür | Açıklama |
|-----|----------|
| **Banner** | Alt/üst sabit küçük reklam |
| **Interstitial / Geçiş reklamı** | Tam ekran |
| **Rewarded / Ödüllü** | İzle-kazan |
| **Native / Yerel** | Tasarıma uyumlu |
| **App Open / Uygulama açılışı** | Açılışta gösterilir |

Her biri için Ad Unit ID not al: `ca-app-pub-XXXX/YYYY`

### 5.4 Koda Entegrasyon

**AndroidManifest.xml:**
```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-XXXX~YYYY"/>
```

**build.gradle:**
```gradle
implementation 'com.google.android.gms:play-services-ads:23.6.0'
implementation 'com.google.android.ump:user-messaging-platform:3.1.0'
```

### 5.5 Test Reklam ID'leri (Geliştirme İçin)

| Tür | Test ID |
|-----|---------|
| Banner | `ca-app-pub-3940256099942544/6300978111` |
| Interstitial | `ca-app-pub-3940256099942544/1033173712` |
| Rewarded | `ca-app-pub-3940256099942544/5224354917` |
| Native | `ca-app-pub-3940256099942544/2247696110` |
| App Open | `ca-app-pub-3940256099942544/9257395921` |

**⚠️ Yayında gerçek ID kullan! Geliştirmede test ID kullan! Aksi hâlde hesap askıya alınır.**

> ✅ **Yaptım** → Adım 6'ya geç

---

## 6. 📱 Play Console'da Uygulama Oluşturma

📍 **Menü yolu:**
- 🇹🇷 Tüm uygulamalar → **Uygulama oluştur**
- 🇬🇧 All apps → **Create app**

Doldur:
| Alan | 🇹🇷 Türkçe | 🇬🇧 English | Detay |
|------|-----------|-------------|-------|
| Uygulama adı | Uygulama adı | App name | Max 30 karakter |
| Varsayılan dil | Varsayılan dil | Default language | Türkçe (tr-TR) |
| Uygulama/Oyun | Uygulama mı, Oyun mu? | App or Game | Seç |
| Ücretsiz/Ücretli | Ücretsiz / Ücretli | Free / Paid | ⚠️ Ücretsiz→Ücretli geçiş yok |

Beyanlar (üçünü de işaretle):
| Beyan | 🇹🇷 | 🇬🇧 |
|-------|------|------|
| ☑️ Politikalar | Uygulamam Google Play Geliştirici Program Politikaları'na uygundur | My app meets the Google Play Developer Program Policies |
| ☑️ İhracat yasaları | ABD ihracat yasalarını kabul ediyorum | I accept the US export laws |
| ☑️ Uygulama imzalama | Play Uygulama İmzalama Hizmet Şartları'nı kabul ediyorum | I accept the Play App Signing Terms of Service |

→ 🇹🇷 **"Uygulama oluştur"** / 🇬🇧 **"Create app"**

> ✅ **Yaptım** → Adım 7'ye geç

---

## 7. 🏪 Store Listing / Mağaza Girişi Hazırlama

📍 **Menü yolu:**
- 🇹🇷 Kullanıcı sayısını artır → Mağaza varlığı → **Ana mağaza girişi**
- 🇬🇧 Grow users → Store presence → **Main store listing**

### 7.1 Metin İçerikleri

| Alan | 🇹🇷 | 🇬🇧 | Limit |
|------|------|------|-------|
| Uygulama adı | Uygulama adı | App name | Max 30 karakter |
| Kısa açıklama | Kısa açıklama | Short description | Max 80 karakter |
| Tam açıklama | Tam açıklama | Full description | Max 4000 karakter |

### 7.2 Görseller

| Görsel | Boyut | Format |
|--------|-------|--------|
| 🇹🇷 Uygulama simgesi / 🇬🇧 App icon | 512×512 px | 32-bit PNG |
| 🇹🇷 Öne çıkan grafik / 🇬🇧 Feature graphic | 1024×500 px | JPEG veya 24-bit PNG |
| 🇹🇷 Ekran görüntüleri (telefon) / 🇬🇧 Screenshots | Min 2, max 8 | 320-3840px, 16:9, max 8MB |
| 🇹🇷 Tablet ekran görüntüleri / 🇬🇧 Tablet screenshots | Opsiyonel | 7" ve 10" ayrı |

### 7.3 Kategori ve İletişim

📍 **Menü yolu:**
- 🇹🇷 Kullanıcı sayısını artır → Mağaza varlığı → **Mağaza ayarları**
- 🇬🇧 Grow users → Store presence → **Store settings**

| Alan | 🇹🇷 | 🇬🇧 |
|------|------|------|
| Kategori | Uygulama kategorisi | App category |
| E-posta | İletişim e-postası | Contact email |
| Telefon | İletişim telefonu | Contact phone |
| Web sitesi | Geliştirici web sitesi | Developer website |

**⚠️ AdMob kullanıyorsan "Geliştirici web sitesi" alanını MUTLAKA doldur (app-ads.txt için gerekli).**

🇹🇷 **"Kaydet"** / 🇬🇧 **"Save"** tıkla.

> ✅ **Yaptım** → Adım 8'e geç

---

## 8. 📝 Uygulama İçeriği Ayarları — Detaylı

📍 **Menü yolu:**
- 🇹🇷 Politika ve programlar → **Uygulama içeriği**
- 🇬🇧 Policy → **App content**

Her bölümü tamamlaman **zorunlu**. "İlgilenilmesi gerekiyor" / "Needs attention" sekmesindeki tüm öğeler tamamlanmalı.

### 8.1 Gizlilik Politikası / Privacy Policy

📍 🇹🇷 Uygulama içeriği → Gizlilik Politikası → **Başlat** (veya **Yönet**)
📍 🇬🇧 App content → Privacy Policy → **Start** (or **Manage**)

- URL'yi gir (herkese açık, erişilebilir)
- 🇹🇷 **"Kaydet"** / 🇬🇧 **"Save"**

### 8.2 Uygulama Erişimi / App Access

📍 🇹🇷 Uygulama içeriği → Uygulama erişimi → **Başlat**
📍 🇬🇧 App content → App access → **Start**

| Seçenek | 🇹🇷 | 🇬🇧 | Ne zaman? |
|---------|------|------|-----------|
| A | Tüm işlevler özel erişim olmadan kullanılabilir | All functionality is available without special access | Login/paywall YOK |
| B | İşlevlerin tamamı veya bir kısmı kısıtlıdır | All or some functionality is restricted | Login/abonelik/yaş doğrulaması VAR |

**B seçtiysen:** 🇹🇷 "Yeni talimat ekle" / 🇬🇧 "Add new instructions" → Test hesap bilgileri gir (e-posta + şifre)

### 8.3 Reklamlar / Ads

📍 🇹🇷 Uygulama içeriği → Reklamlar → **Başlat**
📍 🇬🇧 App content → Ads → **Start**

| Seçenek | 🇹🇷 | 🇬🇧 | Ne zaman? |
|---------|------|------|-----------|
| Evet | Evet, uygulamam reklam içeriyor | Yes, my app contains ads | AdMob veya herhangi reklam SDK'sı varsa |
| Hayır | Hayır, uygulamam reklam içermiyor | No, my app does not contain ads | Hiç reklam yoksa |

"Evet" seçersen mağaza girişinde 🇹🇷 **"Reklam içerir"** / 🇬🇧 **"Contains ads"** etiketi gösterilir.

### 8.4 İçerik Derecelendirmesi / Content Rating

📍 🇹🇷 Uygulama içeriği → İçerik derecelendirmesi → **Başlat**
📍 🇬🇧 App content → Content rating → **Start**

1. E-posta gir
2. Kategori seç:
   - 🇹🇷 "Yardımcı program, Üretkenlik, İletişim veya Diğer" / 🇬🇧 "Utility, Productivity, Communication, or other"
   - 🇹🇷 "Oyun" / 🇬🇧 "Game"
3. Soruları yanıtla:

| Soru | 🇹🇷 | 🇬🇧 |
|------|------|------|
| Şiddet var mı? | Şiddet | Violence |
| Cinsel içerik? | Cinsellik | Sexuality |
| Küfür? | Dil | Language |
| Madde kullanımı? | Kontrollü maddeler | Controlled substances |
| Kumar? | Kumar | Gambling |
| Kullanıcı etkileşimi? | Etkileşimli öğeler | Interactive elements |
| Çevrimiçi içerik? | Çevrimiçi içerik | Online content |
| Konum paylaşımı? | Konum paylaşma | Location sharing |
| IAP var mı? | Uygulama içi satın alma | In-app purchases |
| Reklam ID kullanıyor mu? | Reklam kimliği | Advertising ID |

**Senaryo bazlı tipik cevaplar (standart uygulama/oyun):**

| Soru | A (Basit) | B (AdMob) | C-E (AdMob+IAP) | F (Ücretli) |
|------|-----------|-----------|-----------------|-------------|
| Şiddet | Hayır | Hayır | Hayır | Hayır |
| Cinsellik | Hayır | Hayır | Hayır | Hayır |
| Dil/Küfür | Hayır | Hayır | Hayır | Hayır |
| Kontrollü maddeler | Hayır | Hayır | Hayır | Hayır |
| Kumar | Hayır | Hayır | Hayır | Hayır |
| Etkileşimli öğeler | Hayır* | Hayır* | Hayır* | Hayır* |
| Çevrimiçi içerik | Hayır** | Evet | Evet | Hayır** |
| Konum paylaşma | Hayır | Hayır | Hayır | Hayır |
| IAP | Hayır | Hayır | **Evet** | Hayır |
| Reklam ID | Hayır | **Evet** | **Evet** | Hayır |

*\*Etkileşimli öğeler: Kullanıcılar arası iletişim varsa (chat, yorum vb.) → Evet*
*\*\*Çevrimiçi içerik: İnternet gerektiren içerik varsa → Evet*

⚠️ **Uygulamanın içeriğine göre dürüstçe yanıtla!** Yanlış beyan = ret/kaldırma.

4. 🇹🇷 **"Kaydet"** → **"Derecelendirmeyi uygula"**
   🇬🇧 **"Save"** → **"Apply rating"**

### 8.5 Hedef Kitle ve İçerik / Target Audience and Content

📍 🇹🇷 Uygulama içeriği → Hedef kitle ve içerik → **Başlat**
📍 🇬🇧 App content → Target audience and content → **Start**

| Soru | 🇹🇷 | 🇬🇧 | Önerilen |
|------|------|------|----------|
| Hedef yaş grubu | Hedef yaş grubu | Target age group | 18 ve üzeri (emin değilsen) |
| Çocuklara yönelik mi? | Öncelikli olarak çocuklara yönelik mi? | Primarily designed for children? | Hayır / No |

**Yaş grubu seçenekleri:**
| 🇹🇷 | 🇬🇧 | ⚠️ Not |
|------|------|--------|
| 5 yaş ve altı | Ages 5 and under | Aile politikası devreye girer! |
| 6-8 | Ages 6-8 | Aile politikası devreye girer! |
| 9-12 | Ages 9-12 | Aile politikası devreye girer! |
| 13-15 | Ages 13-15 | — |
| 16-17 | Ages 16-17 | — |
| 18 ve üzeri | 18 and over | ✅ Önerilen (emin değilsen) |

⚠️ 13 yaş altı seçersen: 🇹🇷 "Aile politikası" / 🇬🇧 "Families Policy" devreye girer! Çok sıkı kurallar, uzun inceleme süresi.

### 8.6 Haber Uygulaması / News App

📍 🇹🇷 Uygulama içeriği → Haber uygulaması → **Başlat**
📍 🇬🇧 App content → News app → **Start**

- Çoğu uygulama: 🇹🇷 **"Hayır"** / 🇬🇧 **"No"**

### 8.7 Veri Güvenliği / Data Safety — EN KRİTİK

📍 🇹🇷 Uygulama içeriği → Veri güvenliği → **Başlat**
📍 🇬🇧 App content → Data safety → **Start**

Bu bölümün detaylı senaryo bazlı tıklama rehberi **Adım 10'da**.

Genel akış:

**Sayfa 1 — Genel Bakış / Overview:**
- 🇹🇷 "İleri" / 🇬🇧 "Next" tıkla

**Sayfa 2 — Veri toplama ve güvenlik / Data collection and security:**

| Soru | 🇹🇷 | 🇬🇧 |
|------|------|------|
| Veri topluyor/paylaşıyor mu? | Uygulamanız gerekli kullanıcı veri türlerinden herhangi birini topluyor veya paylaşıyor mu? | Does your app collect or share any of the required user data types? |
| Aktarımda şifreleme? | Uygulamanız tarafından toplanan tüm kullanıcı verileri aktarım sırasında şifreleniyor mu? | Is all of the user data collected by your app encrypted in transit? |
| Veri silme yolu? | Kullanıcılara verilerinin silinmesini isteme yolu sunuyor musunuz? | Do you provide a way for users to request that their data be deleted? |

**Sayfa 3 — Veri türleri / Data types:** İlgili kutuları işaretle (Adım 10'da senaryo bazlı detay)

**Sayfa 4 — Veri kullanımı ve işleme / Data usage and handling:** Her veri türü için amaç seç

**Tüm amaç seçenekleri (Purposes):**
| 🇹🇷 | 🇬🇧 | Ne zaman? |
|------|------|----------|
| Uygulama işlevi | App functionality | Uygulama çalışması için gerekli veri |
| Analiz | Analytics | Crash, kullanım istatistikleri |
| Geliştirici iletişimleri | Developer communications | Push bildirim, e-posta gönderimi |
| Reklamcılık veya pazarlama | Advertising or marketing | AdMob, reklam gösterimi |
| Dolandırıcılık önleme, güvenlik ve uyumluluk | Fraud prevention, security, and compliance | Güvenlik amaçlı veri |
| Kişiselleştirme | Personalization | Kişiselleştirilmiş içerik/öneriler |
| Hesap yönetimi | Account management | Hesap oluşturma, giriş, profil |

**Sayfa 5 — Önizleme ve gönder / Preview and submit:**
- 🇹🇷 **"Gönder"** / 🇬🇧 **"Submit"**

### 8.8 Diğer Bölümler

| Bölüm | 🇹🇷 | 🇬🇧 | Senaryo Cevapları |
|-------|------|------|------------------|
| Devlet uygulamaları | Devlet uygulamaları | Government apps | Kişisel hesap → **kesinlikle "Hayır/No"**. Kurumsal → devlet uygulamasıysa "Evet" |
| Finansal özellikler | Finansal özellikler | Financial features | Kişisel hesap → **"Uygulamam finansal özellik sağlamıyor / My app doesn't provide any financial features"**. Kurumsal + finans uygulaması → ilgili seçenekleri işaretle |
| Sağlık uygulamaları | Sağlık uygulamaları | Health apps | Kişisel hesap → **"Uygulamam sağlık özelliği içermiyor / My app does not have any health features"**. Kurumsal + sağlık uygulaması → ilgili seçenekleri işaretle |

⚠️ **Devlet, finansal ve sağlık uygulamaları yalnızca kurumsal hesaplarla yayınlanabilir!** Kişisel hesaptaysan bu üçüne kesinlikle "Hayır" / "İçermiyor" de.

> ✅ **Yaptım** → Adım 9'a geç

---

## 9. 📊 Senaryo Bazlı Uygulama İçeriği Matrisi

Uygulamanın hangi özellikleri varsa aşağıdaki tabloyu takip et. Her satır bir "Uygulama içeriği" bölümü, her sütun bir senaryo.

### Kısaltmalar:
- **A:** Basit offline (reklamsız, satın almasız)
- **B:** AdMob reklamlı
- **C:** AdMob + Abonelik (aylık/yıllık)
- **D:** AdMob + Lifetime Premium
- **E:** AdMob + Aylık + Yıllık + Lifetime
- **F:** Ücretli uygulama (mağaza fiyatı)
- **G:** Login / Hesap sistemi olan

| Uygulama İçeriği Bölümü | A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|---|
| **Gizlilik Politikası** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Uygulama erişimi: "Tüm işlevler erişilebilir"** | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Uygulama erişimi: "Kısıtlı" + test hesabı** | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Reklamlar: "Evet"** | ❌ | ✅ | ✅ | ✅ | ✅ | ❌ | ? |
| **Reklamlar: "Hayır"** | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ? |
| **İçerik derecelendirmesi** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **İçerik derecelendirmesinde "IAP var mı" sorusu: Evet** | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ? |
| **Hedef kitle: 18+** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Haber uygulaması: Hayır** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Veri güvenliği: "Veri topluyor mu" → Hayır** | ✅ | ❌ | ❌ | ❌ | ❌ | ✅* | ❌ |
| **Veri güvenliği: "Veri topluyor mu" → Evet** | ❌ | ✅ | ✅ | ✅ | ✅ | ❌* | ✅ |

*\*F senaryosu: Ücretli uygulama ama Firebase/Analytics yoksa "Hayır" olabilir. Varsa "Evet".*

*? işareti: G senaryosunda reklam var/yok durumuna göre değişir.*

> ✅ **Yaptım, senaryomu belirledim** → Adım 10'a geç

---

## 10. 🔍 Senaryo Bazlı Veri Güvenliği Tıklama Rehberi

📍 **Menü yolu:**
- 🇹🇷 Politika ve programlar → Uygulama içeriği → Veri güvenliği → **Başlat**
- 🇬🇧 Policy → App content → Data safety → **Start**

Her senaryo için adım adım hangi kutuları işaretleyeceğin aşağıda.

---

### SENARYO A: Basit Offline Uygulama (Reklamsız, Satın Almasız, Login Yok)

**Sayfa 2 — Veri toplama ve güvenlik:**
| Soru | 🇹🇷 | 🇬🇧 | Cevap |
|------|------|------|-------|
| Veri topluyor mu? | Uygulamanız ... topluyor veya paylaşıyor mu? | Does your app collect or share...? | **❌ Hayır / No** |

→ Kalan sayfalar otomatik atlanır → 🇹🇷 **"Gönder"** / 🇬🇧 **"Submit"**

---

### SENARYO B: Ücretsiz + AdMob Reklamlı (Login Yok, IAP Yok)

**Sayfa 2 — Veri toplama ve güvenlik:**
| Soru | Cevap |
|------|-------|
| 🇹🇷 Veri topluyor/paylaşıyor mu? | **✅ Evet / Yes** |
| 🇹🇷 Aktarımda şifreleniyor mu? | **✅ Evet / Yes** |
| 🇹🇷 Veri silme yolu var mı? | **✅ Evet / Yes** (e-posta ile talep kabul ediyorsan) |

**Sayfa 3 — Veri türleri → İşaretlenecekler:**

| Veri Türü | 🇹🇷 | 🇬🇧 | Toplanan | Paylaşılan |
|-----------|------|------|---------|------------|
| ☑️ Yaklaşık konum | Konum → Yaklaşık konum | Location → Approximate location | ✅ | ✅ |
| ☑️ Uygulama etkileşimleri | Uygulama etkinliği → Uygulama etkileşimleri | App activity → App interactions | ✅ | ✅ |
| ☑️ Kilitlenme günlükleri | Uygulama bilgileri ve performans → Kilitlenme günlükleri | App info and performance → Crash logs | ✅ | ❌ |
| ☑️ Tanılama bilgileri | Uygulama bilgileri ve performans → Tanılama | App info and performance → Diagnostics | ✅ | ❌ |
| ☑️ Cihaz kimlikleri | Cihaz veya diğer kimlikler | Device or other IDs | ✅ | ✅ |

**Sayfa 4 — Her veri türü için:**

**Cihaz veya diğer kimlikler (Advertising ID):**
| Soru | 🇹🇷 | 🇬🇧 | Cevap |
|------|------|------|-------|
| Toplanan mı paylaşılan mı? | Toplanan, Paylaşılan | Collected, Shared | **İkisi de / Both** |
| Geçici mi işleniyor? | Veriler geçici olarak mı işleniyor? | Is this data processed ephemerally? | **Hayır / No** |
| Zorunlu mu? | Bu veri, uygulamanız için gerekli mi...? | Is this data required for your app...? | **Evet, veri toplama zorunludur / Yes, data collection is required** |
| Amaç | — | — | ☑️ Reklamcılık veya pazarlama / Advertising or marketing |

**Uygulama etkileşimleri:**
| Amaç | ☑️ Analiz / Analytics, ☑️ Reklamcılık / Advertising |

**Kilitlenme günlükleri + Tanılama:**
| Amaç | ☑️ Analiz / Analytics |

→ 🇹🇷 **"Gönder"** / 🇬🇧 **"Submit"**

---

### SENARYO C: AdMob + Abonelik (Aylık/Yıllık) — Login VAR

**Sayfa 2:** Senaryo B ile aynı (Evet / Evet / Evet)

**Sayfa 3 — Senaryo B'ye EK olarak işaretle:**

| Ek Veri Türü | 🇹🇷 | 🇬🇧 | Toplanan | Paylaşılan |
|--------------|------|------|---------|------------|
| ☑️ Ad | Kişisel bilgiler → Ad | Personal info → Name | ✅ | ❌ |
| ☑️ E-posta adresi | Kişisel bilgiler → E-posta adresi | Personal info → Email address | ✅ | ❌ |
| ☑️ Kullanıcı kimlikleri | Hesap bilgileri → Kullanıcı kimlikleri | Account info → Account identifiers | ✅ | ❌ |
| ☑️ Satın alma geçmişi | Finansal bilgiler → Satın alma geçmişi | Financial info → Purchase history | ✅ | ❌ |

**Sayfa 4 — Ek veri türleri için amaçlar:**

**Ad / E-posta:**
| Amaç | ☑️ Uygulama işlevi / App functionality, ☑️ Hesap yönetimi / Account management |

**Kullanıcı kimlikleri:**
| Amaç | ☑️ Uygulama işlevi / App functionality, ☑️ Hesap yönetimi / Account management |

**Satın alma geçmişi:**
| Amaç | ☑️ Uygulama işlevi / App functionality |

→ 🇹🇷 **"Gönder"** / 🇬🇧 **"Submit"**

---

### SENARYO D: AdMob + Lifetime Premium (Tek Seferlik) — Login VAR

**Senaryo C ile BİREBİR AYNI.** Lifetime da bir satın alma olduğu için "Satın alma geçmişi" işaretlenir.

---

### SENARYO E: AdMob + Aylık + Yıllık + Lifetime — Login VAR

**Senaryo C ile BİREBİR AYNI.** Birden fazla ürün türü olması formu değiştirmez — hepsi "Satın alma geçmişi" altında toplanır.

---

### SENARYO F: Ücretli Uygulama (Mağaza Fiyatı, Reklam/IAP Yok)

**Firebase Analytics yoksa → Senaryo A ile aynı (Hayır).**

**Firebase Analytics VARSA:**

**Sayfa 2:** Evet / Evet / Evet

**Sayfa 3:**
| Veri Türü | 🇹🇷 | Toplanan | Paylaşılan |
|-----------|------|---------|------------|
| ☑️ Kilitlenme günlükleri | Uygulama bilgileri → Kilitlenme günlükleri | ✅ | ❌ |
| ☑️ Tanılama | Uygulama bilgileri → Tanılama | ✅ | ❌ |
| ☑️ Cihaz kimlikleri | Cihaz veya diğer kimlikler | ✅ | ❌ |

Amaç: ☑️ Analiz / Analytics

---

### SENARYO G: Login VAR, AdMob YOK, IAP YOK

**Sayfa 2:** Evet / Evet / Evet

**Sayfa 3:**
| Veri Türü | 🇹🇷 | Toplanan | Paylaşılan |
|-----------|------|---------|------------|
| ☑️ Ad | Kişisel bilgiler → Ad | ✅ | ❌ |
| ☑️ E-posta | Kişisel bilgiler → E-posta adresi | ✅ | ❌ |
| ☑️ Kullanıcı kimlikleri | Hesap bilgileri → Kullanıcı kimlikleri | ✅ | ❌ |

Firebase Analytics de varsa: Kilitlenme günlükleri + Tanılama + Cihaz kimlikleri ekle.

Amaç: ☑️ Uygulama işlevi, ☑️ Hesap yönetimi

---

> ✅ **Yaptım, formu doldurdum** → Adım 11'e geç (IAP/Abonelik varsa) veya Adım 12'ye geç (yoksa)

---

## 11. 💰 Monetization: Abonelik ve Premium Ürün Oluşturma

**⚠️ Ön koşul:** En az bir AAB test track'ine yüklenmiş olmalı. Yoksa önce Adım 12 → 13 → sonra buraya dön.

### 11.1 Satıcı Hesabı / Merchant Account

📍 **Menü yolu:**
- 🇹🇷 Play ile para kazanma → **Para kazanma kurulumu**
- 🇬🇧 Monetize with Play → **Monetization setup**

Adımlar:
1. 🇹🇷 "Satıcı hesabı oluştur" / 🇬🇧 "Set up a merchant account"
2. İşletme bilgileri + banka hesabı gir
3. Google doğrulama tutarı gönderir → doğrula

### 11.2 Abonelik / Subscription Oluşturma

📍 **Menü yolu:**
- 🇹🇷 Play ile para kazanma → Ürünler → **Abonelikler** → **Abonelik oluştur**
- 🇬🇧 Monetize with Play → Products → **Subscriptions** → **Create subscription**

1. Bilgileri gir:
   - 🇹🇷 Ürün Kimliği / 🇬🇧 Product ID: `com.app.premium` (DEĞİŞTİRİLEMEZ!)
   - 🇹🇷 Ad / 🇬🇧 Name: "Premium"
   - 🇹🇷 Açıklama / 🇬🇧 Description
2. 🇹🇷 **"Oluştur"** / 🇬🇧 **"Create"**

#### Temel Plan / Base Plan Ekleme:

1. 🇹🇷 **"Temel plan ekle"** / 🇬🇧 **"Add base plan"**
2. Temel plan kimliği / Base Plan ID: ör. `monthly`
3. Tür / Type:

| 🇹🇷 | 🇬🇧 | Açıklama |
|------|------|----------|
| Otomatik yenilenen | Auto-renewing | Otomatik ödeme — en yaygın |
| Ön ödemeli | Prepaid | Manuel uzatma |

4. Otomatik yenilenen seçtiysen:

| Ayar | 🇹🇷 | 🇬🇧 | Öneri |
|------|------|------|-------|
| Faturalandırma dönemi | Faturalandırma dönemi | Billing period | 1 ay veya 1 yıl |
| Ek süre | Ek süre | Grace period | 7 gün |
| Hesap bekletme | Hesap bekletme | Account hold | 30 gün |
| Yeniden abone olma | Yeniden abone olma | Resubscribe | Açık |

5. Fiyat:
   - 🇹🇷 **"Fiyatları ayarla"** / 🇬🇧 **"Set prices"**
   - Varsayılan fiyat gir (ör: ₺99,99)
   - 🇹🇷 **"Güncelle"** → **"Uygula"** / 🇬🇧 **"Update"** → **"Apply"**

6. 🇹🇷 **"Etkinleştir"** / 🇬🇧 **"Activate"**

#### Birden Fazla Plan (aynı abonelik altında):

| Plan | Base Plan ID | Dönem | Fiyat |
|------|-------------|-------|-------|
| Aylık | `monthly` | 1 ay | ₺99,99/ay |
| Yıllık | `yearly` | 1 yıl | ₺799,99/yıl |

#### Teklif / Offer Ekleme (Ücretsiz Deneme / İndirim):

1. 🇹🇷 **"Teklif ekle"** / 🇬🇧 **"Add offer"**
2. Teklif kimliği / Offer ID: `free-trial-7day`
3. Uygunluk / Eligibility:

| 🇹🇷 | 🇬🇧 | Açıklama |
|------|------|----------|
| Yeni müşteri edinme | New customer acquisition | Sadece yeni kullanıcılar |
| Yükseltme | Upgrade | Mevcut aboneler |
| Geliştirici tarafından belirlenen | Developer determined | Kodda kontrol |

4. 🇹🇷 **"Aşama ekle"** / 🇬🇧 **"Add phase"**:
   - 🇹🇷 Ücretsiz deneme / 🇬🇧 Free trial (3 gün – 3 yıl)
   - 🇹🇷 Tanıtım fiyatlandırması / 🇬🇧 Introductory pricing

5. 🇹🇷 **"Kaydet"** → **"Etkinleştir"** / 🇬🇧 **"Save"** → **"Activate"**

### 11.3 Lifetime Premium (Tek Seferlik Satın Alma)

📍 **Menü yolu:**
- 🇹🇷 Play ile para kazanma → Ürünler → **Uygulama içi ürünler** → **Ürün oluştur**
- 🇬🇧 Monetize with Play → Products → **In-app products** → **Create product**

1. 🇹🇷 Ürün Kimliği / 🇬🇧 Product ID: `com.app.lifetime_premium` (DEĞİŞTİRİLEMEZ!)
2. 🇹🇷 Ad / 🇬🇧 Name: "Ömür Boyu Premium"
3. Fiyat: ör. ₺1499,99
4. 🇹🇷 **"Kaydet"** → **"Etkinleştir"** / 🇬🇧 **"Save"** → **"Activate"**

⚠️ **Lifetime = non-consumable.** `acknowledgePurchase()` çağır, `consumePurchase()` çağırMA! 3 gün acknowledge etmezsen otomatik iade!

### 11.4 Billing Library

```gradle
implementation 'com.android.billingclient:billing:7.1.1'
```
```xml
<uses-permission android:name="com.android.vending.BILLING" />
```

### 11.5 Product ID Stratejisi (Değiştirilemez!)

```
com.vrsinema.premium.monthly     → Aylık
com.vrsinema.premium.yearly      → Yıllık
com.vrsinema.premium.lifetime    → Ömür boyu
com.vrsinema.coins.100           → 100 coin (consumable)
```

> ✅ **Yaptım** → Adım 12'ye geç

---

## 12. 🔧 AAB Dosyası Hazırlama ve İmzalama

### 12.1 Gereksinimler

| | API Level | Billing Library |
|---|---|---|
| Yeni uygulama + güncelleme | API 35+ | v7.0.0+ (v8 önerilir) |

### 12.2 Yayın Öncesi Kontrol
- [ ] Test AdMob ID → gerçek ID
- [ ] AdMob App ID manifest'te doğru
- [ ] Billing izni manifest'te (IAP varsa)
- [ ] Target API 35+
- [ ] versionCode + versionName doğru
- [ ] Release build (debug değil)

### 12.3 AAB Oluşturma

**Android Studio:** `Build` → `Generate Signed Bundle / APK` → `Android App Bundle` → Keystore → `Finish`

**Flutter:** `flutter build appbundle --release`

**React Native:** `cd android && ./gradlew bundleRelease`

### 12.4 Boyut: Max 200 MB (üstü için Play Asset Delivery)

> ✅ **Yaptım** → Adım 13'e geç

---

## 13. 🧪 Internal Testing / Dahili Test

📍 **Menü yolu:**
- 🇹🇷 Sürüm → Test etme → **Dahili test** → **Yeni sürüm oluştur**
- 🇬🇧 Release → Testing → **Internal testing** → **Create new release**

1. İlk kez: Play App Signing / Uygulama imzalama kurulumu → kabul et
2. AAB sürükle-bırak yükle
3. 🇹🇷 Sürüm adı / 🇬🇧 Release name gir
4. 🇹🇷 Sürüm notları / 🇬🇧 Release notes ekle
5. 🇹🇷 **"Kaydet"** → **"Sürümü incele"** → **"Dahili teste yayınlamaya başla"**
   🇬🇧 **"Save"** → **"Review release"** → **"Start rollout to Internal testing"**

**Tester ekle:**
- 🇹🇷 Test kullanıcıları → **E-posta listesi oluştur** → Gmail ekle
- 🇬🇧 Testers → **Create email list** → add Gmail

🇹🇷 **"Katılım bağlantısını kopyala"** / 🇬🇧 **"Copy opt-in link"** → testerlara gönder

⚠️ IAP/abonelik oluşturacaksan → Adım 11'e dön (henüz yapmadıysan).

> ✅ **Yaptım** → Adım 14'e geç

---

## 14. 🔒 Closed Testing / Kapalı Test — KRİTİK ADIM

### Kimin İçin Zorunlu?
13 Kasım 2023'ten sonra oluşturulan **kişisel hesaplar.** Kurumsal → Adım 17'ye atla.

### Gereksinimler:
- Min **12 tester** opt-in
- Min **14 gün sürekli** opt-in
- Uninstall ≠ opt-out (sorun değil)
- "Leave the program" = opt-out (o kişi sayılmaz!)

📍 **Menü yolu:**
- 🇹🇷 Sürüm → Test etme → **Kapalı test** → **Parça oluştur**
- 🇬🇧 Release → Testing → **Closed testing** → **Create track**

1. 🇹🇷 Ülkeler/bölgeler seç → **Yeni sürüm oluştur** → AAB yükle
   🇬🇧 Countries/regions → **Create new release** → upload AAB
2. 🇹🇷 **"Kapalı teste yayınlamaya başla"** / 🇬🇧 **"Start rollout to Closed testing"**

**Tester ekle:**
- 🇹🇷 Test kullanıcıları → **E-posta listesi oluştur** → Min 12 Gmail (15-20 önerilir)
- 🇬🇧 Testers → **Create email list**
- 🇹🇷 **"Katılım bağlantısını kopyala"** / 🇬🇧 **"Copy opt-in link"**

**Testerlara mesaj:**
```
Merhaba! Uygulamamı test etmeni rica ediyorum.
1. Bu linke Android cihazından tıkla: [KATILIM LİNKİ]
2. "Katıl" / "Become a tester" butonuna tıkla
3. Uygulamayı Play Store'dan indir
4. ÖNEMLİ: 14 gün boyunca "Programdan ayrıl" / "Leave" butonuna TIKAMA
Teşekkürler!
```

> ✅ **Yaptım, 14 gün bekledim** → Adım 16'ya geç

---

## 15. 👥 Tester Bulma Yöntemleri

| Yöntem | Detay | Süre |
|--------|-------|------|
| Kişisel çevre | WhatsApp/Telegram'da paylaş | Anında |
| Testers Community | Play Store uygulaması, kredi kazan | ~24 saat |
| Reddit | r/AndroidDev, r/TestMyApp | 1-3 gün |
| Discord | Android dev sunucuları | 1-2 gün |
| Facebook | "Android Developer Turkey" vb. | 1-3 gün |

> ✅ **Yaptım** → Adım 16'ya geç

---

## 16. ✅ Production Erişim Başvurusu

14 gün + 12 tester opt-in → Dashboard'da:

- 🇹🇷 **"Üretime erişim için başvur"**
- 🇬🇧 **"Apply for production access"**

Soruları yanıtla → gönder. Onay: birkaç saat – birkaç gün.

> ✅ **Onaylandı** → Adım 17'ye geç

---

## 17. 🎯 Production'a / Üretime Yayınlama

📍 **Menü yolu:**
- 🇹🇷 Sürüm → **Üretim** → **Yeni sürüm oluştur**
- 🇬🇧 Release → **Production** → **Create new release**

1. AAB yükle
2. Sürüm adı + notları gir
3. 🇹🇷 **"Kaydet"** → **"Sürümü incele"** → hataları kontrol et
   🇬🇧 **"Save"** → **"Review release"**
4. 🇹🇷 **"Üretime yayınlamaya başla"** (önerilir: %20-50 staged rollout)
   🇬🇧 **"Start rollout to Production"**

> ✅ **Yaptım** → Adım 18'e geç

---

## 18. 🔍 Google İnceleme Süreci

| Durum | Süre |
|-------|------|
| Güncelleme | Birkaç saat |
| Yeni uygulama | 1-3 gün |
| Hassas kategoriler | 7 güne kadar |

⚠️ İnceleme sırasında store listing değiştirme!

| Durum | 🇹🇷 | 🇬🇧 |
|-------|------|------|
| İncelemede | İnceleniyor | In review |
| Yayın bekliyor | Yayınlanmayı bekliyor | Pending publication |
| Yayında | Yayınlandı | Published / Available 🎉 |
| Reddedildi | Reddedildi | Rejected |
| Askıda | Askıya alındı | Suspended |

> ✅ **Yayında** → Adım 19'a geç (AdMob varsa) veya Adım 20'ye geç

---

## 19. 📡 Yayın Sonrası: app-ads.txt ve AdMob Bağlama

### 19.1 AdMob'da Bağlama
AdMob → Apps → uygulamanı seç → "Link to app store" → Play Store'dan bul ve bağla

### 19.2 app-ads.txt (Ocak 2025'ten itibaren zorunlu)

1. AdMob → Apps → app-ads.txt sekmesi → snippet kopyala:
   ```
   google.com, pub-XXXX, DIRECT, f08c47fec0942fa0
   ```
2. `app-ads.txt` dosyası oluştur → snippet yapıştır
3. Web sitene yükle: `https://vr-sinema.online/app-ads.txt`
4. Play Console'da developer website ekle:
   - 🇹🇷 Mağaza varlığı → Mağaza ayarları → Mağaza girişi iletişim bilgileri → Geliştirici web sitesi URL'si
   - 🇬🇧 Store presence → Store settings → Store listing contact details → Developer website URL
5. 24 saat bekle → AdMob'da "Verified" olmalı

> ✅ **Yaptım** → Adım 20'ye geç

---

## 20. 📊 Yayın Sonrası İzleme

| Ne | 🇹🇷 Menü | 🇬🇧 Menu |
|----|----------|----------|
| Genel durum | Kontrol paneli | Dashboard |
| Metrikler | İstatistikler | Statistics |
| Yorumlar | Kalite → Derecelendirmeler ve yorumlar | Quality → Ratings and reviews |
| Crash/ANR | Kalite → Android vitals | Quality → Android vitals |
| Gelir | Play ile para kazanma → Finansal raporlar | Monetize → Financial reports |

Hedefler: Crash-free %99+, ANR %0.47 altı, Puan 4.0+

> ✅ **Yaptım** → Güncelleme için Adım 21

---

## 21. 🔄 Güncelleme Yayınlama

1. Kodda değişiklik yap → `versionCode` artır → yeni AAB oluştur
2. 📍 🇹🇷 Sürüm → Üretim → **Yeni sürüm oluştur** / 🇬🇧 Release → Production → **Create new release**
3. AAB yükle → sürüm notları → 🇹🇷 **"Üretime yayınlamaya başla"**

⚠️ Güncelleme için tekrar 12 tester/14 gün gerekmez. Staged rollout önerilir.

---

## 22. ⚠️ Sık Karşılaşılan Sorunlar ve Çözümleri

| Sorun | Çözüm |
|-------|-------|
| 🇹🇷 "Üretime erişiminiz yok" | Kapalı test: 12 tester, 14 gün |
| Version code already used | versionCode artır |
| Veri güvenliği uyarısı | SDK veri toplamalarını incele, formu güncelle |
| Unrated | İçerik derecelendirmesi anketini tamamla |
| İnceleme 7+ gün | Sabret, listing değiştirme |
| Reklam gösterilmiyor | app-ads.txt kontrol, 24 saat bekle |
| IAP çalışmıyor | Billing v7+, ürün "Active/Etkin" mi kontrol |
| Upload key kayıp | Console → Uygulama imzalama → key sıfırlama |
| AdMob askıya alındı | Test ID kullan, appeal gönder |

---

## 23. 🗺️ Senaryo Bazlı Hızlı Yol Haritaları

### A: Basit Ücretsiz (Reklamsız, IAP Yok)
```
1 → 3 → 4 → 6 → 7 → 8 → 9(A) → 10(A) → 12 → 13 → 14 → 16 → 17 → 18 → 20
```

### B: Ücretsiz + AdMob
```
1 → 3 → 4 → 5(AdMob) → 6 → 7 → 8 → 9(B) → 10(B) → 12 → 13 → 14 → 16 → 17 → 18 → 19(ads.txt) → 20
```

### C: AdMob + Aylık/Yıllık Abonelik
```
1 → 3 → 4 → 5(AdMob) → 6 → 7 → 8 → 9(C) → 10(C) → 12 → 13 → 11(Sub) → 14 → 16 → 17 → 18 → 19 → 20
```

### D: AdMob + Lifetime Premium
```
1 → 3 → 4 → 5(AdMob) → 6 → 7 → 8 → 9(D) → 10(D) → 12 → 13 → 11(One-time) → 14 → 16 → 17 → 18 → 19 → 20
```

### E: AdMob + Aylık + Yıllık + Lifetime Hepsi
```
1 → 3 → 4 → 5(AdMob) → 6 → 7 → 8 → 9(E) → 10(E) → 12 → 13 → 11(Sub+One-time) → 14 → 16 → 17 → 18 → 19 → 20
```

### F: Ücretli Uygulama (Mağaza Fiyatı)
```
1 → 3 → 4 → 6(Paid) → 7 → 8 → 9(F) → 10(F) → 12 → 13 → 14 → 16 → 17 → 18 → 20
```

### G: Kurumsal Hesap (Kapalı Test Atla)
```
1 → 3(Org) → 4 → (5/11 ihtiyaca göre) → 6 → 7 → 8 → 9 → 10 → 12 → 13 → 17 → 18 → 20
```

---

## 24. 🔗 Önemli Linkler

| Kaynak | Link |
|--------|------|
| Play Console | https://play.google.com/console |
| Play Console (Türkçe) | https://play.google.com/console?hl=tr |
| Play Console Yardım (TR) | https://support.google.com/googleplay/android-developer?hl=tr |
| Play Console Yardım (EN) | https://support.google.com/googleplay/android-developer |
| Geliştirici Politikaları | https://play.google.com/about/developer-content-policy/ |
| Target API Level | https://developer.android.com/google/play/requirements/target-sdk |
| Veri Güvenliği Rehberi (TR) | https://support.google.com/googleplay/android-developer/answer/10787469?hl=tr |
| Veri Güvenliği Rehberi (EN) | https://support.google.com/googleplay/android-developer/answer/10787469 |
| AdMob Data Disclosure | https://developers.google.com/admob/android/privacy/play-data-disclosure |
| Firebase Data Disclosure | https://firebase.google.com/docs/android/play-data-disclosure |
| Test Gereksinimleri | https://support.google.com/googleplay/android-developer/answer/14151465 |
| Abonelik Oluşturma | https://support.google.com/googleplay/android-developer/answer/140504 |
| In-App Product Oluşturma | https://support.google.com/googleplay/android-developer/answer/1153481 |
| Play Billing Library | https://developer.android.com/google/play/billing |
| Billing Entegrasyon | https://developer.android.com/google/play/billing/integrate |
| app-ads.txt Kurulum | https://support.google.com/admob/answer/9363762 |
| Uygulamayı İncelemeye Hazırlama (TR) | https://support.google.com/googleplay/android-developer/answer/9859455?hl=tr |
| Testers Community | https://play.google.com/store/apps/details?id=com.testerscommunity |
| Ödeme Politikası | https://support.google.com/googleplay/android-developer/answer/10281818 |

---

## 📌 ZAMANLAMA TAHMİNİ

| Adım | Süre |
|------|------|
| Hesap + kimlik doğrulama | 2-5 gün |
| AdMob kurulumu | 1 gün |
| Uygulama + Store listing + App content | 1-2 gün |
| Internal testing | 1 gün |
| Closed testing (kişisel hesap) | **Min 14 gün** |
| Production başvurusu | 1-3 gün |
| Google incelemesi | 1-7 gün |
| app-ads.txt doğrulaması | 24 saat |
| **TOPLAM (kişisel)** | **~3-4 hafta** |
| **TOPLAM (kurumsal)** | **~1 hafta** |

---

> 💡 Her adımı tamamlayınca "Yaptım" de → bir sonraki adıma geçelim.  
> 🗺️ Senaryonu belirle (Bölüm 23) → o yol haritasını izle.  
> 📅 Şubat 2026 güncel. Google Play politikaları sık değişir — resmi kaynakları kontrol et.
