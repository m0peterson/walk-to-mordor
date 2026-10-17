# Маркеры на карте (лист 2×2)

- **Где в приложении:** поверх карты. Маркер игрока, следующая цель, пройденная веха, непройденная веха.
- **Формат:** 1:1, один лист с четырьмя значками (Midjourney `--ar 1:1`)
- **Экспорт:** каждый значок отдельно, 256×256 PNG или WebP с прозрачностью
- **Приоритет:** A

Набор генерируется одним листом, чтобы все четыре значка вышли в одном стиле. Потом лист режется на четыре части, фон удаляется.

| Ячейка | Значок | Когда показывать |
|---|---|---|
| Верх слева | жетон с хоббитом | текущее положение игрока |
| Верх справа | флажок | ближайшая непройденная веха |
| Низ слева | красная сургучная печать | пройденная веха |
| Низ справа | чернильный кружок | вехи впереди |

## Prompt

```text
A sheet of four small map marker icons arranged in a 2 by 2 grid on a plain flat white background, each icon centered in its own quarter with generous empty space around it, all four drawn in the same style and at the same size. Top left: a round token like a board game piece, a thin brass rim around a cream parchment disc, on the disc a dark sepia silhouette of a small hobbit walking to the right with a walking stick and a backpack. Top right: a small pennant flag on a thin wooden pole, the pennant deep red, the pole planted in a tiny mound of earth. Bottom left: a small round red wax seal with a simple embossed eight-pointed star. Bottom right: a small hollow circle hand-drawn in sepia ink with a dot in the middle. Bold simple shapes, clean thick sepia outlines, minimal detail, readable when shown at 48 pixels. Ink and watercolor storybook style on the icons only, the background stays pure flat white. No text, no letters, no shadows on the background, no frame, no grid lines.
```

## Negative prompt

```text
text, letters, numbers, grid lines, frame, border, background texture, drop shadow, gradient background, photorealistic, 3d render, glossy, metallic shine, tiny details, blurry, watermark
```

## Заметки

- Проверяйте читаемость сразу: уменьшите лист до 96×96 пикселей. Если силуэт хоббита превратился в кляксу, просите `simpler silhouette, fewer details`.
- Чернильный кружок и флажок при желании проще нарисовать вектором в коде. Генерировать их стоит, только если векторные выбиваются из стиля.
- Если генератор умеет прозрачный фон (у ChatGPT есть `transparent background`), просите его вместо белого.
