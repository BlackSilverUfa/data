# Описание формата данных

В этой директории находятся все данные, которые используются при генерации статичного
сайта. Так как изначально никакого плана развития не было, формат данных постепенно
обрастает костылями и становится трудно читаемым даже для меня. Представьте, как
трудно будет разобраться желающему добавить запись не через меня, а самостоятельно? :)

## Общая структура

В данный момент вся "база данных" состоит из трёх файлов формата JSON:

* `categories.json` -- описание категорий, отображающихся на главной странице;
* `streams.json` -- список всех стримов, входящих в архив;
* `streams-meta.json` -- модификаторы стримов;
* `games.json` -- список всех прохождений или серий стримов;
* `config.json` -- глобальные настройки проекта;
* `tcd.json` -- настройки Twitch-Chat-Downloader;

Ниже подробнее рассмотрим каждый из них...

## Категории (`categories.json`)

Этот файл представляет собой **массив** JSON, который содержит множество объектов.
Каждый объект представляет собой описание отдельно взятой категории. Так как это
массив, порядок расположения категорий остаётся таким же и на итоговой главной
странице.

```
[{}, {}]
```

### Объект "Категория"

```
{
  "name": "Завершённые прохождения",
  "code": "finished",
  "level": 3
}
```

| Поле | Обязательное | Описание |
| ---- | :----------: | -------- |
| `name` | **Да** | Название категории. Отображается в виде заголовка над списком прохожений. |
| `code` | **Да** | Код категории. На него ссылаются прохождения из файла `games.json`. |
| `level` | Нет | Уровень заголовка этой категории. Должен быть ≥ 2, по умолчанию равен 2. |
| `description` | Нет | Текст между заголовком категории и списком прохождений. |
| `split_by_year` | Нет | Если указать значение `false`, то категория не будет разделяться по годам на главной странице. |
| `search` | Нет | Если указать значение `false`, то категория не будет появляться в поиске по сайту. |

#### Особые коды категорий

| Код | Описание |
| --- | -------- |
| `recent` | Содержит 10 последних сегментов с корректными названиями. |

## Стримы (`streams.json`)

Этот файл содержит **объект** JSON (аналог ассоциативного массива в Java или словаря в Python). Каждый ключ -- ID стрима на Twitch. Этот ключ ссылается на другой объект (или массив таких объектов), который содержит дополнительную информацию о стриме: таймкоды, записи и т.д.

```
{
  "0": {},
  "1": {}
}
```

### Объект "Стрим"

```
"492682718": { "youtube": "di9TMwtyb0I" },
"233032380": [
  { "youtube": "4q-FGOYnXPk" },
  { "youtube": "U8seqd_gNo4", "offset": "1:49:06" },
  { "youtube": "IBDhRIYPURI", "offset": "3:32:56" }
],
```

| Поле | Обязательное | Описание |
| ---- | :----------: | -------- |
| ключ | **Да** | Это не поле, но стоит упомянуть его здесь. Каждый стрим начинается с его ID на Twitch. ID состоит только из цифр, но пишется как строка (требование JSON). |
| `note` | Нет | Заметка, которая будет отображаться в информации о стриме на странице прохождения. |
| `youtube`<sup>*</sup> | Нет | ID записи на YouTube. Самый простой и стабильный способ добавить вечную запись стрима в архив. Только ID (например `43q8zNpTCCE`), не весь URL, это важно! |
| `direct`<sup>*</sup> | Нет | Прямая ссылка на видео. Это поле, в основном, применяется для пассивного добавления временных записей из внешнего автоматизированного архива. Не рекомендуется в качестве источника постоянных записей. |
| `torrent` | Нет | Прямая ссылка на Torrent-файл с записью стрима. В данный момент этот параметр только заменяет прямую ссылку для скачивания видео, т.е. он должен использоваться вместе с параметром `direct`. В будущем планируется реализация плеера WebTorrent, что позволит разгрузить сервер с записями. |
| `official` | Нет | Указывает, является ли запись видео официальной. Проверяется только на значение `false`. |
| `force_start` | Нет | Указывает, что плеер не должен разрешать перемотку видео на более ранние моменты, чем указанный в параметре `start` (см. ниже). |

\* Поле, указывающее на постоянную запись стрима. В объекте должно присутствовать только **одно** поле этого типа.

| Поле | Точка отсчёта | Описание |
| ---- | :-----------: | -------- |
| `start` | стрим | Таймкод, с которого должно начинаться воспроизведение стрима. Эта опция полезна в случаях, когда на записи перед основной игрой идёт другая. В `streams.json` данный параметр ограничивает таймкоды, а в `games.json` не влияет на них. |
| `end` | стрим | Таймкод, на котором нужно **прервать** воспроизведение. Также скрывает все таймкоды, значение которых больше, чем у данного поля. |
| `offset` | стрим | Смещение, применяемое к субтитрам и таймкодам. Полезно в случаях, когда начало записи не совпадает с началом стрима. Пример: стрим `231626788`. |
| `duration` | запись | Переопределяет продолжительность видео, автоматически полученную с помощью youtube-dl или другими способами. Этот параметр не применяется в базе данных, но полезен при тестировании и в CLI. |
| `cuts` | стрим | Массив, содержащий диапазоны, вырезанные из записи. Указанные здесь диапазоны будут вырезаны из таймкодов и субтитров с соответствующей коррекцией таймингов. Для субтитров создаётся отдельный файл `v{стрим}+{сегмент}.ass`, чтобы не терять оригинал. При удалении этого параметра побочные субтитры будут удалены автоматически. |


### Объект "Объединённый стрим"

```
"654380816": {  },
"655265437": {  },
"655528246": {  },
"656257835": {  },
"654380816,655265437,655528246,656257835": {
  "youtube": "ZBHv3E0ZDHo",
  "offsets": ["0:00", "7:32:13", "14:13:20", "17:13:32"],
  "note": "4 стрима были склеены в одну запись."
},
```

Если в `streams.json` вместо ключа стрима указать ключи уже существующих стримов, разделённые запятой, то будет создан ещё один стрим, виртуально объединяющий их в один таймлайн. Субтитры и таймкоды также будут склеены соответствующим образом, принимая во внимание вырезанные моменты и смещения.

В объединённых стримах используются стандартные сегменты. Единственное их отличие в том, что обязательно нужно указывать поле `offsets`, которое содержит список таймкодов, указывающих на начало каждого стрима на записи. Соответственно, таймкодов должно быть столько же, сколько и ID стримов в ключе.


## Сегменты (`segments.json`) [не заполняется вручную]

Этот файл похож на `streams.json`, однако вместо стримов здесь перечислены все сегменты в формате, удобном для внешних приложений и ботов. Из этого файла нельзя загрузить полноценные сегменты — он предназначен только для чтения внешними клиентами. Файл генерируется из базы данных во время каждой сборки и находится в ветке `gh-pages` по пути `data/segments.json`.

### Объект "Скомпилированный сегмент"

```
"399850727": {
  "youtube": "IHc2XRynEQs", "cuts": ["13:23~1:03:17"], "abs_start": "0:00", "abs_end": "7:37:23", "duration": "6:47:29",
  "name": "Sekiro: Shadows Die Twice - 4 (часть 1)"
}
```

Примечание: в таблице не перечислены поля, совпадающие с объектом "Segment", а именно: `youtube`, `direct`, `torrent`, `official` и `cuts`.

| Поле | Описание |
| ---- | -------- |
| `abs_start` | Число секунд, на которое смещён данный сегмент относительно начала стрима. Большую часть времени совпадает со значением поля `offset` соответствующего объекта "Сегмент", однако отличается, если запись не указана. |
| `abs_end` | Число секунд с начала стрима до окончания этого сегмента. Большую часть времени равно сумме поля `offset` и продолжительности записи, однако отличается, если указана запись с переменным смещением или отсутствует. |
| `duration` | Продолжительность записи в секундах. В данный момент поддерживается только для сегментов с YouTube в качестве источника. |
| `name` | Название сегмента, автоматически сгенерированное из ссылающихся на него объектов. Содержит названия игр и номера стримов по ним в хронологическом порядке. |

## Таймкоды (`timecodes.json`)

Этот файл содержит **объект** JSON (аналог ассоциативного массива в Java или словаря в Python). Каждый ключ -- ID стрима на Twitch -- ссылается на объект "Таймкоды" (см. ниже).

# Объект "Таймкоды"

```
"180397849": {
  "17:26": "Road Redemption",
  "1:16:16": "Battle Chasers",
  "2:57:45": "Stardew Valley"
}
```

Объекты "Таймкоды" также могут быть вложенными. В таком случае в списке таймкодов будет выделена отдельная группа.

```
"274651226": {
  "15:49": "Witch Hunt",
  "Cuisine Royale": {
    "4:29:43": "Поиски коопа",
    "4:31:42": "Кооп найден"
  },
  "Realm Royale": {
    "4:13:05": "Поиски коопа",
    "4:15:06": "Кооп найден"
  },
  "The Goatman": {
    "2:07:43": "Поиски коопа",
    "2:11:26": "Кооп найден"
  }
}
```

Ещё кроме простого времени можно указывать диапазоны, разделяя их при помощи тильды (~). В таком случае в одной строке будет показано оба таймкода.

```
"277001302": {
  "2:50:34~2:55:50": "Музыка удалена средствами YouTube"
}
```

* 02:50:34 - 02:55:50 — Музыка удалена средствами YouTube

Если указать отрицательное значение, таймкод не появится при рендеринге HTML. Это полезно в случаях, когда момент не попадает на запись, чтобы не удалять таймкод из истории.

```
"393860794": {
  "-4:10:04": "Трейлер Doom: Annihilation"
}
```

Если в группе таймкодов не требуется указывать имя для каждого пункта, можно указать массив по аналогии с вложенными блоками таймкодов. В таком случае таймкоды будут указаны через запятую. Это особенно полезно на играх типа Dark Souls, чтобы отмечать попытки на боссах.

```
"268497396": {
  "Murderous Pursuits": {
    "2:56:48": "Поиски коопа",
    "Раунды": [
      "3:04:30",
      "3:17:40",
      "3:29:10",
      "3:40:00",
      "3:52:10"
    ]
  }
}
```

## Прохождения (`games.json`)

Этот файл содержит **массив** JSON, который состоит из множества объектов "Прохождение".

```
[{}, {}]
```

### Объект "Черный список"

```
{
  "users": [
    "blackufa"
  ],
  "messages" [
    "^!"
  ]
}
```

Этот объект содержит правила фильтрации сообщений в субтитрах. Подобные объекты используются в `config.json`, Прохождениях и Вложенных ссылках (см. ниже).

Правила, прописанные в `config.json` действуют глобально. А правила, указанные в прохождениях и вложенных ссылках - только на интервале, покрываемом данными ссылками.

Правила наследуются по иерархии, т.е. на каждой вложенной ссылке будут действовать правила, прописанные в ней, в связанном прохождении и глобально.

В списке `users` указываются имена пользователей, которых нужно удалить из субтитров.

В списке `messages` указываются регулярные выражения, по которым будут блокироваться отдельные сообщения.

Сообщения не удаляются из субтитров, а только скрываются. Это означает, что размер файлов и количество сообщений не должны измениться. Также это означает, что эти изменения полностью обратимы.

### Объект "Прохождение"

```
{
  "name": "Black Mirror",
  "category": "finished",
  "id": "black-mirror",
  "streams": [
    { "name": "1", "twitch": "205262158" },
    { "name": "2", "start": "3:07:00", "twitch": "206091904" }
  ],
  "thumbnail": 1
}
```

| Поле | Обязательное | Описание |
| ---- | :----------: | -------- |
| `name` | **Да** | Название прохождения, которое будет отображаться в списке на главной странице и в заголовке страницы самого прохождения. |
| `category` | **Да** | Код категории, к которой относится это прохождение. |
| `type` | Нет | Тип серии стримов. Пока что влияет только на отображение игры на главной странице и в поиске. Допустимые значения: `list`. |
| `id` | **Да** | Уникальный ID прохождения. Используется в имени файла, который будет сгенерирован для данного прохождения и помещен в директорию `/links`. |
| `streams` | **Да** | Массив объектов "Ссылка на сегмент" (см. ниже). |
| `cover` | Нет | Позволяет переопределить сегмент, из которого будет взята иконка для главной страницы и дата первого стрима в серии. По умолчанию эта опция равна 0 (первому сегменту). |
| `blacklist` | Нет | Черный список сообщений для всех стримов по этой игре (см. выше). |

### Объект "Ссылка на сегмент"

```
{ "name": "2", "start": "3:07:00", "twitch": "206091904" },
{ "twitch": "563856575", "subrefs": [
  { "name": "1 (400)", "start": "3:36:11" },
  { "name": "1 (400)", "start": "6:01:02" }
] },
```

| Поле | Обязательное | Описание |
| ---- | :----------: | -------- |
| `name` | **Да**, конфликтует с `subrefs` | Заголовок стрима на странице прохождения. |
| `twitch` | **Да** | ID стрима, на который ссылается данный сегмент. Данный ID также должен присутствовать в файле `streams.json`. |
| `segment` | Нет | Номер сегмента для стримов, заданных массивом. Тип данных — целое число, нумерация сегментов начинается с нуля. Пример: 2-й стрим по Getting Over It, который ссылается на 3-й сегмент стрима `233032380`. |
| `start` | Нет, конфликтует с `subrefs` | Время начала этой ссылки относительно начала стрима. Подсказывает плееру, что нужно перемотать видео, если сохранённая позиция находится левее, чем значение этого поля. Также используется для автоматического разбиения сегментов через CLI. Особый случай: при нуле выбирается не начало стрима, а начало сегмента. |
| `note` | Нет | То же, что и в объекте "Стрим". Переопределяет указанное там значение, если оно есть. |
| `subrefs` | Нет | Если данная ссылка покрывает сразу несколько игр на стриме, их нужно указывать с помощью вложенных ссылок (см. ниже). Вариант с перечислением названий, разделённых слэшем, считается устаревшим и не должен использоваться в новых записях. |
| `blacklist` | Нет | Черный список сообщений для этой части стрима (см. выше). |

### Объект "Вложенная ссылка"

```
{ "name": "1 (400)", "start": "3:36:11" },
{ "name": "1 (400)", "start": "6:01:02" }
```

Это упрощённый вариант ссылки на сегмент, с меньшим количеством доступных полей. Здесь могут использоваться только поля `name`, `start`, `hidden` и `blacklist` (см. выше). При использовании вложенных ссылок в родительской ссылке не должно быть этих полей, т.к. они генерируются автоматически.

При указании в поле `hidden` значения `true` вложенная ссылка будет скрыта с сайта, но по-прежнему будет учитываться при автоматическом добавлении записей.

Значения поля `name` должны быть уникальными в пределах ссылки на сегмент. Если игра повторяется, то к названию 2-й и последующих вложенных ссылок можно добавить "(часть 2)" и т.д. с указанием поля `hidden`. В таком случае при автоматическом разбиении ссылки обе части будут правильно подписаны.

Вложенные ссылки не могут использоваться вне обычных, так как сами они не указывают на какой-либо сегмент.

Заголовок родительской ссылки будет составлен путём соединения всех заголовков вложенных ссылок. Поле `start` берётся из первой вложенной ссылки, как было и до введения этого объекта.

## Стримы (мета) (`streams-meta.json`)

Этот объект содержит модификаторы ко всем стримам. Под ключом `default` задаются значения по умолчанию. В остальных ключах указываются ID стримов, **начиная с которых** будут действовать выбранные модификаторы.

Таким образом можно фиксировать редкие изменения, которые затрагивают сразу множество стримов. Примером такого изменения является включение/отключения хромакея. При включённом хромакее веб-камера всегда находится внизу, из-за чего может перекрываться чатом.

### Список модификаторов

| Поле | Тип | Критерий | Описание |
| ---- | --- | -------- | -------- |
| `chromakey` | `bool` | Стрим с зелёным экраном? | Определяет положение чата на экране (`true` - наверху, `false` - внизу). |