# Передать данные приложения команде {{ tracker-name }}


Передайте [в службу поддержки {{ tracker-name }}](troubleshooting.md) запрос на подключение внешнего приложения и информацию, которая нужна для регистрации вашего приложения в {{ tracker-name }}.


## Обязательные параметры {#required}

Обязательных параметров достаточно, чтобы обеспечить минимальную функциональность связей между задачами в {{ tracker-name }} и объектами в вашем приложении: отображение иконки приложения, ключа объекта и ссылки на объект.

 Параметр | Тип | Описание
 -------- | -------- | ----------
 `id` | Строка | Идентификатор приложения
 `name` | Строка | Имя приложения
 `defaultIconUrl` | Строка | URL иконки для отображения в связях задачи
 `frontUrl` | Строка | Ссылка на объект, с которым создается связь.<br/>Формат: `scheme://host:port*not_var{{remoteKey}}*`<br/>где<br/><ul><li>`remoteKey` — ключ связанного объекта в приложении;</li><li>`*` — любая последовательность символов.</li></ul>Например: `https://my-app.ru:8080/object/1234/`
 `responsible` | Строка | Ответственные за приложение, к которым можно обратиться, если с приложением будут проблемы. 


## Дополнительные параметры {#optional}

Если ваше приложение поддерживает расширенную функциональность связей, например отображение подробной информации об объекте или автоматическое создание связей во внешнем приложении по запросу из {{ tracker-name }}, передайте соответствующие дополнительные параметры приложения.

 Параметр | Тип | Описание
 -------- | -------- | ----------
 `entityInfoUrl` | Строка | URL, который {{ tracker-name }} может вызвать для получения информации о связанном объекте, чтобы отобразить эту информацию в интерфейсе. Подробнее читайте в подразделе [Получение информации о связанном объекте](ext-app-setup.md#object-info).<br/>Формат: `scheme://host:port*not_var{{remoteKey}}*`<br/>где<br/><ul><li>`remoteKey` — ключ связанного объекта в приложении;</li><li>`*` — любая последовательность символов.</li></ul>
 `createLinkUrl` | Строка | URL, который {{ tracker-name }} должен вызвать для создания связи в приложении. Используется при создании связи в {{ tracker-name }}, чтобы продублировать ее в приложении.<br/>Подробнее читайте в подразделе [Создание связей в приложении по запросу из {{ tracker-name }}](ext-app-setup.md#create).
 `removeLinkUrl` | Строка | URL, который {{ tracker-name }} должен вызвать для удаления связи в приложении. Используется при удалении связи в {{ tracker-name }}, чтобы также удалить ее в приложении.<br/>Подробнее читайте в подразделе [Удаление связей в приложении по запросу из {{ tracker-name }}](ext-app-setup.md#delete).<br/>Формат: `scheme://host:port*not_var{{issueKey}}*not_var{{remoteKey}}*`<br/>где<br/><ul><li>`issueKey` — ключ задачи в {{ tracker-name }};</li><li>`remoteKey` — ключ связанного объекта в приложении;</li><li>`*` — любая последовательность символов.</li></ul>
 `canLinkFromFront` | Логический | Параметр включает или отключает возможность создавать связь с внешним приложением из пользовательского интерфейса {{ tracker-name }}.