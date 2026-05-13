# PROJECT_MAP — تطبيق عبد الله الزخيم (Flutter APK)

> Generated: 2026-05-12 | Flutter 3.41.5 | Dart 3.11.3
> Target: Android APK — مشغل صوتيات مخصص للداعية عبد الله الزخيم

---

## [TECH_STACK]

| Layer              | Package                        | Version   | Role                                        |
|--------------------|--------------------------------|-----------|---------------------------------------------|
| **State Mgmt**     | `flutter_riverpod`             | ^3.3.1    | Reactive state, DI, async data              |
| **Code Gen**       | `riverpod_generator`           | ^3.x      | @riverpod annotations                       |
| **Routing**        | `go_router`                    | ^17.2.3   | Declarative navigation + ShellRoute         |
| **Audio Engine**   | `just_audio`                   | ^0.10.5   | Playback (local files, playlists, gapless)  |
| **Background**     | `just_audio_background`        | ^0.0.1-b17| Background playback + media notification    |
| **Waveform**       | `just_waveform`                | ^0.0.7    | Waveform extraction for visual rendering    |
| **Database**       | `drift`                        | ^2.33.0   | SQLite ORM — المحتوى الأساسي (categories + tracks + favorites) |
| **Models**         | `freezed`                      | ^3.2.5    | Immutable data classes                      |
| **Local Storage**  | `shared_preferences`           | ^2.x      | Theme pref, sleep timer, admin auth         |
| **Notif**          | `flutter_local_notifications`  | ^21.0.0   | Sleep timer notification                    |
| **File Picker**    | `file_picker`                  | ^8.x      | Admin picks audio files from device         |
| **Storage Path**   | `path_provider`                | ^2.1.5    | App documents dir (حفظ الملفات الصوتية)     |
| **Permissions**    | `permission_handler`           | ^12.0.1   | Admin only: قراءة ملفات الجهاز              |
| **HTTP Client**    | `http`                         | ^1.4.0    | GitHub CDN downloads for sync               |

---

## [SYSTEM_FLOW — User Journey]

```
[App Launch]
    │
    ├──► Seed import (assets/audio/ → documents dir + drift DB)
    │
    ├──► Auto-sync from GitHub CDN (manifest.json)
    │     ├── Download new audio files
    │     ├── Insert new categories, tracks, benefits, quotes, content
    │     └── Store manifest version for delta updates
    │
    ├──► Main Shell (go_router ShellRoute + BottomNavigationBar)
    │    │
    │    ├── 1. الرئيسية (Daily benefit + Quote + Content list + Categories grid)
    │    ├── 2. المفضلة (Favorites)
    │    ├── 3. بحث (Search)
    │    └── 4. الإعدادات (Settings)
    │
    ├──► Select Category → Select Track
    │
    ├──► Now Playing (mini-player bar + full-screen)
    │    │
    │    ├── Play/Pause/Seek/Next/Prev
    │    ├── Shuffle / Repeat (off / 1 / all)
    │    ├── Sleep Timer
    │    ├── Speed Control (0.5x – 2.0x)
    │    ├── Waveform Visualization
    │    └── Favorite toggle (♥)
    │
    ├──► Background: continues via just_audio_background
    │     • Lock screen controls
    │     • Media notification (play/pause/next/prev)
    │
    └──► [Admin]: Settings → لوحة التحكم → كود سري
         │
         ├── Dashboard (stats: tracks, categories, benefits, quotes, content)
         ├── إعدادات التطبيق (الاسم، الإصدار، رابط الشعار)
         ├── إدارة التصنيفات (CRUD + ترتيب)
         ├── إدارة المقاطع (CRUD + ترتيب + رفع ملف صوتي)
         ├── الفوائد اليومية (CRUD)
         ├── الأقوال (CRUD)
         ├── الخطب والمحتوى (CRUD + نص + PDF + صوت)
         ├── المزامنة عن بعد (سحب من GitHub + تصدير manifest.json)
         └── النسخ الاحتياطي (تصدير/استيراد JSON)
```

---

## [ARCHITECTURE — Feature-First]

```
lib/
├── main.dart                         # ProviderScope wrap + JustAudioBackground.init
├── app.dart                          # MaterialApp.router + seed + auto-sync on startup
│
├── core/
│   ├── theme/
│   │   ├── app_theme.dart            # Light/dark ThemeData (Saudi Vision 2030)
│   │   └── app_colors.dart           # Color palette (green #006C35, gold #C9A84C, cream)
│   ├── router/
│   │   ├── app_router.dart           # GoRouter + ShellRoute + all routes (24 routes)
│   │   └── route_names.dart          # Route path constants
│   ├── constants/
│   │   └── app_constants.dart        # App name, admin code, GitHub CDN URL, speeds
│   ├── logger/
│   │   └── app_logger.dart           # Async logging (FINE/INFO/WARNING/SEVERE)
│   ├── services/
│   │   └── sync_service.dart         # GitHub CDN sync: download manifest → files → DB
│   └── providers/
│       ├── sync_provider.dart        # Riverpod providers for sync state
│       └── content_providers.dart     # Riverpod streams for benefits, quotes, content, meta
│
├── data/
│   ├── seed/                         # البذرة الأولية (تُستورد أول تشغيلة)
│   │   ├── seed_data.dart            # Categories + tracks seed definitions
│   │   └── seed_importer.dart        # Copy seed audio to documents + insert into DB
│   ├── models/
│   │   ├── track.dart                # @freezed Track
│   │   └── category.dart             # @freezed Category
│   └── database/
│       ├── app_database.dart         # Drift DB definition (8 tables, all CRUD, export)
│       ├── tables/
│       │   ├── categories_table.dart # id, name, description, imagePath, sortOrder
│       │   ├── tracks_table.dart     # id, title, subtitle, categoryId, audioPath, ...
│       │   ├── favorites_table.dart  # trackId, dateFavorited
│       │   ├── track_stats_table.dart# trackId, playCount, lastPlayed
│       │   ├── app_meta_table.dart   # key-value store (app_name, version, logo_url, ...)
│       │   ├── daily_benefits_table.dart  # id, title, content, date, sortOrder
│       │   ├── quotes_table.dart     # id, content, author, sortOrder
│       │   └── content_items_table.dart   # id, title, type, textContent, pdfUrl, audioPath
│       └── app_database.g.dart       # Generated by build_runner (Drift)
│
├── features/
│   ├── player/                       # مشغل الصوت
│   │   ├── providers/
│   │   │   ├── audio_player_provider.dart   # Riverpod AudioPlayer
│   │   │   ├── queue_provider.dart          # Current queue + shuffle/repeat
│   │   │   └── sleep_timer_provider.dart    # Timer logic
│   │   ├── screens/
│   │   │   ├── now_playing_screen.dart      # Full player UI
│   │   │   └── mini_player.dart             # Persistent bottom bar
│   │   └── widgets/
│   │       ├── player_controls.dart         # Play/Pause/Skip/Shuffle
│   │       ├── seek_bar.dart                # Slider + time
│   │       ├── speed_selector.dart          # 0.5x–2.0x chip
│   │       └── waveform_view.dart           # just_waveform render
│   │
│   ├── home/                         # الرئيسية
│   │   ├── providers/
│   │   │   └── category_provider.dart
│   │   ├── screens/
│   │   │   ├── home_screen.dart             # Benefits + Quotes + Content + Categories
│   │   │   ├── category_tracks_screen.dart  # Tracks in category
│   │   │   └── content_detail_screen.dart   # View khutbah/lesson text + audio + PDF
│   │   └── widgets/
│   │       ├── category_card.dart
│   │       └── track_list_tile.dart
│   │
│   ├── favorites/                    # المفضلة
│   │   ├── providers/
│   │   │   └── favorites_provider.dart
│   │   ├── screens/
│   │   │   └── favorites_screen.dart
│   │   └── widgets/
│   │       └── favorite_button.dart
│   │
│   ├── search/                       # بحث
│   │   ├── providers/
│   │   │   └── search_provider.dart
│   │   ├── screens/
│   │   │   └── search_screen.dart
│   │   └── widgets/
│   │       └── search_result_tile.dart
│   │
│   ├── settings/                     # الإعدادات
│   │   ├── providers/
│   │   │   └── settings_provider.dart
│   │   ├── screens/
│   │   │   └── settings_screen.dart
│   │   └── widgets/
│   │       ├── theme_selector.dart
│   │       ├── sleep_timer_picker.dart
│   │       └── about_section.dart         # Shows remote app name + version
│   │
│   └── admin/                        # ★ لوحة التحكم
│       ├── providers/
│       │   ├── admin_auth_provider.dart
│       │   ├── admin_categories_provider.dart
│       │   ├── admin_tracks_provider.dart
│       │   └── admin_content_providers.dart  # Benefits, quotes, content item providers
│       ├── screens/
│       │   ├── admin_login_screen.dart
│       │   ├── admin_dashboard_screen.dart   # All management buttons
│       │   ├── admin_app_settings_screen.dart    # Name, version, logo URL
│       │   ├── admin_categories_screen.dart
│       │   ├── admin_category_edit_screen.dart
│       │   ├── admin_tracks_screen.dart
│       │   ├── admin_track_edit_screen.dart
│       │   ├── admin_daily_benefits_screen.dart
│       │   ├── admin_daily_benefit_edit_screen.dart
│       │   ├── admin_quotes_screen.dart
│       │   ├── admin_quote_edit_screen.dart
│       │   ├── admin_content_items_screen.dart
│       │   ├── admin_content_item_edit_screen.dart
│       │   ├── admin_sync_screen.dart          # Sync + export manifest
│       │   └── admin_backup_screen.dart
│       └── widgets/
│           ├── admin_stats_card.dart
│           └── admin_file_picker_button.dart
│
├── assets/                           # (اختياري — يمكن تركها فارغة، المحتوى عبر GitHub CDN)
│   ├── audio/quran/
│   ├── audio/nasheeds/
│   ├── audio/lectures/
│   ├── audio/duas/
│   └── images/categories/
│
└── android/app/src/main/AndroidManifest.xml
    • FOREGROUND_SERVICE + FOREGROUND_SERVICE_MEDIA_PLAYBACK
    • POST_NOTIFICATIONS, WAKE_LOCK, READ_MEDIA_AUDIO
    • AudioService (com.ryanheise.audioservice)
```

---

## [DATA MODEL — Drift Tables]

```sql
categories:
  id          INTEGER PRIMARY KEY AUTOINCREMENT
  name        TEXT NOT NULL
  description TEXT
  image_path  TEXT
  sort_order  INTEGER NOT NULL DEFAULT 0

tracks:
  id          INTEGER PRIMARY KEY AUTOINCREMENT
  title       TEXT NOT NULL
  subtitle    TEXT
  category_id INTEGER NOT NULL REFERENCES categories(id) ON DELETE CASCADE
  audio_path  TEXT NOT NULL
  duration_ms INTEGER NOT NULL DEFAULT 0
  image_path  TEXT
  sort_order  INTEGER NOT NULL DEFAULT 0

favorites:
  track_id       INTEGER PRIMARY KEY REFERENCES tracks(id) ON DELETE CASCADE
  date_favorited TEXT NOT NULL

track_stats:
  track_id    INTEGER PRIMARY KEY REFERENCES tracks(id) ON DELETE CASCADE
  play_count  INTEGER NOT NULL DEFAULT 0
  last_played TEXT

app_meta:
  key   TEXT PRIMARY KEY            -- 'app_name', 'app_version', 'logo_url', 'manifest_version'
  value TEXT

daily_benefits:
  id          INTEGER PRIMARY KEY AUTOINCREMENT
  title       TEXT NOT NULL
  content     TEXT NOT NULL
  date        TEXT
  sort_order  INTEGER NOT NULL DEFAULT 0

quotes:
  id          INTEGER PRIMARY KEY AUTOINCREMENT
  content     TEXT NOT NULL
  author      TEXT
  sort_order  INTEGER NOT NULL DEFAULT 0

content_items:
  id           INTEGER PRIMARY KEY AUTOINCREMENT
  title        TEXT NOT NULL
  type         TEXT NOT NULL            -- 'khutbah' | 'lesson'
  text_content TEXT
  pdf_url      TEXT
  audio_path   TEXT
  sort_order   INTEGER NOT NULL DEFAULT 0
```

---

## [GitHub CDN SYNC FLOW]

```
GitHub Repo (your-username/alzakheim-content)
│
├── manifest.json          ← القائمة الرئيسية (يُصدرها التطبيق)
├── audio/
│   ├── 001_fatiha.mp3
│   ├── 114_nas.mp3
│   └── ...
└── pdfs/
    └── khutbah_001.pdf

[App Launch]
    │
    ├──► GET manifest.json from raw.githubusercontent.com
    │
    ├──► Compare manifestVersion with local stored version
    │     └── If newer → download new files (audio + PDF)
    │
    ├──► Insert new rows into drift DB (categories, tracks, benefits, quotes, content)
    │
    └──► Update local manifest_version
```

### ملف manifest.json المصدَّر من التطبيق:
```json
{
  "manifestVersion": 1,
  "appName": "عبد الله الزخيم",
  "appVersion": "1.0.0",
  "logoUrl": "https://raw.githubusercontent.com/.../logo.png",
  "categories": [...],
  "tracks": [...],
  "dailyBenefits": [...],
  "quotes": [...],
  "contentItems": [...]
}
```

---

## [Admin Panel Features]

| Feature | Description |
|---------|-------------|
| **Secret Code** | Settings → لوحة التحكم → كود سري (2030zakheim) |
| **Dashboard** | إحصائيات: تصنيفات، مقاطع، فوائد، أقوال، محتوى |
| **App Settings** | اسم التطبيق، رقم الإصدار، رابط الشعار — تظهر للمستخدمين فوراً |
| **Categories** | إضافة/تعديل/حذف/ترتيب التصنيفات |
| **Tracks** | إضافة/تعديل/حذف/ترتيب المقاطع الصوتية |
| **Daily Benefits** | إضافة/تعديل/حذف الفوائد اليومية |
| **Quotes** | إضافة/تعديل/حذف أقوال الشيخ |
| **Content Items** | إضافة/تعديل/حذف خطب ودروس (نص + PDF + صوت) |
| **Remote Sync** | سحب المحتوى من GitHub CDN + تصدير manifest.json |
| **Backup** | تصدير/استيراد جميع البيانات كـ JSON |

---

## [LOGGING STRATEGY]

```dart
class AppLogger {
  static final _logger = Logger('AlZakheim');
  static void info(String tag, String msg) => _logger.info('[$tag] $msg');
  static void severe(String tag, String msg, [Object? e, StackTrace? s]) =>
      _logger.severe('[$tag] $msg', e, s);
}
```

---

## [PENDING / ORPHANS]

| Item | Status | Note |
|------|--------|------|
| AndroidManifest permissions | ✅ DONE | FOREGROUND_SERVICE, WAKE_LOCK, READ_MEDIA_AUDIO |
| Admin secret code config | ✅ DONE | `AppConstants.adminSecretCode` |
| ProGuard / R8 rules | PENDING | Keep just_audio + drift native libs |
| Adaptive icon | PENDING | Islamic-themed launcher icon |
| Arabic RTL support | PENDING | Directionality.rtl + Arabic fonts |
| Sleep timer notification | PENDING | Cancel alarm on notification tap |
| Equalizer integration | PENDING | Android AudioEffect — optional |
| Seed audio assets | PENDING | Populate assets/audio/ (or use GitHub CDN) |
| Cached waveform data | PENDING | Store .wave files to avoid re-extraction |
| Track reordering UI | PENDING | Drag-and-drop reorder in admin |
| Unit tests | PENDING | Riverpod + DAO tests |
| Widget tests | PENDING | UI component tests |
| GitHub repo setup | PENDING | Create repo, upload manifest + audio |
| APK signing & build | PENDING | `flutter build apk --release` |

---

## [MILESTONES]

### M1–M8 ✅ Complete
جميع الميزات الأساسية مكتملة: المشغل، الخلفية، المفضلة، البحث، السرعات، مؤقت النوم، الموجات، لوحة التحكم الكاملة.

### M8b — GitHub CDN + Admin Content Control (NEW)
- [x] 8 database tables (categories, tracks, favorites, stats, app_meta, daily_benefits, quotes, content_items)
- [x] SyncService: download manifest → audio files → update DB
- [x] Auto-sync on app launch
- [x] Admin: App Settings (name, version, logo)
- [x] Admin: Daily Benefits CRUD
- [x] Admin: Quotes CRUD
- [x] Admin: Content Items CRUD (text + PDF + audio)
- [x] Admin: Sync screen (manual sync + export manifest)
- [x] Home: Daily benefit, Quote, Content sections
- [x] User-facing Content Detail screen (audio + text + PDF)
- [ ] GitHub repo creation + initial upload instructions

### M9 — Polish & Release
- [ ] Adaptive launcher icon
- [ ] ProGuard / R8 minification
- [ ] Splash screen (native Android)
- [ ] `flutter build apk --release`
- [ ] App signed with keystore
