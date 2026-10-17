# T-202. Движок синхронизации и воркер

- **Этап:** M2
- **Модули:** `:core:steps`, `:app`
- **Зависит от:** T-102, T-104, T-201
- **Размер:** L
- **Прочитать:** [steps.md](../architecture/steps.md) раздел «Движок синхронизации», [game-rules.md](../product/game-rules.md) разделы 6, 7, 9, [overview.md](../architecture/overview.md#фоновая-работа)

## Зачем

Связывает источники шагов, базу и правила игры. Отсюда приходят пройденные вехи, финиш и уведомления. Ошибки здесь выглядят как «шаги задваиваются» или «уведомление о вехе пришло дважды».

## Что сделать

1. **`StepSyncEngine`** (реализует `StepSync`): алгоритм из steps.md по шагам 1-10, источники из мультибиндинга `Map<StepSourceType, StepSource>`. Для расчёта вех: шаги пути до начала окна (`stepRepository.sum`), корзины окна, `WaypointDetector`. Для финиша: `journeyRepository.complete` с моментом из детектора.
2. **`StepSyncWorker`**: `@HiltWorker`, `CoroutineWorker`, вызывает `sync(PERIODIC)`. `Result.retry()` только при `StepFailure.TEMPORARY`, иначе `success()`.
3. **`WorkManagerSyncScheduler`** (реализует `SyncScheduler` из T-201, заменяет пустой биндинг): `ensureScheduled()` ставит уникальную периодическую работу, период 30 мин для Health Connect и 15 мин для датчика, `ExistingPeriodicWorkPolicy.UPDATE`, и отменяет её, когда активного пути нет; идемпотентный. `requestOnce(trigger)` ставит уникальную разовую работу с триггером во входных данных воркера.
4. **Ресиверы в `:core:steps`:** `TimeChangeReceiver` на `ACTION_TIMEZONE_CHANGED` и `ACTION_TIME_CHANGED` зовёт `requestOnce(TIME_CHANGED)`. `BootReceiver` делает T-203.
5. **`:app`:**
   - `LongwalkApplication` реализует `Configuration.Provider` с `HiltWorkerFactory`, стандартный инициализатор WorkManager убран из манифеста;
   - наблюдатель `ProcessLifecycleOwner`: `ON_START` → `sync(APP_RESUME)` (первый раз `APP_START`) и `ensureScheduled()`;
   - Hilt-модуль `NotifierModule` с `NoOpJourneyEventNotifier`. T-501 заменит его реальной реализацией.
6. **Hilt-модуль `SyncModule`:** движок как `StepSync`, планировщик.

## Контракты

- Отдаёт `StepSync` (используют главный экран для ручного обновления и онбординг), `SyncScheduler` для `StepAccess.activate`.
- Берёт `JourneyEventNotifier` (здесь заглушка), репозитории, детектор, источники.

## Чего не делать

- Не писать уведомления (T-501).
- Не писать датчик (T-203), но движок обязан уже обрабатывать `StepCollection.Delta`.
- Не использовать foreground service и expedited work.

## Критерии приёмки

Тесты на фейках из `:core:testing` (`MutableClock`, фейковые источники и репозитории):

- первая синхронизация читает от `startedAt`, следующая от `now − 48 ч`;
- после смены источника Health Connect не читает раньше `source_since` (округлённого вверх до часа) и не затирает корзины датчика;
- повторная синхронизация без новых шагов ничего не пишет и не зовёт уведомления;
- запоздавшие данные за вчера перезаписывают вчерашние корзины;
- `Delta` прибавляется к текущему часу;
- три вехи за одну синхронизацию: три визита, один вызов `onWaypointsReached` с тремя вехами по порядку;
- без источника или без разрешения: ошибка записана, Бэг-Энд всё равно попал в визиты;
- `PERIODIC` при Health Connect без фонового чтения не обращается к источнику;
- финиш: путь `COMPLETED`, `completedAt` по корзине, `onJourneyCompleted` один раз;
- две одновременные синхронизации не читают источник дважды;
- `last_sync_at` обновляется после успеха, `last_sync_error` очищается.

## Тесты

- `StepSyncEngineTest`: всё из критериев выше.
- `StepSyncWorkerTest` (`work-testing`, `TestListenableWorkerBuilder`): успех, retry на временной ошибке.
- `WorkManagerSyncSchedulerTest`: период зависит от источника, `ensureScheduled` не плодит работы.

## Ручная проверка

1. Health Connect с фоновым чтением: оставить телефон на 2 часа с прогулкой, не открывая приложение. В `adb shell dumpsys jobscheduler` или через лог видно, что воркер отрабатывал. После открытия шаги совпадают с Health Connect.
2. Сменить часовой пояс в настройках телефона: синхронизация запускается, «сегодня» считается в новом поясе.
