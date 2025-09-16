# Класс FS

Утилитарный класс для работы с файловой системой, предоставляющий методы для управления файлами и директориями, копирования данных и работы с учетными данными.

## Конструкторы

### FS(IZennoPosterProjectModel project, bool log = false, string classEmoji = null)

Создает новый экземпляр класса FS.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `log` - включить логирование (по умолчанию false)
- `classEmoji` - эмодзи для класса (необязательный параметр)

**Пример использования:**
```csharp
// создание экземпляра без логирования
var fs = new FS(project);

// создание экземпляра с включенным логированием
var fs = new FS(project, true);
```

## Методы

### void RmRf(string path)

Рекурсивно удаляет директорию со всем содержимым.

**Параметры:**
- `path` - путь к удаляемой директории

**Пример использования:**
```csharp
var fs = new FS(project);
//удалить директорию с файлами
fs.RmRf(@"C:\temp\folder");
```

### void CopyDir(string sourceDir, string destDir)

Копирует директорию со всем содержимым в указанное место.

**Параметры:**
- `sourceDir` - путь к исходной директории
- `destDir` - путь к целевой директории

**Пример использования:**
```csharp
var fs = new FS(project);
//скопировать папку с файлами
fs.CopyDir(@"C:\source", @"C:\destination");
```

### string GetRandomFile(string directoryPath)

**Возвращает:** случайный файл из указанной директории или null, если файлы не найдены

Получает случайный файл из директории, включая подпапки.

**Параметры:**
- `directoryPath` - путь к директории для поиска файлов

**Пример использования:**
```csharp
//получить случайный файл из папки
string randomFile = FS.GetRandomFile(@"C:\images");
if (randomFile != null)
{
    project.SendInfoToLog("Выбран файл: " + randomFile);
}
```

### string GetNewCreds(string dataType)

**Возвращает:** строку с учетными данными или null при ошибке

Получает новые учетные данные из файла fresh и перемещает их в файл used.

**Параметры:**
- `dataType` - тип данных для получения (имя файла без расширения)

**Пример использования:**
```csharp
var fs = new FS(project, true);
//получить новый аккаунт из $"{_project.Path}.data\\fresh\\{dataType}.txt"
string account = fs.GetNewCreds("twitter"); 
if (account != null)
{
    project.SendInfoToLog("Получен аккаунт: " + account);
}
else
{
    project.SendWarningToLog("Аккаунты закончились");
}
```