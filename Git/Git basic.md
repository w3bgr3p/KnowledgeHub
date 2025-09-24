# Простая методичка по Git для одного разработчика

## Базовая настройка Git (один раз)

```bash
git config --global user.name "Ваше Имя"
git config --global user.email "your.email@example.com"
```

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

## Работа с удаленным репозиторием

```bash
# Отправить изменения
git push origin main

# Получить изменения
git pull origin main

# Посмотреть удаленные репозитории
git remote -v
```

## Работа в JetBrains Rider

### Основные операции через GUI:

1. **Git панель**: `Alt + 9` или View → Tool Windows → Git
2. **Commit**: `Ctrl + K`
3. **Push**: `Ctrl + Shift + K`
4. **Pull**: VCS → Git → Pull
5. **История**: правый клик на файле → Git → Show History

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

## ⚠️ Важные предупреждения

1. **`git reset --hard`** удаляет ВСЕ несохраненные изменения безвозвратно
2. Всегда делайте `git status` перед операциями
3. Если работаете с удаленным репозиторием, будьте осторожны с `reset` после `push`
4. Регулярно делайте коммиты - это ваши точки сохранения

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