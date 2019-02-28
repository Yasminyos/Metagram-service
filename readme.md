## MetagramService

Создатель: Одинцов Ян

Почта: yanodincov@gmail.com

### Вкратце о сервисе

Сервис для поиска связей между словами из 4 букв путем замены у них одной буквы за итерацию. 

Для получения списка слов из папки 'storage/app/public/dict' собирается содержание всех словарей, затем из них выбираются все слова с 4-мя буквами. После получения списка слов составляется список всех пар слов, которые отличаются на 1 букву.

Поиск осуществляется путем перебора цепочек слов с помощью алгоритма поиска в графе в ширину.

Используемые технологии:
+ Docker
+ Docker Compose
+ Nginx
+ PHP 7
+ Composer
+ Laravel 5.7
+ MySQL
+ VueJS

### Установка

Для установки сервиса необходимо установить [Docker](https://docs.docker.com/v17.12/install/#server) и [Docker Compose](https://docs.docker.com/compose/install/).

После установки зайдите в папку проекта и запустите образ с помощью Docker Compose:
      
    docker-compose up -d

После установки и запуска образа необходимо выполнить следующую команду для устранения зависимостей с помощью Composer:
    
    docker exec metagram-php composer install --prefer-source --no-interaction

Также нужно запустить миграции:

    docker exec -u 33 metagram-php php artisan migrate

После этого по адресу [localhost:8080](http://127.0.0.1:8080) будет доступен сайт сервиса. Перед началом использования сервиса необходимо нажать на кнопку "Сгенерировать таблицу слов и свзяей", которая находится под кнопкой "Найти связь" (процесс сбора слов из словарей и поиска пар занимает не более 2-х минут).

### Процесс создания сервиса

1. Для начала на локальной машине, заранее подготовленной к веб разработке в папке проекта был установлен и настроен Laravel.

2. Были созданы две модели и прилагающиеся к ним файлы миграции: **Word** и **WordChain**. Модель Word используется для хранения списка слов. Модель WordChain используется для хранения пар слов, отличающихся от друг-друга на 1 букуву. Модель WordChain имеет связь с моделбю Word типа hasOne от столбцов `word_first` и `word_second` к `id` в Word.

3. Был создан сервис **MetagramService**. Сервис был добавлен в сервис-контейнер. Внутри сервиса реализуем следующие методы:
	+ **parseWords** Принимает переменную сожержащую путь к папке с словарями (*/storage/app/public/dict*). Возвращает все собранные слова из 4-ех букв.
	+ **generateWordChain** Принимает массив слов из 4-ех букв. Возвращает массив пар слов, которые отличаются друг от друга на 1 букву.
	+ **generateBaseData** Очищает таблицы  содержащие список слов и список связей. Вызывает методы **parseWords** и **generateWordChain** и заносит полученные данные в соответсвтующие таблицы.
	+ **findWay** Принимает id или массив с id слова, от которого нужно начать поиск пути и id слова до которого нужно найти путь. Рекусивная функция. Для поиска пути используется алгорит поиска по графу "в ширину". Возвращает массив из последовательных id слов.
	+ **findChain** Принимает два слова - слово 1 от которого нужно найти путь до слова 2. Производит поиск данных слов в модели **Word** и передает их id в метод **findWay**, обрабатывает полученный результат и возращает массив из цепочки слов.
	
4. Был создан контоллер **MetogramController** для отображения страниц "ввода слов" и "генерации данных" и передачи инфорамции с этих страниц соответсвующим методам в **MetagramService**.

5. Был создан шаблон страницы воода слов */resource/views/metragram/index.blade.php*.

6. Проект был запакован в Docker контейнер. Был создан образ LNMP сервера. Проект при запуске копируется внутрь контейнера, где Docker создает локальный том *metagram-app* доступ к которому имеют все необходимые сервисы.

7. Была создана страница для поиска связей и страница генерации стартовых данных. В новой версии они стали работать через POST-запрос и возвращать ответ в JSON формате.

8. Блоки ввода и вывода сообщений перенесены во Vue компонент. Теперь поиск связей между словами и генерация данных происходит через Ajax-запросы (библиотека "axios").

