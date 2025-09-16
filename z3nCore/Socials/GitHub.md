# GitHub

Класс для автоматизации работы с GitHub аккаунтами, включая авторизацию, двухфакторную аутентификацию и управление паролями.

## Конструкторы

### GitHub(IZennoPosterProjectModel project, Instance instance, bool log = false)

Создает новый экземпляр класса GitHub с автоматической загрузкой учетных данных.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `instance` - экземпляр браузера
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
var github = new GitHub(project, instance, true);
```

## Методы

### void LoadCreds()

Загружает учетные данные GitHub из базы данных и сохраняет их в переменные проекта.

**Пример:**
```csharp
github.LoadCreds();
```

### void InputCreds()

Вводит учетные данные на странице входа GitHub и обрабатывает двухфакторную аутентификацию.

**Пример:**
```csharp
github.InputCreds();
```

### void Go()

Переходит на страницу входа GitHub и принимает cookies если необходимо.

**Пример:**
```csharp
github.Go();
```

### void Verify2fa()

Выполняет верификацию двухфакторной аутентификации с использованием OTP кода.

**Пример:**
```csharp
github.Verify2fa();
```

### string Load()

**Возвращает:** имя текущего пользователя GitHub

Выполняет полный процесс входа в аккаунт GitHub с проверкой корректности авторизации и сохранением cookies.

**Пример:**
```csharp
string currentUser = github.Load();
project.SendInfoToLog($"Вошли как: {currentUser}");
```

### string Current()

**Возвращает:** имя текущего авторизованного пользователя

Получает имя текущего пользователя из навигационного меню GitHub.

**Пример:**
```csharp
string username = github.Current();
project.SendInfoToLog($"Текущий пользователь: {username}");
```

### void ChangePass(string password = null)

Изменяет пароль аккаунта GitHub через функцию восстановления пароля с использованием email.

**Параметры:**
- `password` - новый пароль (не используется в текущей реализации)

**Пример:**
```csharp
github.ChangePass();
```

### void SaveCookies()

Сохраняет текущие cookies браузера в базу данных для последующего использования.

**Пример:**
```csharp
github.SaveCookies();
```