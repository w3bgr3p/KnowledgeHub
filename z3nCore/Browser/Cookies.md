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
  ```csharp
  var cookies = new Cookies(project, instance, true);
  project.SendInfoToLog("Экземпляр Cookies создан с логированием", false);
  ```

## Публичные методы

### Get(string domainFilter = "")
- **Возвращает**: string - JSON-строка с данными cookies
- **Описание**: Получает cookies для указанного домена или всех доменов и возвращает их в формате JSON. Если домен не указан, возвращаются cookies для текущего активного домена.
- **Параметры**:
  - `domainFilter` - фильтр домена (по умолчанию пустая строка, возвращает cookies для текущего домена)
- **Пример**:
  ```csharp
  string cookiesJson = cookies.Get(".example.com");
  project.SendInfoToLog($"Получены cookies: {cookiesJson}", false);
  ```

### Set(string cookieSourse = null, string jsonPath = null)
- **Описание**: Устанавливает cookies из указанного источника (база данных или файл).
- **Параметры**:
  - `cookieSourse` - источник cookies (`dbMain`, `dbProject`, `fromFile` или JSON-строка)
  - `jsonPath` - путь к файлу с cookies (используется, если `cookieSourse` = `fromFile`)
- **Пример**:
  ```csharp
  try
  {
      cookies.Set("fromFile", "C:\\profiles\\cookies\\account.json");
      project.SendInfoToLog("Cookies установлены из файла", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка установки cookies: {ex.Message}", false);
  }
  ```

### Save(string source = null, string jsonPath = null)
- **Описание**: Сохраняет cookies в базу данных или файл. Поддерживает сохранение cookies текущего проекта или всех cookies.
- **Параметры**:
  - `source` - источник cookies (`project` для текущего домена, `all` для всех доменов)
  - `jsonPath` - путь для сохранения cookies в файл (если указан)
- **Пример**:
  ```csharp
  try
  {
      cookies.Save("all", "C:\\profiles\\cookies\\all_cookies.json");
      project.SendInfoToLog("Cookies сохранены", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка сохранения cookies: {ex.Message}", false);
  }
  ```

### GetByJs(string domainFilter = "", bool log = false)
- **Возвращает**: string - JSON-строка с данными cookies, полученных через JavaScript
- **Описание**: Получает cookies текущей страницы через выполнение JavaScript-кода в браузере.
- **Параметры**:
  - `domainFilter` - фильтр домена (по умолчанию пустая строка, возвращает cookies текущего домена)
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  ```csharp
  string jsCookies = cookies.GetByJs(".example.com", true);
  project.SendInfoToLog($"Cookies получены через JS: {jsCookies}", false);
  ```

### SetByJs(string cookiesJson, bool log = false)
- **Описание**: Устанавливает cookies через выполнение JavaScript-кода в браузере.
- **Параметры**:
  - `cookiesJson` - JSON-строка с данными cookies
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  ```csharp
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
  ```