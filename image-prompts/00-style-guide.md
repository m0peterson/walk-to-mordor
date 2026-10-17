# Стиль

Один стиль на все картинки: книжная иллюстрация тушью и акварелью на тёплой бумаге. Почему он:

- он прощает ошибки генератора: акварельная размытость скрывает мелкие артефакты лучше, чем фотореализм;
- карта, маркеры, сцены и интерфейс выглядят как одна книга;
- он далеко от визуального стиля фильмов, а это и про узнаваемость, и про права (см. README).

## Стилевой блок для иллюстраций

Уже вставлен в конец каждого промпта. Здесь на случай, если захотите написать свой промпт для новой картинки.

```text
Hand-painted storybook illustration, ink and watercolor on warm cream paper, fine sepia pen linework, loose transparent watercolor washes, visible paper grain, soft hand-made edges, classic mid-century fantasy book illustration, painterly and atmospheric, not photorealistic, not a 3D render, not a movie still. No text, no letters, no signature, no watermark, no frame or border.
```

## Стилевой блок для карты

```text
Top-down hand-drawn map with small pictorial symbols: rows of little shaded peaks with sepia hatching, clusters of tiny round-topped trees, thin wavy blue-ink rivers. Ink and watercolor on aged cream parchment, fine sepia linework, soft washes. Flat top-down view, even lighting, uniform detail. No text, no labels, no route line, no compass rose, no legend, no frame or border.
```

## Общий negative prompt

```text
text, letters, words, runes, inscriptions, signature, watermark, logo, frame, border, photorealistic, photograph, 3d render, CGI, anime, manga, chibi, cartoon, movie still, film screenshot, actor likeness, celebrity face, modern objects, blurry, lowres, jpeg artifacts, oversaturated, neon colors, deformed hands, extra limbs
```

Куда его вставлять:

- **Stable Diffusion, Flux, Leonardo:** в поле negative prompt.
- **Midjourney:** коротко через `--no text, letters, watermark, frame, photo, 3d`. Длинный список в `--no` работает хуже.
- **ChatGPT, Gemini:** отдельного поля нет. Главные запреты уже вписаны в конец каждого промпта, блок Negative можно не вставлять.

## Палитра по ходу маршрута

Цвет и сезон меняются вместе с сюжетом книги. Это и есть ощущение пути: от тёплой осени Шира к чёрно-красному Мордору.

| Участок | Вехи | Сезон по книге | Палитра |
|---|---|---|---|
| Шир и окрестности | 1-4 | конец сентября | моховой зелёный, охра, золото, тёплый кремовый |
| Дикие земли | 5-7 | октябрь | холодный синий, сухая охра, сумеречное золото |
| Горы и Мория | 8-10 | январь | ледяной белый, сине-чёрный, красный отсвет |
| Лориэн и Андуин | 11-14 | январь-февраль | золото, серебро, серо-зелёный |
| Эмин Муил, Топи, Врата | 15-17 | конец февраля, начало марта | серый, болотный олив, уголь |
| Итилиэн | 18-19 | начало марта | зелень, бронза, закатное золото |
| Мордор | 20-23 | март | чёрный, пепельный, раскалённый красный |

## Как удержать один стиль на всём наборе

1. Сначала эталон: [locations/01-bag-end.md](locations/01-bag-end.md). Генерируйте, пока не понравится, и сохраните лучший вариант.
2. Дальше этот эталон прикладывается ко всем генерациям:
   - **Midjourney:** `--sref <ссылка на эталон>`. Для персонажей ещё omni-reference с листом персонажей.
   - **ChatGPT, Gemini:** приложить эталон и дописать в начало промпта `Match the art style of the attached reference image exactly.`
   - **Stable Diffusion, Flux:** IP-Adapter с эталоном или своя LoRA, если картинок много.
3. Один генератор на весь набор. Смешивать Midjourney и ChatGPT в одном приложении заметно даже с одинаковыми промптами.
4. Сохраняйте seed или ссылку на каждую удачную генерацию: понадобится, если придётся переделать картинку через полгода.

## Персонажи

- В промптах нет имён персонажей и названий книг или фильмов. Имена тянут генератор к лицам актёров.
- Хоббиты везде либо мелкие фигурки со спины, либо по листу персонажей ([characters/character-sheet.md](characters/character-sheet.md)). Крупных лиц в сценах вех нет: так проще сохранить единообразие.
- Хоббиты всегда босые. Для шагомера это ещё и бесплатная шутка.
