
Для создания образа могут быть выбраны следующие источники:
- Диск - создание образа из диска.
- URL - скачивания удаленного образа. 
- Файл - локальный образ, который будет загружен с вашего ПК.

Для загрузки образа в панели управления могут быть использованы следующие форматы: `qcow2`, `iso`, `raw`, `vdi`, `vhd`, `vmdk`, `ploop`. 
## Создание образа из диска
### Панель управления
---
1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Образы**.
3. Нажмите **Создать**.
4. Укажите название образа.
5. В качестве источника образа выберите **Диск** и укажите из какого диска необходимо создать образ.
6. Нажмите кнопку **Создать**.

### OpenStack CLI
---
1. Выполните [установку](ps.kz), если OpenStack CLI не установлен и пройдите [аутентификацию](empty) в проекте.

2. Получите список дисков:

```
openstack volume list
```

3. Создайте образ из диска:

```
openstack image create --volume <ID диска> <название образа>
```

4. Проверьте созданный образ:

```
openstack image list --name <название образа>
```

## Загрузка образа в облако с локального ПК
### Панель управления
---
1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Образы**.
3. Нажмите **Создать**.
4. Укажите название образа. 
5. В качестве источника образа выберите **Файл** и нажмите кнопку **Выбрать файл**.
6. Выберите файл образа на локальном ПК.
7. Укажите формат исходного образа.
8. Нажмите кнопку **Создать**.

### OpenStack CLI
---
1. Выполните [установку](ps.kz), если OpenStack CLI не установлен и пройдите [аутентификацию](empty) в проекте.

```shell
openstack image create --private --container-format bare --disk-format raw --property store=s3 --file --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes <путь к файлу образа> <название образа>
```

### Terraform

```hcl
resource "openstack_images_image_v2" "example" {
	name = "<name>"
	local_file_path = "/path/to/image"
	container_format = "bare"
	disk_format = "qcow2"
	region = "kz-ala-1"
	visibility = "private"
	
	properties = {
		os_distro = "<distro>"
		os_version = "<version>"
		os_require_quiesce = "true"
		os_type = "linux"
		hw_qemu_guest_agent = "yes"
		hw_scsi_model = "virtio-scsi"
		hw_disk_bus = "scsi"
	}
}
```

Где:
- `name` - имя образа
- `local_file_path` - путь до исходного образа. По умолчанию образ скачивается на локальный клиент, после чего импортируется в Glance. Несовместим с аргументом `image_source_url`.
## Загрузка образа по URL
### Панель управления
---
1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Образы**.
3. Нажмите **Создать**.
4. Укажите название образа.
5. В качестве источника образа выберите URL и укажите URL для скачивания образа.
6. Выберите формат исходного образа. 
7. Нажмите кнопку **Создать**.
### OpenStack CLI
---
1. Выполните [установку](ps.kz), если OpenStack CLI не установлен и пройдите [аутентификацию](empty) в проекте.
2. Создайте образ:
```
glance image-create-via-import \
  --import-method web-download \
  --uri <image_url> \
  --name <image_name> \
  --disk-format <image_format> \
  --container-format <container_format> \
  --property hw_disk_bus=scsi \
  --property hw_scsi_model=virtio-scsi \
  --property x_sel_image_owner=Selectel \
  --property hw_qemu_guest_agent=yes \
  --store <pool_segment>
```
## Terraform
---
### Подготовительные шаги:

1. Установите Terraform.
2. Установите [OpenStack provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs)

```hcl
resource "openstack_images_image_v2" "example" {
	name = "<name>"
	image_source_url = "<URL>"
	web_download = true
	container_format = "bare"
	disk_format = "qcow2"
	region = "kz-ala-1"
	visibility = "private"
	
	properties = {
		os_distro = "<distro>"
		os_version = "<version>"
		os_require_quiesce = "true"
		os_type = "linux"
		hw_qemu_guest_agent = "yes"
		hw_scsi_model = "virtio-scsi"
		hw_disk_bus = "scsi"
	}
}
```

Где:

- `name` - имя образа
- `image_source_url` - ссылка на исходный образ. По умолчанию образ скачивается на локальный клиент, после чего импортируется в Glance. Несовместим с аргументом `local_file_path`.
- `web_download` - по умолчанию false. Если true, позволяет скачивать образ напрямую в OpenStack. 

Образ также может быть загружен с локального клиента с помощью аргумента `local_file_path` и указанием пути до образа. Несовместим с `image_source_url` и `web_download`.

- `container_format` - обязательный параметр. Формат контейнера определяет, находится ли образ виртуальной машины в формате файла, который также содержит метаданные о реальной виртуальной машине. Обратите внимание, что строка формата контейнера в настоящее время не используется Glance или другими компонентами OpenStack, поэтому если вы не уверены, можно просто указать в качестве формата контейнера bare.
- `disk_format ` - обязательный параметр. Формат диска образа виртуальной машины — это формат базового образа диска. Поставщики виртуальных устройств используют разные форматы размещения информации, содержащейся в образе диска виртуальной машины.
- `region` - регион в котором необходимо создать ресурс.
- `visibility` - видимость образа. Может быть "public" или "private". 
- `properties` - свойства образа. Подробнее про доступные [свойства](empty)

Если скачиваемый образ сжат с помощью алгоритмов gzip, bzip2 или xz. Необходим аргумент `decompress`, который перед загрузкой в OpenStack выполнит распаковку. 

[Дополнительная информация по используемым аргументам для провайдера OpenStack в Terraform](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/images_image_v2)
