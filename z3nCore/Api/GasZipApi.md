# GazZip

Класс для выполнения операций межсетевого перевода токенов (рефьюел) через протокол GasZip. Позволяет автоматически находить подходящую сеть с достаточным балансом и выполнять переводы в указанную целевую сеть.

## Конструкторы

### GazZip(IZennoPosterProjectModel project, string key = null, bool log = false)

Создает новый экземпляр класса GazZip для работы с межсетевыми переводами.

**Параметры:**
- `project` - проект ZennoPoster для работы с логами и переменными
- `key` - ключ для операций (опционально)  
- `log` - включить подробное логирование операций

**Пример:**
```csharp
// создать экземпляр с логированием
var gazZip = new GazZip(project, log: true);

// создать экземпляр без логирования
var gazZip = new GazZip(project);
```

## Методы

### Refuel(string chainTo, decimal value, string rpc = null, bool log = false)

**Возвращает:** `string` - хеш транзакции или сообщение об ошибке

Выполняет межсетевой перевод указанной суммы ETH в целевую сеть через протокол GasZip.

**Параметры:**
- `chainTo` - целевая сеть (поддерживает: "sepolia", "soneum", "bsc", "gravity", "zero", "opbnb" или hex-адрес)
- `value` - сумма для перевода в ETH
- `rpc` - RPC эндпоинт исходной сети (если не указан, автоматически выбирается подходящая)
- `log` - включить детальное логирование процесса

**Пример:**
```csharp
var gazZip = new GazZip(project, log: true);

// автоматический выбор исходной сети
string txHash = gazZip.Refuel("sepolia", 0.001m);
project.SendInfoToLog($"Транзакция: {txHash}");

// использование конкретной исходной сети
string rpcUrl = "https://rpc.ankr.com/eth";
string result = gazZip.Refuel("bsc", 0.005m, rpcUrl, true);

if (result.StartsWith("fail:"))
{
    project.SendErrorToLog($"Ошибка: {result}");
}
else
{
    project.SendInfoToLog($"Успешно: {result}");
}
```