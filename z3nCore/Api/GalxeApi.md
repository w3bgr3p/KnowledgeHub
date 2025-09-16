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
```csharp
// создать экземпляр для работы с Galxe
var galxe = new Galxe(project, instance);

// создать экземпляр с логированием
var galxe = new Galxe(project, instance, true);
```

## Методы

### ParseTasks(string type = "tasksUnComplete", bool log = false)

Парсит задания на текущей странице Galxe и возвращает список элементов в зависимости от типа.

**Возвращает:** `List<HtmlElement>` - список найденных элементов заданий

**Параметры:**
- `type` (string) - тип заданий для возврата: "tasksComplete", "tasksUnComplete", "reqComplete", "reqUnComplete", "refComplete", "refUnComplete" (по умолчанию "tasksUnComplete")
- `log` (bool) - включить логирование (по умолчанию false)

**Пример:**
```csharp
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
```

### BasicUserInfo(string token, string address)

Получает базовую информацию о пользователе через GraphQL API Galxe.

**Возвращает:** `string` - JSON ответ с информацией о пользователе или null при ошибке

**Параметры:**
- `token` (string) - токен авторизации
- `address` (string) - адрес кошелька пользователя

**Пример:**
```csharp
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
```

### GetLoyaltyPoints(string alias, string address)

Получает информацию о лояльных очках пользователя в конкретном пространстве Galxe.

**Возвращает:** `string` - JSON ответ с информацией об очках лояльности или null при ошибке

**Параметры:**
- `alias` (string) - псевдоним (алиас) пространства в Galxe
- `address` (string) - адрес кошелька пользователя

**Пример:**
```csharp
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
```