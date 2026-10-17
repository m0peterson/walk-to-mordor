# T-101. Домен: модели, интерфейсы, геометрия маршрута

- **Этап:** M1
- **Модули:** `:core:domain`, `:core:testing`
- **Зависит от:** T-001
- **Размер:** M
- **Прочитать:** [overview.md](../architecture/overview.md#контракты), [data.md](../architecture/data.md), [game-rules.md](../product/game-rules.md) разделы 1 и 3

## Зачем

Это общий язык всех остальных задач. После неё данные, шаги, карта и экраны пишутся параллельно, потому что все договорились о моделях и интерфейсах.

## Что сделать

1. **Модели** из раздела «Контракты» overview.md: `MapPoint`, `WaypointKind`, `Waypoint`, `Route`, `Journey`, `JourneyState`, `StepBucket`, `WaypointVisit`, `StepSourceType`, `JourneyPhase`, `PaceStatus`, `Pose`, `JourneyProgress`, `RouteTexts`, `WaypointText`, `ScreenText`, `AppSettings` (поля из data.md), `Availability`, `StepFailure`, `StepSourceStatus`, `SyncTrigger`, `SyncResult`.
2. **Интерфейсы:** `ContentRepository`, `JourneyRepository`, `StepRepository`, `SettingsRepository`, `JourneyEventNotifier`, `Ticker`, `StepAccess`, `StepSync`.
3. **Константы** в `GameRules`: все из раздела 10 game-rules.md.
4. **`RouteGeometry(route)`:**
   - длина каждой ломаной `legs[i]` в «пиксельном» пространстве с учётом `mapAspect`: `dx × mapAspect` и `dy`;
   - `pointAt(routeKm): MapPoint`: находит участок между вехами `i` и `i+1`, долю `f = (routeKm − km[i]) / (km[i+1] − km[i])` и точку на ломаной на расстоянии `f × длина ломаной`. То есть внутри участка движение равномерно по длине линии на карте, а не по прямой между вехами;
   - `pathUntil(routeKm): List<MapPoint>` и `pathFrom(routeKm): List<MapPoint>`: пройденная и оставшаяся части линии, точка `pointAt` в обеих как граница;
   - `routeKm` меньше 0 или больше `lengthKm` зажимается.
5. **`RouteValidator.validate(route): List<RouteError>`:** правила 1-6 из раздела «Проверки» в data.md. `RouteError` с понятным сообщением и id вехи. Правило 7 (тексты) делает T-103.
6. **`:core:testing`:**
   - `MutableClock` (реализует `java.time.Clock`, можно двигать время и менять пояс);
   - `TestRoutes.small()`: 5 вех с нейтральными id (`start`, `a`, `b`, `c`, `finish`), из них 3 больших и 2 малых, `lengthKm = 100`, ломаные с промежуточными точками, `mapAspect = 1.0`; и `TestRoutes.wide()` с `mapAspect = 2.0` для тестов геометрии;
   - фейки всех интерфейсов из пункта 2 на `MutableStateFlow`, с методами для подготовки состояния в тестах (`FakeStepRepository.setBuckets(...)`, `FakeStepAccess.setStatus(...)`, `FakeTicker.tick(instant)` и т.п.);
   - `MainDispatcherRule` для тестов ViewModel.

## Контракты

Отдаёт всё перечисленное. Сигнатуры из overview.md можно уточнять (имена параметров, дополнительные поля по необходимости), но тогда overview.md правится в этом же PR.

## Чего не делать

- Никакой логики плана, статуса и вех (это T-102).
- Никаких реализаций репозиториев (T-104) и загрузки JSON (T-103).
- Никаких Android-зависимостей в `:core:domain`.

## Критерии приёмки

- `:core:domain` собирается как чистый JVM-модуль.
- `pointAt` на границах участков возвращает точно координаты вех.
- Для ломаной из трёх точек с длинами сегментов 1:3 точка на середине участка лежит на втором сегменте.
- `pathUntil(x) + pathFrom(x)` без дублирующей точки дают исходную линию.
- Валидатор находит каждую из ошибок правил 1-6 на испорченных копиях тестового маршрута.
- Фейки покрывают все интерфейсы, и каждый фейк используется хотя бы в одном тесте.

## Тесты

- `RouteGeometryTest`: границы участков, середина, несколько сегментов внутри участка, `mapAspect = 2.0`, зажим за пределами, `pathUntil`/`pathFrom` на 0, в середине, на `lengthKm`.
- `RouteValidatorTest`: по тесту на каждое правило плюс корректный маршрут без ошибок.
- `MutableClockTest`: сдвиг времени, смена пояса.
