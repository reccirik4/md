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
├── voltbuilder.json            ← VoltBuilder release build (opsiyonel)
├── res/                        ← İkon dosyaları (config.xml ile referanslanır)
│   └── icon.png                ← Uygulama ikonu (en az 512x512, ideal 1024x1024)
├── resources/                  ← VoltBuilder otomatik ikon/splash üretimi
│   ├── iconTemplate.png        ← 1024x1024 (VoltBuilder tüm boyutlara çevirir)
│   └── splashTemplate.png      ← 2732x2732 (VoltBuilder splash üretir, opsiyonel)
├── certificates/               ← Release build sertifikaları
│   └── android.keystore
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
<widget id="com.sirket.uygulamaadi" version="1.0.0"
  xmlns="http://www.w3.org/ns/widgets"
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:cdv="http://cordova.apache.org/ns/1.0">

  <name>Uygulama Adı</name>
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
  <preference name="AndroidWindowSplashScreenAnimatedIcon" value="res/screen/android/splashscreen.png" />
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
  <allow-navigation href="https://accounts.google.com/*" />

  <!-- === PLUGİNLER === -->
  <plugin name="cordova-plugin-device" spec="*" />
  <plugin name="cordova-plugin-statusbar" spec="*" />
  <plugin name="cordova-plugin-file" spec="*" />

  <!-- === MOTOR (ZORUNLU — eksik olursa uyumsuz platform yüklenebilir) === -->
  <engine name="android" spec="14.0.1" />

  <!-- === ANDROİD PLATFORM === -->
  <platform name="android">
    <!-- admob-plus-cordova kullanıyorsan bu iki satır ZORUNLU -->
    <preference name="GradlePluginKotlinEnabled" value="true" />
    <preference name="AndroidXEnabled" value="true" />
    <config-file target="AndroidManifest.xml" parent="/manifest">
      <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
      <uses-permission android:name="android.permission.VIBRATE" />
    </config-file>
  </platform>
</widget>
```

---

## 4. index.html Şablonu

```html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="format-detection" content="telephone=no">
  <meta name="msapplication-tap-highlight" content="no">
  <title>Uygulama Adı</title>
  <!-- Fontlar: LOKAL yükle, CDN KULLANMA -->
  <link rel="stylesheet" href="css/app.css">
</head>
<body>
  <!-- İçerik -->

  <!-- cordova.js: projede fiziksel olarak bulunmaz, build sırasında Cordova otomatik ekler -->
  <script src="cordova.js"></script>

  <!-- Firebase SDK: LOKAL yükle (CDN'den DEĞİL — çökme sebebi!) -->
  <script src="lib/firebase-app-compat.js"></script>
  <script src="lib/firebase-auth-compat.js"></script>
  <script src="lib/firebase-database-compat.js"></script>

  <script src="js/app.js"></script>
</body>
</html>
```

> ⚠️ **CDN KULLANMA!** `<script src="https://www.gstatic.com/firebasejs/...">` gibi harici CDN linkleri Cordova WebView'da başarısız olabilir. İnternet yokken veya CDN yavaşken script yüklenmez → `firebase is not defined` → uygulama çöker. Firebase SDK'yı `www/lib/` altına lokal olarak indir.

---

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
**Sebep:** cordova-android 14 varsayılan appcompat (1.7.0) ve 1.6.1, AGP 8.7.3'ün AAPT2 derleyicisiyle uyumsuz. cordova-android master'da artık 1.7.1 — gelecek sürümlerde de dikkat edilmeli.
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
| **Format** | AAB tercih edilir |
| **İmzalama** | Keystore ile imzalı |
| **Hesap** | https://play.google.com/console/signup ($25) |

### Yükleme Adımları
1. Play Console → "Uygulama oluştur"
2. "Üretim" → "Yeni sürüm" → .aab yükle
3. Uygulama bilgileri, ekran görüntüleri, gizlilik politikası
4. İçerik derecelendirmesi + fiyatlandırma
5. Yayınla

---

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
- [ ] PowerShell scriptleri ASCII-only (Türkçe karakter yok!)
- [ ] `cordova build android` → BUILD SUCCESSFUL

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
