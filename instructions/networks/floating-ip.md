---
title: Создание плавающего IP
description: Описание создания плавающего IP
---

# Создание плавающего IP

Плавающий IP-адрес (Floating IP) является публичным IP-адресом, доступным из внешней сети, который может быть назначен виртуальной машине в частной сети. Этот механизм позволяет виртуальным машинам в частной сети быть доступными из интернета, обеспечивая возможность взаимодействия с внешними ресурсами.

При создании плавающего IP-адреса происходит резервирование конкретного адреса за Вашим облачным проектом. Даже если этот адрес не привязан непосредственно к какой-либо виртуальной машине, опалата будет назначаться за зарезервированный IP адрес даже если он не используется.

### Панель управления

---

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта и перейдите в раздел **Плавающие IP**.
3. Нажмите кнопку **Создать** и выберите регион для создания плавающего IP.

### OpenStack CLI

---

1. Создайте плавающий IP адрес при помощи следующией команды:

```
openstack floating ip create "FloatingIP Net"
```

2. Проверьте, что IP адрес создался при помощи команды:

```
openstack floating ip list
```

### Terraform

---

Если у вас еще нет Terraform, [установите его и настройте провайдер Openstack](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs)

1. Для добавления плавающего IP адреса в Ваш проект можно использовать следущую конфигурацию

```hcl
data "openstack_networking_network_v2" "external_network" {
	name = "FloatingIP Net"
}

resource "openstack_networking_floatingip_v2" "fip" {
	pool = data.openstack_networking_network_v2.external_network.name
}
```

## Удаление плавающего IP

### Панель управления

---

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта и перейдите в раздел **Плавающие IP**.
3. Откройте констекстное меню `...` необходимого IP-адреса и нажмите **Удалить**.

### OpenStack CLI

---

1. Выполните [установку](/), если OpenStack CLI не установлен и пройдите [аутентификацию](/) в проекте.
2. Получите список плавающих IP:

```
openstack floating ip list
```

3. Удалите плавающий IP:

```
openstack floating ip delete <ip>
```

Где:

- `<ip>` - IP-адрес или ID.

## Подключение плавающего IP к ВМ

### Панель управления

---

1. [Перейдите](https://auth.ps.kz/?required_keys=uid&redirect_uri=https://console.ps.kz/account) в панель управления PS Cloud.
2. Выберите услугу облачных серверов и перейдите в проект облачного сервера.
3. Откройте вкладку **Cерверы**.
4. Измените состояние ВМ одним из способов:

   - Через контекстное меню `...` — для одной ВМ:

     1. В списке виртуальных машин найдите ВМ, состояние которой необходимо изменить.
     2. Откройте контекстное меню `...` справа от названия ВМ.
     3. Выберите действие **Привязать плавающий IP**.
     4. Выберите плавающий IP из списка или выберите новый.
     5. Нажмите кнопку **Подтвердить**.

   - На странице виртуальной машины:
     1. В списке виртуальных машин нажмите на название ВМ, состояние которой необходимо изменить.
     2. Нажмите кнопку **Управление**.
     3. Выберите действие **Привязать плавающий IP**.
     4. Выберите плавающий IP из списка или выберите новый.
     5. Нажмите кнопку **Подтвердить**.

5. После подключения плавающего IP виртуальная машина будет доступна по публичному IP-адресу.

Отключить плавающий IP вы можете с помощью действия **Отвязать плавающий IP**.

## OpenStack CLI

---

1. Выполните [установку](/), если OpenStack CLI не установлен и пройдите [аутентификацию](/) в проекте.

2. Скопируйте IP-адрес:

```shell
openstack floating ip list
```

\
3. Получите ID или имя ВМ к которой требуется подключить плавающий IP-адрес:

```
openstack server list
```

4. Подключите плавающий IP адрес к ВМ:

```shell
openstack server add floating ip <server> <ip-address>
```

Где:

- `<server>` - имя или ID ВМ.
- `<ip-address>` - плавающий IP-адрес для подключения.

Для отключения плавающего IP введите команду:

```shell
openstack server remove floating ip <server> <ip-address>
```

## Terraform

---

```hcl

data "openstack_networking_network_v2" "external_network" {
	name = "FloatingIP Net"
}

resource "openstack_networking_floatingip_v2" "fip" {
	pool = data.openstack_networking_network_v2.external_network.name
}

resource "openstack_compute_floatingip_associate_v2" "fip_association" {
	floating_ip = openstack_networking_floatingip_v2.fip.address
	instance_id = openstack_compute_instance_v2.instance.id
	fixed_ip = openstack_compute_instance_v2.instance.access_ip_v4
}
```

Где:

- `openstack_networking_floatingip_v2` - описание создания плавающего IP.
- `openstack_compute_floatingip_associate_v2` - описание подключения плавающего IP.
