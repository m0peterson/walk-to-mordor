# T-002. CI и проверки качества

- **Этап:** M0
- **Модули:** корень, `.github/workflows`, `scripts`, `build-logic`
- **Зависит от:** T-001
- **Размер:** S
- **Прочитать:** [AGENTS.md](../../AGENTS.md) (раздел «Команды»)

## Зачем

Код пишут несколько агентов. CI это единственный способ гарантировать, что каждый PR собирается, проходит тесты и не тащит в приложение доступ в интернет.

## Что сделать

1. **Spotless + ktlint** через convention-плагин для всех модулей: Kotlin, Kotlin Gradle scripts, XML (форматирование), `*.md` не трогать.
2. **Android Lint:** `abortOnError = true`, `checkDependencies = true` в `:app`, `lint.xml` с разумными исключениями (без глушения ошибок оптом). Пустой baseline, если на скелете есть предупреждения.
3. **Roborazzi** подключён в `longwalk.android.library.compose`. Каталог эталонов `src/test/snapshots` (или стандартный для выбранной версии), коммитится.
4. **Скрипт** `scripts/check-no-internet.sh`: собирает release-вариант (без подписи), достаёт итоговый манифест (`apkanalyzer manifest permissions` или аналог) и падает, если в нём есть `android.permission.INTERNET`. Печатает полный список разрешений.
5. **GitHub Actions** `.github/workflows/ci.yml`:
   - триггеры: `pull_request` и `push` в `main`;
   - Ubuntu, JDK 17 (Temurin), `gradle/actions/setup-gradle` с кэшем;
   - Android SDK (через установленный на раннере или `scripts/setup-android-sdk.sh`);
   - шаги: `./gradlew spotlessCheck lintDebug testDebugUnitTest verifyRoborazziDebug assembleDebug`, затем `scripts/check-no-internet.sh`;
   - при падении выгружать артефакты: отчёты тестов, lint, диффы Roborazzi;
   - отменять предыдущий запуск на той же ветке (`concurrency`).
6. **Шаблоны** в `.github/` уже лежат (`pull_request_template.md`, `ISSUE_TEMPLATE/agent-task.md`), их не менять.

## Чего не делать

- Не добавлять Detekt, Dependabot, деплой, подпись: это позже и по решению владельца.
- Не запускать инструментальные тесты (эмулятора нет, ADR-012).

## Критерии приёмки

- PR с этой задачей проходит CI зелёным.
- Если в тестовой ветке добавить `<uses-permission android:name="android.permission.INTERNET"/>` в любой модуль, CI падает на шаге проверки разрешений (показать в PR ссылкой на прогон или выводом скрипта).
- Если испортить форматирование, падает `spotlessCheck`.
- Время прогона CI на скелете записано в PR.

## Тесты

Отдельных нет, задача про инфраструктуру.

## Ручная проверка

Владелец включает в настройках репозитория защиту ветки `main`: обязательный зелёный CI и PR перед мёржем.
