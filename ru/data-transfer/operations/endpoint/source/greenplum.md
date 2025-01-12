# Настройка эндпоинта-источника {{ GP }}

При [создании](../index.md#create) или [изменении](../index.md#update) эндпоинта вы можете задать:

* Настройки подключения к [пользовательской инсталляции](#on-premise), в т. ч. на базе виртуальных машин {{ compute-full-name }}. Эти параметры обязательные.
* [Дополнительные параметры](#additional-settings).

## Пользовательская инсталляция {#on-premise}

Подключение к БД с явным указанием сетевых адресов и портов.

{% list tabs %}

- Консоль управления

    {% include [On premise Greenplum UI](../../../../_includes/data-transfer/necessary-settings/ui/on-premise-greenplum.md) %}

    {% include [Common Greenplum table list UI](../../../../_includes/data-transfer/necessary-settings/ui/greenplum-table-list.md) %}

{% endlist %}

## Дополнительные настройки {#additional-settings}

{% list tabs %}

- Консоль управления

    * **Ограничение на размер буфера** — ограничение потребления памяти трансфером (в байтах). Чем больше размер буфера, тем быстрее выполняется перенос данных, но возникает риск аварийного завершения работы трансфера из-за исчерпания доступного объёма памяти. Рекомендуем использовать значение по умолчанию.

    * **Схема служебных объектов** — схема, которая будет использоваться для размещения служебных объектов трансфера.

{% endlist %}

## Особенности работы с источником Greenplum {#shard-copy-settings}

При запуске с настройками шардированного копирования по умолчанию (шардированное копирование отключено, т. е. количество инстансов и процессов не указано) сервис выполняет копирование, взаимодействуя только с координатором кластера {{ GP }}. Доступ к копируемым таблицам осуществляется в [режиме блокировки](https://docs.vmware.com/en/VMware-Tanzu-Greenplum/6/greenplum-database/GUID-ref_guide-sql_commands-LOCK.html) `ACCESS SHARE`.

При запуске с установленными настройками шардированного копирования сервис выполняет копирование, взаимодействуя как с координатором кластера {{ GP }}, так и с его сегментами. Доступ к копируемым таблицам осуществляется в [режиме блокировки](https://docs.vmware.com/en/VMware-Tanzu-Greenplum/6/greenplum-database/GUID-ref_guide-sql_commands-LOCK.html) `SHARE`. При прерывании транзакции на координаторе кластера Greenplum консистентность скопированных данных не гарантируется.

{% include [greenplum-trademark](../../../../_includes/mdb/mgp/trademark.md) %}
