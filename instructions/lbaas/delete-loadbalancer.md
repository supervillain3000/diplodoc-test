---
title: Удаление loadbalancer
description: Описание удаления loadbalancer
---

# Удаление loadbalancer

## Панель управления

1.  Перейдите в [панель управления](https://auth.ps.kz/?required_keys=uid&redirect_uri=https://console.ps.kz/account) PS Cloud и в блоке **Ресурсы облака** нажмите на ресурс **количество балансировщиков**
2.  Нажмите на `...` справа от названия балансировщика и нажмите **удалить**

## OpenStack CLI

1. Выполните [установку](/ru/compute/instructions/compute-nova/instance-management) и пройдите [аутентификацию](ru/compute/instructions/compute-nova/instance-management) в проекте.
2. Выполните установку компонента Octavia для OpenStack CLI:

```
pip3 install python-octaviaclient
```

3. Посмотрите список балансировщиков:

```shell
openstack loadbalancer list
```

4. Удалите балансировщик при помощи следующей команды:

```
openstack loadbalancer delete --cascade <loadbalancer>
```

Где:

- `<loadbalancer>` - имя или ID балансировщика.
