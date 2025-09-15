# AES

Статический класс для работы с AES-шифрованием и MD5-хешированием. Предоставляет методы для шифрования и расшифровки текста с использованием алгоритма AES, а также создания MD5-хешей.

## Методы

### EncryptAES
**Возвращает:** `string` - зашифрованная строка в HEX-формате

Шифрует текст с использованием алгоритма AES. По умолчанию ключ автоматически хешируется через MD5 для обеспечения правильной длины.

**Параметры:**
- `phrase` (string) - текст для шифрования
- `key` (string) - ключ для шифрования  
- `hashKey` (bool) - если true, ключ будет хеширован через MD5 (по умолчанию true)

**Пример использования:**
```csharp
string text = "Мой секретный текст";
string password = "mypassword123";

//шифруем текст с автоматическим хешированием ключа
string encrypted = AES.EncryptAES(text, password);
project.SendInfoToLog($"Зашифрованный текст: {encrypted}");

//шифруем с готовым 32-символьным ключом без хеширования
string hexKey = "1234567890abcdef1234567890abcdef";
string encrypted2 = AES.EncryptAES(text, hexKey, false);
```

### DecryptAES
**Возвращает:** `string` - расшифрованный текст

Расшифровывает текст, зашифрованный методом EncryptAES. Важно использовать тот же ключ и настройку hashKey, что и при шифровании.

**Параметры:**
- `hash` (string) - зашифрованная строка в HEX-формате
- `key` (string) - ключ для расшифровки
- `hashKey` (bool) - если true, ключ будет хеширован через MD5 (по умолчанию true)

**Пример использования:**
```csharp
string encryptedText = "A1B2C3D4..."; //зашифрованный текст
string password = "mypassword123";

//расшифровываем с тем же ключом
string decrypted = AES.DecryptAES(encryptedText, password);
project.SendInfoToLog($"Расшифрованный текст: {decrypted}");

//если шифровали без хеширования ключа, то и расшифровывать нужно так же
string hexKey = "1234567890abcdef1234567890abcdef";
string decrypted2 = AES.DecryptAES(encryptedText, hexKey, false);
```

### HashMD5
**Возвращает:** `string` - MD5-хеш в HEX-формате

Создает MD5-хеш от переданной строки. Полезно для создания фиксированных ключей для AES-шифрования или проверки целостности данных.

**Параметры:**
- `phrase` (string) - строка для хеширования

**Пример использования:**
```csharp
string password = "mypassword123";

//получаем MD5-хеш пароля
string hash = AES.HashMD5(password);
project.SendInfoToLog($"MD5 хеш: {hash}");

//можно использовать как ключ для AES без дополнительного хеширования
string text = "секретные данные";
string encrypted = AES.EncryptAES(text, hash, false);
```

---

# Bech32

Статический класс для работы с адресами в формате Bech32. Обеспечивает конвертацию между HEX-адресами и Bech32-адресами с префиксом "init", что используется в некоторых блокчейн-системах.

## Методы

### Bech32ToHex
**Возвращает:** `string` - HEX-адрес с префиксом "0x"

Конвертирует Bech32-адрес в HEX-формат. Проверяет корректность формата, префикса "init" и контрольной суммы.

**Параметры:**
- `bech32Address` (string) - адрес в формате Bech32 с префиксом "init"

**Пример использования:**
```csharp
string bech32Addr = "init1qw508d6qejxtdg4y5r3zarvary0c5xw7k8a9xz2e";

try
{
    //конвертируем Bech32 в HEX
    string hexAddr = Bech32.Bech32ToHex(bech32Addr);
    project.SendInfoToLog($"HEX адрес: {hexAddr}");
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка конвертации: {ex.Message}");
}
```

### HexToBech32
**Возвращает:** `string` - адрес в формате Bech32

Конвертирует HEX-адрес в формат Bech32 с указанным префиксом. По умолчанию используется префикс "init".

**Параметры:**
- `hexAddress` (string) - HEX-адрес (с префиксом "0x" или без)
- `prefix` (string) - префикс для Bech32-адреса (по умолчанию "init")

**Пример использования:**
```csharp
string hexAddr = "0x1234567890abcdef1234567890abcdef12345678";

try
{
    //конвертируем HEX в Bech32 с префиксом по умолчанию
    string bech32Addr = Bech32.HexToBech32(hexAddr);
    project.SendInfoToLog($"Bech32 адрес: {bech32Addr}");
    
    //можно указать свой префикс
    string customAddr = Bech32.HexToBech32(hexAddr, "custom");
    project.SendInfoToLog($"С префиксом custom: {customAddr}");
}
catch (Exception ex)
{
    project.SendErrorToLog($"Ошибка конвертации: {ex.Message}");
}
```