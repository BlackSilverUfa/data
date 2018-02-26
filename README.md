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
| `type` | Нет | Тип категории (см. ниже). |
| `description` | Нет | Текст между заголовком категории и списком прохождений. |

### Типы категорий

* `list`

Вот тут мы и встретили первый костыль. По крайней мере мне это не кажется идеальным решением. Дело в том, что есть такие категории, которые содержат множество несвязанных прохождений, по одному стриму на каждое (например, "Полные прохождения за один стрим"). Для таких прохождений желательно показывать список стримов на каждой странице.

К категории типа `list` может относиться только одна серия стримов, а остальные будут игнорироваться. Из этой серии стримов берутся все стримы и для каждого выводится своя ссылка на главной странице. Вот, собственно, и всё, что можно сказать об этом типе категорий.

## Стримы (`streams.json`)

Этот файл содержит **объект** JSON (аналог ассоциативного массива в Java или словаря в Python). Каждый ключ -- ID стрима на Twitch. Этот ключ ссылается на другой объект, который содержит дополнительную информацию о стриме: таймкоды, записи и т.д.

```
{
  "0": {},
  "1": {}
}
```

### Объект "Стрим"

```
"225279322": {
  "segments": [
    { "vk": "87862793_456241096" },
    { "vk": "87862793_456241095", "subtitle_offset": "3:46:12" }
  ],
  "note": "Официальной записи не будет",
  "timecodes": [
    ["11:49", "Hunt: Showdown (поиски коопа)"],
    ["24:40", "Hunt: Showdown"],
    ["4:06:04", "S.O.S."]
  ]
}
```

| Поле | Обязательное | Описание |
| ---- | :----------: | -------- |
| ключ | **Да** | Это не поле, но стоит упомянуть его здесь. Каждый стрим начинается с его ID на Twitch. ID состоит только из цифр, но пишется как строка (требование JSON). |
| `note`<sup>**</sup> | Нет | Заметка, которая будет отображаться в информации о стриме на странице прохождения. |
| `youtube`<sup>*</sup> | Нет | ID записи на YouTube. Самый простой и стабильный способ добавить вечную запись стрима в архив. Только ID (например `43q8zNpTCCE`), не весь URL, это важно! |
| `vk`<sup>*</sup> | Нет | ID видео в социальной сети ВКонтакте. В данный момент поддерживаются ID подобного формата: `87862793_456241096`. ВКонтакте требует наличие активного бэкенда, поэтому его стоит избегать, когда это возможно. |
| `direct`<sup>*</sup> | Нет | Прямая ссылка на видео. Поддержка этого поля была добавлена "на всякий случай" и ещё нигде не применялась. |
| `segments`<sup>*, **</sup> | Нет | Содержит массив из объектов "Сегмент" (см. ниже). |
| `timecodes`<sup>**</sup> | Нет | Массив содержащий один или несколько массивов "Таймкод" (см. ниже). |
| `start` | Нет | Таймкод, с которого должно начинаться воспроизведение стрима. |
| `end` | Нет | Таймкод, на котором нужно **прервать** воспроизведение. Это поле использовалось только один раз, когда неофициальная запись была криво склеена со следующей. |
| `subtitle_offset` | Нет | Смещение, применяемое к субтитрам. Полезно в случаях, когда начало записи не совпадает с началом стрима. Пример: стрим `231626788`. |

\* Поле, указывающее на постоянную запись стрима. В объекте должно присутствовать только **одно** поле этого типа.

\*\* Эти поля не должны использовать в объекте "Сегмент" (см. ниже).

### Объект "Сегмент"

```
{ "vk": "87862793_456241095", "subtitle_offset": "3:46:12" }
```

Этот объект похож на объект "Стрим" за исключением того, что в нём не применяются поля, отмеченные \*\*. Для второго и последующих сегментов крайне рекомендуется добавлять поле `subtitle_offset`, так как файл субтитров на все сегменты один, а смещение от начала стрима у них разное.

### Массив "Таймкод"

```
["24:40", "Hunt: Showdown"]
```

| # | Роль | Описание |
| - | ---- | -------- |
| 0 | Время | Строка формата "HH:MM:SS" или "MM:SS" (аналог таймкодов на YouTube). |
| 1 | Текст | Описание этого таймкода, которое будет показано рядом с ссылкой. |

Примечание: Так как это массив, все поля обязательны и не имеют названий.

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
  "filename": "black-mirror.html",
  "streams": [
    { "name": "1", "twitch": "205262158" },
    { "name": "2", "start": "3:07:00", "twitch": "206091904" }
  ]
}
```

| Поле | Обязательное | Описание |
| ---- | :----------: | -------- |
| `name` | **Да** | Название прохождения, которое будет отображаться в списке на главной странице и в заголовке страницы самого прохождения. |
| `category` | **Да** | Код категории, к которой относится это прохождение. |
| `filename` | **Да** | Имя файла, который будет сгенерирован для данного прохождения и помещен в директорию `/links`. Должно обязательно иметь расширение `.html`! |
| `streams` | **Да** | Содержит массив урезанных объектов "Стрим". Может содержать только ключ `twitch`. Дополнительно можно переопределить любой другой параметр объекта "Стрим", если это требуется в контексте прохождения. |
