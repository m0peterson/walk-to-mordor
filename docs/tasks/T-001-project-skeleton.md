# T-001. Скелет проекта и модули

- **Этап:** M0
- **Модули:** все (создаются пустыми), `build-logic`, `scripts`
- **Зависит от:** решение владельца D-2 (`applicationId`)
- **Размер:** M
- **Прочитать:** [AGENTS.md](../../AGENTS.md), [overview.md](../architecture/overview.md)

## Зачем

Дальше задачи идут параллельно. Если каждая будет сама добавлять модули, библиотеки в каталог и маршруты в навигацию, они начнут конфликтовать в одних и тех же файлах. Поэтому скелет заводит всё общее один раз.

## Что сделать

1. **Gradle.** Kotlin DSL, Gradle wrapper (последняя стабильная версия), `gradle.properties` с включёнными configuration cache, parallel, `android.nonTransitiveRClass=true`, разумным `-Xmx`.
2. **Каталог версий** `gradle/libs.versions.toml` со **всеми** библиотеками из таблицы «Стек» в overview.md, даже если они понадобятся позже: Compose BOM, Material 3, activity-compose, lifecycle (runtime-compose, viewmodel-compose, process), navigation-compose, kotlinx-serialization-json, kotlinx-coroutines (core, android, test), Hilt (android, compiler, navigation-compose, work, testing), Room (runtime, ktx, compiler, testing), DataStore Preferences, WorkManager (runtime-ktx, testing), Health Connect (`connect-client` и `connect-testing`, если артефакт существует в выбранной версии), Coil 3 (compose, без network), MapCompose, core-splashscreen, AboutLibraries (плагин и compose-ui), JUnit 4, Turbine, Robolectric, Roborazzi (плагин, compose, junit-rule), androidx.test (core, ext-junit), compose ui-test-junit4 и ui-test-manifest, Spotless. Последние стабильные версии, проверить, что всё резолвится.
3. **Convention-плагины** в `build-logic/` (included build):
   - `longwalk.jvm.library`: Kotlin JVM, toolchain 17, JUnit;
   - `longwalk.android.library`: compileSdk, minSdk 28, toolchain 17, Robolectric для unit-тестов (`isIncludeAndroidResources = true`);
   - `longwalk.android.library.compose`: плюс Compose и Roborazzi;
   - `longwalk.android.feature`: library.compose + Hilt + зависимости на `:core:domain`, `:core:designsystem`, `:content`, lifecycle, navigation, `:core:testing` в `testImplementation`;
   - `longwalk.android.application`: application + Compose + Hilt;
   - `longwalk.hilt`, `longwalk.android.room` (KSP, экспорт схем в `schemas/`).
4. **Модули** из overview.md, все пустые, но собираются: `:app`, `:core:domain`, `:core:data`, `:core:steps`, `:core:notifications`, `:core:designsystem`, `:core:testing`, `:content`, `:feature:onboarding`, `:feature:journey`, `:feature:journal`, `:feature:finale`, `:feature:settings`. Зависимости между ними ровно как на схеме в overview.md. `:core:domain` на `kotlin("jvm")`.
5. **Пакеты:** `io.github.m0peterson.longwalk.<модуль>` (или другой корень, если владелец решил D-2 иначе). `applicationId` тот же корень.
6. **Домен:** только квалификаторы `@IoDispatcher`, `@DefaultDispatcher` (`javax.inject.Qualifier`) и пустой `GameRules`-объект для констант.
7. **App:**
   - `LongwalkApplication` с `@HiltAndroidApp`;
   - `MainActivity` с `@AndroidEntryPoint`, edge-to-edge, SplashScreen API;
   - Hilt-модуль `TimeModule`: `Clock.systemDefaultZone()`, диспетчеры;
   - `NavHost` со **всеми** маршрутами из overview.md (`OnboardingRoute`, `HomeRoute`, `WaypointRoute(id, fromQueue)`, `JournalRoute`, `SettingsRoute(section)`, `VictoryRoute`, `DeadlineMissedRoute`), типобезопасные (`@Serializable`). Каждый маршрут ведёт на заглушку из своего фича-модуля: `NavGraphBuilder.onboardingScreen(...)` и т.д., заглушка показывает имя маршрута. Старт с `OnboardingRoute`;
   - `app_name` берётся из `:content` (`res/values/strings.xml` и `values-ru` в `:content`, значения из решения D-3).
8. **Content:** пустой `ContentPaths`, пустые папки `assets/route`, `assets/texts`, `assets/map`, `assets/illustrations` (с `.gitkeep`).
9. **Скрипт** `scripts/setup-android-sdk.sh`: если `ANDROID_HOME` не задан, ставит command-line tools в `$HOME/android-sdk`, ставит platform, build-tools и platform-tools нужных версий, принимает лицензии, пишет `local.properties`. Идемпотентный, повторный запуск ничего не ломает.
10. `.gitignore` для Android/Gradle/IDE, `.editorconfig` (ktlint official, 4 пробела, `max_line_length = 120`).
11. В корневой `README.md` дописать раздел «Сборка» с командами.

## Контракты

- Отдаёт всем: модули, каталог, convention-плагины, граф навигации с заглушками, `Clock` и диспетчеры через Hilt.
- Фича-задачи потом заменяют только свою заглушку `xxxScreen(...)` и не трогают `NavHost` в `:app`, кроме передачи лямбд навигации.

## Чего не делать

- Никакой логики, экранов, темы (это T-003), CI (T-002).
- Не добавлять модули `:widget` и `:feature:achievements`: их создадут задачи v1.1.
- Не добавлять разрешений в манифест.

## Критерии приёмки

- На чистом Linux с JDK 17: `scripts/setup-android-sdk.sh && ./gradlew assembleDebug testDebugUnitTest` проходит.
- APK ставится и открывает заглушку онбординга с названием маршрута.
- `./gradlew :core:domain:dependencies` не показывает ни одной Android-зависимости.
- Граф зависимостей модулей совпадает со схемой в overview.md (в PR приложен вывод или скриншот графа).
- В `libs.versions.toml` есть все библиотеки из пункта 2, и все версии резолвятся.
- Итоговый манифест debug-сборки не содержит `android.permission.INTERNET`.

## Тесты

- Один тривиальный тест в `:core:domain` (проверка, что тесты запускаются).
- Один Robolectric-тест в `:app`: `MainActivity` стартует.

## Ручная проверка

1. Поставить APK на телефон, открыть: видна заглушка, приложение не падает.
2. Повернуть экран: не падает.
