# Когортный анализ на базе датасета Microsoft Contoso

*[Ссылка](https://www.microsoft.com/en-us/download/details.aspx?id=18279) на датасет*

## Схема базы данных
![db_schema](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Power%20BI%20cohort%20analysis/cohorts_db_schema.png)

На входе имеем классическую "снежинку" с таблицей фактов **Sales** и таблицами измерений.
В качестве подготовки для отображения данных по когортам выполним следующие действия:

### 1. Создадим таблицу **UserData**, которая содержит id покупателя, дату и месяц первой покупки, а также суммарное количество покупок пользователя.

Код создания вычисляемой таблицы:

```
UserData = 
SUMMARIZECOLUMNS('Sales'[CustomerKey],
"Количество транзакций", DISTINCTCOUNT('Sales'[OnlineSalesKey]),
"Дата первой транзакции", MIN('Sales'[Order Date]))
```

Добавим столбец с месяцем первой покупки:

`Месяц первой транзакции = FORMAT('UserData'[Дата первой транзакции], "YYYY-MM")`

### 2. Добавим в таблицу Sales следующие столбцы

Номер транзакции для пользователя:

```
Номер транзакции у клиента = 
VAR tid = 'Sales'[OnlineSalesKey]
VAR uid = 'Sales'[CustomerKey]
RETURN
  CALCULATE(
  COUNTROWS('Sales'),
  FILTER(ALL('Sales'), 'Sales'[CustomerKey] = uid && 'Sales'[OnlineSalesKey] <= tid))
```
    
Количество дней, прошедших с первой покупки пользователя:

```
Дней с первой покупки =
'Sales'[Order Date] - RELATED('UserData'[Дата первой транзакции])
```

Количество дней, прошедших с первой покупки пользователя, округленное вверх кратно 30:

```
дней30 с первой покупки = CEILING('Sales'[Дней с первой покупки], 30)
```

### 3. Напишем следующие меры перед построением визуализаций:

Выручка каждого заказа:

```
Выручка = 
SUMX('Sales', 'Sales'[Quantity]*Sales[Net Price])
```

Все покупки пользователя

```
Все покупки пользователя = SUM('Sales'[Выручка])
```

Количество пользователей

```
Количество пользователей = DISTINCTCOUNT('Sales'[CustomerKey])
```

Доля пользователей от общего количества

```
Доля пользователей = 
DIVIDE(
  DISTINCTCOUNT('Sales'[CustomerKey]), 
  COUNTROWS(SUMMARIZE('UserData', UserData[CustomerKey], UserData[Месяц первой транзакции])), BLANK())
```

### 4. Построим визуализацию типа матрица выручки пользователей по месячным когортам

![cohorts_total_income](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Power%20BI%20cohort%20analysis/cohorts_total_income.png)

В строки матрицы добавим значение столбца *Месяц первой транзакции* таблицы *UserData*, 
в столбцы - столбец *дней30 с первой покупки* таблицы *Sales*,
в качестве значений используем меру *все покупки пользователя*

### 5. Построим визуализацию типа матрица количества пользователей, сделавших заказы по месячным когортам

![cohorts_users_count](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Power%20BI%20cohort%20analysis/cohorts_users_count.png)

В строки матрицы добавим значение столбца *Месяц первой транзакции* таблицы *UserData*, 
в столбцы - столбец *дней30 с первой покупки* таблицы *Sales*,
в качестве значений используем меру *количество пользователей*

### 6. Построим визуализацию типа матрица со значениями Retention Rate для месячных когорт пользователей

![cohorts_retention_rate](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Power%20BI%20cohort%20analysis/cohorts_retention_rate.png)

В строки матрицы добавим значение столбца *Месяц первой транзакции* таблицы *UserData*, 
в столбцы - столбец *дней30 с первой покупки* таблицы *Sales*,
в качестве значений используем меру *доля пользователей*

#### [Ссылка](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Power%20BI%20cohort%20analysis/cohorts.pbix) на Power BI исходник
