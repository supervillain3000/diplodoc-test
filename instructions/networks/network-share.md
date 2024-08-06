---
title: Предоставление доступа к сети
description: Описание предоставления доступа к сети
---

# Предоставление доступа к сети

В ситуации, когда сети двух разных проектов необходимо соединить, можно использовать RBAC для предоставления доступа к сети другому проекту.

На данный момент управление RBAC недоступно в панели управления, поэтому предоставить доступ к сети другому проекту можно с помощью [OpenStack CLI](/ru/cloud/instructions/tools/openstack-cli-install):

1. Выполните [установку](/), если OpenStack CLI не установлен и пройдите [аутентификацию](/) в проекте.
2. Получите список сетей:

```shell
openstack network list --internal
```

3. Для предоставления доступа также потребуется получить ID проекта, которому необходимо предоставить доступ к сети:

```shell
openstack config show -c auth.project_id -f value -c auth.project_name -f value -f yaml
```

4. Предоставьте доступ к сети:

```shell
openstack network rbac create --target-project <project> \
--action access_as_shared --type network <network>
```

Где:

- `<project>` - ID проекта, которому необходимо предоставить доступ к сети.
- `<network>` - ID сети, к которой предоставляется доступ.

5. На стороне проекта, которому был предоставлен доступ к сети, создайте порт в этой сети:

```shell
openstack port create --network <network> <name>
```

Где:

- `<network>` - ID сети
- `<name>` - имя порта

6. Подключите порт к роутеру:

```shell
openstack router add port <router> <port>
```

Где:

- `<router>` - ID роутера, можно посмотреть командой `openstack router list`
- `<port>` - ID порта, можно посмотреть командой `openstack port list`

7. Добавьте статический маршрут к сети:

```shell
openstack router set <router> --route destination=<destination_cidr>,gateway=<ip-address>
```

Где:

- `<router>` - ID роутера
- `--route destination=<destination_cidr>` - подсеть, для которой устанавливается статический маршрут.
- `gateway=<ip-address>` - IP-адрес шлюза, через который будет осуществляться маршрут. Он был создан и подключен к роутеру в 6 пункте.
