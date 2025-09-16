# Google

Класс для автоматизации работы с аккаунтами Google. Загружает данные аккаунта из базы данных, выполняет авторизацию с обходом различных проверок безопасности и сохраняет куки для последующего использования.

## Конструкторы

### Google(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса для работы с Google аккаунтом.

**Параметры:**
- `project` - текущий проект ZennoPoster
- `instance` - экземпляр браузера для работы
- `log` - включить подробное логирование (по умолчанию false)

**Пример:**
```csharp
//создание экземпляра для работы с Google
Google google = new Google(project, instance, log: true);
```

## Публичные методы

### string Load(bool log = false, bool cookieBackup = true)

Выполняет полную авторизацию в аккаунте Google с автоматическим прохождением всех проверок.

**Возвращает:** Строка со статусом авторизации ("ok" при успехе)

**Параметры:**
- `log` - включить логирование процесса (по умолчанию false)
- `cookieBackup` - сохранить куки после успешной авторизации (по умолчанию true)

**Пример:**
```csharp
//авторизация в Google аккаунте
string result = google.Load(log: true, cookieBackup: true);
if (result == "ok")
{
    project.SendInfoToLog("Авторизация успешна");
}
else
{
    project.SendErrorToLog($"Ошибка авторизации: {result}");
}
```

### string State(bool log = false)

Определяет текущее состояние страницы авторизации Google.

**Возвращает:** Строка с текущим состоянием (например: "ok", "inputLogin", "inputPassword", "inputOtp")

**Параметры:**
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
//проверка текущего состояния
string currentState = google.State();
project.SendInfoToLog($"Текущее состояние: {currentState}");

//обработка разных состояний
switch (currentState)
{
    case "ok":
        project.SendInfoToLog("Уже авторизован");
        break;
    case "inputLogin":
        project.SendInfoToLog("Требуется ввод логина");
        break;
    case "CAPTCHA":
        project.SendWarningToLog("Обнаружена капча");
        break;
}
```

### string GAuth(bool log = false)

Выполняет авторизацию через систему Google Auth (для сторонних сервисов).

**Возвращает:** Строка с результатом авторизации ("SUCCESS" при успехе)

**Параметры:**
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
//авторизация через Google Auth
string authResult = google.GAuth(log: true);
if (authResult.Contains("SUCCESS"))
{
    project.SendInfoToLog("Google Auth успешно пройден");
}
else
{
    project.SendErrorToLog($"Ошибка Google Auth: {authResult}");
}
```

### void SaveCookies()

Сохраняет текущие куки Google в базу данных для последующего использования.

**Пример:**
```csharp
//сохранение кук после успешной авторизации
google.SaveCookies();
project.SendInfoToLog("Куки Google сохранены");
```

### void ParseSecurity()

Парсит информацию о настройках безопасности аккаунта Google и сохраняет в базу данных.

**Пример:**
```csharp
//получение информации о безопасности аккаунта
google.ParseSecurity();
project.SendInfoToLog("Информация о безопасности обновлена");
```