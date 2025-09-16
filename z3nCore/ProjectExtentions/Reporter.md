## Reporter

Класс для создания отчётов об ошибках и успешных операциях в проектах. Позволяет собирать информацию о последних ошибках, создавать отчёты в различных форматах и отправлять уведомления в Telegram.

### Конструкторы

#### Reporter(IZennoPosterProjectModel project, Instance instance, bool log = false, string classEmoji = null)
Создаёт новый экземпляр класса Reporter для работы с отчётами.

**Параметры:**
- `project` - объект проекта ZennoPoster
- `instance` - экземпляр браузера
- `log` - флаг включения логирования (по умолчанию false)
- `classEmoji` - эмодзи для класса (по умолчанию null)

```csharp
//создание объекта Reporter
var reporter = new Reporter(project, instance);

//создание с дополнительными параметрами  
var reporter = new Reporter(project, instance, true, "📋");
```

### Методы

#### ErrorReport(bool toTg = false, bool toDb = false, bool screensot = false)
**Возвращаемое значение:** `string` - текстовый отчёт об ошибке

Создаёт подробный отчёт о последней ошибке в проекте и выполняет дополнительные действия по настройке.

**Параметры:**
- `toTg` - отправить отчёт в Telegram (по умолчанию false)
- `toDb` - обновить статус в базе данных (по умолчанию false)  
- `screensot` - создать скриншот страницы (по умолчанию false)

```csharp
//создание простого отчёта об ошибке
string errorReport = reporter.ErrorReport();
project.SendInfoToLog("Отчёт об ошибке: " + errorReport);

//создание отчёта с отправкой в Telegram и скриншотом
string fullReport = reporter.ErrorReport(toTg: true, screensot: true);

//создание отчёта с обновлением базы данных
string dbReport = reporter.ErrorReport(toDb: true);
```

#### SuccessReport(bool log = false, bool ToTg = false)
**Возвращаемое значение:** `string` - текстовый отчёт об успешном выполнении

Создаёт отчёт об успешном завершении операции с информацией о времени выполнения.

**Параметры:**
- `log` - вывести отчёт в лог (по умолчанию false)
- `ToTg` - отправить отчёт в Telegram (по умолчанию false)

```csharp
//создание простого отчёта об успехе
string successReport = reporter.SuccessReport();

//создание отчёта с выводом в лог
string loggedReport = reporter.SuccessReport(log: true);
project.SendInfoToLog("Операция завершена успешно");

//создание отчёта с отправкой в Telegram
string tgReport = reporter.SuccessReport(ToTg: true);
```