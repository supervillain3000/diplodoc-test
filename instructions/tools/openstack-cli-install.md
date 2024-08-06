---
title: Установка Openstack CLI
description: Описание установки Openstack CLI
tags: ['OpenStack', 'CLI', 'RC']
---

# Openstack-cli-install

OpenStack CLI - это интерфейс командной строки для взаимодействия с OpenStack API. С помощью OpenStack CLI можно создавать и управлять облачной инфраструктурой.

Для взаимодействия с OpenStack API требуется:

1. Выполнить установку OpenStack CLI
2. Загрузить OpenStack RC-файл и выполнить аутентификацию в проект.

## Установка OpenStack CLI

### Ubuntu/Debian

Убедитесь, что python3 установлен, если нет -- выполните его установку:

```
sudo apt update
sudo apt install python3
```

Выполните установку pip3:

```
sudo apt install python3-pip
```

Выполните установку OpenStack CLI:

```
pip3 install python-openstackclient
```

### CentOS

Убедитесь, что python3 установлен, если нет - выполните его установку:

```
sudo dnf update
sudo dnf install python3
```

Выполните установку pip3:

```
sudo dnf install python3-pip
```

Выполните установку OpenStack CLI:

```
pip3 install python-openstackclient
```

### MacOS

Вы можете установить OpenStack CLI с помощью пакетного менеджера homebrew или с помощью pip3:

**Homebrew**

```
brew install python3
```

```
brew install openstackclient
```

**pip3**

```
pip3 install python-openstackclient
```

### Windows

Убедитесь что установлен python3 и pip3.

Выполните установку с помощью pip3:

```
pip3 install python-openstackclient
```
