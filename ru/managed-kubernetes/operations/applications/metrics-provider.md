# Установка Metrics Provider

Metrics Provider — это связующий элемент между объектом в кластере {{ managed-k8s-name }} и сервисом [{{ monitoring-full-name }}](../../../monitoring/concepts/index.md).

Провайдер преобразует запрос на получение внешних метрик от объекта в кластере {{ k8s }} в нужный {{ monitoring-full-name }} формат, а также выполняет обратное преобразование — от {{ monitoring-full-name }} до объекта кластера.

Чтобы установить Metrics Provider:

1. [Создайте сервисный аккаунт](#create-sa-key).
1. [Установите провайдер с помощью {{ marketplace-full-name }}](#marketplace-install).

## Создание сервисного аккаунта и статического ключа доступа {#create-sa-key}

Для работы провайдера нужно создать [сервисный аккаунт](../../../iam/concepts/users/service-accounts.md) и получить для него ключ.

1. Установите [утилиту потоковой обработки JSON-файлов `jq`](https://stedolan.github.io/jq/):

    ```bash
    sudo apt update && sudo apt install jq
    ```

1. [Создайте сервисный аккаунт](../../../iam/operations/sa/create.md) с ролью `monitoring.viewer`.

1. Создайте ключ для сервисного аккаунта и сохраните его на локальный компьютер:

    ```bash
    yc iam key create \
       --service-account-id <идентификатор сервисного аккаунта> \
       --folder-id <идентификатор каталога> \
       --cloud-id <идентификатор облака> \
       --description metrics-provider  \
       --format json \
       -o key.json
    ```

    Ожидаемый результат выполнения команды:

    ```text
    {
      "id": "<идентификатор ключа сервисного аккаунта>",
      "service_account_id": "<идентификатор сервисного аккаунта>",
      "created_at": "2022-01-27T03:29:45.139311367Z",
      "description": "metrics-provider",
      "key_algorithm": "RSA_2048"
    }
    ```

    {% note info %}

    Сохраните идентификаторы сервисного аккаунта и его ключа — они понадобятся при дальнейшей установке.

    {% endnote %}

1. Сохраните ключ сервисного аккаунта в формате Base64:

    ```bash
    jq -r .private_key key.json > key.pem
    ```

## Установка с помощью {{ marketplace-full-name }} {#marketplace-install}

1. Перейдите на страницу каталога и выберите сервис **Managed Service for Kubernetes**.
1. Нажмите на имя нужного кластера и выберите вкладку **Marketplace**.
1. В разделе **Доступные для установки приложения** выберите **Metrics Provider** и нажмите кнопку **Использовать**.
1. Задайте настройки приложения:

    * **Пространство имен** — выберите [пространство имен](../../concepts/index.md#namespace) или создайте новое.
    * **Название приложения** — укажите название приложения.
    * **Идентификатор каталога** — укажите [идентификатор каталога](../../../resource-manager/concepts/resources-hierarchy.md#folder), в котором будет работать Metrics Provider.
    * **Ширина временного окна** — укажите ширину временного окна, за которую будут собираться метрики (в формате `DdHhMmSs`, например `5d10h30m20s`).
    * (опционально) **Отключение прореживания** — выберите эту опцию, чтобы не применять к данным [функцию прореживания](../../../monitoring/concepts/decimation.md).
    * (опционально) **Функция агрегации** — выберите [функцию агрегации](../../../monitoring/concepts/querying.md#combine-functions) данных. Значение по умолчанию — `AVG`.
    * (опционально) **Заполнение данных** — выберите настройки заполнения пропусков в данных:
        * `NULL` — возвращает `null` в качестве значения метрики и `timestamp` в качестве временной метки. Значение по умолчанию.
        * `NONE` — не возвращает значений.
        * `PREVIOUS` — возвращает значение из предыдущей точки.
    * (опционально) **Максимальное количество точек** — укажите максимальное количество точек, которое будет получено в ответе на запрос. Значение параметра должно быть больше `10`.
    * (опционально) **Ширина временного окна прореживания** — укажите ширину временного окна (сетки) в миллисекундах. Используется для прореживания: точки внутри окна объединяются в одну при помощи функции агрегации. Значение параметра должно быть больше `0`.

        {% note info %}

        Выберите только одну из настроек **Максимальное количество точек** или **Ширина временного окна прореживания**. Чтобы не использовать эти настройки, оставьте оба поля пустыми. Подробнее см. в [документации API](../../../monitoring/api-ref/MetricsData/read.md).

        {% endnote %}

    * **ID сервисного аккаунта** — укажите идентификатор [созданного ранее](#create-sa-key) сервисного аккаунта.
    * **ID ключа сервисного аккаунта** — укажите идентификатор ключа сервисного аккаунта.
    * **Приватный ключ сервисного аккаунта** — скопируйте в это поле содержимое файла `key.pem`.

1. Нажмите кнопку **Установить**.
