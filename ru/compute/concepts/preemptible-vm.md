# Прерываемые виртуальные машины

Прерываемые виртуальные машины — это дополнительная опция при создании виртуальной машины, которая позволяет использовать вычислительные ресурсы Яндекс.Облака по более низкой цене. Виртуальные машины с такой опцией будут принудительно остановлены в срок, не превышающий 24 часа с момента их создания. Остановленные виртуальные машины можно запустить повторно, без потери данных.

> [!NOTE]
>
> Прерываемые виртуальные машины могут быть принудительно остановлены в любой момент.

Прерываемые виртуальные машины обходятся дешевле обычных, но они не обеспечивают устойчивости к отказу в работе. Подробнее с тарифами на прерываемые виртуальные машины можно ознакомиться в разделе [[!TITLE]](../pricing.md#prices-preemptible-instance-resources).

Чтобы создать прерываемую виртуальную машину, [следуйте инструкции](../operations/vm-create/create-preemptible-vm.md).

## Ограничения

Прерываемые виртуальные машины работают как обычные виртуальные машины, но имеют следующие ограничения:

- Для прерываемых ВМ не действует соглашение об уровне обслуживания (SLA).
- Прерываемая виртуальная машина может быть принудительно остановлена в любой момент. Вероятность такого события низкая, но может меняться изо дня в день и от зоны доступности к зоне доступности.
- Прерываемую виртуальную машину нельзя сделать обычной, и наоборот.
- Прерываемая виртуальная машина всегда завершает работу в срок, не превышающий 24 часа.