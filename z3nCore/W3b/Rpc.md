# Класс Rpc

**Описание**

Статический класс для получения URL-адресов RPC узлов различных блокчейн сетей. Предоставляет удобный доступ к публичным RPC эндпоинтам для популярных блокчейнов, включая Ethereum, Layer 2 сети, альтернативные блокчейны и тестовые сети.

**Публичные методы**

**Get(string name)**

Возвращает: string - URL-адрес RPC узла для указанной сети

Описание: Получает URL RPC узла по имени блокчейн сети. Поиск не чувствителен к регистру и игнорирует символы подчеркивания.

Параметры:
- name - название блокчейн сети

Пример:

```csharp
//получить URL для Ethereum
string ethRpc = Rpc.Get("Ethereum");
project.SendInfoToLog($"Ethereum RPC: {ethRpc}", false);

//получить URL для Solana (нечувствителен к регистру)
string solanaRpc = Rpc.Get("solana");
project.SendInfoToLog($"Solana RPC: {solanaRpc}", false);

//обработка ошибки для неизвестной сети
try
{
    string unknownRpc = Rpc.Get("UnknownChain");
}
catch (ArgumentException ex)
{
    project.SendErrorToLog($"Сеть не найдена: {ex.Message}", false);
}
```

**Публичные свойства**

Класс предоставляет статические свойства для быстрого доступа к популярным блокчейн сетям:

**Ethereum** - Ethereum mainnet RPC
**Arbitrum** - Arbitrum One RPC  
**Base** - Base mainnet RPC
**Blast** - Blast mainnet RPC
**Bsc** - Binance Smart Chain RPC
**Polygon** - Polygon mainnet RPC
**Avalanche** - Avalanche C-Chain RPC
**Optimism** - Optimism mainnet RPC
**Solana** - Solana mainnet RPC
**Zksync** - zkSync Era mainnet RPC

И многие другие сети включая тестовые (Sepolia, Solana_Devnet, Solana_Testnet)

Пример использования свойств:

```csharp
//прямой доступ к Ethereum RPC
string ethUrl = Rpc.Ethereum;
project.SendInfoToLog($"ETH RPC: {ethUrl}", false);

//использование для разных сетей
string[] networks = { Rpc.Arbitrum, Rpc.Base, Rpc.Polygon };
foreach(string rpc in networks)
{
    project.SendInfoToLog($"Network RPC: {rpc}", false);
}
```