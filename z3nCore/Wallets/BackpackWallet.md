# BackpackWallet

Класс для автоматизации работы с кошельком Backpack в браузере Chrome. Обеспечивает установку, импорт, разблокировку и управление кошельком через расширение браузера.

## Конструкторы

### BackpackWallet(IZennoPosterProjectModel project, Instance instance, bool log = false, string key = null, string fileName = "Backpack0.10.94.crx")

Создает новый экземпляр класса для работы с кошельком Backpack.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `instance` - экземпляр браузера
- `log` - включить логирование (по умолчанию false)
- `key` - приватный ключ или seed фраза ("key", "seed" или конкретный ключ)
- `fileName` - имя файла расширения (по умолчанию "Backpack0.10.94.crx")

**Пример:**
```csharp
var wallet = new BackpackWallet(project, instance, log: true, key: "key");
//создать экземпляр с логированием и ключом из базы данных
```

## Методы

### Launch(string fileName = null, bool log = false) : string

Запускает кошелек Backpack, устанавливает расширение и импортирует/разблокирует кошелек.

**Возвращает:** Адрес активного кошелька

**Параметры:**
- `fileName` - имя файла расширения (необязательно)
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
string address = wallet.Launch(log: true);
project.SendInfoToLog($"Адрес кошелька: {address}");
//запустить кошелек и получить адрес
```

### ActiveAddress(bool log = false) : string

Получает адрес текущего активного кошелька.

**Возвращает:** Адрес активного кошелька

**Параметры:**
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
string address = wallet.ActiveAddress();
project.SendInfoToLog($"Текущий адрес: {address}");
//получить адрес активного кошелька
```

### CurrentChain(bool log = true) : string

Определяет текущую блокчейн-сеть кошелька.

**Возвращает:** Название сети ("mainnet", "devnet", "testnet", "ethereum")

**Параметры:**
- `log` - включить логирование (по умолчанию true)

**Пример:**
```csharp
string chain = wallet.CurrentChain();
//определить текущую сеть
if (chain == "mainnet") 
{
    project.SendInfoToLog("Используется основная сеть");
}
```

### Unlock(bool log = false) : void

Разблокирует кошелек с помощью пароля.

**Параметры:**
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
wallet.Unlock(log: true);
//разблокировать кошелек
project.SendInfoToLog("Кошелек разблокирован");
```

### Approve(bool log = false) : void

Подтверждает действие в кошельке (транзакцию, подключение и т.д.).

**Параметры:**
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
wallet.Approve();
//подтвердить действие в кошельке
project.SendInfoToLog("Действие подтверждено");
```

### Connect(bool log = false) : void

Подключает кошелек к веб-сайту, обрабатывает запросы на подключение.

**Параметры:**
- `log` - включить логирование (по умолчанию false)

**Пример:**
```csharp
wallet.Connect();
//подключить кошелек к сайту
project.SendInfoToLog("Кошелек подключен");
```

### Devmode(bool enable = true) : void

Включает или выключает режим разработчика в кошельке.

**Параметры:**
- `enable` - включить режим разработчика (по умолчанию true)

**Пример:**
```csharp
wallet.Devmode(true);
//включить режим разработчика
project.SendInfoToLog("Режим разработчика активирован");
```

### DevChain(string reqmode = "devnet") : void

Переключает кошелек на тестовую сеть (devnet, testnet).

**Параметры:**
- `reqmode` - требуемый режим сети (по умолчанию "devnet")

**Пример:**
```csharp
wallet.DevChain("testnet");
//переключиться на тестовую сеть
project.SendInfoToLog("Переключено на testnet");
```

### Add(string type = "Ethereum", string source = "key") : void

Добавляет новый кошелек указанного типа.

**Параметры:**
- `type` - тип блокчейна ("Ethereum" или "Solana")
- `source` - источник импорта ("key" или "phrase")

**Пример:**
```csharp
wallet.Add("Solana", "key");
//добавить Solana кошелек из приватного ключа
project.SendInfoToLog("Solana кошелек добавлен");
```

### Switch(string type) : void

Переключается между кошельками разных типов.

**Параметры:**
- `type` - тип кошелька для переключения ("Solana" или "Ethereum")

**Пример:**
```csharp
wallet.Switch("Ethereum");
//переключиться на Ethereum кошелек
project.SendInfoToLog("Переключено на Ethereum");
```

### Current() : string

Определяет тип текущего активного кошелька.

**Возвращает:** Тип кошелька ("Solana", "Ethereum" или "Undefined")

**Пример:**
```csharp
string currentType = wallet.Current();
//определить тип текущего кошелька
project.SendInfoToLog($"Текущий тип: {currentType}");
```