Класс EvmTools

Набор инструментов для работы с блокчейнами на базе Ethereum Virtual Machine (EVM). Предоставляет методы для проверки транзакций, получения балансов различных токенов, а также взаимодействия с RPC-узлами блокчейна.

**Публичные методы**

**WaitTxExtended(string rpc, string hash, int deadline = 60, string proxy = "", bool log = false)**

Возвращает: bool - статус выполнения транзакции (true = успех, false = провал)  
Описание: Расширенная версия ожидания транзакции с дополнительной информацией о gas и статусе. Отслеживает как завершенные транзакции, так и находящиеся в ожидании.

Параметры:
- rpc - URL RPC-узла блокчейна
- hash - хэш транзакции для отслеживания
- deadline - максимальное время ожидания в секундах (по умолчанию 60)
- proxy - прокси в формате "login:pass:ip:port" (опционально)
- log - включить подробное логирование процесса

Пример:
```csharp
var evmTools = new EvmTools();
string transactionHash = "0x123abc...";
bool success = await evmTools.WaitTxExtended("https://rpc.ankr.com/eth", transactionHash, 120, "", true);
if (success) 
{
    project.SendInfoToLog("Транзакция выполнена успешно", false);
} 
else 
{
    project.SendWarningToLog("Транзакция провалилась", false);
}
```

**WaitTx(string rpc, string hash, int deadline = 60, string proxy = "", bool log = false)**

Возвращает: bool - статус выполнения транзакции  
Описание: Базовая версия ожидания подтверждения транзакции в блокчейне. Проверяет статус транзакции до ее завершения или истечения времени.

Параметры:
- rpc - URL RPC-узла блокчейна
- hash - хэш транзакции
- deadline - максимальное время ожидания в секундах
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var evmTools = new EvmTools();
bool result = await evmTools.WaitTx("https://bsc-dataseed.binance.org", "0xabc123...", 90);
//проверить результат транзакции
if (result) project.SendInfoToLog("Транзакция подтверждена", false);
```

**Native(string rpc, string address)**

Возвращает: string - баланс в hex-формате  
Описание: Получает баланс нативной криптовалюты (ETH, BNB, MATIC и т.д.) для указанного адреса.

Параметры:
- rpc - URL RPC-узла
- address - адрес кошелька для проверки

Пример:
```csharp
var evmTools = new EvmTools();
string hexBalance = await evmTools.Native("https://polygon-rpc.com", "0x742d35Cc6B...");
//преобразовать hex в decimal используя расширения
decimal balance = hexBalance.ToDecimal(18);
project.SendInfoToLog($"Баланс: {balance} MATIC", false);
```

**Erc20(string tokenContract, string rpc, string address)**

Возвращает: string - баланс токена в hex-формате  
Описание: Получает баланс ERC-20 токена для указанного адреса кошелька.

Параметры:
- tokenContract - адрес контракта токена
- rpc - URL RPC-узла
- address - адрес кошелька

Пример:
```csharp
var evmTools = new EvmTools();
string usdcContract = "0xA0b86a33E6.."; 
string balance = await evmTools.Erc20(usdcContract, "https://mainnet.infura.io/v3/key", "0x123...");
//конвертировать в readable формат
decimal readableBalance = balance.ToDecimal(6); //USDC имеет 6 десятичных знаков
project.SendInfoToLog($"Баланс USDC: {readableBalance}", false);
```

**Erc721(string tokenContract, string rpc, string address)**

Возвращает: string - количество NFT в hex-формате  
Описание: Получает количество NFT-токенов стандарта ERC-721, принадлежащих указанному адресу.

Параметры:
- tokenContract - адрес контракта NFT-коллекции
- rpc - URL RPC-узла
- address - адрес кошелька

Пример:
```csharp
var evmTools = new EvmTools();
string nftContract = "0x60E4d786628Fea6478F785A6d7e704777c86a7c6"; //MAYC
string count = await evmTools.Erc721(nftContract, "https://eth.llamarpc.com", "0xabc123...");
int nftCount = Convert.ToInt32(count, 16);
project.SendInfoToLog($"У адреса {nftCount} NFT", false);
```

**Erc1155(string tokenContract, string tokenId, string rpc, string address)**

Возвращает: string - баланс токена ERC-1155 в hex-формате  
Описание: Получает баланс конкретного токена стандарта ERC-1155 для указанного адреса.

Параметры:
- tokenContract - адрес контракта ERC-1155
- tokenId - ID конкретного токена в коллекции
- rpc - URL RPC-узла
- address - адрес кошелька

Пример:
```csharp
var evmTools = new EvmTools();
string gameItemContract = "0xdef456...";
string tokenId = "1"; //первый предмет в игре
string balance = await evmTools.Erc1155(gameItemContract, tokenId, "https://polygon.llamarpc.com", "0x123...");
int itemCount = Convert.ToInt32(balance, 16);
project.SendInfoToLog($"Игровых предметов: {itemCount}", false);
```

**Nonce(string rpc, string address, string proxy = "", bool log = false)**

Возвращает: string - nonce в hex-формате  
Описание: Получает nonce (счетчик транзакций) для указанного адреса. Используется при создании новых транзакций.

Параметры:
- rpc - URL RPC-узла
- address - адрес кошелька
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var evmTools = new EvmTools();
string nonceHex = await evmTools.Nonce("https://mainnet.infura.io/v3/key", "0xabc123...");
int currentNonce = Convert.ToInt32(nonceHex, 16);
project.SendInfoToLog($"Текущий nonce: {currentNonce}", false);
```

**ChainId(string rpc, string proxy = "", bool log = false)**

Возвращает: string - ID блокчейна в hex-формате  
Описание: Получает уникальный идентификатор блокчейна для указанного RPC-узла.

Параметры:
- rpc - URL RPC-узла
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var evmTools = new EvmTools();
string chainIdHex = await evmTools.ChainId("https://polygon-rpc.com");
int chainId = Convert.ToInt32(chainIdHex, 16);
//137 = Polygon, 1 = Ethereum, 56 = BSC
project.SendInfoToLog($"Chain ID: {chainId}", false);
```

**GasPrice(string rpc, string proxy = "", bool log = false)**

Возвращает: string - цена gas в hex-формате  
Описание: Получает текущую цену gas в блокчейне для оптимизации стоимости транзакций.

Параметры:
- rpc - URL RPC-узла
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var evmTools = new EvmTools();
string gasPriceHex = await evmTools.GasPrice("https://eth.llamarpc.com");
decimal gasPrice = gasPriceHex.ToDecimal(9); //gas измеряется в Gwei (9 decimals)
project.SendInfoToLog($"Текущая цена gas: {gasPrice} Gwei", false);
```

---

Класс SolTools

Инструменты для работы с блокчейном Solana. Предоставляет методы для получения балансов SOL, SPL-токенов и расчета комиссий за транзакции.

**Публичные методы**

**GetSolanaBalance(string rpc, string address)**

Возвращает: decimal - баланс SOL  
Описание: Получает баланс нативной валюты SOL для указанного адреса кошелька.

Параметры:
- rpc - URL RPC-узла Solana
- address - адрес кошелька Solana

Пример:
```csharp
var solTools = new SolTools();
decimal solBalance = await solTools.GetSolanaBalance("https://api.mainnet-beta.solana.com", "DYw8jCTfwHNRJhhmFcbXvVDTqWMEVFBX6ZKUmG5CNSKC");
project.SendInfoToLog($"Баланс SOL: {solBalance}", false);
```

**GetSplTokenBalance(string rpc, string walletAddress, string tokenMint)**

Возвращает: decimal - баланс SPL-токена  
Описание: Получает баланс конкретного SPL-токена для указанного кошелька в блокчейне Solana.

Параметры:
- rpc - URL RPC-узла Solana
- walletAddress - адрес кошелька
- tokenMint - mint-адрес SPL-токена

Пример:
```csharp
var solTools = new SolTools();
string usdcMint = "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"; //USDC on Solana
decimal usdcBalance = await solTools.GetSplTokenBalance("https://api.mainnet-beta.solana.com", "DYw8jCT...", usdcMint);
project.SendInfoToLog($"Баланс USDC: {usdcBalance}", false);
```

**SolFeeByTx(string transactionHash, string rpc = null, string tokenDecimal = "9")**

Возвращает: decimal - размер комиссии в SOL  
Описание: Рассчитывает комиссию, уплаченную за конкретную транзакцию в сети Solana.

Параметры:
- transactionHash - хэш транзакции для анализа
- rpc - URL RPC-узла (по умолчанию mainnet)
- tokenDecimal - количество десятичных знаков (по умолчанию 9 для SOL)

Пример:
```csharp
var solTools = new SolTools();
string txHash = "5VERv8NMvzbJMEkV8xnrLkEaWRtSz9CosKDYjCJjBRnbJLgp8uirBgmQpjKhoR4tjF3ZpRzrFmBV6UjKdiSZkQUW";
decimal fee = await solTools.SolFeeByTx(txHash);
project.SendInfoToLog($"Комиссия за транзакцию: {fee} SOL", false);
```

---

Класс AptTools

Набор инструментов для работы с блокчейном Aptos. Включает методы для получения балансов нативной валюты APT и других токенов в экосистеме Aptos.

**Публичные методы**

**GetAptBalance(string rpc, string address, string proxy = "", bool log = false)**

Возвращает: decimal - баланс APT  
Описание: Получает баланс нативной валюты APT для указанного адреса в блокчейне Aptos.

Параметры:
- rpc - URL RPC-узла Aptos (по умолчанию mainnet)
- address - адрес кошелька Aptos
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var aptTools = new AptTools();
decimal aptBalance = await aptTools.GetAptBalance("https://fullnode.mainnet.aptoslabs.com/v1", "0x123abc...");
project.SendInfoToLog($"Баланс APT: {aptBalance}", false);
```

**GetAptTokenBalance(string coinType, string rpc, string address, string proxy = "", bool log = false)**

Возвращает: decimal - баланс токена  
Описание: Получает баланс конкретного токена в экосистеме Aptos по его типу.

Параметры:
- coinType - тип монеты/токена в формате Aptos
- rpc - URL RPC-узла (по умолчанию mainnet)
- address - адрес кошелька
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var aptTools = new AptTools();
string wethType = "0x1::aptos_coin::AptosCoin"; //пример типа токена
decimal tokenBalance = await aptTools.GetAptTokenBalance(wethType, "", "0xabc123...", "", true);
project.SendInfoToLog($"Баланс токена: {tokenBalance}", false);
```

---

Класс SuiTools

Инструменты для взаимодействия с блокчейном Sui. Предоставляет методы для получения балансов нативной валюты SUI и других токенов в экосистеме.

**Публичные методы**

**GetSuiBalance(string rpc, string address, string proxy = "", bool log = false)**

Возвращает: decimal - баланс SUI  
Описание: Получает баланс нативной валюты SUI для указанного адреса кошелька.

Параметры:
- rpc - URL RPC-узла Sui (по умолчанию mainnet)
- address - адрес кошелька Sui
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var suiTools = new SuiTools();
decimal suiBalance = await suiTools.GetSuiBalance("https://fullnode.mainnet.sui.io", "0x123abc...");
project.SendInfoToLog($"Баланс SUI: {suiBalance}", false);
```

**GetSuiTokenBalance(string coinType, string rpc, string address, string proxy = "", bool log = false)**

Возвращает: decimal - баланс токена  
Описание: Получает баланс конкретного токена в экосистеме Sui по его типу.

Параметры:
- coinType - тип монеты/токена в экосистеме Sui
- rpc - URL RPC-узла (по умолчанию mainnet)
- address - адрес кошелька
- proxy - прокси-сервер (опционально)
- log - включить логирование

Пример:
```csharp
var suiTools = new SuiTools();
string usdcType = "0x5d4b302506645c37ff133b98c4b50a5ae14841659738d6d733d59d0d217a93bf::coin::COIN";
decimal usdcBalance = await suiTools.GetSuiTokenBalance(usdcType, "", "0xabc123...", "", true);
project.SendInfoToLog($"Баланс USDC на Sui: {usdcBalance}", false);
```

---

Класс CoinGecco

API-клиент для работы с сервисом CoinGecko. Позволяет получать актуальную информацию о ценах криптовалют и данные о токенах.

**Публичные методы**

**CoinInfo(string CGid = "ethereum")**

Возвращает: string - JSON с полной информацией о монете  
Описание: Получает подробную информацию о криптовалюте по ее ID в CoinGecko.

Параметры:
- CGid - идентификатор монеты в CoinGecko (по умолчанию "ethereum")

Пример:
```csharp
var coinGecko = new CoinGecco();
string ethInfo = await coinGecko.CoinInfo("ethereum");
project.SendInfoToLog($"Информация об Ethereum: {ethInfo}", false);
```

**TokenByAddress(string CGid = "ethereum")**

Возвращает: string - JSON с информацией о токене по адресу  
Описание: Получает информацию о токене по его контрактному адресу в блокчейне.

Параметры:
- CGid - идентификатор сети для поиска токена

Пример:
```csharp
var coinGecko = new CoinGecco();
string tokenInfo = await coinGecko.TokenByAddress("ethereum");
//обработать полученную информацию
project.SendInfoToLog("Информация получена", false);
```

**PriceByTiker(string tiker)**

Возвращает: decimal - цена в USD  
Описание: Статический метод для быстрого получения цены криптовалюты по ее тикеру. Поддерживает ETH, BNB, SOL.

Параметры:
- tiker - тикер криптовалюты (ETH, BNB, SOL)

Пример:
```csharp
decimal ethPrice = CoinGecco.PriceByTiker("ETH");
decimal bnbPrice = CoinGecco.PriceByTiker("BNB");
project.SendInfoToLog($"ETH: ${ethPrice}, BNB: ${bnbPrice}", false);
```

**PriceById(string CGid = "ethereum")**

Возвращает: decimal - цена в USD  
Описание: Статический метод для получения цены криптовалюты по ее ID в CoinGecko.

Параметры:
- CGid - идентификатор монеты в CoinGecko

Пример:
```csharp
decimal solPrice = CoinGecko.PriceById("solana");
decimal dotPrice = CoinGecko.PriceById("polkadot");
project.SendInfoToLog($"SOL: ${solPrice}, DOT: ${dotPrice}", false);
```

---

Класс DexScreener

API-клиент для работы с сервисом DexScreener. Предоставляет информацию о токенах и их торговле на децентрализованных биржах.

**Публичные методы**

**CoinInfo(string contract, string chain)**

Возвращает: string - JSON с информацией о токене  
Описание: Получает информацию о токене по его контрактному адресу и блокчейну от DexScreener.

Параметры:
- contract - адрес контракта токена
- chain - название блокчейна (ethereum, solana, bsc и т.д.)

Пример:
```csharp
var dexScreener = new DexScreener();
string tokenContract = "0x6B175474E89094C44Da98b954EedeAC495271d0F"; //DAI
string tokenInfo = await dexScreener.CoinInfo(tokenContract, "ethereum");
project.SendInfoToLog($"Данные о токене: {tokenInfo}", false);
```

---

Класс KuCoin

API-клиент для работы с биржей KuCoin. Позволяет получать данные о ценах и торговых парах.

**Публичные методы**

**OrderbookByTiker(string ticker = "ETH")**

Возвращает: string - JSON с данными ордербука  
Описание: Получает данные ордербука (лучшие цены покупки и продажи) для торговой пары с USDT.

Параметры:
- ticker - тикер криптовалюты (по умолчанию "ETH")

Пример:
```csharp
var kuCoin = new KuCoin();
string btcOrderbook = await kuCoin.OrderbookByTiker("BTC");
project.SendInfoToLog($"Ордербук BTC: {btcOrderbook}", false);
```

**KuPrice(string tiker = "ETH")**

Возвращает: decimal - цена в USDT  
Описание: Статический метод для быстрого получения текущей цены торговой пары на KuCoin.

Параметры:
- tiker - тикер криптовалюты

Пример:
```csharp
decimal adaPrice = KuCoin.KuPrice("ADA");
decimal linkPrice = KuCoin.KuPrice("LINK");
project.SendInfoToLog($"ADA: ${adaPrice}, LINK: ${linkPrice}", false);
```

---

Класс W3bTools

Статический класс с упрощенными методами для работы с блокчейнами. Предоставляет удобные обертки над основными классами для быстрого доступа к функциональности.

**Публичные методы**

**EvmNative(string rpc, string address)**

Возвращает: decimal - баланс в читаемом формате  
Описание: Статический метод для получения баланса нативной валюты EVM-блокчейна с автоматическим преобразованием в decimal.

Параметры:
- rpc - URL RPC-узла
- address - адрес кошелька

Пример:
```csharp
decimal ethBalance = W3bTools.EvmNative("https://eth.llamarpc.com", "0x123abc...");
decimal bnbBalance = W3bTools.EvmNative("https://bsc-dataseed.binance.org", "0x123abc...");
project.SendInfoToLog($"ETH: {ethBalance}, BNB: {bnbBalance}", false);
```

**ERC20(string tokenContract, string rpc, string address, string tokenDecimal = "18")**

Возвращает: decimal - баланс токена  
Описание: Упрощенный метод для получения баланса ERC-20 токена с автоматическим преобразованием в читаемый формат.

Параметры:
- tokenContract - адрес контракта токена
- rpc - URL RPC-узла
- address - адрес кошелька
- tokenDecimal - количество десятичных знаков токена (по умолчанию 18)

Пример:
```csharp
string usdtContract = "0xdAC17F958D2ee523a2206206994597C13D831ec7";
decimal usdtBalance = W3bTools.ERC20(usdtContract, "https://eth.llamarpc.com", "0x123abc...", "6");
project.SendInfoToLog($"Баланс USDT: {usdtBalance}", false);
```

**WaitTx(string rpc, string hash, int deadline = 60, string proxy = "", bool log = false, bool extended = false)**

Возвращает: bool - статус транзакции  
Описание: Упрощенный метод ожидания подтверждения транзакции с выбором базовой или расширенной версии.

Параметры:
- rpc - URL RPC-узла
- hash - хэш транзакции
- deadline - время ожидания в секундах
- proxy - прокси-сервер (опционально)
- log - включить логирование
- extended - использовать расширенную версию с дополнительной информацией

Пример:
```csharp
bool success = W3bTools.WaitTx("https://polygon-rpc.com", "0xabc123...", 120, "", true, true);
//проверить результат
if (success) project.SendInfoToLog("Транзакция успешна", false);
```

**SolNative(string address, string rpc = "https://api.mainnet-beta.solana.com")**

Возвращает: decimal - баланс SOL  
Описание: Упрощенный метод для получения баланса SOL.

Параметры:
- address - адрес кошелька Solana
- rpc - URL RPC-узла (по умолчанию mainnet)

Пример:
```csharp
decimal solBalance = W3bTools.SolNative("DYw8jCTfwHNRJhhmFcbXvVDTqWMEVFBX6ZKUmG5CNSKC");
project.SendInfoToLog($"Баланс SOL: {solBalance}", false);
```

**CGPrice(string CGid = "ethereum")**

Возвращает: decimal - цена в USD  
Описание: Упрощенный метод для получения цены криптовалюты через CoinGecko API.

Параметры:
- CGid - идентификатор монеты в CoinGecko

Пример:
```csharp
decimal btcPrice = W3bTools.CGPrice("bitcoin");
decimal ethPrice = W3bTools.CGPrice("ethereum");
project.SendInfoToLog($"BTC: ${btcPrice}, ETH: ${ethPrice}", false);
```

**UsdToToken(decimal usdAmount, string tiker, string apiProvider = "KuCoin")**

Возвращает: decimal - количество токенов  
Описание: Конвертирует сумму в USD в количество токенов по текущему курсу через выбранный API.

Параметры:
- usdAmount - сумма в долларах для конвертации
- tiker - тикер криптовалюты
- apiProvider - источник цен (KuCoin, OKX, CoinGecco)

Пример:
```csharp
//конвертировать $100 в ETH по курсу KuCoin
decimal ethAmount = project.UsdToToken(100m, "ETH", "KuCoin");
project.SendInfoToLog($"$100 = {ethAmount} ETH", false);
```