---
title: Начало работы с Terraform
description: Описание начала работы с Terraform
tags: ['Terraform', 'CLI', 'OpenStack', 'openrc']
---

# Terraform-quick-start

С помощью Terraform можно быстро создать необходимую инфраструктуру в PS Cloud. Состав инфраструктуры определяется с помощью конфигурационных файлов, в которых указываются требуемые облачные ресурсы и их параметры.

Для начала работы с Terraform:

1. Выполните [установку Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
2. Настройте провайдер OpenStack:

```hcl
terraform {
required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.53.0"
    }
  }
}

provider "openstack" {
  region      = "kz-ala-1"
}
```

3. Выполните [аутентификацию в проекте](openrc-auth.md), Terraform возьмет необходимые для подключения данные из переменных окружения после аутентификации.

Дополнительная информация по использованию провайдера OpenStack находится в официальной документации провайдера [OpenStack в Terraform Registry](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs).
