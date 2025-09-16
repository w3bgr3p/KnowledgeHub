[SPOILER= Requests] 

[SPOILER= NetHttp]

# NetHttp

Класс для выполнения HTTP запросов (GET, POST, PUT, DELETE) с поддержкой прокси, кастомных заголовков и автоматического логирования. Предоставляет удобный интерфейс для работы с веб-сервисами и API.

## Конструктор

### NetHttp(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса NetHttp.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `log` - включить логирование запросов (по умолчанию false)

**Пример:**
[CODE=csharp]
var httpClient = new NetHttp(project, true); //создать клиент с логированием
[/CODE]

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
[CODE=csharp]
var headers = new Dictionary<string, string> { {"Authorization", "Bearer token123"} };
string response = httpClient.GET("https://api.example.com/data", "+", headers, true);
//получить данные с авторизацией и парсингом JSON
project.SendInfoToLog(response);
[/CODE]

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
[CODE=csharp]
string jsonData = "{\"name\":\"test\",\"value\":123}";
var headers = new Dictionary<string, string> { {"Content-Type", "application/json"} };
string response = httpClient.POST("https://api.example.com/create", jsonData, "+", headers);
//отправить JSON данные на сервер
project.SendInfoToLog("Ответ: " + response);
[/CODE]

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
[CODE=csharp]
string updateData = "{\"id\":1,\"status\":\"active\"}";
string response = httpClient.PUT("https://api.example.com/update/1", updateData, "+");
//обновить запись с id=1
project.SendInfoToLog("Обновлено: " + response);
[/CODE]

### DELETE(string url, string proxyString = "", Dictionary<string, string> headers = null, string callerName = "")

**Возвращает:** `string` - ответ сервера

Выполняет DELETE запрос для удаления данных на сервере.

**Параметры:**
- `url` - адрес для запроса
- `proxyString` - строка прокси
- `headers` - дополнительные HTTP заголовки
- `callerName` - имя вызывающего метода для логирования

**Пример:**
[CODE=csharp]
string response = httpClient.DELETE("https://api.example.com/delete/1", "+");
//удалить запись с id=1
project.SendInfoToLog("Удалено: " + response);
[/CODE]

### CheckProxy(string proxyString = null)

**Возвращает:** `bool` - true если прокси работает корректно

Проверяет работоспособность прокси, сравнивая IP адреса с прокси и без него.

**Параметры:**
- `proxyString` - строка прокси для проверки (если null, берется из базы данных)

**Пример:**
[CODE=csharp]
bool isProxyWorking = httpClient.CheckProxy("192.168.1.1:8080");
if (isProxyWorking) {
} else {
    project.SendWarningToLog("Прокси не работает");
}
[/CODE]

### ProxySet(Instance instance, string proxyString = null)

**Возвращает:** `bool` - true если прокси успешно установлен

Проверяет и устанавливает прокси для экземпляра браузера.

**Параметры:**
- `instance` - экземпляр браузера для установки прокси
- `proxyString` - строка прокси (если null, берется из базы данных)

**Пример:**
[CODE=csharp]
bool proxySet = httpClient.ProxySet(instance, "user:pass@192.168.1.1:8080");
if (proxySet) {
} else {
    project.SendErrorToLog("Не удалось установить прокси");
}
[/CODE]
[/SPOILER]
[SPOILER= Requests]

# Requests

Расширения для выполнения HTTP-запросов в ZennoPoster с поддержкой прокси, логирования и автоматической обработки JSON. Предоставляет удобные методы GET и POST с настраиваемыми параметрами и обработкой ошибок.

## Методы

### GET

**Возвращает:** `string` - ответ сервера или пустую строку при ошибке

Выполняет HTTP GET-запрос к указанному URL с возможностью настройки прокси, заголовков и таймаута.

**Параметры:**
- `url` (string) - URL для запроса
- `proxy` (string, необязательный) - прокси в формате "ip:port" или "login:pass@ip:port", символ "+" для автоматического получения из переменных
- `headers` (string[], необязательный) - массив заголовков HTTP
- `log` (bool, необязательный) - включить логирование запроса и ответа
- `parseJson` (bool, необязательный) - автоматически парсить JSON-ответ
- `deadline` (int, необязательный) - таймаут в секундах (по умолчанию 15)
- `throwOnFail` (bool, необязательный) - выбрасывать исключение при ошибке

**Пример использования:**
[CODE=csharp]
//простой GET-запрос
string response = project.GET("https://api.example.com/data");

//запрос с прокси и логированием
string response = project.GET("https://api.example.com/data", 
    proxy: "192.168.1.1:8080", 
    log: true);

//запрос с заголовками и JSON-парсингом
string[] headers = {"Authorization: Bearer token123"};
string response = project.GET("https://api.example.com/user", 
    headers: headers, 
    parseJson: true);

//запрос с автоматическим прокси
string response = project.GET("https://api.example.com/data", 
    proxy: "+", 
    deadline: 30);
[/CODE]

### POST

**Возвращает:** `string` - ответ сервера или пустую строку при ошибке

Выполняет HTTP POST-запрос к указанному URL с телом запроса и настраиваемыми параметрами.

**Параметры:**
- `url` (string) - URL для запроса
- `body` (string) - тело POST-запроса
- `proxy` (string, необязательный) - прокси в формате "ip:port" или "login:pass@ip:port", символ "+" для автоматического получения
- `headers` (string[], необязательный) - массив заголовков HTTP
- `log` (bool, необязательный) - включить логирование запроса и ответа
- `parseJson` (bool, необязательный) - автоматически парсить JSON-ответ
- `deadline` (int, необязательный) - таймаут в секундах (по умолчанию 15)
- `throwOnFail` (bool, необязательный) - выбрасывать исключение при ошибке

**Пример использования:**
[CODE=csharp]
//простой POST-запрос с JSON
string jsonData = "{\"name\":\"test\",\"value\":123}";
string response = project.POST("https://api.example.com/create", jsonData);

//POST с прокси и логированием
string response = project.POST("https://api.example.com/update", 
    body: jsonData,
    proxy: "user:pass@192.168.1.1:8080",
    log: true);

//POST с заголовками авторизации
string[] headers = {"Authorization: Bearer token123", "Content-Type: application/json"};
string response = project.POST("https://api.example.com/secure", 
    body: jsonData,
    headers: headers,
    parseJson: true);
[/CODE]

### SetProxy

**Возвращает:** `void`

Устанавливает прокси для экземпляра браузера с проверкой корректности работы. Проверяет изменение IP-адреса для подтверждения работы прокси.

**Параметры:**
- `instance` (Instance) - экземпляр браузера для настройки прокси
- `proxyString` (string, необязательный) - строка прокси, если не указана - берется автоматически

**Пример использования:**
[CODE=csharp]
//установка прокси для текущего экземпляра
project.SetProxy(instance);

//установка конкретного прокси
project.SetProxy(instance, "192.168.1.1:8080");

//установка прокси с авторизацией
project.SetProxy(instance, "user:pass@192.168.1.1:8080");
[/CODE][/SPOILER]
[/SPOILER]
