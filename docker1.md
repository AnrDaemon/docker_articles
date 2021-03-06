Docker myths and receipts. Introduction.
========

Вокруг постоянно говорят про Docker. Это что-то про контейнеры, виртуализацию, облака. У вас все и так работает. Это все баловство. Он не запустится на вашем старом ядре линукса. Точно так же можно подготовить образ для облака и запустить его. Можно просто настроить LXC, chroot или AppArmor. Вам он не нужен. Очередная модная штука. В конце концов, просто лень разобираться. 
Но любопытно! Тогда, читайте.

Если вы не слышали о контейнерах в Линуксе, вот список страниц, которые надо прочитать чтобы понимать о чем вообще здесь речь.
https://en.wikipedia.org/wiki/LXC
https://en.wikipedia.org/wiki/UnionFS
http://habrahabr.ru/post/253877/ или https://www.docker.com/whatisdocker
Поставьте Docker, он небольшой. Для Windows и Mac можно просто поставить Toolbox: https://www.docker.com/toolbox
Создавать виртуальную машину и настраивать надо из командной строки, не 
Заметки для пользоватлей MAC OS [здесь](https://github.com/grikdotnet/docker_articles/blob/master/docker_mac.md)
Прочитайте несколько уроков из мануала, здесь я написал о том, чего в документации нет.

**Docker - это не виртуализация.**

Вот какой у меня линукс:
```
Welcome to Ubuntu 15.04 (GNU/Linux 3.19.0-15-generic x86_64)

Last login: Tue Aug 18 00:43:50 2015 from 192.168.48.1
gri@ubuntu:~$ uname -a
Linux ubuntu 3.19.0-15-generic #15-Ubuntu SMP Thu Apr 16 23:32:37 UTC 2015 x86_64 x86_64 x86_64 GNU/                                       Linux
gri@ubuntu:~$ free -h
             total       used       free     shared    buffers     cached
Mem:          976M       866M       109M        11M       110M       514M
-/+ buffers/cache:       241M       735M
Swap:         1.0G       1.0M       1.0G
```
Запускаю CentOS
```
gri@ubuntu:~$ docker run -ti centos
[root@301fc721eeb9 /]# uname -a
Linux 301fc721eeb9 3.19.0-15-generic #15-Ubuntu SMP Thu Apr 16 23:32:37 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
[root@301fc721eeb9 /]# cat /etc/redhat-release
CentOS Linux release 7.1.1503 (Core)
[root@301fc721eeb9 /]# free -h
              total        used        free      shared  buff/cache   available
Mem:           976M         85M        100M         12M        790M        677M
Swap:          1.0G        1.0M        1.0G

```
То же ядро, та же память, а дистрибутив и файловая система - разные.

Docker - это не chroot, их функционал частично совпадает. Это не система безопасности вроде AppArmor. Docker использует те же контейнеры, что и LXC, но интересен он не контейнерами. Docker - это ничего из того, что я думал о нем до того, как прочитал документацию.

**Docker - это инструмент объекто-ориентированного проектирования.**

Регулярно возникают вопрос является ли конфигурация nginx частью веб-приложения. Системные администраторы ругаются с разработчиками.
Но недавно в мире появились devops и захотели вместо последовательно-процедурного вызова команд из bash думать привычным OOP.
Docker дает инкапсуляцию, наследование и полиморфизмом компонентам системы, таким как база данных и данные.
Это значит, что можно провести декомпозицию всей информационной системы, выделить приложение, web-сервер, базу данных, системные библиотеки, рабочие данные в независимые компоненты, инжектить зависимости из конфигов, и заставить все это работать одной группой, одинаково на разных компьютерах.

Это можно использовать чтобы снизить потери рабочего времени дорогих front-end рахработчиков на настройку базы данных и nginx.
Чтобы уйти от vendor lock-in. Не обломаться когда openssl на сервере не поддерживает cipher, используемый в API госучреждения.
Чтобы приложение работало независимо от версии PHP или Python на сервере заказчика.
Создавать open source не только в виде кода, но и настройкой пакетов из нескольких приложений, написанных на разных языках, работающих на разных слоях OSI.

**Начало**

Итак, я открыл https://docs.docker.com/mac/started/, поставил Docker, выполнил несколько упражнений, и почувствовал, что меня держат за дурачка-двоечника, которого боятся перегрузить информацией.
Первый вопрос: куда это чертов докер поставился, где лежат его файлы, в каком формате, как оно все устроено?
Ответы здесь: http://blog.thoward37.me/articles/where-are-docker-images-stored/

Если вкратце, у Docker несколько драйверов для работы с файловой системой, обычно это AUFS, и все файлы контейнеров лежат в /var/lib/docker/aufs/diff/.
В /var/lib/docker/containers/ служебная информация, а не сами файлы контейнеров.

Изображения - это как классы в коде. Контейнеры - как объекты, созданные из классов. Основное отличие - контейнер можно закомитить и сделать из него образ. 
Образы состоят из так называемых слоев, слои - это папки, которые лежат в /var/lib/docker/aufs/diff/. Обычно образы приложений наследуют какие-то готовые официальные системные образы. Когда Docker скачивает образ, ему нужны только те слои, которых у него нет.

Например, скачаю я официальный образ nginx: https://hub.docker.com/r/library/nginx/tags/
```
docker@dev:~$ docker pull nginx
latest: Pulling from nginx
aface2a79f55: Pull complete
5dd2638d10a1: Pull complete
97df1ddba09e: Pull complete
e7e7a55e9264: Pull complete
72b67c8ad0ca: Downloading [=============>                                     ] 883.6 kB/3.386 MB
9108e25be489: Download complete
6dda3f3a8c05: Download complete
42d2189f6cbe: Download complete
3cb7f49c6bc4: Download complete
a486da044a3f: Download complete
902b87aaaec9: Already exists
9a61b6b1315e: Already exists
```
Пишут, что у образа nginx 1.9.4 размер 52 мб, а качает он всего 3 мб, потому что Nginx собран на образе debian:jessie, который у меня "Already exists".
Есть много образов на базе ubuntu. Конечно, стоит собирать свою систему из образов с одним предком.

**Docker - это системная служба с клиентским приложением**

Служба Docker управляет контейнерами - запускает по команде, полученной от клиентского приложения `docker` и останавливает его когда в контейнере освобождается поток стандартного ввода-вывода. Поэтому в конфигурации Nginx для Docker [пишут](https://hub.docker.com/_/nginx/): 

> Be sure to include daemon off; in your custom configuration to ensure that Nginx stays in the foreground so that Docker can track the process properly (otherwise your container will stop immediately after starting)!

Соответственно, служба Docker может и зависнуть. Например, если в версии 1.8.1 при скачивании образа пропадает связь, может потребоваться перезапустить службу.
```
docker@dev:~$ docker pull debian
Using default tag: latest
latest: Pulling from library/debian
2c49f83e0b13: Downloading [===================>                               ] 19.89 MB/51.37 MB
4a5e6db8c069: Download complete
```
Нажимаю Ctrl-C, затем сразу запускаю скачивание повторно.
```
docker@dev:~$ docker pull debian
Using default tag: latest
```
Картина Репина "Приплыли". То есть, зависли. Надо перезапустить демон.
```
docker@dev:~$ sudo /etc/init.d/docker restart
Need TLS certs for dev,127.0.0.1,10.0.2.15,192.168.99.104
-------------------
docker@dev:~$ sudo /etc/init.d/docker status
Docker daemon is running
docker@dev:~$ docker pull debian
Using default tag: latest
latest: Pulling from library/debian
...
Status: Downloaded newer image for debian:latest
```
Бывает, что при демон Docker не хочет умирать самостоятельно и не освобождает порт, а init-скрипт пограничные случаи еще не отрабатывает.
Так что не забывайте проверять `sudo /etc/init.d/docker status`, `sudo netstat -ntpl`, доставайте бубен и танцуйте.

Еще надо помнить, что порядок операторов для команды docker имеет значение. Если написать `docker create nginx --name=nginx`, --name=nginx будет считаться командой, которую надо выполнить в контейнере, а не именем контейнера.

Теперь вам будет проще разбираться с официальной документацией.

Продолжение: https://github.com/grikdotnet/docker_articles/blob/master/docker2.md