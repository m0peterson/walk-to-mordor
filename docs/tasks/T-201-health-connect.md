# T-201. Источник шагов Health Connect

- **Этап:** M2
- **Модули:** `:core:steps`, `:app` (активность объяснения разрешений), строка в `:core:designsystem`
- **Зависит от:** T-101
- **Размер:** M
- **Прочитать:** [steps.md](../architecture/steps.md) целиком, ADR-004 и ADR-008 в [decisions.md](../architecture/decisions.md), официальная документация Health Connect для версии библиотеки из каталога

## Зачем

Health Connect основной источник шагов. Если интеграция сделана с ошибкой (не тот манифест, фильтр по источнику данных, корзины не по локальным часам), шаги будут теряться или удваиваться у всех пользователей.

## Что сделать

1. **`HealthConnectGateway`**: тонкая обёртка над `HealthConnectClient` (статус SDK, выданные разрешения, доступность фичи фонового чтения, `aggregateGroupByDuration`). Нужна для тестов. Если в каталоге есть `connect-testing` с `FakeHealthConnectClient`, обёртка может быть тривиальной, а тесты используют фейк библиотеки.
2. **`HealthConnectStepSource`** (реализует `StepSource`, `type = HEALTH_CONNECT`):
   - `status()`: доступность по `getSdkStatus`, `READ_STEPS` выдано или нет, фоновое чтение (фича доступна и разрешение выдано);
   - `collect(from, to)`: агрегаты `StepsRecord.COUNT_TOTAL` по часу, куски по 7 дней, **без фильтра по `DataOrigin`**, результат `Absolute`. Часы без данных в результат не попадают. Ошибки переводятся в `Failed` по таблице из steps.md.
3. **`StepAccessImpl`** (реализует `StepAccess` из домена) в `:core:steps`:
   - `status(HEALTH_CONNECT)` через источник; `status(SENSOR)` пока возвращает `UNSUPPORTED` (заполнит T-203);
   - `recommendedSource()`: Health Connect, если `AVAILABLE` или `NEEDS_INSTALL`/`NEEDS_UPDATE` на Android 9-13; иначе датчик, если он доступен; иначе `null`;
   - `deviceCountsStepsItself()`: Android 14+ и SDK Extension ≥ 20;
   - `activate(type)`: пишет `step_source` и `source_since = now` в настройки и зовёт `SyncScheduler.ensureScheduled()`;
   - интерфейс `SyncScheduler` в `:core:steps`: `ensureScheduled()` (периодическая работа под текущий источник) и `requestOnce(trigger: SyncTrigger)` (разовая синхронизация в фоне). В этой задаче пустая реализация в Hilt-модуле, T-202 заменит биндинг настоящей.
4. **Пакет `access.ui`** (единственное, что видят онбординг и настройки):
   - `rememberHealthConnectPermissionLauncher(onResult: (granted: Set<String>) -> Unit)` на `PermissionController.createRequestPermissionResultContract()`;
   - набор разрешений для запроса: `READ_STEPS` всегда и `READ_HEALTH_DATA_IN_BACKGROUND`, если фича доступна;
   - `healthConnectInstallIntent()` (Google Play с `com.google.android.apps.healthdata`), `healthConnectSettingsIntent()`.
5. **Манифест `:core:steps`:** разрешения и `<queries>` из steps.md.
6. **`:app`:** `PermissionsRationaleActivity` (Compose, тема из дизайн-системы) с текстом «зачем нам шаги и что мы с ними делаем» (коротко: только шаги, только на телефоне, никуда не отправляются). Текст строкой `privacy_policy_text` в `:core:designsystem` (ru и en), потому что его же показывают настройки (T-405). Обе регистрации в манифесте: intent-filter `androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE` и `activity-alias` для `VIEW_PERMISSION_USAGE` с категорией `HEALTH_PERMISSIONS`. Точные имена сверить с документацией. Текст потом выровняют с политикой конфиденциальности в T-503.
7. **Hilt-модуль `HealthConnectModule`:** источник в мультибиндинг `Map<StepSourceType, StepSource>` через `@IntoMap` с ключом `HEALTH_CONNECT`, `StepAccessImpl` как `StepAccess`, пустой `SyncScheduler`.

## Контракты

- Отдаёт `StepSource` для Health Connect, `StepAccess`, `access.ui`, мультибиндинг источников, интерфейс `SyncScheduler`.
- Берёт модели и интерфейсы из T-101, `SettingsRepository` (через интерфейс).

## Чего не делать

- Не писать движок синхронизации, воркер и запись в базу (T-202).
- Не писать датчик (T-203).
- Не читать сырые `StepsRecord` списком: только агрегаты.
- Не запрашивать никаких разрешений Health Connect, кроме двух перечисленных.

## Критерии приёмки

- `collect` при начале диапазона на границе локального часа отдаёт корзины, выровненные по локальным часам (тест с поясом +05:30).
- Диапазон 20 дней читается тремя запросами.
- Отсутствие `READ_STEPS` даёт `Failed(PERMISSION_LOST)`, а не исключение.
- В манифесте `:app` после сборки есть и intent-filter, и alias (проверка в тесте на итоговый манифест или в PR выводом `apkanalyzer`).
- В коде нет фильтрации по `DataOrigin` (ревью).

## Тесты

- `HealthConnectStepSourceTest` на фейке: корзины, куски по 7 дней, пустые часы, все ветки ошибок.
- `StepAccessImplTest`: `recommendedSource` для всех сочетаний доступности, `activate` пишет настройки.
- Robolectric-тест, что `PermissionsRationaleActivity` открывается.

## Ручная проверка

1. Android 14+: поставить сборку, вызвать запрос разрешений (временная кнопка на заглушке онбординга допустима, убрать в T-401), выдать оба, проверить в настройках Health Connect, что приложение в списке и экран «как используются данные» открывается из системных настроек.
2. Android 9-13 без Health Connect: интент установки ведёт на страницу в Google Play.
