# Работа со списками в ZennoPoster

## Введение
Списки в ZennoPoster используются для хранения и обработки наборов строковых данных. Они доступны через объект `project.Lists` и представляют собой коллекции типа `IZennoList`. Каждый список должен быть создан заранее в интерфейсе ProjectMaker, так как API `IZennoPosterProjectModel` не позволяет создавать списки динамически в коде. Списки глобальны для проекта и могут использоваться в действиях, кубиках или C#-коде.

**Важные правила:**
- Списки хранят данные только в виде строк (`string`).
- Имена списков чувствительны к регистру.
- Попытка доступа к несуществующему списку вызовет исключение `System.NullReferenceException`.
- API предоставляет ограниченный набор методов для работы со списками: добавление, удаление, очистка. Для сложной обработки используйте стандартные методы .NET.
- Для потокобезопасной работы с файлами списков (если список привязан к файлу) используйте `lock()`.

## Доступ к спискам
Списки доступны через коллекцию `project.Lists`. Каждый список — это объект `IZennoList` с методами для управления элементами.

### Проверка существования списка
Перед обращением к списку проверяйте его существование:

```csharp
if (project.Lists.Contains("имя_списка")) {
    project.SendInfoToLog("Список 'имя_списка' существует", false);
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

### Получение списка
Для работы со списком:

```csharp
IZennoList list = project.Lists["имя_списка"];
```

- Если список не существует, будет выброшено исключение `System.NullReferenceException`.

## Основные методы работы со списками
ZennoPoster предоставляет ограниченный набор методов для работы со списками. Все они работают только с существующими списками.

### Добавление элемента
Добавляет строку в конец списка:

```csharp
if (project.Lists.Contains("имя_списка")) {
    IZennoList list = project.Lists["имя_списка"];
    list.Add("новый элемент");
    project.SendInfoToLog("Элемент добавлен", false);
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

### Удаление элемента
Удаляет элемент по индексу:

```csharp
if (project.Lists.Contains("имя_списка")) {
    IZennoList list = project.Lists["имя_списка"];
    if (list.Count > 0) {
        list.RemoveAt(0); // Удаляет первый элемент
        project.SendInfoToLog("Элемент удалён", false);
    } else {
        project.SendWarningToLog("Список пуст", false);
    }
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

### Очистка списка
Удаляет все элементы из списка:

```csharp
if (project.Lists.Contains("имя_списка")) {
    IZennoList list = project.Lists["имя_списка"];
    list.Clear();
    project.SendInfoToLog("Список очищен", false);
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

### Получение количества элементов
Проверяет количество элементов в списке:

```csharp
if (project.Lists.Contains("имя_списка")) {
    IZennoList list = project.Lists["имя_списка"];
    int count = list.Count;
    project.SendInfoToLog($"В списке {count} элементов", false);
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

## Работа с файлами списков
Если список привязан к файлу (настроено в ProjectMaker), изменения автоматически сохраняются в файл. Для потокобезопасной работы используйте `lock`:

```csharp
if (project.Lists.Contains("имя_списка")) {
    IZennoList list = project.Lists["имя_списка"];
    lock (list) {
        list.Add("новый элемент");
        project.SendInfoToLog("Элемент добавлен в список с блокировкой", false);
    }
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

## Обработка списков
Для сложной обработки (фильтрация, сортировка, поиск) преобразуйте список в массив .NET:

```csharp
if (project.Lists.Contains("имя_списка")) {
    IZennoList list = project.Lists["имя_списка"];
    string[] array = list.ToArray();
    // Пример: сортировка
    Array.Sort(array);
    // Очистка и обновление списка
    lock (list) {
        list.Clear();
        foreach (string item in array) {
            list.Add(item);
        }
    }
    project.SendInfoToLog("Список отсортирован", false);
} else {
    project.SendErrorToLog("Список 'имя_списка' не существует", false);
}
```

## Обработка ошибок
Всегда используйте `try-catch` для обработки исключений:

```csharp
try {
    IZennoList list = project.Lists["имя_списка"];
    list.Add("элемент");
    project.SendInfoToLog("Элемент добавлен", false);
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

## Примеры использования

### Пример 1: Добавление и чтение
```csharp
try {
    if (!project.Lists.Contains("логины")) {
        throw new Exception("Список 'логины' не существует");
    }
    IZennoList list = project.Lists["логины"];
    lock (list) {
        list.Add("user123");
        string firstItem = list.Count > 0 ? list[0] : "пусто";
        project.SendInfoToLog($"Первый элемент: {firstItem}", false);
    }
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

### Пример 2: Фильтрация списка
```csharp
try {
    if (!project.Lists.Contains("emails")) {
        throw new Exception("Список 'emails' не существует");
    }
    IZennoList list = project.Lists["emails"];
    string[] emails = list.ToArray();
    lock (list) {
        list.Clear();
        foreach (string email in emails) {
            if (email.Contains("@gmail.com")) {
                list.Add(email);
            }
        }
    }
    project.SendInfoToLog("Список отфильтрован по @gmail.com", false);
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

### Пример 3: Удаление дубликатов
```csharp
try {
    if (!project.Lists.Contains("данные")) {
        throw new Exception("Список 'данные' не существует");
    }
    IZennoList list = project.Lists["данные"];
    string[] items = list.ToArray();
    string[] uniqueItems = items.Distinct().ToArray();
    lock (list) {
        list.Clear();
        foreach (string item in uniqueItems) {
            list.Add(item);
        }
    }
    project.SendInfoToLog("Дубликаты удалены", false);
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

### Пример 4: Случайный элемент
```csharp
try {
    if (!project.Lists.Contains("имена")) {
        throw new Exception("Список 'имена' не существует");
    }
    IZennoList list = project.Lists["имена"];
    if (list.Count > 0) {
        int index = Global.Classes.rnd.Next(0, list.Count);
        string randomItem = list[index];
        project.SendInfoToLog($"Случайное имя: {randomItem}", false);
    } else {
        project.SendWarningToLog("Список пуст", false);
    }
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```


## Рекомендации
- Создавайте списки заранее в интерфейсе ProjectMaker.
- Используйте `lock()` для списков, привязанных к файлам.

