[SPOILER= Socials]

[SPOILER= Discord]

# Discord

Класс для автоматизации работы с Discord в ZennoPoster. Предоставляет методы для загрузки учетных данных из базы данных, авторизации в Discord и получения списка серверов.


## Конструкторы


### Discord(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса Discord.


**Параметры:**

- `project` - экземпляр проекта ZennoPoster

- `instance` - экземпляр браузера

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
var discord = new Discord(project, instance, true);
//создать экземпляр Discord с включенным логированием

[/CODE]

## Методы


### CredsFromDb()

**Возвращает:** `string` - статус аккаунта Discord


Загружает данные Discord аккаунта из базы данных и сохраняет их в переменные проекта.


**Пример:**

[CODE=csharp]
string status = discord.CredsFromDb();
//загрузить данные аккаунта из БД

project.SendInfoToLog($"Статус аккаунта: {status}");

[/CODE]

### Log(string tolog = "", string callerName = "", bool log = false)

**Возвращает:** `void`


Отправляет сообщение в лог проекта с префиксом Discord.


**Параметры:**

- `tolog` - текст сообщения для лога

- `callerName` - имя вызывающего метода (заполняется автоматически)

- `log` - принудительно показать лог даже если отключен


**Пример:**

[CODE=csharp]
discord.Log("Начинаем авторизацию", log: true);

//отправить сообщение в лог с принудительным показом

[/CODE]

### DSload(bool log = false)

**Возвращает:** `string` - результат загрузки (статус или имя пользователя)


Выполняет полную процедуру авторизации в Discord: загружает страницу, пытается авторизоваться по токену, при неудаче выполняет вход по логину/паролю.


**Параметры:**

- `log` - включить дополнительное логирование


**Пример:**

[CODE=csharp]
string result = discord.DSload(true);
//авторизоваться в Discord

if (result == "ok" || result.Contains("@"))
{
}
[/CODE]

### DSservers()

**Возвращает:** `string` - список серверов через запятую


Получает список всех серверов Discord, к которым подключен текущий аккаунт, включая серверы в папках.


**Пример:**

[CODE=csharp]
string servers = discord.DSservers();
//получить список серверов

project.SendInfoToLog($"Серверы: {servers}");

[/CODE][/SPOILER]
[SPOILER= GitHub]

# GitHub

Класс для автоматизации работы с GitHub аккаунтами, включая авторизацию, двухфакторную аутентификацию и управление паролями.


## Конструкторы


### GitHub(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса GitHub с автоматической загрузкой учетных данных.


**Параметры:**

- `project` - экземпляр проекта ZennoPoster

- `instance` - экземпляр браузера

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
var github = new GitHub(project, instance, true);
[/CODE]

## Методы


### void LoadCreds()

Загружает учетные данные GitHub из базы данных и сохраняет их в переменные проекта.


**Пример:**

[CODE=csharp]
github.LoadCreds();
[/CODE]

### void InputCreds()

Вводит учетные данные на странице входа GitHub и обрабатывает двухфакторную аутентификацию.


**Пример:**

[CODE=csharp]
github.InputCreds();
[/CODE]

### void Go()

Переходит на страницу входа GitHub и принимает cookies если необходимо.


**Пример:**

[CODE=csharp]
github.Go();
[/CODE]

### void Verify2fa()

Выполняет верификацию двухфакторной аутентификации с использованием OTP кода.


**Пример:**

[CODE=csharp]
github.Verify2fa();
[/CODE]

### string Load()

**Возвращает:** имя текущего пользователя GitHub


Выполняет полный процесс входа в аккаунт GitHub с проверкой корректности авторизации и сохранением cookies.


**Пример:**

[CODE=csharp]
string currentUser = github.Load();
project.SendInfoToLog($"Вошли как: {currentUser}");

[/CODE]

### string Current()

**Возвращает:** имя текущего авторизованного пользователя


Получает имя текущего пользователя из навигационного меню GitHub.


**Пример:**

[CODE=csharp]
string username = github.Current();
project.SendInfoToLog($"Текущий пользователь: {username}");

[/CODE]

### void ChangePass(string password = null)

Изменяет пароль аккаунта GitHub через функцию восстановления пароля с использованием email.


**Параметры:**

- `password` - новый пароль (не используется в текущей реализации)


**Пример:**

[CODE=csharp]
github.ChangePass();
[/CODE]

### void SaveCookies()

Сохраняет текущие cookies браузера в базу данных для последующего использования.


**Пример:**

[CODE=csharp]
github.SaveCookies();
[/CODE][/SPOILER]
[SPOILER= Google]

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

[CODE=csharp]
//создание экземпляра для работы с Google

Google google = new Google(project, instance, log: true);
[/CODE]

## Публичные методы


### string Load(bool log = false, bool cookieBackup = true)

Выполняет полную авторизацию в аккаунте Google с автоматическим прохождением всех проверок.


**Возвращает:** Строка со статусом авторизации ("ok" при успехе)


**Параметры:**

- `log` - включить логирование процесса (по умолчанию false)

- `cookieBackup` - сохранить куки после успешной авторизации (по умолчанию true)


**Пример:**

[CODE=csharp]
//авторизация в Google аккаунте

string result = google.Load(log: true, cookieBackup: true);
if (result == "ok")
{
}
else
{
project.SendErrorToLog($"Ошибка авторизации: {result}");

}
[/CODE]

### string State(bool log = false)

Определяет текущее состояние страницы авторизации Google.


**Возвращает:** Строка с текущим состоянием (например: "ok", "inputLogin", "inputPassword", "inputOtp")


**Параметры:**

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
//проверка текущего состояния

string currentState = google.State();
project.SendInfoToLog($"Текущее состояние: {currentState}");


//обработка разных состояний

switch (currentState)
{
case "ok":
break;
case "inputLogin":
break;
case "CAPTCHA":
project.SendWarningToLog("Обнаружена капча");

break;
}
[/CODE]

### string GAuth(bool log = false)

Выполняет авторизацию через систему Google Auth (для сторонних сервисов).


**Возвращает:** Строка с результатом авторизации ("SUCCESS" при успехе)


**Параметры:**

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
//авторизация через Google Auth

string authResult = google.GAuth(log: true);
if (authResult.Contains("SUCCESS"))
{
}
else
{
project.SendErrorToLog($"Ошибка Google Auth: {authResult}");

}
[/CODE]

### void SaveCookies()

Сохраняет текущие куки Google в базу данных для последующего использования.


**Пример:**

[CODE=csharp]
//сохранение кук после успешной авторизации

google.SaveCookies();
[/CODE]

### void ParseSecurity()

Парсит информацию о настройках безопасности аккаунта Google и сохраняет в базу данных.


**Пример:**

[CODE=csharp]
//получение информации о безопасности аккаунта

google.ParseSecurity();
[/CODE][/SPOILER]
[SPOILER= Guild]

# Guild

Класс для работы с Guild.xyz . Предоставляет возможности для парсинга ролей гильдий, анализа подключений и управления элементами интерфейса.


## Конструктор


### Guild(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса Guild для работы с гильдиями.


**Параметры:**

- `project` - проект ZennoPoster

- `instance` - экземпляр браузера

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
// создание экземпляра Guild с логированием

var guild = new Guild(project, instance, true);

// создание экземпляра Guild без логирования

var guild = new Guild(project, instance);
[/CODE]

## Методы


### void ParseRoles(string tablename, bool append = true)

Парсит роли гильдий на текущей странице и сохраняет данные в базу данных.


**Параметры:**

- `tablename` - имя таблицы для сохранения данных

- `append` - добавлять к существующим данным или перезаписывать (по умолчанию true)


**Пример:**

[CODE=csharp]
// парсинг ролей с добавлением к существующим данным

guild.ParseRoles("guild_data", true);

// парсинг ролей с перезаписью данных

guild.ParseRoles("guild_data", false);

// проверка результатов

string doneRoles = project.Variables["guildDone"].Value;
string undoneRoles = project.Variables["guildUndone"].Value;
project.SendInfoToLog($"Завершенные роли: {doneRoles}");

project.SendInfoToLog($"Незавершенные роли: {undoneRoles}");

[/CODE]

### string Svg(string d)

**Возвращает:** string - тип социальной сети


Определяет тип социальной сети по SVG-коду иконки.


**Параметры:**

- `d` - SVG-код или HTML содержимое с иконкой


**Пример:**

[CODE=csharp]
// определение типа по SVG

string svgCode = "M108,136a16,16,0,1,1-16-16A16,16,0,0,1,108,136Z...";
string socialType = guild.Svg(svgCode);
project.SendInfoToLog($"Тип социальной сети: {socialType}"); // выведет "discord"


// если тип не распознан

string unknownSvg = "some-unknown-svg-code";
string result = guild.Svg(unknownSvg);
project.SendInfoToLog($"Результат: {result}"); // выведет пустую строку

[/CODE]

### string Svg(HtmlElement he)

**Возвращает:** string - тип социальной сети


Определяет тип социальной сети по HTML-элементу с иконкой.


**Параметры:**

- `he` - HTML-элемент с SVG-иконкой


**Пример:**

[CODE=csharp]
// получение элемента с иконкой

var iconElement = instance.ActiveTab.FindElementByAttribute("div", "class", "icon-container", "text", 0);

// определение типа социальной сети

string socialType = guild.Svg(iconElement);
if (!string.IsNullOrEmpty(socialType))
{
project.SendInfoToLog($"Найдена иконка: {socialType}");

}
else
{
project.SendWarningToLog("Тип социальной сети не определен");

}
[/CODE]

### Dictionary<string, string> ParseConnections()

**Возвращает:** Dictionary<string, string> - словарь подключений


Парсит информацию о подключенных социальных сетях на странице.


**Пример:**

[CODE=csharp]
// парсинг подключений

var connections = guild.ParseConnections();

// обработка результатов

foreach (var connection in connections)
{
string platform = connection.Key;     // тип платформы (discord, twitter и т.д.)

string status = connection.Value;     // статус ("none" если не подключено, или текст со статусом)


if (status == "none")
{
project.SendWarningToLog($"{platform} не подключен");

}
else
{
project.SendInfoToLog($"{platform}: {status}");
}
}

// проверка конкретной платформы

if (connections.ContainsKey("discord"))
{
project.SendInfoToLog($"Discord статус: {connections["discord"]}");

}
[/CODE]

### HtmlElement MainButton()

**Возвращает:** HtmlElement - главная кнопка на странице


Находит и возвращает основную кнопку действия на странице гильдии.


**Пример:**

[CODE=csharp]
// получение главной кнопки

var mainButton = guild.MainButton();

if (mainButton != null)
{
// проверка текста кнопки

string buttonText = mainButton.InnerText;
project.SendInfoToLog($"Текст кнопки: {buttonText}");


// клик по кнопке

mainButton.Click();
}
else
{
project.SendErrorToLog("Главная кнопка не найдена");

}
[/CODE][/SPOILER]
[SPOILER= X]

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

[CODE=csharp]
var x = new X(project, instance, true);
[/CODE]

## Публичные методы


**Load(bool log = false)**

Возвращает: string - статус аккаунта ("ok", "restricted", "suspended", "emailCapcha")


Описание: Выполняет полную загрузку и авторизацию в X. Проверяет состояние аккаунта, использует сохранённый токен или выполняет вход через логин/пароль при необходимости.


Параметры:

- log - включить подробное логирование процесса


Пример:

[CODE=csharp]
string status = x.Load(true);
if (status == "ok")
{
}
else
{
project.SendWarningToLog($"Проблема с аккаунтом: {status}", false);

}
[/CODE]

**Auth()**

Описание: Выполняет пошаговую авторизацию в X для внешних приложений через OAuth. Обрабатывает все этапы входа: ввод логина, пароля, двухфакторную аутентификацию и подтверждение доступа.


Пример:

[CODE=csharp]
try
{
x.Auth();
}
catch (Exception ex)
{
project.SendErrorToLog($"Ошибка авторизации: {ex.Message}", false);

}
[/CODE]

**State()**

Возвращает: string - текущее состояние страницы авторизации


Описание: Определяет текущий этап процесса авторизации. Возвращает состояния: "InputLogin", "InputPass", "InputOTP", "ClickLogin", "CheckUser", "AuthV1SignIn", "AuthV1Confirm" и другие.


Пример:

[CODE=csharp]
string currentState = x.State();
project.SendInfoToLog($"Текущий этап авторизации: {currentState}", false);

[/CODE]

**UpdXCreds(Dictionary<string, string> data)**

Описание: Обновляет данные аккаунта в базе данных. Принимает словарь с ключами: LOGIN, PASSWORD, EMAIL, EMAIL_PASSWORD, TOKEN, CODE2FA, RECOVERY_SEED.


Параметры:

- data - словарь с новыми данными аккаунта


Пример:

[CODE=csharp]
var newData = new Dictionary<string, string>
{
{"LOGIN", "newusername"},
{"PASSWORD", "newpassword"},
{"EMAIL", "new@email.com"}
};
x.UpdXCreds(newData);
[/CODE]

**ParseProfile()**

Описание: Анализирует профиль пользователя X и сохраняет данные в базу: дату создания, ID, имя пользователя, описание, местоположение, аватар, баннер, количество подписчиков и подписок.


Пример:

[CODE=csharp]
x.ParseProfile();
[/CODE]

**ParseSecurity()**

Описание: Парсит страницу настроек безопасности аккаунта и сохраняет информацию: email, телефон, дату создания, страну, язык, пол, дату рождения.


Пример:

[CODE=csharp]
x.ParseSecurity();
[/CODE][/SPOILER]
[/SPOILER]
