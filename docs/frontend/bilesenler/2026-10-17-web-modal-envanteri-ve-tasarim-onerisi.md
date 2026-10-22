---
tags: [frontend, bilesen, envanter, tasarim]
tarih: 2026-10-17
durum: tasarim-secimi-bekliyor
---

# Web modal envanteri ve ortak bileşen tasarım önerisi

Bu çalışma kullanıcının üretim öncesinde üç varyasyon görme talebine yanıt verir. Üretim kodu değiştirilmedi. Henüz onaylanmış bir üretim tasarımı yoktur.

## Sonuç

Mevcut `apps/web/components/shared/dialog.tsx` ortak temeli vardır; her ekran bu temeli kullanmıyor. `ConfirmDialog` ve `InfoDialog` bu temelden türer. Ayarların `AModal` bileşeni de `Dialog` sarmalayıcısıdır. `WizardSheet` bağımsız bir modal çekmece temelidir. Birçok modül kendi yüzeyini yazmıştır.

Öneri: mevcut `Dialog` geliştirilerek tek modal temeli haline getirilsin. Üç tasarımdan biri seçildikten sonra tüm rollerin mevcut modal işlemleri bu görünüme taşınsın. Üç tasarım, üretimde ekran başına seçilecek üç ayrı kabuk değildir. Rol farkı mevcut rol tema değişkenleriyle taşınır; başlık, boşluklar, kapat düğmesi ve işlem çubuğu aynı kalır.

## Kapsam ve sayım yöntemi

- `apps/web` altındaki 393 TSX dosyası TypeScript sözdizim ağacıyla tarandı. Mobil uygulama kapsam dışı.
- 75 dosyada 94 doğrudan yüzey tanım noktası: 77 `dialog`, 1 `alertdialog`, 14 rolü belirtilmemiş modal kabuğu, 1 `status`, 1 `alert`. Ortak `Dialog` ve `WizardSheet` tanımları da bu sayıya dahildir.
- 117 JSX kabuk çağrısı: 42 ortak klasördeki bileşenlere, 75 özellik içi kabuklara gider. Ortak klasör çağrılarının 17'si `Dialog`, 18'i `ConfirmDialog`, 2'si `InfoDialog`, 5'i `WizardSheet`'tir.
- Toplam 111 dosyada 211 envanter kaydı. **211 farklı modal değildir.** Bir kabuk tanımı ve tüketicileri farklı kayıtlardır. Koşullu JSX durumları da ayrı kaynak noktalarıdır. Sayılar toplanarak benzersiz kullanıcı akışı sayısı çıkarılamaz.
- `GradeModal` içindeki dış `role=dialog` ile iç `.att-modal` aynı yüzey olduğundan bir kez sayıldı.
- Yalnız dosya adına bakılmadı: `role`, `aria-modal`, modal sınıfları, scrim/overlay, gömülü JSX ve `createPortal` kullanımları karşılaştırıldı. Satır menüleri, filtre listeleri ve tooltip'ler modal sayılmadı. Kabuklar geçişli keşfedildi; örneğin `DialogShell` → `ExamModal` zincirindeki 9 çağrı da dahil edildi.
- Bildirim zili popover'ı ve ailenin not detay paneli karşılaştırma kapsamındadır; mevcut modal olmayan davranışları korunmalıdır.
- Çalışma statik kaynak envanteridir. Çalışma anındaki ulaşılabilirlik, sunucu yetkisi ve tüm rol oturumlarında uçtan uca doğrulama yapıldığı anlamına gelmez.

## Modül dağılımı

| Modül | Doğrudan yüzey tanımı | Ortak klasör kabuk çağrısı | Özellik kabuğu çağrısı |
|---|---:|---:|---:|
| Sezon yönetimi | 3 | 1 | 7 |
| Etkinlikler | 2 | 0 | 5 |
| Duyurular | 3 | 0 | 10 |
| Devamsızlık | 9 | 4 | 3 |
| Kulüpler | 6 | 5 | 0 |
| Müfredat | 0 | 11 | 0 |
| Nöbet ve vekâlet | 8 | 2 | 0 |
| Sınav takvimi | 4 | 1 | 16 |
| Notlar ve karne | 2 | 1 | 9 |
| Ödevler | 12 | 0 | 7 |
| Davetler | 2 | 0 | 0 |
| Bildirimler | 1 | 0 | 0 |
| Veliler | 2 | 0 | 0 |
| Platform yönetimi | 3 | 0 | 0 |
| Raporlar | 1 | 0 | 0 |
| Öğretmen yoklaması | 3 | 1 | 0 |
| Ders programı | 16 | 1 | 0 |
| İdari kadro | 2 | 0 | 0 |
| Sınıflar ve şubeler | 1 | 0 | 8 |
| Okul ayarları | 1 | 7 | 10 |
| Ortak bileşenler | 2 | 2 | 0 |
| Öğrenciler | 2 | 1 | 0 |
| Öğretmen görevlendirmeleri | 2 | 3 | 0 |
| Öğretmenler | 4 | 0 | 0 |
| Kullanıcılar | 3 | 2 | 0 |

## Dağınıklığın somut örnekleri

1. `features/homework/homework-dialogs.tsx` içindeki `Dialog`, adı aynı olmasına rağmen ortak `Dialog` değildir. Ödev modülünün kendine ait modal kabuğudur; 7 JSX çağrısı bulundu.
2. `SznModalShell`, `ActModalShell`, `DuyModalShell`, `ExcModal`, `ExamModal`, `GradeModal`, şubelerin `Shell` bileşeni, sınavların `DialogShell` sarmalayıcısı ve ayarların `ADrawer` kabuğu bağımsız temeller veya yerel sarmalayıcılardır.
3. Sezon, davet, veli bağlama, öğrenci yaşam döngüsü, öğretmen ve kullanıcı işlemleri arasında `role` belirtilmemiş modal kabukları vardır. `.att-modal` stilini paylaşmaları ortak React davranışı kullandıkları anlamına gelmez.
4. Bazı detay çekmecelerinde `role=dialog` varken `aria-modal` belirtilmemiştir; yüzey arka sayfayı kilitliyorsa semantik ve davranış birlikte düzeltilmelidir.
5. Ortak `Dialog` odak döndürme, Tab çevrimi ve kapatma kilidi içeriyor. Fakat merkezi katman sırası, arka planı inert yapma ve gövde kaydırmasını kilitleme kodu bu dosyada yok. Bunlar üretim tasarımının ortak sorumluluğu olmalıdır; mevcut olmayan davranış varmış gibi kabul edilmemelidir.
6. Platform okul düzenlemesinin yükleme/hata durumları ayrı `pf-modal` yüzeyleridir. Form, yükleme ve hata aynı kabukta durum değiştirmelidir.
7. Önizlemeler, yazdırılabilir devamsızlık yazıları, belge görüntüleyicileri, portalda açılan seçiciler ve iç içe modal/çekmece akışları geçişin özel senaryolarıdır.

## Rol kapsamı

| Rol | Kaynaktan görülen yüzler ve geçiş odağı |
|---|---|
| Yönetici | Öğrenci/öğretmen/veli/kullanıcı yönetimi, sezon, şube, müfredat, görevlendirme, ders programı, devamsızlık, sınav, duyuru, nöbet, kulüp ve okul ayarları; ayrıca not/ödev idare ekranları. |
| Öğretmen | Yoklama onayı ve düzeltme talebi; not yayınlama/düzeltme; ödev yayınlama, kapatma, tarih uzatma ve teslim önizleme; kendine açık program, sınav, nöbet, kulüp ve duyuru işlemleri. Yetki ve görev kontrolleri korunacak. |
| Öğrenci / Veli | Mevcut aile not ekranı ve bildirim gibi ortak yüzeylerde aynı temel kullanılmalı. Aile not detay paneli bilinçli olarak modal değildir. `HomeworkPage` öğrenci ve veli için halen `PlannedScreen` döndürüyor: tasarım örneği gösterilmesi bu ekranlar var demek değildir. Envanter kaydını role açık işlev saymak yanlış olur. |
| Platform yöneticisi | Dört okul portalından ayrı yüzdür; okul sihirbazı ile yükleme/hata yüzeyleri de görsel ve davranış tutarlılığı kapsamına alındı. |

Rol haritası rota ve bileşen dallarından okundu; her envanter satırı için sunucu izin matrisi çıkarılmadı. Geçiş sırasında her akışın gerçek rol/izin kontrolü ayrıca korunmalı ve doğrulanmalı.

## Tasarım standardı

Kaynak önceliği: kodda `packages/core/src/tokens/brand.ts`, `packages/core/src/roles/roles.ts`, `packages/ui/src/styles/shell.css`; belge merkezindeki `frontend/tasarim-sistemi/marka-tokenlari.md` ve `apps/web/CLAUDE.md`.

- Yazı: uygulamanın Plus Jakarta Sans fontu. Önizleme yereldeki aynı font dosyalarını kullanır; dış font isteği yok.
- Ana palet: lacivert `#1B2B5E`, indigo `#3A4F9C`, vurgu `#4F6BFF`, yüzey `#EEF1FA`, mürekkep `#0D1117`, sınır `#DDE3F1`.
- Başlık 24/28, gövde 17/26, alan etiketi 12/18. Modal köşesi 20px, alan ve düğme 8px; mevcut token karşılıkları kullanılır.
- Rol teması: yönetici `#1B2B5E`, öğretmen ana renk `#0E4D36`, öğrenci `#3B0764`, veli ana renk `#7A3A06`. Rol vurgu ve tint değerleri `WEB_ROLE_THEME` ile aynıdır.
- Tehlike rengi rolden bağımsız `#DC2626`; kırmızı tonu yalnız kritik işlem ve hataya ayrılır.
- Kapatma düğmesi sağ üstte; başlıkla çakışmaz. Vazgeç solda, açık eylem adını taşıyan birincil düğme sağda.
- Formlarda dışarı tıklama varsayılan kapatma yolu değildir. Değişiklik varsa Escape ve kapatma düğmesi de aynı kayıp kontrolünden geçer. Kayıt sürerken yinelenen işlem ve yanlışlıkla kapanma önlenir.
- Uzun içerikte başlık ve işlem çubuğu sabit, yalnız gövde kaydırılır. 320px genişlikten itibaren kontrol edilmeli. C varyasyonundaki yan bağlam küçük ekranda üst bölüme taşınır.

Canlı marka sitesi bu oturumda web aracından açılamadı. Uyum yerel marka kaynaklarıyla değerlendirildi; sitenin güncel sürümüyle birebir teyit edildiği iddia edilmiyor. Belgelerdeki eski «yalnız admin portalı» notu, kodda dört rol temasının zaten bulunması karşısında dört rolü dışlamak için kullanılmadı.

## Üç aday

| Aday | Görünüm | Güçlü yanı | Bedeli |
|---|---|---|---|
| A · Sade — önerilen | Açık başlık, küçük bağlam kutusu, beyaz gövde, belirgin tek eylem. | Küçük onay, bilgi ve kısa formda dengeli; mevcut arayüze en az sürtünmeyle oturur. | Çok karmaşık işlerde tek sütun uzayabilir; bu durumda ayrı sayfa tercih edilmeli. |
| B · Renkli başlık | Rol renginde dolu başlık, beyaz içerik. | İşlem alanını ve portal kimliğini açıkça ayırır. | Sık açılan küçük onaylarda daha baskın; aynı anda çok renkli ekranlarda ağırlaşabilir. |
| C · Bölünmüş | Solda şube/sezon/kapsam, sağda işlem formu. | Bağlama bakarak karar verilen geniş içerikte güçlü. | Küçük onaylarda gereksiz alan kullanır; mobilde tek sütuna inmesi gerekir. |

Üçünde de örnek yönetici işlemi aynıdır: «Rehber öğretmen ata». Rol anahtarı yetki simülasyonu değildir, yalnız temayı değiştirir. Kritik işlem, bilgi ve uzun form örnekleri de aynı kabukta denenebilir. Örnekler sunucuya bağlı değildir.

## Tek ortak bileşenin sözleşmesi — öneri

`Dialog` konum, ölçü, başlık/yardımcı metin, semantik ton, görünürlük, kapatma isteği, kaydetme durumu, gövde ve işlem çubuğunu yönetir. `ConfirmDialog` ve `InfoDialog` yalnız içerik/eylem kalıbıdır; kendi overlay, odak veya Escape kodunu yazmaz. Gerekirse form kalıbı eklenir; yeni bağımsız modal motoru oluşturulmaz.

- Modal davranışın tek sahibi: odak alma/geri verme, odak çevrimi, arka plan inertliği, kaydırma kilidi, Escape, scrim, portal hedefi ve katman yönetimi.
- Dışarı tıklama ile kapatma içerik türüne göre merkezi politika olmalı; feature başına rastgele davranış olmamalı.
- Açılır listelerin portalı, rol teması ve üstteki katmanın Escape olayı birlikte ele alınmalı. Çocuk menü kapanırken ana modal da kapanmamalı.
- İç içe modal gerekiyorsa yalnız en üst yüzey Escape ve odağı yönetir; mümkünse doğrulama aynı yüzeyde bir sonraki durum olarak gösterilir.
- Modal çekmece gerekiyorsa aynı davranış temeli üzerinde sağ yerleşim kullanılabilir. Ekranı kilitlemeyen detay paneli bu modal temelinin dışında kalabilir.
- Durumlar: normal, yükleniyor, kaydediliyor, hata, tamamlandı. Hata formu kapatmamalı; girilen veriler korunmalı.
- Erişilebilir ad görünür başlığa bağlanır. Kritik kısa uyarıda uygun olduğunda `alertdialog` kullanılır. İlk odak kritik işlemde güvenli eylemdedir.
- Veri, doğrulama, rol/izin ve iş kuralları feature/core/api katmanlarında kalır. Ortak `Dialog` öğrenci, not veya duyuru iş kuralını bilmez.

Davranış için birincil kaynak: [W3C modal dialog örüntüsü](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/). Özellikle arka sayfayı kilitleme, odak çevrimi, Escape ve odak geri dönüşü birlikte değerlendirilmelidir.

## Modal dışındaki alternatifler

| Alternatif | OKSİS'te aday ihtiyaç | Kazanç | Sınır / önkoşul |
|---|---|---|---|
| Sayfaya bağlı, kilitlemeyen yan panel | Öğrenci/öğretmen/veli detayı, ders detayı, denetim izi. | Liste ile detayı birlikte kullanma; daha az aç/kapat. | Scrim ve sayfa kilidi varsa bu yine modal olur. Dar ekranda ayrı detay sayfasına yönlenebilir. |
| Yerinde düzenleme | Şube rehber öğretmeni, tek not hücresi, kısa ayar. | Tek alan için sayfa bağlamını kaybetmeme. | Çok alanlı, çapraz doğrulamalı veya kritik işlemde uygun değil; kaydet/vazgeç ve hata ilgili satırda olmalı. |
| Ayrı sayfa / adreslenebilir çok adımlı akış | Öğrenci kaydı, toplu davet, sezon aktifleştirme, kapsamlı program işlemi. | Alan, tarayıcı geri tuşu, bağlantı paylaşımı, uzun form için daha rahat çalışma. | Taslak saklama ayrı bir iş kararıdır; bu öneri otomatik kayıt sözü vermez. |
| Düğmeye bağlı küçük panel | Ders saati, tarih veya birkaç seçenek. | Kısa ve doğrudan etkileşim. | Uzun form, kritik onay veya büyük liste yüklenmemeli. |
| Geri alınabilir işlem + bildirim | Gerçekten geri çevrilebilen küçük tercihler. | Her küçük işte «emin misiniz» kesintisini azaltma. | API'nin gerçek geri alma işlemini desteklemesi gerekir. Sezon, yayın ve not işlemlerinde varsayılmaz. |
| Alan içinde yardım / hata / açılır açıklama | Gerekçe koşulu, tarih sınırı, kısa alan yardımı. | Bilgi ihtiyaç duyulan yerde kalır. | Uzun politika bilgisi mevcut `InfoDialog` içinde kalabilir. Kalıcı genel bilgi kartı yasağı bu öneriyle aşılmıyor. |

Önerilen dağılım: kısa form ve kritik onay ortak modal; salt detay kilitlemeyen yan panel; uzun iş ayrı sayfa; küçük güncelleme yerinde düzenleme. Alternatifler bir UX önerisidir; topluca tüm modalları kaldırma kararı verilmiş değildir.

## Üretime geçiş sırası — seçim sonrası

1. A/B/C'den bir tasarım seçilir; varsayılan görünüm, rol teması ve kapatma politikası kesinleştirilir.
2. Mevcut `Dialog` tek temel olacak şekilde iyileştirilir. `ConfirmDialog`, `InfoDialog`, `AModal` aynı temele bağlı kalır. Bağımsız kabuk açılmaz.
3. İlk doğrulama dilimi: bir kritik onay, bir kısa form, bir uzun kaydırmalı form ve portalda açılan seçim kontrolü. Dört rol temasıyla görsel ve klavye denetimi.
4. Önce yüksek tekrar sağlayan yerel kabuklar (`SznModalShell`, `ActModalShell`, `DuyModalShell`, `ExcModal`, `ExamModal`, `GradeModal`, ödev `Dialog`, şube `Shell`) taşınır; sonra gömülü tekil yüzeyler.
5. Modal çekmeceler, `WizardSheet`, belge/görsel önizlemeleri ve yazdırılabilir yazılar ayrı dilimde aynı temel davranışa bağlanır. Yazdırma stilleri ve büyük içerik kaydırması korunur.
6. Ekranı kilitlemeyen detaylar, popover'lar ve inline alternatifler ihtiyaç bazında ele alınır. Yüzeyleri aynı görünüm uğruna gereksiz modal yapmak yok.
7. Taşınan eski CSS/overlay/Escape kopyaları silinir. Yeni feature-local modal kabuğu eklenmesini denetleyen statik kontrol eklenir; kalan bilinçli istisnalar açıkça listelenir.
8. Her gerçek akışta rol ve izin, kayıt/hata, kaybolan form, Escape, Tab/Shift+Tab, açılır menü, odak dönüşü, dar ekran, uzun içerik ve yazdırma senaryoları doğrulanır; etkilenen lint/typecheck/test denetimleri çalıştırılır.

## Çıktılar ve doğrulama

Önizleme ve envanter dosyaları sohbetin yerel görselleştirme klasöründe, bu belge kod içermeden belge merkezinde tutulur. Üretim dosyaları değiştirilmemiştir. Doğrulama kaydı:

- 111 dosyadaki 211 kayıt JSON, CSV, arama görünümü ve rapor arasında karşılaştırıldı. `DialogShell` araması kabuk içindeki bir temel çağrısını ve 9 tüketici çağrısını gösteriyor.
- Bağımsız builder incelemesi yapıldı. Dolaylı sınav kabuğu ve art arda Escape ile düzenlemeye dönüş bulguları giderildi; ikinci kontrolde önemli açık bildirilmedi.
- Yerel önizlemede üç tasarım × dört içerik türü olmak üzere 12 aç/kapat senaryosu kontrol edildi. Hepsinde pencere görünür alana sığıyor ve alt işlem çubuğu görünür. Uzun içerik üç tasarımda da gövde içinde kayıyor.
- Dört rolün mevcut ana renkleri görünüm üzerinden doğrulandı; gerçek rol oturumu veya sunucu yetkisi testi yapılmadı.
- Boş öğretmen alanıyla gönderme görünür hata üretiyor. Öğretmen seçimi → Escape → kayıp uyarısı → ikinci Escape / düzenlemeye dön akışında seçim korunuyor. Kaydetmeden kapat yolu formu kapatıyor.
- İşleniyor durumunda modal düğmeleri kilitli; önizleme bittiğinde odak açan düğmeye dönüyor. Tab/Shift+Tab sınırlarında odak modal içinde kalıyor.
- 320×720 ve 390×843 CSS piksel görünür alanlarda uzun form kontrol edildi: yatay taşma yok, işlem çubuğu görünür. C tasarımının yan bağlamı küçük ekranda tek sütuna düşüyor. Geçici ekran boyutu ayarı geri alındı.
- Tarayıcı konsolunda hata/uyarı kaydı yok. Önizleme JavaScript sözdizimi kontrolünden geçti.
- Üretim dosyaları değişmediğinden uygulama test paketi, lint/typecheck ve tüm ekranların canlı rol testleri çalıştırılmadı. Üretime geçiş doğrulaması bu çalışmanın tamamlandı iddiasına dahil değildir.

Yerel çıktılar: [tasarım önizlemesi](http://127.0.0.1:8764/), [aranabilir envanter](http://127.0.0.1:8764/inventory.html). Yerel sunucu kapandığında HTML, CSS, font ve veri dosyaları sohbetin `modal-study` klasöründe korunur.

## Dosya ve bileşen bazında eksiksiz kaynak listesi

Aşağıdaki liste taramanın kapsamındaki tüm tanım ve kabuk çağrılarını içerir. `raw` panel tanımı, `shared-use` ortak klasör bileşen çağrısı, `local-use` özellik kabuğu çağrısıdır. Aynı işlevin koşullu render noktaları ayrı satırlardır.

### Sezon yönetimi

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `AcademicSessionsPage` → `div` | Yüzey tanımı | `apps/web/features/academic-sessions/academic-sessions-page.tsx:383` | `Rol belirtilmemiş` |
| `AcademicSessionsPage` → `div` | Yüzey tanımı | `apps/web/features/academic-sessions/academic-sessions-page.tsx:444` | `Rol belirtilmemiş` |
| `SznModalShell` → `div` | Yüzey tanımı | `apps/web/features/academic-sessions/modals.tsx:26` | `Rol belirtilmemiş` |
| `SznActivateModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:90` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznActivateModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:119` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznReopenModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:252` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznDeleteDraftModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:312` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznDeleteSetupModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:375` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznDiscardModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:437` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznPendingBlockModal` → `SznModalShell` | Özellik kabuğu çağrısı | `apps/web/features/academic-sessions/modals.tsx:493` | `apps/web/features/academic-sessions/modals.tsx#SznModalShell` |
| `SznRenewalSheet` → `WizardSheet` | Ortak kabuk çağrısı | `apps/web/features/academic-sessions/renewal-sheet.tsx:164` | `apps/web/components/shared/wizard-sheet.tsx#WizardSheet` |

### Etkinlikler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ActTeacherModal` → `ActModalShell` | Özellik kabuğu çağrısı | `apps/web/features/activities/assign-modals.tsx:76` | `apps/web/features/activities/modals.tsx#ActModalShell` |
| `ActStudentsModal` → `ActModalShell` | Özellik kabuğu çağrısı | `apps/web/features/activities/assign-modals.tsx:303` | `apps/web/features/activities/modals.tsx#ActModalShell` |
| `ActDrawer` → `aside` | Yüzey tanımı | `apps/web/features/activities/drawer.tsx:306` | `dialog` |
| `ActModalShell` → `div` | Yüzey tanımı | `apps/web/features/activities/modals.tsx:37` | `dialog` |
| `ActInfoModal` → `ActModalShell` | Özellik kabuğu çağrısı | `apps/web/features/activities/modals.tsx:106` | `apps/web/features/activities/modals.tsx#ActModalShell` |
| `ActEditModal` → `ActModalShell` | Özellik kabuğu çağrısı | `apps/web/features/activities/modals.tsx:181` | `apps/web/features/activities/modals.tsx#ActModalShell` |
| `ActCancelModal` → `ActModalShell` | Özellik kabuğu çağrısı | `apps/web/features/activities/modals.tsx:262` | `apps/web/features/activities/modals.tsx#ActModalShell` |

### Duyurular

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ImageZoom` → `div` | Yüzey tanımı | `apps/web/features/announcements/attachment-image-preview.tsx:107` | `dialog` |
| `AuditDrawer` → `aside` | Yüzey tanımı | `apps/web/features/announcements/audit-drawer.tsx:68` | `dialog` |
| `DuyModalShell` → `div` | Yüzey tanımı | `apps/web/features/announcements/modals.tsx:51` | `dialog` |
| `WithdrawModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:109` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `RestoreModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:177` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `PublishModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:247` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `MeaningModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:341` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `PreviewModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:418` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `RejectModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:533` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `TemplateFormModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:683` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `TemplateDeleteModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:877` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `TemplateViewModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:950` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |
| `SaveAsTemplateModal` → `DuyModalShell` | Özellik kabuğu çağrısı | `apps/web/features/announcements/modals.tsx:1034` | `apps/web/features/announcements/modals.tsx#DuyModalShell` |

### Devamsızlık

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `CreateExcuseModal` → `ExcModal` | Özellik kabuğu çağrısı | `apps/web/features/attendance/create-excuse-modal.tsx:93` | `apps/web/features/attendance/parts.tsx#ExcModal` |
| `CreateExcuseModal` → `ExcModal` | Özellik kabuğu çağrısı | `apps/web/features/attendance/create-excuse-modal.tsx:133` | `apps/web/features/attendance/parts.tsx#ExcModal` |
| `DayLeaveModal` → `ExcModal` | Özellik kabuğu çağrısı | `apps/web/features/attendance/day-leave-modal.tsx:145` | `apps/web/features/attendance/parts.tsx#ExcModal` |
| `DayLeaveModal` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/attendance/day-leave-modal.tsx:277` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `DocZoom` → `div` | Yüzey tanımı | `apps/web/features/attendance/doc-zoom.tsx:17` | `dialog` |
| `EventWizard` → `WizardSheet` | Ortak kabuk çağrısı | `apps/web/features/attendance/event-wizard.tsx:321` | `apps/web/components/shared/wizard-sheet.tsx#WizardSheet` |
| `ExcuseSheet` → `aside` | Yüzey tanımı | `apps/web/features/attendance/excuse-sheet.tsx:54` | `dialog` |
| `KarneDaySheet` → `aside` | Yüzey tanımı | `apps/web/features/attendance/karne-day-sheet.tsx:60` | `dialog` |
| `AttMissingList` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/attendance/missing-list.tsx:580` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `ExcModal` → `div` | Yüzey tanımı | `apps/web/features/attendance/parts.tsx:358` | `dialog` |
| `ReopenDecisionDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/attendance/reopen-decision-dialog.tsx:62` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `AttRetroModal` → `div` | Yüzey tanımı | `apps/web/features/attendance/retro-modal.tsx:169` | `dialog` |
| `SessionSheet` → `aside` | Yüzey tanımı | `apps/web/features/attendance/session-sheet.tsx:63` | `dialog` |
| `ThresholdLetter` → `div` | Yüzey tanımı | `apps/web/features/attendance/threshold-letter.tsx:93` | `dialog` |
| `ThresholdLetter` → `div` | Yüzey tanımı | `apps/web/features/attendance/threshold-letter.tsx:114` | `dialog` |
| `ThresholdLetter` → `div` | Yüzey tanımı | `apps/web/features/attendance/threshold-letter.tsx:187` | `dialog` |

### Kulüpler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ClubActivityPublishDialog` → `div` | Yüzey tanımı | `apps/web/features/club/activity-dialogs.tsx:53` | `dialog` |
| `ClubActivityCancelDialog` → `div` | Yüzey tanımı | `apps/web/features/club/activity-dialogs.tsx:172` | `dialog` |
| `ClubAnnouncementDrawer` → `aside` | Yüzey tanımı | `apps/web/features/club/announcement-drawer.tsx:80` | `dialog` |
| `ClubFormDrawer` → `WizardSheet` | Ortak kabuk çağrısı | `apps/web/features/club/form-drawer.tsx:316` | `apps/web/components/shared/wizard-sheet.tsx#WizardSheet` |
| `ClubApplicationRejectDialog` → `div` | Yüzey tanımı | `apps/web/features/club/panel-page.tsx:580` | `dialog` |
| `AutoPlaceDrawer` → `aside` | Yüzey tanımı | `apps/web/features/club/placement-board-dialogs.tsx:114` | `dialog` |
| `PublishDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/club/placement-board-dialogs.tsx:332` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `PlacementBoard` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/club/placement-board-page.tsx:711` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `PlacementBoard` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/club/placement-board-page.tsx:795` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `RoundCard` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/club/placement-page.tsx:563` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `ClubStatusDialog` → `div` | Yüzey tanımı | `apps/web/features/club/status-dialog.tsx:127` | `dialog` |

### Müfredat

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ProgramDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:57` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `RebaseDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:188` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `RebaseDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:202` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `RebaseDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:219` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `ResetDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:252` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `ActivationDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:333` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `RebaseResultDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:387` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `ElectivesDeviationDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:439` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `CommonDeviationDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:506` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `AddProfileDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:583` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `DeleteProfileDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/curriculum/modals.tsx:635` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |

### Nöbet ve vekâlet

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `AutoDistributeDrawer` → `aside` | Yüzey tanımı | `apps/web/features/duty/auto-distribute-drawer.tsx:116` | `dialog` |
| `DutyAdminScreen` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/duty/duty-page.tsx:321` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `ExemptModal` → `div` | Yüzey tanımı | `apps/web/features/duty/exempt-modal.tsx:55` | `dialog` |
| `DutyInfoDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/duty/parts.tsx:59` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `CellPopover` → `div` | Yüzey tanımı | `apps/web/features/duty/popover.tsx:127` | `dialog` |
| `PublishModal` → `div` | Yüzey tanımı | `apps/web/features/duty/publish-modal.tsx:105` | `dialog` |
| `RegionModal` → `div` | Yüzey tanımı | `apps/web/features/duty/region-modal.tsx:51` | `dialog` |
| `DeleteRegionConfirm` → `div` | Yüzey tanımı | `apps/web/features/duty/region-modal.tsx:191` | `dialog` |
| `TeacherPreviewDrawer` → `aside` | Yüzey tanımı | `apps/web/features/duty/teacher-preview-drawer.tsx:101` | `dialog` |
| `VersionDrawer` → `aside` | Yüzey tanımı | `apps/web/features/duty/version-drawer.tsx:28` | `dialog` |

### Sınav takvimi

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ExamAdminSessionDialog` → `WizardSheet` | Ortak kabuk çağrısı | `apps/web/features/exam/exam-admin-session-dialog.tsx:199` | `apps/web/components/shared/wizard-sheet.tsx#WizardSheet` |
| `ExamBoardInfoModal` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-board-screen.tsx:195` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |
| `ExamMoveDialog` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-board-screen.tsx:295` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |
| `ExamDeclineDialog` → `div` | Yüzey tanımı | `apps/web/features/exam/exam-hour-requests-screen.tsx:202` | `dialog` |
| `DialogShell` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:108` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |
| `ExamInvigilatorDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:169` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamAddRoomDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:289` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamRemoveRoomDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:374` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamSwapDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:452` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamRegenerateDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:545` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamMergeDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:665` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamSessionInfoDialog` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:749` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |
| `ExamResolveCommentDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:843` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamMoveSessionDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:956` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamSessionSectionsDialog` → `DialogShell` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-session-dialogs.tsx:1074` | `apps/web/features/exam/exam-session-dialogs.tsx#DialogShell` |
| `ExamSessionSlotPicker` → `div` | Yüzey tanımı | `apps/web/features/exam/exam-session-slot-picker.tsx:81` | `dialog` |
| `ExamSlotPicker` → `div` | Yüzey tanımı | `apps/web/features/exam/exam-slot-picker.tsx:239` | `dialog` |
| `ExamModal` → `div` | Yüzey tanımı | `apps/web/features/exam/exam-window-modals.tsx:81` | `dialog` |
| `ExamWindowCreateModal` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-window-modals.tsx:204` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |
| `ExamPublishWindowModal` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-window-modals.tsx:383` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |
| `ExamPublishScheduleModal` → `ExamModal` | Özellik kabuğu çağrısı | `apps/web/features/exam/exam-window-modals.tsx:517` | `apps/web/features/exam/exam-window-modals.tsx#ExamModal` |

### Notlar ve karne

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `GradeUnpublishDialog` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-admin-dialogs.tsx:152` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeLockDialog` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-admin-dialogs.tsx:213` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradePublishForDialog` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-admin-dialogs.tsx:280` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeBulkReminderDialog` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-admin-parts.tsx:555` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeCourseDetailPanel` → `aside` | Yüzey tanımı | `apps/web/features/grade/grade-course-detail-panel.tsx:78` | `dialog` |
| `GradeModal` → `div` | Yüzey tanımı | `apps/web/features/grade/grade-dialogs.tsx:62` | `dialog` |
| `GradePublishDialog` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-dialogs.tsx:113` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeCorrectionDialog` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-dialogs.tsx:256` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeGridScreen` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-grid-screen.tsx:1039` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeGridScreen` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-grid-screen.tsx:1165` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradeGridScreen` → `GradeModal` | Özellik kabuğu çağrısı | `apps/web/features/grade/grade-grid-screen.tsx:1213` | `apps/web/features/grade/grade-dialogs.tsx#GradeModal` |
| `GradePolicyModal` → `InfoDialog` | Ortak kabuk çağrısı | `apps/web/features/grade/parts.tsx:214` | `apps/web/components/shared/info-dialog.tsx#InfoDialog` |

### Ödevler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `AttachmentListDialog` → `div` | Yüzey tanımı | `apps/web/features/homework/attachment-viewer.tsx:177` | `dialog` |
| `AttachmentViewerDialog` → `div` | Yüzey tanımı | `apps/web/features/homework/attachment-viewer.tsx:311` | `dialog` |
| `CompletionStudentSheet` → `aside` | Yüzey tanımı | `apps/web/features/homework/completion-student-sheet.tsx:114` | `dialog` |
| `TransferResponsibleDialog` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-admin-screen.tsx:183` | `dialog` |
| `AdminDetail` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-admin-screen.tsx:1009` | `dialog` |
| `AdminDetail` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-admin-screen.tsx:1066` | `dialog` |
| `AdminDetail` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-admin-screen.tsx:1146` | `dialog` |
| `CellSheet` → `aside` | Yüzey tanımı | `apps/web/features/homework/homework-board-screen.tsx:243` | `dialog` |
| `HomeworkCreateScreen` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-create-screen.tsx:522` | `dialog` |
| `HomeworkCreateScreen` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-create-screen.tsx:708` | `alertdialog` |
| `Dialog` → `div` | Yüzey tanımı | `apps/web/features/homework/homework-dialogs.tsx:53` | `dialog` |
| `BulkCompleteDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:95` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `CancelHomeworkDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:125` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `CloseHomeworkDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:219` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `CloseHomeworkDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:236` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `CloseHomeworkDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:255` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `ExtendDueDateDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:323` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `PublishDraftDialog` → `Dialog` | Özellik kabuğu çağrısı | `apps/web/features/homework/homework-dialogs.tsx:407` | `apps/web/features/homework/homework-dialogs.tsx#Dialog` |
| `SubmissionViewerDialog` → `div` | Yüzey tanımı | `apps/web/features/homework/submission-viewer-dialog.tsx:69` | `dialog` |

### Davetler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `DvtCancelModal` → `div` | Yüzey tanımı | `apps/web/features/invitations/modals.tsx:28` | `Rol belirtilmemiş` |
| `DvtWizard` → `div` | Yüzey tanımı | `apps/web/features/invitations/wizard.tsx:239` | `Rol belirtilmemiş` |

### Bildirimler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `NotificationBell` → `div` | Yüzey tanımı | `apps/web/features/notifications/notification-bell.tsx:143` | `dialog` |

### Veliler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `VelDrawer` → `aside` | Yüzey tanımı | `apps/web/features/parents/drawer.tsx:41` | `dialog` |
| `VelLinkModal` → `div` | Yüzey tanımı | `apps/web/features/parents/modals.tsx:76` | `Rol belirtilmemiş` |

### Platform yönetimi

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `PlatformSchoolsPage` → `div` | Yüzey tanımı | `apps/web/features/platform/platform-schools-page.tsx:230` | `status` |
| `PlatformSchoolsPage` → `div` | Yüzey tanımı | `apps/web/features/platform/platform-schools-page.tsx:238` | `alert` |
| `SchoolWizard` → `div` | Yüzey tanımı | `apps/web/features/platform/school-wizard.tsx:236` | `dialog` |

### Raporlar

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `DonemDrillSheet` → `aside` | Yüzey tanımı | `apps/web/features/reports/drill-sheet.tsx:49` | `dialog` |

### Öğretmen yoklaması

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `TaAmendmentModal` → `div` | Yüzey tanımı | `apps/web/features/roll-call/amendment-modal.tsx:66` | `dialog` |
| `TaConfirmModal` → `div` | Yüzey tanımı | `apps/web/features/roll-call/confirm-modal.tsx:28` | `dialog` |
| `TaReopenRequestDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/roll-call/reopen-request-dialog.tsx:57` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `TaRoster` → `div` | Yüzey tanımı | `apps/web/features/roll-call/roster.tsx:692` | `dialog` |

### Ders programı

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `AutoGenDrawer` → `div` | Yüzey tanımı | `apps/web/features/schedule/autogen-drawer.tsx:225` | `dialog` |
| `BulkPublishDrawer` → `div` | Yüzey tanımı | `apps/web/features/schedule/bulk-publish-drawer.tsx:171` | `dialog` |
| `ConstraintModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/constraints-page.tsx:190` | `Rol belirtilmemiş` |
| `EditorPage` → `div` | Yüzey tanımı | `apps/web/features/schedule/editor-page.tsx:1837` | `dialog` |
| `EditorPage` → `div` | Yüzey tanımı | `apps/web/features/schedule/editor-page.tsx:1892` | `dialog` |
| `EditorPage` → `div` | Yüzey tanımı | `apps/web/features/schedule/editor-page.tsx:1966` | `dialog` |
| `VekilModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/exception-modals.tsx:153` | `dialog` |
| `RoomChangeModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/exception-modals.tsx:373` | `dialog` |
| `IptalModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/exception-modals.tsx:580` | `dialog` |
| `ExceptionsDrawer` → `div` | Yüzey tanımı | `apps/web/features/schedule/exceptions-drawer.tsx:54` | `dialog` |
| `RuleModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/fixed-rules-page.tsx:169` | `Rol belirtilmemiş` |
| `HistoryDrawer` → `div` | Yüzey tanımı | `apps/web/features/schedule/history-drawer.tsx:66` | `dialog` |
| `NewProgramModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/modals.tsx:31` | `dialog` |
| `DeleteProgramModal` → `div` | Yüzey tanımı | `apps/web/features/schedule/modals.tsx:155` | `dialog` |
| `PublishDrawer` → `div` | Yüzey tanımı | `apps/web/features/schedule/publish-drawer.tsx:154` | `dialog` |
| `LessonDrawer` → `aside` | Yüzey tanımı | `apps/web/features/schedule/read-only/lesson-drawer.tsx:105` | `dialog` |
| `SchedulePage` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/schedule/schedule-page.tsx:544` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |

### İdari kadro

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `VicePrincipalDesignateDialog` → `div` | Yüzey tanımı | `apps/web/features/school-administration/designate-dialog.tsx:58` | `dialog` |
| `VicePrincipalRevokeDialog` → `div` | Yüzey tanımı | `apps/web/features/school-administration/revoke-dialog.tsx:56` | `dialog` |

### Sınıflar ve şubeler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `Shell` → `div` | Yüzey tanımı | `apps/web/features/sections/modals.tsx:80` | `dialog` |
| `SnfRehberModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:153` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfDerslikModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:249` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfAssignStudentModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:353` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfMoveStudentModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:450` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfArchiveModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:534` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfDeleteModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:596` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfNewModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:698` | `apps/web/features/sections/modals.tsx#Shell` |
| `SnfBulkModal` → `Shell` | Özellik kabuğu çağrısı | `apps/web/features/sections/modals.tsx:933` | `apps/web/features/sections/modals.tsx#Shell` |

### Okul ayarları

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ZilForm` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/settings/bell-schedule-tab.tsx:507` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `BellRulesButton` → `InfoDialog` | Ortak kabuk çağrısı | `apps/web/features/settings/bell-schedule-tab.tsx:557` | `apps/web/components/shared/info-dialog.tsx#InfoDialog` |
| `ClassroomTab` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/classroom-tab.tsx:551` | `apps/web/features/settings/parts.tsx#AModal` |
| `ClassroomTab` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/classroom-tab.tsx:584` | `apps/web/features/settings/parts.tsx#AModal` |
| `DerslikModal` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/classroom-tab.tsx:673` | `apps/web/features/settings/parts.tsx#AModal` |
| `CourseCatalog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/settings/course-catalog.tsx:328` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `CourseDrawer` → `ADrawer` | Özellik kabuğu çağrısı | `apps/web/features/settings/course-catalog.tsx:419` | `apps/web/features/settings/parts.tsx#ADrawer` |
| `ExamTypeCatalog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/settings/exam-type-catalog.tsx:335` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `ExamTypeDrawer` → `ADrawer` | Özellik kabuğu çağrısı | `apps/web/features/settings/exam-type-catalog.tsx:416` | `apps/web/features/settings/parts.tsx#ADrawer` |
| `HolidayTab` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/holiday-tab.tsx:302` | `apps/web/features/settings/parts.tsx#AModal` |
| `TatilModal` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/holiday-tab.tsx:374` | `apps/web/features/settings/parts.tsx#AModal` |
| `ADrawer` → `aside` | Yüzey tanımı | `apps/web/features/settings/parts.tsx:778` | `dialog` |
| `AModal` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/settings/parts.tsx:845` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `PrefixConsentModal` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/structure-tab.tsx:343` | `apps/web/features/settings/parts.tsx#AModal` |
| `SubjectAreaCatalog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/settings/subject-area-catalog.tsx:246` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `SubjectAreaModal` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/subject-area-catalog.tsx:311` | `apps/web/features/settings/parts.tsx#AModal` |
| `SubjectBranchRulesCatalog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/settings/subject-branch-rules-catalog.tsx:236` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `AddRuleModal` → `AModal` | Özellik kabuğu çağrısı | `apps/web/features/settings/subject-branch-rules-catalog.tsx:302` | `apps/web/features/settings/parts.tsx#AModal` |

### Ortak bileşenler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `ConfirmDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/components/shared/confirm-dialog.tsx:63` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `Dialog` → `div` | Yüzey tanımı | `apps/web/components/shared/dialog.tsx:136` | `dialog` |
| `InfoDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/components/shared/info-dialog.tsx:48` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `WizardSheet` → `div` | Yüzey tanımı | `apps/web/components/shared/wizard-sheet.tsx:93` | `dialog` |

### Öğrenciler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `StudentDetailDrawer` → `aside` | Yüzey tanımı | `apps/web/features/students/detail-drawer.tsx:74` | `dialog` |
| `EnrollWizard` → `WizardSheet` | Ortak kabuk çağrısı | `apps/web/features/students/enroll/enroll-wizard.tsx:138` | `apps/web/components/shared/wizard-sheet.tsx#WizardSheet` |
| `StuConfirmModal` → `div` | Yüzey tanımı | `apps/web/features/students/modals.tsx:31` | `Rol belirtilmemiş` |

### Öğretmen görevlendirmeleri

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `CloseAssignmentDialog` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/teacher-assignments/close-assignment-dialog.tsx:40` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `AssignDrawer` → `aside` | Yüzey tanımı | `apps/web/features/teacher-assignments/drawer.tsx:167` | `dialog` |
| `AssignDrawer` → `ConfirmDialog` | Ortak kabuk çağrısı | `apps/web/features/teacher-assignments/drawer.tsx:359` | `apps/web/components/shared/confirm-dialog.tsx#ConfirmDialog` |
| `InFieldRuleDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/teacher-assignments/in-field-rule-dialog.tsx:68` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `CopyModal` → `div` | Yüzey tanımı | `apps/web/features/teacher-assignments/modals.tsx:45` | `dialog` |

### Öğretmenler

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `TchDrawer` → `aside` | Yüzey tanımı | `apps/web/features/teachers/drawer.tsx:84` | `dialog` |
| `TchConfirm` → `div` | Yüzey tanımı | `apps/web/features/teachers/modals.tsx:59` | `Rol belirtilmemiş` |
| `TchCapacityModal` → `div` | Yüzey tanımı | `apps/web/features/teachers/modals.tsx:138` | `Rol belirtilmemiş` |
| `TchProfileModal` → `div` | Yüzey tanımı | `apps/web/features/teachers/modals.tsx:430` | `Rol belirtilmemiş` |

### Kullanıcılar

| Bileşen / sahibi | Kayıt | Dosya:satır | Temel / semantik |
|---|---|---|---|
| `UsrDrawer` → `aside` | Yüzey tanımı | `apps/web/features/users/drawer.tsx:48` | `dialog` |
| `UsrCreateModal` → `div` | Yüzey tanımı | `apps/web/features/users/modals.tsx:71` | `Rol belirtilmemiş` |
| `UsrConfirmModal` → `div` | Yüzey tanımı | `apps/web/features/users/modals.tsx:247` | `Rol belirtilmemiş` |
| `PersonNameDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/users/person-name-dialog.tsx:53` | `apps/web/components/shared/dialog.tsx#Dialog` |
| `PersonNameFormDialog` → `Dialog` | Ortak kabuk çağrısı | `apps/web/features/users/person-name-dialog.tsx:108` | `apps/web/components/shared/dialog.tsx#Dialog` |
