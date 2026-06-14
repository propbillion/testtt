# PropBillion Attendance — Android App banane ka recipe

Yeh app `index.html` ko Capacitor se Android app mein wrap karti hai.
Tabhi **fake-GPS block** aur **shift-ke-dauran live tracking** ON hote hain.
(Browser mein wahi file web-mode pe chalti hai — anti-spoof OFF, honestly.)

Tumhare MacBook Pro M5 pe ye one-time ~1.5 ghante ka setup hai.

---

## Pehle ye install karo (one-time)
1. **Node.js** — https://nodejs.org (LTS)
2. **Android Studio** — https://developer.android.com/studio
   (install ke waqt "Android SDK" + "SDK Platform" + "Build-Tools" select rakho)
3. **Java JDK 17** — Android Studio ke saath aata hai, alag se nahi chahiye.

---

## Project banao (terminal)

```bash
mkdir propbillion-app && cd propbillion-app
npm init -y
npm install @capacitor/core @capacitor/cli
npx cap init "PropBillion Attendance" "com.propbillion.attendance" --web-dir=www

mkdir www
# apni index.html ko www/ ke andar daalo (naam index.html hi rahe)
cp /path/to/index.html www/index.html

npm install @capacitor/android
npx cap add android

# dono problem solve karne wala plugin:
npm install @capacitor-community/background-geolocation
npx cap sync
```

---

## Android permissions add karo

File: `android/app/src/main/AndroidManifest.xml`
`<application>` tag ke **upar** ye lines daalo:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<uses-permission android:name="android.permission.CAMERA" />
```

File: `android/app/src/main/res/values/strings.xml`
`</resources>` ke upar:

```xml
<string name="capacitor_background_geolocation_notification_channel_name">PropBillion — on shift</string>
```

Phir:
```bash
npx cap sync
npx cap open android
```

---

## APK build karo (Android Studio)
1. Android Studio khulega. Upar menu: **Build → Build App Bundle(s) / APK(s) → Build APK(s)**.
2. Build hone ke baad "locate" click karo → `app-debug.apk` mil jaayega.
3. Yeh APK employees ko WhatsApp/Drive se bhejo, woh install kar lein
   (Settings mein "Install unknown apps" allow karna padega — ek baar).

> Play Store pe daalna ho to "Signed Bundle (.aab)" banao — woh baad mein.

---

## Important: employee ko ye permissions deni hongi
- Location → **"Allow all the time"** (sirf "while using" se background tracking nahi chalega)
- Camera → Allow
- Notification → Allow (shift ke time ek persistent notification dikhegi — yahi background ko zinda rakhti hai)
- **Developer Options → "Select mock location app" khaali hona chahiye.**
  Agar koi fake-GPS app set hai, punch-in **block** ho jaayega aur admin ko `⚑ FAKE GPS` dikhega.

---

## Kya ab clear ho gaya
| Problem | Web (GitHub) | Android App (yeh) |
|---|---|---|
| Fake/mock GPS block | ❌ deterrent only | ✅ punch-in hard block + admin flag |
| Background live location | ❌ tab band = stop | ✅ punch-in → punch-out tak chalu |
| Selfie + geo at punch | ✅ | ✅ |
| Late/half-day/7PM/break rules | ✅ | ✅ |

Tracking sirf **shift ke dauraan** chalti hai (punch-in se punch-out), 24/7 nahi —
yeh consent ke hisaab se safe hai. Employment letter mein ek line add kar do:
*"Work location is tracked during active shift hours via the company attendance app."*

---

## Firebase (cross-device admin ke liye — zaroori)
Bina Firebase ke admin sirf apne device ka data dekhega. `index.html` ke top
`CONFIG.FIREBASE` block mein apne Firebase web-app keys paste karo — phir
sabki live location ek hi admin dashboard pe aa jaayegi.
Steps chahiye to bolo, step-by-step de dunga.
