# Описание формата данных

В этой директории находятся все данные, которые используются при генерации статичного
сайта. Так как изначально никакого плана развития не было, формат данных постепенно
обрастает костылями и становится трудно читаемым даже для меня. Представьте, как
трудно будет разобраться желающему добавить запись не через меня, а самостоятельно? :)

## Общая структура

В данный момент вся "база данных" состоит из трёх файлов формата JSON:

* `categories.json` -- описание категорий, отображающихся на главной странице;
* `streams.json` -- список всех стримов, входящих в архив;
* `games.json` -- список всех прохождений или серий стримов.

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
| `level` | **Да** | Уровень заголовка этой категории. Должен быть ≥ 3. |
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
"225279322": {
  "vk": "87862793_456241095",
  "offset": "3:46:12"
  "note": "Официальной записи не будет"
},
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
| `vk`<sup>*</sup> | Нет | ID видео в социальной сети ВКонтакте. В данный момент поддерживаются ID подобного формата: `87862793_456241096`. ВКонтакте требует наличие активного бэкенда, поэтому его стоит избегать, когда это возможно. |
| `direct`<sup>*</sup> | Нет | Прямая ссылка на видео. Это поле, в основном, применяется для пассивного добавления временных записей из внешнего автоматизированного архива. Не рекомендуется в качестве источника постоянных записей. |
| `official` | Нет | Указывает, является ли запись видео официальной. Проверяется только на значение `false`. |
| `segment` | **Да** (для сегментированных стримов) | Номер сегмента для стримов, заданных массивом. Не имеет смысла в файле `stream.json`, поэтому должен применяться только в `games.json` (см. ниже). Тип данных -- целое число, нумерация сегментов начинается с нуля. Пример: 2-й стрим по Getting Over It, который ссылается на 3-й сегмент стрима `233032380`. |

\* Поле, указывающее на постоянную запись стрима. В объекте должно присутствовать только **одно** поле этого типа.

| Поле | Точка отсчёта | Описание |
| ---- | :-----------: | -------- |
| `start` | стрим | Таймкод, с которого должно начинаться воспроизведение стрима. Эта опция полезна в случаях, когда на записи перед основной игрой идёт другая. Значение этого поля должно присутствовать в таймкодах соответствующего стрима (см. `timecodes.json`). |
| `end` | запись | Таймкод, на котором нужно **прервать** воспроизведение. Также скрывает все таймкоды, значение которых больше, чем у данного поля. |
| `offset` | стрим | Смещение, применяемое к субтитрам и таймкодам. Полезно в случаях, когда начало записи не совпадает с началом стрима. Пример: стрим `231626788`. |


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
| `type` | Нет | Тип серии стримов. Пока что влияет только на отображение игры на главной странице. Допустимые значения: `list`. |
| `id` | **Да** | Уникальный ID прохождения. Используется в имени файла, который будет сгенерирован для данного прохождения и помещен в директорию `/links`. |
| `streams` | **Да** | Содержит массив урезанных объектов "Стрим". Должен содержать ключ `twitch`, присутствующий в `streams.json`. Также желательно заполнить поле `name` — заголовок стрима на странице прохождения. Дополнительно можно переопределить параметры `start`, `end`, `offset` и `note` объекта "Стрим", если это требуется в контексте прохождения. |
| `thumbnail` | Нет | Позволяет переопределить сегмент, из которого будет взята иконка для главной страницы. По умолчанию эта опция равна 0 (первому сегменту). |
