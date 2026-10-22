# OKSİS — OYSS Geçiş Planı

| | |
|---|---|
| **Sürüm** | v1.0 |
| **Tarih** | 22.10.2026 |
| **Onay** | Faruk, 22.10.2026 |
| **Kapsam** | Sınav Takvimi modülünün MEB Ortak Yazılı Sınav Sistemi'ne (OYSS) uyarlanması; Sürüm 1 (30 Ekim) ve Sürüm 2 (2–18 Kasım) |
| **Karar** | [[0028-merkezi-sinav-takvimi-oyss]] |
| **Modül** | [[Sınav Takvimi]] |
| **Kaynaklar** | OYSS — OKSİS Entegrasyon Raporu: https://claude.ai/artifact/LgBJ2e1L648YdSNcJAk9cX · OYSS Geçiş Planı (asıl): https://claude.ai/artifact/AatFzUknJCw1qUB5pNmrPM |

---

## 1. Amaç

1. yazılılar 3–13 Kasım 2026'da başlıyor. MEB takviminin OKSİS'te kilitli, kaynağı belli ve okulun ders programıyla eşleşmiş hâlde görünmesi gerekiyor. Sürüm 1, **dev ortamında** (K1) Altınay test okuluyla (K4) uçtan uca çalışır. Okul hattında akış şöyledir: platform takvimi yükler ve ikinci kişi doğrular; okul önizleyip içe alır; idare gerekirse saati gerekçeyle değiştirir; öğretmen kendi OYSS satırını ve "Bugün" kartını görür; öğrenci ve veli mobilde OYSS rozetini görür.

## 2. Takvim

| Tarih | Nokta | Çıktı |
|---|---|---|
| 23 Eki 12:00 | **B0 sözleşme** | Uç listesi, DTO alanları, hata kodları ve `EX-S10` metni donar; UI mock'u bu sözleşmeye yazılır |
| 26 Eki | **Ara teslim** | Sözleşme + katalog + MEB CSV (B1'in içe alma tarafı çalışır) |
| 28 Eki öğlen | **Kontrol noktası** | Gerideysek §5'teki kesme sırası uygulanır |
| 30 Eki | **Sürüm 1** | Dev ortamında uçtan uca senaryo (Q1) yeşil |
| 2–18 Kas | **Sürüm 2** | Mazeret ve telafi; 23–27 Kasım mazeret haftasından önce hazır |

## 3. Sürüm 1 dilimleri

| Dilim | Tarih | Katman | İçerik |
|---|---|---|---|
| **B0** | 23 Eki 12:00 | Backend | Sözleşme: platform uçları, okul içe alma önizleme/uygulama uçları, `:change-central-period`, `ScheduledExam` DTO'larına `centralEntryId` / `centralDate` / `centralPeriod` / `isCentral`, `me/*` alanları, `EX-S10` |
| **B1** | 23–26 Eki | Backend | Göç M1 (`master.central_exam_calendars`, `master.central_exam_calendar_entries`), platform uçları (yükle, doğrula, listele), MEB CSV ayrıştırma, iki kişi kuralı, okul içe alma önizlemesi (eşleşen, eşleşmeyen dersler, `outOfWindowDates`) ve idempotent içe alma |
| **B2** | 23–27 Eki | Backend | Göç M2 (`ScheduledExam` üç alan), domain kilidi (tarih değişmez, gün seçilemez, yerleşim kaldırılamaz), kural denetleyicisinde `EX-S10` ve yumuşama, `:change-central-period` + revizyon türü 6 `CentralPeriodChanged`, `me/*` okuma alanları |
| **W1** | 27–30 Eki | Web | Platform OYSS ekranı: takvim listesi, CSV yükleme, satır önizleme, doğrulama (yükleyen doğrulayamaz) |
| **W2** | 27–30 Eki | Web | Yönetici: içe alma önizlemesi ve uygulama, panoda kilit rozeti, saat değişikliği diyaloğu (gerekçe zorunlu, S10 metni) |
| **W3** | 27–30 Eki | Web | Öğretmen: OYSS satırı salt okunur (gün seçici yok), uygulayan öğretmen bilgisi |
| **M1** | 27–30 Eki | Mobil | OYSS rozeti, kilit gösterimi, "puanlar e-Okul'da" şeridi, öğretmen "Bugün" kartı (bugün uygulayacağı sınav ve ders saati), MSW mock, bağlantı yok ekranı |
| **Q1** | 27–30 Eki | QA | Uçtan uca senaryo, aşağıda §6 |

Sıralama: B0 her şeyi açar. UI dilimleri B0'dan sonra mock'la başlar, B1/B2 geldikçe gerçek uca bağlanır. Göçler M1 → M2 sırasıyla tek dalda yazılır; şema değişikliği koordinatör panosunda duyurulur.

## 4. Kesilemezler

Kontrol noktasında geride olsak da bunlar Sürüm 1'den çıkmaz:

1. Merkezi satırın tarihinin kilitli olması
2. `EX-S10` ve sert/yumuşak ayrımı
3. Saat değişikliğinin revizyon kaydı yazması
4. İki kişi doğrulaması
5. İdempotent içe alma
6. Eşleşmeyen ders listesi
7. Öğretmenin OYSS satırında gün seçememesi
8. Mobil OYSS rozeti ve öğretmen "Bugün" kartı

## 5. Kesme sırası (28 Ekim öğlen)

Gerideysek sırayla kesilir, Sürüm 2'ye taşınır:

1. `:new-version` (katalog düzeltme sürümü; o zamana kadar düzeltme = yeni taslak takvim)
2. "Saat değişti" rozeti
3. Önizlemede `outOfWindowDates`
4. Platform ekranı cilası
5. Ülke geneli satırlar (katalogda kapsam bilgisi)
6. Mobil "e-Okul'da" şeridi

## 6. Uçtan uca senaryo (Q1)

1. Platform hesabı A, 2026-27 OYSS CSV'sini yükler. A doğrulamayı dener ve reddedilir. Platform hesabı B doğrular.
2. Altınay yöneticisi 1. dönem 1. yazılı penceresinde önizleme açar. Eşleşen satırları ve eşleşmeyen dersleri görür. İçe alır. İkinci içe alma yeni satır üretmez.
3. Öğretmen OYSS satırında gün değiştiremez; uygulayan öğretmen MEB saatindeki dersin öğretmenidir. Hücre boşsa dersin öğretmeni olur ve S10 görünür.
4. Yönetici bir satırın saatini gerekçeyle 2. dersten 3. derse alır. S10 gerekçe ister. Takvim yayındaysa revizyon 6 yazılır ve "sınav taşındı" bildirimi gider.
5. Bir günde 3 sınav bulunan şube `EX-H01` yerine `EX-S10` uyarısı alır. Aynı hücrede iki sınav (`EX-H13`) hâlâ reddedilir.
6. Mobilde öğrenci ve veli OYSS rozetini ve e-Okul şeridini görür. Öğretmen sınav günü "Bugün" kartını görür. Bağlantı yokken "bağlantı yok" ekranı açılır.

## 7. Sürüm 2 (2–18 Kasım)

- `MakeupExam` agregatı: okulun mazeret haftası (23–27 Kasım) ve sonrasında 5 iş günü içinde öğretmen telafisi
- Yoklama bağı: sınav günü devamsız öğrenci mazeret adayı olur
- Mazeret ekranları (yönetici planlama, öğretmen telafi, mobil görünüm)
- Platform katalog giriş ekranı (CSV dışı elle satır girişi)
- KVKK denetimi (mazeret gerekçesi ve sağlık belgesi kişisel veridir)
- Sürüm 1'den kesilmiş maddeler

Not girişi Notlar modülünde kalır; soru bazlı puan yoktur (K6).

## 8. Dal ve çalışma düzeni

- Dal: `feature/oyss-entegrasyonu`, `dev`'den açılır.
- Worktree'ler: `~/Repositories/oksis/oksis-api-oyss` ve `~/Repositories/oksis/oksis-ui-oyss`.
- `dev`'e birleştirme yalnız Faruk onayıyla yapılır. `test` ve `master` bu planın kapsamında değildir (K1).

## 9. Kapsam dışı

- Soru, cevap anahtarı ve soru bazlı puan (e-Okul'da kalır)
- e-Okul entegrasyonu
- Çevrimdışı çalışma (K3); canlıya geçiş öncesi listesine not
- Öğrenci/veli sınav ekranının web sürümü (K2)
- Ayrı "merkezi sınav" türü (K5)

## 10. Açık konular

- MEB ders saati kuralı ve ülke geneli sınavın yazılı adlandırması eğitim uzmanınca birincil kaynaktan doğrulanıyor. Sonuç yalnız katalog değerlerini etkiler, tasarımı değiştirmez ([[0028-merkezi-sinav-takvimi-oyss]] §Doğrulama bekleyenler).
- [[Sınav Takvimi]] modül notunun kural tablosu kodun gerisinde: `EX-H13` ve `EX-S07…S09` kodda var, tabloda yok. `domain-map` senkronunda düzeltilmeli.
