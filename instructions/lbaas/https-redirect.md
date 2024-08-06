---
title: Редирект https
description: Описание редиректа https
---

# Редирект https

Чтобы настроить редирект с HTTP на HTTPS в OpenStack, нужно создать два слушателя (listener): один для HTTP (порт 80) и один для HTTPS (порт 443). Затем настроить политику редиректа на слушателе HTTP, чтобы он перенаправлял все запросы на HTTPS. Сделать Вы это можете через **Openstack CLI**

## OpenStack CLI

1. Выполните [установку](/ru/compute/instructions/compute-nova/instance-management) и пройдите [аутентификацию](ru/compute/instructions/compute-nova/instance-management) в проекте.
2. Также дополнительно необходимо установить компонент Oсtavia:

```shell
pip3 install python-octaviaclient
```

3. Выполните шаги из инструкции по [установке сертификата](/)
4. Создайте слушатель HTTP:

```shell
openstack loadbalancer listener create \
  --protocol-port 80 \
  --protocol HTTP \
  --name listener_http \
  <имя_балансировщика>
```

> Посмотреть созданные балансировщики можно при помощи команды:
> `openstack loadbalancer list`

5. Создайте политику для слушателя http:

```shell
openstack loadbalancer l7policy create \
  --action REDIRECT_TO_URL \
  --redirect-url https://<your-domain> \
  --name redirect_to_https_policy \
  <имя_слушателя_http>

```

6. Создайте правило для политики:

```shell
os loadbalancer l7rule create \
--compare-type REGEX \
--type HOST_NAME \
--value '.*' \
<имя_политики>
```
