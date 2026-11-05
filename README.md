# Адаптер проектов DipTrace к новым XML-базам (.elixml / .libxml)

Здесь --> **[Открыть инструмент онлайн: Адаптер проектов DipTrace к новым XML-базам](https://wadi-li.github.io/diptrace-project-xml-adapter/)** <-- рабочая веб-версия на GitHub Pages.

- **Рабочий инструмент на сайте (GitHub Pages):** [https://wadi-li.github.io/diptrace-project-xml-adapter/](https://wadi-li.github.io/diptrace-project-xml-adapter/)
- **Скачать автономный HTML-файл (работает без интернета):** [DipTrace-Project-XML-Adapter.html](https://github.com/wadi-li/diptrace-project-xml-adapter/raw/main/DipTrace-Project-XML-Adapter.html) · [index.html](https://github.com/wadi-li/diptrace-project-xml-adapter/raw/main/index.html)

Автономная однофайловая HTML-утилита для адаптации файлов проектов DipTrace (принципиальной схемы `.dchxml` и печатной платы `.dipxml`), созданных на старых бинарных библиотеках `.eli` / `.lib`, к работе с новыми XML-базами элементов (`.elixml` и `.libxml`). Сервер и подключение к интернету не нужны — все файлы обрабатываются локально в браузере.

## Решаемые задачи

1. **Сверка и исправление сквозной цепочки из 6 звеньев:**
   - **Звено 1 — 3D-модель (STEP/WRL):** проверка и нормализация путей `<Model3D><Filename><Path>` и `<Var>` по путям из настроек DipTrace (`%my3dmodels%` и `%maindir%\models3d`).
   - **Звено 2 — Посадочное место в `.libxml`:** сверка корпуса `<Pattern><Name>` с актуальными базами корпусов `.libxml`, включая автоматическое сопоставление переименованных посадочных мест (например, `_CAP_0402sm` → `_CAP_0402`, `MB8F` → `Bridge MB8F`, `_IND_0603` → `_FB_0603 (1608)`, `SOP8P65_300X640X120L60X25N` → `_TSSOP8-P650-Z21`, `_SOT23-5` → `_SOT23-5 (SOT-25, SC-74A, DBV, MO-178AA)`, `_MSOP10-P500-3X3-Z21` → `_MSOP10-P500-3X3-Z21 (DGS, MO-187BA, SOT552-1, RM-10, MS, UB, VSSOP10)`, `YANGXING_SMD_3225_4L` → `SMD3225 Crystal Rezonator`).
   - **Звено 3 — УГО и запись в `.elixml`:** поиск компонента в подключённых библиотеках `.elixml` (в том числе при переносе компонента между библиотеками, например `MB8F` из `__user_sim-it` в `__user_diodes.elixml`) и синхронизация `<BaseName>` с `<Name>`.
   - **Звено 4 — Элемент в схеме (`.dchxml`):** перевод ссылок `<LibPath>` со старых библиотек `.eli` на `%mylibs%\*.elixml`, восстановление утерянных привязок `<Pattern Style="..." />` в кэше схемы и обновление 3D-моделей.
   - **Звено 5 — Элемент на плате (`.dipxml`) по `RefDes` и `UpdateId`:** сверка и синхронизация имени компонента (`<Name>`), номинала (`<Value>`), пользовательских полей (`<AddFields>`), посадочного места и путей `<LibPath>` между схемой и платой как по прямому `RefDes`, так и по иерархическому `UpdateId` (например, `C108` → `1C108..4C108` в многоканальных иерархических проектах). Переходные отверстия `Type="Via"` и виртуальные иерархические порты (`NetPort*`, `HConnector*`, иерархические блоки) обрабатываются корректно.
   - **Звено 6 — Вывод в 3D-модель:** сквозная синхронизация параметров `<Model3D>` (`Rotate`, `Offset`, `Zoom`, `IPC_XOff`, `IPC_YOff`, `AutoHeight`, `AutoColor`) между эталоном в `.libxml`, схемой и платой, включая очистку устаревших 3D-моделей у немонтируемых перемычек (`_JP_*_NO`), если в `.libxml` модель не назначена.
2. **Наглядная индикация состояния:**
   - **Зелёным цветом** выделяются исправные зависимости по каждому из 6 звеньев.
   - **Красным цветом** выделяются потерянные связи, устаревшие пути `.eli`, переименованные корпуса и расхождения между схемой и платой.
3. **Построчное и пакетное исправление:**
   - Кнопка **«Исправить все потерянные связи»** устраняет все обнаруженные расхождения в один клик.
   - Кнопка **«Исправить»** в строке элемента исправляет цепочку выбранного `RefDes`.
   - Кнопка **«Связи»** открывает подробную карточку всех 6 звеньев элемента с возможностью вручную выбрать целевую библиотеку `.elixml` и посадочное место `.libxml`.
4. **Сохранение исправленных файлов и отчёта:**
   - Выгрузка исправленной схемы (`.dchxml`) и печатной платы (`.dipxml`).
   - Экспорт полного отчёта сверки в CSV (разделитель `;`, кодировка UTF-8 с BOM, экранирование полей).

## Настройки путей DipTrace (`libgroups_var.dat`)

По умолчанию в инструменте предзагружены пути из настроек библиотек DipTrace (`C:\ProgramData\DipTrace\Data_Unicode\libgroups_var.dat`):

- `%mylibs%` = `C:\Users\limar\OneDrive\DipTrace\lib`
- `%my3dmodels%` = `C:\Users\limar\OneDrive\DipTrace\models3d`
- `%maindir%` = `C:\Program Files\DipTrace`
- `%standardlibs%` = `C:\Program Files\DipTrace\Lib`

В страницу встроен снимок каталога из 16 библиотек `.elixml` (3146 УГО) и 14 библиотек `.libxml` (1223 посадочных места). При запуске слоты схемы и платы пусты («Не загружена») — рабочие файлы `.dchxml` и `.dipxml` загружаются вручную перетаскиванием или через кнопки выбора файлов, чтобы исключить случайное сравнение с чужим проектом. Кнопкой «Обновить базы (.elixml, .libxml)» можно в любой момент подгрузить свежие файлы библиотек.

## Как сохраняются файлы XML

- XML не перестраивается целиком через DOM-сериализатор: выполняются точечные замены только в обновляемых узлах (`<LibPath>`, `<BaseName>`, `<Pattern><Name>`, `<Model3D>`, синхронизируемых полях `<Component>` на плате).
- Побайтово сохраняются метка UTF-8 с BOM (`\ufeff`), переводы строк `CRLF`, порядок атрибутов и форматирование неизменённых секций DipTrace.
- При обновлении блока `<Model3D>` учитываются единицы измерения корневого документа: для схемы (`.dchxml`, `Units="mil"`) смещения `Offset` (`X`, `Y`, `Z`) и `IPC_XOff`/`IPC_YOff` из библиотеки `.libxml` (`Units="mm"`) автоматически пересчитываются в милы (`мм / 0.0254`), а для платы (`.dipxml`, `Units="mm"`) записываются в миллиметрах.
- Светлая тема включена по умолчанию, доступен переключатель тёмной темы.

## Другие утилиты DipTrace XML

- [Ремонт библиотек DipTrace XML (.libxml / .elixml)](https://wadi-li.github.io/diptrace-xml-repair/) — [репозиторий](https://github.com/wadi-li/diptrace-xml-repair)
- [Очистка и упорядочивание полей библиотек DipTrace (.elixml)](https://wadi-li.github.io/diptrace-field-cleaner/) — [репозиторий](https://github.com/wadi-li/diptrace-field-cleaner)
- [Заполнение пропусков позиционных обозначений (RefDes) в схемах .dchxml](https://wadi-li.github.io/diptrace-refdes-repair/) — [репозиторий](https://github.com/wadi-li/diptrace-refdes-repair)
- [Устранение пропусков RefDes с заменой надписей в .dchxml](https://wadi-li.github.io/diptrace-refdes-renumberator/) — [репозиторий](https://github.com/wadi-li/diptrace-refdes-renumberator)
- [Исправление интерактивного HTML BOM DipTrace](https://wadi-li.github.io/diptrace-html-bom-repair/) — [репозиторий](https://github.com/wadi-li/diptrace-html-bom-repair)
- [ЕСКД: перечень элементов (ПЭ3) и спецификация из CSV / .dchxml](https://wadi-li.github.io/eskd-csv-pdf/) — [репозиторий](https://github.com/wadi-li/eskd-csv-pdf)
