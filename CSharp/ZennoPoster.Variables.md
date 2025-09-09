# Работа с переменными в ZennoPoster

## Введение
В ZennoPoster переменные используются для хранения и манипуляции данными в проектах. Все переменные проекта доступны через объект `project.Variables` и хранятся в виде строк (тип `string`). Если нужно работать с другими типами данных (например, числами или датами), значение следует конвертировать вручную. Переменные создаются в интерфейсе ProjectMaker или в настройках проекта, а в коде доступны только для чтения и изменения.

Переменные проекта глобальны для всего проекта и могут использоваться в действиях, кубиках или C#-коде. 

**Важные правила:**
- Имена переменных чувствительны к регистру.
- Переменные должны быть созданы заранее в ProjectMaker. Попытка доступа к несуществующей переменной через `project.Variables["имя"].Value` вызовет исключение `System.NullReferenceException`.
- API `IZennoPosterProjectModel` предоставляет доступ только к чтению и изменению существующих локальных переменных, но не к их созданию.
- Для безопасности конфиденциальных данных (например, паролей) используйте переменные проекта, а не жёсткий код.
- Для списков используйте `project.Lists` или `project.Tables`.

## Доступ к переменным
Переменные доступны через коллекцию `project.Variables`. Каждый элемент — это объект `ILocalVariable` с свойством `Value` для чтения/записи.

### Проверка существования переменной
Перед обращением к переменной проверяйте её существование, чтобы избежать исключения:

```csharp
if (project.Variables.Contains("имя_переменной")) {
    string value = project.Variables["имя_переменной"].Value;
    project.SendInfoToLog($"Значение переменной: {value}", false);
} else {
    project.SendErrorToLog("Переменная 'имя_переменной' не существует", false);
}
```

### Получение значения переменной
Если переменная существует, её значение можно получить:

```csharp
string value = project.Variables["имя_переменной"].Value;
```

- Если переменная существует, но пуста, `value` будет равно пустой строке (`""`).
- Если переменная не существует, будет выброшено исключение `System.NullReferenceException`.

### Установка значения переменной
Для изменения значения существующей переменной:

```csharp
if (project.Variables.Contains("имя_переменной")) {
    project.Variables["имя_переменной"].Value = "новое значение";
} else {
    project.SendErrorToLog("Переменная 'имя_переменной' не существует и не может быть создана", false);
}
```

- **Важно:** Нельзя создать новую переменную через `project.Variables["имя"].Value`. Переменная должна быть предварительно создана в интерфейсе ProjectMaker.
- Значение всегда преобразуется в строку. Для чисел используйте `ToString()`:

```csharp
if (project.Variables.Contains("число")) {
    int number = 42;
    project.Variables["число"].Value = number.ToString();
} else {
    project.SendErrorToLog("Переменная 'число' не существует", false);
}
```

Пример: Изменение значения переменной "username":

```csharp
if (project.Variables.Contains("username")) {
    project.Variables["username"].Value = "student123";
    project.SendInfoToLog("Значение переменной 'username' обновлено", false);
} else {
    project.SendErrorToLog("Переменная 'username' не существует", false);
}
```

## Конвертация типов данных
Поскольку переменные хранят строки, для работы с другими типами используйте методы .NET Framework 4.0.

### Конвертация строки в число
```csharp
if (project.Variables.Contains("число")) {
    string strValue = project.Variables["число"].Value; // "42"
    int intValue = int.Parse(strValue); // 42
} else {
    project.SendErrorToLog("Переменная 'число' не существует", false);
}
```

### Конвертация числа в строку
```csharp
if (project.Variables.Contains("сумма")) {
    int num = 100;
    project.Variables["сумма"].Value = num.ToString(); // "100"
} else {
    project.SendErrorToLog("Переменная 'сумма' не существует", false);
}
```

### Конвертация в дату
```csharp
if (project.Variables.Contains("дата")) {
    string dateStr = project.Variables["дата"].Value; // "2023-09-08"
    DateTime date = DateTime.Parse(dateStr);
} else {
    project.SendErrorToLog("Переменная 'дата' не существует", false);
}
```

### Проверка на пустоту
```csharp
if (project.Variables.Contains("var") && !string.IsNullOrEmpty(project.Variables["var"].Value)) {
    project.SendInfoToLog($"Значение переменной: {project.Variables["var"].Value}", false);
} else {
    project.SendWarningToLog("Переменная 'var' не существует или пуста", false);
}
```

## Конфиденциальные данные
Для паролей или ключей используйте переменные проекта, созданные в ProjectMaker. ZennoPoster автоматически шифрует такие данные.

Пример:
```csharp
if (project.Variables.Contains("api_key")) {
    string apiKey = project.Variables["api_key"].Value;
    project.SendInfoToLog("API-ключ получен", false);
} else {
    project.SendErrorToLog("API-ключ не найден", false);
}
```

## Логирование и обработка ошибок
Для отладки используйте логи:

```csharp
if (project.Variables.Contains("var")) {
    project.SendInfoToLog($"Значение переменной: {project.Variables["var"].Value}", false);
} else {
    project.SendErrorToLog("Переменная 'var' не существует", false);
}
```

Обработка исключений:

```csharp
try {
    string value = project.Variables["var"].Value;
    project.SendInfoToLog($"Значение: {value}", false);
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

## Примеры использования

### Пример 1: Проверка и чтение
```csharp
if (project.Variables.Contains("город")) {
    string city = project.Variables["город"].Value;
    project.SendInfoToLog($"Город: {city}", false);
} else {
    project.SendErrorToLog("Переменная 'город' не существует", false);
}
```

### Пример 2: Конвертация и расчёт
```csharp
try {
    if (!project.Variables.Contains("число1") || !project.Variables.Contains("число2")) {
        throw new Exception("Одна из переменных 'число1' или 'число2' не существует");
    }
    int num1 = int.Parse(project.Variables["число1"].Value);
    int num2 = int.Parse(project.Variables["число2"].Value);
    int sum = num1 + num2;
    if (project.Variables.Contains("сумма")) {
        project.Variables["сумма"].Value = sum.ToString();
        project.SendInfoToLog($"Сумма: {sum}", false);
    } else {
        project.SendErrorToLog("Переменная 'сумма' не существует", false);
    }
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

### Пример 3: Условная проверка
```csharp
try {
    if (!project.Variables.Contains("возраст")) {
        throw new Exception("Переменная 'возраст' не существует");
    }
    int age = int.Parse(project.Variables["возраст"].Value);
    if (age >= 18) {
        project.SendInfoToLog("Доступ разрешён", false);
    } else {
        project.SendErrorToLog("Доступ запрещён", false);
    }
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

### Пример 4: Конкатенация текста
```csharp
try {
    if (!project.Variables.Contains("имя") || !project.Variables.Contains("фамилия")) {
        throw new Exception("Переменные 'имя' или 'фамилия' не существуют");
    }
    string fullName = project.Variables["имя"].Value + " " + project.Variables["фамилия"].Value;
    if (project.Variables.Contains("полное_имя")) {
        project.Variables["полное_имя"].Value = fullName;
        project.SendInfoToLog($"Полное имя: {fullName}", false);
    } else {
        project.SendErrorToLog("Переменная 'полное_имя' не существует", false);
    }
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

## Задания для студентов
1. Проверьте существование переменной "логин". Если она существует, выведите её значение в лог, иначе выведите ошибку.
2. Сложите два числа из переменных "число1" и "число2", сохраните результат в переменную "сумма" (если она существует). Обработайте исключения.
3. Проверьте, существует ли переменная "пароль" и не пуста ли она. Если нет — выведите ошибку в лог.

## Рекомендации
- Создавайте все переменные заранее в интерфейсе ProjectMaker.
- Всегда проверяйте существование переменной через `project.Variables.Contains()`.
- Используйте `try-catch` для обработки исключений.
- Для списков используйте `project.Lists["имя_списка"]`.
- Тестируйте код в отладчике ZennoPoster.

Этот конспект основан на документации ZennoPoster 7.x и исправляет ошибочные утверждения о создании переменных. Источники:,,.[](https://help.zennolab.com/en/v7/zennoposter/7.1.4/topic1238.html)[](https://help.zennolab.com/en/v7/zennoposter/7.1.4/topic1201.html)[](https://help.zennolab.com/en/v7/zennoposter/7.1.4/topic1202.html)