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
```csharp
// создание экземпляра Guild с логированием
var guild = new Guild(project, instance, true);

// создание экземпляра Guild без логирования  
var guild = new Guild(project, instance);
```

## Методы

### void ParseRoles(string tablename, bool append = true)

Парсит роли гильдий на текущей странице и сохраняет данные в базу данных.

**Параметры:**
- `tablename` - имя таблицы для сохранения данных
- `append` - добавлять к существующим данным или перезаписывать (по умолчанию true)

**Пример:**
```csharp
// парсинг ролей с добавлением к существующим данным
guild.ParseRoles("guild_data", true);

// парсинг ролей с перезаписью данных
guild.ParseRoles("guild_data", false);

// проверка результатов
string doneRoles = project.Variables["guildDone"].Value;
string undoneRoles = project.Variables["guildUndone"].Value;
project.SendInfoToLog($"Завершенные роли: {doneRoles}");
project.SendInfoToLog($"Незавершенные роли: {undoneRoles}");
```

### string Svg(string d)

**Возвращает:** string - тип социальной сети

Определяет тип социальной сети по SVG-коду иконки.

**Параметры:**
- `d` - SVG-код или HTML содержимое с иконкой

**Пример:**
```csharp
// определение типа по SVG
string svgCode = "M108,136a16,16,0,1,1-16-16A16,16,0,0,1,108,136Z...";
string socialType = guild.Svg(svgCode);
project.SendInfoToLog($"Тип социальной сети: {socialType}"); // выведет "discord"

// если тип не распознан
string unknownSvg = "some-unknown-svg-code";
string result = guild.Svg(unknownSvg);
project.SendInfoToLog($"Результат: {result}"); // выведет пустую строку
```

### string Svg(HtmlElement he)

**Возвращает:** string - тип социальной сети

Определяет тип социальной сети по HTML-элементу с иконкой.

**Параметры:**
- `he` - HTML-элемент с SVG-иконкой

**Пример:**
```csharp
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
```

### Dictionary<string, string> ParseConnections()

**Возвращает:** Dictionary<string, string> - словарь подключений

Парсит информацию о подключенных социальных сетях на странице.

**Пример:**
```csharp
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
```

### HtmlElement MainButton()

**Возвращает:** HtmlElement - главная кнопка на странице

Находит и возвращает основную кнопку действия на странице гильдии.

**Пример:**
```csharp
// получение главной кнопки
var mainButton = guild.MainButton();

if (mainButton != null)
{
    // проверка текста кнопки
    string buttonText = mainButton.InnerText;
    project.SendInfoToLog($"Текст кнопки: {buttonText}");
    
    // клик по кнопке
    mainButton.Click();
    project.SendInfoToLog("Кнопка нажата");
}
else
{
    project.SendErrorToLog("Главная кнопка не найдена");
}
```