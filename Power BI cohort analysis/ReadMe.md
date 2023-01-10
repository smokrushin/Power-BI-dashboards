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
