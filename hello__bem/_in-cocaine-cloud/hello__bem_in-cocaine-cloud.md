## hello__bem в кокаиновом облаке

<div style="width:400px;height:400px;float:left;margin:0 15px 15px 0;-webkit-border-radius:200px;-khtml-border-radius:200px;-moz-border-radius:200px;border-radius:200px;-webkit-box-shadow:10px 10px 5px #ccc;-moz-box-shadow:10px 10px 5px #ccc;box-shadow:10px 10px 5px #ccc"><div style="margin:0;width:400px;height:400px;background-color:#000;background-image:url('http://img-fotki.yandex.ru/get/9836/204961608.0/0_ad3b0_2d1c4a20_L.png');background-repeat:no-repeat;background-position:center;display:block;position:relative;-webkit-border-radius:200px;-khtml-border-radius:200px;-moz-border-radius:200px;border-radius:200px;-webkit-box-shadow:inset 0 0 15px #fff;-moz-box-shadow:inset 0 0 15px #fff;box-shadow:inset 0 0 15px #fff"></div></div>

Что под капотом у облака?

Давайте попробуем его создать, а на домашнее задание останется разобрать, как оно работает, что есть мифы,
а что реальность.

Но что здесь БЭМ? А БЭМ решил устроить поход Frontend-еров в Backend, и осквернить храмы высоконагруженных,
распределенных, масштабируемых систем языком JavaScript. По правде, JavaScript уже это сделал.

Приступим

---

### Предистория

Пытаясь помочь подготовить доклад ["Свое веб-приложение в облаке – просто и удобно"](http://clubs.ya.ru/yasubbotnik/replies.xml?item_no=847)
к Я.Субботнику скопилось немного косвенной информации. И как часто случается, ничего из этого в доклад не попало,
а выбросить добро было жалко. И тут занесло на YaC-2013, Я ближе познакомился с хорошими ребятами из Кокаина, которые
разъяснили, что, по чём и под какие статьи РФ это подпадает.

### Немного теории

> Хочу заранее предупредить, все технологии рассматриваются поверхностно, детали опущены, и для целей практического
> использования требуется более глубокое изучение.

Возьмем себе Кокаин, но не ввиде порошка. Оказывается он иметет более илюзорное состояние — "облачная платформа",
— все слышали, хочется, а понюхать не всем позволено.

Как гласит документация — это PaaS

<!-- TODO: Описать теорию, надрать из выступлений -->

__Рис. 1:__ Схема облака в вакууме

![Рис. 1](https://rawgithub.com/levonet/articles/master/hello__bem/_in-cocaine-cloud/hello__bem_in-cocaine-cloud-01.svg)

На верхнем уровне находится Load balancer, распространенный случай его Round-robin DNS. Далее Fronts, как правило это
машинкы всречающие сетевую нагрузку и распределяющие/проксирующие запросы на Backends, где крутятся приложения. Между
Load balancer и HTTP-proxy кокаина, над Fronts можно было нарисовать nginx-proxy, который может заниматься разруливанием
роутов.

~~К рассмотрению предлагаю следующую схему домашней облачной платформы:~~

Рис. 2

<!-- ![Рис. 2](hello__bem_in-cocaine-cloud-02.svg) -->

  - server — собственно это и есть наш компъютор на котором поднимаем виртуалки
  - gentooXX — виртуалки с ОС Gentoo
  - ubuntuXX — виртуалки с ОС Ubuntu

<!-- TODO: Описать схему, что и с чем взаимодействует -->

<!--  Много теории -->

### Суровая реальность

Для эксперимента я имею старенький IBM с ограниченными ресурсами и установленной на нем Gentoo. По этой причине,
в качестве поднятия виртуальных машин попробуем использовать самое легковесное решение — LXC.

Для наглядности, под параллельные задачи, используется по две виртуальные машины, для пущего веселья,
с разными гостевыми системами:
  - на правую печеньку шоколад наносим вдоль — Gentoo из репозитория на момент сборки;
  - на левую наносим поперек — Ubuntu Precise (LTS) — настоящий выбор HL-админа.

#### Подготовка

Изначально было желание опустить процесс создания и запуска виртуальных машин в пользу существующих,
хоть и устаревших, документов: [LXC - Gentoo Wiki](http://wiki.gentoo.org/wiki/LXC), etc. Нужно уточнить,
что LXC поднята на ядре linux 3.8.13 и установка Ubuntu Precise (LTS) не обошлась без небольшого хака хелпера:

```diff
--- /usr/share/lxc/templates/lxc-ubuntu~	2013-07-10 09:34:33.000000000 +0300
+++ /usr/share/lxc/templates/lxc-ubuntu	2013-07-11 14:16:51.000000000 +0300
@@ -205,7 +205,7 @@
 EOF
     chmod +x "$1/partial-${arch}"/usr/sbin/policy-rc.d
 
-    lxc-unshare -s MOUNT -- chroot "$1/partial-${arch}" apt-get dist-upgrade -y || { suggest_flush; false; }
+    lxc-unshare -s MOUNT -- chroot "$1/partial-${arch}" apt-get dist-upgrade -y --force-yes || { suggest_flush; false; }
     rm -f "$1/partial-${arch}"/usr/sbin/policy-rc.d
 
     chroot "$1/partial-${arch}" apt-get clean
```

Чтоб не писать о настройке DNS, на __server__ добавим в `/etc/hosts`:

    172.16.0.1      server site
    192.168.9.1     gentoo01-gw
    192.168.9.2     gentoo01 cocaine01
    192.168.9.5     gentoo02-gw
    192.168.9.6     gentoo02 cocaine02
    192.168.9.9     ubuntu01-gw
    192.168.9.10    ubuntu01 cocaine03
    192.168.9.13    ubuntu02-gw
    192.168.9.14    ubuntu02 cocaine04
    192.168.9.17    gentoo03-gw
    192.168.9.18    gentoo03 elliptics
    192.168.9.21    gentoo04-gw
    192.168.9.22    gentoo04 mongo

К именам хостов с названием системы в имени добавим алиасы с функциональным именем, их и будем использовать в
конфигурации.

Этот же `/etc/hosts` распространим по всем виртуальным машинам.

Если Вы самостоятельно справились с поднятием виртуалок, то в памяти гудят воображаемыми кулерами 4 операционных
системы доступных по ssh, и сеть настроена как на Рис. 1. Теперь это наш собственный мини-дата-центр.

Если требуется подсказка, можно посмотреть как это было у меня.

---

Устанавливаем пакет с инструментами дляработы с LXC:

```
echo "=app-emulation/lxc-0.8.0-r1 ~x86" >> /etc/portage/package.accept_keywords
emerge app-emulation/lxc
```

```
#FIXIT:
emerge layman
echo "source /var/lib/layman/make.conf" >> /etc/portage/make.conf
layman -a hacking-gentoo
echo "=sys-apps/openrc-runhooks-plugin-0.1 ~x86" >> /etc/portage/package.accept_keywords
echo "=sys-apps/openrc-runhooks-plugin-0.1" >> /etc/portage/package.unmask
emerge sys-apps/openrc-runhooks-plugin
```

В качестве ядра использую пакет `sys-kernel/gentoo-sources-3.8.13`, по этому все действия будем выполнять с ним.
Запускаем конфигуратор и включаем в ядре всё для запуска `lxc`.

```
General setup  --->
  [*] Control Group support  --->
    [*]   Freezer cgroup subsystem
    [*]   Device controller for cgroups
    [*]   Cpuset support
    [*]     Include legacy /proc/<pid>/cpuset file
    [*]   Simple CPU accounting cgroup subsystem
    [*]   Resource counters
    [*]     Memory Resource Controller for Control Groups
    [*]       Memory Resource Controller Swap Extension
    [*]         Memory Resource Controller Swap Extension enabled by default
    [*]       Memory Resource Controller Kernel Memory accounting (EXPERIMENTAL)
    [*]   Enable perf_event per-cpu per-container group (cgroup) monitoring
    [*]   Group CPU scheduler  --->
      [*]   Group scheduling for SCHED_OTHER
      [*]   Group scheduling for SCHED_RR/FIFO
    [*]   Block IO controller
  [*] Namespaces support  --->
    [*]   UTS namespace
    [*]   IPC namespace
    [*]   PID Namespaces
    [*]   Network namespace
[*] Networking support  --->
  Networking options  --->
    <M> 802.1d Ethernet Bridging
    <M> 802.1Q VLAN Support
Device Drivers  --->
  [*] Network device support  --->
    -*-   Network core driver support
    <M>     MAC-VLAN support (EXPERIMENTAL)
    <M>     Virtual ethernet pair device
  Character devices  --->
    [*] Unix98 PTY support
    [*]   Support multiple instances of devpts
```

Заодно подключим модулями свё, что может пригодится для IPVS

```
[*] Networking support  --->
  Networking options  --->
    [*] Network packet filtering framework (Netfilter)  --->
      <M>   IP virtual server support  --->
        [*]   IPv6 support for IPVS
        (12)  IPVS connection table size (the Nth power of 2)
              *** IPVS transport protocol load balancing support ***
        [*]   TCP load balancing support
        [*]   UDP load balancing support
              *** IPVS scheduler ***
        <M>   round-robin scheduling
        <M>   weighted round-robin scheduling
        <M>   least-connection scheduling
        <M>   weighted least-connection scheduling
        <M>   locality-based least-connection scheduling
        <M>   locality-based least-connection with replication scheduling
        <M>   destination hashing scheduling
        <M>   source hashing scheduling
        <M>   shortest expected delay scheduling
        <M>   never queue scheduling
              *** IPVS SH scheduler ***
        (8)   IPVS source hashing table size (the Nth power of 2)
              *** IPVS application helper ***
        [*]   Netfilter connection tracking
```

Пакет `app-emulation/lxc` содержит скрипт для проверк конфигурации ядра `lxc-checkconfig` который помогает
провалидировать включение необходимых опций:

```
CONFIG=/usr/src/linux-3.8.13-gentoo/.config lxc-checkconfig
```

У меня показало всё зеленым кроме `User namespace: missing` и `Cgroup memory controller: missing`. Не беда, если
вникать, то так и должно быть.

Следующий шаг, собираем ядро и модули, прописываем новое ядро в загрузку, перезагружем. 

<!-- TODO: перенести -->

/etc/conf.d/net

    config_gentoo01="192.168.9.1/30"
    config_gentoo02="192.168.9.5/30"
    config_ubuntu01="192.168.9.9/30"
    config_ubuntu02="192.168.9.13/30"

##### Gentoo

###### Установка

```
git clone git://github.com/globalcitizen/lxc-gentoo
cp lxc-gentoo/lxc-gentoo /usr/local/bin
mkdir /var/lxc
cd /var/lxc
NAME=gentoo01 UTSNAME=gentoo01 IPV4=192.168.9.2/30 GATEWAY=192.168.9.1 CONFFILE=/etc/lxc/gentoo01.conf lxc-gentoo create
Select desired container architecture:
2) amd64
10) x86
Select desired container subarchitecture/variant for amd64:
4) amd64
3) i686
...
mkdir /etc/lxc/gentoo01
cd /etc/lxc/gentoo01
ln -s ../gentoo01.conf config
```

Добавляем симлинки для дальнйшей работы средствами OpenRC:

```
cd /etc/init.d
ln -s lxc lxc.gentoo01
ln -s net.lo net.gentoo01
```

###### Настройка

Проверяем установку, меняем пароль/добавляем пользователей, ставим в автозапуск ssh, по вкусу правим 

```
lxc-start -n "gentoo01"
passwd
...
rc-update add sshd default
```

При вкусу вносим изменения в конфигурацию сборки пакетов Gentoo `nano /etc/portage/make.conf`

Для дальнейшей простоты изложения проделываю дырку для root в sshd контейнера:
`echo "PermitRootLogin yes" >> /etc/ssh/sshd_config`

Из родительской консоли останавливаю контейнер `lxc-stop -n "gentoo01"`

###### Запус и остановка виртуальных машин

Последовательность ручного запуска следующая, сначала запускаем контейнеры, затем поднимаем интерфейсы:

```
/etc/init.d/lxc.gentoo01 start
/etc/init.d/lxc.gentoo02 start
/etc/init.d/net.gentoo01 start
/etc/init.d/net.gentoo02 start
```

Причина такой последовательности в том, что во время запуска lxc-контейнера автоматически поднимаются сетевые
интерфейсы с обеих сторон (тунель). При этом внутренний интерфейс lxc получает сетевые настройки из конфигурации
lxc-контейнера, а внешний интерфейс требуется конфигурировать.

Подобная схема запуска не годится для автоматической загрузки/выгрузки используя стандартные средства без
использования костылей. Особо не разбирался с этим вопросом, предложения приветствуются.

Чтоб убедится, что у нас всё получилось, пробуем зайти на виртуальный сервер `ssh gentoo01`. Также, для внешнего
мониторинга можно использовать инструменты `lxc-console`, `lxc-monitor`, `lxc-info` и `lxc-ps`.

Остановка виртульных машин выполняется в тойже последовательности, что и запуск! Сначала останавливаем контейнер,
потом опускаем интерфейс:

```
/etc/init.d/lxc.gentoo01 stop
/etc/init.d/lxc.gentoo02 stop
/etc/init.d/net.gentoo01 stop
/etc/init.d/net.gentoo02 stop
```

##### Ubuntu

    emerge dev-util/debootstrap

    /etc/lxc/lxc.conf
    lxc.arch = i686
    lxc.network.type = veth
    lxc.network.flags = up
    lxc.network.name = br0

```
--- /usr/share/lxc/templates/lxc-ubuntu~	2013-07-10 09:34:33.000000000 +0300
+++ /usr/share/lxc/templates/lxc-ubuntu	2013-07-11 14:16:51.000000000 +0300
@@ -205,7 +205,7 @@
 EOF
     chmod +x "$1/partial-${arch}"/usr/sbin/policy-rc.d
 
-    lxc-unshare -s MOUNT -- chroot "$1/partial-${arch}" apt-get dist-upgrade -y || { suggest_flush; false; }
+    lxc-unshare -s MOUNT -- chroot "$1/partial-${arch}" apt-get dist-upgrade -y --force-yes || { suggest_flush; false; }
     rm -f "$1/partial-${arch}"/usr/sbin/policy-rc.d
 
     chroot "$1/partial-${arch}" apt-get clean

```

    lxc-create -t ubuntu -n ubuntu01 -f /etc/lxc/lxc.conf -- -r precise -d -p /var/lxc/ubuntu01
    ln -sf /var/lxc/ubuntu01/config /etc/lxc/ubuntu01/config

Добавляем в начало полученного конфига `/etc/lxc/ubuntu01/config` знания о сети:

    # network interface
    lxc.network.type = veth
    lxc.network.flags = up
    lxc.network.name = br0
    lxc.network.veth.pair = ubuntu01
    lxc.network.ipv4 = 192.168.9.10/30
    lxc.network.ipv4.gateway = 192.168.9.9

??? `lxc-console -n "ubuntu01"`

Запускаем из консоли и меняем пароль root. В `/var/lxc/ubuntu01/rootfs/etc/shadow` удаляем `*` в строчке с root.

У меня запуск контейнера подвисал на 2-3 минуты, наверно проще было воспользоваться `chroot`.

    lxc-start -n "ubuntu01"


    root@ubuntu01:~# passwd
    …

Незабываем прибить стандартного пользователя `ubuntu` или сменить ему пароль


Из родительской консоли останавливаю контейнер

    lxc-stop -n "ubuntu01"

Запускаем

    cd /etc/init.d
    ln -s lxc lxc.ubuntu01
    ln -s lxc lxc.ubuntu02
    ln -s net.lo net.ubuntu01
    ln -s net.lo net.ubuntu02

    /etc/init.d/lxc.ubuntu01 start
    /etc/init.d/lxc.ubuntu02 start
    /etc/init.d/net.ubuntu01 start
    /etc/init.d/net.ubuntu02 start

    ssh ubuntu01


```
/etc/init.d/lxc.ubuntu01 stop
/etc/init.d/lxc.ubuntu02 stop
/etc/init.d/net.ubuntu01 stop
/etc/init.d/net.ubuntu02 stop
```

Знание о разворачивании голой виртуалки или изолированной среды дает много полезностей рядовому программисту.

---

Приступим к созданию облака.

Создаем минимальное окружение для сборки облачного ПО.

| Ubuntu | Gentoo |
|--------|--------|
|        |        |


`apt-get install …`

** Если кто из гуру Ubuntu подскажет как проще установить зависимые библиотеки, буду безмерно благодарен*

#### Сборка

Согласно инструкции

Так как мы хотим попробовать, намерено изменяем конфигурационный файл для возможности мониторинга состояния.

#### Запуск


    "loggers": {
        "core": {
            "type": "files",
            "args": {
                "path": "/dev/stdout",
                "identity": "cocaine",
                "verbosity": "debug"
            }
        }
    }

    cocaine-runtime -c /usr/local/etc/cocaine-runtime.conf



#### Проверка

`"Hello from Cloud"`
Шампанское, салют!

~~Инкримент цифры на стороне сервера~~

<!-- напишем простенький пример, проверим, а потом погрузим в облако -->

#### Выкатка в продакшен (тема другой статьи)


~~на github выложить библиотеку с подключаемым примером к project-stub для быстрого разворачивания и проверки~~

На самом деле за два часа не построить полноценную облачную платформу для высоконагруженных проектов, за кадром
осталось тестирование, масштабирование, балансировка, мониторинг, резервирование и другие инфраструктурные и
архитектурные вопросы, которые совсем не про Cocaine и БЭМ. Но буду очень рад, если данное изложение кому-нибудь
поможет одной ногой вступить в океан знаний о высоконагруженных системах.





---


apt-get update
apt-get install git apt-file python-software-properties

add-apt-repository ppa:chris-lea/node.js
apt-cache policy nodejs
apt-get install nodejs
node -v

apt-get install python-setuptools
apt-get install cmake cdbs debhelper libev-dev libmsgpack-dev libboost-dev libboost-filesystem-dev libboost-program-options-dev libssl-dev libltdl-dev uuid-dev libarchive-dev binutils-dev
    
    echo 'USE="$USE python"' >> /etc/portage/make.conf
    emerge dev-python/setuptools
    emerge dev-util/cmake dev-libs/libev dev-libs/msgpack dev-python/msgpack dev-libs/boost app-arch/libarchive dev-vcs/git net-libs/nodejs

    # Если попросит 
    /etc/portage/package.accept_keywords


git clone https://github.com/cocaine/cocaine-core.git -b v0.10
cd cocaine-core
?git checkout -b 0.10.5?

 # Совместимость с binutils 2.23
 в 

git submodule init
git submodule update
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=/usr/local
make
make install
ldconfig
cp debian/cocaine-runtime.conf /usr/local/etc/cocaine-runtime.conf
mkdir /usr/lib/cocaine /var/run/cocaine /var/spool/cocaine /var/lib/cocaine /var/cache/cocaine

cd ~
git clone https://github.com/cocaine/cocaine-framework-python.git
cd cocaine-framework-python
python setup.py install
cd ~

// Запуск демона в отдельном терминале в режиме отладки

cp /usr/local/etc/cocaine-runtime.conf /usr/local/etc/cocaine-runtime.conf.orig
vim /usr/local/etc/cocaine-runtime.conf

```
--- cocaine-runtime.conf.orig	2013-07-12 21:53:18.000000000 +0300
+++ cocaine-runtime.conf	2013-07-12 22:01:30.000000000 +0300
@@ -38,10 +38,11 @@
     },
     "loggers": {
         "core": {
-            "type": "syslog",
+            "type": "files",
             "args": {
+                "path": "/dev/stdout",
                 "identity": "cocaine",
-                "verbosity": "info"
+                "verbosity": "debug"
             }
         }
     }
```

echo "192.168.9.2 gentoo01" >> /etc/hosts
cocaine-runtime -c /usr/local/etc/cocaine-runtime.conf

---

    eselect python set python2.7
    npm install -g node-gyp

mkdir app
mkdir app/node_modules
cd app/node_modules

    #git clone https://github.com/cocaine/cocaine-framework-nodejs.git
    #cd cocaine-framework-nodejs
    #git checkout final-fixes
git clone https://github.com/diunko/no-cocaine.git -b final-fixes cocaine-framework-nodejs
cd cocaine-framework-nodejs
node-gyp rebuild
cd ../..
npm install q
npm install msgpack
npm install bindings
node
>require("cocaine-framework-nodejs");
^D
npm install optimist

cp node_modules/cocaine-framework-nodejs/sample/app.http.js index.js
 # меняем путь к ноде
 # #!/usr/bin/env node
 # и 
 # var co = require("..")
 # на
 # var co = require("cocaine-framework-nodejs")

tar -zcf ../app.tar.gz *
./manifest.json
…
./profile.json
…
./runlist.json
…

cocaine-tool app upload -n hello --package app.tar.gz --manifest manifest.json
cocaine-tool profile upload -n hello --profile profile.json
cocaine-tool app start -n hello -r hello

ежели чего сносим
rm /var/run/cocaine/hello


 # Запускаем прокси
cd ~/app/node_modules/cocaine-framework-nodejs/
node sample/proxy.js

---
 # другой вариант прокси
git clone https://github.com/diunko/cocaine-http-proxy-nodejs.git
cd cocaine-http-proxy-nodejs
mkdir node_modules
npm install optimist
ln -s ~/app/node_modules/cocaine-framework-nodejs node_modules/cocaine-framework
ln -s .. node_modules/cocaine-http-proxy
./bin/cocaine-proxy.js

---

 # локально или из родителя
curl -X PUT -v -H "X-smth:value" http://gentoo01:8181/hello/hash/ --data-binary XxXxXxXxYxXy




apt-get install bash-completion
apt-get install libboost-thread-dev
c++filt _ZN7cocaine2io15make_error_codeENS0_8rpc_errcE
ldd build/Release/nodejs_cocaine_framework.node
nm build/Release/nodejs_cocaine_framework.node |grep make_error_code



# IPVS

http://wiki.yandex-team.ru/paulus/ipvs

# mongo

```
echo "=dev-db/mongodb-2.4.5 ~x86" >> /etc/portage/ackage.accept_keywords
echo "=dev-libs/boost-1.53.0 ~x86" >> /etc/portage/ackage.accept_keywords
echo "=dev-util/boost-build-1.53.0 ~x86" >> /etc/portage/ackage.accept_keywords
emerge mongodb
rc-update add mongodb default
nano /etc/hosts
#ping mongodb.host
nano /etc/conf.d/mongodb
/etc/init.d/mongodb start
#mongo --host mongodb.host
```

### Что дальше?

А дальше тонкая настройка и тестирование каждой технологии в отдельности, проектирование и строительство инфраструктуры,
горы железа, информации, времени и труда.

А Вы что думали? Будет просто?
