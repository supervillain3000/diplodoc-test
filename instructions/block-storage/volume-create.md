## Создание диска
#### Подготовительные шаги

1. [Зарегистрируйтесь](https://auth.ps.kz/register) на платформе PS Cloud.
2. [Закажите услугу облачных серверов](https://www.ps.kz/hosting/vpc) 

Баланс счета должен быть положительным, а квот должно быть достаточно для создания дисков нужного объема и типа.

Диски могут быть созданы из следуюших источников: 
- пустой диск — который, не содержит в себе данные. Может быть использован для подключения к ВМ для масштабирования дискового пространства. 
- образ —  может быть использован для создания диска с образом, с помощью которого может быть запущена ВМ. 
- другой диск, снапшота или бэкапа — может быть использован для восстановления ВМ.
### Панель управления
---
1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название необходимого проекта, и перейдите в раздел **Диски**.
3. Нажмите **Создать**.
4. Укажите название диска. 
5. Выберите регион доступности, в котором необходимо создать диск. 
6. Выберите [тип диска](empty). Типы диска отличаются скоростью чтения и записи, а также пропускной способностью. 
7. Укажите необходимый размер диска в ГБ. После создания диска, уменшить его размер будет нельзя.
8. Укажите источник диска. 
9. Укажите опцию загрузочного диска, если диск на этот диск будет установлена ОС и необходимо будет загрузить ВМ с этого диска.

### OpenStack CLI
---
1. Выполните [установку](ps.kz), если OpenStack CLI не установлен и пройдите [аутентификацию](empty) в проекте.
2. Создайте диск:

```shell
openstack volume create
    [--image <image> | --snapshot <snapshot> | --source <volume> | --backup <backup>] \
    --size <size> \
    --type <volume_type> \
    [--availability-zone <availability-zone>] \
    [--bootable] \
    <name>
```

Где: 

Аргументы заключенные в [ ] являются опциональными. 

- `[--image <image> | --snapshot <snapshot> | --source <volume>]` - указание типа источника диска. Если не указывать, то будет создан пустой диск.
- `--image <image>` - использование образа в качестве источника. Вместо параметра `<image>` необходимо указать название или ID образа. Посмотреть список доступных образов можно с помощью команды:

```shell
openstack volume list
```

- `--snapshot <snapshot>` - использование снимка диска в качестве источника. Вместо параметра `<snapshot>` необходимо указать название или ID снимка диска. Посмотреть список доступных снимков можно  с помощью команды:

```
openstack volume snapshot list
```

- ``--source <volume>`` - клонирование и использование другого диска в качестве источника.  Вместо параметра `<volume>` необходимо указать название или ID диска. Посмотреть список доступных снимков можно  с помощью команды:

```
openstack volume list
```

- `--size <size>` - размер диска в ГБ.
- `--type <volume_type>` - тип диска. Вместо параметра `<volume_type>` указать название или ID типа диска. Посмотреть типы диска можно с помощью команды:

```shell
openstack volume type list
```

- `[--bootable]` - если в дальнейшем требуется запустить ВМ из диска и диск не является пустым. 
- `<name>` - название диска. 

### Terraform
---
#### Подготовительные шаги:

1. Установите Terraform.
2. Установите [OpenStack provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs)
3. Для создания диска укажите примеры конфигураций в tf файле:

```hcl
resource "openstack_blockstorage_volume_v3" "volume1" {
  name  = "volume"
  size  = 1
  type = "ceph-ssd"
}
```

4. Для создания диска из образа, можно использовать следующие пример:

```hcl
resource "openstack_blockstorage_volume_v3" "volume1" {
  name  = "volume"
  size  = 8
  volume_type = "ceph-ssd"
  image_id = "f2ddbb1c-038c-40a4-9c1a-52c6d8a19c40"
  enable_online_resize = true
}

```

Где:

- [`name`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#name) - (Опционально) Имя диска, при изменении меняет имя диска. 
- `size` - (Обязательно) размер диска в ГБ. При изменеии значения, которое больше текущего, увеличит размер диска. Значение ниже текущего указать нельзя.
- [`volume_type`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#volume_type) - (Опционально) [тип диска](empty). Изменение типа диск создаст новый диск. 
- [`image_id`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#image_id) - (Опционально) ID образа из которого необходимо создать диск. Аргумент нельзя использовать вместе с другими источниками диска. Изменение этого аргумента создаст новый диск. 
- [`enable_online_resize`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#enable_online_resize) - (Опционально) если true, разрешает изменять размер подключенного диска. 

Для создания диска из других источников можно использовать следующие аргументы: 

- [`snapshot_id`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#snapshot_id) - (Опционально) ID снимка диска из которого необходимо создать диск. Аргумент нельзя использовать вместе с другими источниками диска. Изменение этого аргумента создаст новый диск. 
- [`source_vol_id`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#source_vol_id) - (Опционально) ID диска из которого необходимо создать новый диск. Аргумент нельзя использовать вместе с другими источниками диска. Изменение этого аргумента создаст новый диск. 
- [`backup_id`](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/blockstorage_volume_v3#backup_id) - (Опционально) ID резервной копии диска из которого необходимо новый создать диск. Аргумент нельзя использовать вместе с другими источниками диска. Изменение этого аргумента создаст новый диск. 

Дополнительная информация описана в [документации провайдера OpenStack](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/)


5. Выполните команду для проверки:

```hcl
terraform plan
```

6. Создайте диск:

```
terraform apply
```

7. Для удаления ресурса:

```
terraform destroy
```