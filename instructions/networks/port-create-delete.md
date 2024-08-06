---
title: Создание/удаление порта
description: Описание создания и удаления порта
---

# Создание/удаление порта

## Создание порта

### Панель управления

---

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта и перейдите в раздел **Сети**.
3. Выберите сеть, в которой необходимо создать порт и перейдите в раздел **Порты**.
4. Нажмите кнопку **Создать**.
5. Укажите статический адрес для порта и выберите [группу безопасности](/).

### OpenStack CLI

---

1. Получите список сетей и подсетей:

```
openstack network list
```

2. Получите список групп безопасности:

```
openstack security group list
```

3. Создайте порт в нужной сети и с группой безопасности `default`:

```
openstack port create <имя порта> --network <network>
```

Где:

- `<network>` - имя или ID сети.

4. Или с указанием необходимых параметров:

```shell
  openstack port create <имя порта> \
  --network <network> \
  --fixed-ip subnet=<subnet>,ip-address=<ip-address> \
  --security-group <security-group>
```

Где:

- `<network>` - имя или ID сети.
- `<subnet>` ID подсети.
- `<ip-address>` - IP-адресс для порта.
- `<security-group>` - имя или ID группы безопасности.

## Удаление порта ВМ

### Панель управления

---

1. Перейдите в [панель управления PS Cloud](https://console.ps.kz/) и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта и перейдите в раздел **Сети**.
3. Выберите сеть, в которой необходимо создать порт и перейдите в раздел **Порты**.
4. Откройте контекстное меню `...` необходимого порта и нажмите кнопку **Удалить**.

### OpenStack CLI

---

1. Получите ID виртуальной машины:

```
openstack server list
```

2. Получите список подключенных портов к ВМ:

```
openstack port list --server <server>
```

Где:

- `<server>` - ID или имя ВМ.

3. Удалите порт:

```
openstack port delete <port>
```

Где:

- `<port>` - ID или имя ВМ.

### Terraform

```hcl
resource "openstack_networking_port_v2" "port_1" {
  name           = "<port_name>"
  network_id     = openstack_networking_network_v2.<network>.id
  admin_state_up = "true"
}
```

Где:

- `<port_name>` - имя для порта.
- `openstack_networking_network_v2.<network>.id` - вместо `<network>`, необходимо подставить название ресурса, который создает сеть.

Если необходимо создать порты в сети, которая уже существует вне terraform конфигурации, используйте `data` для получения ID сети:

```hcl
data "openstack_networking_network_v2" "network" {
  name = "<network>"
}

resource "openstack_networking_port_v2" "port_1" {
  name           = "<port_name>"
  network_id     = data.openstack_networking_network_v2.network.id
  admin_state_up = "true"
}
```

Где:

- `<network>` - имя сети, в которой требуется создать порт.
