# F0rms

Класс F0rms предоставляет набор методов для создания интерактивных диалоговых окон с различными типами пользовательского ввода. Позволяет создавать формы для ввода текста, ключ-значение пар, выбора из списка и других типов данных.

## Конструкторы

### F0rms(IZennoPosterProjectModel project)

Создает новый экземпляр класса F0rms.

**Параметры:**
- `project` - экземпляр проекта для логирования

**Пример:**
```csharp
var forms = new F0rms(project);
```

## Методы

### InputBox

**Возвращает:** `string`

Отображает простое диалоговое окно для ввода многострочного текста.

**Параметры:**
- `message` - заголовок окна (по умолчанию "input data please")
- `width` - ширина окна в пикселях (по умолчанию 600)
- `height` - высота окна в пикселях (по умолчанию 600)

**Пример:**
```csharp
//получить текст от пользователя
string userText = F0rms.InputBox("Введите ваш текст:", 800, 400);
project.SendInfoToLog($"Введенный текст: {userText}");
```

### GetLinesByKey

**Возвращает:** `Dictionary<string, string>`

Создает диалоговое окно для ввода данных построчно с указанием имени ключевого столбца. Возвращает словарь где ключи - номера строк, а значения - SQL-подобные выражения.

**Параметры:**
- `keycolumn` - имя столбца для ключа (по умолчанию "input Column Name")
- `title` - заголовок окна (по умолчанию "Input data line per line")

**Пример:**
```csharp
//получить данные построчно
var dataDict = forms.GetLinesByKey("email", "Введите email адреса");
if (dataDict != null)
{
    foreach (var pair in dataDict)
    {
        project.SendInfoToLog($"Строка {pair.Key}: {pair.Value}");
    }
}
```

### GetLines

**Возвращает:** `List<string>`

Создает диалоговое окно для ввода данных построчно. Возвращает список SQL-подобных выражений.

**Параметры:**
- `keycolumn` - имя столбца для ключа (по умолчанию "input Column Name")
- `title` - заголовок окна (по умолчанию "Input data line per line")

**Пример:**
```csharp
//получить список строк
var lines = forms.GetLines("username", "Введите имена пользователей");
if (lines != null)
{
    foreach (string line in lines)
    {
        project.SendInfoToLog($"Строка: {line}");
    }
}
```

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
```csharp
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
```

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
```csharp
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
```

### GetKeyValueString

**Возвращает:** `string`

Создает диалоговое окно для ввода ключ-значение пар и возвращает их как строку в формате "key1='value1', key2='value2'".

**Параметры:**
- `quantity` - количество пар для ввода
- `keyPlaceholders` - список значений по умолчанию для ключей (может быть null)
- `valuePlaceholders` - список значений по умолчанию для значений (может быть null)
- `title` - заголовок окна (по умолчанию "Input Key-Value Pairs")

**Пример:**
```csharp
//получить пары как строку
var pairsString = forms.GetKeyValueString(2, 
    new List<string> { "name", "email" }, 
    new List<string> { "Иван", "ivan@example.com" },
    "Контактные данные");

project.SendInfoToLog($"Пары: {pairsString}");
//результат: "name='Иван', email='ivan@example.com'"
```

### GetSelectedItem

**Возвращает:** `string`

Создает диалоговое окно с выпадающим списком для выбора одного элемента из предоставленного списка.

**Параметры:**
- `items` - список элементов для выбора
- `title` - заголовок окна (по умолчанию "Select an Item")
- `labelText` - текст метки (по умолчанию "Select:")

**Пример:**
```csharp
//выбор из списка браузеров
var browsers = new List<string> { "Chrome", "Firefox", "Safari", "Edge" };
string selectedBrowser = forms.GetSelectedItem(browsers, "Выберите браузер", "Браузер:");

if (!string.IsNullOrEmpty(selectedBrowser))
{
    project.SendInfoToLog($"Выбран браузер: {selectedBrowser}");
}
```