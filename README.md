# Виртуальные машины для лабораторных работ курсов LL

## Описание

Тестовая среда состоит из 5 виртуальных машин с CentOS 7, либо CentOS 6.

Пользователь    | Пароль
----------------|--------
vagrant         | vagrant
student (wheel) | student
root            | redhat

### c7-nat01.example.com

* ip: 192.168.2.1
* DHCP сервер, раздающий аренды для подсети 192.168.2.0/24
* DNS сервер.
* PXE для загрузки по сети
* ftp сервер для репозитория пакетов с dvd диска дистрибутива <ftp://192.168.2.1/pub/dvd>.
* ipa сервер: <https://c7-nat01.ll-101.local>
* keytab файлы cifs nfs для c7-server01: <ftp://c7-nat01.ll-101.local/pub/>

Пользователи IPA     | Пароль
---------------------|--------
admin                | password
lisa                 | password
linda                | password

### c7-server01.example.com

* ip: 192.168.2.10
* minimal server
* Дополнительная сетевая карта для NIC Teaming ip: 192.168.2.11
* Дополнительный пустой жесткий диск на 10 GiB
* Настроен как ipa клиент к домену EXAMPLE.COM

### c7-server02.example.com

* ip: 192.168.2.20
* minimal server
* Дополнительная сетевая карта для NIC Teaming ip: 192.168.2.21
* Дополнительный пустой жесткий диск на 10 GiB

### c6-server01.example.com

* ip: 192.168.2.10
* minimal server
* Дополнительный пустой жесткий диск на 10 GiB

### c7-client01.example.com

* ip: 192.168.2.40
* Дополнительный пустой жесткий диск на 10 GiB
* Поставлен комплект пакетов "Server with GUI", запуск по умолчанию с графической оболочкой

### c7-client02.example.com

* ip: 192.168.2.50
* Дополнительный пустой жесткий диск на 10 GiB
* Поставлен комплект пакетов "Server with GUI", запуск по умолчанию с графической оболочкой

## Требования

* Vagrant
* libvirt или VirtualBox
* Минимум 16 GiB Памяти (Используя рекомандованные настройки)
* Минимум 30 GiB на жестком диске (Используя рекомандованные настройки)

## Установка и настройка

1. [Hashicorp Vagrant](https://www.vagrantup.com/downloads.html)
2. [Cmder](http://cmder.net/)
3. [Vagrant образ CentOS 7 под VirtualBox](https://vagrantcloud.com/centos/boxes/7/versions/1708.01/providers/virtualbox.box)
4. [Oracle VirtualBox](http://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html#vbox)
5. [Extension Pack для VirtualBox](http://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html#vbox)
6. [DVD образ дистрибутива CentOS 7](http://mirror.yandex.ru/centos/7.4.1708/isos/x86_64/)
7. [Файлы из этого репозитороия для установки виртуальных машин](https://github.com/dmi3mis/LL)

### Запуск среды, используя версию CentOS

```bash
cd LL
vagrant up
```

### Полезные плагины

* [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest)

Плагин автоматом при запуске виртуальной машины загружает и ставит службы интеграции. Автоматически поставит всё необходимое. Править конфиг не нужно.
Из минусов только, что он не работает на Microsoft Windows.

Установка:

```bash
vagrant plugin install vagrant-vbguest
```

* [vagrant-timezone](https://github.com/tmatilai/vagrant-timezone)

Плагин автоматически ставит часовой пояс в виртуальной машине, к сожалению уже пару лет не обновляется и ввиду этого может вызыватьпроблемы на новых операционках.
Может брать часовой пояс с хоста.

Чтобы заставить его работать, в `Vagrantfile` нужно добавить такую строку

```ruby
if Vagrant.has_plugin?("vagrant-timezone")
    config.timezone.value = :host
  end
```

Установка:

```bash
vagrant plugin install vagrant-timezone
```
