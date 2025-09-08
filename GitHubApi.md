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

### Рекомендации

1. **Всегда проверяйте коды ответов** перед обработкой данных
2. **Используйте условные запросы** с ETag для кэширования
3. **Обрабатывайте ошибки 403** - возможно превышен лимит
4. **Используйте пагинацию** для больших списков данных
5. **Следите за rate limit** и реализуйте retry логику

Эта методичка покрывает основные операции с GitHub API для управления репозиториями и коллабораторами. Для более детальной информации обращайтесь к официальной документации GitHub API.
