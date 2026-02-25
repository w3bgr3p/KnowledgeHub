---
created: 2026-02-20
tags:
  - csharp
  - zenno
  - autoz3n
area: course
project: autoZ3N
---


## Что такое циклы?

Цикл - это повторение одних и тех же действий несколько раз. Например, нужно открыть 10 сайтов подряд - вместо копирования кода 10 раз, используем цикл.

---

## 1. Цикл FOR - когда знаем сколько раз повторять

### Базовый синтаксис

csharp

```csharp
for (int i = 0; i < 5; i++)
{
    // код выполнится 5 раз
}
```

**Расшифровка:**

- `int i = 0` - начинаем с нуля
- `i < 5` - повторяем пока i меньше 5 (0,1,2,3,4)
- `i++` - после каждого повтора увеличиваем i на 1


### Пример 1: Пролистать ленту 10 раз

csharp

```csharp
for (int i = 0; i < 10; i++)
{
    instance.ScrollDown();
    System.Threading.Thread.Sleep(1000);
    project.SendInfoToLog("Прокрутка " + (i + 1) + " из 10");
}
```

### Пример 2: Перебор списка по индексу

csharp

```csharp
List<string> cities = new List<string> { "Москва", "Питер", "Казань" };

for (int i = 0; i < cities.Count; i++)
{
    project.SendInfoToLog("Город " + i + ": " + cities[i]);
}
```

---

## 2. Цикл WHILE - повторяем пока условие истинно

### Базовый синтаксис

csharp

```csharp
while (условие)
{
    // код выполняется пока условие = true
}
```

### Пример 1: Ждём появления элемента

csharp

```csharp
int попыток = 0;
while (instance.GetCountElementByXpath("//button[@class='login']") == 0)
{
    System.Threading.Thread.Sleep(500);
    попыток++;
    project.SendInfoToLog("Ожидание кнопки... попытка " + попыток);
    
    if (попыток > 20)
    {
        project.SendInfoToLog("Кнопка не появилась за 10 секунд");
        break; // выходим из цикла
    }
}
```

### Пример 2: Скроллим пока есть кнопка "Загрузить ещё"

csharp

```csharp
while (instance.GetCountElementByXpath("//button[text()='Загрузить ещё']") > 0)
{
    HtmlElement кнопка = instance.GetElementByXpath("//button[text()='Загрузить ещё']");
    кнопка.Click();
    System.Threading.Thread.Sleep(2000);
    project.SendInfoToLog("Подгрузили ещё контент");
}
```

---

## 3. Цикл FOREACH - перебор коллекций

### Базовый синтаксис

csharp

```csharp
foreach (тип переменная in коллекция)
{
    // код для каждого элемента
}
```

### Пример 1: Перебор списка товаров

csharp

```csharp
List<string> товары = new List<string> { "iPhone", "Samsung", "Xiaomi" };

foreach (string товар in товары)
{
    project.SendInfoToLog("Ищем товар: " + товар);
    instance.SearchElement(товар);
}
```

### Пример 2: Клик по всем ссылкам

csharp

```csharp
HtmlElementCollection ссылки = instance.GetElementsByXpath("//a[@class='product-link']");

foreach (HtmlElement ссылка in ссылки)
{
    string url = ссылка.GetAttribute("href");
    project.SendInfoToLog("Найдена ссылка: " + url);
}
```

### Пример 3: Обработка таблицы

csharp

```csharp
List<string> строки = project.Lists["emails"];

foreach (string email in строки)
{
    project.SendInfoToLog("Обработка: " + email);
    instance.GetElementByXpath("//input[@type='email']").SetValue(email);
}
```

---

## 4. DO-WHILE - сначала делаем, потом проверяем

### Базовый синтаксис

csharp

```csharp
do
{
    // код выполнится минимум 1 раз
}
while (условие);
```

### Пример: Решение капчи с повторами

csharp

```csharp
bool решена = false;

do
{
    string результат = SolveRecaptcha();
    
    if (результат != "")
    {
        решена = true;
        project.SendInfoToLog("Капча решена!");
    }
    else
    {
        project.SendInfoToLog("Капча не решилась, пробуем снова...");
        System.Threading.Thread.Sleep(2000);
    }
}
while (!решена);
```

---

## 5. Управление циклами

### BREAK - выход из цикла

csharp

```csharp
for (int i = 0; i < 100; i++)
{
    if (instance.GetCountElementByXpath("//div[@class='captcha']") > 0)
    {
        project.SendInfoToLog("Обнаружена капча, останавливаем цикл");
        break; // полностью выходим из цикла
    }
    
    instance.ScrollDown();
}
```

### CONTINUE - пропуск итерации

csharp

```csharp
List<string> urls = project.Lists["sites"];

foreach (string url in urls)
{
    if (url == "")
    {
        continue; // пропускаем пустые строки
    }
    
    instance.GoToUrl(url);
    project.SendInfoToLog("Обработан: " + url);
}
```

---

## 6. Вложенные циклы

### Пример: Два уровня вложенности

csharp

```csharp
List<string> категории = new List<string> { "Одежда", "Обувь" };
List<string> бренды = new List<string> { "Nike", "Adidas" };

foreach (string категория in категории)
{
    foreach (string бренд in бренды)
    {
        string запрос = категория + " " + бренд;
        project.SendInfoToLog("Ищем: " + запрос);
        instance.SearchElement(запрос);
    }
}
```

---

## Практические задания

### Задание 1

Напишите цикл, который откроет 7 страниц новостей (page=1, page=2... page=7)

### Задание 2

Создайте цикл while, который будет скроллить страницу вниз пока не найдёт элемент с классом "footer"

### Задание 3

Есть список email'ов в project.Lists["emails"]. Переберите его через foreach и выведите каждый email в лог

---

## Частые ошибки

❌ **Бесконечный цикл**

csharp

```csharp
int i = 0;
while (i < 10)
{
    project.SendInfoToLog("i = " + i);
    // забыли написать i++
}
```

❌ **Неправильное условие**

csharp

```csharp
for (int i = 0; i <= 5; i++) // выполнится 6 раз (0,1,2,3,4,5)
```

✅ **Правильно**

csharp

```csharp
for (int i = 0; i < 5; i++) // выполнится 5 раз (0,1,2,3,4)
```