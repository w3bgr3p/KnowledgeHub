[SPOILER= Essentials] 

[SPOILER= Cryptogarphy]

# AES

Статический класс для работы с AES-шифрованием и MD5-хешированием. Предоставляет методы для шифрования и расшифровки текста с использованием алгоритма AES, а также создания MD5-хешей.

## Методы

### EncryptAES
**Возвращает:** `string` - зашифрованная строка в HEX-формате

Шифрует текст с использованием алгоритма AES. По умолчанию ключ автоматически хешируется через MD5 для обеспечения правильной длины.

**Параметры:**
- `phrase` (string) - текст для шифрования
- `key` (string) - ключ для шифрования  
- `hashKey` (bool) - если true, ключ будет хеширован через MD5 (по умолчанию true)

**Пример использования:**
[CODE=csharp]
string text = "Мой секретный текст";
string password = "mypassword123";

//шифруем текст с автоматическим хешированием ключа
string encrypted = AES.EncryptAES(text, password);
project.SendInfoToLog($"Зашифрованный текст: {encrypted}");

//шифруем с готовым 32-символьным ключом без хеширования
string hexKey = "1234567890abcdef1234567890abcdef";
string encrypted2 = AES.EncryptAES(text, hexKey, false);
[/CODE]

### DecryptAES
**Возвращает:** `string` - расшифрованный текст

Расшифровывает текст, зашифрованный методом EncryptAES. Важно использовать тот же ключ и настройку hashKey, что и при шифровании.

**Параметры:**
- `hash` (string) - зашифрованная строка в HEX-формате
- `key` (string) - ключ для расшифровки
- `hashKey` (bool) - если true, ключ будет хеширован через MD5 (по умолчанию true)

**Пример использования:**
[CODE=csharp]
string encryptedText = "A1B2C3D4..."; //зашифрованный текст
string password = "mypassword123";

//расшифровываем с тем же ключом
string decrypted = AES.DecryptAES(encryptedText, password);
project.SendInfoToLog($"Расшифрованный текст: {decrypted}");

//если шифровали без хеширования ключа, то и расшифровывать нужно так же
string hexKey = "1234567890abcdef1234567890abcdef";
string decrypted2 = AES.DecryptAES(encryptedText, hexKey, false);
[/CODE]

### HashMD5
**Возвращает:** `string` - MD5-хеш в HEX-формате

Создает MD5-хеш от переданной строки. Полезно для создания фиксированных ключей для AES-шифрования или проверки целостности данных.

**Параметры:**
- `phrase` (string) - строка для хеширования

**Пример использования:**
[CODE=csharp]
string password = "mypassword123";

//получаем MD5-хеш пароля
string hash = AES.HashMD5(password);
project.SendInfoToLog($"MD5 хеш: {hash}");

//можно использовать как ключ для AES без дополнительного хеширования
string text = "секретные данные";
string encrypted = AES.EncryptAES(text, hash, false);
[/CODE]

---

# Bech32

Статический класс для работы с адресами в формате Bech32. Обеспечивает конвертацию между HEX-адресами и Bech32-адресами с префиксом "init", что используется в некоторых блокчейн-системах.

## Методы

### Bech32ToHex
**Возвращает:** `string` - HEX-адрес с префиксом "0x"

Конвертирует Bech32-адрес в HEX-формат. Проверяет корректность формата, префикса "init" и контрольной суммы.

**Параметры:**
- `bech32Address` (string) - адрес в формате Bech32 с префиксом "init"

**Пример использования:**
[CODE=csharp]
string bech32Addr = "init1qw508d6qejxtdg4y5r3zarvary0c5xw7k8a9xz2e";

try
{
    //конвертируем Bech32 в HEX
    string hexAddr = Bech32.Bech32ToHex(bech32Addr);
    project.SendInfoToLog($"HEX адрес: {hexAddr}");
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка конвертации: {ex.Message}");
}
[/CODE]

### HexToBech32
**Возвращает:** `string` - адрес в формате Bech32

Конвертирует HEX-адрес в формат Bech32 с указанным префиксом. По умолчанию используется префикс "init".

**Параметры:**
- `hexAddress` (string) - HEX-адрес (с префиксом "0x" или без)
- `prefix` (string) - префикс для Bech32-адреса (по умолчанию "init")

**Пример использования:**
[CODE=csharp]
string hexAddr = "0x1234567890abcdef1234567890abcdef12345678";

try
{
    //конвертируем HEX в Bech32 с префиксом по умолчанию
    string bech32Addr = Bech32.HexToBech32(hexAddr);
    project.SendInfoToLog($"Bech32 адрес: {bech32Addr}");
    
    //можно указать свой префикс
    string customAddr = Bech32.HexToBech32(hexAddr, "custom");
    project.SendInfoToLog($"С префиксом custom: {customAddr}");
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка конвертации: {ex.Message}");
}
[/CODE][/SPOILER]
[SPOILER= Init]

# Класс Init

## Описание
Этот класс представляет собой центральный инициализатор для проектов ZennoPoster. Он упрощает создание и настройку автоматизированных задач, работая с браузерами, базами данных, аккаунтами и социальными сетями. В целом работает как фабрика для инициализации всех компонентов проекта: получает список аккаунтов для работы, настраивает браузер, подключает социальные сети и кошельки, а затем запускает основной скрипт проекта.

## Конструкторы

### Init(IZennoPosterProjectModel project, Instance instance, bool log = false)
- **Описание**: Создает экземпляр инициализатора с указанным проектом, экземпляром браузера и настройкой логирования.
- **Параметры**: 
  - `project` - модель проекта ZennoPoster
  - `instance` - экземпляр браузера
  - `log` - включить/выключить подробное логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  var init = new Init(project, instance, true);
  project.SendInfoToLog("Инициализатор создан с логированием", false);
  [/CODE]

### Init(IZennoPosterProjectModel project, bool log = false)
- **Описание**: Создает экземпляр инициализатора только с проектом (без браузера).
- **Параметры**:
  - `project` - модель проекта ZennoPoster
  - `log` - включить/выключить подробное логирование
- **Пример**:
  [CODE=csharp]
  var init = new Init(project, false);
  project.SendInfoToLog("Инициализатор создан без браузера", false);
  [/CODE]

## Публичные методы

### InitProject(string author = "w3bgr3p", string[] customQueries = null, bool log = false)
- **Описание**: Выполняет полную инициализацию проекта - подготавливает базу данных, создает список аккаунтов для работы и настраивает все необходимые компоненты.
- **Параметры**:
  - `author` - имя автора скрипта для отображения в логах
  - `customQueries` - дополнительные SQL-запросы для выборки аккаунтов
  - `log` - включить подробное логирование
- **Пример**:
  [CODE=csharp]
  string[] queries = { "SELECT id FROM accounts WHERE status = 'active'" };
  init.InitProject("myusername", queries, true);
  project.SendInfoToLog("Проект инициализирован", false);
  [/CODE]

### PrepareProject()
- **Описание**: Подготавливает проект к выполнению - выбирает аккаунт для работы, проверяет фильтры (блокчейн, социальные сети) и настраивает экземпляр браузера.
- **Пример**:
  [CODE=csharp]
  try 
  {
      init.PrepareProject();
      project.SendInfoToLog("Проект готов к работе", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка подготовки: {ex.Message}", false);
  }
  [/CODE]

### PrepareInstance()
- **Описание**: Настраивает экземпляр браузера - запускает браузер, устанавливает прокси, загружает cookies и настраивает профиль.
- **Пример**:
  [CODE=csharp]
  try
  {
      init.PrepareInstance();
      project.SendInfoToLog("Браузер настроен и готов", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка настройки браузера: {ex.Message}", false);
  }
  [/CODE]

### LoadSocials(string requiredSocial)
- **Возвращает**: string - список загруженных социальных сетей или "noBrowser" если браузер не запущен
- **Описание**: Загружает и авторизует указанные социальные сети в браузере (Google, Twitter, Discord, GitHub).
- **Пример**:
  [CODE=csharp]
  string result = init.LoadSocials("Google,Twitter,Discord");
  if (result == "noBrowser")
  {
      project.SendWarningToLog("Браузер не запущен", false);
  }
  else
  {
      project.SendInfoToLog($"Загружены соц.сети: {result}", false);
  }
  [/CODE]

### LoadWallets(string walletsToUse)
- **Возвращает**: string - список загруженных кошельков или "noBrowser" если браузер не запущен
- **Описание**: Подключает и настраивает указанные криптовалютные кошельки (Backpack, Zerion, Keplr).
- **Пример**:
  [CODE=csharp]
  string result = init.LoadWallets("Backpack,Zerion");
  if (result == "noBrowser")
  {
      project.SendWarningToLog("Браузер не запущен для кошельков", false);
  }
  else
  {
      project.SendInfoToLog($"Кошельки подключены: {result}", false);
  }
  [/CODE]

### RunProject(List<string> additionalVars = null, bool add = true)
- **Возвращает**: bool - успешность выполнения проекта
- **Описание**: Запускает основной скрипт проекта с передачей всех необходимых переменных.
- **Параметры**:
  - `additionalVars` - дополнительные переменные для передачи в скрипт
  - `add` - добавить к стандартным переменным (true) или заменить их (false)
- **Пример**:
  [CODE=csharp]
  var extraVars = new List<string> { "customVariable", "mySpecialSetting" };
  bool success = init.RunProject(extraVars, true);
  if (success)
  {
      project.SendInfoToLog("Проект выполнен успешно", false);
  }
  else
  {
      project.SendErrorToLog("Проект завершился с ошибкой", false);
  }
  [/CODE][/SPOILER]
[SPOILER= ISAFU]

# SAFU (Simple Authentication and Functional Utilities)

SAFU - это утилитарный класс для шифрования и дешифрования данных с привязкой к аппаратным характеристикам системы. Класс предоставляет простые методы для безопасного хранения конфиденциальной информации в проектах ZennoPoster с использованием AES-шифрования и аппаратного ключа на основе серийного номера материнской платы.

## Публичные методы

### Initialize
**Возвращаемое значение:** `void`

**Описание:** Инициализирует систему SAFU, регистрируя функции шифрования в глобальном хранилище. Метод автоматически определяет доступные реализации и использует базовую версию SimpleSAFU по умолчанию.

**Параметры:**
- `project` (IZennoPosterProjectModel) - экземпляр проекта ZennoPoster

**Пример использования:**
[CODE=csharp]
//инициализация SAFU перед использованием
SAFU.Initialize(project);
[/CODE]

### Encode
**Возвращаемое значение:** `string`

**Описание:** Шифрует переданную строку с использованием PIN-кода из переменной проекта cfgPin. Если PIN-код не задан, возвращает исходную строку без изменений.

**Параметры:**
- `project` (IZennoPosterProjectModel) - экземпляр проекта ZennoPoster
- `toEncrypt` (string) - строка для шифрования
- `log` (bool, необязательный) - флаг логирования (по умолчанию false)

**Пример использования:**
[CODE=csharp]
//шифрование пароля
string password = "mySecretPassword";
string encryptedPassword = SAFU.Encode(project, password);
project.SendInfoToLog($"Пароль зашифрован: {encryptedPassword}");
[/CODE]

### Decode
**Возвращаемое значение:** `string`

**Описание:** Расшифровывает переданную строку с использованием PIN-кода из переменной проекта cfgPin. Если PIN-код не задан, возвращает исходную строку. В случае ошибки расшифровки отправляет предупреждение в лог.

**Параметры:**
- `project` (IZennoPosterProjectModel) - экземпляр проекта ZennoPoster
- `toDecrypt` (string) - строка для расшифровки
- `log` (bool, необязательный) - флаг логирования (по умолчанию false)

**Пример использования:**
[CODE=csharp]
//расшифровка пароля
string encryptedPassword = "зашифрованная_строка";
string originalPassword = SAFU.Decode(project, encryptedPassword);
project.SendInfoToLog($"Пароль расшифрован: {originalPassword}");
[/CODE]

### HWPass
**Возвращаемое значение:** `string`

**Описание:** Генерирует аппаратный пароль на основе серийного номера материнской платы и значения переменной acc0. Используется для создания уникального ключа, привязанного к конкретному компьютеру.

**Параметры:**
- `project` (IZennoPosterProjectModel) - экземпляр проекта ZennoPoster
- `log` (bool, необязательный) - флаг логирования (по умолчанию false)

**Пример использования:**
[CODE=csharp]
//получение аппаратного пароля
string hardwarePassword = SAFU.HWPass(project);
project.SendInfoToLog($"Аппаратный пароль: {hardwarePassword}");
[/CODE][/SPOILER]
[SPOILER= Logger]

# Logger

Класс для удобного логирования в ZennoPoster с расширенными возможностями форматирования, цветовой разметки и отправки отчетов в Telegram. Позволяет настраивать отображение дополнительной информации (аккаунт, время выполнения, использование памяти) и автоматически определять цвет сообщений по содержимому.

## Конструкторы

### Logger(IZennoPosterProjectModel project, bool log = false, string classEmoji = null)

Создает новый экземпляр логгера с базовыми настройками.

**Параметры:**
- `project` - проект ZennoPoster для отправки логов
- `log` - принудительно включить логирование (по умолчанию false)
- `classEmoji` - эмодзи для отображения перед каждым сообщением

**Пример:**
[CODE=csharp]
// создание базового логгера
var logger = new Logger(project);

// логгер с включенным отображением и эмодзи
var logger = new Logger(project, true, "🔧");
project.SendInfoToLog("Логгер создан");
[/CODE]

## Публичные методы

### Send(string toLog, [CallerMemberName] string callerName = "", bool show = false, bool thr0w = false, bool toZp = true, int cut = 0, bool wrap = true)

**Возвращает:** void

Отправляет сообщение в лог с автоматическим определением цвета и типа на основе содержимого.

**Параметры:**
- `toLog` - текст сообщения для логирования
- `callerName` - имя вызывающего метода (заполняется автоматически)
- `show` - принудительно показать сообщение независимо от настроек
- `thr0w` - выбросить исключение после записи в лог
- `toZp` - отправить в основной лог ZennoPoster
- `cut` - максимальное количество строк (0 = без ограничений)
- `wrap` - использовать обертку с заголовком

**Пример:**
[CODE=csharp]
var logger = new Logger(project, true);

// обычное сообщение
logger.Send("Операция выполнена успешно");

// предупреждение (автоматически оранжевым цветом)
logger.Send("!W Внимание: низкий баланс");

// ошибка с исключением
logger.Send("!E Критическая ошибка", thr0w: true);

// принудительное отображение
logger.Send("Важная информация", show: true);
[/CODE]

### SendToTelegram(string message = null)

**Возвращает:** void

Отправляет отчет о выполнении проекта в Telegram. Использует данные из базы данных проекта для получения токена бота и настроек чата.

**Параметры:**
- `message` - дополнительное сообщение для включения в отчет

**Пример:**
[CODE=csharp]
var logger = new Logger(project);

// отправка базового отчета
logger.SendToTelegram();

// отчет с дополнительным сообщением
logger.SendToTelegram("Обработано 150 элементов");

// настройка токена в БД (выполняется заранее)
project.DbSet("_api", "tg_logger", "apikey, extra", "YOUR_BOT_TOKEN|CHAT_ID/TOPIC_ID");
[/CODE]

**Настройка конфигурации:**
Логгер считывает настройки из переменной `cfgLog`, которая может содержать флаги через запятую:
- `acc` - показывать аккаунт
- `port` - показывать порт инстанса  
- `time` - показывать время выполнения
- `memory` - показывать использование памяти
- `caller` - показывать вызывающий метод
- `wrap` - использовать обертку заголовка
- `force` - принудительное отображение всех сообщений

[CODE=csharp]
// настройка отображения времени и памяти
project.Variables["cfgLog"].Value = "time,memory,caller";

var logger = new Logger(project, true);
logger.Send("Тест с расширенной информацией");
[/CODE][/SPOILER]
[SPOILER= Time]

# Time - Утилиты работы со временем

Статический класс для работы со временем в проектах ZennoPoster. Предоставляет методы для получения текущего времени, расчета времени выполнения, установки таймаутов и задержек.

## Статические методы

### Now
**Возвращает:** `string`

Получает текущее время в указанном формате.

**Параметры:**
- `format` (string, по умолчанию "unix") - формат времени: "unix", "iso", "short", "utcToId"

**Пример:**
[CODE=csharp]
//получить время в unix формате
string unixTime = Time.Now();
project.SendInfoToLog($"Unix время: {unixTime}");

//получить время в ISO формате
string isoTime = Time.Now("iso");
project.SendInfoToLog($"ISO время: {isoTime}");
[/CODE]

### Cd
**Возвращает:** `string`

Рассчитывает время до конца дня или добавляет указанное время к текущему моменту.

**Параметры:**
- `input` (object, по умолчанию null) - количество минут для добавления или строка времени, null для конца дня
- `o` (string, по умолчанию "iso") - формат вывода: "unix" или "iso"

**Пример:**
[CODE=csharp]
//время до конца дня
string endOfDay = Time.Cd();
project.SendInfoToLog($"До конца дня: {endOfDay}");

//добавить 30 минут к текущему времени
string futureTime = Time.Cd(30, "unix");
project.SendInfoToLog($"Через 30 минут: {futureTime}");
[/CODE]

## Методы-расширения для IZennoPosterProjectModel

### TimeElapsed
**Возвращает:** `int`

Вычисляет количество секунд, прошедших с момента начала сессии.

**Параметры:**
- `varName` (string, по умолчанию "varSessionId") - имя переменной с временем начала сессии

**Пример:**
[CODE=csharp]
//узнать сколько секунд прошло с начала сессии
int elapsed = project.TimeElapsed();
project.SendInfoToLog($"Прошло секунд: {elapsed}");
[/CODE]

### Age<T>
**Возвращает:** `T`

Получает возраст сессии в указанном типе данных.

**Пример:**
[CODE=csharp]
//получить возраст сессии как строку
string ageString = project.Age<string>();
project.SendInfoToLog($"Возраст сессии: {ageString}");

//получить возраст как TimeSpan
TimeSpan ageTimeSpan = project.Age<TimeSpan>();
project.SendInfoToLog($"Часов работы: {ageTimeSpan.Hours}");
[/CODE]

### TimeOut
Проверяет, не превышен ли глобальный таймаут выполнения. Выбрасывает исключение при превышении.

**Параметры:**
- `min` (int, по умолчанию 0) - лимит времени в минутах, если 0 - берется из переменной "timeOut"

**Пример:**
[CODE=csharp]
//проверить таймаут в 60 минут
project.TimeOut(60);

//проверить таймаут из переменной timeOut
project.TimeOut();
[/CODE]

### Deadline
Устанавливает дедлайн или проверяет его превышение.

**Параметры:**
- `sec` (int, по умолчанию 0) - лимит времени в секундах, если 0 - устанавливает начальное время

**Пример:**
[CODE=csharp]
//установить начальное время для дедлайна
project.Deadline();

//проверить дедлайн в 300 секунд
project.Deadline(300);
[/CODE]

### Sleep
Приостанавливает выполнение на указанное время.

**Параметры:**
- `min` (int, по умолчанию 0) - минимальное время задержки в секундах
- `max` (int, по умолчанию 0) - максимальное время задержки в секундах, если 0 - используется значение из "cfgDelay"

**Пример:**
[CODE=csharp]
//задержка из настройки cfgDelay
project.Sleep();

//случайная задержка от 5 до 10 секунд
project.Sleep(5, 10);
[/CODE]

### StartSession
Инициализирует новую сессию, устанавливая текущее время в переменную "varSessionId".

**Пример:**
[CODE=csharp]
//начать новую сессию
project.StartSession();
project.SendInfoToLog("Сессия запущена");
[/CODE][/SPOILER]
[/SPOILER]
