Для работы образа с облачной платформой требуется подготовить образ

Обычно, производители дистрибутива публикуют образы, которые подготовлены к использованию в облаке. Например:

- [Debian 12 Bookworm](https://cloud.debian.org/images/cloud/bookworm/)
- [Ubuntu 24.04 Noble](https://cloud-images.ubuntu.com/noble/)
- [AlmaLinux 9](https://repo.almalinux.org/almalinux/9/cloud/x86_64/images/)

## Подготовка образа 

Вы можете установить образ в формате ISO на локальный ПК и запустить виртуальную машину.

В этом примере используется QEMU-KVM и образ Ubuntu 24.04:

Скачайте образ в формате ISO на локальный ПК:
```
wget -O /var/lib/libvirt/boot/<name>.iso \ 
https://releases.ubuntu.com/24.04/ubuntu-24.04-live-server-amd64.iso 
```

Установите права на образ, необходимые для работы с QEMU:

```
chown libvirt-qemu:kvm /var/lib/libvirt/boot/<name>.iso
```

Конвертируйте образ в qcow2:

```
qemu-img create -f qcow2 /var/lib/libvirt/images/<name>.qcow2 10G
```

Установите права на образ, необходимые для работы с QEMU:

```
chown libvirt-qemu:kvm /var/lib/libvirt/images/<name>.qcow2
```

Создайте виртуальную машину:

```
virt-install --virt-type kvm --name <name> --ram 1024 --cdrom=/var/lib/libvirt/boot/<name>.iso --disk /var/lib/libvirt/images/<name>.qcow2,bus=virtio,size=10,format=qcow2 --network network=default --graphics vnc,listen=0.0.0.0 --noautoconsole --os-variant=ubuntu24.04
```

Выполните установку нужных пакетов на виртуальную машину для работы на облачной плафторме:

```
sudo apt-get update
```

```
sudo apt-get install cloud-init cloud-guest-utilsqemu-guest-agent
```

```
dpkg-reconfigure cloud-init
```

Отключите виртуальную машину:

```
/sbin/shutdown -h now
```

Выполните очистку образа командой:

```
virt-sysprep -d <name>
```

Где:
`<name>` - имя виртуальной машины

Отключите образ от libvirt:

```
virsh undefine <name>
```

Образ готов к [загрузке на облачную плафторму](empty).