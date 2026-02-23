# Работа с JSON в C# 7 с использованием Newtonsoft.Json

## Введение

JSON (JavaScript Object Notation) — это текстовый формат обмена данными, основанный на подмножестве языка JavaScript. В C# для работы с JSON чаще всего используется библиотека Newtonsoft.Json (также известная как Json.NET), которая предоставляет мощные и гибкие инструменты для сериализации и десериализации данных.

## Основные подходы к парсингу JSON

### 1. Использование dynamic (как в вашем примере с ZennoPoster)

```csharp
using Newtonsoft.Json;
using System.Dynamic;

// Ваш JSON в виде строки
string jsonString = @"{...ваш JSON...}";

// Десериализация в dynamic объект
dynamic data = JsonConvert.DeserializeObject<ExpandoObject>(jsonString);

// Доступ к свойствам через точечную нотацию
string doneMissions = data.payload?.doneMissions?.ToString() ?? "";
string dailyMintStreak = data.payload?.dailyMintStreak?.ToString() ?? "";
```

**Преимущества:**
- Не требует создания классов
- Быстрая реализация для простых задач
- Гибкость при работе с неизвестной структурой

**Недостатки:**
- Отсутствие IntelliSense
- Ошибки только во время выполнения
- Снижение производительности
- Сложность отладки

### 2. Использование JObject/JToken (рекомендуемый подход для гибкости)

```csharp
using Newtonsoft.Json.Linq;

string jsonString = @"{
    ""errors"":[],
    ""payload"":{
        ""doneMissions"":""xion-daily-mint"",
        ""dailyMintStreak"":10,
        ""balance"":0.35167,
        ""account"":{
            ""userId"":""ph3Wb4NK"",
            ""username"":""blockbarronbe""
        }
    }
}";

// Парсинг в JObject
JObject root = JObject.Parse(jsonString);

// Извлечение значений разными способами
string doneMissions = root["payload"]?["doneMissions"]?.ToString() ?? "";
int dailyMintStreak = root["payload"]?["dailyMintStreak"]?.Value<int>() ?? 0;
decimal balance = root["payload"]?["balance"]?.Value<decimal>() ?? 0m;

// Альтернативный способ через SelectToken (путь в стиле JSONPath)
string userId = root.SelectToken("payload.account.userId")?.ToString() ?? "";
string username = root.SelectToken("payload.account.username")?.ToString() ?? "";

// Проверка существования свойства
bool hasPayload = root["payload"] != null;
bool hasErrors = root["errors"] != null && root["errors"].HasValues;
```

### 3. Использование строго типизированных классов (рекомендуемый подход для производства)

```csharp
// Определяем структуру классов согласно JSON
public class RootObject
{
    public List<object> Errors { get; set; } = new List<object>();
    public Payload Payload { get; set; }
    public bool Success { get; set; }
    public DateTime Now { get; set; }
}

public class Payload
{
    public string DoneMissions { get; set; }
    public Dictionary<string, bool> EligibleToMint { get; set; }
    public Dictionary<string, int> MintedCount { get; set; }
    public int DailyMintStreak { get; set; }
    public decimal Balance { get; set; }
    public int TotalApprovedReferrals { get; set; }
    public double EarnByHour { get; set; }
    public Session Session { get; set; }
    public decimal Staked { get; set; }
    public int UserXp { get; set; }
    public bool PartnerNftXpClaimed { get; set; }
    public Account Account { get; set; }
}

public class Session
{
    public DateTime ExpiresOn { get; set; }
    public string Token { get; set; }
}

public class Account
{
    public string UserId { get; set; }
    public DateTime CreatedOn { get; set; }
    public DateTime ModifiedOn { get; set; }
    public int InvitedCount { get; set; }
    public string Twitter { get; set; }
    public string Discord { get; set; }
    public string Github { get; set; }
    public string Telegram { get; set; }
    public string Reddit { get; set; }
    public string Email { get; set; }
    public string Username { get; set; }
    public bool HasAvatar { get; set; }
    public decimal XpMultiplier { get; set; }
    public Dictionary<string, object> Metadata { get; set; }
    public List<Wallet> Wallets { get; set; }
}

public class Wallet
{
    public string Network { get; set; }
    public string Address { get; set; }
    public DateTime CreatedOn { get; set; }
    public DateTime ModifiedOn { get; set; }
}

// Использование
string jsonString = @"{...ваш JSON...}";
RootObject data = JsonConvert.DeserializeObject<RootObject>(jsonString);

// Теперь у нас есть строгая типизация и IntelliSense
string doneMissions = data.Payload.DoneMissions;
int dailyMintStreak = data.Payload.DailyMintStreak;
decimal balance = data.Payload.Balance;
string userId = data.Payload.Account.UserId;
```

## Примеры парсинга в различные структуры данных

### Парсинг в словарь (Dictionary)

```csharp
// Пример 1: Парсинг объекта eligibleToMint в словарь
JObject root = JObject.Parse(jsonString);
JObject eligibleToMintObj = root["payload"]["eligibleToMint"] as JObject;

// Способ с LINQ
Dictionary<string, bool> eligibleDict = eligibleToMintObj
    .Properties()
    .ToDictionary(
        prop => prop.Name,
        prop => prop.Value.Value<bool>()
    );

// Способ без LINQ (для понимания логики)
Dictionary<string, bool> eligibleDictNoLinq = new Dictionary<string, bool>();
foreach (JProperty property in eligibleToMintObj.Properties())
{
    string key = property.Name;
    bool value = property.Value.Value<bool>();
    eligibleDictNoLinq.Add(key, value);
}

// Вывод результата
foreach (var kvp in eligibleDict)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
// Результат:
// xion-early: false
// xion-bridge-2: false
// xion-faithful: false
// и т.д.
```

```csharp
// Пример 2: Парсинг всего payload в словарь для гибкой работы
Dictionary<string, object> payloadDict = root["payload"]
    .ToObject<Dictionary<string, object>>();

// Доступ к значениям
string doneMissions = payloadDict["doneMissions"]?.ToString();
int dailyMintStreak = Convert.ToInt32(payloadDict["dailyMintStreak"]);
```

### Парсинг в список (List)

```csharp
// Пример 1: Парсинг массива wallets в список
JArray walletsArray = root["payload"]["account"]["wallets"] as JArray;

// Способ с LINQ - получение списка адресов
List<string> addresses = walletsArray
    .Select(w => w["address"].ToString())
    .ToList();

// Способ без LINQ
List<string> addressesNoLinq = new List<string>();
foreach (JToken wallet in walletsArray)
{
    string address = wallet["address"].ToString();
    addressesNoLinq.Add(address);
}

// Пример 2: Парсинг в список объектов
List<Wallet> wallets = walletsArray.ToObject<List<Wallet>>();

// Теперь можно работать с типизированными объектами
foreach (Wallet wallet in wallets)
{
    Console.WriteLine($"Network: {wallet.Network}");
    Console.WriteLine($"Address: {wallet.Address}");
    Console.WriteLine($"Created: {wallet.CreatedOn}");
}
```

```csharp
// Пример 3: Извлечение всех булевых значений из eligibleToMint в список
JObject eligibleObj = root["payload"]["eligibleToMint"] as JObject;

// С LINQ
List<KeyValuePair<string, bool>> eligibleList = eligibleObj
    .Properties()
    .Select(p => new KeyValuePair<string, bool>(p.Name, p.Value.Value<bool>()))
    .ToList();

// Без LINQ
List<KeyValuePair<string, bool>> eligibleListNoLinq = 
    new List<KeyValuePair<string, bool>>();
foreach (JProperty prop in eligibleObj.Properties())
{
    var kvp = new KeyValuePair<string, bool>(
        prop.Name, 
        prop.Value.Value<bool>()
    );
    eligibleListNoLinq.Add(kvp);
}

// Фильтрация - получить только доступные для минта
List<string> availableForMint = eligibleList
    .Where(kvp => kvp.Value == true)
    .Select(kvp => kvp.Key)
    .ToList();

// Без LINQ
List<string> availableForMintNoLinq = new List<string>();
foreach (var kvp in eligibleList)
{
    if (kvp.Value == true)
    {
        availableForMintNoLinq.Add(kvp.Key);
    }
}
```

### Парсинг в строку с форматированием

```csharp
// Пример 1: Извлечение и форматирование данных аккаунта
JObject root = JObject.Parse(jsonString);
JObject account = root["payload"]["account"] as JObject;

// Создание форматированной строки с информацией об аккаунте
string accountInfo = $@"
=== Информация об аккаунте ===
User ID: {account["userId"]}
Username: {account["username"]}
Создан: {account["createdOn"]}
Изменен: {account["modifiedOn"]}
Приглашено: {account["invitedCount"]}
Аватар: {(account["hasAvatar"].Value<bool>() ? "Да" : "Нет")}
Множитель XP: {account["xpMultiplier"]}
";

Console.WriteLine(accountInfo);

// Пример 2: Создание CSV строки из данных
StringBuilder csvBuilder = new StringBuilder();
csvBuilder.AppendLine("Параметр,Значение");
csvBuilder.AppendLine($"User ID,{root.SelectToken("payload.account.userId")}");
csvBuilder.AppendLine($"Username,{root.SelectToken("payload.account.username")}");
csvBuilder.AppendLine($"Balance,{root.SelectToken("payload.balance")}");
csvBuilder.AppendLine($"Daily Streak,{root.SelectToken("payload.dailyMintStreak")}");
csvBuilder.AppendLine($"User XP,{root.SelectToken("payload.userXp")}");

string csvData = csvBuilder.ToString();
```

## Работа с разными уровнями вложенности

```csharp
// Пример навигации по разным уровням JSON
JObject root = JObject.Parse(jsonString);

// Уровень 1 - корневые свойства
bool success = root["success"].Value<bool>();
DateTime now = root["now"].Value<DateTime>();

// Уровень 2 - payload
decimal balance = root["payload"]["balance"].Value<decimal>();
int userXp = root["payload"]["userXp"].Value<int>();

// Уровень 3 - вложенные объекты
string userId = root["payload"]["account"]["userId"].ToString();
string sessionToken = root["payload"]["session"]["token"].ToString();

// Уровень 4 - массивы внутри вложенных объектов
string firstWalletAddress = root["payload"]["account"]["wallets"][0]["address"].ToString();

// Безопасная навигация с проверкой null
string safeWalletAddress = root["payload"]?["account"]?["wallets"]?[0]?["address"]?.ToString() 
    ?? "Адрес не найден";

// Использование SelectToken для глубокой навигации
string deepValue = root.SelectToken("payload.account.wallets[0].network")?.ToString() 
    ?? "Значение не найдено";
```

## Сравнение производительности разных подходов

```csharp
public class PerformanceComparison
{
    public static void CompareApproaches(string jsonString)
    {
        var sw = new Stopwatch();
        
        // 1. Dynamic подход (как в ZennoPoster)
        sw.Start();
        for (int i = 0; i < 10000; i++)
        {
            dynamic data = JsonConvert.DeserializeObject<ExpandoObject>(jsonString);
            string value = data.payload?.balance?.ToString() ?? "";
        }
        sw.Stop();
        Console.WriteLine($"Dynamic: {sw.ElapsedMilliseconds} ms");
        
        // 2. JObject подход
        sw.Restart();
        for (int i = 0; i < 10000; i++)
        {
            JObject data = JObject.Parse(jsonString);
            string value = data["payload"]?["balance"]?.ToString() ?? "";
        }
        sw.Stop();
        Console.WriteLine($"JObject: {sw.ElapsedMilliseconds} ms");
        
        // 3. Типизированный подход
        sw.Restart();
        for (int i = 0; i < 10000; i++)
        {
            RootObject data = JsonConvert.DeserializeObject<RootObject>(jsonString);
            string value = data.Payload?.Balance.ToString() ?? "";
        }
        sw.Stop();
        Console.WriteLine($"Typed: {sw.ElapsedMilliseconds} ms");
    }
}
```

## Обработка ошибок и особых случаев

```csharp
public class SafeJsonParser
{
    // Безопасный парсинг с обработкой ошибок
    public static T SafeDeserialize<T>(string json) where T : class, new()
    {
        try
        {
            // Настройки для обработки ошибок
            var settings = new JsonSerializerSettings
            {
                NullValueHandling = NullValueHandling.Ignore,
                MissingMemberHandling = MissingMemberHandling.Ignore,
                Error = (sender, args) =>
                {
                    // Логирование ошибки, но продолжение парсинга
                    Console.WriteLine($"Ошибка парсинга: {args.ErrorContext.Error.Message}");
                    args.ErrorContext.Handled = true;
                }
            };
            
            return JsonConvert.DeserializeObject<T>(json, settings) ?? new T();
        }
        catch (JsonException ex)
        {
            Console.WriteLine($"Критическая ошибка JSON: {ex.Message}");
            return new T();
        }
    }
    
    // Безопасное извлечение значения
    public static string SafeGetValue(JObject obj, string path, string defaultValue = "")
    {
        try
        {
            var token = obj.SelectToken(path);
            return token?.ToString() ?? defaultValue;
        }
        catch
        {
            return defaultValue;
        }
    }
    
    // Проверка структуры JSON перед парсингом
    public static bool ValidateJsonStructure(string json)
    {
        try
        {
            JObject.Parse(json);
            return true;
        }
        catch
        {
            return false;
        }
    }
}

// Использование
string jsonString = GetJsonFromApi();

if (SafeJsonParser.ValidateJsonStructure(jsonString))
{
    var data = SafeJsonParser.SafeDeserialize<RootObject>(jsonString);
    
    // Или с JObject
    JObject jObj = JObject.Parse(jsonString);
    string balance = SafeJsonParser.SafeGetValue(jObj, "payload.balance", "0");
}
```

## Оптимизированная альтернатива коду ZennoPoster

```csharp
// Вместо медленного dynamic подхода
public class OptimizedXionParser
{
    // Метод 1: Использование JObject (быстрее dynamic, но гибкий)
    public static XionData ParseWithJObject(string jsonString)
    {
        JObject root = JObject.Parse(jsonString);
        JObject payload = root["payload"] as JObject;
        JObject account = payload["account"] as JObject;
        
        return new XionData
        {
            DoneMissions = payload["doneMissions"]?.ToString() ?? "",
            TotalApprovedReferrals = payload["totalApprovedReferrals"]?.ToString() ?? "",
            DailyMintStreak = payload["dailyMintStreak"]?.ToString() ?? "",
            PartnerNftXpClaimed = payload["partnerNftXpClaimed"]?.ToString() ?? "",
            RefCode = account["userId"]?.ToString() ?? "",
            Staked = payload["staked"]?.ToString() ?? "",
            UserXp = payload["userXp"]?.ToString() ?? "",
            Balance = payload["balance"]?.ToString() ?? "",
            EarnByHour = payload["earnByHour"]?.ToString() ?? "",
            UserId = account["userId"]?.ToString() ?? "",
            CreatedOn = account["createdOn"]?.ToString() ?? "",
            ModifiedOn = account["modifiedOn"]?.ToString() ?? "",
            Username = account["username"]?.ToString() ?? "",
            HasAvatar = account["hasAvatar"]?.ToString() ?? ""
        };
    }
    
    // Метод 2: Прямая десериализация (самый быстрый)
    public static XionData ParseWithTypes(string jsonString)
    {
        var root = JsonConvert.DeserializeObject<RootObject>(jsonString);
        
        return new XionData
        {
            DoneMissions = root.Payload.DoneMissions ?? "",
            TotalApprovedReferrals = root.Payload.TotalApprovedReferrals.ToString(),
            DailyMintStreak = root.Payload.DailyMintStreak.ToString(),
            PartnerNftXpClaimed = root.Payload.PartnerNftXpClaimed.ToString(),
            RefCode = root.Payload.Account.UserId ?? "",
            Staked = root.Payload.Staked.ToString(),
            UserXp = root.Payload.UserXp.ToString(),
            Balance = root.Payload.Balance.ToString(),
            EarnByHour = root.Payload.EarnByHour.ToString(),
            UserId = root.Payload.Account.UserId ?? "",
            CreatedOn = root.Payload.Account.CreatedOn.ToString(),
            ModifiedOn = root.Payload.Account.ModifiedOn.ToString(),
            Username = root.Payload.Account.Username ?? "",
            HasAvatar = root.Payload.Account.HasAvatar.ToString()
        };
    }
}

public class XionData
{
    public string DoneMissions { get; set; }
    public string TotalApprovedReferrals { get; set; }
    public string DailyMintStreak { get; set; }
    public string PartnerNftXpClaimed { get; set; }
    public string RefCode { get; set; }
    public string Staked { get; set; }
    public string UserXp { get; set; }
    public string Balance { get; set; }
    public string EarnByHour { get; set; }
    public string UserId { get; set; }
    public string CreatedOn { get; set; }
    public string ModifiedOn { get; set; }
    public string Username { get; set; }
    public string HasAvatar { get; set; }
}

// Использование
var x = new Xion(project, instance);
var stats = x.GetWithHeaders("https://api-xion2.bonusblock.io/api/xion/get-status");

// Вместо медленного dynamic
var data = OptimizedXionParser.ParseWithJObject(stats);
// или еще быстрее
var data2 = OptimizedXionParser.ParseWithTypes(stats);

// Теперь все данные доступны через свойства
Console.WriteLine($"Balance: {data.Balance}");
Console.WriteLine($"Username: {data.Username}");
```

## Полезные утилиты для работы с JSON

```csharp
public static class JsonHelpers
{
    // Красивое форматирование JSON
    public static string PrettyPrint(string json)
    {
        var parsedJson = JObject.Parse(json);
        return parsedJson.ToString(Formatting.Indented);
    }
    
    // Сравнение двух JSON объектов
    public static bool AreJsonEqual(string json1, string json2)
    {
        var obj1 = JObject.Parse(json1);
        var obj2 = JObject.Parse(json2);
        return JToken.DeepEquals(obj1, obj2);
    }
    
    // Слияние двух JSON объектов
    public static string MergeJson(string json1, string json2)
    {
        var obj1 = JObject.Parse(json1);
        var obj2 = JObject.Parse(json2);
        
        obj1.Merge(obj2, new JsonMergeSettings
        {
            MergeArrayHandling = MergeArrayHandling.Union
        });
        
        return obj1.ToString();
    }
    
    // Извлечение всех значений определенного типа
    public static List<T> ExtractAllValues<T>(string json, string propertyName)
    {
        var root = JObject.Parse(json);
        return root.Descendants()
            .OfType<JProperty>()
            .Where(p => p.Name == propertyName)
            .Select(p => p.Value.Value<T>())
            .ToList();
    }
}
```

## Рекомендации по выбору подхода

### Используйте **dynamic** когда:
- Структура JSON неизвестна или часто меняется
- Нужен быстрый прототип
- Производительность не критична

### Используйте **JObject/JToken** когда:
- Нужна гибкость без потери производительности
- Требуется манипулировать JSON (добавлять/удалять свойства)
- Структура JSON частично известна

### Используйте **типизированные классы** когда:
- Структура JSON стабильна и известна
- Важна производительность
- Нужна поддержка IntelliSense и проверка типов
- Проект долгосрочный и требует поддержки

## Заключение

Выбор метода парсинга JSON зависит от конкретной задачи. Для производственного кода рекомендуется использовать типизированные классы или JObject, избегая dynamic из-за проблем с производительностью. Помните, что правильная обработка ошибок и валидация данных так же важны, как и сам парсинг.