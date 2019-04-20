# Виртуальные машины для лабораторных работ курсов LL

## Описание

Тестовая среда состоит из 6 виртуальных машин с CentOS.

Пользователь    | Пароль
----------------|--------
vagrant (wheel) | vagrant
student (wheel) | student
root            | redhat

### c7-nat01.example.com

* ip: 192.168.2.1
* DNS, DHCP, PXE сервер
* vsftpd сервер для репозитория пакетов с dvd диска дистрибутива
* ipa сервер
* keytab файлы cifs nfs для c7-server01

Пользователи IPA     | Пароль
---------------------|--------
admin                | password
lisa                 | password
linda                | password

### c7-server01.example.com

* ip: 192.168.2.10
* minimal server
* Дополнительная сетевая карта для NIC Teaming ip: 192.168.2.11
* Дополнительный пустой жесткий диск на 2 GiB
* Настроен как ipa клиент к домену EXAMPLE.COM

### c7-server02.example.com

* ip: 192.168.2.20
* minimal server
* Дополнительная сетевая карта для NIC Teaming ip: 192.168.2.21
* Дополнительный пустой жесткий диск на 2 GiB

### c6-server01.example.com

* ip: 192.168.2.30
* minimal server
* Дополнительный пустой жесткий диск на 2 GiB

### c7-client01.example.com

* ip: 192.168.2.40
* Дополнительный пустой жесткий диск на 2 GiB
* Поставлен комплект пакетов "Server with GUI", запуск по умолчанию с графической оболочкой

### c7-client02.example.com

* ip: 192.168.2.50
* Дополнительный пустой жесткий диск на 2 GiB
* Поставлен комплект пакетов "Server with GUI", запуск по умолчанию с графической оболочкой

## Требования

* Vagrant
* Oracle VirtualBox
* ~ 16 GiB памяти
* ~ 30 GiB на жестком диске

## Установка и настройка

1. [Hashicorp Vagrant](https://www.vagrantup.com/downloads.html)
2. [Cmder](http://cmder.net/)
3. [Oracle VirtualBox и его Extension Pack](https://www.virtualbox.org/wiki/Downloads)
4. [CentOS-7-x86_64-DVD-1804.iso](http://mirror.yandex.ru/centos/7.5.1804/isos/x86_64/CentOS-7-x86_64-DVD-1804.iso)
5. [Этот репозиторий](https://github.com/dmi3mis/LL)

### Запуск 

```bash
git clone https://github.com/dmi3mis/LL
cd LL
vagrant up
```

### Снимки состояния

Сразу после успешной установки рекомендуется сохранить состояние виртуальных машин с помощью снимков состояния.
Для этого дайте команду

```bash
vagrant snapshot save [vm-name] snapshot-name
```
Чтобы потом вернуться к ранее сохранённому состоянию машин и не повторять долгую процедуру установки дайте команду.
Не забудьте указать параметр `--no-provision`, чтобы машины только восстановились из ранее сохранённого состояния, запустились и не запускали заново скрипты развертывания.

> ВНИМАНИЕ!
> При возврате к ранее сохранённому снимку все изменения, проделанные с машиной удалятся.
> Чтобы вернуться и не потерять изменения, перед возвратом создайте ещё один снимок под другим именем.

```bash
 vagrant snapshot restore --no-provision [vm-name] snapshot-name
```

### Полезные плагины

При первом запуске `vagrant up` попросит установить необходимые плагины vagrant-vbguest vagrant-timezone vagrant-hosts если не было установлено. Ответьте на вопросы, нажав кнопку "y" и после установки плагинов повторите вашу команду. 
Во время установки необходимых плагинов вы увидите примерно такой текст:

```bash
 vagrant up
Vagrant has detected project local plugins configured for this
project which are not installed.

  vagrant-hosts, vagrant-timezone, vagrant-vbguest
Install local plugins (Y/N) [N]: y
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching: micromachine-2.0.0.gem (100%)
Fetching: vagrant-vbguest-0.16.0.gem (100%)
Installed the plugin 'vagrant-vbguest (0.16.0)'!
Installing the 'vagrant-timezone' plugin. This can take a few minutes...
Fetching: vagrant-timezone-1.2.0.gem (100%)
Installed the plugin 'vagrant-timezone (1.2.0)'!
Installing the 'vagrant-hosts' plugin. This can take a few minutes...
Fetching: vagrant-hosts-2.8.3.gem (100%)
Installed the plugin 'vagrant-hosts (2.8.3)'!


Vagrant has completed installing local plugins for the current Vagrant
project directory. Please run the requested command again.
```

Описание плагинов:

* [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest)

Плагин автоматом при запуске виртуальной машины загружает и ставит службы интеграции. Автоматически поставит всё необходимое. Править конфиг не нужно.
Из минусов только, что он не работает на Microsoft Windows.

Установка:

```bash
vagrant plugin install vagrant-vbguest
```

* [vagrant-timezone](https://github.com/tmatilai/vagrant-timezone)

Плагин автоматически ставит часовой пояс в виртуальной машине, к сожалению уже пару лет не обновляется и ввиду этого может вызыватьпроблемы на новых операционках. Может брать часовой пояс с хоста. Для курсов не обязателен, но будет полезен не потеряться во времени.

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

