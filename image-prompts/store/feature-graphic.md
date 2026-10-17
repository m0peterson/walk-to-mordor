# Обложка для Google Play (feature graphic)

- **Где:** страница приложения в Google Play, верхний баннер.
- **Формат:** генерировать 2:1 (Midjourney `--ar 2:1`), потом обрезать
- **Экспорт:** ровно 1024×500, JPEG или 24-битный PNG без прозрачности (требование Google Play)
- **Приоритет:** A, без неё приложение не опубликовать
- **Референс:** [лист персонажей](../characters/character-sheet.md) и эталон стиля

Название приложения добавляйте в редакторе, не в генераторе. Google может наложить поверх баннера кнопку воспроизведения видео, поэтому ничего важного в самом центре.

## Prompt

```text
A wide panoramic storybook landscape that tells the whole journey from left to right. On the far left, a green hill with a round green door in it under a warm golden morning sky. A long winding road runs from that door across the whole image, through fields, woods, a river and snowy mountains, toward the far right, where a distant dark land with a red-glowing volcano sits under smoky clouds. Near the left, on the road, two tiny hobbit travelers seen from behind, one with a green cloak and a walking stick, one with a huge backpack. A smooth gradient of mood from warm and bright on the left to dark and ember-red on the right. A calm, less detailed area of sky in the upper middle for a title to be added later. Wide 2:1 composition. Hand-painted storybook illustration, ink and watercolor on warm cream paper, fine sepia pen linework, loose transparent washes, painterly and atmospheric, not photorealistic, not a 3D render, not a movie still. No text, no letters, no signature, no watermark, no frame or border.
```

## Negative prompt

```text
text, letters, title, logo, signature, watermark, frame, border, close-up faces, actor likeness, movie still, photorealistic, 3d render, CGI, anime, blurry, oversaturated, neon colors
```
