[SPOILER= Wallets] 

[SPOILER= BackpackWallet]

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

[CODE=csharp]
var wallet = new BackpackWallet(project, instance, log: true, key: "key");
//создать экземпляр с логированием и ключом из базы данных

[/CODE]

## Методы


### Launch(string fileName = null, bool log = false) : string

Запускает кошелек Backpack, устанавливает расширение и импортирует/разблокирует кошелек.


**Возвращает:** Адрес активного кошелька


**Параметры:**

- `fileName` - имя файла расширения (необязательно)

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
string address = wallet.Launch(log: true);
project.SendInfoToLog($"Адрес кошелька: {address}");

//запустить кошелек и получить адрес

[/CODE]

### ActiveAddress(bool log = false) : string

Получает адрес текущего активного кошелька.


**Возвращает:** Адрес активного кошелька


**Параметры:**

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
string address = wallet.ActiveAddress();
project.SendInfoToLog($"Текущий адрес: {address}");

//получить адрес активного кошелька

[/CODE]

### CurrentChain(bool log = true) : string

Определяет текущую блокчейн-сеть кошелька.


**Возвращает:** Название сети ("mainnet", "devnet", "testnet", "ethereum")


**Параметры:**

- `log` - включить логирование (по умолчанию true)


**Пример:**

[CODE=csharp]
string chain = wallet.CurrentChain();
//определить текущую сеть

if (chain == "mainnet") 
{
}
[/CODE]

### Unlock(bool log = false) : void

Разблокирует кошелек с помощью пароля.


**Параметры:**

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
wallet.Unlock(log: true);
//разблокировать кошелек

[/CODE]

### Approve(bool log = false) : void

Подтверждает действие в кошельке (транзакцию, подключение и т.д.).


**Параметры:**

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
wallet.Approve();
//подтвердить действие в кошельке

[/CODE]

### Connect(bool log = false) : void

Подключает кошелек к веб-сайту, обрабатывает запросы на подключение.


**Параметры:**

- `log` - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
wallet.Connect();
//подключить кошелек к сайту

[/CODE]

### Devmode(bool enable = true) : void

Включает или выключает режим разработчика в кошельке.


**Параметры:**

- `enable` - включить режим разработчика (по умолчанию true)


**Пример:**

[CODE=csharp]
wallet.Devmode(true);
//включить режим разработчика

[/CODE]

### DevChain(string reqmode = "devnet") : void

Переключает кошелек на тестовую сеть (devnet, testnet).


**Параметры:**

- `reqmode` - требуемый режим сети (по умолчанию "devnet")


**Пример:**

[CODE=csharp]
wallet.DevChain("testnet");
//переключиться на тестовую сеть

[/CODE]

### Add(string type = "Ethereum", string source = "key") : void

Добавляет новый кошелек указанного типа.


**Параметры:**

- `type` - тип блокчейна ("Ethereum" или "Solana")

- `source` - источник импорта ("key" или "phrase")


**Пример:**

[CODE=csharp]
wallet.Add("Solana", "key");
//добавить Solana кошелек из приватного ключа

[/CODE]

### Switch(string type) : void

Переключается между кошельками разных типов.


**Параметры:**

- `type` - тип кошелька для переключения ("Solana" или "Ethereum")


**Пример:**

[CODE=csharp]
wallet.Switch("Ethereum");
//переключиться на Ethereum кошелек

[/CODE]

### Current() : string

Определяет тип текущего активного кошелька.


**Возвращает:** Тип кошелька ("Solana", "Ethereum" или "Undefined")


**Пример:**

[CODE=csharp]
string currentType = wallet.Current();
//определить тип текущего кошелька

project.SendInfoToLog($"Текущий тип: {currentType}");

[/CODE][/SPOILER]
[SPOILER= KeplrWallet]

# KeplrWallet

Класс **KeplrWallet** предоставляет инструменты для автоматизации работы с криптокошельком Keplr через браузерное расширение. Позволяет устанавливать расширение, импортировать кошельки, управлять ими и подписывать транзакции.


## Конструкторы


### KeplrWallet(IZennoPosterProjectModel, Instance, bool, string, string)
Инициализирует новый экземпляр класса KeplrWallet для работы с кошельком Keplr.


**Параметры:**

- **project** (IZennoPosterProjectModel) - проект ZennoPoster

- **instance** (Instance) - экземпляр браузера  

- **log** (bool) - включить логирование (по умолчанию false)

- **key** (string) - приватный ключ (опционально)

- **seed** (string) - мнемоническая фраза (опционально)


**Пример:**

[CODE=csharp]
// инициализация KeplrWallet с логированием

var keplrWallet = new KeplrWallet(project, instance, log: true);

//отправить информацию в лог

[/CODE]

## Публичные методы


### Launch(string, string, bool)
**Возвращаемое значение:** void


Запускает расширение Keplr, устанавливает его при необходимости и импортирует кошелек из указанного источника.


**Параметры:**

- **source** (string) - источник для импорта ("seed" или "keyEvm", по умолчанию "seed")

- **fileName** (string) - имя файла расширения (опционально, по умолчанию используется встроенное значение)

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
// запуск Keplr с импортом из seed-фразы

keplrWallet.Launch("seed", log: true);

//отправить информацию в лог

[/CODE]

### SetSource(string, bool)
**Возвращаемое значение:** void


Устанавливает активный источник кошелька в Keplr. Автоматически управляет импортированными кошельками и переключается на нужный.


**Параметры:**

- **source** (string) - источник кошелька ("seed" или "keyEvm")

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
// переключение на кошелек из приватного ключа

keplrWallet.SetSource("keyEvm", log: true);

//отправить информацию в лог

[/CODE]

### Unlock(bool)
**Возвращаемое значение:** void


Разблокирует кошелек Keplr, вводя пароль. Автоматически определяет, нужна ли разблокировка.


**Параметры:**

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
// разблокировка кошелька

keplrWallet.Unlock(log: true);

//отправить информацию в лог

[/CODE]

### Sign(bool)
**Возвращаемое значение:** void


Подписывает транзакцию в Keplr, ожидая появления окна подтверждения и автоматически нажимая кнопку "Approve".


**Параметры:**

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
// подписание транзакции

keplrWallet.Sign(log: true);

//отправить информацию в лог

[/CODE]

### KeplrApprove(bool)
**Возвращаемое значение:** string


Устаревший метод для подписания транзакций. Рекомендуется использовать метод Sign().


**Параметры:**

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
// использование устаревшего метода (не рекомендуется)

string result = keplrWallet.KeplrApprove(log: true);

//отправить предупреждение в лог

project.SendWarningToLog("Используется устаревший метод KeplrApprove");

[/CODE][/SPOILER]
[SPOILER= ZerionWallet]

# ZerionWallet

Класс для автоматизации кошелька Zerion в браузерных скриптах. Позволяет устанавливать расширение, импортировать кошельки по приватному ключу или мнемонике, подписывать транзакции и проверять доступные для получения награды.


## Конструктор


### ZerionWallet(project, instance, log, key, fileName)

Создает новый экземпляр класса для работы с кошельком Zerion.


**Параметры:**

- **project** (IZennoPosterProjectModel) - текущий проект

- **instance** (Instance) - экземпляр браузера

- **log** (bool) - включить логирование (по умолчанию false)

- **key** (string) - ключ кошелька ("key", "seed" или прямое значение, по умолчанию "key")  

- **fileName** (string) - имя файла расширения (по умолчанию "Zerion1.21.3.crx")


**Пример:**

[CODE=csharp]
var wallet = new ZerionWallet(project, instance, log: true);
[/CODE]

## Методы


### Launch(fileName, log, source, refCode)

**Возвращает:** string - адрес активного кошелька


Устанавливает расширение Zerion, импортирует кошелек и подготавливает его к использованию. Автоматически определяет состояние кошелька и выполняет необходимые действия для полной настройки.


**Параметры:**

- **fileName** (string) - имя файла расширения (необязательно)

- **log** (bool) - включить логирование (по умолчанию false)

- **source** (string) - источник ключа ("key" или "seed", по умолчанию "key")

- **refCode** (string) - реферальный код (необязательно)


**Пример:**

[CODE=csharp]
//запустить кошелек с основным ключом

string address = wallet.Launch();
project.SendInfoToLog($"Кошелек запущен: {address}");


//запустить с реферальным кодом

string address = wallet.Launch(refCode: "ABC123");
project.SendInfoToLog($"Кошелек запущен с рефералом: {address}");

[/CODE]

### Sign(log, deadline)

**Возвращает:** bool - true если транзакция успешно подписана


Подписывает транзакцию или сообщение в кошельке Zerion. Автоматически находит кнопки "Confirm" или "Sign" и нажимает их.


**Параметры:**

- **log** (bool) - включить логирование (по умолчанию false)

- **deadline** (int) - максимальное время ожидания в секундах (по умолчанию 10)


**Пример:**

[CODE=csharp]
//подписать транзакцию

bool success = wallet.Sign();
if (success)
{
}

//подписать с увеличенным временем ожидания

bool success = wallet.Sign(deadline: 30);
[/CODE]

### Connect(log)

Подключает кошелек к веб-сайту. Автоматически обрабатывает процесс подключения, включая добавление сайта в список доверенных и подтверждение подключения.


**Параметры:**

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
//подключить кошелек к сайту

wallet.Connect();
[/CODE]

### SwitchSource(addressToUse)

Переключает активный адрес кошелька на указанный источник.


**Параметры:**

- **addressToUse** (string) - источник адреса ("key" для основного ключа, "seed" для мнемоники, по умолчанию "key")


**Пример:**

[CODE=csharp]
//переключиться на основной ключ

wallet.SwitchSource("key");

//переключиться на адрес из мнемоники

wallet.SwitchSource("seed");
[/CODE]

### WaitTx(deadline, log)

**Возвращает:** bool - true если транзакция успешна, false если неудачна


Ожидает завершения транзакции в истории кошелька. Проверяет статус последней транзакции до её завершения.


**Параметры:**

- **deadline** (int) - максимальное время ожидания в секундах (по умолчанию 60)

- **log** (bool) - включить логирование (по умолчанию false)


**Пример:**

[CODE=csharp]
//дождаться завершения транзакции

bool success = wallet.WaitTx();
if (success)
{
}
else
{
project.SendErrorToLog("Транзакция не удалась");

}
[/CODE]

### Claimable(address)

**Возвращает:** List<string> - список ID доступных для получения наград


Получает список квестов и наград, доступных для получения по указанному адресу через API Zerion.


**Параметры:**

- **address** (string) - адрес кошелька для проверки


**Пример:**

[CODE=csharp]
//получить список доступных наград

var rewards = wallet.Claimable("0x742d35Cc6634C0532925a3b8D46d07c30e5b7A0c");
foreach (string rewardId in rewards)
{
project.SendInfoToLog($"Доступна награда: {rewardId}");

}
[/CODE]

### ActiveAddress()

**Возвращает:** string - адрес активного кошелька


Получает адрес текущего активного кошелька в расширении Zerion.


**Пример:**

[CODE=csharp]
//получить адрес активного кошелька

string currentAddress = wallet.ActiveAddress();
project.SendInfoToLog($"Текущий адрес: {currentAddress}");

[/CODE]

## Статические методы


### TxFromUrl(url)

**Возвращает:** string - JSON с данными транзакции


Извлекает данные транзакции из URL расширения Zerion.


**Параметры:**

- **url** (string) - URL с данными транзакции


**Пример:**

[CODE=csharp]
//извлечь данные транзакции из URL

string txData = ZerionWallet.TxFromUrl(instance.ActiveTab.URL);
project.SendInfoToLog($"Данные транзакции: {txData}");

[/CODE][/SPOILER]
[/SPOILER]
