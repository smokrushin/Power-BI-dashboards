## Личные финансы

### Источник данных

В качестве источника данных взят csv файл cо сгенерированными данными о доходах, расходах и инвестициях.
Под такой формат подходят выписки по банковским картам

![dataset](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/dataset.png)

Для наглядности и раскрытия возможностей инструмента используются данные за 2 полных года.

Схема базы данных максимально простая, к исходной таблице джойним календарь с датами.

![db](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/db_personal_finance.png)

Итоговый дашборд выглядит следующим образом:

![dashboard](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/personal_finance.png)

В фактоиды вынесены суммарные доходы, расходы, инвестиции, норма сбережений и остаток свободных денег.

![factoids](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/factoids.png)

Меры для отображения данных в фактоидах:

Доходы

```
Доходы = CALCULATE(SUM(raw[сумма]), raw[тип]="доход")
```

Расходы

```
Расходы = CALCULATE(SUM(raw[сумма]), raw[тип]="расход")
```

Инвестиции

```
Инвестиции = CALCULATE(SUM(raw[сумма]), raw[тип]="инвестиции")
```

Норма сбережений

```
Норма сбережений = IF(ISBLANK([Инвестиции]), 0, [Инвестиции]) / [Доходы]
```

Остаток свободных денег

```
Остаток = [Доходы] - [Расходы] - [Инвестиции]
```

В качестве фильтра дат используется модуль *Timeline*, позволяющий переключать детализацию на уровне месяцев, кварталов или лет.

![timeline](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/timeline.png)

На графике динамики доходов и расходов используем комбинацию барчартов и лайнчарта. Единица измерения во времени - месяц. График кликабелен, при выборе конкретного месяца остальные визуализации перестраиваются.

![income_expenses](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/income_expenses.png)

Визуализация динамики нормы сбережений исполнена в виде лайнчарта с закрашенной областью. Позволяет увидеть как менялся процент средств, отправляемых на инвестиции во времени.

![savings_rate](https://github.com/smokrushin/Power-BI-dashboards/blob/main/Powe%20BI%20personal%20finance/savings_rate.png)
