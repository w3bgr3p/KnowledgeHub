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
```csharp
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
```

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
```csharp
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
```

### SetProxy

**Возвращает:** `void`

Устанавливает прокси для экземпляра браузера с проверкой корректности работы. Проверяет изменение IP-адреса для подтверждения работы прокси.

**Параметры:**
- `instance` (Instance) - экземпляр браузера для настройки прокси
- `proxyString` (string, необязательный) - строка прокси, если не указана - берется автоматически

**Пример использования:**
```csharp
//установка прокси для текущего экземпляра
project.SetProxy(instance);

//установка конкретного прокси
project.SetProxy(instance, "192.168.1.1:8080");

//установка прокси с авторизацией
project.SetProxy(instance, "user:pass@192.168.1.1:8080");
```