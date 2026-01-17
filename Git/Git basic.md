## Базовая настройка Git (один раз)

### Глобальная настройка (для всех проектов)
```bash
git config --global user.name "Ваше Имя"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
```

### Локальная настройка (только для текущего проекта)
```bash
git config user.name "Другое Имя"
git config user.email "work.email@company.com"
```

### Настройка для сессии (временно)
```bash
# Установить переменные окружения для текущей сессии терминала
export GIT_AUTHOR_NAME="Временное Имя"
export GIT_AUTHOR_EMAIL="temp.email@example.com"
export GIT_COMMITTER_NAME="Временное Имя"
export GIT_COMMITTER_EMAIL="temp.email@example.com"
```

### Посмотреть текущие настройки
```bash
# Все настройки
git config --list

# Глобальные настройки
git config --global --list

# Локальные настройки проекта
git config --local --list

# Конкретный параметр
git config user.name
git config user.email
```

### Приоритет настроек (от высшего к низшему)
1. **Сессия** (переменные окружения)
2. **Локальные** (файл `.git/config` в проекте)
3. **Глобальные** (файл `~/.gitconfig`)
4. **Системные** (файл `/etc/gitconfig`)

## Создание репозитория

### Новый проект
```bash
git init
```

### Клонирование существующего репозитория
```bash
git clone https://github.com/username/repository.git
```

## Ежедневная работа

### 1. Проверить статус файлов
```bash
git status
```

### 2. Добавить файлы для коммита
```bash
# Добавить конкретный файл
git add filename.txt

# Добавить все измененные файлы
git add .

# Добавить все файлы определенного типа
git add *.js
```

### 3. Сделать коммит
```bash
git commit -m "Описание изменений"
```

### 4. Посмотреть историю коммитов
```bash
# Краткая история
git log --oneline

# Подробная история
git log

# История с графиком веток
git log --oneline --graph
```

## Работа с историей и откатами

### Посмотреть изменения
```bash
# Что изменилось с последнего коммита
git diff

# Изменения в конкретном файле
git diff filename.txt

# Сравнить два коммита
git diff commit1 commit2
```

### Отменить изменения в рабочей директории
```bash
# Отменить изменения в конкретном файле (вернуть к последнему коммиту)
git checkout -- filename.txt

# Отменить ВСЕ изменения в рабочей директории
git checkout -- .

# Современная команда (Git 2.23+)
git restore filename.txt
git restore .
```

### Отменить добавление файлов в staging area
```bash
# Убрать файл из staging (но оставить изменения)
git reset HEAD filename.txt

# Убрать все файлы из staging
git reset HEAD

# Современная команда (Git 2.23+)
git restore --staged filename.txt
```

### Вернуться к определенному коммиту

#### Временно посмотреть состояние
```bash
# Перейти к коммиту (только посмотреть)
git checkout commit_hash

# Вернуться к последнему коммиту
git checkout main
# или
git checkout master
```

#### Окончательно откатиться
```bash
# Мягкий откат (оставить изменения в файлах)
git reset --soft commit_hash

# Смешанный откат (убрать из staging, оставить в файлах)
git reset commit_hash

# Жесткий откат (удалить ВСЕ изменения)
git reset --hard commit_hash

# Откатиться на один коммит назад
git reset --hard HEAD~1
```

### Отменить конкретный коммит
```bash
# Создать новый коммит, отменяющий изменения
git revert commit_hash
```

## Управление настройками конфигурации

### Удаление настроек
```bash
# Удалить глобальную настройку
git config --global --unset user.name

# Удалить локальную настройку
git config --local --unset user.email

# Удалить секцию настроек
git config --global --remove-section alias
```

### Редактирование настроек в редакторе
```bash
# Открыть глобальный конфиг в редакторе
git config --global --edit

# Открыть локальный конфиг проекта
git config --edit
```

### Полезные глобальные настройки
```bash
# Настроить редактор по умолчанию
git config --global core.editor "code --wait"

# Включить подсветку в консоли
git config --global color.ui auto

# Настроить алиасы
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.visual '!gitk'
```

### Временная смена пользователя для коммита
```bash
# Сделать один коммит от другого пользователя
git commit --author="Другой Пользователь <other@example.com>" -m "Сообщение коммита"

# Или через переменные окружения
GIT_AUTHOR_NAME="Другой Пользователь" GIT_AUTHOR_EMAIL="other@example.com" git commit -m "Сообщение"
```

## Полезные команды для просмотра

### Найти хэш коммита
```bash
# Последние 5 коммитов
git log --oneline -5

# Поиск по сообщению коммита
git log --grep="ключевое слово"

# Коммиты за определенную дату
git log --since="2024-01-01" --until="2024-01-31"
```

### Посмотреть конкретный коммит
```bash
git show commit_hash
```

## Работа с ветками (опционально)

```bash
# Создать новую ветку
git branch feature-name

# Перейти на ветку
git checkout feature-name

# Создать и сразу перейти на ветку
git checkout -b feature-name

# Посмотреть все ветки
git branch

# Слить ветку в main
git checkout main
git merge feature-name

# Удалить ветку
git branch -d feature-name
```

Ситуация 1: У вас уже есть локальный проект, нужно привязать к GitHub/GitLab
```bash Посмотреть текущие удаленные репозитории (если есть)
git remote -v

### Добавить удаленный репозиторий
git remote add origin https://github.com/username/repository.git

### Проверить, что добавилось
git remote -v

### Отправить код в первый раз
git push -u origin main
### или
git push -u origin master
```
Ситуация 2: Изменить URL существующего удаленного репозитория
```bash Если нужно изменить адрес origin
git remote set-url origin https://github.com/username/new-repository.git

### Проверить
git remote -v
Ситуация 3: Удалить привязку к удаленному репозиторию
bashgit remote remove origin
Ситуация 4: Несколько удаленных репозиториев
bash# Добавить второй удаленный репозиторий
git remote add backup https://gitlab.com/username/repository.git

### Теперь можно пушить в разные места
git push origin main      # в GitHub
git push backup main      # в GitLab
```

Типичный сценарий с нуля:
```bash 1. Создаете проект локально
mkdir my-project
cd my-project
git init

### 2. Делаете первый коммит
git add .
git commit -m "Initial commit"

### 3. Создаете репозиторий на GitHub (через браузер)

### 4. Привязываете удаленный репозиторий
git remote add origin https://github.com/username/my-project.git

### 5. Отправляете код
git push -u origin main
Флаг -u (или --set-upstream) нужен только в первый раз — он создает связь между локальной и удаленной веткой. После этого можно просто писать git push.
```
## Работа в JetBrains Rider

### Основные операции через GUI:

1. **Git панель**: `Alt + 9` или View → Tool Windows → Git
2. **Commit**: `Ctrl + K`
3. **Push**: `Ctrl + Shift + K`
4. **Pull**: VCS → Git → Pull
5. **История**: правый клик на файле → Git → Show History

### Настройка пользователя в Rider:

1. **Глобальные настройки**:
   - File → Settings → Version Control → Git
   - Указать имя пользователя и email

2. **Локальные настройки проекта**:
   - File → Settings → Version Control → Git
   - Поставить галочку "Use project Git config file"
   - Указать отдельные настройки для проекта

### Откаты в Rider:

1. **Отменить изменения в файле**:
   - Правый клик на файле → Git → Rollback

2. **Вернуться к коммиту**:
   - Git панель → Log → правый клик на коммите → Reset Current Branch to Here
   - Выбрать тип reset (Soft/Mixed/Hard)

3. **Посмотреть изменения**:
   - Git панель → Log → двойной клик на коммите
   - Или: правый клик → Show Diff

### Полезные настройки в Rider:
- File → Settings → Version Control → Git
- Включить "Use credential helper"
- Настроить автопуш после коммита

## Частые сценарии

### "Я накосячил, хочу вернуть все как было"
```bash
# Если еще не делали commit
git checkout -- .

# Если уже сделали commit
git reset --hard HEAD~1
```

### "Хочу посмотреть, что было вчера"
```bash
git log --oneline --since="yesterday"
git checkout commit_hash
# ... посмотрели ...
git checkout main
```

### "Случайно удалил файл"
```bash
git checkout -- filename.txt
```

### "Хочу отменить последний коммит, но оставить изменения"
```bash
git reset --soft HEAD~1
```

### "Нужно сменить автора коммита временно"
```bash
# Для одного коммита
git commit --author="Новый Автор <new@example.com>" -m "Сообщение"

# Для сессии работы
export GIT_AUTHOR_NAME="Временный Автор"
export GIT_AUTHOR_EMAIL="temp@example.com"
# Все последующие коммиты будут от этого автора
```

### "Работаю над разными проектами с разными аккаунтами"
```bash
# В рабочем проекте
cd work-project
git config user.name "Рабочий Аккаунт"
git config user.email "work@company.com"

# В личном проекте
cd personal-project
git config user.name "Личный Аккаунт"
git config user.email "personal@gmail.com"
```

## ⚠️ Важные предупреждения

1. **`git reset --hard`** удаляет ВСЕ несохраненные изменения безвозвратно
2. Всегда делайте `git status` перед операциями
3. Если работаете с удаленным репозиторием, будьте осторожны с `reset` после `push`
4. Регулярно делайте коммиты - это ваши точки сохранения
5. Локальные настройки переопределяют глобальные
6. Переменные окружения переопределяют все остальные настройки

## Шпаргалка команд

| Задача | Команда |
|--------|---------|
| Статус | `git status` |
| Добавить все | `git add .` |
| Коммит | `git commit -m "message"` |
| История | `git log --oneline` |
| Отменить изменения | `git checkout -- .` |
| Откат к коммиту | `git reset --hard commit_hash` |
| Посмотреть коммит | `git show commit_hash` |
| Глобальные настройки | `git config --global --list` |
| Локальные настройки | `git config --local --list` |
| Установить пользователя глобально | `git config --global user.name "Имя"` |
| Установить пользователя локально | `git config user.name "Имя"` |
| Временная смена автора | `git commit --author="Автор <email>" -m "msg"` |
| Запушить все что есть одной строкой | `git add . && git commit -m "update" && git push origin master` |

| применить .gitignore | `git rm -r --cached . && git add . && git commit -m "Apply .gitignore"` | Bash
| применить .gitignore | `git rm -r --cached . ; git add . ; git commit -m "Apply .gitignore"` | PowerShell