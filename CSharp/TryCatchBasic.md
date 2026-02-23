
---

# Try / Catch — Методичка (Базовый уровень)

---

## Зачем вообще try / catch?

Когда что-то идёт не так — программа **падает**. Try/catch позволяет это падение **поймать** и решить что делать дальше.

```csharp
// Без try/catch — скрипт упадёт с ошибкой
string text = project.Variables["price"].Value;
int price = int.Parse(text); // если text = "нет в наличии" — CRASH
```

```csharp
// С try/catch — скрипт продолжит работу
try
{
    string text = project.Variables["price"].Value;
    int price = int.Parse(text);
    project.SendInfoToLog("Цена: " + price);
}
catch
{
    project.SendInfoToLog("Не удалось прочитать цену");
}
```

---

## Базовая структура

```csharp
try
{
    // код который может сломаться
}
catch
{
    // что делать если сломалось
}
```

---

## Получить текст ошибки

```csharp
try
{
    string text = project.Variables["price"].Value;
    int price = int.Parse(text);
}
catch (Exception ex)
{
    project.SendInfoToLog("Ошибка: " + ex.Message);
}
```

`ex.Message` — человекочитаемое описание что пошло не так.

---

## Своё исключение — throw new Exception

Используется когда хочется остановить выполнение и передать своё сообщение:

```csharp
try
{
    string login = project.Variables["login"].Value;

    if (login == "")
    {
        throw new Exception("Логин не заполнен");
    }

    // дальнейшая работа с логином
}
catch (Exception ex)
{
    project.SendInfoToLog("Ошибка: " + ex.Message);
}
```

Удобно тем что не нужно городить цепочку if/return — бросил исключение с понятным текстом, поймал, залогировал.

---

## Finally — выполнится всегда

```csharp
try
{
    // код
}
catch
{
    // если сломалось
}
finally
{
    // выполнится В ЛЮБОМ случае — и при ошибке и без
    project.SendInfoToLog("Блок завершён");
}
```

---

## Ловить конкретный тип ошибки

```csharp
try
{
    int price = int.Parse("нет в наличии");
}
catch (FormatException)
{
    project.SendInfoToLog("Строка не является числом");
}
catch (Exception ex)
{
    project.SendInfoToLog("Какая-то другая ошибка: " + ex.Message);
}
```

Блоки catch проверяются **сверху вниз** — первый подходящий и сработает.  
`Exception` ловит всё — поэтому он всегда идёт **последним**.

---

## Типичные кейсы в зеннопостере

**Парсинг числа:**
```csharp
try
{
    int count = int.Parse(project.Variables["count"].Value);
    project.SendInfoToLog("Количество: " + count);
}
catch
{
    project.SendInfoToLog("Переменная count не число, ставим 0");
    project.Variables["count"].Value = "0";
}
```

**Работа с элементом на странице:**
```csharp
try
{
    var element = instance.ActiveTab.FindElementByXPath("//button[@id='submit']", 0);
    element.Click();
}
catch
{
    project.SendInfoToLog("Кнопка submit не найдена");
}
```

**Вложенные try/catch — каждый шаг отдельно:**
```csharp
try
{
    instance.ActiveTab.Navigate("https://example.com", "");
}
catch
{
    project.SendInfoToLog("Не удалось загрузить страницу");
    return;
}

try
{
    var element = instance.ActiveTab.FindElementByXPath("//h1", 0);
    project.SendInfoToLog("Заголовок: " + element.InnerText);
}
catch
{
    project.SendInfoToLog("Заголовок не найден");
}
```

---

## Пробросить ошибку дальше — throw

```csharp
try
{
    int price = int.Parse(project.Variables["price"].Value);
}
catch (Exception ex)
{
    project.SendInfoToLog("КРИТИЧЕСКАЯ ошибка: " + ex.Message);
    throw;
}
```

---

## Частые ошибки новичков

**Молчащий catch:**
```csharp
// ПЛОХО — ошибка есть, но мы о ней никогда не узнаем
catch
{
}

// ХОРОШО
catch (Exception ex)
{
    project.SendInfoToLog("Ошибка: " + ex.Message);
}
```

**Один большой try на весь код:**
```csharp
// ПЛОХО — непонятно где именно упало
try
{
    // 50 строк кода
}
catch
{
    project.SendInfoToLog("Что-то пошло не так");
}
```

Лучше несколько маленьких try/catch с понятными сообщениями.