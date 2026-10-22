# Bildirim alıcıları ve kişisel tercihler — BLG-0021

17 Ekim 2026. Kaynak: API `422fc9bc` + düzeltme çalışma kopyası; UI `fix/anlik-duzeltmeler`. Alıcı tanımı yorumdan değil, üretici işleyiciler ve ortak alıcı çözümleme çağrılarından karşılaştırıldı.

## Sonuç

59 bildirim türü tarandı. 30 tür, 29 ayrı telefon bildirimi tercihi kullanıyor; vekâlet atama ve geri alma aynı anahtarı paylaşır. Diğer türlerin bu ekranda telefon anahtarı yok. Ekrana üreticisi veya push kapsamı olmayan anahtar eklenmedi.

Tercih **hesap düzeyinde** saklanır. Sunucu kişinin tüm profillerinin ve idare rolünün birleşimine uygun satırları döndürür. Öğretmen-veli hesabını yalnız aktif öğretmen rolüyle tekrar filtrelemek veli tercihlerini saklıyordu. Mobil ekran artık sunucunun yetkili listesini yalnız gruplar. Anahtarın açık olması, o kullanıcıya her olayın gönderileceği anlamına gelmez: gerçek öğrenci, şube, kulüp, görev, veli ilişkisi ve duyuru hedefi yine üreticide belirlenir. Veli ilişkisinin kapsamı olaya göre değişir; bazı devamsızlık olayları bilgi erişimi kısıtlanmış bağlı velileri de içerir.

Saf kitlelerin ayarlanabilir anahtar sayıları (mevcut harita):

| Kitle | Anahtar sayısı |
|---|---:|
| Okul idaresi | 5 |
| Öğretmen | 14 |
| Öğrenci | 16 |
| Veli | 12 |
| Diğer personel | 2 |

Çok profilli hesabın listesi bu kümelerin tekilleştirilmiş birleşimidir. Örneğin müdür yardımcısının öğretmen profili varsa öğretmen bildirimleri de listelenir.

## Tam alıcı envanteri

“Yok”, bu ekranda kişisel telefon tercihi bulunmadığını söyler; uygulama içi bildirimin veya başka kanalın kapatıldığını söylemez. İdare, geçerli okul/sezon kapsamındaki müdür ve müdür yardımcısıdır. Bağlı hesabı olmayan kişi bildirim almaz.

| Olay / tür | Gerçek alıcı kapsamı | Telefon tercihi |
|---|---|---|
| Ders programı yayınlandı · `TimetablePublished` | İlgili şubenin öğrencileri ve velileri | Yok |
| Ders programı değişikliği · `TimetableException` | Şubenin öğrenci/velileri ve asıl öğretmen; vekâlet dışındaki yeni öğretmen varsa o da | Yok |
| Ders iptal edildi · `TimetableCancelled` | Şubenin öğrenci/velileri ve asıl öğretmen | Yok |
| Program değişikliği geri alındı · `TimetableExceptionRevoked` | İlgili şubenin öğrenci/velileri | Yok |
| Program kaldırıldı · `TimetableProgramDeleted` | İlgili şubenin öğrenci/velileri | Yok |
| Nöbet çizelgesi yayınlandı · `DutyRosterPublished` | Çizelgede görevlendirilen öğretmenler | Yok |
| Kayıt yenilendi · `EnrollmentRenewed` | Yenileme olayındaki öğrencinin bağlı velileri | Yok |
| Devamsızlık kaydı · `AttendanceAbsent` | İlk ders yoklamasında gelmeyen öğrencinin ilgili velileri | `ATT_ABSENT` |
| Günlük yoklama özeti · `AttendanceDailySummary` | İlgili öğrencinin ilgili velileri | Yok |
| Yoklama hatırlatması · `AttendanceReminder` | Yoklama oturumunun öğretmeni; gün sonu kapatma kendiliğinden hatırlatma üretmez | `ATT_REMINDER` |
| Mazeret kararı · `ExcuseDecided` | İlgili öğrencinin ilgili velileri | Yok |
| Yoklama düzeltme kararı · `AmendmentRequestDecided` | Talep eden öğretmen ve ilgili öğrencinin ilgili velileri | Yok |
| Gün içi izin verildi/geri alındı · `DailyLeaveGranted` | İlgili öğrencinin bağlı velileri; geri alma aynı bildirim türünü kullanır | Yok |
| Etkinlik için toplu izin · `ActivityBulkExcused` | Etkinliğe alınan öğrencilerin ilgili velileri | Yok |
| Devamsızlık eşiği · `AbsenceThresholdReached` | İlgili öğrencinin ilgili velileri ve okul idaresi | Yok |
| Duyuru yayınlandı · `AnnouncementPublished` | Duyurunun kayıtlı alıcıları: hedefe göre öğrenci, veli, öğretmen, personel/idare | `ANNOUNCEMENT` |
| Duyuru geri çekildi · `AnnouncementWithdrawn` | Geri çeken başka kişiyse duyuruyu yayınlayan kişi | Yok |
| Duyuru güncellendi · `AnnouncementAmended` | Duyurunun kayıtlı alıcıları | Yok |
| Duyuru onaya sunuldu · `AnnouncementSubmittedForApproval` | Duyuru onayına yetkili kişiler | Yok |
| Duyuru onaylandı · `AnnouncementApproved` | Duyuruyu hazırlayıp yayınlayan kişi | Yok |
| Duyuru reddedildi · `AnnouncementRejected` | Duyuruyu hazırlayıp yayınlayan kişi | Yok |
| Zamanlanmış duyuru yayınlandı · `AnnouncementScheduledExecuted` | Duyuruyu zamanlayan/yayınlayan kişi | Yok |
| Zamanlanmış duyuru yayınlanamadı · `AnnouncementScheduleFailed` | Duyuruyu zamanlayan/yayınlayan kişi | Yok |
| Not yayınlandı · `GradePublished` | Değerlendirmedeki öğrenciler ve ilgili velileri | `GRADE_PUBLISHED` |
| Yeni ödev/değişiklik · `HomeworkPublished` | Ödevin öğrencileri ve ilgili veli kapsamı | `HOMEWORK_PUBLISHED` |
| Ödev teslim hatırlatması · `HomeworkDueReminder` | Hatırlatmaya uygun ödev öğrencileri ve ilgili velileri | `HOMEWORK_DUE` |
| Eksik ödev bildirimi · `HomeworkMissing` | Eksik ödevli öğrencinin ilgili velileri; anlık ve günlük özet üreticileri var | Yok |
| Kulüp etkinliği yayınlandı · `ClubActivityPublished` | Etkinlik öğrencileri ve ilgili velileri | `CLUB_ACTIVITY_PUBLISHED` |
| Kulüp etkinliği iptal edildi · `ClubActivityCancelled` | Etkinlikteki öğrenciler ve ilgili velileri | `CLUB_ACTIVITY_CANCELLED` |
| Kulüp duyurusu · `ClubAnnouncementPublished` | Kulübün ilgili öğrenci üyeleri | `CLUB_ANNOUNCEMENT_PUBLISHED` |
| Kulüp başvuru sonucu · `ClubApplicationDecided` | Başvuruyu yapan öğrenci | `CLUB_APPLICATION_DECIDED` |
| Not girişi hatırlatması · `GradeEntryReminder` | Not girmesi beklenen sorumlu öğretmen | `GRADE_ENTRY_REMINDER` |
| Sınav haftası duyuruldu · `ExamWindowPublished` | Aktif sezonun tüm şubelerinin öğrenci/velileri ve okul öğretmenleri | Yok |
| Sınav saati seçme hatırlatması · `ExamPlacementReminder` | Yerleştirme yapması gereken öğretmen | `EXAM_PLACEMENT_REMINDER` |
| Sınav için saat istendi · `ExamHourRequested` | Ders saati istenen öğretmen | `EXAM_HOUR_REQUESTED` |
| Saat isteği yanıtlandı · `ExamHourAnswered` | İsteği açan öğretmen | `EXAM_HOUR_ANSWERED` |
| Sınav takvimi yayınlandı · `ExamSchedulePublished` | Yerleşmiş sınavların şubelerindeki öğrenci/veliler ve sınav sahibi öğretmenler | `EXAM_SCHEDULE_PUBLISHED` |
| Sınav planı değişti · `ExamMoved` | Etkilenen öğrenci/veliler ve sınav sahibi öğretmen; oturma yeri değişiminde ilgili öğrenci/veliler | `EXAM_MOVED` |
| Yarın sınav var · `ExamTomorrow` | Sınav şubesinin öğrenci/velileri ve sınav sahibi öğretmen | `EXAM_TOMORROW` |
| Gözetmenlik değişti · `ExamInvigilationChanged` | Görevi kaldırılan ve yeni görevlendirilen öğretmenler | `EXAM_INVIGILATION_CHANGED` |
| Sınav planı görüşe açıldı · `ExamReviewOpened` | İlgili oturumların sorumlu öğretmenleri ve gözetmenleri | Yok |
| Sınav görüşü eklendi · `ExamReviewCommentAdded` | Okul idaresi | Yok |
| Sınav görüşü yanıtlandı · `ExamReviewCommentResolved` | Görüşü yazan öğretmen | Yok |
| Yeni mazeret bildirimi · `ExcuseSubmitted` | Okul idaresi | `ATT_EXCUSE_SUBMITTED` |
| Yeni yoklama düzeltme talebi · `AmendmentRequestSubmitted` | Okul idaresi | `ATT_AMENDMENT_REQUESTED` |
| Acil duyuru · `AnnouncementUrgentPublished` | Duyurunun kayıtlı alıcıları: hedefe göre öğrenci, veli, öğretmen, personel/idare | `ANNOUNCEMENT_URGENT` |
| Vekâlet atandı · `SubstitutionAssigned` | Vekil öğretmen | `SUBSTITUTION_ASSIGNED` |
| Vekâlet geri alındı · `SubstitutionRevoked` | Vekil öğretmen | `SUBSTITUTION_ASSIGNED` |
| Yoklamayı yeniden açma talebi · `AttendanceReopenRequestSubmitted` | Okul idaresi | `ATT_REOPEN_REQUESTED` |
| Yoklama yeniden açıldı · `AttendanceSessionReopened` | Yoklama oturumunun öğretmeni | `ATT_SESSION_REOPENED` |
| Yeniden açma talebi reddedildi · `AttendanceReopenRequestDecided` | Talebi açan öğretmen; onayda bunun yerine oturum açıldı bildirimi gider | Yok |
| Öğrenci ödev gönderdi · `HomeworkSubmitted` | Ödevden sorumlu öğretmen | `HOMEWORK_SUBMITTED` |
| Ödev değerlendirildi/kapandı · `HomeworkReviewed` | İlgili öğrenci; veli bu türü almaz | `HOMEWORK_REVIEWED` |
| Kulüp tercih dönemi açıldı · `ClubPreferenceRoundOpened` | Kapsamdaki, kulüp üyeliği olmayan öğrenciler | `CLUB_PREFERENCE_OPENED` |
| Kulüp tercihi hatırlatması · `ClubPreferenceReminder` | Tercihini tamamlamamış kapsam öğrencileri | `CLUB_PREFERENCE_REMINDER` |
| Kulüp tercih dönemi kapandı · `ClubPreferenceRoundClosed` | Kapatan hariç diğer okul idarecileri | Yok |
| Kulüp yerleştirme sonucu · `ClubPlacementPublished` | Kulübü belirlenen/değişen/kaldırılan öğrenci ve ilgili velileri | `CLUB_PLACEMENT_PUBLISHED` |
| Kulübe yerleşemedi · `ClubPlacementUnplaced` | İlk yayında hiçbir kulübe yerleşmeyen öğrenci | Yok |
| Kulüp üye özeti · `ClubPlacementAdvisorSummary` | Üye girişi/çıkışı olan kulübün danışman öğretmeni | Yok |

## Tercih kaydı ve görünüm

- `GET /notifications/preferences`, hesaba uygun her push anahtarını döndürür; kaydı olmayan anahtar açıktır.
- `PUT /notifications/preferences` yalnız gönderilen anahtarları upsert eder. Hesap kimliği oturumdan gelir; gönderilmeyen tercihler silinmez.
- Mobil anahtar hemen güncellenir. Hata halinde eski değer geri yüklenir ve hata gösterilir; ardından sunucu listesi yeniden okunur. Bir kayıt sürerken anahtarlar kilitlenir; üst üste iki iyimser güncellemenin birbirini geri alması önlenir.
- Tercih hesapla birlikte diğer cihazlarda da geçerlidir. Telefonun işletim sistemi izni ayrıca açık olmalıdır.
- Telefon izni, izin isteme/ayar açma işlemleri, otomatik kayıt ve uygulama içi listenin devam etmesi açıklamaları sağ üstteki bilgi panelindedir.
- Eksik üç etiket eklendi: `ANNOUNCEMENT`, `ATT_REOPEN_REQUESTED`, `ATT_SESSION_REOPENED`. Eski yerel rol sözlüğünde acil duyurunun idare kapsamı ve yarınki sınavın öğretmen kapsamı düzeltildi; üretimde sunucunun hesap listesi yeniden daraltılmaz.

## Kaynaklar

- [Push anahtarı ve kitle haritası](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Notifications/Internal/PushEventKeyMap.cs)
- [Tüm bildirim türleri](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Domain/Modules/Notifications/Enums/NotificationKind.cs)
- [Hesabın tercih sorgusu](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Notifications/Queries/GetNotificationPreferences/GetNotificationPreferencesQueryHandler.cs)
- [Tercih kaydı](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Notifications/Commands/UpdateNotificationPreferences/UpdateNotificationPreferencesCommandHandler.cs)
- [Öğrenci, veli ve idare alıcı çözümleme](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Infrastructure/Notifications/NotificationRecipientResolver.cs)
- [Sınav alıcı çözümleme](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Exams/Events/Notifications/ExamNotificationAudience.cs)
- [Yarınki sınavın öğretmen alıcısı](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Exams/Commands/SendExamTomorrowReminders/SendExamTomorrowRemindersCommandHandler.cs)
- [Duyuru yayını](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Announcements/Events/Notifications/AnnouncementPublishedNotificationHandler.cs)
- [Kayıt yenileme alıcıları](/Users/farukkaya/Repositories/oksis/oksis-api-anlik-duzeltmeler/src/Oksis.Application/Modules/Students/Events/EnrollmentRenewedEventHandler.cs)
- [Mobil etiket/grup kataloğu](/Users/farukkaya/Repositories/oksis/oksis-ui-anlik-duzeltmeler/packages/core/src/notifications/preferences.ts)

## Doğrulama

Kaynak türleriyle envanter satırlarının 59/59 eşitliği, push haritasındaki30tür/29anahtar ve tüm29anahtarın mobil katalogda adlandırılması otomatik karşılaştırıldı. Mobilde çift profil satırının saklanmaması ve kayıt sırasında anahtar kilidi için önce başarısız iki regresyon testi; eksik etiket/yanlış rol eşlemesi için önce başarısız dört core testi yazıldı. Son test ve tarayıcı kayıt sonuçları bulgu çözüm notunda ve FINDINGS-HANDOFF.md içinde tutulur.
