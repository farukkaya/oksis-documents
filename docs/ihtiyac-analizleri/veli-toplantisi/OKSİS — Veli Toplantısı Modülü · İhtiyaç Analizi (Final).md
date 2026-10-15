
| | |
|---|---|
| **Belge türü** | İhtiyaç analizi — nihai fonksiyonel analiz (ürün + iş perspektifi) |
| **Talep** | REQ-043 · Veli Toplantısı modülü V1 · Faz 1 |
| **Tarih** | 15 Ekim 2026 |
| **Durum** | **Kararlar kesinleşti** (Faruk, 2026-10-15; karar turu 1 ve 2) + paydaş paneli bulguları işlendi → sıradaki adım teknik analiz |
| **Kapsam (V1)** | Okul geneli/şube bazlı planlama, yayın, veli katılım yanıtı (beyan) ve geçmişi, idarenin veli adına yanıtı, hatırlatmalar, iptal ve tarih değişikliği, öğretmenin fiili katılım işaretlemesi, öğrenci başına görüşme notu, idare katılım raporu, push + uygulama içi + **e-posta** (E-44), SMS altyapısı (gönderimsiz) |
| **Kapsam dışı** | Gerçek SMS gönderimi, branş öğretmeni katılımı, e-postadan yanıt verme, birebir randevu sistemi (§14) |
| **Kaynak talep** | Faruk'un V1 analizi (ChatGPT taslağı, "OKSİS – Veli Toplantısı Modülü · Fonksiyonel ihtiyaç analizi ve iş kuralları · V1") |
| **Kod dalı** | `feature/veli-toplantilari` (oksis-api-veli-toplantilari, oksis-ui-veli-toplantilari) — modül için henüz kod yok (2026-10-15 tarandı) |

> **Okuma notu:** Bu belge davranışı tanımlar; veri modeli ve API teknik analizde çıkar. Kod adları (`parentMeeting` vb.) **öneridir**; sözlüğe girmeden kodlanmaz (§15).

---

## 1. Yönetici Özeti

Veli Toplantısı modülü, idarenin okul geneli ya da seçili şubeler için **tek bir organizasyon** altında planladığı, ama **her şubenin toplantısını ayrı yönettiği** (farklı gün/saat, tek şube iptali, tek şube tarih değişikliği) bir süreç modülüdür. Toplantı yayınlandığında veliler push, uygulama içi bildirim ve (tercih ettilerse) e-postayla haberdar olur; her veli **Katılacağım / Katılamayacağım** yanıtını kendisi verir ve dilediği zaman değiştirir. İdare telefonla ulaştığı velinin adına yanıt girebilir; kim, ne zaman, kimin adına sorusu kaybolmaz.

Toplantı günü öğretmen (şubenin sınıf rehber öğretmeni) **beyandan bağımsız** olarak fiili katılımı işaretler ve **öğrenci başına** görüşme notu tutar. Not velilere gösterilmez; yalnız sınıf rehber öğretmeni ve idare görür, her okuma ve değişiklik kayda geçer ve süre dolunca metni silinir. İdare raporunda **beyan** ile **fiili katılım** iki ayrı eksendir.

Modül sıfırdan başlamıyor: bildirim hattı (uygulama içi + push, sessiz saat, alıcı tercihi), veli-öğrenci ilişkisi ve TB-78 veli süzgeci, şubenin rehber öğretmeni (`ClassRoom.HomeroomTeacherId`), müdür yardımcısı rolü (E-29), e-posta kanalının gönderim/adres/hata davranışı ve `ISmsSender` arayüzü hazır. Yeniler: toplantı organizasyonu + şube toplantısı, yanıt ve yanıt geçmişi, fiili katılım, görüşme notu ve okuma denetimi, beş-altı bildirim türü, **E-44 alıcı e-posta tercihi** (yalnız bu modülü değil tüm bildirim türlerini açar), okul ayarı eki, ~17 ekran.

> **Tek cümle:** İdare bir sihirbazla tüm okula toplantıyı yayınlar; veli telefonundan tek dokunuşla "Katılacağım" der; toplantı günü öğretmen gelenleri tek dokunuşla işaretler, öğrenci başına notunu yazar ve "Toplantıyı tamamla" der; idare akşam hangi şubede kaç velinin geleceğini söylediğini ve kaçının gerçekten geldiğini tek tabloda görür.

---

## 2. Kesinleşen Kararlar (K-1…K-16)

| Kod | Karar | Kesin hüküm | Sonucu |
|---|---|---|---|
| **K-1** | Kim planlar | **İdare** (= okul yöneticisi + müdür yardımcısı) toplantıyı planlar, yayınlar, tarihini değiştirir, iptal eder. Öğretmen planlamaz. | Planlama yüzeyleri yalnız idarede (§5). |
| **K-2** | Organizasyon modeli | Okul geneli **tek organizasyon**; altında **her şubenin toplantısı ayrı kayıt** — kendi tarihi, saati, yeri, durumu, iptali. | İptal/tarih değişikliği şube satırında yapılır; diğer şubeler etkilenmez. Toplu işlem = seçili satırlara aynı işlem. |
| **K-3** | Sorumlu | Şubenin **sınıf rehber öğretmeni** (`ClassRoom.HomeroomTeacherId`). Branş öğretmeni kapsam dışı. | Sorumlu kayıtta dondurulmaz; işlem anındaki güncel rehber öğretmendir (BR-PM-07). |
| **K-4** | Yaşam döngüsü | **Taslak → Yayınlandı → Başladı** (başlangıç saatinde otomatik) **→ Tamamlandı** (öğretmen elle; güvenlik ağı: sistem X gün sonra otomatik). **İptal** yayından sonra da mümkün, kayıtlar silinmez, iptal bildirimi gider. | §7 durum makinesi. X için öneri: **3 gün** (§16 S-3). |
| **K-5** | Tarih kısıtı | Geçmiş tarih olamaz, **bugün olabilir**; başka kısıt yok (hafta sonu, tatil, akşam saati serbest). | Yorum: başlangıç **anı** "şimdi"den sonra olmalı — teyit §16 S-5. |
| **K-6** | Öğretmen gelemezse | Başka öğretmen **atanmaz**. İdare iptal eder ya da etmez; etmezse idare toplantıyı kendisi yürütür — fiili katılım ve not yetkisi idarede de vardır. | Vekil/devir kavramı yok. |
| **K-7** | Veli yanıtı | **Katılacağım / Katılamayacağım** (+ isteğe bağlı neden) / **Yanıt yok**. Her veli ayrı yanıt verir; çoklu çocukta her şube toplantısı için ayrı yanıt. Veli kararını değiştirebilir. | Yanıt kaydı veli **kişisi** üzerinedir, hesap üzerine değil (hesapsız veliye de idare yanıt girebilir). |
| **K-8** | Veli adına yanıt | İdare veli adına yanıt girebilir/güncelleyebilir; kim / ne zaman / kimin adına geçmişi korunur; veli sonra yine değiştirebilir. | Yanıt geçmişi eklemeli (append-only). |
| **K-9** | Tarih/saat değişikliği | Yanıtlar **sıfırlanır**, yeniden yanıt istenir; eski yanıtlar geçmişte kalır; değişiklik bildirimi gider. | "Plan sürümü" kavramı (§3); idareye önceden etki önizlemesi (BR-PM-17). |
| **K-10** | Beyan ≠ fiili | Beyan (yanıt) ile fiili katılım ayrı veridir; raporlar ikisini bağımsız gösterir. | Fiili katılım beyandan **ön doldurulmaz**. |
| **K-11** | Görüşme notu | **Öğrenci başına.** Görenler: sınıf rehber öğretmeni + idare. Veli görmez. Not için ayrı izin yok. Erişim ve değişiklikler kayıt altında. | Okuma dahil denetim (BR-PM-31). |
| **K-12** | Not saklama | Okul ayarı; varsayılan **sezon kapanışından (arşivlenmesinden) sonra 2 yıl**. Süre dolunca not **metni** gece işiyle silinir (anonimleştirme değil), katılım kaydı kalır. Öğrenci ayrılırsa aynı süre işler. | §12 KVKK. |
| **K-13** | Kanallar | Push + uygulama içi (mevcut hat) + **e-posta**. E-posta bu modülde açılır: **E-44** — `NotificationPreference`'a e-posta tercihi, tüm bildirim türleri için alıcı seçimi. E-posta **yalnız bilgilendirme**: toplantı ayrıntısı; **düğme/katılım bağlantısı yok**. | Kaynak talepteki "e-postadan güvenli onay ekranı" kaldırıldı (karar turu). |
| **K-14** | Hesabı olmayan veli | SMS ile bilgilendirilecek; **V1'de yalnız altyapı** (gerçek gönderim yok). | V1'de bu veliye fiilen hiçbir kanal ulaşmaz; idare listeden telefonla arar (BR-PM-40). |
| **K-15** | Hatırlatmalar | Yayın → ilk duyuru; toplantı günü sabahı (saat okul ayarı) → "Katılacağım" diyenlere; başlangıç saatinde isteğe bağlı bildirim (okul ayarıyla aç/kapa); iptal → iptal bildirimi; tarih değişikliği → değişiklik bildirimi. Toplantı günü veli ve öğretmen anasayfasında kart. | §9. |
| **K-16** | Rapor ve eşitlik | İdare katılım özeti/raporu (beyan vs fiili). **Web ve mobil özellik eşitliği.** | §10 envanter her satırı iki platformda. |

### Karar turu — soru/cevap kaydı

| Konu | Kaynak talepteki öneri/belirsizlik | Cevap (Faruk, 2026-10-15) | Analizdeki karşılığı |
|---|---|---|---|
| Planlayan | İdare | İdare = müdür + müdür yardımcısı | K-1 |
| Organizasyon | Önerildi: tek organizasyon, şube bazlı yönetim | Kabul | K-2 |
| Sorumlu | "Mevcut sınıf öğretmeni" | Şubenin sınıf rehber öğretmeni (`HomeroomTeacherId`); branş kapsam dışı | K-3 |
| Yaşam döngüsü | Taslak/Yayınlandı/Başladı/Kapandı/Gerçekleşti/İptal | Taslak → Yayınlandı → Başladı (otomatik) → Tamamlandı (elle + X gün güvenlik ağı); İptal. "Kapandı" ve "Gerçekleşti" tek duruma indi | K-4 |
| Tarih kısıtı | "Sistemsel kısıt yok" | Geçmiş olamaz, bugün olabilir, başka kısıt yok | K-5 |
| Öğretmen uygun değil | "İdare iptal edebilir" | Başka öğretmen atanmaz; iptal ya da idare yürütür | K-6 |
| Yanıt modeli | Üç durum | Her veli ayrı; çoklu çocukta toplantı başına ayrı; değiştirilebilir | K-7 |
| Veli adına yanıt | Önerildi | Kabul, geçmişli | K-8 |
| Tarih değişince yanıtlar | Belirsizdi | Sıfırlanır, geçmişte kalır, bildirim gider | K-9 |
| Beyan / fiili | Ayrı tutulsun önerisi | Kabul | K-10 |
| Görüşme notu | "İdare: yetkiye bağlı"; veliye gösterilmesin önerisi | Öğrenci başına; öğretmen + idare görür; ayrı izin yok; veli görmez | K-11 |
| Not saklama | Yoktu | Okul ayarı, varsayılan sezon kapanışı + 2 yıl, gece işiyle metin silinir | K-12 |
| E-posta | "Toplantı detayı ve **katılım butonları**" + güvenli onay ekranı | **Kaldırıldı.** E-posta yalnız bilgilendirme, bağlantı/düğme yok. E-44 ile alıcı e-posta tercihi açılır | K-13 |
| Uygulaması olmayan veli | "E-posta gönderilmesi desteklenir" | Hesabı olmayan veli SMS ile; V1'de yalnız altyapı | K-14 (e-posta kanalı hesap ister, §11) |
| Hatırlatmalar | Sabah + başlangıç + iptal | Aynen + tarih değişikliği bildirimi + anasayfa kartı | K-15 |
| SMS, branş öğretmeni | Sonraki faz | Teyit | §14 |

### Paydaş paneli bulguları (2026-10-15) — işlenme yeri

| Bulgu | İşlendiği yer |
|---|---|
| Not erişiminde okuma dahil denetim | BR-PM-31 |
| Tarih değişikliğinde sıfırlama önizlemesi + sessiz olmayan sıfırlama | BR-PM-17, BR-PM-18, PM-A-05 |
| Yanıtsız veliler listesi: telefon kısayolu + "veli adına işaretle" | BR-PM-24, PM-A-04 |
| Otomatik kapanışta not kaybı yok, otomatik kayıt, kapanış sonrası düzenleme penceresi | BR-PM-12, BR-PM-29, BR-PM-30 |
| Not panelinde kalıcı "veliye gösterilmez" etiketi | BR-PM-32, PM-T-03 |
| Kanal tekrarı / tekilleştirme / yanıt verene yanıt çağrısı tekrarlanmaz | BR-PM-35…37 |
| Çoklu çocuk: çocuk başına kart + çakışma uyarısı | BR-PM-22, BR-PM-23, PM-P-01, PM-A-02 |
| Ayrı yaşayan ebeveynler, `CanViewInfo` süzgeci | BR-PM-14, BR-PM-21 |
| Katılamama nedeni öğrenciye/diğer veliye görünmez | BR-PM-21 |
| Planlama sihirbazı, şube durum tablosu, toplu işlem | PM-A-02, PM-A-03 |
| Öğretmen katılım ekranı ögeleri | PM-T-02 |
| Veli kartı ögeleri | PM-P-01, PM-P-02 |
| İstisnalar (öğretmen değişti, şube değişti/ayrıldı, iptal arşivi, saati geçmiş bugün, silme ≠ anonimleştirme) | §11, §16 |
| Hesapsız veliye V1'de fiilen ne ulaşır | BR-PM-40, §11 |

---

## 3. Kavramlar & Temas Eden Modüller

### Kavramlar

| Kavram | Tanım |
|---|---|
| **Veli Toplantısı (organizasyon)** | İdarenin tek seferde planladığı çatı kayıt: başlık, açıklama, varsayılan tarih/saat/yer, kapsadığı şubeler. Kendi yaşamı yalnız Taslak ve Yayınlandı'dır; asıl iş şube toplantılarındadır. Sezona bağlıdır. |
| **Şube Toplantısı** | Organizasyonun bir şubeye inmiş örneği: kendi tarihi, başlangıç saati, isteğe bağlı bitiş saati, yeri, durumu, sorumlusu (şubenin güncel rehber öğretmeni). İptal, tarih değişikliği, katılım ve notlar bu kayda bağlıdır. |
| **Davetli veli** | Şube toplantısının öğrencisi için aktif ve bilgi görme yetkisi olan (TB-78) veli-öğrenci ilişkisi taşıyan veli kişisi. Liste toplantı **Başladı** olana dek dinamik, Başladı anında sabitlenir. |
| **Katılım yanıtı (beyan)** | Velinin "Katılacağım / Katılamayacağım" beyanı; hiç yanıt yoksa "Yanıt yok". Katılamayacağım'da isteğe bağlı neden. Veli × şube toplantısı başına tek güncel yanıt. |
| **Yanıt geçmişi** | Her yanıt değişikliğinin eklemeli kaydı: önceki → yeni, kim girdi (veli / idare kullanıcısı), kimin adına, ne zaman, hangi plan sürümünde. Silinmez. |
| **Plan sürümü** | Şube toplantısının tarih/saat değiştikçe artan sürüm numarası. Güncel yanıtlar güncel sürüme aittir; sürüm artınca güncel yanıtlar "Yanıt yok"a döner, öncekiler geçmişte kalır. Bildirim tekilleştirmesinin de anahtarıdır. |
| **Fiili katılım** | Toplantıda öğretmenin (ya da yürüten idarecinin) veli başına işaretlediği "Geldi" bilgisi; işaretsiz = "Gelmedi" sayılır ama arayüzde "İşaretlenmedi" ile karışmaması için toplantı Tamamlandı olana dek nötr gösterilir. Beyandan ön doldurulmaz. |
| **Görüşme notu** | Öğrenci başına, toplantıda tutulan serbest metin. Sınıf rehber öğretmeni + idare görür; veli ve öğrenci asla görmez. Her okuma ve değişiklik denetim kaydına yazılır. Saklama süresi sonunda metin silinir. |
| **Düzenleme penceresi** | Toplantı Tamamlandı olduktan sonra fiili katılım ve notların düzenlenebildiği süre (öneri: 7 gün). Sonrası salt okunur. |
| **Hesapsız veli** | Veli-öğrenci ilişkisi olan ama bağlı OKSİS hesabı olmayan veli kişisi. V1'de hiçbir kanaldan ulaşılamaz; idarenin "telefonla ara" listesinde görünür. |

### Temas eden modüller

| Modül | İlişki | Durum | Ne kullanılır / ne değişir |
|---|---|---|---|
| **Classes & Sections** (`ClassRoom`) | Girdi | Hazır | Şube roster'ı; `HomeroomTeacherId` = sorumlu; rehbersiz şube mümkün (`ClassRoom` §5.8). |
| **Veli-öğrenci ilişkisi** (`ParentStudentRelationship`) | Girdi | Hazır | Davetli veli kümesi `NotificationRecipientResolver.ResolveGuardianPersonIdsByStudentAsync` ile, **varsayılan güvenli taraf** (`includeInfoRestricted: false`, `CanViewInfo` süzgeci, `RevokedAt == null`). |
| **Academic Sessions** | Girdi | Hazır | Sezon sert bölüm; `Archived` sezon salt okunur ve saklama sayacının başlangıcı. |
| **Müdür yardımcısı** (E-29) | Girdi | Hazır | İdare = SchoolAdmin + VicePrincipal; alıcı çözümünde de idare böyle tanımlı. |
| **Notifications** | Çıktı | Genişleme | Yeni türler (§9). Hat aynen: domain olayı → enqueuer → dispatch; sessiz saat yalnız push'ta (`PushQuietHours`). |
| **E-posta kanalı** (`EmailNotificationChannel`) | Çıktı | **E-44 ile açılır** | Bugün tüm olaylarda kapalı (`RecipientCanChooseEmail => false`). Gönderim, adres çözümü (Hesap → Kişi → `PrimaryEmail`) ve hata davranışı hazır. |
| **Bildirim tercihleri** (`NotificationPreference`) | Çıktı | Genişleme | Bugün yalnız `PushEnabled`; E-44 e-posta tercihini ekler (tüm türler). |
| **SMS** (`ISmsSender`, SMS kotası) | Altyapı | Arayüz var, uygulama yok | V1'de hesapsız veli için gönderim **niyeti** üretilir ama gönderilmez (§9.3). |
| **Anasayfa** (`home`) | Çıktı | Ekleme | Veli ve öğretmen anasayfasına toplantı günü kartı; öğretmen **Bekleyen İşler**'e "tamamlanmayı bekleyen toplantı". |
| **Rehber** (`contact`) | Girdi | Hazır | Telefon kısayolu velinin kayıtlı telefonunu kullanır. |
| **Raporlar** | Çıktı | Ekleme | Katılım raporu (§13). |
| **Duyuru** | Yok | — | Veli toplantısı duyuru değildir; toplu duyuru akışı kullanılmaz (kendi bildirim türleri var). |

---

## 4. Rol İhtiyaçları

**İdare — "tek seferde planla, şube şube yönet, kimin geleceğini bil"**
Dönem başı toplantısını tüm okula iki dakikada kurmak; ilkokulu 17:00'ye, liseyi 18:30'a koymak; tek şubenin öğretmeni hastalanınca yalnız onu iptal etmek ya da kendisi yürütmek; toplantıdan iki gün önce yanıt vermeyen velileri telefonla aramak ve sonucu sisteme işlemek; akşam beyan/fiili tablosunu görmek.

**Sınıf rehber öğretmeni — "kim gelecek, kim geldi, kime ne konuştum"**
Toplantıdan önce gelecekleri görmek; toplantıda tek dokunuşla "Geldi" işaretlemek; görüşürken öğrencinin notunu yazmak (telefonda, kesintiye dayanıklı); bitince "Toplantıyı tamamla". Korkusu: notun veliye sızması ve notun kaybolması — kalıcı "veliye gösterilmez" etiketi ve otomatik kayıt bunu karşılar.

**Veli — "ne zaman, nerede, gelebilecek miyim"**
Bildirimden tek dokunuşla yanıt vermek; iki çocuğun toplantısını karıştırmamak; planı değişirse yanıtını değiştirmek; toplantı günü hatırlatma almak. Ayrı yaşayan ebeveyn, diğer ebeveynin yanıtını ve nedenini görmez.

**Öğrenci** — V1'de modülün kullanıcısı değildir (bildirim almaz, ekran görmez). Ortaokul/lise öğrencisine salt okunur görünürlük §16 S-2'de.

---

## 5. Yetki Matrisi (rol × işlem)

İdare = okul yöneticisi (SchoolAdmin) + müdür yardımcısı (VicePrincipal). "Rehber öğr." = ilgili şubenin **güncel** sınıf rehber öğretmeni.

| İşlem | İdare | Rehber öğr. (kendi şubesi) | Diğer öğretmen | Veli | Öğrenci |
|---|---|---|---|---|---|
| Organizasyon oluşturma, taslak düzenleme, taslak silme | ✓ | — | — | — | — |
| Yayınlama, sonradan şube ekleme | ✓ | — | — | — | — |
| Tarih/saat/yer değiştirme (şube veya toplu) | ✓ | — | — | — | — |
| İptal (şube veya toplu) | ✓ | — | — | — | — |
| Toplantı ayrıntısını görme | ✓ (tümü) | ✓ | — | ✓ (kendi davetli olduğu) | — (§16 S-2) |
| Kendi yanıtını verme/değiştirme | — | — | — | ✓ | — |
| Veli adına yanıt girme | ✓ | — | — | — | — |
| Yanıtları ve yanıt geçmişini görme | ✓ | ✓ (geçmiş dahil) | — | Yalnız kendi yanıtı | — |
| Katılamama nedenini görme | ✓ | ✓ | — | Yalnız kendi nedeni | — |
| Fiili katılım işaretleme | ✓ (K-6) | ✓ | — | — | — |
| Görüşme notu yazma/düzenleme | ✓ (K-6) | ✓ | — | — | — |
| Görüşme notunu okuma | ✓ | ✓ (eski rehber öğr.: §16 S-4) | — | **Asla** | **Asla** |
| Not denetim kaydını görme | ✓ | — | — | — | — |
| Toplantıyı tamamlama | ✓ | ✓ | — | — | — |
| Katılım raporu | ✓ (okul) | ✓ (kendi şubesi) | — | — | — |
| Hesapsız veli listesi + telefon kısayolu | ✓ | ✓ (yalnız görme + arama; veli adına yanıt yok) | — | — | — |
| Modül okul ayarları | ✓ | — | — | — | — |

**İzin önerisi** (mevcut `{modül}.{eylem}` kalıbı — `homework.*`, `clubs.*` emsali): `parentMeetings.read` (görme; kapsam kapısıyla daralır), `parentMeetings.write` (fiili katılım, not, tamamlama — rehber öğretmen kapsamında), `parentMeetings.manage` (planlama, yayın, iptal, tarih, veli adına yanıt, ayarlar, okul raporu). **Not için ayrı izin yok** (K-11): not erişimi `write` + kapsam kapısıyla çözülür. Velinin yanıt vermesi `read` + veli-öğrenci kapsamıdır. Kapsam dışı istek 404, izinsiz istek 403.

---

## 6. Uçtan Uca Akışlar

### 6.1 İdare — planlama ve yayın

1. **Veli Toplantıları** listesinden "Yeni toplantı" (PM-A-01 → PM-A-02).
2. **Adım 1 · Şubeler:** kademe/seviye/şube ağacından seçim ("Tüm okul", "Tüm ilkokul" kısayolları). Rehber öğretmeni olmayan şube uyarı rozetiyle görünür (seçilebilir).
3. **Adım 2 · Tarih ve saat:** başlık, açıklama, varsayılan tarih/başlangıç saati/(isteğe bağlı) bitiş saati/yer; altında şube tablosu — satır satır ya da seçili satırlara toplu farklı tarih/saat/yer. Geçmiş an girilemez (BR-PM-04).
4. **Adım 3 · Önizleme:** şube sayısı, davetli öğrenci ve veli sayısı, kanal kırılımı tahmini (uygulama içi/push ulaşacak, e-posta tercihli, hesapsız veli sayısı), aynı veliye düşen **çakışmalar** listesi (BR-PM-23).
5. "Taslak kaydet" (bildirim yok) ya da "Yayınla" (onay diyaloğu: "N şube, M veliye duyuru gidecek").
6. Yayında her şube toplantısı **Yayınlandı** olur, ilk duyuru gider (N-01), rehber öğretmene bilgi gider (N-02).
7. Takip: organizasyon detayında (PM-A-03) şube durum tablosu — durum, yanıt sayaçları (Katılacak / Katılamayacak / Yanıt yok), fiili katılım (toplantı sonrası).
8. Toplantıdan önce: şube detayında (PM-A-04) **Yanıt yok** ve **Hesapsız** sekmeleri → telefon kısayolu → "Veli adına işaretle".
9. Gerekirse şube satırında "Tarih değiştir" (PM-A-05 etki önizlemesiyle) ya da "İptal".
10. Sonrası: katılım raporu (PM-A-06).

### 6.2 Öğretmen — toplantıyı yürütme

1. Yayında bildirim + "Toplantılarım" listesinde kayıt (PM-T-01).
2. Toplantı günü anasayfada kart (PM-T-04): saat, yer, "Katılacak 18 / 32 veli".
3. Başlangıç saatinde durum **Başladı** olur; katılım ekranı (PM-T-02) işaretlemeye açılır.
4. Gelen her veli için tek dokunuş "Geldi"; üst sayaç beyan/fiili günceller.
5. Öğrenci satırına dokununca not paneli (PM-T-03) açılır; yazılan metin otomatik kaydedilir.
6. "Toplantıyı tamamla" → onay (işaretsiz veliler "Gelmedi" sayılacak bilgisi) → **Tamamlandı**.
7. Tamamladıktan sonra 7 gün içinde düzeltme yapabilir; sonra salt okunur.
8. Unutursa: Bekleyen İşler'de "Tamamlanmayı bekleyen toplantı"; X gün sonra sistem kapatır, notlar korunur.

### 6.3 Veli — yanıt

1. Bildirim (push/uygulama içi; tercih ettiyse bilgilendirme e-postası).
2. Bildirimden toplantı ayrıntısı (PM-P-02): çocuk adı, şube, tarih-saat, yer, açıklama; iki büyük düğme.
3. "Katılamayacağım" seçerse isteğe bağlı neden alanı (diğer veliye/öğrenciye görünmez uyarısıyla).
4. Kart yanıtı gösterir; "Yanıtı değiştir" her zaman (toplantı Başladı olana dek — BR-PM-19).
5. Tarih değişirse: "Toplantı saati değişti, lütfen yeniden yanıt verin" bildirimi; kart "Yanıt bekleniyor"a döner.
6. Toplantı günü sabahı (Katılacağım dediyse) hatırlatma; anasayfada kart.
7. İptal edilirse iptal bildirimi; kart iptal bandıyla görünür kalır.

### 6.4 İdare — öğretmen gelemedi

1. İdare şube satırını açar. Seçenek A: **İptal** (gerekçe isteğe bağlı, velilere iptal bildirimi). Seçenek B: **Tarih değiştir** (yanıtlar sıfırlanır). Seçenek C: hiçbir şey yapmaz ve toplantıyı kendisi yürütür — PM-A-04'ün "Yürüt" görünümü PM-T-02/PM-T-03 ile aynı işlevleri sunar.
2. Başka öğretmen atanmaz (K-6); kayıtta "yürüten" bilgisi işaretleyen kullanıcıdan okunur (denetim kaydı).

---

## 7. Yaşam Döngüsü ve Geçiş Kuralları

### 7.1 Şube toplantısı durumları

| Durum | Anlam | Kim görür |
|---|---|---|
| **Taslak** | Planlanıyor; kimseye ulaşmadı. | Yalnız idare |
| **Yayınlandı** | Duyuruldu; yanıt toplanıyor. | İdare, rehber öğretmen, davetli veliler |
| **Başladı** | Başlangıç anı geldi (otomatik). Davetli listesi sabitlendi; fiili katılım ve not açık. | Aynı |
| **Tamamlandı** | Öğretmen/idare kapattı ya da sistem X gün sonra kapattı. Düzenleme penceresi boyunca katılım/not düzeltilebilir. | Aynı |
| **İptal** | Yayınlandı ya da Başladı'dan iptal edildi. Kayıtlar (yanıtlar, varsa katılım/not) korunur. | Aynı (iptal bandıyla) |

Organizasyonun kendi durumu yalnız **Taslak / Yayınlandı**dır; "kaç şube tamamlandı/iptal" özeti şube durumlarından türetilir.

### 7.2 Geçiş tablosu

| Geçiş | Tetik | Koşul | Yan etki |
|---|---|---|---|
| — → Taslak | İdare oluşturur | Başlangıç anı gelecekte | Bildirim yok |
| Taslak → (silindi) | İdare taslağı siler | Yalnız Taslak | Yumuşak silme; bildirim yok |
| Taslak → Yayınlandı | İdare yayınlar | Başlangıç anı hâlâ gelecekte (yayın anında yeniden denetlenir) | N-01, N-02; davetli listesi oluşur |
| Yayınlandı → Yayınlandı | Tarih/saat değişikliği | Yeni an gelecekte | Plan sürümü +1, yanıtlar sıfırlanır, N-03 |
| Yayınlandı → Yayınlandı | Yalnız yer/açıklama değişikliği | — | Yanıt sıfırlanmaz; yer değiştiyse N-03 (bilgi gövdesi); başlık/açıklama sessiz, denetimde |
| Yayınlandı → Başladı | Sistem, başlangıç anında | — | Davetli listesi sabitlenir; isteğe bağlı N-06 |
| Başladı → Yayınlandı | Tarih değişikliği (erteleme) | Hiç fiili katılım ve not girilmemiş, yeni an gelecekte | Plan sürümü +1, yanıtlar sıfırlanır, N-03 |
| Başladı → Tamamlandı | Rehber öğretmen / idare "Tamamla" | — | Düzenleme penceresi başlar |
| Başladı → Tamamlandı | Sistem, başlangıç gününden X gün sonra gece | Hâlâ Başladı | Notlar korunur; denetimde "otomatik kapandı" |
| Yayınlandı / Başladı → İptal | İdare | — | N-04; kayıtlar korunur |
| Tamamlandı → * | — | **Yok** (terminal) | — |
| İptal → * | — | **Yok** (terminal; yeniden planlama = yeni şube toplantısı) | — |

---

## 8. İş Kuralları (BR-PM-01…42)

### Planlama ve yayın

| Kod | Kural | Gerekçe / not |
|---|---|---|
| **BR-PM-01** | Organizasyon oluşturma, düzenleme, yayın, tarih değişikliği, iptal ve veli adına yanıt yalnız idare (SchoolAdmin, VicePrincipal) tarafından yapılır; öğretmen bu uçlara 403 alır. | K-1 |
| **BR-PM-02** | Bir organizasyon en az bir şube içerir; her şube bir organizasyonda en çok bir şube toplantısına sahiptir. | K-2; aynı organizasyonda çift kayıt engeli |
| **BR-PM-03** | Her şube toplantısının tarihi, başlangıç saati, isteğe bağlı bitiş saati ve yeri bağımsızdır; organizasyonun değeri yalnız varsayılandır. Bitiş saati varsa başlangıçtan sonra olmalıdır. | K-2 |
| **BR-PM-04** | Oluşturma, yayın ve tarih değişikliği anında şube toplantısının başlangıç **anı** (tarih + saat, okul saat dilimi) şimdiden sonra olmalıdır; bugünün ileri bir saati kabul edilir. Başka tarih kısıtı yoktur (hafta sonu, tatil, saat aralığı serbest). | K-5; "saati geçmiş bugün" yorumu §16 S-5 |
| **BR-PM-05** | Taslak hiçbir bildirim üretmez ve yalnız idareye görünür. Taslak silinebilir (yumuşak silme); yayınlanmış şube toplantısı silinemez, yalnız iptal edilir. | Ödev BR-HW-03/04 emsali |
| **BR-PM-06** | Yayınlanmış organizasyona sonradan şube eklenebilir; eklenen şube onayla doğrudan Yayınlandı olur ve yalnız o şubenin velilerine N-01 gider. Yayınlanmış şube organizasyondan çıkarılamaz, iptal edilir. | K-2 |
| **BR-PM-07** | Şube toplantısının sorumlusu kayıtta dondurulmaz; her işlemde şubenin **güncel** `HomeroomTeacherId`'si yetkilidir. Rehber öğretmeni olmayan şube planlanabilir ve yayınlanabilir; planlama ve önizlemede "rehber öğretmen yok — toplantıyı idare yürütür" uyarısı gösterilir. | K-3, K-6 |
| **BR-PM-08** | Kapalı (arşivlenmiş) sezonda yeni toplantı oluşturulamaz; mevcut kayıtlar salt okunurdur. | Platform kuralı |

### Yaşam döngüsü

| Kod | Kural | Gerekçe / not |
|---|---|---|
| **BR-PM-09** | Yayınlandı → Başladı geçişi başlangıç anında sistemce yapılır; kullanıcı eylemi gerekmez. Gecikme toleransı en çok 5 dk (zamanlanmış iş). | K-4 |
| **BR-PM-10** | Fiili katılım ve görüşme notu yalnız Başladı ve (düzenleme penceresi içindeki) Tamamlandı durumlarında girilebilir; Yayınlandı'da kapalıdır. | Beyan/fiili ayrımı |
| **BR-PM-11** | "Toplantıyı tamamla" rehber öğretmen ya da idare tarafından yapılır; onay diyaloğu işaretsiz veli sayısını ve "Gelmedi sayılacak" bilgisini gösterir. | K-4 |
| **BR-PM-12** | Başlangıç gününden **X = 3 gün** sonra (gece işi) hâlâ Başladı olan şube toplantısı sistemce Tamamlandı'ya geçer; girilmiş notlar ve katılım aynen korunur, kayda "otomatik kapandı" yazılır. X sistem sabitidir (V1'de okul ayarı değil). | K-4 güvenlik ağı; değer §16 S-3 |
| **BR-PM-13** | İptal Yayınlandı ya da Başladı'dan yapılabilir; gerekçe isteğe bağlıdır (veliye gidecek gövdeye **yazılmaz**, yalnız idare görür). İptal edilen kaydın yanıtları, geçmişi, varsa katılım ve notları silinmez; raporlarda "iptal" olarak ayrı sayılır. Tamamlandı ve İptal terminaldir. | K-4 |

### Davetli veli ve yanıt

| Kod | Kural | Gerekçe / not |
|---|---|---|
| **BR-PM-14** | Davetli veliler, şubenin aktif öğrencileri için **iptal edilmemiş** (`RevokedAt == null`) ve **bilgi görme yetkisi olan** (`CanViewInfo = true`) veli-öğrenci ilişkileridir — mevcut alıcı çözümünün varsayılan güvenli tarafı (`ResolveGuardianPersonIdsByStudentAsync`, `includeInfoRestricted: false`, TB-78). `CanViewInfo = false` veliye toplantı bildirimi gitmez, yanıt satırı açılmaz. | TB-78; ders programındaki `includeInfoRestricted: true` istisnası bu modüle **uygulanmaz** |
| **BR-PM-15** | Davetli listesi Yayınlandı süresince dinamiktir (BR-PM-41); Başladı anında sabitlenir. | İstisnalar §11 |
| **BR-PM-16** | Yanıt veli × şube toplantısı başına tektir; değerleri Katılacağım / Katılamayacağım / Yanıt yok. İkiz/aynı şubede iki çocuk varsa veli o toplantıya **tek** yanıt verir; yanıt her iki öğrencinin satırında görünür. Farklı şubelerdeki çocuklar için yanıtlar ayrıdır. | K-7 |
| **BR-PM-17** | Tarih/saat değişikliği kaydedilmeden önce idareye etki önizlemesi gösterilir: "**X** veli yanıtı sıfırlanacak, **Y** veliye değişiklik bildirimi gidecek" (şube bazında ve toplamda). Onaysız değişiklik yapılamaz. | Paydaş paneli |
| **BR-PM-18** | Tarih/saat değişikliğinde plan sürümü artar; tüm güncel yanıtlar "Yanıt yok"a döner, önceki yanıtlar sürüm bilgisiyle geçmişte kalır; **sıfırlama sessiz değildir** — tüm davetli velilere "Toplantı saati değişti, lütfen yeniden yanıt verin" (N-03) gider. Yalnız yer/açıklama değişikliği yanıtları sıfırlamaz. | K-9 |
| **BR-PM-19** | Veli yanıtını Yayınlandı süresince istediği kadar değiştirebilir; Başladı ve sonrasında veli yanıtı kilitlenir (yanıt bir **beyan**dır, toplantı başladıktan sonra anlamı fiili katılımdır). İdare veli adına yanıtı Başladı'da da güncelleyebilir. | K-7, K-10 |
| **BR-PM-20** | Veli adına yanıt girildiğinde geçmiş kaydı: önceki → yeni değer, giren kullanıcı, kimin adına, zaman, plan sürümü, isteğe bağlı kanal notu ("telefonla"). Veli sonra kendi yanıtını değiştirebilir; o değişiklik de geçmişe yazılır. Geçmiş silinmez ve düzenlenmez. | K-8 |
| **BR-PM-21** | Veli **yalnız kendi** yanıtını ve nedenini görür. Aynı öğrencinin diğer velisinin yanıtı, nedeni ya da varlığı veliye gösterilmez. Katılamama nedeni öğrenciye hiçbir yüzeyde görünmez; yalnız rehber öğretmen ve idare görür. | Ayrı yaşayan ebeveyn; KVKK |
| **BR-PM-22** | Çoklu çocuklu veli her şube toplantısını ayrı kart olarak görür; kartta çocuk adı ve şube her zaman yazar. | K-7 |
| **BR-PM-23** | Aynı veliye düşen iki şube toplantısının zaman aralıkları kesişiyorsa (bitiş saati yoksa başlangıçtan itibaren 60 dk varsayılır): veli kartında "Aynı saatte başka toplantınız var" uyarısı; idare planlama önizlemesinde ve tarih değişikliği önizlemesinde çakışan veli sayısı ve listesi. Çakışma **engellemez**. | Paydaş paneli |
| **BR-PM-24** | İdarenin şube detayında "Yanıt yok" ve "Hesapsız veliler" listeleri bulunur; her satırda velinin telefonuna arama kısayolu ve "Veli adına işaretle" eylemi vardır. Rehber öğretmen listeyi görür ve arayabilir, veli adına yanıt giremez. | Paydaş paneli; K-8 |

### Fiili katılım ve görüşme notu

| Kod | Kural | Gerekçe / not |
|---|---|---|
| **BR-PM-25** | Fiili katılım davetli veli başına "Geldi" işaretidir; beyandan ön doldurulmaz. Toplantı Tamamlandı olduğunda işaretsiz veliler raporda "Gelmedi" sayılır. | K-10 |
| **BR-PM-26** | Davetli listesinde olmayan bir kişi (ör. öğrencinin kayıtlı olmayan yakını) V1'de işaretlenemez; öğretmen bunu ilgili öğrencinin notuna yazar. | Kapsam sınırı |
| **BR-PM-27** | Görüşme notu öğrenci × şube toplantısı başına tek metindir (en çok 4.000 karakter önerisi); yazan ve son düzenleyen kullanıcı ile zamanları tutulur. | K-11 |
| **BR-PM-28** | Not okuma/yazma yetkisi: şubenin güncel rehber öğretmeni ve idare. Veli ve öğrenci hiçbir uçta, bildirimde, e-postada, raporda, dışa aktarımda not metnine erişemez. Diğer öğretmenler erişemez. | K-11; eski rehber öğretmenin erişimi §16 S-4 |
| **BR-PM-29** | Not alanı yazarken otomatik kaydedilir (kısa gecikmeli); bağlantı koparsa yerel taslak tutulur ve yeniden bağlanınca gönderilir. Otomatik kapanış (BR-PM-12) kaydedilmiş hiçbir notu silmez. | Paydaş paneli |
| **BR-PM-30** | Tamamlandı'dan sonra **7 gün** düzenleme penceresi: rehber öğretmen ve idare fiili katılımı ve notları düzeltebilir. Pencere sonrası salt okunur; değişiklik gerekiyorsa V1'de yapılamaz. | Paydaş paneli önerisi |
| **BR-PM-31** | Not üzerindeki **her okuma** (not panelinin açılması / not içeren listenin getirilmesi) ve her oluşturma/değişiklik denetim kaydına yazılır: kim, ne zaman, hangi öğrenci, hangi işlem; değişiklikte önceki metnin özeti değil **sürüm kaydı**. Denetim kaydını yalnız idare görür. Denetim kaydı not metni silindiğinde de kalır (metin içermez). | K-11, KVKK m.12 |
| **BR-PM-32** | Not paneli ve not alanı her platformda kalıcı "Veliye gösterilmez — yalnız sınıf rehber öğretmeni ve idare görür" etiketini taşır. | Paydaş paneli |
| **BR-PM-33** | Saklama süresi (okul ayarı, varsayılan sezon arşivlenmesinden sonra 2 yıl) dolan notların **metni** gece işiyle kalıcı silinir (anonimleştirme değil); not kaydının var olduğu bilgisi, katılım kayıtları ve denetim kaydı kalır. Öğrenci okuldan ayrılmışsa da aynı süre işler. Süre en az 0 (sezon kapanışında sil), en çok 10 yıl. | K-12 |

### Bildirim davranışı

| Kod | Kural | Gerekçe / not |
|---|---|---|
| **BR-PM-34** | Teslim = olay kataloğu ∩ alıcının tercihi (K-34). Uygulama içi kapatılamaz; push ve e-posta alıcının tür bazlı tercihiyle. Sessiz saat yalnız push'a uygulanır; e-posta ve uygulama içi ertelenmez. | Mevcut hat |
| **BR-PM-35** | Her bildirim deterministik bir olay kimliği taşır: (şube toplantısı, tür, plan sürümü, [hatırlatma günü]). Aynı kimlikle ikinci gönderim yapılmaz; aynı işlemde ardışık değişiklikler (ör. tarih değiştir + hemen iptal) aynı alıcıya kuyruktaki eski bildirimi **geçersiz kılar**: gönderim anında şube toplantısının güncel durumu/sürümü denetlenir, eskimiş bildirim atılır. | Paydaş paneli |
| **BR-PM-36** | Bir olay bir alıcıya kanal başına en çok bir kez gider; push ve uygulama içi aynı bildirimin iki yüzüdür (zilde tek kayıt). E-posta yalnız N-01 (yayın), N-03 (değişiklik) ve N-04 (iptal) için üretilir; hatırlatmalar e-postaya gitmez. | Kanal tekrarı hissi |
| **BR-PM-37** | Yanıt vermiş veliye yanıt çağrısı tekrarlanmaz: N-01'den sonra "yanıt bekleniyor" türü bir çağrı yalnız güncel sürümde yanıtı olmayanlara gider (gönderilip gönderilmeyeceği §16 S-1). | Paydaş paneli |
| **BR-PM-38** | Sabah hatırlatması (N-05) yalnız güncel sürümde "Katılacağım" yanıtı olan davetli velilere, toplantı günü okul ayarındaki saatte gider. Yayın anı o günün hatırlatma saatinden sonraysa hatırlatma gönderilmez (ilk duyuru yeterlidir). | K-15 |
| **BR-PM-39** | Başlangıç bildirimi (N-06) okul ayarı açıksa, başlangıç anında "Katılacağım" diyen velilere gider. Başlangıç anı sessiz saat içindeyse push **atlanır** (ertelenmez — ertelenmiş "başladı" haberi anlamsızdır); uygulama içi kayıt yazılır. | K-15 |

### İstisna kuralları

| Kod | Kural | Gerekçe / not |
|---|---|---|
| **BR-PM-40** | Hesapsız veli için uygulama içi, push ve e-posta üretilmez (e-posta kanalı hesap üzerinden adres çözer). Yayın/değişiklik/iptal olaylarında hesapsız veli için **SMS gönderim niyeti** kaydedilir ama gönderilmez (V1 no-op). Bu veliler idarenin "Hesapsız veliler" listesinde görünür; idare telefonla arayıp veli adına işaretler. | K-14 |
| **BR-PM-41** | Davetli liste değişimi (Yayınlandı'da): yeni aktif öğrenci/veli ilişkisi → yeni davetli satırı + N-01; iptal edilen ilişki ya da şubeden ayrılan öğrenci → satır "davet düştü" olarak kapanır (silinmez), artık bildirim gitmez, veli kartı kaybolur. Başladı'dan sonra liste değişmez. | §11 |
| **BR-PM-42** | Sezon arşivlenirken: Başladı olanlar Tamamlandı'ya geçer (otomatik kapanış kaydıyla); Yayınlandı olanlar İptal olur (iptal bildirimi gider); Taslaklar silinir. Sonrası salt okunur; saklama sayacı bu andan başlar. | BR-PM-08, K-12 |

---

## 9. Bildirim Senaryoları

### 9.1 Tablo

Kod adları öneridir (`NotificationKind` / push anahtarı). "Uİ" = uygulama içi.

| # | Tür (öneri) | Tetikleyici | Alıcı | Kanal | Sessiz saat | İçerik (başlık · gövde) |
|---|---|---|---|---|---|---|
| **N-01** | `ParentMeetingPublished` / `PARENT_MEETING_PUBLISHED` | Şube toplantısı yayınlandı (ilk yayın ya da sonradan eklenen şube ya da davetliye sonradan katılan veli) | Davetli veliler (BR-PM-14) | Uİ + push + e-posta (tercih) ; hesapsıza SMS niyeti | Push sessiz saat bitişine ertelenir | "Veli toplantısı · 5-A" · "Ayşe için veli toplantısı 20 Ekim 18:00, Konferans Salonu. Katılımınızı bildirin." E-posta: aynı bilgiler + açıklama, **düğme/bağlantı yok**, "Yanıtınızı OKSİS uygulamasından verebilirsiniz" cümlesi |
| **N-02** | `ParentMeetingAssigned` (öğretmen) | Şube toplantısı yayınlandı / tarihi değişti / iptal edildi | Şubenin güncel rehber öğretmeni | Uİ + push | Ertelenir | "5-A veli toplantınız planlandı · 20 Ekim 18:00" (değişiklik/iptal gövdeleri aynı türde) |
| **N-03** | `ParentMeetingChanged` | Tarih/saat değişti (yanıt sıfırlandı) **ya da** yer değişti | Davetli veliler | Uİ + push + e-posta (tercih) ; SMS niyeti | Ertelenir | Tarih: "Toplantı saati değişti · Yeni: 22 Ekim 17:30. Lütfen yeniden yanıt verin." Yer: "Toplantı yeri değişti · Yeni yer: Spor Salonu" (yanıt istenmez) |
| **N-04** | `ParentMeetingCancelled` | Şube toplantısı iptal edildi | Davetli veliler | Uİ + push + e-posta (tercih) ; SMS niyeti | Ertelenir | "Veli toplantısı iptal edildi · 5-A, 20 Ekim 18:00 toplantısı iptal edildi." Gerekçe gövdeye yazılmaz |
| **N-05** | `ParentMeetingDayReminder` | Toplantı günü, okul ayarındaki sabah saati | Güncel sürümde "Katılacağım" diyen veliler; ayrıca rehber öğretmen | Uİ + push | Saat sessiz saate düşerse push bitişe ertelenir (ayar ekranı uyarır) | "Bugün veli toplantısı · 18:00, Konferans Salonu" |
| **N-06** | `ParentMeetingStarting` | Başlangıç anı (okul ayarıyla aç/kapa, varsayılan kapalı) | "Katılacağım" diyen veliler | Uİ + push | Push **atlanır** (BR-PM-39) | "Veli toplantısı başlıyor · 5-A, Konferans Salonu" |
| **N-07** | `ParentMeetingResponsePending` | Toplantıdan N gün önce | Güncel sürümde yanıtı olmayan veliler | Uİ + push | Ertelenir | "Toplantıya katılımınızı bildirmediniz · 20 Ekim 18:00" |

**Üretilmeyecek bildirimler (bilinçli):** veli yanıtı öğretmene/idareye bildirim üretmez (sayaçlar yeterli); otomatik kapanış bildirim üretmez — öğretmen anasayfasında Bekleyen İşler'de "Tamamlanmayı bekleyen toplantı" (bekleyen iş, okununca kaybolmaz) görünür; not yazımı/okunması hiçbir bildirim üretmez; başlık/açıklama düzeltmesi bildirim üretmez (denetimde kalır).

**Tür sayısı gerekçesi:** değişikliğin iki gövdesi (tarih/yer) tek türdür — veli ayarlarda ayırt edemeyeceği iki satır görmesin (`HomeworkMissing`, `ClubApplicationDecided` emsali). Öğretmen bilgisi tek tür (N-02), çünkü alıcı ve derin bağlantı aynı.

### 9.2 E-posta (E-44)

- E-44 `NotificationPreference`'a tür bazlı **e-posta tercihi** ekler ve tüm bildirim türleri için alıcı seçimini açar; e-posta kanalının `RecipientCanChooseEmail` kapısı tercih sorgusuna döner.
- Alıcının e-posta adresi yoksa (`PrimaryEmail` boş) e-posta atlanır, diğer kanallar etkilenmez; idare özetinde "e-posta adresi olmayan veli" sayısı görünür.
- Satır yokken varsayılan (açık mı kapalı mı) §16 S-6.
- E-posta içeriği: okul adı, çocuk adı, şube, tarih-saat, yer, açıklama. Not, neden, diğer veli bilgisi asla.

### 9.3 SMS (altyapı)

V1'de N-01/N-03/N-04 için hesapsız veliye SMS **niyeti** kaydedilir (alıcı, tür, olay kimliği, "gönderilmedi — kanal kapalı" durumu). Gerçek gönderim, şablon, kota (`DailySmsLimit`) ve İYS değerlendirmesi sonraki faz. Amaç: SMS açıldığında modülün değişmeden bağlanması ve idarenin "kaç veliye ulaşılamadı" sayısını bugünden görmesi.

---

## 10. Ekran Envanteri

### 10.1 Rol × platform

| Kod | Ekran | Rol | Web | Mobil |
|---|---|---|---|---|
| PM-A-01 | Veli Toplantıları listesi | İdare | ✓ | ✓ |
| PM-A-02 | Planlama sihirbazı (oluştur / taslak düzenle) | İdare | ✓ | ✓ |
| PM-A-03 | Organizasyon detayı — şube durum tablosu | İdare | ✓ | ✓ |
| PM-A-04 | Şube toplantısı detayı (yanıtlar, yanıtsızlar, hesapsızlar, yürüt) | İdare | ✓ | ✓ |
| PM-A-05 | Tarih değiştir / İptal diyaloğu (etki önizlemeli) | İdare | ✓ | ✓ |
| PM-A-06 | Katılım raporu | İdare | ✓ | ✓ |
| PM-A-07 | Veli Toplantısı ayarları (okul ayarı eki) | İdare | ✓ | ✓ |
| PM-A-08 | Not denetim kaydı | İdare | ✓ | ✓ |
| PM-T-01 | Toplantılarım | Rehber öğr. | ✓ | ✓ |
| PM-T-02 | Katılım takibi | Rehber öğr. (+ idare "Yürüt") | ✓ | ✓ |
| PM-T-03 | Görüşme notu paneli | Rehber öğr. + idare | ✓ | ✓ |
| PM-T-04 | Anasayfa toplantı kartı + Bekleyen İşler satırı | Rehber öğr. | ✓ | ✓ |
| PM-P-01 | Toplantılarım (çocuk başına kartlar) | Veli | ✓ | ✓ |
| PM-P-02 | Toplantı ayrıntısı + yanıt | Veli | ✓ | ✓ |
| PM-P-03 | Anasayfa toplantı kartı | Veli | ✓ | ✓ |
| PM-X-01 | Bildirim tercihleri — e-posta sütunu (E-44, tüm roller, tüm türler) | Tümü | ✓ | ✓ |
| PM-E-01 | Bilgilendirme e-postası şablonu (ekran değil, içerik) | Veli | — | — |

### 10.2 Ekran ayrıntıları

**PM-A-01 · Veli Toplantıları listesi** — Amaç: sezonun toplantılarını görmek. Öğeler: organizasyon kartı/satırı (başlık, tarih aralığı, şube sayısı, durum özeti "12 yayında · 1 iptal · 3 tamamlandı", yanıt oranı), süzgeç (durum, tarih), "Yeni toplantı". Taslaklar ayrı sekme.

**PM-A-02 · Planlama sihirbazı** — Üç adım. (1) **Şubeler:** kademe/seviye/şube ağacı, "Tüm okul" kısayolu, rehbersiz şube rozeti. (2) **Tarih ve saat:** başlık, açıklama, varsayılan tarih/başlangıç/bitiş/yer; şube tablosu — satır içi düzenleme, çoklu seçimle "seçilenlere uygula". Geçmiş an satır hatası. (3) **Önizleme:** şube, öğrenci, davetli veli sayısı; kanal kırılımı (uygulama/push ulaşacak · e-posta tercihli · hesapsız); çakışan veliler listesi; rehbersiz şubeler. Düğmeler: "Taslak kaydet", "Yayınla" (onay diyaloğu sayılarla). Mobilde adımlar tam ekran, tablo kart listesine döner.

**PM-A-03 · Organizasyon detayı** — Şube durum tablosu: şube, rehber öğretmen, tarih-saat, yer, durum rozeti, Katılacak/Katılamayacak/Yanıt yok, fiili (toplantı sonrası). Satır eylemleri: Aç, Tarih değiştir, İptal. Çoklu seçim + toplu "Tarih değiştir" / "İptal". Üstte "Şube ekle" (BR-PM-06).

**PM-A-04 · Şube toplantısı detayı (idare)** — Sekmeler: **Tümü** (öğrenci → velileri, yanıt rozeti, neden ikonu, "geçmiş" bağlantısı), **Yanıt yok** (telefon kısayolu + "Veli adına işaretle"), **Hesapsız veliler** (telefon + "Veli adına işaretle"), **Yürüt** (Başladı'dan itibaren; PM-T-02 ile aynı). Veli adına işaretle diyaloğu: yanıt, isteğe bağlı neden, isteğe bağlı not ("telefonla görüşüldü"). Yanıt geçmişi yan paneli: tarih, değer, giren, kimin adına, sürüm.

**PM-A-05 · Tarih değiştir / İptal diyaloğu** — Tarih değiştir: yeni tarih/saat/yer; **etki önizlemesi** "X yanıt sıfırlanacak, Y veliye bildirim gidecek, Z yeni çakışma"; onay. Toplu kullanımda şube kırılımı. İptal: isteğe bağlı gerekçe (yalnız idare görür), "N veliye iptal bildirimi gidecek" bilgisi.

**PM-A-06 · Katılım raporu** — §13.

**PM-A-07 · Veli Toplantısı ayarları** — Alanlar: sabah hatırlatma saati (varsayılan 08:00; sessiz saat içindeyse uyarı), başlangıç bildirimi (aç/kapa, varsayılan kapalı), görüşme notu saklama süresi (yıl, varsayılan 2, 0–10). Otomatik kapanış X ve düzenleme penceresi sistem sabiti, ekranda bilgi olarak.

**PM-A-08 · Not denetim kaydı** — Öğrenci/toplantı/kullanıcı/tarih süzgeçli liste: kim, ne zaman, hangi öğrencinin notu, işlem (okudu/oluşturdu/değiştirdi/sistem sildi). Metin içermez.

**PM-T-01 · Toplantılarım** — Rehber öğretmenin şube toplantıları (yaklaşan / geçmiş); kart: şube, tarih-saat, yer, durum, Katılacak/Yanıt yok sayaçları.

**PM-T-02 · Katılım takibi** — Üstte sayaç: "Katılacak 18 · Katılamayacak 4 · Yanıt yok 10 · **Geldi 15**". Liste öğrenci başına; altında veli satırları: beyan rozeti (salt okunur) + ayrı **"Geldi"** tek dokunuş düğmesi. Satıra dokununca not paneli. Öğrencinin not durumu ikonu (not var/yok). Altta sabit "Toplantıyı tamamla". Yayınlandı'da salt okunur yanıt listesi; Tamamlandı'da düzenleme penceresi sayacı ("6 gün daha düzenlenebilir").

**PM-T-03 · Görüşme notu paneli** — Öğrenci adı, şube; çok satırlı metin alanı; kalıcı etiket "Veliye gösterilmez — yalnız sınıf rehber öğretmeni ve idare görür"; otomatik kayıt durumu ("Kaydedildi 18:42" / "Çevrimdışı — bağlanınca kaydedilecek"); son düzenleyen ve zaman. Mobilde alttan tam yükseklik panel; web'de yan panel.

**PM-T-04 · Öğretmen anasayfası** — Toplantı günü kartı: şube, saat, yer, "Katılacak 18/32", "Katılım ekranına git". Bekleyen İşler: "Tamamlanmayı bekleyen toplantı · 5-A".

**PM-P-01 · Toplantılarım (veli)** — Çocuk başına kart: çocuk adı, şube, tarih-saat, yer, yanıt durumu; yanıtsızsa iki büyük düğme ("Katılacağım" / "Katılamayacağım") doğrudan kartta; yanıtlıysa durum + "Yanıtı değiştir". Bantlar: **iptal** ("Bu toplantı iptal edildi"), **değişiklik** ("Saat değişti — yeniden yanıt verin"), **çakışma** ("Aynı saatte başka toplantınız var"). Geçmiş toplantılar ayrı bölüm.

**PM-P-02 · Toplantı ayrıntısı + yanıt** — Kart bilgileri + açıklama; iki büyük düğme; Katılamayacağım'da isteğe bağlı neden alanı ve "Bu açıklamayı yalnız öğretmen ve okul idaresi görür" notu; yanıt sonrası onay ("Yanıtınız kaydedildi"). Başladı'dan sonra yanıt kilitli. Veli diğer velinin yanıtını görmez.

**PM-P-03 · Veli anasayfası** — Toplantı günü kartı (çocuk adı, saat, yer, yanıt durumu); yanıtsızsa düğmeler kartta.

**PM-X-01 · Bildirim tercihleri (E-44)** — Mevcut tür listesine e-posta sütunu; e-posta adresi yoksa sütun pasif + "Profilinize e-posta ekleyin" bilgisi.

**PM-E-01 · Bilgilendirme e-postası** — §9.2 içerik; bağlantı/düğme yok.

---

## 11. İstisnai Durumlar

| Durum | Davranış |
|---|---|
| **Rehber öğretmen yayından sonra değişti** | Sorumlu dinamik (BR-PM-07): yeni rehber öğretmen katılım ekranını ve notları görür, N-02 yalnız sonraki olaylarda ona gider. Eski rehber öğretmenin yazdığı notlara erişimi: öneri — **notu yazan öğretmen + şubenin güncel rehber öğretmeni + idare** (§16 S-4). |
| **Şubenin rehber öğretmeni yok** | Planlanır/yayınlanır, uyarıyla (BR-PM-07); N-02 gitmez; idare yürütür (K-6). |
| **Öğrenci yayından sonra şube değiştirdi** | Yayınlandı'da: eski şube satırları "davet düştü" (yanıt geçmişi kalır); yeni şubenin toplantısı Yayınlandı ise velilere N-01 ile davet. Başladı/Tamamlandı'da: kayıt değişmez; not eski şube toplantısında kalır, yeni şubenin rehber öğretmeni o notu görmez (eski şube toplantısının güncel rehber öğretmeni + idare görür). |
| **Öğrenci okuldan ayrıldı** | Yayınlandı'da davet düşer, bildirim kesilir. Notlar ve katılım kalır; saklama süresi aynen işler (K-12). |
| **Velilik ilişkisi iptal edildi** (`Revoke`) | Yayınlandı'da: veli davetten düşer, kartı kaybolur, sonraki bildirimler gitmez; yanıt geçmişi kalır. Başladı'dan sonra: kayıt raporda kalır, veli artık göremez. Yeni ilişki eklenirse Yayınlandı'da davet + N-01. Not: alıcı çözümü bugün yalnız `RevokedAt`'e bakar, `ValidUntil`'i denetlemez — bitiş tarihi geçmiş ama iptal edilmemiş ilişki davetli sayılır (teknik analizde ele alınmalı). |
| **`CanViewInfo` sonradan kapatıldı** | Yayınlandı'da davet düşer (BR-PM-14), mevcut yanıt geçmişte kalır. |
| **İptal edilen toplantının yanıtları** | Silinmez; arşivde ve raporda "iptal" olarak görünür; veli kartı iptal bandıyla geçmiş bölümünde kalır. |
| **Tarih değişikliği + hemen iptal** | Kuyruktaki N-03 gönderim anında güncel durum İptal olduğu için atılır, yalnız N-04 gider (BR-PM-35). |
| **Sezon arşivlendi** | BR-PM-42. |
| **Aynı veliye çakışan toplantılar** | Uyarı, engel yok (BR-PM-23). |
| **Veli hesabı yok** | V1'de **hiçbir kanal ulaşmaz** (SMS no-op, e-posta hesap ister). İdare "Hesapsız veliler" listesinden telefonla arar ve veli adına işaretler (BR-PM-24, BR-PM-40). Önizleme ve raporda sayı görünür. |
| **Veli hesabı var, e-posta adresi yok** | E-posta atlanır; push + uygulama içi gider. |
| **Veli hesabı var ama hiç oturum açmamış / push izni yok** | Uygulama içi kayıt bekler; ulaşılamama riski idare özetinde "son 30 günde giriş yapmamış veli" sayısıyla görünür (öneri, bilgi amaçlı). |
| **Bugün, saati geçmiş toplantı** | Yayınlanamaz / o saate taşınamaz (BR-PM-04); teyit §16 S-5. |
| **Toplantı anında bağlantı kopması** | "Geldi" işaretleri ve not yerelde tutulur, bağlanınca gönderilir; çakışmada son yazan kazanır, denetimde ikisi de kalır. |
| **İki idareci aynı anda değişiklik yapar** | İkinci kaydeden "kayıt değişti, yenileyin" uyarısı alır (eşzamanlılık denetimi). |
| **Davetli olmayan yakını geldi** | İşaretlenemez; nota yazılır (BR-PM-26). |

---

## 12. KVKK

- **Veri sınıfları:** yanıt ve neden (kişisel veri); fiili katılım (kişisel veri); görüşme notu (kişisel veri, **özel nitelikli veri içerebilir** — sağlık, aile durumu; [6698 sayılı KVKK m.6](https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf)). Not alanı bu nedenle en dar erişim, okuma dahil denetim ve süreli saklamaya tabidir.
- **Erişim:** BR-PM-21, BR-PM-28; veli ve öğrenci not metnine hiçbir yüzeyden erişemez; dışa aktarımda not metni yoktur (rapor yalnız "not var/yok").
- **Denetim:** not okuma/yazma (BR-PM-31), yanıt geçmişi (BR-PM-20), idari işlemler (yayın, tarih, iptal) değiştirilemez kayıt.
- **Saklama ve imha:** not metni K-12/BR-PM-33 ile kalıcı silinir ([KVKK m.7](https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf) silme/yok etme); denetim kaydı metin taşımadığı için kalır. Yanıt/katılım kayıtları sezon arşiviyle platformun genel saklama politikasına tabidir.
- **Bildirim yüzeyi daraltma:** push/kilit ekranı gövdesinde çocuk adı ve toplantı bilgisi yer alır; neden, not, diğer veli bilgisi asla. E-posta yalnız bilgilendirme (K-13).
- **Ayrı yaşayan ebeveyn:** BR-PM-14 (`CanViewInfo`), BR-PM-21.
- **E-posta tercihi:** alıcı seçimi E-44 ile; bilgilendirme amaçlı işlem e-postası. Varsayılan için §16 S-6.
- **Mevzuat notu:** SMS açıldığında ileti türünün (bilgilendirme / ticari) sınıflandırması ve İYS gerekliliği değerlendirilmeli — doğrulanmadı, sonraki faz.

---

## 13. Raporlama

**PM-A-06 · Katılım raporu (idare: okul, rehber öğretmen: kendi şubesi)**

| Görünüm | İçerik |
|---|---|
| Organizasyon özeti | Şube başına: davetli veli, Katılacak, Katılamayacak, Yanıt yok, **Geldi**, Gelmedi; yanıt oranı, fiili katılım oranı. Toplam satırı. İptal edilen şubeler ayrı satır grubu. |
| Beyan × fiili çapraz tablosu | Katılacak→Geldi, Katılacak→Gelmedi, Katılamayacak→Geldi, Yanıt yok→Geldi… dört kutu bağımsız. |
| Kanal ulaşımı | Hesapsız veli sayısı, e-posta adresi olmayan, idare tarafından veli adına işaretlenen yanıt sayısı. |
| Veli listesi (süzgeçli) | Yanıt + fiili + "veli adına" işareti; süzgeç: "Katılacağım dedi ama gelmedi", "Hiç yanıt vermedi". Not metni **yok**, yalnız "not var" işareti. |
| Dışa aktarım | Excel (not metni hariç), ödev raporu emsali. |

Rapor öğretmen bazlı kıyas/sıralama üretmez; öznesi şube ve toplantıdır (ödev K-8 felsefesi).

---

## 14. Kapsam Dışı

| Ne | Nereye | Neden şimdi değil |
|---|---|---|
| Gerçek SMS gönderimi | Sonraki faz | K-14; V1 altyapı (niyet kaydı) |
| Branş öğretmeni katılımı / çok öğretmenli toplantı | Sonraki faz | K-3 |
| E-postadan yanıt (düğme, güvenli onay ekranı) | Yok | K-13 kararı |
| Birebir veli görüşme randevusu (zaman dilimi seçimi) | Değerlendirilmedi | Ayrı ürün davranışı |
| Görüşme notunun veliye paylaşımı / özet çıkarılması | Yok | K-11 |
| Davetli dışı katılımcı kaydı | Yok | BR-PM-26 |
| Çevrim içi (görüntülü) toplantı | Yok | Kapsam dışı |
| Yapay zekâ ile not özeti | Yok | Pilot kapsamı dışı |
| e-Okul aktarımı | Yok | Platform kararı |

---

## 15. Terminoloji (sözlük önerileri)

Aşağıdakiler `oksis-ui/CLAUDE.md` MEB sözlüğüne **önerilir**; onaylanmadan kodlanmaz. Mevcut çakışmalar özellikle önlendi: `attendance` (devamsızlık) ve `rollCall` (yoklama) kullanılmaz; `session` (`academicSession`, `attendanceSession`) kullanılmaz; `contact` "Rehber" (kişi rehberi) ile "sınıf rehber öğretmeni" karıştırılmaz.

| Türkçe terim | Kod adı (öneri) | Not |
|---|---|---|
| Veli Toplantısı (organizasyon) | `parentMeeting` | Rota `/parent-meetings`; backend `ParentMeeting`; izin `parentMeetings.*` |
| Şube Toplantısı | `parentMeetingSection` | Organizasyonun şubeye inmiş kaydı; `section` = şube sözlüğüyle uyumlu |
| Sınıf Rehber Öğretmeni | `homeroomTeacher` | Backend `ClassRoom.HomeroomTeacherId` zaten var; sözlükte yok, eklenmeli |
| Davetli Veli | `meetingInvitee` | `invitee` (davetle gelen kullanıcı) ile karışmasın diye önekli |
| Katılım Yanıtı (beyan) | `meetingResponse` | Değerler `attending` / `notAttending` / `noResponse` |
| Katılamama Nedeni | `declineReason` | |
| Yanıt Geçmişi | `responseHistory` | |
| Veli Adına Yanıt | `onBehalfResponse` | Girenin kimliği `respondedBy`, sahibi `parentPersonId` |
| Plan Sürümü | `scheduleRevision` | Tarih/saat değişince artar |
| Fiili Katılım | `meetingPresence` | Değerler `present` / `absent` / `unmarked`; `attendance` DEĞİL |
| Görüşme Notu | `meetingNote` | |
| Not Denetim Kaydı | `meetingNoteAudit` | `auditTrail` kalıbı |
| Düzenleme Penceresi | `editWindow` | |
| Hesapsız Veli | `parentWithoutAccount` | |
| Durumlar | `draft` / `published` / `started` / `completed` / `cancelled` | Ödev/duyuru durum adlarıyla uyumlu |

---

## 16. Karar Turu 2 — kapandı (S-1…S-6)

> **Faruk, 2026-10-15: altı sorunun tamamında öneri kabul edildi.** Aşağıdaki "Öneri" sütunu kesin hükümdür; belgedeki "§16 S-x" atıfları bu kararlara gider. N-07 V1 kapsamındadır (N = 2 gün). S-6'nın KVKK dayanağı teknik analizde güvenlik denetçisince incelenir.

| # | Soru | Öneri |
|---|---|---|
| **S-1** | Yanıt vermemiş velilere toplantıdan önce "yanıt bekleniyor" hatırlatması (N-07) gitsin mi? Şu an sabah hatırlatması yalnız "Katılacağım" diyenlere. | Evet, toplantıdan **2 gün önce** bir kez, yalnız yanıtsızlara; okul ayarıyla kapatılabilir. Yanıt oranını en çok artıracak tek kaldıraç. |
| **S-2** | Ortaokul/lise öğrencisi velisinin toplantısını salt okunur görsün mü? | Hayır (V1). Toplantı veliyle okul arasındadır; öğrenciye görünürlük neden/not sızıntısı riskini büyütür. İstenirse yalnız tarih/saat kartı, yanıt ve neden olmadan. |
| **S-3** | Otomatik kapanış X gün değeri. | **3 gün** (başlangıç gününden itibaren gece işi). Hafta sonuna denk gelen akşam toplantısında öğretmene pazartesi günü tamamlama fırsatı verir; düzenleme penceresi (7 gün) ayrıca işler. |
| **S-4** | Rehber öğretmen değişince eski notlara erişim. | Notu **yazan öğretmen** + şubenin **güncel** rehber öğretmeni + idare. Yazan öğretmen başka okula/şubeye geçse de kendi notunu saklama süresince görebilir mi — öneri: okuldan ayrılınca erişim biter. |
| **S-5** | "Bugün olabilir" kuralı, saati geçmiş bugünkü toplantıyı kapsıyor mu? | Hayır: kural "geçmiş **an** olamaz" diye yorumlandı (BR-PM-04). Geriye dönük kayıt (toplantı yapıldı, sisteme sonra girilecek) ihtiyacı varsa ayrı bir "geçmiş toplantıyı kaydet" akışı gerekir — V1 dışı önerilir. |
| **S-6** | E-44: alıcının e-posta tercihi satırı yokken varsayılan açık mı kapalı mı? Kod bugün "tercihsiz e-posta yasak" diyerek kanalı kapalı tutuyor. | Tür bazlı katalog varsayılanı (`DefaultEmailEnabled`); veli toplantısı N-01/N-03/N-04 için **açık**, diğer türler mevcut katalogdaki değerle. Alıcı kapatabilir. Aksi hâlde (varsayılan kapalı) pilotta e-posta fiilen hiç gitmez. |

---

## 17. Riskler

| Risk | Etki | Önlem |
|---|---|---|
| Veli uygulamayı kurmamış/hesabı yok; V1'de SMS yok → davetlerin bir kısmı hiç ulaşmaz. | **Yüksek** | Önizleme ve raporda hesapsız sayısı; telefon kısayolu + veli adına işaretle; SMS'in sonraki fazı. |
| E-44 varsayılanı kapalı seçilirse e-posta kanalı pilotta ölü kalır. | **Orta** | S-6 kararı. |
| Öğretmen "Tamamla"yı unutur → raporda Gelmedi'ler gecikir. | Orta | Bekleyen İşler + X=3 gün otomatik kapanış. |
| Görüşme notuna özel nitelikli veri yazılır, yanlış kişiye sızar. | **Yüksek** | En dar erişim, okuma denetimi, kalıcı etiket, dışa aktarımda metin yok, süreli silme. |
| Tarih değişikliği velide "spam" hissi ve yanıt kaybı yaratır. | Orta | Etki önizlemesi (BR-PM-17); tekilleştirme (BR-PM-35); yer değişikliği yanıt sıfırlamaz. |
| `ValidUntil` geçmiş ama iptal edilmemiş velilik ilişkisi davetli sayılır. | Düşük | Teknik analizde alıcı çözümüne `ValidUntil` denetimi değerlendirilmeli (tüm modülleri etkiler). |
| Akşam toplantısı + sessiz saat ayarı çakışması (ör. 21:00 başlangıç bildirimi). | Düşük | BR-PM-39 push atlar; ayar ekranı uyarır. |

---

## 18. Başarı Kriterleri (pilot)

1. İdare tüm okul toplantısını **5 dakikanın altında** planlayıp yayınlıyor.
2. Hesabı olan davetli velilerin **%70'i** toplantıdan önce yanıt veriyor.
3. Öğretmen 30 velilik şubede fiili katılımı **toplantı süresince** işaretliyor; şube toplantılarının **%90'ı** öğretmen tarafından (otomatik değil) tamamlanıyor.
4. Görüşme notlarına veli/öğrenci tarafından erişim **sıfır**; denetim kaydı her okumayı içeriyor.
5. İdare, toplantı akşamı beyan/fiili tablosunu kâğıt liste olmadan görüyor.

---

## 19. Sonraki Adım

1. §16 açık sorularının cevaplanması.
2. Sözlük önerilerinin (§15) `oksis-ui/CLAUDE.md`'ye işlenmesi.
3. **Teknik analiz:** varlık seti (organizasyon, şube toplantısı, davetli, yanıt + geçmiş, fiili katılım, not + denetim, SMS niyeti), durum makinesi, zamanlanmış işler (başlatma, sabah hatırlatma, otomatik kapanış, saklama silme), E-44 tercih modeli ve göç, bildirim türleri ve deterministik olay kimliği, izinler.
4. Tasarım siparişi — §10 envanteri girdi.

---

*OKSİS · Veli Toplantısı Modülü İhtiyaç Analizi (Final) · REQ-043 · 15 Ekim 2026 · Geliştirme başladığında davranışın tek yetkili kaynağı çalışan sözleşme (mock/kod) olacaktır — belge onunla çelişirse kod kazanır.*
