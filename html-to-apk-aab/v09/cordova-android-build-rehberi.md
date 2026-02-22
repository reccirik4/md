# Cordova Android Uygulama Geliştirme — Proje Talimatları

## SOHBETE BAŞLARKEN
Bilgi bankasındaki `cordova-android-build-rehberi.md` ve `cordove-html-to-apk.md` dosyalarını oku. Tüm adımlar, bilinen hatalar ve çözümleri orada.

## İŞ AKIŞI

Kullanıcı bir uygulama fikri paylaştığında şu sırayla ilerle:

### ADIM 1: Tasarım
- Uygulamanın özelliklerini ve kapsamını anla
- İhtiyaç duyulacak Cordova plugin'lerini belirle
- Kullanılacak TÜM değişken isimlerini (global ve lokal) özet olarak listele:
  - Veritabanı adı, store adları, versiyon numarası
  - Global değişkenler ve fonksiyon isimleri
  - CSS class/id isimleri (önemli olanlar)
  - config.xml widget id, version, android-versionCode
- Dosya listesini sun, kullanıcıdan onay al

### ADIM 2: Dosya Oluşturma (TEK TEK)
- Her dosyayı AYRI AYRI oluşturup sun
- Her dosyadan sonra kullanıcıdan bir sonraki için onay al
- Sıralama:
  1. `config.xml`
  2. `www/index.html`
  3. `www/css/app.css`
  4. `www/js/` altındaki modüller (db.js, app.js vb.)
  5. `package.json`
  6. `build.json`
  7. `voltbuilder.json`
- Her dosya EKSIKSIZ olmalı (kısmi kod YASAK)

### ADIM 3: ZIP ve Build Talimatları
- Tüm dosyalar onaylandıktan sonra hepsini ZIP olarak sun
- ZIP yapısı (Google Play'de yayınlanan referans yapı):
  ```
  proje-adi/
  ├── config.xml
  ├── package.json
  ├── build.json
  ├── voltbuilder.json
  ├── release.keystore          ← Kullanıcı kendi keystore'unu koyacak
  ├── certificates/
  │   └── android.keystore      ← VoltBuilder için (release.keystore ile aynı dosya olabilir)
  ├── res/
  │   └── icon.png              ← 512x512+ PNG ikon
  ├── resources/
  │   ├── iconTemplate.png      ← 1024x1024 (VoltBuilder ikon üretimi için)
  │   └── splashTemplate.png    ← Splash ekranı görseli
  └── www/
      ├── index.html
      ├── css/
      │   └── app.css
      ├── js/
      │   ├── db.js
      │   ├── app.js
      │   └── (diğer modüller)
      ├── lib/                  ← Harici kütüphanelerin LOKAL kopyaları
      │   ├── firebase-app-compat.js
      │   ├── firebase-auth-compat.js
      │   ├── firebase-database-compat.js
      │   └── chart.umd.min.js
      ├── fonts/                ← Özel fontlar (varsa)
      └── img/                  ← Uygulama görselleri (varsa)
  ```
- ZIP'te tüm klasörleri OLUŞTUR, ikon/keystore dosyalarını kullanıcıya hatırlat
- Build talimatlarını ver (lokal + VoltBuilder)

## KESİN KURALLAR

---

### config.xml — ŞABLON

```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.gelistirici.uygulamaadi" version="1.0.0" android-versionCode="10000"
  xmlns="http://www.w3.org/ns/widgets"
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:cdv="http://cordova.apache.org/ns/1.0">

    <n>Uygulama Adı</n>
    <description>Uygulama açıklaması</description>
    <author email="gelistirici@gmail.com" href="https://example.com">
        Geliştirici Adı
    </author>
    <content src="index.html" />

    <icon src="res/icon.png" />

    <!-- ===== GENEL TERCİHLER ===== -->
    <preference name="StatusBarOverlaysWebView" value="false" />
    <preference name="StatusBarBackgroundColor" value="#1565C0" />
    <preference name="StatusBarStyle" value="lightcontent" />
    <preference name="BackgroundColor" value="0xff1565C0" />
    <preference name="Orientation" value="portrait" />
    <preference name="DisallowOverscroll" value="true" />
    <preference name="KeyboardDisplayRequiresUserAction" value="false" />
    <preference name="android-minSdkVersion" value="24" />
    <preference name="android-targetSdkVersion" value="35" />

    <!-- ===== SPLASH ===== -->
    <preference name="SplashScreenDelay" value="2000" />
    <preference name="AutoHideSplashScreen" value="true" />
    <preference name="AndroidWindowSplashScreenAnimatedIcon" value="resources/splashTemplate.png" />
    <preference name="AndroidWindowSplashScreenBackground" value="#1565C0" />

    <!-- ===== ERİŞİM ===== -->
    <access origin="*" />
    <allow-intent href="http://*/*" />
    <allow-intent href="https://*/*" />
    <allow-intent href="tel:*" />
    <allow-intent href="sms:*" />
    <allow-intent href="mailto:*" />
    <allow-intent href="market:*" />
    <!-- Firebase kullanılıyorsa: -->
    <allow-navigation href="https://*.firebaseapp.com/*" />
    <allow-navigation href="https://*.googleapis.com/*" />

    <!-- ===== PLUGİNLER ===== -->
    <!-- Projede kullanılacak plugin'ler buraya -->
    <plugin name="cordova-plugin-device" spec="*" />
    <plugin name="cordova-plugin-statusbar" spec="*" />
    <!-- ÖNEMLİ: cordova-plugin-splashscreen EKLEME (cordova-android 14 ile uyumsuz) -->
    <!-- ÖNEMLİ: Social sharing = cordova-plugin-x-socialsharing -->

    <!-- ===== ANDROID PLATFORM ===== -->
    <platform name="android">
        <preference name="GradlePluginKotlinEnabled" value="true" />
        <preference name="AndroidXEnabled" value="true" />
        <config-file parent="/manifest" target="AndroidManifest.xml">
            <uses-permission android:name="android.permission.INTERNET" />
            <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
            <!-- Bildirim plugin'i kullanılıyorsa: -->
            <!-- <uses-permission android:maxSdkVersion="32" android:name="android.permission.SCHEDULE_EXACT_ALARM" /> -->
            <!-- <uses-permission android:name="android.permission.USE_EXACT_ALARM" /> -->
            <!-- <uses-permission android:name="android.permission.POST_NOTIFICATIONS" /> -->
            <!-- <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" /> -->
            <!-- <uses-permission android:name="android.permission.VIBRATE" /> -->
            <!-- <uses-permission android:name="android.permission.WAKE_LOCK" /> -->
            <!-- In-app purchase kullanılıyorsa: -->
            <!-- <uses-permission android:name="com.android.vending.BILLING" /> -->
        </config-file>
    </platform>

    <!-- ===== iOS PLATFORM (opsiyonel) ===== -->
    <platform name="ios">
        <preference name="WKWebViewOnly" value="true" />
        <preference name="deployment-target" value="13.0" />
    </platform>

    <engine name="android" spec="14.0.1" />
</widget>
```

**config.xml KURALLARI:**
- `<icon src="res/icon.png" />` MUTLAKA ekle (www/ altına ikon KOYMA)
- `cordova-plugin-splashscreen` EKLEME (cordova-android 14 ile uyumsuz, native splash kullanılıyor)
- Social sharing plugin adı: `cordova-plugin-x-socialsharing`
- `android-targetSdkVersion` = 35
- `<engine name="android" spec="14.0.1" />`
- `BackgroundColor` format: `0xffRRGGBB` (0xff prefix + hex renk)
- `android-versionCode` ZORUNLU: Google Play her yüklemede artan bir versionCode ister. Format: version x.y.z → versionCode = x*10000 + y*100 + z (örn: 1.5.0 → 10500)
- `xmlns:android` namespace'i ZORUNLU: AndroidManifest config-file için gerekli
- `<platform name="android">` bloğu içinde `GradlePluginKotlinEnabled` ve `AndroidXEnabled` ZORUNLU
- Manifest permission'ları `<config-file>` ile eklenir, doğrudan config.xml'e DEĞİL

---

### package.json — ŞABLON

```json
{
  "devDependencies": {
    "cordova-android": "^14.0.1",
    "cordova-plugin-device": "*",
    "cordova-plugin-statusbar": "*"
  },
  "cordova": {
    "platforms": [
      "android"
    ],
    "plugins": {
      "cordova-plugin-device": {},
      "cordova-plugin-statusbar": {}
    }
  }
}
```

**package.json KURALLARI:**
- Her plugin hem `devDependencies` hem `cordova.plugins` içinde olmalı
- Plugin'lerin özel variable'ları varsa `cordova.plugins` altında belirtilmeli (örn: admob APP_ID, local-notification ANDROIDX_CORE_VERSION vb.)
- `cordova-android` versiyonu `"^14.0.1"` olmalı

---

### build.json — ŞABLON (Lokal Release Build İçin)

```json
{
  "android": {
    "release": {
      "keystore": "release.keystore",
      "storePassword": "KULLANICI_SIFRESI",
      "alias": "KULLANICI_ALIAS",
      "password": "KULLANICI_SIFRESI",
      "keystoreType": "jks"
    }
  }
}
```

**build.json KURALLARI:**
- Sadece LOKAL release build için kullanılır
- Keystore dosyası proje kökünde olmalı (path: `release.keystore`)
- VoltBuilder bu dosyayı KULLANMAZ (voltbuilder.json kullanır)
- Şifre ve alias değerleri kullanıcının kendi keystore bilgileri

---

### voltbuilder.json — ŞABLON

```json
{
  "androidAlias": "KULLANICI_ALIAS",
  "androidAliasPassword": "KULLANICI_SIFRESI",
  "androidKeystore": "certificates/android.keystore",
  "androidKeystorePassword": "KULLANICI_SIFRESI",
  "release": "release"
}
```

**voltbuilder.json KURALLARI:**
- Keystore dosyası `certificates/` klasörü altında olmalı
- `"release": "release"` → release build üretir
- AAB (Google Play bundle) istiyorsa: `"androidPackageType": "bundle"` ekle
- Google Play'e otomatik yükleme istiyorsa: `"googlePlayReleaseStatus": "completed"` ekle (diğer seçenekler: draft, halted, inProgress)
- VoltBuilder JSON dosyalarında yorum (`//`) ve trailing comma destekler

---

### index.html — ŞABLON (HEAD bölümü)

```html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <meta name="theme-color" content="#1976D2">
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="format-detection" content="telephone=no">
  <meta name="msapplication-tap-highlight" content="no">
  <meta http-equiv="Content-Security-Policy" content="
    default-src 'self' https://*.firebaseapp.com https://*.googleapis.com
      https://*.firebaseio.com https://*.firebasedatabase.app https://*.google.com;
    script-src 'self' 'unsafe-inline' 'unsafe-eval';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    connect-src 'self'
      https://*.firebaseio.com https://*.firebasedatabase.app
      wss://*.firebaseio.com wss://*.firebasedatabase.app
      https://*.googleapis.com https://*.firebaseapp.com
      https://pagead2.googlesyndication.com;">
  <title>Uygulama Adı</title>
  <link rel="stylesheet" href="css/app.css">
</head>
```

**index.html HEAD KURALLARI:**
- `viewport-fit=cover` → notch'lu cihazlarda düzgün görünüm
- `maximum-scale=1.0, user-scalable=no` → kullanıcı zoom yapmasın
- `format-detection: telephone=no` → numaraların otomatik link olmasını engelle
- `msapplication-tap-highlight: no` → tap highlight'ı kaldır
- CSP (Content-Security-Policy) → kullanılan servislere göre özelleştir
  - Firebase kullanmıyorsan Firebase domain'lerini çıkar
  - AdMob kullanmıyorsan `pagead2.googlesyndication.com` çıkar
  - Harici API kullanıyorsan `connect-src`'ye ekle

---

### index.html — SCRIPT YÜKLEME SIRASI (BODY sonunda)

```html
  <!-- ===== COLD START: Plugin event'leri otomatik ateşlemesini engelle ===== -->
  <!-- cordova-plugin-local-notification kullanılıyorsa: -->
  <script>window.skipLocalNotificationReady = true;</script>

  <!-- ===== 1. cordova.js: projede fiziksel bulunmaz, build sırasında otomatik eklenir ===== -->
  <script src="cordova.js"></script>

  <!-- ===== 2. Harici Kütüphaneler: LOKAL KOPYA (CDN'den DEĞİL!) ===== -->
  <script src="lib/firebase-app-compat.js"></script>
  <script src="lib/firebase-auth-compat.js"></script>
  <script src="lib/firebase-database-compat.js"></script>
  <script src="lib/chart.umd.min.js"></script>

  <!-- ===== 3. Uygulama Modülleri ===== -->
  <script src="js/db.js"></script>
  <script src="js/app.js"></script>

</body>
</html>
```

**SCRIPT YÜKLEME KURALLARI:**
1. `cordova.js` HER ZAMAN ilk sırada (diğer JS'lerden ÖNCE)
2. `cordova.js` dosyası projede FİZİKSEL BULUNMAZ — build sırasında otomatik eklenir, www/ altına koyma
3. Harici kütüphaneler (Firebase, Chart.js vb.) LOKAL KOPYA olarak `www/lib/` altında — CDN KULLANMA (çünkü offline çalışma gerekli)
4. Uygulama modülleri en sonda, bağımlılık sırasına göre (önce db.js, sonra app.js gibi)
5. `skipLocalNotificationReady` → cordova-plugin-local-notification kullanılıyorsa, handler'lar register olmadan event ateşlenmesini engeller

---

### HARİCİ KÜTÜPHANE KURALI (www/lib/)

Cordova uygulamalarında harici kütüphaneler CDN'den DEĞİL, lokal kopya olarak kullanılmalı:
- **Neden:** Uygulama offline çalışabilmeli, CDN erişimi garanti değil
- **Nasıl:** Kütüphanenin minified dosyasını indir, `www/lib/` altına koy
- **Firebase SDK:** `firebase-app-compat.js`, `firebase-auth-compat.js`, `firebase-database-compat.js` (compat sürümü — modüler değil)
- **Chart.js:** `chart.umd.min.js`
- **Diğer:** İhtiyaca göre ekle

---

### CSP (Content-Security-Policy) — REFERANS

Projede kullanılan servislere göre CSP özelleştirilmeli:

| Servis | Eklenecek Domain'ler |
|--------|---------------------|
| Firebase Auth | `https://*.firebaseapp.com`, `https://*.googleapis.com` |
| Firebase Realtime DB | `https://*.firebaseio.com`, `https://*.firebasedatabase.app`, `wss://*.firebaseio.com`, `wss://*.firebasedatabase.app` |
| Firebase Firestore | `https://firestore.googleapis.com` |
| AdMob | `https://pagead2.googlesyndication.com` |
| Google Maps | `https://maps.googleapis.com` |
| Harici API | İlgili domain (connect-src'ye) |

**Minimal CSP (Firebase yok):**
```
default-src 'self';
script-src 'self' 'unsafe-inline' 'unsafe-eval';
style-src 'self' 'unsafe-inline';
img-src 'self' data: https:;
connect-src 'self';
```

---

### ALARM/BİLDİRİM PERMISSION'LARI — REFERANS

Android 12+ (SDK 31+) ile alarm permission'ları değişti:

```xml
<!-- SDK 31 ve altı için: -->
<uses-permission android:maxSdkVersion="32" android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<!-- SDK 33+ için: -->
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
<!-- Android 13+ bildirim izni: -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<!-- Cihaz yeniden başladığında alarm'ları geri yükle: -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

**ÖNEMLİ:** `SCHEDULE_EXACT_ALARM` SDK 33+'da Google Play policy gereği yasak (alarm/saat uygulamaları hariç). Onun yerine `USE_EXACT_ALARM` kullan. `maxSdkVersion="32"` ile iki permission'ı birlikte kullanmak en güvenli yol.

---

## Encoding
- Tüm dosyalar UTF-8 (BOM olmadan)
- Türkçe karakterler: ğüşıöçĞÜŞİÖÇ

---

## LOKAL BUILD TALİMATLARI

ZIP sunduktan sonra şu adımları bildir:

```
GEREKSINIMLER:
  - Node.js 20.5.0+ (zorunlu — cordova-android 14 gereksinimidir)
  - JDK 17+ (zorunlu — cordova-android 14 gereksinimidir)
  - Android SDK Platform 35 + Build Tools 35.0.0
  - Android Studio Ladybug+ (opsiyonel — sadece Android Studio'dan build yapılacaksa)
  - Cordova CLI: npm install -g cordova

ADIMLAR:
1. ZIP'i bir klasöre aç (örn: E:\projeler\uygulama-adi)
   ⚠️ Klasör yolunda BOŞLUK OLMAMALI (cordova-android 14.0.0 bug'ı, 14.0.1'de düzeltildi ama dikkat)

2. res/icon.png olarak 512x512+ PNG ikonunu koy
3. (VoltBuilder için) resources/iconTemplate.png (1024x1024) koy

4. PowerShell'de:
   cd E:\projeler\uygulama-adi
   cordova platform add android

5. ⚠️ KRİTİK — appcompat düzeltmesi (SADECE lokal build):
   cordova-android 14.0.1 varsayılan olarak appcompat 1.7.0 kullanır.
   Bazı plugin'lerle (özellikle local-notification) çakışma yaşanabilir.
   Sorun çıkarsa:
   (Get-Content "platforms\android\cdv-gradle-config.json") -replace '"1.7.0"','"1.5.1"' | Set-Content "platforms\android\cdv-gradle-config.json"

   NOT: cordova-android master branch'ta appcompat 1.7.1'e güncellendi.
   Gelecek sürümlerde bu fix gerekli olmayabilir.

6. Debug Build:
   cordova build android

7. Telefona yükle:
   adb install platforms\android\app\build\outputs\apk\debug\app-debug.apk

8. Release APK (build.json ile):
   cordova build android --release

9. Release AAB (Google Play için):
   cordova build android --release -- --packageType=bundle

10. İmzalı APK konumu:
    platforms\android\app\build\outputs\apk\release\app-release.apk
    
11. İmzalı AAB konumu:
    platforms\android\app\build\outputs\bundle\release\app-release.aab
```

---

## VOLTBUILDER TALİMATLARI

```
1. ZIP'i hazırla:
   - Kökte: config.xml, package.json, voltbuilder.json
   - Klasörler: www/, res/, resources/, certificates/
   - certificates/android.keystore dosyası (release build için)
   
   NOT: build.json ve release.keystore VoltBuilder tarafından KULLANILMAZ.
   ZIP'e dahil olsalar da sorun çıkarmaz.

2. res/icon.png VEYA resources/iconTemplate.png (1024x1024) koy

3. https://volt.build → Giriş → ZIP'i Android kutusuna sürükle

4. APK/AAB indir

AAB için: voltbuilder.json'a "androidPackageType": "bundle" ekle
```

---

## cdv-gradle-config.json — REFERANS

`cordova platform add android` sonrası `platforms/android/cdv-gradle-config.json` otomatik oluşur.
cordova-android 14.0.1 varsayılan değerleri:

| Parametre | Değer |
|-----------|-------|
| SDK_VERSION | 35 |
| GRADLE_VERSION | 8.11 |
| AGP_VERSION | 8.3.0 |
| KOTLIN_VERSION | 1.9.24 |
| ANDROIDX_APP_COMPAT_VERSION | 1.7.0 |
| ANDROIDX_WEBKIT_VERSION | 1.6.0 |
| ANDROIDX_CORE_SPLASHSCREEN_VERSION | 1.0.0 |
| JAVA_SOURCE_COMPATIBILITY | 11 |
| JAVA_TARGET_COMPATIBILITY | 11 |

Master branch (gelecek sürüm) değerleri:

| Parametre | Değer |
|-----------|-------|
| GRADLE_VERSION | 8.14.2 |
| AGP_VERSION | 8.10.1 |
| KOTLIN_VERSION | 2.1.21 |
| ANDROIDX_APP_COMPAT_VERSION | 1.7.1 |
| ANDROIDX_WEBKIT_VERSION | 1.14.0 |
| ANDROIDX_CORE_SPLASHSCREEN_VERSION | 1.0.1 |

Bu dosyayı elle düzenlemek mümkün ama dikkatli olunmalı. `cordova platform remove/add android` yapıldığında varsayılana döner.

---

## DEĞİŞKEN İSİMLERİ KURALI
- Proje başında TÜM değişken isimlerini listele
- Dosyalar arası tutarlılık sağla
- Sonradan değişken adı değiştirme YASAK (kullanıcı izni olmadan)
- config.xml widget id, Firebase config, API key gibi sabitler DOKUNULMAZ

---

## BİLİNEN SORUNLAR VE ÇÖZÜMLER

### 1. appcompat Çakışması (Lokal Build)
**Sorun:** cordova-android 14 appcompat 1.7.0 kullanır, bazı plugin'ler eski sürüm ister.
**Çözüm:** `cdv-gradle-config.json` içinde `ANDROIDX_APP_COMPAT_VERSION` değerini düşür.
**NOT:** VoltBuilder'da bu sorun genelde oluşmaz.

### 2. Gradle Daemon Lock
**Sorun:** Build sırasında "Could not create service" veya lock hatası.
**Çözüm:**
```powershell
taskkill /F /IM java.exe
# Sonra tekrar build
cordova build android
```

### 3. Yol Boşluk Sorunu
**Sorun:** cordova-android 14.0.0'da proje yolunda boşluk varsa build başarısız.
**Çözüm:** 14.0.1'e güncelle veya yolda boşluk kullanma.

### 4. Edge-to-Edge (Android 15+)
**Durum:** Android 15 Edge-to-Edge'i varsayılan yapar. cordova-android 14 bunu OPT-OUT ediyor.
**Uyarı:** Android 16'da opt-out kalkacak, uygulamaların Edge-to-Edge desteklemesi gerekecek.
**Hazırlık:** `viewport-fit=cover` meta tag'ını şimdiden ekle (şablonda zaten var).

### 5. JAVA_HOME Sorunu
**Sorun:** Gradle, sistem JAVA_HOME'u bulamıyor.
**Çözüm:** cordova-android 14.0.1'de düzeltildi. `JAVA_HOME` veya `CORDOVA_JAVA_HOME` ortam değişkenini ayarla.

### 6. cordova platform remove/add Sonrası
**Hatırlatma:** Platform yeniden eklendiğinde:
- appcompat düzeltmesi tekrarlanmalı (gerekiyorsa)
- cdv-gradle-config.json varsayılana döner
- Plugin'ler otomatik yeniden kurulur

### 7. İlk Build Uzun Sürer
İlk lokal build 3-5 dakika sürebilir (Gradle wrapper + bağımlılık indirme). Sonraki build'ler çok daha hızlı.

---

## ÖNEMLİ NOTLAR
- Kullanıcının sistemi: Windows 10 Pro, VS Code, RTX 3050 laptop
- Kullanıcının domaini: vr-sinema.online (Namecheap)
- Kullanıcının CORS proxy'si: https://mycors.recepyeni.workers.dev
- Kullanıcının GitHub Pages: https://recinilt.github.io/mefeypublic/recinilt/index.html
