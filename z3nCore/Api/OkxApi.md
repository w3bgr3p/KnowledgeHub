# OkxApi

Класс для работы с OKX API, предоставляющий функциональность для управления криптовалютными счетами, субсчетами, выводом средств и получением рыночных данных.

## Конструктор

### OkxApi(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр OkxApi для работы с биржей OKX.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `log` - включить ли логирование (по умолчанию false)

```csharp
// Создание экземпляра OkxApi
var okx = new OkxApi(project, log: true);
```

## Методы

### OKXGetSubAccs(string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список имен субсчетов

Получает список всех субсчетов на основном аккаунте OKX.

**Параметры:**
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Получить список всех субсчетов
var subAccounts = okx.OKXGetSubAccs();
foreach (string subName in subAccounts)
{
    project.SendInfoToLog($"Найден субсчет: {subName}");
}
```

### OKXGetSubMax(string accName, string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список в формате "валюта:максимальная_сумма_вывода"

Получает максимальные суммы для вывода с торгового счета указанного субаккаунта.

**Параметры:**
- `accName` - имя субаккаунта
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Получить максимальные суммы для вывода с торгового счета
var maxWithdrawals = okx.OKXGetSubMax("sub1");
foreach (string balance in maxWithdrawals)
{
    project.SendInfoToLog($"Доступно для вывода: {balance}");
}
```

### OKXGetSubTrading(string accName, string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список торговых балансов

Получает торговые балансы указанного субаккаунта.

**Параметры:**
- `accName` - имя субаккаунта
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Получить торговые балансы субсчета
var tradingBalances = okx.OKXGetSubTrading("sub1");
```

### OKXGetSubFunding(string accName, string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список в формате "валюта:доступный_баланс"

Получает балансы фандинг-счета указанного субаккаунта.

**Параметры:**
- `accName` - имя субаккаунта
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Получить балансы фандинг-счета
var fundingBalances = okx.OKXGetSubFunding("sub1");
foreach (string balance in fundingBalances)
{
    string[] parts = balance.Split(':');
    project.SendInfoToLog($"Валюта: {parts[0]}, Баланс: {parts[1]}");
}
```

### OKXGetSubsBal(string proxy = null, bool log = false)

**Возвращает:** `List<string>` - список в формате "субсчет:валюта:баланс"

Получает все ненулевые балансы со всех субсчетов (как с фандинг, так и с торговых счетов).

**Параметры:**
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Получить все балансы со всех субсчетов
var allBalances = okx.OKXGetSubsBal(log: true);
foreach (string balance in allBalances)
{
    project.SendInfoToLog($"Баланс: {balance}");
}
```

### OKXWithdraw(string toAddress, string currency, string chain, decimal amount, decimal fee, string proxy = null, bool log = false)

**Возвращает:** `void`

Выводит криптовалюту на указанный адрес.

**Параметры:**
- `toAddress` - адрес получателя
- `currency` - валюта для вывода (например, "USDT")
- `chain` - блокчейн сеть (ethereum, bsc, polygon и др.)
- `amount` - сумма для вывода
- `fee` - комиссия за вывод
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Вывести 100 USDT на Ethereum
okx.OKXWithdraw(
    "0x1234567890abcdef1234567890abcdef12345678", 
    "USDT", 
    "ethereum", 
    100m, 
    1m
);
```

### OKXCreateSub(string subName, string accountType = "1", string proxy = null, bool log = false)

**Возвращает:** `void`

Создает новый субаккаунт.

**Параметры:**
- `subName` - имя нового субаккаунта
- `accountType` - тип аккаунта (по умолчанию "1")
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Создать новый субаккаунт
okx.OKXCreateSub("myNewSubAccount");
```

### OKXDrainSubs()

**Возвращает:** `void`

Переводит все доступные средства со всех субсчетов на основной счет.

```csharp
// Собрать все средства с субсчетов на основной счет
okx.OKXDrainSubs();
project.SendInfoToLog("Все средства переведены на основной счет");
```

### OKXAddMaxSubs()

**Возвращает:** `void`

Создает максимально возможное количество субсчетов до достижения лимита.

```csharp
// Создать максимальное количество субсчетов
okx.OKXAddMaxSubs();
project.SendInfoToLog("Созданы все возможные субсчета");
```

### OKXPrice<T>(string pair, string proxy = null, bool log = false)

**Возвращает:** `T` - цена в указанном типе данных

Получает текущую рыночную цену торговой пары.

**Параметры:**
- `pair` - торговая пара (например, "BTC-USDT")
- `proxy` - прокси для запроса (необязательно)
- `log` - включить логирование (по умолчанию false)

```csharp
// Получить цену BTC в USDT как decimal
decimal btcPrice = okx.OKXPrice<decimal>("BTC-USDT");
project.SendInfoToLog($"Цена BTC: {btcPrice} USDT");

// Получить цену как строку
string priceString = okx.OKXPrice<string>("ETH-USDT");
```