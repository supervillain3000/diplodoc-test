---
title: SSL-сертификаты
description: Описание SSL-сертификатов
---

# SSL-сертификаты

SSL-сертификаты на балансировщиках нагрузки обеспечивают безопасность соединений между клиентами и серверами. Когда балансировщик настроен с протоколом **TERMINATED_HTTPS**, он принимает зашифрованные запросы от клиентов, расшифровывает их и передает незашифрованные данные на backend-серверы. Это разгружает backend-серверы от работы по шифрованию и дешифрованию данных и централизует управление SSL-сертификатами.

Для добавления сертификата в проект Openstack необходимо также оставить запрос в техническую поддержку.

## Openstack CLI

1. Выполните [установку](/ru/compute/instructions/compute-nova/instance-management) и пройдите [аутентификацию](ru/compute/instructions/compute-nova/instance-management) в проекте.
2. Также дополнительно необходимо установить компонент OCtavia:
   `pip3 install python-octaviaclient`

3. Создайте PKCS12 файл используя сертификат и приватный ключ сертификата.

> [! note] - Внимание!
> При создании файла p12 не указывайте на него пароль.

`openssl pkcs12 -export -in server.crt -inkey server.key -out test1.p12`

Где:

- `server.crt` - файл с сертификатом.
- `server.key` - файл с приватным ключом.

4. Импортируйте SSL p12 файл в секрет в проекте openstack:

```
openstack secret store --name='tls_secret2' -t 'application/octet-stream'
-e 'base64' --payload="$(base64 < test1.p12)"
```

Где:

- `--name='tls_secret2'` - задайте имя секрета, которое будет использоваться для хранения.
- `--payload="$(base64 < test1.p12)"` - Задает содержимое секрета, которое берется из файла `test1.p12` (созданный в предыдущенм шаге) и кодируется в base64 перед отправкой.

5.  Cоздайте балансировщик нагрузки:

```
openstack loadbalancer create \
  --vip-subnet-id <ID_подсети> \
  --vip-address <IP_адрес> \
  --flavor <ID_флейвора> \
  --name <имя_балансировщика>
```

Где:

- `ID_подсети` - ID подсети, в которой будет размещен виртуальный IP-адрес (VIP) балансировщика нагрузки. VIP используется для приема входящего трафика. (Может использоваться [внтурення подсеть](/) с последующей привязкой [плавающего IP адреса](Плавающий_IP))
- `IP_адрес` - Задает IP-адрес, который будет использоваться в качестве виртуального IP-адреса для балансировщика нагрузки. Этот адрес должен быть в пределах указанной подсети. Например для подсети `192.168.0.0/24` можно указать `192.168.0.19`, также с этим IP адресом _не должно_ быть созданных портов. (опционально - если не указать то будет выдан рандомный свободный IP)
- `ID_флейвора` - ID доступного флейвора для балансировщика, посмотреть можно при помощи команды: `openstack loadbalancer flavor list`
- `имя_балансировщика` - задайте имя для Вашего балансировщика

6. Создайте слушателя (правило для балансировки) с поддержкой HTTPS и привяжите его к контейнеру TLS, который хранит ваш сертификат:

```
openstack loadbalancer listener create \
--protocol-port 443 \
--protocol TERMINATED_HTTPS \
--name <имя_слушателя> \
--default-tls-container-ref <ссылка на контейнер TLS> \
<имя_балансировщика>
```

Где:

- `--name <имя_слушателя>` - Укажите имя нового слушателя.
- `--default-tls-container-ref <ссылка на контейнер TLS>`
- `<имя_балансировщика>` - имя балансировщика нагрузки, к которому будет привязан этот слушатель.

## Terraform

> [! note] - Внимание!  
> При создании балансировщика с помощью Terraform в блоке, в котором описываются параметры подключения к OpenStack, необходимо указать параметр `use_octavia = true`

Если у вас еще нет Terraform, [установите его и настройте провайдер Openstack](ru/compute/instructions/compute)

1. Для создания балансировщика нагрузки можно использовать следующий блок:

```
### Certificate Details ###
resource "openstack_keymanager_secret_v1" "certificate" {
name = "lb1-example-crt"
payload = "${file("/путь/до/сертификата")}"
secret_type = "certificate"
payload_content_type = "text/plain"
}

resource "openstack_keymanager_secret_v1" "private_key" {
name = "lb1-example-key"
payload = "${file("/путь/до/ключа")}"
secret_type = "private"
payload_content_type = "text/plain"
}

resource "openstack_keymanager_container_v1" "tls_1" {
name = "lb1-example-tls"
type = "certificate"
secret_refs {
name = "certificate"
secret_ref = "${openstack_keymanager_secret_v1.certificate.secret_ref}"
}
secret_refs {
name = "private_key"
secret_ref = "${openstack_keymanager_secret_v1.private_key.secret_ref}"
}
}
### End Certificate Detais ###

### Load Balancer Details ###
resource "openstack_lb_loadbalancer_v2" "lb1-https-test" {
name = "lb1-https-test"
vip_subnet_id = "39af68eb-2f7d-4cc5-a1f9-0a15b0632238"
}
### End Load Balancer Details ###

### Listener Details ###
resource "openstack_lb_listener_v2" "listen-https-test" {
name = "listen-https-test"
description = "what to listen?"
protocol = "TERMINATED_HTTPS"
protocol_port = 443
connection_limit = -1
loadbalancer_id = openstack_lb_loadbalancer_v2.lb1-https-test.id
default_tls_container_ref = openstack_keymanager_container_v1.tls_1.container_ref
}
### End Listener Details ###

### Pool Details ###
resource "openstack_lb_pool_v2" "https-pool-test" {
name = "https-pool-test"
protocol = "HTTP"
lb_method = "ROUND_ROBIN"
listener_id = openstack_lb_listener_v2.listen-https-test.id
}
### End Pool Details ###

### Monitor Details ###
resource "openstack_lb_monitor_v2" "https-monitor-test" {
name = "https-monitor-test"
delay = 5
max_retries = 3
timeout = 5
type = "HTTP"
url_path = "/"
http_method = "GET"
expected_codes = "200"
pool_id = openstack_lb_pool_v2.https-pool-test.id
}
### End Monitor Details ###

### Pool Members Details ###
resource "openstack_lb_member_v2" "centos-lb2-https-test" {
name = "centos-lb2-https-test"
address = "ip_address"
protocol_port = 80
pool_id = openstack_lb_pool_v2.https-pool-test.id
}

```

### Блок Load Balancer Details

Описывает задание имени для балансировщика и подсеть, которую необходимо прокинуть в балансировщик. Можно использовать как серую, так и белую сети.

Подробнее по ссылке: [https://registry.terraform.io/providers/terraform-...](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/lb_loadbalancer_v2)

### Блок Listener Details

Описывает детали прослушивания: с помощью какого протокола, на каком порту, количество соединений.

Подробнее по ссылке [https://registry.terraform.io/providers/terraform-...](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/lb_listener_v2)

### Блок Pool Details

Описывает задание параметров непосредственно для пула: используемый протокол и алгоритм для балансировки.

Подробнее по ссылке:  [https://registry.terraform.io/providers/terraform-...](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/lb_pool_v2)

### Блок Monitor Details

Описывает детали мониторинга пула инстансов: протокол, с помощью которого будет производится проверка, после выбора которого указываются более подробные детали для выбранного протокола.

Подробнее по ссылке: [https://registry.terraform.io/providers/terraform-...](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/lb_monitor_v2)

### Блок Pool Members Details

В данном блоке непосредственно указываются имена инстансов-участников, которые будут в пуле, и их адреса, которые необходимо добавить в пул. В качестве примеров указаны имена двух инстансов: alpha_http_member и beta_http_member. Имена можно задавать на своё усмотрение, но необходимо будет указать корректные имена далее по шаблону, если таковые будут встречаться.

### Блок Load Balancer IP Output

Данный блок является опциональным. Он описывает вывод IP-адреса балансировщика непосредственно после его создания.
