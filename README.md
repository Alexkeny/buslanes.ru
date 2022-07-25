# Рейтинг выделенных полос

В этом контейнере (виртуальной машине) Docker лежат все скрипты, необходимые чтобы посчитать показатели рейтинга и сверстать страницу сайта. Вы можете взять эти файлы и пользоваться ими только для расчётов.

Для удобства, здесь описано, как работает весь цикл.

[Сайт с рейтингом](http://buslanes.peshemove.org/).

# Предварительная подготовка

Скрипты сайта можно выполнять либо прямо на вашей машине (если есть Python 3.8 и нужные пакеты), либо в контейнере Докера.

## Вариант 1. Прямо в системе

### Установка зависимостей

Системные зависимости в Ubuntu:

    $ sudo apt-get install libspatialindex-dev libgdal-dev libgeos-dev python3-dev

Зависимости пакетов Питона:

    $ pip3 install -r requirements.txt

В файле зафиксированы версии следующие пакетов, если эти пакеты уже есть, удалите из `requirements.txt` строку с именем пакета и установите той же командой:

    Jinja2==3.0.3
    Rtree==0.9.7
    Shapely==1.8.1.post1
    argh==0.26.2
    decorator==5.1.1
    fastkml==0.12
    geojson==2.5.0
    geopandas==0.10.2
    lxml==4.9.1
    polyline==1.4.0

### Запуск

Скачиваете файлы KML и кладёте их в `/tmp/`. Запускаете скрипты:

    $ cd calc
    $ python3 main.py

(могут быть ошибки в путях, т.к. этот проект обычно работает в контейнере, некоторые пути без контейнера могут не совпадать)

Если надо править шаблоны и только обновлять страницу, запускаете

    $ python3 render.py html/index.template.html build/bus-lanes.geojson build/bus-lanes.csv build/index.html

### Просмотр результата

В корне проекта:

    $ sh server.sh

Затем открываете в браузере адрес [http://localhost:8000](http://localhost:8000).

### Загрузка на хостинг

Если всё устраивает, на хостинг надо выложить только файлы в папке `calc/build`. Например, при помощи `rsync`:

    $ rsync -ave ssh calc/build/*  вы@ваш_хостинг:~/папка_сайта --delete

## Вариант 2. Через Docker

### Установка Docker

1. Скачайте и установите Docker [Community Edition](https://www.docker.com/community-edition/) (Docker CE).
2. Скачайте и установите [Docker-Compose](https://docs.docker.com/compose/install/)

Теперь можно собирать контейнер.

### Сборка контейнера

Зайдите в командной строке в корневую папку проекта dedicated-lanes (в которой лежит `docker-compose.yml`) и запустите сборку командой:

    $ docker-compose build

Вы увидите как Docker скачивает образ python:3.8, устанавливает бинарные пакеты через apt, и питоновские через pip.

### Запуск контейнера

Можно не менять параметров скриптов и сразу запустить выполнение расчётов.

    $ docker-compose up

Эта команда запустит контейнер и в нём - команду `python3 /calc/main.py`, которая посчитает данные и соберёт веб-страницу. Если всё прошло хорошо, файлы с результатами будут лежать в папке `calculator/build`.

### Просмотр получившегося рейтинга

В корне проекта:

    $ sh server.sh

Затем открываете в браузере адрес [http://localhost:8000](http://localhost:8000).

### Загрузка на хостинг

Если всё устраивает, выкладываем на хостинг как и в варианте №1:

    $ rsync -ave ssh calc/build/*  вы@ваш_хостинг:~/папка_сайта --delete

# О данных и технологии

## Входные данные

Карта с выделенками: https://www.google.com/maps/d/kml?mid=1PWvRWfdKEV4SVs5ONkDgRPLbRiQ

Имена слоёв должны совпадать с тем, что написано в , в файле `calc/main.py`:

    if f.name == 'односторонние сущ.' or f.name == 'Существующие. Односторонние':
        lanes = 1
    elif f.name == 'двусторонние сущ.' or f.name == 'Существующие':
        lanes = 2

Нужно будет вставить сюда имя слоя в кавычках.

3. Контуры муниципалитетов лежат в файле `calc/src/muni.geojson`. Его можно редактировать в QGIS или ArcGIS или вставить на его место новый. Новый файл должен иметь систему координат 4326 (WGS84), обратите на это внимание, когда сохраняете файл. Объекты в этом файле должны иметь названия текстом.

4. Численность населения находится в `calc/src/city-population.csv`. Чтобы скрипт взял отсюда численность населения, имя города в этом файле должно быть в поле имени города в названном выше файле `muni.geojson`, например для строки с именем `Иваново` скрипт `main.py` найдёт объект с названием `городской округ Иваново` (регистр букв не важен). Скрипт может найти города, заканчивающиеся на мягкий знак, например, Пермь-Пермский [городской округ].

## Веб-страница

Скрипт складывает файлы GeoJSON в папку `calculator/build`, и из них данные вставляются в шаблон `calculator/html/index.template.html`. Язык шаблона - [Jinja2](http://jinja.pocoo.org/docs/2.9/).


[Дмитрий Лебедев](http://dl.one-giant-leap.info), 2017-2022.
