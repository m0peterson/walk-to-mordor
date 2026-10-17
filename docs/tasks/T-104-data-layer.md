# T-104. Данные: Room, DataStore, репозитории

- **Этап:** M1
- **Модули:** `:core:data`
- **Зависит от:** T-101
- **Размер:** M
- **Прочитать:** [data.md](../architecture/data.md), [overview.md](../architecture/overview.md#домен-интерфейсы-данных), [game-rules.md](../product/game-rules.md) разделы 4 и 8

## Зачем

Путь пользователя длится год. Данные должны пережить обновления приложения, миграции схемы и переезд на новый телефон. Ошибка тут стоит пользователю всего прогресса.

## Что сделать

1. **Room:** `LongwalkDatabase` версии 1, сущности `journey`, `step_bucket`, `waypoint_visit` ровно по data.md, DAO, конвертеры `Instant ↔ Long` (epoch seconds) и `LocalDate ↔ String` (ISO). Экспорт схемы в `core/data/schemas/`, файл схемы в репозитории.
2. **`RoomJourneyRepository`** (реализует `JourneyRepository`):
   - `start(...)`: в транзакции проверяет, что нет `ACTIVE`, иначе `IllegalStateException`; `startedAt` = начало сегодняшнего дня по `clock.zone`; якорь плана = `clock.instant()` (момент старта) и 0 шагов; `originalDeadline = deadline`;
   - `observeCurrent()`: активный путь, иначе последний `COMPLETED`, иначе `null` (`ABANDONED` не показываем);
   - остальные методы из интерфейса; `addVisits` игнорирует уже существующие (`INSERT OR IGNORE`).
3. **`RoomStepRepository`** (реализует `StepRepository`):
   - `replaceRange`: в одной транзакции удаляет корзины с `hour_start` в `[from, to)` и вставляет новые;
   - `add`: прибавляет к существующей корзине или создаёт её (upsert);
   - `sum` и `observeSum`: `SELECT SUM`, конец диапазона не включается, пустой диапазон 0;
   - отрицательные шаги отвергаются (`IllegalArgumentException`).
4. **`DataStoreSettingsRepository`** (реализует `SettingsRepository`): ключи и значения по умолчанию из data.md, `update` атомарный.
5. **`SystemTicker`** (реализует `Ticker`): эмитит текущее время из `Clock` сразу и потом в начале каждой минуты.
6. **Hilt-модуль `DataModule`:** база (синглтон), DAO, репозитории, DataStore, тикер.
7. Заготовка для миграций: `MigrationTestHelper` настроен, тест открывает базу версии 1 из экспортированной схемы.

## Контракты

- Реализует `JourneyRepository`, `StepRepository`, `SettingsRepository`, `Ticker` из T-101.
- Схема базы из data.md. Любое отличие: правка data.md в этом PR.

## Чего не делать

- Не считать план, статус, вехи (это домен).
- Не писать загрузку контента (T-103) и синхронизацию (T-202).
- Не включать `fallbackToDestructiveMigration`.

## Критерии приёмки

- Второй `start` при активном пути падает, при `ABANDONED` или `COMPLETED` проходит.
- `observeCurrent` после `complete` отдаёт завершённый путь, после `abandon` и без других путей отдаёт `null`.
- `replaceRange` атомарный: при исключении внутри транзакции старые корзины на месте.
- `add` дважды в один час складывает шаги.
- `sum(from, to)` не включает корзину с `hour_start == to`.
- Потоки эмитят новое значение после каждой записи.
- Файл схемы версии 1 закоммичен.

## Тесты

Robolectric, Room in-memory, DataStore во временном файле, `MutableClock`:

- `RoomJourneyRepositoryTest`: все методы, инвариант одного активного пути, `startedAt` в разных поясах.
- `RoomStepRepositoryTest`: `replaceRange`, `add`, суммы и границы, потоки через Turbine.
- `DataStoreSettingsRepositoryTest`: значения по умолчанию, обновление, переживает пересоздание.
- `SystemTickerTest`: первый тик сразу, следующие на границе минуты.
- `DatabaseSchemaTest`: база открывается из экспортированной схемы.
