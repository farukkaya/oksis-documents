# OKSİS — Veli Toplantısı Modülü · Teknik Analiz (V1)

| | |
|---|---|
| **Belge türü** | Teknik analiz (mimari, veri modeli, API sözleşmesi, dilimleme) |
| **Talep** | REQ-043 · Veli Toplantısı modülü V1 · Faz 2 |
| **Girdi** | `ihtiyac-analizleri/veli-toplantisi/OKSİS — Veli Toplantısı Modülü · İhtiyaç Analizi (Final).md` (K-1…K-16, BR-PM-01…42, §16 S-1…S-6 kapandı) + güvenlik ön incelemesi (2026-10-15, K1-K3 / Y1-Y6 / O1-O4 / D1-D2 — **tasarım şartı**) |
| **Kod tabanı** | `oksis-api-veli-toplantilari` ve `oksis-ui-veli-toplantilari` (dal `feature/veli-toplantilari`), 2026-10-15 tarandı — modül için kod yok |
| **Tarih** | 15 Ekim 2026 |
| **Domain notu** | [[Veli Toplantıları]] |
| **Durum** | Görüşe açık — kararlar Faruk'ta. ADR'ler: [[0025-sube-toplantisi-ayri-agregat]], [[0026-yanit-plan-surumune-baglanir]], [[0027-eposta-varsayilani-tur-bazli]] (üçü de `proposed`) |
| **Kapsam dışı** | Gerçek SMS gönderimi, branş öğretmeni, e-postadan yanıt, birebir randevu, geçmiş toplantıyı geriye dönük kaydetme (S-5) |

> **Okuma notu.** Her iddia kod taramasıyla doğrulandı; dosya yolları `oksis-api-veli-toplantilari/` köküne görelidir. `[D]` = uygulamada doğrulanacak küçük ayrıntı, `[F-n]` = Faruk'a açık soru (§17). Güvenlik şartları metinde **`[GŞ-K1]`** biçiminde, ilgili bölümün içinde işlenmiştir ve §15'te teste bağlanmıştır.

---

## 0. Kodda doğrulanan zemin (2026-10-15)

| Konu | Kodun söylediği | Bu modüle etkisi |
|---|---|---|
| Ödev emsali | `Domain/Modules/Homework/Entities/Homework.cs` — `TenantEntity` kökü, alt koleksiyonlar arka alanla (`PropertyAccessMode.Field`), domain olayı yok; bildirim `SaveChanges` sonrası handler'da kuyruklanır (`PublishHomeworkCommandHandler`) | Aynı kalıp: kök üzerinden yazma, bildirim kayıttan sonra |
| Tablo yeri | `ToAcademicTable("homework")`, snake_case kolonlar, filtreli unique (`is_deleted = 0`), `RowVersion` | Modül `academic` şemasında |
| Temel tipler | `TenantEntity` (soft delete + audit + RowVersion), `PermanentTenantEntity` (soft delete YOK) | Geçmiş ve denetim tabloları `PermanentTenantEntity` |
| Rehber öğretmen | `ClassRoom.HomeroomTeacherId : Guid?` = **Person.Id**; `ClassRoomHomeroomChangedEvent` / `…RemovedEvent` var | Sorumlu dondurulmaz, her istekte okunur |
| Veli süzgeci | `NotificationRecipientResolver.GuardianRelations`: `RevokedAt == null` + (varsayılan) `CanViewInfo`; **`ValidFrom`/`ValidUntil` denetlenmiyor** | §11.4 |
| Çağıran | `Application/Common/Security/CallerPerson.ResolveIdAsync` + bekçi `CallerPersonSchoolPredicateTests` | Tüm kapsam kapıları bunu kullanır `[GŞ-Y1]` |
| Bildirim hattı | `INotificationEnqueuer` → `DispatchNotificationJob` → `NotificationDispatcher` (kanal başına `NotificationDeliveryLog(EventId, Recipient, Channel)` idempotentliği) → `InApp` / `Push` (toplu) / `Email` | Kanal kararı komutta yok (`NoChannelDecisionInCommandsTests`) |
| Push kapıları | `PushNotificationChannel`: kapsam (`PushEventKeyMap`) → katalog `DefaultPushEnabled` → alıcı tercihi (satır yoksa açık) → sessiz saat (**erteler**, `DeferredPushJob`; yalnız `AnnouncementUrgentPublished` atlar) | N-06 için "atla" kipi yok → §8.4 |
| E-posta kanalı | `EmailNotificationChannel.RecipientCanChooseEmail => false`; kapsam kapısı **`PushEventKeyMap`** üzerinden (push dışı türler e-postaya hiç giremez) | §9 |
| Katalog | `NotificationEventTypeSeedData`: `DefaultEmailEnabled = true` olan satırlar ATT_THRESHOLD, GRADE_PUBLISHED, REPORT_CARD_PUBLISHED, PAYMENT_REMINDER, PAYMENT_RECEIVED, ANNOUNCEMENT_URGENT, HOMEWORK_MISSING | E-44 bunları sessizce açmamalı `[GŞ-K2]` |
| Tercih | `NotificationPreference(AccountId, EventKey, PushEnabled)`; satır yoksa açık | `EmailEnabled bool?` eklenir |
| SMS | `Application/Common/Abstractions/ISmsSender` var (OTP iskeleti), uygulaması yok | §10 |
| Deterministik kimlik | `Application/Common/Utilities/DeterministicGuid.Combine(...)`; günler `yyyy-MM-dd` Invariant (`HomeworkDueReminderJob`) | §8.3 |
| İşler | `Api/Extensions/HangfireSetup.cs` sabit cron'lar (Europe/Istanbul), kademelendirme kuralı; iş içinde `SetForLoginFlow` + açık `SchoolId` + `ChangeTracker.Clear()` + `IPostCommitDispatcher.RunPendingAsync` | §8.5 |
| İzin | `PermissionSeedData` + `RolePermissionSeedData` (`MirrorVicePrincipal` SchoolAdmin satırlarını VP'ye kopyalar) | §6 |
| Modül kataloğu | `SeedDefaultModuleConfigsHandler.DefaultCatalog` (11 anahtar, **homework yok**) + UI `packages/core/src/modules/constants.ts` `MODULE_CATALOG`. **Backend'de modül kapısı yok** (`RequireModule` benzeri hiçbir şey yok) — modül kapatma yalnız UI'yı gizler | §6.3 |
| Sezon | `AcademicSessionArchivedEvent(SessionId, SchoolId, ArchivedAt)`, `AcademicSession.ArchivedAt` | BR-PM-42, saklama sayacı |
| Kayıt olayları | `StudentAssignedToClassRoomEvent`, `StudentTransferredEvent`, `ParentStudentLinkedEvent`, `ParentStudentRevokedEvent`, `PersonArchivedEvent`. **`CanViewInfo` değişikliği olay üretmiyor** (`UpdateRelationshipPermissions`) | Davetli uzlaştırma işi gerekli (§5.4) |
| Kaynak gizleme | `NotificationSource(Type, Id)` + `NotificationSourceTypes` + `Notification.Hide()` (duyuru emsali) | Tazelik denetimi bu kaynağa dayanır (§8.3) |
| Hız sınırı | `RateLimitingSetup.cs`: `authenticated` politikası tanımlı ama bölümlenmemiş, hiçbir uca bağlı değil | `[GŞ-Y6]` §7.5 |
| Loglama | `RequestLoggingBehavior` beklenen hatalarda `ex.Message`'ı loglar; `StructuredAuditLogger` (Elasticsearch) | Not metni hiçbir log/olay mesajına girmez `[GŞ-K3, Y4]` |
| E-posta sağlayıcısı | `appsettings.json` → `Smtp:Host = smtp-mail.outlook.com` (kişisel Hotmail) | `[GŞ-K1]` üretim kapısı, `[F-1]` |

**Kodla çelişen belge bulgusu:** ADR 0012'nin "Uygulama durumu" notu e-postanın "kapsam listesindeki olaylarda çalıştığını" söylüyor; kod 2026-10-09'dan beri e-postayı tüm olaylarda kapalı tutuyor. ADR 0027 kabul edilince 0012'nin notu güncellenmeli.

---

## 1. Modül sınırı ve şema yeri

| Karar | Değer |
|---|---|
| Modül adı (backend) | `ParentMeetings` — `Domain/Modules/ParentMeetings`, `Application/Modules/ParentMeetings`, `Infrastructure/Persistence/Configurations/ParentMeetings` |
| Şema | `academic` (`ToAcademicTable`) — ödev gibi sezon × şube koordinatlı süreç verisi. SMS niyet tablosu `notifications` şemasında (Notifications modülünün malı) |
| Rota kökü | `api/v1/parent-meetings` |
| İzin öneki | `parentMeetings.*` |
| Modül kataloğu anahtarı | **`parentMeetings`** öneriyorum (mevcut anahtarlar çoğul: `announcements`, `clubs`); talepteki `parentMeeting` de çalışır — `[F-9]` |

**Bağımlılık yönü (yalnız soyutlama / kimlik):**

```
ParentMeetings ──okur──▶ AcademicSessions (AcademicSession, ClassRoom.HomeroomTeacherId, ClassRoomStudent)
               ──okur──▶ Users            (Person, Profile, ParentStudentRelationship, RoleAssignment)  — yalnız Id + IGuardianRelationFilter
               ──okur──▶ Schools          (SchoolSettings, ISchoolCalendarService)
               ──yazar─▶ Notifications    (INotificationEnqueuer, INotificationRecipientResolver, ISmsIntentRecorder — soyutlama)
               ◀─dinler─ AcademicSessions/Users domain olayları (§5.4, BR-PM-42)
Notifications  ──çağırır▶ INotificationSourceFreshness  (ParentMeetings kaydeder; Notifications modülü ParentMeetings'i BİLMEZ)
```

Hiçbir mevcut modül ParentMeetings'e bağımlı olmaz. Notifications → ParentMeetings yönündeki tek temas, Notifications'ın tanımladığı bir porttur (`INotificationSourceFreshness`), uygulaması ParentMeetings'tedir (bağımlılığın tersine çevrilmesi).

---

## 2. Aggregate tasarımı

### 2.1 Sınırlar

```
ParentMeeting (kök — "organizasyon", çatı)                  [academic].parent_meetings
   yalnız başlık/açıklama/varsayılanlar + Draft|Published

ParentMeetingSection (kök — şube toplantısı)                [academic].parent_meeting_sections
├── ParentMeetingInvitee        (çocuk; öğrenci × veli)     [academic].parent_meeting_invitees
├── ParentMeetingResponse       (çocuk; veli × plan sürümü) [academic].parent_meeting_responses
├── ParentMeetingPresence       (çocuk; veli)               [academic].parent_meeting_presences
└── (append-only, kök dışı) ParentMeetingResponseHistory    [academic].parent_meeting_response_history
                            ParentMeetingAuditEntry         [academic].parent_meeting_audit_entries

ParentMeetingNote (kök — öğrenci başına görüşme notu)       [academic].parent_meeting_notes
├── ParentMeetingNoteVersion    (çocuk; sürüm)              [academic].parent_meeting_note_versions
└── (append-only, kök dışı) ParentMeetingNoteAuditEntry     [academic].parent_meeting_note_audit_entries

SmsIntent (Notifications modülü, kök)                       [notifications].sms_intents
```

**Neden üç kök** (ayrıntı [[0025-sube-toplantisi-ayri-agregat]]):

- **Organizasyon ↔ şube ayrı kök.** K-2 her şubenin tarihini, iptalini ve durumunu bağımsız kılar; organizasyon yalnız "aynı sihirbazdan doğdular" bilgisidir. Tek kök olsaydı 40 şubelik okulda bir şubenin "Geldi" işareti diğer 39 şubenin yazmalarıyla aynı `RowVersion` üzerinde yarışırdı ve otomatik işler (başlat/kapat) bütün organizasyonu yüklerdi.
- **Not ayrı kök.** En dar erişimli, okuma denetimli, süreli imha edilen veridir; şube toplantısını yükleyen hiçbir yol (liste, rapor, bildirim) not metnini yanlışlıkla getiremez. Yazma sıklığı (otomatik kayıt) katılım işaretlemesinden bağımsızdır.
- **Yanıt şubenin çocuğu.** Yanıtın her invariantı şubenin durumuna, plan sürümüne ve davetli listesine bağlıdır. Yüksek yazma hacmine karşı handler kökü **filtreli Include** ile yükler (yalnız o velinin güncel sürüm satırı + o velinin davetli satırları) — ödevdeki "satırı kökten hedefle" kalıbı.

### 2.2 Varlıklar

**`ParentMeeting`** (`TenantEntity`)

| Alan | Tip | Not |
|---|---|---|
| `Id`, `SchoolId` | Guid | tenant filtresi |
| `AcademicSessionId` | Guid | sezon sert bölümü |
| `Title` | nvarchar(200) | zorunlu |
| `Description` | nvarchar(2000)? | e-postaya **girmez** (`[F-6]`) |
| `DefaultMeetingDate` | date | yalnız varsayılan (BR-PM-03) |
| `DefaultStartTime` / `DefaultEndTime` | time(0) / time(0)? | |
| `DefaultLocation` | nvarchar(200) | |
| `Status` | `ParentMeetingStatus` | `Draft` \| `Published` |
| `PublishedAt`, `PublishedByPersonId` | | |
| `IsDeleted`, `RowVersion` | | yalnız taslak silinir |

Metotlar: `Create(...)`, `UpdateDraft(title, description, defaults)`, `UpdateHeader(title, description)` (yayında; sessiz, denetimli), `MarkPublished(now, actor)`, `EnsureDeletable()` (yalnız `Draft`).

**`ParentMeetingSection`** (`TenantEntity`)

| Alan | Tip | Not |
|---|---|---|
| `Id`, `SchoolId`, `ParentMeetingId` | Guid | `ParentMeetingId` yalnız kimlik (navigasyon yok) |
| `AcademicSessionId` | Guid | denormalize — iş sorguları organizasyona join etmesin |
| `ClassRoomId` | Guid | |
| `MeetingDate` | date | okul-yerel gün |
| `StartTime` / `EndTime` | time(0) / time(0)? | `EndTime > StartTime` (BR-PM-03) |
| `StartsAt` / `EndsAt` | datetimeoffset / datetimeoffset? | yazma anında okul saat diliminden hesaplanır; işlerin tek sorgu ekseni |
| `Location` | nvarchar(200) | |
| `Status` | `ParentMeetingSectionStatus` | `Draft`/`Published`/`Started`/`Completed`/`Cancelled` |
| `ScheduleRevision` | int | 1'den başlar; yalnız tarih/saat değişince +1 (ADR 0026) |
| `LocationRevision` | int | yer değişince +1; N-03 yer gövdesinin tekilleştirme anahtarı |
| `PublishedAt`, `StartedAt`, `InviteesFrozenAt` | datetimeoffset? | `InviteesFrozenAt = StartedAt` |
| `CompletedAt`, `CompletedByPersonId?`, `CompletionKind?` | | `Manual` \| `Auto` \| `SeasonArchive`; sistemde aktör null |
| `CancelledAt`, `CancelledByPersonId?`, `CancelReason?` | nvarchar(500)? | gerekçe **yalnız idare** DTO'sunda; veliye/bildirime girmez |
| `DayReminderSentRevision`, `ResponsePendingSentRevision`, `StartingNotifiedRevision` | int? | iş idempotentliği: "bu sürüm için gönderildi" (ödevin `DueReminderSentAt` dersi: damga sürüme bağlı, erteleme onu kendiliğinden geçersiz kılar) |
| `IsDeleted`, `RowVersion` | | yalnız `Draft` silinir |

Türetilen (kolonsuz, EF `Ignore`): `EditableUntil = CompletedAt + 7 gün`, `IsFieldDataOpen(now)`, `IsParentResponseOpen(now)`.

**`ParentMeetingInvitee`** (`TenantEntity`, şubenin çocuğu) — `SectionId`, `StudentPersonId`, `ParentPersonId`, `InvitedAt`, `InvitedAtRevision`, `AddedAfterPublish bit`, `DroppedAt?`, `DropReason?` (`RelationshipRevoked` \| `InfoRestricted` \| `RelationshipExpired` \| `StudentLeftSection` \| `StudentLeftSchool`). Tekillik `(section_id, student_person_id, parent_person_id)`; düşen satır yeniden davet edilirse **aynı satır** yeniden açılır (`DroppedAt = null`, `AddedAfterPublish = true`).

**`ParentMeetingResponse`** (`TenantEntity`, şubenin çocuğu) — `SectionId`, `ParentPersonId`, `Revision`, `Status` (`Attending` \| `NotAttending`), `DeclineReason nvarchar(500)?`, `RespondedAt`, `RespondedByPersonId`, `IsOnBehalf bit`. Tekillik `(section_id, parent_person_id, revision)`. **"Yanıt yok" satır yokluğudur**; güncel yanıt = `Revision == section.ScheduleRevision` olan satır.

**`ParentMeetingResponseHistory`** (`PermanentTenantEntity`, append-only `[GŞ-D1]`) — `SectionId`, `ParentPersonId`, `Revision`, `FromStatus` (`noResponse`/`attending`/`notAttending`), `ToStatus`, `DeclineReason?` (yeni değerin nedeni), `ActorPersonId?` (null = sistem), `OnBehalfOfPersonId?` (idare girdiyse = `ParentPersonId`), `Cause` (`Self` \| `OnBehalf` \| `RescheduleReset`), `ChannelNote nvarchar(200)?` ("telefonla"), `OccurredAt`, `CorrelationId`. Domain'de yalnız fabrika metodu vardır; güncelleme/silme metodu yoktur, `IApplicationDbContext` üzerinden `Update`/`Remove` çağrısı mimari testle yasaklanır.

**`ParentMeetingPresence`** (`TenantEntity`, şubenin çocuğu) — `SectionId`, `ParentPersonId`, `IsPresent bit`, `MarkedAt`, `MarkedByPersonId`. Tekillik `(section_id, parent_person_id)`. Satır yoksa "işaretlenmedi"; `Completed` olunca işaretsiz = "gelmedi" (türetilir, yazılmaz). İkiz çocuklu velinin tek satırı vardır (BR-PM-16 ile simetrik).

**`ParentMeetingAuditEntry`** (`PermanentTenantEntity`, append-only) — `SectionId?`, `ParentMeetingId`, `Kind`, `ActorPersonId?` (null = sistem `[GŞ-D2]`), `OccurredAt`, `FromStatus?`, `ToStatus?`, `Detail nvarchar(1000)?` (yapısal JSON: eski/yeni tarih, sürüm, sayılar — **not metni, neden, gerekçe dışında**; iptal gerekçesi yalnız `Cancelled` satırında, yalnız idare okur). Türler: `Created`, `Published`, `SectionAdded`, `Rescheduled`, `LocationChanged`, `HeaderEdited`, `Cancelled`, `Started`, `Completed`, `AutoCompleted`, `SeasonArchived`, `InviteeAdded`, `InviteeDropped`, `PresenceMarked`, `PresenceUnmarked`.

**`ParentMeetingNote`** (`TenantEntity`, kök)

| Alan | Tip | Not |
|---|---|---|
| `Id`, `SchoolId`, `SectionId`, `StudentPersonId` | Guid | tekillik `(section_id, student_person_id)` |
| `AcademicSessionId` | Guid | saklama sayacı (sezon `ArchivedAt`) için denormalize |
| `Body` | nvarchar(4000)? | **null = imha edildi** ya da henüz boş; soft delete değil `[GŞ-Y5]` |
| `CurrentVersion` | int | iyimser eşzamanlılık (istemci `baseVersion` yollar) |
| `CreatedByPersonId`, `LastEditedByPersonId`, `LastEditedAt` | | |
| `PurgedAt` | datetimeoffset? | imha damgası |
| `RowVersion` | | |

**`ParentMeetingNoteVersion`** (çocuk) — `NoteId`, `Version`, `Body nvarchar(4000)?`, `EditorPersonId`, `CreatedAt`, `LastSavedAt`. **Otomatik kayıt birleştirme `[GŞ-O4]`:** aynı yazar son sürümü 5 dk içinde yeniden kaydederse yeni sürüm açılmaz, son sürümün gövdesi ve `LastSavedAt`'i güncellenir. İmhada `Body = NULL`.

**`ParentMeetingNoteAuditEntry`** (`PermanentTenantEntity`, append-only `[GŞ-K3]`) — `Id`, `SchoolId`, `NoteId?` (okuma, henüz not yokken de yazılır), `SectionId`, `StudentPersonId`, `ActorPersonId?` (null = sistem imhası), `ActorRole` (`Homeroom` \| `Author` \| `Admin` \| `System`), `Action` (`Read` \| `Create` \| `Update` \| `Purge`), `At`, `Version?`, `CorrelationId`. **Metin, özet, karakter sayısı yok.** Not denetimi `IAuditLogger`/`StructuredAuditLogger`'dan **geçmez** (Elasticsearch, 5 yıl, okul dışı saklama); tenant kapsamlı tablodadır ve not imha edilse de kalır.

### 2.3 Enum'lar (wire'da string, camelCase)

`ParentMeetingStatus` (`draft|published`), `ParentMeetingSectionStatus` (`draft|published|started|completed|cancelled`), `MeetingResponseStatus` (`attending|notAttending|noResponse` — `noResponse` yalnız wire'da), `MeetingPresenceStatus` (`present|absent|unmarked` — wire türevi), `CompletionKind` (`manual|auto|seasonArchive`), `InviteeDropReason`, `ResponseChangeCause` (`self|onBehalf|rescheduleReset`), `ParentMeetingAuditKind`, `NoteAuditAction` (`read|create|update|purge`), `NoteAuditActorRole`. Sayılar kolonda kalıcıdır; sona eklenir (`NotificationEventGroup` kuralı).

### 2.4 Invariant'lar

| # | Invariant | Yeri |
|---|---|---|
| I-1 | Organizasyon en az bir şube içerir; aynı organizasyonda bir şube en çok bir kez (BR-PM-02) | `CreateParentMeeting` doğrulayıcısı + filtreli unique `(parent_meeting_id, class_room_id)` |
| I-2 | `EndTime` varsa `> StartTime` (BR-PM-03) | `ParentMeetingSection.Schedule(...)` |
| I-3 | Oluşturma/yayın/tarih değişikliğinde `StartsAt > now` (BR-PM-04, S-5: "geçmiş **an** olamaz") | domain metotları `now` alır; okul saat dilimi handler'da çözülür |
| I-4 | Arşiv sezonda yazma yok (BR-PM-08) | `active-season-write` politikası (sınıf düzeyi) + handler'da sezon `Archived` → 409 |
| I-5 | Yalnız `Draft` silinir; yayınlanan şube organizasyondan çıkarılamaz (BR-PM-05/06) | `EnsureDeletable`, `RemoveDraftSection` |
| I-6 | Veli kendi yanıtını yalnız `Published` **ve** `now < StartsAt` iken verir (BR-PM-19) — 5 dk iş gecikmesi kilidi açmaz | `IsParentResponseOpen(now)` |
| I-7 | Veli adına yanıt yalnız `Published`'da (BR-PM-19, §17.1a) ve hedef veli **aktif davetli** olmalı `[GŞ-Y2]` | `RecordOnBehalfResponse` |
| I-8 | Fiili katılım ve not yalnız `Started` ya da `Completed && now < EditableUntil` (BR-PM-10, BR-PM-30) | `IsFieldDataOpen(now)`; not handler'ı şubeden okur |
| I-9 | `Started → Published` (erteleme) yalnız hiç `IsPresent` satırı ve hiç dolu not yokken | domain `hasFieldData` parametresi alır (not ayrı kök; sorgu handler'da) |
| I-10 | `Completed`, `Cancelled` terminal | durum makinesi |
| I-11 | Davetli listesi `Started` sonrası değişmez (BR-PM-15/41) | `Reconcile` `InviteesFrozenAt != null` iken no-op |
| I-12 | Yanıt geçmişi, şube denetimi, not denetimi append-only | tip (`PermanentTenantEntity`) + fabrika-yalnız + mimari test |
| I-13 | Not gövdesi ≤ 4000, kontrol karakterleri temizlenir (`\t\n` hariç), düz metin `[GŞ-O4]` | `ParentMeetingNote.Write` |

### 2.5 Durum makinesi ve geçiş metotları (`ParentMeetingSection`)

```
          publish                start (sistem)             complete (öğr./idare | auto 3g | sezon)
 Draft ───────────▶ Published ─────────────────▶ Started ─────────────────────────────▶ Completed
   │ delete(soft)     │  ▲  │ reschedule (rev+1)    │                                    (7 gün düzenleme)
   ▼                  │  └──┘ changeLocation        │ reschedule (alan verisi yoksa, rev+1) ─▶ Published
 (silindi)            │                             │
                      └──── cancel ─────▶ Cancelled ◀──── cancel ────┘
```

| Metot | Ön koşul | Etki | Dönen |
|---|---|---|---|
| `Create(…, schedule, now)` | I-2, I-3 | `Draft`, `ScheduleRevision = 1`, `LocationRevision = 1` | |
| `UpdateDraftSchedule(date, start, end, location, startsAt, now)` | `Draft` | sürüm artmaz (kimseye ulaşmadı) | |
| `Publish(invitees, now, actor)` | `Draft`, I-3 yeniden | `Published`, davetliler açılır, `PublishedAt` | `PublishResult(inviteeCount)` |
| `PublishDirect(...)` | organizasyon `Published` iken eklenen şube (BR-PM-06) | `Create` + `Publish` tek adım | |
| `Reschedule(date, start, end, location, startsAt, now, actor, hasFieldData)` | `Published` ya da (`Started` && `!hasFieldData`); I-2, I-3 | tarih/saat değiştiyse `ScheduleRevision++`, durum `Published`'a döner, damgalar kendiliğinden geçersiz (sürüme bağlı), güncel yanıtı olan her veli için `RescheduleReset` geçmiş satırı; yalnız yer değiştiyse `LocationRevision++` | `RescheduleOutcome(ScheduleChanged, LocationChanged, ResetParentIds)` |
| `Start(now)` | `Published` && `now >= StartsAt` | `Started`, `InviteesFrozenAt` | |
| `Complete(now, actor?, kind)` | `Started` | `Completed` + kind | |
| `Cancel(now, actor?, reason?)` | `Published` \| `Started` | `Cancelled`; kayıtlar korunur | |
| `ReconcileInvitees(current, now)` | `Published` (değilse no-op) | ekler/düşürür | `InviteeDiff(Added, Dropped)` |
| `RecordParentResponse(parentId, status, reason, now)` | I-6, aktif davetli | güncel sürüm satırı upsert + geçmiş | `ResponseChange?` (değişmediyse null — çift dokunuş geçmiş üretmez) |
| `RecordOnBehalfResponse(parentId, status, reason, channelNote, actor, now)` | I-7 | aynı + `IsOnBehalf` | |
| `MarkPresence(parentId, isPresent, actor, now)` | I-8, aktif (dondurulmuş) davetli | upsert presence | |
| `HasFieldData()` | | yüklü presence'tan | |

Organizasyon: `Draft → Published` (`ParentMeetingPublishService` tüm `Draft` şubeleri aynı işlemde yayınlar); organizasyonun "tamamlandı/iptal" özeti şubelerden türetilir, saklanmaz.

### 2.6 Domain olayları

Ödev kalıbı izlenir: **bildirim için domain olayı açılmaz**, yazma handler'ı `SaveChanges` sonrası modülün `ParentMeetingNotifier`'ını çağırır (geri alınamayan tek şey haberdir). Not hiçbir domain olayına girmez `[GŞ-Y4]`.

---

## 3. DDL taslağı (MSSQL, `academic` şeması)

> EF Core fluent konfigürasyonla üretilecek; aşağıdaki SQL göç çıktısının hedef şeklidir. Tüm tablolarda `school_id` + tenant global filtresi; Dapper/ham sorgularda açık `WHERE school_id = @schoolId`.

```sql
CREATE TABLE academic.parent_meetings (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  academic_session_id uniqueidentifier NOT NULL,
  title nvarchar(200) NOT NULL,
  description nvarchar(2000) NULL,
  default_meeting_date date NOT NULL,
  default_start_time time(0) NOT NULL,
  default_end_time time(0) NULL,
  default_location nvarchar(200) NOT NULL,
  status int NOT NULL,                       -- 1 Draft, 2 Published
  published_at datetimeoffset NULL,
  published_by_person_id uniqueidentifier NULL,
  created_at datetimeoffset NOT NULL, created_by uniqueidentifier NOT NULL,
  updated_at datetimeoffset NULL, updated_by uniqueidentifier NULL,
  is_deleted bit NOT NULL DEFAULT 0, deleted_at datetimeoffset NULL, deleted_by uniqueidentifier NULL,
  row_version rowversion NOT NULL
);
CREATE INDEX ix_parent_meetings_school_session_status
  ON academic.parent_meetings (school_id, academic_session_id, status, default_meeting_date) WHERE is_deleted = 0;

CREATE TABLE academic.parent_meeting_sections (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  parent_meeting_id uniqueidentifier NOT NULL
    REFERENCES academic.parent_meetings(id),          -- ON DELETE NO ACTION (soft delete)
  academic_session_id uniqueidentifier NOT NULL,
  class_room_id uniqueidentifier NOT NULL,             -- cross-modül: FK YOK, yalnız kimlik (ödev emsali)
  meeting_date date NOT NULL,
  start_time time(0) NOT NULL,
  end_time time(0) NULL,
  starts_at datetimeoffset NOT NULL,
  ends_at datetimeoffset NULL,
  location nvarchar(200) NOT NULL,
  status int NOT NULL,                               -- 1 Draft … 5 Cancelled
  schedule_revision int NOT NULL DEFAULT 1,
  location_revision int NOT NULL DEFAULT 1,
  published_at datetimeoffset NULL,
  started_at datetimeoffset NULL,
  invitees_frozen_at datetimeoffset NULL,
  completed_at datetimeoffset NULL, completed_by_person_id uniqueidentifier NULL, completion_kind int NULL,
  cancelled_at datetimeoffset NULL, cancelled_by_person_id uniqueidentifier NULL, cancel_reason nvarchar(500) NULL,
  day_reminder_sent_revision int NULL,
  response_pending_sent_revision int NULL,
  starting_notified_revision int NULL,
  /* audit + soft delete + row_version (TenantEntity) */
  CONSTRAINT ck_pm_sections_end_after_start CHECK (end_time IS NULL OR end_time > start_time)
);
CREATE UNIQUE INDEX ux_pm_sections_meeting_classroom
  ON academic.parent_meeting_sections (parent_meeting_id, class_room_id) WHERE is_deleted = 0;
CREATE INDEX ix_pm_sections_school_status_starts            -- başlatma + hatırlatma işleri
  ON academic.parent_meeting_sections (school_id, status, starts_at) WHERE is_deleted = 0;
CREATE INDEX ix_pm_sections_school_classroom_date           -- rehber "Toplantılarım", veli kartları
  ON academic.parent_meeting_sections (school_id, class_room_id, meeting_date) WHERE is_deleted = 0;

CREATE TABLE academic.parent_meeting_invitees (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  section_id uniqueidentifier NOT NULL REFERENCES academic.parent_meeting_sections(id) ON DELETE CASCADE,
  student_person_id uniqueidentifier NOT NULL,
  parent_person_id uniqueidentifier NOT NULL,
  invited_at datetimeoffset NOT NULL,
  invited_at_revision int NOT NULL,
  added_after_publish bit NOT NULL DEFAULT 0,
  dropped_at datetimeoffset NULL,
  drop_reason int NULL
  /* + TenantEntity kolonları */
);
CREATE UNIQUE INDEX ux_pm_invitees_section_student_parent
  ON academic.parent_meeting_invitees (section_id, student_person_id, parent_person_id) WHERE is_deleted = 0;
CREATE INDEX ix_pm_invitees_school_parent_active            -- veli "Toplantılarım"
  ON academic.parent_meeting_invitees (school_id, parent_person_id) INCLUDE (section_id, student_person_id)
  WHERE is_deleted = 0 AND dropped_at IS NULL;

CREATE TABLE academic.parent_meeting_responses (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  section_id uniqueidentifier NOT NULL REFERENCES academic.parent_meeting_sections(id) ON DELETE CASCADE,
  parent_person_id uniqueidentifier NOT NULL,
  revision int NOT NULL,
  status int NOT NULL,                       -- 1 Attending, 2 NotAttending
  decline_reason nvarchar(500) NULL,
  responded_at datetimeoffset NOT NULL,
  responded_by_person_id uniqueidentifier NOT NULL,
  is_on_behalf bit NOT NULL
  /* + TenantEntity kolonları */
);
CREATE UNIQUE INDEX ux_pm_responses_section_parent_revision
  ON academic.parent_meeting_responses (section_id, parent_person_id, revision) WHERE is_deleted = 0;
CREATE INDEX ix_pm_responses_section_revision_status
  ON academic.parent_meeting_responses (section_id, revision, status) WHERE is_deleted = 0;

CREATE TABLE academic.parent_meeting_response_history (   -- PermanentTenantEntity: is_deleted YOK
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  section_id uniqueidentifier NOT NULL REFERENCES academic.parent_meeting_sections(id),
  parent_person_id uniqueidentifier NOT NULL,
  revision int NOT NULL,
  from_status int NOT NULL, to_status int NOT NULL,       -- 0 NoResponse
  decline_reason nvarchar(500) NULL,
  actor_person_id uniqueidentifier NULL,
  on_behalf_of_person_id uniqueidentifier NULL,
  cause int NOT NULL,
  channel_note nvarchar(200) NULL,
  occurred_at datetimeoffset NOT NULL,
  correlation_id nvarchar(64) NULL,
  created_at datetimeoffset NOT NULL, created_by uniqueidentifier NOT NULL,
  updated_at datetimeoffset NULL, updated_by uniqueidentifier NULL,
  row_version rowversion NOT NULL
);
CREATE INDEX ix_pm_response_history_section_parent
  ON academic.parent_meeting_response_history (section_id, parent_person_id, occurred_at);

CREATE TABLE academic.parent_meeting_presences (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  section_id uniqueidentifier NOT NULL REFERENCES academic.parent_meeting_sections(id) ON DELETE CASCADE,
  parent_person_id uniqueidentifier NOT NULL,
  is_present bit NOT NULL,
  marked_at datetimeoffset NOT NULL,
  marked_by_person_id uniqueidentifier NOT NULL
  /* + TenantEntity kolonları */
);
CREATE UNIQUE INDEX ux_pm_presences_section_parent
  ON academic.parent_meeting_presences (section_id, parent_person_id) WHERE is_deleted = 0;

CREATE TABLE academic.parent_meeting_audit_entries (      -- PermanentTenantEntity
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  parent_meeting_id uniqueidentifier NOT NULL,
  section_id uniqueidentifier NULL,
  kind int NOT NULL,
  actor_person_id uniqueidentifier NULL,
  occurred_at datetimeoffset NOT NULL,
  from_status int NULL, to_status int NULL,
  detail nvarchar(1000) NULL
  /* + PermanentTenantEntity kolonları */
);
CREATE INDEX ix_pm_audit_section_occurred
  ON academic.parent_meeting_audit_entries (school_id, section_id, occurred_at DESC);

CREATE TABLE academic.parent_meeting_notes (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  section_id uniqueidentifier NOT NULL REFERENCES academic.parent_meeting_sections(id),
  academic_session_id uniqueidentifier NOT NULL,
  student_person_id uniqueidentifier NOT NULL,
  body nvarchar(4000) NULL,
  current_version int NOT NULL,
  created_by_person_id uniqueidentifier NOT NULL,
  last_edited_by_person_id uniqueidentifier NOT NULL,
  last_edited_at datetimeoffset NOT NULL,
  purged_at datetimeoffset NULL
  /* + TenantEntity kolonları; soft delete kullanılmaz (Body NULL'lanır) */
);
CREATE UNIQUE INDEX ux_pm_notes_section_student
  ON academic.parent_meeting_notes (section_id, student_person_id) WHERE is_deleted = 0;
CREATE INDEX ix_pm_notes_retention                          -- imha işi
  ON academic.parent_meeting_notes (school_id, academic_session_id) WHERE purged_at IS NULL;

CREATE TABLE academic.parent_meeting_note_versions (
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  note_id uniqueidentifier NOT NULL REFERENCES academic.parent_meeting_notes(id) ON DELETE CASCADE,
  version int NOT NULL,
  body nvarchar(4000) NULL,
  editor_person_id uniqueidentifier NOT NULL,
  created_at_version datetimeoffset NOT NULL,
  last_saved_at datetimeoffset NOT NULL
  /* + TenantEntity kolonları */
);
CREATE UNIQUE INDEX ux_pm_note_versions_note_version ON academic.parent_meeting_note_versions (note_id, version);
CREATE INDEX ix_pm_note_versions_editor ON academic.parent_meeting_note_versions (school_id, editor_person_id);  -- "yazan" kapısı

CREATE TABLE academic.parent_meeting_note_audit_entries (  -- PermanentTenantEntity, metin YOK
  id uniqueidentifier NOT NULL PRIMARY KEY,
  school_id uniqueidentifier NOT NULL,
  note_id uniqueidentifier NULL,
  section_id uniqueidentifier NOT NULL,
  student_person_id uniqueidentifier NOT NULL,
  actor_person_id uniqueidentifier NULL,
  actor_role int NOT NULL,
  action int NOT NULL,
  at datetimeoffset NOT NULL,
  version int NULL,
  correlation_id nvarchar(64) NULL
  /* + PermanentTenantEntity kolonları */
);
CREATE INDEX ix_pm_note_audit_school_at ON academic.parent_meeting_note_audit_entries (school_id, at DESC);
CREATE INDEX ix_pm_note_audit_student   ON academic.parent_meeting_note_audit_entries (school_id, student_person_id, at DESC);
```

**`school_settings` +4 kolon** (`SchoolSettings.UpdateParentMeetingSettings(...)`):

| Kolon | Tip | Varsayılan | Aralık |
|---|---|---|---|
| `parent_meeting_morning_reminder_time` | time(0) | 08:00 | 05:00–12:00 |
| `parent_meeting_starting_notification_enabled` | bit | 0 | — |
| `parent_meeting_response_pending_enabled` | bit | 1 (S-1) | — |
| `parent_meeting_note_retention_years` | int | 2 | 0–10 (BR-PM-33) |

Sistem sabitleri (kodda, `ParentMeetingPolicy`): `AutoCompleteAfterDays = 3`, `EditWindowDays = 7`, `ResponsePendingDaysBefore = 2`, `ConflictDefaultDuration = 60 dk`, `MaxNoteLength = 4000`, `NoteAutosaveMergeWindow = 5 dk`.

**`notifications.sms_intents`** (B7, §10), **`notifications.notification_preferences` +1**, **`master.notification_event_types` +1** (B6, §9) ayrı göçlerdedir.

---

## 4. Yetki ve kapsam kapıları

### 4.1 İzin kodları

| Kod | Açıklama (seed) | SchoolAdmin | VicePrincipal | Teacher | Parent | Student | SuperAdmin |
|---|---|---|---|---|---|---|---|
| `parentMeetings.read` | Veli toplantısını görüntüle (kapsamla daralır) | ✓ | ✓ (ayna) | ✓ | ✓ | — (S-2) | — |
| `parentMeetings.write` | Fiili katılım, görüşme notu, tamamlama (rehber kapsamında) | ✓ | ✓ | ✓ | — | — | — |
| `parentMeetings.manage` | Planlama, yayın, tarih, iptal, veli adına yanıt, rapor, not denetimi, ayarlar | ✓ | ✓ | — | — | — | — |

Üçü de **`AllPermissionIds()` kataloğunun dışında** (yalnız `SchoolAdmin` satırı + `MirrorVicePrincipal`): platform hesabı okul içi veli toplantısını, hele görüşme notunu görmemeli. Ödev `read`'i katalogdadır; burada farklılaşma bilinçlidir ve `[F-8]` olarak sunulur. Not için ayrı izin yok (K-11).

### 4.2 Kapsam kapısı (`ParentMeetingScope`)

İzin yüzeyi açar; kapsam kaydı belirler; ihlal **404** (varlık sızdırılmaz), izinsiz **403**. Çağıran her zaman `CallerPerson.ResolveIdAsync(db, accountId, schoolId)` ile çözülür ve **her sorguda açık `SchoolId` yüklemi** yazılır (süper yönetici oturumunda global filtre düşer — `TB-139/141`) `[GŞ-Y1]`.

| Görünüm | Koşul | Kaynak |
|---|---|---|
| `Admin` | `parentMeetings.manage` | izin |
| `Homeroom` | `ClassRoom.HomeroomTeacherId == callerPersonId` — **her istekte güncel değer**, kayıtta dondurulmaz (BR-PM-07) `[GŞ-Y2]` | `academic.class_rooms` |
| `NoteAuthor` | yalnız not uçlarında: çağıran notun herhangi bir sürümünü yazmış **ve** 0016 ölçütüyle çalışan öğretmen **ve** güncel sezonda aktif öğretmen rol ataması **ve** `parentMeetings.write` `[GŞ-Y3]`, S-4 ("okuldan ayrılınca biter") | `note_versions.editor_person_id`, `Profile`, `RoleAssignment` |
| `Family` | çağıranın **kendi** `PersonId`'si için aktif davetli satırı + `IGuardianRelationFilter.IsActive(today)` (§11.4) | `parent_meeting_invitees` + ilişki |

Kural: veli uçları istekten `parentPersonId` **almaz**; yanıt sahibi her zaman çağırandır `[GŞ-Y2]`. İstemciden rol/görünüm parametresi kabul edilmez.

### 4.3 Yetki × işlem (özet)

| İşlem | İzin | Kapsam | Durum |
|---|---|---|---|
| Planla/düzenle/sil/yayınla/şube ekle | manage | — | Draft/Published |
| Tarih, yer, iptal (tekil + toplu) | manage | — | Published/Started |
| Veli adına yanıt | manage | hedef veli aktif davetli | Published/Started |
| Şube detayı (yanıtlar, yanıtsızlar, hesapsızlar, telefon) | read | Admin \| Homeroom | ≠ Draft (Draft yalnız Admin) |
| Yanıt geçmişi (neden dahil) | read | Admin \| Homeroom | — |
| Fiili katılım, tamamlama | write | Admin \| Homeroom | I-8 |
| Not oku | write | Admin \| Homeroom \| NoteAuthor | her durumda (imha edilmemişse) |
| Not yaz | write | Admin \| Homeroom | I-8 |
| Not denetimi | manage | — | — |
| Veli kartları, yanıt | read | Family | I-6 |
| Rapor | read | Admin (okul) \| Homeroom (şubesi) | — |
| Ayarlar | read / manage | — | — |

---

## 5. Uygulama katmanı

### 5.1 Klasör yerleşimi

```
src/Oksis.Domain/Modules/ParentMeetings/
  Entities/   ParentMeeting · ParentMeetingSection · ParentMeetingInvitee · ParentMeetingResponse
              ParentMeetingResponseHistory · ParentMeetingPresence · ParentMeetingAuditEntry
              ParentMeetingNote · ParentMeetingNoteVersion · ParentMeetingNoteAuditEntry
  Enums/ · ValueObjects/ (MeetingSchedule, InviteeKey, RescheduleOutcome, InviteeDiff)
  Policies/   ParentMeetingPolicy (sabitler)
  Exceptions/ ParentMeetingDomainException (+ InvalidStateException, StartInPastException,
              ResponseLockedException, FieldDataClosedException, HasFieldDataException,
              NoteVersionConflictException) — mesajlar ASLA not metni/neden taşımaz

src/Oksis.Application/Modules/ParentMeetings/
  Abstractions/ IParentMeetingRosterReader · IParentMeetingGuardianReader
  Commands/    CreateParentMeeting · UpdateParentMeeting · DeleteParentMeeting · PublishParentMeeting
               AddParentMeetingSections · UpdateSectionPlan (tekil+toplu) · CancelSections (tekil+toplu)
               RecordOnBehalfResponse · RespondToParentMeeting (veli) · MarkPresence
               CompleteParentMeetingSection · SaveParentMeetingNote · UpdateParentMeetingSettings
  Queries/     ListParentMeetings · GetParentMeeting · GetPlanPreview · GetSectionPlanChangePreview
               GetParentMeetingSection · GetResponseHistory · ListMyParentMeetings (rehber)
               ListFamilyParentMeetings · GetFamilyParentMeetingSection · GetParentMeetingNote
               GetParentMeetingReport · ExportParentMeetingReport · ListNoteAudit
               GetParentMeetingFormContext · GetParentMeetingSettings
  Contracts/   DTO'lar (contract.ts ile alan alan eş)
  Internal/    ParentMeetingScope · ParentMeetingReader · ParentMeetingCounters · InviteeResolver
               ParentMeetingNotifier · ParentMeetingNotificationText · ParentMeetingEventIds
               ParentMeetingConflictDetector · ParentMeetingErrors · ParentMeetingAuditEvents
               ParentMeetingSourceFreshness (INotificationSourceFreshness uygulaması)
  Events/      SessionArchived… · ParentStudentLinked/Revoked… · StudentAssigned/Transferred…
               HomeroomChanged… (yalnız N-02 alıcısı için gerekmez — dinamik okunur; dinlenmez)

src/Oksis.Infrastructure/BackgroundJobs/Jobs/
  ParentMeetingStartJob · ParentMeetingReminderSweepJob · ParentMeetingAutoCompleteJob
  ParentMeetingInviteeReconcileJob · ParentMeetingNoteRetentionJob
src/Oksis.Infrastructure/Persistence/Configurations/ParentMeetings/*.cs (10)
src/Oksis.Api/Controllers/V1/ParentMeetingsAdminController.cs · ParentMeetingsTeacherController.cs
                             ParentMeetingsFamilyController.cs · ParentMeetingsReportController.cs
```

Dört denetleyici aynı `api/v1/parent-meetings` kökünü paylaşır; ayrım hem rol yüzeyini okunur kılar hem de paralel dilimlerin aynı dosyaya dokunmasını önler (§13).

### 5.2 Davetli çözümü (`InviteeResolver`)

1. Şubenin aktif öğrencileri (`IParentMeetingRosterReader` → `ClassRoomStudent` aktifleri; ödevdeki okuyucunun eşi, modül içinde).
2. Öğrenci başına veliler: `IParentMeetingGuardianReader` → `IGuardianRelationFilter.ActiveForInfo(today)` = `RevokedAt == null && CanViewInfo && ValidFrom <= today && (ValidUntil == null || ValidUntil >= today)` `[GŞ-Y1]`. `includeInfoRestricted` istisnası bu modüle **uygulanmaz** (BR-PM-14).
3. Sonuç `(StudentPersonId, ParentPersonId)` kümesi → `section.ReconcileInvitees`.
4. Hesap durumu (bildirim/önizleme için) ayrı: `INotificationRecipientResolver.ResolvePersonAccountsMapAsync`.

### 5.3 Kritik handler akışları

**`PublishParentMeetingCommandHandler`** — (1) manage; organizasyon `Draft`; `expectedRowVersion` eşleşmesi. (2) Okul saat dilimi `ISchoolCalendarService.GetLocalNowAsync`. (3) Her `Draft` şube: `InviteeResolver` → `section.Publish(invitees, now, actor)`; geçmiş an → 400 `ParentMeeting.StartInPast` (hangi şubeler olduğu `errors[].field = sections[i]`). (4) `meeting.MarkPublished`. (5) Denetim. (6) `SaveChanges` (tek işlem). (7) Kayıttan **sonra** `ParentMeetingNotifier.Published(sections)` → N-01 (veliler), N-02 (rehber), SMS niyeti (hesapsız).

**`UpdateSectionPlanCommandHandler`** (tekil ve toplu aynı komut: `sectionIds[]`) — manage; her şube için `expectedScheduleRevision` + `rowVersion`; `hasFieldData` sorgusu (presence `is_present = 1` ya da `notes.body IS NOT NULL`); `section.Reschedule(...)`; `RescheduleOutcome` → bildirim (§8). Önizleme (`GetSectionPlanChangePreview`) aynı hesap çekirdeğini (`SectionPlanChangeCalculator`) kullanır — önizleme ile uygulama ayrışamaz (BR-PM-17). Önizlemesiz değişiklik: istek `previewAcknowledged: true` ve önizlemede dönen `impactHash` taşımazsa 409 `ParentMeeting.PreviewRequired` (UI onay diyaloğunu atlayamaz).

**`RespondToParentMeetingCommandHandler`** (veli) — read; çağıran kişi; şube `Family` kapsamı (davetli değilse 404); filtreli Include (o velinin güncel sürüm yanıtı + davetli satırları); `section.RecordParentResponse`; değişiklik yoksa 200 + mevcut durum, geçmiş yazılmaz. **Bildirim üretmez** (§9.1 "üretilmeyecekler") `[GŞ-Y6]`. Hız sınırı §7.5.

**`RecordOnBehalfResponseCommandHandler`** — manage; hedef `parentPersonId` rotadan; **sunucuda** aktif davetli doğrulaması (Published'da canlı, Started'da dondurulmuş liste) `[GŞ-Y2]`; geçmişe `Cause = OnBehalf`, `ActorPersonId = idareci`, `OnBehalfOfPersonId = veli`, `ChannelNote`.

**`SaveParentMeetingNoteCommandHandler`** — write; kapsam `Admin | Homeroom`; şube `IsFieldDataOpen(now)`; not yoksa oluştur (`Create` denetimi), varsa `baseVersion == CurrentVersion` değilse 409 `ParentMeeting.NoteVersionConflict` (yanıt güncel sürüm numarası + son düzenleyen + zaman döner, **metin dönmez**; istemci yerel taslağı korur); otomatik kayıtta 5 dk birleştirme; denetim `Update` (birleşen kayıtta denetim satırı da birleşir — aynı yazar/5 dk tek satır). Doğrulama mesajları `AttemptedValue`/`{PropertyValue}` içermez `[GŞ-Y4]`. Bu komut ve not okuma sorgusu `ISensitiveRequest` işaretçisi taşır: `RequestLoggingBehavior` ve süper yönetici tam-yük loglaması bu istekler için gövde/mesaj yazmaz (işaretçi yeni; B4'te `Application/Common` altına eklenir) `[GŞ-Y4]`.

**`GetParentMeetingNoteQueryHandler`** — write; kapsam `Admin | Homeroom | NoteAuthor`; **her getirmede** `Read` denetim satırı (not yoksa da: "boş paneli açtı") `[GŞ-Y3]`. Bu bir GET yan etkisidir ve bilinçlidir (KVKK m.12 okuma izi); ödevdeki "GET yan etkisiz" kuralının belgeli istisnasıdır. İmha edilmişse `{ body: null, purgedAt }`.

**`CompleteParentMeetingSectionCommandHandler`** — write; `Admin | Homeroom`; `Started`; yanıt `unmarkedCount` (onay diyaloğu bunu önceden `GET section` sayaçlarından okur).

### 5.4 Davetli değişimi (BR-PM-41) — olay + uzlaştırma

| Tetik | Yol |
|---|---|
| `ParentStudentLinkedEvent`, `ParentStudentRevokedEvent`, `StudentAssignedToClassRoomEvent`, `StudentTransferredEvent`, `PersonArchivedEvent` | ilgili öğrencinin şubesindeki `Published` şube toplantılarını uzlaştırır |
| `CanViewInfo` değişimi, `ValidUntil` geçmesi (olay yok) | `ParentMeetingInviteeReconcileJob` (saatlik) + yayın, her hatırlatma süpürmesi ve başlatma anı uzlaştırması |

Eklenen davetliye N-01 (aynı olay kimliği, yalnız yeni alıcı); düşen satır `DroppedAt` ile kapanır, yanıt geçmişi kalır, sonraki bildirimler gitmez. `Started` sonrası no-op (I-11).

### 5.5 Sezon arşivi (BR-PM-42)

`AcademicSessionArchivedEvent` handler'ı (B5): `Started` → `Complete(SeasonArchive)`; `Published` → `Cancel(system)` + N-04; `Draft` → soft delete (organizasyon + şubeler). Hepsi `ActorPersonId = null` denetimiyle `[GŞ-D2]`. Saklama sayacı `AcademicSession.ArchivedAt`'tir.

---

## 6. Katalog, izin seed'i, modül anahtarı

### 6.1 `PermissionSeedData` / `MasterSeedIds`

```csharp
// MasterSeedIds.Permissions
public static Guid ParentMeetingsRead   { get; } = SeedGuid.From("perm:parentMeetings.read");
public static Guid ParentMeetingsWrite  { get; } = SeedGuid.From("perm:parentMeetings.write");
public static Guid ParentMeetingsManage { get; } = SeedGuid.From("perm:parentMeetings.manage");

// PermissionSeedData
Row(ParentMeetingsRead,   "PARENT_MEETINGS", "READ",   "parentMeetings.read",   "Veli toplantısını görüntüle"),
Row(ParentMeetingsWrite,  "PARENT_MEETINGS", "WRITE",  "parentMeetings.write",  "Fiili katılımı işaretle, görüşme notu yaz, toplantıyı tamamla"),
Row(ParentMeetingsManage, "PARENT_MEETINGS", "MANAGE", "parentMeetings.manage", "Veli toplantısı planla, yayınla, ertele, iptal et, veli adına yanıt, rapor ve ayarlar"),
```

`RolePermissionSeedData`: SchoolAdmin ← read/write/manage (ayna VP'ye kopyalar); Teacher ← read/write; Parent ← read. Bekçi: `ParentMeetingPermissionSeedTests` (`HomeworkPermissionSeedTests` emsali) — Student'ta hiçbiri yok, SuperAdmin'de hiçbiri yok, VP = SchoolAdmin.

### 6.2 Okul ayarları

`SchoolSettingsController` + `GET/PUT /school-settings/parent-meeting-settings` (ödev ayarları emsali): `{ morningReminderTime: "08:00", startingNotificationEnabled, responsePendingEnabled, noteRetentionYears, autoCompleteAfterDays: 3 (salt okunur), editWindowDays: 7 (salt okunur), quietHoursWarning: boolean }` — `quietHoursWarning` sunucu hesaplar (sabah saati okulun sessiz saatine düşüyorsa).

### 6.3 Modül kataloğu

- Backend `SeedDefaultModuleConfigsHandler.DefaultCatalog` → `new("parentMeetings", IsEnabled: true, PlanRestricted: false, Tier: ModuleTier.Standard)`.
- Mevcut okullar: B1 göçünde idempotent backfill (`INSERT … WHERE NOT EXISTS`, kulüp `20260829_clubs_faz0` emsali).
- UI `MODULE_CATALOG.parentMeetings = { name: "Veli Toplantıları", description: "Veli toplantısı planlama, katılım yanıtı ve görüşme notları.", tag: "free" }`.
- `master.plan_modules`'te karşılığı `[D]` (UI yorumu bu tablodan söz ediyor; seed'i bu depoda bulunamadı).
- **Bulgu:** backend'de modül kapısı yok; modül kapatılsa da uçlar çalışır (tüm modüller için geçerli). Bu modül için ek kapı önerilmez (tutarsızlık yaratır); genel bir `RequireModule` işi ayrı TB adayıdır.

---

## 7. API sözleşmesi

Zarf her uçta `{ data, meta, errors, correlationId }`; enum'lar string; tarih `YYYY-MM-DD`, saat `HH:mm` (okul-yerel), an `ISO-8601 offset`'li; eşzamanlılık `concurrencyToken` (base64 rowversion). Web ve mobil **aynı uçları** kullanır; ekran farkı istemcidedir. Rota sırası: sabit yollar (`mine`, `family`, `sections`, `note-audit`, `form-context`, `:preview`) `{id:guid}`'den önce. Yaşam döngüsü fiilleri `{id}:verb`.

### 7.1 İdare uçları (`ParentMeetingsAdminController`)

| # | Fiil + rota | İzin | İstek | Yanıt |
|---|---|---|---|---|
| A1 | `GET /parent-meetings/form-context` | manage | — | `{ today, now, classRooms: [{ classRoomId, name, gradeLevelId, gradeLevelName, stage: "primary"\|"middle"\|"high", homeroomTeacher: { personId, name } \| null, studentCount }] }` |
| A2 | `GET /parent-meetings?status=&from=&to=&tab=published\|draft` | manage | — | `ParentMeetingListItemDto[]`: `{ id, title, status, dateFrom, dateTo, sectionCount, sectionStatusSummary: { published, started, completed, cancelled, draft }, responseRate, concurrencyToken }` |
| A3 | `POST /parent-meetings:preview` | manage | `PlanInputDto` (A4 gövdesi) | `PlanPreviewDto`: `{ sectionCount, studentCount, inviteeCount, reach: { inApp, push, emailEligible \| null, withoutAccount, withoutEmail }, conflicts: [{ parentPersonId, parentName, items: [{ classRoomName, startsAt }] }], sectionsWithoutHomeroom: [classRoomId], pastStarts: [classRoomId] }` |
| A4 | `POST /parent-meetings` | manage | `{ title, description?, defaults: { date, startTime, endTime?, location }, sections: [{ classRoomId, date, startTime, endTime?, location }] }` | 201 `ParentMeetingDetailDto` |
| A5 | `GET /parent-meetings/{id}` | manage | — | `ParentMeetingDetailDto`: `{ id, title, description, status, defaults, concurrencyToken, sections: [SectionRowDto] }`; `SectionRowDto = { id, classRoomId, classRoomName, homeroomTeacher, date, startTime, endTime, startsAt, location, status, scheduleRevision, counts: { invited, attending, notAttending, noResponse, present \| null, absent \| null, withoutAccount }, cancelled: { at } \| null, concurrencyToken }` |
| A6 | `PUT /parent-meetings/{id}` | manage | Draft: A4 gövdesi + `concurrencyToken` (şubeler yer değiştirir); Published: yalnız `{ title, description, concurrencyToken }` | `ParentMeetingDetailDto` |
| A7 | `DELETE /parent-meetings/{id}` | manage | — | 204 (yalnız Draft) |
| A8 | `POST /parent-meetings/{id}:publish` | manage | `{ concurrencyToken }` | `ParentMeetingDetailDto` |
| A9 | `POST /parent-meetings/{id}/sections` | manage | `{ sections: [...], concurrencyToken }` | `ParentMeetingDetailDto` (Published'da şubeler doğrudan yayınlanır, BR-PM-06) |
| A10 | `POST /parent-meetings/sections:plan-preview` | manage | `{ sectionIds[], date?, startTime?, endTime?, location? }` | `{ impactHash, sections: [{ sectionId, scheduleChanges, locationChanges, responsesToReset, notifyCount, newConflicts }], totals }` |
| A11 | `POST /parent-meetings/sections:update-plan` | manage | A10 gövdesi + `{ items: [{ sectionId, expectedScheduleRevision, concurrencyToken }], impactHash }` | `SectionRowDto[]` — tekil kullanım tek öğeli dizi |
| A12 | `POST /parent-meetings/sections:cancel` | manage | `{ items: [{ sectionId, concurrencyToken }], reason? }` | `SectionRowDto[]` |
| A13 | `PUT /parent-meetings/sections/{sectionId}/responses/{parentPersonId}` | manage | `{ status: "attending"\|"notAttending", declineReason?, channelNote? }` | `InviteeParentDto` |
| A14 | `GET /parent-meetings/{id}/report` | manage | — | `ParentMeetingReportDto` (§7.4) |
| A15 | `GET /parent-meetings/{id}/report/export` | manage | — | `.xlsx` (not metni yok; aktarım kim/kapsam/satır sayısıyla loglanır) |
| A16 | `GET /parent-meetings/note-audit?studentPersonId=&sectionId=&actorPersonId=&from=&to=&page=` | manage | — | `[{ at, actor: { personId, name, role }, student: { personId, name }, classRoomName, meetingTitle, action, version }]` |
| A17 | `GET/PUT /school-settings/parent-meeting-settings` | read / manage | §6.2 | §6.2 |

Toplu uçlar (A11, A12) tek işlemdir: bir öğe başarısızsa hiçbiri uygulanmaz ve hata `errors[].field = items[i]` ile döner.

### 7.2 Ortak şube uçları (idare + rehber)

| # | Fiil + rota | İzin | Kapsam | Yanıt |
|---|---|---|---|---|
| S1 | `GET /parent-meetings/sections/{sectionId}` | read | Admin \| Homeroom | `SectionDetailDto`: `SectionRowDto` + `{ meeting: { id, title, description }, viewerRole: "admin"\|"homeroom", capabilities: { canMarkPresence, canWriteNotes, canComplete, canRespondOnBehalf, canReschedule, canCancel }, editableUntil, students: [{ studentPersonId, name, studentNo, hasNote, parents: [InviteeParentDto] }] }`; `InviteeParentDto = { parentPersonId, name, relation, hasAccount, phoneE164 \| null, response: "attending"\|"notAttending"\|"noResponse", declineReason \| null, respondedAt, isOnBehalf, presence: "present"\|"absent"\|"unmarked", dropped: { at, reason } \| null }`. Sekmeler (Tümü/Yanıt yok/Hesapsız/Yürüt) istemci süzgecidir — tek uç |
| S2 | `GET /parent-meetings/sections/{sectionId}/responses/{parentPersonId}/history` | read | Admin \| Homeroom | `[{ occurredAt, revision, from, to, declineReason, actor: { personId, name } \| null, isOnBehalf, cause, channelNote }]` |
| S3 | `PUT /parent-meetings/sections/{sectionId}/presence/{parentPersonId}` | write | Admin \| Homeroom | `{ isPresent: bool }` → `InviteeParentDto` |
| S4 | `POST /parent-meetings/sections/{sectionId}:complete` | write | Admin \| Homeroom | `{ concurrencyToken }` → `SectionDetailDto` |
| S5 | `GET /parent-meetings/sections/{sectionId}/notes/{studentPersonId}` | write | Admin \| Homeroom \| NoteAuthor | `{ studentPersonId, body \| null, version, lastEditedBy: { personId, name }, lastEditedAt, purgedAt \| null, canEdit }` — **okuma denetimi yazar** |
| S6 | `PUT /parent-meetings/sections/{sectionId}/notes/{studentPersonId}` | write | Admin \| Homeroom | `{ body, baseVersion, isAutosave }` → `{ version, lastEditedAt }` (metni geri döndürmez) |
| S7 | `GET /parent-meetings/sections/{sectionId}/report` | read | Admin \| Homeroom | şube kırılımlı `ParentMeetingReportDto` |

Not metni S1 dahil **hiçbir liste DTO'sunda yoktur**; yalnız `hasNote` `[GŞ-Y3]`. `declineReason` yalnız Admin/Homeroom görünümünde serileştirilir.

### 7.3 Rehber ve veli uçları

| # | Fiil + rota | İzin | Kapsam | Yanıt |
|---|---|---|---|---|
| T1 | `GET /parent-meetings/mine?scope=upcoming\|past\|today\|pendingCompletion` | read | Homeroom (çağıranın güncel rehber şubeleri) | `[{ sectionId, meetingTitle, classRoomName, date, startTime, startsAt, location, status, counts, editableUntil }]` — `today` anasayfa kartı (PM-T-04), `pendingCompletion` Bekleyen İşler |
| P1 | `GET /parent-meetings/family?scope=upcoming\|past\|today` | read | Family | `FamilyMeetingCardDto[]`: `{ sectionId, child: { personId, firstName }, classRoomName, meetingTitle, date, startTime, endTime, startsAt, location, status, myResponse: "attending"\|"notAttending"\|"noResponse", myDeclineReason, canRespond, banners: { cancelled, rescheduledNeedsResponse, conflict } }` — çoklu çocuk = çocuk başına kart (BR-PM-22) |
| P2 | `GET /parent-meetings/family/sections/{sectionId}` | read | Family | P1 kartı + `{ description, children: [{ personId, firstName }] }` (ikizde iki çocuk, tek yanıt) |
| P3 | `PUT /parent-meetings/family/sections/{sectionId}/response` | read | Family | `{ status, declineReason? }` → P2 |

Veli DTO'larında **öteki velinin** varlığı, yanıtı ve nedeni yoktur (alan "boş" değil "yok") `[GŞ-Y2]`; iptal gerekçesi yoktur; rehber öğretmen adı vardır (toplantıyı kim yürütecek). Öğrenci rolüne uç yok (S-2).

**Uç sayısı:** idare 17 (A17 iki fiil) + ortak 7 + rehber 1 + veli 3 = **28 rota / 29 uç**.

### 7.4 Rapor DTO'su (PM-A-06)

```ts
ParentMeetingReportDto = {
  meetingId, title,
  sections: [{ sectionId, classRoomName, status, invited, attending, notAttending, noResponse,
               present, absent, responseRate, presenceRate }],   // iptal edilenler status ile ayrı gruplanır
  totals: { …aynı alanlar },
  crossTab: { attendingPresent, attendingAbsent, notAttendingPresent, notAttendingAbsent,
              noResponsePresent, noResponseAbsent },
  reach: { withoutAccount, withoutEmail, onBehalfResponses },
  parents: [{ parentName, childName, classRoomName, response, presence, isOnBehalf, hasNote }]  // süzgeç istemcide
}
```

Öğretmen kıyası/sıralaması yok (ödev K-8 felsefesi). Oranlar sunucuda hesaplanır, sayaçlar `ParentMeetingCounters` tek sınıfından (liste/detay/rapor aynı).

### 7.5 Hata kodları ve hız sınırı

| Kod | HTTP | Kaynak |
|---|---|---|
| `ParentMeeting.Validation` | 400 | FluentValidation (mesajlar değer taşımaz) |
| `ParentMeeting.StartInPast` | 400 | BR-PM-04 |
| `ParentMeeting.EndBeforeStart` | 400 | BR-PM-03 |
| `ParentMeeting.EmptySections` / `ParentMeeting.DuplicateSection` | 400 / 409 | BR-PM-02 |
| `ParentMeeting.InvalidState` | 409 | durum makinesi |
| `ParentMeeting.ResponseLocked` | 409 | BR-PM-19 |
| `ParentMeeting.FieldDataClosed` | 409 | BR-PM-10/30 (pencere doldu) |
| `ParentMeeting.HasFieldData` | 409 | I-9 |
| `ParentMeeting.PreviewRequired` | 409 | BR-PM-17 (`impactHash` yok/eskimiş) |
| `ParentMeeting.Concurrency` | 409 | rowversion / beklenen sürüm (iki idareci, §11) |
| `ParentMeeting.NoteVersionConflict` | 409 | not `baseVersion` |
| `ParentMeeting.SeasonArchived` | 409 | BR-PM-08 (+ `active-season-write` 403) |
| `ParentMeeting.NotFound` | 404 | yok **veya kapsam dışı**, davetli değil |
| `RateLimited` | 429 | §7.5 |

**Hız sınırı `[GŞ-Y6]`** (B1'de `RateLimitingSetup.cs`'e eklenir): `parent-meeting-response` — kullanıcı (hesap) × şube toplantısı başına dakikada 10 değişiklik (P3, A13); `parent-meeting-bulk` — kullanıcı başına dakikada 5 toplu istek (A11, A12, A8). Mevcut bölümlenmemiş `authenticated` politikası kullanılmaz. Yanıt değişikliği bildirim üretmediği ve bildirim plan sürümü başına tek kimlik taşıdığı için spam yüzeyi ayrıca kapalıdır.

---

## 8. Bildirimler ve zamanlanmış işler

### 8.1 Yeni `NotificationKind` değerleri (sona eklenir)

| Değer | Ad | Olay anahtarı | Grup | Severity | Actionable | Push | Push kitlesi |
|---|---|---|---|---|---|---|---|
| 60 | `ParentMeetingPublished` (N-01) | `PARENT_MEETING_PUBLISHED` | `ParentMeeting` | Info | false | ✓ | Parent |
| 61 | `ParentMeetingAssigned` (N-02) | `PARENT_MEETING_ASSIGNED` | `ParentMeeting` | Info | false | ✓ | Teacher |
| 62 | `ParentMeetingChanged` (N-03) | `PARENT_MEETING_CHANGED` | `ParentMeeting` | Warning | false | ✓ | Parent |
| 63 | `ParentMeetingCancelled` (N-04) | `PARENT_MEETING_CANCELLED` | `ParentMeeting` | Critical | false | ✓ | Parent |
| 64 | `ParentMeetingDayReminder` (N-05) | `PARENT_MEETING_DAY_REMINDER` | `ParentMeeting` | Info | false | ✓ | Parent \| Teacher |
| 65 | `ParentMeetingStarting` (N-06) | `PARENT_MEETING_STARTING` | `ParentMeeting` | Info | false | ✓ (sessiz saatte **atlanır**) | Parent |
| 66 | `ParentMeetingResponsePending` (N-07) | `PARENT_MEETING_RESPONSE_PENDING` | `ParentMeeting` | Warning | false | ✓ | Parent |

`NotificationEventGroup.ParentMeeting = 7` (sona; ödev/kulüp gerekçesi: veli "duyuru"yu susturunca toplantıyı susturmamalı). "Tamamlanmayı bekleyen toplantı" bildirim **değildir**; T1 `pendingCompletion` sorgusudur (okununca kaybolmaz, BR-PM-12 notu).

Değişecek dosyalar (hepsi B5): `NotificationKind.cs`, `NotificationKindMetadata.cs`, `NotificationEventGroup.cs`, `PushEventKeyMap.cs` (Map + Audiences), `NotificationEventKeyMap.cs`, `PushDeepLinks.cs` (`ParentMeetingFamily(sectionId)`, `ParentMeetingSection(sectionId)`), `NotificationSourceTypes.ParentMeetingSection`, `NotificationEventTypeSeedData` + `MasterSeedIds.NotificationEventTypes` (7 satır).

### 8.2 Seed satırları

Hepsi `supportsSms: false` (SMS kanalı yok — TB-44 sahte toggle), `portal: true`, `push: true` (K-34: kapsamdaki olayın push varsayılanı açık), `delivered: true` (üreticileriyle aynı commit). E-posta sütunları B6'dan sonra anlamlıdır (§9): `supportsEmail` = N-01/N-03/N-04 `true`, diğerleri `false` (BR-PM-36); `defaultEmailEnabled` = N-01/N-03/N-04 `true` (S-6), diğerleri `false`.

### 8.3 Olay kimliği, tekilleştirme ve tazelik

`ParentMeetingEventIds` (tek yer; günler `yyyy-MM-dd` Invariant):

| Tür | `DeterministicGuid.Combine(schoolId, sectionId, …)` |
|---|---|
| N-01 | `ScheduleRevision, "PM_PUBLISHED"` — sonradan eklenen velide **aynı** kimlik; dispatcher (EventId, alıcı, kanal) bazında tekler, yeni alıcıya gider |
| N-02 | `ScheduleRevision, LocationRevision, phase ("published"\|"changed"\|"cancelled"), homeroomPersonId` |
| N-03 | `ScheduleRevision, LocationRevision, "PM_CHANGED"` |
| N-04 | `"PM_CANCELLED"` |
| N-05 | `ScheduleRevision, meetingDate, "PM_DAY"` |
| N-06 | `ScheduleRevision, "PM_STARTING"` |
| N-07 | `ScheduleRevision, "PM_PENDING"` |

**Tazelik kapısı (BR-PM-35)** — Notifications modülünde yeni port:

```csharp
// Application/Modules/Notifications/Abstractions
public interface INotificationSourceFreshness
{
    string SourceType { get; }
    /// <summary>false → bildirim eskimiştir; gönderilmez (in-app satırı da yazılmaz).</summary>
    Task<bool> IsCurrentAsync(Guid sourceId, NotificationKind kind, Guid eventId, CancellationToken ct);
}
```

`DispatchNotificationJob.RunAsync(…, sourceType, sourceId, …)` dispatcher'dan önce, `DeferredPushJob` göndermeden önce (yeni aşırı yükleme; eski imza korunur — kuyrukta bekleyen işler için) kayıtlı tazelik kontrolünü çağırır. `ParentMeetingSourceFreshness`: şubenin güncel `Status`, `ScheduleRevision`, `LocationRevision`'ından türüne göre **beklenen** olay kimliğini yeniden hesaplar ve gelenle karşılaştırır; N-04 yalnız `Cancelled` iken, N-06 yalnız `Started` iken, diğerleri `Cancelled` değilken günceldir. Sürüm olay kimliğinin içinde olduğu için `Notification` tablosuna kolon eklenmez. Sonuç: "tarih değiştir + hemen iptal" senaryosunda kuyruktaki N-03 atılır, yalnız N-04 gider; sessiz saatte ertelenmiş eski N-03 sabah gönderilmez.

**Kanal tekilleştirme (BR-PM-36):** push ve uygulama içi aynı `EventId` (zilde tek satır, push `notificationId` taşır — mevcut). E-posta yalnız katalogda `supportsEmail` olan üç türde.

**Yanıt çağrısı tekrarlanmaz (BR-PM-37):** N-07 alıcı kümesi iş anında "güncel sürümde yanıt satırı olmayan aktif davetli" olarak hesaplanır.

**Gövde ve alıcı kümeleri:** gövde çocuk adını taşır ("Ayşe için…"; yalnız ad, soyad yok `[GŞ-O2]`) → aynı şube için alıcılar çocuk adı kümesine göre gruplanır ve her grup aynı `EventId` ile ayrı `Enqueue` edilir (kulüp yerleştirme emsali). Gövdeye **neden, iptal gerekçesi, not, öteki veli bilgisi asla** girmez. Bildirimler `NotificationSource(ParentMeetingSection, sectionId)` taşır.

### 8.4 Sessiz saat

| Tür | Davranış |
|---|---|
| N-01, N-02, N-03, N-04, N-07 | push ertelenir (mevcut) |
| N-05 | ertelenir; sabah saati sessiz saate düşüyorsa ayar ekranı uyarır (`quietHoursWarning`) |
| N-06 | push **atlanır** (BR-PM-39): `PushQuietHours.DropsDuringQuietHours(kind)` yeni yüklem; kanal erteleme planlamadan `none` döner; uygulama içi satır yazılır |

E-posta ve uygulama içi ertelenmez (BR-PM-34).

### 8.5 Hangfire işleri

| İş | Cron (Istanbul) | Ne yapar | İdempotentlik |
|---|---|---|---|
| `ParentMeetingStartJob` | `2-59/5 * * * *` (5 dk, yoklama `*/5` ile aynı dakikaya düşmez) | `Published && starts_at <= now` → uzlaştır, `Start`, ayar açıksa ve `now - starts_at < 30 dk` ise N-06 (geç kalmış "başlıyor" haberi gönderilmez) | durum kapısı + rowversion; N-06 için `StartingNotifiedRevision` |
| `ParentMeetingReminderSweepJob` | `7,22,37,52 * * * *` (15 dk) | okul başına yerel saat: (a) N-05 — `meeting_date == bugün`, yerel saat ≥ sabah saati, `DayReminderSentRevision != ScheduleRevision`; sürümün yayın/değişiklik anı o günün sabah saatinden sonraysa **gönderilmeden damgalanır** (BR-PM-38); alıcı = güncel sürümde Attending + güncel rehber. (b) N-07 — ayar açık, `meeting_date - 2 == bugün`, yerel saat ≥ sabah saati, `ResponsePendingSentRevision != ScheduleRevision`; sürüm bu eşikten sonra doğduysa gönderilmeden damgalanır; alıcı = güncel sürümde yanıtsız aktif davetli | damga + deterministik kimlik (iki katman, ödev emsali) |
| `ParentMeetingAutoCompleteJob` | `10 1 * * *` | `Started && meeting_date + 3 <= bugün` → `Complete(Auto)`, sistem denetimi; bildirim yok | durum kapısı |
| `ParentMeetingInviteeReconcileJob` | `25 * * * *` | `Published` ve gelecekteki şubeleri uzlaştırır; eklenene N-01 | unique + `Reconcile` diff |
| `ParentMeetingNoteRetentionJob` | `50 4 * * *` (04:40 duyuru süresi ile çakışmaz) | sezonu arşivli ve `ArchivedAt + retentionYears <= now` notlarda `Body = NULL` + tüm sürümlerde `Body = NULL`, `PurgedAt`, `Purge` denetimi (sistem), `data.delete.kvkk` yapısal olayı (yalnız sayılar + okul) `[GŞ-Y5]` | `purged_at IS NULL` süzgeci; yedekten geri dönüşte yeniden koşunca aynı notları yeniden imha eder |

Hepsi: aday okul taraması `IgnoreQueryFilters` + her sorguda açık `SchoolId`, okul döngüsünde `SetForLoginFlow`, okul başına hata yalıtımı + `ChangeTracker.Clear()`, kayıttan sonra `IPostCommitDispatcher.RunPendingAsync(CancellationToken.None)`, `[AutomaticRetry]` politikası (K-02), yapısal log (sayaçlar; kişi verisi yok).

---

## 9. E-44 — alıcı e-posta tercihi

Ayrıntılı gerekçe: [[0027-eposta-varsayilani-tur-bazli]].

### 9.1 Model

| Değişiklik | Ayrıntı |
|---|---|
| `NotificationEventType` | **yeni** `SupportsEmail bit` (alıcı bu tür için e-postayı seçebilir mi) — `SupportsSms` ile simetrik. `DefaultEmailEnabled` artık gerçekten "satır yokken varsayılan"dır |
| Göç (B6) | (1) tüm mevcut satırlarda `default_email_enabled = 0` `[GŞ-K2]`; (2) `supports_email = 1` yalnız `is_delivered = 1` ve aile/öğrenci kitleli satırlarda — öneri listesi `[F-3]`; (3) N-01/N-03/N-04 `supports_email = 1, default_email_enabled = 1`; N-02/N-05/N-06/N-07 `0/0` |
| `NotificationPreference` | **yeni** `EmailEnabled bit NULL` — `null` = katalog varsayılanı; mevcut push satırları dokunulmadan `null` kalır. `Create(..., bool? pushEnabled, bool? emailEnabled)`, `Update(bool? push, bool? email)` (kısmi) |
| Tercih uçları | `GET /notifications/preferences` öğesi: `{ eventKey, name, group, pushEnabled, pushSupported, emailEnabled (etkin değer), emailSupported, emailIsDefault }` + üst düzey `hasEmailAddress`; `PUT` öğesi `{ eventKey, pushEnabled?, emailEnabled? }`. Liste kaynağı: push anahtarları ∪ e-posta destekli anahtarlar (kitle haritası e-posta-yalnız anahtarları da kapsamalı) |
| Doğrulayıcı | e-posta satırı yalnız `SupportsEmail` anahtarına yazılır (TB-44 sahte toggle yasağı) |

### 9.2 `EmailNotificationChannel` açılışı

Kapılar (ucuzdan pahalıya):

1. **Yapılandırma kapısı** `[GŞ-K1]`: `NotificationEmailOptions.Enabled` + `Smtp:Host` ve `FromAddress` üretimde izin listesinde (`AllowedHosts`, `AllowedFromDomains`); `ValidateOnStart` — liste dışıysa uygulama **başlamaz** (önerilen) ya da kanal kapalı kalır (`[F-1]`). Bugünkü `smtp-mail.outlook.com` üretim için reddedilir.
2. Kapsam: `NotificationEventKeyMap.TryGetEventKey` (**push haritası değil** — e-posta push kapsamından bağımsız).
3. Katalog: `SupportsEmail`.
4. Tercih: `pref.EmailEnabled ?? catalog.DefaultEmailEnabled`.
5. Adres: Hesap → Kişi → `PrimaryEmail`; yoksa atla (diğer kanallar etkilenmez).
6. Gönderim; hata → `false` (Ruling-28 aynen).

`RecipientCanChooseEmail` sabiti kaldırılır; sınıf doc'undaki "tercihsiz e-posta yasak" notu ADR 0027'ye göre yeniden yazılır `[GŞ-O1]`. Kanal tekil alıcı yolu yerine toplu tercih okuması (`IN` sorgusu) önerilir — N-01 okul geneli fan-out'tur.

### 9.3 Şablon

`NotificationEmail.Build` genişler: konu = bildirim başlığı; gövde = okul adı, şube, tarih, saat, yer, (onaylanırsa) çocuğun adı `[F-6]`; sabit cümle "Yanıtınızı OKSİS uygulamasından verebilirsiniz."; **düğme/katılım bağlantısı yok** (K-13); altbilgi: "Bu e-postaları Ayarlar › Bildirimler'den kapatabilirsiniz." + `List-Unsubscribe` başlığı `[GŞ-O1]`. Tek dokunuşla vazgeçme bağlantısı imzalı belirteç ucu gerektirir ve K-13'ün "bağlantı yok" hükmüyle gerilir → `[F-5]`. Açıklama, iptal gerekçesi, neden, not **girmez** `[GŞ-O2]`.

### 9.4 Etkilenen mevcut olaylar

Göç (1) sayesinde hiçbir mevcut olay E-44 açılışıyla kendiliğinden e-posta göndermeye başlamaz; `supports_email = 1` yapılanlar yalnız alıcı açarsa gider. Etkilenen ekran: tüm rollerin bildirim tercihleri (PM-X-01). Test: tercihsiz veliye `GRADE_PUBLISHED` → e-posta kanalı `false` `[GŞ-K2]`.

### 9.5 Riskler

`PrimaryEmail` doğrulanmamış bir alandır (yazım hatası → başkasına toplantı bilgisi) `[GŞ-O2]`; V1'de kabul, doğrulama akışı ayrı iş. Varsayılan-açık e-postanın KVKK dayanağı (aydınlatma metninde e-posta kanalı) hukuk teyidi ister `[F-4]`.

---

## 10. SMS altyapısı (gönderimsiz)

| Parça | Tasarım |
|---|---|
| `ISmsSender` | mevcut port korunur; **`NoOpSmsSender`** (Infrastructure) eklenir — numara ve gövde loglanmaz; yalnız çağıran bağlamı varsa tür/eventId |
| Niyet kaydı | Notifications modülünde `ISmsIntentRecorder.RecordAsync(eventId, kind, IReadOnlyList<Guid> personIds, ct)` → `notifications.sms_intents(id, school_id, event_id, kind, recipient_person_id, status, created_at)`; unique `(event_id, recipient_person_id)`; `status = NotSentChannelDisabled`. **Numara saklanmaz** (gönderim anında kişiden çözülecek) `[GŞ-O3]` |
| Alıcı çözümü | kişi tabanlı: aktif davetli veli kişilerinden `LinkedAccountId == null` olanlar (hesaplı veli hesap kanallarından gider) |
| Kanal yeri | `INotificationChannel` **değil** (o arayüz hesap tabanlı). Bildirimden ayrı, aynı `EventId` ile yan yol; ileride `SmsDispatchJob` niyetleri `ISmsSender` + `DailySmsLimit` + İYS kararıyla gönderir — modül değişmez |
| Kapsam | yalnız N-01/N-03/N-04 (`SmsIntentScope` sabit kümesi); tazelik kapısı gönderim fazında da uygulanır |
| Görünürlük | rapor `reach.withoutAccount` ve önizleme bu sayıyı niyet tablosundan değil davetli × hesap durumundan hesaplar (niyet yalnız denetim izi) |

Test: log sink'te E.164 deseni (`\+90\d{10}`) geçmez `[GŞ-O3]`.

---

## 11. Kesişen konular

### 11.1 Eşzamanlılık
Organizasyon ve şube `RowVersion` + istemci `concurrencyToken`; tarih değişikliğinde ayrıca `expectedScheduleRevision`. İkinci kaydeden 409 `ParentMeeting.Concurrency` ("kayıt değişti, yenileyin") `[GŞ-D2]`. Fiili katılımda son yazan kazanır (satır başına), iz denetimdedir; not `baseVersion` ile korunur.

### 11.2 Çevrimdışı
Mobil "Geldi" işaretleri kuyruklanır ve sırayla gönderilir (idempotent PUT). Not taslağı **şifresiz kalıcı depoya yazılmaz**: mobilde `expo-secure-store` (ya da yalnız oturum belleği), web'de bellek; çıkışta silinir `[GŞ-O4]`.

### 11.3 KVKK ve loglama
Not metni: hiçbir domain olayı, bildirim, e-posta, push, rapor, dışa aktarım, log mesajı, `IAuditLogger` kaydı taşımaz `[GŞ-K3, Y4]`. Yanıt nedeni: yalnız Admin/Homeroom DTO'su ve yanıt geçmişi (geçmiş de yalnız onlara) `[GŞ-D1]`.

### 11.4 `ValidUntil` bulgusu — öneri
`IGuardianRelationFilter` (Users modülü `Application/Common` altında, saf ifade + `today` parametresi) **bu modülde** kullanılır: davetli = `RevokedAt == null && CanViewInfo && ValidFrom <= today && (ValidUntil == null || ValidUntil >= today)`. `NotificationRecipientResolver.GuardianRelations` ve diğer veli kapsamlarının (ödev aile kapsamı, `ChildScopeAuthorizationHandler`, yoklama) aynı süzgece taşınması **ayrı TB işi** olarak önerilir — tüm modüllerin alıcı kümesini değiştirir ve kendi regresyon turunu ister `[F-7]`. Bu modülde bitiş tarihi geçen ilişki saatlik uzlaştırmada `RelationshipExpired` ile düşer.

---

## 12. UI sözleşmesi (oksis-ui)

| Yer | İş |
|---|---|
| `packages/api/src/parent-meetings/contract.ts` | Mock-first: §7 DTO'ları + string union enum'lar; backend gelene kadar `declare module "../generated/schema"` **augmentation** bloğu path tiplerini enjekte eder (kulüp/ödev emsali); `npm run codegen` gerçek şemayı getirince blok silinir (drift bekçisi) |
| `packages/api/src/parent-meetings/endpoints.ts`, `queries.ts` | 29 uç sarmalayıcısı; TanStack anahtarları `['parentMeetings', 'admin', filters]`, `['parentMeetings','meeting',id]`, `['parentMeetings','section',sectionId]`, `['parentMeetings','history',sectionId,parentId]`, `['parentMeetings','note',sectionId,studentId]` (**`gcTime: 0`, önbelleğe yazılmaz, persist edilmez**), `['parentMeetings','mine',scope]`, `['parentMeetings','family',scope]`, `['parentMeetings','report',id]`, `['parentMeetings','noteAudit',filters]`, `['schoolSettings','parentMeeting']` |
| Geçersizleme | plan/iptal/yayın → `meeting`, `admin`, `section`, `mine`, `family`; yanıt → `family` (veli) / `section` (idare); presence → `section`; complete → `section`, `mine` |
| `packages/core/src/parent-meetings/` | `types.ts` (domain union'ları), `constants.ts` (durum etiketleri, rozet tonları, "Veliye gösterilmez — yalnız sınıf rehber öğretmeni ve idare görür" sabiti — BR-PM-32), `logic.ts` (`canRespond`, `presenceLabel` — sunucu alanlarından okuma, istemci türetmesi yok), `schemas.ts` (Zod: not ≤ 4000, neden ≤ 500, kanal notu ≤ 200, saat `HH:mm`, bitiş > başlangıç) |
| `packages/core/src/modules/constants.ts` | `MODULE_CATALOG.parentMeetings` |
| `packages/api-mocks/src/parent-meetings/` | `parent-meetings-data.ts` + `parent-meetings-handlers.ts` + testleri: tek okul fixture'ı (3 kademe, 6 şube, biri rehbersiz, ikiz kardeş, ayrı yaşayan ebeveyn `CanViewInfo=false`, hesapsız veli, e-postasız veli, çakışan iki toplantı); durum makinesi, plan sürümü sıfırlama, yanıt kilidi, not 409 sürüm çatışması, okuma denetimi satırı, 429 senaryosu mock'ta da uygulanır |
| Bildirim tercihleri (PM-X-01) | `notifications` sözleşmesine `emailEnabled/emailSupported/emailIsDefault/hasEmailAddress` |

Kural: dilim sonunda MSW kapatılınca ekran aynı davranır (ödev H1).

---

## 13. Dilimleme ve dosya sahipliği

### 13.1 Ortak dosya stratejisi

Paralel ajanların çakıştığı yerler: `OksisDbContext`/`IApplicationDbContext`, `Migrations/*` + `OksisDbContextModelSnapshot.cs`, seed dosyaları, `NotificationKind.cs` ve eşlemeleri, `SchoolSettings`, `HangfireSetup.cs`, `DependencyInjection.cs`, `RateLimitingSetup.cs`. Kural:

1. **Ön yükleme:** B1 modülün bütün ortak-dosya dokunuşlarını (DbSet'ler, 10 konfigürasyon, `SchoolSettings` 4 kolon, izin seed'i, modül kataloğu + backfill, hız sınırı politikaları, `ISensitiveRequest` işaretçisi) **tek göçle** yapar. B2-B4 yalnız kendi klasörlerine ve kendi denetleyici dosyalarına dokunur → üçü paralel koşabilir.
2. **Göç tek kulvar:** göç yalnız bir dilimde ve bir anda üretilir; B5, B6, B7 göçleri **sıralı** (B5 → B6 → B7) ve her biri bir öncekinin `dev`'e birleşmesinden sonra `dotnet ef migrations add` koşar (snapshot çakışması önlenir). `MigrationsMatchModelTests` her dilimde yeşil.
3. **Bildirim dosyaları tek sahip:** `NotificationKind`, metadata, key map'ler, seed satırları, deep link'ler, tazelik portu, iş jobs, `HangfireSetup` → yalnız B5. E-posta kanalı, tercih varlığı, tercih uçları → yalnız B6. SMS → yalnız B7. B5 ile B6 aynı `NotificationEventTypeSeedData`'ya dokunur → **B6, B5'ten sonra**.
4. `DependencyInjection.cs`: her dilim kendi `Add…()` uzatma metodunu kendi dosyasında yazar; `DependencyInjection.cs`'e tek satır ekler → birleşmede satır çakışması kolay çözülür; yine de B1 `AddParentMeetings()` iskeletini önceden ekler.

### 13.2 Dilimler

| Dilim | İçerik | Sahip olduğu dosyalar | Bağımlılık | Göç |
|---|---|---|---|---|
| **B1** domain + kalıcılık | 10 varlık + enum + istisna + `ParentMeetingPolicy`; 10 EF konfigürasyonu; DbSet'ler; `SchoolSettings` 4 alan + `UpdateParentMeetingSettings`; izin seed'i + rol eşlemesi; `DefaultCatalog` + backfill; `RateLimitingSetup` iki politika; `ISensitiveRequest` + `RequestLoggingBehavior` dalı; `IGuardianRelationFilter`; domain birim testleri | `Domain/Modules/ParentMeetings/**`, `Infrastructure/Persistence/Configurations/ParentMeetings/**`, `OksisDbContext.cs`, `IApplicationDbContext.cs`, `SchoolSettings.cs` + konfigürasyonu, `MasterSeedIds.cs`, `PermissionSeedData.cs`, `RolePermissionSeedData.cs`, `SeedDefaultModuleConfigsHandler.cs`, `RateLimitingSetup.cs`, `RequestLoggingBehavior.cs`, `Users/…/GuardianRelationFilter.cs`, göç | — | `YYYYMMDD_veli_toplantisi_cekirdek` |
| **B2** planlama/yayın/iptal/erteleme + sorgular | A1-A12, A17, S1 (okuma), `InviteeResolver`, `ParentMeetingScope`, `SectionPlanChangeCalculator`, `ParentMeetingConflictDetector`, `ParentMeetingCounters`, `ParentMeetingNotifier` **arayüzü** (no-op uygulamayla; B5 doldurur) | `Application/Modules/ParentMeetings/{Commands/Create…,Update…,Delete…,Publish…,AddSections…,UpdateSectionPlan…,CancelSections…,UpdateSettings…}`, `{Queries/List…,Get…,GetPlanPreview…,GetSectionPlanChangePreview…,GetSection…,GetFormContext…,GetSettings…}`, `Internal/{Scope,Reader,InviteeResolver,Counters,Conflict,Calculator,Errors,AuditEvents}`, `ParentMeetingsAdminController.cs`, `SchoolSettingsController.cs` (yalnız iki uç) | B1 | yok |
| **B3** yanıt + geçmiş + veli adına | P1-P3, A13, S2; aile kapsamı; hız sınırı bağlama | `Commands/{RespondToParentMeeting,RecordOnBehalfResponse}`, `Queries/{ListFamily…,GetFamilySection…,GetResponseHistory}`, `ParentMeetingsFamilyController.cs` | B1 (B2 ile paralel; `ParentMeetingScope`'u B2 yazar → B3 aynı dosyaya **dokunmaz**, gerekirse `Internal/FamilyScope.cs` kendi dosyası) | yok |
| **B4** katılım + not + tamamlama + denetim + rapor | S3-S7, T1, A14-A16; not okuma denetimi; otomatik kayıt birleştirme; Excel dışa aktarım | `Commands/{MarkPresence,CompleteSection,SaveNote}`, `Queries/{GetNote,ListMy…,GetReport,ExportReport,ListNoteAudit}`, `Internal/{NoteAccess,ReportBuilder}`, `ParentMeetingsTeacherController.cs` | B1 (B2/B3 ile paralel; A14-A16 rotaları `ParentMeetingsTeacherController`'da değil `ParentMeetingsReportController.cs`'te — B2'nin dosyasına girilmez) | yok |
| **B5** bildirim + işler + ayar etkisi | 7 tür + grup + eşlemeler + seed; `ParentMeetingNotifier` uygulaması; `ParentMeetingEventIds`; `INotificationSourceFreshness` + `DispatchNotificationJob`/`DeferredPushJob` kancası; `PushQuietHours.DropsDuringQuietHours`; 5 iş + cron; davetli olay handler'ları; sezon arşivi handler'ı | `Notifications/**` ilgili dosyalar, `PushNotificationChannel.cs` (tek satır), `BackgroundJobs/Jobs/ParentMeeting*.cs`, `HangfireSetup.cs`, `Application/Modules/ParentMeetings/Events/**`, `Internal/{Notifier,NotificationText,EventIds,SourceFreshness}` | B2 (notifier arayüzü) | `YYYYMMDD_veli_toplantisi_bildirim_turleri` (7 katalog satırı) |
| **B6** E-44 | `NotificationEventType.SupportsEmail`; `NotificationPreference.EmailEnabled`; katalog göçü (K2); `EmailNotificationChannel` kapıları; `NotificationEmailOptions` + `ValidateOnStart`; tercih uçları + doğrulayıcı; `NotificationEmail` şablonu | `Domain/Modules/Notifications/Entities/{NotificationEventType,NotificationPreference}.cs`, `Infrastructure/Notifications/{EmailNotificationChannel,NotificationEmail}.cs`, `…/Preferences/**`, `NotificationEventTypeSeedData.cs` (sütun), ilgili konfigürasyonlar, `appsettings*.json` | B5 (seed dosyası) | `YYYYMMDD_e44_eposta_tercihi` |
| **B7** SMS altyapısı | `NoOpSmsSender`, `ISmsIntentRecorder` + `SmsIntent` + konfigürasyon; notifier'dan çağrı (B5'in `ParentMeetingNotifier`'ına tek satır) | `Infrastructure/Notifications/Sms/**`, `Domain/Modules/Notifications/Entities/SmsIntent.cs`, `IApplicationDbContext` (tek DbSet satırı) | B5 | `YYYYMMDD_sms_niyet_kaydi` |

Paralellik: **B1 → (B2 ‖ B3 ‖ B4) → B5 → (B6 → B7)**. B7'nin kodu B6 ile paralel yazılabilir; göçü B6 birleştikten sonra üretilir. B7'nin `IApplicationDbContext` dokunuşu B1'e taşınabilir (DbSet'i önceden açmak) — öneri: taşı, B7 yalnız göç üretsin.

UI dilimleri: **U0** sözleşme + mock (B1 ile paralel, ilk iş) → U1 idare ekranları (B2) → U2 veli (B3) → U3 öğretmen + not (B4) → U4 tercih ekranı e-posta sütunu (B6).

---

## 14. Göç adı önerisi

Depo kalıbı `YYYYMMDD_ad` (EF zaman damgası öne eklenir; ör. `20261013164655_20261013_kulup_uyelik_kaynagi`):

1. `YYYYMMDD_veli_toplantisi_cekirdek` — 10 tablo + `school_settings` 4 kolon + 3 izin + rol eşlemeleri + modül backfill
2. `YYYYMMDD_veli_toplantisi_bildirim_turleri` — 7 katalog satırı (`delivered = 1`)
3. `YYYYMMDD_e44_eposta_tercihi` — `supports_email`, `email_enabled`, `default_email_enabled` sıfırlama + PM satırları
4. `YYYYMMDD_sms_niyet_kaydi` — `notifications.sms_intents`

Geri alma: 1 ve 4 yeni tablo → `Down` temiz; 3 veri dönüştürür → `Down` eski `default_email_enabled` değerlerini sabit listeden geri yazar (seed'de kayıtlı).

---

## 15. Test stratejisi

### 15.1 BR-PM → test eşlemesi

| Kural | Test | Katman | Dilim |
|---|---|---|---|
| BR-PM-01 | öğretmen A4/A8/A11/A12/A13 → 403 | Application (yetki davranışı) | B2/B3 |
| BR-PM-02 | aynı şube iki kez → 400/409; boş şube listesi → 400 | Domain + Application | B1/B2 |
| BR-PM-03 | bitiş ≤ başlangıç → istisna; şube değeri varsayılandan bağımsız | Domain | B1 |
| BR-PM-04 / S-5 | geçmiş an (bugün geçmiş saat dahil) → `StartInPast`; bugünün ileri saati ✓ — okul saat dilimiyle | Domain + Application | B1/B2 |
| BR-PM-05 | Draft bildirim üretmez; yayınlanmış şube silinemez | Application | B2 |
| BR-PM-06 | yayınlanmış organizasyona şube ekleme → doğrudan Published + yalnız o şubeye N-01 | Application | B2/B5 |
| BR-PM-07 | rehber değişince eski rehber 404, yeni rehber erişir; rehbersiz şube yayınlanır, uyarı önizlemede | Application | B2 |
| BR-PM-08 / 42 | arşiv sezonda yazma 409; arşiv olayı: Started→Completed(seasonArchive), Published→Cancelled+N-04, Draft silinir | Application + entegrasyon | B2/B5 |
| BR-PM-09 | `ParentMeetingStartJob` StartsAt geçenleri başlatır, iki kez koşunca tek geçiş | Entegrasyon | B5 |
| BR-PM-10 / 30 | Published'da presence/not 409; Completed+6g ✓, +8g 409 | Domain | B1/B4 |
| BR-PM-11 | complete onay sayacı = işaretsiz veli sayısı | Application | B4 |
| BR-PM-12 | 3. gün gece otomatik kapanış, notlar aynen, denetim `AutoCompleted` aktörsüz | Entegrasyon | B5 |
| BR-PM-13 | iptal gerekçesi veli DTO'sunda ve bildirim gövdesinde yok (şema testi) | Api sözleşme | B2/B5 |
| BR-PM-14 | `CanViewInfo=false`, revoked, `ValidUntil` geçmiş ilişki davetli değil | Application | B2 |
| BR-PM-15 / 41 | Published'da ekleme/düşme + N-01 yalnız yeniye; Started'da uzlaştırma no-op | Domain + Application | B1/B5 |
| BR-PM-16 | ikiz: tek yanıt iki öğrenci satırında; iki şube iki yanıt | Application | B3 |
| BR-PM-17 | önizleme sayıları = uygulamanın sıfırladığı sayılar (aynı çekirdek, golden); `impactHash`'siz 409 | Application | B2 |
| BR-PM-18 | tarih değişince güncel yanıtlar `noResponse`, eski sürüm satırları + `RescheduleReset` geçmişi; yalnız yer değişince sürüm artmaz | Domain | B1 |
| BR-PM-19 | veli Started'da 409, `now ≥ StartsAt` iken (iş koşmamış olsa da) 409; idare Started'da ✓ | Domain | B1/B3 |
| BR-PM-20 / D1 | veli adına geçmiş alanları; geçmiş tablosuna Update/Remove çağrısı yok (mimari test) | Application + Mimari | B3 |
| BR-PM-21 / Y2 | veli DTO'sunda öteki veli alanı **şemada yok**; istekte `parentPersonId` verilse yok sayılır | Api sözleşme | B3 |
| BR-PM-22 / 23 | çocuk başına kart; çakışma 60 dk varsayımıyla; engellemez | Application | B3/B2 |
| BR-PM-24 | rehber yanıtsız listesini görür, A13'e 403 | Application | B3 |
| BR-PM-25 / 26 | presence beyandan ön dolmaz; Completed'da işaretsiz → `absent`; davetli olmayan veliye presence 404 | Domain | B1/B4 |
| BR-PM-27 / O4 | 4000 üstü 400, kontrol karakteri temizliği, 5 dk birleştirme | Domain | B1/B4 |
| BR-PM-28 / Y3 | yazan öğretmen (aktif) okur; okuldan ayrılmış yazan 404; diğer öğretmen 404; veli/öğrenci için not ucu yok | Application | B4 |
| BR-PM-29 | otomatik kapanış notu silmez | Entegrasyon | B5 |
| BR-PM-31 / K3 | her S5 çağrısı `Read` satırı; denetim satırında metin kolonu yok (şema); `IAuditLogger` çağrılmaz | Application + Mimari | B4 |
| BR-PM-32 | sabit metin core'da tek yerde | UI birim | U3 |
| BR-PM-33 / Y5 | imha: not + tüm sürümlerde `Body NULL`, denetim kalır, ikinci koşu no-op | Entegrasyon | B5 |
| BR-PM-34 / 39 | N-06 sessiz saatte push atlanır, in-app yazılır; N-01 ertelenir | Birim (`PushQuietHours`) + Application | B5 |
| BR-PM-35 | tarih değiştir + iptal: kuyruktaki N-03 tazelik kontrolünde atılır; ertelenmiş push da | Application | B5 |
| BR-PM-36 | N-05/N-06/N-07 için e-posta kanalı `false` (katalog) | Entegrasyon | B6 |
| BR-PM-37 / 38 | N-07 yalnız yanıtsızlara; N-05 yalnız Attending'e; sabah saatinden sonra doğan sürümde gönderilmeden damga | Application | B5 |
| BR-PM-40 | hesapsız veliye in-app/push/e-posta yok, SMS niyeti var | Application | B7 |

### 15.2 Güvenlik şartı testleri

| Şart | Test |
|---|---|
| K1 | üretim ortamında izin listesi dışı SMTP host → `OptionsValidationException` (başlangıç testi) |
| K2 | tercihsiz veli + `GRADE_PUBLISHED` → e-posta `false`; tercihsiz veli + `PARENT_MEETING_PUBLISHED` → `true`; `EmailEnabled=false` → `false` |
| K3 / Y4 | log sink'te (test sink) not gövdesindeki işaret dizgesi hiçbir log olayında geçmez — not kaydet, 409, doğrulama hatası yollarında; mimari bekçi: `ParentMeetingNote.Body` okuyan tipler yalnız `GetParentMeetingNote`, `SaveParentMeetingNote`, `NoteRetentionJob` |
| Y1 | süper yönetici oturumunda başka okulun şubesi 404; `CallerPersonSchoolPredicateTests` yeni dosyaları da tarar |
| Y6 | 11. yanıt değişikliği 429 |
| O3 | SMS no-op sonrası log sink'te E.164 deseni yok |
| D2 | otomatik geçişlerin denetim satırı `ActorPersonId = null` |

### 15.3 Diğer

- Sözleşme testleri: her uç mock fixture → JSON şekil karşılaştırması (`HomeworkTrackingWireShapeTests` emsali).
- Rota testi: `ParentMeetingsControllerRouteTests` (sabit yollar `{id}`'den önce).
- `NotificationKindMetadataTests`, `PushEventKeyMapTests`, `NotificationEventKeyMapTests` yeni türleri kapsar (mevcut bekçiler enum'un her değerini ister).
- `ParentMeetingCounters` = SQL sayaçları (golden).
- UI: MSW kapalı koşum; not paneli çevrimdışı taslak; 409 sürüm çatışmasında yerel metin kaybolmaz.

---

## 16. Mimari uyum

| Kural | Durum |
|---|---|
| Application EF bilmez, Controller tek satır, pipeline sırası | Uygun |
| `IHasTenant` + global filtre + açık `SchoolId` yüklemi; `IgnoreQueryFilters` yalnız iş aday taramasında, gerekçeli | Uygun |
| Cross-modül yalnız kimlik + soyutlama | Uygun (`INotificationSourceFreshness` bağımlılığı tersine çevirir) |
| Hard delete yok | İstisna: not metni **kolon düzeyinde** NULL'lanır (KVKK imhası); satır silinmez — `[GŞ-Y5]` gereği |
| GET yan etkisiz | İstisna: not okuma denetimi (KVKK m.12) — belgeli |
| Kanal kararı komutta yok | Uygun (E-44 alıcı tercihidir; SMS niyeti kanal seçimi değildir, kapsam sabiti) |

---

## 17. Açık sorular ve riskler

### 17.1 Faruk'a karar

| # | Soru | Öneri |
|---|---|---|
| **F-1** | Üretim e-posta sağlayıcısı (bugün kişisel Hotmail SMTP). İzin listesi dışında uygulama başlamasın mı, yoksa yalnız e-posta kanalı mı kapalı kalsın? | Kurumsal bir işlem e-postası sağlayıcısı (SPF/DKIM'li okul alan adı); izin listesi dışı → **başlangıç hatası** (sessiz kapanış pilotta fark edilmez) |
| **F-2** | E-44'te mevcut katalogdaki 7 `default_email_enabled = 1` satırının sıfırlanması | Evet (göçle sıfırla) — ADR 0027 |
| **F-3** | Hangi mevcut türler e-posta **seçilebilir** olsun (`supports_email`)? | Teslim edilen + aile/öğrenci kitleli türler; öğretmen dürtmeleri (ATT_REMINDER, GRADE_ENTRY_REMINDER…) hariç |
| **F-4** | Varsayılan-açık e-postanın KVKK/aydınlatma metni teyidi (hukuk) | Aydınlatma metnine e-posta kanalı eklenmeden B6 üretime çıkmasın |
| **F-5** | E-postada tek dokunuş vazgeçme bağlantısı (K-13 "bağlantı yok" ile gerilir) | `List-Unsubscribe` başlığı + ayarlar yönergesi; imzalı bağlantı V1 dışı |
| **F-6** | E-posta ve push gövdesinde çocuğun adı (yalnız ad) ve toplantı açıklaması | Ad: evet (yalnız ad); açıklama: hayır |
| **F-7** | `ValidUntil` süzgecinin tüm veli kapsamlarına yayılması | Ayrı TB; bu modül yeni süzgeci baştan kullanır |
| **F-8** | `parentMeetings.read` SuperAdmin'e (AllPermissionIds) girsin mi? | Hayır — platform hesabı veli toplantısını görmez |
| **F-9** | Modül kataloğu anahtarı `parentMeetings` mı `parentMeeting` mı? | `parentMeetings` (mevcut çoğul kalıp) |
| **F-10** | ADR 0025/0026/0027 kabulü | Üçü de öneriyle |

### 17.1a Karar kaydı (Faruk, 2026-10-15)

F-2…F-10 ve tasarım soruları için öneriler kabul edildi. Bağlayıcı hükümler:

- **F-1 açık:** Üretim e-posta sağlayıcısı henüz seçilmedi. Karar gelene kadar e-posta kanalı izin listesi kapısıyla **kapalı** kalır (liste dışı `Smtp:Host`/`FromAddress` → kanal devre dışı ve uyarı logu); diğer dilimler bunu beklemez.
- **F-2:** 7 mevcut `default_email_enabled = 1` satırı göçle sıfırlanır.
- **F-3:** Push kapsamındaki tüm türler e-posta için **seçilebilir**; varsayılan kapalı, yalnız veli toplantısı yayın/değişiklik/iptal türlerinde açık.
- **F-4:** Aydınlatma metni hukuk teyidi pilot öncesi; `dev` merge'ünü engellemez.
- **F-5:** E-posta gövdesinde **bağlantı yok**; altta "Uygulamada Ayarlar › Bildirimler'den kapatabilirsiniz" metni.
- **F-6:** E-posta/push gövdesinde çocuğun yalnız adı; toplantı açıklaması yok.
- **F-7:** `ValidUntil` süzgecinin diğer modüllere yayılması ayrı iş (TB); bu modül ortak yardımcıyı baştan kullanır.
- **F-8:** SuperAdmin hiçbir `parentMeetings.*` iznini almaz.
- **F-9:** Modül anahtarı `parentMeetings`.
- **F-10:** ADR 0025, 0026, 0027 kabul edildi.
- **Tasarım:** mobil sihirbaz tam boy Sheet; rehber rapor görünümü PM-A-06 rol varyantı, yanıtsız/hesapsız liste PM-T-02 segmenti; PM-A-07'de N-07 aç/kapa (2 gün sabit) ve saklama kısaltma onayı; veli anasayfa kartı yayından yanıt verene kadar "yanıt bekleniyor" + toplantı günü kartı, öğretmen kartı toplantı günü + tamamlanmayı bekleyen; veli ekranları tüm çocukları gösterir, fiili katılım veliye gösterilmez; PM-X-01 web + mobil; Başladı'dan sonra beyan donar (idare veli adına yanıt dahil), rehber öğretmen Excel dışa aktaramaz (yalnız idare). Veli web menüsünün varlığı Faz 3'te UI temelinde doğrulanır.

### 17.2 Teknik riskler

| Risk | Etki | Önlem |
|---|---|---|
| Tazelik kancası `DispatchNotificationJob`/`DeferredPushJob`'a dokunur — tüm modüllerin bildirim yolu | Orta | Port yalnız kayıtlı `SourceType` için çağrılır; kaynaksız bildirim eski yoldan; `DeferredPushJob` eski imzası korunur |
| `DeferredPushJob` bellek içi zamanlayıcıda restart'ta kaybolur (mevcut borç) | Orta | Değişmez; Hangfire SQL depolamada kalıcı |
| Okul geneli N-01 fan-out (≈ 1.000-2.000 veli) — `PushEventKeyMap` "fan-out ölçüsü" ilkesi | Orta | Toplu push yolu (500'lük paket) var; şube başına ayrı `Enqueue` doğal dilimleme sağlar; ilk pilotta izlenir |
| E-posta kanalı toplu yol taşımıyor (alıcı başına sorgu) | Orta | B6'da toplu tercih okuması |
| Not okuma denetimi hacmi (panel her açılış) | Düşük | İndeksli append-only; 5 dk birleştirme yalnız yazmada |
| `PrimaryEmail` doğrulanmamış | Orta | §9.5; doğrulama ayrı iş |
| Backend modül kapısı yok | Düşük | §6.3 bulgusu; genel TB adayı |
| Hesapsız veli V1'de hiçbir kanaldan ulaşılamaz (K-14) | Yüksek (ürün) | Önizleme + rapor sayısı + telefon kısayolu |

---

*OKSİS · Veli Toplantısı Modülü Teknik Analiz (V1) · REQ-043 · 15 Ekim 2026 · Geliştirme başladığında davranışın tek yetkili kaynağı çalışan sözleşme (mock/kod) olacaktır.*
