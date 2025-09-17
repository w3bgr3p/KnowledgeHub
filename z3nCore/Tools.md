[SPOILER= Tools] 

[SPOILER= Accountant]

# Accountant

Класс для отображения таблиц балансов в удобном графическом интерфейсе. Создает окна с данными из базы данных или списков, поддерживает цветовое кодирование балансов и расчет итоговых сумм.

## Конструкторы

### `Accountant(IZennoPosterProjectModel project, bool log = false)`

Создает новый экземпляр класса Accountant.

**Параметры:**
- `project` *(IZennoPosterProjectModel)* - объект проекта ZennoPoster
- `log` *(bool)* - включить логирование операций (по умолчанию false)

**Пример:**
[CODE=csharp]
var accountant = new Accountant(project, true);
//включаем логирование операций

[/CODE]

## Методы

### `void ShowBalanceTable(string chains = null)`

Отображает таблицу балансов из базы данных в графическом окне с пагинацией и цветовым кодированием.

**Параметры:**
- `chains` *(string)* - список колонок через запятую для отображения (если null, показываются все колонки)

**Пример:**
[CODE=csharp]
//показать все колонки таблицы балансов

accountant.ShowBalanceTable();

//показать только определенные колонки

accountant.ShowBalanceTable("acc0,eth,btc,usdt");
[/CODE]

### `void ShowBalanceTableFromList(List<string> data)`

Отображает таблицу балансов из готового списка строк формата "счет:баланс".

**Параметры:**
- `data` *(List<string>)* - список строк в формате "счет:баланс"

**Пример:**
[CODE=csharp]
var balances = new List<string>
{
    "wallet1:0.12345",
    "wallet2:0.00123",
    "wallet3:1.5678"
};
//отобразить балансы из списка

accountant.ShowBalanceTableFromList(balances);
[/CODE]

## Особенности

- **Цветовое кодирование**: Балансы окрашиваются разными цветами в зависимости от размера (от синего для больших сумм до красного для маленьких)
- **Автоматические суммы**: В конце таблицы добавляется строка с итоговыми суммами по всем колонкам
- **Пагинация**: Большие таблицы разбиваются на страницы по 100 строк с кнопками навигации
- **Форматирование**: Все балансы отображаются с точностью до 7 знаков после запятой[/SPOILER]
[SPOILER= Converter]

# Класс Converer

## Описание
Статический класс для преобразования данных между различными форматами: hex (шестнадцатеричные строки), base64, bech32 (адреса блокчейна), bytes и обычный текст. Поддерживает двустороннее преобразование между любыми из этих форматов, что полезно при работе с криптографическими данными, адресами кошельков и кодированием информации.

## Публичные методы

**ConvertFormat(IZennoPosterProjectModel project, string toProcess, string input, string output, bool log = false)**

**Возвращает:** string - преобразованная строка или null в случае ошибки

**Описание:** Преобразует строку из одного формата в другой. Поддерживает форматы: hex, base64, bech32, bytes, text.

**Параметры:**
- project - модель проекта для логирования
- toProcess - строка для преобразования
- input - исходный формат (hex/base64/bech32/bytes/text)
- output - целевой формат (hex/base64/bech32/bytes/text)
- log - включить логирование операции (по умолчанию false)

**Пример:**
[CODE=csharp]
// преобразование hex в base64

string result = Converer.ConvertFormat(project, "48656c6c6f", "hex", "base64", true);
project.SendInfoToLog($"Результат: {result}");

// преобразование текста в hex

string hexResult = Converer.ConvertFormat(project, "Hello World", "text", "hex");
project.SendInfoToLog($"Текст в hex: {hexResult}");

// преобразование bech32 адреса в hex

string bech32ToHex = Converer.ConvertFormat(project, "init1qqqsyqcyq5rqwzqfpg9scrgwpugpzysnpswf", "bech32", "hex");
//результат преобразования

if (bech32ToHex != null)
{
    project.SendInfoToLog($"Bech32 в hex: {bech32ToHex}");
}
else
{
    project.SendErrorToLog("Ошибка преобразования bech32");
}
[/CODE][/SPOILER]
[SPOILER= F0rms]

# F0rms

Класс F0rms предоставляет набор методов для создания интерактивных диалоговых окон с различными типами пользовательского ввода. Позволяет создавать формы для ввода текста, ключ-значение пар, выбора из списка и других типов данных.

## Конструкторы

### F0rms(IZennoPosterProjectModel project)

Создает новый экземпляр класса F0rms.

**Параметры:**
- `project` - экземпляр проекта для логирования

**Пример:**
[CODE=csharp]
var forms = new F0rms(project);
[/CODE]

## Методы

### InputBox

**Возвращает:** `string`

Отображает простое диалоговое окно для ввода многострочного текста.

**Параметры:**
- `message` - заголовок окна (по умолчанию "input data please")
- `width` - ширина окна в пикселях (по умолчанию 600)
- `height` - высота окна в пикселях (по умолчанию 600)

**Пример:**
[CODE=csharp]
//получить текст от пользователя

string userText = F0rms.InputBox("Введите ваш текст:", 800, 400);
project.SendInfoToLog($"Введенный текст: {userText}");
[/CODE]

### GetLinesByKey

**Возвращает:** `Dictionary<string, string>`

Создает диалоговое окно для ввода данных построчно с указанием имени ключевого столбца. Возвращает словарь где ключи - номера строк, а значения - SQL-подобные выражения.

**Параметры:**
- `keycolumn` - имя столбца для ключа (по умолчанию "input Column Name")
- `title` - заголовок окна (по умолчанию "Input data line per line")

**Пример:**
[CODE=csharp]
//получить данные построчно

var dataDict = forms.GetLinesByKey("email", "Введите email адреса");
if (dataDict != null)
{
    foreach (var pair in dataDict)
    {
        project.SendInfoToLog($"Строка {pair.Key}: {pair.Value}");
    }
}
[/CODE]

### GetLines

**Возвращает:** `List<string>`

Создает диалоговое окно для ввода данных построчно. Возвращает список SQL-подобных выражений.

**Параметры:**
- `keycolumn` - имя столбца для ключа (по умолчанию "input Column Name")
- `title` - заголовок окна (по умолчанию "Input data line per line")

**Пример:**
[CODE=csharp]
//получить список строк

var lines = forms.GetLines("username", "Введите имена пользователей");
if (lines != null)
{
    foreach (string line in lines)
    {
        project.SendInfoToLog($"Строка: {line}");
    }
}
[/CODE]

### GetKeyValuePairs

**Возвращает:** `Dictionary<string, string>`

Создает диалоговое окно с указанным количеством полей для ввода ключ-значение пар.

**Параметры:**
- `quantity` - количество пар для ввода
- `keyPlaceholders` - список значений по умолчанию для ключей (может быть null)
- `valuePlaceholders` - список значений по умолчанию для значений (может быть null)
- `title` - заголовок окна (по умолчанию "Input Key-Value Pairs")
- `prepareUpd` - если true, подготавливает данные в SQL-формате (по умолчанию true)

**Пример:**
[CODE=csharp]
//получить 3 пары ключ-значение

var keyPlaceholders = new List<string> { "name", "age", "city" };
var valuePlaceholders = new List<string> { "Введите имя", "Введите возраст", "Введите город" };

var pairs = forms.GetKeyValuePairs(3, keyPlaceholders, valuePlaceholders, "Данные пользователя");
if (pairs != null)
{
    foreach (var pair in pairs)
    {
        project.SendInfoToLog($"Пара: {pair.Value}");
    }
}
[/CODE]

### GetKeyBoolPairs

**Возвращает:** `Dictionary<string, bool>`

Создает диалоговое окно для ввода пар ключ-булево значение с использованием чекбоксов.

**Параметры:**
- `quantity` - количество пар для ввода
- `keyPlaceholders` - список текстов для ключей (может быть null)
- `valuePlaceholders` - список текстов для чекбоксов (может быть null)
- `title` - заголовок окна (по умолчанию "Input Key-Bool Pairs")
- `prepareUpd` - если true, использует номера как ключи (по умолчанию true)

**Пример:**
[CODE=csharp]
//получить настройки с чекбоксами

var options = new List<string> { "option1", "option2", "option3" };
var labels = new List<string> { "Включить уведомления", "Автосохранение", "Темная тема" };

var boolPairs = forms.GetKeyBoolPairs(3, options, labels, "Настройки");
if (boolPairs != null)
{
    foreach (var pair in boolPairs)
    {
        project.SendInfoToLog($"Настройка {pair.Key}: {pair.Value}");
    }
}
[/CODE]

### GetKeyValueString

**Возвращает:** `string`

Создает диалоговое окно для ввода ключ-значение пар и возвращает их как строку в формате "key1='value1', key2='value2'".

**Параметры:**
- `quantity` - количество пар для ввода
- `keyPlaceholders` - список значений по умолчанию для ключей (может быть null)
- `valuePlaceholders` - список значений по умолчанию для значений (может быть null)
- `title` - заголовок окна (по умолчанию "Input Key-Value Pairs")

**Пример:**
[CODE=csharp]
//получить пары как строку

var pairsString = forms.GetKeyValueString(2, 
    new List<string> { "name", "email" }, 
    new List<string> { "Иван", "ivan@example.com" },
    "Контактные данные");

project.SendInfoToLog($"Пары: {pairsString}");
//результат: "name='Иван', email='ivan@example.com'"

[/CODE]

### GetSelectedItem

**Возвращает:** `string`

Создает диалоговое окно с выпадающим списком для выбора одного элемента из предоставленного списка.

**Параметры:**
- `items` - список элементов для выбора
- `title` - заголовок окна (по умолчанию "Select an Item")
- `labelText` - текст метки (по умолчанию "Select:")

**Пример:**
[CODE=csharp]
//выбор из списка браузеров

var browsers = new List<string> { "Chrome", "Firefox", "Safari", "Edge" };
string selectedBrowser = forms.GetSelectedItem(browsers, "Выберите браузер", "Браузер:");

if (!string.IsNullOrEmpty(selectedBrowser))
{
    project.SendInfoToLog($"Выбран браузер: {selectedBrowser}");
}
[/CODE][/SPOILER]
[SPOILER= Helper]

# Helper

Статический класс с утилитами для отладки и поиска документации по методам ZennoPoster API. Предоставляет удобный способ поиска информации о методах и их параметрах прямо из проекта.

## Методы

### Help
**Возвращает**: `void`

Ищет документацию по методам в XML-файлах ZennoPoster и отображает результаты в удобном окне просмотра.

**Параметры**:
- `project` (IZennoPosterProjectModel) - экземпляр проекта ZennoPoster
- `toSearch` (string, необязательный) - строка для поиска в названиях методов. Если не указан, появится диалог ввода

**Пример использования**:
[CODE=csharp]
//поиск всех методов содержащих слово "Click"

project.Help("Click");

//вызов с диалогом ввода поискового запроса  

project.Help();

//поиск методов работы с элементами

project.Help("Element");
[/CODE]

Метод автоматически просматривает XML-файлы документации ZennoPoster (ZennoLab.CommandCenter.xml, ZennoLab.InterfacesLibrary.xml, ZennoLab.Macros.xml, ZennoLab.Emulation.xml) и извлекает полную информацию о найденных методах, включая описание, параметры, возвращаемые значения, примеры использования и исключения. Результаты отображаются в удобном окне с возможностью копирования текста.[/SPOILER]
[SPOILER= OTP]

## OTP

Статический класс для получения одноразовых паролей (OTP) различными способами. Позволяет генерировать TOTP коды в автономном режиме и получать их из сервиса FirstMail.

---

### Методы

#### Offline

**Возвращаемое значение:** `string` - сгенерированный TOTP код

Генерирует одноразовый пароль TOTP в автономном режиме по секретному ключу. Если до истечения текущего кода осталось мало времени, метод дожидается генерации нового кода для повышения надежности.

**Параметры:**
- `keyString` *(string)* - секретный ключ в формате Base32
- `waitIfTimeLess` *(int, необязательный)* - если до истечения кода осталось меньше указанных секунд, метод дожидается нового кода (по умолчанию 5)

**Пример использования:**
[CODE=csharp]
string secret = "JBSWY3DPEHPK3PXP";
string otpCode = OTP.Offline(secret);
project.SendInfoToLog($"Сгенерированный OTP код: {otpCode}");

//дождаться нового кода если осталось меньше 10 секунд

string otpCode2 = OTP.Offline(secret, 10);
project.SendInfoToLog($"Новый OTP код: {otpCode2}");
[/CODE]

#### FirstMail

**Возвращаемое значение:** `string` - OTP код, полученный из сервиса FirstMail

Получает одноразовый пароль из сервиса FirstMail для указанного email адреса. Использует внутренний механизм для извлечения кода из почтовых сообщений.

**Параметры:**
- `project` *(IZennoPosterProjectModel)* - экземпляр проекта ZennoPoster
- `email` *(string)* - email адрес для получения OTP кода

**Пример использования:**
[CODE=csharp]
string email = "test@example.com";
string otpCode = OTP.FirstMail(project, email);
project.SendInfoToLog($"OTP код из FirstMail: {otpCode}");
[/CODE][/SPOILER]
[SPOILER= Snapper]

# GHsync

Класс для автоматической синхронизации локальных папок с репозиториями GitHub. Позволяет массово коммитить и пушить изменения в несколько проектов одновременно, с проверкой размеров файлов и автоматическим созданием .gitignore.

## Конструкторы

### GHsync(IZennoPosterProjectModel project)

Создает новый экземпляр класса для синхронизации с GitHub.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster для логирования

**Пример использования:**
[CODE=csharp]
var ghSync = new GHsync(project);
//создаем экземпляр для работы с GitHub

[/CODE]

## Публичные методы

### GetFileHash(string filePath) : string

Вычисляет MD5-хеш файла для проверки изменений.

**Возвращает:** MD5-хеш файла в виде строки или пустую строку при ошибке

**Параметры:**
- `filePath` - полный путь к файлу

**Пример использования:**
[CODE=csharp]
string hash = GHsync.GetFileHash(@"C:\myproject\file.txt");
project.SendInfoToLog($"Hash файла: {hash}");
//получаем хеш файла для проверки изменений

[/CODE]

### Main(string baseDir, string token, string username, string commmit = "ts")

Основной метод для синхронизации всех проектов из базовой директории с GitHub репозиториями.

**Параметры:**
- `baseDir` - базовая директория, содержащая папки проектов
- `token` - GitHub Personal Access Token (должен начинаться с "ghp_")
- `username` - имя пользователя GitHub
- `commmit` - текст коммита (по умолчанию "ts" - будет заменен на текущее время)

**Пример использования:**
[CODE=csharp]
var ghSync = new GHsync(project);
//создаем экземпляр синхронизации


ghSync.Main(@"C:\MyProjects", "ghp_your_token_here", "yourusername", "Обновление проектов");
//синхронизируем все проекты из папки

[/CODE]

**Особенности работы:**
- Читает конфигурацию из файла `.sync.txt` в базовой директории
- Пропускает папки помеченные как `false` в конфигурации
- Проверяет размеры файлов и репозиториев (максимум 100MB на файл, 1GB на репозиторий)
- Исключает бинарные файлы и медиа-контент
- Автоматически создает и обновляет `.gitignore`
- Выводит подробную статистику по завершению[/SPOILER]
[/SPOILER]
