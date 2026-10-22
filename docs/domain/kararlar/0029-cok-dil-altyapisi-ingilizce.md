---
tags: [decision, teknik/i18n]
date: 2026-10-22
status: accepted
---

# 0029 — Çok dil altyapısı: ilk ek dil İngilizce; çeviri yönetimi ayrı, çalışma anı gömülü

## Bağlam

Bugün i18n yok: arayüz metinleri doğrudan Türkçe yazılır ([[tech-stack]] §247, [[bilesen-ve-stil-kurallari]] §92), [[urun-tanimi]] çoklu dili kapsam dışı sayar. Faruk'un 2026-10-22 kararı:

- İlk ek dil **yalnız İngilizce** (`tr` varsayılan, `en` ek). RTL yok.
- **Kullanıcının girdiği içerik hariç her şey** çevrilir.
- Çok dil "ayrı servis olarak yönetilmek" isteniyor.

Ölçülen kapsam (2026-10-22, Türkçe karakter içeren yorum dışı satırlar; kaba ölçü, alt sınır değil):

| Yer | Dosya | Satır |
|---|---|---|
| oksis-ui web + `packages/ui` + `packages/core` | 878 | ~13 900 |
| oksis-ui mobile | 415 | ~3 200 |
| oksis-api (`.cs` string literal) | 1 464 | ~8 200 |

API'de yoğunluk: Clubs, Academics, Exams, Attendance, Homework, Timetable. Türler: 100 validator, 11 `*Errors.cs`, 86 bildirim içerik sınıfı, `EmailTemplates.cs` (davet, şifre sıfırlama), ClosedXML Excel çıktıları, 22 seed dosyası (referans veri). API'de `RequestLocalization` kurulu değil; kullanıcıda dil alanı yok.

## "Çevrilir" ile "çevrilmez" sınırı

| Çevrilir (sistem metni) | Çevrilmez (kullanıcı içeriği) |
|---|---|
| Arayüz metinleri, boş durum, onay pencereleri | Duyuru, ödev, kulüp duyurusu metni |
| Doğrulama ve hata mesajları | Okul/şube/kişi adları, açıklamalar, notlar |
| Bildirim (push, e-posta) şablonları | Duyuru şablonları (okul yazar) |
| Excel/PDF başlıkları, sütun adları | Kullanıcının yüklediği belgeler |
| Enum etiketleri (devamsızlık türü, kulüp kategorisi…) | |
| **Referans (katalog) verisi:** sınıf seviyeleri, sınav türleri, resmî tatiller, nöbet yeri şablonları, yabancı dil adları, roller, izinler, bildirim türleri, sistem ayarı etiketleri | Okulun kendisinin eklediği ya da adını değiştirdiği kayıtlar |
| Tarih, sayı, saat biçimleri | **Ders adları** ve ders kataloğu (özel isim gibi ele alınır: "Matematik" İngilizce arayüzde de "Matematik") |
| | İl, ilçe, mahalle, okul adları (özel isim) |

Referans verisi DB'de durur; çevirisi **anahtar kolonuyla** çözülür (aşağıda).

## Seçenekler

- **A — Çalışma anında ayrı i18n servisi (API):** web, mobil ve API metinleri ağdan çeker. Her ekran ağa bağlanır, servis düşünce arayüz anahtar gösterir, mobilde paket içi metin kaybolur, modüler monolite ikinci dağıtım birimi eklenir. Reddedildi.
- **B — Yalnız depo içi dosyalar:** çeviriler JSON/`.resx` olarak kod depolarında. Ucuz, ama çeviriyi geliştirici dışında kimse düzenleyemez; "ayrı yönetim" isteği karşılanmaz.
- **C — Ayrı yönetim, gömülü çalışma anı (seçildi):** çeviriler kendi sunucumuzda bir çeviri yönetim aracında (Tolgee, self-hosted) düzenlenir. CI bunları JSON olarak dışa aktarır, oksis-ui ve oksis-api'ye derleme anında girer. Çalışma anında ağ çağrısı yoktur.

## Karar

**C kabul edildi** (Faruk, 2026-10-23).

### Dil seçimi
- **Okul varsayılan dili v1'dedir** (Faruk, 2026-10-22): `SchoolSettings.DefaultLanguage` (`tr` | `en`, varsayılan `tr`). Okul yöneticisi ayarlardan değiştirir.
- `UserAccount.PreferredLanguage` (`tr` | `en` | boş) eklenir. Boşsa okul varsayılanı geçerlidir. Kullanıcı profilden değiştirir.
- API: `RequestLocalization` kurulur. Kültür sırası: kullanıcının tercihi → okulun varsayılanı → `Accept-Language` (oturum yokken, örn. giriş ekranı) → `tr`. Kültür yalnız istek kapsamında; `DefaultThreadCurrentCulture` değiştirilmez.
- Arka plan işleri (Hangfire) ve bildirimler istek kültürü **kullanmaz**; metni **alıcının** etkin diline (tercih ?? okul varsayılanı) göre üretir.
- Okul varsayılanı değişince kullanıcıların açık tercihi korunur; yalnız tercihi boş olanlar etkilenir.

### oksis-ui
- Yeni `packages/i18n`: `tr.json`, `en.json` (ad alanı = modül), tipli anahtar üretimi, ortak tarih/sayı biçimleyicileri (`Intl`).
- Web: `next-intl` (App Router, URL'de dil öneki **yok**; dil kullanıcı tercihinden ve çerezden). Mobil: `i18next` + `react-i18next` + `expo-localization`.
- Anahtar biçimi: `<modül>.<ekran>.<öğe>` (örn. `attendance.roster.saveButton`).
- ESLint kuralı: JSX'te sabit metin yasak (göç biten modülde açılır).

### oksis-api
- Hatalar: `Error(Code, MessageKey)` kalıbı bütün modüllere yayılır. API `Code` + çevrilmiş `message` döner; istemci isterse `Code`'a göre kendi metnini gösterir.
- FluentValidation: `WithMessage` sabit metin yerine anahtar; `ValidatorOptions.Global.LanguageManager` kültüre bağlanır.
- Bildirim içerik sınıfları ve `EmailTemplates`: `IStringLocalizer` ile alıcı kültüründe üretilir.
- Excel çıktıları: başlıklar anahtar üzerinden.
- Enum etiketleri: API kod döner, etiketi istemci çevirir (`ClubLabels` gibi sunucu etiketleri kaldırılır).
- **Referans verisi: anahtar kolonu, tek yöntem** (Faruk, 2026-10-22; karma ve çeviri tablosu reddedildi).
  - Çeviri gereken her katalog tablosuna `NameKey` (gerekirse `DescriptionKey`) eklenir; değeri arayüz anahtarlarıyla aynı dosyalarda çevrilir. Türkçe `Name` kolonu kalır (Türkçe kaynak metin ve geriye uyum).
  - Gösterim: `NameKey` doluysa etkin dilde çeviri, boşsa `Name` olduğu gibi.
  - Okula kopyalanan kayıtlar (örn. sınav türü) `NameKey`'i taşır. Okul adı değiştirirse `NameKey` boşaltılır; kayıt artık okulun içeriğidir, çevrilmez. Okulun sıfırdan eklediği kayıtta `NameKey` hiç olmaz.
  - **Ders kataloğu ve dersler `NameKey` almaz.** `SubjectBranch` (branş adları) da derslerle aynı gerekçeyle çevrilmez — derslerle aynı ad uzayını paylaşıyor.
  - Bedel (kabul edildi): platforma yeni referans kaydı eklemek (örn. yeni resmî tatil) kod değişikliği ve dağıtım ister, çünkü anahtarın çevirisi dosyadadır.

### Çeviri yönetimi (ayrı birim)
- **Tolgee, self-hosted, ücretsiz sürüm** (Faruk, 2026-10-23: lisans ücreti ödenmez). oksis.net altında, Docker + PostgreSQL. Kaynak dil `tr`, hedef `en`.
- Ücretsiz sürüm sınırları bağlayıcıdır: **en çok 10 kullanıcı**; Enterprise (`ee/`) özellikleri (SSO, ayrıntılı yetki vb.) kullanılmaz ve bunlara bağımlı akış tasarlanmaz. Sınır aşılacaksa önce Faruk'a sorulur (alternatif: Weblate, GPL, sınırsız).
- Güvenlik: `TOLGEE_AUTHENTICATION_ENABLED` açık olmadan internete açılmaz; yönetici hesabı ve API anahtarları gizli değişkenlerde durur.
- Tolgee çalışma anı bağımlılığı değildir: düşerse yalnız çeviri düzenleme durur, OKSİS etkilenmez. Çeviriler her zaman depodaki JSON/kaynak dosyalarıdır; Tolgee bırakılırsa dosyalar kalır.
- Akış: geliştirici yeni anahtarı `tr` ile koda yazar → CI anahtarları Tolgee'ye yükler → çeviri Tolgee'de yapılır → CI `en.json`/kaynakları dışa aktarıp PR açar.
- MEB terimleri için Tolgee sözlüğü (glossary): devamsızlık, şube, nöbet, karne, rehberlik vb. `oksis-education-expert` onaylar.
- Eksik çeviri: `tr`'ye düşer (fallback), CI eksik anahtar raporu üretir; üretim dağıtımı eksik `en` anahtarında **uyarır, durdurmaz**.

## Değerlendirme

| Ölçüt | A | B | C |
|---|---|---|---|
| Çalışma anı ağ bağımlılığı | Var | Yok | Yok |
| Mobil çevrimdışı metin | Yok | Var | Var |
| Geliştirici dışı çeviri | Var | Yok | Var |
| Yeni işletim birimi | API servisi (kritik yol) | Yok | Tolgee (kritik yol dışı) |
| Tenant izolasyonu | Ek tasarım | Gerekmez | Gerekmez (çeviri tenant'sız) |

## Göç planı (taslak)

1. **Zemin:** `packages/i18n`, `next-intl`/`i18next` kurulumu, API `RequestLocalization`, `PreferredLanguage` göçü, Tolgee kurulumu ve CI akışı, dil seçici. Türkçe görünüm değişmez.
2. **Modül modül göç:** her modülde UI metinleri + API hata/validasyon + bildirim + Excel aynı dilimde taşınır. Öncelik veli ve öğrenci tarafı (İngilizce talebi en çok buradan gelir), sonra öğretmen, en son yönetim.
3. **Kapanış:** ESLint sabit metin kuralı bütün depoda açılır, [[tech-stack]] ve [[bilesen-ve-stil-kurallari]] güncellenir, [[urun-tanimi]] kapsam dışı listesinden çoklu dil çıkarılır.

Göç sırasında yeni yazılan her kod baştan anahtarla yazılır (zemin bittiği gün kural olur).

## Sonuçları

- Kural değişikliği: "i18n yok, metinler doğrudan Türkçe" kuralı kalkar; "sistem metni anahtarla yazılır" kuralı gelir.
- Bildirim kodu alıcı dilini bilmek zorunda: fan-out sorgularına `PreferredLanguage` eklenir.
- Testler: Türkçe metin karşılaştıran testler anahtar ya da `tr` kültürü sabitlenerek yazılır.
- Postman koleksiyonlarına `Accept-Language` örnekleri eklenir.

## Kararlar (Faruk)

1. ~~Çeviri yönetim aracı~~ → **Tolgee, self-hosted, ücretsiz sürüm** (2026-10-23).
2. ~~Çeviriyi kim yapacak~~ → **karar verildi** (2026-10-22):
   - **Mevcut metinlerin tamamı** modül göçü sırasında Claude tarafından çevrilir. Terimler için önce bir İngilizce terim sözlüğü (MEB terimleri: şube, devamsızlık, karne, nöbet, rehberlik…) çıkarılır, `oksis-education-expert` denetler, Faruk onaylar; toplu çeviri bu sözlüğe bağlı yapılır.
   - **İleride eklenen metinler** insan çevirmen tarafından çevrilir. Geliştirici yeni anahtarı yalnız `tr` ile ekler; `en` çevirmen doldurana kadar `tr`'ye düşer (fallback) ve CI eksik anahtar raporunda görünür.
3. ~~Okul varsayılan dili~~ → **v1'de var** (2026-10-22).
4. ~~Referans verisi~~ → **anahtar kolonu, tek yöntem; dersler çevrilmez** (2026-10-22).

## Geri alma maliyeti

**Orta.** Zemin aşaması geri alınabilir (Türkçe görünüm değişmez). Modül göçü başladıktan sonra geri dönmek, anahtarları yeniden sabit metne çevirmek demektir; anlamlı değil. Tolgee bırakılırsa JSON dosyaları depoda kalır, B seçeneğine düşülür.
