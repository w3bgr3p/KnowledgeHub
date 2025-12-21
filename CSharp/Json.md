# Работа с JSON в C# (Json.NET / Newtonsoft.Json)

## Иерархия типов - что есть что

В библиотеке Json.NET есть три основных типа, которые образуют иерархию. Давай разберем их от общего к конкретному.

**JToken** - это базовый абстрактный класс для всех JSON-элементов. Думай о нем как о `object` в мире JSON - это может быть что угодно: объект, массив, строка, число, булево значение. Когда ты итерируешься по JArray через `foreach`, каждый элемент возвращается именно как JToken, потому что компилятор не знает, что именно там внутри.

**JObject** - это конкретный класс, который представляет JSON-объект (ключ-значение пары). Это как словарь. Он наследуется от JToken, поэтому является его частным случаем.

**JArray** - это конкретный класс для JSON-массива. Тоже наследуется от JToken. Это коллекция других JToken'ов.

Думай о них как о иерархии: JToken → JObject и JToken → JArray. Аналогично тому, как в C# все наследуется от object, в JSON-мире все наследуется от JToken.

## Проблема в твоем коде

Когда ты пишешь `var taskToken in tasks`, переменная `taskToken` имеет тип JToken. Потом ты делаешь `var task = JObject.Parse(taskToken)` - это ошибка, потому что Parse принимает строку, а не JToken. И дальше пытаешься обратиться `task.id` - но у JToken нет свойства `id`, это же не C#-объект со строгими полями.

## Как правильно работать с полями

Есть три основных способа доступа к полям JSON-объекта:

**Способ 1: Через индексатор (самый надежный)**

```csharp
var id = task["id"].ToString();
var name = task["name"].ToString();
var done = task["done"].ToString();
```

Здесь `task["id"]` возвращает JToken, который ты потом конвертируешь в строку. Это работает независимо от того, JToken у тебя или JObject.

**Способ 2: Через Value<T> (с типизацией)**

```csharp
var id = task.Value<int>("id");
var name = task.Value<string>("name");
var done = task.Value<bool>("done");
```

Это лучше, потому что сразу получаешь нужный тип. Если поле отсутствует, получишь default(T).

**Способ 3: Через приведение типа + dynamic (небезопасный, но короткий)**

```csharp
dynamic task = taskToken;
var id = task.id.ToString();
```

Работает, но теряешь проверку типов на этапе компиляции. В ZennoPoster может быть глючно.

## Правильная версия твоего кода

Вот несколько вариантов исправления:

**Вариант A: Минимальное исправление твоего кода**

```csharp
var tasksResp = project.GET("https://api-mm.pip.world/xp-tasks", "+");
JArray tasks = JArray.Parse(tasksResp);

foreach(var taskToken in tasks)
{
    // Приводим JToken к JObject - это безопасное приведение типа
    var task = (JObject)taskToken;
    
    // Обращаемся через индексатор
    var id = task["id"].ToString();
    var done = task["done"].ToString();
    var name = task["name"].ToString();
    
    project.log($"{id} {name} {done}");
}
```

**Вариант B: С типизацией (рекомендую)**

```csharp
var tasksResp = project.GET("https://api-mm.pip.world/xp-tasks", "+");
JArray tasks = JArray.Parse(tasksResp);

foreach(JObject task in tasks) // Сразу указываем тип в foreach
{
    // Используем Value<T> для правильной типизации
    var id = task.Value<int>("id");
    var name = task.Value<string>("name");
    var done = task.Value<bool>("done");
    
    project.log($"{id} {name} {done}");
}
```

**Вариант C: Через LINQ (если нужна коллекция объектов)**

```csharp
var tasksResp = project.GET("https://api-mm.pip.world/xp-tasks", "+");
JArray tasks = JArray.Parse(tasksResp);

// Преобразуем массив JToken'ов в список объектов C#
var taskList = tasks.Select(t => new 
{
    id = t.Value<int>("id"),
    name = t.Value<string>("name"),
    done = t.Value<bool>("done")
}).ToList();

foreach(var task in taskList)
{
    project.log($"{task.id} {task.name} {task.done}");
}
```

## Частые ситуации и как их решать

**Когда поле может отсутствовать:**

```csharp
var description = task.Value<string>("description") ?? "нет описания";
// Или проверка на существование
if(task["description"] != null)
{
    var desc = task["description"].ToString();
}
```

**Когда нужно достать вложенный объект:**

```csharp
// JSON: { "user": { "name": "John", "age": 30 } }
var userName = task["user"]["name"].ToString();
// Или
var user = task["user"] as JObject;
var age = user.Value<int>("age");
```

**Когда нужно достать массив:**

```csharp
// JSON: { "tags": ["tag1", "tag2"] }
var tagsArray = task["tags"] as JArray;
foreach(var tag in tagsArray)
{
    var tagValue = tag.ToString();
}
```

## Ключевой момент

Проблема в том, что JSON - это динамическая структура без строгой типизации, а C# - строго типизированный язык. JToken - это мост между ними. Когда ты парсишь JSON, все приходит как JToken, и тебе нужно явно сказать компилятору, что именно там внутри (JObject, JArray, строка, число) через приведение типа или через методы вроде Value<T>.

Твоя ошибка была в том, что ты пытался обращаться к JToken как к C#-объекту с полями, но у JToken нет полей `id`, `name`, `done` - это просто контейнер для JSON-данных. Нужно использовать индексатор `["field"]` или метод `Value<T>("field")`.