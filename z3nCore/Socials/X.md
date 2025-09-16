# Класс X

Упрощённый менеджер аккаунтов Twitter/X для автоматизации входа и работы с профилем. Выполняет авторизацию через токены или данные логина, проверяет статус аккаунта, анализирует информацию профиля и настройки безопасности.

## Конструкторы

**X(IZennoPosterProjectModel project, Instance instance, bool log = false)**

Описание: Создает экземпляр менеджера X с указанным проектом, экземпляром браузера и настройкой логирования. Автоматически загружает данные аккаунта из базы данных.

Параметры:
- project - модель проекта ZennoPoster
- instance - экземпляр браузера  
- log - включить/выключить подробное логирование (по умолчанию false)

Пример:
```csharp
var x = new X(project, instance, true);
project.SendInfoToLog("Менеджер X создан с логированием", false);
```

## Публичные методы

**Load(bool log = false)**

Возвращает: string - статус аккаунта ("ok", "restricted", "suspended", "emailCapcha")

Описание: Выполняет полную загрузку и авторизацию в X. Проверяет состояние аккаунта, использует сохранённый токен или выполняет вход через логин/пароль при необходимости.

Параметры:
- log - включить подробное логирование процесса

Пример:
```csharp
string status = x.Load(true);
if (status == "ok")
{
    project.SendInfoToLog("Успешно авторизован в X", false);
}
else
{
    project.SendWarningToLog($"Проблема с аккаунтом: {status}", false);
}
```

**Auth()**

Описание: Выполняет пошаговую авторизацию в X для внешних приложений через OAuth. Обрабатывает все этапы входа: ввод логина, пароля, двухфакторную аутентификацию и подтверждение доступа.

Пример:
```csharp
try
{
    x.Auth();
    project.SendInfoToLog("OAuth авторизация завершена", false);
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка авторизации: {ex.Message}", false);
}
```

**State()**

Возвращает: string - текущее состояние страницы авторизации

Описание: Определяет текущий этап процесса авторизации. Возвращает состояния: "InputLogin", "InputPass", "InputOTP", "ClickLogin", "CheckUser", "AuthV1SignIn", "AuthV1Confirm" и другие.

Пример:
```csharp
string currentState = x.State();
project.SendInfoToLog($"Текущий этап авторизации: {currentState}", false);
```

**UpdXCreds(Dictionary<string, string> data)**

Описание: Обновляет данные аккаунта в базе данных. Принимает словарь с ключами: LOGIN, PASSWORD, EMAIL, EMAIL_PASSWORD, TOKEN, CODE2FA, RECOVERY_SEED.

Параметры:
- data - словарь с новыми данными аккаунта

Пример:
```csharp
var newData = new Dictionary<string, string>
{
    {"LOGIN", "newusername"},
    {"PASSWORD", "newpassword"},
    {"EMAIL", "new@email.com"}
};
x.UpdXCreds(newData);
project.SendInfoToLog("Данные аккаунта обновлены", false);
```

**ParseProfile()**

Описание: Анализирует профиль пользователя X и сохраняет данные в базу: дату создания, ID, имя пользователя, описание, местоположение, аватар, баннер, количество подписчиков и подписок.

Пример:
```csharp
x.ParseProfile();
project.SendInfoToLog("Данные профиля сохранены в базу", false);
```

**ParseSecurity()**

Описание: Парсит страницу настроек безопасности аккаунта и сохраняет информацию: email, телефон, дату создания, страну, язык, пол, дату рождения.

Пример:
```csharp
x.ParseSecurity();  
project.SendInfoToLog("Данные безопасности собраны", false);
```