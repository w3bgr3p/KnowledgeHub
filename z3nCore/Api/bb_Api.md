[SPOILER= Api] 

[SPOILER= AiApi]

# Класс AI

Описание
Класс AI предназначен для интеграции с различными сервисами искусственного интеллекта в проектах ZennoPoster. Он позволяет отправлять запросы к AI-сервисам (Perplexity, AI.IO) и получать текстовые ответы. Класс поддерживает различные предустановленные функции для генерации твитов, оптимизации кода и создания обращений в службу поддержки Google.

## Конструкторы

**AI(IZennoPosterProjectModel project, string provider, string model = null, bool log = false)**

Описание: Создает экземпляр AI-клиента с указанным провайдером и настройками.
Параметры:
- project - модель проекта ZennoPoster
- provider - провайдер AI-сервиса ("perplexity" или "aiio")
- model - конкретная модель для использования (необязательно)
- log - включить подробное логирование (по умолчанию false)

Пример:
[CODE=csharp]
var ai = new AI(project, "perplexity", "llama-3.3-70b", true);
project.SendInfoToLog("AI клиент создан с провайдером Perplexity", false);
[/CODE]

## Публичные методы

**Query(string systemContent, string userContent, string aiModel = "rnd", bool log = false)**

Возвращает: string - ответ от AI-сервиса
Описание: Отправляет запрос к AI-сервису с системным промптом и пользовательским контентом. Если модель не указана, выбирается случайная из доступных.
Параметры:
- systemContent - системный промпт для настройки поведения AI
- userContent - пользовательский запрос или контент для обработки
- aiModel - модель AI для использования (по умолчанию "rnd" - случайная)
- log - включить логирование запроса

Пример:
[CODE=csharp]
string systemPrompt = "Ты помощник по программированию";
string userQuery = "Объясни что такое циклы в C#";
string response = ai.Query(systemPrompt, userQuery, "deepseek-ai/DeepSeek-R1-0528");
project.SendInfoToLog($"Ответ AI: {response}", false);
[/CODE]

**GenerateTweet(string content, string bio = "", bool log = false)**

Возвращает: string - сгенерированный текст твита (до 220 символов)
Описание: Генерирует твит на основе переданного контента и биографии аккаунта. Автоматически проверяет длину и перегенерирует при превышении лимита.
Параметры:
- content - контент или тема для твита
- bio - биография аккаунта для персонализации стиля (необязательно)
- log - включить логирование

Пример:
[CODE=csharp]
string tweetContent = "Новости из мира криптовалют";
string accountBio = "Криптоэнтузиаст и блокчейн-разработчик";
string tweet = ai.GenerateTweet(tweetContent, accountBio);
project.SendInfoToLog($"Сгенерирован твит: {tweet}", false);
[/CODE]

**OptimizeCode(string content, bool log = false)**

Возвращает: string - оптимизированный код
Описание: Оптимизирует переданный код с точки зрения веб3-разработки. Возвращает только чистый код без объяснений и комментариев.
Параметры:
- content - исходный код для оптимизации
- log - включить логирование

Пример:
[CODE=csharp]
string originalCode = "function transfer(address, amount) { /* неоптимальный код */ }";
string optimizedCode = ai.OptimizeCode(originalCode, true);
project.SendInfoToLog("Код оптимизирован", false);
[/CODE]

**GoogleAppeal(bool log = false)**

Возвращает: string - текст обращения в службу поддержки Google (до 200 символов)
Описание: Генерирует обращение в службу поддержки Google для разблокировки аккаунта. Имитирует стиль обычного пользователя с небольшими грамматическими ошибками.
Параметры:
- log - включить логирование

Пример:
[CODE=csharp]
string appealMessage = ai.GoogleAppeal();
project.SendInfoToLog($"Обращение в Google: {appealMessage}", false);
//отправить сообщение в форму поддержки
[/CODE][/SPOILER]
[SPOILER= BinanceApi]

# BinanceApi

Класс для работы с API биржи Binance в ZennoPoster. Позволяет выводить средства, получать информацию о балансах и истории выводов. Автоматически обрабатывает авторизацию и подпись запросов.

## Конструкторы

### BinanceApi(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса для работы с Binance API.

**Параметры:**
- `project` - модель проекта ZennoPoster
- `log` - включить ли логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
var binanceApi = new BinanceApi(project, log: true);
//создать экземпляр с включенным логированием
[/CODE]

## Методы

### string Withdraw(string coin, string network, string address, string amount)

**Возвращает:** Ответ сервера в виде JSON строки

Выполняет вывод криптовалюты на указанный адрес через выбранную сеть.

**Параметры:**
- `coin` - символ монеты (например, "USDT")
- `network` - название сети (например, "ethereum", "bsc", "arbitrum")
- `address` - адрес кошелька для вывода
- `amount` - сумма вывода

**Пример:**
[CODE=csharp]
string result = binanceApi.Withdraw("USDT", "ethereum", "0x742d35Cc6565C42c3928...", "100");
project.SendInfoToLog($"Результат вывода: {result}");
[/CODE]

### Dictionary<string, string> GetUserAsset()

**Возвращает:** Словарь с балансами всех активов (ключ - символ монеты, значение - доступный баланс)

Получает информацию о всех балансах пользователя.

**Пример:**
[CODE=csharp]
var balances = binanceApi.GetUserAsset();
foreach(var balance in balances)
{
    project.SendInfoToLog($"{balance.Key}: {balance.Value}");
    //вывести баланс каждой монеты
}
[/CODE]

### string GetUserAsset(string coin)

**Возвращает:** Доступный баланс указанной монеты в виде строки

Получает баланс конкретной криптовалюты.

**Параметры:**
- `coin` - символ монеты

**Пример:**
[CODE=csharp]
string usdtBalance = binanceApi.GetUserAsset("USDT");
project.SendInfoToLog($"Баланс USDT: {usdtBalance}");
[/CODE]

### List<string> GetWithdrawHistory()

**Возвращает:** Список строк с историей выводов в формате "id:amount:coin:status"

Получает полную историю всех выводов.

**Пример:**
[CODE=csharp]
var history = binanceApi.GetWithdrawHistory();
foreach(string withdrawal in history)
{
    project.SendInfoToLog($"Вывод: {withdrawal}");
    //показать каждую транзакцию
}
[/CODE]

### string GetWithdrawHistory(string searchId = "")

**Возвращает:** Информацию о конкретной транзакции или "NoIdFound: {searchId}"

Поиск конкретной транзакции вывода по ID.

**Параметры:**
- `searchId` - ID транзакции для поиска (необязательный)

**Пример:**
[CODE=csharp]
string transaction = binanceApi.GetWithdrawHistory("12345");
if(!transaction.Contains("NoIdFound"))
{
    project.SendInfoToLog($"Найдена транзакция: {transaction}");
}
[/CODE][/SPOILER]
[SPOILER= DMailApi]

# DMail

Класс для работы с почтовым сервисом DMail. Позволяет авторизоваться в системе, получать список писем, читать сообщения, отмечать их как прочитанные или перемещать в корзину.

## Конструкторы

### DMail(IZennoPosterProjectModel project, string key = null, bool log = false)

Создает новый экземпляр класса DMail.

**Параметры:**
- `project` - проект ZennoPoster
- `key` - приватный ключ кошелька (необязательно, может быть получен из базы данных проекта)
- `log` - включить логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
//создать экземпляр без логирования
var dmail = new DMail(project);

//создать экземпляр с логированием и указанным ключом
var dmail = new DMail(project, "your_private_key", true);
[/CODE]

## Публичные методы

### CheckAuth() : void

Проверяет и устанавливает авторизацию в системе DMail. Автоматически пытается получить данные авторизации из переменных проекта или выполнить повторную авторизацию.

**Возвращаемое значение:** нет

**Описание:** Проверяет наличие токенов авторизации и при необходимости выполняет процедуру входа в систему.

**Пример:**
[CODE=csharp]
var dmail = new DMail(project);
//проверить авторизацию
dmail.CheckAuth();
project.SendInfoToLog("Авторизация проверена");
[/CODE]

### GetAll() : dynamic

Получает список всех писем из папки "Входящие".

**Возвращаемое значение:** динамический объект с массивом писем

**Описание:** Загружает первые 20 писем из папки входящих сообщений.

**Пример:**
[CODE=csharp]
var dmail = new DMail(project);
//получить все письма
dynamic allMails = dmail.GetAll();
project.SendInfoToLog($"Получено писем: {allMails.Count}");
[/CODE]

### ReadMsg(int index = 0, dynamic mail = null, bool markAsRead = true, bool trash = true) : Dictionary<string, string>

Читает конкретное письмо по индексу.

**Возвращаемое значение:** словарь с данными письма (отправитель, дата, тема, содержимое, идентификаторы)

**Описание:** Извлекает содержимое письма и опционально отмечает его как прочитанное или перемещает в корзину.

**Параметры:**
- `index` - индекс письма в списке (по умолчанию 0)
- `mail` - объект с письмами (если не указан, использует последний загруженный список)
- `markAsRead` - отметить как прочитанное (по умолчанию true)
- `trash` - переместить в корзину (по умолчанию true)

**Пример:**
[CODE=csharp]
var dmail = new DMail(project);
dmail.GetAll();
//прочитать первое письмо без удаления
var message = dmail.ReadMsg(0, markAsRead: true, trash: false);
project.SendInfoToLog($"От: {message["sender"]}, Тема: {message["subj"]}");
[/CODE]

### GetUnread(bool parse = false, string key = null) : string

Получает информацию о непрочитанных сообщениях.

**Возвращаемое значение:** JSON-строка с информацией или конкретное значение, если указан ключ

**Описание:** Возвращает статистику непрочитанных писем и использования почтового ящика.

**Параметры:**
- `parse` - парсировать ответ (по умолчанию false)
- `key` - конкретный ключ для извлечения ("mail_unread_count", "message_unread_count", "not_read_count", "used_total_size")

**Пример:**
[CODE=csharp]
var dmail = new DMail(project);
//получить количество непрочитанных писем
string unreadCount = dmail.GetUnread(key: "mail_unread_count");
project.SendInfoToLog($"Непрочитанных писем: {unreadCount}");

//получить полную информацию
string fullInfo = dmail.GetUnread();
project.SendInfoToLog($"Полная статистика: {fullInfo}");
[/CODE]

### Trash(int index = 0, string dm_scid = null, string dm_smid = null) : void

Перемещает письмо в корзину.

**Возвращаемое значение:** нет

**Описание:** Перемещает указанное письмо из папки "Входящие" в корзину.

**Параметры:**
- `index` - индекс письма (по умолчанию 0)
- `dm_scid` - идентификатор беседы (если не указан, получает автоматически)
- `dm_smid` - идентификатор сообщения (если не указан, получает автоматически)

**Пример:**
[CODE=csharp]
var dmail = new DMail(project);
dmail.GetAll();
//переместить первое письмо в корзину
dmail.Trash(0);
project.SendInfoToLog("Письмо перемещено в корзину");
[/CODE]

### MarkAsRead(int index = 0, string dm_scid = null, string dm_smid = null) : void

Отмечает письмо как прочитанное.

**Возвращаемое значение:** нет

**Описание:** Изменяет статус письма на "прочитано" без перемещения в корзину.

**Параметры:**
- `index` - индекс письма (по умолчанию 0)
- `dm_scid` - идентификатор беседы (если не указан, получает автоматически)
- `dm_smid` - идентификатор сообщения (если не указан, получает автоматически)

**Пример:**
[CODE=csharp]
var dmail = new DMail(project);
dmail.GetAll();
//отметить первое письмо как прочитанное
dmail.MarkAsRead(0);
project.SendInfoToLog("Письмо отмечено как прочитанное");
[/CODE][/SPOILER]
[SPOILER= FirstMail]

# FirstMail

Класс для работы с API сервиса временной почты FirstMail. Позволяет получать письма, извлекать из них коды подтверждения и ссылки.

## Конструкторы

### FirstMail(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса FirstMail.

**Параметры:**
- `project` (IZennoPosterProjectModel) - проект ZennoPoster
- `log` (bool) - включить логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
// создать экземпляр без логирования
var firstMail = new FirstMail(project);

// создать экземпляр с логированием
var firstMail = new FirstMail(project, true);
[/CODE]

## Методы

### GetMail(string email)

Получает последнее письмо для указанного email адреса и загружает его в JSON объект проекта.

**Возвращает:** `string` - JSON ответ от API

**Параметры:**
- `email` (string) - email адрес для получения писем

**Пример:**
[CODE=csharp]
// получить последнее письмо
string jsonResponse = firstMail.GetMail("user@example.com");

// данные письма теперь доступны через project.Json
string subject = project.Json.subject; // тема письма
string text = project.Json.text;       // текст письма
[/CODE]

### GetOTP(string email)

Извлекает 6-значный код подтверждения из последнего письма. Ищет код в теме, тексте и HTML содержимом письма.

**Возвращает:** `string` - найденный 6-значный код

**Параметры:**
- `email` (string) - email адрес для поиска кода

**Пример:**
[CODE=csharp]
try
{
    // получить код подтверждения
    string otpCode = firstMail.GetOTP("user@example.com");
    project.SendInfoToLog($"Получен код: {otpCode}");
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка получения кода: {ex.Message}");
}
[/CODE]

### GetLink(string email)

Извлекает первую найденную ссылку (http/https) из текста последнего письма.

**Возвращает:** `string` - найденная ссылка

**Параметры:**
- `email` (string) - email адрес для поиска ссылки

**Пример:**
[CODE=csharp]
try
{
    // получить ссылку из письма
    string link = firstMail.GetLink("user@example.com");
    project.SendInfoToLog($"Найдена ссылка: {link}");
    
    // можно использовать ссылку для перехода
    tab.Navigate(link);
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ссылка не найдена: {ex.Message}");
}
[/CODE]

### Delete(string email, bool seen = false)

Удаляет письма для указанного email адреса.

**Возвращает:** `string` - JSON ответ от API

**Параметры:**
- `email` (string) - email адрес
- `seen` (bool) - удалить только прочитанные письма (по умолчанию false)

**Пример:**
[CODE=csharp]
// удалить все письма
string result = firstMail.Delete("user@example.com");

// удалить только прочитанные письма
string result = firstMail.Delete("user@example.com", true);
[/CODE]

### GetOne(string email)

Получает одно последнее письмо для указанного email адреса.

**Возвращает:** `string` - JSON ответ от API

**Параметры:**
- `email` (string) - email адрес

**Пример:**
[CODE=csharp]
// получить одно письмо
string message = firstMail.GetOne("user@example.com");
project.Json.FromString(message);
[/CODE]

### GetAll(string email)

Получает все письма для указанного email адреса.

**Возвращает:** `string` - JSON ответ от API с массивом писем

**Параметры:**
- `email` (string) - email адрес

**Пример:**
[CODE=csharp]
// получить все письма
string allMessages = firstMail.GetAll("user@example.com");
project.Json.FromString(allMessages);

// обработать каждое письмо в цикле
for (int i = 0; i < project.Json.messages.Count; i++)
{
    string subject = project.Json.messages[i].subject;
    project.SendInfoToLog($"Письмо {i}: {subject}");
}
[/CODE][/SPOILER]
[SPOILER= GalxeApi]

# Galxe

Класс для работы с платформой Galxe. Позволяет парсить задания на странице, получать информацию о пользователе и лояльные очки через GraphQL API.

## Конструкторы

### Galxe(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса Galxe.

**Параметры:**
- `project` (IZennoPosterProjectModel) - проект ZennoPoster
- `instance` (Instance) - экземпляр браузера
- `log` (bool) - включить логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
// создать экземпляр для работы с Galxe
var galxe = new Galxe(project, instance);

// создать экземпляр с логированием
var galxe = new Galxe(project, instance, true);
[/CODE]

## Методы

### ParseTasks(string type = "tasksUnComplete", bool log = false)

Парсит задания на текущей странице Galxe и возвращает список элементов в зависимости от типа.

**Возвращает:** `List<HtmlElement>` - список найденных элементов заданий

**Параметры:**
- `type` (string) - тип заданий для возврата: "tasksComplete", "tasksUnComplete", "reqComplete", "reqUnComplete", "refComplete", "refUnComplete" (по умолчанию "tasksUnComplete")
- `log` (bool) - включить логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
// получить невыполненные задания
var uncompletedTasks = galxe.ParseTasks("tasksUnComplete");
project.SendInfoToLog($"Найдено невыполненных заданий: {uncompletedTasks.Count}");

// получить выполненные требования
var completedRequirements = galxe.ParseTasks("reqComplete");

// пройти по всем невыполненным заданиям
foreach (var task in uncompletedTasks)
{
    project.SendInfoToLog($"Задание: {task.InnerText}");
    // кликнуть по заданию
    task.Click();
}
[/CODE]

### BasicUserInfo(string token, string address)

Получает базовую информацию о пользователе через GraphQL API Galxe.

**Возвращает:** `string` - JSON ответ с информацией о пользователе или null при ошибке

**Параметры:**
- `token` (string) - токен авторизации
- `address` (string) - адрес кошелька пользователя

**Пример:**
[CODE=csharp]
string userToken = "Bearer your_token_here";
string walletAddress = "0x1234567890abcdef...";

// получить информацию о пользователе
string userInfo = galxe.BasicUserInfo(userToken, walletAddress);

if (userInfo != null)
{
    project.Json.FromString(userInfo);
    
    // извлечь данные пользователя
    string username = project.Json.data.addressInfo.username;
    string userLevel = project.Json.data.addressInfo.userLevel.level.name;
    int experience = project.Json.data.addressInfo.userLevel.exp;
    
    project.SendInfoToLog($"Пользователь: {username}, Уровень: {userLevel}, Опыт: {experience}");
}
else
{
    project.SendErrorToLog("Не удалось получить информацию о пользователе");
}
[/CODE]

### GetLoyaltyPoints(string alias, string address)

Получает информацию о лояльных очках пользователя в конкретном пространстве Galxe.

**Возвращает:** `string` - JSON ответ с информацией об очках лояльности или null при ошибке

**Параметры:**
- `alias` (string) - псевдоним (алиас) пространства в Galxe
- `address` (string) - адрес кошелька пользователя

**Пример:**
[CODE=csharp]
string spaceAlias = "example-space";
string walletAddress = "0x1234567890abcdef...";

// получить очки лояльности
string loyaltyInfo = galxe.GetLoyaltyPoints(spaceAlias, walletAddress);

if (loyaltyInfo != null)
{
    project.Json.FromString(loyaltyInfo);
    
    // извлечь данные об очках
    int points = project.Json.data.space.addressLoyaltyPoints.points;
    int rank = project.Json.data.space.addressLoyaltyPoints.rank;
    
    project.SendInfoToLog($"Очки лояльности: {points}, Ранг: {rank}");
}
else
{
    project.SendErrorToLog("Не удалось получить очки лояльности");
}
[/CODE][/SPOILER]
[SPOILER= GasZipApi]

# GazZip

Класс для выполнения операций межсетевого перевода токенов (рефьюел) через протокол GasZip. Позволяет автоматически находить подходящую сеть с достаточным балансом и выполнять переводы в указанную целевую сеть.

## Конструкторы

### GazZip(IZennoPosterProjectModel project, string key = null, bool log = false)

Создает новый экземпляр класса GazZip для работы с межсетевыми переводами.

**Параметры:**
- `project` - проект ZennoPoster для работы с логами и переменными
- `key` - ключ для операций (опционально)  
- `log` - включить подробное логирование операций

**Пример:**
[CODE=csharp]
// создать экземпляр с логированием
var gazZip = new GazZip(project, log: true);

// создать экземпляр без логирования
var gazZip = new GazZip(project);
[/CODE]

## Методы

### Refuel(string chainTo, decimal value, string rpc = null, bool log = false)

**Возвращает:** `string` - хеш транзакции или сообщение об ошибке

Выполняет межсетевой перевод указанной суммы ETH в целевую сеть через протокол GasZip.

**Параметры:**
- `chainTo` - целевая сеть (поддерживает: "sepolia", "soneum", "bsc", "gravity", "zero", "opbnb" или hex-адрес)
- `value` - сумма для перевода в ETH
- `rpc` - RPC эндпоинт исходной сети (если не указан, автоматически выбирается подходящая)
- `log` - включить детальное логирование процесса

**Пример:**
[CODE=csharp]
var gazZip = new GazZip(project, log: true);

// автоматический выбор исходной сети
string txHash = gazZip.Refuel("sepolia", 0.001m);
project.SendInfoToLog($"Транзакция: {txHash}");

// использование конкретной исходной сети
string rpcUrl = "https://rpc.ankr.com/eth";
string result = gazZip.Refuel("bsc", 0.005m, rpcUrl, true);

if (result.StartsWith("fail:"))
{
    project.SendErrorToLog($"Ошибка: {result}");
}
else
{
    project.SendInfoToLog($"Успешно: {result}");
}
[/CODE][/SPOILER]
[SPOILER= GitHubApi]

# GitHubApi

Класс для работы с GitHub API, позволяющий управлять репозиториями, коллабораторами и их правами доступа. Предоставляет простые методы для создания репозиториев, изменения их видимости, добавления и удаления участников проекта.

## Конструкторы

### GitHubApi(string token, string username)

Создает новый экземпляр для работы с GitHub API.

**Параметры:**
- `token` - токен доступа к GitHub API
- `username` - имя пользователя GitHub

**Пример:**
[CODE=csharp]
//создать подключение к GitHub API
var github = new GitHubApi("your_token_here", "your_username");
[/CODE]

## Методы

### GetRepositoryInfo(string repoName)

**Возвращает:** `string` - информация о репозитории в формате JSON или сообщение об ошибке

Получает подробную информацию о указанном репозитории.

**Параметры:**
- `repoName` - название репозитория

**Пример:**
[CODE=csharp]
//получить информацию о репозитории
string repoInfo = github.GetRepositoryInfo("my-project");
project.SendInfoToLog(repoInfo);
[/CODE]

### GetCollaborators(string repoName)

**Возвращает:** `string` - список коллабораторов в формате JSON или сообщение об ошибке

Получает список всех участников репозитория.

**Параметры:**
- `repoName` - название репозитория

**Пример:**
[CODE=csharp]
//получить список участников репозитория
string collaborators = github.GetCollaborators("my-project");
project.SendInfoToLog(collaborators);
[/CODE]

### CreateRepository(string repoName)

**Возвращает:** `string` - данные созданного репозитория в формате JSON или сообщение об ошибке

Создает новый приватный репозиторий.

**Параметры:**
- `repoName` - название нового репозитория

**Пример:**
[CODE=csharp]
//создать новый приватный репозиторий
string result = github.CreateRepository("new-project");
project.SendInfoToLog("Репозиторий создан: " + result);
[/CODE]

### ChangeVisibility(string repoName, bool makePrivate)

**Возвращает:** `string` - результат операции в формате JSON или сообщение об ошибке

Изменяет видимость репозитория (публичный/приватный).

**Параметры:**
- `repoName` - название репозитория
- `makePrivate` - true для приватного, false для публичного

**Пример:**
[CODE=csharp]
//сделать репозиторий публичным
string result = github.ChangeVisibility("my-project", false);
project.SendInfoToLog("Видимость изменена: " + result);
[/CODE]

### AddCollaborator(string repoName, string collaboratorUsername, string permission = "pull")

**Возвращает:** `string` - результат операции в формате JSON или сообщение об ошибке

Добавляет нового участника в репозиторий с указанными правами.

**Параметры:**
- `repoName` - название репозитория
- `collaboratorUsername` - имя пользователя нового участника
- `permission` - права доступа: "pull", "push", "admin", "maintain", "triage" (по умолчанию "pull")

**Пример:**
[CODE=csharp]
//добавить участника с правами на запись
string result = github.AddCollaborator("my-project", "developer123", "push");
project.SendInfoToLog("Участник добавлен: " + result);
[/CODE]

### RemoveCollaborator(string repoName, string collaboratorUsername)

**Возвращает:** `string` - результат операции или сообщение об ошибке

Удаляет участника из репозитория.

**Параметры:**
- `repoName` - название репозитория
- `collaboratorUsername` - имя пользователя для удаления

**Пример:**
[CODE=csharp]
//удалить участника из репозитория
string result = github.RemoveCollaborator("my-project", "old-developer");
project.SendInfoToLog("Участник удален: " + result);
[/CODE]

### ChangeCollaboratorPermission(string repoName, string collaboratorUsername, string permission = "pull")

**Возвращает:** `string` - результат операции или сообщение об ошибке

Изменяет права доступа существующего участника репозитория.

**Параметры:**
- `repoName` - название репозитория
- `collaboratorUsername` - имя пользователя
- `permission` - новые права: "pull", "push", "admin", "maintain", "triage" (по умолчанию "pull")

**Пример:**
[CODE=csharp]
//изменить права участника на администраторские
string result = github.ChangeCollaboratorPermission("my-project", "developer123", "admin");
project.SendInfoToLog("Права изменены: " + result);
[/CODE][/SPOILER]
[SPOILER= OkxApi]

# OkxApi

Класс для работы с OKX API, предоставляющий функциональность для управления криптовалютными счетами, субсчетами, выводом средств и получением рыночных данных.

## Конструктор

### OkxApi(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр OkxApi для работы с биржей OKX.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `log` - включить ли логирование (по умолчанию false)

[CODE=csharp]
// Создание экземпляра OkxApi
var okx = new OkxApi(project, log: true);
[/CODE]

## Методы

### OKXGetSubAccs(string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список имен субсчетов

Получает список всех субсчетов на основном аккаунте OKX.

**Параметры:**
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Получить список всех субсчетов
var subAccounts = okx.OKXGetSubAccs();
foreach (string subName in subAccounts)
{
    project.SendInfoToLog($"Найден субсчет: {subName}");
}
[/CODE]

### OKXGetSubMax(string accName, string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список в формате "валюта:максимальная_сумма_вывода"

Получает максимальные суммы для вывода с торгового счета указанного субаккаунта.

**Параметры:**
- `accName` - имя субаккаунта
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Получить максимальные суммы для вывода с торгового счета
var maxWithdrawals = okx.OKXGetSubMax("sub1");
foreach (string balance in maxWithdrawals)
{
    project.SendInfoToLog($"Доступно для вывода: {balance}");
}
[/CODE]

### OKXGetSubTrading(string accName, string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список торговых балансов

Получает торговые балансы указанного субаккаунта.

**Параметры:**
- `accName` - имя субаккаунта
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Получить торговые балансы субсчета
var tradingBalances = okx.OKXGetSubTrading("sub1");
[/CODE]

### OKXGetSubFunding(string accName, string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список в формате "валюта:доступный_баланс"

Получает балансы фандинг-счета указанного субаккаунта.

**Параметры:**
- `accName` - имя субаккаунта
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Получить балансы фандинг-счета
var fundingBalances = okx.OKXGetSubFunding("sub1");
foreach (string balance in fundingBalances)
{
    string[] parts = balance.Split(':');
    project.SendInfoToLog($"Валюта: {parts[0]}, Баланс: {parts[1]}");
}
[/CODE]

### OKXGetSubsBal(string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список в формате "субсчет:валюта:баланс"

Получает все ненулевые балансы со всех субсчетов (как с фандинг, так и с торговых счетов).

**Параметры:**
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Получить все балансы со всех субсчетов
var allBalances = okx.OKXGetSubsBal(log: true);
foreach (string balance in allBalances)
{
    project.SendInfoToLog($"Баланс: {balance}");
}
[/CODE]

### OKXWithdraw(string toAddress, string currency, string chain, decimal amount, decimal fee, string proxy = null, bool log = false)

**Возвращает:** `void`

Выводит криптовалюту на указанный адрес.

**Параметры:**
- `toAddress` - адрес получателя
- `currency` - валюта для вывода (например, "USDT")
- `chain` - блокчейн сеть (ethereum, bsc, polygon и др.)
- `amount` - сумма для вывода
- `fee` - комиссия за вывод
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Вывести 100 USDT на Ethereum
okx.OKXWithdraw(
    "0x1234567890abcdef1234567890abcdef12345678", 
    "USDT", 
    "ethereum", 
    100m, 
    1m
);
[/CODE]

### OKXCreateSub(string subName, string accountType = "1", string proxy = null, bool log = false)

**Возвращает:** `void`

Создает новый субаккаунт.

**Параметры:**
- `subName` - имя нового субаккаунта
- `accountType` - тип аккаунта (по умолчанию "1")
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Создать новый субаккаунт
okx.OKXCreateSub("myNewSubAccount");
[/CODE]

### OKXDrainSubs()

**Возвращает:** `void`

Переводит все доступные средства со всех субсчетов на основной счет.

[CODE=csharp]
// Собрать все средства с субсчетов на основной счет
okx.OKXDrainSubs();
project.SendInfoToLog("Все средства переведены на основной счет");
[/CODE]

### OKXAddMaxSubs()

**Возвращает:** `void`

Создает максимально возможное количество субсчетов до достижения лимита.

[CODE=csharp]
// Создать максимальное количество субсчетов
okx.OKXAddMaxSubs();
project.SendInfoToLog("Созданы все возможные субсчета");
[/CODE]

### OKXPrice<T>(string pair, string proxy = null, bool log = false)

**Возвращает:** `T` - цена в указанном типе данных

Получает текущую рыночную цену торговой пары.

**Параметры:**
- `pair` - торговая пара (например, "BTC-USDT")
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

[CODE=csharp]
// Получить цену BTC в USDT как decimal
decimal btcPrice = okx.OKXPrice<decimal>("BTC-USDT");
project.SendInfoToLog($"Цена BTC: {btcPrice} USDT");

// Получить цену как строку
string priceString = okx.OKXPrice<string>("ETH-USDT");
[/CODE][/SPOILER]
[SPOILER= UnlockApi]

# UnlockApi

Класс для работы с Unlock Protocol смарт-контрактами в блокчейне Optimism. Позволяет получать информацию о владельцах NFT-ключей и времени истечения их действия.

## Конструкторы

### `UnlockApi(IZennoPosterProjectModel project, bool log = false)`

Создает новый экземпляр UnlockApi для работы с Unlock Protocol.

**Параметры:**
- `project` - ссылка на проект ZennoPoster для логирования
- `log` - включить дополнительное логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
//создать экземпляр UnlockApi
var unlockApi = new UnlockApi(project, true);
[/CODE]

## Методы

### `string keyExpirationTimestampFor(string addressTo, int tokenId, bool decode = true)`

**Возвращает:** string - временную метку истечения ключа

Получает временную метку истечения действия NFT-ключа по его ID.

**Параметры:**
- `addressTo` - адрес смарт-контракта Unlock
- `tokenId` - ID токена (ключа)
- `decode` - декодировать результат в читаемый формат (по умолчанию true)

**Пример:**
[CODE=csharp]
//получить время истечения ключа с ID 1
string expiration = unlockApi.keyExpirationTimestampFor("0x1234567890abcdef", 1);
project.SendInfoToLog($"Ключ истекает: {expiration}");
[/CODE]

### `string ownerOf(string addressTo, int tokenId, bool decode = true)`

**Возвращает:** string - адрес владельца ключа

Получает адрес владельца NFT-ключа по его ID.

**Параметры:**
- `addressTo` - адрес смарт-контракта Unlock
- `tokenId` - ID токена (ключа)
- `decode` - декодировать результат в читаемый формат (по умолчанию true)

**Пример:**
[CODE=csharp]
//получить владельца ключа с ID 1
string owner = unlockApi.ownerOf("0x1234567890abcdef", 1);
project.SendInfoToLog($"Владелец ключа: {owner}");
[/CODE]

### `string Decode(string toDecode, string function)`

**Возвращает:** string - декодированное значение

Декодирует результат вызова функции смарт-контракта из HEX формата в читаемый вид.

**Параметры:**
- `toDecode` - HEX строка для декодирования
- `function` - имя функции для определения типа декодирования

**Пример:**
[CODE=csharp]
//декодировать HEX результат
string result = unlockApi.Decode("0x000000000000000000000000a1b2c3d4e5f6", "ownerOf");
project.SendInfoToLog($"Декодированный результат: {result}");
[/CODE]

### `Dictionary<string, string> Holders(string contract)`

**Возвращает:** Dictionary<string, string> - словарь где ключ это адрес владельца, значение - время истечения

Получает всех держателей NFT-ключей контракта и время истечения их ключей.

**Параметры:**
- `contract` - адрес смарт-контракта Unlock

**Пример:**
[CODE=csharp]
//получить всех держателей ключей
var holders = unlockApi.Holders("0x1234567890abcdef");
foreach(var holder in holders)
{
    //вывести информацию о каждом держателе
    project.SendInfoToLog($"Владелец: {holder.Key}, истекает: {holder.Value}");
}
[/CODE][/SPOILER]
[/SPOILER]
