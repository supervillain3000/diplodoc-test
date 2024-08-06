---
title: Подключение/отключение порта
description: Описание подключения/отключения порта
---

# Подключение/отключение порта

## Подключение порта к ВМ

### Панель управления

---

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Серверы**.
3. Нажмите на название необходимой ВМ и откройте раздел **Сеть**.
4. Нажмите кнопку **Добавить**.
5. Вы можете выбрать создание нового порта в необходимой подсети со случайным IP-адресом, или подключить созданный ранее порт.
6. Выберите порт и нажмите кнопку **Применить**.

### OpenStack CLI

---

1. Получите ID виртуальной машины, которой необходимо подключить порт:

```
openstack server list
```

2. Получите список портов и выберите необходимый:

```
openstack port list
```

4. Подключите созданный порт к ВМ:

```
openstack server add port <server> <port>
```

Где:

- `<server>` - имя или название ВМ.
- `<port>` - ID порта.

5. Проверьте наличие порта у ВМ:

```
openstack port list --server <server>
```

## Отключение порта от ВМ

### Панель управления

---

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Серверы**.
3. Нажмите на название необходимой ВМ и откройте раздел **Сеть**.
4. Откройте контекстное меню порта и нажмите кнопку **Отключить**.

### OpenStack CLI

---

1. Получите ID виртуальной машины, которой необходимо подключить порт:

```
openstack server list
```

2. Получите список портов и выберите необходимый:

```
openstack port list
```

3. Подключите созданный порт к ВМ:

```
openstack server remove port <server> <port>
```

Где:

- `<server>` - имя или название ВМ.
- `<port>` - ID порта.

### Terraform

Подключение существующего порта к ВМ:

```hcl
resource "openstack_networking_network_v2" "network_1" {
  name           = "network_1"
  admin_state_up = "true"
}

resource "openstack_networking_port_v2" "port_1" {
  name           = "port_1"
  network_id     = openstack_networking_network_v2.network_1.id
  admin_state_up = "true"
}

resource "openstack_compute_instance_v2" "instance_1" {
  name            = "instance_1"
  security_groups = ["default"]
}

resource "openstack_compute_interface_attach_v2" "ai_1" {
  instance_id = openstack_compute_instance_v2.instance_1.id
  port_id     = openstack_networking_port_v2.port_1.id
}
```

Если требуется подключение к существующим ресурсам, можно получить с помощью `data`:

```
data "openstack_networking_network_v2" "instance" {
  name = "<name>"
}

data "openstack_networking_network_v2" "port" {
  name = "<port_name>"
}

resource "openstack_compute_interface_attach_v2" "ai_1" {
  instance_id = data.openstack_compute_instance_v2.instance.id
  port_id     = data.openstack_networking_port_v2.port.id
}
```
