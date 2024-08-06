---
title: Подключение/отключение роутера
description: Описание подключения/отключения роутера
---

# Подключение/отключение роутера

### Панель управления

---

Для того чтобы подключить маршутизатор к сети:

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта и перейдите в раздел **Маршрутизаторы**.
3. Нажмите на имя маршрутизатора и в разделе управления маршрутизатором откройте вкладку **Порты**.
4. Нажмите кнопку **Создать**.
5. Выберите подсеть. в которой необходимо создать порт маршрутизатора.
6. Выберите [группу безопасности](/) для порта.

### OpenStack CLI

1. Выполните [установку](/), если OpenStack CLI не установлен и пройдите [аутентификацию](/) в проекте.
2. Получите список роутеров:

```
openstack router list
```

3. Для подключения роутера к внешней сети:

```
openstack router set <router> --external-gateway "FloatingIP Net"
```

Где:

- `<name>` - имя или ID роутера

4.  Для подключения роутера к сети получите список сетей:

```
openstack network list
```

5. Подключите роутер к сети:

```
openstack router add subnet <router> <network>
```

Где:

- `<router>` - имя или ID роутера.
- `<network>` - имя или ID сети.

### Terraform

---

### Подготовительные шаги:

1. Установите Terraform.
2. Установите [OpenStack provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs)

Для подключения роутера к сети используется следующий ресурс:

```
resource "openstack_networking_router_interface_v2" "router_interface" {
	router_id = openstack_networking_router_v2.router.id
	subnet_id = openstack_networking_subnet_v2.subnet.id
	depends_on = [openstack_networking_router_v2.router]
}
```

Где:

- `router_id` - идентификатор роутера, к которому вы хотите привязать интерфейс.
- `subnet_id` - идентификатор подсети, которую вы хотите присоединить к указанному роутеру.
- `depends_on` - параметр указывает Terraform, что этот ресурс должен быть создан только после того, как будет создан роутер.
