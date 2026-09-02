# День 7 · Урок 1 — теорія

## Розширені можливості SELECT. Фільтрування: WHERE та все навколо

> **Вчительський файл.** Учнівський файл дня: `day-07-practice.md`.

---

## Мета дня

Учень уміє точно вирізати з таблиці саме той набір рядків, який потрібен:
діапазон, список, шаблон, заперечення, кілька умов разом.

**Після дня учень має вміти:**

- відрізнити `COUNT(*)` від `COUNT(DISTINCT стовпець)`
- шукати за шаблоном через `LIKE` із `%` та `_`
- користуватись `IN`, `BETWEEN`, `NOT`
- правильно розставити дужки, коли в умові є і `AND`, і `OR`
- пояснити, чому `SELECT TOP` у SQL Server, а `LIMIT` у PostgreSQL

**Потрібно:** SSMS із відновленою `AdventureWorksLT`, проектор.

---

## Хронометраж уроку 1

| Хв | Що |
|---|---|
| 0–4 | Повторення: SELECT, FROM, WHERE. Швидко, питаннями |
| 4–14 | `DISTINCT`, `COUNT`, `COUNT(DISTINCT …)`, `TOP`, `SELECT INTO` |
| 14–24 | `WHERE`: значення, діапазон, `AND`/`OR` |
| 24–33 | **`LIKE`, `IN`, `BETWEEN`.** Головна частина |
| 33–40 | `NOT` у всіх варіантах. **Пріоритет `AND` над `OR`** — найважливіше на уроці |
| 40–43 | Діалекти: `TOP` / `LIMIT` / `FETCH FIRST` |
| 43–45 | Анонс практики |

---

## Виклад

### 1. Розширений SELECT (10 хв)

**`SELECT *`** — усі стовпці. Швидко подивитись, але в програмі так не пишуть:
тягне зайві дані, а якщо в таблицю додадуть стовпець, код може зламатись.

**`DISTINCT`** — унікальні значення:

```sql
SELECT DISTINCT City FROM SalesLT.Address;
```

**`COUNT`** — скільки рядків:

```sql
SELECT COUNT(*) FROM SalesLT.Product;
```

**`COUNT(DISTINCT …)`** — скільки різних значень:

```sql
SELECT COUNT(DISTINCT City) FROM SalesLT.Address;
```

> ⚠ **Тут головна плутанина уроку.** Постав питання: у таблиці `Address` 450
> рядків. Чи означає це, що там 450 різних міст? Ні: у місті може бути багато
> адрес. `COUNT(*)` рахує **рядки**, `COUNT(DISTINCT City)` рахує **різні
> міста**. Покажи обидва числа на проекторі поспіль — різниця запам'ятається.

**`SELECT TOP`** — перші рядки:

```sql
SELECT TOP 5 * FROM SalesLT.Product WHERE ListPrice > 2000;
SELECT TOP 10 PERCENT * FROM SalesLT.Product ORDER BY ListPrice DESC;
```

⚠ Без `ORDER BY` «перші 5» означає «які трапились», а не «найдорожчі». Це
частий джерел непорозумінь: `TOP` сам по собі нічого не впорядковує.

**`SELECT INTO`** — створити нову таблицю з результату запиту:

```sql
SELECT * INTO SalesLT.ExpensiveProducts
FROM SalesLT.Product WHERE ListPrice > 2000;
```

Таблиця-приймач мусить **не існувати**. Якщо вона вже є — потрібен
`INSERT INTO … SELECT`, про це буде день 14.

### 2. WHERE: базові варіанти (10 хв)

```sql
-- точне значення
SELECT * FROM SalesLT.Address WHERE PostalCode = '98011';

-- порівняння
SELECT * FROM SalesLT.Product WHERE ListPrice > 1000;

-- дві умови разом
SELECT * FROM SalesLT.Product WHERE ListPrice > 1000 AND Color = 'Black';

-- одна з двох
SELECT * FROM SalesLT.Product WHERE ListPrice > 1000 OR Color = 'Red';
```

Оператори: `=`, `>`, `<`, `>=`, `<=`, `<>` (не дорівнює).

⚠ Текст завжди в одинарних лапках: `'Black'`. Числа без лапок. Український
текст — із `N`: `N'Іваненко'`.

### 3. LIKE, IN, BETWEEN (9 хв, головна частина)

**`LIKE`** — пошук за шаблоном. Два символи-підстановки:

| Символ | Значення |
|---|---|
| `%` | будь-яка кількість символів, зокрема нуль |
| `_` | рівно один будь-який символ |

```sql
WHERE Name LIKE 'Sport%'      -- починається на Sport
WHERE Name LIKE '%sport%'     -- містить sport будь-де
WHERE Name LIKE '%XL'         -- закінчується на XL
WHERE Name LIKE 'Mountain Bike Socks, _'   -- і один довільний символ у кінці
```

**`IN`** — значення входить у список:

```sql
WHERE Color IN ('Black', 'Red', 'Blue')
```

Те саме можна написати через `OR`, але `IN` коротший і читабельніший.

**`BETWEEN`** — діапазон, **включно з межами**:

```sql
WHERE ListPrice BETWEEN 1000 AND 2000
```

⚠ «Включно» варто наголосити: товар за рівно 1000 у результат **потрапить**.

### 4. NOT і пріоритет операторів (7 хв — найважливіше)

`NOT` працює з усім: `NOT LIKE`, `NOT IN`, `NOT BETWEEN`, `NOT Color = 'Black'`.

```sql
WHERE Name NOT LIKE 'Sport%'
WHERE Color NOT IN ('Black', 'Red', 'Blue')
WHERE ListPrice NOT BETWEEN 1000 AND 2000
```

**А тепер головне.** `AND` виконується **раніше** за `OR` — так само, як
множення раніше за додавання. Порівняй два запити:

```sql
-- (A) без дужок: AND зв'язав другу й третю умови
SELECT * FROM SalesLT.Product
WHERE Color = 'White' OR Color = 'Black' AND ListPrice < 1000;

-- (B) з дужками: зовсім інший набір рядків
SELECT * FROM SalesLT.Product
WHERE (Color = 'White' OR Color = 'Black') AND ListPrice < 1000;
```

У (A) білі товари попадають **за будь-якої ціни**. У (B) обмеження ціни діє на
обидва кольори.

🖥 Запусти обидва на проекторі й покажи різницю в кількості рядків. Це найкорисніші
дві хвилини уроку — і саме на цьому спіткнеться половина класу на практиці.

**Правило для запису:** якщо в умові є і `AND`, і `OR` — **став дужки завжди**,
навіть коли впевнений. Дужки безкоштовні, а помилка тиха: запит виконається й
дасть неправильну відповідь без жодної помилки.

### 5. Діалекти (3 хв)

Одна й та сама задача — «дай перші 5 рядків»:

```sql
SELECT TOP 5 * FROM ...                       -- SQL Server
SELECT * FROM ... LIMIT 5;                    -- PostgreSQL, MySQL, SQLite
SELECT * FROM ... FETCH FIRST 5 ROWS ONLY;    -- Oracle
```

Це те, що ми на дні 6 називали діалектами. `WHERE`, `LIKE`, `IN`, `BETWEEN`
однакові всюди — а от обмеження кількості рядків у кожного своє. Повний огляд
буде на дні 17.

---

## ⚠ Що піде не так

| Симптом | Причина | Що робити |
|---|---|---|
| Результат правильний на вигляд, але рядків не стільке | `AND` і `OR` без дужок | Розділ 4. Правило: змішав — став дужки |
| `LIKE 'sport%'` нічого не знайшов | шукав на початку, а слово в середині | `'%sport%'` |
| `BETWEEN` не включив межу | включає; проблема в іншому | перевірити тип: текст порівнюється не як число |
| `TOP 5` дає не найдорожчі | немає `ORDER BY` | додати сортування |
| `SELECT INTO` дає помилку | таблиця вже існує | інша назва або `DROP TABLE` спершу |
| Порівняння з `NULL` не працює | `= NULL` завжди хибне | `IS NULL` — детально на дні 8 |

---

## Урок 2 — практика: відповіді й критерії

Учнівський файл: `day-07-practice.md`. Файл здачі — `day07.sql`.

> ⚠ Числа рядків — для стандартної `AdventureWorksLT`. Прогони перед уроком.

### Відповіді

**Крок 1. Розминка**

```sql
SELECT COUNT(*) FROM SalesLT.Address;                  -- 450
SELECT COUNT(DISTINCT City) FROM SalesLT.Address;      -- 269
SELECT COUNT(DISTINCT CountryRegion) FROM SalesLT.Address;  -- 6
```

Питання «чому перше число більше за друге» — ключове. Відповідь: в одному місті
кілька адрес.

**Крок 2. Ціни**

```sql
SELECT Name, ListPrice FROM SalesLT.Product WHERE ListPrice > 2000;
SELECT Name, ListPrice FROM SalesLT.Product WHERE ListPrice BETWEEN 500 AND 1000;
SELECT Name, ListPrice FROM SalesLT.Product WHERE ListPrice NOT BETWEEN 500 AND 1000;
```

Сума двох останніх мусить дати 295 (усі товари) — **якщо в жодного товару ціна
не `NULL`**. У цій базі так і є. Якщо в іншій базі числа не зійдуться, причина
буде саме в `NULL`, і це відмінний привід забігти на день 8.

**Крок 3. Кольори**

```sql
SELECT Name, Color FROM SalesLT.Product WHERE Color IN ('Black','Red','Blue');
SELECT Name, Color FROM SalesLT.Product WHERE Color NOT IN ('Black','Red','Blue');
```

⚠ **Тут закладена головна пастка практики.** Сума двох запитів **не** дасть 295,
бо в частини товарів `Color` дорівнює `NULL`, а `NOT IN` рядки з `NULL` не
повертає. Учень, який це помітив і написав про це — розуміє `NULL` ще до того,
як ми його проходили. Це рівень 12 балів.

**Крок 4. LIKE**

```sql
WHERE Name LIKE 'Mountain%'
WHERE Name LIKE '%Frame%'
WHERE Name LIKE '%, L'          -- закінчується розміром L
WHERE Name LIKE 'HL Road Frame_%'
```

**Крок 5. AND і OR**

Обидва варіанти з розділу 4 виклада. Учень мусить отримати **різні числа** й
письмово пояснити, чому. Це головний крок практики; якщо учень зробив лише
його — це вже 7 балів.

**Крок 6. TOP і копія таблиці**

```sql
SELECT TOP 10 Name, ListPrice FROM SalesLT.Product ORDER BY ListPrice DESC;
SELECT TOP 5 PERCENT Name, ListPrice FROM SalesLT.Product ORDER BY ListPrice DESC;

SELECT * INTO SalesLT.MyExpensive
FROM SalesLT.Product WHERE ListPrice > 2000;
```

Після `SELECT INTO` учень мусить знайти нову таблицю в дереві (`Refresh` на
папці `Tables`) і потім видалити її через `DROP TABLE`.

### Оцінювання

| Бали | Ознаки |
|---|---|
| 1–3 | Кроки 1–2 за зразком |
| 4–6 | Кроки 1–4; `LIKE` працює хоча б у двох варіантах |
| 7–9 | Кроки 1–5; **різницю `AND`/`OR` показано й пояснено письмово**; числа в коментарях |
| 10–12 | Усе + помічено проблему з `NOT IN` і `NULL` (крок 3), або зіркове завдання |

### Що зібрати

`day07.sql` із коментарями-номерами й числами. Знадобиться на дні 8.
