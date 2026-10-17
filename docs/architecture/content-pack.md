# Контент-пакет

Модуль `:content` хранит всё, что относится к Средиземью: маршрут, тексты, карту, иллюстрации, значки, название приложения. Остальной код про Средиземье не знает. Зачем это:

1. **Права.** Если решим публиковать «по мотивам» (вариант C в PRD), меняется только этот модуль.
2. **Параллельная работа.** Владелец генерирует картинки, агенты пишут код. Пока картинок нет, приложение работает на заглушках, и никто никого не ждёт.
3. **Проверяемость.** Данные маршрута проверяются тестами, а не глазами.

## Структура

```
content/
  LICENSE                          отдельная лицензия на арт и тексты (решение D-4)
  src/main/
    assets/
      route/route.json             маршрут (формат в data.md)
      texts/ru.json, en.json       названия и истории вех
      map/meta.json                размеры карты
      map/tiles/{level}/{row}/{col}.webp
      illustrations/waypoints/01-bag-end.webp ... 23-mount-doom.webp
      illustrations/poses/on-track.webp, ahead.webp, behind.webp, rest.webp
      illustrations/screens/onboarding-1.webp, onboarding-2.webp, onboarding-3.webp, victory.webp, deadline-missed.webp
      markers/player.webp, next.webp, passed.webp, ahead.webp
      badges/                      v1.1
    res/values/strings.xml          app_name (английский)
    res/values-ru/strings.xml       app_name (русский)
    java/.../content/ContentPaths.kt пути к экранным картинкам, без логики
```

## Контракт на картинки

Картинки складываются в `assets` (а не в `res/drawable-nodpi`, как было написано в `image-prompts/README.md` раньше): пути к ним лежат в данных маршрута, и отсутствующая картинка не ломает сборку.

| Файл в `assets/` | Откуда | Размер | Приоритет | Если файла нет |
|---|---|---|---|---|
| `map/tiles/...` | `image-prompts/map/world-map.md` | мастер от 6144×6144 → тайлы | A | заглушка из схемы, уже в репозитории после T-301 |
| `markers/player.webp` | `map/map-markers.md`, верх слева | 256×256, альфа | A | векторный круг |
| `markers/next.webp` | там же, верх справа | 256×256, альфа | A | векторный флажок |
| `markers/passed.webp` | там же, низ слева | 256×256, альфа | A | векторная печать |
| `markers/ahead.webp` | там же, низ справа | 256×256, альфа | A | векторный кружок |
| `illustrations/poses/*.webp` | `characters/status-poses.md` | 768×768, альфа | A | значок статуса из Material Symbols |
| `illustrations/screens/onboarding-1..3.webp` | `screens/onboarding-*.md` | 1440×1800 | A | пергамент с заголовком |
| `illustrations/screens/victory.webp` | `screens/victory.md` | 1440×2560 | A | пергамент |
| `illustrations/screens/deadline-missed.webp` | `screens/deadline-missed.md` | 1440×2560 | B | пергамент |
| `illustrations/waypoints/NN-<id>.webp` | `locations/NN-*.md` | 1800×1200 | A или B, см. README промптов | пергамент с номером вехи |
| `badges/NN-<id>.webp` | `ui/achievement-badges.md` | 384×384, альфа | B (v1.1) | значок из Material Symbols |

Вне `:content`:

| Файл | Где | Откуда |
|---|---|---|
| Текстура пергамента | `core/designsystem/src/main/res/drawable-nodpi/parchment_tile.webp` | `ui/parchment-texture.md`, это общий стиль, не Средиземье |
| Иконка приложения | `app/src/main/res/mipmap-*` через Image Asset в Android Studio | `ui/app-icon.md` |
| Иконка уведомлений | `core/notifications/src/main/res/drawable/ic_stat_*.xml`, вектор | рисуется по контуру иконки |

Правила:

- WebP, иллюстрации с потерями ~85%, всё с прозрачностью тоже WebP с альфой.
- Имена файлов строчными латинскими буквами, номер вехи двумя цифрами.
- Пустое место под отсутствующую картинку: дизайн-система (`Illustration` в T-003) сама рисует запасной вариант. Фичи не проверяют наличие файла.
- Новый файл картинки: только копирование в `assets`, код не трогается.

## Правила текстов

Тексты пишут агенты (T-103), владелец вычитывает.

- **Своими словами.** Никаких цитат из книг Толкина, ни целиком, ни фразами. Стихи и песни не цитировать и не перелагать.
- **По книге, не по фильмам.** Не описывать сцены, придуманные для фильмов, и не ссылаться на актёров.
- **Коротко.** Большая веха: `story` 2-4 предложения, до 400 знаков. Малая: 1-2 предложения, до 200 знаков.
- **Без спойлеров вперёд.** Текст вехи говорит о том, что случилось здесь, и не рассказывает, что будет дальше.
- **Тон.** Тёплый, немного книжный, без пафоса. Можно шутить про босые ноги и второй завтрак.
- **Имена персонажей.** Можно, это нужно для истории. Но в коде вне `:content` их нет.
- **Русские названия.** Как в таблице `image-prompts/README.md` и в [game-rules.md](../product/game-rules.md). Для малых вех самый распространённый вариант перевода.
- `season`: из поля «Сезон и время по книге» в файлах `image-prompts/locations/`.
- Английский текст это не перевод русского слово в слово, а такой же короткий текст на английском.

## Как поменять мир

Если владелец выберет вариант C («по мотивам»):

1. Новые названия и истории в `texts/*.json`, новые `id` в `route.json`.
2. Новая карта по своей схеме → `tools/map-tiles/make_tiles.py`.
3. Новые координаты через `tools/route-editor/`.
4. Новые иллюстрации по тем же размерам и именам.
5. Новое `app_name` в `res/values*/strings.xml`.

Код и тесты кода не меняются. Тесты контента (T-103) проверят новый маршрут.
