# Методическое руководство по работе со строками в C# (версия 6 и ниже)

## Введение: Почему важно правильно работать со строками

Строки в C# — это неизменяемые (immutable) объекты. Это означает, что каждый раз, когда вы "изменяете" строку, на самом деле создается новый объект в памяти. Понимание этого факта критически важно для написания эффективного кода.

Представьте строку как высеченную в камне надпись. Вы не можете изменить букву в этой надписи — вам придется высечь новую надпись целиком. Именно поэтому операции со строками могут быть дорогостоящими с точки зрения производительности.

## Часть 1: Основные методы работы со строками

### 1.1 Методы форматирования и очистки

#### **Trim(), TrimStart(), TrimEnd()**
Эти методы удаляют пробельные символы (пробелы, табуляции, переносы строк) с начала и/или конца строки.

```csharp
string userInput = "  Иван Петров   ";
string cleanName = userInput.Trim(); // "Иван Петров"

// Можно указать конкретные символы для удаления
string data = "---данные---";
string cleaned = data.Trim('-'); // "данные"
```

**Когда использовать:** Всегда применяйте Trim() при обработке пользовательского ввода из форм, файлов или баз данных. Пользователи часто случайно добавляют лишние пробелы, и это может привести к проблемам при сравнении строк или сохранении в базу данных.

#### **Replace()**
Заменяет все вхождения подстроки или символа на другую подстроку или символ.

```csharp
string template = "Здравствуйте, {name}! Ваш заказ {order} готов.";
string message = template.Replace("{name}", "Александр")
                         .Replace("{order}", "#12345");
// Результат: "Здравствуйте, Александр! Ваш заказ #12345 готов."
```

**Важное замечание:** Replace() создает новую строку при каждом вызове. Если нужно выполнить множество замен, рассмотрите использование StringBuilder (см. раздел 2).

### 1.2 Методы поиска и проверки

#### **Contains(), StartsWith(), EndsWith()**
Эти методы проверяют наличие подстроки в строке.

```csharp
string fileName = "document.pdf";

if (fileName.EndsWith(".pdf"))
{
    // Это PDF-файл, обрабатываем соответствующим образом
}

string email = "user@example.com";
if (!email.Contains("@"))
{
    // Email невалидный
}
```

#### **IndexOf() и LastIndexOf()**
Находят позицию первого или последнего вхождения подстроки.

```csharp
string path = "C:/Users/Documents/file.txt";
int lastSlash = path.LastIndexOf('/');
string fileName = path.Substring(lastSlash + 1); // "file.txt"

// IndexOf возвращает -1, если подстрока не найдена
if (text.IndexOf("ошибка") != -1)
{
    // В тексте есть слово "ошибка"
}
```

### 1.3 Методы преобразования

#### **ToUpper(), ToLower()**
Преобразуют регистр строки. Критически важны при сравнении строк без учета регистра.

```csharp
string userAnswer = "ДА";
string normalized = userAnswer.ToLower();

if (normalized == "да" || normalized == "yes")
{
    // Пользователь согласился
}
```

**Важный момент:** Для сравнения без учета регистра лучше использовать специальные методы сравнения:

```csharp
// Более правильный подход
if (string.Equals(userAnswer, "да", StringComparison.OrdinalIgnoreCase))
{
    // Сравнение без учета регистра и более эффективное
}
```

#### **Substring()**
Извлекает часть строки.

```csharp
string phoneNumber = "+7-495-123-45-67";
string countryCode = phoneNumber.Substring(0, 2);  // "+7"
string localNumber = phoneNumber.Substring(3);      // "495-123-45-67"
```

### 1.4 Методы разделения и объединения

#### **Split()**
Разделяет строку на массив подстрок по указанному разделителю.

```csharp
string csvLine = "Иванов;Петр;35;Москва";
string[] parts = csvLine.Split(';');
// parts[0] = "Иванов", parts[1] = "Петр", и т.д.

// Можно использовать несколько разделителей
string text = "слово1, слово2; слово3 | слово4";
string[] words = text.Split(new char[] {',', ';', '|'});
```

#### **Join()**
Объединяет элементы массива в строку с указанным разделителем.

```csharp
string[] tags = {"C#", "программирование", "разработка"};
string tagString = string.Join(", ", tags);
// Результат: "C#, программирование, разработка"
```

### 1.5 Методы парсинга

#### **Parse() и TryParse()**
Преобразуют строку в другой тип данных. TryParse безопаснее, так как не выбрасывает исключение при ошибке.

```csharp
// Опасный способ (может выбросить исключение)
string ageText = "25";
int age = int.Parse(ageText);

// Безопасный способ
string userInput = "не число";
int value;
if (int.TryParse(userInput, out value))
{
    // Преобразование успешно, value содержит число
    Console.WriteLine($"Вы ввели число: {value}");
}
else
{
    // Преобразование не удалось
    Console.WriteLine("Введите корректное число");
}

// Работа с датами
DateTime date;
if (DateTime.TryParse("2024-01-15", out date))
{
    // Дата успешно распознана
}
```

**Правило:** Всегда используйте TryParse при работе с пользовательским вводом. Parse используйте только когда уверены в формате данных (например, при чтении конфигурационных файлов).

## Часть 2: StringBuilder — когда и зачем

### Проблема конкатенации строк

Рассмотрим простой пример:

```csharp
// ПЛОХОЙ подход
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += "Строка " + i + "\n";  // Создается 3000+ промежуточных строк!
}
```

При каждой операции `+=` создается новая строка в памяти. Для 1000 итераций это означает создание тысяч промежуточных объектов, что приводит к:
- Замедлению работы программы
- Излишней нагрузке на сборщик мусора
- Избыточному использованию памяти

### Решение: StringBuilder

StringBuilder — это изменяемый буфер символов. Он работает как динамический массив, который может эффективно изменять свое содержимое.

```csharp
// ХОРОШИЙ подход
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append("Строка ");
    sb.Append(i);
    sb.AppendLine();  // Добавляет перенос строки
}
string result = sb.ToString();
```

### Основные методы StringBuilder

```csharp
StringBuilder sb = new StringBuilder();

// Добавление в конец
sb.Append("Текст");
sb.AppendLine("Строка с переносом");
sb.AppendFormat("Число: {0}, Дата: {1}", 42, DateTime.Now);

// Вставка в произвольную позицию
sb.Insert(0, "Начало: ");

// Замена
sb.Replace("старый", "новый");

// Удаление
sb.Remove(0, 5);  // Удаляет 5 символов начиная с позиции 0

// Очистка
sb.Clear();

// Установка длины (обрезка)
sb.Length = 10;  // Оставляет только первые 10 символов
```

### Когда использовать StringBuilder

**Используйте StringBuilder когда:**
- Выполняете множественные операции конкатенации (более 5-10)
- Строите строку в цикле
- Генерируете отчеты, HTML, XML или другой текстовый контент
- Работаете с большими объемами текста

**НЕ используйте StringBuilder когда:**
- Выполняете 2-4 простые конкатенации
- Работаете с константными строками
- Используете string.Format() или интерполяцию строк (в C# 6)

```csharp
// Для простых случаев обычная конкатенация приемлема
string fullName = firstName + " " + lastName;

// Или используйте string.Format
string message = string.Format("Привет, {0}!", userName);

// В C# 6 доступна интерполяция строк
string greeting = $"Привет, {userName}! Сегодня {DateTime.Now:dd.MM.yyyy}";
```

## Часть 3: Работа с путями файловой системы — класс Path

Класс Path предоставляет методы для безопасной работы с путями файлов и директорий. Никогда не собирайте пути вручную через конкатенацию строк!

### Основные методы Path

```csharp
// Объединение частей пути (учитывает разделители ОС)
string folder = @"C:\Users\Documents";
string fileName = "report.txt";
string fullPath = Path.Combine(folder, fileName);
// Результат: C:\Users\Documents\report.txt

// Получение компонентов пути
string filePath = @"C:\Projects\MyApp\src\main.cs";
string directory = Path.GetDirectoryName(filePath);   // C:\Projects\MyApp\src
string fileName = Path.GetFileName(filePath);         // main.cs
string fileNameWithoutExt = Path.GetFileNameWithoutExtension(filePath); // main
string extension = Path.GetExtension(filePath);       // .cs

// Изменение расширения
string newPath = Path.ChangeExtension(filePath, ".txt");
// Результат: C:\Projects\MyApp\src\main.txt

// Получение полного пути из относительного
string relativePath = @"..\..\file.txt";
string absolutePath = Path.GetFullPath(relativePath);

// Генерация уникального имени временного файла
string tempFile = Path.GetTempFileName();

// Получение пути к временной директории
string tempPath = Path.GetTempPath();
```

### Почему важно использовать Path

```csharp
// ПЛОХО: Ручная конкатенация
string badPath = folder + "\\" + subfolder + "\\" + file;
// Проблемы: 
// - Не работает на Linux/Mac (там разделитель /)
// - Двойные слеши при ошибках
// - Нет проверки корректности

// ХОРОШО: Использование Path.Combine
string goodPath = Path.Combine(folder, subfolder, file);
// Преимущества:
// - Кроссплатформенность
// - Автоматическая обработка разделителей
// - Обработка пустых и null значений
```

## Часть 4: Практические сценарии и оптимизация

### Сценарий 1: Обработка CSV-файла

```csharp
// Задача: Прочитать CSV и построить отчет
StringBuilder report = new StringBuilder();
report.AppendLine("Отчет по продажам");
report.AppendLine(new string('-', 50));

string[] lines = File.ReadAllLines("sales.csv");
foreach (string line in lines.Skip(1)) // Пропускаем заголовок
{
    string[] parts = line.Split(',');
    if (parts.Length >= 3)
    {
        string product = parts[0].Trim();
        int quantity;
        decimal price;
        
        if (int.TryParse(parts[1].Trim(), out quantity) &&
            decimal.TryParse(parts[2].Trim(), out price))
        {
            decimal total = quantity * price;
            report.AppendFormat("{0,-20} {1,10:C}\n", product, total);
        }
    }
}

string finalReport = report.ToString();
```

### Сценарий 2: Валидация и нормализация email

```csharp
public static string NormalizeEmail(string email)
{
    if (string.IsNullOrWhiteSpace(email))
        return null;
    
    // Убираем пробелы и приводим к нижнему регистру
    email = email.Trim().ToLower();
    
    // Базовая проверка формата
    int atIndex = email.IndexOf('@');
    int lastDotIndex = email.LastIndexOf('.');
    
    if (atIndex <= 0 || 
        lastDotIndex <= atIndex || 
        lastDotIndex >= email.Length - 2)
    {
        return null; // Невалидный формат
    }
    
    return email;
}
```

### Сценарий 3: Генерация HTML-отчета

```csharp
public static string GenerateHtmlReport(List<Product> products)
{
    // Для генерации HTML всегда используем StringBuilder
    StringBuilder html = new StringBuilder();
    
    html.AppendLine("<!DOCTYPE html>");
    html.AppendLine("<html>");
    html.AppendLine("<head><title>Отчет</title></head>");
    html.AppendLine("<body>");
    html.AppendLine("<table border='1'>");
    html.AppendLine("<tr><th>Название</th><th>Цена</th><th>Количество</th></tr>");
    
    foreach (var product in products)
    {
        html.AppendFormat("<tr><td>{0}</td><td>{1:C}</td><td>{2}</td></tr>\n",
            System.Net.WebUtility.HtmlEncode(product.Name),  // Экранирование HTML
            product.Price,
            product.Quantity);
    }
    
    html.AppendLine("</table>");
    html.AppendLine("</body>");
    html.AppendLine("</html>");
    
    return html.ToString();
}
```

### Сценарий 4: Работа с путями в кроссплатформенном приложении

```csharp
public static string GetDataFilePath(string fileName)
{
    // Получаем базовую директорию приложения
    string baseDir = AppDomain.CurrentDomain.BaseDirectory;
    
    // Создаем путь к папке данных
    string dataDir = Path.Combine(baseDir, "Data");
    
    // Проверяем существование директории
    if (!Directory.Exists(dataDir))
    {
        Directory.CreateDirectory(dataDir);
    }
    
    // Возвращаем полный путь к файлу
    return Path.Combine(dataDir, fileName);
}

// Использование
string configPath = GetDataFilePath("config.json");
string logPath = GetDataFilePath($"log_{DateTime.Now:yyyy-MM-dd}.txt");
```

## Часть 5: Советы по производительности и лучшие практики

### 1. Сравнение строк

```csharp
// Для точного сравнения используйте Ordinal
bool areEqual = string.Equals(str1, str2, StringComparison.Ordinal);

// Для сравнения без учета регистра
bool areEqualIgnoreCase = string.Equals(str1, str2, 
    StringComparison.OrdinalIgnoreCase);

// Для сравнения с учетом культуры (для пользовательского интерфейса)
bool areEqualCulture = string.Equals(str1, str2, 
    StringComparison.CurrentCulture);
```

### 2. Проверка на пустоту

```csharp
// Используйте специальные методы вместо сравнения с ""
if (string.IsNullOrEmpty(text))
{
    // Строка null или пустая
}

if (string.IsNullOrWhiteSpace(text))
{
    // Строка null, пустая или содержит только пробелы
    // Это предпочтительный метод для валидации пользовательского ввода
}
```

### 3. Оптимизация StringBuilder

```csharp
// Если знаете примерный размер, укажите начальную емкость
StringBuilder sb = new StringBuilder(1000);

// Это предотвратит множественные перевыделения памяти
// при росте внутреннего буфера
```

### 4. Интернирование строк

```csharp
// Для часто используемых строк можно использовать интернирование
string internedStr = string.Intern(someString);

// Но будьте осторожны — интернированные строки 
// остаются в памяти до конца работы приложения
```

### 5. Форматирование строк

```csharp
// В порядке предпочтения для C# 6:

// 1. Интерполяция строк (C# 6) — самый читаемый способ
string message = $"Пользователь {userName} вошел в {loginTime:HH:mm}";

// 2. String.Format — универсальный способ
string message = string.Format("Пользователь {0} вошел в {1:HH:mm}", 
    userName, loginTime);

// 3. StringBuilder — для сложной логики построения
StringBuilder sb = new StringBuilder();
sb.AppendFormat("Пользователь {0} ", userName);
sb.AppendFormat("вошел в {0:HH:mm}", loginTime);

// 4. Конкатенация — только для 2-3 простых строк
string message = "Пользователь " + userName + " вошел";
```

## Заключение: Ключевые принципы

1. **Помните о неизменяемости строк.** Каждая "модификация" создает новый объект.

2. **Используйте StringBuilder для множественных операций.** Это не преждевременная оптимизация, а базовая грамотность.

3. **Всегда используйте Path для работы с путями.** Это гарантирует кроссплатформенность и безопасность.

4. **Применяйте TryParse для пользовательского ввода.** Исключения дорого обходятся производительности.

5. **Выбирайте правильный метод сравнения строк.** StringComparison.Ordinal для технических строк, CurrentCulture для UI.

6. **Валидируйте и нормализуйте входные данные.** Trim() и проверка на IsNullOrWhiteSpace — ваши друзья.

7. **Думайте о производительности заранее.** Правильный выбор между string и StringBuilder может ускорить программу в десятки раз.

Эти принципы помогут вам писать не только работающий, но и эффективный, поддерживаемый код. Помните: строки — один из самых используемых типов данных, и правильная работа с ними — признак профессионализма разработчика.