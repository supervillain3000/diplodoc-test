**Cloud-init** - это инструмент для автоматической настройки ВМ в облаке с помощью скриптов cloud-config во время запуска. Cloud-config может быть передан в YAML-формате или в формате bash-скрипта.

Во время [создания сервера](instance-create.md) в панели управления, можно передать cloud-config в пункте **Установка приложения**. 

Скрипт cloud-config также может быть передан с помощью Terraform или Openstack CLI во время создания. Для Terraform этот параметр называется `user_data`, а для передачи с помощью OpenStack CLI необходимо использовать опцию `--user-data` с указанием пути до скрипта cloud-config. 

## Примеры использования

### Изменение пользователя по умолчанию

Пользователь по умолчанию создается с именем названия дистрибутива. Можно изменить стандартного пользователя: 

```
#cloud-config
system_info:
  default_user:
    name: <username>
    gecos: <username>
    shell: /bin/bash
    groups: <username, wheel, admin>
    sudo: ALL=(ALL) ALL
    ssh_authorized_keys:
    - "<ssh-public-key>"

```

### Установка пароля пользователю по умолчанию

```
#cloud-config
password: <password>
chpasswd: { expire: False }
ssh_pwauth: True
```

### Размещение публичного SSH-ключа для пользователя по умолчанию

```
#cloud-config
ssh_authorized_keys:  
- ssh-rsa AAAAC3N…kMoG 
```

### Обновление и установка пакетов

```
#cloud-config
package_update: true
package_upgrade: true

packages:
- python3-pip
```

Дополнительную информацию по использованию cloud-init можно получить в [официальной документации](https://cloudinit.readthedocs.io/en/latest/)




