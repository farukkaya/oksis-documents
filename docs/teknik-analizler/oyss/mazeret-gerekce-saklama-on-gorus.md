# Mazeret sınavı serbest metin gerekçeleri — saklama süresi ön görüş taslağı

> **Durum:** Taslak (2026-10-26, güvenlik/KVKK denetçisi). Hukukçuya sunulmak için hazırlanmıştır, bağlayıcı hukuki görüş değildir. "Teyit edilmeli" diye işaretlenen her madde numarası ve süre birincil kaynaktan doğrulanmalıdır.
> **Kapsam:** OKSİS mazeret sınavı modülü ([ADR 0028](../../domain/kararlar/0028-merkezi-sinav-takvimi-oyss.md), M2, M3 ve M12). Kod durumu: `oksis-api` dev `254bcf66`.

## 1. İşlenen veri ve niteliği

| Alan | İçerik | Nitelik |
|---|---|---|
| Öğrenci kimliği | Kişi kimliği; asıl sınava ve şubeye bağ | Kişisel veri, öğrenci reşit değil |
| Gerekçe kategorisi | Sağlık / Aile / Resmî görev / Diğer | "Sağlık" değeri tek başına sağlık verisidir (KVKK m.6, özel nitelikli) |
| Gerekçe notu | 15–300 karakter serbest metin; Sağlık kategorisinde yazılamaz | Kişisel veri. Diğer kategorilerde metne sağlık, aile durumu ya da inanç bilgisi girebilir. Özel nitelikli veri ihtimali yüksek. |
| Karar (ret) notu | 15–300 karakter, idarenin yazdığı metin | Kişisel veri; gerekçeye atıf yaparak aynı riski taşır |
| İptal gerekçesi | 15–500 karakter | Aynı risk |
| "Belge kâğıtta görüldü" işareti | Evet / hayır | Kategoriyle birlikte okununca dolaylı olarak "sağlık raporu var" anlamına gelir |
| Durum ve tarihler | Başvuru, karar, planlama, sonuç (girdi / girmedi), kapanış | Kişisel veri (öğrencinin sınav geçmişi) |
| Denetim izi | Kimin, ne zaman, hangi işlemi yaptığı; metin içermez | Kişisel veri (personel ve öğrenci) |

- Mazeret sınavı modülüne belge yüklenmez. Ancak talep bir yoklama mazeretine bağlı olarak kendiliğinden kabul edilirse (M1), gerekçe Yoklama modülünde kalır. Yoklama mazeretinin serbest metni (`Reason`, `DecisionNote`) ve **yüklenmiş belge** bağlantısı (`StoredFileId`) vardır. Değerlendirme bu kaydı da kapsamalıdır; bkz. §7.
- Veri sorumlusu okuldur, OKSİS veri işleyendir (teyit edilmeli: sözleşme ve aydınlatma metni).
- Öğrenci reşit değildir. KVKK'da çocuklara özgü ayrı bir hüküm yoktur; ilgili kişi hakları veli veya vasi üzerinden kullanılır.

## 2. İşleme amacı ve hukuki dayanak adayları

**Amaç:** Sınava giremeyen öğrencinin mazeretinin değerlendirilmesi, kabul ya da red kararının verilmesi, mazeret sınavının planlanıp yapılması, kararın gerekçelendirilmesi ve itirazda gösterilebilmesi.

**Dayanak adayları:**

- Mazeret sınavına ilişkin MEB yönetmelik hükümleri (ortaokul ve ortaöğretim için ilgili yönetmelik adı, madde ve fıkra teyit edilmeli; özel okullar için Özel Öğretim Kurumları Yönetmeliği'nin resmî okul mevzuatına atfı teyit edilmeli).
- Özel nitelikli olmayan veri için KVKK m.5/2: "kanunlarda açıkça öngörülmesi" ve "veri sorumlusunun hukuki yükümlülüğü" (bent harfleri teyit edilmeli).
- Sağlık verisi için KVKK m.6/3 (7499 sayılı Kanunla 2024'te değişen metin): "kanunlarda açıkça öngörülmesi" ve "bir hakkın tesisi, kullanılması veya korunması" bentleri aday (yürürlük ve bent harfleri teyit edilmeli). Açık rıza bu bağlamda zayıf dayanaktır: rıza verilmezse mazeret hakkı fiilen engellenir.
- Özel nitelikli veri için Kurul'un "yeterli önlemler" kararı (2018; tarih ve sayı teyit edilmeli). Gerekçeyi yalnız iki iznin birlikte görmesi, denetim izi ve log maskeleme bu kararla uyumlu önlemlerdir.
- Saklama ilkesi: KVKK m.4/2, "işlendikleri amaç için gerekli olan süre kadar muhafaza edilme" (bent harfi teyit edilmeli).
- Silme, yok etme ve anonimleştirme yönetmeliği: periyodik imha aralığı (bilinen en çok 6 ay), imha kayıtlarının saklanma süresi, saklama ve imha politikası yükümlülüğü (madde ve süreler teyit edilmeli).

**Önemli bağlam:** OKSİS mazeretin resmî kayıt sistemi değildir. Puan ve "G" işareti e-Okul'a girilir; dilekçe ve rapor okulun kâğıt dosyasında MEB standart dosya planına göre saklanır (dosya kodu ve süre teyit edilmeli). Bu yüzden OKSİS'teki kopya veri minimizasyonu gereği resmî arşiv süresinden **kısa** tutulabilir.

## 3. Saklama süresine etki eden etkenler

| Etken | Etkisi | Not |
|---|---|---|
| Not itirazı | Sonuç bildiriminden sonraki kısa sürede yapılır | Süre ve madde teyit edilmeli (gün düzeyinde) |
| Dönem kapanışı | Mazeret sınavı dönem sonuna kadar yapılır (M6–M8); puan e-Okul'a dönem sonundan önce işlenir; OKSİS'te düzeltme kapısı dönem sonunda kapanır (M5) | Gerekçenin işlevsel ömrü büyük ölçüde dönem sonunda biter |
| Öğretim yılı kapanışı | Yılsonu puanı ve sınıf geçme 2. dönem sonunda kesinleşir; sorumluluk sınavları sonbahara uzayabilir | 2. dönem kayıtları için ek birkaç ay |
| e-Okul'a işleme | Sonrasında OKSİS'teki gerekçe yalnız "neden kabul/red" sorusuna yanıt verir | Kalıcı kanıt e-Okul ve kâğıt dosyadır |
| Şikâyet ve denetim | Veli şikâyeti, CİMER, KVKK m.13 başvurusu, Kurul şikâyeti, MEB denetimi | Süreler ayrı ayrı teyit edilmeli |
| Dava | Özel okul kararına karşı yargı yolu ve süresi belirsiz | Hukukçuya soru; uzun zamanaşımı serbest metni yıllarca tutmayı tek başına haklı kılmaz |

**Makul aralık:** asgari **dönem sonu + yaklaşık 30 gün**; azami **öğretim yılı sonu + yaz ve sorumluluk sınavları** (yeni öğretim yılının ilk ayı).

**180 günün yeri:** aralığın üst yarısında. 1. dönem kaydı (kasımda kapanmış) mayısta, 2. dönem kaydı (mayısta kapanmış) kasımda temizlenir. Hiçbir durumda dönem kapanışından önce silinmez.

## 4. Süre seçenekleri

| Seçenek | Artı | Eksi |
|---|---|---|
| **A. Terminalden 180 gün (bugünkü kod)** | Kod hazır, testli, yalnız ayar değişir; asgari koşulu her durumda karşılar | Dönemden bağımsız; 2. dönem kayıtları amaçtan sonra birkaç ay fazla kalır |
| **B. Terminalden 90 gün** | Daha az veri | Yılsonu ve sonbahar süreçlerini kaçırabilir |
| **C. Dönem sonu + 60 gün** | Amaçla doğrudan bağlı; dönem başına toplu imha | Kod değişikliği gerekir; yılsonunda 1. dönem gerekçesi gerekirse erken silinmiş olur |
| **D. Öğretim yılı sonu + 30/60 gün** | Okulun yıllık arşiv düzeniyle aynı; yılda tek imha tarihi | Erken 1. dönem kayıtlarını 8–9 ay tutar; minimizasyonda A ve C'nin gerisinde |

**Öneri:** Kısa vadede A kalsın. Hukukçu dönem kapanışının amacı bitirdiğini teyit ederse C'ye geçilsin; yılsonunda 1. dönem gerekçesine ihtiyaç görürse D değerlendirilsin. Kâğıt dosya ve e-Okul resmî kayıt olarak kalır.

## 5. Temizlikten sonra kalan veri

Temizlikten sonra öğrenci kimliği, kategori (Sağlık dahil), durum, tarihler, belge işareti, yoklama bağlantı kimlikleri, karar veren ve denetim izi kalıyor. Bunlar için saklama kuralı yok, şu an **süresiz**.

| Veri | Değerlendirme | Öneri |
|---|---|---|
| Kategori = Sağlık | Tek başına sağlık verisi; metin silindikten sonra tutulma amacı belirsiz | Serbest metinle aynı anda nötr değere indirilsin (tüm kategoriler) |
| Belge işareti | Kategoriyle birlikte sağlık raporuna işaret eder | Kategoriyle aynı anda silinsin |
| Durum, tarihler, sonuç | Öğrencinin sınav geçmişi | Okulun genel öğrenci kaydı saklama süresine bağlansın |
| Yoklama bağlantı kimlikleri | Bağlı yoklama mazeretine yol gösterir | Yoklama mazeretinin saklama süresiyle uyumlu olsun (§7.1) |
| Denetim izi | Hesap verebilirlik kaydı | Durumla aynı süre ya da yönetmelikteki imha kaydı süresi (teyit edilmeli) |
| İstatistik | Kişisel veri gerektirmez | Öğrenci bağından koparılmış sayım olarak tutulsun |

## 6. Hukukçuya sorular

1. Özel okulda mazeret gerekçesi ve sağlık kategorisi için doğru dayanak hangisi: m.6/3'teki "kanunlarda açıkça öngörülme" mi, "hakkın tesisi ve korunması" mı? Hangi MEB yönetmelik maddesi bunu karşılıyor? Açık rıza aranmalı mı?
2. OKSİS'teki serbest metin, kâğıt dosya ve e-Okul yanında ikincil kopya sayılabilir mi? Öyleyse saklama süresi standart dosya planından bağımsız ve kısa belirlenebilir mi?
3. Amacın bittiği an hangisi: dönem kapanışı, öğretim yılı kapanışı, yoksa şikâyet ve dava süreleri?
4. Serbest metin silindikten sonra "Sağlık" kategorisinin öğrenciye bağlı tutulması hukuka uygun mu? Aynı anda nötrleştirilmesi yeterli mi?
5. Saklama süresini her okul (veri sorumlusu) kendisi mi belirlemeli, yoksa tek süre mi? Alt ve üst sınır ne olmalı?
6. Yedeklerde süresi dolmuş metin bir süre daha kalıyor; ayrı süre ya da açıklama gerekir mi? İmha için hangi kayıt tutulmalı, ne kadar saklanmalı?

## 7. Koddaki uygulama ile ADR uyumu

**Uyumlu:** terminal kayıtların ve kilitli pencerede bayat açık kayıtların serbest metni 180 gün sonra temizleniyor (`ExamDailySweepJob.cs:184-246`, `MakeupExam.cs:376-421`; ayar `Exams:Makeup:ReasonRetentionDays`, 1'den küçükse 180). Yeni metin yazılırsa süre yeniden işliyor. Olaylar gerekçe taşımıyor; gerekçe yanıtta yalnız yetkiyle dönüyor (`MakeupDtoBuilder.cs:105-112`); logda yalnız sayı var. Sağlık kategorisinde serbest not yasağı kodda (`MakeupExam.cs:466-468`).

**Açıklar:**

1. **(Yüksek) Bağlı yoklama mazereti temizlenmiyor.** M1 otomatik kabulünde gerekçe Yoklama'da kalıyor (`AttendanceExcuse.cs:28`, `:38`) ve yüklenmiş belge bağlantısı var (`:43`, `:100-109`). Yoklama mazereti için saklama işi yok (ADR 0028 açık sorular). Politika bu kaydı kapsamazsa M12'nin etkisi kısmi kalır.
2. **(Orta) Sağlık kategorisi ve belge işareti süresiz kalıyor** (`MakeupExam.cs:68`, `:74`).
3. **(Orta) Sağlık dışı kategorilerde sağlık bilgisi yazılabiliyor.** Arayüzde "sağlık bilgisi yazmayın" uyarısı önerilir (ürün kararı).
4. **(Düşük) Süre okul bazında değil, sunucu genelinde tek ayar** (`ExamDailySweepJob.cs:186`); veli toplantısı notu saklama süresi ise okul ayarı.
5. **(Düşük) İmha kaydı zayıf.** Temizlik denetim izine (`MakeupAuditEntry`) yazılmıyor.
6. **(Düşük) Denetim izi ve durum alanları için saklama kuralı yok** (§5).
7. **(Belge) Atıf yapılan KVKK belgesi yok.** Kod yorumları `kvkk.md`'ye atıf yapıyor (`ExamDailySweepJob.cs:66`, `MakeupExam.cs:37`, `MakeupOptions.cs:17`); dosya `oksis-documents/docs` içinde bulunamadı.
