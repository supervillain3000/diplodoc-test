На данный момент существует два способа скачать образ через API:

### Скачивание образа через OpenStack CLI

### Подготовительные шаги

1. Выполните [установку](ps.kz) и пройдите [аутентификацию](empty) в проекте.

```bash
openstack image list
```

```bash
openstack image save --file <image_name>.raw <image>
```

Где:

`<image_name>` - имя файла для образа.
`<image>` - ID образа.

### Скачивание образа через API

1. [Получите токен Keystone](empty)
2. Скачайте образ командой:

```bash
curl -i -X GET -H "X-Auth-Token: $token" https://image.kz-ala-1.pscloud.io/v2/images/<image>/file --output <image>.raw
```

Где:

`<image>` - ID образа.