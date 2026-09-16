# Datenbank-Schema

> Das komplette PostgreSQL-Schema (40 Migrationen, 32 Tabellen) der UntisX-Server-Datenbank.

---

## ER-Diagramm (vereinfacht)

```
                          ┌───────────────┐
                          │  users        │  id, user_name(unique), role,
                          │               │  password_hash, class_id ──┐
                          └───┬───────┬───┘                            │
                              │       │                                │
                 ┌────────────┴───┐   │                                │
                 │  sessions      │   │                                │
                 │  id, user_id,  │   │                                │
                 │  token_hash    │   │                                │
                 └────────────────┘   │                                │
                                      ▼                                ▼
                    ┌──────────────────────────┐   ┌──────────────────────────┐
                    │ school_settings (id=1)    │   │ classes                  │
                    │ school_name, timezone, …  │   │ id, name(unique),        │
                    └──────────────────────────┘   │ class_teacher(FK users)   │
                                                   └──────────┬───────────────┘
                                                              │
    ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
    │ subjects      │  │ rooms         │  │ bell_schedule │   │
    │ id,name,color │  │ id,building…  │  │ id,start,end  │   │
    └───────┬───────┘  └───────┬───────┘  └───────────────┘   │
            │                  │                              │
            └──────────┬───────┴──────────┐                   │
                       ▼                  ▼                   ▼
              ┌───────────────────────────────────────────────┐
              │ timetable_entries                              │
              │ id, subject_id, teacher_id, room_id, class_id,│
              │ first_started_at, first_ended_at,              │
              │ repeats_every_days, valid_until                │
              └───────┬───────────────┬───────────────┬────────┘
                      │               │               │
                      ▼               ▼               ▼
         ┌─────────────────┐ ┌────────────────┐ ┌─────────────────┐
         │ cancellations   │ │ substitutions  │ │ homework        │
         └─────────────────┘ └────────────────┘ └─────────────────┘
   ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
   │ grades          │ │ absences        │ │ messages        │ │ custom_events   │
   │ student_id,     │ │ student_id,     │ │ sender,receiver │ │ user_id,        │
   │ subject_id,     │ │ start,end,      │ │ subject,body,   │ │ date,color,     │
   │ grade,weight    │ │ excused         │ │ priority        │ │ recurrence      │
   └─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────┘

   ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
   │ conversations   │ │ chat_messages   │ │ notifications   │ │ audit_log       │
   │ id,type,name,   │ │ conversation_id,│ │ user_id,type,   │ │ user_id,action, │
   │ created_by      │ │ sender_id,body  │ │ title,message   │ │ entity,ip       │
   └────────┬────────┘ └─────────────────┘ └─────────────────┘ └─────────────────┘
            │
   ┌────────▼─────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
   │ conversation_    │ │ user_preferences│ │ settings_tabs   │ │ api_keys        │
   │ members          │ │ user_id(PK),    │ │ role(PK),tabs   │ │ name,key_hash,  │
   │ role,muted,      │ │ theme,accent,   │ │ moodle_label    │ │ scopes,active   │
   │ last_read_id     │ │ dashboard_layout│ └─────────────────┘ └─────────────────┘
   └──────────────────┘ └─────────────────┘   ┌─────────────────┐
                                               │ integrations    │
    ┌─────────────────┐ ┌─────────────────┐   │ id,name,url,    │
    │ resource_versions│ │ teacher_absences│   │ icon,enabled,   │
    │ resource,version│ │ teacher_id,date │   └─────────────────┘
    └─────────────────┘ └─────────────────┘
    ┌─────────────────┐ ┌─────────────────┐   ┌─────────────────┐
    │ video_meetings  │ │ class_book_     │   │ resources       │
    │ title,room_name │ │ entries         │   │ name,type,      │
    │ host_id,active  │ │ class,subject,  │   │ location,cap    │
    └─────────────────┘ │ period,content  │   └────────┬────────┘
    ┌─────────────────┐ └─────────────────┘            │
    │ chat_attachments│                        ┌───────▼────────┐
    │ storage_key,    │                        │ bookings       │
    │ data (BYTEA)    │                        │ resource_id,   │
    └─────────────────┘                        │ user_id,dates  │
                                               └────────────────┘
```

---

## Tabellen im Detail

### `users` – Benutzer

| Spalte | Typ | Hinweis |
|--------|-----|---------|
| `id` | `BIGSERIAL` PK | |
| `user_name` | `VARCHAR` UNIQUE | Login-Name |
| `real_name` | `VARCHAR` | |
| `short_name` | `VARCHAR` | Abkürzung (optional) |
| `role` | `VARCHAR` | `admin`/`teacher`/`student` |
| `email` | `VARCHAR` UNIQUE (nullable) | |
| `password_hash` | `VARCHAR` nullable | Erst nach Aktivierung gesetzt |
| `activation_key_hash` | `VARCHAR` nullable | Aktivierungsschlüssel (Hash) |
| `activation_expires_at` | `TIMESTAMP` nullable | Ablauf des Schlüssels |
| `activated_at` | `TIMESTAMP` nullable | Wann aktiviert |
| `class_id` | FK → `classes` | Nur für Schüler/Lehrer relevant |
| `created_at` | `TIMESTAMP` | |

### `sessions` – Login-Sessions

| Spalte | Typ | Hinweis |
|--------|-----|---------|
| `id` | `BIGSERIAL` PK | |
| `user_id` | FK → `users` | |
| `token_hash` | `BYTEA` UNIQUE indexed | SHA-256 des Bearer-Tokens |
| `expires_at` | `TIMESTAMP` | |
| `created_at` | `TIMESTAMP` | |

### `school_settings` – Schuldaten (einzeilig)

| Spalte | Hinweis |
|--------|---------|
| `id` | SMALLINT PK, immer `1` |
| `school_name` | |
| `timezone` | |
| `address` | eingeführt in Migration 020 |
| `phone` | Migration 020 |
| `email` | Migration 020 |
| `updated_at` | |

### `classes` – Klassen

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `name` UNIQUE | z.B. `5a` |
| `grade_level` | |
| `room` | |
| `class_teacher` | FK → users |
| `class_deputy_teacher` | FK → users |

### `subjects` – Fächer

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `name` | z.B. `Mathematik` |
| `short_name` | z.B. `MATH` |
| `color` | Hex-Farbe `#RRGGBB` |
| `active` | Soft-Delete-Flag (Default true) |
| `created_at` / `updated_at` | |

### `rooms` – Räume

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `name` | z.B. `R202` |
| `building` | |
| `capacity` | |
| `room_type` | z.B. `Klassenzimmer`/`Fachraum` |
| `floor` | |
| `accessible` | Barrierefrei |
| `notes` | |
| `active` | Soft-Delete-Flag |

### `bell_schedule` – Klingelzeiten

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `kind` | z.B. `lesson` / `break` |
| `label` | z.B. `1. Stunde` |
| `start_time` | `HH:MM` |
| `end_time` | `HH:MM` |

### `timetable_entries` – Stundenplan-Einträge

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `subject_id` FK → subjects | |
| `teacher_id` FK → users (nullable) | Migration 029 |
| `room_id` FK → rooms (nullable) | Migration 029 |
| `class_id` FK → classes | Migration 024 |
| `first_started_at` | Datum der ersten Stunde |
| `first_ended_at` | Ende der ersten Stunde |
| `repeats_every_days` | Default 7 (wöchentlich) |
| `valid_until` | Default `2099-12-31` |

> **Berechnete Felder in API-Antworten:** `day_of_week` (aus first_started_at), `period` + `period_count` (aus harter Codierung der 12 Zeit-Slots), `week_type` (ungerade/gerade Woche).

### `cancellations` – Stundenausfälle

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `timetable_entry_id` FK | Welcher Eintrag fällt aus |
| `date` | Datum |
| `reason` | Grund |
| `cancelled_by` FK → users | |
| `created_at` | |

### `substitutions` – Vertretungen

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `timetable_entry_id` FK | |
| `date` | |
| `original_teacher_id` FK | |
| `substitute_teacher_id` FK | |
| `original_subject_id` FK | |
| `substitute_subject_id` FK | |
| `room` | Raum |
| `reason` | |
| `type` | `teacher`/`room`/`subject`/`merge` |
| `status` | `active`/`cancelled` (Soft-Delete) |
| `created_by` FK | |
| `created_at` | |

### `teacher_absences` – Lehrer-Abwesenheiten

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `teacher_id` FK | |
| `date` | |
| `reason` | |
| `created_at` | |

### `homework` – Hausaufgaben

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `subject_id` FK | |
| `teacher_id` FK | |
| `title` | |
| `content` | |
| `date` | Fälligkeitsdatum |
| `date_until` | optional (Migration 030) |
| `created_at` | |

### `grades` – Noten

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `student_id` FK | |
| `subject_id` FK | |
| `teacher_id` FK | |
| `grade` | Dezimal 1.0–6.0 |
| `weight` | Default 1 |
| `kind` | `oral`/`test`/`exam`/`other` |
| `comment` | |
| `created_at` | |

### `absences` – Fehlzeiten

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `student_id` FK | |
| `start_time` | |
| `end_time` | |
| `reason` | |
| `excused` | Entschuldigt? |
| `recorded_by` FK → users | |
| `created_at` | |

### `messages` – (In-App-)Nachrichten

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `sender_id` FK | |
| `receiver_id` FK | |
| `subject` | Betreff |
| `body` | Inhalt |
| `priority` | `low`/`normal`/`high`/`urgent` |
| `read_at` | gelesen? |
| `created_at` | |

### `custom_events` – Eigene Kalendereinträge

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `user_id` FK | Eigentümer |
| `title` | |
| `description` | |
| `date` | |
| `time_start` / `time_end` | |
| `color` | Default `#4A90D9` |
| `recurrence` | `none`/`daily`/`weekly`/`biweekly`/`monthly`/`yearly` |

### `conversations` – Chats

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `type` | `direct`/`group` |
| `name` | |
| `created_by` FK | |
| `created_at` | |

### `conversation_members` – Chat-Mitglieder

| Spalte | Hinweis |
|--------|---------|
| `conversation_id` + `user_id` | Composite PK |
| `role` | `member`/`admin` |
| `muted` | |
| `last_read_message_id` | Read-Status |
| `joined_at` | |

### `chat_messages` – Chat-Nachrichten

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `conversation_id` FK | |
| `sender_id` FK | |
| `body` | max. 5000 Zeichen |
| `created_at` | |

### `notifications` – Benachrichtigungen

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `user_id` FK | |
| `type` | |
| `title` | |
| `message` | |
| `entity_type` / `entity_id` | Verknüpfung |
| `read` | |
| `created_at` | |

### `audit_log` – Audit-Trail

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `user_id` FK | Wer |
| `action` | `CREATE`/`UPDATE`/`DELETE` |
| `entity_type` / `entity_id` | Woran |
| `details` | Details (JSON) |
| `ip_address` | |
| `created_at` | |

### `settings_tabs` – Tab-Konfiguration

| Spalte | Hinweis |
|--------|---------|
| `role` PK | `admin`/`teacher`/`student` |
| `tabs` | JSONB-Liste sichtbarer Seiten |

### `integrations` – Drittanbieter-Integrationen

| Spalte | Hinweis |
|--------|---------|
| `id` TEXT PK | z.B. `moodle` |
| `name` | |
| `url` | |
| `icon` | |
| `enabled` | |
| `roles` | JSONB |

### `api_keys` – API-Schlüssel

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `name` | |
| `key_hash` UNIQUE indexed | SHA-256-Hex |
| `key_hint` | letzte Zeichen zur Identifikation |
| `scopes` | JSON-String der erlaubten Pfade |
| `api_type` | `read`/`write`/`full` |
| `active` | |
| `created_at` / `updated_at` | |

### `resource_versions` – Sync

| Spalte | Hinweis |
|--------|---------|
| `resource_name` PK | max 20 Zeichen |
| `version` | Monoton steigende Version |
| `updated_at` | |

### `user_preferences` – Benutzereinstellungen

| Spalte | Hinweis |
|--------|---------|
| `user_id` PK FK | |
| `theme` | `light`/`dark` |
| `accent_color` | Default `#3498db` |
| `dashboard_layout` | JSONB (Widget-Grid) |
| `tutorial_seen` | |
| `created_at` / `updated_at` | |

### `video_meetings` – Video-Meetings

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `title` | Meeting-Name |
| `description` | Beschreibung |
| `room_name` UNIQUE | Eindeutiger Raumname (für WebRTC) |
| `host_id` FK → users | Ersteller |
| `starts_at` | Startzeit |
| `ends_at` | Endzeit |
| `jitsi_server` | Default `meet.jit.si` |
| `is_active` | INTEGER (als Bool serialisiert) |
| `created_at` | |

### `resources` – Buchbare Ressourcen

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `name` UNIQUE | z.B. `IT-Raum 101` |
| `resource_type` | `computer_room`/`tablet`/`beamer`/`subject_room`/`laptop`/`other` |
| `location` | Standort |
| `description` | |
| `capacity` | Kapazität |
| `active` | INTEGER (Soft-Delete) |
| `created_at` | |

### `bookings` – Zeitbuchungen

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `resource_id` FK → resources | |
| `user_id` FK → users | |
| `booking_date` | `YYYY-MM-DD` |
| `time_start` | `HH:MM` |
| `time_end` | muss > time_start |
| `purpose` | Zweck |
| `status` | `confirmed`/`cancelled` |
| `created_at` | |

### `class_book_entries` – Digitales Klassenbuch

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `class_id` FK → classes | |
| `teacher_id` FK → users | |
| `subject_id` FK → subjects | |
| `date` | `YYYY-MM-DD` |
| `period` | 1–12 |
| `content` | Unterrichtsinhalt |
| `homework` | Hausaufgabe (optional) |
| `entry_type` | `lesson`/`homework`/`test`/`project`/`other` |
| `remarks` | Anmerkungen |
| `created_at` / `updated_at` | |

### `chat_attachments` – Chat-Anhänge

| Spalte | Hinweis |
|--------|---------|
| `id` PK | |
| `conversation_id` FK | |
| `sender_id` FK | |
| `storage_key` UNIQUE | Speicher-Schlüssel |
| `file_name` | Originaldateiname |
| `file_size` | Größe in Bytes |
| `mime_type` | MIME-Typ |
| `data` | BYTEA (Datenbank-Speicherung) |
| `created_at` | |

---

## Migrationen (40 Stück)

Sie liegen in `server-default/migrations/` und werden beim Server-Start automatisch (in Reihenfolge) angewendet.

| Nr. | Inhalt (grob) |
|-----|---------------|
| 001–010 | Basis: users, sessions, school_settings, subjects, rooms, classes, timetable_entries, cancellations, substitutions, homework |
| 011–020 | grades, absences, messages, notifications, audit_log, settings_tabs, custom_events, teacher_absences, school_settings erweitert |
| 021–030 | conversations, conversation_members, chat_messages, resource_versions, user_preferences, integrations, API-Keys, Klasse am Timetable, teacher/room nullable, date_until |
| 031–040 | Performance-Indizes, Chat-Anhänge (m31/m40), class_book_entries, resources, bookings, video_meetings, Constraints |

> sqlx (v0.9) prüft Queries **zur Compilezeit** – Migrations-Schema und SQL müssen also exakt übereinstimmen, sonst baut das Projekt nicht.