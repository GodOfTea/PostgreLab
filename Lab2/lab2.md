# Использование FTS в PostgreSQL
#### 1. Тип tsvector
Выполните запрос
```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog.');
```
В ответ будет возвращён список [лексем](https://en.wikipedia.org/wiki/Lexeme)
```sql
                to_tsvector
'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
```

> Задание 1
1. Изучите документацию к функции `to_tsvector`
2. Вызовите эту функцию для следующей строки: `Съешь ещё этих мягких французских булок, да выпей чаю`

<img src = './screens/to_tsvector.png' />

3. Почему в векторе нет слова `да`?
Ответ: Тип tsvector представляет собой что-то вроде нормализованной строки, по которой будет производиться поиск. Под нормализацией понимается выкидывание стоп-слов, таких, как предлоги, обрезание окончаний слов, и так далее. В данном примере слово да, распознается как стоп-слово,
поэтому выкидывается из списка. В предыдущем примере, стоп-словом являлось the, из-за чего оно 
тоже отсутствовало в выводе.

#### 2. Тип tsquery
Выполните по очереди запросы
```sql
--№1
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox');
--№2
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('foxes');
--№3 
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('foxhound');
```

<img src = './screens/3.png' />
<img src = './screens/4.png' />

> Задание 2
1. Что означают символы `@@`
@@ - оператор сопоставления , возвращает true, если tsvector соответствует tsquery
2. Почему второй запрос возвращает `true`, а третий не возвращает
Т.к. лексема слова foxhound, является foxhound. А данной лексемы нет в предложении.
3. Выполните запрос
```sql
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских булок, да выпей чаю.')
    @@ to_tsquery('Russian','булка');
```

<img src = './screens/5.png' />

Почему слово булка не находится?
4. Используйте функцию `select ts_lexize('russian_stem', 'булок');` для того чтобы понять почему.
У данных слов разные лексемы, поэтому лексему булка не нашло.

<img src = './screens/6.png' />
<img src = './screens/7.png' />

5. Замените в предложении слово `булок`, на слово `пирожков`
Выполните запросы
```sql
--№1
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских пирожков, да выпей чаю.')
    @@ to_tsquery('Russian','пирожки');
--№2
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских пирожков, да выпей чаю.')
    @@ to_tsquery('Russian','пирожок');
```

<img src = './screens/8.png' />
<img src = './screens/9.png' />

Почему первый запрос возвращает `true`, а второй не возвращает?
По той же самой причине, у слов разные лексемы.

#### 3. Операторы
Выполните запрос
```sql
--И
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox & dog');

--ИЛИ
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox | rat');

--отрицание
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('!clown');

--группировка
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox & (dog | rat) & !mice');
```

<img src = './screens/10.png' />
<img src = './screens/11.png' />
<img src = './screens/12.png' />
<img src = './screens/13.png' />

> Задание 3
1. Приведите аналогичные запросы для любого предложения на русском

<img src = './screens/o1.png' />
<img src = './screens/o2.png' />
<img src = './screens/o3.png' />
<img src = './screens/o4.png' />

2. Почему для английского языка не нужно указывать язык в первом аргументе и какой анализатор используется если никакой не указан?
Т.К. Первым аргументом по умолчанию устоновленно english и оно будет выставленно, если явно не указать язык. 


#### 4. Поиск фраз
Изучите документацию по [операторам](https://www.postgresql.org/docs/current/functions-textsearch.html) FTS
Выполните запрос
```sql

SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских булок, да выпей чаю.')
    @@ to_tsquery('Russian','мягких<2>булок');
```

<img src = './screens/p1.png' />

> Задание 4
1. Что означает число 2 в операторе `<->`
Данный оператор помогает находить слова, которые идут друг за другом, через n значений, в данном случае n = 2
2. Модифицируйте запрос так, чтобы можно было найти фразу `съешь ещё`

<img src = './screens/p2.png' />

3. Для чего нужно использовать функцию `phraseto_tsquery`
Данная функция приводит запрос к типу данных tsquery. Ну и в документации написано, что он игнорирует пунктуацию

<img src = './screens/p3.png' />

#### 5. Утилиты
1. Приведите примеры использования функций `ts_debug` и  `ts_headline`
ts_debug - отображает информацию о каждом элементе документа.

<img src = './screens/f1.png' />

ts_headline - принимает документ вместе с запросом и возвращает выдержку из документа, в которой выделяются слова из запроса.

<img src = './screens/f2.png' />

Конец!