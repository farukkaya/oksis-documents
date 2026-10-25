# Metin ve Çeviri Kuralları (Web + Mobil)

> [!info] Belge bilgisi
> **Amaç:** `oksis-ui`'de arayüz metninin nasıl yazıldığını ve iki dile (`tr` + `en`) nasıl hazırlandığını tanımlamak.
> **Son doğrulama:** 2026-10-25. `oksis-ui` dev `9a442dac`.
> **Kaynak:** ADR [[0029-cok-dil-altyapisi-ingilizce]], `packages/i18n/` (README, `src/`, `messages/`, `tools/i18n-cli.mjs`), `packages/eslint-config/i18n.js`, `apps/web/eslint.config.js`, `apps/mobile/eslint.config.mjs`, `apps/mobile/src/lib/i18n/`, `.github/workflows/ceviri.yml`, `.tolgeerc.json`.

İlgili notlar: [[bilesen-ve-stil-kurallari]] §8 · [[mobil]] · [[i18n-terim-sozlugu]] · backend karşılığı [[teknik/kurallar/backend/metin-ve-ceviri|metin-ve-ceviri (backend)]]

---

## 1. Temel kural

**Sistem metni anahtarla yazılır.** Yeni ekran, yeni bileşen ve göçü biten modül metni JSX'e sabit Türkçe olarak yazmaz; metin mesaj dosyasından gelir. Kullanıcının yazdığı içerik ve sunucudan gelen veri (ad, duyuru metni) çevrilmez.

- Göçü bitmemiş ekranlar bugün Türkçe sabit metinle çalışır; modül göçüne kadar kalır.
- Terimler [[i18n-terim-sozlugu]]'ne uyar (en-US). Dersler, branşlar, yabancı dil adları, eğitim programları ve il/ilçe/mahalle çevrilmez.
- Dil seçeneklerinin etiketi kendi dilinde yazılır ("Türkçe", "English").

## 2. Mesaj dosyaları

- Yer: `packages/i18n/messages/<ad-alanı>/{tr,en}.json`. Ad alanı = modül.
- Anahtar biçimi `<ad-alanı>.<ekran>.<öğe>`. Anahtarlar tiplidir (`MessageKey`); olmayan anahtar typecheck'i kırar.
- **Yeni ad alanı** üç yere birer satır ister: `src/namespaces.ts`, `src/messages-tr.ts`, `src/messages-en.ts`. `messages/` altında listede olmayan klasör bekçi testini kırar.
- `tr` gömülü gelir ve tipin kaynağıdır. `en` ayrı parça olarak yalnız gerektiğinde yüklenir; eksik ya da boş `en` metni `tr`'ye düşer. Ham anahtar ekranda görünmez.
- Söz dizimi **ICU**'dur, web ve mobilde ortak: `{ad}`, `{sayi, plural, one {…} other {…}}`. Mobilde i18next'in `{{ad}}` biçimi kullanılmaz (bekçi testi var).
- Bekçi testi (`messages-guard.test.ts`) bütün ad alanlarında şunları sınar: `tr` metni boş değil, `en` anahtarı `tr`'de var, yer tutucular iki dilde aynı, `en` ile yüklenen her metin dolu. Ayrıca: `tr` metinlerinde ağ suçlama kalıbı (X-08, ESLint `base.js` ile aynı kalıp) yalnız izinli anahtarlarda bulunur; onay içeriği anahtarları `en`'de yoktur; bölünerek çizilen `{link}` cümleleri tam bir `{link}` içerir.

## 3. Kullanım

- **Web:** `next-intl`, `useTranslations("<ad-alanı>")`. Dil istemcide çözülür (`oksis_locale` çerezi + `<html lang>`); metin doğru dil yüklenene kadar gizlenir. Barındırma kararı üretime ertelendi; kod sunucu tarafı çözüme geçebilecek biçimde yazıldı.
- **Mobil:** `i18next` + `react-i18next` + `i18next-icu`, `useTranslation("<ad-alanı>")`. Dil sırası: kalıcı tercih (`oksis-locale`, SecureStore) → cihaz dili (`expo-localization`) → `tr`. Hermes'te eksik olan `Intl.PluralRules` dolgusu yalnız eksikken yüklenir.
- Oturum açınca etkin dil `me/context.effectiveLanguage`'dan alınır; sunucunun dili kazanır. Dil değişince TanStack Query önbelleği geçersizlenir (sunucuda çözülen adlar yenilensin).
- API istekleri `Accept-Language` başlığını taşır (`packages/api` `getLanguage`).
- Tarih/sayı biçimi için `packages/i18n` biçimleyicileri kullanılır. Elle `toLocale*` çağrısında dil açıkça verilir (`toLocaleUpperCase(locale)` dahil).
- Sabit sayı metne yazılmaz; yer tutucuyla koddaki sabitten beslenir (`{min}` ← `RESET_LINK_VALID_MINUTES`, `PASSWORD_MIN_LENGTH`). `en`'de sayıya bağlı sözcük ICU `plural` ile yazılır.

### 3.1 Doğrulama iletileri (zod)

- `packages/core` şemaları Türkçe cümle değil `MessageKey` döndürür (ör. `auth/validation-messages.ts`: `AUTH_VALIDATION_MESSAGES`). Ekran iletiyi çevirerek gösterir: web `useValidationText()`, mobil `features/auth/validation-text.ts`. Çeviri `translateValidationMessage(message, t)` ile yapılır; tanınmayan ileti olduğu gibi geçer.
- `packages/core` → `@workspace/i18n` bağımlılığı yalnız tip (`MessageKey`) içindir.
- Aynı hata türünden metin seçen mantık (ör. parola değişimi kilit/hız sınırı) core'da tek saf yardımcıdır (`changePasswordFailureMessage` → `{ key, values }`); web ve mobil yalnız `t` çağırır.

### 3.2 Onay (KVKK) metinleri

- Onay **içeriği** hukuki metindir ve her dilde Türkçe gösterilir; yalnız başlık çevrilir. Bu anahtarlar `en.json`'da bulunmaz ve `consentStatement(key)` (`packages/i18n/src/consent.ts`) ile gömülü `tr`'den okunur. Bekçi testi `en`'de bu anahtarı görürse kırmızı olur.

### 3.3 Mobil anahtarlar

- Mobilde i18next anahtarları tipsizdir: yanlış anahtar derlemeyi kırmaz, ekranda ham anahtar görünür. Yeni anahtar `tr.json`'a eklenmeden kullanılmaz; dinamik anahtarlar (`portal.${rol}` gibi) bütün değerleriyle dosyada bulunur.
- Bugünkü mobil metin web'den farklıysa davranışı korumak için `<ad-alanı>.mobile.*` altında ayrı anahtar açılır. Metinlerin yakınsatılması ayrı bir karardır.

## 4. Bekçiler

- ESLint `react/jsx-no-literals` (`packages/eslint-config/i18n.js`) **dizin listesiyle** açılır: web `apps/web/eslint.config.js`, mobil `apps/mobile/eslint.config.mjs` içindeki `I18N_MIGRATED`. Göçü biten modülün klasörü listeye eklenir.
- Dil seçici ve okul varsayılan dili ayarı **bayrak** arkasındadır: `NEXT_PUBLIC_FEATURE_LANGUAGE_SWITCH`, `EXPO_PUBLIC_FEATURE_LANGUAGE_SWITCH` (yalnız `"enabled"` açar). Dev ve test dağıtımında açık, üretimde ilk aile yüzeyi göçü bitene kadar kapalı.

## 5. Çeviri hattı (Tolgee)

- `tr` dosyalarının sahibi koddur; Tolgee `tr`'ye yazmaz.
- `dev`'e `tr`/`en` değişikliği gelince `ceviri.yml`: `tr` yüklenir (OVERRIDE), koddaki `en` yalnız Tolgee'de boş olanları doldurur (KEEP), anahtar açıklamalarına sözlük terimleri yazılır.
- Haftalık (pazartesi) ya da elle: Tolgee'den `en` çekilir, `tr` sırasına dizilir, sözlük denetimi uyarı olarak eklenir ve `dev`'e PR açılır. "Actions'ın PR açma izni" kapalıysa dal gönderilir, PR elle açılır.
- Tolgee `ceviri.oksis.net`'tedir (Cloudflare Access arkasında, ücretsiz sürüm). Tolgee kapalıyken derleme ve testler etkilenmez.

## 6. Modül göçü sırası (öneri)

ADR önceliği: veli/öğrenci → öğretmen → yönetim. Birim "modül × yüzey"dir; modülün API hata, doğrulama ve bildirim metinleri o modülün ilk yüzeyiyle birlikte taşınır.

1. **Ortak kabuk ve kimlik:** giriş, parola değiştirme/sıfırlama, davet kabulü ve e-posta şablonları bitti (2026-10-25). Kalan: hata yüzeyi, gezinme, bildirim zili, profil.
2. **Aile yüzeyleri (mobil önce):** duyurular, yemek menüsü, ders programı, devamsızlık, notlar ve sınav takvimi, ödev, kulüpler, veli toplantısı, etkinlikler.
3. **Öğretmen yüzeyleri:** yoklama, not girişi, ödev yönetimi, kulüp danışmanlığı, nöbet, veli toplantısı.
4. **Yönetim yüzeyleri (web):** okul ayarları, kullanıcılar/roller/izinler, müfredat, ders atama, sezon sihirbazı, raporlar ve Excel.
5. **Platform (merkez) yüzeyi:** v1'de dil desteği yok; yalnız `Accept-Language`/çerez, metinler `tr`.

Bilinen göç notu: düğme metni "Vazgeç" → *Cancel*, durum "İptal" → *Canceled* (sözlük). `common:actions.cancel` bugün "İptal"; göçte "Vazgeç" olmalı.
