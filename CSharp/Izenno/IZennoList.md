# Работа со списками проекта (IZennoList) в ZennoPoster

## 🔴 ВАЖНЫЕ ОГРАНИЧЕНИЯ И ОСОБЕННОСТИ

### ⚠️ Списки должны быть созданы заранее
- Все списки должны быть созданы в ProjectMaker до выполнения кода
- Попытка доступа к несуществующему списку вызовет исключение

### ⚠️ Связь с внешними файлами
- Списки могут быть привязаны к внешним файлам (txt, csv)
- Изменения в коде автоматически сохраняются в файл
- При работе с файловыми списками `lock()` обязателен для предотвращения повреждения файла

### ⚠️ Потокобезопасность
- **Обычные списки** (созданные в ProjectMaker без привязки к файлу) - потокобезопасны, `lock()` не нужен
- **Файловые списки** (привязанные через Bind к внешнему файлу) - требуют `lock()` для всех операций

## 📋 ОСНОВНЫЕ СВОЙСТВА И МЕТОДЫ

### Свойства
```csharp
IZennoList list = project.Lists["MyList"];

// Получение количества элементов
int count = list.Count;
project.SendInfoToLog($"В списке {count} элементов", false);

// Проверка на только для чтения
bool isReadOnly = list.IsReadOnly;

// Получение элемента по индексу
string item = list[0];
string lastItem = list[list.Count - 1];

// Установка элемента по индексу
list[0] = "новое значение";
```

### Добавление элементов

#### Add - добавление одного элемента
```csharp
IZennoList userList = project.Lists["Users"];

// Простое добавление
userList.Add("ivan_petrov");
userList.Add("maria_sidorova");

// Добавление с формированием данных
string userId = "12345";
string email = "user@example.com";
string userData = $"{userId}|{email}";
userList.Add(userData);
```

#### AddRange - добавление нескольких элементов
```csharp
IZennoList targetList = project.Lists["Target"];

// Из обычного списка
List<string> tempList = new List<string>();
tempList.Add("элемент1");
tempList.Add("элемент2");
tempList.Add("элемент3");
targetList.AddRange(tempList);

// Из другого IZennoList
IZennoList sourceList = project.Lists["Source"];
targetList.AddRange(sourceList);
```

### Получение элементов

#### Прямой доступ по индексу
```csharp
IZennoList list = project.Lists["MyList"];

// Получение элемента
if (list.Count > 0) {
    string first = list[0];
    string last = list[list.Count - 1];
    
    project.SendInfoToLog($"Первый: {first}", false);
    project.SendInfoToLog($"Последний: {last}", false);
}

// Безопасное получение с проверкой
int index = 5;
if (index >= 0 && index < list.Count) {
    string item = list[index];
    project.SendInfoToLog($"Элемент [{index}]: {item}", false);
}
```

#### GetItem - получение с удалением
```csharp
IZennoList queue = project.Lists["Queue"];

// Получение первого элемента без удаления
string item = queue.GetItem("0", false);

// Получение первого элемента с удалением
string nextItem = queue.GetItem("0", true);

// Получение случайного элемента с удалением
string randomItem = queue.GetItem("random", true);

// Проверка результата
if (!string.IsNullOrEmpty(nextItem)) {
    project.SendInfoToLog($"Обработка: {nextItem}", false);
}
```

#### GetItems - получение нескольких элементов
```csharp
IZennoList batch = project.Lists["Batch"];

// Получение диапазона элементов (0, 1, 2)
string[] items = batch.GetItems("0-2", false);

// Получение первых 5 элементов с удалением
string[] firstFive = batch.GetItems("0-4", true);

// Получение 3 случайных элементов
string[] randomThree = batch.GetItems("3", true);

// Обработка результата
foreach (string item in items) {
    project.SendInfoToLog($"Элемент: {item}", false);
}
```

### Удаление элементов

#### RemoveAt - удаление по индексу
```csharp
IZennoList processing = project.Lists["Processing"];

// Удаление первого элемента
if (processing.Count > 0) {
    processing.RemoveAt(0);
}

// Удаление последнего элемента
if (processing.Count > 0) {
    processing.RemoveAt(processing.Count - 1);
}

// Удаление нескольких элементов с конца
int toRemove = 5;
for (int i = 0; i < toRemove && processing.Count > 0; i++) {
    processing.RemoveAt(processing.Count - 1);
}
```

#### Remove - удаление по значению
```csharp
IZennoList items = project.Lists["Items"];

// Удаление конкретного значения
string valueToRemove = "completed_task";
if (items.Contains(valueToRemove)) {
    items.Remove(valueToRemove);
}
```

#### Clear - полная очистка
```csharp
IZennoList temp = project.Lists["Temp"];
temp.Clear();
project.SendInfoToLog($"Список очищен. Размер: {temp.Count}", false);
```

### Поиск и проверка

#### Contains - проверка наличия элемента
```csharp
IZennoList users = project.Lists["Users"];

string newUser = "user_123";
if (!users.Contains(newUser)) {
    users.Add(newUser);
} else {
    project.SendInfoToLog($"Пользователь {newUser} уже существует", false);
}
```

#### IndexOf - поиск индекса элемента
```csharp
IZennoList items = project.Lists["Items"];

string searchItem = "target_value";
int index = items.IndexOf(searchItem);

if (index >= 0) {
    project.SendInfoToLog($"Найден на позиции: {index}", false);
} else {
    project.SendInfoToLog("Элемент не найден", false);
}
```

### Insert - вставка по индексу
```csharp
IZennoList list = project.Lists["MyList"];

// Вставка в начало
list.Insert(0, "новый_первый");

// Вставка в середину
if (list.Count >= 5) {
    list.Insert(5, "вставка_в_середину");
}
```

### CopyTo - копирование в массив
```csharp
IZennoList source = project.Lists["Source"];

// Создание массива нужного размера
string[] array = new string[source.Count];

// Копирование всех элементов
source.CopyTo(array, 0);

// Использование скопированных данных
foreach (string item in array) {
    project.SendInfoToLog(item, false);
}
```

### Bind - привязка к файлу
```csharp
IZennoList list = project.Lists["FileList"];

// Привязка к файлу
string filePath = Path.Combine(project.Directory, "data.txt");
list.Bind(filePath);

// После привязки все операции требуют lock()
lock (list) {
    list.Add("новая_строка");
}

project.SendInfoToLog($"Список привязан к {filePath}", false);
```

## 🔒 ПОТОКОБЕЗОПАСНАЯ РАБОТА

### Когда нужен lock()

#### Файловые списки - ВСЕГДА используйте lock()
```csharp
IZennoList fileList = project.Lists["FileList"];
string filePath = Path.Combine(project.Directory, "data.txt");
fileList.Bind(filePath);

// Все операции с файловым списком
lock (fileList) {
    fileList.Add("данные");
    string item = fileList[0];
    fileList.RemoveAt(0);
}
```

#### Обычные списки - lock() НЕ нужен
```csharp
IZennoList normalList = project.Lists["NormalList"];

// Операции без lock() - потокобезопасно
normalList.Add("элемент");
string item = normalList[0];
normalList.RemoveAt(0);
```

### Правильная синхронизация
```csharp
IZennoList fileList = project.Lists["FileList"];

// Подготовка данных ВНЕ блокировки
string processedData = ProcessData();

// Быстрая операция В блокировке
lock (fileList) {
    fileList.Add(processedData);
}

// Неправильно - долгая операция в lock
lock (fileList) {
    Thread.Sleep(5000); // ❌ Блокирует другие потоки
    fileList.Add("data");
}
```

## 📊 ПРАКТИЧЕСКИЕ ПРИМЕРЫ

### Очередь задач (обычный список)
```csharp
IZennoList tasks = project.Lists["Tasks"];

// Добавление задачи
tasks.Add("новая_задача");

// Получение следующей задачи
string nextTask = "";
if (tasks.Count > 0) {
    nextTask = tasks[0];
    tasks.RemoveAt(0);
}

if (!string.IsNullOrEmpty(nextTask)) {
    project.SendInfoToLog($"Выполняю: {nextTask}", false);
}
```

### Пул прокси (файловый список)
```csharp
IZennoList proxies = project.Lists["Proxies"];
string proxyFile = Path.Combine(project.Directory, "proxies.txt");
proxies.Bind(proxyFile);

// Получение прокси
string proxy = "";
lock (proxies) {
    if (proxies.Count > 0) {
        proxy = proxies[0];
        proxies.RemoveAt(0);
    }
}

// Использование прокси
if (!string.IsNullOrEmpty(proxy)) {
    // Работа с прокси
}

// Возврат прокси
lock (proxies) {
    proxies.Add(proxy);
}
```

### Накопление результатов (обычный список)
```csharp
IZennoList results = project.Lists["Results"];

// Добавление результата
string result = "обработанные_данные";
results.Add(result);

// Обработка батча при достижении размера
if (results.Count >= 100) {
    // Копирование для обработки
    string[] batch = new string[results.Count];
    results.CopyTo(batch, 0);
    
    // Очистка основного списка
    results.Clear();
    
    // Обработка батча
    foreach (string item in batch) {
        // Обработка
    }
}
```

### Система логирования (обычный список)
```csharp
IZennoList log = project.Lists["ActionLog"];

// Добавление записи
string logEntry = $"{DateTime.Now:yyyy-MM-dd HH:mm:ss}|login|success";
log.Add(logEntry);

// Ограничение размера лога
if (log.Count > 1000) {
    log.RemoveAt(0);
}

// Поиск ошибок
int errorCount = 0;
for (int i = 0; i < log.Count; i++) {
    if (log[i].Contains("|error") || log[i].Contains("|failed")) {
        errorCount++;
    }
}
project.SendInfoToLog($"Найдено ошибок: {errorCount}", false);
```

### Работа со структурированными данными
```csharp
IZennoList users = project.Lists["Users"];

// Добавление профиля
string userId = "123";
string email = "user@mail.com";
string name = "Иван";
string profile = $"{userId}|{email}|{name}";
users.Add(profile);

// Получение и парсинг
string userData = users[0];
string[] parts = userData.Split('|');
if (parts.Length >= 3) {
    string id = parts[0];
    string userEmail = parts[1];
    string userName = parts[2];
    
    project.SendInfoToLog($"ID: {id}, Email: {userEmail}, Имя: {userName}", false);
}
```

### Перебор всех элементов
```csharp
IZennoList items = project.Lists["Items"];

// Через цикл for
for (int i = 0; i < items.Count; i++) {
    string item = items[i];
    project.SendInfoToLog($"[{i}]: {item}", false);
}

// Через GetEnumerator
foreach (string item in items) {
    project.SendInfoToLog(item, false);
}
```

## 💡 СОВЕТЫ И РЕКОМЕНДАЦИИ

### Производительность
- `RemoveAt(index)` быстрее чем `Remove(value)` - не требует поиска
- `Contains()` медленный для больших списков - проходит весь список
- Для сложных операций копируйте в `List<string>` и работайте с ним

### Безопасность
- Всегда проверяйте `Count` перед доступом по индексу
- Используйте `lock()` для файловых списков
- Проверяйте результат `GetItem()` и `GetItems()` на пустоту

### Типичные ошибки
```csharp
// ❌ Неправильно - нет проверки
string item = list[0];

// ✅ Правильно
if (list.Count > 0) {
    string item = list[0];
}

// ❌ Неправильно - изменение во время перебора
foreach (string item in list) {
    list.Remove(item); // Исключение!
}

// ✅ Правильно - копирование или обратный проход
for (int i = list.Count - 1; i >= 0; i--) {
    list.RemoveAt(i);
}
```