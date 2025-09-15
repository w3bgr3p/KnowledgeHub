# Класс Init

## Описание
Этот класс представляет собой центральный инициализатор для проектов ZennoPoster. Он упрощает создание и настройку автоматизированных задач, работая с браузерами, базами данных, аккаунтами и социальными сетями. В целом работает как фабрика для инициализации всех компонентов проекта: получает список аккаунтов для работы, настраивает браузер, подключает социальные сети и кошельки, а затем запускает основной скрипт проекта.

## Конструкторы

### Init(IZennoPosterProjectModel project, Instance instance, bool log = false)
- **Описание**: Создает экземпляр инициализатора с указанным проектом, экземпляром браузера и настройкой логирования.
- **Параметры**: 
  - `project` - модель проекта ZennoPoster
  - `instance` - экземпляр браузера
  - `log` - включить/выключить подробное логирование (по умолчанию false)
- **Пример**:
  ```csharp
  var init = new Init(project, instance, true);
  project.SendInfoToLog("Инициализатор создан с логированием", false);
  ```

### Init(IZennoPosterProjectModel project, bool log = false)
- **Описание**: Создает экземпляр инициализатора только с проектом (без браузера).
- **Параметры**:
  - `project` - модель проекта ZennoPoster
  - `log` - включить/выключить подробное логирование
- **Пример**:
  ```csharp
  var init = new Init(project, false);
  project.SendInfoToLog("Инициализатор создан без браузера", false);
  ```

## Публичные методы

### InitProject(string author = "w3bgr3p", string[] customQueries = null, bool log = false)
- **Описание**: Выполняет полную инициализацию проекта - подготавливает базу данных, создает список аккаунтов для работы и настраивает все необходимые компоненты.
- **Параметры**:
  - `author` - имя автора скрипта для отображения в логах
  - `customQueries` - дополнительные SQL-запросы для выборки аккаунтов
  - `log` - включить подробное логирование
- **Пример**:
  ```csharp
  string[] queries = { "SELECT id FROM accounts WHERE status = 'active'" };
  init.InitProject("myusername", queries, true);
  project.SendInfoToLog("Проект инициализирован", false);
  ```

### PrepareProject()
- **Описание**: Подготавливает проект к выполнению - выбирает аккаунт для работы, проверяет фильтры (блокчейн, социальные сети) и настраивает экземпляр браузера.
- **Пример**:
  ```csharp
  try 
  {
      init.PrepareProject();
      project.SendInfoToLog("Проект готов к работе", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка подготовки: {ex.Message}", false);
  }
  ```

### PrepareInstance()
- **Описание**: Настраивает экземпляр браузера - запускает браузер, устанавливает прокси, загружает cookies и настраивает профиль.
- **Пример**:
  ```csharp
  try
  {
      init.PrepareInstance();
      project.SendInfoToLog("Браузер настроен и готов", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка настройки браузера: {ex.Message}", false);
  }
  ```

### LoadSocials(string requiredSocial)
- **Возвращает**: string - список загруженных социальных сетей или "noBrowser" если браузер не запущен
- **Описание**: Загружает и авторизует указанные социальные сети в браузере (Google, Twitter, Discord, GitHub).
- **Пример**:
  ```csharp
  string result = init.LoadSocials("Google,Twitter,Discord");
  if (result == "noBrowser")
  {
      project.SendWarningToLog("Браузер не запущен", false);
  }
  else
  {
      project.SendInfoToLog($"Загружены соц.сети: {result}", false);
  }
  ```

### LoadWallets(string walletsToUse)
- **Возвращает**: string - список загруженных кошельков или "noBrowser" если браузер не запущен
- **Описание**: Подключает и настраивает указанные криптовалютные кошельки (Backpack, Zerion, Keplr).
- **Пример**:
  ```csharp
  string result = init.LoadWallets("Backpack,Zerion");
  if (result == "noBrowser")
  {
      project.SendWarningToLog("Браузер не запущен для кошельков", false);
  }
  else
  {
      project.SendInfoToLog($"Кошельки подключены: {result}", false);
  }
  ```

### RunProject(List<string> additionalVars = null, bool add = true)
- **Возвращает**: bool - успешность выполнения проекта
- **Описание**: Запускает основной скрипт проекта с передачей всех необходимых переменных.
- **Параметры**:
  - `additionalVars` - дополнительные переменные для передачи в скрипт
  - `add` - добавить к стандартным переменным (true) или заменить их (false)
- **Пример**:
  ```csharp
  var extraVars = new List<string> { "customVariable", "mySpecialSetting" };
  bool success = init.RunProject(extraVars, true);
  if (success)
  {
      project.SendInfoToLog("Проект выполнен успешно", false);
  }
  else
  {
      project.SendErrorToLog("Проект завершился с ошибкой", false);
  }
  ```