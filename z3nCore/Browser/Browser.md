[SPOILER= Browser] 


[SPOILER= BrowserScan]

# BrowserScan

Класс для работы с сервисом BrowserScan.net в ZennoPoster. Позволяет анализировать отпечаток браузера, получать оценку анонимности, парсить характеристики браузера и автоматически исправлять настройки временной зоны для лучшей маскировки.

## Конструкторы

### BrowserScan(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса BrowserScan.

**Параметры:**
- `project` - модель проекта ZennoPoster
- `instance` - экземпляр браузера
- `log` - включить логирование (по умолчанию false)

**Пример:**
[CODE=csharp]
var browserScan = new BrowserScan(project, instance, true);
//создать объект для работы с BrowserScan с включенным логированием
[/CODE]

## Методы

### ParseStats()

**Возвращаемое значение:** `void`

Загружает страницу BrowserScan.net, парсит все характеристики браузера (WebGL, аудио, шрифты, часовой пояс и др.) и сохраняет их в таблицу базы данных проекта.

**Пример:**
[CODE=csharp]
browserScan.ParseStats();
//проанализировать и сохранить все характеристики браузера
[/CODE]

### GetScore()

**Возвращаемое значение:** `string`

Получает оценку анонимности браузера от BrowserScan.net. Если оценка не 100%, дополнительно возвращает список проблем с их описанием.

**Пример:**
[CODE=csharp]
string score = browserScan.GetScore();
project.SendInfoToLog($"Оценка браузера: {score}");
//получить оценку анонимности браузера
[/CODE]

### FixTime()

**Возвращаемое значение:** `void`

Автоматически исправляет настройки временной зоны браузера на основе данных, полученных от BrowserScan.net. Устанавливает корректный часовой пояс и смещение времени.

**Пример:**
[CODE=csharp]
browserScan.FixTime();
//автоматически настроить временную зону браузера
[/CODE][/SPOILER]

[SPOILER= Captcha]

# Captcha

Статический класс для работы с различными видами captcha, включая Cloudflare Turnstile и интеграцию с сервисом Cap.Guru.

## Методы

### CFSolve

[CODE=csharp]
public static void CFSolve(this Instance instance)
[/CODE]

**Возвращаемое значение:** `void`

**Описание:** Автоматически решает Cloudflare Turnstile captcha путем клика по защитному элементу на странице.

**Пример использования:**
[CODE=csharp]
//решить Cloudflare captcha
instance.CFSolve();
[/CODE]

---

### CFToken

[CODE=csharp]
public static string CFToken(this Instance instance, int deadline = 60, bool strict = false)
[/CODE]

**Возвращаемое значение:** `string` - токен от решённой Cloudflare captcha

**Описание:** Получает токен от решённой Cloudflare Turnstile captcha с возможностью настройки времени ожидания.

**Параметры:**
- `deadline` (`int`) - максимальное время ожидания решения в секундах (по умолчанию 60)
- `strict` (`bool`) - строгий режим проверки (по умолчанию false)

**Пример использования:**
[CODE=csharp]
//получить токен с ожиданием 120 секунд
string token = instance.CFToken(120);
project.SendInfoToLog($"Получен токен: {token}");
[/CODE]

---

### CapGuru

[CODE=csharp]
public static bool CapGuru(this IZennoPosterProjectModel project)
[/CODE]

**Возвращаемое значение:** `bool` - результат выполнения плагина Cap.Guru (true - успешно, false - ошибка)

**Описание:** Запускает плагин Cap.Guru для решения различных типов captcha. Автоматически использует API ключ из базы данных проекта.

**Пример использования:**
[CODE=csharp]
//использовать Cap.Guru для решения captcha
bool result = project.CapGuru();
if (result)
{
    project.SendInfoToLog("Cap.Guru успешно решил captcha");
}
else
{
    project.SendErrorToLog("Ошибка при решении captcha через Cap.Guru");
}
[/CODE][/SPOILER]

[SPOILER= ChromeExt]

# Класс ChromeExt

## Описание
Этот класс предоставляет инструменты для управления расширениями Chrome в проектах ZennoPoster. Он позволяет устанавливать расширения из CRX-файлов, переключать их состояние (включать или отключать), удалять расширения и получать версию установленного расширения. Класс работает как утилита для автоматизации взаимодействия с браузерными расширениями, упрощая настройку профиля браузера.

## Конструкторы

### ChromeExt(IZennoPosterProjectModel project, bool log = false)
- **Описание**: Создает экземпляр класса для управления расширениями с указанным проектом и опцией логирования.
- **Параметры**: 
  - `project` - модель проекта ZennoPoster
  - `log` - включить/выключить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  var chromeExt = new ChromeExt(project, true);
  project.SendInfoToLog("Экземпляр ChromeExt создан с логированием", false);
  [/CODE]

### ChromeExt(IZennoPosterProjectModel project, Instance instance, bool log = false)
- **Описание**: Создает экземпляр класса для управления расширениями с указанным проектом, экземпляром браузера и опцией логирования.
- **Параметры**:
  - `project` - модель проекта ZennoPoster
  - `instance` - экземпляр браузера
  - `log` - включить/выключить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  var chromeExt = new ChromeExt(project, instance, false);
  project.SendInfoToLog("Экземпляр ChromeExt создан без логирования", false);
  [/CODE]

## Публичные методы

### GetVer(string extId)
- **Возвращает**: string - версия расширения
- **Описание**: Получает версию указанного расширения по его ID из настроек профиля.
- **Параметры**:
  - `extId` - ID расширения
- **Пример**:
  [CODE=csharp]
  string version = chromeExt.GetVer("pbgjpgbpljobkekbhnnmlikbbfhbhmem");
  project.SendInfoToLog($"Версия расширения: {version}", false);
  [/CODE]

### Install(string extId, string fileName, bool log = false)
- **Возвращает**: bool - успешность установки
- **Описание**: Устанавливает расширение из CRX-файла, если оно еще не установлено.
- **Параметры**:
  - `extId` - ID расширения
  - `fileName` - имя CRX-файла
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  bool installed = chromeExt.Install("pbgjpgbpljobkekbhnnmlikbbfhbhmem", "One-Click-Extensions-Manager.crx", true);
  if (installed)
  {
      project.SendInfoToLog("Расширение установлено", false);
  }
  else
  {
      project.SendWarningToLog("Расширение уже установлено", false);
  }
  [/CODE]

### Switch(string toUse = "", bool log = false)
- **Описание**: Переключает состояние расширений (включает или отключает) на основе списка имен или ID.
- **Параметры**:
  - `toUse` - список расширений для включения (имена или ID через запятую)
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  try
  {
      chromeExt.Switch("Wallet,ExtensionManager", true);
      project.SendInfoToLog("Расширения переключены", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка переключения: {ex.Message}", false);
  }
  [/CODE]

### Rm(string[] ExtToRemove)
- **Описание**: Удаляет массив указанных расширений по их ID.
- **Параметры**:
  - `ExtToRemove` - массив ID расширений для удаления
- **Пример**:
  [CODE=csharp]
  string[] exts = { "id1", "id2" };
  chromeExt.Rm(exts);
  project.SendInfoToLog("Расширения удалены", false);
  [/CODE]

[/SPOILER]

[SPOILER= Cookies]

# Класс Cookies

## Описание
Класс `Cookies` предоставляет инструменты для работы с cookies в проектах ZennoPoster. Он позволяет получать, устанавливать и сохранять cookies для текущего домена или всех доменов, а также управлять cookies через JavaScript. Класс упрощает автоматизацию задач, связанных с сохранением и загрузкой cookies, включая взаимодействие с файлами и базами данных.

## Конструкторы

### Cookies(IZennoPosterProjectModel project, Instance instance, bool log = false)
- **Описание**: Создает экземпляр класса для управления cookies с указанным проектом, экземпляром браузера и опцией логирования.
- **Параметры**:
  - `project` - модель проекта ZennoPoster
  - `instance` - экземпляр браузера
  - `log` - включить/выключить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  var cookies = new Cookies(project, instance, true);
  project.SendInfoToLog("Экземпляр Cookies создан с логированием", false);
  [/CODE]

## Публичные методы

### Get(string domainFilter = "")
- **Возвращает**: string - JSON-строка с данными cookies
- **Описание**: Получает cookies для указанного домена или всех доменов и возвращает их в формате JSON. Если домен не указан, возвращаются cookies для текущего активного домена.
- **Параметры**:
  - `domainFilter` - фильтр домена (по умолчанию пустая строка, возвращает cookies для текущего домена)
- **Пример**:
  [CODE=csharp]
  string cookiesJson = cookies.Get(".example.com");
  project.SendInfoToLog($"Получены cookies: {cookiesJson}", false);
  [/CODE]

### Set(string cookieSourse = null, string jsonPath = null)
- **Описание**: Устанавливает cookies из указанного источника (база данных или файл).
- **Параметры**:
  - `cookieSourse` - источник cookies (`dbMain`, `dbProject`, `fromFile` или JSON-строка)
  - `jsonPath` - путь к файлу с cookies (используется, если `cookieSourse` = `fromFile`)
- **Пример**:
  [CODE=csharp]
  try
  {
      cookies.Set("fromFile", "C:\\profiles\\cookies\\account.json");
      project.SendInfoToLog("Cookies установлены из файла", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка установки cookies: {ex.Message}", false);
  }
  [/CODE]

### Save(string source = null, string jsonPath = null)
- **Описание**: Сохраняет cookies в базу данных или файл. Поддерживает сохранение cookies текущего проекта или всех cookies.
- **Параметры**:
  - `source` - источник cookies (`project` для текущего домена, `all` для всех доменов)
  - `jsonPath` - путь для сохранения cookies в файл (если указан)
- **Пример**:
  [CODE=csharp]
  try
  {
      cookies.Save("all", "C:\\profiles\\cookies\\all_cookies.json");
      project.SendInfoToLog("Cookies сохранены", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка сохранения cookies: {ex.Message}", false);
  }
  [/CODE]

### GetByJs(string domainFilter = "", bool log = false)
- **Возвращает**: string - JSON-строка с данными cookies, полученных через JavaScript
- **Описание**: Получает cookies текущей страницы через выполнение JavaScript-кода в браузере.
- **Параметры**:
  - `domainFilter` - фильтр домена (по умолчанию пустая строка, возвращает cookies текущего домена)
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  string jsCookies = cookies.GetByJs(".example.com", true);
  project.SendInfoToLog($"Cookies получены через JS: {jsCookies}", false);
  [/CODE]

### SetByJs(string cookiesJson, bool log = false)
- **Описание**: Устанавливает cookies через выполнение JavaScript-кода в браузере.
- **Параметры**:
  - `cookiesJson` - JSON-строка с данными cookies
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  [CODE=csharp]
  try
  {
      string json = @"[{""domain"":"".example.com"",""name"":""session"",""value"":""12345""}]";
      cookies.SetByJs(json, true);
      project.SendInfoToLog("Cookies установлены через JS", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка установки cookies через JS: {ex.Message}", false);
  }
  [/CODE][/SPOILER]

[SPOILER= Instance.Extentions]

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
[CODE=csharp]
// Поиск по id
var element = instance.GetHe(("submitButton", "id"));
project.SendInfoToLog("Элемент найден", false);

// Поиск по атрибуту
var button = instance.GetHe(("button", "class", "submit-btn", "exact", 0));
project.SendInfoToLog("Кнопка найдена", false);
[/CODE]

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
[CODE=csharp]
// Получить текст элемента
string text = instance.HeGet(("title", "id"), deadline: 15, atr: "innertext");
project.SendInfoToLog($"Текст заголовка: {text}", false);

// Проверить отсутствие элемента
string result = instance.HeGet(("error", "class"), method: "!");
if (result == null)
{
    project.SendInfoToLog("Ошибок нет на странице", false);
}
[/CODE]

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
[CODE=csharp]
// Обычный клик
instance.HeClick(("submit", "id"), delay: 2, comment: "Кнопка отправки");
project.SendInfoToLog("Клик выполнен", false);

// Клик до исчезновения popup
instance.HeClick(("popup", "class"), method: "clickOut", comment: "Закрытие popup");
project.SendInfoToLog("Popup закрыт", false);
[/CODE]

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
[CODE=csharp]
// Ввод в поле email
instance.HeSet(("email", "name"), "user@example.com", delay: 1);
project.SendInfoToLog("Email введен", false);

// Ввод в поле по атрибуту
instance.HeSet(("input", "placeholder", "Введите имя", "exact", 0), "Иван Петров");
project.SendInfoToLog("Имя введено", false);
[/CODE]

**JsClick(string selector, int delay = 2)**

Возвращает: string - результат выполнения ("Click successful" или ошибка)  
Описание: Выполняет клик по элементу через JavaScript с использованием селектора.  
Параметры:
- selector - JavaScript-селектор элемента
- delay - задержка перед кликом в секундах

Пример:
[CODE=csharp]
// Клик через querySelector
string result = instance.JsClick("document.querySelector('.submit-button')", delay: 1);
project.SendInfoToLog($"Результат JS клика: {result}", false);

// Клик по элементу по ID
string result2 = instance.JsClick("document.getElementById('confirmBtn')");
project.SendInfoToLog("JS клик выполнен", false);
[/CODE]

**JsSet(string selector, string value, int delay = 2)**

Возвращает: string - результат выполнения ("Value set successfully" или ошибка)  
Описание: Устанавливает значение в поле через JavaScript с использованием селектора.  
Параметры:
- selector - JavaScript-селектор элемента
- value - значение для установки
- delay - задержка перед установкой в секундах

Пример:
[CODE=csharp]
// Установка значения через querySelector
string result = instance.JsSet("document.querySelector('input[name=\"username\"]')", "myuser");
project.SendInfoToLog($"Результат JS установки: {result}", false);

// Установка в скрытое поле
string result2 = instance.JsSet("document.getElementById('hiddenField')", "secretValue");
project.SendInfoToLog("Скрытое поле заполнено", false);
[/CODE]

**JsPost(string script, int delay = 0)**

Возвращает: string - результат выполнения JavaScript или ошибка  
Описание: Выполняет произвольный JavaScript-код на странице.  
Параметры:
- script - JavaScript-код для выполнения
- delay - задержка перед выполнением в секундах

Пример:
[CODE=csharp]
// Получение URL страницы
string url = instance.JsPost("window.location.href");
project.SendInfoToLog($"Текущий URL: {url}", false);

// Выполнение сложного скрипта
string result = instance.JsPost("document.title + ' - ' + document.readyState", delay: 1);
project.SendInfoToLog($"Информация о странице: {result}", false);
[/CODE]

**ClFl(int deadline = 60, bool strict = false)**

Возвращает: string - токен Cloudflare или исключение при таймауте  
Описание: Автоматически решает капчу Cloudflare Turnstile и возвращает токен. Метод устарел, рекомендуется использовать CFToken.  
Параметры:
- deadline - максимальное время ожидания в секундах
- strict - строгий режим проверки

Пример:
[CODE=csharp]
try
{
    string token = instance.ClFl(deadline: 90);
    project.SendInfoToLog($"Cloudflare токен получен: {token}", false);
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка решения CF: {ex.Message}", false);
}
[/CODE]

**Go(string url, bool strict = false)**

Описание: Переходит на указанный URL только если текущая страница отличается.  
Параметры:
- url - URL для перехода
- strict - строгое сравнение URL (точное совпадение)

Пример:
[CODE=csharp]
// Переход если страница отличается
instance.Go("https://example.com/login");
project.SendInfoToLog("Переход выполнен", false);

// Строгая проверка URL
instance.Go("https://example.com/dashboard", strict: true);
project.SendInfoToLog("Точный переход выполнен", false);
[/CODE]

**F5()**

Описание: Обновляет текущую страницу (аналог F5).

Пример:
[CODE=csharp]
instance.F5();
project.SendInfoToLog("Страница обновлена", false);
[/CODE]

**CtrlV(string ToPaste)**

Описание: Вставляет текст через буфер обмена (эмуляция Ctrl+V).  
Параметры:
- ToPaste - текст для вставки

Пример:
[CODE=csharp]
// Вставка текста
instance.CtrlV("Текст для вставки");
project.SendInfoToLog("Текст вставлен", false);

// Вставка пароля
instance.CtrlV("mySecretPassword123");
project.SendInfoToLog("Пароль вставлен", false);
[/CODE]

**ClearShit(string domain)**

Описание: Очищает кэш, куки и закрывает все вкладки для указанного домена.  
Параметры:
- domain - домен для очистки

Пример:
[CODE=csharp]
// Полная очистка для домена
instance.ClearShit("example.com");
project.SendInfoToLog("Данные очищены", false);
[/CODE]

**CloseExtraTabs(bool blank = false, int tabToKeep = 1)**

Описание: Закрывает все вкладки кроме указанной.  
Параметры:
- blank - перейти на пустую страницу после закрытия
- tabToKeep - номер вкладки для сохранения (начиная с 1)

Пример:
[CODE=csharp]
// Закрыть лишние вкладки
instance.CloseExtraTabs(blank: true);
project.SendInfoToLog("Лишние вкладки закрыты", false);

// Оставить вторую вкладку
instance.CloseExtraTabs(tabToKeep: 2);
project.SendInfoToLog("Оставлена 2-я вкладка", false);
[/CODE][/SPOILER]

[SPOILER= Traffic]

# Traffic - Класс для работы с сетевым трафиком

Класс Traffic предназначен для мониторинга и анализа HTTP-трафика в ZennoPoster. Он позволяет отслеживать запросы на активной вкладке, извлекать данные из них и получать конкретные параметры запросов/ответов. Особенно полезен для работы с API, перехвата токенов авторизации и анализа сетевых взаимодействий.

## Конструкторы

### Traffic(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса для мониторинга трафика.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `instance` - экземпляр браузера  
- `log` - включить логирование процесса (по умолчанию false)

**Пример:**
[CODE=csharp]
//создать экземпляр для работы с трафиком
var traffic = new Traffic(project, instance, true);
[/CODE]

## Методы

### string Get(string url, string parametr, bool reload = false, bool parse = false, int deadline = 15, int delay = 3)

**Возвращаемое значение:** string - значение указанного параметра из трафика

Извлекает конкретный параметр из HTTP-запроса, содержащего указанный URL. Ожидает появления запроса и возвращает данные когда находит совпадение.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `parametr` - параметр для извлечения: "Method", "ResultCode", "Url", "ResponseContentType", "RequestHeaders", "RequestCookies", "RequestBody", "ResponseHeaders", "ResponseCookies", "ResponseBody"
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false)
- `parse` - автоматически парсить JSON ответ (по умолчанию false)
- `deadline` - максимальное время ожидания в секундах (по умолчанию 15)
- `delay` - задержка между проверками в секундах (по умолчанию 3)

**Пример:**
[CODE=csharp]
//получить токен из заголовков запроса
string token = traffic.Get("api/login", "RequestHeaders");

//получить тело ответа от API
string response = traffic.Get("api/user", "ResponseBody", parse: true);
[/CODE]

### Dictionary<string, string> Get(string url, bool reload = false, int deadline = 10)

**Возвращаемое значение:** Dictionary<string, string> - все параметры найденного запроса

Возвращает все доступные параметры HTTP-запроса в виде словаря. Содержит Method, ResultCode, Url, ResponseContentType, RequestHeaders, RequestCookies, RequestBody, ResponseHeaders, ResponseCookies, ResponseBody.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false) 
- `deadline` - максимальное время ожидания в секундах (по умолчанию 10)

**Пример:**
[CODE=csharp]
//получить все данные о запросе
var requestData = traffic.Get("api/users");

//использовать данные запроса
string method = requestData["Method"];
string statusCode = requestData["ResultCode"];
string responseBody = requestData["ResponseBody"];
[/CODE]

### string GetHeader(string url, string headerToGet = "Authorization", bool reload = false)

**Возвращаемое значение:** string - значение указанного заголовка

Извлекает конкретный заголовок из HTTP-запроса. По умолчанию ищет заголовок Authorization, часто используемый для токенов.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `headerToGet` - имя заголовка для получения (по умолчанию "Authorization")
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false)

**Пример:**
[CODE=csharp]
//получить токен авторизации
string authToken = traffic.GetHeader("api/profile");

//получить user-agent
string userAgent = traffic.GetHeader("api/data", "User-Agent");
[/CODE]

### string GetParam(string url, string parametr, bool reload = false, int deadline = 10)

**Возвращаемое значение:** string - значение указанного параметра

Альтернативный метод для получения конкретного параметра из HTTP-запроса. Работает аналогично первому методу Get, но с упрощенными параметрами.

**Параметры:**
- `url` - часть URL для поиска в трафике
- `parametr` - параметр для извлечения
- `reload` - перезагрузить страницу перед мониторингом (по умолчанию false)
- `deadline` - максимальное время ожидания в секундах (по умолчанию 10)

**Пример:**
[CODE=csharp]
//получить код ответа
string statusCode = traffic.GetParam("api/login", "ResultCode");

//получить тело запроса
string requestBody = traffic.GetParam("api/submit", "RequestBody");
[/CODE][/SPOILER]
[/SPOILER]
