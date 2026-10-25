# Metin ve Çeviri Kuralları (Backend)

> [!info] Belge bilgisi
> **Amaç:** `oksis-api`'de kullanıcıya giden metnin (hata, bildirim, referans verisi adı) nasıl yazıldığını ve iki dile (`tr` + `en`) nasıl hazırlandığını tanımlamak.
> **Son doğrulama:** 2026-10-25. `oksis-api` dev `3d889c70`.
> **Kaynak:** ADR [[0029-cok-dil-altyapisi-ingilizce]], `src/Oksis.Infrastructure/Localization/`, `src/Oksis.Api/Localization/`, `src/Oksis.Api/Errors/ErrorMessageCatalog.cs`, `src/Oksis.Application/Common/Localization/`, `src/Oksis.Application/Modules/Notifications/Abstractions/`, `src/Oksis.Infrastructure/Persistence/Seed/MasterData/ReferenceKeys.cs`.

İlgili notlar: [[logging-error-rules]] §8.5 · [[i18n-terim-sozlugu]] · frontend karşılığı [[teknik/kurallar/frontend/metin-ve-ceviri|metin-ve-ceviri (frontend)]]

---

## 1. Temel kural

**Sistem metni anahtarla yazılır.** Kullanıcıya giden yeni bir metin kodda Türkçe cümle olarak değil, bir kaynak anahtarı olarak yazılır. Türkçe karşılığı `tr` dosyasına, İngilizcesi `en` dosyasına girer. Kullanıcının yazdığı içerik (duyuru metni, not, ad) çevrilmez.

- Göçü bitmemiş modüllerdeki mevcut Türkçe metin, o modülün göçüne kadar kalır. Dokunulan yeni hata, bildirim ya da ad ise anahtarla yazılır.
- `en` eksikse metin `tr`'ye düşer; anahtar hiçbir zaman kullanıcıya basılmaz.
- Terimler [[i18n-terim-sozlugu]]'ne uyar (en-US). Dersler, branşlar, yabancı dil adları, eğitim programları ve il/ilçe/mahalle çevrilmez.

## 2. Kaynak dosyaları

- Yer: `src/Oksis.Infrastructure/Localization/Resources/<ad-alanı>/{tr,en}.json`, derlemeye gömülü (`EmbeddedResource`).
- İçerik düz `anahtar → metin` nesnesidir (anahtarlar noktalı, iç içe değil).
- Ad alanları: `errors`, `notifications`, `common`, `reference`, `validation`, `email`, `excel`. Bugün `validation`, `email`, `excel` boştur; modül göçünde dolar.
- Okuma `IStringLocalizerFactory` (`JsonStringLocalizerFactory`) ile: `Create("<ad-alanı>", string.Empty)`. `IStringLocalizer<T>` kayıtlı değildir, kullanılmaz.
- Düşüş zinciri: etkin UI dili → `tr` → nötr cümle (`common`) + uyarı günlüğü. Bozuk çeviri kalıbı (`{` hatası) bir sonraki dile düşer; günlüğe yalnız anahtar, ad alanı ve dil yazılır, parametre yazılmaz.
- Parametreler `string.Format` sırasıyla (`{0}`, `{1}`) yazılır. `en` ile `tr`'nin yer tutucuları aynı olmalı (bekçi testi var).

## 3. Kültür

- İstekte **yalnız `CurrentUICulture`** değişir (`UiCultureMiddleware`). `CurrentCulture` her zaman `tr-TR` kalır: arama (`ToLower()` + LIKE), sıralama ve sayı/tarih biçimi Türkçe kurala bağlıdır (TB-271).
- Etkin dil sırası: hesabın tercihi (`Account.PreferredLanguage`) → okulun varsayılanı (`SchoolSettings.DefaultLanguage`) → `Accept-Language` → `tr`. Yalnız `tr` | `en`.
- Açık kültürle metin üretmek için `UiCultureScope` (Application) kullanılır. Kapsam eşzamanlı kapanır; içinde `await` olmaz.
- Arka plan işleri istek kültürünü kullanmaz; `tr-TR` koşar. Metin, alıcının diliyle açıkça üretilir.

## 4. Hata mesajları

- `Error(Code, Message)`'da `Message` bir anahtardır: `<modül>.errors.<kebab-anahtar>`. API sınırında `ErrorMessageCatalog.Humanize` onu `errors` ad alanından istek dilinde cümleye çevirir.
- Yeni anahtar aynı commit'te `errors/tr.json`'a eklenir. Bekçi testi (`ErrorMessageCatalogTests`) kodda kullanılan her anahtarın `tr` karşılığını arar.
- `code` dile göre değişmez; istemci mantığı koda bağlanır.

## 5. Bildirimler

- Yeni bildirim `NotificationMessage.FromKeys(titleKey, bodyKey, args)` ile kurulur ve `INotificationEnqueuer.EnqueueLocalized` ile kuyruğa girer. Metin, dağıtım anında her alıcının kendi dilinde üretilir. Alıcılar dile göre gruplanır.
- Anahtarlar `notifications/tr.json`'da bulunur (kapsam bekçisi var). Eksik anahtar bildirime uygun nötr metne düşer.
- **Kuyruk yüküne kişisel veri girmez.** `args` Hangfire deposunda düz metin durur. Ad, telefon ya da TC kimlik no taşınmaz; kimlik taşınır, ad işte çözülür.
- Uygulama içi bildirim metni üretildiği dilde kalır. Kullanıcı sonradan dil değiştirirse eski bildirimler eski dilde görünür (kabul edildi).
- Hazır metinli eski biçim (`NotificationMessage(Kind, Title, Body, …)`) göç bitene kadar çalışır.

## 6. Referans verisi adları

- Çevrilen çekirdek tablolar anahtar kolonu taşır: `NameKey`, `DescriptionKey`, `ConsentBundle.TitleKey`. Liste: GradeLevel, MasterExamType + okul ExamType, OfficialHoliday, DutyLocationTemplate + DutyLocation, SystemRole, Permission (yalnız açıklama), NotificationEventType, AcademicTermType, GradeScale, ConsentBundle (yalnız başlık; içerik Türkçe), Country.
- Anahtar biçimi `reference.<tür>.<kalıcı-kod>`, açıklama için sonuna `.description`. Üretim tek yerde: `ReferenceKeys`. Değer `reference/tr.json`'da satırın `Name`'iyle birebir aynıdır (bekçi testi var).
- Okula kopyalanan satır anahtarı da kopyalar. Okulun eklediği satır anahtarsızdır. Ad (ya da açıklama) değişince anahtar boşalır; ad sonra şablon adına geri dönse de anahtar geri gelmez.
- Ad sunucuda çözülür: API `name` alanını etkin dilde döndürür, sözleşme değişmez. Çözüm `IReferenceNameResolver` ile yapılır: anahtar doluysa çeviri, yoksa `tr`, o da yoksa satırın `Name`'i.
- **Önbellekli sorguya çözümleyici bağlanmaz.** `[Cacheable]` anahtarı dil içermiyorsa çözülmüş ad başka dildeki kullanıcıya sızar. Önce önbellek anahtarı ve geçersizleştirme birlikte dile duyarlı yapılır (mimari bekçi bunu sınar). Bugün bağlı uçlar: resmî tatil aralığı, sezon tatilleri, `HolidayCalendarReader`.

## 7. Testler

- Test derlemeleri süreç kültürünü kodda `tr-TR`'ye sabitler (`TestProcessCulture`). Türkçe metin karşılaştıran test makinenin dil ayarına bağlı değildir.
- Kültürü bilerek değiştiren test bunu kendi kapsamında geri koyar (`ProcessCultureCollection`).

## 8. Çeviri hattı

- `tr` dosyalarının sahibi koddur. `en` dosyalarını Claude ya da insan çevirmen doldurur; çeviriler Tolgee'de (`ceviri.oksis.net`, ücretsiz sürüm) yönetilir.
- API kaynaklarının Tolgee eşitlemesi henüz kurulmadı; bugün yalnız `oksis-ui` mesajları eşitlenir (`.github/workflows/ceviri.yml`).
