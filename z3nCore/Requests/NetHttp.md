# NetHttp

Класс для выполнения HTTP запросов (GET, POST, PUT, DELETE) с поддержкой прокси, кастомных заголовков и автоматического логирования. Предоставляет удобный интерфейс для работы с веб-сервисами и API.

## Конструктор

### NetHttp(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса NetHttp.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `log` - включить логирование запросов (по умолчанию false)

**Пример:**
```csharp
var httpClient = new NetHttp(project, true); //создать клиент с логированием
```

## Методы

### GET(string url, string proxyString = "", Dictionary<string, string> headers = null, bool parse = false, int deadline = 15, bool throwOnFail = false)

**Возвращает:** `string` - ответ сервера

Выполняет GET запрос к указанному URL.

**Параметры:**
- `url` - адрес для запроса
- `proxyString` - строка прокси (формат: "ip:port" или "user:pass@ip:port")
- `headers` - дополнительные HTTP заголовки
- `parse` - автоматически парсить JSON ответ
- `deadline` - таймаут запроса в секундах
- `throwOnFail` - выбрасывать исключение при ошибке

**Пример:**
```csharp
var headers = new Dictionary<string, string> { {"Authorization", "Bearer token123"} };
string response = httpClient.GET("https://api.example.com/data", "+", headers, true);
//получить данные с авторизацией и парсингом JSON
project.SendInfoToLog(response);
```

### POST(string url, string body, string proxyString = "", Dictionary<string, string> headers = null, bool parse = false, int deadline = 15, string callerName = "", bool throwOnFail = false)

**Возвращает:** `string` - ответ сервера

Выполняет POST запрос с передачей данных в теле запроса.

**Параметры:**
- `url` - адрес для запроса
- `body` - тело запроса (обычно JSON)
- `proxyString` - строка прокси
- `headers` - дополнительные HTTP заголовки
- `parse` - автоматически парсить JSON ответ
- `deadline` - таймаут запроса в секундах
- `callerName` - имя вызывающего метода для логирования
- `throwOnFail` - выбрасывать исключение при ошибке

**Пример:**
```csharp
string jsonData = "{\"name\":\"test\",\"value\":123}";
var headers = new Dictionary<string, string> { {"Content-Type", "application/json"} };
string response = httpClient.POST("https://api.example.com/create", jsonData, "+", headers);
//отправить JSON данные на сервер
project.SendInfoToLog("Ответ: " + response);
```

### PUT(string url, string body = "", string proxyString = "", Dictionary<string, string> headers = null, bool parse = false, string callerName = "")

**Возвращает:** `string` - ответ сервера

Выполняет PUT запрос для обновления данных на сервере.

**Параметры:**
- `url` - адрес для запроса
- `body` - тело запроса (необязательно)
- `proxyString` - строка прокси
- `headers` - дополнительные HTTP заголовки
- `parse` - автоматически парсить JSON ответ
- `callerName` - имя вызывающего метода для логирования

**Пример:**
```csharp
string updateData = "{\"id\":1,\"status\":\"active\"}";
string response = httpClient.PUT("https://api.example.com/update/1", updateData, "+");
//обновить запись с id=1
project.SendInfoToLog("Обновлено: " + response);
```

### DELETE(string url, string proxyString = "", Dictionary<string, string> headers = null, string callerName = "")

**Возвращает:** `string` - ответ сервера

Выполняет DELETE запрос для удаления данных на сервере.

**Параметры:**
- `url` - адрес для запроса
- `proxyString` - строка прокси
- `headers` - дополнительные HTTP заголовки
- `callerName` - имя вызывающего метода для логирования

**Пример:**
```csharp
string response = httpClient.DELETE("https://api.example.com/delete/1", "+");
//удалить запись с id=1
project.SendInfoToLog("Удалено: " + response);
```

### CheckProxy(string proxyString = null)

**Возвращает:** `bool` - true если прокси работает корректно

Проверяет работоспособность прокси, сравнивая IP адреса с прокси и без него.

**Параметры:**
- `proxyString` - строка прокси для проверки (если null, берется из базы данных)

**Пример:**
```csharp
bool isProxyWorking = httpClient.CheckProxy("192.168.1.1:8080");
if (isProxyWorking) {
    project.SendInfoToLog("Прокси работает корректно");
} else {
    project.SendWarningToLog("Прокси не работает");
}
```

### ProxySet(Instance instance, string proxyString = null)

**Возвращает:** `bool` - true если прокси успешно установлен

Проверяет и устанавливает прокси для экземпляра браузера.

**Параметры:**
- `instance` - экземпляр браузера для установки прокси
- `proxyString` - строка прокси (если null, берется из базы данных)

**Пример:**
```csharp
bool proxySet = httpClient.ProxySet(instance, "user:pass@192.168.1.1:8080");
if (proxySet) {
    project.SendInfoToLog("Прокси установлен для браузера");
} else {
    project.SendErrorToLog("Не удалось установить прокси");
}
```
