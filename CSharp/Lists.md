# Полное руководство по работе со списками в C#

## Введение

`List<T>` в C# — это одна из самых мощных и универсальных коллекций в .NET. В отличие от массивов, списки динамически изменяют свой размер и предоставляют богатый набор методов для работы с данными.

## Базовые операции

### Создание и инициализация списков

```csharp
// Создание пустого списка
List<int> numbers = new List<int>();

// Инициализация с элементами
List<string> names = new List<string> { "Анна", "Борис", "Виктор" };

// Создание списка определенной емкости (для оптимизации)
List<int> largeList = new List<int>(1000);

// Создание списка из другой коллекции
int[] array = { 1, 2, 3, 4, 5 };
List<int> fromArray = new List<int>(array);
```

### Добавление элементов

```csharp
List<string> fruits = new List<string>();

// Добавление одного элемента
fruits.Add("Яблоко");

// Добавление нескольких элементов
fruits.AddRange(new[] { "Банан", "Апельсин", "Груша" });

// Вставка элемента по индексу
fruits.Insert(1, "Киви"); // Вставляет "Киви" на позицию 1

// Вставка коллекции элементов по индексу
fruits.InsertRange(2, new[] { "Манго", "Ананас" });

Console.WriteLine(string.Join(", ", fruits));
// Выводит: Яблоко, Киви, Манго, Ананас, Банан, Апельсин, Груша
```

### Удаление элементов

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 2, 4, 5, 2 };

// Удаление первого вхождения элемента
numbers.Remove(2); // Удаляет первую "2"

// Удаление по индексу
numbers.RemoveAt(0); // Удаляет элемент с индексом 0

// Удаление диапазона элементов
numbers.RemoveRange(1, 2); // Удаляет 2 элемента начиная с индекса 1

// Удаление всех элементов по условию
numbers.RemoveAll(x => x > 3); // Удаляет все элементы больше 3

// Очистка всего списка
numbers.Clear();
```

### Поиск и проверка

```csharp
List<string> cities = new List<string> { "Москва", "Санкт-Петербург", "Новосибирск", "Екатеринбург" };

// Проверка наличия элемента
bool hasMoscow = cities.Contains("Москва");

// Поиск индекса элемента
int index = cities.IndexOf("Новосибирск"); // Возвращает 2

// Поиск последнего индекса
int lastIndex = cities.LastIndexOf("Москва");

// Поиск по условию
string longCity = cities.Find(city => city.Length > 10); // "Санкт-Петербург"

// Поиск всех элементов по условию
List<string> longCities = cities.FindAll(city => city.Length > 10);

// Проверка условия для всех элементов
bool allLong = cities.All(city => city.Length > 5);

// Проверка условия хотя бы для одного элемента
bool anyLong = cities.Any(city => city.Length > 15);
```

## Продвинутые операции

### 1. Удаление дубликатов с помощью HashSet

```csharp
// Простое удаление дубликатов
List<int> numbersWithDuplicates = new List<int> { 1, 2, 3, 2, 4, 1, 5 };
List<int> uniqueNumbers = new HashSet<int>(numbersWithDuplicates).ToList();

// Удаление дубликатов с сохранением порядка
List<int> uniqueOrdered = numbersWithDuplicates
    .GroupBy(x => x)
    .Select(g => g.First())
    .ToList();

// Удаление дубликатов для сложных объектов
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

List<Person> people = new List<Person>
{
    new Person { Name = "Анна", Age = 25 },
    new Person { Name = "Борис", Age = 30 },
    new Person { Name = "Анна", Age = 25 }, // Дубликат
    new Person { Name = "Виктор", Age = 35 }
};

// Удаление дубликатов по имени
var uniqueByName = people
    .GroupBy(p => p.Name)
    .Select(g => g.First())
    .ToList();

// Удаление дубликатов с помощью DistinctBy (C# 6+)
var uniqueByNameNew = people.DistinctBy(p => p.Name).ToList();
```

### 2. Сортировка и упорядочивание

```csharp
List<int> numbers = new List<int> { 5, 2, 8, 1, 9, 3 };

// Простая сортировка
numbers.Sort(); // Изменяет исходный список

// Сортировка в обратном порядке
numbers.Sort((a, b) => b.CompareTo(a));

// Сортировка сложных объектов
List<Person> people = GetPeople();

// Сортировка по возрасту
people.Sort((p1, p2) => p1.Age.CompareTo(p2.Age));

// Множественная сортировка с LINQ
var sortedPeople = people
    .OrderBy(p => p.Age)
    .ThenBy(p => p.Name)
    .ToList();

// Пользовательская сортировка
people.Sort((p1, p2) =>
{
    int ageComparison = p1.Age.CompareTo(p2.Age);
    return ageComparison != 0 ? ageComparison : p1.Name.CompareTo(p2.Name);
});
```

### 3. Группировка и агрегация

```csharp
List<Sale> sales = new List<Sale>
{
    new Sale { Product = "Хлеб", Category = "Продукты", Amount = 100 },
    new Sale { Product = "Молоко", Category = "Продукты", Amount = 80 },
    new Sale { Product = "Футболка", Category = "Одежда", Amount = 1500 },
    new Sale { Product = "Джинсы", Category = "Одежда", Amount = 3000 }
};

// Группировка по категории
var groupedByCategory = sales
    .GroupBy(s => s.Category)
    .Select(g => new
    {
        Category = g.Key,
        TotalAmount = g.Sum(s => s.Amount),
        Count = g.Count(),
        Products = g.Select(s => s.Product).ToList()
    })
    .ToList();

// Создание словаря из списка
Dictionary<string, List<Sale>> salesByCategory = sales
    .GroupBy(s => s.Category)
    .ToDictionary(g => g.Key, g => g.ToList());
```

### 4. Разбиение списка на части (Chunking)

```csharp
List<int> numbers = Enumerable.Range(1, 20).ToList();

// Разбиение на части по 5 элементов (C# 6+)
var chunks = numbers.Chunk(5).ToList();

// Альтернативный способ для старых версий C#
public static IEnumerable<IEnumerable<T>> Chunk<T>(IEnumerable<T> source, int chunkSize)
{
    return source
        .Select((x, i) => new { Index = i, Value = x })
        .GroupBy(x => x.Index / chunkSize)
        .Select(x => x.Select(v => v.Value));
}

var manualChunks = Chunk(numbers, 5).ToList();
```

### 5. Операции с множествами

```csharp
List<int> list1 = new List<int> { 1, 2, 3, 4, 5 };
List<int> list2 = new List<int> { 4, 5, 6, 7, 8 };

// Пересечение
var intersection = list1.Intersect(list2).ToList(); // { 4, 5 }

// Объединение без дубликатов
var union = list1.Union(list2).ToList(); // { 1, 2, 3, 4, 5, 6, 7, 8 }

// Разность (элементы из первого списка, которых нет во втором)
var except = list1.Except(list2).ToList(); // { 1, 2, 3 }

// Симметричная разность
var symmetricDifference = list1.Except(list2).Union(list2.Except(list1)).ToList();
```

### 6. Преобразования и проекции

```csharp
List<string> words = new List<string> { "привет", "мир", "программирование" };

// Преобразование к другому типу
List<int> wordLengths = words.Select(w => w.Length).ToList();

// Преобразование в анонимный тип
var wordInfo = words.Select(w => new
{
    Word = w,
    Length = w.Length,
    FirstChar = w[0]
}).ToList();

// Flatten (разглаживание) вложенных коллекций
List<List<int>> nestedLists = new List<List<int>>
{
    new List<int> { 1, 2, 3 },
    new List<int> { 4, 5 },
    new List<int> { 6, 7, 8, 9 }
};

List<int> flatList = nestedLists.SelectMany(list => list).ToList();
// Результат: { 1, 2, 3, 4, 5, 6, 7, 8, 9 }
```

### 7. Условная обработка и фильтрация

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Фильтрация по условию
List<int> evenNumbers = numbers.Where(n => n % 2 == 0).ToList();

// Пропуск и взятие элементов
List<int> middle = numbers.Skip(3).Take(4).ToList(); // Элементы с 4 по 7

// Взятие элементов пока условие истинно
List<int> whileLessThan5 = numbers.TakeWhile(n => n < 5).ToList(); // { 1, 2, 3, 4 }

// Пропуск элементов пока условие истинно
List<int> afterSkipping = numbers.SkipWhile(n => n < 5).ToList(); // { 5, 6, 7, 8, 9, 10 }
```

### 8. Работа с индексами

```csharp
List<string> items = new List<string> { "A", "B", "C", "D" };

// Получение элементов с их индексами
var itemsWithIndex = items
    .Select((item, index) => new { Index = index, Value = item })
    .ToList();

// Поиск индексов элементов по условию
var indices = items
    .Select((item, index) => new { Index = index, Value = item })
    .Where(x => x.Value.CompareTo("B") > 0)
    .Select(x => x.Index)
    .ToList();
```

## Практические примеры из реальной работы

### Пример 1: Обработка данных с сервера

```csharp
public class ApiResponse
{
    public List<User> Users { get; set; }
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime LastLogin { get; set; }
    public bool IsActive { get; set; }
}

// Обработка ответа API
public List<User> ProcessApiResponse(ApiResponse response)
{
    return response.Users
        .Where(u => u.IsActive) // Только активные пользователи
        .Where(u => u.LastLogin > DateTime.Now.AddDays(-30)) // Заходили в последний месяц
        .DistinctBy(u => u.Email.ToLower()) // Убираем дубликаты по email
        .OrderBy(u => u.Name) // Сортируем по имени
        .ToList();
}
```

### Пример 2: Пакетная обработка данных

```csharp
public async Task ProcessLargeDataset(List<DataItem> items)
{
    const int batchSize = 100;
    
    // Разбиваем на батчи и обрабатываем
    var batches = items.Chunk(batchSize);
    
    foreach (var batch in batches)
    {
        await ProcessBatchAsync(batch.ToList());
    }
}

public List<T> ProcessInParallel<T>(List<T> items, Func<T, T> processor)
{
    return items.AsParallel().Select(processor).ToList();
}
```

## Оптимизация производительности

### Советы по производительности

1. **Предварительная емкость**: Если знаете примерный размер списка, задайте начальную емкость
```csharp
List<int> numbers = new List<int>(1000); // Избегаем реаллокации
```

2. **Используйте StringBuilder для конкатенации строк**:
```csharp
// Неэффективно
string result = "";
foreach (string item in items)
    result += item;

// Эффективно
StringBuilder sb = new StringBuilder();
foreach (string item in items)
    sb.Append(item);
```

3. **Кеширование результатов LINQ**:
```csharp
// Неэффективно - пересчитывается каждый раз
if (list.Where(x => x.IsValid).Count() > 0 && list.Where(x => x.IsValid).First().Value > 10)

// Эффективно - вычисляется один раз
var validItems = list.Where(x => x.IsValid).ToList();
if (validItems.Count > 0 && validItems.First().Value > 10)
```

## Заключение

Списки в C# предоставляют огромные возможности для работы с коллекциями данных. Комбинируя базовые операции с LINQ и продвинутыми техниками, можно элегантно решать сложные задачи обработки данных.

Помните о производительности при работе с большими объемами данных и выбирайте подходящие структуры данных для каждой конкретной задачи.
