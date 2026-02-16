

### Методическое пособие по работе с IZennoList в ZennoPoster
`IZennoList` используется для работы со списками в ZennoPoster, которые представляют собой коллекции строк, синхронизированные с файлами проекта. Все элементы списков — строки, и для потокобезопасности в многопоточных сценариях рекомендуется использовать `lock()`.

**Важные замечания:**
- `IZennoList` поддерживает ограниченный набор методов: `Add`, `AddRange`, `RemoveAt`, `Clear`, `Bind`, `GetItem`, `GetItems`. Дополнительные операции (например, сортировка, поиск) выполняйте через временные списки .NET (`List<string>`).
- Списки проекта привязаны к файлам, поэтому операции добавления/удаления требуют осторожности в многопоточных сценариях.
- Все значения в `IZennoList` — строки. Нестроковые данные конвертируются в строку.
- Логирование ошибок и действий используйте только при необходимости: `project.SendInfoToLog`, `project.SendWarningToLog`, `project.SendErrorToLog`.

#### 1. Получение объекта IZennoList
Списки проекта доступны через коллекцию `project.Lists`.

```csharp
IZennoList list = project.Lists["ИмяСписка"];
```

- **Описание:** Возвращает объект `IZennoList` для указанного имени списка. Если список не существует, ZennoPoster может создать его автоматически.
- **Пример:**
  ```csharp
  IZennoList myList = project.Lists["MyList"];
  ```
- **Потокобезопасность:** Используйте `lock(list)` для операций в многопоточной среде.
- **Ошибки:** Если список недоступен, оберните в `try-catch`:
  ```csharp
  try {
      IZennoList list = project.Lists["MyList"];
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка получения списка: {ex.Message}", false);
  }
  ```

#### 2. Методы IZennoList
Ниже описаны все методы интерфейса `IZennoList`, найденные в предоставленных сигнатурах.

##### 2.1. Add (Добавление элемента)
Добавляет элемент в конец списка.

```csharp
list.Add("новый элемент");
```

- **Параметры:** Элемент (тип: `object`, но ожидается строка или конвертируемое в строку значение).
- **Описание:** Добавляет элемент в конец списка. Нестроковые данные автоматически преобразуются в строку. Изменения сохраняются в файл списка.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  list.Add("Элемент1");
  list.Add(123); // Конвертируется в "123"
  list.Add(project.Variables["VarName"].Value);
  ```
- **Потокобезопасность:**
  ```csharp
  lock(list) {
      list.Add("Элемент");
  }
  ```
- **Ошибки:** Если файл списка заблокирован, может выбросить исключение:
  ```csharp
  try {
      list.Add("Элемент");
      project.SendInfoToLog("Элемент добавлен", false);
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка добавления: {ex.Message}", false);
  }
  ```

##### 2.2. AddRange (Добавление коллекции элементов)
Добавляет элементы из коллекции в список. Есть два варианта метода.

###### 2.2.1. AddRange(IEnumerable<string>)
Добавляет элементы из коллекции .NET, реализующей `IEnumerable<string>`.

```csharp
List<string> tempList = new List<string> { "Элемент1", "Элемент2" };
list.AddRange(tempList);
```

- **Параметры:** Коллекция строк (тип: `IEnumerable<string>`).
- **Описание:** Добавляет все элементы из коллекции в конец списка. Элементы должны быть строками.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  List<string> tempList = new List<string> { "Элемент1", "Элемент2", "Элемент3" };
  lock(list) {
      list.AddRange(tempList);
      project.SendInfoToLog($"Добавлено {tempList.Count} элементов", false);
  }
  ```
- **Ошибки:** Если коллекция пустая или файл заблокирован, может выбросить исключение:
  ```csharp
  try {
      List<string> tempList = new List<string> { "Элемент1", "Элемент2" };
      lock(list) {
          list.AddRange(tempList);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка AddRange: {ex.Message}", false);
  }
  ```

###### 2.2.2. AddRange(IZennoList)
Добавляет все элементы из другого `IZennoList`.

```csharp
IZennoList sourceList = project.Lists["SourceList"];
list.AddRange(sourceList);
```

- **Параметры:** Другой список `IZennoList`.
- **Описание:** Копирует все элементы из исходного `IZennoList` в целевой список.
- **Пример:**
  ```csharp
  IZennoList sourceList = project.Lists["SourceList"];
  IZennoList targetList = project.Lists["TargetList"];
  lock(targetList) {
      targetList.AddRange(sourceList);
      project.SendInfoToLog("Элементы скопированы из SourceList", false);
  }
  ```
- **Потокобезопасность:** Используйте `lock` для обоих списков:
  ```csharp
  lock(targetList) {
      lock(sourceList) {
          targetList.AddRange(sourceList);
      }
  }
  ```
- **Ошибки:** Проверяйте доступность списков:
  ```csharp
  try {
      lock(targetList) {
          targetList.AddRange(sourceList);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка копирования списка: {ex.Message}", false);
  }
  ```

##### 2.3. RemoveAt (Удаление элемента по индексу)
Удаляет элемент по указанному индексу.

```csharp
list.RemoveAt(0);
```

- **Параметры:** Индекс элемента (тип: `int`, начиная с 0).
- **Описание:** Удаляет элемент по индексу. Изменения сохраняются в файл списка.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  try {
      if (list.Count > 0) { // Проверка (если Count доступен)
          lock(list) {
              list.RemoveAt(0);
              project.SendInfoToLog("Первый элемент удален", false);
          }
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка удаления: {ex.Message}", false);
  }
  ```
- **Ошибки:** Проверяйте индекс:
  ```csharp
  if (list.Count <= index) {
      project.SendWarningToLog("Недопустимый индекс", false);
  }
  ```

##### 2.4. Clear (Очистка списка)
Очищает весь список.

```csharp
list.Clear();
```

- **Параметры:** Нет.
- **Описание:** Удаляет все элементы из списка. Файл списка очищается.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  try {
      lock(list) {
          list.Clear();
          project.SendInfoToLog("Список очищен", false);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка очистки: {ex.Message}", false);
  }
  ```

##### 2.5. Bind (Привязка списка к файлу)
Привязывает список к текстовому файлу.

```csharp
list.Bind("C:\\mylist.txt");
```

- **Параметры:** Путь к файлу (тип: `string`).
- **Описание:** Связывает список с файлом. Содержимое файла загружается в список, а изменения списка синхронизируются с файлом.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  string filePath = Path.Combine(project.Directory, "mylist.txt");
  try {
      lock(list) {
          list.Bind(filePath);
          project.SendInfoToLog($"Список привязан к {filePath}", false);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка привязки: {ex.Message}", false);
  }
  ```
- **Ошибки:** Проверяйте доступность файла:
  ```csharp
  if (!File.Exists(filePath)) {
      project.SendWarningToLog("Файл не существует, будет создан", false);
  }
  ```

##### 2.6. GetItem (Получение элемента)
Получает элемент по указанному индексу или случайный элемент.

```csharp
string item = list.GetItem("0", false); // Получить первый элемент
string randomItem = list.GetItem("random", true); // Получить случайный элемент и удалить
```

- **Параметры:**
  - `indexOrRandom`: Индекс (строка, например, `"0"`) или `"random"` для случайного элемента.
  - `remove`: Если `true`, элемент удаляется после получения.
- **Описание:** Возвращает строку из списка. Может удалить элемент при получении.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  try {
      lock(list) {
          string item = list.GetItem("0", false);
          if (!string.IsNullOrEmpty(item)) {
              project.SendInfoToLog($"Получен элемент: {item}", false);
          }
          string randomItem = list.GetItem("random", true);
          if (!string.IsNullOrEmpty(randomItem)) {
              project.SendInfoToLog($"Случайный элемент: {randomItem}", false);
          }
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка получения элемента: {ex.Message}", false);
  }
  ```
- **Ошибки:** Если список пуст или индекс неверный, возвращается пустая строка или выбросится исключение.

##### 2.7. GetItems (Получение нескольких элементов)
Получает несколько элементов по диапазону или случайным образом.

```csharp
string[] items = list.GetItems("0-2", false); // Получить первые три элемента
string[] randomItems = list.GetItems("3", true); // Получить три случайных элемента с удалением
```

- **Параметры:**
  - `rangeOrCount`: Диапазон (например, `"0-2"`) или количество случайных элементов (например, `"3"`).
  - `remove`: Если `true`, элементы удаляются после получения.
- **Описание:** Возвращает массив строк из списка. Поддерживает диапазоны или случайный выбор.
- **Пример:**
  ```csharp
  IZennoList list = project.Lists["MyList"];
  try {
      lock(list) {
          string[] items = list.GetItems("0-2", false);
          project.SendInfoToLog($"Получено {items.Length} элементов", false);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка получения элементов: {ex.Message}", false);
  }
  ```
- **Ошибки:** Проверяйте диапазон и наличие элементов.

#### 3. Связанные методы из других классов

##### 3.1. Macros.TextProcessing.ToList
Конвертирует строку в список.

```csharp
Macros.TextProcessing.ToList(source, rowSplitter, rowType, project, list);
```

- **Параметры:**
  - `source`: Исходная строка.
  - `rowSplitter`: Разделитель строк (например, `";"`).
  - `rowType`: Тип разделения (`"Text"` или `"Regex"`).
  - `project`: Объект проекта.
  - `list`: Целевой `IZennoList`.
- **Пример:**
  ```csharp
  string data = "Элемент1;Элемент2;Элемент3";
  IZennoList list = project.Lists["MyList"];
  try {
      lock(list) {
          Macros.TextProcessing.ToList(data, ";", "Text", project, list);
          project.SendInfoToLog("Список заполнен из строки", false);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка заполнения: {ex.Message}", false);
  }
  ```

##### 3.2. ZennoPoster.Db.ExecuteQuery
Выполняет SQL-запрос к базе данных и возвращает результат в `IZennoList`.

```csharp
OrderedDictionary parameters = new OrderedDictionary();
parameters.Add("param1", "value1");
IZennoList resultList = project.Lists["ResultList"];
ZennoPoster.Db.ExecuteQuery("SELECT * FROM table WHERE column = :param1", parameters, ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SQLite, "path_to_db.db", out resultList, "column_name", false);
```

- **Параметры:**
  - `query`: SQL-запрос.
  - `parameters`: Параметры запроса (`OrderedDictionary`).
  - `dbProvider`: Тип базы данных (например, `SQLite`).
  - `dbPath`: Путь к базе данных.
  - `resultList`: Выходной `IZennoList` (передается как `out`).
  - `columnName`: Имя столбца для извлечения данных.
  - `distinct`: Если `true`, возвращаются только уникальные значения.
- **Пример:**
  ```csharp
  IZennoList resultList = project.Lists["ResultList"];
  OrderedDictionary parameters = new OrderedDictionary();
  parameters.Add("id", "1");
  try {
      lock(resultList) {
          ZennoPoster.Db.ExecuteQuery("SELECT name FROM users WHERE id = :id", parameters, ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SQLite, Path.Combine(project.Directory, "db.sqlite"), out resultList, "name", true);
          project.SendInfoToLog($"Получено {resultList.Count} записей", false);
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка запроса: {ex.Message}", false);
  }
  ```

##### 3.3. IInstanceManagerService.DbExecuteQuery
Аналогичен `ZennoPoster.Db.ExecuteQuery`, но вызывается через `instance`.

```csharp
OrderedDictionary parameters = new OrderedDictionary();
parameters.Add("param1", "value1");
IZennoList resultList = project.Lists["ResultList"];
instance.DbExecuteQuery("SELECT * FROM table WHERE column = :param1", parameters, ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SQLite, "path_to_db.db", out resultList, "column_name", false);
```

- **Описание:** Выполняет SQL-запрос через менеджер экземпляров.
- **Пример:** Аналогичен предыдущему, но использует `instance` вместо `ZennoPoster`.

#### 4. Рекомендации по работе с IZennoList
- **Потокобезопасность:** Всегда используйте `lock(list)` для операций в многопоточных сценариях.
- **Проверка данных:** Проверяйте входные данные перед добавлением:
  ```csharp
  if (!string.IsNullOrEmpty(item)) {
      lock(list) {
          list.Add(item);
      }
  }
  ```
- **Ошибки:** Оборачивайте все операции в `try-catch` и логируйте:
  ```csharp
  try {
      lock(list) {
          list.Add("Элемент");
      }
  } catch (Exception ex) {
      project.SendErrorToLog($"Ошибка: {ex.Message}", false);
  }
  ```
- **Производительность:** Для сложных операций (сортировка, фильтрация) используйте `List<string>`:
  ```csharp
  List<string> tempList = new List<string>();
  tempList.AddRange(list.GetItems("all", false));
  tempList.Sort();
  lock(list) {
      list.Clear();
      list.AddRange(tempList);
  }
  ```
- **Работа с файлами:** После `Bind` проверяйте доступность файла:
  ```csharp
  string filePath = Path.Combine(project.Directory, "mylist.txt");
  if (!File.Exists(filePath)) {
      File.WriteAllText(filePath, "");
  }
  ```

#### 5. Комплексный пример
Сценарий: загрузка данных из файла, обработка, запрос к базе данных и сохранение в `IZennoList`.

```csharp
string filePath = Path.Combine(project.Directory, "mylist.txt");
IZennoList list = project.Lists["MyList"];
try {
    // Привязка к файлу
    lock(list) {
        list.Bind(filePath);
        project.SendInfoToLog("Список привязан к файлу", false);
    }

    // Загрузка строк из файла
    string[] lines = FileSystem.FileGetLines(filePath, "all", false);
    List<string> tempList = new List<string>();
    foreach (string line in lines) {
        string trimmed = Macros.TextProcessing.Trim(line, "Full");
        if (!string.IsNullOrEmpty(trimmed)) {
            tempList.Add(trimmed);
        }
    }

    // Добавление в список
    lock(list) {
        list.Clear();
        list.AddRange(tempList);
        project.SendInfoToLog($"Добавлено {tempList.Count} элементов", false);
    }

    // Запрос к базе данных
    OrderedDictionary parameters = new OrderedDictionary();
    parameters.Add("id", "1");
    IZennoList dbList = project.Lists["DbResults"];
    lock(dbList) {
        ZennoPoster.Db.ExecuteQuery("SELECT name FROM users WHERE id = :id", parameters, ZennoLab.InterfacesLibrary.Enums.Db.DbProvider.SQLite, Path.Combine(project.Directory, "db.sqlite"), out dbList, "name", true);
        project.SendInfoToLog($"Получено {dbList.Count} записей из БД", false);
    }

    // Получение и обработка элемента
    lock(list) {
        string item = list.GetItem("0", false);
        if (!string.IsNullOrEmpty(item)) {
            project.Variables["Result"].Value = item;
            project.SendInfoToLog($"Получен элемент: {item}", false);
        }
    }
} catch (Exception ex) {
    project.SendErrorToLog($"Ошибка: {ex.Message}", false);
}
```

---

