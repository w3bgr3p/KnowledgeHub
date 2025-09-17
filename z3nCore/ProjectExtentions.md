[SPOILER= ProjectExtentions] 

[SPOILER= Exceptions]

## TracedException

Специализированный класс исключения, который автоматически собирает информацию о месте возникновения ошибки и вызывающем методе для упрощения отладки.


### Конструкторы


#### TracedException(Exception, string)

Создает новое исключение с трассировкой на основе исходного исключения.


**Параметры:**

- `innerException` *(Exception)* - Исходное исключение, которое нужно обернуть

- `caller` *(string, необязательный)* - Имя вызывающего метода


[CODE=csharp]
try 
{
// какой-то код, который может выдать ошибку

var result = SomeMethod();
}
catch (Exception ex)
{
//создать трассируемое исключение

var tracedException = new TracedException(ex, "MyMethod");
throw tracedException;
}
[/CODE]

## ExceptionExtensions

Набор методов расширения для работы с исключениями, позволяющий легко логировать и выбрасывать ошибки с трассировкой.


### Методы


#### Throw(Exception, IZennoPosterProjectModel, string, bool)

**Возвращаемое значение:** `string` - Сообщение об ошибке


Обрабатывает исключение: логирует его и либо выбрасывает, либо возвращает сообщение об ошибке.


**Параметры:**

- `ex` *(Exception)* - Исключение для обработки

- `project` *(IZennoPosterProjectModel)* - Объект проекта для логирования

- `caller` *(string, необязательный)* - Имя вызывающего метода (заполняется автоматически)

- `throwEx` *(bool, по умолчанию true)* - Выбрасывать ли исключение после логирования


[CODE=csharp]
try 
{
// код, который может выдать ошибку

DoSomething();
}
catch (Exception ex)
{
//логировать и выбросить исключение

ex.Throw(project);
}

//или только залогировать без выброса исключения

try 
{
DoSomething();
}
catch (Exception ex)
{
//получить сообщение об ошибке без выброса исключения

string errorMessage = ex.Throw(project, throwEx: false);
project.SendInfoToLog("Ошибка обработана: " + errorMessage);

}
[/CODE]

#### Throw(Exception, string)

**Возвращаемое значение:** `Exception` - Трассируемое исключение


Создает и выбрасывает трассируемое исключение на основе исходного.


**Параметры:**

- `ex` *(Exception)* - Исключение для обработки

- `caller` *(string, необязательный)* - Имя вызывающего метода (заполняется автоматически)


[CODE=csharp]
try 
{
// код, который может выдать ошибку

DoSomething();
}
catch (Exception ex)
{
//создать и выбросить трассируемое исключение

ex.Throw();
}
[/CODE]

#### Throw(IZennoPosterProjectModel, string, string)

**Возвращаемое значение:** `Exception` - Трассируемое исключение


Создает и выбрасывает новое исключение с указанным сообщением, логируя информацию о последнем выполненном действии.


**Параметры:**

- `project` *(IZennoPosterProjectModel)* - Объект проекта

- `exMessage` *(string, необязательный)* - Сообщение об ошибке

- `caller` *(string, необязательный)* - Имя вызывающего метода (заполняется автоматически)


[CODE=csharp]
//проверить условие и выбросить ошибку при необходимости

if (someCondition)
{
project.Throw("Критическая ошибка: условие не выполнено");

}
[/CODE]

#### Throw(IZennoPosterProjectModel, Exception, string)

**Возвращаемое значение:** `Exception` - Трассируемое исключение


Обрабатывает существующее исключение, логирует его с информацией о последнем действии и выбрасывает трассируемую версию.


**Параметры:**

- `project` *(IZennoPosterProjectModel)* - Объект проекта

- `ex` *(Exception)* - Исключение для обработки

- `caller` *(string, необязательный)* - Имя вызывающего метода (заполняется автоматически)


[CODE=csharp]
try 
{
// код, который может выдать ошибку

DoSomething();
}
catch (Exception ex)
{
//обработать и выбросить с логированием контекста проекта

project.Throw(ex);
}
[/CODE][/SPOILER]
[SPOILER= FS]

# Класс FS


Утилитарный класс для работы с файловой системой, предоставляющий методы для управления файлами и директориями, копирования данных и работы с учетными данными.


## Конструкторы


### FS(IZennoPosterProjectModel project, bool log = false, string classEmoji = null)

Создает новый экземпляр класса FS.


**Параметры:**

- `project` - экземпляр проекта ZennoPoster

- `log` - включить логирование (по умолчанию false)

- `classEmoji` - эмодзи для класса (необязательный параметр)


**Пример использования:**

[CODE=csharp]
// создание экземпляра без логирования

var fs = new FS(project);

// создание экземпляра с включенным логированием

var fs = new FS(project, true);
[/CODE]

## Методы


### void RmRf(string path)

Рекурсивно удаляет директорию со всем содержимым.


**Параметры:**

- `path` - путь к удаляемой директории


**Пример использования:**

[CODE=csharp]
var fs = new FS(project);
//удалить директорию с файлами

fs.RmRf(@"C:\temp\folder");
[/CODE]

### void CopyDir(string sourceDir, string destDir)

Копирует директорию со всем содержимым в указанное место.


**Параметры:**

- `sourceDir` - путь к исходной директории

- `destDir` - путь к целевой директории


**Пример использования:**

[CODE=csharp]
var fs = new FS(project);
//скопировать папку с файлами

fs.CopyDir(@"C:\source", @"C:\destination");
[/CODE]

### string GetRandomFile(string directoryPath)

**Возвращает:** случайный файл из указанной директории или null, если файлы не найдены


Получает случайный файл из директории, включая подпапки.


**Параметры:**

- `directoryPath` - путь к директории для поиска файлов


**Пример использования:**

[CODE=csharp]
//получить случайный файл из папки

string randomFile = FS.GetRandomFile(@"C:\images");
if (randomFile != null)
{
project.SendInfoToLog("Выбран файл: " + randomFile);

}
[/CODE]

### string GetNewCreds(string dataType)

**Возвращает:** строку с учетными данными или null при ошибке


Получает новые учетные данные из файла fresh и перемещает их в файл used.


**Параметры:**

- `dataType` - тип данных для получения (имя файла без расширения)


**Пример использования:**

[CODE=csharp]
var fs = new FS(project, true);
//получить новый аккаунт из $"{_project.Path}.data\\fresh\\{dataType}.txt"

string account = fs.GetNewCreds("twitter"); 
if (account != null)
{
project.SendInfoToLog("Получен аккаунт: " + account);

}
else
{
project.SendWarningToLog("Аккаунты закончились");

}
[/CODE][/SPOILER]
[SPOILER= Reporter]

## Reporter

Класс для создания отчётов об ошибках и успешных операциях в проектах. Позволяет собирать информацию о последних ошибках, создавать отчёты в различных форматах и отправлять уведомления в Telegram.


### Конструкторы


#### Reporter(IZennoPosterProjectModel project, Instance instance, bool log = false, string classEmoji = null)
Создаёт новый экземпляр класса Reporter для работы с отчётами.


**Параметры:**

- `project` - объект проекта ZennoPoster

- `instance` - экземпляр браузера

- `log` - флаг включения логирования (по умолчанию false)

- `classEmoji` - эмодзи для класса (по умолчанию null)


[CODE=csharp]
//создание объекта Reporter

var reporter = new Reporter(project, instance);

//создание с дополнительными параметрами  

var reporter = new Reporter(project, instance, true, "📋");
[/CODE]

### Методы


#### ErrorReport(bool toTg = false, bool toDb = false, bool screensot = false)
**Возвращаемое значение:** `string` - текстовый отчёт об ошибке


Создаёт подробный отчёт о последней ошибке в проекте и выполняет дополнительные действия по настройке.


**Параметры:**

- `toTg` - отправить отчёт в Telegram (по умолчанию false)

- `toDb` - обновить статус в базе данных (по умолчанию false)  

- `screensot` - создать скриншот страницы (по умолчанию false)


[CODE=csharp]
//создание простого отчёта об ошибке

string errorReport = reporter.ErrorReport();
project.SendInfoToLog("Отчёт об ошибке: " + errorReport);


//создание отчёта с отправкой в Telegram и скриншотом

string fullReport = reporter.ErrorReport(toTg: true, screensot: true);

//создание отчёта с обновлением базы данных

string dbReport = reporter.ErrorReport(toDb: true);
[/CODE]

#### SuccessReport(bool log = false, bool ToTg = false)
**Возвращаемое значение:** `string` - текстовый отчёт об успешном выполнении


Создаёт отчёт об успешном завершении операции с информацией о времени выполнения.


**Параметры:**

- `log` - вывести отчёт в лог (по умолчанию false)

- `ToTg` - отправить отчёт в Telegram (по умолчанию false)


[CODE=csharp]
//создание простого отчёта об успехе

string successReport = reporter.SuccessReport();

//создание отчёта с выводом в лог

string loggedReport = reporter.SuccessReport(log: true);

//создание отчёта с отправкой в Telegram

string tgReport = reporter.SuccessReport(ToTg: true);
[/CODE][/SPOILER]
[SPOILER= Rnd]

## Класс Rnd


Статический класс для генерации различных случайных значений: строк, чисел, никнеймов и других данных, используемых в автоматизации.


---

## Методы


### Seed()

**Возвращаемое значение:** `string`  

**Описание:** Генерирует мнемоническую фразу из 12 слов на английском языке


[CODE=csharp]
string mnemonic = Rnd.Seed();
project.SendInfoToLog($"Generated mnemonic: {mnemonic}");
[/CODE]

---

### RndHexString(int length)

**Возвращаемое значение:** `string`  

**Описание:** Генерирует случайную hex-строку заданной длины с префиксом "0x"


**Параметры:**

- `length` (int) - длина генерируемой hex-строки (без учета префикса "0x")


[CODE=csharp]
string hexString = Rnd.RndHexString(8);
project.SendInfoToLog($"Generated hex: {hexString}"); // Например: 0x1a2b3c4d

[/CODE]

---

### RndString(int length)

**Возвращаемое значение:** `string`  

**Описание:** Генерирует случайную строку из букв и цифр заданной длины


**Параметры:**

- `length` (int) - длина генерируемой строки


[CODE=csharp]
string randomString = Rnd.RndString(10);
project.SendInfoToLog($"Random string: {randomString}"); // Например: Kj8Xm2Qr9Z

[/CODE]

---

### RndNickname()

**Возвращаемое значение:** `string`  

**Описание:** Генерирует случайный никнейм из прилагательного, существительного и суффикса (максимум 15 символов)


[CODE=csharp]
string nickname = Rnd.RndNickname();
project.SendInfoToLog($"Generated nickname: {nickname}"); // Например: MysticWolfX

[/CODE]

---

### RndInvite(this IZennoPosterProjectModel project, object limit = null, bool log = false)

**Возвращаемое значение:** `string`  

**Описание:** Получает случайный реферальный код из базы данных или использует существующий из переменной cfgRefCode


**Параметры:**

- `limit` (object, опционально) - максимальный ID записи для выборки

- `log` (bool) - включить логирование (по умолчанию false)


[CODE=csharp]
string refCode = project.RndInvite(100, true);
project.SendInfoToLog($"Using ref code: {refCode}");
[/CODE]

---

### RndPercent(decimal input, double percent, double maxPercent)

**Возвращаемое значение:** `double`  

**Описание:** Вычисляет процент от числа с случайным уменьшением до максимального процента


**Параметры:**

- `input` (decimal) - исходное число

- `percent` (double) - процент от числа (0-100)

- `maxPercent` (double) - максимальный процент случайного уменьшения (0-100)


[CODE=csharp]
double result = Rnd.RndPercent(1000m, 50, 10);
project.SendInfoToLog($"Result: {result}"); // 50% от 1000 с уменьшением до 10%

[/CODE]

---

### RndDecimal(this IZennoPosterProjectModel project, string Var)

**Возвращаемое значение:** `decimal`  

**Описание:** Получает случайное decimal число из переменной проекта (поддерживает диапазон через дефис)


**Параметры:**

- `Var` (string) - имя переменной проекта


[CODE=csharp]
// Переменная содержит "10.5-20.7"

decimal randomDecimal = project.RndDecimal("myRange");
project.SendInfoToLog($"Random decimal: {randomDecimal}");
[/CODE]

---

### RndInt(this IZennoPosterProjectModel project, string Var)

**Возвращаемое значение:** `int`  

**Описание:** Получает случайное целое число из переменной проекта (поддерживает диапазон через дефис)


**Параметры:**

- `Var` (string) - имя переменной проекта


[CODE=csharp]
// Переменная содержит "5-15"

int randomInt = project.RndInt("myIntRange");
project.SendInfoToLog($"Random integer: {randomInt}");
[/CODE]

---

### RndBool(this int truePercent)

**Возвращаемое значение:** `bool`  

**Описание:** Возвращает случайное булево значение с заданной вероятностью true в процентах


**Параметры:**

- `truePercent` (int) - вероятность получить true в процентах (0-100)


[CODE=csharp]
bool result = 75.RndBool(); // 75% вероятность получить true

project.SendInfoToLog($"Random bool: {result}");
[/CODE][/SPOILER]
[SPOILER= Utils]

# Utils

Статический класс с вспомогательными методами для упрощения работы с ZennoPoster.


## Методы


### L0g
Отправляет сообщение в лог проекта.


**Параметры:**

- `toLog` (string) - сообщение для записи в лог

- `callerName` (string, опционально) - имя вызывающего метода (заполняется автоматически)

- `show` (bool, по умолчанию true) - показывать ли сообщение в интерфейсе

- `thr0w` (bool, по умолчанию false) - выбрасывать ли исключение

- `toZp` (bool, по умолчанию true) - отправлять ли в ZennoPoster


**Пример использования:**

[CODE=csharp]
//отправить простое сообщение в лог

project.L0g("Операция выполнена успешно");


//отправить сообщение без отображения в интерфейсе

project.L0g("Отладочное сообщение", show: false);

[/CODE]

### Range
Обрабатывает диапазон аккаунтов и устанавливает соответствующие переменные проекта.


**Возвращает:** int - конечное значение диапазона


**Параметры:**

- `accRange` (string, опционально) - строка диапазона, если не указана, берется из переменной cfgAccRange

- `output` (string, опционально) - не используется в текущей реализации

- `log` (bool, по умолчанию false) - выводить ли информацию в лог


**Пример использования:**

[CODE=csharp]
//обработать диапазон из переменной проекта

int maxRange = project.Range();

//обработать конкретный диапазон

int maxRange = project.Range("1-10");

//обработать список аккаунтов

int maxRange = project.Range("1,5,8,12");
[/CODE]

### Clean
Очищает ресурсы браузера и профиля после завершения работы.


**Параметры:**

- `instance` (Instance) - экземпляр браузера для очистки


**Пример использования:**

[CODE=csharp]
//очистить ресурсы браузера

project.Clean(instance);
[/CODE]

### Finish
Завершает работу с аккаунтом, сохраняет данные и отправляет уведомления.


**Параметры:**

- `instance` (Instance) - экземпляр браузера


**Пример использования:**

[CODE=csharp]
//корректно завершить работу с аккаунтом

project.Finish(instance);
[/CODE]


### GetExtVer
Получает версию расширения из файла настроек Chrome.


**Возвращает:** string - версия расширения


**Параметры:**

- `securePrefsPath` (string) - путь к файлу Secure Preferences

- `extId` (string) - ID расширения


**Пример использования:**

[CODE=csharp]
//получить версию расширения

string version = Utils.GetExtVer(@"C:\Chrome\Default\Secure Preferences", "extension_id");
project.SendInfoToLog($"Версия расширения: {version}");

[/CODE]

### ObsoleteCode
Выводит предупреждение об использовании устаревшего кода.


**Параметры:**

- `newName` (string, по умолчанию "unknown") - рекомендуемый новый метод


**Пример использования:**

[CODE=csharp]
//предупредить об устаревшем коде

project.ObsoleteCode("NewMethod.DoSomething");
[/CODE]

### RunZp
Выполняет другой проект ZennoPoster с передачей переменных.


**Возвращает:** bool - результат выполнения проекта


**Параметры:**

- `vars` (List<string>, опционально) - список имен переменных для передачи


**Пример использования:**

[CODE=csharp]
//выполнить проект без передачи переменных

bool result = project.RunZp();

//выполнить проект с передачей переменных

var varsToPass = new List<string> { "var1", "var2", "login" };
bool result = project.RunZp(varsToPass);
[/CODE]
[/SPOILER]
[SPOILER= Vars]

# Vars - Работа с переменными проекта


Утилиты для упрощения работы с переменными, математическими операциями и случайными значениями в проектах.


## Методы


### Var(string Var)

**Возвращает**: `string`


Получает значение переменной проекта по имени. Если переменная не существует, возвращает пустую строку.


**Параметры:**

- `Var` - имя переменной


**Пример использования:**

[CODE=csharp]
string userLogin = project.Var("userLogin");
project.SendInfoToLog($"Логин пользователя: {userLogin}");

[/CODE]

### Var(string var, object value)

**Возвращает**: `string`


Устанавливает значение переменной проекта. Возвращает пустую строку.


**Параметры:**

- `var` - имя переменной

- `value` - значение для установки


**Пример использования:**

[CODE=csharp]
//установить значение переменной

project.Var("userLogin", "myusername123");
project.Var("userAge", 25);
[/CODE]

### VarRnd(string Var)

**Возвращает**: `string`


Обрабатывает переменную как диапазон (формат "мин-макс") и возвращает случайное число из этого диапазона. Если переменная не содержит диапазон, возвращает её значение без изменений.


**Параметры:**

- `Var` - имя переменной


**Пример использования:**

[CODE=csharp]
//если переменная delay содержит "5-10", получим случайное число от 5 до 10

string randomDelay = project.VarRnd("delay");
project.SendInfoToLog($"Случайная задержка: {randomDelay} секунд");

[/CODE]

### VarCounter(string varName, int input)

**Возвращает**: `int`


Увеличивает числовое значение переменной на указанное число и возвращает новое значение.


**Параметры:**

- `varName` - имя переменной-счётчика

- `input` - число для добавления (может быть отрицательным)


**Пример использования:**

[CODE=csharp]
//увеличить счётчик попыток на 1

int attempts = project.VarCounter("loginAttempts", 1);
project.SendInfoToLog($"Попытка входа номер: {attempts}");

[/CODE]

### VarsMath(string varA, string operation, string varB, string varRslt = null)

**Возвращает**: `decimal`


Выполняет математические операции между двумя переменными. Поддерживает операции: +, -, *, /


**Параметры:**

- `varA` - имя первой переменной

- `operation` - математическая операция ("+", "-", "*", "/")

- `varB` - имя второй переменной  

- `varRslt` - имя переменной для сохранения результата (необязательно)


**Пример использования:**

[CODE=csharp]
//сложить значения двух переменных и сохранить результат

decimal sum = project.VarsMath("price", "+", "tax", "totalPrice");
project.SendInfoToLog($"Общая сумма: {sum}");

[/CODE]

---

# Constantas - Константы и настройки проекта


Методы для получения системных констант и настроек проекта.


## Методы


### ProjectName()

**Возвращает**: `string`


Получает имя текущего проекта из пути к файлу скрипта и сохраняет в переменную "projectName".


**Пример использования:**

[CODE=csharp]
string name = project.ProjectName();
project.SendInfoToLog($"Имя проекта: {name}");

[/CODE]

### ProjectTable()

**Возвращает**: `string`


Генерирует имя таблицы для проекта в формате "__ИмяПроекта" и сохраняет в переменную "projectTable".


**Пример использования:**

[CODE=csharp]
string tableName = project.ProjectTable();
project.SendInfoToLog($"Имя таблицы: {tableName}");

[/CODE]

### SessionId()

**Возвращает**: `string`


Генерирует уникальный идентификатор сессии и сохраняет в переменную "sessionId".


**Пример использования:**

[CODE=csharp]
string sessionId = project.SessionId();
project.SendInfoToLog($"ID сессии: {sessionId}");

[/CODE]

### CaptchaModule()

**Возвращает**: `string`


Получает настройки модуля капчи из глобальных переменных. Создает глобальную переменную в формате "CAPTCHA_ИмяПроекта".


**Пример использования:**

[CODE=csharp]
try
{
string captchaModule = project.CaptchaModule();
project.SendInfoToLog($"Модуль капчи: {captchaModule}");

}
catch (Exception ex)
{
project.SendErrorToLog($"Ошибка получения модуля капчи: {ex.Message}");

}
[/CODE]

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

[CODE=csharp]
string globalConfig = project.GVar("config");
project.SendInfoToLog($"Глобальная конфигурация: {globalConfig}");

[/CODE]

### GVar(string var, object value)

**Возвращает**: `string`


Устанавливает значение глобальной переменной. Возвращает пустую строку.


**Параметры:**

- `var` - имя переменной

- `value` - значение для установки


**Пример использования:**

[CODE=csharp]
//установить глобальную настройку

project.GVar("maxThreads", 5);
project.GVar("status", "running");
[/CODE]

### GGetBusyList(bool log = false)

**Возвращает**: `List<string>`


Получает список занятых аккаунтов в формате "номер:значение". Проверяет глобальные переменные от acc1 до rangeEnd.


**Параметры:**

- `log` - выводить информацию в лог (по умолчанию false)


**Пример использования:**

[CODE=csharp]
var busyAccounts = project.GGetBusyList(true);
foreach (var account in busyAccounts)
{
project.SendInfoToLog($"Занятый аккаунт: {account}");

}
[/CODE]

### GSetAcc(string input = null, bool force = false, bool log = false)

**Возвращает**: `bool`


Привязывает текущий поток к аккаунту через глобальные переменные. Возвращает true при успешной привязке.


**Параметры:**

- `input` - значение для привязки (по умолчанию имя проекта)

- `force` - принудительная привязка даже если аккаунт занят (по умолчанию false)

- `log` - выводить информацию в лог (по умолчанию false)


**Пример использования:**

[CODE=csharp]
//попробовать привязать аккаунт

if (project.GSetAcc("MyBot_v1.0", false, true))
{
}
else
{
project.SendWarningToLog("Не удалось привязать аккаунт");

}
[/CODE]

### GClean(bool log = false)

**Возвращает**: `List<int>`


Очищает все глобальные переменные аккаунтов (от acc1 до rangeEnd). Возвращает список номеров очищенных аккаунтов.


**Параметры:**

- `log` - выводить информацию в лог (по умолчанию false)


**Пример использования:**

[CODE=csharp]
//очистить все привязки аккаунтов

var cleanedAccounts = project.GClean(true);
project.SendInfoToLog($"Очищено {cleanedAccounts.Count} аккаунтов");

[/CODE][/SPOILER]
[/SPOILER]
