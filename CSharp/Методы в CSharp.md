
## Что такое функция (метод)

Функция — это именованный блок кода, который можно вызвать из любого места программы. Вместо того чтобы писать одно и то же несколько раз — пишем функцию один раз и вызываем её когда нужно.

---

## Аргументы

Аргументы — это данные, которые мы передаём в функцию. Они указываются в скобках после имени функции.

```csharp
// Функция без аргументов
void SayHello()
{
    project.SendInfoToLog("Привет!");
}

// Функция с одним аргументом
void SayHelloTo(string name)
{
    project.SendInfoToLog("Привет, " + name);
}

// Вызов
SayHelloTo("Вася"); // напечатает: Привет, Вася
```

Аргументов может быть несколько, они перечисляются через запятую:

```csharp
void SayHelloTo(string name, int age)
{
    project.SendInfoToLog(name + ", тебе " + age + " лет");
}

SayHelloTo("Вася", 25);
```

---

## Типы аргументов

Аргумент — это просто переменная, поэтому он может быть любого типа:

```csharp
void Example(string text, int number, bool flag)
```

Нас больше всего будут интересовать три случая:

**Простые типы** — строка, число, булево:

```csharp
void DoSomething(string tag, string attribute, string value)
```

**Объекты ZennoPoster** — например элемент страницы:

```csharp
void ClickElement(HtmlElement element)
```

**Кортеж** — несколько значений в одном аргументе. Об этом подробнее ниже.

---

## Что функция возвращает

Функция может вернуть результат своей работы. Тип возвращаемого значения пишется перед именем функции.

```csharp
// void — ничего не возвращает, просто выполняет действия
void PrintLog(string message)
{
    project.SendInfoToLog(message);
}

// int — возвращает число
int Add(int a, int b)
{
    return a + b;
}

// string — возвращает строку
string GetGreeting(string name)
{
    return "Привет, " + name;
}

// HtmlElement — возвращает элемент страницы
HtmlElement FindButton(string text)
{
    return instance.ActiveTab.FindElementByAttribute("button", "innertext", text, "regexp", 0);
}
```

Ключевое слово `return` заканчивает выполнение функции и возвращает значение.

---

## Кортеж

Кортеж — это способ сгруппировать несколько значений без создания отдельного класса. Выглядит как перечисление типов в скобках:

```csharp
// Кортеж из трёх строк
(string tag, string attribute, string value)
```

Это удобно когда нужно передать в функцию "набор параметров для поиска" — именно наш случай.

---

## Пишем функции для работы с кошельком

### Задача 1: ожидание и клик по кнопке

Было — код написан прямо в проекте:

```csharp
int maxRetries = 30;
while (instance.ActiveTab.FindElementByAttribute("button", "innertext", "I\\ already\\ have\\ an\\ address", "regexp", 0).IsVoid && maxRetries > 0) 
{
    maxRetries--;
    Thread.Sleep(1000);
}
if (instance.ActiveTab.FindElementByAttribute("button", "innertext", "I\\ already\\ have\\ an\\ address", "regexp", 0).IsVoid)
    throw new Exception("no butn by parametres I\\ already\\ have\\ an\\ address");

instance.ActiveTab.FindElementByAttribute("button", "innertext", "I\\ already\\ have\\ an\\ address", "regexp", 0).RiseEvent("click", instance.EmulationLevel);
```

Стало — выносим в функцию. Функция принимает кортеж с параметрами поиска:

```csharp
void WaitAndClick((string tag, string attribute, string value) selector)
{
    int maxRetries = 30;
    while (instance.ActiveTab.FindElementByAttribute(selector.tag, selector.attribute, selector.value, "regexp", 0).IsVoid && maxRetries > 0)
    {
        maxRetries--;
        Thread.Sleep(1000);
    }

    var element = instance.ActiveTab.FindElementByAttribute(selector.tag, selector.attribute, selector.value, "regexp", 0);

    if (element.IsVoid)
        throw new Exception("Элемент не найден: " + selector.tag + " " + selector.attribute + "=" + selector.value);

    element.RiseEvent("click", instance.EmulationLevel);
}
```

Вызов функции:

```csharp
WaitAndClick(("button", "innertext", "I\\ already\\ have\\ an\\ address"));
```

Теперь для любой другой кнопки достаточно одной строки:

```csharp
WaitAndClick(("button", "innertext", "Create\\ a\\ new\\ wallet"));
WaitAndClick(("button", "innertext", "I\\ agree"));
```

---

### Задача 2: ввод сид-фразы

Было:

```csharp
var inputs = instance.ActiveTab.FindElementsByAttribute("input:password", "fulltagname", "input:password", "regexp").ToList();
var seed = project.Variables["seed"].Value;
var seedParts = seed.Split(' ');
for (int i = 0; i < 12; i++)
{
    inputs[i].SetValue(seedParts[i], "Full", false);
}
```

Стало — выносим в функцию. Сид-фразу передаём аргументом:

```csharp
void EnterSeedPhrase(string seed)
{
    var inputs = instance.ActiveTab.FindElementsByAttribute("input:password", "fulltagname", "input:password", "regexp").ToList();
    var seedParts = seed.Split(' ');

    for (int i = 0; i < seedParts.Length; i++)
    {
        inputs[i].SetValue(seedParts[i], "Full", false);
    }
}
```

Вызов:

```csharp
EnterSeedPhrase(project.Variables["seed"].Value);
```

---

## Итог

|Что|Как выглядит|Пример|
|---|---|---|
|Нет аргументов|`void DoIt()`|`DoIt()`|
|Один аргумент|`void DoIt(string x)`|`DoIt("text")`|
|Несколько аргументов|`void DoIt(string x, int y)`|`DoIt("text", 5)`|
|Кортеж|`void DoIt((string a, string b) t)`|`DoIt(("x", "y"))`|
|Возврат значения|`string GetIt()`|`var r = GetIt()`|

Главный смысл функций — **написал один раз, вызываешь много раз**. Код становится короче, понятнее, и ошибку нужно исправлять только в одном месте.