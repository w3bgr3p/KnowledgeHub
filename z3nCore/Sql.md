[SPOILER= Sql] 

[SPOILER= Db]

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
[CODE=csharp]
//получить статус текущей записи

string status = project.DbGet("status");

//получить email по конкретному ID

string email = project.DbGet("email", "users", key: "user_id", acc: "123");
[/CODE]

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
[CODE=csharp]
//обновить статус текущей записи

project.DbUpd("status='completed'");

//обновить несколько полей без автоматического last

project.DbUpd("email='test@test.com', phone='+123456'", last: false);
[/CODE]

### DbKey
**Возвращает:** `string` - криптографический ключ указанного типа

**Описание:** Получает криптографический ключ из таблицы _wallets в зависимости от типа блокчейна

**Параметры:**
- `chainType` (string, optional) - тип блокчейна: "evm" (Ethereum), "sol" (Solana), "seed" (мнемоническая фраза)

**Пример использования:**
[CODE=csharp]
//получить приватный ключ для EVM сетей

string evmKey = project.DbKey("evm");

//получить приватный ключ для Solana

string solKey = project.DbKey("sol");
[/CODE]

## Методы для работы с таблицами

### TblAdd
**Описание:** Создает новую таблицу с заданной структурой, если она не существует

**Параметры:**
- `tableStructure` (Dictionary<string, string>) - структура таблицы (имя колонки -> тип данных)
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
var structure = new Dictionary<string, string>
{
    {"id", "INTEGER PRIMARY KEY"},
    {"name", "TEXT DEFAULT ''"},
    {"email", "TEXT DEFAULT ''"}
};
project.TblAdd(structure, "users");
[/CODE]

### TblExist
**Возвращает:** `bool` - true если таблица существует

**Описание:** Проверяет существование таблицы в базе данных

**Параметры:**
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
//проверить существование таблицы

if (project.TblExist("users"))
{
}
[/CODE]

### TblColumns
**Возвращает:** `List<string>` - список названий колонок

**Описание:** Получает список всех колонок в таблице

**Параметры:**
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
//получить список колонок

var columns = project.TblColumns("users");
foreach(var column in columns)
{
    project.SendInfoToLog($"Колонка: {column}");
}
[/CODE]

### TblForProject
**Возвращает:** `Dictionary<string, string>` - структура таблицы для проекта

**Описание:** Создает стандартную структуру таблицы для проекта на основе переменной cfgToDo и дополнительных колонок

**Параметры:**
- `projectColumns` (List<string> или string[], optional) - дополнительные колонки
- `defaultType` (string, optional) - тип данных по умолчанию

**Пример использования:**
[CODE=csharp]
//создать структуру с базовыми колонками

var structure = project.TblForProject();

//добавить дополнительные колонки

var customColumns = new List<string> {"email", "phone", "address"};
var extendedStructure = project.TblForProject(customColumns);
[/CODE]

### TblPrepareDefault
**Описание:** Полная подготовка таблицы проекта - создание, добавление недостающих колонок и заполнение записями

**Параметры:**
- `log` (bool, optional) - логировать ли операции

**Пример использования:**
[CODE=csharp]
//подготовить таблицу проекта со всеми необходимыми полями

project.TblPrepareDefault(log: true);
[/CODE]

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
[CODE=csharp]
//добавить одну колонку

project.ClmnAdd("phone", "users");

//добавить несколько колонок

string[] newColumns = {"phone", "address", "city"};
project.ClmnAdd(newColumns, "users");
[/CODE]

### ClmnExist
**Возвращает:** `bool` - true если колонка существует

**Описание:** Проверяет существование колонки в таблице

**Параметры:**
- `clmnName` (string) - название колонки
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
//проверить наличие колонки перед использованием

if (!project.ClmnExist("email", "users"))
{
    project.ClmnAdd("email", "users");
}
[/CODE]

### ClmnDrop
**Описание:** Удаляет колонку(и) из таблицы

**Параметры:**
- `clmnName` (string) - название колонки ИЛИ
- `tableStructure` (Dictionary<string, string>) - структура колонок для удаления
- `tblName` (string) - название таблицы
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
//удалить одну колонку

project.ClmnDrop("old_column", "users");
[/CODE]

## Методы для миграции и диапазонов

### MigrateTable
**Описание:** Копирует данные из одной таблицы в другую с переименованием ключевых полей

**Параметры:**
- `source` (string) - исходная таблица
- `dest` (string) - таблица назначения

**Пример использования:**
[CODE=csharp]
//мигрировать данные между таблицами

project.MigrateTable("old_accounts", "new_accounts");
[/CODE]

### AddRange
**Описание:** Добавляет записи с ID от текущего максимального до указанного значения

**Параметры:**
- `tblName` (string) - название таблицы
- `range` (int, optional) - конечный ID (по умолчанию из переменной rangeEnd)
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
//заполнить таблицу записями до ID 100

project.AddRange("accounts", 100);

//использовать значение из переменной rangeEnd

project.AddRange("accounts");
[/CODE]

### DbSettings
**Описание:** Загружает настройки из таблицы _settings и устанавливает их как переменные проекта

**Параметры:**
- `set` (bool, optional) - устанавливать ли переменные (по умолчанию true)
- `log` (bool, optional) - логировать ли операцию

**Пример использования:**
[CODE=csharp]
//загрузить все настройки из базы в переменные проекта

project.DbSettings(log: true);

//только прочитать настройки без установки переменных

project.DbSettings(set: false);
[/CODE]

[/SPOILER]
[SPOILER= dSql]

# Класс dSql (Dapper based)

## Описание

Универсальный класс для работы с базами данных SQLite и PostgreSQL. Предоставляет единый интерфейс для подключения, выполнения запросов, миграции данных и управления таблицами. Автоматически определяет тип базы данных и использует соответствующие методы для выполнения операций. Особенно полезен для проектов, где нужна поддержка нескольких типов баз данных одновременно.

## Конструкторы

### dSql(string dbPath, string dbPass)

**Описание:** Создает подключение к базе данных SQLite по указанному пути.

**Параметры:**
- `dbPath` - путь к файлу базы данных SQLite
- `dbPass` - пароль базы данных (в данной реализации не используется)

**Пример:**
[CODE=csharp]
var db = new dSql(@"C:\MyProject\database.sqlite", "");
[/CODE]

### dSql(string hostname, string port, string database, string user, string password)

**Описание:** Создает подключение к базе данных PostgreSQL с указанными параметрами подключения.

**Параметры:**
- `hostname` - адрес сервера базы данных
- `port` - порт подключения
- `database` - имя базы данных
- `user` - имя пользователя
- `password` - пароль пользователя

**Пример:**
[CODE=csharp]
var db = new dSql("localhost", "5432", "mydb", "postgres", "password123");
[/CODE]

### dSql(string connectionstring)

**Описание:** Создает подключение к базе данных PostgreSQL используя строку подключения.

**Параметры:**
- `connectionstring` - полная строка подключения к базе данных

**Пример:**
[CODE=csharp]
string connStr = "Host=localhost;Port=5432;Database=mydb;Username=postgres;Password=pass;";
var db = new dSql(connStr);
[/CODE]

### dSql(IDbConnection connection)

**Описание:** Создает экземпляр используя существующее подключение к базе данных.

**Параметры:**
- `connection` - готовое подключение к базе данных

**Пример:**
[CODE=csharp]
var existingConnection = new SqliteConnection("Data Source=mydb.sqlite");
var db = new dSql(existingConnection);
[/CODE]

## Публичные методы

### DbReadAsync(string sql, string columnSeparator = "|", string rawSeparator = "\r\n")

**Возвращает:** Task<string> - результат запроса в виде строки

**Описание:** Выполняет SQL-запрос на чтение данных и возвращает результат в виде текстовой строки с настраиваемыми разделителями.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `columnSeparator` - символ для разделения колонок (по умолчанию "|")
- `rawSeparator` - символ для разделения строк (по умолчанию "\r\n")

**Пример:**
[CODE=csharp]
var db = new dSql(@"C:\data.sqlite", "");
string result = await db.DbReadAsync("SELECT name, email FROM users WHERE active = 1", "|", "\n");
project.SendInfoToLog($"Получены данные: {result}", false);
[/CODE]

### DbRead(string sql, string separator = "|")

**Возвращает:** string - результат запроса в виде строки

**Описание:** Синхронная версия DbReadAsync. Выполняет SQL-запрос на чтение данных.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `separator` - разделитель для колонок (по умолчанию "|")

**Пример:**
[CODE=csharp]
var db = new dSql("localhost", "5432", "testdb", "user", "pass");
string users = db.DbRead("SELECT id, username FROM accounts LIMIT 10");
project.SendInfoToLog($"Список пользователей: {users}", false);
[/CODE]

### DbWriteAsync(string sql, params IDbDataParameter[] parameters)

**Возвращает:** Task<int> - количество затронутых строк

**Описание:** Выполняет SQL-запрос для записи, обновления или удаления данных с возможностью использования параметров.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `parameters` - параметры запроса для безопасной передачи значений

**Пример:**
[CODE=csharp]
var db = new dSql(@"C:\accounts.sqlite", "");
var param = db.CreateParameter("@username", "john_doe");
int affected = await db.DbWriteAsync("UPDATE users SET last_login = NOW() WHERE username = @username", param);
project.SendInfoToLog($"Обновлено записей: {affected}", false);
[/CODE]

### DbWrite(string sql, params IDbDataParameter[] parameters)

**Возвращает:** int - количество затронутых строк

**Описание:** Синхронная версия DbWriteAsync. Выполняет запросы на изменение данных.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `parameters` - параметры запроса

**Пример:**
[CODE=csharp]
var db = new dSql("localhost", "5432", "mydb", "admin", "secret");
var nameParam = db.CreateParameter("@name", "Новый пользователь");
var emailParam = db.CreateParameter("@email", "user@example.com");
int result = db.DbWrite("INSERT INTO users (name, email) VALUES (@name, @email)", nameParam, emailParam);
project.SendInfoToLog($"Добавлено записей: {result}", false);
[/CODE]

### CreateParameter(string name, object value)

**Возвращает:** IDbDataParameter - параметр для SQL-запроса

**Описание:** Создает параметр для безопасной передачи значений в SQL-запросы, автоматически подбирая правильный тип в зависимости от типа базы данных.

**Параметры:**
- `name` - имя параметра (например "@username")
- `value` - значение параметра

**Пример:**
[CODE=csharp]
var db = new dSql(@"C:\test.sqlite", "");
var param1 = db.CreateParameter("@userId", 123);
var param2 = db.CreateParameter("@status", "active");
//использование параметров в запросе

int result = db.DbWrite("UPDATE accounts SET status = @status WHERE id = @userId", param1, param2);
[/CODE]

### CreateParameters(params (string name, object value)[] parameters)

**Возвращает:** IDbDataParameter[] - массив параметров

**Описание:** Создает массив параметров для удобной передачи нескольких значений в SQL-запрос.

**Параметры:**
- `parameters` - массив кортежей с именами и значениями параметров

**Пример:**
[CODE=csharp]
var db = new dSql("localhost", "5432", "shop", "user", "pass");
var parameters = db.CreateParameters(
    ("@productName", "Товар 1"),
    ("@price", 1250.50),
    ("@categoryId", 5)
);
int result = db.DbWrite("INSERT INTO products (name, price, category_id) VALUES (@productName, @price, @categoryId)", parameters);
[/CODE]

### CopyTableAsync(string sourceTable, string destinationTable)

**Возвращает:** Task<int> - количество скопированных строк

**Описание:** Полностью копирует структуру и данные таблицы, включая первичные ключи и ограничения. Поддерживает копирование между схемами в PostgreSQL.

**Параметры:**
- `sourceTable` - имя исходной таблицы (может включать схему: "schema.table")
- `destinationTable` - имя новой таблицы

**Пример:**
[CODE=csharp]
var db = new dSql("localhost", "5432", "database", "user", "password");
try
{
    int copiedRows = await db.CopyTableAsync("public.users", "public.users_backup");
    project.SendInfoToLog($"Таблица скопирована, строк: {copiedRows}", false);
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка копирования: {ex.Message}", false);
}
[/CODE]

### MigrateAllTablesAsync(dSql sourceDb, dSql destinationDb)

**Возвращает:** Task<int> - общее количество мигрированных строк

**Описание:** Статический метод для миграции всех таблиц между разными типами баз данных. Переносит структуру таблиц, данные и пытается сохранить ограничения.

**Параметры:**
- `sourceDb` - исходная база данных
- `destinationDb` - целевая база данных

**Пример:**
[CODE=csharp]
var sqliteDb = new dSql(@"C:\old_data.sqlite", "");
var postgresDb = new dSql("localhost", "5432", "newdb", "admin", "pass");

try 
{
    int totalRows = await dSql.MigrateAllTablesAsync(sqliteDb, postgresDb);
    project.SendInfoToLog($"Миграция завершена, перенесено строк: {totalRows}", false);
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка миграции: {ex.Message}", false);
}
finally
{
    sqliteDb.Dispose();
    postgresDb.Dispose();
}
[/CODE]

### Upd(string toUpd, object id, string tableName = null, string where = null, bool last = false)

**Возвращает:** Task<int> - количество обновленных строк

**Описание:** Обновляет данные в таблице с возможностью автоматического добавления времени последнего изменения.

**Параметры:**
- `toUpd` - строка с полями для обновления в формате "field1 = value1, field2 = value2"
- `id` - идентификатор записи для обновления
- `tableName` - имя таблицы
- `where` - дополнительное условие WHERE (если не указано, используется id)
- `last` - добавить поле last с текущим временем

**Пример:**
[CODE=csharp]
var db = new dSql(@"C:\accounts.sqlite", "");
int updated = await db.Upd("status = 'completed', progress = 100", 15, "tasks", null, true);
project.SendInfoToLog($"Обновлено записей: {updated}", false);
[/CODE]

### Get(string toGet, string id, string tableName = null, string where = null)

**Возвращает:** Task<string> - значение поля из базы данных

**Описание:** Получает значение указанного поля из таблицы по условию.

**Параметры:**
- `toGet` - имя поля для получения значения
- `id` - идентификатор записи
- `tableName` - имя таблицы
- `where` - дополнительное условие WHERE

**Пример:**
[CODE=csharp]
var db = new dSql("localhost", "5432", "shop", "user", "pass");
string userName = await db.Get("username", "123", "users");
project.SendInfoToLog($"Имя пользователя: {userName}", false);

//с дополнительным условием

string email = await db.Get("email", "", "users", "username = 'admin' AND active = 1");
[/CODE]

### AddRange(int range, string tableName = null)

**Возвращает:** Task - асинхронная операция

**Описание:** Добавляет записи с последовательными ID до указанного диапазона. Полезно для предварительного заполнения таблиц.

**Параметры:**
- `range` - максимальный ID для добавления
- `tableName` - имя таблицы

**Пример:**
[CODE=csharp]
var db = new dSql(@"C:\accounts.sqlite", "");
await db.AddRange(1000, "accounts");
[/CODE][/SPOILER]
[/SPOILER]
