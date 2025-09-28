## ЧАСТЬ 1: СТАНДАРТНЫЕ МЕТОДЫ ZENNOPOSTER

### 1.1 Встроенные методы работы с БД

ZennoPoster предоставляет набор стандартных методов для работы с базами данных через класс `ZennoPoster.Db`:

```csharp
// Основные методы класса ZennoPoster.Db:
// 1. ExecuteNonQuery - выполнение запросов без возврата данных (INSERT, UPDATE, DELETE)
// 2. ExecuteScalar - получение одного значения
// 3. ExecuteQuery - выполнение запросов с заполнением таблицы/списка
```

### 1.2 ExecuteNonQuery - Выполнение команд без возврата данных

```csharp
// Удаление записи из базы данных
int rowsAffected = ZennoPoster.Db.ExecuteNonQuery(
    "DELETE FROM User WHERE Id = 2", 
    null,  // параметры запроса
    ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SqlClient,
    "Data Source=SQLSERVER;Initial Catalog=TestDb;Integrated Security=True;max pool size=500"
);
// Под капотом: отправляется SQL запрос на удаление, возвращается количество затронутых строк

// INSERT запрос
int inserted = ZennoPoster.Db.ExecuteNonQuery(
    "INSERT INTO Users (Name, Age) VALUES ('John', 25)", 
    null,
    ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SqlClient,
    "connection_string_here"
);
// Результат: inserted = 1 (если вставка успешна)
```

### 1.3 ExecuteScalar - Получение одного значения

```csharp
// Подсчет количества записей
string count = ZennoPoster.Db.ExecuteScalar(
    "SELECT COUNT(*) FROM User", 
    null,
    ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SqlClient, 
    "Data Source=SQLSERVER;Initial Catalog=TestDb;Integrated Security=True"
);
// Под капотом: выполняется запрос, возвращается ТОЛЬКО первая колонка первой строки
// Если результат: COUNT(*) = 150, то count = "150"

// Получение конкретного значения
string userName = ZennoPoster.Db.ExecuteScalar(
    "SELECT Name FROM User WHERE Id = 1", 
    null,
    ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SqlClient,
    "connection_string_here"
);
// Если в БД: Id=1, Name="Иван", Age=25
// Результат: userName = "Иван" (Age игнорируется, берется только первая колонка)
```

### 1.4 ExecuteQuery - Заполнение таблиц и списков

```csharp
// Заполнение таблицы проекта
IZennoTable table = project.Tables["Table 1"];
int rowCount = ZennoPoster.Db.ExecuteQuery(
    "SELECT * FROM User", 
    null,
    ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SqlClient,
    "Data Source=SQLSERVER;Initial Catalog=TestDb;Integrated Security=True",
    ref table
);
// Под капотом: все строки результата добавляются в таблицу ZennoPoster
// Если в User 10 записей с полями Id, Name, Age - все 10 строк попадут в Table 1

// Заполнение списка с разделителем
IZennoList list = project.Lists["List 1"];
int rows = ZennoPoster.Db.ExecuteQuery(
    "SELECT Name, Age FROM User", 
    null,
    ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SqlClient,
    "connection_string_here",
    ref list, 
    " | "  // разделитель полей
);
// Результат в списке:
// "Иван | 25"
// "Петр | 30"
// "Мария | 22"
```

---

## ЧАСТЬ 2: РАСШИРЕННАЯ БИБЛИОТЕКА z3nCore.Db

### 2.1 Основные методы работы с данными - ДЕТАЛЬНЫЙ РАЗБОР

#### DbGet - Получение данных из таблицы

```csharp
// ПРИМЕР 1: Простое получение одного поля
project.Variables["acc0"].Value = "5";  // устанавливаем ID записи
string username = project.DbGet("username", "users");

// Что происходит под капотом:
// 1. Формируется SQL запрос: SELECT "username" FROM "users" WHERE "id" = 5
// 2. Выполняется запрос к БД
// 3. Возвращается значение поля username для записи с id=5
// Результат: username = "john_doe"

// ПРИМЕР 2: Получение нескольких полей
string userData = project.DbGet("name,email,phone", "clients");

// SQL под капотом: SELECT "name", "email", "phone" FROM "clients" WHERE "id" = {acc0}
// Если в БД: id=5, name="Иван", email="ivan@mail.ru", phone="+7900111"
// Результат: userData = "Иван\nivan@mail.ru\n+7900111" (поля разделены \n)

// ПРИМЕР 3: Использование альтернативного ключа
string data = project.DbGet("balance", "accounts", key: "login", acc: "admin");

// SQL под капотом: SELECT "balance" FROM "accounts" WHERE "login" = admin
// Ищет запись не по id, а по полю login со значением admin
```

#### DbUpd - Обновление данных в таблице

```csharp
// ПРИМЕР 1: Обновление одного поля
project.Variables["acc0"].Value = "10";
project.DbUpd("status = 'active'", "users");

// SQL под капотом: 
// UPDATE "users" SET "status" = 'active', "last" = '12-28T15:30' WHERE "id" = 10
// Обратите внимание: автоматически добавляется обновление поля last с текущей датой!

// ПРИМЕР 2: Обновление нескольких полей
project.DbUpd("balance = 100, verified = 1, comment = 'VIP client'", "accounts");

// SQL под капотом:
// UPDATE "accounts" SET "balance" = 100, "verified" = 1, "comment" = 'VIP client', "last" = '12-28T15:30' WHERE "id" = {acc0}

// ПРИМЕР 3: Отключение автоматического обновления last
project.DbUpd("name = 'Test'", "users", last: false);

// SQL под капотом:
// UPDATE "users" SET "name" = 'Test' WHERE "id" = {acc0}
// Поле last НЕ обновляется
```

#### DbGetRandom - Получение случайной записи

```csharp
// ПРИМЕР 1: Простое получение случайного значения
string randomUser = project.DbGetRandom("username", "users");

// SQL под капотом:
// SELECT "username" FROM "users" 
// WHERE TRIM("username") != ''  -- исключаем пустые значения
// AND id < {range}              -- ограничиваем диапазоном (если указан)
// ORDER BY RANDOM()             -- случайная сортировка
// LIMIT 1;                      -- берем только одну запись

// Из таблицы users с записями:
// id=1, username=""           <- пропускается (пустое)
// id=2, username="alice"      <- может быть выбрана
// id=3, username="bob"        <- может быть выбрана
// id=4, username="charlie"    <- может быть выбрана
// Результат: randomUser = "bob" (случайно выбранный непустой username)

// ПРИМЕР 2: Получение с ID записи
string randomWithId = project.DbGetRandom("email", "users", acc: true);

// SQL под капотом:
// SELECT "id", "email" FROM "users" 
// WHERE TRIM("email") != ''
// AND id < {range}
// ORDER BY RANDOM()
// LIMIT 1;

// Результат: randomWithId = "7\nuser7@mail.com" 
// Первая строка - ID записи, вторая - значение email

// ПРИМЕР 3: С указанием диапазона
string randomFromRange = project.DbGetRandom("phone", "contacts", range: 100);

// SQL под капотом:
// SELECT "phone" FROM "contacts"
// WHERE TRIM("phone") != ''
// AND id < 100              -- только записи с id меньше 100
// ORDER BY RANDOM()
// LIMIT 1;

// ПРИМЕР 4: Инверсия условия (получить ТОЛЬКО пустые)
string emptyField = project.DbGetRandom("description", "products", invert: true);

// SQL под капотом:
// SELECT "description" FROM "products"
// WHERE TRIM("description") = ''  -- ищем ТОЛЬКО пустые
// AND id < {range}
// ORDER BY RANDOM()
// LIMIT 1;
```

#### SqlGet и SqlUpd - Расширенные методы с WHERE

```csharp
// ПРИМЕР 1: SqlGet с условием WHERE
string activeUsers = project.SqlGet("id,login,email", "accounts", 
    where: "status = 'active' AND created_at > '2024-01-01'");

// SQL под капотом:
// SELECT "id", "login", "email" FROM "accounts" 
// WHERE status = 'active' AND created_at > '2024-01-01'

// Результат для нескольких записей:
// "1\njohn\njohn@mail.com\n5\nalice\nalice@mail.com\n8\nbob\nbob@mail.com"
// Все поля всех записей идут подряд через \n

// ПРИМЕР 2: SqlUpd с условием WHERE
project.SqlUpd("status = 'expired', discount = 0", "subscriptions",
    where: "end_date < '2024-12-01'");

// SQL под капотом:
// UPDATE "subscriptions" 
// SET "status" = 'expired', "discount" = 0, "last" = '12-28T15:30'
// WHERE end_date < '2024-12-01'

// Обновятся ВСЕ записи, удовлетворяющие условию WHERE
```

### 2.2 Управление таблицами - ДЕТАЛЬНЫЙ РАЗБОР

#### TblAdd - Создание таблицы

```csharp
var tableStructure = new Dictionary<string, string>
{
    { "id", "INTEGER PRIMARY KEY" },
    { "username", "TEXT DEFAULT ''" },
    { "balance", "REAL DEFAULT 0" },
    { "is_active", "INTEGER DEFAULT 1" }
};
project.TblAdd(tableStructure, "accounts");

// SQL под капотом для SQLite:
// CREATE TABLE "accounts" (
//     "id" INTEGER PRIMARY KEY,
//     "username" TEXT DEFAULT '',
//     "balance" REAL DEFAULT 0,
//     "is_active" INTEGER DEFAULT 1
// );

// SQL под капотом для PostgreSQL:
// CREATE TABLE "accounts" (
//     "id" SERIAL,  -- AUTOINCREMENT заменяется на SERIAL
//     "username" TEXT DEFAULT '',
//     "balance" REAL DEFAULT 0,
//     "is_active" INTEGER DEFAULT 1
// );

// Если таблица уже существует - ничего не произойдет
```

#### TblExist - Проверка существования таблицы

```csharp
if (project.TblExist("users"))
{
    project.SendInfoToLog("Таблица существует", true);
}

// SQL под капотом для SQLite:
// SELECT COUNT(*) FROM sqlite_master 
// WHERE type='table' AND name='users';
// Возвращает "1" если существует, "0" если нет

// SQL под капотом для PostgreSQL:
// SELECT COUNT(*) FROM information_schema.tables 
// WHERE table_schema = 'public' AND table_name = 'users';
```

#### AddRange - Инициализация диапазона записей

```csharp
// ПРИМЕР 1: Создание 50 пустых записей
project.AddRange("tasks", 50);

// Что происходит под капотом:
// 1. Проверяется максимальный существующий ID:
//    SELECT COALESCE(MAX("id"), 0) FROM "tasks";  -- например, вернет 10
// 
// 2. Добавляются недостающие записи:
//    INSERT INTO "tasks" ("id") VALUES (11) ON CONFLICT DO NOTHING;
//    INSERT INTO "tasks" ("id") VALUES (12) ON CONFLICT DO NOTHING;
//    ...
//    INSERT INTO "tasks" ("id") VALUES (50) ON CONFLICT DO NOTHING;
//
// Результат: в таблице tasks будут записи с id от 1 до 50
// Все поля кроме id будут иметь значения по умолчанию

// ПРИМЕР 2: Использование переменной rangeEnd
project.Variables["rangeEnd"].Value = "100";
project.AddRange("accounts");  // range берется из переменной

// Создаст записи до id=100 включительно
```

### 2.3 Управление колонками - ДЕТАЛЬНЫЙ РАЗБОР

#### ClmnAdd - Добавление колонок

```csharp
// ПРИМЕР 1: Добавление одной колонки
project.ClmnAdd("phone", "users");

// SQL под капотом:
// ALTER TABLE "users" ADD COLUMN "phone" TEXT DEFAULT '';

// Таблица ДО:
// id | username | email
// 1  | alice    | alice@mail.com
// 2  | bob      | bob@mail.com

// Таблица ПОСЛЕ:
// id | username | email          | phone
// 1  | alice    | alice@mail.com | 
// 2  | bob      | bob@mail.com   | 

// ПРИМЕР 2: Добавление колонки с другим типом
project.ClmnAdd("age", "users", defaultValue: "INTEGER DEFAULT 0");

// SQL под капотом:
// ALTER TABLE "users" ADD COLUMN "age" INTEGER DEFAULT 0;

// ПРИМЕР 3: Массовое добавление колонок
var newColumns = new Dictionary<string, string>
{
    { "created_at", "TEXT DEFAULT CURRENT_TIMESTAMP" },
    { "updated_at", "TEXT DEFAULT ''" },
    { "deleted", "INTEGER DEFAULT 0" }
};
project.ClmnAdd(newColumns, "users");

// Выполнится 3 SQL запроса:
// ALTER TABLE "users" ADD COLUMN "created_at" TEXT DEFAULT CURRENT_TIMESTAMP;
// ALTER TABLE "users" ADD COLUMN "updated_at" TEXT DEFAULT '';
// ALTER TABLE "users" ADD COLUMN "deleted" INTEGER DEFAULT 0;
```

#### ClmnDrop - Удаление колонок

```csharp
project.ClmnDrop("temporary_field", "users");

// SQL под капотом для PostgreSQL:
// ALTER TABLE "users" DROP COLUMN "temporary_field" CASCADE;

// SQL под капотом для SQLite:
// ALTER TABLE "users" DROP COLUMN "temporary_field";

// ВАЖНО: В SQLite удаление колонок может не поддерживаться в старых версиях
```

#### ClmnPrune - Удаление лишних колонок

```csharp
// Определяем какие колонки должны остаться
var requiredColumns = new Dictionary<string, string>
{
    { "id", "INTEGER PRIMARY KEY" },
    { "username", "TEXT DEFAULT ''" },
    { "email", "TEXT DEFAULT ''" }
};

// В таблице есть колонки: id, username, email, phone, address, temp_data
project.ClmnPrune(requiredColumns, "users");

// Что происходит под капотом:
// 1. Получается список текущих колонок
// 2. Сравнивается с requiredColumns
// 3. Удаляются лишние:
//    ALTER TABLE "users" DROP COLUMN "phone";
//    ALTER TABLE "users" DROP COLUMN "address";
//    ALTER TABLE "users" DROP COLUMN "temp_data";

// В результате останутся только: id, username, email
```

### 2.4 Специальные методы - ДЕТАЛЬНЫЙ РАЗБОР

#### DbSettings - Загрузка настроек из БД

```csharp
project.DbSettings();

// SQL под капотом:
// SELECT "id", "value" FROM "_settings"

// Таблица _settings:
// id          | value
// ------------|----------------
// apiKey      | sk-1234567890
// maxRetries  | 5
// timeout     | 30000
// baseUrl     | https://api.example.com

// После выполнения создаются переменные проекта:
// project.Variables["apiKey"].Value = "sk-1234567890"
// project.Variables["maxRetries"].Value = "5"
// project.Variables["timeout"].Value = "30000"
// project.Variables["baseUrl"].Value = "https://api.example.com"
```

#### MigrateTable - Копирование таблицы

```csharp
project.MigrateTable("users_old", "users_new");

// Что происходит под капотом:
// 1. Создается новая таблица users_new с той же структурой
// 2. Копируются все данные: 
//    INSERT INTO "users_new" SELECT * FROM "users_old"
// 3. Переименовываются специальные колонки:
//    ALTER TABLE "users_new" RENAME COLUMN "acc0" TO "id"
//    ALTER TABLE "users_new" RENAME COLUMN "key" TO "id"
```

### 2.5 Низкоуровневый метод DbQ

```csharp
// DbQ - прямое выполнение любого SQL запроса

// SELECT запросы возвращают данные
string result = project.DbQ("SELECT username FROM users WHERE id = 5");
// Результат: "alice"

string multiRow = project.DbQ("SELECT username FROM users LIMIT 3");
// Результат: "alice\nbob\ncharlie" (строки через \n)

// Запросы модификации возвращают количество затронутых строк
string affected = project.DbQ("UPDATE users SET status = 'active' WHERE age > 18");
// Результат: "15" (обновлено 15 записей)

// С логированием для отладки
project.DbQ("DELETE FROM logs WHERE created_at < '2024-01-01'", log: true);
// В лог выведется: 🐘 [DELETE FROM logs WHERE created_at < '2024-01-01'] - [120]
// (удалено 120 записей, 🐘 означает PostgreSQL)

// Защита от опасных запросов
project.DbQ("DELETE FROM users");  // ОШИБКА! DELETE без WHERE запрещен
project.DbQ("DROP TABLE users");    // ОШИБКА! DROP без WHERE запрещен
```

### 2.6 Практические примеры комплексного использования

```csharp
// СЦЕНАРИЙ: Система управления аккаунтами с проксями

// 1. Создаем структуру БД
var accountStructure = new Dictionary<string, string>
{
    { "id", "INTEGER PRIMARY KEY" },
    { "login", "TEXT DEFAULT ''" },
    { "password", "TEXT DEFAULT ''" },
    { "proxy", "TEXT DEFAULT ''" },
    { "status", "TEXT DEFAULT 'new'" },
    { "last_check", "TEXT DEFAULT ''" },
    { "error_count", "INTEGER DEFAULT 0" },
    { "cookies", "TEXT DEFAULT ''" }
};

project.TblAdd(accountStructure, "accounts");
project.AddRange("accounts", 100);  // Создаем 100 записей

// 2. Заполняем аккаунты данными
for (int i = 1; i <= 100; i++)
{
    string login = "user_" + Guid.NewGuid().ToString().Substring(0, 8);
    string password = "pass_" + new Random().Next(100000, 999999);
    
    project.Variables["acc0"].Value = i.ToString();
    project.DbUpd($"login = '{login}', password = '{password}'", "accounts");
}

// 3. Получаем случайный необработанный аккаунт
string accountData = project.DbGetRandom("login,password,proxy", "accounts", acc: true);
// accountData = "42\nuser_a3b4c5d6\npass_567890\nhttp://proxy.com:8080"

string[] parts = accountData.Split('\n');
string accId = parts[0];
string login = parts[1];
string password = parts[2];
string proxy = parts[3];

// 4. Работаем с аккаунтом
project.Variables["acc0"].Value = accId;
try 
{
    // ... выполняем действия ...
    project.DbUpd("status = 'active', cookies = 'cookie_data_here'", "accounts");
}
catch (Exception ex)
{
    // Увеличиваем счетчик ошибок
    string currentErrors = project.DbGet("error_count", "accounts");
    int errors = int.Parse(currentErrors) + 1;
    project.DbUpd($"status = 'error', error_count = {errors}", "accounts");
}

// 5. Анализ статистики
string stats = project.DbQ(@"
    SELECT status, COUNT(*) as cnt 
    FROM accounts 
    GROUP BY status
");
// Результат: "active\n45\nnew\n30\nerror\n25"
```

### ⚠️ ВАЖНЫЕ ОСОБЕННОСТИ РЕАЛИЗАЦИИ

1. **Автоматическое экранирование**: Все имена таблиц и колонок автоматически оборачиваются в кавычки
2. **Поле last**: При DbUpd автоматически обновляется с текущим временем (можно отключить параметром last: false)
3. **Переменная acc0**: Используется как ID текущей записи по умолчанию
4. **Разделители в результатах**: Многострочные результаты разделяются символом \n
5. **Защита от опасных запросов**: DROP и DELETE без WHERE автоматически блокируются
6. **Различия PostgreSQL/SQLite**: Библиотека автоматически адаптирует SQL под нужную БД