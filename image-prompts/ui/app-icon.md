# Иконка приложения

- **Где в приложении:** иконка в лаунчере, сплэш-экран Android 12+, иконка в Google Play.
- **Формат:** 1:1 (Midjourney `--ar 1:1`)
- **Экспорт:** мастер 1024×1024 PNG. Для Google Play 512×512 PNG. Для лаунчера adaptive icon через Android Studio: File → New → Image Asset, слой Foreground = эта картинка, Background = цвет пергамента (например `#EFE3C8`).
- **Приоритет:** A

Google сам скругляет углы и режет иконку маской (круг, сквирл и т.д.), поэтому картинка квадратная, на весь кадр, без своих скруглений. Всё важное в центральных 60%: края маска отрежет.

## Prompt (основной вариант: путь к вулкану)

```text
A mobile app icon, square, full bleed. Warm cream parchment background with a subtle paper texture. In the center, a bold simple emblem: a dark volcanic mountain with a glowing red-orange peak, and a winding golden path that starts at the bottom left and curves up toward the mountain. Flat, clean, bold shapes, thick sepia outlines, very few details, strong silhouette, readable at 48 pixels. All important elements inside the central 60 percent of the square, plain parchment around them. Storybook ink and watercolor flavor, but simplified like a modern app icon. No text, no letters, no border, no rounded corners, no drop shadow, no gloss.
```

## Prompt (запасной вариант: след)

```text
A mobile app icon, square, full bleed. Warm cream parchment background with a subtle paper texture. In the center, one large bare hobbit footprint printed in dark sepia ink, and inside the heel of the footprint a small silhouette of a volcano with a glowing red-orange peak. Flat, clean, bold shapes, thick outlines, very few details, strong silhouette, readable at 48 pixels. All important elements inside the central 60 percent of the square. No text, no letters, no border, no rounded corners, no drop shadow, no gloss.
```

## Negative prompt

```text
text, letters, numbers, border, frame, rounded corners, drop shadow, gloss, 3d, photorealistic, gradient mesh, tiny details, busy background, multiple objects, watermark, ring with inscription
```

## Что не генерировать

- **Маленькую иконку уведомлений.** Android требует белый силуэт на прозрачном фоне. Её проще нарисовать вектором (SVG → Vector Asset) по контуру основной иконки.
- **Монохромную иконку для тем Android 13+.** Тоже одноцветный векторный силуэт.
- **Отдельный сплэш.** На Android 12+ система сама показывает иконку на цветном фоне (SplashScreen API). Отдельная картинка не нужна.

## Заметки

- Проверка: уменьшите результат до 48×48 и посмотрите на телефоне рядом с другими иконками. Если вулкан и тропинка не читаются, упрощайте.
