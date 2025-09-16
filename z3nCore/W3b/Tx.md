# Tx

Класс для выполнения транзакций в блокчейн-сетях Ethereum. Предоставляет методы для отправки нативной валюты, ERC20 токенов, NFT, а также специальные операции типа approve и wrap.

## Конструкторы

### Tx(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса для работы с блокчейн-транзакциями.

**Параметры:**
- `project` - экземпляр проекта ZennoPoster
- `log` - включить логирование операций (по умолчанию false)

**Пример:**
```csharp
var tx = new Tx(project, log: true);
//создание экземпляра с включенным логированием
```

## Методы

### string SendTx(string chainRpc, string contractAddress, string encodedData, decimal value, string walletKey, int txType = 2, int speedup = 1)

Отправляет транзакцию в блокчейн с указанными параметрами.

**Возвращает:** string - хеш транзакции

**Параметры:**
- `chainRpc` - URL RPC узла блокчейн-сети
- `contractAddress` - адрес контракта или получателя
- `encodedData` - закодированные данные транзакции
- `value` - количество нативной валюты для отправки
- `walletKey` - приватный ключ кошелька (если пустой, берется из базы проекта)
- `txType` - тип транзакции (0 - legacy, 2 - EIP-1559, по умолчанию 2)
- `speedup` - множитель для увеличения комиссии (по умолчанию 1)

**Пример:**
```csharp
string hash = tx.SendTx("https://rpc.ankr.com/eth", "0x123...", "0xabcd...", 0.1m, "", 2, 2);
//отправка транзакции с удвоенной комиссией
project.SendInfoToLog($"Транзакция отправлена: {hash}");
```

### string Approve(string contract, string spender, string amount, string rpc)

Выдает разрешение на использование ERC20 токенов другому адресу или контракту.

**Возвращает:** string - хеш транзакции

**Параметры:**
- `contract` - адрес контракта токена
- `spender` - адрес, которому выдается разрешение
- `amount` - количество токенов ("max" для максимального значения, "cancel" для отмены)
- `rpc` - URL RPC узла

**Пример:**
```csharp
//выдать максимальное разрешение на USDT
string hash = tx.Approve("0xdAC17F958D2ee523a2206206994597C13D831ec7", "0x456...", "max", "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Approve выполнен: {hash}");
```

### string Wrap(string contract, decimal value, string rpc)

Обменивает нативную валюту на wrapped-токен (например, ETH на WETH).

**Возвращает:** string - хеш транзакции

**Параметры:**
- `contract` - адрес wrapped-контракта
- `value` - количество нативной валюты для обмена
- `rpc` - URL RPC узла

**Пример:**
```csharp
//обменять 1 ETH на WETH
string hash = tx.Wrap("0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2", 1.0m, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Wrapped {hash}");
```

### string SendNative(string to, decimal amount, string rpc)

Отправляет нативную валюту блокчейна на указанный адрес.

**Возвращает:** string - хеш транзакции

**Параметры:**
- `to` - адрес получателя
- `amount` - количество для отправки
- `rpc` - URL RPC узла

**Пример:**
```csharp
//отправить 0.5 ETH
string hash = tx.SendNative("0x789...", 0.5m, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Отправлено: {hash}");
```

### string SendERC20(string contract, string to, decimal amount, string rpc)

Отправляет ERC20 токены на указанный адрес.

**Возвращает:** string - хеш транзакции

**Параметры:**
- `contract` - адрес контракта токена
- `to` - адрес получателя
- `amount` - количество токенов для отправки
- `rpc` - URL RPC узла

**Пример:**
```csharp
//отправить 100 USDT
string hash = tx.SendERC20("0xdAC17F958D2ee523a2206206994597C13D831ec7", "0x789...", 100m, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Токены отправлены: {hash}");
```

### string SendERC721(string contract, string to, BigInteger tokenId, string rpc)

Отправляет NFT (ERC-721 токен) на указанный адрес.

**Возвращает:** string - хеш транзакции

**Параметры:**
- `contract` - адрес контракта NFT
- `to` - адрес получателя
- `tokenId` - ID токена для отправки
- `rpc` - URL RPC узла

**Пример:**
```csharp
//отправить NFT с ID 123
string hash = tx.SendERC721("0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D", "0x789...", 123, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"NFT отправлена: {hash}");
```