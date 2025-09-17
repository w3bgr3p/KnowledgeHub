# StringExtensions - Методы расширения для работы со строками

Статический класс с методами расширения для строк, предоставляющий удобные функции для работы с криптографией, блокчейн-данными, парсингом и другими распространенными операциями.

---

## Публичные методы

### EscapeMarkdown

**Возвращаемое значение:** `string` - строка с экранированными символами разметки Markdown

Экранирует специальные символы Markdown в строке, добавляя перед ними обратный слэш.

**Параметры:**
- `text` (string) - исходная строка для экранирования

**Пример использования:**
```csharp
string text = "Текст с *жирным* и _курсивом_";
string escaped = text.EscapeMarkdown();
project.SendInfoToLog(escaped); // Текст с \*жирным\* и \_курсивом\_
```

---

### GetTxHash

**Возвращаемое значение:** `string` - хеш транзакции из ссылки

Извлекает хеш транзакции из URL-ссылки, беря последнюю часть после слэша.

**Параметры:**
- `link` (string) - ссылка на транзакцию

**Пример использования:**
```csharp
string url = "https://etherscan.io/tx/0x1234567890abcdef";
string hash = url.GetTxHash();
project.SendInfoToLog(hash); // 0x1234567890abcdef
```

---

### ToPubEvm

**Возвращаемое значение:** `string` - публичный адрес Ethereum

Преобразует приватный ключ или seed-фразу в публичный адрес Ethereum.

**Параметры:**
- `key` (string) - приватный ключ в hex формате или seed-фраза

**Пример использования:**
```csharp
string privateKey = "1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef";
string address = privateKey.ToPubEvm();
project.SendInfoToLog($"Адрес: {address}");
```

---

### ToSecp256k1

**Возвращаемое значение:** `string` - приватный ключ в hex формате

Генерирует приватный ключ из seed-фразы по указанному пути деривации.

**Параметры:**
- `seed` (string) - seed-фраза для генерации ключа
- `path` (int, по умолчанию: 0) - индекс пути деривации

**Пример использования:**
```csharp
string seedPhrase = "word1 word2 word3 word4 word5 word6 word7 word8 word9 word10 word11 word12";
string privateKey = seedPhrase.ToSecp256k1(0);
project.SendInfoToLog($"Приватный ключ: {privateKey}");
```

---

### ToEvmPrivateKey

**Возвращаемое значение:** `string` - приватный ключ в hex формате

Создает приватный ключ из произвольной строки используя SHA256 хеширование.

**Параметры:**
- `input` (string) - исходная строка для генерации ключа

**Пример использования:**
```csharp
string password = "мойпароль123";
string privateKey = password.ToEvmPrivateKey();
project.SendInfoToLog($"Сгенерированный ключ: {privateKey}");
```

---

### TxToString

**Возвращаемое значение:** `string[]` - массив строк с данными транзакции

Парсит JSON данные транзакции и возвращает основные параметры в удобном формате.

**Параметры:**
- `txJson` (string) - JSON строка с данными транзакции

**Пример использования:**
```csharp
string txJson = "{\"gas\":\"0x5208\",\"value\":\"0x0\",\"from\":\"0x123...\",\"to\":\"0x456...\",\"data\":\"0x\"}";
string[] txData = txJson.TxToString();
//txData[0] - gas, txData[1] - value, txData[2] - sender, txData[3] - data, txData[4] - recipient, txData[5] - gas в gwei
project.SendInfoToLog($"Gas: {txData[0]}, От: {txData[2]}");
```

---

### ChkAddress

**Возвращаемое значение:** `bool` - результат проверки соответствия

Проверяет соответствие сокращенного адреса (с символом …) полному адресу.

**Параметры:**
- `shortAddress` (string) - сокращенный адрес вида "0x1234…5678"
- `fullAddress` (string) - полный адрес для проверки

**Пример использования:**
```csharp
string shortAddr = "0x1234…5678";
string fullAddr = "0x1234567890abcdef1234567890abcdef12345678";
bool isMatch = shortAddr.ChkAddress(fullAddr);
project.SendInfoToLog($"Адреса совпадают: {isMatch}");
```

---

### ParseCreds

**Возвращаемое значение:** `Dictionary<string, string>` - словарь с разобранными данными

Разбирает строку с данными по заданному формату с разделителями.

**Параметры:**
- `data` (string) - строка данных для разбора
- `format` (string) - формат с названиями полей в фигурных скобках
- `devider` (char, по умолчанию: ':') - символ-разделитель

**Пример использования:**
```csharp
string data = "admin:password123:user@email.com";
string format = "{login}:{password}:{email}";
var parsed = data.ParseCreds(format, ':');
project.SendInfoToLog($"Логин: {parsed["login"]}, Email: {parsed["email"]}");
```

---

### ParseByMask

**Возвращаемое значение:** `Dictionary<string, string>` - словарь с извлеченными значениями

Извлекает значения из строки по заданной маске с переменными в фигурных скобках.

**Параметры:**
- `input` (string) - исходная строка для парсинга
- `mask` (string) - маска с переменными в фигурных скобках

**Пример использования:**
```csharp
string text = "Пользователь John имеет баланс 100.50";
string mask = "Пользователь {name} имеет баланс {balance}";
var result = text.ParseByMask(mask);
project.SendInfoToLog($"Имя: {result["name"]}, Баланс: {result["balance"]}");
```

---

### StringToHex

**Возвращаемое значение:** `string` - число в hex формате с префиксом 0x

Преобразует строковое число в hex формат с возможностью конвертации единиц измерения.

**Параметры:**
- `value` (string) - числовое значение в виде строки
- `convert` (string, по умолчанию: "") - тип конвертации: "gwei", "eth" или пустая строка

**Пример использования:**
```csharp
string amount = "1.5";
string hexEth = amount.StringToHex("eth");
string hexGwei = amount.StringToHex("gwei");
project.SendInfoToLog($"1.5 ETH = {hexEth}");
project.SendInfoToLog($"1.5 GWEI = {hexGwei}");
```

---

### HexToString

**Возвращаемое значение:** `string` - числовое значение в виде строки

Преобразует hex значение в строку с возможностью конвертации единиц измерения.

**Параметры:**
- `hexValue` (string) - hex значение (с префиксом 0x или без)
- `convert` (string, по умолчанию: "") - тип конвертации: "gwei", "eth" или пустая строка

**Пример использования:**
```csharp
string hexValue = "0x1C9C380";
string ethAmount = hexValue.HexToString("eth");
string gweiAmount = hexValue.HexToString("gwei");
project.SendInfoToLog($"Значение в ETH: {ethAmount}");
project.SendInfoToLog($"Значение в GWEI: {gweiAmount}");
```

---

### Range

**Возвращаемое значение:** `string[]` - массив строковых чисел

Создает массив чисел из диапазона или списка, заданного строкой.

**Параметры:**
- `accRange` (string) - диапазон в формате "1-10", список "1,2,3,5" или одно число "10"

**Пример использования:**
```csharp
string range1 = "1-5";
string[] numbers1 = range1.Range(); // ["1", "2", "3", "4", "5"]

string range2 = "1,3,5,7";
string[] numbers2 = range2.Range(); // ["1", "3", "5", "7"]

string range3 = "3";
string[] numbers3 = range3.Range(); // ["1", "2", "3"]

project.SendInfoToLog($"Диапазон: {string.Join(", ", numbers1)}");
```

---

### KeyType

**Возвращаемое значение:** `string` - тип ключа: "keyEvm", "keySol" или "seed"

Определяет тип ключа или seed-фразы по формату строки.

**Параметры:**
- `input` (string) - строка с ключом или seed-фразой

**Пример использования:**
```csharp
string evmKey = "1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef";
string seedPhrase = "word1 word2 word3 word4 word5 word6 word7 word8 word9 word10 word11 word12";
string type1 = evmKey.KeyType(); // "keyEvm"
string type2 = seedPhrase.KeyType(); // "seed"
project.SendInfoToLog($"Тип ключа: {type1}");
```

---

### GetLink

**Возвращаемое значение:** `string` - извлеченная ссылка

Извлекает URL-ссылку из текста.

**Параметры:**
- `text` (string) - текст, содержащий ссылку

**Пример использования:**
```csharp
string message = "Перейдите по ссылке https://example.com для подтверждения";
string link = message.GetLink();
project.SendInfoToLog($"Найденная ссылка: {link}");
```

---

### GetOTP

**Возвращаемое значение:** `string` - найденный OTP код

Извлекает 6-значный OTP код из текста.

**Параметры:**
- `text` (string) - текст, содержащий OTP код

**Пример использования:**
```csharp
string smsText = "Ваш код подтверждения: 123456. Никому не сообщайте его";
string otp = smsText.GetOTP();
project.SendInfoToLog($"OTP код: {otp}"); // 123456
```

---

### CleanFilePath

**Возвращаемое значение:** `string` - строка без недопустимых символов

Удаляет из строки символы, недопустимые для имен файлов.

**Параметры:**
- `text` (string) - исходная строка для очистки

**Пример использования:**
```csharp
string fileName = "Файл<с>недопустимыми:символами?.txt";
string cleanName = fileName.CleanFilePath();
project.SendInfoToLog($"Очищенное имя: {cleanName}"); // ФайлснедопустимымисимволамиТХТ
```

---

### ConvertUrl

**Возвращаемое значение:** `string` - отформатированные параметры URL

Извлекает и форматирует параметры из URL, с особой обработкой параметра addEthereumChainParameter.

**Параметры:**
- `url` (string) - URL для парсинга
- `oneline` (bool, по умолчанию: false) - выводить результат в одну строку

**Пример использования:**
```csharp
string url = "https://example.com?param1=value1&param2=value2";
string formatted = url.ConvertUrl();
string oneline = url.ConvertUrl(true);
project.SendInfoToLog($"Параметры:\n{formatted}");
project.SendInfoToLog($"В одну строку: {oneline}");
```

---

### NewPassword

**Возвращаемое значение:** `string` - сгенерированный пароль

Генерирует случайный пароль заданной длины, содержащий символы всех типов.

**Параметры:**
- `length` (int) - длина пароля (минимум 8 символов)

**Пример использования:**
```csharp
string password = StringExtensions.NewPassword(12);
project.SendInfoToLog($"Сгенерированный пароль: {password}");
```