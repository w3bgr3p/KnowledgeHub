## Класс InstanceExtensions

Статический класс расширений для работы с экземплярами браузера в ZennoPoster. Предоставляет удобные методы для поиска, взаимодействия с элементами веб-страниц, выполнения JavaScript-кода и работы с защитой Cloudflare. Все методы позволяют автоматизировать типичные задачи браузерной автоматизации с упрощенным синтаксисом.

### Публичные методы

**GetHe(object obj, string method = "")**

Возвращает: HtmlElement - найденный HTML-элемент  
Описание: Универсальный поиск HTML-элементов на странице. Поддерживает поиск по различным критериям: готовый HtmlElement, кортежи с id/name или полные атрибуты.  
Параметры:
- obj - объект для поиска (HtmlElement, кортеж с id/name или полный кортеж атрибутов)
- method - дополнительный метод поиска ("last" для последнего элемента)

Пример:
```csharp
// Поиск по id
var element = instance.GetHe(("submitButton", "id"));
project.SendInfoToLog("Элемент найден", false);

// Поиск по атрибуту
var button = instance.GetHe(("button", "class", "submit-btn", "exact", 0));
project.SendInfoToLog("Кнопка найдена", false);
```

**HeGet(object obj, string method = "", int deadline = 10, string atr = "innertext", int delay = 1, string comment = "", bool thr0w = true)**

Возвращает: string - значение атрибута элемента или null  
Описание: Получает значение атрибута HTML-элемента с ожиданием появления. Поддерживает режим проверки отсутствия элемента (method="!").  
Параметры:
- obj - объект для поиска элемента
- method - метод поиска ("!" для проверки отсутствия)
- deadline - время ожидания в секундах
- atr - атрибут для получения
- delay - задержка после нахождения в секундах
- comment - комментарий для ошибок
- thr0w - бросать исключение при неудаче

Пример:
```csharp
// Получить текст элемента
string text = instance.HeGet(("title", "id"), deadline: 15, atr: "innertext");
project.SendInfoToLog($"Текст заголовка: {text}", false);

// Проверить отсутствие элемента
string result = instance.HeGet(("error", "class"), method: "!");
if (result == null)
{
    project.SendInfoToLog("Ошибок нет на странице", false);
}
```

**HeClick(object obj, string method = "", int deadline = 10, int delay = 1, string comment = "", bool thr0w = true, int emu = 0)**

Описание: Кликает по HTML-элементу с ожиданием его появления. Поддерживает специальный режим "clickOut" для клика до исчезновения элемента.  
Параметры:
- obj - объект для поиска элемента
- method - метод поиска ("clickOut" для клика до исчезновения)
- deadline - время ожидания в секундах
- delay - задержка перед кликом в секундах
- comment - комментарий для ошибок
- thr0w - бросать исключение при неудаче
- emu - настройка эмуляции мыши (1 - включить, -1 - выключить, 0 - не менять)

Пример:
```csharp
// Обычный клик
instance.HeClick(("submit", "id"), delay: 2, comment: "Кнопка отправки");
project.SendInfoToLog("Клик выполнен", false);

// Клик до исчезновения popup
instance.HeClick(("popup", "class"), method: "clickOut", comment: "Закрытие popup");
project.SendInfoToLog("Popup закрыт", false);
```

**HeSet(object obj, string value, string method = "id", int deadline = 10, int delay = 1, string comment = "", bool thr0w = true)**

Описание: Устанавливает значение в поле ввода с ожиданием появления элемента.  
Параметры:
- obj - объект для поиска элемента
- value - значение для установки
- method - метод поиска элемента
- deadline - время ожидания в секундах
- delay - задержка перед вводом в секундах
- comment - комментарий для ошибок
- thr0w - бросать исключение при неудаче

Пример:
```csharp
// Ввод в поле email
instance.HeSet(("email", "name"), "user@example.com", delay: 1);
project.SendInfoToLog("Email введен", false);

// Ввод в поле по атрибуту
instance.HeSet(("input", "placeholder", "Введите имя", "exact", 0), "Иван Петров");
project.SendInfoToLog("Имя введено", false);
```

**JsClick(string selector, int delay = 2)**

Возвращает: string - результат выполнения ("Click successful" или ошибка)  
Описание: Выполняет клик по элементу через JavaScript с использованием селектора.  
Параметры:
- selector - JavaScript-селектор элемента
- delay - задержка перед кликом в секундах

Пример:
```csharp
// Клик через querySelector
string result = instance.JsClick("document.querySelector('.submit-button')", delay: 1);
project.SendInfoToLog($"Результат JS клика: {result}", false);

// Клик по элементу по ID
string result2 = instance.JsClick("document.getElementById('confirmBtn')");
project.SendInfoToLog("JS клик выполнен", false);
```

**JsSet(string selector, string value, int delay = 2)**

Возвращает: string - результат выполнения ("Value set successfully" или ошибка)  
Описание: Устанавливает значение в поле через JavaScript с использованием селектора.  
Параметры:
- selector - JavaScript-селектор элемента
- value - значение для установки
- delay - задержка перед установкой в секундах

Пример:
```csharp
// Установка значения через querySelector
string result = instance.JsSet("document.querySelector('input[name=\"username\"]')", "myuser");
project.SendInfoToLog($"Результат JS установки: {result}", false);

// Установка в скрытое поле
string result2 = instance.JsSet("document.getElementById('hiddenField')", "secretValue");
project.SendInfoToLog("Скрытое поле заполнено", false);
```

**JsPost(string script, int delay = 0)**

Возвращает: string - результат выполнения JavaScript или ошибка  
Описание: Выполняет произвольный JavaScript-код на странице.  
Параметры:
- script - JavaScript-код для выполнения
- delay - задержка перед выполнением в секундах

Пример:
```csharp
// Получение URL страницы
string url = instance.JsPost("window.location.href");
project.SendInfoToLog($"Текущий URL: {url}", false);

// Выполнение сложного скрипта
string result = instance.JsPost("document.title + ' - ' + document.readyState", delay: 1);
project.SendInfoToLog($"Информация о странице: {result}", false);
```

**ClFl(int deadline = 60, bool strict = false)**

Возвращает: string - токен Cloudflare или исключение при таймауте  
Описание: Автоматически решает капчу Cloudflare Turnstile и возвращает токен. Метод устарел, рекомендуется использовать CFToken.  
Параметры:
- deadline - максимальное время ожидания в секундах
- strict - строгий режим проверки

Пример:
```csharp
try
{
    string token = instance.ClFl(deadline: 90);
    project.SendInfoToLog($"Cloudflare токен получен: {token}", false);
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка решения CF: {ex.Message}", false);
}
```

**Go(string url, bool strict = false)**

Описание: Переходит на указанный URL только если текущая страница отличается.  
Параметры:
- url - URL для перехода
- strict - строгое сравнение URL (точное совпадение)

Пример:
```csharp
// Переход если страница отличается
instance.Go("https://example.com/login");
project.SendInfoToLog("Переход выполнен", false);

// Строгая проверка URL
instance.Go("https://example.com/dashboard", strict: true);
project.SendInfoToLog("Точный переход выполнен", false);
```

**F5()**

Описание: Обновляет текущую страницу (аналог F5).

Пример:
```csharp
instance.F5();
project.SendInfoToLog("Страница обновлена", false);
```

**CtrlV(string ToPaste)**

Описание: Вставляет текст через буфер обмена (эмуляция Ctrl+V).  
Параметры:
- ToPaste - текст для вставки

Пример:
```csharp
// Вставка текста
instance.CtrlV("Текст для вставки");
project.SendInfoToLog("Текст вставлен", false);

// Вставка пароля
instance.CtrlV("mySecretPassword123");
project.SendInfoToLog("Пароль вставлен", false);
```

**ClearShit(string domain)**

Описание: Очищает кэш, куки и закрывает все вкладки для указанного домена.  
Параметры:
- domain - домен для очистки

Пример:
```csharp
// Полная очистка для домена
instance.ClearShit("example.com");
project.SendInfoToLog("Данные очищены", false);
```

**CloseExtraTabs(bool blank = false, int tabToKeep = 1)**

Описание: Закрывает все вкладки кроме указанной.  
Параметры:
- blank - перейти на пустую страницу после закрытия
- tabToKeep - номер вкладки для сохранения (начиная с 1)

Пример:
```csharp
// Закрыть лишние вкладки
instance.CloseExtraTabs(blank: true);
project.SendInfoToLog("Лишние вкладки закрыты", false);

// Оставить вторую вкладку
instance.CloseExtraTabs(tabToKeep: 2);
project.SendInfoToLog("Оставлена 2-я вкладка", false);
```