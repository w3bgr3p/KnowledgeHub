# MSBuild: Полная методичка

## Что такое MSBuild?

**MSBuild** (Microsoft Build Engine) — это система сборки проектов .NET. Это XML-based язык для автоматизации процесса компиляции, тестирования и развертывания.

### Где используется:
- `.csproj` файлы (проекты C#)
- `.vbproj` файлы (проекты VB.NET)
- `.targets` файлы (переиспользуемые сценарии)
- `Directory.Build.props` и `Directory.Build.targets` (настройки для всей solution)

---

## Основные концепции MSBuild

### 1. Properties (Свойства) — переменные

```xml
<PropertyGroup>
    <!-- Простое свойство -->
    <MyVariable>Значение</MyVariable>
    
    <!-- Использование другого свойства -->
    <OutputPath>$(SolutionDir)bin\</OutputPath>
    
    <!-- Встроенные свойства -->
    <!-- $(SolutionDir) - папка solution -->
    <!-- $(ProjectDir) - папка проекта -->
    <!-- $(TargetPath) - путь к собранному файлу -->
    <!-- $(TargetFileName) - имя файла (example.dll) -->
    <!-- $(Configuration) - Debug или Release -->
</PropertyGroup>
```

**Использование:**
```xml
<Message Text="Значение: $(MyVariable)" />
```

---

### 2. Items (Элементы) — списки/массивы

```xml
<ItemGroup>
    <!-- Один элемент -->
    <MyFiles Include="file1.txt" />
    
    <!-- Несколько элементов -->
    <MyFiles Include="file2.txt" />
    <MyFiles Include="file3.txt" />
    
    <!-- С wildcards -->
    <SourceFiles Include="**/*.cs" />
    
    <!-- С метаданными -->
    <Folders Include="Releases">
        <Type>Output</Type>
    </Folders>
</ItemGroup>
```

**Использование:**
```xml
<!-- Обратиться ко всему списку -->
<Copy SourceFiles="@(MyFiles)" DestinationFolder="output\" />

<!-- Батчинг (цикл) - выполнится для каждого элемента -->
<Message Text="Файл: %(MyFiles.Identity)" />
```

---

### 3. Targets (Цели) — функции/задачи

```xml
<Target Name="MyTarget" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
    <!-- Код выполняется после сборки, только в Release -->
</Target>
```

**Атрибуты Target:**
- `Name` — имя цели (обязательно)
- `AfterTargets` — выполнить после указанной цели
- `BeforeTargets` — выполнить до указанной цели
- `Condition` — условие выполнения

**Стандартные цели:**
- `Build` — сборка проекта
- `Clean` — очистка
- `Rebuild` — пересборка
- `Publish` — публикация

---

### 4. Tasks (Задачи) — команды

```xml
<!-- Создать папку -->
<MakeDir Directories="$(OutputPath)" />

<!-- Копировать файлы -->
<Copy SourceFiles="input.txt" DestinationFolder="output\" />

<!-- Удалить файлы -->
<Delete Files="@(FilesToDelete)" />

<!-- Вывести сообщение -->
<Message Text="Hello from MSBuild!" Importance="high" />

<!-- Выполнить команду -->
<Exec Command="dotnet --version" />
```

---

### 5. Batching (Батчинг) — циклы в MSBuild

MSBuild **не имеет** классического `foreach`, вместо этого используется **батчинг** через `%()`:

```xml
<ItemGroup>
    <Numbers Include="1" />
    <Numbers Include="2" />
    <Numbers Include="3" />
</ItemGroup>

<!-- Это выполнится 3 раза! -->
<Message Text="Number: %(Numbers.Identity)" />
```

**Вывод:**
```
Number: 1
Number: 2
Number: 3
```

---

## Структура файлов MSBuild

### Directory.Build.props
Автоматически импортируется **ДО** `.csproj`. Используется для общих **свойств**.

```xml
<Project>
    <PropertyGroup>
        <LangVersion>latest</LangVersion>
        <Nullable>enable</Nullable>
    </PropertyGroup>
</Project>
```

### Directory.Build.targets
Автоматически импортируется **ПОСЛЕ** `.csproj`. Используется для общих **целей и задач**.

```xml
<Project>
    <Target Name="MyCustomTarget" AfterTargets="Build">
        <!-- Ваш код -->
    </Target>
</Project>
```

### .targets файлы
Переиспользуемые сценарии, которые можно импортировать вручную:

```xml
<!-- В .csproj -->
<Import Project="$(SolutionDir)MyCustom.targets" />
```

---

## Решение задачи: Копирование для проектов z3n.*

### Задача:
1. Для всех проектов, начинающихся с `z3n.`
2. При сборке в Release
3. Копировать выходной файл в две папки
4. Функция должна быть переиспользуемой
5. Папки должны быть переменными

---

## Шаг 1: Анализ задачи

**Что нам нужно:**
- Определить список папок назначения (переменные)
- Создать эти папки, если их нет
- Скопировать файл в каждую папку (цикл)
- Вывести сообщение для каждого копирования
- Сделать это переиспользуемым

**Инструменты MSBuild:**
- `PropertyGroup` — для хранения путей к папкам
- `ItemGroup` — для списка папок (чтобы можно было делать цикл)
- `MakeDir` — создание папок
- `Copy` с батчингом — копирование в цикле
- `Message` с батчингом — вывод для каждой папки
- `Directory.Build.targets` — автоматическое применение ко всем проектам

---

## Шаг 2: Создаем структуру файла

Создаем файл `Directory.Build.targets` в корне solution:

```
YourSolution/
├── Directory.Build.targets  ← создаем этот файл
├── z3n.Project1/
│   └── z3n.Project1.csproj
├── z3n.Project2/
│   └── z3n.Project2.csproj
├── OtherProject/            ← этот проект НЕ затронется
│   └── OtherProject.csproj
└── YourSolution.sln
```

---

## Шаг 3: Определяем переменные (PropertyGroup)

Сначала определим **статические пути** к папкам назначения:

```xml
<Project>
    <Target Name="CopyToReleases" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
        <PropertyGroup>
            <!-- Переменная 1: папка Releases -->
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
            
            <!-- Переменная 2: папка ExternalAssemblies -->
            <ExternalAssemblies>$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
        </PropertyGroup>
    </Target>
</Project>
```

**Что происходит:**
- `$(SolutionDir)` — встроенная переменная MSBuild, указывает на корень solution
- Мы создаем две переменные с полными путями
- Эти переменные будут доступны внутри Target

---

## Шаг 4: Создаем список папок (ItemGroup)

Теперь превратим эти пути в **список**, чтобы можно было делать цикл:

```xml
<Project>
    <Target Name="CopyToReleases" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
        <PropertyGroup>
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
            <ExternalAssemblies>$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
        </PropertyGroup>

        <!-- Создаем список папок -->
        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
        </ItemGroup>
    </Target>
</Project>
```

**Что происходит:**
- `<DestinationFolders>` — это список (массив)
- `Include` добавляет элемент в список
- Теперь `@(DestinationFolders)` содержит обе папки

---

## Шаг 5: Создаем папки (MakeDir с батчингом)

Создадим каждую папку, если она не существует:

```xml
<Project>
    <Target Name="CopyToReleases" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
        <PropertyGroup>
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
            <ExternalAssemblies>$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
        </PropertyGroup>

        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
        </ItemGroup>

        <!-- Создаем папки -->
        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />
    </Target>
</Project>
```

**Что происходит:**
- `%(DestinationFolders.Identity)` — **батчинг**: MSBuild выполнит `MakeDir` для КАЖДОГО элемента
- `Identity` — это содержимое элемента (путь к папке)
- `Condition="!Exists(...)"` — создать, только если не существует
- Эта одна строка создаст ОБЕ папки

---

## Шаг 6: Копируем файлы (Copy с батчингом)

Скопируем собранный файл в каждую папку:

```xml
<Project>
    <Target Name="CopyToReleases" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
        <PropertyGroup>
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
            <ExternalAssemblies>$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
        </PropertyGroup>

        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
        </ItemGroup>

        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />

        <!-- Копируем файл в каждую папку -->
        <Copy
            SourceFiles="$(TargetPath)"
            DestinationFolder="%(DestinationFolders.Identity)"
            SkipUnchangedFiles="true" />
    </Target>
</Project>
```

**Что происходит:**
- `$(TargetPath)` — путь к собранному файлу (например, `bin\Release\z3n.MyProject.dll`)
- `%(DestinationFolders.Identity)` — батчинг: копирование выполнится для КАЖДОЙ папки
- `SkipUnchangedFiles="true"` — не копировать, если файл не изменился (оптимизация)

---

## Шаг 7: Выводим сообщения (Message с батчингом)

Добавим вывод для каждого копирования:

```xml
<Project>
    <Target Name="CopyToReleases" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
        <PropertyGroup>
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
            <ExternalAssemblies>$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
        </PropertyGroup>

        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
        </ItemGroup>

        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />

        <Copy
            SourceFiles="$(TargetPath)"
            DestinationFolder="%(DestinationFolders.Identity)"
            SkipUnchangedFiles="true" />

        <!-- Выводим сообщение для каждой папки -->
        <Message 
            Text="Copied $(TargetFileName) to %(DestinationFolders.Identity)" 
            Importance="high" />
    </Target>
</Project>
```

**Что происходит:**
- `$(TargetFileName)` — имя файла (например, `z3n.MyProject.dll`)
- `%(DestinationFolders.Identity)` — путь к папке назначения
- `Importance="high"` — важное сообщение, будет выделено в логе
- Выведется ДВА сообщения (по одному на каждую папку)

---

## Шаг 8: Добавляем фильтр по имени проекта

Сейчас это применяется ко ВСЕМ проектам. Добавим условие только для проектов `z3n.*`:

```xml
<Project>
    <Target Name="CopyToReleases" 
            AfterTargets="Build" 
            Condition="'$(Configuration)' == 'Release' AND $(MSBuildProjectName.StartsWith('z3n.'))">
        
        <PropertyGroup>
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
            <ExternalAssemblies>$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
        </PropertyGroup>

        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
        </ItemGroup>

        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />

        <Copy
            SourceFiles="$(TargetPath)"
            DestinationFolder="%(DestinationFolders.Identity)"
            SkipUnchangedFiles="true" />

        <Message 
            Text="Copied $(TargetFileName) to %(DestinationFolders.Identity)" 
            Importance="high" />
    </Target>
</Project>
```

**Что изменилось:**
- Добавили `AND $(MSBuildProjectName.StartsWith('z3n.'))`
- `$(MSBuildProjectName)` — имя проекта (без расширения)
- `.StartsWith('z3n.')` — вызов .NET метода прямо в MSBuild!
- Теперь Target выполнится ТОЛЬКО для проектов, начинающихся с `z3n.`

---

## Шаг 9: Делаем папки настраиваемыми

Сейчас папки захардкожены. Сделаем их переменными, которые можно переопределить:

```xml
<Project>
    <!-- Определяем значения по умолчанию -->
    <PropertyGroup>
        <!-- Если переменная не задана, используем значение по умолчанию -->
        <ReleasesFolder Condition="'$(ReleasesFolder)' == ''">$(SolutionDir)Releases\</ReleasesFolder>
        <ExternalAssemblies Condition="'$(ExternalAssemblies)' == ''">$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
    </PropertyGroup>

    <Target Name="CopyToReleases" 
            AfterTargets="Build" 
            Condition="'$(Configuration)' == 'Release' AND $(MSBuildProjectName.StartsWith('z3n.'))">
        
        <!-- Создаем список из переменных -->
        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
        </ItemGroup>

        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />

        <Copy
            SourceFiles="$(TargetPath)"
            DestinationFolder="%(DestinationFolders.Identity)"
            SkipUnchangedFiles="true" />

        <Message 
            Text="Copied $(TargetFileName) to %(DestinationFolders.Identity)" 
            Importance="high" />
    </Target>
</Project>
```

**Что изменилось:**
- Вынесли `PropertyGroup` наружу Target
- `Condition="'$(ReleasesFolder)' == ''"` — задать значение, ТОЛЬКО если не задано
- Теперь можно переопределить в конкретном проекте или через командную строку

**Переопределение в проекте:**
```xml
<!-- В z3n.MyProject.csproj -->
<PropertyGroup>
    <ReleasesFolder>C:\MyCustomFolder\</ReleasesFolder>
</PropertyGroup>
```

**Переопределение через командную строку:**
```bash
dotnet build -c Release /p:ReleasesFolder=C:\Custom\
```

---

## Шаг 10: Добавляем возможность добавлять папки

Сделаем так, чтобы можно было добавлять папки через `ItemGroup`:

```xml
<Project>
    <PropertyGroup>
        <ReleasesFolder Condition="'$(ReleasesFolder)' == ''">$(SolutionDir)Releases\</ReleasesFolder>
        <ExternalAssemblies Condition="'$(ExternalAssemblies)' == ''">$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
    </PropertyGroup>

    <Target Name="CopyToReleases" 
            AfterTargets="Build" 
            Condition="'$(Configuration)' == 'Release' AND $(MSBuildProjectName.StartsWith('z3n.'))">
        
        <!-- Базовые папки -->
        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
            
            <!-- Дополнительные папки (если заданы) -->
            <!-- Можно добавить в проекте через <AdditionalCopyFolders> -->
        </ItemGroup>

        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />

        <Copy
            SourceFiles="$(TargetPath)"
            DestinationFolder="%(DestinationFolders.Identity)"
            SkipUnchangedFiles="true" />

        <Message 
            Text="Copied $(TargetFileName) to %(DestinationFolders.Identity)" 
            Importance="high" />
    </Target>
</Project>
```

**Как добавить дополнительные папки в проекте:**
```xml
<!-- В z3n.MyProject.csproj -->
<ItemGroup>
    <AdditionalCopyFolders Include="C:\MyFolder\" />
    <AdditionalCopyFolders Include="D:\AnotherFolder\" />
</ItemGroup>
```

Затем в `Directory.Build.targets` добавьте их в список:
```xml
<ItemGroup>
    <DestinationFolders Include="$(ReleasesFolder)" />
    <DestinationFolders Include="$(ExternalAssemblies)" />
    <DestinationFolders Include="@(AdditionalCopyFolders)" />
</ItemGroup>
```

---

## Итоговое решение

**Файл: `Directory.Build.targets` в корне solution**

```xml
<Project>
    <!-- Переменные по умолчанию (можно переопределить) -->
    <PropertyGroup>
        <ReleasesFolder Condition="'$(ReleasesFolder)' == ''">$(SolutionDir)Releases\</ReleasesFolder>
        <ExternalAssemblies Condition="'$(ExternalAssemblies)' == ''">$(SolutionDir)ZennoLab\ExternalAssemblies\</ExternalAssemblies>
    </PropertyGroup>

    <!-- Автоматическое копирование для проектов z3n.* -->
    <Target Name="CopyToReleases" 
            AfterTargets="Build" 
            Condition="'$(Configuration)' == 'Release' AND $(MSBuildProjectName.StartsWith('z3n.'))">
        
        <!-- Создаем список папок назначения -->
        <ItemGroup>
            <DestinationFolders Include="$(ReleasesFolder)" />
            <DestinationFolders Include="$(ExternalAssemblies)" />
            <DestinationFolders Include="@(AdditionalCopyFolders)" Condition="'@(AdditionalCopyFolders)' != ''" />
        </ItemGroup>

        <!-- Создаем папки (если не существуют) -->
        <MakeDir 
            Directories="%(DestinationFolders.Identity)" 
            Condition="!Exists('%(DestinationFolders.Identity)')" />

        <!-- Копируем файл в каждую папку (батчинг) -->
        <Copy
            SourceFiles="$(TargetPath)"
            DestinationFolder="%(DestinationFolders.Identity)"
            SkipUnchangedFiles="true" />

        <!-- Выводим сообщение для каждой папки (батчинг) -->
        <Message 
            Text="✓ Copied $(TargetFileName) → %(DestinationFolders.Identity)" 
            Importance="high" />
    </Target>
</Project>
```

---

## Как это работает

### 1. Автоматическое применение
- Файл `Directory.Build.targets` автоматически подключается ко **всем** проектам в solution
- **Не нужно** ничего добавлять в `.csproj` файлы

### 2. Фильтр проектов
```xml
Condition="'$(Configuration)' == 'Release' AND $(MSBuildProjectName.StartsWith('z3n.'))"
```
- Выполнится **только** для проектов, начинающихся с `z3n.`
- Выполнится **только** в конфигурации Release

### 3. Батчинг (цикл)
```xml
<Copy
    SourceFiles="$(TargetPath)"
    DestinationFolder="%(DestinationFolders.Identity)"
    SkipUnchangedFiles="true" />
```
- `%(DestinationFolders.Identity)` — MSBuild выполнит `Copy` **для каждого** элемента в `DestinationFolders`
- Это аналог `foreach` в C#

### 4. Переопределение переменных
```xml
<PropertyGroup>
    <ReleasesFolder Condition="'$(ReleasesFolder)' == ''">$(SolutionDir)Releases\</ReleasesFolder>
</PropertyGroup>
```
- Если переменная **не задана** (`== ''`), используется значение по умолчанию
- Можно переопределить в `.csproj` или через командную строку

---

## Вывод при сборке

```
Build started...
1>------ Build started: Project: z3n.Project1, Configuration: Release Any CPU ------
1>z3n.Project1 -> W:\code\z3n.Project1\bin\Release\net8.0\z3n.Project1.dll
1>✓ Copied z3n.Project1.dll → W:\code\Releases\
1>✓ Copied z3n.Project1.dll → W:\code\ZennoLab\ExternalAssemblies\
========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```

---

## Расширение функциональности

### Добавить дополнительные папки в конкретном проекте

**В `z3n.MyProject.csproj`:**
```xml
<ItemGroup>
    <AdditionalCopyFolders Include="C:\Backup\" />
    <AdditionalCopyFolders Include="\\NetworkShare\Deploy\" />
</ItemGroup>
```

### Переопределить папки для всех проектов

**В `Directory.Build.props`:**
```xml
<PropertyGroup>
    <ReleasesFolder>C:\BuildOutput\</ReleasesFolder>
    <ExternalAssemblies>C:\Libraries\</ExternalAssemblies>
</PropertyGroup>
```

### Отключить для конкретного проекта

**В `z3n.SomeProject.csproj`:**
```xml
<PropertyGroup>
    <DisableCopyToReleases>true</DisableCopyToReleases>
</PropertyGroup>
```

**В `Directory.Build.targets` добавить в условие:**
```xml
Condition="'$(Configuration)' == 'Release' AND $(MSBuildProjectName.StartsWith('z3n.')) AND '$(DisableCopyToReleases)' != 'true'"
```

---

## Справочник MSBuild

### Часто используемые переменные

| Переменная | Описание | Пример |
|-----------|----------|--------|
| `$(SolutionDir)` | Путь к solution | `W:\code\MySolution\` |
| `$(ProjectDir)` | Путь к проекту | `W:\code\MySolution\MyProject\` |
| `$(TargetPath)` | Полный путь к собранному файлу | `W:\code\bin\Release\MyProject.dll` |
| `$(TargetFileName)` | Имя собранного файла | `MyProject.dll` |
| `$(TargetDir)` | Папка собранного файла | `W:\code\bin\Release\` |
| `$(Configuration)` | Конфигурация | `Debug` или `Release` |
| `$(MSBuildProjectName)` | Имя проекта | `MyProject` |
| `$(MSBuildProjectFile)` | Имя файла проекта | `MyProject.csproj` |

### Часто используемые Tasks

| Task | Описание | Пример |
|------|----------|--------|
| `<Copy>` | Копировать файлы | `<Copy SourceFiles="@(Files)" DestinationFolder="out\" />` |
| `<MakeDir>` | Создать папку | `<MakeDir Directories="$(OutputPath)" />` |
| `<Delete>` | Удалить файлы | `<Delete Files="@(TempFiles)" />` |
| `<Message>` | Вывести сообщение | `<Message Text="Hello!" Importance="high" />` |
| `<Exec>` | Выполнить команду | `<Exec Command="dotnet --version" />` |
| `<RemoveDir>` | Удалить папку | `<RemoveDir Directories="$(TempPath)" />` |

### Операторы в Condition

| Оператор | Описание | Пример |
|----------|----------|--------|
| `==` | Равно | `'$(Config)' == 'Release'` |
| `!=` | Не равно | `'$(Config)' != 'Debug'` |
| `AND` | Логическое И | `'$(A)' == 'X' AND '$(B)' == 'Y'` |
| `OR` | Логическое ИЛИ | `'$(A)' == 'X' OR '$(B)' == 'Y'` |
| `!` | Отрицание | `!Exists('file.txt')` |

### Функции MSBuild

| Функция | Описание | Пример |
|---------|----------|--------|
| `Exists()` | Проверить существование | `Exists('$(OutputPath)')` |
| `.StartsWith()` | Начинается с | `$(Name.StartsWith('z3n'))` |
| `.EndsWith()` | Заканчивается на | `$(File.EndsWith('.dll'))` |
| `.Contains()` | Содержит | `$(Path.Contains('test'))` |
| `.Replace()` | Заменить | `$(Text.Replace('old', 'new'))` |

---

## Отладка MSBuild

### Подробный лог сборки

```bash
dotnet build -v detailed
```

### Очень подробный лог

```bash
dotnet build -v diagnostic
```

### Сохранить лог в файл

```bash
dotnet build > build.log
```

### Вывести значение переменной

```xml
<Message Text="SolutionDir = $(SolutionDir)" Importance="high" />
<Message Text="ProjectName = $(MSBuildProjectName)" Importance="high" />
```

---

## Заключение

MSBuild — это мощный инструмент автоматизации сборки. Основные принципы:

1. **Properties** — переменные (одиночные значения)
2. **Items** — списки/массивы (коллекции)
3. **Targets** — функции/задачи (что выполнять)
4. **Tasks** — команды (конкретные действия)
5. **Batching** — циклы через `%(Item.Identity)`
6. **Conditions** — условия выполнения

### Ключевые моменты:

- MSBuild **НЕ имеет** классического `foreach` — вместо него используется **батчинг**
- `$(Variable)` — обращение к переменной
- `@(ItemList)` — обращение ко всему списку
- `%(ItemList.Identity)` — батчинг (цикл по каждому элементу)
- `Directory.Build.targets` — автоматически применяется ко всем проектам
- Файлы `.targets` можно переиспользовать через `<Import>`

### Паттерны использования:

**Для одиночных значений** → используйте `PropertyGroup`
```xml
<PropertyGroup>
    <OutputPath>bin\Release\</OutputPath>
</PropertyGroup>
```

**Для списков** → используйте `ItemGroup`
```xml
<ItemGroup>
    <Files Include="*.txt" />
</ItemGroup>
```

**Для циклов** → используйте батчинг `%()`
```xml
<Copy SourceFiles="@(Files)" DestinationFolder="%(Files.RelativeDir)" />
```

**Для условий** → используйте `Condition`
```xml
<Target Name="MyTarget" Condition="'$(Configuration)' == 'Release'">
```

---

## Полезные ссылки

- [Официальная документация MSBuild](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild)
- [MSBuild Property Functions](https://learn.microsoft.com/en-us/visualstudio/msbuild/property-functions)
- [MSBuild Reserved Properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-reserved-and-well-known-properties)
- [MSBuild Batching](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-batching)

---

## Частые ошибки и их решения

### Ошибка: "Target выполняется для всех проектов"
**Проблема:** Нет фильтра по проектам

**Решение:** Добавьте условие в `Condition`:
```xml
Condition="$(MSBuildProjectName.StartsWith('z3n.'))"
```

---

### Ошибка: "Переменная не определена"
**Проблема:** Обращение к несуществующей переменной

**Решение:** Задайте значение по умолчанию:
```xml
<PropertyGroup>
    <MyVar Condition="'$(MyVar)' == ''">DefaultValue</MyVar>
</PropertyGroup>
```

---

### Ошибка: "Батчинг не работает"
**Проблема:** Используете `@(Items)` вместо `%(Items.Identity)`

**Неправильно:**
```xml
<Message Text="File: @(Files)" />  <!-- Выведет ВСЕ файлы одной строкой -->
```

**Правильно:**
```xml
<Message Text="File: %(Files.Identity)" />  <!-- Выведет КАЖДЫЙ файл отдельно -->
```

---

### Ошибка: "Папка не создается"
**Проблема:** Путь содержит переменную, которая пустая

**Решение:** Проверьте значение переменной:
```xml
<Message Text="Path = $(MyPath)" Importance="high" />
```

---

## Шпаргалка команд

| Задача | Команда |
|--------|---------|
| Собрать проект | `dotnet build` |
| Собрать в Release | `dotnet build -c Release` |
| Подробный лог | `dotnet build -v detailed` |
| Очистить | `dotnet clean` |
| Пересобрать | `dotnet build --no-incremental` |
| Задать переменную | `dotnet build /p:MyVar=Value` |
| Посмотреть доступные Targets | `dotnet msbuild -targets` |

---

## Примеры расширения

### Копировать также .pdb и .xml файлы

```xml
<ItemGroup>
    <FilesToCopy Include="$(TargetPath)" />
    <FilesToCopy Include="$(TargetDir)$(TargetName).pdb" Condition="Exists('$(TargetDir)$(TargetName).pdb')" />
    <FilesToCopy Include="$(TargetDir)$(TargetName).xml" Condition="Exists('$(TargetDir)$(TargetName).xml')" />
</ItemGroup>

<Copy
    SourceFiles="@(FilesToCopy)"
    DestinationFolder="%(DestinationFolders.Identity)"
    SkipUnchangedFiles="true" />
```

---

### Копировать только если тесты прошли

```xml
<Target Name="CopyToReleases" 
        AfterTargets="Build" 
        DependsOnTargets="RunTests"
        Condition="'$(Configuration)' == 'Release' AND '$(TestsPassed)' == 'true'">
    <!-- Код копирования -->
</Target>

<Target Name="RunTests">
    <Exec Command="dotnet test" />
    <PropertyGroup>
        <TestsPassed>true</TestsPassed>
    </PropertyGroup>
</Target>
```

---

### Добавить версию в имя файла

```xml
<PropertyGroup>
    <VersionedFileName>$(TargetName)_$(Version)$(TargetExt)</VersionedFileName>
</PropertyGroup>

<Copy
    SourceFiles="$(TargetPath)"
    DestinationFiles="%(DestinationFolders.Identity)$(VersionedFileName)"
    SkipUnchangedFiles="true" />
```

Результат: `z3n.MyProject_1.2.3.dll`

---

### Архивировать перед копированием

```xml
<Target Name="ArchiveBeforeCopy" BeforeTargets="CopyToReleases">
    <ZipDirectory
        SourceDirectory="$(TargetDir)"
        DestinationFile="$(TargetDir)$(TargetName)_$(Version).zip"
        Overwrite="true" />
</Target>
```

---

Теперь методичка **полная**! 🎯