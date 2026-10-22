# Raporlar merkezi — BLG-0024

17 Ekim 2026. Kullanıcının açık isteğiyle eğitim alan uzmanı ajanı mevcut raporları salt okunur inceledi; uygulama sonrasında ikinci kontrol yapıldı. Yeni rapor, mevzuat eşiği veya veri kaynağı eklenmedi.

## Eğitimci değerlendirmesi ve karar

Yönetici raporu adı için değil, yanıt aradığı soru için açar. Eski merkezde yedi kartın yalnız üçü çalışıyordu; dört gelecek kartı ve sekiz kategori, kullanılabilen raporları gölgeliyordu. Merkez üç çalışan alana indirildi; alt görünümler doğrudan bağlantıdır.

| Alan | Yanıt aranan soru | Mevcut görünümler |
|---|---|---|
| Devamsızlık | Şubelerin devam durumu nasıl, hangi öğrencilerin takibe ihtiyacı var? | Risk listesi, dönem özeti, eğilim analizi |
| Ödev | Ödev yoğunluğu ve öğrencilerin tamamlama durumu nasıl? | Ödev panosu, tamamlama oranı |
| Nöbet | Öğretmenlerin dönem içindeki nöbet dağılımı nasıl? | Nöbet dağılımı |

Ortak PageHeader, SearchBox, FilterDropdown ve EmptyState kullanıldı. Yeni sayaç veya grafik türetilmedi. Türkçe karakterleri katlayan arama ve yalnız bu üç alanın filtresi var. Kullanılmayan gelecek raporlar açılabilir işlem gibi sunulmuyor.

## Raporun doğru yorumlanması

- Devamsızlık raporunun dönem kimliği gerçek üst çubuk bağlamından gelir. Sabit “2. Dönem (aktif)” seçicisi kaldırıldı.
- Tarih aralığı sunucu sorgusunu etkilemiyordu; kaldırıldı. Eğilim görünümünde sorguyu etkilemeyen kademe ve şube kontrolleri de gösterilmez.
- Excel/PDF düğmeleri yalnız bildirim mesajı üretiyordu; indirme vaat eden bu işlemler kaldırıldı. Şube kırılımında da sahte PDF işlemi yok.
- “Karneye aktarılacak resmi toplam” güvencesi yerine seçili dönemin devamsızlık toplamları denir.
- Öğrenci kırılımındaki sabit 3/5 gün renkleri kaldırıldı. Risk kararı sunucunun eşik listesinden gelir. Risk listesinde henüz çalışmayan görüşme kaydı düğmesi yok.
- Kademe eşlemesi yüklenir veya hata verirse sonuç “uyarı yok” diye açıklanmaz. Boş sonuç yalnız seçili filtrelerde öğrenci bulunmadığını söyler; genel sağlık hükmü vermez.

## Gerçek veri sözleşmesinde bulunan hata

API `TrendReportDto` ve `GetTrendAbsenceReportQueryHandler` kaynakları karşılaştırıldı: `heatmap` devamsızlık yüzdesidir; `weeklyRates` devam yüzdesidir; `termAverage` dönem devamsızlık yüzdesidir. Arayüz ısı hücresine ikinci kez `100-v` uyguluyordu. Yüzde5 devamsızlığı yüzde95 olarak gösteren bu işlem kaldırıldı. Dönem ortalamasının etiketi de düzeltildi. Geliştirme örnekleri aynı sözleşmeye getirildi.

Isı haritasının sütun sayısı okulun gerçek ders sayısına uyar. Sıfır hücre nötr görünür. Sunucunun en yüksek/en düşük hücre etiketleri kullanılır; veri olmayan hücreler de0 olabildiğinden istemci bunlardan yeni kesin minimum yüzdesi çıkarmaz.

Haftalık seri takvim tarihi taşımıyor. Bugünden geriye uydurma ay ve tarih üretmek yerine veri bulunan haftalar sıra numarasıyla gösterilir. Kesikli çizgi, haftalık oranların aritmetik ortalaması olarak açıklanır.

## Doğrulama

Önce başarısız regresyonlar yazıldı:3 dönem/kontrol,3 eğilim anlamı,2 risk eşleme durumu; düzeltmeden sonra8 test geçti. Örnek veri sözleşmesi için1 test de önce başarısız, sonra başarılı oldu. Son web test, tip ve tarayıcı sonuçları FINDINGS-HANDOFF.md ve bulgu çözüm notunda kaydedilir.

Gerçek tarayıcıda üst çubuk1. dönemden2. döneme geçince `/attendance/risk` ve `/attendance/reports/term` sorgularının aynı seçili dönem kimliğini taşıdığı görüldü. Kişisel veya okul verisi değiştirilmedi.

## Kaynaklar

- [Rapor merkezi](/Users/farukkaya/Repositories/oksis/oksis-ui-anlik-duzeltmeler/apps/web/features/reports/reports-hub.tsx)
- [Devamsızlık kapsamı](/Users/farukkaya/Repositories/oksis/oksis-ui-anlik-duzeltmeler/apps/web/features/reports/attendance-reports.tsx)
- [Eğilim görünümü](/Users/farukkaya/Repositories/oksis/oksis-ui-anlik-duzeltmeler/apps/web/features/reports/trend-tab.tsx)
- [Sunucu veri sözleşmesi](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Attendance/DTOs/AttendanceDtos.cs)
- [Gerçek eğilim hesabı](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Attendance/Queries/GetTrendAbsenceReport/GetTrendAbsenceReportQueryHandler.cs)
