# GitHubApi

Класс для работы с GitHub API, позволяющий управлять репозиториями, коллабораторами и их правами доступа. Предоставляет простые методы для создания репозиториев, изменения их видимости, добавления и удаления участников проекта.

## Конструкторы

### GitHubApi(string token, string username)

Создает новый экземпляр для работы с GitHub API.

**Параметры:**
- `token` - токен доступа к GitHub API
- `username` - имя пользователя GitHub

**Пример:**
```csharp
//создать подключение к GitHub API
var github = new GitHubApi("your_token_here", "your_username");
```

## Методы

### GetRepositoryInfo(string repoName)

**Возвращает:** `string` - информация о репозитории в формате JSON или сообщение об ошибке

Получает подробную информацию о указанном репозитории.

**Параметры:**
- `repoName` - название репозитория

**Пример:**
```csharp
//получить информацию о репозитории
string repoInfo = github.GetRepositoryInfo("my-project");
project.SendInfoToLog(repoInfo);
```

### GetCollaborators(string repoName)

**Возвращает:** `string` - список коллабораторов в формате JSON или сообщение об ошибке

Получает список всех участников репозитория.

**Параметры:**
- `repoName` - название репозитория

**Пример:**
```csharp
//получить список участников репозитория
string collaborators = github.GetCollaborators("my-project");
project.SendInfoToLog(collaborators);
```

### CreateRepository(string repoName)

**Возвращает:** `string` - данные созданного репозитория в формате JSON или сообщение об ошибке

Создает новый приватный репозиторий.

**Параметры:**
- `repoName` - название нового репозитория

**Пример:**
```csharp
//создать новый приватный репозиторий
string result = github.CreateRepository("new-project");
project.SendInfoToLog("Репозиторий создан: " + result);
```

### ChangeVisibility(string repoName, bool makePrivate)

**Возвращает:** `string` - результат операции в формате JSON или сообщение об ошибке

Изменяет видимость репозитория (публичный/приватный).

**Параметры:**
- `repoName` - название репозитория
- `makePrivate` - true для приватного, false для публичного

**Пример:**
```csharp
//сделать репозиторий публичным
string result = github.ChangeVisibility("my-project", false);
project.SendInfoToLog("Видимость изменена: " + result);
```

### AddCollaborator(string repoName, string collaboratorUsername, string permission = "pull")

**Возвращает:** `string` - результат операции в формате JSON или сообщение об ошибке

Добавляет нового участника в репозиторий с указанными правами.

**Параметры:**
- `repoName` - название репозитория
- `collaboratorUsername` - имя пользователя нового участника
- `permission` - права доступа: "pull", "push", "admin", "maintain", "triage" (по умолчанию "pull")

**Пример:**
```csharp
//добавить участника с правами на запись
string result = github.AddCollaborator("my-project", "developer123", "push");
project.SendInfoToLog("Участник добавлен: " + result);
```

### RemoveCollaborator(string repoName, string collaboratorUsername)

**Возвращает:** `string` - результат операции или сообщение об ошибке

Удаляет участника из репозитория.

**Параметры:**
- `repoName` - название репозитория
- `collaboratorUsername` - имя пользователя для удаления

**Пример:**
```csharp
//удалить участника из репозитория
string result = github.RemoveCollaborator("my-project", "old-developer");
project.SendInfoToLog("Участник удален: " + result);
```

### ChangeCollaboratorPermission(string repoName, string collaboratorUsername, string permission = "pull")

**Возвращает:** `string` - результат операции или сообщение об ошибке

Изменяет права доступа существующего участника репозитория.

**Параметры:**
- `repoName` - название репозитория
- `collaboratorUsername` - имя пользователя
- `permission` - новые права: "pull", "push", "admin", "maintain", "triage" (по умолчанию "pull")

**Пример:**
```csharp
//изменить права участника на администраторские
string result = github.ChangeCollaboratorPermission("my-project", "developer123", "admin");
project.SendInfoToLog("Права изменены: " + result);
```