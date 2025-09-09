# Работа с переменными проекта в ZennoPoster 


### ⚠️ Переменные должны быть созданы заранее
- **Все переменные должны быть созданы в ProjectMaker до выполнения кода**
- Попытка доступа к несуществующей переменной вызовет исключение
- API `IZennoPosterProjectModel` позволяет только читать и изменять существующие переменные, но НЕ создавать новые

### ⚠️ Тип данных переменных
- **Все переменные в ZennoPoster имеют тип `string`**
- При работе с числами, булевыми значениями и другими типами необходимо выполнять конвертацию



## 📋 ОСНОВЫ РАБОТЫ С ЛОКАЛЬНЫМИ ПЕРЕМЕННЫМИ

### Получение значения переменной
```csharp
// Базовое получение значения
string userName = project.Variables["UserName"].Value;

// Получение с проверкой на null/пустое значение
string email = project.Variables["Email"].Value;
if (string.IsNullOrEmpty(email)) {
    project.SendWarningToLog("Переменная Email не задана или пуста", false);
}
```

### Установка значения переменной
```csharp
// Установка строкового значения
project.Variables["UserName"].Value = "ivan_petrov";

// Установка числового значения (конвертация в string)
project.Variables["Age"].Value = "25";

// Установка булевого значения
project.Variables["IsLoggedIn"].Value = "True";

// Установка текущей даты и времени
project.Variables["CurrentDateTime"].Value = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
```

## 🔢 РАБОТА С ЧИСЛОВЫМИ ДАННЫМИ

### Конвертация из string в числовые типы
```csharp
// Получение числового значения из переменной
string countStr = project.Variables["Count"].Value;
int count = 0;
if (int.TryParse(countStr, out count)) {
    // Успешная конвертация
    count++;
    project.Variables["Count"].Value = count.ToString();
} else {
    // Ошибка конвертации, устанавливаем значение по умолчанию
    project.Variables["Count"].Value = "1";
    project.SendWarningToLog($"Не удалось преобразовать '{countStr}' в число, установлено значение 1", false);
}

// Работа с double
string priceStr = project.Variables["Price"].Value;
double price = 0.0;
if (double.TryParse(priceStr, out price)) {
    price *= 1.2; // Увеличиваем цену на 20%
    project.Variables["Price"].Value = price.ToString("F2"); // Форматируем до 2 знаков после запятой
}
```

### Арифметические операции
```csharp
// Инкремент счетчика
string attemptStr = project.Variables["Attempts"].Value;
int attempts = int.TryParse(attemptStr, out int temp) ? temp : 0;
attempts++;
project.Variables["Attempts"].Value = attempts.ToString();

// Вычисление суммы
string sum1Str = project.Variables["Sum1"].Value;
string sum2Str = project.Variables["Sum2"].Value;
int sum1 = int.TryParse(sum1Str, out int s1) ? s1 : 0;
int sum2 = int.TryParse(sum2Str, out int s2) ? s2 : 0;
int total = sum1 + sum2;
project.Variables["TotalSum"].Value = total.ToString();
```

## 🔤 РАБОТА СО СТРОКОВЫМИ ДАННЫМИ

### Конкатенация строк
```csharp
// Простая конкатенация
string firstName = project.Variables["FirstName"].Value;
string lastName = project.Variables["LastName"].Value;
project.Variables["FullName"].Value = firstName + " " + lastName;

// Форматирование строки
string template = project.Variables["MessageTemplate"].Value;
string userName = project.Variables["UserName"].Value;
project.Variables["PersonalMessage"].Value = template.Replace("{username}", userName);
```

### Обработка текста
```csharp
// Удаление лишних пробелов
string rawText = project.Variables["RawInput"].Value;
project.Variables["CleanInput"].Value = rawText.Trim();

// Изменение регистра
string email = project.Variables["Email"].Value;
project.Variables["EmailLower"].Value = email.ToLower();

// Проверка на содержание подстроки
string url = project.Variables["CurrentUrl"].Value;
if (url.Contains("success")) {
    project.Variables["IsSuccess"].Value = "True";
} else {
    project.Variables["IsSuccess"].Value = "False";
}
```

## 🔀 РАБОТА С БУЛЕВЫМИ ЗНАЧЕНИЯМИ

### Установка и проверка булевых переменных
```csharp
// Установка булевого значения
project.Variables["IsProcessed"].Value = "True";
project.Variables["HasErrors"].Value = "False";

// Проверка булевого значения
string isLoggedInStr = project.Variables["IsLoggedIn"].Value;
bool isLoggedIn = isLoggedInStr == "True" || isLoggedInStr.ToLower() == "True";

if (isLoggedIn) {
    // Пользователь авторизован
    project.Variables["LastLoginTime"].Value = DateTime.Now.ToString();
} else {
    // Необходима авторизация
    project.Variables["LoginRequired"].Value = "True";
}

// Переключение булевого значения
string currentState = project.Variables["FeatureEnabled"].Value;
bool isEnabled = currentState == "True";
project.Variables["FeatureEnabled"].Value = (!isEnabled).ToString().ToLower();
```

## 📅 РАБОТА С ДАТОЙ И ВРЕМЕНЕМ

### Установка даты и времени
```csharp
// Текущая дата и время
project.Variables["CurrentDateTime"].Value = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");

// Только дата
project.Variables["CurrentDate"].Value = DateTime.Now.ToString("yyyy-MM-dd");

// Только время
project.Variables["CurrentTime"].Value = DateTime.Now.ToString("HH:mm:ss");

// Дата в различных форматах
project.Variables["DateRus"].Value = DateTime.Now.ToString("dd.MM.yyyy");
project.Variables["DateUS"].Value = DateTime.Now.ToString("MM/dd/yyyy");

// Unix timestamp
project.Variables["Timestamp"].Value = ((DateTimeOffset)DateTime.Now).ToUnixTimeSeconds().ToString();
```

### Операции с датами
```csharp
// Добавление дней к дате
string currentDateStr = project.Variables["StartDate"].Value;
if (DateTime.TryParse(currentDateStr, out DateTime startDate)) {
    DateTime endDate = startDate.AddDays(30);
    project.Variables["EndDate"].Value = endDate.ToString("yyyy-MM-dd");
}

// Вычисление разности между датами
string date1Str = project.Variables["Date1"].Value;
string date2Str = project.Variables["Date2"].Value;
if (DateTime.TryParse(date1Str, out DateTime date1) && DateTime.TryParse(date2Str, out DateTime date2)) {
    TimeSpan difference = date2 - date1;
    project.Variables["DaysDifference"].Value = difference.Days.ToString();
}
```

## 🌐 РАБОТА С URL И ПУТЯМИ

### Формирование URL
```csharp
// Базовый URL с параметрами
string baseUrl = project.Variables["BaseUrl"].Value;
string userId = project.Variables["UserId"].Value;
string apiKey = project.Variables["ApiKey"].Value;
project.Variables["FullUrl"].Value = $"{baseUrl}/user/{userId}?key={apiKey}";

// URL-кодирование параметров
string searchQuery = project.Variables["SearchQuery"].Value;
string encodedQuery = Macros.TextProcessing.UrlEncode(searchQuery, "utf-8");
project.Variables["EncodedQuery"].Value = encodedQuery;
```

### Извлечение частей URL
```csharp
// Извлечение домена из URL
string fullUrl = project.Variables["FullUrl"].Value;
if (Uri.TryCreate(fullUrl, UriKind.Absolute, out Uri uri)) {
    project.Variables["Domain"].Value = uri.Host;
    project.Variables["Path"].Value = uri.AbsolutePath;
    project.Variables["Query"].Value = uri.Query;
}
```

## 🔍 УСЛОВНАЯ ЛОГИКА С ПЕРЕМЕННЫМИ

### If-else конструкции
```csharp
// Простое условие
string status = project.Variables["Status"].Value;
if (status == "success") {
    project.Variables["Message"].Value = "Операция выполнена успешно";
} else if (status == "error") {
    project.Variables["Message"].Value = "Произошла ошибка";
} else {
    project.Variables["Message"].Value = "Неизвестный статус";
}

// Множественные условия
string userType = project.Variables["UserType"].Value;
string isVip = project.Variables["IsVip"].Value;
string discount = "0";

if (userType == "premium") {
    discount = "15";
} else if (userType == "regular" && isVip == "True") {
    discount = "10";
} else if (userType == "regular") {
    discount = "5";
}
project.Variables["Discount"].Value = discount;
```

### Switch-подобная логика
```csharp
// Эмуляция switch через if-else
string action = project.Variables["Action"].Value;
string result = "";

if (action == "login") {
    result = "Выполняется авторизация...";
} else if (action == "logout") {
    result = "Выполняется выход...";
} else if (action == "register") {
    result = "Выполняется регистрация...";
} else {
    result = "Неизвестное действие";
}
project.Variables["ActionResult"].Value = result;
```

## 🔧 ПРАКТИЧЕСКИЕ ПРИМЕРЫ

### Счетчик попыток с ограничением
```csharp
// Получаем текущее количество попыток
string attemptsStr = project.Variables["LoginAttempts"].Value;
int attempts = int.TryParse(attemptsStr, out int temp) ? temp : 0;

// Увеличиваем счетчик
attempts++;
project.Variables["LoginAttempts"].Value = attempts.ToString();

// Проверяем лимит
int maxAttempts = 3;
if (attempts >= maxAttempts) {
    project.Variables["MaxAttemptsReached"].Value = "True";
    project.SendWarningToLog($"Достигнуто максимальное количество попыток: {attempts}", false);
    
    // Прерываем выполнение
    ZennoPoster.SetTries(new Guid(project.TaskId), 0);
}
```

### Генерация уникальных идентификаторов
```csharp
// GUID
project.Variables["UniqueId"].Value = Guid.NewGuid().ToString();

// Timestamp + случайное число
string timestamp = DateTimeOffset.Now.ToUnixTimeSeconds().ToString();
string random = Global.Classes.rnd.Next(1000, 9999).ToString();
project.Variables["SessionId"].Value = timestamp + "_" + random;
```

### Работа с массивами данных в одной переменной
```csharp
// Сохранение массива как строки с разделителем
string[] emails = {"test1@mail.com", "test2@mail.com", "test3@mail.com"};
project.Variables["EmailList"].Value = string.Join("|", emails);

// Извлечение массива из строки
string emailListStr = project.Variables["EmailList"].Value;
if (!string.IsNullOrEmpty(emailListStr)) {
    string[] emailArray = emailListStr.Split('|');
    if (emailArray.Length > 0) {
        project.Variables["FirstEmail"].Value = emailArray[0];
        project.Variables["EmailCount"].Value = emailArray.Length.ToString();
    }
}
```

### Валидация данных
```csharp
// Проверка email
string email = project.Variables["Email"].Value;
bool isValidEmail = !string.IsNullOrEmpty(email) && email.Contains("@") && email.Contains(".");
project.Variables["IsValidEmail"].Value = isValidEmail.ToString().ToLower();

// Проверка телефона (простая)
string phone = project.Variables["Phone"].Value;
bool isValidPhone = !string.IsNullOrEmpty(phone) && phone.Length >= 10 && phone.All(char.IsDigit);
project.Variables["IsValidPhone"].Value = isValidPhone.ToString().ToLower();

// Комплексная валидация
bool allValid = isValidEmail && isValidPhone;
project.Variables["AllFieldsValid"].Value = allValid.ToString().ToLower();
```

## ⚡ ОПТИМИЗАЦИЯ И ЛУЧШИЕ ПРАКТИКИ

### Избегание повторных обращений
```csharp
// ❌ Плохо - два поиска в коллекции дороже создания ссылки
if (project.Variables["Status"].Value == "processing") {
    project.SendInfoToLog($"Статус: {project.Variables["Status"].Value}", false);
}

// ✅ Хорошо - один поиск + консистентность данных
string status = project.Variables["Status"].Value;
if (status == "processing") {
    project.SendInfoToLog($"Статус: {status}", false);
}
```

### Инициализация переменных по умолчанию
```csharp
// Проверка и установка значений по умолчанию
string counter = project.Variables["Counter"].Value;
if (string.IsNullOrEmpty(counter)) {
    project.Variables["Counter"].Value = "0";
}

string status = project.Variables["Status"].Value;
if (string.IsNullOrEmpty(status)) {
    project.Variables["Status"].Value = "idle";
}
```

### Централизованная обработка ошибок
```csharp
// Функция для безопасного получения числовой переменной
int GetIntVariable(string variableName, int defaultValue = 0) {
    string valueStr = project.Variables[variableName].Value;
    if (int.TryParse(valueStr, out int result)) {
        return result;
    } else {
        project.Variables[variableName].Value = defaultValue.ToString();
        return defaultValue;
    }
}

// Использование
int attempts = GetIntVariable("Attempts", 0);
int maxRetries = GetIntVariable("MaxRetries", 5);
```

## 🚨 ОБРАБОТКА ОШИБОК

### Безопасная работа с переменными
```csharp
try {
    // Попытка доступа к переменной
    string value = project.Variables["SomeVariable"].Value;
    
    // Обработка значения
    if (!string.IsNullOrEmpty(value)) {
        project.Variables["ProcessedValue"].Value = value.ToUpper();
    }
    
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка при работе с переменной: {ex.Message}", false);
    
    // Установка безопасного значения по умолчанию
    project.Variables["ProcessedValue"].Value = "DEFAULT_VALUE";
}
```

### Проверка существования переменной через try-catch
```csharp
// Функция для проверки существования переменной
bool VariableExists(string variableName) {
    try {
        string value = project.Variables[variableName].Value;
        return true;
    } catch {
        return false;
    }
}

// Использование
if (VariableExists("OptionalSetting")) {
    string setting = project.Variables["OptionalSetting"].Value;
    // Работаем с переменной
} else {
    project.SendInfoToLog($"Переменная OptionalSetting не найдена", false);
}
```


## 🌐 ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ

### Что такое глобальные переменные?
Глобальные переменные в ZennoPoster - это переменные, которые:
- Доступны всем потокам и проектам в рамках одного экземпляра ZennoPoster
- Организованы по пространствам имен (namespaces) для лучшей структуризации
- Могут быть созданы программно во время выполнения
- Сохраняют свои значения между запусками разных потоков

### Получение глобальной переменной
```csharp
// Получение глобальной переменной по namespace и имени
IGlobalVariable gv = project.GlobalVariables["MyNamespace", "VariableName"];

// Получение значения
string value = gv.Value.ToString();

// Более безопасное получение с проверкой на null
IGlobalVariable gv = project.GlobalVariables["Settings", "ApiKey"];
if (gv != null && gv.Value != null) {
    string apiKey = gv.Value.ToString();
    project.SendInfoToLog($"API ключ получен: {apiKey}", false);
} else {
    project.SendWarningToLog("Глобальная переменная ApiKey не найдена", false);
}
```

### Создание и установка глобальных переменных
```csharp
// Создание новой глобальной переменной с установкой значения
project.GlobalVariables.SetVariable("MyNamespace", "NewVariableName", "NewValue");

// Создание переменных для разных целей
project.GlobalVariables.SetVariable("Counters", "TotalProcessed", "0");
project.GlobalVariables.SetVariable("Settings", "MaxRetries", "5");
project.GlobalVariables.SetVariable("Auth", "AccessToken", "your_token_here");
project.GlobalVariables.SetVariable("Timing", "LastUpdate", DateTime.Now.ToString());
```

### Изменение значения существующей глобальной переменной
```csharp
// Получаем переменную и изменяем её значение
IGlobalVariable counter = project.GlobalVariables["Counters", "TotalProcessed"];
if (counter != null) {
    int currentValue = 0;
    if (int.TryParse(counter.Value.ToString(), out currentValue)) {
        currentValue++;
        counter.Value = currentValue.ToString();
    } else {
        counter.Value = "1";
    }
}

// Более короткий способ через SetVariable (перезапишет существующую)
string currentCountStr = project.GlobalVariables["Counters", "TotalProcessed"]?.Value?.ToString() ?? "0";
int currentCount = int.TryParse(currentCountStr, out int temp) ? temp : 0;
currentCount++;
project.GlobalVariables.SetVariable("Counters", "TotalProcessed", currentCount.ToString());
```

### Работа с метаданными глобальной переменной
```csharp
// Получение информации о переменной
IGlobalVariable gv = project.GlobalVariables["Settings", "DatabaseUrl"];
if (gv != null) {
    string name = gv.Name;           // Имя переменной
    string ns = gv.Namespace;        // Пространство имен
    string comment = gv.Comment;     // Комментарий
    object value = gv.Value;         // Значение
    
    // Установка комментария
    if (string.IsNullOrWhiteSpace(comment)) {
        gv.Comment = "URL для подключения к базе данных";
    }
    
    project.SendInfoToLog($"Переменная {ns}.{name}: {value} ({comment})", false);
}
```

## 🔄 ПРАКТИЧЕСКИЕ СЦЕНАРИИ ИСПОЛЬЗОВАНИЯ ГЛОБАЛЬНЫХ ПЕРЕМЕННЫХ

### Глобальный счетчик для всех потоков
```csharp
// Инициализация счетчика (выполнять только один раз)
project.GlobalVariables.SetVariable("Statistics", "ProcessedItems", "0");

// Увеличение счетчика в каждом потоке (потокобезопасно)
IGlobalVariable counter = project.GlobalVariables["Statistics", "ProcessedItems"];
if (counter != null) {
    lock (counter) {
        int current = int.TryParse(counter.Value.ToString(), out int temp) ? temp : 0;
        current++;
        counter.Value = current.ToString();
        
        project.SendInfoToLog($"Обработано элементов: {current}", false);
        
        // Проверка лимита
        if (current >= 1000) {
            project.GlobalVariables.SetVariable("Control", "StopProcessing", "True");
        }
    }
}
```

### Общие настройки для всех потоков
```csharp
// Инициализация настроек (в главном потоке или один раз)
project.GlobalVariables.SetVariable("Config", "ApiUrl", "https://api.example.com");
project.GlobalVariables.SetVariable("Config", "Timeout", "30");
project.GlobalVariables.SetVariable("Config", "UserAgent", "MyBot/1.0");

// Использование настроек в любом потоке
string apiUrl = project.GlobalVariables["Config", "ApiUrl"]?.Value?.ToString() ?? "";
string timeoutStr = project.GlobalVariables["Config", "Timeout"]?.Value?.ToString() ?? "30";
string userAgent = project.GlobalVariables["Config", "UserAgent"]?.Value?.ToString() ?? "DefaultUA";

int timeout = int.TryParse(timeoutStr, out int t) ? t * 1000 : 30000;

// Использование в HTTP запросе
string response = ZennoPoster.HTTP.Request(
    method: ZennoLab.InterfacesLibrary.Enums.Http.HttpMethod.GET,
    url: apiUrl + "/data",
    proxy: project.GetProxy(),
    UserAgent: userAgent,
    Timeout: timeout,
    respType: ZennoLab.InterfacesLibrary.Enums.Http.ResponceType.BodyOnly
);
```

### Система глобальных флагов управления
```csharp
// Инициализация флагов управления
project.GlobalVariables.SetVariable("Control", "PauseAllThreads", "False");
project.GlobalVariables.SetVariable("Control", "StopOnError", "False");
project.GlobalVariables.SetVariable("Control", "MaintenanceMode", "False");

// Проверка флагов в каждом потоке
string pauseFlag = project.GlobalVariables["Control", "PauseAllThreads"]?.Value?.ToString() ?? "False";
if (pauseFlag == "True") {
    project.SendInfoToLog("Поток приостановлен по глобальному флагу", false);
    
    // Ожидание снятия флага
    while (pauseFlag == "True") {
        Thread.Sleep(5000);
        pauseFlag = project.GlobalVariables["Control", "PauseAllThreads"]?.Value?.ToString() ?? "False";
    }
}

// Установка флага при критической ошибке
try {
    // Критически важная операция
} catch (Exception ex) {
    project.SendErrorToLog($"Критическая ошибка: {ex.Message}", true);
    project.GlobalVariables.SetVariable("Control", "StopOnError", "True");
    
    // Прерывание выполнения
    ZennoPoster.SetTries(new Guid(project.TaskId), 0);
}
```

### Кеширование данных между потоками
```csharp
// Кеширование токена авторизации
string cachedToken = project.GlobalVariables["Cache", "AuthToken"]?.Value?.ToString() ?? "";
string tokenExpiry = project.GlobalVariables["Cache", "TokenExpiry"]?.Value?.ToString() ?? "";

bool needNewToken = string.IsNullOrEmpty(cachedToken);
if (!string.IsNullOrEmpty(tokenExpiry)) {
    if (DateTime.TryParse(tokenExpiry, out DateTime expiry)) {
        needNewToken = DateTime.Now >= expiry;
    }
}

if (needNewToken) {
    // Получение нового токена (только один поток должен это делать)
    IGlobalVariable tokenVar = project.GlobalVariables["Cache", "AuthToken"];
    if (tokenVar != null) {
        lock (tokenVar) {
            // Повторная проверка после получения блокировки
            cachedToken = tokenVar.Value?.ToString() ?? "";
            if (string.IsNullOrEmpty(cachedToken)) {
                // Получаем новый токен
                string newToken = GetAuthToken(); // Ваша функция получения токена
                DateTime newExpiry = DateTime.Now.AddHours(1);
                
                project.GlobalVariables.SetVariable("Cache", "AuthToken", newToken);
                project.GlobalVariables.SetVariable("Cache", "TokenExpiry", newExpiry.ToString());
                
                cachedToken = newToken;
            }
        }
    }
}

// Использование кешированного токена
string headers = $"Authorization: Bearer {cachedToken}";
```

### Глобальная статистика и мониторинг
```csharp
// Инициализация статистики
project.GlobalVariables.SetVariable("Stats", "TotalRequests", "0");
project.GlobalVariables.SetVariable("Stats", "SuccessfulRequests", "0");
project.GlobalVariables.SetVariable("Stats", "FailedRequests", "0");
project.GlobalVariables.SetVariable("Stats", "StartTime", DateTime.Now.ToString());

// Обновление статистики в потоке
void UpdateStats(bool isSuccess) {
    // Общее количество запросов
    IGlobalVariable totalVar = project.GlobalVariables["Stats", "TotalRequests"];
    if (totalVar != null) {
        lock (totalVar) {
            int total = int.TryParse(totalVar.Value.ToString(), out int temp) ? temp : 0;
            total++;
            totalVar.Value = total.ToString();
        }
    }
    
    // Успешные или неуспешные
    string statType = isSuccess ? "SuccessfulRequests" : "FailedRequests";
    IGlobalVariable statVar = project.GlobalVariables["Stats", statType];
    if (statVar != null) {
        lock (statVar) {
            int count = int.TryParse(statVar.Value.ToString(), out int temp) ? temp : 0;
            count++;
            statVar.Value = count.ToString();
        }
    }
}

// Использование
try {
    // Выполнение запроса
    string response = ZennoPoster.HTTP.Request(/* параметры */);
    UpdateStats(true);
} catch {
    UpdateStats(false);
}

// Вывод статистики
string total = project.GlobalVariables["Stats", "TotalRequests"]?.Value?.ToString() ?? "0";
string success = project.GlobalVariables["Stats", "SuccessfulRequests"]?.Value?.ToString() ?? "0";
string failed = project.GlobalVariables["Stats", "FailedRequests"]?.Value?.ToString() ?? "0";
project.SendInfoToLog($"Статистика: Всего={total}, Успешно={success}, Ошибок={failed}", false);
```

## 🔒 ПОТОКОБЕЗОПАСНОСТЬ ГЛОБАЛЬНЫХ ПЕРЕМЕННЫХ

### Правильное использование lock
```csharp
// Правильный способ - блокировка на объекте переменной
IGlobalVariable sharedCounter = project.GlobalVariables["Shared", "Counter"];
if (sharedCounter != null) {
    lock (sharedCounter) {
        int current = int.TryParse(sharedCounter.Value.ToString(), out int temp) ? temp : 0;
        current++;
        sharedCounter.Value = current.ToString();
        
        // Любые другие операции с этой переменной
        project.SendInfoToLog($"Счетчик: {current}", false);
    }
}

// Комплексная операция с несколькими переменными
IGlobalVariable var1 = project.GlobalVariables["Data", "Variable1"];
IGlobalVariable var2 = project.GlobalVariables["Data", "Variable2"];

if (var1 != null && var2 != null) {
    // Блокируем по одной переменной для избежания deadlock
    lock (var1) {
        lock (var2) {
            // Атомарная операция с двумя переменными
            int val1 = int.TryParse(var1.Value.ToString(), out int temp1) ? temp1 : 0;
            int val2 = int.TryParse(var2.Value.ToString(), out int temp2) ? temp2 : 0;
            
            val1++;
            val2 += val1;
            
            var1.Value = val1.ToString();
            var2.Value = val2.ToString();
        }
    }
}
```

## ⚖️ СРАВНЕНИЕ: ЛОКАЛЬНЫЕ vs ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ

| Характеристика | Локальные переменные | Глобальные переменные |
|----------------|----------------------|----------------------|
| **Область видимости** | Только текущий поток | Все потоки |
| **API доступа** | `project.Variables["name"]` | `project.GlobalVariables["namespace", "name"]` |
| **Создание** | Только в ProjectMaker | В коде или ProjectMaker |
| **Пространства имен** | Нет | Да |
| **Потокобезопасность** | Не требуется | Требуется lock |
| **Производительность** | Высокая | Ниже (из-за синхронизации) |
| **Использование** | Данные потока | Общие настройки, счетчики |

### Когда использовать глобальные переменные:
- Общие настройки для всех потоков
- Счетчики и статистика
- Флаги управления выполнением
- Кеширование данных между потоками
- Координация между потоками

### Когда использовать локальные переменные:
- Данные специфичные для потока
- Промежуточные результаты
- Состояние выполнения конкретного потока
- Временные значения


---
