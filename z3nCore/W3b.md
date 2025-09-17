[SPOILER= W3b] 

[SPOILER= Rpc]

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


[CODE=csharp]
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
[/CODE]

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


[CODE=csharp]
//прямой доступ к Ethereum RPC

string ethUrl = Rpc.Ethereum;
project.SendInfoToLog($"ETH RPC: {ethUrl}", false);

//использование для разных сетей

string[] networks = { Rpc.Arbitrum, Rpc.Base, Rpc.Polygon };
foreach(string rpc in networks)
{
project.SendInfoToLog($"Network RPC: {rpc}", false);
}
[/CODE][/SPOILER]
[SPOILER= Tx]

# Tx

Класс для выполнения транзакций в блокчейн-сетях Ethereum. Предоставляет методы для отправки нативной валюты, ERC20 токенов, NFT, а также специальные операции типа approve и wrap.


## Конструкторы


### Tx(IZennoPosterProjectModel project, bool log = false)

Создает новый экземпляр класса для работы с блокчейн-транзакциями.


**Параметры:**

- `project` - экземпляр проекта ZennoPoster

- `log` - включить логирование операций (по умолчанию false)


**Пример:**

[CODE=csharp]
var tx = new Tx(project, log: true);
//создание экземпляра с включенным логированием

[/CODE]

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

[CODE=csharp]
string hash = tx.SendTx("https://rpc.ankr.com/eth", "0x123...", "0xabcd...", 0.1m, "", 2, 2);
//отправка транзакции с удвоенной комиссией

project.SendInfoToLog($"Транзакция отправлена: {hash}");

[/CODE]

### string Approve(string contract, string spender, string amount, string rpc)

Выдает разрешение на использование ERC20 токенов другому адресу или контракту.


**Возвращает:** string - хеш транзакции


**Параметры:**

- `contract` - адрес контракта токена

- `spender` - адрес, которому выдается разрешение

- `amount` - количество токенов ("max" для максимального значения, "cancel" для отмены)

- `rpc` - URL RPC узла


**Пример:**

[CODE=csharp]
//выдать максимальное разрешение на USDT

string hash = tx.Approve("0xdAC17F958D2ee523a2206206994597C13D831ec7", "0x456...", "max", "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Approve выполнен: {hash}");

[/CODE]

### string Wrap(string contract, decimal value, string rpc)

Обменивает нативную валюту на wrapped-токен (например, ETH на WETH).


**Возвращает:** string - хеш транзакции


**Параметры:**

- `contract` - адрес wrapped-контракта

- `value` - количество нативной валюты для обмена

- `rpc` - URL RPC узла


**Пример:**

[CODE=csharp]
//обменять 1 ETH на WETH

string hash = tx.Wrap("0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2", 1.0m, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Wrapped {hash}");
[/CODE]

### string SendNative(string to, decimal amount, string rpc)

Отправляет нативную валюту блокчейна на указанный адрес.


**Возвращает:** string - хеш транзакции


**Параметры:**

- `to` - адрес получателя

- `amount` - количество для отправки

- `rpc` - URL RPC узла


**Пример:**

[CODE=csharp]
//отправить 0.5 ETH

string hash = tx.SendNative("0x789...", 0.5m, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Отправлено: {hash}");

[/CODE]

### string SendERC20(string contract, string to, decimal amount, string rpc)

Отправляет ERC20 токены на указанный адрес.


**Возвращает:** string - хеш транзакции


**Параметры:**

- `contract` - адрес контракта токена

- `to` - адрес получателя

- `amount` - количество токенов для отправки

- `rpc` - URL RPC узла


**Пример:**

[CODE=csharp]
//отправить 100 USDT

string hash = tx.SendERC20("0xdAC17F958D2ee523a2206206994597C13D831ec7", "0x789...", 100m, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"Токены отправлены: {hash}");

[/CODE]

### string SendERC721(string contract, string to, BigInteger tokenId, string rpc)

Отправляет NFT (ERC-721 токен) на указанный адрес.


**Возвращает:** string - хеш транзакции


**Параметры:**

- `contract` - адрес контракта NFT

- `to` - адрес получателя

- `tokenId` - ID токена для отправки

- `rpc` - URL RPC узла


**Пример:**

[CODE=csharp]
//отправить NFT с ID 123

string hash = tx.SendERC721("0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D", "0x789...", 123, "https://rpc.ankr.com/eth");
project.SendInfoToLog($"NFT отправлена: {hash}");

[/CODE][/SPOILER]
[SPOILER= W3b]

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

[CODE=csharp]
var evmTools = new EvmTools();
string transactionHash = "0x123abc...";
bool success = await evmTools.WaitTxExtended("https://rpc.ankr.com/eth", transactionHash, 120, "", true);
if (success) 
{
} 
else 
{
project.SendWarningToLog("Транзакция провалилась", false);

}
[/CODE]

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

[CODE=csharp]
var evmTools = new EvmTools();
bool result = await evmTools.WaitTx("https://bsc-dataseed.binance.org", "0xabc123...", 90);
//проверить результат транзакции

[/CODE]

**Native(string rpc, string address)**

Возвращает: string - баланс в hex-формате  

Описание: Получает баланс нативной криптовалюты (ETH, BNB, MATIC и т.д.) для указанного адреса.


Параметры:

- rpc - URL RPC-узла

- address - адрес кошелька для проверки


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string hexBalance = await evmTools.Native("https://polygon-rpc.com", "0x742d35Cc6B...");
//преобразовать hex в decimal используя расширения

decimal balance = hexBalance.ToDecimal(18);
project.SendInfoToLog($"Баланс: {balance} MATIC", false);

[/CODE]

**Erc20(string tokenContract, string rpc, string address)**

Возвращает: string - баланс токена в hex-формате  

Описание: Получает баланс ERC-20 токена для указанного адреса кошелька.


Параметры:

- tokenContract - адрес контракта токена

- rpc - URL RPC-узла

- address - адрес кошелька


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string usdcContract = "0xA0b86a33E6.."; 
string balance = await evmTools.Erc20(usdcContract, "https://mainnet.infura.io/v3/key", "0x123...");
//конвертировать в readable формат

decimal readableBalance = balance.ToDecimal(6); //USDC имеет 6 десятичных знаков

project.SendInfoToLog($"Баланс USDC: {readableBalance}", false);

[/CODE]

**Erc721(string tokenContract, string rpc, string address)**

Возвращает: string - количество NFT в hex-формате  

Описание: Получает количество NFT-токенов стандарта ERC-721, принадлежащих указанному адресу.


Параметры:

- tokenContract - адрес контракта NFT-коллекции

- rpc - URL RPC-узла

- address - адрес кошелька


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string nftContract = "0x60E4d786628Fea6478F785A6d7e704777c86a7c6"; //MAYC
string count = await evmTools.Erc721(nftContract, "https://eth.llamarpc.com", "0xabc123...");
int nftCount = Convert.ToInt32(count, 16);
project.SendInfoToLog($"У адреса {nftCount} NFT", false);

[/CODE]

**Erc1155(string tokenContract, string tokenId, string rpc, string address)**

Возвращает: string - баланс токена ERC-1155 в hex-формате  

Описание: Получает баланс конкретного токена стандарта ERC-1155 для указанного адреса.


Параметры:

- tokenContract - адрес контракта ERC-1155

- tokenId - ID конкретного токена в коллекции

- rpc - URL RPC-узла

- address - адрес кошелька


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string gameItemContract = "0xdef456...";
string tokenId = "1"; //первый предмет в игре

string balance = await evmTools.Erc1155(gameItemContract, tokenId, "https://polygon.llamarpc.com", "0x123...");
int itemCount = Convert.ToInt32(balance, 16);
project.SendInfoToLog($"Игровых предметов: {itemCount}", false);

[/CODE]

**Nonce(string rpc, string address, string proxy = "", bool log = false)**

Возвращает: string - nonce в hex-формате  

Описание: Получает nonce (счетчик транзакций) для указанного адреса. Используется при создании новых транзакций.


Параметры:

- rpc - URL RPC-узла

- address - адрес кошелька

- proxy - прокси-сервер (опционально)

- log - включить логирование


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string nonceHex = await evmTools.Nonce("https://mainnet.infura.io/v3/key", "0xabc123...");
int currentNonce = Convert.ToInt32(nonceHex, 16);
project.SendInfoToLog($"Текущий nonce: {currentNonce}", false);

[/CODE]

**ChainId(string rpc, string proxy = "", bool log = false)**

Возвращает: string - ID блокчейна в hex-формате  

Описание: Получает уникальный идентификатор блокчейна для указанного RPC-узла.


Параметры:

- rpc - URL RPC-узла

- proxy - прокси-сервер (опционально)

- log - включить логирование


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string chainIdHex = await evmTools.ChainId("https://polygon-rpc.com");
int chainId = Convert.ToInt32(chainIdHex, 16);
//137 = Polygon, 1 = Ethereum, 56 = BSC
project.SendInfoToLog($"Chain ID: {chainId}", false);
[/CODE]

**GasPrice(string rpc, string proxy = "", bool log = false)**

Возвращает: string - цена gas в hex-формате  

Описание: Получает текущую цену gas в блокчейне для оптимизации стоимости транзакций.


Параметры:

- rpc - URL RPC-узла

- proxy - прокси-сервер (опционально)

- log - включить логирование


Пример:

[CODE=csharp]
var evmTools = new EvmTools();
string gasPriceHex = await evmTools.GasPrice("https://eth.llamarpc.com");
decimal gasPrice = gasPriceHex.ToDecimal(9); //gas измеряется в Gwei (9 decimals)

project.SendInfoToLog($"Текущая цена gas: {gasPrice} Gwei", false);

[/CODE]

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

[CODE=csharp]
var solTools = new SolTools();
decimal solBalance = await solTools.GetSolanaBalance("https://api.mainnet-beta.solana.com", "DYw8jCTfwHNRJhhmFcbXvVDTqWMEVFBX6ZKUmG5CNSKC");
project.SendInfoToLog($"Баланс SOL: {solBalance}", false);

[/CODE]

**GetSplTokenBalance(string rpc, string walletAddress, string tokenMint)**

Возвращает: decimal - баланс SPL-токена  

Описание: Получает баланс конкретного SPL-токена для указанного кошелька в блокчейне Solana.


Параметры:

- rpc - URL RPC-узла Solana

- walletAddress - адрес кошелька

- tokenMint - mint-адрес SPL-токена


Пример:

[CODE=csharp]
var solTools = new SolTools();
string usdcMint = "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"; //USDC on Solana
decimal usdcBalance = await solTools.GetSplTokenBalance("https://api.mainnet-beta.solana.com", "DYw8jCT...", usdcMint);
project.SendInfoToLog($"Баланс USDC: {usdcBalance}", false);

[/CODE]

**SolFeeByTx(string transactionHash, string rpc = null, string tokenDecimal = "9")**

Возвращает: decimal - размер комиссии в SOL  

Описание: Рассчитывает комиссию, уплаченную за конкретную транзакцию в сети Solana.


Параметры:

- transactionHash - хэш транзакции для анализа

- rpc - URL RPC-узла (по умолчанию mainnet)

- tokenDecimal - количество десятичных знаков (по умолчанию 9 для SOL)


Пример:

[CODE=csharp]
var solTools = new SolTools();
string txHash = "5VERv8NMvzbJMEkV8xnrLkEaWRtSz9CosKDYjCJjBRnbJLgp8uirBgmQpjKhoR4tjF3ZpRzrFmBV6UjKdiSZkQUW";
decimal fee = await solTools.SolFeeByTx(txHash);
project.SendInfoToLog($"Комиссия за транзакцию: {fee} SOL", false);

[/CODE]

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

[CODE=csharp]
var aptTools = new AptTools();
decimal aptBalance = await aptTools.GetAptBalance("https://fullnode.mainnet.aptoslabs.com/v1", "0x123abc...");
project.SendInfoToLog($"Баланс APT: {aptBalance}", false);

[/CODE]

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

[CODE=csharp]
var aptTools = new AptTools();
string wethType = "0x1::aptos_coin::AptosCoin"; //пример типа токена

decimal tokenBalance = await aptTools.GetAptTokenBalance(wethType, "", "0xabc123...", "", true);
project.SendInfoToLog($"Баланс токена: {tokenBalance}", false);

[/CODE]

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

[CODE=csharp]
var suiTools = new SuiTools();
decimal suiBalance = await suiTools.GetSuiBalance("https://fullnode.mainnet.sui.io", "0x123abc...");
project.SendInfoToLog($"Баланс SUI: {suiBalance}", false);

[/CODE]

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

[CODE=csharp]
var suiTools = new SuiTools();
string usdcType = "0x5d4b302506645c37ff133b98c4b50a5ae14841659738d6d733d59d0d217a93bf::coin::COIN";
decimal usdcBalance = await suiTools.GetSuiTokenBalance(usdcType, "", "0xabc123...", "", true);
project.SendInfoToLog($"Баланс USDC на Sui: {usdcBalance}", false);

[/CODE]

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

[CODE=csharp]
var coinGecko = new CoinGecco();
string ethInfo = await coinGecko.CoinInfo("ethereum");
project.SendInfoToLog($"Информация об Ethereum: {ethInfo}", false);

[/CODE]

**TokenByAddress(string CGid = "ethereum")**

Возвращает: string - JSON с информацией о токене по адресу  

Описание: Получает информацию о токене по его контрактному адресу в блокчейне.


Параметры:

- CGid - идентификатор сети для поиска токена


Пример:

[CODE=csharp]
var coinGecko = new CoinGecco();
string tokenInfo = await coinGecko.TokenByAddress("ethereum");
//обработать полученную информацию

[/CODE]

**PriceByTiker(string tiker)**

Возвращает: decimal - цена в USD  

Описание: Статический метод для быстрого получения цены криптовалюты по ее тикеру. Поддерживает ETH, BNB, SOL.


Параметры:

- tiker - тикер криптовалюты (ETH, BNB, SOL)


Пример:

[CODE=csharp]
decimal ethPrice = CoinGecco.PriceByTiker("ETH");
decimal bnbPrice = CoinGecco.PriceByTiker("BNB");
project.SendInfoToLog($"ETH: ${ethPrice}, BNB: ${bnbPrice}", false);
[/CODE]

**PriceById(string CGid = "ethereum")**

Возвращает: decimal - цена в USD  

Описание: Статический метод для получения цены криптовалюты по ее ID в CoinGecko.


Параметры:

- CGid - идентификатор монеты в CoinGecko


Пример:

[CODE=csharp]
decimal solPrice = CoinGecko.PriceById("solana");
decimal dotPrice = CoinGecko.PriceById("polkadot");
project.SendInfoToLog($"SOL: ${solPrice}, DOT: ${dotPrice}", false);
[/CODE]

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

[CODE=csharp]
var dexScreener = new DexScreener();
string tokenContract = "0x6B175474E89094C44Da98b954EedeAC495271d0F"; //DAI
string tokenInfo = await dexScreener.CoinInfo(tokenContract, "ethereum");
project.SendInfoToLog($"Данные о токене: {tokenInfo}", false);

[/CODE]

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

[CODE=csharp]
var kuCoin = new KuCoin();
string btcOrderbook = await kuCoin.OrderbookByTiker("BTC");
project.SendInfoToLog($"Ордербук BTC: {btcOrderbook}", false);

[/CODE]

**KuPrice(string tiker = "ETH")**

Возвращает: decimal - цена в USDT  

Описание: Статический метод для быстрого получения текущей цены торговой пары на KuCoin.


Параметры:

- tiker - тикер криптовалюты


Пример:

[CODE=csharp]
decimal adaPrice = KuCoin.KuPrice("ADA");
decimal linkPrice = KuCoin.KuPrice("LINK");
project.SendInfoToLog($"ADA: ${adaPrice}, LINK: ${linkPrice}", false);
[/CODE]

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

[CODE=csharp]
decimal ethBalance = W3bTools.EvmNative("https://eth.llamarpc.com", "0x123abc...");
decimal bnbBalance = W3bTools.EvmNative("https://bsc-dataseed.binance.org", "0x123abc...");
project.SendInfoToLog($"ETH: {ethBalance}, BNB: {bnbBalance}", false);
[/CODE]

**ERC20(string tokenContract, string rpc, string address, string tokenDecimal = "18")**

Возвращает: decimal - баланс токена  

Описание: Упрощенный метод для получения баланса ERC-20 токена с автоматическим преобразованием в читаемый формат.


Параметры:

- tokenContract - адрес контракта токена

- rpc - URL RPC-узла

- address - адрес кошелька

- tokenDecimal - количество десятичных знаков токена (по умолчанию 18)


Пример:

[CODE=csharp]
string usdtContract = "0xdAC17F958D2ee523a2206206994597C13D831ec7";
decimal usdtBalance = W3bTools.ERC20(usdtContract, "https://eth.llamarpc.com", "0x123abc...", "6");
project.SendInfoToLog($"Баланс USDT: {usdtBalance}", false);

[/CODE]

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

[CODE=csharp]
bool success = W3bTools.WaitTx("https://polygon-rpc.com", "0xabc123...", 120, "", true, true);
//проверить результат

[/CODE]

**SolNative(string address, string rpc = "https://api.mainnet-beta.solana.com")**

Возвращает: decimal - баланс SOL  

Описание: Упрощенный метод для получения баланса SOL.


Параметры:

- address - адрес кошелька Solana

- rpc - URL RPC-узла (по умолчанию mainnet)


Пример:

[CODE=csharp]
decimal solBalance = W3bTools.SolNative("DYw8jCTfwHNRJhhmFcbXvVDTqWMEVFBX6ZKUmG5CNSKC");
project.SendInfoToLog($"Баланс SOL: {solBalance}", false);

[/CODE]

**CGPrice(string CGid = "ethereum")**

Возвращает: decimal - цена в USD  

Описание: Упрощенный метод для получения цены криптовалюты через CoinGecko API.


Параметры:

- CGid - идентификатор монеты в CoinGecko


Пример:

[CODE=csharp]
decimal btcPrice = W3bTools.CGPrice("bitcoin");
decimal ethPrice = W3bTools.CGPrice("ethereum");
project.SendInfoToLog($"BTC: ${btcPrice}, ETH: ${ethPrice}", false);
[/CODE]

**UsdToToken(decimal usdAmount, string tiker, string apiProvider = "KuCoin")**

Возвращает: decimal - количество токенов  

Описание: Конвертирует сумму в USD в количество токенов по текущему курсу через выбранный API.


Параметры:

- usdAmount - сумма в долларах для конвертации

- tiker - тикер криптовалюты

- apiProvider - источник цен (KuCoin, OKX, CoinGecco)


Пример:

[CODE=csharp]
//конвертировать $100 в ETH по курсу KuCoin

decimal ethAmount = project.UsdToToken(100m, "ETH", "KuCoin");
project.SendInfoToLog($"$100 = {ethAmount} ETH", false);
[/CODE][/SPOILER]
[/SPOILER]
