С помощью **Облачных серверов** можно создавать виртуальные машины через личный кабинет, OpenStack CLI, Terraform и REST API. 

## Подготовительные шаги

1. [Зарегистрируйтесь](https://auth.ps.kz/register) на платформе PS Cloud.
2. [Закажите услугу облачных серверов](https://www.ps.kz/hosting/vpc) 

Баланс счета должен быть положительным, а квот должно быть достаточно для создания желаемой конфигурации виртуальной машины.

### Панель управления
---

1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название проекта, который был зарегистрирован и выберите пункт **Серверы**.
3. Нажмите кнопку **Создать**.
4. Задайте необходимые параметры ВМ:
    - **Название сервера и зона доступности**: опциоанальный пункт. Вы можете указать необходимое название сервера или оставить случайно сгенерированное.  
    - **Образ**: выберите из списка публичный образ операционной системы. Также можно [загрузить собственный образ](ps.kz). Для создания виртуальной машины также может быть использован ранее созданный [диск](ps.kz) или [снимок диска](empty).
    - **Опции**: Выберите пункт **Использование [конфигурационного диска](ps.kz).** Если используется образ, который не подготовлен к использованию в облачной платформе, выберите пункт **Запустить установку ОС**. [Подробнее про образы](empty).
    - **Конфигурации**: Выберите желаемую **конфигурацию** из предоставленного списка. Если нужной конфигурации нет в списке, создайте тикет в техническую поддержку: support@ps.kz 
    - **Диски**: Откройте контекстное меню `...` и укажите размер и тип **[Загрузочного диска](empty)**. Также можно добавить к ВМ дополнительные диски с нужным типом и размером.
    - **Сеть**: К серверу необходимо подключить одну из существующих сетей либо создать новую. Для пункта **Подсети** нажмите кнопку **Добавить**. Если вы ранее создали сеть, в которой достаточно свободных IP-адресов, выберите ее. Если сеть не создана, нажмите кнопку **Создать**. Укажите имя сети в поле **Название**. В поле **Подсеть** укажите необходимую подсеть. Укажите использование [плавающего IP](empty), если необходим доступ к виртуальной машине по публичному IP-адресу.
    - **Firewall**: Укажите предустановленые настройки [групп безопасности](empty). Группы безопасности можно будет изменить или настроить после создания ВМ.
    - **Доступ**: Выберите или добавьте **SSH-ключ**. Можно также выбрать **Пароль**, он будет отправлен на вашу почту после установки ОС.
6. Если необходимо, вы можете добавить сценарий настройки виртуальной машины в пункте **Установка приложения**. Подробнее про сценарии настройки [cloud-init](empty).
7. **Расписание резервного копирования**: Если необходимо, настройте правила расписание [резервного копирования](empty). Это можно будет сделать после создания ВМ.
8. Нажмите кнопку **Создать сервер**.
9. Дождитесь, когда процесс создания виртуальной машины завершится. Статус ВМ изменится с `CREATING` на `ACTIVE`.

### OpenStack CLI
---
1. Выполните [установку](ps.kz), если OpenStack CLI не установлен и пройдите [аутентификацию](empty) в проекте.

2. Получите список доступных образов ОС и сохраните ID образа:
   
```shell
openstack image list
```

3. Получите список доступных [типов конфигураций](empty) ВМ и сохраните нужный ID:

```shell
openstack flavor list
```

4. Получите список доступных сетей и сохраните ID нужной сети:
   
```shell
openstack network list
```
  
5. Получите список доступных [групп безопасности](empty) и сохраните ID:
   
```shell
openstack security group list
```

5. Получите список доступных ключевых пар и сохраните `keypair_name`:
   
```shell
openstack keypair list
```

- Чтобы создать новую ключевую пару:

1. Сгенерируйте ключ:

```shell
ssh-keygen -q -N ""
```

2. Загрузите ключ:
		
```shell
openstack keypair create --public-key ~/.ssh/id_rsa.pub --type ssh <keypair_name>
```

6. Создайте загрузочный диск: 
   
```shell
openstack volume create root-volume --size 10 --image <image_id> --bootable
```

7. Создайте ВМ:
   
```shell
openstack server create <VM_name> \ 
   --volume <volume_ID> \
   --network <network_ID> \ 
   --flavor <flavor_ID> \ 
   --key-name <keypair_name> \
```

8. Проверьте состояние созданной ВМ:
```shell
openstack server list
```
	

Созданная машина должна появиться в списке доступных ВМ и иметь статус `ACTIVE`.

---
## Terraform

### Подготовительные шаги:

1. Установите Terraform.
2. Установите [OpenStack provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs)
3. Для создания диска укажите примеры конфигураций в tf файле:

Опишите в конфигурационном файле параметры создания локальной сети, маршрутизатора, группы безопасности и виртуальной машины с плавающим IP:

```hcl
data "openstack_images_image_v2" "image_data" {
	most_recent = true
	properties = {
	os_distro = "almalinux"
	os_version = "90"
	}
}

resource "openstack_networking_network_v2" "network" {
	name = "network_name"
	admin_state_up = true
}

resource "openstack_networking_subnet_v2" "subnet" {
	name = "subnet_name"
	network_id = openstack_networking_network_v2.network.id
	cidr = "192.168.10.0/24"
	enable_dhcp = true
	depends_on = [openstack_networking_network_v2.network]
}

resource "openstack_networking_router_v2" "router" {
	name = "router_name"
	admin_state_up = true
	external_network_id = data.openstack_networking_network_v2.external_network.id
}

resource "openstack_networking_router_interface_v2" "router_interface" {
	router_id = openstack_networking_router_v2.router.id
	subnet_id = openstack_networking_subnet_v2.subnet.id
	depends_on = [openstack_networking_router_v2.router]
}

resource "openstack_compute_secgroup_v2" "security_group" {
	name = "my_security_group"
	description = "open ssh and http"
	dynamic "rule" {
		for_each = ["22", "80"]
		content {
			from_port = rule.value
			to_port = rule.value
			ip_protocol = "tcp"
			cidr = "0.0.0.0/0"
		}
	}
	rule {
		from_port = -1
		to_port = -1
		ip_protocol = "icmp"
		cidr = "0.0.0.0/0"
	}
}

resource "openstack_compute_keypair_v2" "keypair" {
	name = "my-keypair"
	public_key = file("~/.ssh/public_key")
}

resource "openstack_compute_instance_v2" "instance" {
	name = "instance_name"
	flavor_name = "d1.ram1cpu1"
	security_groups = [openstack_compute_secgroup_v2.security_group.id]
	config_drive = true
	key_pair = openstack_compute_keypair_v2.keypair.name
	depends_on = [openstack_networking_subnet_v2.subnet]
	network {
		uuid = openstack_networking_network_v2.network.id
	} 
	block_device {
		source_type = "image"
		uuid = data.openstack_images_image_v2.image_data.id
		destination_type = "volume"
		volume_type = "ceph-ssd" 
		volume_size = 10
		delete_on_termination = true # false
	}
}
```

Где:
- `openstack_networking_network_v2` - описание создания сети.
- `openstack_networking_subnet_v2` - описание создания подсети.
- `openstack_networking_router_v2` - описание создания маршрутизатора.
- `openstack_networking_router_interface_v2` - описание подключения маршрутизатора.
- `openstack_compute_secgroup_v2` - описание создания группы безопасности.
- `openstack_compute_keypair_v2` - описание создания ключевой пары.
- `openstack_compute_instance_v2` - описание создания виртуальной машины.
	- `name` - название ВМ.
	- `flavor_name` - тип конфигурации ВМ.
	- `security_groups` - группы безопасности ВМ.
	- `config_drive` - [конфигурационный диск](empty)
	- `key_pair` - ключевая пара.
	- `network` - сеть ВМ.
	- `block_device` - описание блочного устройства.
		- `volume_type` - тип диска.
		- `volume_size` - размер диска.
		- `delete_on_termination` - политика удаления диска при удалении ВМ.

Если вы ранее создавали какие-либо ресурсы (облачная сеть и подсеть), вы можете не описывать их повторно. Используйте их имена и идентификаторы в соответствующих параметрах.

### Создайте ресурсы

1. Перейдите в терминале в директорию где находится созданный вами конфигурационный файл.
2. Проверьте корректность конфигурационного файла:

```shell
terraform validate
```

3. Выполните команду:

```shell
terraform plan
```
В терминале будет выведен список ресурсов с параметрами. На этом этапе изменения не будут внесены. Если в конфигурации есть ошибки, Terraform на них укажет.

4. Примените изменения конфигурации:

```shell
terraform apply
```
5. Подтвердите изменения: введите в терминале слово `yes` и нажмите **Enter**.
6.  Удалите ресурсы с помощью команды:

```shell
terraform destroy
```

---
## Смотрите также

- [Управление виртуальной машиной](empty).
- [Подключение к виртуальной машине по SSH](empty).

