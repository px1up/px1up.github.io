Первым делом проверим поддерживает-ли процессор виртуализацию:

    cat /proc/cpuinfo | egrep "(vmx|svm)"

**_Если вывод команды схож с изображением выше, то продолжаем_**

Для дальнейшей работы установим необходимые библиотеки и утилиты:

    yum install bridge-utils qemu-kvm libvirt virt-install

-   `bridge-utils` - утилиты для создания сетевого моста
-   `qemu-kvm` - гипервизор
-   `libvirt` - библиотеки для виртуализации
-   `virt-install` -утилита для управления виртульными машинами

Включаем виртуализацию при старте системы:

systemctl start libvirtd

Далее создаем каталоги для нашей KVM:

    mkdir -p /kvm/{images,iso,xml}

-   `/images` - папка хранения образов KVM
-   `/iso` - папка для iso с дистрибутивами
-   `/xml` - папка хранения xml файлов(для монтирования usb или других устройств к KVM

Теперь настроим сетевой мост чтобы наша машина получала IP-адрес из общей сети:

    ifconfig

Редактируем настройки интерфейса eth0 (у вас он может называется по-другому):

    nano /etc/sysconfig/network-scripts/ifcfg-eth0

Заменяем все содержимое файла на:

    ONBOOT=yes
    BRIDGE=br0
    TYPE=Ethernet
    DEVICE=eth0
    BOOTPROTO=none

Создаем сетевой мост:

nano /etc/sysconfig/network-scripts/ifcfg-br0

Прописываем:

    DEVICE=br0
    TYPE=Bridge
    ONBOOT=yes
    BOOTPROTO=static
    IPADDR=192.168.88.136
    NETMASK=255.255.255.0
    GATEWAY=192.168.88.1
    DNS1=8.8.8.8
    DNS2=8.8.4.4

После, перезапускаем сеть:

service network restart

Далее нужно включить перенаправление сетевых пакетов, редактируем *sysctl.conf:*

    nano /etc/sysctl.conf

Меняем значение на:

    net.ipv4.ip_forward=1

Применяем настройки:

    sysctl -p /etc/sysctl.conf

Теперь можно приступить к созданию виртуальной машины  
Команда создания KVM для CentOS:

    virt-install -n CentOSVM --noautoconsole --network=bridge:br0 --ram 4096 --arch=x86_64 --vcpus=1 --cpu host --check-cpu --disk path=/kvm/images/CentOSHDD.img,size=16 --cdrom /kvm/iso/Distr.iso --graphics vnc,listen=0.0.0.0,password=qwerty1 --os-type linux --os-variant=rhel7 --boot cdrom,hd,menu=on

-   `CentOSVM` - название виртуального хоста
-   `--ram` - количество оперативной памяти
-   `--vcpus` - количество вирутальных процессоров
-   `CentOSHDD.img, size=16` - Имя виртуального HDD и его размер (GB)
-   `--cdrom /kvm/iso/Distr.iso` -Путь к установочному .iso образу
-   `--graphics vnc,listen=0.0.0.0,password=qwerty1` - Подключение консоли с помощью VNC, password - пароль

Для просмотра списка виртуальных машин введем:

    virsh list --all

Запускаем ее командой:

    virsh start CentOSVM

Можно включить автозапуск виртуальной машины при запуске системы:

    virsh autostart CentOSVM

Для установки операционной системы подключимся с помощью VNC-клиента, смотрим VNC-порт командой:

    virsh vncdisplay CentOSVM

и получаем вывод:

    :0

Соответственно соединяемся по порту  **5900  
***если значение будет :1 то соединяемся по порту 5901 и т.д.  
Для соединения нужно открыть диапазон портов  **5900-5910**

    firewall-cmd --permanent --add-port=5900-5905/tcp

В VNC клиенте вводим адрес и порт, пароль, который указывали при создании виртуальной машины

После подключения устанавливаем ОС


