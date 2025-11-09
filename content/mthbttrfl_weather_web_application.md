## AppConfig
- Имя метода apiKey() не отражает смысл — лучше возвращать DTO или конфигурационный объект, а не строку-ключ как бин. Лучше создать WeatherApiProperties с @ConfigurationProperties(prefix = "weather.api")
- @Value("${key}") без префикса (weather.api.key) — может конфликтовать при масштабировании конфигураций, если появятся какие-то другие ключи
## DataSourceConfig
- Лучше использовать @ConfigurationProperties(prefix = "database")
## HibernateConfig
- sessionFactory.setPackagesToScan("org.example.webweatherapplication.models"). Лучше вынести путь в конфигурацию или константу
## SpringWebConfig
- @ComponentScan("org.example") — слишком широкий. Лучше ограничить пакетами controllers, services, repositories
- SignUpControllerAdvice создаётся вручную — стоит аннотировать его @ControllerAdvice и убрать из конфигурации
## SpringWebInitializer
- Я бы добавил CharacterEncodingFilter. Важно для UTF-8 кодировки
## ErrorController
- Логика redirect:/ при null выглядит костыльно — лучше использовать глобальный @ControllerAdvice с @ExceptionHandler
## HomeController
- postRemoveLocation - избыточное название. Лучше removeLocation
- Для удаления локации больше подходит DELETE метод http
- Нет проверки на наличие пользователя (если user не найден)
## SearchController
- В getSearchPageWithLocations стоит проверять city на пустую строку и на null
- Неудачные названия методов
## SignUpController
- метод контроллера называется postNewUser, метод сервиса - registration. Оба названия неудачные. Необходимо ознакомиться с конвенциями об именованиях в java. Я бы назвал оба метода register
## SingInController
- Название с ошибкой. Наверное имелось в виду SignInController
- postNewSessionForValidUser некорректное название
## GlobalControllerAdvice
- Проверка ex.getClass().getSimpleName().equals("WeatherApiException") не нравится. Лучше использовать instanceof WeatherApiException или проверку на подклассы
- exceptionHandler в названии методов - избыточно. И так понятно, что это exceptionHandler. Пример: handleWeatherApiException
- Нет логирования
## SignInControllerAdvice
- Повторение шаблонного кода (создание new SignInUserDTO(), добавление "error") можно вынести в приватный метод
- Нет логирования
## SignUpControllerAdvice
- Нет логирования
## MainDTO
- поле humidity — String. Лучше будет int или BigDecimal
- components в spring разработке ассоциируется с бинами. Лучше не использовать это название для пакета
## WeatherApiException и подклассы
- Подклассы не содержат уникального поведения (кроме имени) — можно использовать фабрику (WeatherApiExceptionFactory) вместо кучи отдельных классов
- Лучше добавить поля statusCode и errorCode для точной диагностики
- BadRequestWeatherApiException → WeatherBadRequestException — читается проще
## RedirectIncorrectUrlFilter
- Метод doFilter можно упростить через ранние возвраты (guard clauses) и минимизацию вложенности
## SessionFilter
- Можно использовать UUID.fromString(sessionId) с обработкой исключения вместо regex — это надёжнее и быстрее
- Метод extractSessionId можно сократить с использованием stream api
## Location
- в @ManyToOne стоит указать fetch type LAZY, чтобы избежать ненужной загрузки
- Убедиться в необходимости equals/hashCode
## Session
- в @ManyToOne стоит указать fetch type LAZY, чтобы избежать ненужной загрузки
- Название поля time неочевидно — лучше createdAt или lastAccessTime
- Убедиться в необходимости equals/hashCode
## User
- Убедиться в необходимости equals/hashCode
## LocationsRepository
- Метод readByUser может возвращать пустой список, но не проверяет null у user
- Метод remove логичен, но можно добавить проверку количества удалённых строк — чтобы понимать, был ли удалён объект
## SessionsRepository
- getSingleResult() выбрасывает исключение, если результат не найден — стоит заменить на uniqueResultOptional()
- Метод removeByTime можно дополнить логом количества удалённых строк
## UsersRepository
- findByName возвращает исключение при отсутствии пользователя — также лучше использовать uniqueResultOptional()
- Перед сохранением пользователя нужно явно хэшировать пароль
- Название метода findByName(User user) сбивает с толку: параметр — User, но ищем по username. Лучше findByUsername(String username)
## changelogs
- Тип varchar без длины — лучше явно указать, например varchar(255)
- decimal для координат лучше уточнить с масштабом: decimal(10,7)
## LocationService
- try/catch с ex.printStackTrace() в create() — не лучший подход. Лучше логировать и возвращать понятное исключение бизнес-уровня
- В read() логика сбора DTO через ручной цикл может быть преобразована в stream api
- read - неудачное название. Лучше getUserLocations()
- в read будет открыта транзакция к БД на протяжении всей работы метода. Выполнять внешний вызов api из транзакции крайне не желательно. Это критическая ошибка, которую нужно убирать.
## SchedulerCleaningSessionService
- try/catch с ex.printStackTrace() — заменить на логирование
## SessionService
- в remove() нет обработки возможных ошибок
## UserService
- Название findValid() неочевидно, лучше authenticateUser() или validateCredentials()
- Нет проверки на существующего пользователя при регистрации (перед repository.create())
## WeatherApiService
- apiKey лучше получать через @Value("${key}"), чтобы явно показать источник
- @Transactional(propagation = Propagation.REQUIRES_NEW) не нужна. HTTP-запрос не требует участия в транзакции БД
## JsonHttpStatusDecoderUtil
- Лучше логировать статус и тело при ошибке для диагностики
- Можно сделать validStatusCodes неизменяемым Set.of() для O(1) проверки
## LocationRoundingTemperature
- Лучше назвать InfoLocationTemperatureRounder
- Хорошо бы добавить null-проверку на parameter.getMain()
## BCryptUtil
- Метод equals() лучше назвать matches(), чтобы не путать с Object.equals()
## UuidUtil
- Не вижу смысла выносить это в отдельный класс. По сути добавился класс, где ты делаешь стандартные вызовы методов. Получается лишний слой абстракции, который на самом деле не добавляет никакой логики
## README.md
- Нет используемого стека технологий
- Нет информации о структуре проекта и его функциональности
## Итог:
### Минусы
- Отсутствуют тесты. Среди разработчиков есть фраза "Если нет тестов, то код можно считать нерабочим"
- При запуске приложения локально при негативных сценариях появляются ошибки, которые сейчас не обрабатываются. Повод задуматься о валидации и краевых случаях.
- Стоит изучить какие файлы нужно добавлять в .gitignore
- Есть регулярные проблемы с именованием методов, пакетов и переменных
- Игнорирование подсказок редактора кода
- Проблемы с форматированием кода. Современные редакторы кода способы одной горячей клавишей форматировать код по стандартам java code style
### Плюсы
- Выдержана слоистая архитектура
- Сервисы декомпозированы по функциональности
- Подключение внешнего api реализовано грамотно
- Управление БД средствами миграций

## Вывод
- После устранения недостатков и добавления тестов согласно ТЗ можно будет считать проект выполненным. На текущем этапе - проект не удовлетворяет требованиям, чтобы пойти дальше
