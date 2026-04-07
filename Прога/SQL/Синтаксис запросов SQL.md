### Зарезервированные слова
[Зарезервированные ключевые слова](https://www.sqlstyle.guide/ru/#reserved-keyword-reference "Reserved keyword reference") всегда пишите прописными буквами, например `SELECT`, `WHERE`.
Не используйте сокращённый вариант ключевого слова, если имеется полный. Например, используйте `ABSOLUTE` вместо `ABS`.
Не используйте специфичные для какого-либо поставщика СУБД ключевые слова, если в ANSI SQL есть ключевые слова, выполняющие такие же функции. Это сделает ваш код более переносимым.
```
SELECT model_num
  FROM phones AS p
 WHERE p.release_date > '2014-09-30';
```

### Пробельные символы
Для лучшей удобочитаемости кода важно правильно использовать пробельные символы. Не нужно нагромождать код или удалять пробелы, присущие естественному языку.

#### Пробелы
Можно и нужно использовать пробелы для выравнивания основных ключевых слов по их правому краю. В типографике получающиеся таким образом «[коридоры](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D1%80%D0%B8%D0%B4%D0%BE%D1%80_\(%D1%82%D0%B8%D0%BF%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D0%BA%D0%B0\) "Коридоры в типографике")» стараются избегать, в то же время в нашем случае они, напротив, помогают лучше вычленять важные ключевые слова.
```
(SELECT f.species_name,
        AVG(f.height) AS average_height, AVG(f.diameter) AS average_diameter
   FROM flora AS f
  WHERE f.species_name = 'Banksia'
     OR f.species_name = 'Sheoak'
     OR f.species_name = 'Wattle'
  GROUP BY f.species_name, f.observation_date)

  UNION ALL

(SELECT b.species_name,
        AVG(b.height) AS average_height, AVG(b.diameter) AS average_diameter
   FROM botanic_garden_flora AS b
  WHERE b.species_name = 'Banksia'
     OR b.species_name = 'Sheoak'
     OR b.species_name = 'Wattle'
  GROUP BY b.species_name, b.observation_date);
```

Обратите внимание, что ключевые слова `SELECT`, `FROM` и т.д. выровнены по правому краю, при этом названия столбцов и различные условия — по левому.
Помимо этого, старайтесь расставлять пробелы:
- **до** и **после** знака равно (`=`)
- **после** запятых (`,`)
- **до** открывающего и **после** закрывающего апострофов (`'`), если последний не внутри скобок, или без последующих запятой или точки с запятой, или не в конце строки
```
SELECT a.title, a.release_date, a.recording_date
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```

#### Переводы строки
Всегда делайте перенос строки:
- **перед** `AND` или `OR`
- **после** точки с запятой (для разделения запросов)
- **после** каждого основного ключевого слова
- **после** запятой (при выделении логических групп столбцов)

Следуя принципу, что ключевые слова выравниваются по правому краю, а всё остальное — по левому, мы добиваемся достаточно удобного расположения частей кода, вследствие чего улучшается зрительная навигация по нему.
```
INSERT INTO albums (title, release_date, recording_date)
VALUES ('Charcoal Lane', '1990-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000'),
       ('The New Danger', '2008-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000');
```

```
UPDATE albums
   SET release_date = '1990-01-01 01:01:01.00000'
 WHERE title = 'The New Danger';
```

```
SELECT a.title,
       a.release_date, a.recording_date, a.production_date -- grouped dates together
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```

### Отступы
Для того, чтобы SQL был удобочитаем, важно также следовать стандартам расстановки отступов.
#### `JOIN`[](https://www.sqlstyle.guide/ru/#join)
Объединения (`JOIN`) должны располагаться по правую часть «коридора». При необходимости между ними можно добавить пустую строку.
```
SELECT r.last_name
  FROM riders AS r
       INNER JOIN bikes AS b
       ON r.bike_vin_num = b.vin_num
          AND b.engine_tally > 2

       INNER JOIN crew AS c
       ON r.crew_chief_last_name = c.last_name
          AND c.chief = 'Y';
```

#### Подзапросы
Подзапросы тоже должны быть выровнены по правому краю «коридора», а внутри них самих применяются те же правила форматирования, что и в любом другом запросе. Если используются вложенные подзапросы, может иметь смысл поставить закрывающую скобку на новой строке ровно под парной ей открывающей скобкой.
```
SELECT r.last_name,
       (SELECT MAX(YEAR(championship_date))
          FROM champions AS c
         WHERE c.last_name = r.last_name
           AND c.confirmed = 'Y') AS last_championship_year
  FROM riders AS r
 WHERE r.last_name IN
       (SELECT c.last_name
          FROM champions AS c
         WHERE YEAR(championship_date) > '2008'
           AND c.confirmed = 'Y');
```

### Формальные тонкости
- **Используйте** `BETWEEN`, где возможно, вместо нагромождения условий `AND`.
- Таким же образом старайтесь **использовать** `IN()` вместо `OR`.
- **Используйте** `CASE`, если значение должно быть интерпретировано до окончания выполнения запроса. С помощью `CASE` можно также формировать сложные логические структуры.
- По возможности **избегайте** использования `UNION` и временных таблиц.
```
SELECT CASE postcode
       WHEN 'BN1' THEN 'Brighton'
       WHEN 'EH1' THEN 'Edinburgh'
       END AS city
  FROM office_locations
 WHERE country = 'United Kingdom'
   AND opening_time BETWEEN 8 AND 9
   AND postcode IN ('EH1', 'BN1', 'NN1', 'KW1');
```

Теги: #SQL 
Ссылки:
[[Стиль написания запросов SQL. Основные положения]]