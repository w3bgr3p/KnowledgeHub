# Traffic - Класс для работы с сетевым трафиком

Класс Traffic предназначен для мониторинга и анализа HTTP-трафика в ZennoPoster. Он позволяет отслеживать запросы на активной вкладке, извлекать данные из них и получать конкретные параметры запросов/ответов. Особенно полезен для работы с API, перехвата токенов авторизации и анализа сетевых взаимодействий.

## Конструкторы

### Traffic(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса для мониторинга трафика.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `instance` - экземпляр браузера  
- `log` - включить логирование процесса (по умолчанию false)

**Пример:**
```csharp
//создать экземпляр для работы с трафиком
var traffic = new Traffic(project, instance, true);
```

## Методы

### string Get(string url, string parametr, bool reload = false, bool parse = false, int deadline = 15, int delay = 3)

**Возвращаемое значение:** string - значение указанного параметра из трафика

Извлекает конкретный параметр из HTTP-запроса, содержащего указанный URL. Ожидает появления запроса и возвращает данные когда находит совпадение.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `parametr` - параметр для извлечения: "Method", "ResultCode", "Url", "ResponseContentType", "RequestHeaders", "RequestCookies", "RequestBody", "ResponseHeaders", "ResponseCookies", "ResponseBody"
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false)
- `parse` - автоматически парсить JSON ответ (по умолчанию false)
- `deadline` - максимальное время ожидания в секундах (по умолчанию 15)
- `delay` - задержка между проверками в секундах (по умолчанию 3)

**Пример:**
```csharp
//получить токен из заголовков запроса
string token = traffic.Get("api/login", "RequestHeaders");

//получить тело ответа от API
string response = traffic.Get("api/user", "ResponseBody", parse: true);
```

### Dictionary<string, string> Get(string url, bool reload = false, int deadline = 10)

**Возвращаемое значение:** Dictionary<string, string> - все параметры найденного запроса

Возвращает все доступные параметры HTTP-запроса в виде словаря. Содержит Method, ResultCode, Url, ResponseContentType, RequestHeaders, RequestCookies, RequestBody, ResponseHeaders, ResponseCookies, ResponseBody.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false) 
- `deadline` - максимальное время ожидания в секундах (по умолчанию 10)

**Пример:**
```csharp
//получить все данные о запросе
var requestData = traffic.Get("api/users");

//использовать данные запроса
string method = requestData["Method"];
string statusCode = requestData["ResultCode"];
string responseBody = requestData["ResponseBody"];
```

### string GetHeader(string url, string headerToGet = "Authorization", bool reload = false)

**Возвращаемое значение:** string - значение указанного заголовка

Извлекает конкретный заголовок из HTTP-запроса. По умолчанию ищет заголовок Authorization, часто используемый для токенов.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `headerToGet` - имя заголовка для получения (по умолчанию "Authorization")
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false)

**Пример:**
```csharp
//получить токен авторизации
string authToken = traffic.GetHeader("api/profile");

//получить user-agent
string userAgent = traffic.GetHeader("api/data", "User-Agent");
```

### string GetParam(string url, string parametr, bool reload = false, int deadline = 10)

**Возвращаемое значение:** string - значение указанного параметра

Альтернативный метод для получения конкретного параметра из HTTP-запроса. Работает аналогично первому методу Get, но с упрощенными параметрами.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `parametr` - параметр для извлечения
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false)
- `deadline` - максимальное время ожидания в секундах (по умолчанию 10)

**Пример:**
```csharp
//получить код ответа
string statusCode = traffic.GetParam("api/login", "ResultCode");

//получить тело запроса
string requestBody = traffic.GetParam("api/submit", "RequestBody");
```