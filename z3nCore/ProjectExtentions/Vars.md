# Vars - Работа с переменными проекта

Утилиты для упрощения работы с переменными, математическими операциями и случайными значениями в проектах.

## Методы

### Var(string Var)

**Возвращает**: `string`

Получает значение переменной проекта по имени. Если переменная не существует, возвращает пустую строку.

**Параметры:**
- `Var` - имя переменной

**Пример использования:**
```csharp
string userLogin = project.Var("userLogin");
project.SendInfoToLog($"Логин пользователя: {userLogin}");
```

### Var(string var, object value)

**Возвращает**: `string`

Устанавливает значение переменной проекта. Возвращает пустую строку.

**Параметры:**
- `var` - имя переменной
- `value` - значение для установки

**Пример использования:**
```csharp
//установить значение переменной
project.Var("userLogin", "myusername123");
project.Var("userAge", 25);
```

### VarRnd(string Var)

**Возвращает**: `string`

Обрабатывает переменную как диапазон (формат "мин-макс") и возвращает случайное число из этого диапазона. Если переменная не содержит диапазон, возвращает её значение без изменений.

**Параметры:**
- `Var` - имя переменной

**Пример использования:**
```csharp
//если переменная delay содержит "5-10", получим случайное число от 5 до 10
string randomDelay = project.VarRnd("delay");
project.SendInfoToLog($"Случайная задержка: {randomDelay} секунд");
```

### VarCounter(string varName, int input)

**Возвращает**: `int`

Увеличивает числовое значение переменной на указанное число и возвращает новое значение.

**Параметры:**
- `varName` - имя переменной-счётчика
- `input` - число для добавления (может быть отрицательным)

**Пример использования:**
```csharp
//увеличить счётчик попыток на 1
int attempts = project.VarCounter("loginAttempts", 1);
project.SendInfoToLog($"Попытка входа номер: {attempts}");
```

### VarsMath(string varA, string operation, string varB, string varRslt = null)

**Возвращает**: `decimal`

Выполняет математические операции между двумя переменными. Поддерживает операции: +, -, *, /

**Параметры:**
- `varA` - имя первой переменной
- `operation` - математическая операция ("+", "-", "*", "/")
- `varB` - имя второй переменной  
- `varRslt` - имя переменной для сохранения результата (необязательно)

**Пример использования:**
```csharp
//сложить значения двух переменных и сохранить результат
decimal sum = project.VarsMath("price", "+", "tax", "totalPrice");
project.SendInfoToLog($"Общая сумма: {sum}");
```

---

# Constantas - Константы и настройки проекта

Методы для получения системных констант и настроек проекта.

## Методы

### ProjectName()

**Возвращает**: `string`

Получает имя текущего проекта из пути к файлу скрипта и сохраняет в переменную "projectName".

**Пример использования:**
```csharp
string name = project.ProjectName();
project.SendInfoToLog($"Имя проекта: {name}");
```

### ProjectTable()

**Возвращает**: `string`

Генерирует имя таблицы для проекта в формате "__ИмяПроекта" и сохраняет в переменную "projectTable".

**Пример использования:**
```csharp
string tableName = project.ProjectTable();
project.SendInfoToLog($"Имя таблицы: {tableName}");
```

### SessionId()

**Возвращает**: `string`

Генерирует уникальный идентификатор сессии и сохраняет в переменную "sessionId".

**Пример использования:**
```csharp
string sessionId = project.SessionId();
project.SendInfoToLog($"ID сессии: {sessionId}");
```

### CaptchaModule()

**Возвращает**: `string`

Получает настройки модуля капчи из глобальных переменных. Создает глобальную переменную в формате "CAPTCHA_ИмяПроекта".

**Пример использования:**
```csharp
try
{
    string captchaModule = project.CaptchaModule();
    project.SendInfoToLog($"Модуль капчи: {captchaModule}");
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка получения модуля капчи: {ex.Message}");
}
```

---

# GVars - Глобальные переменные

Утилиты для работы с глобальными переменными и управления потоками выполнения.

## Методы

### GVar(string var)

**Возвращает**: `string`

Получает значение глобальной переменной. Имя переменной автоматически формируется как "_ИмяПроекта_ИмяПеременной".

**Параметры:**
- `var` - имя переменной

**Пример использования:**
```csharp
string globalConfig = project.GVar("config");
project.SendInfoToLog($"Глобальная конфигурация: {globalConfig}");
```

### GVar(string var, object value)

**Возвращает**: `string`

Устанавливает значение глобальной переменной. Возвращает пустую строку.

**Параметры:**
- `var` - имя переменной
- `value` - значение для установки

**Пример использования:**
```csharp
//установить глобальную настройку
project.GVar("maxThreads", 5);
project.GVar("status", "running");
```

### GGetBusyList(bool log = false)

**Возвращает**: `List<string>`

Получает список занятых аккаунтов в формате "номер:значение". Проверяет глобальные переменные от acc1 до rangeEnd.

**Параметры:**
- `log` - выводить информацию в лог (по умолчанию false)

**Пример использования:**
```csharp
var busyAccounts = project.GGetBusyList(true);
foreach (var account in busyAccounts)
{
    project.SendInfoToLog($"Занятый аккаунт: {account}");
}
```

### GSetAcc(string input = null, bool force = false, bool log = false)

**Возвращает**: `bool`

Привязывает текущий поток к аккаунту через глобальные переменные. Возвращает true при успешной привязке.

**Параметры:**
- `input` - значение для привязки (по умолчанию имя проекта)
- `force` - принудительная привязка даже если аккаунт занят (по умолчанию false)
- `log` - выводить информацию в лог (по умолчанию false)

**Пример использования:**
```csharp
//попробовать привязать аккаунт
if (project.GSetAcc("MyBot_v1.0", false, true))
{
    project.SendInfoToLog("Аккаунт успешно привязан");
}
else
{
    project.SendWarningToLog("Не удалось привязать аккаунт");
}
```

### GClean(bool log = false)

**Возвращает**: `List<int>`

Очищает все глобальные переменные аккаунтов (от acc1 до rangeEnd). Возвращает список номеров очищенных аккаунтов.

**Параметры:**
- `log` - выводить информацию в лог (по умолчанию false)

**Пример использования:**
```csharp
//очистить все привязки аккаунтов
var cleanedAccounts = project.GClean(true);
project.SendInfoToLog($"Очищено {cleanedAccounts.Count} аккаунтов");
```