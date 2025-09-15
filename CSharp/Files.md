# SystemIO (Файлы)

# Полное руководство по работе с файловой системой в C#

## Введение

Работа с файловой системой — это одна из фундаментальных задач в программировании. В C# для этого предоставлен богатый набор классов в пространстве имен `System.IO`, которые позволяют эффективно работать с файлами, папками и путями.

## Основные классы для работы с файловой системой

- `File` и `FileInfo` — для работы с файлами
- `Directory` и `DirectoryInfo` — для работы с папками
- `Path` — для работы с путями
- `FileStream`, `StreamReader`, `StreamWriter` — для потокового чтения/записи
- `DriveInfo` — для работы с дисками

## Работа с путями

### Класс Path - основа всех операций

```csharp
using System;
using System.IO;

// Построение путей кросс-платформенно
string documentsPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string filePath = Path.Combine(documentsPath, "MyApp", "data.txt");

// Получение информации о пути
string fileName = Path.GetFileName(filePath);           // "data.txt"
string fileNameWithoutExt = Path.GetFileNameWithoutExtension(filePath); // "data"
string extension = Path.GetExtension(filePath);        // ".txt"
string directory = Path.GetDirectoryName(filePath);    // путь к папке
string fullPath = Path.GetFullPath("../data.txt");     // абсолютный путь

// Проверка и валидация путей
bool isValidPath = Path.IsPathRooted(filePath);         // true для абсолютных путей
char[] invalidChars = Path.GetInvalidFileNameChars();  // недопустимые символы
char[] invalidPathChars = Path.GetInvalidPathChars();  // недопустимые символы для пути

// Создание временных файлов
string tempFile = Path.GetTempFileName();              // создает временный файл
string tempPath = Path.GetTempPath();                  // путь к временной папке
string randomName = Path.GetRandomFileName();          // случайное имя файла

```

### Валидация и нормализация путей

```csharp
public static class PathValidator
{
    public static bool IsValidFileName(string fileName)
    {
        if (string.IsNullOrWhiteSpace(fileName))
            return false;

        char[] invalidChars = Path.GetInvalidFileNameChars();
        return !fileName.Any(c => invalidChars.Contains(c));
    }

    public static bool IsValidPath(string path)
    {
        try
        {
            Path.GetFullPath(path);
            return true;
        }
        catch
        {
            return false;
        }
    }

    public static string SanitizeFileName(string fileName)
    {
        char[] invalidChars = Path.GetInvalidFileNameChars();
        return string.Join("_", fileName.Split(invalidChars, StringSplitOptions.RemoveEmptyEntries));
    }

    public static string NormalizePath(string path)
    {
        return Path.GetFullPath(path).TrimEnd(Path.DirectorySeparatorChar, Path.AltDirectorySeparatorChar);
    }
}

// Использование
bool isValid = PathValidator.IsValidFileName("my*file?.txt");    // false
string cleanName = PathValidator.SanitizeFileName("my*file?.txt"); // "my_file_.txt"

```

## Работа с файлами

### Основные операции с файлами

```csharp
string filePath = @"C:\temp\example.txt";
string content = "Пример содержимого файла";

// Проверка существования файла
bool exists = File.Exists(filePath);

// Создание и запись файла
File.WriteAllText(filePath, content, Encoding.UTF8);

// Добавление к существующему файлу
File.AppendAllText(filePath, "\nДополнительная строка", Encoding.UTF8);

// Чтение файла
string fileContent = File.ReadAllText(filePath, Encoding.UTF8);
string[] lines = File.ReadAllLines(filePath, Encoding.UTF8);
byte[] bytes = File.ReadAllBytes(filePath);

// Копирование файлов
string destinationPath = @"C:\temp\copy.txt";
File.Copy(filePath, destinationPath);                    // без перезаписи
File.Copy(filePath, destinationPath, overwrite: true);   // с перезаписью

// Перемещение файла
string newPath = @"C:\temp\moved.txt";
File.Move(filePath, newPath);

// Удаление файла
File.Delete(newPath);

// Получение атрибутов файла
FileAttributes attributes = File.GetAttributes(filePath);
DateTime created = File.GetCreationTime(filePath);
DateTime modified = File.GetLastWriteTime(filePath);
long size = new FileInfo(filePath).Length;

```

### Безопасное копирование файлов с проверками

```csharp
public static class SafeFileOperations
{
    public enum CopyResult
    {
        Success,
        SourceNotFound,
        DestinationExists,
        InsufficientSpace,
        AccessDenied,
        Error
    }

    public static CopyResult SafeCopy(string sourcePath, string destinationPath, bool overwrite = false)
    {
        try
        {
            // Проверка существования исходного файла
            if (!File.Exists(sourcePath))
                return CopyResult.SourceNotFound;

            // Проверка перезаписи
            if (!overwrite && File.Exists(destinationPath))
                return CopyResult.DestinationExists;

            // Создание папки назначения если не существует
            string destinationDir = Path.GetDirectoryName(destinationPath);
            if (!Directory.Exists(destinationDir))
                Directory.CreateDirectory(destinationDir);

            // Проверка свободного места
            long fileSize = new FileInfo(sourcePath).Length;
            DriveInfo drive = new DriveInfo(Path.GetPathRoot(destinationPath));
            if (drive.AvailableFreeSpace < fileSize)
                return CopyResult.InsufficientSpace;

            // Копирование
            File.Copy(sourcePath, destinationPath, overwrite);
            return CopyResult.Success;
        }
        catch (UnauthorizedAccessException)
        {
            return CopyResult.AccessDenied;
        }
        catch
        {
            return CopyResult.Error;
        }
    }

    public static bool BackupFile(string filePath)
    {
        if (!File.Exists(filePath))
            return false;

        string backupPath = $"{filePath}.backup.{DateTime.Now:yyyyMMddHHmmss}";
        return SafeCopy(filePath, backupPath) == CopyResult.Success;
    }
}

// Использование
var result = SafeFileOperations.SafeCopy(@"C:\source.txt", @"C:\destination.txt");
switch (result)
{
    case SafeFileOperations.CopyResult.Success:
        Console.WriteLine("Файл успешно скопирован");
        break;
    case SafeFileOperations.CopyResult.SourceNotFound:
        Console.WriteLine("Исходный файл не найден");
        break;
    // обработка других случаев...
}

```

### FileInfo - объектно-ориентированный подход

```csharp
FileInfo fileInfo = new FileInfo(@"C:\temp\example.txt");

// Проверка существования и создание
if (!fileInfo.Exists)
{
    using (var stream = fileInfo.Create())
    {
        byte[] data = Encoding.UTF8.GetBytes("Новый файл");
        stream.Write(data, 0, data.Length);
    }
}

// Информация о файле
Console.WriteLine($"Имя: {fileInfo.Name}");
Console.WriteLine($"Размер: {fileInfo.Length} байт");
Console.WriteLine($"Создан: {fileInfo.CreationTime}");
Console.WriteLine($"Изменен: {fileInfo.LastWriteTime}");
Console.WriteLine($"Папка: {fileInfo.DirectoryName}");
Console.WriteLine($"Только чтение: {fileInfo.IsReadOnly}");

// Операции с файлом
FileInfo copy = fileInfo.CopyTo(@"C:\temp\copy.txt");
fileInfo.MoveTo(@"C:\temp\moved.txt");

// Изменение атрибутов
fileInfo.Attributes |= FileAttributes.Hidden;     // скрыть файл
fileInfo.Attributes &= ~FileAttributes.Hidden;    // показать файл
fileInfo.IsReadOnly = true;                       // сделать только для чтения

```

## Работа с папками и директориями

### Основные операции с папками

```csharp
string directoryPath = @"C:\temp\MyFolder";

// Проверка существования папки
bool exists = Directory.Exists(directoryPath);

// Создание папки
Directory.CreateDirectory(directoryPath);

// Создание вложенных папок
string nestedPath = Path.Combine(directoryPath, "SubFolder1", "SubFolder2");
Directory.CreateDirectory(nestedPath);

// Получение содержимого папки
string[] files = Directory.GetFiles(directoryPath);
string[] directories = Directory.GetDirectories(directoryPath);
string[] allEntries = Directory.GetFileSystemEntries(directoryPath);

// Поиск с фильтрами
string[] txtFiles = Directory.GetFiles(directoryPath, "*.txt");
string[] allTxtFiles = Directory.GetFiles(directoryPath, "*.txt", SearchOption.AllDirectories);

// Получение информации о папке
DirectoryInfo dirInfo = new DirectoryInfo(directoryPath);
DateTime created = Directory.GetCreationTime(directoryPath);
DateTime modified = Directory.GetLastWriteTime(directoryPath);

// Удаление папки
Directory.Delete(directoryPath);                  // только пустую
Directory.Delete(directoryPath, recursive: true); // с содержимым

```

### Рекурсивные операции с папками

```csharp
public static class DirectoryOperations
{
    public static void CopyDirectory(string sourceDir, string destinationDir, bool copySubDirs = true)
    {
        DirectoryInfo dir = new DirectoryInfo(sourceDir);
        if (!dir.Exists)
            throw new DirectoryNotFoundException($"Папка не найдена: {sourceDir}");

        // Создание целевой папки
        Directory.CreateDirectory(destinationDir);

        // Копирование файлов
        FileInfo[] files = dir.GetFiles();
        foreach (FileInfo file in files)
        {
            string tempPath = Path.Combine(destinationDir, file.Name);
            file.CopyTo(tempPath, false);
        }

        // Рекурсивное копирование подпапок
        if (copySubDirs)
        {
            DirectoryInfo[] subDirs = dir.GetDirectories();
            foreach (DirectoryInfo subDir in subDirs)
            {
                string tempPath = Path.Combine(destinationDir, subDir.Name);
                CopyDirectory(subDir.FullName, tempPath, copySubDirs);
            }
        }
    }

    public static long GetDirectorySize(string directoryPath)
    {
        DirectoryInfo dirInfo = new DirectoryInfo(directoryPath);
        return GetDirectorySize(dirInfo);
    }

    private static long GetDirectorySize(DirectoryInfo dir)
    {
        long size = 0;

        // Размер всех файлов в папке
        FileInfo[] files = dir.GetFiles();
        foreach (FileInfo file in files)
        {
            size += file.Length;
        }

        // Рекурсивно для всех подпапок
        DirectoryInfo[] subDirs = dir.GetDirectories();
        foreach (DirectoryInfo subDir in subDirs)
        {
            size += GetDirectorySize(subDir);
        }

        return size;
    }

    public static int CountFiles(string directoryPath, string searchPattern = "*", bool recursive = false)
    {
        SearchOption option = recursive ? SearchOption.AllDirectories : SearchOption.TopDirectoryOnly;
        return Directory.GetFiles(directoryPath, searchPattern, option).Length;
    }

    public static void CleanDirectory(string directoryPath, bool deleteSubdirectories = false)
    {
        DirectoryInfo dir = new DirectoryInfo(directoryPath);

        // Удаление всех файлов
        foreach (FileInfo file in dir.GetFiles())
        {
            file.Delete();
        }

        // Удаление всех подпапок
        if (deleteSubdirectories)
        {
            foreach (DirectoryInfo subDir in dir.GetDirectories())
            {
                subDir.Delete(true);
            }
        }
    }
}

// Использование
DirectoryOperations.CopyDirectory(@"C:\source", @"C:\destination");
long size = DirectoryOperations.GetDirectorySize(@"C:\MyFolder");
Console.WriteLine($"Размер папки: {size / (1024 * 1024)} МБ");

```

### DirectoryInfo - объектно-ориентированный подход

```csharp
DirectoryInfo dirInfo = new DirectoryInfo(@"C:\temp\TestDirectory");

// Создание папки если не существует
if (!dirInfo.Exists)
    dirInfo.Create();

// Информация о папке
Console.WriteLine($"Имя: {dirInfo.Name}");
Console.WriteLine($"Полный путь: {dirInfo.FullName}");
Console.WriteLine($"Родительская папка: {dirInfo.Parent?.Name}");
Console.WriteLine($"Корень: {dirInfo.Root}");
Console.WriteLine($"Создана: {dirInfo.CreationTime}");

// Получение содержимого
FileInfo[] files = dirInfo.GetFiles("*.txt");
DirectoryInfo[] subDirs = dirInfo.GetDirectories();
FileSystemInfo[] allItems = dirInfo.GetFileSystemInfos();

// Создание подпапки
DirectoryInfo subDir = dirInfo.CreateSubdirectory("NewSubFolder");

// Перемещение папки
DirectoryInfo moved = dirInfo.MoveTo(@"C:\temp\MovedDirectory");

```

## Продвинутые операции с файловой системой

### Мониторинг изменений в файловой системе

```csharp
public class FileSystemMonitor : IDisposable
{
    private readonly FileSystemWatcher _watcher;

    public event EventHandler<FileSystemEventArgs> FileChanged;
    public event EventHandler<FileSystemEventArgs> FileCreated;
    public event EventHandler<FileSystemEventArgs> FileDeleted;
    public event EventHandler<RenamedEventArgs> FileRenamed;

    public FileSystemMonitor(string path, string filter = "*.*")
    {
        _watcher = new FileSystemWatcher(path, filter)
        {
            IncludeSubdirectories = true,
            EnableRaisingEvents = false
        };

        _watcher.Changed += OnChanged;
        _watcher.Created += OnCreated;
        _watcher.Deleted += OnDeleted;
        _watcher.Renamed += OnRenamed;
        _watcher.Error += OnError;
    }

    public void StartMonitoring()
    {
        _watcher.EnableRaisingEvents = true;
    }

    public void StopMonitoring()
    {
        _watcher.EnableRaisingEvents = false;
    }

    private void OnChanged(object sender, FileSystemEventArgs e)
    {
        Console.WriteLine($"Файл изменен: {e.Name}");
        FileChanged?.Invoke(this, e);
    }

    private void OnCreated(object sender, FileSystemEventArgs e)
    {
        Console.WriteLine($"Файл создан: {e.Name}");
        FileCreated?.Invoke(this, e);
    }

    private void OnDeleted(object sender, FileSystemEventArgs e)
    {
        Console.WriteLine($"Файл удален: {e.Name}");
        FileDeleted?.Invoke(this, e);
    }

    private void OnRenamed(object sender, RenamedEventArgs e)
    {
        Console.WriteLine($"Файл переименован: {e.OldName} -> {e.Name}");
        FileRenamed?.Invoke(this, e);
    }

    private void OnError(object sender, ErrorEventArgs e)
    {
        Console.WriteLine($"Ошибка мониторинга: {e.GetException().Message}");
    }

    public void Dispose()
    {
        _watcher?.Dispose();
    }
}

// Использование
using var monitor = new FileSystemMonitor(@"C:\MonitoredFolder");
monitor.FileCreated += (sender, e) => Console.WriteLine($"Обработка нового файла: {e.Name}");
monitor.StartMonitoring();

// Мониторинг работает...
Console.ReadKey();

```

### Поиск файлов с расширенными критериями

```csharp
public static class FileSearcher
{
    public class SearchCriteria
    {
        public string Pattern { get; set; } = "*";
        public long? MinSize { get; set; }
        public long? MaxSize { get; set; }
        public DateTime? CreatedAfter { get; set; }
        public DateTime? CreatedBefore { get; set; }
        public DateTime? ModifiedAfter { get; set; }
        public DateTime? ModifiedBefore { get; set; }
        public FileAttributes? RequiredAttributes { get; set; }
        public FileAttributes? ForbiddenAttributes { get; set; }
        public bool IncludeSubdirectories { get; set; } = true;
    }

    public static IEnumerable<FileInfo> SearchFiles(string directory, SearchCriteria criteria)
    {
        SearchOption searchOption = criteria.IncludeSubdirectories
            ? SearchOption.AllDirectories
            : SearchOption.TopDirectoryOnly;

        var files = Directory.GetFiles(directory, criteria.Pattern, searchOption)
            .Select(f => new FileInfo(f));

        return files.Where(file => MatchesCriteria(file, criteria));
    }

    private static bool MatchesCriteria(FileInfo file, SearchCriteria criteria)
    {
        // Проверка размера
        if (criteria.MinSize.HasValue && file.Length < criteria.MinSize.Value)
            return false;
        if (criteria.MaxSize.HasValue && file.Length > criteria.MaxSize.Value)
            return false;

        // Проверка дат
        if (criteria.CreatedAfter.HasValue && file.CreationTime < criteria.CreatedAfter.Value)
            return false;
        if (criteria.CreatedBefore.HasValue && file.CreationTime > criteria.CreatedBefore.Value)
            return false;
        if (criteria.ModifiedAfter.HasValue && file.LastWriteTime < criteria.ModifiedAfter.Value)
            return false;
        if (criteria.ModifiedBefore.HasValue && file.LastWriteTime > criteria.ModifiedBefore.Value)
            return false;

        // Проверка атрибутов
        if (criteria.RequiredAttributes.HasValue &&
            (file.Attributes & criteria.RequiredAttributes.Value) != criteria.RequiredAttributes.Value)
            return false;
        if (criteria.ForbiddenAttributes.HasValue &&
            (file.Attributes & criteria.ForbiddenAttributes.Value) != 0)
            return false;

        return true;
    }

    public static IEnumerable<FileInfo> FindDuplicates(string directory)
    {
        var files = Directory.GetFiles(directory, "*", SearchOption.AllDirectories)
            .Select(f => new FileInfo(f));

        var duplicates = files
            .GroupBy(f => f.Length)
            .Where(g => g.Count() > 1)
            .SelectMany(g => g)
            .GroupBy(f => ComputeFileHash(f.FullName))
            .Where(g => g.Count() > 1)
            .SelectMany(g => g.Skip(1)); // Пропускаем первый файл, оставляем дубликаты

        return duplicates;
    }

    private static string ComputeFileHash(string filePath)
    {
        using var md5 = System.Security.Cryptography.MD5.Create();
        using var stream = File.OpenRead(filePath);
        var hash = md5.ComputeHash(stream);
        return Convert.ToBase64String(hash);
    }
}

// Использование
var criteria = new FileSearcher.SearchCriteria
{
    Pattern = "*.log",
    MinSize = 1024 * 1024, // минимум 1 МБ
    CreatedAfter = DateTime.Now.AddDays(-7), // за последнюю неделю
    RequiredAttributes = FileAttributes.Archive
};

var foundFiles = FileSearcher.SearchFiles(@"C:\Logs", criteria);
foreach (var file in foundFiles)
{
    Console.WriteLine($"{file.Name} ({file.Length} bytes)");
}

```

### Работа с дисками и свободным местом

```csharp
public static class DriveManager
{
    public static void DisplayDriveInfo()
    {
        DriveInfo[] drives = DriveInfo.GetDrives();

        foreach (DriveInfo drive in drives)
        {
            Console.WriteLine($"Диск: {drive.Name}");

            if (drive.IsReady)
            {
                Console.WriteLine($"  Тип: {drive.DriveType}");
                Console.WriteLine($"  Метка: {drive.VolumeLabel}");
                Console.WriteLine($"  Файловая система: {drive.DriveFormat}");
                Console.WriteLine($"  Общий размер: {FormatBytes(drive.TotalSize)}");
                Console.WriteLine($"  Свободно: {FormatBytes(drive.AvailableFreeSpace)}");
                Console.WriteLine($"  Использовано: {FormatBytes(drive.TotalSize - drive.AvailableFreeSpace)}");

                double usedPercentage = ((double)(drive.TotalSize - drive.AvailableFreeSpace) / drive.TotalSize) * 100;
                Console.WriteLine($"  Заполнено: {usedPercentage:F1}%");
            }
            else
            {
                Console.WriteLine("  Диск не готов");
            }
            Console.WriteLine();
        }
    }

    public static bool HasEnoughSpace(string path, long requiredBytes)
    {
        DriveInfo drive = new DriveInfo(Path.GetPathRoot(path));
        return drive.IsReady && drive.AvailableFreeSpace >= requiredBytes;
    }

    public static DriveInfo GetDriveWithMostSpace(DriveType driveType = DriveType.Fixed)
    {
        return DriveInfo.GetDrives()
            .Where(d => d.IsReady && d.DriveType == driveType)
            .OrderByDescending(d => d.AvailableFreeSpace)
            .FirstOrDefault();
    }

    public static string FormatBytes(long bytes)
    {
        string[] suffixes = { "B", "KB", "MB", "GB", "TB", "PB" };
        int counter = 0;
        decimal number = bytes;

        while (Math.Round(number / 1024) >= 1)
        {
            number /= 1024;
            counter++;
        }

        return $"{number:n1} {suffixes[counter]}";
    }
}

// Использование
DriveManager.DisplayDriveInfo();
bool hasSpace = DriveManager.HasEnoughSpace(@"C:\", 1024 * 1024 * 100); // 100 МБ
DriveInfo bestDrive = DriveManager.GetDriveWithMostSpace();

```

## Потоковая работа с файлами

### Эффективное чтение и запись больших файлов

```csharp
public static class StreamOperations
{
    public static void ProcessLargeTextFile(string inputPath, string outputPath,
        Func<string, string> processor)
    {
        using var reader = new StreamReader(inputPath, Encoding.UTF8);
        using var writer = new StreamWriter(outputPath, false, Encoding.UTF8);

        string line;
        while ((line = reader.ReadLine()) != null)
        {
            string processedLine = processor(line);
            writer.WriteLine(processedLine);
        }
    }

    public static void CopyFileWithProgress(string sourcePath, string destinationPath,
        IProgress<(long transferred, long total)> progress = null)
    {
        const int bufferSize = 64 * 1024; // 64 KB buffer
        byte[] buffer = new byte[bufferSize];

        using var sourceStream = new FileStream(sourcePath, FileMode.Open, FileAccess.Read);
        using var destinationStream = new FileStream(destinationPath, FileMode.Create, FileAccess.Write);

        long totalBytes = sourceStream.Length;
        long transferredBytes = 0;
        int bytesRead;

        while ((bytesRead = sourceStream.Read(buffer, 0, bufferSize)) > 0)
        {
            destinationStream.Write(buffer, 0, bytesRead);
            transferredBytes += bytesRead;

            progress?.Report((transferredBytes, totalBytes));
        }
    }

    public static async Task<string> ReadFileAsync(string filePath)
    {
        using var reader = new StreamReader(filePath);
        return await reader.ReadToEndAsync();
    }

    public static async Task WriteFileAsync(string filePath, string content)
    {
        using var writer = new StreamWriter(filePath);
        await writer.WriteAsync(content);
    }

    public static IEnumerable<string> ReadLines(string filePath)
    {
        using var reader = new StreamReader(filePath);
        string line;
        while ((line = reader.ReadLine()) != null)
        {
            yield return line;
        }
    }
}

// Использование
var progress = new Progress<(long, long)>(p =>
{
    double percentage = (double)p.Item1 / p.Item2 * 100;
    Console.WriteLine($"Скопировано: {percentage:F1}%");
});

StreamOperations.CopyFileWithProgress(@"C:\large_file.dat", @"C:\copy.dat", progress);

// Обработка больших файлов построчно
foreach (string line in StreamOperations.ReadLines(@"C:\huge_log.txt"))
{
    // Обработка каждой строки без загрузки всего файла в память
    if (line.Contains("ERROR"))
    {
        Console.WriteLine(line);
    }
}

```

## Обработка исключений при работе с файловой системой

### Типичные исключения и их обработка

```csharp
public static class SafeFileSystem
{
    public static bool TryReadFile(string filePath, out string content, out string error)
    {
        content = null;
        error = null;

        try
        {
            content = File.ReadAllText(filePath, Encoding.UTF8);
            return true;
        }
        catch (FileNotFoundException)
        {
            error = "Файл не найден";
            return false;
        }
        catch (DirectoryNotFoundException)
        {
            error = "Папка не найдена";
            return false;
        }
        catch (UnauthorizedAccessException)
        {
            error = "Недостаточно прав доступа";
            return false;
        }
        catch (PathTooLongException)
        {
            error = "Слишком длинный путь к файлу";
            return false;
        }
        catch (IOException ex)
        {
            error = $"Ошибка ввода-вывода: {ex.Message}";
            return false;
        }
        catch (Exception ex)
        {
            error = $"Неожиданная ошибка: {ex.Message}";
            return false;
        }
    }

    public static async Task<(bool success, string error)> TryWriteFileAsync(
        string filePath, string content, bool createDirectories = true)
    {
        try
        {
            if (createDirectories)
            {
                string directory = Path.GetDirectoryName(filePath);
                if (!string.IsNullOrEmpty(directory) && !Directory.Exists(directory))
                {
                    Directory.CreateDirectory(directory);
                }
            }

            await File.WriteAllTextAsync(filePath, content, Encoding.UTF8);
            return (true, null);
        }
        catch (UnauthorizedAccessException)
        {
            return (false, "Недостаточно прав для записи файла");
        }
        catch (DirectoryNotFoundException)
        {
            return (false, "Папка назначения не существует");
        }
        catch (IOException ex) when ((ex.HResult & 0xFFFF) == 0x27) // Disk full
        {
            return (false, "Недостаточно места на диске");
        }
        catch (IOException ex)
        {
            return (false, $"Ошибка записи файла: {ex.Message}");
        }
        catch (Exception ex)
        {
            return (false, $"Неожиданная ошибка: {ex.Message}");
        }
    }

    public static bool IsFileLocked(string filePath)
    {
        try
        {
            using var stream = File.Open(filePath, FileMode.Open, FileAccess.Read, FileShare.None);
            return false;
        }
        catch (IOException)
        {
            return true;
        }
        catch
        {
            return false; // Файл не существует или другая ошибка
        }
    }

    public static async Task<bool> WaitForFileUnlock(string filePath,
        TimeSpan timeout, CancellationToken cancellationToken = default)
    {
        var endTime = DateTime.Now.Add(timeout);

        while (DateTime.Now < endTime && !cancellationToken.IsCancellationRequested)
        {
            if (!IsFileLocked(filePath))
                return true;

            await Task.Delay(100, cancellationToken);
        }

        return false;
    }
}

// Использование
if (SafeFileSystem.TryReadFile(@"C:\data.txt", out string content, out string error))
{
    Console.WriteLine($"Файл прочитан: {content.Length} символов");
}
else
{
    Console.WriteLine($"Ошибка чтения файла: {error}");
}

var (success, writeError) = await SafeFileSystem.TryWriteFileAsync(@"C:\output.txt", "test content");
if (!success)
{
    Console.WriteLine($"Ошибка записи: {writeError}");
}

```

## Практические примеры из реальной работы

### Пример 1: Система резервного копирования

```csharp
public class BackupManager
{
    public class BackupOptions
    {
        public string SourceDirectory { get; set; }
        public string BackupDirectory { get; set; }
        public string[] IncludePatterns { get; set; } = { "*" };
        public string[] ExcludePatterns { get; set; } = Array.Empty<string>();
        public bool Incremental { get; set; } = true;
        public bool Compress { get; set; } = false;
        public int MaxBackups { get; set; } = 10;
    }

    private readonly BackupOptions _options;

    public BackupManager(BackupOptions options)
    {
        _options = options ?? throw new ArgumentNullException(nameof(options));
    }

    public async Task<bool> CreateBackupAsync(IProgress<string> progress = null)
    {
        try
        {
            string backupPath = GenerateBackupPath();
            progress?.Report($"Создание резервной копии: {backupPath}");

            Directory.CreateDirectory(backupPath);

            var filesToBackup = GetFilesToBackup();
            int totalFiles = filesToBackup.Count();
            int processedFiles = 0;

            foreach (var file in filesToBackup)
            {
                string relativePath = Path.GetRelativePath(_options.SourceDirectory, file.FullName);
                string destPath = Path.Combine(backupPath, relativePath);

                // Создание папки назначения
                Directory.CreateDirectory(Path.GetDirectoryName(destPath));

                // Копирование файла
                File.Copy(file.FullName, destPath, true);

                processedFiles++;
                progress?.Report($"Скопировано файлов: {processedFiles}/{totalFiles}");
            }

            // Очистка старых резервных копий
            await CleanupOldBackupsAsync();

            progress?.Report("Резервное копирование завершено успешно");
            return true;
        }
        catch (Exception ex)
        {
            progress?.Report($"Ошибка резервного копирования: {ex.Message}");
            return false;
        }
    }

    private string GenerateBackupPath()
    {
        string timestamp = DateTime.Now.ToString("yyyyMMdd_HHmmss");
        return Path.Combine(_options.BackupDirectory, $"backup_{timestamp}");
    }

    private IEnumerable<FileInfo> GetFilesToBackup()
    {
        var sourceDir = new DirectoryInfo(_options.SourceDirectory);
        var allFiles = sourceDir.GetFiles("*", SearchOption.AllDirectories);

        return allFiles.Where(file =>
        {
            string relativePath = Path.GetRelativePath(_options.SourceDirectory, file.FullName);

            // Проверка включающих паттернов
            bool included = _options.IncludePatterns.Any(pattern =>
                FilePatternMatcher.IsMatch(relativePath, pattern));

            // Проверка исключающих паттернов
            bool excluded = _options.ExcludePatterns.Any(pattern =>
                FilePatternMatcher.IsMatch(relativePath, pattern));

            return included && !excluded;
        });
    }

    private async Task CleanupOldBackupsAsync()
    {
        var backupDir = new DirectoryInfo(_options.BackupDirectory);
        var backupFolders = backupDir.GetDirectories("backup_*")
            .OrderByDescending(d => d.CreationTime)
            .Skip(_options.MaxBackups);

        foreach (var folder in backupFolders)
        {
            await Task.Run(() => folder.Delete(true));
        }
    }
}

public static class FilePatternMatcher
{
    public static bool IsMatch(string input, string pattern)
    {
        return new Regex(
            "^" + Regex.Escape(pattern).Replace(@"\*", ".*").Replace(@"\?", ".") + "$",
            RegexOptions.IgnoreCase).IsMatch(input);
    }
}

// Использование
var options = new BackupManager.BackupOptions
{
    SourceDirectory = @"C:\ImportantData",
    BackupDirectory = @"D:\Backups",
    IncludePatterns = new[] { "*.txt", "*.docx", "*.pdf" },
    ExcludePatterns = new[] { "temp\\*", "*.tmp" },
    MaxBackups = 5
};

var backupManager = new BackupManager(options);
var progress = new Progress<string>(message => Console.WriteLine(message));
await backupManager.CreateBackupAsync(progress);

```

### Пример 2: Система логирования в файлы

```csharp
public class FileLogger : IDisposable
{
    public enum LogLevel
    {
        Debug, Info, Warning, Error, Fatal
    }

    private readonly string _logDirectory;
    private readonly string _fileNamePrefix;
    private readonly long _maxFileSize;
    private readonly int _maxFiles;
    private readonly object _lockObject = new object();
    private StreamWriter _currentWriter;
    private string _currentFileName;

    public FileLogger(string logDirectory, string fileNamePrefix = "log",
        long maxFileSize = 10 * 1024 * 1024, int maxFiles = 10)
    {
        _logDirectory = logDirectory;
        _fileNamePrefix = fileNamePrefix;
        _maxFileSize = maxFileSize;
        _maxFiles = maxFiles;

        Directory.CreateDirectory(logDirectory);
        InitializeLogFile();
    }

    public void Log(LogLevel level, string message, Exception exception = null)
    {
        lock (_lockObject)
        {
            try
            {
                CheckAndRotateLogFile();

                string logEntry = FormatLogEntry(level, message, exception);
                _currentWriter.WriteLine(logEntry);
                _currentWriter.Flush();
            }
            catch (Exception ex)
            {
                // Fallback логирование в консоль
                Console.WriteLine($"Logger error: {ex.Message}");
                Console.WriteLine($"Original message: [{level}] {message}");
            }
        }
    }

    public void Debug(string message) => Log(LogLevel.Debug, message);
    public void Info(string message) => Log(LogLevel.Info, message);
    public void Warning(string message) => Log(LogLevel.Warning, message);
    public void Error(string message, Exception exception = null) => Log(LogLevel.Error, message, exception);
    public void Fatal(string message, Exception exception = null) => Log(LogLevel.Fatal, message, exception);

    private void InitializeLogFile()
    {
        string fileName = GenerateLogFileName();
        _currentFileName = Path.Combine(_logDirectory, fileName);
        _currentWriter = new StreamWriter(_currentFileName, true, Encoding.UTF8) { AutoFlush = false };
    }

    private void CheckAndRotateLogFile()
    {
        if (new FileInfo(_currentFileName).Length > _maxFileSize)
        {
            RotateLogFile();
        }
    }

    private void RotateLogFile()
    {
        _currentWriter?.Close();
        _currentWriter?.Dispose();

        CleanupOldLogFiles();
        InitializeLogFile();
    }

    private void CleanupOldLogFiles()
    {
        var logFiles = Directory.GetFiles(_logDirectory, $"{_fileNamePrefix}_*.log")
            .Select(f => new FileInfo(f))
            .OrderByDescending(f => f.CreationTime)
            .Skip(_maxFiles - 1);

        foreach (var file in logFiles)
        {
            try
            {
                file.Delete();
            }
            catch
            {
                // Игнорируем ошибки удаления
            }
        }
    }

    private string GenerateLogFileName()
    {
        string timestamp = DateTime.Now.ToString("yyyyMMdd_HHmmss");
        return $"{_fileNamePrefix}_{timestamp}.log";
    }

    private string FormatLogEntry(LogLevel level, string message, Exception exception)
    {
        var sb = new StringBuilder();
        sb.Append(DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss.fff"));
        sb.Append(" [");
        sb.Append(level.ToString().ToUpper());
        sb.Append("] ");
        sb.Append(message);

        if (exception != null)
        {
            sb.AppendLine();
            sb.Append("Exception: ");
            sb.Append(exception.ToString());
        }

        return sb.ToString();
    }

    public void Dispose()
    {
        lock (_lockObject)
        {
            _currentWriter?.Close();
            _currentWriter?.Dispose();
        }
    }
}

// Использование
using var logger = new FileLogger(@"C:\Logs\MyApp");
logger.Info("Приложение запущено");
logger.Warning("Это предупреждение");
try
{
    // Некоторая операция
    throw new InvalidOperationException("Тестовая ошибка");
}
catch (Exception ex)
{
    logger.Error("Произошла ошибка в операции", ex);
}

```

### Пример 3: Менеджер временных файлов

```csharp
public class TempFileManager : IDisposable
{
    private readonly string _tempDirectory;
    private readonly List<string> _managedFiles;
    private readonly List<string> _managedDirectories;
    private bool _disposed;

    public TempFileManager(string customTempDir = null)
    {
        _tempDirectory = customTempDir ?? Path.Combine(Path.GetTempPath(), $"TempManager_{Guid.NewGuid():N}");
        _managedFiles = new List<string>();
        _managedDirectories = new List<string>();

        Directory.CreateDirectory(_tempDirectory);
        _managedDirectories.Add(_tempDirectory);
    }

    public string CreateTempFile(string extension = ".tmp", string content = null)
    {
        ThrowIfDisposed();

        string fileName = $"{Guid.NewGuid():N}{extension}";
        string filePath = Path.Combine(_tempDirectory, fileName);

        if (content != null)
        {
            File.WriteAllText(filePath, content, Encoding.UTF8);
        }
        else
        {
            File.Create(filePath).Close();
        }

        _managedFiles.Add(filePath);
        return filePath;
    }

    public string CreateTempDirectory(string prefix = "temp")
    {
        ThrowIfDisposed();

        string dirName = $"{prefix}_{Guid.NewGuid():N}";
        string dirPath = Path.Combine(_tempDirectory, dirName);

        Directory.CreateDirectory(dirPath);
        _managedDirectories.Add(dirPath);
        return dirPath;
    }

    public string GetTempFilePath(string extension = ".tmp")
    {
        ThrowIfDisposed();

        string fileName = $"{Guid.NewGuid():N}{extension}";
        string filePath = Path.Combine(_tempDirectory, fileName);
        _managedFiles.Add(filePath);
        return filePath;
    }

    public async Task<string> DownloadToTempFileAsync(string url, string extension = null)
    {
        ThrowIfDisposed();

        using var httpClient = new HttpClient();
        var response = await httpClient.GetAsync(url);
        response.EnsureSuccessStatusCode();

        if (string.IsNullOrEmpty(extension))
        {
            // Попытка определить расширение из URL или Content-Type
            extension = Path.GetExtension(new Uri(url).LocalPath);
            if (string.IsNullOrEmpty(extension))
            {
                var contentType = response.Content.Headers.ContentType?.MediaType;
                extension = GetExtensionFromContentType(contentType) ?? ".tmp";
            }
        }

        string tempFile = CreateTempFile(extension);
        using var fileStream = File.Create(tempFile);
        await response.Content.CopyToAsync(fileStream);

        return tempFile;
    }

    public void CleanupFile(string filePath)
    {
        try
        {
            if (File.Exists(filePath))
            {
                File.Delete(filePath);
            }
            _managedFiles.Remove(filePath);
        }
        catch
        {
            // Игнорируем ошибки очистки
        }
    }

    public void CleanupDirectory(string directoryPath)
    {
        try
        {
            if (Directory.Exists(directoryPath))
            {
                Directory.Delete(directoryPath, true);
            }
            _managedDirectories.Remove(directoryPath);
        }
        catch
        {
            // Игнорируем ошибки очистки
        }
    }

    private string GetExtensionFromContentType(string contentType)
    {
        return contentType switch
        {
            "text/plain" => ".txt",
            "application/json" => ".json",
            "application/xml" => ".xml",
            "image/jpeg" => ".jpg",
            "image/png" => ".png",
            "image/gif" => ".gif",
            "application/pdf" => ".pdf",
            _ => null
        };
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(TempFileManager));
    }

    public void Dispose()
    {
        if (_disposed) return;

        // Очистка всех управляемых файлов
        foreach (string file in _managedFiles.ToList())
        {
            CleanupFile(file);
        }

        // Очистка всех управляемых директорий (в обратном порядке)
        foreach (string directory in _managedDirectories.AsEnumerable().Reverse().ToList())
        {
            CleanupDirectory(directory);
        }

        _disposed = true;
    }
}

// Использование
using var tempManager = new TempFileManager();

// Создание временного файла с содержимым
string configFile = tempManager.CreateTempFile(".json", "{\"setting\": \"value\"}");

// Создание временной директории
string tempDir = tempManager.CreateTempDirectory("processing");

// Скачивание файла во временную папку
string downloadedFile = await tempManager.DownloadToTempFileAsync("https://example.com/data.csv");

// Все временные файлы автоматически удалятся при вызове Dispose()

```

## Оптимизация производительности

### Рекомендации по производительности

```csharp
public static class FileSystemOptimization
{
    // 1. Используйте буферизацию для больших файлов
    public static void BufferedCopy(string source, string destination, int bufferSize = 64 * 1024)
    {
        using var sourceStream = new FileStream(source, FileMode.Open, FileAccess.Read,
            FileShare.Read, bufferSize, FileOptions.SequentialScan);
        using var destStream = new FileStream(destination, FileMode.Create, FileAccess.Write,
            FileShare.None, bufferSize, FileOptions.WriteThrough);

        sourceStream.CopyTo(destStream, bufferSize);
    }

    // 2. Кешируйте результаты проверок существования
    private static readonly ConcurrentDictionary<string, bool> _existenceCache =
        new ConcurrentDictionary<string, bool>();

    public static bool CachedFileExists(string path)
    {
        return _existenceCache.GetOrAdd(path, File.Exists);
    }

    public static void InvalidateExistenceCache(string path)
    {
        _existenceCache.TryRemove(path, out _);
    }

    // 3. Используйте асинхронные операции для множественных файлов
    public static async Task ProcessFilesAsync(IEnumerable<string> filePaths,
        Func<string, Task> processor, int maxConcurrency = 10)
    {
        using var semaphore = new SemaphoreSlim(maxConcurrency);
        var tasks = filePaths.Select(async filePath =>
        {
            await semaphore.WaitAsync();
            try
            {
                await processor(filePath);
            }
            finally
            {
                semaphore.Release();
            }
        });

        await Task.WhenAll(tasks);
    }

    // 4. Группируйте операции в одной директории
    public static void BatchProcessDirectory(string directoryPath,
        Action<DirectoryInfo, FileInfo[]> processor)
    {
        var directory = new DirectoryInfo(directoryPath);
        var files = directory.GetFiles();
        processor(directory, files);
    }
}

```

## Безопасность файловой системы

### Проверки безопасности

```csharp
public static class FileSystemSecurity
{
    public static bool IsPathSecure(string path, string allowedBasePath)
    {
        try
        {
            string fullPath = Path.GetFullPath(path);
            string fullBasePath = Path.GetFullPath(allowedBasePath);

            return fullPath.StartsWith(fullBasePath, StringComparison.OrdinalIgnoreCase);
        }
        catch
        {
            return false;
        }
    }

    public static bool HasWritePermission(string path)
    {
        try
        {
            using (var fs = File.Create(Path.Combine(path, Path.GetRandomFileName()),
                1, FileOptions.DeleteOnClose))
            {
                return true;
            }
        }
        catch
        {
            return false;
        }
    }

    public static void SetFilePermissions(string filePath, bool readOnly = false)
    {
        var attributes = File.GetAttributes(filePath);

        if (readOnly)
            attributes |= FileAttributes.ReadOnly;
        else
            attributes &= ~FileAttributes.ReadOnly;

        File.SetAttributes(filePath, attributes);
    }

    public static bool IsExecutable(string filePath)
    {
        string extension = Path.GetExtension(filePath).ToLowerInvariant();
        return extension is ".exe" or ".bat" or ".cmd" or ".com" or ".scr" or ".msi";
    }
}

```

## Заключение

Работа с файловой системой в C# предоставляет множество возможностей для эффективного управления файлами и папками. Основные принципы эффективной работы:

- **Всегда используйте Path.Combine()** для построения путей
- **Проверяйте существование файлов и папок** перед операциями
- **Обрабатывайте исключения** - операции с файловой системой могут быть непредсказуемыми
- **Используйте using** для автоматического освобождения ресурсов
- **Кешируйте результаты** для повышения производительности
- **Проверяйте права доступа** и свободное место на диске
- **Используйте асинхронные методы** для операций с множественными файлами
- **Валидируйте пути** для обеспечения безопасности

Правильное использование этих принципов позволит создавать надежные и производительные приложения, работающие с файловой системой.