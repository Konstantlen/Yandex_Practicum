# Финальный Проект: Анализ оттока клиентов банка «Метанпром»

# Материалы
* Презентация - https://drive.google.com/file/d/10BEL970F96DoQ4egf9qnWHy07WS1uKdV/view?usp=share_link
* Дашборд - https://public.tableau.com/views/_16806142669710/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link

Цель: Проанализировать клиентов регионального банка и выделить сегменты клиентов, которые склонны уходить из банка, предложить рекомендации для удержания клиентов по сегментам.

Задачи: 
- Провести исследовательский анализ данных, включающий исследование корреляций и портретов клиентов
- Выделить портреты клиентов, которые склонны уходить из банка,
- Сформулировать и проверить статистические гипотезы:  
    Проверить гипотезу различия дохода между теми клиентами, которые ушли и теми, которые остались.  
    Сформулировать и проверить статистическую гипотезу относительно представленных данных
- Сделать выводы о том, какие признаки стратегическим образом влияют на отток и какие значения признаков или интервалы этих значений связаны с оттоком.
- Выделить не мелкие, но компактные и высокоточные сегменты, приоритезировать их
- Дать конкретные рекомендации по приоритетным сегментам
- По итогам исследования подготовить презентацию.
- Составить дашборд по полученным в ходе исследования данным

Описание датасета

* Датасет содержит данные о клиентах банка «Метанпром». Банк располагается в Ярославле и областных городах: Ростов Великий и Рыбинск.
* Данные находятся в файле bank_dataset.csv

Колонки:

- `userid` — идентификатор пользователя,
- `score` — баллы кредитного скоринга,
- `City` — город,
- `Gender` — пол,
- `Age` — возраст,
- `Objects` — количество объектов в собственности,
- `Balance` — баланс на счёте,
- `Products` — количество продуктов, которыми пользуется клиент,
- `CreditCard` — есть ли кредитная карта,
- `Loyalty` — активный клиент,
- `estimated_salary` — заработная плата клиента,
- `Churn` — ушёл или нет.
