# Cordova ile Android APK/AAB Oluşturma Rehberi

> Sıfırdan Cordova projesi oluşturup hem lokal bilgisayarda hem VoltBuilder ile APK/AAB build edip Google Play'e yüklemeye hazır hale getirme rehberi. Bilinen hatalar ve çözümleri dahildir.

> ⚠️ **MD GÜNCELLEME KURALI:** Bu dosyayı hemen güncelleme. Önce çözümü uygula ve test et. "İşe yaradı mı çözüm, md dosyasını güncelleyeyim mi?" diye sor. Kullanıcı "evet işe yaradı, md'yi güncelle" derse ancak o zaman güncelle.

---

## 1. Gerekli Yazılımlar (Lokal Build İçin)

> ⚠️ VoltBuilder kullanıyorsan bu bölümü ATLAYIP doğrudan **Bölüm 2**'ye geç. VoltBuilder kendi sunucusunda build yapar, lokal kurulum gerektirmez.

### 1.1 Node.js (v20+)
- **İndir:** https://nodejs.org → LTS sürümü (Windows x64 .msi)
- Kurulumda **"Add to PATH"** seçili olsun
- **Kontrol:**
```
node -v
npm -v
```

### 1.2 JDK 17 (Zorunlu — JDK 8, 11, 21, 25 ÇALIŞMAZ)
- **İndir:** https://adoptium.net/temurin/releases/?version=17&package=jdk → Windows x64 .msi
- Kurulumda **"Set JAVA_HOME"** seçeneğini mutlaka işaretle
- **Kontrol:**
```
java -version
```
`openjdk version "17.0.x"` görmeli.

> ⚠️ cordova-android 14 kesinlikle JDK 17 ister. Başka versiyon ile build başarısız olur.

### 1.3 Android Studio (SDK için)
- **İndir:** https://developer.android.com/studio
- İlk açılışta **"Standard Setup"** seç
- **SDK Manager** aç: `File → Settings → Languages & Frameworks → Android SDK`

**SDK Platforms sekmesi:**
- ✅ Android 15.0 (API 35)

**SDK Tools sekmesi:**
- ✅ Android SDK Build-Tools
- ✅ Android SDK Command-line Tools
- ✅ Android SDK Platform-Tools
- **Apply** → indir

### 1.4 Ortam Değişkenleri (Windows)
Windows Arama → **"Ortam değişkenleri"** → **"Sistem ortam değişkenlerini düzenleyin"**

| Değişken | Değer (örnek) |
|---|---|
| `JAVA_HOME` | `C:\Program Files\Eclipse Adoptium\jdk-17.0.18.8-hotspot` |
| `ANDROID_HOME` | `C:\Users\KULLANICI\AppData\Local\Android\Sdk` |

**Path** değişkenine ekle:
```
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\cmdline-tools\latest\bin
%ANDROID_HOME%\build-tools\35.0.0
```

PowerShell'i kapat/aç sonra kontrol et:
```powershell
echo $env:JAVA_HOME
echo $env:ANDROID_HOME
java -version
adb version
```

### 1.5 Apache Cordova CLI
```
npm install -g cordova
cordova -v
```

---

## 2. Proje Dosya Yapısı

```
proje-adi/
├── config.xml                  ← Cordova konfigürasyonu
├── package.json                ← Cordova plugin metadata + devDependencies
├── build.json                  ← Lokal release build signing bilgileri
├── voltbuilder.json            ← VoltBuilder release build (opsiyonel)
├── release.keystore            ← Lokal release build keystore dosyası
├── res/                        ← İkon dosyaları (config.xml ile referanslanır)
│   └── icon.png                ← Uygulama ikonu (en az 512x512, ideal 1024x1024)
├── resources/                  ← VoltBuilder otomatik ikon/splash üretimi
│   ├── iconTemplate.png        ← 1024x1024 (VoltBuilder tüm boyutlara çevirir)
│   └── splashTemplate.png      ← 2732x2732 (VoltBuilder splash üretir, opsiyonel)
├── certificates/               ← VoltBuilder release build sertifikaları
│   └── android.keystore        ← release.keystore ile aynı dosya olabilir
└── www/                        ← Web uygulaması dosyaları
    ├── index.html
    ├── css/
    │   └── app.css
    ├── js/
    │   ├── app.js
    │   └── (diğer modüller)
    ├── lib/                    ← Harici JS kütüphaneleri (Firebase vb.) LOKAL KOPYA
    │   ├── firebase-app-compat.js
    │   ├── firebase-auth-compat.js
    │   └── firebase-database-compat.js
    ├── fonts/                  ← Google Fonts vb. LOKAL woff2 dosyaları
    │   └── (font dosyaları)
    └── img/                    ← Uygulamanın İÇİNDEKİ görseller (ikon DEĞİL!)
        └── logo.png
```

### ⚠️ KRİTİK: İKON KONUMU
- Uygulama ikonu `res/` klasöründe olmalı ve `config.xml`'de `<icon>` tag'ı ile tanımlanmalı
- `www/img/` klasörü sadece uygulamanın İÇİNDEKİ görseller içindir (HTML'de `<img>` ile kullanılanlar)
- `www/` altına koyulan ikon dosyası Cordova tarafından uygulama ikonu olarak TANINMAZ

### ⚠️ KRİTİK: HARİCİ BAĞIMLILIKLAR (CDN)
- Firebase SDK, Google Fonts gibi harici kütüphaneleri **CDN'den DEĞİL, lokal dosya olarak** yükle
- Cordova WebView'da internet bağlantısı garanti değildir — CDN'den yüklenen script başarısız olursa tüm uygulama çöker
- Firebase SDK → `www/lib/` klasörüne indir: `firebase-app-compat.js`, `firebase-auth-compat.js`, `firebase-database-compat.js`
- Google Fonts → `www/fonts/` klasörüne woff2 olarak indir, CSS'de `@font-face` ile tanımla
- CSS'de mutlaka fallback font belirt: `font-family: 'CustomFont', Georgia, serif;`

---

## 3. config.xml Şablonu

```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.sirket.uygulamaadi" version="1.0.0" android-versionCode="10000"
  xmlns="http://www.w3.org/ns/widgets"
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:cdv="http://cordova.apache.org/ns/1.0">

  <n>Uygulama Adı</n>
  <description>Uygulama açıklaması</description>
  <author email="info@example.com" href="https://example.com">
    Geliştirici Adı
  </author>
  <content src="index.html" />

  <!-- === İKON === -->
  <icon src="res/icon.png" />

  <!-- === TERCIHLER === -->
  <preference name="StatusBarOverlaysWebView" value="false" />
  <preference name="StatusBarBackgroundColor" value="#1565C0" />
  <preference name="StatusBarStyle" value="lightcontent" />
  <preference name="BackgroundColor" value="0xff1565C0" />
  <preference name="Orientation" value="portrait" />
  <preference name="DisallowOverscroll" value="true" />
  <preference name="KeyboardDisplayRequiresUserAction" value="false" />
  <preference name="android-minSdkVersion" value="24" />
  <preference name="android-targetSdkVersion" value="35" />

  <!-- === SPLASH SCREEN (cordova-android 12+ core) === -->
  <preference name="SplashScreenDelay" value="2000" />
  <preference name="AutoHideSplashScreen" value="true" />
  <preference name="AndroidWindowSplashScreenAnimatedIcon" value="resources/splashTemplate.png" />
  <preference name="AndroidWindowSplashScreenBackground" value="#1565C0" />

  <!-- === ERİŞİM === -->
  <access origin="*" />
  <allow-intent href="http://*/*" />
  <allow-intent href="https://*/*" />
  <allow-intent href="tel:*" />
  <allow-intent href="sms:*" />
  <allow-intent href="mailto:*" />
  <allow-intent href="market:*" />

  <!-- Firebase kullanıyorsan: -->
  <allow-navigation href="https://*.firebaseapp.com/*" />
  <allow-navigation href="https://*.googleapis.com/*" />

  <!-- === PLUGİNLER === -->
  <plugin name="cordova-plugin-device" spec="*" />
  <plugin name="cordova-plugin-statusbar" spec="*" />
  <plugin name="cordova-plugin-file" spec="*" />
  <plugin name="cordova-plugin-inappbrowser" spec="*" />
  <plugin name="cordova-android-movetasktoback" spec="*" />

  <!-- === ANDROİD PLATFORM === -->
  <platform name="android">
    <preference name="GradlePluginKotlinEnabled" value="true" />
    <preference name="AndroidXEnabled" value="true" />
    <config-file target="AndroidManifest.xml" parent="/manifest">
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
      <!-- Bildirim plugin'i kullanılıyorsa: -->
      <uses-permission android:maxSdkVersion="32" android:name="android.permission.SCHEDULE_EXACT_ALARM" />
      <uses-permission android:name="android.permission.USE_EXACT_ALARM" />
      <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
      <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
      <uses-permission android:name="android.permission.VIBRATE" />
      <uses-permission android:name="android.permission.WAKE_LOCK" />
      <!-- In-app purchase kullanılıyorsa: -->
      <!-- <uses-permission android:name="com.android.vending.BILLING" /> -->
    </config-file>
  </platform>

  <!-- === MOTOR (ZORUNLU — eksik olursa uyumsuz platform yüklenebilir) === -->
  <engine name="android" spec="14.0.1" />
</widget>
```

> ⚠️ **`android-versionCode` ZORUNLU:** Google Play her yükleme için artan bir versionCode ister. Hesaplama: `x.y.z → x*10000 + y*100 + z` (örn: 1.5.0 → 10500). Her güncelleme yüklemesinde artırılmalı.

> ⚠️ **`xmlns:android` ZORUNLU:** Widget tag'ında `xmlns:android` namespace'i olmazsa manifest permission'lar yazılamaz.

> ⚠️ **Alarm Permission'lar (Android 12+):** `SCHEDULE_EXACT_ALARM` maxSdkVersion=32 ile sınırlı (Android 12-12L). Android 13+ için `USE_EXACT_ALARM` kullanılıyor. İkisi birlikte tanımlanmalı — Cordova merge sırasında doğru olanı seçer.

> ⚠️ **Splash path:** `resources/splashTemplate.png` kullan (`res/screen/android/` DEĞİL!). VoltBuilder bu dosyadan tüm boyutları üretir.

---

## 4. index.html Şablonu

```html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <meta name="format-detection" content="telephone=no">
  <meta name="msapplication-tap-highlight" content="no">
  <meta name="theme-color" content="#1565C0">
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

  <!-- CSP — Firebase + AdMob kullanan proje şablonu (kullanılmayan servisleri çıkar) -->
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
      https://pagead2.googlesyndication.com">

  <title>Uygulama Adı</title>
  <!-- Fontlar: LOKAL yükle, CDN KULLANMA -->
  <link rel="stylesheet" href="css/app.css">
</head>
<body>
  <!-- İçerik -->

  <!-- cordova.js: projede fiziksel olarak bulunmaz, build sırasında Cordova otomatik ekler -->
  <script src="cordova.js"></script>

  <!-- Bildirim plugin'i kullanılıyorsa — cordova.js'den SONRA, app kodundan ÖNCE -->
  <script>
    // cordova-plugin-local-notification skipReady — plugin kendi deviceready'sini beklemez
    window.skipLocalNotificationReady = true;
  </script>

  <!-- Firebase SDK: LOKAL yükle (CDN'den DEĞİL — çökme sebebi!) -->
  <script src="lib/firebase-app-compat.js"></script>
  <script src="lib/firebase-auth-compat.js"></script>
  <script src="lib/firebase-database-compat.js"></script>

  <!-- Uygulama modülleri — sıra önemli: db.js (logger+firebase) → i18n → notifications → app -->
  <script src="js/db.js"></script>
  <script src="js/i18n.js"></script>
  <script src="js/notifications.js"></script>
  <script src="js/app.js"></script>
</body>
</html>
```

> ⚠️ **CDN KULLANMA!** `<script src="https://www.gstatic.com/firebasejs/...">` gibi harici CDN linkleri Cordova WebView'da başarısız olabilir. İnternet yokken veya CDN yavaşken script yüklenmez → `firebase is not defined` → uygulama çöker. Firebase SDK'yı `www/lib/` altına lokal olarak indir.

> ⚠️ **Script yükleme sırası KRİTİK:** `cordova.js` her zaman İLK, ardından `skipLocalNotificationReady`, sonra lib/ kütüphaneleri, en son uygulama modülleri. Sıra değişirse undefined hatası alınır.

> ⚠️ **`viewport-fit=cover`:** Çentikli (notch) cihazlarda içeriğin safe area dışına taşmaması için. Edge-to-Edge hazırlığı.

> ⚠️ **CSP (Content-Security-Policy):** Kullanılmayan servisleri çıkar. Bölgesel Firebase DB (`europe-west1` vb.) kullanıyorsan `*.firebasedatabase.app` ve `wss://*.firebasedatabase.app` ZORUNLU — yoksa veri okunamaz/yazılamaz (Bkz. 9.19).


## 5. voltbuilder.json

### Debug Build:
```json
{
  "release": "debug"
}
```

### Release Build:
```json
{
  "androidAlias": "uygulamam",
  "androidAliasPassword": "SIFRE",
  "androidKeystore": "certificates/android.keystore",
  "androidKeystorePassword": "SIFRE",
  "release": "release"
}
```

---

## 6. Lokal Build İşlemi

### 6.1 Platform Ekle
```powershell
cd C:\Projeler\proje-adi
cordova platform add android
```

### 6.2 ⚠️ KRİTİK: appcompat Düzeltmesi (Build Öncesi Zorunlu)

cordova-android 14'ün varsayılan appcompat versiyonu (1.7.0) AAPT2 ile uyumsuz. Build'den ÖNCE düzelt:

```powershell
(Get-Content "platforms\android\cdv-gradle-config.json") -replace '"1.7.0"','"1.5.1"' | Set-Content "platforms\android\cdv-gradle-config.json"
```

> Bu düzeltme lokal build için kesin gerekli. VoltBuilder da aynı hatayı verdi (appcompat 1.7.0 ve 1.6.1 ile) ancak sonradan kendi sunucu ortamını güncelleyerek çözdü. VoltBuilder'da hata alırsan support@volt.build'a build ID ile birlikte bildir.
> `cordova platform remove/add android` yapılırsa dosya sıfırlanır — düzeltme tekrarlanmalı.

### 6.3 ⚠️ AdMob Plugin Seçimi (KRİTİK!)

> **`cordova-plugin-admob-free` KULLANMA!** Bu plugin 2019'dan beri terk edilmiş. cordova-android 14 ile native crash veriyor ve uygulama açılır açılmaz "sürekli duruyor" hatası çıkıyor. Java import düzeltmesi tek başına yetmez — native kütüphaneler de uyumsuz.
>
> Yerine **`admob-plus-cordova`** kullan. Aktif geliştirilen, AndroidX uyumlu alternatif.

config.xml'de:
```xml
<plugin name="admob-plus-cordova" spec="*">
    <variable name="ADMOB_APP_ID" value="ca-app-pub-3940256099942544~3347511713" />
</plugin>
```

Eski `cordova-plugin-admob-free` import düzeltmesi referans olarak aşağıda (sadece eski projeleri migrate edemeyenler için):

```powershell
(Get-Content "platforms\android\app\src\main\java\name\ratson\cordova\admob\AdMob.java") -replace 'import android.support.annotation.NonNull;','import androidx.annotation.NonNull;' | Set-Content "platforms\android\app\src\main\java\name\ratson\cordova\admob\AdMob.java"
```

> Bu düzeltme de `cordova platform remove/add android` yapılırsa sıfırlanır — appcompat düzeltmesiyle birlikte tekrarlanmalı.

### 6.4 Build
```powershell
cordova build android
```

Başarılı:
```
BUILD SUCCESSFUL
platforms\android\app\build\outputs\apk\debug\app-debug.apk
```

### 6.5 Telefona Yükle
```powershell
adb install platforms\android\app\build\outputs\apk\debug\app-debug.apk
```

---

## 7. VoltBuilder ile Build

1. Proje dosyalarını ZIP'le (config.xml kökte, www/, res/, resources/, voltbuilder.json)
2. https://volt.build → Giriş yap
3. ZIP'i Android kutusuna sürükle
4. Build tamamlanınca APK/AAB indir

### VoltBuilder Otomatik İkon Üretimi
`resources/iconTemplate.png` (1024x1024) koyarsan VoltBuilder tüm boyutları otomatik üretir ve config.xml'i günceller.

---

## 8. Release Build — Keystore

```powershell
keytool -genkey -v -keystore release.keystore -alias uygulamam -keyalg RSA -keysize 2048 -validity 9125
```

- **Şifreyi KAYDET!** Kaybedersen güncelleme yükleyemezsin.

### Lokal Release — build.json
```json
{
  "android": {
    "release": {
      "keystore": "release.keystore",
      "storePassword": "SIFRE",
      "alias": "uygulamam",
      "password": "SIFRE",
      "keystoreType": "jks"
    }
  }
}
```

### Release AAB
```powershell
cordova build android --release -- --packageType=bundle
```

---

## 9. Bilinen Hatalar ve Çözümleri

### 9.1 ❌ appcompat "Invalid color" (Lokal Build + VoltBuilder)
```
appcompat-1.7.0/res/values/values.xml:27:4: Invalid <color>
```
**Sebep:** cordova-android 14.0.1 varsayılan appcompat (1.7.0), AGP 8.3.0'ın AAPT2 derleyicisiyle uyumsuz. cordova-android master branch'ında appcompat 1.7.1'e yükseltildi — gelecek sürümde (14.1.0+) bu sorun düzelmiş olabilir. Yine de build hatası alırsan bu düzeltmeyi uygula.
**Çözüm:** Bölüm 6.2 — appcompat → 1.5.1

### 9.2 ❌ İkon Görünmüyor
**Sebep:** İkon `www/` altında veya config.xml'de `<icon>` yok.
**Çözüm:** `res/icon.png` koy + config.xml'e `<icon src="res/icon.png" />` ekle.

### 9.3 ❌ cordova-plugin-splashscreen Gereksiz
**Sebep:** cordova-android 11+ splash screen desteğini core'a taşıdı (AndroidX SplashScreen Core). Plugin'in engine requirement'ı `<11.0.0` olduğundan cordova-android 11+ ile zaten kurulamaz. Yeni projelerde bu plugin'i EKLEME — `AndroidWindowSplashScreen*` preference'ları yeterli.

### 9.4 ❌ Social Sharing Plugin Adı Yanlış
**Çözüm:** Doğru ad: `cordova-plugin-x-socialsharing`

### 9.5 ❌ Gradle Cache Bozulması
```powershell
taskkill /F /IM java.exe 2>$null
Remove-Item -Recurse -Force $env:USERPROFILE\.gradle\caches\* 2>$null
cordova build android
```

### 9.6 ❌ JAVA_HOME Yanlış
```powershell
echo $env:JAVA_HOME    # JDK 17 yolunu göstermeli
java -version           # 17.0.x olmalı
```

### 9.7 ❌ cordova-plugin-admob-free "android.support.annotation does not exist"
```
error: package android.support.annotation does not exist
import android.support.annotation.NonNull;
```
**Sebep:** `cordova-plugin-admob-free` eski `android.support` kütüphanesini kullanıyor, cordova-android 14 ise AndroidX kullanıyor.
**Çözüm:** Bölüm 6.3 — AdMob.java'daki import'u AndroidX ile değiştir:
```powershell
(Get-Content "platforms\android\app\src\main\java\name\ratson\cordova\admob\AdMob.java") -replace 'import android.support.annotation.NonNull;','import androidx.annotation.NonNull;' | Set-Content "platforms\android\app\src\main\java\name\ratson\cordova\admob\AdMob.java"
```
> `cordova platform remove/add android` yapılırsa tekrarlanmalı.

### 9.8 ❌ cordova-plugin-admob-free "Variable(s) missing: ADMOB_APP_ID"
```
Failed to install 'cordova-plugin-admob-free': Error: Variable(s) missing: ADMOB_APP_ID
```
**Sebep:** Plugin kurulurken AdMob App ID değişkeni gerekli.
**Çözüm:** config.xml'de plugin tanımına `<variable>` ekle:
```xml
<plugin name="cordova-plugin-admob-free" spec="*">
  <variable name="ADMOB_APP_ID" value="ca-app-pub-XXXXXXXXXXXXXXXX~YYYYYYYYYY" />
</plugin>
```
Test için Google'ın resmi test ID'si kullanılabilir: `ca-app-pub-3940256099942544~3347511713`
Yayınlamadan önce gerçek AdMob App ID ile değiştirilmeli.

### 9.9 ❌ cordova-plugin-admob-free Native Crash (Uygulama "Sürekli Duruyor")
**Belirti:** Uygulama açılır açılmaz kapanıyor. Android "sürekli duruyor" hatası veriyor. Logcat'te native crash.
**Sebep:** `cordova-plugin-admob-free` 2019'dan beri terk edilmiş. cordova-android 14'ün AndroidX ortamında:
- Eski `android.support` kütüphanesini kullanıyor (sadece Java import düzeltmesi yetmez)
- İçindeki Google AdMob SDK versiyonu modern Android ile uyumsuz
- Native kütüphaneler çakışıyor → uygulama başlatılırken native crash
**Çözüm:** `cordova-plugin-admob-free` KULLANMA. Yerine `admob-plus-cordova` kullan:
```xml
<!-- config.xml -->
<plugin name="admob-plus-cordova" spec="*">
    <variable name="ADMOB_APP_ID" value="ca-app-pub-3940256099942544~3347511713" />
</plugin>
```
**API değişiklikleri (admob-free → admob-plus):**
```javascript
// ESKİ (admob-free) — KULLANMA
admob.banner.config({ id: BANNER_ID, autoShow: true });
admob.banner.prepare();
admob.interstitial.config({ id: INTERSTITIAL_ID, autoShow: false });
admob.interstitial.prepare();
admob.interstitial.show();

// YENİ (admob-plus-cordova)
var bannerAd = new admob.BannerAd({ adUnitId: BANNER_ID, position: 'bottom' });
bannerAd.show();
var interstitialAd = new admob.InterstitialAd({ adUnitId: INTERSTITIAL_ID });
interstitialAd.load().then(function() { interstitialAd.show(); });
```

### 9.10 ❌ config.xml'de `<engine>` Etiketi Eksik
**Belirti:** `cordova platform add android` beklenmedik bir cordova-android versiyonu yükler. Build hataları.
**Sebep:** `<engine>` etiketi olmadan Cordova hangi platform versiyonunu kullanacağını bilemez.
**Çözüm:** config.xml'e `</platform>` kapanışından sonra, `</widget>` kapanışından önce ekle:
```xml
    </platform>
    <engine name="android" spec="14.0.1" />
</widget>
```

### 9.11 ❌ Firebase/Harici SDK CDN'den Yüklenince Crash
**Belirti:** Uygulama bazen açılıyor bazen çöküyor. Console'da `firebase is not defined`.
**Sebep:** Firebase SDK `<script src="https://www.gstatic.com/firebasejs/...">` ile CDN'den yüklenmiş. Cordova WebView'da ağ bağlantısı olmadığında veya yavaş olduğunda script yüklenemez.
**Çözüm:** Firebase SDK'yı lokal olarak `www/lib/` altına indir:
```powershell
# PowerShell ile indir:
$v = "10.12.0"
Invoke-WebRequest "https://www.gstatic.com/firebasejs/$v/firebase-app-compat.js" -OutFile "www\lib\firebase-app-compat.js"
Invoke-WebRequest "https://www.gstatic.com/firebasejs/$v/firebase-auth-compat.js" -OutFile "www\lib\firebase-auth-compat.js"
Invoke-WebRequest "https://www.gstatic.com/firebasejs/$v/firebase-database-compat.js" -OutFile "www\lib\firebase-database-compat.js"
```
index.html'de:
```html
<script src="lib/firebase-app-compat.js"></script>
<script src="lib/firebase-auth-compat.js"></script>
<script src="lib/firebase-database-compat.js"></script>
```

### 9.12 ❌ firebase.initializeApp() Hata Koruması Olmadan Çöküyor
**Belirti:** `firebase is not defined` veya `Cannot read property 'initializeApp' of undefined`.
**Sebep:** Firebase SDK yüklenemezse (CDN hatası, dosya eksik vb.) `firebase` global objesi tanımsız olur ve uygulama çöker.
**Çözüm:** Firebase init'i try-catch ile sar ve bir flag kullan:
```javascript
var firebaseReady = false;
var auth = null;
var db = null;

try {
    if (typeof firebase !== 'undefined') {
        firebase.initializeApp(firebaseConfig);
        auth = firebase.auth();
        db = firebase.database();
        firebaseReady = true;
    } else {
        console.error("Firebase SDK yüklenemedi!");
    }
} catch (e) {
    console.error("Firebase başlatma hatası:", e);
}

// Her Firebase işleminden önce kontrol et:
if (firebaseReady && auth) {
    auth.onAuthStateChanged(function(user) { ... });
}
```

### 9.13 ❌ Google ile Giriş (signInWithRedirect) Cordova'da Çalışmaz
**Belirti:** Google ile giriş butonu çalışmıyor, hata veriyor veya boş sayfa açılıyor.
**Sebep:** `auth.signInWithRedirect()` ve `auth.signInWithPopup()` Cordova WebView'da çalışmaz. WebView gerçek bir tarayıcı değildir — OAuth redirect/popup akışını desteklemez.
**Çözüm:** Cordova'da Google ile giriş için native plugin gerekir:
- `cordova-plugin-googleplus` veya
- `cordova-plugin-firebase-authentication`

Geçici çözüm olarak Google butonunu devre dışı bırak ve kullanıcıyı bilgilendir:
```javascript
function googleIleGiris() {
    bildirimGoster("Google ile giriş henüz desteklenmiyor. E-posta ile giriş yapın.", "uyari");
}
```

### 9.14 ❌ deviceready İçinde Plugin Çağrısı Çöküyor
**Belirti:** Uygulama `deviceready` olayında çöküyor. Logcat'te bir plugin'in init fonksiyonunda hata.
**Sebep:** `deviceready` handler'da bir plugin çağrısı hata verirse (plugin yüklü değil, uyumsuz vb.) tüm handler durur ve sonraki kodlar çalışmaz.
**Çözüm:** Her plugin çağrısını ayrı ayrı try-catch ile sar:
```javascript
document.addEventListener('deviceready', function() {
    cihazHazir = true;

    try { StatusBar.backgroundColorByHexString('#1565C0'); }
    catch (e) { console.warn("StatusBar hatası:", e); }

    try { admobInterstitialHazirla(); }
    catch (e) { console.warn("AdMob hatası:", e); }

    try { iapBaslat(); }
    catch (e) { console.warn("IAP hatası:", e); }

    try { bildirimIzniIste(); }
    catch (e) { console.warn("Bildirim hatası:", e); }
}, false);
```

### 9.15 ❌ PowerShell Script Türkçe Karakterle Çalışmaz
**Belirti:**
```
Unexpected token '}' in expression or statement.
```
**Sebep:** PowerShell, UTF-8 BOM olmadan Türkçe/Unicode karakter (ğ, ü, ş, İ, ö, ç, — vb.) içeren .ps1 dosyalarını parse edemiyor.
**Çözüm:** PowerShell scriptlerinde:
- Sadece ASCII karakter kullan (Türkçe karakter, em dash, vb. KULLANMA)
- Veya dosyayı UTF-8 BOM ile kaydet: VS Code'da `File → Save with Encoding → UTF-8 with BOM`
- Çalıştırırken: `powershell -ExecutionPolicy Bypass -File .\setup.ps1`

### 9.16 ❌ PowerShell "Execution Policy" Script Engeli
**Belirti:**
```
File ... cannot be loaded. The file ... is not digitally signed.
```
**Sebep:** Windows varsayılan PowerShell execution policy'si imzasız scriptleri engelliyor.
**Çözüm:** Scripti bypass modunda çalıştır:
```powershell
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

### 9.17 ❌ admob-plus-cordova Yanlış Variable Adı → Açılır Açılmaz Crash
**Belirti:** BUILD SUCCESSFUL ama uygulama açılır açılmaz kapanıyor ("sürekli duruyor"). Logcat'te `MobileAdsInitProvider.attachInfo()` hatası.
**Sebep:** `admob-plus-cordova` plugin'i `APP_ID_ANDROID` variable adı bekliyor. config.xml'de `ADMOB_APP_ID` yazılırsa (eski `cordova-plugin-admob-free`'nin adı) AdMob App ID AndroidManifest.xml'e **hiç yazılmaz** → Google Mobile Ads SDK başlatılamaz → native crash.
**Çözüm:** config.xml'deki plugin tanımında variable adını düzelt:
```xml
<!-- ❌ YANLIŞ (eski admob-free'nin adı) -->
<plugin name="admob-plus-cordova" spec="*">
    <variable name="ADMOB_APP_ID" value="ca-app-pub-xxx~yyy" />
</plugin>

<!-- ✅ DOĞRU (admob-plus-cordova'nın beklediği) -->
<plugin name="admob-plus-cordova" spec="*">
    <variable name="APP_ID_ANDROID" value="ca-app-pub-xxx~yyy" />
</plugin>
```
> Variable adı değişikliği sonrası `cordova platform remove android` + `cordova platform add android` gerekli (appcompat düzeltmesi tekrarlanmalı).

### 9.18 ❌ cordova-plugin-purchase (IAP) Variable Adı
**Dikkat:** `cordova-plugin-purchase` config.xml'de `<variable>` gerektirmez. Plugin spec'i `*` ile eklenebilir:
```xml
<plugin name="cordova-plugin-purchase" spec="*" />
```

### 9.19 ❌ Firebase Database Bağlanmıyor — CSP'de `firebasedatabase.app` Eksik
**Belirti:** Auth çalışıyor (giriş yapılabiliyor) ama veri okunamıyor/yazılamıyor. "Yükleniyor..." yazısı hiç gitmiyor. Kaydet butonuna basınca bir şey olmuyor. Console'da `Refused to connect` hatası.
**Sebep:** Firebase Realtime Database URL'in bölgesel ise (Europe-West1 vb.) domain `*.firebasedatabase.app` olur. Varsayılan US database'ler `*.firebaseio.com` kullanır. index.html'deki Content-Security-Policy (CSP) meta tag'ında `firebasedatabase.app` yoksa tarayıcı/WebView bu bağlantıyı sessizce engeller. Auth farklı domain kullandığı için (`googleapis.com`) etkilenmez.
**Ö (bölgesel database URL'i olan projeler):**
```
databaseURL: "https://PROJE-default-rtdb.europe-west1.firebasedatabase.app"
```
**Çözüm:** index.html'deki CSP meta tag'ına `firebasedatabase.app` domainini ekle:
```html
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
    https://pagead2.googlesyndication.com">
```
> `wss://` (WebSocket Secure) eklemeyi unutma — Firebase Realtime Database gerçek zamanlı dinleme için WebSocket kullanır.

### 9.20 ❌ admob-plus-cordova Banner/Reklam Görünmüyor — `admob.start()` Eksik
**Belirti:** Uygulama açılıyor, crash yok, ama hiçbir reklam (banner, interstitial) görünmüyor. Console'da hata yok veya sessizce başarısız oluyor.
**Sebep:** `admob-plus-cordova` plugin'i AdMob SDK'yı **otomatik başlatmaz**. Resmi dokümantasyon açıkça belirtir: `admob.start()` çağrılmadan hiçbir reklam yüklenemez. Bu, eski `cordova-plugin-admob-free`'den farklı bir davranıştır — orada SDK otomatik başlıyordu.
**Çözüm:** Banner oluşturmadan ÖNCE `await admob.start()` çağır. Fonksiyon `async` olmalı:
```javascript
// ❌ YANLIŞ — admob.start() yok, reklam yüklenmez
function admobBaslat() {
    var bannerAd = new admob.BannerAd({
        adUnitId: 'ca-app-pub-xxx/yyy',
        position: 'bottom'
    });
    bannerAd.show();
}

// ✅ DOĞRU — önce SDK başlat, sonra reklam göster
async function admobBaslat() {
    if (typeof admob === 'undefined') {
        console.warn('AdMob plugin yüklü değil');
        return;
    }
    try {
        await admob.start();  // ← KRİTİK: Bu olmadan reklam yüklenmez!
        console.log('AdMob SDK başlatıldı');

        var bannerAd = new admob.BannerAd({
            adUnitId: 'ca-app-pub-xxx/yyy',
            position: 'bottom'
        });
        await bannerAd.show();
        document.body.style.paddingBottom = '60px';
    } catch (e) {
        console.warn('AdMob hatası:', e);
    }
}
```
> **Not:** `admob.start()` bir Promise döner, bu yüzden fonksiyon `async` olmalı ve `await` kullanılmalı. `deviceready` handler'da çağırırken Promise hatasını yakala:
> ```javascript
> try { admobBaslat().catch(function(e) { console.warn("AdMob:", e); }); }
> catch (e) { console.warn("AdMob:", e); }
> ```
> **Ek bilgi:** Test reklamlar (`ca-app-pub-3940256099942544/6300978111`) bile `admob.start()` olmadan yüklenemez. "No fill" hatası da bazen bu sebepten kaynaklanır.

### 9.21 ❌ admob-plus-cordova Reklam Görünmüyor — `GradlePluginKotlinEnabled` Eksik
**Belirti:** Uygulama açılıyor, crash yok, console'da hata yok, ama reklam hiç görünmüyor. `typeof admob === 'undefined'` — `admob` objesi JavaScript tarafında hiç tanımlanmıyor.
**Sebep:** `admob-plus-cordova` plugin'i **Kotlin** ile yazılmış. config.xml'de `GradlePluginKotlinEnabled` preference'ı yoksa Cordova, plugin'in Kotlin native kodunu **derlemez**. Plugin build'e dahil olmaz → JavaScript bridge oluşmaz → `admob` objesi tanımsız kalır → kodda `typeof admob === 'undefined'` dalına girer ve sessizce `return` eder. Hata mesajı yok ama reklam da yok.
**Çözüm:** config.xml'de `<platform name="android">` bloğunun **içine** şu iki preference'ı ekle:
```xml
<platform name="android">
    <preference name="GradlePluginKotlinEnabled" value="true" />
    <preference name="AndroidXEnabled" value="true" />
    <!-- ... diğer config-file tanımları ... -->
</platform>
```
> **Önemli:** Bu preference'lar `<platform name="android">` bloğunun **İÇİNDE** olmalı, dışında olursa Android build'e uygulanmaz.
> **Önemli:** config.xml değişikliği sonrası platform'u sıfırdan kurman gerekir:
> ```
> cordova platform remove android
> cordova platform add android
> ```
> Sadece `cordova build` yetmez — preference değişiklikleri ancak platform eklenirken işlenir.

### 9.22 ❌ URL Tıklayınca "net::ERR_CONNECTION_REFUSED (https://localhost/...)" Hatası
**Belirti:** Uygulamada kayıtlı bir URL'ye tıklayınca `Application Error — net::ERR_CONNECTION_REFUSED (https://localhost/turkiye.gov.tr)` hatası çıkıyor. URL harici tarayıcıda açılmıyor.
**Sebep:** HTML'de `<a href="https://site.com" target="_blank">` kullanılmış. Cordova WebView gerçek bir tarayıcı değildir — `target="_blank"` harici tarayıcıyı açmaz. WebView URL'yi kendi içinde `https://localhost/` altında yüklemeye çalışır ve hata verir.
**Çözüm:** `<a href>` yerine `window.open(url, '_system')` kullan. `_system` parametresi URL'yi cihazın varsayılan tarayıcısında açar:
```javascript
// ❌ YANLIŞ — Cordova WebView'da çalışmaz
'<a href="' + url + '" target="_blank">' + url + '</a>'

// ✅ DOĞRU — Harici tarayıcıda açar
'<a href="#" onclick="hariciUrlAc(\'' + url + '\');return false;">' + url + '</a>'

// Yardımcı fonksiyon:
function hariciUrlAc(url) {
    if (!url) return;
    // Protokol yoksa https:// ekle
    if (!/^https?:\/\//i.test(url)) {
        url = 'https://' + url;
    }
    // _system = cihazın varsayılan tarayıcısında aç
    window.open(url, '_system');
}
```
> **Not:** `window.open(url, '_system')` Cordova'nın temel özelliğidir, ek plugin gerektirmez. Üç parametre kullanılabilir: `_system` (harici tarayıcı), `_blank` (InAppBrowser — plugin gerektirir), `_self` (WebView içinde).

### 9.23 ❌ Dışa Aktarma (JSON Export) — Dosya Paylaşılamıyor / İndirilenlerde Yok
**Belirti (Aşama 1):** Dışa aktar butonu toast "Dışa aktarıldı" gösteriyor ama İndirilenlerde dosya yok. Sessiz başarısızlık.

**Sebep (Aşama 1):** `URL.createObjectURL()` + `<a download>` yöntemi **sadece tarayıcıda** çalışır. Cordova WebView `download` HTML attribute'ünü tanımaz.
```javascript
// ❌ YANLIŞ — Cordova WebView'da çalışmaz
var a = document.createElement('a');
a.download = 'yedek.json';   // WebView bu attribute'ü tanımaz!
a.href = url;
a.click();                     // Sessizce başarısız olur
```

**Belirti (Aşama 2):** `cordova-plugin-file` ile `cacheDirectory`'ye yazıp `fileEntry.toURL()` ile socialsharing'e verince:
- Toast: `keysafe_yedek_2026-02-13.json cache'e kaydedildi (Konum: https://localhost/__cdvfile_cache__/...)`
- Paylaşım menüsü açılıyor, WhatsApp seçilince: "Paylaşım başarısız, lütfen tekrar deneyin"
- WhatsApp sadece `message` textini gönderiyor, dosya eki YOK

**Sebep (Aşama 2):** `fileEntry.toURL()` cordova-plugin-file v7+'da artık `https://localhost/__cdvfile_cache__/...` sanal URL dönüyor. Bu URL sadece WebView DOM'da kullanılmak içindir — WhatsApp/e-posta bu URL'yi tanımaz. `fileEntry.nativeURL` (`file:///data/...`) kullanılsa bile cordova-android 14'ün HTTPS scheme'i ile socialsharing plugin'inin FileProvider'ı uyumsuz çalışır — dosya `content://` URI'ya doğru çevrilemez, WhatsApp dosyayı okuyamaz ve sadece text mesajını gönderir.

**Çözüm (Kesin — Base64 Data URI):** Dosya sistemini ve FileProvider sorunlarını **tamamen bypass et**. Socialsharing plugin'inin `df:dosyaadi;data:mimetype;base64,DATA` formatını kullan. Bu formatta plugin dosyayı **kendi temp dizinine** yazar ve **kendi FileProvider'ı** ile `content://` URI oluşturur.

**1) config.xml'e pluginleri ekle:**
```xml
<plugin name="cordova-plugin-file" spec="*" />
<plugin name="cordova-plugin-x-socialsharing" spec="*" />
```

**2) Dışa aktarma fonksiyonu:**
```javascript
async function jsonExport() {
    var dosyaAdi = 'keysafe_yedek_' + new Date().toISOString().slice(0, 10) + '.json';
    var jsonStr = JSON.stringify(exportVeri, null, 2);

    if (window.plugins && window.plugins.socialsharing) {
        // JSON -> base64 -> df:dosyaadi;data:mimetype;base64,DATA
        var base64 = btoa(unescape(encodeURIComponent(jsonStr)));
        var dataUri = 'df:' + dosyaAdi + ';data:application/json;base64,' + base64;

        window.plugins.socialsharing.shareWithOptions({
            files: [dataUri],
            subject: 'Yedek (' + kayitSayisi + ' kayit)',
            chooserTitle: 'Yedek dosyasini paylas veya kaydet'
            // ⚠️ message EKLEME! WhatsApp message varken dosyayı görmezden gelip
            // sadece text gönderebilir. Sadece files + subject kullan.
        }, function(result) {
            toast(dosyaAdi + ' paylasildi', 'basari');
        }, function(err) {
            toast('Paylasim iptal edildi', 'uyari');
        });
    } else {
        // Tarayıcı fallback (geliştirme/test için)
        var blob = new Blob([jsonStr], { type: 'application/json' });
        var url = URL.createObjectURL(blob);
        var a = document.createElement('a');
        a.href = url; a.download = dosyaAdi; a.click();
        URL.revokeObjectURL(url);
    }
}
```

> **Neden `df:` data URI?** `cordova-plugin-file` ile dosya yazıp `nativeURL` veya `toURL()` versen de cordova-android 14'ün HTTPS hosting scheme'i ile socialsharing'in FileProvider'ı uyumsuz. Plugin `file://` path'i `content://` URI'ya çevirirken başarısız oluyor. `df:` formatında plugin veriyi **kendi** temp dosyasına yazıp **kendi** FileProvider'ıyla paylaşır — tüm ara katmanları bypass eder.

> **⚠️ `message` parametresi tuzağı:** `shareWithOptions`'a hem `message` hem `files` verirsen WhatsApp dosyayı görmezden gelip sadece text mesajı gönderebilir. Dosya paylaşımında `message` EKLEME, sadece `subject` kullan.

### 9.24 ❌ Firebase'den isPremium=true Yapılınca Premium Tanınmıyor — `checkPremiumStatus()` Eziyor
**Belirti:** Firebase Console'dan kullanıcının `settings/isPremium` değerini `true` yapıyorsun. Uygulama açılıyor, ama Raporlar'a tıklayınca premium overlay çıkıyor. Ayarlarda "Premium: Aktif değil" yazıyor. Premium hiç tanınmıyor.
**Sebep:** Uygulama açılışında akış şöyle:
1. `onAppReady()` → `loadPremiumFromSettings()` → Firebase'den `isPremium = true` okunur ✅
2. `initStore()` → Google Play Store async başlatılır
3. Store hazır olunca `checkPremiumStatus()` çağrılır
4. `checkPremiumStatus()` içinde: `var lifetimeOwned = lifetime && lifetime.owned;` → kullanıcı Google Play'den satın almadığı için `false`
5. **`isPremium = lifetimeOwned;`** → `isPremium = false` — Firebase'den okunan `true` EZİLDİ!
6. **`savePremiumStatus(isPremium);`** → Firebase'e `false` GERİ YAZILDI — admin değişikliği de silindi!

Yani `checkPremiumStatus()` koşulsuz olarak Google Play durumunu `isPremium`'a yazıyor. Google Play'de satın alma yoksa, Firebase'den gelen premium durumu hem bellekte hem veritabanında sıfırlanıyor.
**Çözüm:** `checkPremiumStatus()` fonksiyonunda, Google Play'de satın alma yoksa mevcut `isPremium` değerine dokunma:
```javascript
// ❌ YANLIŞ — Firebase premium durumunu eziyor
function checkPremiumStatus() {
  if (!storeReady || !window.CdvPurchase) {
    loadPremiumFromSettings();
    return;
  }
  var lifetime = CdvPurchase.store.get(PRODUCT_ID_LIFETIME);
  var lifetimeOwned = lifetime && lifetime.owned;
  isPremium = lifetimeOwned;           // ← Google Play'de yoksa false yazıyor!
  savePremiumStatus(isPremium);        // ← false'u Firebase'e geri yazıyor!
  updatePremiumUI();
}

// ✅ DOĞRU — Sadece Google Play'de varsa true yap, yoksa DOKUNMA
function checkPremiumStatus() {
  if (!storeReady || !window.CdvPurchase) {
    loadPremiumFromSettings();
    return;
  }
  var lifetime = CdvPurchase.store.get(PRODUCT_ID_LIFETIME);
  var lifetimeOwned = lifetime && lifetime.owned;

  if (lifetimeOwned) {
    // Google Play'de satın alma VAR — premium yap ve kaydet
    isPremium = true;
    savePremiumStatus(true);
  }
  // else: Google Play'de yok AMA Firebase/settings'ten true gelmişse DOKUNMA!
  // isPremium zaten loadPremiumFromSettings()'te set edildi, ezme.

  updatePremiumUI();
}
```
> **Mantık:** Google Play satın alması `true` ise → her zaman premium yap. Google Play'de yoksa → `isPremium`'a dokunma, Firebase'den gelen değer korunsun. Bu sayede hem gerçek satın alma hem admin tarafından verilen premium çalışır.

### 9.25 ❌ Bildirim "Ödendi" Aksiyonu Uygulama Kapalıyken Çalışmıyor
**Belirti:** Bildirimde "Ödendi İşaretle" butonuna basınca:
- Uygulama **arka plandaysa** (Home tuşu ile) → çalışıyor ✅
- Uygulama **kapalıysa** (Çıkış / exitApp) → uygulama açılıyor ama ödeme ödendi olarak işaretlenmiyor ❌

**Sebep (3 ayrı sorun):**
1. **`launchDetails.data` boş geliyor (bilinen plugin bug'ı — GitHub #1640, #1631, #1581):** `launchDetails` objesi `{id: 12345, action: 'paid'}` dönüyor ama `data` alanı `null/undefined`. `paymentId` oradan alınamıyor.
2. **`notification.get(id)` async race condition:** Fallback olarak `get()` çağırılsa bile callback döndüğünde `onAppReady()` çoktan çalışmış, `localStorage.getItem()` → `null` bulmuş.
3. **`fireQueuedEvents` + `on('paid')` timing sorunu (GitHub #1798):** Plugin, event'i WebView hazır olmadan ateşliyor. Listener kaydedilmeden event geliyor → kaybolıyor.

**Kök neden:** Tüm mekanizmalar `notification.data`'dan `paymentId` almaya bağımlıydı. Ama `data` alanı cold start'ta güvenilir değil. Güvenilir olan tek şey: `launchDetails.id` (numeric bildirim ID) ve `launchDetails.action`.

**Çözüm — Notification ID → Payment Ters Eşleme:**
Bildirim ID'leri `paymentIdToNumeric(paymentId) * 10 + i` formülüyle üretiliyor. Ters hesaplama: `Math.floor(notifId / 10)` = baseId → `paymentsCache`'te eşleşen payment bulunur. `notification.data`'ya hiç ihtiyaç kalmaz.

**1) `findPaymentByNotifId()` fonksiyonu (notifications.js):**
```javascript
function findPaymentByNotifId(notifId) {
  if (!notifId && notifId !== 0) return null;
  var baseId = Math.floor(Number(notifId) / 10);
  if (isNaN(baseId)) return null;
  if (typeof paymentsCache === 'undefined' || !paymentsCache || paymentsCache.length === 0) return null;
  for (var i = 0; i < paymentsCache.length; i++) {
    if (paymentIdToNumeric(paymentsCache[i].id) === baseId) return paymentsCache[i];
  }
  return null;
}
```

**2) `on('paid')` handler'da data eksikse notifId fallback (notifications.js):**
```javascript
var paymentId = (data && data.paymentId) ? data.paymentId : null;
if (!paymentId && notification && notification.id) {
  var foundPayment = findPaymentByNotifId(notification.id);
  if (foundPayment) paymentId = foundPayment.id;
}
```

**3) `deviceready`'de notifId tabanlı marker (app.js) — data'ya bağımlılık yok:**
```javascript
if (earlyAction === 'paid') {
  localStorage.setItem('_pendingPaidNotifId', String(earlyLd.id));
}
```

**4) `onAppReady`'de notifId → findPaymentByNotifId → ödendi (app.js):**
```javascript
var pendingNotifId = localStorage.getItem('_pendingPaidNotifId');
if (pendingNotifId && !_coldStartPaidProcessing) {
  var payment = findPaymentByNotifId(Number(pendingNotifId));
  if (payment && payment.status !== 'paid') {
    await markPaymentPaid(payment.id, payment.amount);
  }
}
```

**5) `resume` event listener (app.js) — ek güvenlik katmanı:**
```javascript
document.addEventListener('resume', function() {
  var pendingNotifId = localStorage.getItem('_pendingPaidNotifId');
  if (pendingNotifId) {
    var payment = findPaymentByNotifId(Number(pendingNotifId));
    // payment bulunursa ödendi yap
  }
}, false);
```

> **3 katlı mekanizma:** (1) `fireQueuedEvents` → `on('paid')` + `findPaymentByNotifId` || (2) `launchDetails.id` + `findPaymentByNotifId` || (3) `localStorage marker` + `onAppReady`/`resume`.
> **Not:** Tüm projelerde `exitApp()` yerine `arkaPlanaGonder()` (moveTaskToBack) kullanılıyor. Uygulama kapanmak yerine arka plana gider, cold start yerine warm resume olur. Bkz. Bölüm 9.29.

### 9.26 ❌ Firebase Sync Çalışmıyor — goOffline/goOnline Auth Token Race
**Belirti:** Auth çalışıyor (giriş yapılabiliyor) ama veriler Firebase'e yazılmıyor. Sync sonrası `oh_lastSync` null kalıyor. Catch bloğunda hata maskeleniyor: `Sync hatasi (internet yok olabilir)`.
**Sebep:** `firebase.database()` oluşturulur oluşmaz `goOffline()` çağrılıyor (local-first mimari). Auth state değiştiğinde database SDK offline olduğu için token bildirimini alamıyor. `goOnline()` yapıldığında eski/null token ile bağlanıyor → `PERMISSION_DENIED`. Ayrıca catch bloğu tüm hataları aynı mesajla yuttuğu için gerçek sebep görünmüyor.
**Çözüm (2 adım):**
1. `syncWithFirebase()` içinde `goOnline()`'dan ÖNCE `await mevcutKullanici.getIdToken(true)` çağır. Bu, database SDK'ya taze token gitmesini garanti eder:
```javascript
// syncWithFirebase() başlangıcı:
try {
    await mevcutKullanici.getIdToken(true);
    console.log('syncWithFirebase: token yenilendi');
} catch (tokenErr) {
    console.error('syncWithFirebase: TOKEN YENILEME HATASI:', tokenErr.code || tokenErr.message);
    throw tokenErr;
}
fbDb.goOnline();
await new Promise(function(r) { setTimeout(r, 800); });
```
2. Catch bloğundaki hata maskelemeyi kaldır, gerçek hatayı logla:
```javascript
catch (e) {
    var hataMesaji = e.message || e.code || String(e);
    console.error('syncWithFirebase HATA:', hataMesaji);
    if (hataMesaji.indexOf('PERMISSION_DENIED') !== -1) {
        console.error('>>> PERMISSION_DENIED: Firebase rules erisimi engelliyor!');
    }
}
```
> `hesapSilKalici()` fonksiyonunda da aynı `getIdToken(true)` düzeltmesi uygulanmalı.
> Bekleme süresini 600ms'den 800ms'ye çıkar — Europe-West1 gibi bölgesel database'lerde WebSocket kurulumu daha uzun sürebilir.

### 9.27 ✅ Kalıcı Logger + Fonksiyon İzleme Sistemi (TÜM UYGULAMALARDA ZORUNLU)

**Amaç:** Kullanıcı "çalışmıyor" dediğinde logu mail ile gönderip sorunu anında teşhis etmek. Uygulama kapansa bile loglar korunur. Her fonksiyon girişi izlenir.

**Mimari — 3 katmanlı:**
1. **Kalıcı Ring Buffer Logger** (db.js'in en başına, Firebase config'den ÖNCE):
   - `console.log/warn/error` intercept → `_logBuffer` array'e push
   - Her satır: `HH:MM:SS.mmm [LOG/WRN/ERR/FN] mesaj`
   - Max 1000 satır (eski loglar otomatik silinir)
   - **localStorage'a kalıcı yazma:** 5 saniyede bir otomatik flush + pause/beforeunload'da anında flush
   - Uygulama yeniden açılınca önceki oturum logları da yüklenir
   - `window.error` ve `unhandledrejection` event'leri de yakalanır
2. **`_fn()` Fonksiyon İzleme** (her fonksiyonun ilk satırına eklenir):
   - `_fn('fonksiyonAdi')` → `[FN] fonksiyonAdi` satırı loglar
   - Gürültü filtresi: sık çağrılan yardımcı fonksiyonlar **exclude** listesinde → loglanmaz
   - Exclude listesi projeye göre özelleştirilir (aşağıda standart liste var)
3. **Hata Raporu Gönder** butonu (Ayarlar sayfasında):
   - Cihaz bilgisi + uygulama durumu + son 1000 log satırı → TXT dosyası
   - socialsharing `df:dosyaadi;data:text/plain;base64,...` formatıyla paylaşım

#### ⚠️ `_fn()` EXCLUDE LİSTESİ — Gürültü Filtresi
Bu fonksiyonlar saniyede düzinelerce kez çağrılır ve teşhis değeri yoktur. `_fn()` EKLEME:
```
t                    ← i18n çeviri (translateUI her çağrıda 50-70 kez tetikler)
escapeHtml           ← XSS koruması, her render'da onlarca kez
formatMoney          ← Para formatlama
formatDate           ← Tarih formatlama (tüm varyantlar)
formatDateShort
formatDateStr
formatDateStrDb
getDaysUntil         ← Gün hesaplama
getTodayStr          ← Bugünün tarihi
getCurrencySymbol    ← Para birimi sembolü
getCategoryLabel     ← Kategori etiketi
getRecurrenceLabel   ← Tekrar etiketi
getIntervalMonths
getIntervalMonthsDb
getPeriodPayStatus   ← Dönem ödeme durumu (her ödeme için çağrılır)
getCurrentPeriodDate ← Dönem tarih hesaplama
getPeriodsInMonth    ← Ay dönemleri
enrichPaymentStatus  ← Ödeme zenginleştirme (cache güncellemede N kez)
```
> **Kural:** Bir fonksiyon her sayfa render'da 5+ kez çağrılıyorsa ve sadece veri dönüştürme/formatlama yapıyorsa → exclude listesine ekle. İş mantığı, navigasyon, veri yazma, API çağrısı, plugin kullanımı yapan fonksiyonlara **HER ZAMAN** `_fn()` ekle.

#### ⚠️ YENİ FONKSİYON YAZARKEN ZORUNLU KURAL
- **Her yeni fonksiyonun ilk satırına `_fn('fonksiyonAdi');` ekle** (exclude listesindekiler hariç)
- Mevcut dosyaya fonksiyon eklerken de aynı kural geçerli
- `_fn()` satırı fonksiyon gövdesinin İLK satırı olmalı (parametrelerden sonra, iş mantığından önce)
- Örnek:
```javascript
function yeniOzellik() {
  _fn('yeniOzellik');
  // ... iş mantığı
}
async function veriKaydet(data) {
  _fn('veriKaydet');
  // ... iş mantığı
}
```

**Uygulama:**

**1) db.js'in EN BAŞINA (tüm kodlardan önce, Firebase config'den bile önce):**
```javascript
// ========== LOGGER — Kalici Ring Buffer (her zaman aktif) ==========
var _logBuffer = [];
var _LOG_MAX = 1000;
var _LOG_STORAGE_KEY = 'PROJE_logBuffer';  // ← Projeye gore degistir (ornek: oh_logBuffer, ks_logBuffer)
var _logDirty = false;
var _logFlushTimer = null;

// Onceki oturum loglarini yukle
try {
  var _saved = localStorage.getItem(_LOG_STORAGE_KEY);
  if (_saved) _logBuffer = JSON.parse(_saved);
} catch(e) {}

function _logEkle(seviye, args) {
  var zaman = new Date().toISOString().substr(11, 12);
  var mesaj = '';
  for (var i = 0; i < args.length; i++) {
    if (i > 0) mesaj += ' ';
    try {
      if (typeof args[i] === 'object') mesaj += JSON.stringify(args[i]);
      else mesaj += String(args[i]);
    } catch (e) { mesaj += '[object]'; }
  }
  _logBuffer.push(zaman + ' [' + seviye + '] ' + mesaj);
  if (_logBuffer.length > _LOG_MAX) _logBuffer.shift();
  _logDirty = true;
}

function _fn(name) { _logEkle('FN', [name]); }

function _logFlush() {
  if (!_logDirty) return;
  try { localStorage.setItem(_LOG_STORAGE_KEY, JSON.stringify(_logBuffer)); }
  catch(e) {}
  _logDirty = false;
}

// 5 saniyede bir localStorage'a yaz
_logFlushTimer = setInterval(_logFlush, 5000);

var _origLog = console.log;
var _origWarn = console.warn;
var _origError = console.error;
console.log = function() { _logEkle('LOG', arguments); _origLog.apply(console, arguments); };
console.warn = function() { _logEkle('WRN', arguments); _origWarn.apply(console, arguments); };
console.error = function() { _logEkle('ERR', arguments); _origError.apply(console, arguments); };

window.addEventListener('error', function(e) {
  _logEkle('ERR', ['UNCAUGHT: ' + (e.message || '') + ' @ ' + (e.filename || '') + ':' + (e.lineno || '')]);
  _logFlush();
});
window.addEventListener('unhandledrejection', function(e) {
  _logEkle('ERR', ['UNHANDLED_PROMISE: ' + (e.reason ? (e.reason.message || e.reason) : 'unknown')]);
  _logFlush();
});

// Uygulama arka plana alininca / kapatilinca aninda flush
document.addEventListener('pause', function() {
  _logEkle('LOG', ['APP_LIFECYCLE: pause (arka plana alindi)']);
  _logFlush();
}, false);
document.addEventListener('resume', function() {
  _logEkle('LOG', ['APP_LIFECYCLE: resume (on plana geldi)']);
}, false);
window.addEventListener('beforeunload', function() {
  _logEkle('LOG', ['APP_LIFECYCLE: beforeunload']);
  _logFlush();
});

console.log('APP_LIFECYCLE: db.js yuklendi (' + new Date().toISOString() + ')');
// ========== LOGGER SONU ==========
```

**2) db.js'in sonuna:**
```javascript
function hataRaporuOlustur() {
  var satirlar = [];
  satirlar.push('=== UYGULAMA ADI - HATA RAPORU ===');
  satirlar.push('Tarih: ' + new Date().toISOString());
  satirlar.push('');
  satirlar.push('--- CIHAZ ---');
  try {
    if (typeof device !== 'undefined') {
      satirlar.push('Platform: ' + (device.platform || '?'));
      satirlar.push('Versiyon: ' + (device.version || '?'));
      satirlar.push('Model: ' + (device.model || '?'));
      satirlar.push('Manufacturer: ' + (device.manufacturer || '?'));
    } else {
      satirlar.push('Platform: ' + navigator.platform);
      satirlar.push('UserAgent: ' + navigator.userAgent.substr(0, 120));
    }
  } catch (e) { satirlar.push('Cihaz bilgisi alinamadi'); }
  satirlar.push('Ekran: ' + screen.width + 'x' + screen.height);
  satirlar.push('Online: ' + navigator.onLine);
  satirlar.push('');
  satirlar.push('--- UYGULAMA DURUMU ---');
  // ↓ Projeye gore ozellestir (Firebase, cache, premium vb.)
  satirlar.push('Firebase Ready: ' + (typeof firebaseReady !== 'undefined' ? firebaseReady : '?'));
  satirlar.push('Kullanici: ' + (typeof mevcutKullanici !== 'undefined' && mevcutKullanici ? mevcutKullanici.email : 'YOK'));
  satirlar.push('');
  satirlar.push('--- LOGLAR (son ' + _logBuffer.length + ' satir) ---');
  for (var i = 0; i < _logBuffer.length; i++) {
    satirlar.push(_logBuffer[i]);
  }
  return satirlar.join('\n');
}

function hataRaporuGonder() {
  var raporText = hataRaporuOlustur();
  var dosyaAdi = 'hata_raporu_' + new Date().toISOString().slice(0, 19).replace(/[T:]/g, '-') + '.txt';

  if (window.plugins && window.plugins.socialsharing) {
    var base64 = btoa(unescape(encodeURIComponent(raporText)));
    var dataUri = 'df:' + dosyaAdi + ';data:text/plain;base64,' + base64;
    window.plugins.socialsharing.shareWithOptions({
      files: [dataUri],
      subject: 'Uygulama Adi - Hata Raporu',
      chooserTitle: 'Hata raporunu gonder'
    }, function() {
      if (typeof showToast === 'function') showToast('Rapor gonderildi');
    }, function() {
      if (typeof showToast === 'function') showToast('Gonderim iptal edildi');
    });
  } else {
    var blob = new Blob([raporText], { type: 'text/plain' });
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.href = url; a.download = dosyaAdi; a.click();
    URL.revokeObjectURL(url);
  }
}
```

**3) index.html — Ayarlar sayfasına buton:**
```html
<div class="settings-item" onclick="hataRaporuGonder()">
  <div class="set-label"><span class="set-icon">🐛</span> <span data-i18n="set_bug_report">Hata Raporu Gönder</span></div>
  <span class="set-value">›</span>
</div>
```

**4) i18n.js — Çeviri:**
```javascript
set_bug_report: { tr: 'Hata Raporu Gönder', en: 'Send Bug Report' },
```

> **Not:** `_LOG_STORAGE_KEY` projeye göre değiştirilmeli (örnek: `oh_logBuffer`, `ks_logBuffer`). Logger altyapısı (`_logBuffer`, `_logEkle`, `_fn`, `_logFlush`, console intercept, lifecycle) tüm projelerde BİREBİR AYNI kalır. `hataRaporuOlustur()` içindeki uygulama durumu bilgileri projeye göre özelleştirilir.
> **Plugin:** `cordova-plugin-x-socialsharing` ve `cordova-plugin-device` gerekli.
> **Performans:** 1000 satırlık JSON localStorage yazma ~1-2ms. 5sn interval ile CPU etkisi ölçülemez düzeyde. `_fn()` çağrısı ~0.01ms. Exclude listesi sayesinde gereksiz noise önlenir.

### 9.28 ❌ Firebase Sync Timeout — `goOffline()` WebSocket'i Öldürüyor (finally + pauseSync)
**Belirti:** Task kill ile kapatıp açınca `syncWithFirebase` her çağrıda 20sn timeout'a düşüyor (`FIREBASE_TIMEOUT`). `getIdToken(true)` başarılı (HTTPS), ama `.once('value')` (WebSocket) hiç dönmüyor. `Firebase Bagli: false`, `Sync Devam: true` takılı kalıyor.
**Sebep (2 ayrı noktada aynı hata):**
1. **`syncWithFirebase` finally bloğu:** Her sync sonrası `goOffline()` çağrılıyordu → WebSocket kesiliyordu → sonraki sync'te "forever dead socket"
2. **`pauseSync()` fonksiyonu:** `syncWithFirebase()` async başlatılıp **hemen ardından** `goOffline()` çağrılıyordu. `syncWithFirebase` henüz token bile yenilemeden WebSocket kesiliyordu. Sonraki `goOnline()` bozuk state'de kalıyordu. Task kill bu durumda gelince yeniden açılışta SDK recover edemiyordu.

**Race condition detayı (pauseSync):**
```
pauseSync() çağrılır
  → syncWithFirebase() başlatılır (async, Promise döner)
  → goOffline() HEMEN çalışır (senkron!) → WebSocket kesilir
  → syncWithFirebase içinde goOnline() çağrılır → yeni WebSocket açılmaya çalışılır
  → TASK KILL → OS her şeyi öldürür
  → Yeniden açılışta: Firebase SDK'nın dahili WebSocket state'i bozuk
```

**Çözüm (3 adım):**

**Adım 1 — finally bloğundaki `goOffline()` kaldırıldı:**
```javascript
// ❌ YANLIŞ — WebSocket'i öldürüyor
} finally {
    try { fbDb.goOffline(); } catch(ignore) {}
    _syncDevam = false;
}

// ✅ DOĞRU — SDK 60sn sonra otomatik keser
} finally {
    _syncDevam = false;
}
```

**Adım 2 — `pauseSync()`teki `goOffline()` kaldırıldı:**
```javascript
// ❌ YANLIŞ — async sync başlatıp hemen WebSocket'i kesiyor
function pauseSync() {
  if (mevcutKullanici) {
    syncWithFirebase().catch(function() {});
    if (fbDb) { fbDb.goOffline(); }  // ← sync'in WebSocket'ini öldürüyor!
  }
}

// ✅ DOĞRU — goOffline yok, SDK kendi yönetir
function pauseSync() {
  _fn('pauseSync');
  if (mevcutKullanici) {
    syncWithFirebase().catch(function() {});
    // goOffline() KALDIRILDI — aktif sync'in WebSocket'ini öldürüyordu
    // Firebase SDK 60sn inaktiviteden sonra bağlantıyı otomatik keser
    console.log('pauseSync: sync tetiklendi (goOffline yok)');
  }
}
```

**Adım 3 — Sabit 2.5sn bekleme yerine `.info/connected` ile gerçek bağlantı doğrulama:**
```javascript
// ❌ YANLIŞ — 2.5sn bekleme, bağlantı kuruldu mu bilinmiyor
fbDb.goOnline();
await new Promise(function(r) { setTimeout(r, 2500); });

// ✅ DOĞRU — .info/connected ile gerçek doğrulama
function _waitForConnection(maxMs) {
  return new Promise(function(resolve) {
    if (!fbDb) { resolve(false); return; }
    var ref = fbDb.ref('.info/connected');
    var resolved = false;
    var timer = setTimeout(function() {
      if (!resolved) { resolved = true; ref.off(); resolve(false); }
    }, maxMs);
    ref.on('value', function(snap) {
      if (snap.val() === true && !resolved) {
        resolved = true;
        clearTimeout(timer);
        ref.off();
        resolve(true);
      }
    });
  });
}

// syncWithFirebase içinde:
fbDb.goOnline();
var baglandi = await _waitForConnection(8000);
if (!baglandi) {
  // Bağlanamadıysa sıfırla ve tekrar dene
  fbDb.goOffline();
  await new Promise(function(r) { setTimeout(r, 500); });
  fbDb.goOnline();
  baglandi = await _waitForConnection(5000);
}
```

> **Doğru `goOffline()` kullanım yerleri (KALDIRILMAMALI):**
> - Firebase init (satır ~151) — local-first, başlangıçta offline ✅
> - `cikisYap()` — oturum kapatma ✅
> - `hesapSilKalici()` — hesap silme ✅
> - `syncWithFirebase` retry reset — bağlantı kurulamadığında kasıtlı sıfırlama ✅
>
> **Endüstri standardı:** Firebase resmi dokümantasyonu ve Firebase ekibinden Kato: "Genel kural olarak bağlantıyı kendiniz yönetmenize gerek yok, Firebase kendi ilgilenir." `goOffline()` sadece bilinçli bağlantı kesme noktalarında (çıkış, hesap silme) kullanılmalı. Sync döngülerinde ve pause handler'larda KULLANILMAMALI.
> **İlişkili:** 9.26'daki `getIdToken(true)` düzeltmesiyle birlikte çalışır. Token yenileme + goOffline kaldırma + .info/connected doğrulama birlikte tam çözümü oluşturur.

### 9.29 ✅ exitApp() Yerine moveTaskToBack — Arka Plana Gönderme (TÜM UYGULAMALARDA ZORUNLU)

**Amaç:** Kullanıcı çıkış yaptığında veya bildirimden işlem sonrası uygulama kapanmasın, arka plana gitsin. Tekrar açılışta anında gelsin (cold start yok). WhatsApp, Telegram gibi standart Android davranışı.

**Google Play uyumluluğu:** `Activity.moveTaskToBack(true)` standart Android API'sidir. Google Play politikalarında (2026 Ocak güncellemesi dahil) bunu yasaklayan madde yoktur. Çoğu popüler uygulama bu davranışı kullanır.

**Plugin:** `cordova-android-movetasktoback` — tek Java dosyası, `Activity.moveTaskToBack(true)` çağırır. cordova-android 14 ile tam uyumlu (AndroidX/support bağımlılığı yok).

**config.xml:**
```xml
<plugin name="cordova-android-movetasktoback" spec="*" />
```

**Helper fonksiyon (app.js — back button bölümünden önce):**
```javascript
// ══════════════════════════════════════════════════════════
// ARKA PLANA GONDER (exitApp yerine)
// ══════════════════════════════════════════════════════════
function arkaPlanaGonder() {
  _fn('arkaPlanaGonder');
  if (window.mayflower && mayflower.moveTaskToBack) {
    mayflower.moveTaskToBack();
  }
}
```

**Değiştirilecek noktalar:** Projedeki TÜM `navigator.app.exitApp()` çağrıları `arkaPlanaGonder()` ile değiştirilmeli. Tipik yerler:
1. **Back button handler** — dashboard'dayken çıkış onayı sonrası
2. **Bildirim "Ödendi" aksiyonu** — `on('paid')` handler'da 3sn sonra
3. **Cold start pending paid** — işlem tamamlandıktan sonra
4. **Resume pending paid** — işlem tamamlandıktan sonra
5. **Cold start launchDetails** — notifications.js'deki tüm paid akışları

**Örnek — Back button:**
```javascript
// ❌ ESKİ
showConfirmDialog(t('confirm_exit'), function() {
    navigator.app.exitApp();
});

// ✅ YENİ
showConfirmDialog(t('confirm_exit'), function() {
    arkaPlanaGonder();
});
```

**Örnek — Bildirim ödendi sonrası:**
```javascript
// ❌ ESKİ
setTimeout(function() { if (navigator && navigator.app && navigator.app.exitApp) navigator.app.exitApp(); }, 3000);

// ✅ YENİ
setTimeout(function() { arkaPlanaGonder(); }, 3000);
```

> **Kural:** Projede hiçbir yerde `navigator.app.exitApp()` KALMAMALI. Tüm çıkış noktaları `arkaPlanaGonder()` kullanmalı.
> **Not:** `cikisYap()` (auth.signOut) dokunulmaz — o sadece oturum kapatır, uygulama kapatmaz.

### 9.30 ❌ `hideLoading is not defined` — cikisYapildi() app.js Yüklenmeden Çağrılıyor
**Belirti:** Uygulama açılışında console'da `UNHANDLED_PROMISE: hideLoading is not defined`. Bir kez oluyor, uygulama çalışmaya devam ediyor.
**Sebep:** Script yükleme sırası: `db.js` → ... → `app.js`. db.js'de `onAuthStateChanged` senkron ateşlenir (kullanıcı yok) → `cikisYapildi()` → `hideLoading()` çağrılır. Ama `hideLoading` app.js'de tanımlı ve app.js henüz yüklenmemiş.
**Çözüm:** `cikisYapildi()` içindeki `hideLoading()` çağrısını `typeof` guard ile sar:
```javascript
// ❌ YANLIŞ — app.js yüklenmeden çağrılırsa hata verir
hideLoading();

// ✅ DOĞRU — app.js henüz yüklenmemişse sessizce atla
if (typeof hideLoading === 'function') hideLoading();
```
> **Kural:** db.js'den app.js'deki fonksiyonları çağırırken HER ZAMAN `typeof` guard kullan. Projedeki diğer çapraz-dosya çağrıları (`showToast`, `refreshCurrentPage` vb.) zaten bu pattern'ı kullanıyor.

### 9.31 ❌ Bildirimden "Ödendi" — Ödeme Cache/Firebase'de Bulunamıyor (Stale Notification)
**Belirti:** Bildirimden "Ödendi İşaretle" tıklanıyor. Toast: "Ödeme bulunamadı". Log: `Odeme bulunamadi: local_xxx`.
**Sebep:** `on('paid')` handler'da `paymentId` bildirim data'sından alınıyor ve `paymentsCache.find()` ile aranıyor. Ödeme silinmiş (veya isActive=false yapılmış) ama bildirim hâlâ zamanlanmış durumda. Cache'de ve Firebase'de ödeme bulunamıyor.
**Çözüm (2 katmanlı):**
1. `paymentId` ile bulunamazsa `findPaymentByNotifId(notification.id)` ile ters eşleme dene
2. Hiç bulunamazsa stale bildirimi iptal et ve kullanıcıya bilgi ver

```javascript
// Cache'de yoksa Firebase'den dene (mevcut kod)
if (!payment && fbDb && mevcutKullanici) { ... }

// ✅ YENİ: paymentId ile bulunamadıysa notifId ile ters eşleme dene
if (!payment && notification && notification.id) {
    var foundByNotifId = findPaymentByNotifId(notification.id);
    if (foundByNotifId) {
        payment = foundByNotifId;
        paymentId = foundByNotifId.id;
        console.log('on(paid): paymentId ile bulunamadi, notifId ile bulundu:', paymentId);
    }
}

if (!payment) {
    console.warn('Odeme bulunamadi (silinmis olabilir): ' + paymentId);
    if (typeof showToast === 'function') showToast(typeof t === 'function' ? t('pay_not_found') : 'Odeme bulunamadi');
    // Stale bildirimi iptal et
    if (notification && notification.id) {
        try { cordova.plugins.notification.local.cancel(notification.id); } catch(e) {}
    }
    return;
}
```
> **i18n:** `pay_not_found` anahtarı i18n.js'e eklenmeli: `{ tr: 'Ödeme bulunamadı', en: 'Payment not found' }`

### 9.32 ❌ Stale Bildirimler — local_ ID → Firebase ID Dönüşümü Sonrası Bildirimlerin Güncellenmemesi
**Belirti:** Yeni eklenen ödeme için bildirim geliyor, "Ödendi İşaretle" tıklanıyor → "Ödeme bulunamadı" toast mesajı. Logda: `Odeme bulunamadi (silinmis olabilir): local_1771184903976_fwg8gu1mw`. Firebase ID'li (`-OlXq9...`) bildirimler sorunsuz çalışıyor.
**Sebep:** `syncWithFirebase()` (db.js) `local_` ID'leri Firebase push key'e dönüştürüyor ve `idMap` oluşturuyor. Ancak bildirimler güncellenmiyordu:
1. Ödeme eklenir → `local_xxx` ID → `schedulePaymentNotifications()` ile bildirim zamanlanır (`data.paymentId = 'local_xxx'`)
2. `debouncedSync` → `syncWithFirebase()` → `local_xxx` → `-OlXq9...` dönüşümü
3. `paymentsCache` güncellenir (artık sadece Firebase ID)
4. Bildirim tetiklenir → `data.paymentId = 'local_xxx'` → cache'te bulunamaz
5. `findPaymentByNotifId` de bulamaz çünkü `paymentIdToNumeric('local_xxx') ≠ paymentIdToNumeric('-OlXq9...')`

**Çözüm:** `syncWithFirebase()` sonunda, `paymentsCacheGuncelle()` sonrasına bildirim temizleme bloğu eklendi. `idMap` doluysa eski `local_` ID'li bildirimler iptal edilip yeni Firebase ID ile yeniden zamanlanıyor:
```javascript
// === STALE BİLDİRİM TEMİZLİĞİ ===
// local_ ID -> Firebase ID dönüşümü olduysa eski bildirimleri iptal et, yeni ID ile yeniden zamanla
if (Object.keys(idMap).length > 0 && typeof cancelPaymentNotifications === 'function' && typeof schedulePaymentNotifications === 'function') {
  console.log('syncWithFirebase: idMap dolu (' + Object.keys(idMap).length + ' donusum), bildirimler guncelleniyor...');
  for (var eskiLocalId in idMap) {
    if (idMap.hasOwnProperty(eskiLocalId)) {
      try {
        cancelPaymentNotifications(eskiLocalId);
        var yeniFirebaseId = idMap[eskiLocalId];
        var guncelOdeme = mergedPayments.find(function(p) { return p.id === yeniFirebaseId; });
        if (guncelOdeme && guncelOdeme.isActive !== false && guncelOdeme.status !== 'paid' && guncelOdeme.status !== 'completed') {
          schedulePaymentNotifications(guncelOdeme);
        }
      } catch (notifErr) { console.warn('Bildirim guncelleme hatasi:', notifErr); }
    }
  }
}
```
> **Konum:** db.js — `syncWithFirebase()` fonksiyonu, `paymentsCacheGuncelle();` satırından hemen sonra, `settingsCache = {};` bloğundan önce.
> **Etki:** Ödeme ekle → sync → local_ ID dönüşür → eski bildirim iptal + yeni bildirim zamanlanır. Stale bildirim kalmaz.

### 9.33 ❌ Bildirim Deprecated Property Uyarıları (cordova-plugin-local-notification v1.1.0+)
**Belirti:** Her bildirim zamanlamada console'da 5 uyarı tekrarlanıyor:
```
Property "foreground" is deprecated since version 1.1.0. Use "iOSForeground" instead.
Property "vibrate" is deprecated since version 1.1.1. Use "androidChannelEnableVibration" instead.
Property "smallIcon" is deprecated since version 1.1.0. Use "androidSmallIcon" instead.
Property "priority" is deprecated since version 1.1.0. Use androidChannelImportance, androidAlarmType and androidAllowWhileIdle instead.
Unknown property: iOSForeground
```
**Sebep:** Plugin v1.1.0'da property adları değişti. Eski adlar hâlâ çalışıyor ama deprecated uyarı basıyor. Ödeme başına 2-3 bildirim × 5 uyarı = loglar gereksiz kirleniyor.
**Çözüm:** `notifications.js`'deki 3 `schedule()` bloğunda (schedulePaymentNotifications, scheduleOverdueReminders, snooze handler) eski property'ler yenileriyle değiştirildi:

| Eski (deprecated) | Yeni |
|---|---|
| `foreground: true` | Kaldırıldı (Android'de gereksiz, `iOSForeground` zaten true by default) |
| `vibrate: true` | `androidChannelEnableVibration: true` |
| `smallIcon: 'res://icon'` | `androidSmallIcon: 'res://icon'` |
| `priority: 2` / `priority: 1` | `androidChannelImportance: 'IMPORTANCE_HIGH'` / `'IMPORTANCE_DEFAULT'` |

> **Etki:** Console'daki 10-15 deprecated uyarı satırı tamamen temizlenir. Fonksiyonel değişiklik yok — bildirimler aynı şekilde çalışır.

### 9.34 ❌ `getSetting()` Promise Dönüyor Ama `await` Olmadan Kullanılıyor
**Belirti:** Yaklaşan gün sayısı ayarı 30 seçilse bile dashboard her zaman 7 gün kullanıyor. Ayarlarda seçili değer boş/yanlış görünüyor.
**Sebep:** `getSetting()` fonksiyonu **Promise** döner. `parseInt(getSetting('upcomingDays'))` → `parseInt(Promise)` = `NaN` → `NaN || 7` = **her zaman 7**. Benzer şekilde `String(getSetting('upcomingDays'))` → `"[object Promise]"` → select'te eşleşen option yok → seçim boş görünür.
**Çözüm:** `getSetting()` çağrıları HER ZAMAN `await` ile kullanılmalı:
```javascript
// ❌ YANLIŞ — parseInt(Promise) = NaN, her zaman default'a düşer
var upcomingDays = parseInt(getSetting('upcomingDays')) || 7;

// ✅ DOĞRU
var upcomingDays = parseInt(await getSetting('upcomingDays')) || 7;
```
> **Kural:** `getSetting()` dönen değeri doğrudan kullanan HER YERDE `await` olmalı. `getSetting` senkron görünse de Promise wrapper döner. Yeni kod yazarken veya mevcut kodu değiştirirken bu kontrol edilmeli.

### 9.35 ❌ Not/Kayıt Düzenlenince Kopya Oluşuyor — `local_` → Firebase ID Dönüşümü Race Condition
**Belirti:** Bir notu (veya kaydı) düzenleyip kaydedince orijinal not duruyor, bir de kopyası oluşuyor. Her düzenlemede yeni kopya ekleniyor.
**Sebep:** `updateNote()` (veya benzer update fonksiyonu) içinde `if (!bulundu) rawArray.push(item)` satırı var. Akış:
1. Not eklenir → `local_xxx` ID → DOM'a render edilir
2. 2sn sonra `debouncedSync()` → `local_xxx` → Firebase push key `-OlXxx` olur → localStorage güncellenir
3. Ama DOM yeniden render edilmez — düzenle butonu hâlâ `local_xxx` ID'sini tutuyor
4. Kullanıcı "Düzenle" tıklar → form'daki hidden input `local_xxx` (eski ID)
5. Kaydet → `update({id: 'local_xxx', ...})` → localStorage'da artık `-OlXxx` var → `local_xxx` BULUNAMAZ
6. `if (!bulundu) rawArray.push(item)` → **KOPYA OLUŞUR!**
7. Sonraki sync'te `local_xxx` tekrar Firebase'e push edilir → 2 ayrı kayıt

**Çözüm (2 katmanlı — update fonksiyonu + save fonksiyonu):**

**1) `updateNote()` / `updateItem()` — ID bulunamazsa `createdAt` ile eşleştir, push YAPMA:**
```javascript
async function updateNote(note) {
  _fn('updateNote');
  if (!note || !note.id) return;
  if (!mevcutKullanici) return;

  note.ts = Date.now();

  var rawNotes = lokalVeriOku('oh_notes') || [];
  var bulundu = false;
  for (var i = 0; i < rawNotes.length; i++) {
    if (rawNotes[i].id === note.id) {
      rawNotes[i] = Object.assign({}, note);
      bulundu = true;
      break;
    }
  }

  // local_ ID sync sirasinda Firebase ID'ye donusmus olabilir — createdAt ile eslestir
  if (!bulundu && note.createdAt) {
    for (var j = 0; j < rawNotes.length; j++) {
      if (rawNotes[j].createdAt === note.createdAt && !rawNotes[j]._deleted) {
        console.log('updateNote: ID degismis (' + note.id + ' -> ' + rawNotes[j].id + '), createdAt ile eslestirildi');
        note.id = rawNotes[j].id;
        rawNotes[j] = Object.assign({}, note);
        bulundu = true;
        break;
      }
    }
  }

  if (!bulundu) {
    console.warn('updateNote: Not bulunamadi, kopya olusturulmadi. ID:', note.id);
  }

  lokalVeriYaz('oh_notes', rawNotes);
  notesCacheGuncelle();
  _degisiklikVar = true;
  debouncedSync();
}
```

**2) `saveNote()` / `saveItem()` — kaydetmeden önce ID dönüşmüşse güncel ID'yi bul:**
```javascript
// Global değişken (app.js başına):
var _editingNoteCreatedAt = null;

// showEditNoteForm() — form açılırken createdAt kaydet:
_editingNoteCreatedAt = note.createdAt || null;

// showAddNoteForm() — sıfırla:
_editingNoteCreatedAt = null;

// saveNote() — editId cache'te yoksa createdAt ile eşleştir:
if (editId) {
    noteObj.id = editId;
    var existing = getNote(editId);

    if (!existing && _editingNoteCreatedAt) {
      for (var ni = 0; ni < notesCache.length; ni++) {
        if (notesCache[ni].createdAt === _editingNoteCreatedAt && !notesCache[ni]._deleted) {
          existing = notesCache[ni];
          noteObj.id = existing.id;
          console.log('saveNote: editId degismis, createdAt ile eslestirildi');
          break;
        }
      }
    }

    if (existing) noteObj.createdAt = existing.createdAt;
    await updateNote(noteObj);
}
```

> **Kök kural:** `update` fonksiyonlarında `if (!bulundu) array.push(item)` KULLANMA — bu kopya oluşturur. ID bulunamazsa `createdAt` fallback ile ara, o da bulamazsa sessizce logla ve push YAPMA.
> **Not:** Bu sorun sadece notes değil, `local_` → Firebase ID dönüşümü yapan TÜM koleksiyonlarda (payments, paidInstances vb.) olabilir. Her `update` fonksiyonunu kontrol et.

### 9.36 ❌ İnternet Kesilip Geri Gelince Sync Çalışmıyor — WebSocket Bağlanamıyor (REST API Fallback)
**Belirti:** İnternet kapatılıp geri açıldıktan sonra `syncWithFirebase` her çağrıda 20sn TIMEOUT'a düşüyor. Token yenilenebiliyor (HTTPS çalışıyor) ama `.once('value')` (WebSocket) hiç dönmüyor. Task kill ile kapatıp açınca da aynı sorun devam ediyor (cold start bile çözmüyor).
**Sebep:** Firebase JS SDK (compat) WebSocket bağlantısı internet kesintisinden sonra recover edemiyor. Bu bilinen bir sorun (GitHub #7670, #8718, #594). `goOffline()/goOnline()` reset döngüleri SDK'nın dahili exponential backoff retry mekanizmasını bozuyor. Token yenileme HTTPS ile çalışıyor ama WebSocket ayrı bir kanal ve bağımsız olarak bozuluyor.
**Denenen ve işe yaramayan çözümler:**
- `syncWithFirebase` finally bloğundaki `goOffline()` kaldırma → kısmen yardımcı ama WebSocket hâlâ recover edemiyor
- `pauseSync()` içindeki `goOffline()` kaldırma → race condition düzeldi ama WebSocket sorunu devam
- `goOffline()/goOnline()` reset döngülerini tamamen kaldırma → SDK kendi reconnect'ini yapamıyor
- `.info/connected` ile bağlantı doğrulama → WebSocket bağlanamayınca hiç `true` dönmüyor

**Çözüm — Firebase REST API Fallback (HTTPS):**
Firebase Realtime Database iki farklı kanalla erişilebilir:
1. **WebSocket (SDK)** — `once('value')`, `ref.set()` → normal çalışırken hızlı
2. **REST API (HTTPS)** — `https://DB_URL/path.json?auth=TOKEN` ile GET/PUT/POST → WebSocket bozulunca fallback

Sync akışı:
1. `goOnline()` → `_waitForConnection(5sn)` → bağlandıysa **WebSocket** ile devam
2. 5sn bağlanamadıysa → `getIdToken()` ile auth token al → **REST API** ile `fetch()` (HTTPS)
3. REST API okuma: `GET /users/UID/payments.json?auth=TOKEN`
4. REST API yazma: `PUT /users/UID/payments.json?auth=TOKEN` + body
5. REST API push (local_ ID): `POST /users/UID/payments.json?auth=TOKEN` → `{ name: "-OlXq9..." }` döner

**db.js — REST API helper fonksiyonları (`_waitForConnection`'dan sonra):**
```javascript
var _FIREBASE_DB_URL = firebaseConfig.databaseURL;

async function _restApiRead(path, token) {
  var url = _FIREBASE_DB_URL + '/' + path + '.json?auth=' + token;
  var resp = await fetch(url, { method: 'GET' });
  if (!resp.ok) throw new Error('REST_READ_FAILED: ' + resp.status);
  return await resp.json();
}

async function _restApiWrite(path, data, token) {
  var url = _FIREBASE_DB_URL + '/' + path + '.json?auth=' + token;
  var resp = await fetch(url, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(data) });
  if (!resp.ok) throw new Error('REST_WRITE_FAILED: ' + resp.status);
  return await resp.json();
}

async function _restApiPush(path, data, token) {
  var url = _FIREBASE_DB_URL + '/' + path + '.json?auth=' + token;
  var resp = await fetch(url, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(data) });
  if (!resp.ok) throw new Error('REST_PUSH_FAILED: ' + resp.status);
  var result = await resp.json();
  return result.name; // Firebase push key
}

async function _restApiDelete(path, token) {
  var url = _FIREBASE_DB_URL + '/' + path + '.json?auth=' + token;
  var resp = await fetch(url, { method: 'DELETE' });
  if (!resp.ok) throw new Error('REST_DELETE_FAILED: ' + resp.status);
}
```

**syncWithFirebase — WebSocket/REST mod seçimi:**
```javascript
var _useRest = false;
var _authToken = null;

// WebSocket dene
fbDb.goOnline();
var baglandi = await _waitForConnection(5000);
if (baglandi) {
  console.log('syncWithFirebase: Firebase BAGLI (WebSocket)');
} else {
  console.warn('syncWithFirebase: WebSocket 5sn baglanamadi, REST API fallback...');
  _useRest = true;
  _authToken = await mevcutKullanici.getIdToken(false);
}

// Okuma
if (_useRest) {
  fbPayments = (await _restApiRead('users/' + uid + '/payments', _authToken)) || {};
  // ... diğer koleksiyonlar
} else {
  var fbPaySnap = await _firebaseTimeout(fbDb.ref('users/' + uid + '/payments').once('value'), 20000);
  // ... diğer koleksiyonlar
}

// Yazma
if (_useRest) {
  await _restApiWrite('users/' + uid + '/payments', fbPayObj, _authToken);
  // ... diğer koleksiyonlar
} else {
  await fbDb.ref('users/' + uid + '/payments').set(fbPayObj);
  // ... diğer koleksiyonlar
}
```
> **Log:** Başarılı sync'te `syncWithFirebase BASARILI (REST)` veya `(WebSocket)` yazılır.
> **Performans:** REST API ~100-300ms/istek (HTTPS). WebSocket'e göre biraz yavaş ama TIMEOUT'tan (20sn × 3 retry = 60sn) çok daha iyi.
> **CSP:** `connect-src` içinde `https://*.firebasedatabase.app` zaten mevcut — REST API aynı domain'i kullanır.
> **İlişkili:** 9.26 (getIdToken), 9.28 (goOffline kaldırma), 9.19 (CSP) ile birlikte çalışır.


### 9.37 ⚠️ Edge-to-Edge — Android 16 (API 36) ile Zorunlu Hale Geldi
**Durum:** cordova-android 14.0.0/14.0.1 Edge-to-Edge'i opt-out etti — uygulamalar eski davranışı (status bar + navigation bar ayrı) koruyor. Ancak Android 16 (API 36) hedefleyen uygulamalarda Google opt-out'u kaldırdı — Edge-to-Edge zorunlu.
**Şu an etki:** `android-targetSdkVersion="35"` kullanan uygulamalar etkilenmez. cordova-android 14 bu durumu yönetiyor.
**Gelecek etki:** Google Play 2026 sonrası API 36 zorunlu kıldığında TÜM uygulamalar etkilenecek.
**Hazırlık:**
1. `viewport-fit=cover` meta tag'ını şimdiden ekle (Bölüm 4'teki şablonda mevcut)
2. CSS'de safe area inset'leri kullan: `padding-top: env(safe-area-inset-top);`
3. `StatusBarOverlaysWebView` preference'ını `true` yapmayı test et
4. cordova-android 15+ çıktığında Edge-to-Edge desteğini kontrol et

### 9.38 ⚠️ 16KB Page Size — Kasım 2025+ Google Play Zorunluluğu
**Durum:** 1 Kasım 2025'ten itibaren API 35+ hedefleyen ve native library (.so) içeren uygulamalar 16KB page size uyumlu olmalı. Google Play uyumsuz uygulamaları reddeder.
**Cordova etkisi:** Cordova ve WebView kendisi native lib içermez — saf JS/HTML/CSS uygulamaları otomatik uyumludur. Ancak bazı plugin'ler native lib ekler:
- `cordova-sqlite-storage` — SQLite native lib (16KB uyumsuz, güncel versiyon gerekli)
- `cordova-plugin-iroot` — root detection lib (16KB uyumsuz)
- `@sentry/capacitor` — crash reporting (v1.5.0+ uyumlu)
**Kontrol komutu:**
```powershell
# APK'daki .so dosyalarının 16KB alignment kontrolü
$ANDROID_HOMEuild-tools.0.0\zipalign -c -p -v 16 4 app.apk
```
**Çözüm:** Kullandığın plugin'lerin en güncel versiyonlarını yükle. Plugin'de 16KB uyumsuz .so varsa plugin geliştiricisine bildir veya alternatif plugin kullan. AGP 8.5.1+ kullanıldığında Cordova build'leri genellikle otomatik uyumludur.
> **Not:** Android 16 16KB uyumsuz uygulamalar için "backcompat mode" ekledi — uygulama çalışır ama performans kaybı olur ve kullanıcıya uyarı gösterilir.

---

## 10. Gerekli PNG Dosyaları (KRİTİK HATIRLATMA)

Projeye **üç farklı PNG** gerekir. Bunlar projede fiziksel olarak bulunmalı:

| Dosya | Konum | Boyut | Açıklama |
|---|---|---|---|
| **icon.png** | `res/icon.png` | min 512×512 | Uygulama ikonu. Lokal build + VoltBuilder'da kullanılır. config.xml'de `<icon src="res/icon.png" />` ile referanslanır. |
| **iconTemplate.png** | `resources/iconTemplate.png` | 1024×1024 | VoltBuilder otomatik ikon üretimi. Koyarsan VoltBuilder tüm boyutları (mdpi, hdpi, xhdpi vb.) kendisi üretir. |
| **splashTemplate.png** | `resources/splashTemplate.png` | 2732×2732 | VoltBuilder splash screen üretimi. Ortada ikon, etraf boş/şeffaf. config.xml'de `AndroidWindowSplashScreenAnimatedIcon` ile referanslanır. |

### ⚠️ Kurallar
- `res/icon.png` ZORUNLU — yoksa varsayılan Cordova robotu görünür
- `www/img/` altına ikon KOYMA — Cordova bunu uygulama ikonu olarak tanımaz
- `resources/` dosyaları sadece VoltBuilder için — lokal build'de kullanılmaz
- PNG formatı olmalı (JPG, WebP OLMAZ)
- iconTemplate **şeffaf arka plan** önerilir (adaptive icon uyumu)

---

## 11. Google Play Gereksinimleri (2025-2026)

| Gereksinim | Değer |
|---|---|
| **Yeni uygulama/güncelleme targetSdkVersion** | 35 (31 Ağustos 2025+ zorunlu) |
| **Mevcut uygulama targetSdkVersion** | 34 minimum (yoksa Android 15+ cihazlarda görünmez) |
| **Uzatma talebi** | 1 Kasım 2025'e kadar (Play Console'dan başvuru) |
| **16KB page size** | 1 Kasım 2025+ API 35 hedefleyen uygulamalarda zorunlu (native lib varsa) |
| **Edge-to-Edge** | Android 16 (API 36) ile opt-out kalktı — zorunlu hale geldi |
| **Format** | AAB tercih edilir |
| **İmzalama** | Keystore ile imzalı |
| **Hesap** | https://play.google.com/console/signup ($25) |

### 16KB Page Size Notu (Cordova)
Cordova ve Capacitor kendileri native lib (.so) içermez — saf JS/HTML/CSS uygulamaları otomatik uyumludur. Ancak bazı plugin'ler native lib ekleyebilir (`cordova-sqlite-storage`, `cordova-plugin-iroot`, `@sentry/capacitor` vb.). Plugin'lerin 16KB uyumlu versiyonlarını kullanın. Kontrol: `zipalign -c -p -v 16 4 app.apk`

### Edge-to-Edge Notu (Cordova Android 14)
cordova-android 14.0.0 Edge-to-Edge'i opt-out etti (eski davranışı koruyor). Ancak Android 16 (API 36) hedefleyen uygulamalarda opt-out kalktı — Edge-to-Edge zorunlu. `viewport-fit=cover` ve safe area inset CSS'leri hazırlanmalı. Şimdilik API 35 hedefleyenler etkilenmiyor.

### Yükleme Adımları
1. Play Console → "Uygulama oluştur"
2. "Üretim" → "Yeni sürüm" → .aab yükle
3. Uygulama bilgileri, ekran görüntüleri, gizlilik politikası
4. İçerik derecelendirmesi + fiyatlandırma
5. Yayınla


## 12. Faydalı Komutlar

```powershell
cordova platform add android
cordova platform remove android
cordova plugin add cordova-plugin-device
cordova plugin list
cordova build android                                         # Debug APK
cordova build android --release                               # Release APK
cordova build android --release -- --packageType=bundle       # Release AAB
cordova clean android
adb install platforms\android\app\build\outputs\apk\debug\app-debug.apk
adb devices
taskkill /F /IM java.exe                                      # Gradle daemon kill
```

---

## 13. Kontrol Listesi

### Lokal Build
- [ ] Node.js v20+, JDK 17, Android SDK (API 35), Cordova CLI kurulu
- [ ] `res/icon.png` mevcut + config.xml'de `<icon src="res/icon.png" />`
- [ ] config.xml'de `<engine name="android" spec="14.0.1" />` mevcut (EKSİKSE EKLEME UNUT!)
- [ ] `index.html`'de `<script src="cordova.js"></script>` var
- [ ] Firebase/harici SDK'lar `www/lib/` altında LOKAL (CDN'den DEĞİL!)
- [ ] CSP meta tag'ında `*.firebasedatabase.app` + `wss://*.firebasedatabase.app` var (bölgesel DB kullanıyorsan ZORUNLU!)
- [ ] Google Fonts vb. `www/fonts/` altında LOKAL (CDN'den DEĞİL!)
- [ ] `firebase.initializeApp()` try-catch ile sarılı + `firebaseReady` flag kullanılıyor
- [ ] `deviceready` handler'da her plugin çağrısı ayrı try-catch içinde
- [ ] `cordova platform add android` başarılı
- [ ] `cdv-gradle-config.json`'da appcompat `"1.5.1"` (lokal build zorunlu, VoltBuilder hata verirse de uygula)
- [ ] AdMob kullanıyorsan: `admob-plus-cordova` kullanılıyor (`cordova-plugin-admob-free` DEĞİL!)
- [ ] AdMob plugin variable adı: `APP_ID_ANDROID` (`ADMOB_APP_ID` DEĞİL — crash sebebi!)
- [ ] AdMob: `await admob.start()` banner oluşturmadan ÖNCE çağrılıyor (yoksa reklam yüklenmez!)
- [ ] IAP: `checkPremiumStatus()` Google Play'de satın alma yoksa `isPremium`'ı ezmiyor (Firebase premium korunsun!)
- [ ] AdMob: `<platform name="android">` içinde `GradlePluginKotlinEnabled` + `AndroidXEnabled` = `true` (yoksa admob objesi tanımsız!)
- [ ] Google ile giriş: `signInWithRedirect` kullanılmıyor (Cordova'da çalışmaz)
- [ ] Harici URL'ler: `<a href target="_blank">` DEĞİL, `window.open(url, '_system')` kullanılıyor
- [ ] Dosya dışa aktarma: `<a download>` DEĞİL, socialsharing `df:dosyaadi;data:mimetype;base64,DATA` formatı kullanılıyor (message parametresi YOK!)
- [ ] `res/icon.png` (512×512+), `resources/iconTemplate.png` (1024×1024), `resources/splashTemplate.png` (2732×2732) mevcut
- [ ] Bildirim ödendi aksiyonu: `findPaymentByNotifId()` mevcut + `on('paid')` ve cold start'ta `notification.id`'den ters eşleme kullanılıyor (data'ya bağımlılık YOK!)
- [ ] PowerShell scriptleri ASCII-only (Türkçe karakter yok!)
- [ ] `getSetting()` çağrıları HER YERDE `await` ile kullanılıyor (Promise döner — `await` olmadan NaN/[object Promise] olur!)
- [ ] Logger: Kalıcı ring buffer (1000 satır) db.js'in EN BAŞINDA (Firebase config'den önce)
- [ ] Logger: `_fn()` fonksiyonu tanımlı + localStorage flush (5sn interval + pause/beforeunload)
- [ ] Logger: `_LOG_STORAGE_KEY` projeye özel ayarlanmış (örn: `oh_logBuffer`)
- [ ] Logger: Tüm fonksiyonlara `_fn('fonksiyonAdi')` eklenmiş (exclude listesindekiler HARİÇ!)
- [ ] Logger: Exclude listesindeki yardımcı fonksiyonlarda (t, escapeHtml, formatMoney vb.) `_fn()` YOK
- [ ] Logger: `hataRaporuGonder()` fonksiyonu db.js'in sonunda
- [ ] Logger: Ayarlar sayfasında "🐛 Hata Raporu Gönder" butonu mevcut
- [ ] Logger: i18n.js'de `set_bug_report` çevirisi mevcut
- [ ] Sync: `syncWithFirebase()` içinde `getIdToken(true)` goOnline'dan ÖNCE çağrılıyor
- [ ] Sync: `syncWithFirebase()` finally bloğunda `goOffline()` YOK (WebSocket'i öldürür!)
- [ ] Sync: `pauseSync()` içinde `goOffline()` YOK (async sync'in WebSocket'ini öldürür!)
- [ ] Sync: `_waitForConnection()` helper mevcut — `.info/connected` ile gerçek bağlantı doğrulama (sabit bekleme DEĞİL!)
- [ ] Sync: REST API fallback mevcut — WebSocket bağlanamadığında `_restApiRead/Write/Push` ile HTTPS üzerinden sync
- [ ] Sync: catch bloğunda gerçek hata loglanıyor (maskeleme yok!)
- [ ] moveTaskToBack: `cordova-android-movetasktoback` plugin config.xml'de mevcut
- [ ] moveTaskToBack: `arkaPlanaGonder()` fonksiyonu app.js'de tanımlı
- [ ] moveTaskToBack: Projede `navigator.app.exitApp()` KALMADI — tümü `arkaPlanaGonder()` ile değiştirildi
- [ ] db.js: `cikisYapildi()` içinde `hideLoading()` çağrısı `typeof` guard ile sarılı
- [ ] notifications.js: `on('paid')` handler'da ödeme bulunamazsa `findPaymentByNotifId` fallback + stale bildirim iptal
- [ ] i18n.js: `pay_not_found` çevirisi mevcut
- [ ] db.js: `syncWithFirebase()` sonunda `idMap` doluysa eski `local_` ID'li bildirimleri iptal + yeni Firebase ID ile yeniden zamanla
- [ ] notifications.js: `schedule()` bloklarında deprecated property YOK (`foreground`, `vibrate`, `smallIcon`, `priority` → yeni adlarla değiştirilmiş)
- [ ] update fonksiyonlarında `if (!bulundu) array.push(item)` YOK — `createdAt` fallback ile eşleştir, kopya oluşturma!
- [ ] `cordova build android` → BUILD SUCCESSFUL
- [ ] config.xml'de `android-versionCode` mevcut ve her güncellemede artırılıyor
- [ ] config.xml'de `xmlns:android` namespace'i widget tag'ında mevcut
- [ ] index.html'de CSP meta tag'ı mevcut (Firebase/AdMob domain'leri dahil)
- [ ] index.html'de `viewport-fit=cover` mevcut (Edge-to-Edge hazırlığı)
- [ ] index.html'de `skipLocalNotificationReady = true` mevcut (bildirim plugin'i varsa)
- [ ] Splash path: `resources/splashTemplate.png` (`res/screen/android/` DEĞİL!)
- [ ] Alarm permission'lar: `SCHEDULE_EXACT_ALARM` (maxSdkVersion=32) + `USE_EXACT_ALARM` birlikte tanımlı
- [ ] 16KB page size: Plugin'lerde native lib (.so) varsa 16KB uyumlu versiyonlar kullanılıyor

### VoltBuilder
- [ ] config.xml doğru yapıda
- [ ] config.xml'de `<engine name="android" spec="14.0.1" />` mevcut
- [ ] `admob-plus-cordova` variable adı: `APP_ID_ANDROID` (ADMOB_APP_ID DEĞİL!)
- [ ] AdMob: `await admob.start()` banner oluşturmadan ÖNCE çağrılıyor
- [ ] IAP: `checkPremiumStatus()` Google Play'de satın alma yoksa `isPremium`'ı ezmiyor (Firebase premium korunsun!)
- [ ] AdMob: `GradlePluginKotlinEnabled` + `AndroidXEnabled` = `true` (`<platform name="android">` içinde)
- [ ] `res/icon.png` (512×512+) mevcut
- [ ] `resources/iconTemplate.png` (1024×1024) mevcut
- [ ] `resources/splashTemplate.png` (2732×2732) mevcut (opsiyonel)
- [ ] voltbuilder.json mevcut
- [ ] ZIP yapısı doğru (config.xml kökte)
- [ ] Harici CDN bağımlılıkları lokal dosyalarla değiştirilmiş
- [ ] Logger: Kalıcı ring buffer + `_fn()` izleme + Hata Raporu Gönder butonu mevcut
- [ ] Sync: `getIdToken(true)` goOnline'dan ÖNCE çağrılıyor + hata maskeleme kaldırılmış
- [ ] Sync: `syncWithFirebase()` finally bloğunda `goOffline()` YOK
- [ ] Sync: `pauseSync()` içinde `goOffline()` YOK (async sync'in WebSocket'ini öldürür!)
- [ ] Sync: `_waitForConnection()` helper mevcut — `.info/connected` ile gerçek bağlantı doğrulama
- [ ] Sync: REST API fallback mevcut — WebSocket bağlanamadığında HTTPS ile sync
- [ ] moveTaskToBack: `cordova-android-movetasktoback` plugin config.xml'de mevcut
- [ ] moveTaskToBack: Projede `navigator.app.exitApp()` KALMADI — tümü `arkaPlanaGonder()` ile değiştirildi
- [ ] db.js: `cikisYapildi()` içinde `hideLoading()` çağrısı `typeof` guard ile sarılı
- [ ] notifications.js: `on('paid')` handler'da ödeme bulunamazsa `findPaymentByNotifId` fallback + stale bildirim iptal
- [ ] i18n.js: `pay_not_found` çevirisi mevcut
- [ ] db.js: `syncWithFirebase()` sonunda `idMap` doluysa eski `local_` ID'li bildirimleri iptal + yeni Firebase ID ile yeniden zamanla
- [ ] notifications.js: `schedule()` bloklarında deprecated property YOK (`foreground`, `vibrate`, `smallIcon`, `priority` → yeni adlarla değiştirilmiş)
- [ ] `getSetting()` çağrıları HER YERDE `await` ile kullanılıyor (Promise döner — `await` olmadan NaN/[object Promise] olur!)
- [ ] update fonksiyonlarında `if (!bulundu) array.push(item)` YOK — `createdAt` fallback ile eşleştir, kopya oluşturma!

### Google Play
- [ ] Keystore oluşturuldu + yedeklendi
- [ ] AAB build başarılı
- [ ] Play Console hesabı açık

---

## 14. Geliştirici Bilgileri (Proje Sabitleri)

> Bu bilgiler config.xml, privacy policy, Google Play Console ve diğer tüm dosyalarda kullanılır. Tutarsızlık olmaması için tek kaynak burası.

| Bilgi | Değer |
|---|---|
| **Ad Soyad** | Recep Yeni |
| **E-posta** | recepyeni@gmail.com |
| **Telefon** | +905056279048 |
| **Şehir / İl / Ülke** | Tekirdağ / Tekirdağ / Türkiye |
| **GitHub Pages** | recinilt.github.io |
| **CORS Proxy** | https://mycors.recepyeni.workers.dev |
| **Bilgisayar** | Windows 10 Pro, RTX 3050 laptop |
| **config.xml author email** | recepyeni@gmail.com |


### Google Play Console Bilgileri
| Alan | Değer |
|---|---|
| Geliştirici adı | Recep Yeni |
| İletişim e-postası | recepyeni@gmail.com |
| İletişim telefonu | +905056279048 |
| Adres | Tekirdağ, Türkiye |

### ⚠️ Kurallar
- `recepyeni.com` domaini YOK — kullanma
- Aktif domain: `https://recinilt.github.io/mefeypublic/recinilt/`
- Privacy policy GitHub Pages'te barındırılıyor (yukarıdaki URL, / işaretinden sonraki kısmı sor.)

---

## 15. Hesap Silme Özelliği (Google Play Zorunluluğu)

> Google Play, Kasım 2023'ten itibaren kullanıcılara uygulama içinden hesap ve veri silme imkanı sunmayı zorunlu kıldı. 

### Uygulama İçi Akış
1. Menü → "Hesabımı Sil" butonuna tıkla
2. Onay modalı açılır:
   - Silinecekler listesi (şifreler, hesap, yerel veriler)
   - Premium kullanıcıysa ek uyarı: "Premium özelliğiniz de silinecektir, tekrar satın almanız gerekecektir"
3. Kullanıcı giriş şifresini girer (Firebase reauthentication zorunlu)
4. "Kalıcı Olarak Sil" → sırasıyla:
   - Firebase Realtime Database'den tüm veriler silinir
   - localStorage temizlenir (IAP durumu dahil)
   - Firebase Auth hesabı silinir
   - Uygulama giriş ekranına döner

### İlgili Dosyalar
- `index.html`: "Hesabımı Sil" butonu (alt menüde) + onay modalı (`#modal-hesap-sil`)
- `app.js`: `hesapSilOnay()` + `hesapSilKalici()` fonksiyonları

### Privacy Policy
Gizlilik politikasında "Hesabımı Sil" özelliği ve e-posta ile silme talebi bilgisi mevcut (Bölüm 5).

---

## 16. ZIP Teslimi Sonrası Analiz Akışı (Claude İçin Talimat)

> Bu bölüm, proje dosyaları hazırlanıp ZIP olarak sunulduktan sonra Claude'un uygulaması gereken otomatik analiz akışını tanımlar.

### Akış Adımları

ZIP dosyası kullanıcıya sunulduktan sonra Claude şu **üç soruyu sırasıyla** sorar:

**Adım 1 — Özellik Sayımı:**
> "Bu uygulamanın tüm özelliklerini, madde madde, madde atlamadan derin analiz ile kısaca sayayım mı?"

Kullanıcı onaylarsa: Uygulamadaki tüm özellikler (UI, backend, plugin, güvenlik, veri yönetimi vb.) tek tek kısaca listelenir. Hiçbir özellik atlanmaz.

**Adım 2 — Akış Sıralaması ve Dallandırma:**
> "Şimdi bu saydığım maddeleri, akış sırasına göre sıralayıp ve dallandırayım mı?"

Kullanıcı onaylarsa: Adım 1'deki maddeler kullanıcı akışına göre (uygulama açılışından itibaren) sıralanır ve dallanma noktaları (if/else, koşullu akışlar) gösterilir.

**Adım 3 — Çalışırlık Kontrolü:**
> "Bu tüm maddelerin her birinin çalışırlığını akış sırasına göre kontrol edip, fonksiyonlar normal çalışıyor mu, akışta tıkanan aksayan yer var mı, bir şey atlamadan, akışta problem olan noktaları not alıp ve sanki düzelmiş gibi akışa devam edip bakayım mı? Sonra da problem olan noktaları sana yazayım mı?"

Kullanıcı onaylarsa: Her madde akış sırasında fonksiyonel olarak kontrol edilir. Problem tespit edilen noktalar not alınır ama akış kesilmez — sanki düzelmiş gibi devam edilir. En sonda tüm problemli noktalar toplu olarak raporlanır.

### ⚠️ Kurallar
- Her adım için kullanıcıdan **onay** beklenir
- Onay gelmeden bir sonraki adıma geçilmez
- Bu akış sadece ZIP tesliminden sonra tetiklenir
- Dosyalara dokunulmaz — sadece analiz ve bilgilendirme yapılır
- Problem tespiti yapılır ama düzeltme **uygulanmaz** (onay beklenir)

---

## 17. Claude Instructions (Sohbet Kuralları)

> Bu bölüm, Claude ile yapılan sohbetlerde uyulması gereken kuralları tanımlar. Her sohbetin başına bu kurallar eklenir.

<KURAL_ÖZETI — HER MESAJDA OKU>
K1: KISA YAZ. K2: Soru=sadece cevapla, izinsiz düzeltme yok.
K3: Dosya güncellemede: kaynak belirt, tek tek güncelle, tam dosya yaz, değişmeyenler aynen kalsın, API/credential dokunulmaz.
K4.0: Normal istek=2 döngü (websiz analiz + webli doğrulama), "Uygulayayım mı?" sor.
K4.1: Web'li derin analiz=4 döngü web_search, dosyaya dokunma, "Uygulayayım mı?" sor.
K4.2: Derin analiz=4 döngü, dosyaya dokunma, "Uygulayayım mı?" sor.
K4.3: Kod yazarken önce/sırasında/sonrasında web doğrula.
K5: YASAK: gereksiz açıklama, özür, kısmi kod, web atlamak, izinsiz değişken/isim/config değiştirme.
K6: "güncelle/yap/düzelt/uygula" YOK = KOD YAZMA. Onay bekle.
K7: Sadece istenen değişiklik. İyileştirme, ekleme, çıkarma yok.
K-LOGGER: Yeni fonksiyonda ilk satır _fn('ad'); — exclude listesi hariç.
UTF-8, Türkçe karakterler bozulmasın: ğüşıöçĞÜŞİÖÇ
</KURAL_ÖZETI>

### Karakter Kodlama Kuralı
- TÜM DOSYALAR UTF-8 (BOM olmadan)
- TÜRKÇE KARAKTERLER: ğüşıöçĞÜŞİÖÇ
- TEST CÜMLESİ: "Çöğüş işini böyle yapmışsın"
- Kod yazarken/okurken Türkçe karakterleri ASLA bozma
- Emoji desteği: 🎬🚀📋▶️⏸️

### Genel Kurallar

**1. KISA YAZ**
- Rapor kelimesi yok mu? Rapor yok. Kısa, net, öz cevap ver.

**2. SORU SORULUYORSA**
- Sadece cevapla. Düzelt demeden düzeltme yapma.

**3. DOSYA GÜNCELLERKEN**
- Hangi dosya ile çalışacağını ve hangi kaynakta (bilgi bankası/mesaj/sohbetteki şu sırada eklenen dosya vb.) olduğu bilgisini mutlaka ver.
- Bu dosya 1000+ satır. Lütfen sadece gerekli satırları değiştir.
- OKU VE HAFIZAYA AL: tüm değişkenler, fonksiyon isimleri, config değerleri, API anahtarları
- Mevcut özellikleri içinden say
- Proje akışını anla
- Güncellemeyi planla
- Çalışacağını kontrol et
- Güncellenecek dosyaların sadece isimlerini listele.
- DOSYALARI TOPLU DEĞİL, TEK TEK GÜNCELLE: Birden fazla dosya güncellenecekse, her dosyayı ayrı ayrı oluştur. Bir sonraki dosyaya geçmeden önce "Sıradaki dosya: [dosya adı]. Güncelleyeyim mi?" diye onay al. Onay gelmeden sonraki dosyaya geçme.
- DOSYANIN TAMAMINI EKSİKSİZ YAZ (baştan sona tam kod)
- SONRASINDA KONTROL ET: güncelleme dışındaki her şey AYNEN mi
- Kontrol: değişken isimleri, fonksiyon isimleri, Firebase ayarları, API anahtarları vb. değişmemiş mi
- Etkilenen özellikleri bildir (hangileri eklendi/çıkarıldı/değişti)
- Eğer online bir sitedeki kod ile ilişkili bir proje üzerinde çalışıyorsan (mesela firebase rules gibi), orayı da güncelleme gerekir mi bilgilendir.
- İstenmeyen özellik ekleme
- İstenmedikçe mevcut özellik çıkarma
- Mevcut koda "iyileştirme" yapma
- Sadece istenen güncellemeyi yap
- Geri kalan her şey AYNEN kalsın
- Firebase ayarları, API anahtarları, credentials = DOKUNULMAZdır (özel istek olmadıkça)
- BAĞIMLILIK UYUM KONTROLÜ: Programa eklenen her kütüphane/paket için, mevcut tüm bağımlılıklarla versiyon uyumunu KONTROL ET. "uv pip install X Y Z" gibi toplu kurulumlarda X↔Y, X↔Z, Y↔Z çapraz uyumunu doğrula. Uyumsuzluk varsa KOD YAZMADAN ÖNCE bildir.
- PROGRAM AKIŞ KONTROLÜ: Güncelleme SONRASINDA programın TÜM dallanmalarını (if/else, try/except, fonksiyon çağrıları, değişken kullanımları) baştan sona tara. Bir hücrede/fonksiyonda tanımlanan değişkenin, kullanıldığı TÜM diğer hücrelerde/fonksiyonlarda doğru şekilde set edildiğini, geçirildiğini ve okunduğunu doğrula. Eksik bağlantı varsa (örn: HF_TOKEN tanımlandı ama okunamıyor, DOSYA_YOLU set edildi ama sonraki hücre görmüyor) KOD YAZMADAN ÖNCE bildir.
- Programın tasarımı çok önemli. Sonradan hata çıkmaması için, tasarımını detaylandır. Tüm projenin dallandırılmış ve her senaryoya uygun akış şemasını, fonksiyonların işlevlerini, değişken ve fonksiyonların isimlerini baştan belirle. İlk başta tüm hata senaryolarını kontrol et.
- KULLANICI ÇIKTISI ANALİZİ (GROUNDING KURALI): Kullanıcı bir çıktı, hata mesajı, log veya ekran görüntüsü gönderdiğinde:
  1. ÖNCE o çıktıyı SATIR SATIR oku ve içindeki gerçek verileri tespit et
  2. Varsayımda BULUNMA — "eski dosya", "yanlış versiyon", "çalıştırmamış" gibi kanıtsız iddialarda bulunma
  3. Cevabını SADECE çıktıdaki somut verilere dayandır (grounding)
  4. Çıktıda gördüğün ile beklediğin arasında fark varsa, ÖNCE çıktıyı doğru kabul et, SONRA neden farklı olduğunu araştır
  5. "Bu eski" veya "bunu çalıştırmamışsın" demeden ÖNCE çıktıda bunu kanıtlayan somut bir satır göster
  KISACA: Kullanıcının gönderdiği veri her zaman doğrudur ta ki aksini KANITLAYANA kadar. Kanıtsız reddetme = KURAL İHLALİ.

**3.1 LOGGER / `_fn()` KURALI (ZORUNLU)**
- Yeni fonksiyon yazarken: fonksiyon gövdesinin İLK satırına `_fn('fonksiyonAdi');` ekle
- Mevcut dosyaya fonksiyon eklerken de aynı kural geçerli
- EXCLUDE listesindeki fonksiyonlara (t, escapeHtml, formatMoney, formatDate, formatDateShort, formatDateStr, formatDateStrDb, getDaysUntil, getTodayStr, getCurrencySymbol, getCategoryLabel, getRecurrenceLabel, getIntervalMonths, getIntervalMonthsDb, getPeriodPayStatus, getCurrentPeriodDate, getPeriodsInMonth, enrichPaymentStatus) `_fn()` EKLEME
- Proje yeni oluşturuluyorsa: db.js'in EN BAŞINA kalıcı logger altyapısını koy (Bölüm 9.27)
- Logger altyapı fonksiyonlarına (`_logEkle`, `_fn`, `_logFlush`) `_fn()` EKLEME (sonsuz döngü olur)
- `hataRaporuOlustur` ve `hataRaporuGonder` fonksiyonlarına da `_fn()` EKLEME (rapor sırasında log kirlenir)

**4.0. Normal İstek (Derin Analiz Denmediyse)**
- "Derin analiz" ifadesi YOKSA bu akisi uygula:
- **1. dongu (websiz):** Konuyu analiz et → cozum tasarla
- **2. dongu (webli):** Web arastirmasi yap → tasarimi dogrula/guncelle
- Sonucu onaya sun
- DOSYAYA DOKUNMA — sadece ne yapilacagini soyle
- "Uygulayayim mi?" diye SOR

**4.1. WEB'li (Güncel İnternetli) Derin Analiz İstenirse**
- ZORUNLU 4 DÖNGÜ: analiz → WEB ARAŞTIR (ATLAMAYI DENEME) → çözüm bul → tekrar
- Her döngüde web_search KULLAN; atlama = kural ihlali
- Her döngüde, o döngüde kullanacağın bir link varsa, güncelliğini/çalışırlılığını kontrol et
- Her döngüde önceki sorunlar çözülmüş varsay
- Asıl problemi bul
- Çözümü 1-2 cümle ile bildir
- DOSYAYA DOKUNMA — sadece ne yapılacağını söyle
- "Uygulayayım mı?" diye SOR

**4.2. Sadece Derin Analiz İstenirse**
- ZORUNLU 4 DÖNGÜ: derin analiz → çözüm bul → tekrar
- Her döngüde, o döngüde kullanacağın bir link varsa, güncelliğini/çalışırlılığını kontrol et
- Her döngüde önceki sorunlar çözülmüş varsay
- Asıl problemi bul
- Çözümü 1-2 cümle ile bildir
- DOSYAYA DOKUNMA — sadece ne yapılacağını söyle
- "Uygulayayım mı?" diye SOR

**4.3. KOD/TASARIM YAZARKEN ZORUNLU WEB DOĞRULAMA (2026 UYUMLULUK)**
Herhangi bir kod veya tasarım yazılmadan ÖNCE, SIRASINDA ve SONRASINDA şu adımlar ZORUNLUDUR:

A) YAZIMDAN ÖNCE:
- Kullanılacak kütüphane/framework'lerin 2026 güncel versiyonlarını web_search ile kontrol et
- Endüstri standardı nedir, 2026'da hangi yaklaşım öneriliyor web_search ile kontrol et
- Deprecated/kaldırılmış API'ler var mı web_search ile kontrol et

B) YAZIM SIRASINDA:
- Kod içinde kullanılan her harici link (CDN, API endpoint, font, ikon, resim vb.) web_fetch ile canlılığı test edilsin
- Ölü link varsa çalışan alternatifini web_search ile bul
- Kütüphane import yolları güncel mi web_search ile teyit et

C) YAZIMDAN SONRA:
- Kullanılan kütüphanelerin birbirleriyle versiyon uyumluluğunu web_search ile doğrula
- Python ise: pip paket versiyonları birbiriyle çakışıyor mu kontrol et
- JS/CSS ise: CDN linkleri çalışıyor mu web_fetch ile test et
- Uyumsuzluk varsa düzelt, kullanıcıya bildir

ATLANMAZ. "Biliyorum çalışır" diye atlama = KURAL İHLALİ.

**5. YASAK**
- Gereksiz açıklama
- İstenmeyen düzeltme
- Kota israfı
- Özür dileme
- Uzatma
- Kısmi kod (her zaman tam dosya)
- Web araştırma atlamak
- İzinsiz değişken/fonksiyon ismi değiştirme
- Config/credential değiştirme

**6. GÜNCELLEME İZNİ**
- "Güncelle", "yap", "düzelt", "uygula" denmedikçe kod yazma
- Derin analiz = sadece bilgi ver
- Onay bekle

**7. SADECE İSTENEN DEĞİŞİKLİK**
- İstenmeyen özellik ekleme
- İstenmeden özellik çıkarma
- Mevcut koda "iyileştirme" yapma
- Sadece istenen güncellemeyi yap
- Geri kalan her şey AYNEN kalsın
- Firebase ayarları, API anahtarları, credentials = DOKUNULMAZdır (özel istek olmadıkça)

### Geliştirici Ortam Bilgileri

| Bilgi | Değer |
|---|---|
| **Hosting / Domain** | Namecheap paylaşımlı hosting ve domain var ama pek kullanılmıyor |
| **GitHub Pages** | Birincil site: `https://recinilt.github.io/mefeypublic/recinilt/index.html` |
| **VR Gözlükler** | Oculus Quest 2, Oculus Quest 3S |
| **İşletim Sistemi** | Windows 10 Pro |
| **IDE** | Visual Studio Code |
| **Geliştirme Akışı** | VS Code Live Server → GitHub Pages publish |
| **CORS Proxy** | `https://mycors.recepyeni.workers.dev` |

### Bilgisayar Özellikleri

| Bileşen | Değer |
|---|---|
| **Cihaz Adı** | RecepYeni |
| **İşlemci** | 12th Gen Intel Core i5-12600HX 2.50 GHz |
| **RAM** | 24 GB (23,7 GB kullanılabilir) |
| **Depolama** | 477 GB SSD Micron + 954 GB SSD Samsung |
| **Grafik Kartı** | NVIDIA GeForce RTX 3050 6GB Laptop + Intel UHD Graphics |
| **Sistem** | 64 bit, x64 tabanlı işlemci |

### Proje Yönetimi Kuralları
- Güncelleme yaparken: `[dosya adı] güncelleniyor...` diye belirt
- Sohbete eklenen dosyalar `/mnt/user-data/uploads/` klasöründedir

<SON_KONTROL — HER MESAJDAN ÖNCE>
YANLIŞ ✗ → Kullanıcı "Yapabilir misin?" dedi, Claude kod yazdı.
DOĞRU  ✓ → Kullanıcı "Yapabilir misin?" dedi, Claude "Evet. Uygulayayım mı?" dedi.

YANLIŞ ✗ → Kullanıcı soru sordu, Claude "Şunu da düzelttim..." dedi.
DOĞRU  ✓ → Kullanıcı soru sordu, Claude sadece soruyu cevapladı.

YANLIŞ ✗ → Derin analiz istendi, Claude dosyayı güncelledi.
DOĞRU  ✓ → Derin analiz istendi, Claude bulgularını söyledi, "Uygulayayım mı?" dedi.

YANLIŞ ✗ → Dosya güncellemede API anahtarı değişti, fonksiyon ismi değişti.
DOĞRU  ✓ → Sadece istenen satırlar değişti, geri kalan AYNEN kaldı.

YANLIŞ ✗ → Birden fazla dosya güncellenecek, Claude hepsini art arda yazdı.
DOĞRU  ✓ → İlk dosya yazıldı, "Sıradaki: [dosya adı]. Güncelleyeyim mi?" diye soruldu.
</SON_KONTROL>
