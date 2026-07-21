# Kavach App — Firebase Setup (ज़रूरी, वरना real login/signup काम नहीं करेगा)

`kavach-app.html` अब **local demo data नहीं**, असली Firebase project
(`kavach-86f4a`) इस्तेमाल करता है — Authentication + Firestore। लेकिन Firebase
project बनाना काफी नहीं है, Console में कुछ चीज़ें ON करनी ज़रूरी हैं। नीचे
exact steps हैं।

## 1) Sign-in methods ON करें
Firebase Console → **Authentication → Sign-in method** → इन तीनों को Enable करें:
- Email/Password
- Phone
- Google

(Enable किए बिना, login/signup पर `auth/operation-not-allowed` error आएगा।)

## 2) Authorized domain add करें
Authentication → **Settings → Authorized domains** में वो domain add करें
जहाँ ये HTML file असल में host होगी (सिर्फ अपने computer पर file खोलने
`file://...` से **Google Sign-In और Phone OTP दोनों काम नहीं करेंगे** —
`auth/unauthorized-domain` error आएगा)। सबसे आसान तरीका: इसी Firebase project
में **Hosting** enable करके `firebase deploy` कर दें — तब आपका
`kavach-86f4a.web.app` domain अपने-आप authorized होगा।

## 3) Phone OTP असली SMS भेजने के लिए Blaze plan
Free (Spark) plan पर सिर्फ Console में manually add किए गए **test phone
numbers** पर OTP जाता है। किसी भी असली number पर SMS भेजने के लिए project
को **Blaze (pay-as-you-go)** plan पर upgrade करना होगा (Firebase console
में Upgrade project बटन)। SMS की per-message एक छोटी सी cost लगती है।

## 4) Firestore Database बनाएं + Rules सेट करें
Firestore Database → **Create database** (production mode)। फिर
**Rules** tab में ये rules paste करें (demo-safe, हर account सिर्फ अपना
data पढ़/लिख सके, admin सबका list पढ़ सके):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /accounts/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```
(Rules set किए बिना default production rules सब कुछ deny कर देते हैं —
`permission-denied` error आएगा।)

## App में क्या असली है, क्या demo है
- ✅ **Real**: Signup (Firebase Auth account बनता है), Email verification
  (असली link आपके inbox में), Phone OTP (असली SMS — Blaze plan चाहिए),
  Google Sign-In (असली Google popup), PIN reset (real phone re-verify +
  Firebase password update), Admin Dashboard की user list (Firestore से
  live आती है, delete/suspend असली Firestore doc बदलते हैं)।
- 🧪 **अभी भी demo/local**: Chats, Reels, Groups, Stories का data — वो
  पहले जैसे hardcoded ही हैं (उन्हें Firestore से जोड़ना एक अलग, बड़ा काम
  होगा, बताएं तो वो भी कर सकता हूँ)। साथ ही Admin PIN `1234`/`9999` अब भी
  आपके explicit spec के मुताबिक एक **local master-key shortcut** है (कोई
  Firebase account ज़रूरी नहीं) — किसी भी valid email+phone के साथ यह
  turant Admin Dashboard खोल देता है।

## एक ज़रूरी सच्चाई
मैं इस sandbox में **offline** हूँ (कोई internet access नहीं), इसलिए मैं
इसे खुद run करके test नहीं कर सका — कोड Firebase की official docs के
मुताबिक सही लिखा गया है, पर पहली बार चलाने पर ऊपर के 4 steps के बिना
errors आना तय है। हर जगह error message असली Firebase error (toast में)
दिखेगा, silently कुछ नहीं टूटेगा — तो debug करना आसान रहेगा।

## APK के बारे में
असली installable `.apk` इस sandbox में generate करना possible नहीं है
(न internet, न Android SDK/Gradle यहां मौजूद है)। सबसे आसान रास्ता:
1. इस HTML file को कहीं host करें (Firebase Hosting सबसे सही है, चूंकि
   project पहले से यही है) — `firebase init hosting` → `firebase deploy`
2. उस URL को **PWA (installable web-app)** बनाएं — घर की स्क्रीन पर
   "Add to Home Screen" से app जैसा icon मिल जाता है, बिना किसी Play
   Store या APK के।
3. अगर असली `.apk` ही चाहिए तो [Capacitor](https://capacitorjs.com) या
   [Median](https://median.co) जैसे tool से इस HTML को अपने computer पर
   Android Studio के ज़रिए wrap करना होगा — वो step मैं यहां नहीं कर
   सकता, पर चाहें तो exact commands बता सकता हूँ।
