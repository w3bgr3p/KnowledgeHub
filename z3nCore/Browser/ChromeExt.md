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
  ```csharp
  var chromeExt = new ChromeExt(project, true);
  project.SendInfoToLog("Экземпляр ChromeExt создан с логированием", false);
  ```

### ChromeExt(IZennoPosterProjectModel project, Instance instance, bool log = false)
- **Описание**: Создает экземпляр класса для управления расширениями с указанным проектом, экземпляром браузера и опцией логирования.
- **Параметры**:
  - `project` - модель проекта ZennoPoster
  - `instance` - экземпляр браузера
  - `log` - включить/выключить логирование (по умолчанию false)
- **Пример**:
  ```csharp
  var chromeExt = new ChromeExt(project, instance, false);
  project.SendInfoToLog("Экземпляр ChromeExt создан без логирования", false);
  ```

## Публичные методы

### GetVer(string extId)
- **Возвращает**: string - версия расширения
- **Описание**: Получает версию указанного расширения по его ID из настроек профиля.
- **Параметры**:
  - `extId` - ID расширения
- **Пример**:
  ```csharp
  string version = chromeExt.GetVer("pbgjpgbpljobkekbhnnmlikbbfhbhmem");
  project.SendInfoToLog($"Версия расширения: {version}", false);
  ```

### Install(string extId, string fileName, bool log = false)
- **Возвращает**: bool - успешность установки
- **Описание**: Устанавливает расширение из CRX-файла, если оно еще не установлено.
- **Параметры**:
  - `extId` - ID расширения
  - `fileName` - имя CRX-файла
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  ```csharp
  bool installed = chromeExt.Install("pbgjpgbpljobkekbhnnmlikbbfhbhmem", "One-Click-Extensions-Manager.crx", true);
  if (installed)
  {
      project.SendInfoToLog("Расширение установлено", false);
  }
  else
  {
      project.SendWarningToLog("Расширение уже установлено", false);
  }
  ```

### Switch(string toUse = "", bool log = false)
- **Описание**: Переключает состояние расширений (включает или отключает) на основе списка имен или ID.
- **Параметры**:
  - `toUse` - список расширений для включения (имена или ID через запятую)
  - `log` - включить логирование (по умолчанию false)
- **Пример**:
  ```csharp
  try
  {
      chromeExt.Switch("Wallet,ExtensionManager", true);
      project.SendInfoToLog("Расширения переключены", false);
  }
  catch (Exception ex)
  {
      project.SendErrorToLog($"Ошибка переключения: {ex.Message}", false);
  }
  ```

### Rm(string[] ExtToRemove)
- **Описание**: Удаляет массив указанных расширений по их ID.
- **Параметры**:
  - `ExtToRemove` - массив ID расширений для удаления
- **Пример**:
  ```csharp
  string[] exts = { "id1", "id2" };
  chromeExt.Rm(exts);
  project.SendInfoToLog("Расширения удалены", false);
  ```

