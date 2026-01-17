# Методичка: .csproj файлы и сборка .NET проектов

## 1. Что такое .csproj файл?

**.csproj** — это XML-файл, который описывает:
- Какой код компилировать
- Какие зависимости подключить
- Как собирать проект
- Куда класть результат

Это **манифест** вашего проекта. MSBuild читает его и понимает, что делать.

---

## 2. Два формата .csproj

### Старый формат (.NET Framework)
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" />
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProjectGuid>{GUID}</ProjectGuid>
    <OutputType>Library</OutputType>
    <TargetFrameworkVersion>v4.8</TargetFrameworkVersion>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Core" />
  </ItemGroup>
  <ItemGroup>
    <Compile Include="Class1.cs" />
  </ItemGroup>
  <Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" />    <Target Name="CopyToReleases" AfterTargets="Build" Condition="'$(Configuration)' == 'Release'">
        <PropertyGroup>
            <ReleasesFolder>$(SolutionDir)Releases\</ReleasesFolder>
        </PropertyGroup>

        <MakeDir Directories="$(ReleasesFolder)" Condition="!Exists('$(ReleasesFolder)')" />

        <Copy
                SourceFiles="$(TargetPath)"
                DestinationFolder="$(ReleasesFolder)"
                SkipUnchangedFiles="true" />

        <Message Text="Copied $(TargetFileName) to $(ReleasesFolder)" Importance="high" />
    </Target>

</Project>
```

**Проблемы старого формата:**
- Многословный (сотни строк)
- Нужно вручную перечислять все файлы
- Сложно читать и редактировать
- NuGet пакеты хранятся в `packages.config`

### Новый формат (SDK-style)
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
  </PropertyGroup>
</Project>
```

**Преимущества:**
- Компактный (5-20 строк вместо 200+)
- Автоматически включает все `.cs` файлы
- Встроенная поддержка NuGet
- Можно таргетировать на .NET Framework, .NET Core, .NET 5+

---

## 3. Почему файлы отличаются?

### Для .NET Standard изначально:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>
</Project>
```

### Для .NET Framework изначально:
Создается **старый формат** — огромный XML со всеми `<Import>`, `<Compile>` и т.д.

### Если вы руками меняете netstandard на net48:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
  </PropertyGroup>
</Project>
```

**Это работает!** SDK-style формат поддерживает .NET Framework 4.x, но Visual Studio по умолчанию создает старый формат для совместимости.

---

## 4. Основные элементы .csproj

### PropertyGroup — настройки проекта
```xml
<PropertyGroup>
  <TargetFramework>net48</TargetFramework>           <!-- Целевая платформа -->
  <OutputType>Library</OutputType>                    <!-- Library или Exe -->
  <RootNamespace>MyProject</RootNamespace>           <!-- Namespace по умолчанию -->
  <AssemblyName>MyProject</AssemblyName>             <!-- Имя DLL -->
  <LangVersion>latest</LangVersion>                   <!-- Версия C# -->
  <Nullable>enable</Nullable>                         <!-- Nullable reference types -->
</PropertyGroup>
```

### ItemGroup — зависимости и файлы

#### NuGet пакеты
```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

#### Ссылки на другие проекты
```xml
<ItemGroup>
  <ProjectReference Include="..\z3n.Core\z3n.Core.csproj" />
</ItemGroup>
```

#### Прямые ссылки на DLL
```xml
<ItemGroup>
  <Reference Include="MyLibrary">
    <HintPath>..\libs\MyLibrary.dll</HintPath>
  </Reference>
</ItemGroup>
```

#### Файлы для включения/исключения
```xml
<ItemGroup>
  <Compile Remove="OldCode\**" />                    <!-- Исключить папку -->
  <None Include="config.json">                       <!-- Копировать файл -->
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

---

## 5. Как работает сборка?

### Процесс компиляции:
```
1. MSBuild читает .csproj
2. Восстанавливает NuGet пакеты (dotnet restore)
3. Компилирует C# код в IL (Intermediate Language)
4. Создает .dll или .exe файл
5. Копирует зависимости в bin\Debug или bin\Release
6. Выполняет пост-сборочные команды (если есть)
```

### Путь файлов:
```
YourProject/
├── YourProject.csproj
├── Class1.cs
└── bin/
    ├── Debug/
    │   └── net48/
    │       ├── YourProject.dll          ← Ваша DLL
    │       ├── Dependency1.dll          ← Зависимости
    │       └── Dependency2.dll
    └── Release/
        └── net48/
            └── YourProject.dll          ← Оптимизированная версия
```

---

## 6. Что такое Costura и как она работает?

### Costura — это Fody плагин

**Fody** — это инструмент для модификации скомпилированной сборки (IL weaving).

**Процесс:**
```
1. MSBuild компилирует ваш код в YourProject.dll
2. Fody запускается после компиляции
3. Costura читает FodyWeavers.xml
4. Встраивает зависимые DLL как ресурсы в вашу DLL
5. Добавляет код для распаковки при загрузке
```

### Что происходит внутри DLL:

**До Costura:**
```
bin/Release/
├── YourProject.dll        (100 KB)
├── Newtonsoft.Json.dll    (500 KB)
└── Nethereum.Web3.dll     (2 MB)
```

**После Costura:**
```
bin/Release/
└── YourProject.dll        (2.6 MB) ← Все внутри!
```

### Как это работает в рантайме:
```csharp
// При загрузке YourProject.dll автоматически выполняется код:
AppDomain.CurrentDomain.AssemblyResolve += (sender, args) => 
{
    // Costura ищет нужную DLL в ресурсах
    var resourceName = "costura.newtonsoft.json.dll.compressed";
    var stream = Assembly.GetExecutingAssembly()
        .GetManifestResourceStream(resourceName);
    
    // Распаковывает и загружает в память
    return Assembly.Load(Decompress(stream));
};
```

### Costura сама является NuGet пакетом:
```xml
<PackageReference Include="Costura.Fody" Version="5.7.0">
  <PrivateAssets>all</PrivateAssets>  ← Не попадает в зависимости
</PackageReference>
```

`PrivateAssets=all` означает: "Используй при сборке, но не включай в зависимости".

---

## 7. Важные концепции

### PrivateAssets
```xml
<PackageReference Include="Fody" Version="6.8.0">
  <PrivateAssets>all</PrivateAssets>
</PackageReference>
```
- `all` — пакет только для сборки, не передается дальше
- Используется для build-time инструментов (Fody, analyzers)

### CopyLocal
```xml
<Reference Include="MyLib">
  <HintPath>..\MyLib.dll</HintPath>
  <Private>false</Private>  ← Не копировать в bin
</Reference>
```

### Multiple targeting
```xml
<PropertyGroup>
  <TargetFrameworks>net48;netstandard2.0;net6.0</TargetFrameworks>
</PropertyGroup>
```
Собирает один проект для нескольких платформ одновременно.

### Conditional compilation
```xml
<PropertyGroup Condition="'$(Configuration)' == 'Debug'">
  <DefineConstants>DEBUG;TRACE</DefineConstants>
</PropertyGroup>
```

```csharp
#if DEBUG
    Console.WriteLine("Debug mode");
#endif
```

---

## 8. Targets — кастомные задачи сборки

### Копирование файлов после сборки:
```xml
<Target Name="CopyToReleases" AfterTargets="Build">
  <Copy SourceFiles="$(TargetPath)" 
        DestinationFolder="$(SolutionDir)Releases\" />
</Target>
```

### Запуск команды перед сборкой:
```xml
<Target Name="PreBuild" BeforeTargets="PreBuildEvent">
  <Exec Command="echo Building $(ProjectName)..." />
</Target>
```

### Генерация версии из Git:
```xml
<Target Name="SetVersion" BeforeTargets="BeforeBuild">
  <Exec Command="git describe --tags --abbrev=0" 
        ConsoleToMSBuild="true">
    <Output TaskParameter="ConsoleOutput" PropertyName="GitVersion" />
  </Exec>
  <PropertyGroup>
    <Version>$(GitVersion)</Version>
  </PropertyGroup>
</Target>
```

---

## 9. Практические советы

### Миграция на SDK-style
Если у вас старый формат, можно конвертировать:
1. Создайте новый SDK-style проект
2. Скопируйте только `<PackageReference>` и `<ProjectReference>`
3. Все `.cs` файлы подхватятся автоматически

### Debugging сборки
```bash
# Посмотреть подробности сборки
dotnet build -v detailed

# Посмотреть что MSBuild делает
msbuild /v:diag YourProject.csproj > build.log
```

### Условная компиляция по фреймворку:
```xml
<ItemGroup Condition="'$(TargetFramework)' == 'net48'">
  <PackageReference Include="System.Memory" Version="4.5.4" />
</ItemGroup>
```

### Автоматическая версия:
```xml
<PropertyGroup>
  <Version>1.0.0</Version>
  <FileVersion>1.0.0.$(BuildNumber)</FileVersion>
  <AssemblyVersion>1.0.0.0</AssemblyVersion>
</PropertyGroup>
```

---

## 10. Типичные ошибки и решения

### Ошибка: "Could not load file or assembly"
**Причина:** Зависимая DLL не скопировалась  
**Решение:** Проверьте `CopyLocal=true` или используйте Costura

### Ошибка: "The type exists in both assemblies"
**Причина:** Одна DLL подключена дважды  
**Решение:** Проверьте дубликаты в `<Reference>` и `<PackageReference>`

### Зависимости занимают слишком много места
**Решение:** 
- Используйте Costura для упаковки
- Или IL Merge для слияния
- Или публикуйте как single-file

### .csproj слишком большой
**Решение:** Мигрируйте на SDK-style формат

---

## 11. Чек-лист для вашего проекта

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <!-- 1. Основные настройки -->
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <!-- 2. NuGet пакеты -->
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

  <!-- 3. Ссылки на проекты -->
  <ItemGroup>
    <ProjectReference Include="..\Core\Core.csproj" />
  </ItemGroup>

  <!-- 4. Прямые ссылки -->
  <ItemGroup>
    <Reference Include="ThirdParty">
      <HintPath>..\libs\ThirdParty.dll</HintPath>
    </Reference>
  </ItemGroup>

  <!-- 5. Build-time инструменты -->
  <ItemGroup>
    <PackageReference Include="Costura.Fody" Version="5.7.0">
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>

  <!-- 6. Пост-сборочные команды -->
  <Target Name="PostBuild" AfterTargets="Build">
    <Copy SourceFiles="$(TargetPath)" 
          DestinationFolder="$(SolutionDir)Output\" />
  </Target>

</Project>
```

---

## Заключение

**.csproj** — это сердце вашего проекта. Понимание его структуры дает полный контроль над сборкой, зависимостями и деплоем. SDK-style формат делает это простым и понятным.

**Costura** — это магия, которая превращает ворох DLL в одну, встраивая зависимости как ресурсы и автоматически распаковывая их при загрузке.
