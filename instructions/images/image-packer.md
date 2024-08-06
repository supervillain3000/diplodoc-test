## Подготовительные шаги.

1. Выполните установку [Packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli). 
2. Выполните [установку OpenStack CLI](ps.kz), если не установлен и пройдите [аутентификацию](empty) в проекте.

## Создание нового образа из образа источника с помощью Packer.

1. Получите ID внешней сети `FloatingIP Net`, а также ID приватной сети с помощью OpenStack CLI:

```
openstack network list
```

2. Получите ID образа, который будет использован в качестве источника. Можно [загрузить собственный образ](empty) и использовать его в качестве источника.
3. Экспортируйте переменные `NETWORK_ID SOURCE_IMAGE FIP_NET_ID`

```bash
export FIP_NET_ID=<fip_id>
export NETWORK_ID=<net_id>
export SOURCE_IMAGE=<image_id>
```

Где:
`NETWORK_ID` - ID приватной сети.
`SOURCE_IMAGE` - ID образа источника.
`FIP_NET_ID` - ID внешней сети `FloatingIP Net`

4. Создайте конфигурационный файл Packer с расширением .pkr.hcl:

```
variable "network_id" {
  type = string
  default = "${env("NETWORK_ID")}"
  validation {
    condition     = length(var.network_id) > 0
    error_message = <<EOF
The NETWORK_ID environment variable must be set.
EOF
  }
}

variable "source_image" {
  type = string
  default = "${env("SOURCE_IMAGE")}"
  validation {
    condition     = length(var.source_image) > 0
    error_message = <<EOF
The SOURCE_IMAGE environment variable must be set.
EOF
  }
}

variable "floating_ip_net_id" {
  type = string
  default = "${env("FIP_NET_ID")}"
  validation {
    condition     = length(var.floating_ip_net_id) > 0
    error_message = <<EOF
The FIP_NET_ID environment variable must be set.
EOF
  }
}

source "openstack" "example" {
  flavor                   = "d1.ram1cpu1"
  availability_zone        = "kz-ala-1"
  image_name               = "ubuntu-nginx"
  source_image             = "${var.source_image}"
  config_drive             = "true"
  networks                 = ["${var.network_id}"]
  security_groups          = ["ssh"]
  ssh_username             = "ubuntu"
  image_visibility          = "private"
  use_blockstorage_volume  = "true"
  floating_ip_network      = "${var.floating_ip_net_id}"
  volume_availability_zone = "kz-ala-1"
  volume_type              = "ceph-ssd"
  volume_size              = "1"
}

build {
  sources = ["source.openstack.example"]
  provisioner "shell" {
    execute_command = "sudo {{ .Path }}"
    inline = [
      "apt-get update",
      "apt-get install -y nginx"
      ]
  }
}

packer {
  required_plugins {
    openstack = {
      version = ">= 1.1.2"
      source  = "github.com/hashicorp/openstack"
    }
  }
}
```

Для работы с OpenStack спользуется [плагин OpenStack для Packer](https://github.com/hashicorp/packer-plugin-openstack).

5.  После завершения создания образа, проверьте его наличие:

### Панель управления

1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Образы**.

После загрузки образ должен появится в списке.
### OpenStack CLI

```
openstack image list --name <name>
```

Где:
`<name>` - имя указанного образа.