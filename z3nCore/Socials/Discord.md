# Discord

Класс для автоматизации работы с Discord в ZennoPoster. Предоставляет методы для загрузки учетных данных из базы данных, авторизации в Discord и получения списка серверов.

## Конструкторы

### Discord(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса Discord.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `instance` - экземпляр браузера
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
var discord = new Discord(project, instance, true);
//создать экземпляр Discord с включенным логированием
```

## Методы

### CredsFromDb()

**Возвращает:** `string` - статус аккаунта Discord

Загружает данные Discord аккаунта из базы данных и сохраняет их в переменные проекта.

**Пример:**
```csharp
string status = discord.CredsFromDb();
//загрузить данные аккаунта из БД
project.SendInfoToLog($"Статус аккаунта: {status}");
```

### Log(string tolog = "", string callerName = "", bool log = false)

**Возвращает:** `void`

Отправляет сообщение в лог проекта с префиксом Discord.

**Параметры:**
- `tolog` - текст сообщения для лога
- `callerName` - имя вызывающего метода (заполняется автоматически)
- `log` - принудительно показать лог даже если отключен

**Пример:**
```csharp
discord.Log("Начинаем авторизацию", log: true);
//отправить сообщение в лог с принудительным показом
```

### DSload(bool log = false)

**Возвращает:** `string` - результат загрузки (статус или имя пользователя)

Выполняет полную процедуру авторизации в Discord: загружает страницу, пытается авторизоваться по токену, при неудаче выполняет вход по логину/паролю.

**Параметры:**
- `log` - включить дополнительное логирование

**Пример:**
```csharp
string result = discord.DSload(true);
//авторизоваться в Discord
if (result == "ok" || result.Contains("@"))
{
    project.SendInfoToLog("Успешная авторизация");
}
```

### DSservers()

**Возвращает:** `string` - список серверов через запятую

Получает список всех серверов Discord, к которым подключен текущий аккаунт, включая серверы в папках.

**Пример:**
```csharp
string servers = discord.DSservers();
//получить список серверов
project.SendInfoToLog($"Серверы: {servers}");
```