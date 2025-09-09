# Работа с переменными проекта в ZennoPoster 


### ⚠️ Переменные должны быть созданы заранее
- **Все переменные должны быть созданы в ProjectMaker до выполнения кода**
- Попытка доступа к несуществующей переменной вызовет исключение
- API `IZennoPosterProjectModel` позволяет только читать и изменять существующие переменные, но НЕ создавать новые

### ⚠️ Тип данных переменных
- **Все переменные в ZennoPoster имеют тип `string`**
- При работе с числами, булевыми значениями и другими типами необходимо выполнять конвертацию

## 📋 ОСНОВЫ РАБОТЫ С ПЕРЕМЕННЫМИ

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
project.Variables["IsLoggedIn"].Value = "true";

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
    project.Variables["IsSuccess"].Value = "true";
} else {
    project.Variables["IsSuccess"].Value = "false";
}
```

## 🔀 РАБОТА С БУЛЕВЫМИ ЗНАЧЕНИЯМИ

### Установка и проверка булевых переменных
```csharp
// Установка булевого значения
project.Variables["IsProcessed"].Value = "true";
project.Variables["HasErrors"].Value = "false";

// Проверка булевого значения
string isLoggedInStr = project.Variables["IsLoggedIn"].Value;
bool isLoggedIn = isLoggedInStr == "true" || isLoggedInStr.ToLower() == "true";

if (isLoggedIn) {
    // Пользователь авторизован
    project.Variables["LastLoginTime"].Value = DateTime.Now.ToString();
} else {
    // Необходима авторизация
    project.Variables["LoginRequired"].Value = "true";
}

// Переключение булевого значения
string currentState = project.Variables["FeatureEnabled"].Value;
bool isEnabled = currentState == "true";
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
} else if (userType == "regular" && isVip == "true") {
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
    project.Variables["MaxAttemptsReached"].Value = "true";
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
// ❌ Плохо - много обращений к переменной
if (project.Variables["Status"].Value == "processing") {
    project.SendInfoToLog($"Статус: {project.Variables["Status"].Value}", false);
}

// ✅ Хорошо - сохраняем в локальную переменную
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

## 📝 ЧЕКЛИСТ ПЕРЕД ИСПОЛЬЗОВАНИЕМ ПЕРЕМЕННЫХ

1. ✅ **Переменная создана в ProjectMaker**
2. ✅ **Имя переменной указано корректно (с учетом регистра)**
3. ✅ **Предусмотрена обработка пустых значений**
4. ✅ **Выполнена корректная конвертация типов**
5. ✅ **Добавлена обработка исключений при необходимости**
6. ✅ **Переменные инициализированы значениями по умолчанию**

---

**Помните:** Переменные в ZennoPoster - это мощный инструмент для хранения и передачи данных между различными частями проекта. Правильное их использование обеспечит стабильную работу ваших автоматизированных сценариев.