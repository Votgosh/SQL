# Домашнее задание к занятию «3.2. SQL»

## Volumes

Пожалуйста, ознакомьтесь с кратким руководством по работе с [Volumes](volumes.md)

## SQL

Пожалуйста, ознакомьтесь с кратким руководством по работе с клиентами [SQL](mysql-psql.md)

## Задача №1 - Скоро deadline

Случилось то, что обычно случается ближе к дедлайну: никто ничего не успевает и винит во всём остальных других.

Разработчикам не особо до вас ("им ведь нужно пилить новые фичи"), поэтому они подготовили для вас сборку, работающую с СУБД и даже приложили схему БД (см. файл [schema.sql](schema.sql)), но при этом сказали "остальное вам нужно сделать самим, там не сложно" 😈.

Что вам нужно сделать:
1. Внимательно изучить схему
1. Создать Docker Container на базе MySQL 8 (прописать создание БД, пользователя, пароля)
1. Запустить SUT ([app-deadline.jar](app-deadline.jar)): для указания параметров подключения к БД можно использовать:
- либо переменные окружения `DB_URL`, `DB_USER`, `DB_PASS`
- либо указать их через флаги командной строки при запуске: `-P:jdbc.url=...`, `-P:jdbc.user=...`, `-P:jdbc.password=...` (внимание: при запуске флаги не нужно указывать через запятую!). Данное приложение не использует файл `application.properties` в качестве конфигурации, конфигурационный файл находится внутри jar архива.
- либо можете схитрить и попробовать подобрать значения, зашитые в саму SUT

А дальше выясняется куча забавных вещей 😈. Рекомендуем вам попробовать разобраться самим, но если будет сложно, загляните в подсказку.

### Проблема первая: SUT не стартует

<details>
   <summary>Подсказка</summary>

   Проблема: SUT не создаёт самостоятельно таблицы в БД.

   Поэтому вам нужно сходить на сайт-описание Docker Image MySQL и посмотреть, как при инициализации скармливать схему (будет использоваться технология volumes).
</details>

### Проблема вторая: SUT валится при повторном перезапуске

<details>
   <summary>Подсказка</summary>

   Проблема: SUT вставляет в БД демо-данные, а поскольку там есть ограничение уникальности, это приводит к ошибкам.

   Поэтому вам нужно где-то настроить вычистку данных за SUT.
</details>

### Проблема третья (опциональна): пароли

Если вы решите вдруг генерировать пользователей, чтобы под ними тестировать "Вход в приложение", то не должны удивляться тому, что в базе данных пароль пользователя хранится в зашифрованном виде.

Попытка его записать туда в открытом виде ни к чему хорошему не приведёт.

Настойчивые требования к разработчикам "раскрыть" алгоритм генерации пароля - ни к чему не привели.

Что же делать?

<details>
   <summary>Подсказка</summary>

   Если вы внимательно присмотритесь к демо-данным, то они очень похожи (прямо подозрительно) на те, что были в одной из предыдущих задач.

   Значит вы можете попробовать использовать уже готовые "зашифрованные пароли", зная то, какие они были в незашифрованном виде.
</details>

Если вы добрались до этого шага и всё-таки успешно запустили SUT, то вы уже герой!

Но теперь выяснилась следующая забавная информация: разработчики фронтенда поругались с разработчиками бэкенда и вы можете протестировать только "Вход в систему".

Внимательно посмотрите, как и куда сохраняются коды генерации в СУБД и напишите тест, который взяв информацию из БД о сгенерированном коде позволит вам протестировать "Вход в систему" через веб-интерфейс.

P.S. Неплохо бы ещё проверить, что при трёхкратном неверном вводе пароля система блокируется.

Итого в результате у вас должно получиться:
1. docker-compose.yml*
1. app-deadline.jar
1. schema.sql
1. код ваших авто-тестов

Если ваша система не поддерживает Docker, то вам придётся (к сожалению) вручную установить MySQL на свой компьютер и отрабатывать тесты уже на ней. В этом случае положите в репозиторий файлик `README.md`, в котором опишите последовательность действий (со скриншотами) для установки сервера MySQL и загрузки в него файла `schema.sql`.
