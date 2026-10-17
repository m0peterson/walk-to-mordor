# Лист персонажей (референс)

- **Где в приложении:** нигде напрямую. Это референс, чтобы хоббиты выглядели одинаково на всех картинках, где они есть: позы статуса, онбординг, победа, обложка для Google Play.
- **Формат:** 3:2 (Midjourney `--ar 3:2`)
- **Экспорт:** не нужен, хранить мастер
- **Приоритет:** A, делать сразу после эталона стиля

## Почему без имён

В промптах нет слов Frodo, Sam, hobbit from Lord of the Rings. Имена тянут генератор к лицам актёров из фильмов, а это и плохо выглядит, и юридически опасно для приложения в Google Play. Внешность описана по книге и своими словами.

| | Персонаж A (хранитель кольца) | Персонаж B (верный спутник) |
|---|---|---|
| Сложение | стройный | крепкий, плотнее |
| Волосы | тёмно-каштановые кудри | светло-русые кудри |
| Плащ | лесной зелёный | серо-коричневый |
| Отличие | посох, маленькая сумка | огромный рюкзак со сковородой и верёвкой |

## Prompt

```text
Character reference sheet of two hobbit companions on a plain warm white background. Top row: the first character shown three times, front view, side view and back view. Bottom row: the second character shown three times, front view, side view and back view. Full body, neutral standing pose, same scale, evenly spaced, nothing overlapping. Character one: a slender, young-looking hobbit about half the height of a man, a pale face with a gentle thoughtful expression, large grey-blue eyes, thick dark-brown curly hair, slightly pointed ears, large bare feet with curly brown hair on top; a cream linen shirt, a mustard-yellow waistcoat with brass buttons, brown knee-length breeches, a forest-green hooded travelling cloak fastened at the throat with a small green leaf-shaped brooch veined with silver, a slim wooden walking stick and a small leather satchel. Character two: a sturdier, stockier hobbit of the same height, a round cheerful freckled face, honest brown eyes, light sandy-brown curly hair, slightly pointed ears, large bare hairy feet; a white shirt with rolled sleeves, a brown waistcoat, dark green breeches, a grey-brown hooded cloak, and a huge overloaded backpack with a frying pan, a small cooking pot, a coil of grey rope and a blanket roll strapped to it. Stylized storybook proportions, slightly large heads, hands and feet, original character designs that do not resemble any actor or film character. Hand-painted ink and watercolor illustration, fine sepia pen linework, soft transparent washes, muted earthy palette. No text, no labels, no letters, no frame, no background scenery.
```

## Negative prompt

```text
text, labels, letters, numbers, frame, background scenery, actor likeness, celebrity face, movie still, film costume, photorealistic, 3d render, CGI, anime, manga, chibi, shoes, boots, modern clothes, deformed hands, extra limbs, extra fingers, inconsistent outfits
```

## Заметки

- Хоббиты босые, это важно для темы шагомера. Если генератор надевает ботинки, добавьте в начало `barefoot`.
- Если оба персонажа выходят на одно лицо, генерируйте каждого отдельно (тот же промпт, одна строка) и собирайте лист в редакторе.
- Дальше этот лист прикладывается как character reference: в Midjourney через omni-reference, в ChatGPT и Gemini просто картинкой с фразой `the two characters must look exactly like in the attached reference sheet`.
