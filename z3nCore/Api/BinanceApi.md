# BinanceApi

Класс для работы с API биржи Binance в ZennoPoster. Позволяет выводить средства, получать информацию о балансах и истории выводов. Автоматически обрабатывает авторизацию и подпись запросов.

## Конструкторы

### BinanceApi(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса для работы с Binance API.

**Параметры:**
- `project` - модель проекта ZennoPoster
- `log` - включить ли логирование (по умолчанию false)

**Пример:**
```csharp
var binanceApi = new BinanceApi(project, log: true);
//создать экземпляр с включенным логированием
```

## Методы

### string Withdraw(string coin, string network, string address, string amount)

**Возвращает:** Ответ сервера в виде JSON строки

Выполняет вывод криптовалюты на указанный адрес через выбранную сеть.

**Параметры:**
- `coin` - символ монеты (например, "USDT")
- `network` - название сети (например, "ethereum", "bsc", "arbitrum")
- `address` - адрес кошелька для вывода
- `amount` - сумма вывода

**Пример:**
```csharp
string result = binanceApi.Withdraw("USDT", "ethereum", "0x742d35Cc6565C42c3928...", "100");
project.SendInfoToLog($"Результат вывода: {result}");
```

### Dictionary<string, string> GetUserAsset()

**Возвращает:** Словарь с балансами всех активов (ключ - символ монеты, значение - доступный баланс)

Получает информацию о всех балансах пользователя.

**Пример:**
```csharp
var balances = binanceApi.GetUserAsset();
foreach(var balance in balances)
{
    project.SendInfoToLog($"{balance.Key}: {balance.Value}");
    //вывести баланс каждой монеты
}
```

### string GetUserAsset(string coin)

**Возвращает:** Доступный баланс указанной монеты в виде строки

Получает баланс конкретной криптовалюты.

**Параметры:**
- `coin` - символ монеты

**Пример:**
```csharp
string usdtBalance = binanceApi.GetUserAsset("USDT");
project.SendInfoToLog($"Баланс USDT: {usdtBalance}");
```

### List<string> GetWithdrawHistory()

**Возвращает:** Список строк с историей выводов в формате "id:amount:coin:status"

Получает полную историю всех выводов.

**Пример:**
```csharp
var history = binanceApi.GetWithdrawHistory();
foreach(string withdrawal in history)
{
    project.SendInfoToLog($"Вывод: {withdrawal}");
    //показать каждую транзакцию
}
```

### string GetWithdrawHistory(string searchId = "")

**Возвращает:** Информацию о конкретной транзакции или "NoIdFound: {searchId}"

Поиск конкретной транзакции вывода по ID.

**Параметры:**
- `searchId` - ID транзакции для поиска (необязательный)

**Пример:**
```csharp
string transaction = binanceApi.GetWithdrawHistory("12345");
if(!transaction.Contains("NoIdFound"))
{
    project.SendInfoToLog($"Найдена транзакция: {transaction}");
}
```