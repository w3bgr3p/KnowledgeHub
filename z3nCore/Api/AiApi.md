# Класс AI

Описание
Класс AI предназначен для интеграции с различными сервисами искусственного интеллекта в проектах ZennoPoster. Он позволяет отправлять запросы к AI-сервисам (Perplexity, AI.IO) и получать текстовые ответы. Класс поддерживает различные предустановленные функции для генерации твитов, оптимизации кода и создания обращений в службу поддержки Google.

## Конструкторы

**AI(IZennoPosterProjectModel project, string provider, string model = null, bool log = false)**

Описание: Создает экземпляр AI-клиента с указанным провайдером и настройками.
Параметры:
- project - модель проекта ZennoPoster
- provider - провайдер AI-сервиса ("perplexity" или "aiio")
- model - конкретная модель для использования (необязательно)
- log - включить подробное логирование (по умолчанию false)

Пример:
```csharp
var ai = new AI(project, "perplexity", "llama-3.3-70b", true);
project.SendInfoToLog("AI клиент создан с провайдером Perplexity", false);
```

## Публичные методы

**Query(string systemContent, string userContent, string aiModel = "rnd", bool log = false)**

Возвращает: string - ответ от AI-сервиса
Описание: Отправляет запрос к AI-сервису с системным промптом и пользовательским контентом. Если модель не указана, выбирается случайная из доступных.
Параметры:
- systemContent - системный промпт для настройки поведения AI
- userContent - пользовательский запрос или контент для обработки
- aiModel - модель AI для использования (по умолчанию "rnd" - случайная)
- log - включить логирование запроса

Пример:
```csharp
string systemPrompt = "Ты помощник по программированию";
string userQuery = "Объясни что такое циклы в C#";
string response = ai.Query(systemPrompt, userQuery, "deepseek-ai/DeepSeek-R1-0528");
project.SendInfoToLog($"Ответ AI: {response}", false);
```

**GenerateTweet(string content, string bio = "", bool log = false)**

Возвращает: string - сгенерированный текст твита (до 220 символов)
Описание: Генерирует твит на основе переданного контента и биографии аккаунта. Автоматически проверяет длину и перегенерирует при превышении лимита.
Параметры:
- content - контент или тема для твита
- bio - биография аккаунта для персонализации стиля (необязательно)
- log - включить логирование

Пример:
```csharp
string tweetContent = "Новости из мира криптовалют";
string accountBio = "Криптоэнтузиаст и блокчейн-разработчик";
string tweet = ai.GenerateTweet(tweetContent, accountBio);
project.SendInfoToLog($"Сгенерирован твит: {tweet}", false);
```

**OptimizeCode(string content, bool log = false)**

Возвращает: string - оптимизированный код
Описание: Оптимизирует переданный код с точки зрения веб3-разработки. Возвращает только чистый код без объяснений и комментариев.
Параметры:
- content - исходный код для оптимизации
- log - включить логирование

Пример:
```csharp
string originalCode = "function transfer(address, amount) { /* неоптимальный код */ }";
string optimizedCode = ai.OptimizeCode(originalCode, true);
project.SendInfoToLog("Код оптимизирован", false);
```

**GoogleAppeal(bool log = false)**

Возвращает: string - текст обращения в службу поддержки Google (до 200 символов)
Описание: Генерирует обращение в службу поддержки Google для разблокировки аккаунта. Имитирует стиль обычного пользователя с небольшими грамматическими ошибками.
Параметры:
- log - включить логирование

Пример:
```csharp
string appealMessage = ai.GoogleAppeal();
project.SendInfoToLog($"Обращение в Google: {appealMessage}", false);
//отправить сообщение в форму поддержки
```