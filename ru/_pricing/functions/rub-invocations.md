Услуга | Цена за 1 млн вызовов, <br>вкл. НДС
----- | -----
Вызов функций, первые 1 млн в месяц | {{ sku|RUB|serverless.functions.invocations.v1|string }}
Вызов функций, свыше 1 млн в месяц | {{ sku|RUB|serverless.functions.invocations.v1|pricingRate.1|string }}