# Текстура пергамента

- **Где в приложении:** фон экранов и карточек, поля вокруг карты, когда карту отдалили до краёв.
- **Формат:** 1:1, бесшовная (Midjourney `--ar 1:1 --tile`)
- **Экспорт:** 1024×1024, WebP ~90%, повторяется плиткой
- **Приоритет:** A

## Prompt

```text
Seamless tileable texture of aged cream parchment paper, top-down flat scan. Subtle fibers and fine grain, very soft uneven light-brown stains, low contrast, flat even lighting. Uniform across the whole image so that it repeats without visible seams. No vignette, no darker edges, no folds, no tears, no burnt edges, no text, no drawings, no frame.
```

## Negative prompt

```text
vignette, dark edges, burnt edges, folds, creases, tears, wrinkles, strong stains, text, writing, drawings, frame, border, shadows, perspective, 3d, high contrast
```

## Заметки

- Бесшовность из коробки умеет Midjourney (`--tile`, но поддержка параметра зависит от версии модели, проверьте в документации). В остальных генераторах швы придётся убрать руками: в Photoshop Filter → Other → Offset на половину размера, потом замазать шов штампом или Generative Fill. В Krita то же самое через режим Wrap Around.
- Текстура должна быть почти незаметной. На ней лежит текст, контраст текста важнее красоты бумаги.
