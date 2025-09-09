# Методичка по работе с GitHub API

## Содержание
1. [Подготовка к работе](#подготовка-к-работе)
2. [Создание репозитория](#создание-репозитория)
3. [Управление видимостью репозитория](#управление-видимостью-репозитория)
4. [Управление коллабораторами](#управление-коллабораторами)
5. [Примеры использования](#примеры-использования)
6. [Обработка ошибок](#обработка-ошибок)

## Подготовка к работе

### 1. Получение токена доступа

Для работы с GitHub API необходим Personal Access Token (PAT):

1. Зайдите в **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Нажмите **Generate new token (classic)**
3. Выберите необходимые разрешения (scopes):
   - `repo` - полный доступ к приватным репозиториям
   - `public_repo` - доступ к публичным репозиториям
   - `admin:repo_hook` - управление webhooks
   - `delete_repo` - удаление репозиториев

### 2. Базовая аутентификация

```bash
# Через заголовки
curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user

# Через Bearer token (рекомендуется)
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.github.com/user
```

### 3. Базовый URL API
```
https://api.github.com
```

## Создание репозитория

### Создание репозитория для пользователя

**Endpoint:** `POST /user/repos`

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/user/repos \
  -d '{
    "name": "my-new-repo",
    "description": "Описание репозитория",
    "private": false,
    "auto_init": true,
    "gitignore_template": "Node",
    "license_template": "mit"
  }'
```

### Создание репозитория для организации

**Endpoint:** `POST /orgs/{org}/repos`

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/orgs/YOUR_ORG/repos \
  -d '{
    "name": "org-repo",
    "description": "Репозиторий организации",
    "private": true,
    "auto_init": true
  }'
```

### Параметры создания репозитория

| Параметр | Тип | Описание | По умолчанию |
|----------|-----|----------|--------------|
| `name` | string | Имя репозитория (обязательно) | - |
| `description` | string | Описание репозитория | null |
| `homepage` | string | URL домашней страницы | null |
| `private` | boolean | Приватный репозиторий | false |
| `auto_init` | boolean | Создать начальный commit | false |
| `gitignore_template` | string | Шаблон .gitignore | null |
| `license_template` | string | Шаблон лицензии | null |
| `allow_squash_merge` | boolean | Разрешить squash merge | true |
| `allow_merge_commit` | boolean | Разрешить merge commit | true |
| `allow_rebase_merge` | boolean | Разрешить rebase merge | true |
| `delete_branch_on_merge` | boolean | Удалять ветку после merge | false |

### Пример ответа при создании

```json
{
  "id": 123456789,
  "name": "my-new-repo",
  "full_name": "username/my-new-repo",
  "private": false,
  "html_url": "https://github.com/username/my-new-repo",
  "clone_url": "https://github.com/username/my-new-repo.git",
  "ssh_url": "git@github.com:username/my-new-repo.git",
  "created_at": "2024-01-01T12:00:00Z",
  "updated_at": "2024-01-01T12:00:00Z"
}
```

## Управление видимостью репозитория

### Изменение видимости репозитория

**Endpoint:** `PATCH /repos/{owner}/{repo}`

```bash
# Сделать репозиторий приватным
curl -X PATCH \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME \
  -d '{
    "private": true
  }'

# Сделать репозиторий публичным
curl -X PATCH \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME \
  -d '{
    "private": false
  }'
```

### Получение информации о репозитории

**Endpoint:** `GET /repos/{owner}/{repo}`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/USERNAME/REPO_NAME
```

### Другие настройки видимости

```bash
# Изменение дополнительных настроек
curl -X PATCH \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME \
  -d '{
    "private": true,
    "has_issues": true,
    "has_projects": false,
    "has_wiki": false,
    "has_downloads": true,
    "allow_forking": false,
    "web_commit_signoff_required": true
  }'
```

## Управление коллабораторами

### Получение списка коллабораторов

**Endpoint:** `GET /repos/{owner}/{repo}/collaborators`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators
```

### Проверка коллаборатора

**Endpoint:** `GET /repos/{owner}/{repo}/collaborators/{username}`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/COLLABORATOR_USERNAME
```

Коды ответа:
- `204` - пользователь является коллаборатором
- `404` - пользователь не является коллаборатором

### Добавление коллаборатора

**Endpoint:** `PUT /repos/{owner}/{repo}/collaborators/{username}`

```bash
# Добавление с базовыми правами
curl -X PUT \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/NEW_COLLABORATOR \
  -d '{
    "permission": "push"
  }'
```

### Уровни доступа (permissions)

| Уровень | Описание |
|---------|----------|
| `pull` | Только чтение, клонирование и pull |
| `triage` | Чтение + управление issues и PR без записи кода |
| `push` | Чтение и запись в репозиторий |
| `maintain` | Чтение, запись и управление репозиторием без доступа к деструктивным операциям |
| `admin` | Полный доступ ко всем функциям репозитория |

### Добавление с разными уровнями доступа

```bash
# Только чтение
curl -X PUT \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/USER1 \
  -d '{"permission": "pull"}'

# Запись
curl -X PUT \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/USER2 \
  -d '{"permission": "push"}'

# Администратор
curl -X PUT \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/USER3 \
  -d '{"permission": "admin"}'
```

### Удаление коллаборатора

**Endpoint:** `DELETE /repos/{owner}/{repo}/collaborators/{username}`

```bash
curl -X DELETE \
  -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/COLLABORATOR_TO_REMOVE
```

### Получение разрешений коллаборатора

**Endpoint:** `GET /repos/{owner}/{repo}/collaborators/{username}/permission`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/COLLABORATOR_USERNAME/permission
```

Пример ответа:
```json
{
  "permission": "admin",
  "user": {
    "login": "collaborator_username",
    "id": 123456,
    "type": "User"
  }
}
```

## Примеры использования

### Python с библиотекой requests

```python
import requests
import json

class GitHubAPI:
    def __init__(self, token):
        self.token = token
        self.headers = {
            'Authorization': f'Bearer {token}',
            'Accept': 'application/vnd.github.v3+json',
            'Content-Type': 'application/json'
        }
        self.base_url = 'https://api.github.com'
    
    def create_repo(self, name, description='', private=False):
        """Создание репозитория"""
        url = f'{self.base_url}/user/repos'
        data = {
            'name': name,
            'description': description,
            'private': private,
            'auto_init': True
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        return response.json()
    
    def set_repo_visibility(self, owner, repo, private=True):
        """Изменение видимости репозитория"""
        url = f'{self.base_url}/repos/{owner}/{repo}'
        data = {'private': private}
        
        response = requests.patch(url, headers=self.headers, json=data)
        return response.json()
    
    def add_collaborator(self, owner, repo, username, permission='push'):
        """Добавление коллаборатора"""
        url = f'{self.base_url}/repos/{owner}/{repo}/collaborators/{username}'
        data = {'permission': permission}
        
        response = requests.put(url, headers=self.headers, json=data)
        return response.status_code == 201
    
    def remove_collaborator(self, owner, repo, username):
        """Удаление коллаборатора"""
        url = f'{self.base_url}/repos/{owner}/{repo}/collaborators/{username}'
        
        response = requests.delete(url, headers=self.headers)
        return response.status_code == 204
    
    def get_collaborators(self, owner, repo):
        """Получение списка коллабораторов"""
        url = f'{self.base_url}/repos/{owner}/{repo}/collaborators'
        
        response = requests.get(url, headers=self.headers)
        return response.json()

# Использование
github = GitHubAPI('YOUR_TOKEN')

# Создание репозитория
repo = github.create_repo('test-repo', 'Тестовый репозиторий', private=True)
print(f"Создан репозиторий: {repo['html_url']}")

# Добавление коллаборатора
success = github.add_collaborator('username', 'test-repo', 'collaborator', 'push')
print(f"Коллаборатор добавлен: {success}")

# Список коллабораторов
collaborators = github.get_collaborators('username', 'test-repo')
for collab in collaborators:
    print(f"Коллаборатор: {collab['login']}")
```

### JavaScript/Node.js

```javascript
const axios = require('axios');

class GitHubAPI {
    constructor(token) {
        this.token = token;
        this.headers = {
            'Authorization': `Bearer ${token}`,
            'Accept': 'application/vnd.github.v3+json',
            'Content-Type': 'application/json'
        };
        this.baseURL = 'https://api.github.com';
    }

    async createRepo(name, description = '', isPrivate = false) {
        try {
            const response = await axios.post(`${this.baseURL}/user/repos`, {
                name: name,
                description: description,
                private: isPrivate,
                auto_init: true
            }, { headers: this.headers });
            
            return response.data;
        } catch (error) {
            console.error('Ошибка создания репозитория:', error.response.data);
            throw error;
        }
    }

    async addCollaborator(owner, repo, username, permission = 'push') {
        try {
            const response = await axios.put(
                `${this.baseURL}/repos/${owner}/${repo}/collaborators/${username}`,
                { permission: permission },
                { headers: this.headers }
            );
            
            return response.status === 201;
        } catch (error) {
            console.error('Ошибка добавления коллаборатора:', error.response.data);
            return false;
        }
    }

    async removeCollaborator(owner, repo, username) {
        try {
            const response = await axios.delete(
                `${this.baseURL}/repos/${owner}/${repo}/collaborators/${username}`,
                { headers: this.headers }
            );
            
            return response.status === 204;
        } catch (error) {
            console.error('Ошибка удаления коллаборатора:', error.response.data);
            return false;
        }
    }
}

// Использование
const github = new GitHubAPI('YOUR_TOKEN');

async function example() {
    try {
        // Создание репозитория
        const repo = await github.createRepo('test-repo', 'Описание', true);
        console.log('Репозиторий создан:', repo.html_url);

        // Добавление коллаборатора
        const added = await github.addCollaborator('owner', 'test-repo', 'username', 'push');
        console.log('Коллаборатор добавлен:', added);
        
    } catch (error) {
        console.error('Ошибка:', error);
    }
}

example();
```

## Обработка ошибок

### Основные коды ошибок

| Код | Описание | Возможные причины |
|-----|----------|-------------------|
| 401 | Unauthorized | Неверный или отсутствующий токен |
| 403 | Forbidden | Недостаточно прав или превышен лимит |
| 404 | Not Found | Репозиторий/пользователь не найден |
| 409 | Conflict | Репозиторий уже существует |
| 422 | Unprocessable Entity | Неверные данные запроса |

### Примеры обработки ошибок

```bash
# Проверка ответа
response=$(curl -s -w "\n%{http_code}" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/repos/USERNAME/REPO_NAME/collaborators/USER)

http_code=$(echo "$response" | tail -n1)
body=$(echo "$response" | head -n -1)

case $http_code in
    200|204)
        echo "Успешно"
        ;;
    401)
        echo "Ошибка аутентификации"
        ;;
    403)
        echo "Недостаточно прав"
        ;;
    404)
        echo "Не найдено"
        ;;
    *)
        echo "Неизвестная ошибка: $http_code"
        echo "$body"
        ;;
esac
```

### Лимиты API

GitHub API имеет следующие ограничения:
- **Аутентифицированные запросы**: 5000 в час
- **Неаутентифицированные запросы**: 60 в час
- **Search API**: 30 запросов в минуту

Проверка лимитов:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.github.com/rate_limit
```

### Установка зависимостей для C#

Для работы с JSON в .NET Framework 4.6.2 необходимо установить Newtonsoft.Json:

**Package Manager Console:**
```powershell
Install-Package Newtonsoft.Json
```

**packages.config:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Newtonsoft.Json" version="12.0.3" targetFramework="net462" />
</packages>
```

### Обработка ошибок в C#

```csharp
public class GitHubApiException : Exception
{
    public HttpStatusCode StatusCode { get; }
    public string ResponseContent { get; }
    
    public GitHubApiException(HttpStatusCode statusCode, string responseContent, string message) 
        : base(message)
    {
        StatusCode = statusCode;
        ResponseContent = responseContent;
    }
}

// Улучшенная обработка ошибок
public async Task<GitHubRepository> CreateRepositoryWithErrorHandlingAsync(string name, string description = "", bool isPrivate = false)
{
    var createRepoRequest = new
    {
        name = name,
        description = description,
        @private = isPrivate,
        auto_init = true
    };

    var json = JsonConvert.SerializeObject(createRepoRequest);
    var content = new StringContent(json, Encoding.UTF8, "application/json");

    var response = await _httpClient.PostAsync(_baseUrl + "/user/repos", content);
    var responseContent = await response.Content.ReadAsStringAsync();

    switch (response.StatusCode)
    {
        case HttpStatusCode.Created:
            return JsonConvert.DeserializeObject<GitHubRepository>(responseContent);
        
        case HttpStatusCode.Unauthorized:
            throw new GitHubApiException(response.StatusCode, responseContent, "Неверный токен авторизации");
        
        case HttpStatusCode.Forbidden:
            throw new GitHubApiException(response.StatusCode, responseContent, "Недостаточно прав или превышен лимит запросов");
        
        case HttpStatusCode.UnprocessableEntity:
            throw new GitHubApiException(response.StatusCode, responseContent, "Репозиторий с таким именем уже существует");
        
        default:
            throw new GitHubApiException(response.StatusCode, responseContent, 
                string.Format("Неожиданная ошибка: {0}", response.StatusCode));
    }
}
```

### Пример приложения Console App

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    private const string GITHUB_TOKEN = "YOUR_TOKEN_HERE";
    
    static void Main(string[] args)
    {
        // Для .NET Framework 4.6.2 используем GetAwaiter().GetResult()
        MainAsync().GetAwaiter().GetResult();
    }
    
    static async Task MainAsync()
    {
        var github = new GitHubAPI(GITHUB_TOKEN);
        
        try
        {
            Console.WriteLine("=== Тестирование GitHub API ===");
            
            // Создание репозитория
            Console.Write("Введите название репозитория: ");
            var repoName = Console.ReadLine();
            
            Console.Write("Приватный репозиторий? (y/N): ");
            var isPrivateInput = Console.ReadLine();
            var isPrivate = isPrivateInput.ToLower() == "y";
            
            Console.WriteLine("Создание репозитория...");
            var repo = await github.CreateRepositoryAsync(repoName, "Тестовый репозиторий", isPrivate);
            Console.WriteLine("Репозиторий создан: " + repo.HtmlUrl);
            
            // Управление коллабораторами
            while (true)
            {
                Console.WriteLine("\nВыберите действие:");
                Console.WriteLine("1. Добавить коллаборатора");
                Console.WriteLine("2. Удалить коллаборатора");
                Console.WriteLine("3. Показать коллабораторов");
                Console.WriteLine("4. Изменить видимость репозитория");
                Console.WriteLine("0. Выход");
                
                var choice = Console.ReadLine();
                
                switch (choice)
                {
                    case "1":
                        await AddCollaboratorInteractive(github, "your-username", repoName);
                        break;
                    case "2":
                        await RemoveCollaboratorInteractive(github, "your-username", repoName);
                        break;
                    case "3":
                        await ShowCollaborators(github, "your-username", repoName);
                        break;
                    case "4":
                        await ChangeVisibilityInteractive(github, "your-username", repoName);
                        break;
                    case "0":
                        return;
                    default:
                        Console.WriteLine("Неверный выбор");
                        break;
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Ошибка: " + ex.Message);
        }
        finally
        {
            github.Dispose();
        }
    }
    
    static async Task AddCollaboratorInteractive(GitHubAPI github, string owner, string repo)
    {
        Console.Write("Имя пользователя коллаборатора: ");
        var username = Console.ReadLine();
        
        Console.Write("Уровень доступа (pull/push/admin): ");
        var permission = Console.ReadLine();
        if (string.IsNullOrEmpty(permission)) permission = "push";
        
        var success = await github.AddCollaboratorAsync(owner, repo, username, permission);
        if (success)
            Console.WriteLine("Коллаборатор добавлен успешно");
        else
            Console.WriteLine("Ошибка добавления коллаборатора");
    }
    
    static async Task RemoveCollaboratorInteractive(GitHubAPI github, string owner, string repo)
    {
        Console.Write("Имя пользователя для удаления: ");
        var username = Console.ReadLine();
        
        var success = await github.RemoveCollaboratorAsync(owner, repo, username);
        if (success)
            Console.WriteLine("Коллаборатор удален успешно");
        else
            Console.WriteLine("Ошибка удаления коллаборатора");
    }
    
    static async Task ShowCollaborators(GitHubAPI github, string owner, string repo)
    {
        var collaborators = await github.GetCollaboratorsAsync(owner, repo);
        
        if (collaborators.Count == 0)
        {
            Console.WriteLine("Коллабораторов нет");
            return;
        }
        
        Console.WriteLine("Список коллабораторов:");
        foreach (var collab in collaborators)
        {
            var permissions = "";
            if (collab.Permissions != null)
            {
                if (collab.Permissions.Admin) permissions = "admin";
                else if (collab.Permissions.Push) permissions = "push";
                else if (collab.Permissions.Pull) permissions = "pull";
            }
            
            Console.WriteLine(string.Format("- {0} ({1})", collab.Login, permissions));
        }
    }
    
    static async Task ChangeVisibilityInteractive(GitHubAPI github, string owner, string repo)
    {
        Console.Write("Сделать приватным? (y/N): ");
        var input = Console.ReadLine();
        var makePrivate = input.ToLower() == "y";
        
        var success = await github.SetRepositoryVisibilityAsync(owner, repo, makePrivate);
        if (success)
            Console.WriteLine("Видимость изменена успешно");
        else
            Console.WriteLine("Ошибка изменения видимости");
    }
}
```

### Рекомендации для .NET Framework 4.6.2

1. **Используйте HttpClient правильно** - создавайте один экземпляр и переиспользуйте
2. **Не забывайте Dispose** - вызывайте Dispose() для HttpClient
3. **Обрабатывайте исключения** - сетевые операции могут падать
4. **Используйте ConfigureAwait(false)** в библиотечном коде для избежания deadlock
5. **Проверяйте StatusCode** - не полагайтесь только на IsSuccessStatusCode
6. **Логируйте ошибки** - сохраняйте детали ошибок для отладки

