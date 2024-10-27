# goit-rdb-fp

## Завдання 1

Завантажте дані:

- Створіть схему `pandemic` у базі даних за допомогою SQL-команди.
- Оберіть її як схему за замовчуванням за допомогою SQL-команди.
- Імпортуйте дані `infectious_cases.csv` за допомогою Import wizard.

## Розвязання завдання 1

![task1](./1/p1.png)

```sql
CREATE DATABASE IF NOT EXISTS pandemic;

 USE pandemic;

 SELECT * FROM pandemic.infectious_cases;
```

## Завдання 2

Нормалізуйте таблицю `infectious_cases` до 3ї нормальної форми. Збережіть у цій же схемі дві таблиці з нормалізованими даними.

## Розвязання завдання 2

![task2](./2/p2.png)

```sql
 USE pandemic;

CREATE TABLE entity (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(45),
    code VARCHAR(10)
);

INSERT INTO entity (name, code)
SELECT DISTINCT Entity, Code
FROM infectious_cases;

CREATE TABLE disease_cases (
    id INT PRIMARY KEY AUTO_INCREMENT,
    entity_id INT,
    Year INT,
    Number_yaws INT,
    polio_cases INT,
    cases_guinea_worm INT,
    Number_rabies DOUBLE,
    Number_malaria DOUBLE,
    Number_hiv DOUBLE,
    Number_tuberculosis DOUBLE,
    Number_smallpox INT,
    Number_cholera_cases INT,
    FOREIGN KEY (entity_id) REFERENCES entity(id)
);

INSERT INTO disease_cases (
    entity_id,
    Year,
    Number_yaws,
    polio_cases,
    cases_guinea_worm,
    Number_rabies,
    Number_malaria,
    Number_hiv,
    Number_tuberculosis,
    Number_smallpox,
    Number_cholera_cases
)
SELECT
    (SELECT id FROM entity WHERE entity.name = infectious_cases.Entity),
    Year,
    NULLIF(Number_yaws, '') AS Number_yaws,
    NULLIF(polio_cases, '') AS polio_cases,
    NULLIF(cases_guinea_worm, '') AS cases_guinea_worm,
    NULLIF(Number_rabies, '') AS Number_rabies,
    NULLIF(Number_malaria, '') AS Number_malaria,
    NULLIF(Number_hiv, '') AS Number_hiv,
    NULLIF(Number_tuberculosis, '') AS Number_tuberculosis,
    NULLIF(Number_smallpox, '') AS Number_smallpox,
    NULLIF(Number_cholera_cases, '') AS Number_cholera_cases
FROM infectious_cases;
```

## Завдання 3

Проаналізуйте дані:

- Для кожної унікальної комбінації `Entity` та `Code` або їх `id` порахуйте середнє, мінімальне, максимальне значення та суму для атрибута `Number_rabies`.

💡 Врахуйте, що атрибут `Number_rabies` може містити порожні значення `‘’` — вам попередньо необхідно їх відфільтрувати.

- Результат відсортуйте за порахованим середнім значенням у порядку спадання.
- Оберіть тільки 10 рядків для виведення на екран.

## Розвязання завдання 3

![task3](./3/p3.png)

```sql
SELECT
    entity_id,
    AVG(Number_rabies) AS avg_rabies,
    MIN(Number_rabies) AS min_rabies,
    MAX(Number_rabies) AS max_rabies,
    SUM(Number_rabies) AS sum_rabies
FROM
    disease_cases
WHERE
    Number_rabies IS NOT NULL
GROUP BY
    entity_id
ORDER BY
    avg_rabies DESC
LIMIT 10;
```

## Завдання 4

Побудуйте колонку різниці в роках.

Для оригінальної або нормованої таблиці для колонки `Year` побудуйте з використанням вбудованих SQL-функцій:

- атрибут, що створює дату першого січня відповідного року,

💡 Наприклад, якщо атрибут містить значення ’1996’, то значення нового атрибута має бути ‘1996-01-01’.

- атрибут, що дорівнює поточній даті,
- атрибут, що дорівнює різниці в роках двох вищезгаданих колонок.
  💡 Перераховувати всі інші атрибути, такі як Number_malaria, не потрібно.

## Розвязання завдання 4

![task4](./4/p4.png)

```sql
SELECT
    Year,
    DATE(CONCAT(Year, '-01-01')) AS first_year_date,
    CURDATE() AS today_date,
    TIMESTAMPDIFF(YEAR, DATE(CONCAT(Year, '-01-01')), CURDATE()) AS year_difference
FROM
    disease_cases;
```

## Завдання 5

Побудуйте власну функцію.

- Створіть і використайте функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → ‘1996-01-01’).

## Розвязання завдання 5

![task5](./5/p5.png)

```sql
DELIMITER //

CREATE FUNCTION year_difference(input_year INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE diff INT;
    SET diff = TIMESTAMPDIFF(YEAR, DATE(CONCAT(input_year, '-01-01')), CURDATE());
    RETURN diff;
END //

DELIMITER ;
```
