# Карта мира

- **Где в приложении:** главный экран. Карта с зумом и прокруткой, поверх неё приложение рисует маршрут, пройденную часть, маркер игрока и точки вех.
- **Формат:** 1:1 (Midjourney `--ar 1:1`)
- **Экспорт:** мастер не меньше 6144×6144 PNG после апскейла, в приложение тайлами WebP (см. заметки)
- **Приоритет:** A, без неё приложения нет

## Что на карте быть не должно

- **Маршрута.** Его рисует приложение линией по точкам: так можно закрашивать пройденную часть и двигать маркер. Нарисованный в картинке маршрут потом не убрать.
- **Подписей.** Генераторы пишут с ошибками, текст размывается при зуме, его нельзя перевести. Названия мест выводит приложение поверх карты.
- **Рамки, розы ветров, легенды, масштабной линейки.** Это тоже слой интерфейса.

## Схема расположения

Генератор сам не знает, где в Средиземье что находится, и нарисует случайную фэнтези-карту. Поэтому есть две схемы:

- [layout-reference-clean.png](layout-reference-clean.png): только формы (горы, реки, леса, Мордор). Её прикладывать к генерации.
- [layout-reference-annotated.png](layout-reference-annotated.png): то же с подписями, номерами вех и маршрутом. Это для вас, генератору её не давать: он перерисует подписи и красный пунктир.

Схема приблизительная: я строил её по расстояниям из книги, без точной привязки к карте Кристофера Толкина. Сверьте с картой из книги и при желании поправьте схему перед генерацией. Совпадение один в один и не нужно (см. раздел про права в README).

## Prompt A: с приложенной схемой (ChatGPT, Gemini, img2img)

Приложите `layout-reference-clean.png`. Если у генератора есть отдельное поле для референса стиля, туда идёт эталон из `locations/01-bag-end.md`.

```text
Redraw the attached rough layout sketch as a finished illustrated fantasy map. Keep the position, size and shape of every element exactly as in the sketch: rivers (blue lines), mountain ranges (rows of triangles), forests (green and dark green areas), the small golden forest (yellow area), grassland (pale yellow-green), barren lands (tan), marshes (olive), the thin brown road, small dots for settlements, and the dark walled land in the lower right. Use the sketch only as a layout guide: do not copy its flat colors or geometric shapes. Fill the empty areas of the sketch with plausible wild land: gentle hills, scattered small woods, moors and open plains, kept quiet and low in detail. Draw a top-down hand-drawn map with small pictorial symbols: mountains as rows of little shaded peaks with sepia hatching, forests as dense clusters of tiny round-topped trees, hills as small rounded mounds, rivers as thin wavy blue-ink lines that widen downstream, marshes as tufts of reeds and small pools, settlements as tiny clusters of houses or a small tower. The green country in the upper left: rolling hills, hedgerows, small fields. The dark land in the lower right: walled by black jagged mountain ranges on its north and west sides, a barren grey ash plain inside, a lone volcano glowing red and a black tower beyond it. Ink and watercolor on aged cream parchment, fine sepia linework, soft washes. The palette shifts gradually from fresh greens and gold in the north-west to grey, charcoal and ember red in the south-east. Flat top-down view, even lighting, the same level of detail across the whole map. No text, no labels, no place names, no route line, no dotted path, no compass rose, no legend, no scale bar, no grid, no frame or border, no sea monsters.
```

## Prompt B: только текст (Midjourney и генераторы без референса)

Точность расположения будет хуже. Генерируйте много вариантов и берите тот, где горы, река и Мордор легли ближе к схеме.

```text
Illustrated fantasy map, top-down view, square, north is up. Upper left: a gentle green country of rolling hills, hedgerows and small farms, bordered on its east by a slow river running north to south; just east of that river, a small dark old forest and low grassy downs. A thin road runs east from the green country across the upper part of the map, past a small hill village, past a lone tall bare hill at the southern end of a north-south line of low hills, over a second river, through wooded hills, to a hidden valley at the foot of a great mountain range. That mountain range runs north to south down the middle of the map, from the top edge to just below the center, with one reddish snowy peak at its middle. Just east of the mountains near the center, a small golden forest. A wide river flows from the top edge down through the center toward the lower right, widening into a long narrow lake with a rocky island and a waterfall, below and right of the center. Upper right quarter: a vast dark green forest. Below the southern end of the mountains: a green forest, then open yellow-green grassland. A second mountain range curves from the lower left-center to the lower right, ending near a small white walled city beside the river. Right of the lake: a maze of grey rocky hills, then wide olive marshes. Lower right corner: a dark land enclosed by black jagged mountain ranges on its north and west sides, a barren ash plain inside with a lone volcano glowing red and a black tower beyond it. Between the river and the dark land, a narrow strip of green woods. Small pictorial map symbols: rows of little shaded peaks, clusters of tiny trees, thin wavy blue rivers. Ink and watercolor on aged cream parchment, fine sepia linework, soft washes, palette shifting from fresh green and gold in the north-west to charcoal and ember red in the south-east. Flat top-down view, even lighting, uniform detail. No text, no labels, no route line, no compass rose, no legend, no frame or border.
```

## Negative prompt

```text
text, letters, labels, place names, words, runes, route line, dotted path, arrows, compass rose, legend, scale bar, grid, frame, border, cartouche, sea monsters, ships, perspective view, 3d terrain, satellite view, photorealistic, 3d render, CGI, blurry, lowres, oversaturated, neon colors, watermark, signature
```

## Вехи на схеме

Координаты в процентах от левого и верхнего края схемы. На готовой карте точки сместятся: их нужно будет заново снять с финального изображения (в пикселях), это и станет данными маршрута для приложения. Номер вехи совпадает с номером файла в `locations/`.

| # | Веха | x, % | y, % | Картинка |
|---|---|---|---|---|
| 1 | Бэг-Энд, Хоббитон | 10.3 | 15.9 | [01-bag-end](../locations/01-bag-end.md) |
| 2 | Паром в Баклбери | 15.4 | 16.6 | [02-bucklebury-ferry](../locations/02-bucklebury-ferry.md) |
| 3 | Старый лес | 17.8 | 17.6 | [03-old-forest](../locations/03-old-forest.md) |
| 4 | Бри | 22.0 | 15.9 | [04-bree](../locations/04-bree.md) |
| 5 | Заверть | 30.4 | 15.4 | [05-weathertop](../locations/05-weathertop.md) |
| 6 | Брод через Бруинен | 46.3 | 15.0 | [06-ford-of-bruinen](../locations/06-ford-of-bruinen.md) |
| 7 | Ривенделл | 47.7 | 13.6 | [07-rivendell](../locations/07-rivendell.md) |
| 8 | Карадрас | 51.4 | 33.2 | [08-caradhras](../locations/08-caradhras.md) |
| 9 | Врата Дурина | 50.0 | 35.0 | [09-doors-of-durin](../locations/09-doors-of-durin.md) |
| 10 | Залы Мории | 52.3 | 35.5 | [10-moria-halls](../locations/10-moria-halls.md) |
| 11 | Лотлориэн | 55.5 | 37.2 | [11-lothlorien](../locations/11-lothlorien.md) |
| 12 | Великая река | 60.7 | 46.7 | [12-great-river](../locations/12-great-river.md) |
| 13 | Аргонат | 65.4 | 58.1 | [13-argonath](../locations/13-argonath.md) |
| 14 | Рэрос и Амон Хен | 65.8 | 62.9 | [14-rauros-amon-hen](../locations/14-rauros-amon-hen.md) |
| 15 | Эмин Муил | 70.1 | 63.1 | [15-emyn-muil](../locations/15-emyn-muil.md) |
| 16 | Мёртвые топи | 76.2 | 66.8 | [16-dead-marshes](../locations/16-dead-marshes.md) |
| 17 | Чёрные врата | 80.6 | 70.3 | [17-black-gate](../locations/17-black-gate.md) |
| 18 | Хеннет Аннун | 77.8 | 78.7 | [18-henneth-annun](../locations/18-henneth-annun.md) |
| 19 | Перекрёсток | 79.0 | 82.4 | [19-crossroads](../locations/19-crossroads.md) |
| 20 | Минас Моргул | 81.0 | 82.8 | [20-minas-morgul](../locations/20-minas-morgul.md) |
| 21 | Кирит Унгол | 81.8 | 81.0 | [21-cirith-ungol](../locations/21-cirith-ungol.md) |
| 22 | Горгорот | 85.0 | 80.4 | [22-gorgoroth](../locations/22-gorgoroth.md) |
| 23 | Роковая гора | 88.3 | 79.4 | [23-mount-doom](../locations/23-mount-doom.md) |

Вехи 18-23 стоят тесно: в книге последние сотни миль насыщены событиями. На карте это значит, что юго-восточный угол должен хорошо читаться при максимальном зуме.

## Заметки

- **Разрешение.** Генераторы отдают 1-2K, а для зума нужно 6-8K. Путь такой: сгенерировать на максимуме, апскейлить в 4 раза (Upscayl бесплатный, Topaz, Krea или Magnific), при необходимости дорисовать проблемные куски инпейнтингом. Честно: апскейлер при 4x додумывает мелочи сам, на максимальном зуме будет видно. Ограничьте максимальный зум в приложении примерно уровнем 1 пиксель карты = 1 пиксель экрана.
- **Android.** Картинка 6144×6144 целиком в память не влезет (около 150 МБ как bitmap). Карту режут на тайлы 512×512 и показывают через тайловый просмотрщик, например subsampling-scale-image-view или аналог на Compose. Это задача кода, картинке достаточно быть одним большим PNG.
- **Правки.** Если на удачном варианте одна область легла не так (например, озеро не на месте), исправляйте её инпейнтингом (ChatGPT edit, Photoshop Generative Fill, Krita AI Diffusion), а не перегенерацией всей карты.
- **Пустые зоны.** Нижний левый и верхний правый углы схемы почти пустые, это нормально: маршрут идёт по диагонали. Генератор заполнит их холмами и равнинами.
