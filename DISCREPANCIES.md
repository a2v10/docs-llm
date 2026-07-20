# Расхождения — рабочий реестр

Не часть публикуемой доки: файла нет в `llms.txt` и в секционных индексах, он живёт в корне
рядом с `CONVENTIONS.md`. Здесь копятся места, где docs-llm расходится сам с собой, со
справкой или со скиллом `a2v10`. Разбирать по одному пункту за раз; ничего не править «пачкой».

Заполнено 2026-07-20 по итогам сплошного прочтения доки (49 файлов) и скилла.

## Правила доказательности

Установлено в ходе разбора: **ни один из вторичных источников не является эталоном.**

| Источник | Где | Чего стоит |
|---|---|---|
| Реализация | `C:\A2v10_Net6` (Core, актуальная), `C:\A2v10_Net48` | Первично. Не смотрели — отложено сознательно, дорого по времени |
| JSON-схемы рантайма | `.../App_application/@schemas/model-json-schema.json` | Машинно-проверяемый контракт `model.json`. Дёшево, закрыло бы половину раздела B |
| Реальные приложения | `A2v10.Web.Sample`, примеры в скилле (`examples/*.vxaml`, `model.json`, `*.sql`) | Рабочий код — сильнее любой прозы |
| Укр. справка | `C:\A2v10_Net48\A2v10.Uk.Help\A2v10.Help\html\` | Отстаёт от реализации. Не эталон |
| Скилл `a2v10` | папка плагина Claude | Местами впереди справки, местами выдумывает |

Практическое следствие: формулировка «проверено по справке» ничего не доказывает. Пункты
ниже, где расхождение подтверждено только справкой, помечены как **не закрытые**.

## Решено (правки согласованы, но НЕ внесены)

| # | Что | Решение |
|---|---|---|
| R1 | UTC-модификатор | Правильно `!Utc`. Вариант `!UtcDate` из скилла — неверен. Добавить `!Utc` в таблицу модификаторов `sql/markers.md` |
| R2 | `!RowNumber` | Существует как маркер результирующего набора: нужен для правильной перенумерации строк на клиенте. Добавить в `sql/markers.md`. К TVP отношения не имеет — там это обычное поле с обычным именем |
| R3 | Stimulsoft | Устарел, выпилить полностью: тип `stimulsoft`, свойства `report` и `variables`, расширение `.mrt`, пример «Stimulsoft report» в `model/reports.md`, упоминания в `model.md` и `xaml.md`. **Открыто:** чем заменён — `pdf`/`xlsx` (так в скилле) или в `reports` остаются только `xml`/`json` |

## Снято (ошибки анализа, оставлено чтобы не повторить)

| # | Что утверждалось | Почему неверно |
|---|---|---|
| X1 | XAML-примеры «забыли `$`»: надо `Documents.$selected` вместо `Documents.Selected` | Спутаны два разных namespace: `$` в SQL-модели и `$` в TS-объекте клиента. XAML-биндинг ходит по модели и к TS-членам не обращается. Правило «all service members are prefixed with `$`» из `client/overview.md` относится только к клиентскому API. Исключение `{Bind Agents.$selected}` в `Slot.Scope` — частный случай, не общий паттерн. Реальная проблема в тех же файлах осталась, но другая — см. A1 |
| X2 | `RowNo` в TableType — дефект, платформа не заполнит | Неверно. В TVP это обычное поле с обычным именем, примеры в `sql/update-model.md` корректны. Остаток находки перенесён в B1 |

## A. Внутренние противоречия docs-llm

Здесь внешний эталон не нужен — дока спорит сама с собой.

### A1. Несуществующий путь `.Selected` в аргументах команд

`xaml/bind.md:118`, `xaml/controls/datagrid.md:99`, `xaml/layouts/page.md:62,70`,
`xaml/layouts/toolbar.md:49,53` пишут `Argument={Bind Documents.Selected}`.

В реальных вьюхах команды с суффиксом `Selected` принимают **весь массив**, выбранный элемент
платформа достаёт сама:

```xml
Command="{BindCmd DbRemoveSelected, Argument={Bind Samples}}"
Command="{BindCmd OpenSelected, Argument={Bind Documents}}"
DoubleClick="{BindCmd Dialog, Action=EditSelected, Argument={Bind Samples}}"
```

Пути `Documents.Selected` нет ни в SQL-модели, ни в TS-объекте.
Источник: `examples/catalog/simple/index.view.vxaml`, `examples/document/operation/invoice/index.view.vxaml`.

### A2. Совет гасить кнопки руками противоречит нашей же доке

`xaml/layouts/toolbar.md:91` — «Buttons in toolbars often use `Disabled="{Bind !Items.HasSelected}"`»,
плюс `Disabled="{Bind !Docs.HasSelected}"` в примере (`toolbar.md:51,56`) и
`If="{Bind Items.HasSelected}"` (`page.md:64`, `bind.md:119`).

`template/commands.md` в примечаниях: «Controls bound to a command are enabled and disabled
automatically from `canExec`; never set `Disabled` yourself». Реальные вьюхи ничего не гасят.
Свойства `HasSelected` в справке нет вообще.

### A3. `client/overview.md` — битый пример

```js
'TDocument.RowCount'() { return this.Rows.Count; }
```

Члена `.Count` нет ни в JS-массиве, ни в `IElementArray`. Должно быть `.length` (или `$RowCount`,
если он там читается — см. A4).

### A4. `$RowCount` — неясно, откуда читается

`sql/paging.md` (Notes) утверждает: значение `!RowCount` попадает в свойство `$RowCount` массива.
Это же сказано в `models/paging.html`. Но в таблице членов `IElementArray` (`client/array.md`)
`$RowCount` отсутствует.

Возможно, это тот же случай, что X1 — SQL-side свойство модели против TS-интерфейса, и
противоречия нет. **Требует уточнения**, прежде чем что-либо править.

### A5. `$isDirty` не документирован там, где на него ссылаются

`template/options.md` в Overview: «The model exposes `$isDirty`». В `client/root.md` (IRoot)
такого члена нет; `$isDirty` документирован только на `IController`. По доке непонятно, как
шаблон читает его с модели.

### A6. `sql/markers.md` неполон относительно собственных страниц

Каталог маркеров — хаб раздела, но в таблице модификаторов полей нет:

| Маркер | Где используется у нас |
|---|---|
| `!HasChildren` | `sql/tree.md` (динамическое дерево) |
| `!Metadata` | `sql/update-model.md` |
| `!Utc` | `sql/system-datasets.md` (`$Defaults`) |
| `!RowNumber` | нигде — но существует, см. R2 |
| `!Permissions` | нигде — см. C4 |

Скилл прямо инструктирует модель: «Not in the references → it does not exist». Неполный
каталог работает как активная дезинформация. Это дефект, внесённый при заполнении `markers.md`
2026-07-20.

### A7. Ссылки в заглушки

`sql/procedures.md` и `sql/overview.md` остаются `TODO`, но на `sql/procedures.md` ссылаются
как минимум `model/actions.md`, `model/commands.md`, `model/dialogs.md`,
`xaml/controls/selector.md`. Каждая ссылка приводит на пустую страницу.

## B. docs-llm против укр. справки

Справка отстаёт от реализации — **ни один пункт раздела не закрыт**, каждый требует сверки с
кодом или схемой.

### B1. Служебные колонки TVP не документированы

`models/update.html` перечисляет колонки, которые платформа заполняет автоматически, если
объявить их в табличном типе: `GUID`, `RowNumber`, `CurrentKey`, `ParentId`, `ParentGUID`,
`ParentKey`, `ParentRowNumber`.

`sql/update-model.md` документирует только `GUID` / `ParentGUID` и подаёт их как единственный
способ связать детей с родителем. Про `CurrentKey` / `ParentKey` (заполнение таблиц типа `Map`)
и `ParentRowNumber` не сказано ничего.

### B2. Экспорт не упоминает процедуру `.Export`

`app/actions.html`: «При завантаженні моделі для експорту буде викликатися збережена процедура
з суфіксом **.Export**». В `model/actions.md` (раздел `export`) этого нет — процедура просто не
найдётся. Верба `Export` нет и в списке стандартных в `CLAUDE.md`.

### B3. Свойство `file` у команд описано неверно

`model/commands.md`: «`file` | File path — used with `file` type».
`app/commands.html`: `file` — имя серверного js-модуля для типа `javascript` и имя файла
процесса для `startProcess` / `resumeProcess`.

### B4. `report` и `xmlSchemas` — расширения

`app/reports.html`: `report` — имя файла **без расширения**, `.mrt` добавляется платформой;
`xmlSchemas` — имена **без** `.xsd`, относительно текущей папки (допускается `../`).
В `model/reports.md` таблица говорит «filename (`.mrt`)» и «Paths to XSD schema files».

Частично снимается решением R3 (стимулсофт выпиливается вместе с `report`), но `xmlSchemas`
остаётся.

### B5. `name` в отчёте — не имя файла

`app/reports.html`: `name` — имя (заголовок) отчёта; именем экспортируемого файла оно
становится только в режиме экспорта. В `model/reports.md` описано как «Downloaded filename», а
в примере стоит `"name": "Agent.mrt"` — расширение шаблона выдано за имя скачиваемого файла.

### B6. `checkTypes` — выдуманное примечание

`model/actions.md` (Notes): «`checkTypes` is resolved at development time only; it has no effect
in production builds». В A2v10 нет шага сборки вообще. `app/actions.html`: `.d.ts` для
**динамической** проверки типов.

## C. docs-llm против скилла `a2v10`

Ни одна сторона не эталон. Похоже на разрыв NET48 ↔ Core.

### C1. Типы отчётов

Наша дока (вслед за справкой): `stimulsoft | xml | json`. Скилл: `xml | json | pdf | xlsx`.
Пересекается с R3.

### C2. Формат `clrType`

Наша дока: `"MyApp.Commands.SendReportCommand, MyApp"` (сборочно-квалифицированное имя).
Скилл: `clr-type:My.Type;assembly=MyAssembly`.

### C3. Раздел `files` — оба списка неполны

Наши типы: `parse | sql | azureBlob | clr`; наши `parse`: `excel | xlsx | csv | dbf | xml | auto`;
наши свойства: `imageCompress`, `async`, `container`, `azureSource`, `locale`.
Скилл добавляет типы `blobStorage | json | excel | text`, форматы `xls | json` и свойства
`outputFileName`, `zip`, `blobSource`, `key`. Ни один список не является надмножеством другого.

### C4. `permissions` отсутствует у нас полностью

Скилл документирует `permissions` на каждом элементе `model.json` (значения `view | edit |
delete | apply | create | unapply | flag64 | flag128 | flag256`) и биты
`view=1, edit=2, delete=4, apply=8`. В справке этого нет, в docs-llm — тоже.

Прямое следствие: `sql/system-datasets.md` описывает модификатор `Permissions` как «bit mask of
access rights», не говоря, какие это биты и где задаются права. Ссылаться некуда.

### C5. Прочие свойства команд

Скилл перечисляет у команд `command` и `target` (`Object.Method`), у actions — `skipDataStack`,
`plain`. У нас их нет.

### C6. Маркеры, которых нет у нас

Скилл: `!Json` (текстовая колонка с JSON, десериализуется в объект, **не реактивный**),
`!Expanded` (узел дерева отрисован развёрнутым, только статические деревья), `!Permissions`.
В справке `!Json` и `!UtcDate` не встречаются вообще.

## D. Устаревшее и мета

### D1. Расширение файлов вью

`xaml/overview.md`: «XML files with `.xaml` extension». Актуально `.vxaml` (`.xaml` — legacy,
одно расширение на проект). `model/actions.md` в прозе: «renders `index.xaml`».

### D2. Шаблоны названы JavaScript

Везде «JavaScript template file». Фактически `.template.ts` (TypeScript) плюс `.d.ts` для
`checkTypes`.

### D3. Понятия модулей нет

`app/menu.md` (Notes): URL, начинающиеся с `$`, «target built-in platform endpoints». По скиллу
`$<segment>` — маркер **модуля**, резолвящегося через конфиг в source root; встроенные
эндпойнты лишь частный случай. Модулей в docs-llm нет вообще.

### D4. `CLAUDE.md` разошёлся с репозиторием

- «The root `llms.txt` is a flat, one-level index with only **three** entries — SQL, XAML,
  MODEL» — записей шесть (добавились TEMPLATE, CLIENT, APP).
- В дереве Content Structure не хватает `xaml/layouts/sheet.md`.

### D5. Непокрытые страницы справки

`models/blob.html` (бинарные объекты) и `models/rowversion.html` (отслеживание изменений).
`models/update.html` ссылается на rowversion как «дивись також» — у нас ссылаться некуда.
