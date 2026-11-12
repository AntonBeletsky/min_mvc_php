# min_php_mvc

Каркас минимального MVC-фреймворка на PHP: дерево каталогов и файлов под десять функций ядра — единая точка входа, запрос и ответ, маршрутизация, диспетчеризация, конвейер middleware, контейнер зависимостей, конфигурация, шаблоны, доступ к данным, обработка ошибок и журнал.

Файлы PHP являются заготовками: каждый содержит объявление пространства имён и пустого класса, файлы каталога `config/` возвращают пустые массивы, `bootstrap.php` и `public/index.php` содержат только строгую типизацию. Шаблоны в `resources/views/` и файлы `.gitkeep` пусты.

## Дерево

```text
min_php_mvc/
├── app/                # код приложения, пространство имён `App\`
│   ├── Controllers/
│   │   └── HomeController.php
│   └── Models/
│       └── .gitkeep
├── config/             # конфигурация
│   ├── app.php
│   ├── container.php
│   ├── database.php
│   ├── middleware.php
│   └── routes.php
├── public/             # корень веб-сервера
│   ├── .htaccess
│   └── index.php
├── resources/          # шаблоны страниц
│   └── views/
│       ├── errors/
│       │   ├── 404.php
│       │   └── 500.php
│       ├── home.php
│       └── layout.php
├── src/                # ядро фреймворка, пространство имён `MinMvc\`
│   ├── Config/
│   │   ├── Config.php
│   │   └── Env.php
│   ├── Container/
│   │   ├── Container.php
│   │   ├── ContainerException.php
│   │   └── NotFoundException.php
│   ├── Database/
│   │   └── Connection.php
│   ├── Http/
│   │   ├── HttpException.php
│   │   └── ResponseEmitter.php
│   ├── Log/
│   │   └── FileLogger.php
│   ├── Middleware/
│   │   ├── ErrorHandler.php
│   │   └── Pipeline.php
│   ├── Routing/
│   │   ├── Dispatcher.php
│   │   ├── Route.php
│   │   └── Router.php
│   ├── View/
│   │   └── View.php
│   └── Kernel.php
├── storage/            # файлы времени выполнения
│   └── logs/
│       └── .gitkeep
├── .env.example
├── .gitignore
├── bootstrap.php
├── composer.json
└── readme.md
```

## Файлы

| Файл | Назначение |
| --- | --- |
| `app/Controllers/HomeController.php` | Контроллер главной страницы. |
| `app/Models/.gitkeep` | Каталог моделей предметной области; файлов пока нет. |
| `config/app.php` | Параметры приложения: окружение, режим отладки, пути. |
| `config/container.php` | Привязки контейнера: интерфейсы → реализации, общие экземпляры. |
| `config/database.php` | Параметры подключения к базе данных. |
| `config/middleware.php` | Список middleware в порядке выполнения. |
| `config/routes.php` | Таблица маршрутов: метод, путь, обработчик. |
| `public/.htaccess` | Перенаправление всех запросов, кроме существующих файлов, в `index.php`. |
| `public/index.php` | Единая точка входа (Front Controller): создаёт запрос, передаёт его ядру, отправляет ответ. |
| `resources/views/errors/404.php` | Страница ошибки 404. |
| `resources/views/errors/500.php` | Нейтральная страница ошибки 500 без технических подробностей. |
| `resources/views/home.php` | Шаблон главной страницы. |
| `resources/views/layout.php` | Общий макет страницы. |
| `src/Config/Config.php` | Доступ к значениям конфигурации по ключу со значением по умолчанию. |
| `src/Config/Env.php` | Загрузка переменных окружения из `.env`. |
| `src/Container/Container.php` | DI-контейнер (PSR-11) с автоподстановкой зависимостей по типам параметров конструктора. |
| `src/Container/ContainerException.php` | Исключение контейнера (PSR-11): общая ошибка создания объекта. |
| `src/Container/NotFoundException.php` | Исключение контейнера (PSR-11): идентификатор не найден. |
| `src/Database/Connection.php` | Подключение к базе данных поверх PDO: подготовленные запросы и транзакции. |
| `src/Http/HttpException.php` | Исключение с HTTP-статусом (404, 405 и другие). |
| `src/Http/ResponseEmitter.php` | Отправка ответа PSR-7: статус, заголовки, тело. |
| `src/Log/FileLogger.php` | Журнал (PSR-3) с записью в файл. |
| `src/Middleware/ErrorHandler.php` | Самый внешний middleware: превращает исключения в HTTP-ответы и пишет подробности в журнал. |
| `src/Middleware/Pipeline.php` | Конвейер middleware (PSR-15): выстраивает цепочку и вызывает конечный обработчик. |
| `src/Routing/Dispatcher.php` | Диспетчер: создаёт контроллер через контейнер и вызывает действие с аргументами. |
| `src/Routing/Route.php` | Описание маршрута: метод, шаблон пути, обработчик. |
| `src/Routing/Router.php` | Маршрутизатор: сопоставляет метод и путь с маршрутом, извлекает параметры пути, сообщает 404 и 405. |
| `src/View/View.php` | Шаблонизатор: отрисовка шаблона в макете, экранирование вывода по умолчанию. |
| `src/Kernel.php` | Ядро: обработчик запроса (PSR-15); собирает конвейер middleware, маршрутизатор и диспетчер. |
| `storage/logs/.gitkeep` | Каталог журналов; содержимое не хранится в репозитории. |
| `.env.example` | Перечень переменных окружения с примерами значений. |
| `.gitignore` | Исключения из репозитория: `vendor/`, `.env`, файлы журналов. |
| `bootstrap.php` | Сборка приложения: загрузка `.env` и конфигурации, создание контейнера и ядра. |
| `composer.json` | Зависимости (PHP ^8.3, интерфейсы PSR-3, PSR-7, PSR-11, PSR-15, PSR-17 и реализация PSR-7 `nyholm/psr7`) и автозагрузка PSR-4: `MinMvc\` → `src/`, `App\` → `app/`. |
| `readme.md` | Описание дерева каталогов и файлов. |
