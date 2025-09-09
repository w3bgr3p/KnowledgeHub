# Работа со списками проекта (IZennoList) в ZennoPoster - Методичка

## 🔴 ВАЖНЫЕ ОГРАНИЧЕНИЯ И ОСОБЕННОСТИ

### ⚠️ Списки должны быть созданы заранее
- **Все списки должны быть созданы в ProjectMaker до выполнения кода**
- Попытка доступа к несуществующему списку вызовет исключение
- API `IZennoList` позволяет только работать с существующими списками

### ⚠️ Потокобезопасность
- **Списки НЕ являются потокобезопасными по умолчанию**
- При работе в многопоточном режиме необходимо использовать `lock()`
- Особенно критично при операциях добавления/удаления элементов

### ⚠️ Связь с внешними файлами
- Списки могут быть привязаны к внешним файлам (txt, csv)
- Изменения в коде автоматически сохраняются в файл
- При работе с файловыми списками `lock()` обязателен для предотвращения повреждения файла

## 📋 ОСНОВНЫЕ ОПЕРАЦИИ СО СПИСКАМИ

### Получение списка
```csharp
// Получение существующего списка
IZennoList list = project.Lists["MyList"];

// Безопасное получение с проверкой на null
IZennoList emailList = project.Lists["EmailList"];
if (emailList == null) {
    project.SendErrorToLog("Список EmailList не найден", false);
    throw new Exception("Список EmailList не существует");
}
```

### Добавление элементов
```csharp
// Простое добавление строки
IZennoList userList = project.Lists["Users"];
userList.Add("ivan_petrov");

// Добавление нескольких элементов
userList.Add("maria_sidorova");
userList.Add("alex_kozlov");
userList.Add("elena_volkova");

// Добавление с проверкой дублей - медленнее из-за поиска по всему списку
string newUser = "new_user_123";
if (!userList.Contains(newUser)) {
    userList.Add(newUser);
} else {
    project.SendInfoToLog($"Пользователь {newUser} уже существует", false);
}

// Формирование и добавление составных данных
string userId = "12345";
string email = "user@example.com";
string phone = "+1234567890";
string userData = $"{userId}|{email}|{phone}";
userList.Add(userData);
```

### Получение элементов
```csharp
IZennoList dataList = project.Lists["DataList"];

// Получение элемента по индексу
if (dataList.Count > 0) {
    string firstItem = dataList[0];
    string lastItem = dataList[dataList.Count - 1];
    
    project.SendInfoToLog($"Первый элемент: {firstItem}", false);
    project.SendInfoToLog($"Последний элемент: {lastItem}", false);
}

// Безопасное получение с проверкой границ
int index = 5;
if (index >= 0 && index < dataList.Count) {
    string item = dataList[index];
    project.SendInfoToLog($"Элемент [{index}]: {item}", false);
} else {
    project.SendWarningToLog($"Индекс {index} выходит за границы списка (размер: {dataList.Count})", false);
}

// Получение случайного элемента
if (dataList.Count > 0) {
    int randomIndex = Global.Classes.rnd.Next(0, dataList.Count);
    string randomItem = dataList[randomIndex];
    project.SendInfoToLog($"Случайный элемент: {randomItem}", false);
}
```

### Удаление элементов
```csharp
IZennoList processingList = project.Lists["ProcessingList"];

// Удаление по индексу - быстрее
if (processingList.Count > 0) {
    processingList.RemoveAt(0);  // Удаляем первый элемент
}

// Удаление последнего элемента
if (processingList.Count > 0) {
    processingList.RemoveAt(processingList.Count - 1);
}

// Удаление по значению - медленнее из-за поиска
string itemToRemove = "completed_task_123";
if (processingList.Contains(itemToRemove)) {
    processingList.Remove(itemToRemove);
}

// Безопасное удаление нескольких элементов с конца
int itemsToRemove = 5;
for (int i = 0; i < itemsToRemove && processingList.Count > 0; i++) {
    processingList.RemoveAt(processingList.Count - 1);
}
```

### Очистка списка
```csharp
// Полная очистка списка
IZennoList tempList = project.Lists["TempData"];
tempList.Clear();

project.SendInfoToLog($"Список очищен. Текущий размер: {tempList.Count}", false);
```

## 🔒 ПОТОКОБЕЗОПАСНАЯ РАБОТА СО СПИСКАМИ

### Основные принципы синхронизации
```csharp
// Правильная синхронизация при добавлении - блокировка минимизирует время удержания
IZennoList sharedList = project.Lists["SharedResults"];
string newResult = "processing_result_data";

lock (sharedList) {
    sharedList.Add(newResult);
}

// Неправильно - долгие операции внутри lock блокируют другие потоки
lock (sharedList) {
    Thread.Sleep(5000); // ❌ Плохо - блокируем доступ на 5 секунд
    sharedList.Add("data");
}

// Правильно - подготавливаем данные вне lock
string processedData = ProcessData(); // Долгая операция вне блокировки
lock (sharedList) {
    sharedList.Add(processedData); // ✅ Быстрая операция в блокировке
}
```

### Безопасное чтение из списка
```csharp
IZennoList taskQueue = project.Lists["TaskQueue"];
string currentTask = "";

// Безопасное получение и удаление первого элемента
lock (taskQueue) {
    if (taskQueue.Count > 0) {
        currentTask = taskQueue[0];
        taskQueue.RemoveAt(0);
    }
}

if (!string.IsNullOrEmpty(currentTask)) {
    // Обрабатываем задачу вне блокировки
    ProcessTask(currentTask);
}
```

### Безопасное копирование данных списка
```csharp
// Создание локальной копии для долгой обработки - освобождаем блокировку быстро
IZennoList sourceList = project.Lists["SourceData"];
string[] localCopy;

lock (sourceList) {
    localCopy = new string[sourceList.Count];
    for (int i = 0; i < sourceList.Count; i++) {
        localCopy[i] = sourceList[i];
    }
}

// Обрабатываем копию без блокировки оригинального списка
foreach (string item in localCopy) {
    ProcessItem(item);
}
```

## 📊 ПРАКТИЧЕСКИЕ СЦЕНАРИИ ИСПОЛЬЗОВАНИЯ

### Очередь задач (обычные списки - потокобезопасны)
```csharp
IZennoList highPriorityTasks = project.Lists["HighPriority"];
IZennoList normalTasks = project.Lists["NormalTasks"];

// Добавление задач - lock() не нужен для обычных списков
void AddTask(string task, bool highPriority = false) {
    if (highPriority) {
        highPriorityTasks.Add(task); // ✅ Потокобезопасно без lock()
    } else {
        normalTasks.Add(task); // ✅ Потокобезопасно без lock()
    }
}

// Получение следующей задачи
string GetNextTask() {
    // Сначала проверяем высокоприоритетные задачи
    if (highPriorityTasks.Count > 0) {
        string task = highPriorityTasks[0];
        highPriorityTasks.RemoveAt(0); // ✅ Потокобезопасно
        return task;
    }
    
    // Затем обычные задачи
    if (normalTasks.Count > 0) {
        string task = normalTasks[0];
        normalTasks.RemoveAt(0); // ✅ Потокобезопасно
        return task;
    }
    
    return "";
}

// Использование
AddTask("urgent_user_request", true);
AddTask("routine_cleanup", false);

string nextTask = GetNextTask();
if (!string.IsNullOrEmpty(nextTask)) {
    project.SendInfoToLog($"Выполняю задачу: {nextTask}", false);
}
```

### Система логирования действий (обычный список)
```csharp
IZennoList actionLog = project.Lists["ActionLog"];

// Добавление записи в лог - lock() не нужен
void LogAction(string action, string result = "success") {
    string logEntry = $"{DateTime.Now:yyyy-MM-dd HH:mm:ss}|{action}|{result}";
    
    actionLog.Add(logEntry); // ✅ Потокобезопасно
    
    // Ограничиваем размер лога - удаляем старые записи
    if (actionLog.Count > 1000) {
        actionLog.RemoveAt(0); // ✅ Потокобезопасно
    }
}

// Использование
LogAction("Login attempt", "success");
LogAction("Data processing", "completed");
LogAction("File upload", "failed");

// Поиск ошибок в логе
void CheckForErrors() {
    int errorCount = 0;
    
    // Проходим по списку напрямую - чтение потокобезопасно
    for (int i = 0; i < actionLog.Count; i++) {
        string entry = actionLog[i];
        if (entry.Contains("|failed") || entry.Contains("|error")) {
            errorCount++;
        }
    }
    
    project.SendInfoToLog($"Найдено ошибок в логе: {errorCount}", false);
}
```

### Управление пулом прокси (файловый список - нужен lock)
```csharp
IZennoList availableProxies = project.Lists["AvailableProxies"]; // Привязан к файлу
IZennoList failedProxies = project.Lists["FailedProxies"];       // Обычный список

// Получение рабочего прокси из файлового списка
string GetWorkingProxy() {
    string proxy = "";
    
    // Файловый список - нужен lock()
    lock (availableProxies) {
        if (availableProxies.Count > 0) {
            proxy = availableProxies[0];
            availableProxies.RemoveAt(0);
        }
    }
    
    return proxy;
}

// Возврат прокси обратно в пул или в список неработающих
void ReturnProxy(string proxy, bool isWorking) {
    if (isWorking) {
        // Файловый список - нужен lock()
        lock (availableProxies) {
            availableProxies.Add(proxy);
        }
    } else {
        // Обычный список - lock() не нужен
        if (!failedProxies.Contains(proxy)) { // Проверка дублей - медленнее
            failedProxies.Add(proxy);
        }
    }
}
```

### Система накопления результатов (обычные списки)
```csharp
IZennoList collectedData = project.Lists["CollectedData"];
IZennoList processedBatch = project.Lists["ProcessedBatch"];

// Накопление данных - lock() не нужен
void CollectData(string data) {
    collectedData.Add(data); // ✅ Потокобезопасно
    
    // Если накопилось достаточно - обрабатываем батч
    if (collectedData.Count >= 100) {
        ProcessBatch();
    }
}

// Обработка батча
void ProcessBatch() {
    // Очищаем предыдущий батч
    processedBatch.Clear(); // ✅ Потокобезопасно
    
    // Перемещаем все элементы
    for (int i = 0; i < collectedData.Count; i++) {
        processedBatch.Add(collectedData[i]); // ✅ Потокобезопасно
    }
    collectedData.Clear(); // ✅ Потокобезопасно
    
    project.SendInfoToLog($"Сформирован батч из {processedBatch.Count} элементов", false);
    
    SaveBatchToFile();
}

void SaveBatchToFile() {
    string fileName = $"batch_{DateTime.Now:yyyyMMdd_HHmmss}.txt";
    string filePath = Path.Combine(project.Directory, fileName);
    
    // Копируем данные батча для сохранения
    string[] batchData = new string[processedBatch.Count];
    for (int i = 0; i < processedBatch.Count; i++) {
        batchData[i] = processedBatch[i]; // ✅ Чтение потокобезопасно
    }
    
    // Сохраняем в файл
    File.WriteAllLines(filePath, batchData);
    project.SendInfoToLog($"Батч сохранен в файл: {fileName}", false);
}
```

## 🔄 РАБОТА С СОСТАВНЫМИ ДАННЫМИ В СПИСКАХ

### Хранение структурированных данных
```csharp
IZennoList userProfiles = project.Lists["UserProfiles"];

// Добавление профиля пользователя как строки с разделителями
void AddUserProfile(string userId, string email, string name, string status) {
    string profile = $"{userId}|{email}|{name}|{status}|{DateTime.Now:yyyy-MM-dd}";
    
    lock (userProfiles) {
        userProfiles.Add(profile);
    }
}

// Получение и парсинг профиля
class UserProfile {
    public string UserId;
    public string Email;
    public string Name;
    public string Status;
    public string CreatedDate;
}

UserProfile GetUserProfile(int index) {
    string profileData = "";
    
    lock (userProfiles) {
        if (index >= 0 && index < userProfiles.Count) {
            profileData = userProfiles[index];
        }
    }
    
    if (string.IsNullOrEmpty(profileData)) {
        return null;
    }
    
    string[] parts = profileData.Split('|');
    if (parts.Length >= 5) {
        return new UserProfile {
            UserId = parts[0],
            Email = parts[1],
            Name = parts[2],
            Status = parts[3],
            CreatedDate = parts[4]
        };
    }
    
    return null