# Класс Db

Утилитарная библиотека для работы с базами данных PostgreSQL и SQLite. Предоставляет удобные методы для выполнения SQL-операций, управления таблицами и колонками, а также миграции данных между различными типами баз данных.

## Основные методы для работы с данными

### DbGet
**Возвращает:** `string` - результат SQL-запроса SELECT в виде строки

**Описание:** Получает данные из указанной таблицы по заданным параметрам

**Параметры:**
- `toGet` (string) - названия колонок через запятую для получения
- `tableName` (string, optional) - название таблицы (по умолчанию из переменной projectTable)
- `log` (bool, optional) - логировать ли запрос (по умолчанию false)
- `throwOnEx` (bool, optional) - выбрасывать ли исключения (по умолчанию false)
- `key` (string, optional) - название ключевого поля (по умолчанию "id")
- `acc` (string, optional) - значение для поиска (по умолчанию из переменной acc0)
- `where` (string, optional) - дополнительное условие WHERE

**Пример использования:**
```csharp
//получить статус текущей записи
string status = project.DbGet("status");

//получить email по конкретному ID
string email = project.DbGet("email", "users", key: "user_id", acc: "123");
```

### DbUpd
**Описание:** Обновляет данные в таблице

**Параметры:**
- `toUpd` (string) - строка обновления в формате "column1=value1, column2=value2"
- `tableName` (string, optional) - название таблицы
- `log` (bool, optional) - логировать ли запрос
- `throwOnEx` (bool, optional) - выбрасывать ли исключения
- `last` (bool, optional) - автоматически обновлять поле last текущим временем (по умолчанию true)
- `key` (string, optional) - название ключевого поля
- `acc` (object, optional) - значение для поиска
- `where` (string, optional) - дополнительное условие WHERE

**Пример использования:**
```csharp
//обновить статус текущей записи
project.DbUpd("status='completed'");

//обновить несколько полей без автоматического last
project.DbUpd("email='test@test.com', phone='+123456'", last: false);
```

### DbKey
**Возвращает:** `string` - криптографический ключ указанного типа

**Описание:** Получает криптографический ключ из таблицы _wallets в зависимости от типа блокчейна

**Параметры:**
- `chainType` (string, optional) - тип блокчейна: "evm" (Ethereum), "sol" (Solana), "seed" (мнемоническая фраза)

**Пример использования:**
```csharp
//получить приватный ключ для EVM сетей
string evmKey = project.DbKey("evm");

//получить приватный ключ для Solana
string solKey = project.DbKey("sol");
```

## Методы для работы с таблицами

### TblAdd
**Описание:** Создает новую таблицу с заданной структурой, если она не существует

**Параметры:**
- `tableStructure` (Dictionary<string, string>) - структура таблицы (имя колонки -> тип данных)
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
var structure = new Dictionary<string, string>
{
    {"id", "INTEGER PRIMARY KEY"},
    {"name", "TEXT DEFAULT ''"},
    {"email", "TEXT DEFAULT ''"}
};
project.TblAdd(structure, "users");
```

### TblExist
**Возвращает:** `bool` - true если таблица существует

**Описание:** Проверяет существование таблицы в базе данных

**Параметры:**
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
//проверить существование таблицы
if (project.TblExist("users"))
{
    project.SendInfoToLog("Таблица users найдена");
}
```

### TblColumns
**Возвращает:** `List<string>` - список названий колонок

**Описание:** Получает список всех колонок в таблице

**Параметры:**
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
//получить список колонок
var columns = project.TblColumns("users");
foreach(var column in columns)
{
    project.SendInfoToLog($"Колонка: {column}");
}
```

### TblForProject
**Возвращает:** `Dictionary<string, string>` - структура таблицы для проекта

**Описание:** Создает стандартную структуру таблицы для проекта на основе переменной cfgToDo и дополнительных колонок

**Параметры:**
- `projectColumns` (List<string> или string[], optional) - дополнительные колонки
- `defaultType` (string, optional) - тип данных по умолчанию

**Пример использования:**
```csharp
//создать структуру с базовыми колонками
var structure = project.TblForProject();

//добавить дополнительные колонки
var customColumns = new List<string> {"email", "phone", "address"};
var extendedStructure = project.TblForProject(customColumns);
```

### TblPrepareDefault
**Описание:** Полная подготовка таблицы проекта - создание, добавление недостающих колонок и заполнение записями

**Параметры:**
- `log` (bool, optional) - логировать ли операции

**Пример использования:**
```csharp
//подготовить таблицу проекта со всеми необходимыми полями
project.TblPrepareDefault(log: true);
```

## Методы для работы с колонками

### ClmnAdd
**Описание:** Добавляет колонку(и) в существующую таблицу

**Параметры:**
- `clmnName` (string) - название колонки ИЛИ
- `columns` (string[]) - массив названий колонок ИЛИ  
- `tableStructure` (Dictionary<string, string>) - структура колонок
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию
- `defaultValue` (string, optional) - значение по умолчанию

**Пример использования:**
```csharp
//добавить одну колонку
project.ClmnAdd("phone", "users");

//добавить несколько колонок
string[] newColumns = {"phone", "address", "city"};
project.ClmnAdd(newColumns, "users");
```

### ClmnExist
**Возвращает:** `bool` - true если колонка существует

**Описание:** Проверяет существование колонки в таблице

**Параметры:**
- `clmnName` (string) - название колонки
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
//проверить наличие колонки перед использованием
if (!project.ClmnExist("email", "users"))
{
    project.ClmnAdd("email", "users");
}
```

### ClmnDrop
**Описание:** Удаляет колонку(и) из таблицы

**Параметры:**
- `clmnName` (string) - название колонки ИЛИ
- `tableStructure` (Dictionary<string, string>) - структура колонок для удаления
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
//удалить одну колонку
project.ClmnDrop("old_column", "users");
```

## Методы для миграции и диапазонов

### MigrateTable
**Описание:** Копирует данные из одной таблицы в другую с переименованием ключевых полей

**Параметры:**
- `source` (string) - исходная таблица
- `dest` (string) - таблица назначения

**Пример использования:**
```csharp
//мигрировать данные между таблицами
project.MigrateTable("old_accounts", "new_accounts");
```

### AddRange
**Описание:** Добавляет записи с ID от текущего максимального до указанного значения

**Параметры:**
- `tblName` (string) - название таблицы
- `range` (int, optional) - конечный ID (по умолчанию из переменной rangeEnd)
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
//заполнить таблицу записями до ID 100
project.AddRange("accounts", 100);

//использовать значение из переменной rangeEnd
project.AddRange("accounts");
```

### DbSettings
**Описание:** Загружает настройки из таблицы _settings и устанавливает их как переменные проекта

**Параметры:**
- `set` (bool, optional) - устанавливать ли переменные (по умолчанию true)
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
```csharp
//загрузить все настройки из базы в переменные проекта
project.DbSettings(log: true);

//только прочитать настройки без установки переменных
project.DbSettings(set: false);
```

