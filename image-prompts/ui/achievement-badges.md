# Значки достижений (лист 4×3)

- **Где в приложении:** экран достижений, всплывашка при получении.
- **Формат:** 4:3, один лист с двенадцатью значками (Midjourney `--ar 4:3`)
- **Экспорт:** каждый значок отдельно, 384×384 WebP с прозрачностью
- **Приоритет:** B

Все значки сделаны как красные сургучные печати с выдавленным символом: это единый стиль и хорошая читаемость. Закрытые (ещё не полученные) значки приложение показывает обесцвеченными, отдельные картинки для них не нужны. Цифры (100 км, 7 дней и т.д.) пишет приложение под значком.

| # | Символ | Достижение (пример) |
|---|---|---|
| 1 | босой след | первые шаги |
| 2 | восходящее солнце | дневная цель выполнена |
| 3 | фонарь | 7 дней подряд |
| 4 | факел | 30 дней подряд |
| 5 | петух | прогулка до 8 утра |
| 6 | месяц и звёзды | прогулка после 22:00 |
| 7 | посох | 100 км |
| 8 | рюкзак | 500 км |
| 9 | горная вершина | 1000 км |
| 10 | песочные часы | половина пути |
| 11 | моток верёвки | вернулся в график после отставания |
| 12 | чашка и пирог | «второй завтрак»: дневная норма до обеда |

## Prompt

```text
A sheet of twelve round red wax seal badges arranged in a neat grid of 4 columns and 3 rows on a plain flat white background, evenly spaced with empty space between them, all the same size, shape and style. Each seal is deep crimson wax with slightly irregular dripped edges and a different simple emblem pressed into its center in relief. Row one, left to right: a single bare footprint; a rising sun with rays; a lit lantern; a burning torch. Row two: a rooster; a crescent moon with three stars; a wooden walking stick; a travel backpack. Row three: a mountain peak; an hourglass; a coiled rope; a teacup with a slice of cake. Bold simple emblems, clear silhouettes, readable when shown at 64 pixels, soft highlights on the wax. Storybook ink and watercolor flavor. No text, no letters, no numbers, no ribbons, no frame, no grid lines, nothing on the background.
```

## Negative prompt

```text
text, letters, numbers, ribbons, banners, frame, grid lines, background texture, drop shadow, paper, envelope, photorealistic, glossy plastic, metallic, 3d render, tiny details, inconsistent sizes, blurry, watermark
```

## Заметки

- Двенадцать разных символов за один раз генератор часто путает: повторяет один символ или меняет порядок. Не страшно: порядок можно поменять при нарезке, а неудачные значки перегенерировать отдельно тем же промптом, оставив в нём одну печать.
