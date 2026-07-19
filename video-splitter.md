# Різалка відео (iPhone) — «Порізати відео 2.6»

Робоча версія: **Порізати відео 2.6.shortcut** (у цій папці, підписаний `shortcuts sign --mode anyone`).
Повна інструкція: https://claude.ai/code/artifact/b3e3b1dd-1f7e-4367-a5bd-404f142cf949

Сценарій: Фото → Поділитися → «Порізати відео 2.6» → вказати секунди (типово 60) → частини в галереї. Все локально, без перекодування (`-c copy`), тому миттєво і без втрати якості.

## Налаштування на новому iPhone

1. Встановити повний **a-Shell** (не mini), відкрити, перевірити `ffmpeg -version`.
2. Імпортувати `Порізати відео 2.6.shortcut` (AirDrop або через Mac + iCloud-синк).
3. В a-Shell: `pickFolder` → iCloud Drive → **Shortcuts** → підтвердити; далі `showmarks` і `renamemark <ім'я-з-showmarks> Shortcuts`.
4. Перший запуск: роздати всі дозволи (Фото, a-Shell, збереження).

## Схема дій шортката

1. Запитати вхідні дані (число, типово 60) — секунди на частину
2. Задати назву → `input.mov` (розширення включене!)
3. Зберегти файл → папка Shortcuts, шлях `VideoSplit.nosync/input.mov`, перезапис
4. a-Shell Execute Command — **run in extension** (`openWindow=close`), keepGoing off:
   ```
   cd ~Shortcuts
   cd VideoSplit.nosync
   rm -f part_*.mov
   ffmpeg -y -i input.mov -c copy -map_metadata -1 -map 0:v:0 -map 0:a:0? -f segment -segment_time [Надана відповідь] -segment_format mov -reset_timestamps 1 part_%03d.mov
   rm -f input.mov
   ```
5. Отримати файл → шлях `VideoSplit.nosync` (error if not found on)
6. Отримати вміст папки ← результат п.5
7. Відфільтрувати файли ← сортувати за Name, Ascending
8. Повторити для кожного → Зберегти у фотоальбом ← Повторюваний елемент
9. a-Shell Execute Command (extension): `cd ~Shortcuts` / `cd VideoSplit.nosync` / `rm -f part_*.mov input.mov`
10. Сповіщення «Готово!»

## Граблі, на які вже наступали (не повторювати)

- **a-Shell Put File / Get File** кладуть/читають файли в контейнері, який не видно ні командам в extension, ні в app — не використовувати взагалі. Обмін файлами тільки через iCloud Drive/Shortcuts/VideoSplit.nosync стандартними діями «Команд».
- **Команди з «Open the app: Yes»** не повертають stdout у Shortcuts і не блокують виконання (шорткат біжить далі, поки ffmpeg ще працює). Тільки «No (run in extension)» — у plist це `openWindow=close`.
- **«Задати назву»**: тумблер «Не включати розширення файлу» має бути ВИМКНЕНИЙ, інакше файл стає `input.mov.mov`.
- Закладка `~Shortcuts` створюється на кожному телефоні окремо (`pickFolder`), a-Shell дає їй технічне ім'я — треба `renamemark … Shortcuts`.
- Робоча папка `VideoSplit.nosync`: суфікс `.nosync` вимикає iCloud-синхронізацію папки — проміжні файли не вивантажуються в хмару.
- «Зупинити і відповісти» при прямому запуску запечено в plist: `WFWorkflowNoInputBehavior` → Name `WFWorkflowNoInputBehaviorShowError`, Parameters.Error = текст.
- Порядок у галереї: ffmpeg копіює у частини дату зйомки оригіналу → всі частини «зняті» в одну секунду → Фото їх тасує. Фікс: `-map_metadata -1`; порядок реально забезпечує алфавітне розгортання вмісту папки при збереженні (сортувальний фільтр у плісті фактично сортує список з одного елемента — папки).
- Частини — 60±1–2 с: різання по ключових кадрах (ціна `-c copy`). Точні різи можливі лише з перекодуванням (повільно).
- «Отримати вміст папки» так і не вдалось зв'язати з папкою: WFTextTokenAttachment у WFFilePickerParameter ігнорується (папка проїжджає далі транзитом), а БЕЗ параметра неявний вхід у імпортованому плісті теж не підхоплюється (v2.7: порожній список, нуль ітерацій, все зникло — відкочено). РОБОЧИЙ механізм 2.6: у цикл потрапляє сама папка, і «Зберегти у фотоальбом» сам розгортає її вміст. Тому запит дозволу «save 1 folder and N media items» — це НОРМА, не лагодити.
- У діях a-Shell Execute перемикач «Показувати» (`ShowWhenRun`) типово увімкнений — показує вікно з виводом (для `rm` — порожнє). В обох діях має бути вимкнений.

## Як зібрано (для майбутніх правок)

Плист шортката редагувався напряму (XML → binary plist → `shortcuts sign --mode anyone`). Проміжні версії і скрипти були в scratchpad сесії; фінальний XML — це v2.6 (scratchpad: shortcut-v26.xml); v2.7 (прибраний WFFolder заради «чистого» ланцюжка) виявився неробочим — не повторювати: ask + setname + save(VideoSplit.nosync/input.mov) + a-Shell execute + get file + get folder contents + repeat/save + cleanup + notification. Схеми параметрів дій — з `/System/Library/PrivateFrameworks/WorkflowKit.framework/Versions/A/Resources/WFActions.plist`, enum openWindow — з `Base.lproj/Intents.intentdefinition` репозиторію holzschu/a-shell.
