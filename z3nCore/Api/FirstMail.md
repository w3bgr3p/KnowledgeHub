# FirstMail

Класс для работы с API сервиса временной почты FirstMail. Позволяет получать письма, извлекать из них коды подтверждения и ссылки.

## Конструкторы

### FirstMail(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса FirstMail.

**Параметры:**
- `project` (IZennoPosterProjectModel) - проект ZennoPoster
- `log` (bool) - включить логирование (по умолчанию false)

**Пример:**
```csharp
// создать экземпляр без логирования
var firstMail = new FirstMail(project);

// создать экземпляр с логированием
var firstMail = new FirstMail(project, true);
```

## Методы

### GetMail(string email)

Получает последнее письмо для указанного email адреса и загружает его в JSON объект проекта.

**Возвращает:** `string` - JSON ответ от API

**Параметры:**
- `email` (string) - email адрес для получения писем

**Пример:**
```csharp
// получить последнее письмо
string jsonResponse = firstMail.GetMail("user@example.com");

// данные письма теперь доступны через project.Json
string subject = project.Json.subject; // тема письма
string text = project.Json.text;       // текст письма
```

### GetOTP(string email)

Извлекает 6-значный код подтверждения из последнего письма. Ищет код в теме, тексте и HTML содержимом письма.

**Возвращает:** `string` - найденный 6-значный код

**Параметры:**
- `email` (string) - email адрес для поиска кода

**Пример:**
```csharp
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
```

### GetLink(string email)

Извлекает первую найденную ссылку (http/https) из текста последнего письма.

**Возвращает:** `string` - найденная ссылка

**Параметры:**
- `email` (string) - email адрес для поиска ссылки

**Пример:**
```csharp
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
```

### Delete(string email, bool seen = false)

Удаляет письма для указанного email адреса.

**Возвращает:** `string` - JSON ответ от API

**Параметры:**
- `email` (string) - email адрес
- `seen` (bool) - удалить только прочитанные письма (по умолчанию false)

**Пример:**
```csharp
// удалить все письма
string result = firstMail.Delete("user@example.com");

// удалить только прочитанные письма
string result = firstMail.Delete("user@example.com", true);
```

### GetOne(string email)

Получает одно последнее письмо для указанного email адреса.

**Возвращает:** `string` - JSON ответ от API

**Параметры:**
- `email` (string) - email адрес

**Пример:**
```csharp
// получить одно письмо
string message = firstMail.GetOne("user@example.com");
project.Json.FromString(message);
```

### GetAll(string email)

Получает все письма для указанного email адреса.

**Возвращает:** `string` - JSON ответ от API с массивом писем

**Параметры:**
- `email` (string) - email адрес

**Пример:**
```csharp
// получить все письма
string allMessages = firstMail.GetAll("user@example.com");
project.Json.FromString(allMessages);

// обработать каждое письмо в цикле
for (int i = 0; i < project.Json.messages.Count; i++)
{
    string subject = project.Json.messages[i].subject;
    project.SendInfoToLog($"Письмо {i}: {subject}");
}
```