# Позы статуса (лист 2×2)

- **Где в приложении:** экран прогресса и виджет. Показывает, как идут дела относительно выбранного срока.
- **Формат:** 1:1, один лист с четырьмя сценками (Midjourney `--ar 1:1`)
- **Экспорт:** каждая сценка отдельно, 768×768 WebP с прозрачностью
- **Приоритет:** A
- **Референс:** приложить [лист персонажей](character-sheet.md) и эталон стиля

| Ячейка | Состояние | Условие в приложении |
|---|---|---|
| Верх слева | идут в графике | темп совпадает с планом |
| Верх справа | опережают | впереди графика |
| Низ слева | отстают | позади графика |
| Низ справа | привал | сегодня ещё нет шагов |

## Prompt

```text
A sheet of four small vignette illustrations of the same two hobbit companions, arranged in a 2 by 2 grid on a plain flat white background, each vignette centered in its own quarter with empty space around it, no scenery except a small patch of grass and earth under their feet. The two characters must look exactly like in the attached reference sheet: the slender one with dark curly hair, a forest-green cloak and a walking stick; the sturdier one with light sandy curly hair, a grey-brown cloak and a huge backpack with a frying pan and a coil of rope. Both barefoot. Top left: the two walking side by side at a steady, confident pace to the right, calm and content. Top right: the two hurrying forward energetically, cloaks flying, big happy smiles, the sturdier one's pans bouncing. Bottom left: the two tired, the slender one sitting on a rock catching his breath, the sturdier one leaning on the walking stick and wiping his forehead, weary but not giving up. Bottom right: the two asleep by a tiny campfire under a blanket, a small kettle hanging over the fire, a thin curl of smoke. Same scale and style in all four. Hand-painted ink and watercolor storybook illustration, fine sepia linework, soft washes, muted earthy palette. No text, no letters, no frames, no grid lines, no background scenery.
```

## Negative prompt

```text
text, letters, numbers, frames, grid lines, background scenery, landscape, actor likeness, celebrity face, movie still, photorealistic, 3d render, anime, chibi, shoes, boots, different outfits between vignettes, deformed hands, extra limbs
```

## Заметки

- Самое частое место, где генератор ломает персонажей. Если одежда или волосы плывут между ячейками, генерируйте каждую сценку отдельно с тем же референсом.
- Состояния опережения и победы по эмоции близки. Не путайте: победа это отдельный экран `screens/victory.md`.
