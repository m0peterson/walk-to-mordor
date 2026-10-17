# Данные

Три места хранения: Room (шаги, пути, журнал), DataStore (настройки), `assets` модуля `:content` (маршрут, тексты, карта, картинки, только чтение).

## Room

База `longwalk.db`, класс `LongwalkDatabase` в `:core:data`. Схемы экспортируются в `core/data/schemas/` и коммитятся. Любое изменение схемы: новая версия и миграция с тестом. `fallbackToDestructiveMigration` запрещён: это стёр бы пользователю путь.

### `journey`

| Колонка | Тип | Примечание |
|---|---|---|
| `id` | INTEGER PK autoincrement | |
| `route_id` | TEXT | `"shire-to-mordor"` |
| `started_at` | INTEGER | epoch seconds, начало локального дня старта |
| `start_date` | TEXT | ISO `2026-10-17` |
| `deadline` | TEXT | ISO, включительно |
| `original_deadline` | TEXT | ISO |
| `plan_anchor_at` | INTEGER | epoch seconds |
| `plan_anchor_steps` | INTEGER | |
| `stride_m` | REAL | |
| `route_scale` | REAL | 1.0 |
| `state` | TEXT | `ACTIVE`, `COMPLETED`, `ABANDONED` |
| `completed_at` | INTEGER NULL | epoch seconds |
| `victory_seen` | INTEGER | 0/1 |
| `missed_notice_for` | TEXT NULL | ISO-дата срока, для которого уже показан экран «срок вышел» |
| `created_at` | INTEGER | epoch seconds |

Инвариант: не больше одной строки с `state = ACTIVE`. Репозиторий проверяет это в транзакции при `start`.

### `step_bucket`

| Колонка | Тип | Примечание |
|---|---|---|
| `hour_start` | INTEGER PK | epoch seconds начала локального часа |
| `steps` | INTEGER | ≥ 0 |
| `source` | TEXT | `HEALTH_CONNECT` или `SENSOR` |
| `updated_at` | INTEGER | epoch seconds |

- Корзины не привязаны к пути. Путь берёт сумму корзин с `hour_start ≥ journey.started_at`. Поэтому новый путь после победы не требует копировать шаги.
- Часы выравниваются по **локальному** времени в момент записи (важно для поясов со сдвигом в полчаса, например +05:30).
- Индекс по `hour_start` уже есть как PK. Сумма за любой период: один `SELECT SUM`.
- Объём: 24 строки в день, ~17 500 за два года. Чистить не нужно.

### `waypoint_visit`

| Колонка | Тип | Примечание |
|---|---|---|
| `journey_id` | INTEGER | FK → `journey.id`, ON DELETE CASCADE |
| `waypoint_id` | TEXT | id из `route.json` |
| `reached_at` | INTEGER | epoch seconds, по корзинам (см. правила игры, раздел 6) |
| `seen` | INTEGER | 0/1, показана ли карточка или плашка |

PK: (`journey_id`, `waypoint_id`).

### v1.1: `achievement_unlock`

| Колонка | Тип |
|---|---|
| `achievement_id` | TEXT PK |
| `journey_id` | INTEGER NULL |
| `unlocked_at` | INTEGER |
| `seen` | INTEGER |

Добавляется миграцией 1 → 2 в T-602.

## DataStore

Файл `settings.preferences_pb` в `:core:data`, модель `AppSettings`:

| Ключ | Тип | По умолчанию | Что |
|---|---|---|---|
| `onboarding_done` | Boolean | false | |
| `step_source` | String? | null | `HEALTH_CONNECT`, `SENSOR` или нет источника |
| `source_since` | Long? | null | epoch seconds, с какого момента пишет текущий источник |
| `last_sync_at` | Long? | null | для «обновлено N мин назад» |
| `last_sync_error` | String? | null | код последней ошибки, null если всё хорошо |
| `notify_waypoints` | Boolean | true | |
| `notify_reminder` | Boolean | true | вечернее напоминание, если отстаёте |
| `reminder_time` | String | `"19:00"` | |
| `notify_deadline` | Boolean | true | за 7 дней до срока и в день срока |
| `dismissed_no_background` | Boolean | false | пользователь закрыл плашку «шаги обновляются только при открытии» |

Отдельный DataStore `sensor_state.preferences_pb` в `:core:steps` (не в резервной копии):

| Ключ | Тип | Что |
|---|---|---|
| `last_counter` | Long? | последнее значение аппаратного счётчика |
| `last_boot_count` | Int? | `Settings.Global.BOOT_COUNT` при последнем замере |
| `last_sample_at` | Long? | epoch seconds |

## Контент

Все файлы в `content/src/main/assets/`. Как называть картинки и что делать, если их нет: [content-pack.md](content-pack.md).

### `route/route.json`

```json
{
  "schemaVersion": 1,
  "id": "shire-to-mordor",
  "lengthKm": 2863,
  "waypoints": [
    {
      "id": "bag-end",
      "kind": "major",
      "number": 1,
      "km": 0,
      "point": [0.103, 0.159],
      "illustration": "illustrations/waypoints/01-bag-end.webp"
    },
    {
      "id": "woody-end",
      "kind": "minor",
      "km": 35,
      "point": [0.128, 0.162],
      "via": [[0.115, 0.165]]
    }
  ]
}
```

- `point`: `[x, y]`, доли от ширины и высоты карты, 0..1, от левого верхнего угла.
- `via`: промежуточные точки линии маршрута **от предыдущей вехи до этой**. У первой вехи нет. Нужны, чтобы линия шла по дороге или реке, а не по прямой.
- `illustration`: необязательно. Если поле есть, а файла нет, это не ошибка (картинки приоритета B появятся позже).
- `number`: только у `major`.

Проверки (`RouteValidator`, в тестах на реальном файле):

1. `schemaVersion == 1`.
2. `id` уникальны, только `[a-z0-9-]`.
3. Первая веха `km = 0`, `km` строго растут, последняя `km == lengthKm`.
4. У `major` номера 1..23 подряд, у `minor` номера нет.
5. Все координаты в 0..1.
6. Разрыв между соседними вехами не больше 100 км (`MAX_WAYPOINT_GAP_KM`).
7. Для каждой вехи есть тексты во всех языках из `texts/`.

### `texts/ru.json`, `texts/en.json`

```json
{
  "appTitle": "Путь в Мордор",
  "routeTitle": "От Шира до Роковой горы",
  "disclaimer": "Неофициальное фанатское приложение...",
  "canonPaceLabel": "Как в книге",
  "screens": {
    "onboarding-journey": { "title": "...", "body": "..." },
    "onboarding-deadline": { "title": "...", "body": "..." },
    "onboarding-steps": { "title": "...", "body": "..." },
    "victory": { "title": "...", "body": "..." },
    "victory-extended": { "title": "...", "body": "..." },
    "deadline-missed": { "title": "...", "body": "..." }
  },
  "waypoints": {
    "bag-end": {
      "title": "Бэг-Энд",
      "place": "Хоббитон, Шир",
      "season": "конец сентября",
      "story": "Два-четыре предложения своими словами."
    }
  }
}
```

- `title` обязательно, остальное необязательно. У малых вех обычно только `title` и `story` в одно-два предложения.
- `screens`: тексты экранов, в которых есть отсылки к миру (онбординг, победа, срок вышел). Все шесть ключей обязательны. Нейтральные строки интерфейса («Далее», «шагов в день») живут в `strings.xml` фич, а не здесь.
- Язык выбирается по локали телефона, запасной английский.

### `map/meta.json`

```json
{ "width": 6144, "height": 6144, "tileSize": 256, "levelCount": 6, "format": "webp" }
```

Пишется скриптом нарезки (T-301) вместе с тайлами `map/tiles/{level}/{row}/{col}.webp`. Приложение не знает реальный размер карты заранее: всё берёт отсюда. Поэтому замена заглушки на финальную карту: только перегенерация тайлов, код не меняется.

### Связь `point` и карты

Координаты в долях, поэтому не зависят от разрешения. Но если на финальной карте горы или река легли не там, где на схеме, точки нужно снять заново с финального изображения редактором `tools/route-editor/` (T-105). Это задача владельца A-3 в [roadmap](../roadmap.md).
