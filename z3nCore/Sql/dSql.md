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
```csharp
var db = new dSql(@"C:\MyProject\database.sqlite", "");
project.SendInfoToLog("Подключение к SQLite установлено", false);
```

### dSql(string hostname, string port, string database, string user, string password)

**Описание:** Создает подключение к базе данных PostgreSQL с указанными параметрами подключения.

**Параметры:**
- `hostname` - адрес сервера базы данных
- `port` - порт подключения
- `database` - имя базы данных
- `user` - имя пользователя
- `password` - пароль пользователя

**Пример:**
```csharp
var db = new dSql("localhost", "5432", "mydb", "postgres", "password123");
project.SendInfoToLog("Подключение к PostgreSQL установлено", false);
```

### dSql(string connectionstring)

**Описание:** Создает подключение к базе данных PostgreSQL используя строку подключения.

**Параметры:**
- `connectionstring` - полная строка подключения к базе данных

**Пример:**
```csharp
string connStr = "Host=localhost;Port=5432;Database=mydb;Username=postgres;Password=pass;";
var db = new dSql(connStr);
project.SendInfoToLog("Подключение через строку подключения установлено", false);
```

### dSql(IDbConnection connection)

**Описание:** Создает экземпляр используя существующее подключение к базе данных.

**Параметры:**
- `connection` - готовое подключение к базе данных

**Пример:**
```csharp
var existingConnection = new SqliteConnection("Data Source=mydb.sqlite");
var db = new dSql(existingConnection);
project.SendInfoToLog("Подключение через готовый объект установлено", false);
```

## Публичные методы

### DbReadAsync(string sql, string columnSeparator = "|", string rawSeparator = "\r\n")

**Возвращает:** Task<string> - результат запроса в виде строки

**Описание:** Выполняет SQL-запрос на чтение данных и возвращает результат в виде текстовой строки с настраиваемыми разделителями.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `columnSeparator` - символ для разделения колонок (по умолчанию "|")
- `rawSeparator` - символ для разделения строк (по умолчанию "\r\n")

**Пример:**
```csharp
var db = new dSql(@"C:\data.sqlite", "");
string result = await db.DbReadAsync("SELECT name, email FROM users WHERE active = 1", "|", "\n");
project.SendInfoToLog($"Получены данные: {result}", false);
```

### DbRead(string sql, string separator = "|")

**Возвращает:** string - результат запроса в виде строки

**Описание:** Синхронная версия DbReadAsync. Выполняет SQL-запрос на чтение данных.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `separator` - разделитель для колонок (по умолчанию "|")

**Пример:**
```csharp
var db = new dSql("localhost", "5432", "testdb", "user", "pass");
string users = db.DbRead("SELECT id, username FROM accounts LIMIT 10");
project.SendInfoToLog($"Список пользователей: {users}", false);
```

### DbWriteAsync(string sql, params IDbDataParameter[] parameters)

**Возвращает:** Task<int> - количество затронутых строк

**Описание:** Выполняет SQL-запрос для записи, обновления или удаления данных с возможностью использования параметров.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `parameters` - параметры запроса для безопасной передачи значений

**Пример:**
```csharp
var db = new dSql(@"C:\accounts.sqlite", "");
var param = db.CreateParameter("@username", "john_doe");
int affected = await db.DbWriteAsync("UPDATE users SET last_login = NOW() WHERE username = @username", param);
project.SendInfoToLog($"Обновлено записей: {affected}", false);
```

### DbWrite(string sql, params IDbDataParameter[] parameters)

**Возвращает:** int - количество затронутых строк

**Описание:** Синхронная версия DbWriteAsync. Выполняет запросы на изменение данных.

**Параметры:**
- `sql` - SQL-запрос для выполнения
- `parameters` - параметры запроса

**Пример:**
```csharp
var db = new dSql("localhost", "5432", "mydb", "admin", "secret");
var nameParam = db.CreateParameter("@name", "Новый пользователь");
var emailParam = db.CreateParameter("@email", "user@example.com");
int result = db.DbWrite("INSERT INTO users (name, email) VALUES (@name, @email)", nameParam, emailParam);
project.SendInfoToLog($"Добавлено записей: {result}", false);
```

### CreateParameter(string name, object value)

**Возвращает:** IDbDataParameter - параметр для SQL-запроса

**Описание:** Создает параметр для безопасной передачи значений в SQL-запросы, автоматически подбирая правильный тип в зависимости от типа базы данных.

**Параметры:**
- `name` - имя параметра (например "@username")
- `value` - значение параметра

**Пример:**
```csharp
var db = new dSql(@"C:\test.sqlite", "");
var param1 = db.CreateParameter("@userId", 123);
var param2 = db.CreateParameter("@status", "active");
//использование параметров в запросе
int result = db.DbWrite("UPDATE accounts SET status = @status WHERE id = @userId", param1, param2);
```

### CreateParameters(params (string name, object value)[] parameters)

**Возвращает:** IDbDataParameter[] - массив параметров

**Описание:** Создает массив параметров для удобной передачи нескольких значений в SQL-запрос.

**Параметры:**
- `parameters` - массив кортежей с именами и значениями параметров

**Пример:**
```csharp
var db = new dSql("localhost", "5432", "shop", "user", "pass");
var parameters = db.CreateParameters(
    ("@productName", "Товар 1"),
    ("@price", 1250.50),
    ("@categoryId", 5)
);
int result = db.DbWrite("INSERT INTO products (name, price, category_id) VALUES (@productName, @price, @categoryId)", parameters);
```

### CopyTableAsync(string sourceTable, string destinationTable)

**Возвращает:** Task<int> - количество скопированных строк

**Описание:** Полностью копирует структуру и данные таблицы, включая первичные ключи и ограничения. Поддерживает копирование между схемами в PostgreSQL.

**Параметры:**
- `sourceTable` - имя исходной таблицы (может включать схему: "schema.table")
- `destinationTable` - имя новой таблицы

**Пример:**
```csharp
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
```

### MigrateAllTablesAsync(dSql sourceDb, dSql destinationDb)

**Возвращает:** Task<int> - общее количество мигрированных строк

**Описание:** Статический метод для миграции всех таблиц между разными типами баз данных. Переносит структуру таблиц, данные и пытается сохранить ограничения.

**Параметры:**
- `sourceDb` - исходная база данных
- `destinationDb` - целевая база данных

**Пример:**
```csharp
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
```

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
```csharp
var db = new dSql(@"C:\accounts.sqlite", "");
int updated = await db.Upd("status = 'completed', progress = 100", 15, "tasks", null, true);
project.SendInfoToLog($"Обновлено записей: {updated}", false);
```

### Get(string toGet, string id, string tableName = null, string where = null)

**Возвращает:** Task<string> - значение поля из базы данных

**Описание:** Получает значение указанного поля из таблицы по условию.

**Параметры:**
- `toGet` - имя поля для получения значения
- `id` - идентификатор записи
- `tableName` - имя таблицы
- `where` - дополнительное условие WHERE

**Пример:**
```csharp
var db = new dSql("localhost", "5432", "shop", "user", "pass");
string userName = await db.Get("username", "123", "users");
project.SendInfoToLog($"Имя пользователя: {userName}", false);

//с дополнительным условием
string email = await db.Get("email", "", "users", "username = 'admin' AND active = 1");
```

### AddRange(int range, string tableName = null)

**Возвращает:** Task - асинхронная операция

**Описание:** Добавляет записи с последовательными ID до указанного диапазона. Полезно для предварительного заполнения таблиц.

**Параметры:**
- `range` - максимальный ID для добавления
- `tableName` - имя таблицы

**Пример:**
```csharp
var db = new dSql(@"C:\accounts.sqlite", "");
await db.AddRange(1000, "accounts");
project.SendInfoToLog("Добавлены записи до ID 1000", false);
```