# Минимальное ядро MVC-фреймворка на PHP

Nov 12, 2026 · @Tom

Минимальный MVC-фреймворк — это десять функций: единая точка входа, объекты запроса и ответа, маршрутизатор, диспетчер контроллеров, конвейер middleware, DI-контейнер, конфигурация, шаблонизация с экранированием, слой доступа к данным и централизованная обработка ошибок. Сессии, CSRF-защита, валидация, ORM, консоль и кэш подключаются поверх — через middleware и интерфейсы PSR. В ноябре 2026 года такое ядро разумно строить на PHP 8.3+ и стандартах PHP-FIG, а не на собственных контрактах.

## Паттерны проектирования в основе ядра

Ядро MVC-фреймворка складывается из семи ролей, и у каждой есть классический паттерн; сам MVC — лишь общая рамка. Паттерны из каталога Фаулера ведут на его описания; Chain of Responsibility описан по [Refactoring.Guru](https://refactoring.guru/design-patterns/chain-of-responsibility), остальные паттерны «банды четырёх» (GoF) — по книге Design Patterns 1994 года.

| Паттерн | Что решает | Где в ядре |
| --- | --- | --- |
| [Model View Controller](https://martinfowler.com/eaaCatalog/modelViewController.html) | Делит работу с интерфейсом на три роли: модель, представление, контроллер | Общая структура приложения |
| [Front Controller](https://martinfowler.com/eaaCatalog/frontController.html) | Один обработчик принимает все запросы сайта; общие задачи вроде безопасности и локализации решаются в одном месте | `public/index.php` и класс ядра |
| [Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility) (GoF) | Запрос идёт по цепочке обработчиков; каждый отвечает сам или передаёт дальше | Конвейер middleware |
| [Page Controller](https://martinfowler.com/eaaCatalog/pageController.html) | Один объект отвечает за одну страницу или действие | Классы контроллеров |
| [Dependency Injection](https://martinfowler.com/articles/injection.html) | Зависимости приходят снаружи, сборка объектов отделена от их использования | DI-контейнер |
| [Template View](https://martinfowler.com/eaaCatalog/) и [Two Step View](https://martinfowler.com/eaaCatalog/) | Шаблон с маркерами для данных; сначала логическая страница, затем общий макет | Шаблонизатор и макеты |
| [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html), [Active Record](https://martinfowler.com/eaaCatalog/), [Table Data Gateway](https://martinfowler.com/eaaCatalog/), [Row Data Gateway](https://martinfowler.com/eaaCatalog/rowDataGateway.html) | Четыре способа связать объекты с таблицами базы | Слой доступа к данным |
| [Service Layer](https://martinfowler.com/eaaCatalog/) | Граница приложения: набор операций, которые координируют ответ | Сервисы между контроллером и моделью |

Два вывода из первоисточников. Для переиспользуемого кода Фаулер предпочитает внедрение зависимостей, для закрытого приложения считает допустимым и Service Locator; внедрение через конструктор он ставит на первое место, а главным называет разделение сборки сервисов и их использования. Фреймворк — как раз переиспользуемый компонент, поэтому ядро строится на DI. MVC родился в Smalltalk в конце 1970-х, и в вебе его трактуют вольно: «веб-MVC» на практике — это Front Controller, контроллеры-действия и Template View.

Из GoF в ядре встречаются ещё Strategy и Adapter (взаимозаменяемые логгер, кэш и шаблонизатор за интерфейсом) и Factory (фабрики PSR-17). С Decorator middleware легко спутать: у обоих рекурсивная структура, но middleware может остановить цепочку и вернуть ответ сам, а декоратор поток не прерывает. Singleton и Registry в коде приложения нежелательны: общие экземпляры хранит контейнер.

## Современные практики PHP, которые определяют ядро

В ноябре 2026 года новое ядро разумно строить на PHP 8.3+ и стандартах PHP-FIG, а не на собственных контрактах.

### Версии PHP

Данные [php.net](https://www.php.net/supported-versions.php) на 12 ноября 2026 года:

- PHP 8.5 вышел 20 ноября 2025 года, активная поддержка — до 31 декабря 2027 года.
- PHP 8.4 — активная поддержка до 31 декабря 2026 года.
- PHP 8.3 и 8.2 получают только исправления безопасности: до 31 декабря 2027 и до 31 декабря 2026 года.
- PHP 8.6 на стадии RC5 (5 ноября 2026 года); финальный релиз [запланирован](https://www.php.net/archive/2026.php) на 19 ноября 2026 года.

[Laravel 13](https://laravel.com/docs/13.x/releases) (17 марта 2026 года) требует PHP 8.3 или новее; такой же нижний порог даёт новому ядру широкую аудиторию. [Symfony](https://symfony.com/releases) поддерживает, в частности, LTS-ветку 7.4 и текущую ветку 8.1.

### Возможности языка

Для ядра полезнее всего строгая типизация (`declare(strict_types=1)`), неизменяемые `readonly`-объекты для запроса и ответа, `enum` для HTTP-методов и статусов и атрибуты: маршрут или middleware объявляются прямо над методом контроллера. Laravel 13 как раз расширил набор таких атрибутов (`#[Middleware]`, `#[Authorize]` и другие). Новинки [PHP 8.5](https://www.php.net/releases/8.5/en.php) — оператор `|>`, `clone` с заменой свойств и атрибут `#[\NoDiscard]` — удобны, но ядро с порогом 8.3 на них опираться не может.

### Стандарты PHP-FIG

Там, где есть PSR, ядро не придумывает собственный интерфейс: так оно совместимо с чужими middleware, контейнерами и логгерами.

| Стандарт | Что стандартизирует | Роль в ядре |
| --- | --- | --- |
| [PSR-4](https://www.php-fig.org/psr/) | Автозагрузка классов по пространствам имён | Обязателен, настраивается в `composer.json` |
| [PSR-7](https://www.php-fig.org/psr/) и [PSR-17](https://www.php-fig.org/psr/) | Неизменяемые объекты запроса и ответа, фабрики для них | Основа HTTP-слоя |
| [PSR-15](https://www.php-fig.org/psr/psr-15/) | `RequestHandlerInterface::handle()` и `MiddlewareInterface::process()` | Основа конвейера |
| [PSR-11](https://www.php-fig.org/psr/psr-11/) | `ContainerInterface` с методами `get()` и `has()` | Контракт DI-контейнера |
| [PSR-3](https://www.php-fig.org/psr/) | Интерфейс журнала | Контракт логирования |
| [PSR-6](https://www.php-fig.org/psr/), [PSR-16](https://www.php-fig.org/psr/), [PSR-14](https://www.php-fig.org/psr/), [PSR-18](https://www.php-fig.org/psr/), [PSR-20](https://www.php-fig.org/psr/) | Кэш, события, HTTP-клиент, часы | Подключаются по мере появления функций |

Все перечисленные стандарты имеют статус Accepted. PSR-0 и PSR-2 признаны устаревшими (Deprecated), а PSR-5, PSR-19, PSR-21 и PSR-22 остаются черновиками, и опираться на них нельзя. Для стиля кода действует [PER Coding Style 3.1](https://www.php-fig.org/per/coding-style/): он расширяет и заменяет PSR-12 (в общем списке PSR-12 пока значится принятым) и не требует `declare(strict_types=1)`, поэтому ядру нужно закрепить это правило линтером.

### Архитектура приложения поверх ядра

- Тонкие контроллеры и Service Layer: контроллер разбирает вход и возвращает ответ, а логика живёт в сервисах и доменных классах.
- Явные зависимости и отсутствие глобального состояния. [FrankenPHP в worker-режиме](https://frankenphp.dev/docs/worker/) загружает приложение один раз и держит его в памяти: между запросами сбрасывается только состояние запроса, а статические свойства, глобальные переменные и кэши в памяти сохраняются. Код без глобального состояния одинаково работает в классической модели «запрос — процесс» и в долгоживущих процессах (FrankenPHP, RoadRunner, Swoole).
- Домен не зависит от фреймворка (гексагональная архитектура, «порты и адаптеры»): фреймворк только доставляет запрос, поэтому ядро не требует наследовать доменные классы от своего базового класса.
- [Action–Domain–Responder](https://github.com/pmjones/adr) — веб-ориентированная альтернатива MVC: Action передаёт данные запроса в Domain (Transaction Script, Service Layer и т. п.), а Responder строит HTTP-ответ — статус, заголовки, шаблон. Ядро на PSR-15 допускает оба стиля.
- Статический анализ (PHPStan, Psalm) и автотесты входят в проект самого ядра: типы и неизменяемые объекты делают его поведение проверяемым.

## Минимальное ядро: десять обязательных функций

Функция входит в минимум, если без неё нельзя принять запрос и безопасно вернуть ответ либо её нельзя добавить снаружи без правки ядра. Таких функций десять.

### 1. Автозагрузка классов и единая точка входа

Все запросы, кроме статических файлов, попадают в один файл `public/index.php` — это Front Controller. Он подключает автозагрузчик Composer (PSR-4), собирает контейнер и передаёт запрос ядру. Достаточно, если вне каталога `public` нет файлов, доступных по URL, а в коде приложения нет `require` для классов.

### 2. Объекты запроса и ответа

Код приложения не читает `$_GET`, `$_POST` и `$_SERVER` и не вызывает `header()` и `echo`: он получает объект запроса и возвращает объект ответа. Минимум — метод, путь, заголовки, тело и разобранные параметры запроса; статус, заголовки и тело ответа. Лучший выбор — неизменяемые объекты PSR-7 с фабриками PSR-17: ядро совместимо с чужими middleware, а тесты создают запрос в коде, без веб-сервера.

### 3. Маршрутизатор

Сопоставляет метод HTTP и путь с обработчиком и извлекает параметры из пути (`/posts/{id}`). Минимум: статические и параметризованные маршруты, ответ 404 для неизвестного пути и 405 для неподходящего метода. На грани минимума — генерация URL по имени маршрута: без неё адреса дублируются в шаблонах. Маршруты объявляют атрибутами над методом контроллера или в одном файле; кэш таблицы маршрутов — оптимизация, а не часть минимума.

### 4. Диспетчер контроллеров

Превращает найденный маршрут в вызов. Он создаёт контроллер через контейнер, подставляет аргументы метода — запрос и параметры из пути — и принимает объект ответа. Контроллер остаётся тонким (Page Controller): разобрать вход, вызвать сервис, вернуть ответ. Минимум — поддержка пары «класс и метод» и вызываемого объекта; метод обязан вернуть ответ, неявные преобразования строк и массивов не нужны.

### 5. Конвейер middleware

Цепочка обработчиков: запрос идёт по ней к контроллеру, ответ — обратно. Это Chain of Responsibility: каждое звено вызывает следующее или возвращает ответ само, например при отказе в доступе. [PSR-15](https://www.php-fig.org/psr/psr-15/) описывает два интерфейса: `MiddlewareInterface::process()` принимает запрос и следующий обработчик, `RequestHandlerInterface::handle()` принимает запрос. Спецификация прямо разрешает вернуть ответ, не вызывая следующий обработчик. Ядру достаточно собрать цепочку; сессии, CSRF и аутентификация приходят как middleware и в минимум не входят.

### 6. Контейнер внедрения зависимостей

Создаёт объекты вместе с их зависимостями и хранит общие экземпляры. По [PSR-11](https://www.php-fig.org/psr/psr-11/) нужны методы `get()` и `has()` и исключение `NotFoundExceptionInterface` для неизвестного идентификатора. Сверх стандарта пригодятся автоподстановка по типам параметров конструктора, привязка интерфейса к реализации в конфигурации и признак общего экземпляра. Сам PSR-11 не советует передавать контейнер внутрь объектов: так он превращается в Service Locator, и зависимости исчезают из сигнатур. Контейнер — единственное место, где собирается приложение.

### 7. Конфигурация и окружения

Параметры (подключение к БД, режим отладки, ключи) лежат вне кода: файлы конфигурации плюс переменные окружения. Различие между dev, test и prod задаётся окружением, а не правкой кода. Минимум — единый доступ к значениям с дефолтами, загрузка из `.env` или настоящих переменных и секреты вне репозитория. Конфигурация читается один раз при старте и не меняется во время запроса.

### 8. Шаблонизация (View)

Превращает данные в HTML. Минимум — Template View: шаблон получает переменные, поддерживает общий макет и вложенные фрагменты. Главное требование безопасности — экранирование вывода по умолчанию, а «сырой» вывод включается явно. [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) советует то же: кодировать данные в соответствии с контекстом вывода, начинать с автоэкранирования фреймворка и знать, где оно заканчивается. Подойдут и обычные PHP-шаблоны с функцией экранирования; Twig или Blade подключаются адаптером.

### 9. Слой доступа к данным

«Модель» в MVC — это предметная логика, а не базовый класс. Задача фреймворка — дать ей доступ к хранилищу: управление подключением, подготовленные запросы с привязкой параметров, транзакции и единообразные ошибки — тонкая обёртка над PDO. [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) называет параметризованные запросы главной защитой от SQL-инъекций; для названий таблиц и столбцов и направления сортировки нужен строгий список допустимых значений, а ручное экранирование не рекомендуется. ORM (Active Record или Data Mapper) в минимум не входит: репозиторий пишется в приложении поверх этой обёртки.

### 10. Обработка ошибок и журналирование

Центральный обработчик превращает любое исключение в HTTP-ответ: 404 и 405 от маршрутизатора, 4xx от самого приложения (исключение с кодом статуса), 500 для остального. В режиме отладки показывается трассировка, в продакшене — нейтральная страница, а подробности пишутся в журнал через интерфейс PSR-3 (минимум — файл или stderr). Обработчик — самое внешнее звено конвейера, поэтому ошибка в любом другом звене не превращается в пустую страницу.

## Жизненный цикл запроса

Запрос проходит десять шагов; любое исключение на любом из них превращается в ответ самым внешним звеном конвейера.

1. Веб-сервер направляет все запросы, кроме статических файлов, в `public/index.php`.
2. Точка входа подключает автозагрузчик Composer, читает конфигурацию и собирает контейнер.
3. Из суперглобальных переменных строится объект запроса.
4. Ядро запускает конвейер middleware: первым работает обработчик ошибок, затем сессия, CSRF-защита и другие звенья.
5. Маршрутизатор сопоставляет метод и путь с маршрутом; при неудаче бросает исключение со статусом 404 или 405.
6. Контейнер создаёт контроллер вместе с зависимостями, диспетчер передаёт методу запрос и параметры из пути.
7. Контроллер вызывает сервисы и слой данных и получает результат.
8. Контроллер отдаёт данные шаблонизатору (или собирает JSON) и возвращает объект ответа.
9. Ответ проходит через middleware в обратном порядке: добавляются заголовки и cookie.
10. Ядро отправляет статус, заголовки и тело; в обычном режиме процесс завершается, в worker-режиме ждёт следующий запрос.

## За рамками минимума

Всё перечисленное ниже добавляется поверх ядра как middleware, консольные команды или пакеты за интерфейсами, и ядро для этого не правится. Первыми в очереди стоят сессии, защита от CSRF и валидация: без них нельзя безопасно принимать формы.

| Компонент | Когда добавлять | Как подключается |
| --- | --- | --- |
| Сессии и flash-сообщения | Появились вход пользователей или многошаговые формы | Middleware и хранилище (файлы, БД, Redis) |
| Защита от CSRF | Браузер отправляет изменяющие запросы с cookie-аутентификацией | Middleware и токен в форме |
| Валидация ввода | Появилась первая форма или JSON-эндпоинт | Отдельный компонент: правила на входе, ошибки на выходе |
| Аутентификация и авторизация | Есть закрытые разделы | Middleware и политики доступа |
| Консоль (CLI) | Нужны миграции и фоновые задачи | Вторая точка входа `bin/console` с тем же контейнером |
| Миграции схемы БД | Схему меняют несколько человек или чаще раза в релиз | Консольные команды |
| ORM | Много однотипных запросов и сложные связи | Пакет поверх слоя данных (например, Doctrine или Eloquent) |
| Кэш | Профилирование показало узкие места | Адаптер за интерфейсом PSR-6 или PSR-16 |
| События | Нужно развязать модули | Диспетчер событий по PSR-14 |
| Очереди, почта, файлы, локализация | По требованию продукта | Отдельные пакеты |
| Кэш маршрутов и контейнера | Выросло время старта | Оптимизация после профилирования |

[OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) перечисляет для защиты от CSRF синхронизирующий токен, подписанный double-submit cookie, проверку Origin и пользовательские заголовки для API; SameSite лучше работает как дополнительная мера. Менять состояние запросом GET нельзя. В Laravel 13 за это отвечает middleware `PreventRequestForgery`.

## Выводы и рекомендации

Ядро стоит держать в границах десяти функций и принять при проектировании шесть решений:

- Строить ядро как композицию PSR-4, PSR-7 и PSR-17, PSR-15, PSR-11 и PSR-3, а не как набор собственных контрактов.
- Решать, входит ли функция в ядро, одним вопросом: можно ли добавить её снаружи без правки ядра? Если можно, это middleware или пакет.
- Не навязывать модель: не требовать наследования доменных классов от базового `Model` и давать только доступ к данным.
- Обеспечить безопасность по умолчанию: экранирование в шаблонах, подготовленные запросы, нейтральные страницы ошибок в продакшене; сессии, CSRF-защиту и валидацию добавить первыми.
- Проектировать ядро как функцию «запрос → ответ» без глобального состояния: так оно тестируется без веб-сервера и работает в долгоживущих процессах.
- Взять PHP 8.3 нижним порогом, PER Coding Style стилем кода, а статический анализ включить в проверки с первого дня.

## Приложение: рабочий пример ядра

В примере (PHP 8.3, без зависимостей, 115 строк) собраны точка входа, объекты запроса и ответа, маршрутизатор, контейнер с автоподстановкой, конвейер middleware, шаблонизатор с экранированием и обработчик ошибок. Конфигурация, слой данных и автозагрузка Composer опущены ради краткости, а маршрутизатор отвечает 404 и на неподходящий метод. Файл проверен на PHP 8.3.6; запуск: `php -S localhost:8000 index.php`.

```php
<?php
declare(strict_types=1);

final class HttpException extends RuntimeException
{
    public function __construct(public readonly int $status)
    {
        parent::__construct("HTTP $status", $status);
    }
}

final class Request { public function __construct(public readonly string $method, public readonly string $path) {} }
final class Response { public function __construct(public readonly string $body, public readonly int $status = 200) {} }

// Template View: маркеры {{имя}}, значения экранируются по умолчанию
final class View
{
    public function render(string $tpl, array $vars): string
    {
        $map = [];
        foreach ($vars as $name => $value) {
            $map['{{' . $name . '}}'] = htmlspecialchars((string) $value, ENT_QUOTES | ENT_SUBSTITUTE);
        }
        return strtr($tpl, $map);
    }
}

// DI-контейнер: создаёт объекты по типам параметров конструктора
final class Container
{
    private array $shared = [];

    public function get(string $id): object
    {
        return $this->shared[$id] ??= $this->build($id);
    }

    private function build(string $id): object
    {
        $params = (new ReflectionClass($id))->getConstructor()?->getParameters() ?? [];
        return new $id(...array_map(fn ($p) => $this->get($p->getType()->getName()), $params));
    }
}

final class Router
{
    private array $routes = [];

    public function add(string $method, string $pattern, array $handler): void
    {
        $regex = '#^' . preg_replace('#\{(\w+)\}#', '(?P<$1>[^/]+)', $pattern) . '$#';
        $this->routes[] = [$method, $regex, $handler];
    }

    public function match(Request $r): array
    {
        foreach ($this->routes as [$method, $regex, $handler]) {
            if ($method === $r->method && preg_match($regex, $r->path, $m)) {
                return [$handler, array_filter($m, 'is_string', ARRAY_FILTER_USE_KEY)];
            }
        }
        throw new HttpException(404);
    }
}

// Ядро: конвейер middleware, в конце которого — маршрутизатор и контроллер
final class Kernel
{
    public function __construct(private Container $c, private Router $router, private array $middleware) {}

    public function handle(Request $request): Response
    {
        $dispatch = function (Request $r): Response {
            [[$class, $action], $params] = $this->router->match($r);
            return $this->c->get($class)->$action($r, ...$params);
        };
        $pipeline = array_reduce(
            array_reverse($this->middleware),
            fn (callable $next, callable $mw) => fn (Request $r) => $mw($r, $next),
            $dispatch,
        );
        return $pipeline($request);
    }
}

// ---- приложение ----
final class PostController
{
    public function __construct(private View $view) {}

    public function show(Request $r, string $id): Response
    {
        return new Response($this->view->render('<h1>Статья №{{id}}</h1>', ['id' => $id]));
    }
}

$errors = function (Request $r, callable $next): Response { // самый внешний middleware
    try {
        return $next($r);
    } catch (HttpException $e) {
        return new Response("Ошибка {$e->status}", $e->status);
    } catch (Throwable $e) {
        error_log((string) $e);
        return new Response('Внутренняя ошибка', 500);
    }
};

$router = new Router();
$router->add('GET', '/posts/{id}', [PostController::class, 'show']);

$kernel = new Kernel(new Container(), $router, [$errors]);
$response = $kernel->handle(new Request($_SERVER['REQUEST_METHOD'], parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH)));

http_response_code($response->status);
echo $response->body;
```

## Источники

Страницы, открытые при подготовке доклада (данные на 12 ноября 2026 года):

- [PHP Standards Recommendations — PHP-FIG](https://www.php-fig.org/psr/): список PSR и их статусы; [PSR-15](https://www.php-fig.org/psr/psr-15/) и [PSR-11](https://www.php-fig.org/psr/psr-11/); [PER Coding Style 3.1](https://www.php-fig.org/per/coding-style/)
- [PHP: Supported Versions](https://www.php.net/supported-versions.php), [новости php.net за 2026 год](https://www.php.net/archive/2026.php), [PHP 8.5 Release Announcement](https://www.php.net/releases/8.5/en.php)
- [Laravel 13: Release Notes](https://laravel.com/docs/13.x/releases) и [Symfony: releases](https://symfony.com/releases)
- Мартин Фаулер: [каталог Patterns of Enterprise Application Architecture](https://martinfowler.com/eaaCatalog/), [Front Controller](https://martinfowler.com/eaaCatalog/frontController.html), [Model View Controller](https://martinfowler.com/eaaCatalog/modelViewController.html), [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
- [Refactoring.Guru: Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility) и [pmjones/adr: Action Domain Responder](https://github.com/pmjones/adr)
- [FrankenPHP: worker mode](https://frankenphp.dev/docs/worker/)
- OWASP Cheat Sheet Series: [SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html), [Cross Site Scripting Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html), [Cross-Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

Описания остальных паттернов Фаулера взяты из его каталога (отдельные страницы Page Controller, Data Mapper и Row Data Gateway не открывались). Паттерны GoF, кроме Chain of Responsibility, описаны по книге Design Patterns (Gamma, Helm, Johnson, Vlissides, 1994) и общим знаниям. Упоминания RoadRunner, Swoole, PHPStan, Psalm, Doctrine, Eloquent, Twig, Blade и гексагональной архитектуры даны по общим знаниям.
