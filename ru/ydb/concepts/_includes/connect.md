---
sourcePath: ru/ydb/ydb-docs-core/ru/core/concepts/_includes/connect.md
---
# Соединение с базой данных

{% include [overlay/ui.md](connect_overlay/ui.md) %}

Для соединения с БД YDB из YDB CLI или приложения, использующего YDB SDK, вам потребуется предоставить следующие данные:

1. Эндпоинт
3. Аутентификационная информация
2. Размещение базы данных

## Эндпоинт {#endpoint}

Эндпоинт (`endpoint`) - строка в формате `protocol://host:port`, предоставляемая владельцем кластера YDB для корректной маршрутизации запросов клиента к его базам данных через сетевую инфраструктуру, и установки сетевого соединения. Для облачных баз данных `endpoint` показывается в консоли управления на странице требуемой БД, а также обычно может быть получено через CLI облачного поставщика. В корпоративных окружениях имена эндпоинтов YDB предоставляются командой администраторов, или также могут быть получены в консоли управления внутренним облаком.

{% include [overlay/endpoint.md](connect_overlay/endpoint.md) %}

Примеры:
- `grpc://localhost:7135` - протокол обмена данными без шифрования (gRPC), сервер запущен на том же хосте что и клиент, принимает соединения на порту 7135
- `grpcs://ydb.somecorp-int.ru` - протокол обмена данными с шифрованием (gRPCs), сервер запущен на хосте ydb.somecorp-int.ru в изолированной корпоративной интрасети компании "SomeCorp", принмимает соединения на порту YDB по-умолчанию 2135
- `grpcs://ydb.serverless.yandexcloud.net:2135` - протокол обмена данными с шифрованием (gRPCs), публичный сервер Serverless YDB Yandex.Cloud ydb.serverless.yandexcloud.net, порт 2135

{% include [overlay/endpoint_example.md](connect_overlay/endpoint_example.md) %}

## Аутентификационная информация {#auth}

После успешной установки сетевого соединения сервер принимает к обработке запросы от клиента, в которых передается аутентификационная информация. На её основании сервер определяет учетную запись (аккаунт) клиента и проверяет доступ на выполнение запроса.

Основная схема работы предполагает наличие отдельной системы управления учетными данными и доступами - [IAM (Identity and Access Management)](https://en.wikipedia.org/wiki/Identity_management). IAM выдает привязанный к учетной записи токен, который передается клиентом YDB на сервер вместе с запросом. Сервер YDB обращается к IAM для проверки токена и доступа на выполнение запрошенной операции, и кеширует результат.

Также YDB поддерживает аутентификацию по логину и паролю без взаимодействий с IAM, но практическое применение такой схемы ограничено простыми конфигурациями, в первую очередь локальными.

### Режимы аутентификации {#auth-modes}

Клиент YDB (CLI или SDK) поддерживают следующие режимы выбора токена для передачи в запросах:

- **Anonymous** - в запросах передается пустой токен
- **Access Token** - параметром для клиента (SDK или CLI) задается фиксированный токен, который передается в запросы
- **Refresh Token** - параметром для клиента (SDK или CLI) задается [OAuth токен](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/) персональной учетной записи, на основании которого клиент периодически в фоновом режиме обращается к IAM API для ротации (получения следующего) токена, передаваемого в запросы.
- **Service Account Key** - параметром для клиента (SDK или CLI) задаются атрибуты сервисной учетной записи и ключ для подписи, на основании которых клиент периодически в фоновом режиме обращается к IAM API для ротации (получения следующего) токена, передаваемого в запросы
- **Metadata** - клиент (SDK или CLI) периодически обращается к локальному сервису для ротации (получения следующего) токена, передаваемого в запросы

Любой обладатель валидного токена может получить доступ на выполнение операций, поэтому главная задача системы безопасности -- обеспечение секретности токена и предотвращение его компрометации.

Режимы аутентификации с ротацией токена **Refresh Token** и **Service Account Key** обеспечивают больший уровень безопасности по сравнению с режимом с фиксированным токеном **Access Token**, так как на сервер YDB по сети передаются только короткоживущие секреты.

Максимальная безопасность и производительность обеспечивается при использовании режима **Metadata**, так как он исключает необходимость работы с секретами при развертывании приложения, а также позволяет обратиться к IAM и закешировать токен заранее, до запуска приложения.

При выборе режима аутентификации среди поддерживаемых сервером и окружением, следует руководствоваться следующими рекомендациями:

- **Anonymous** обычно применяется на самостоятельно разворачиваемых локальных кластерах YDB, недоступных по сети
- **Access Token** применяется при отсутствии поддержки других режимов на стороне сервера, или в настроечных/отладочных целях. Он не требует взаимодействий клиента с IAM. Однако, если IAM поддерживает API для ротации токенов, то обычно выдываемые таким IAM фиксированные токены имеют короткий срок жизни, что вынуждает регулярно обновлять их в IAM вручную заново.
- **Refresh Token** может применяться при выполнении разовых ручных операций под персональной учетной записью, например, связанных с обслуживанием данных в БД, выполнением ad-hoc операций в CLI, или запусками приложений с рабочей станции. Такой токен можно получить вручную в IAM один раз на долгий срок, и сохранить в переменной окружения на личной рабочей станции для автоматического применения при запуске CLI без дополнительных параметров аутентификации.
- **Service Account Key** применяется в первую очередь для приложений, рассчитанных на эксплуатацию в окружениях, где поддерживается режим **Metadata**, при их тестировании вне таких окружений (например, на рабочей станции). Также он может применяться для приложений вне таких окружений, работая как аналог **Refresh Token** для сервисных учетных записей.
- **Metadata** применяется при разворачивании приложений в облаках. В настоящее время этот режим поддерживается на виртуальных машинах и в Cloud Functions Yandex.Cloud. 

Токен для указания в параметрах может быть получен в системе IAM, с которой связана конкретная установка YDB. В частности, для сервиса YDB в Yandex.Cloud применяется Yandex.Passport OAuth и севисные аккаунты Yandex.Cloud. При использовании YDB в корпоративных контекстах могут применяться стандартные для данной организации системы централизованной аутентификации.

{% include [overlay/auth_choose.md](connect_overlay/auth_choose.md) %}

При использовании режимов, предусматривающих обращение клиента YDB к IAM, дополнительно может быть задан URL IAM, предоставляющий API выдачи токенов. По-умолчанию в существующих SDK и CLI производится попытка обращения к API IAM Yandex.Cloud, размещенному на `iam.api.cloud.yandex.net:443`.

## Размещение базы данных {#database}

Размещение базы данных (`database`) - строка, определяющая где на кластере YDB находится база данных, к которой адресован запрос. Имеет структуру [пути к файлу](https://ru.wikipedia.org/wiki/Путь_к_файлу), с использованием символа `/` в качестве разделителя. Начинается всегда с символа `/`.

На кластере YDB может быть развернуто множество баз данных, и их размещение определяется конфигурацией кластера. Как и эндпоинт, для облачных баз данных `database` показывается в консоли управления на странице требуемой БД, а также обычно может быть получено через CLI облачного поставщика.

При использовании облачных решений базы данных на кластере YDB создаются и размещаются в режиме самообслуживания, не требуя участия владельца и администраторов кластера.

{% include [overlay/database.md](connect_overlay/database.md) %}

{% note warning %}

Приложения не должны как-либо интерпретировать количество и значение каталогов в `database`, так как они определяются конфигурацией кластера YDB. Например, при работе с сервисом YDB в Yandex.Cloud `database` сейчас имеет структуру `region_name/cloud_id/database_id`, однако этот формат может быть изменен в будущем для новых БД.

{% endnote %}

Примеры:
- `/ru-central1/b1g8skpblkos03malf3s/etn01q5ko6sh271beftr` -- база данных Yandex.Cloud с идентификатором `etn01q3ko8sh271beftr` в облаке `b1g8skpbljhs03malf3s`, развернутая в регионе `ru-central1`
- `/local` -- база данных по-умолчанию при самостоятельном развертывании [с использованием Docker](../../getting_started/ydb_docker.md)

{% include [overlay/database_example.md](connect_overlay/database_example.md) %}

## Конфигурация параметров соединения на клиенте {#client-config}

В специализированных статьях описано как определять параметры соединения на клиенте:

* [Соединение с БД и аутентификация в YDB CLI](../../reference/ydb-cli/connect.md)
* [Аутентификация в YDB SDK](../../reference/ydb-sdk/auth.md)

## Дополнительная информация {#addition}

### Корневой сертификат для TLS {#tls-cert}

При использовании протокола с шифрованием ([gRPC over TLS](https://grpc.io/docs/guides/auth/), или grpcs) продолжение сетевого соединения возможно только в том случае, если клиент уверен в том, что ему действительно отвечает подлинный сервер, с которым он пытается установить соединение, а не кто-то посередине, перехвативший запрос в сети. Это обеспечивается проверкой через [цепочки доверия](https://ru.wikipedia.org/wiki/Цепочка_доверия), для работы которой на клиенте должен быть установлен т.н. корневой сертификат.

В состав операционных систем, на которых запускается клиент, уже входит набор корневых сертификатов от основных мировых центров сертификации. Однако, владелец кластера YDB может использовать свой центр сертификации, не связанный ни с одним из мировых, что часто встречается в корпоративных окружениях, и почти всегда применяется при самостоятельном развертывании кластеров с поддержкой шифрования на соединениях. В этом случае владелец кластера должен каким-либо образом передать свой корневой сертификат для использования на стороне клиента. Такой сертификат может быть установлен в хранилище сертификатов операционной системы, где запускается клиент (вручную пользователем или корпоративной командой администраторов ОС), или встроен в состав самого клиента (как это сделано для Yandex.Cloud в YDB CLI и SDK).

### API получения токенов IAM {#token-refresh-api}

Для ротации токенов YDB SDK и CLI используют gRPC-запрос к Yandex.Cloud IAM API [IamToken - create](https://cloud.yandex.ru/docs/iam/api-ref/grpc/iam_token_service#Create). В режиме **Refresh Token** заданный параметром OAuth токен передается в атрибуте `yandex_passport_oauth_token`. В режиме **Service Account Key** на основании заданных атрибутов сервисной учетной записи и ключа шифрования формируется короткоживущий JWT-токен, и передается в атрибуте `jwt`. Исходный код формирования JWT-токена доступен в составе SDK (например, метод `get_jwt()` в [коде на Python](https://github.com/ydb-platform/ydb-python-sdk/blob/main/ydb/iam/auth.py)).

YDB SDK и CLI позволяют подменить хост для обращения к API получения токенов, что дает возможность реализовать аналогичный API в корпоративных контекстах.