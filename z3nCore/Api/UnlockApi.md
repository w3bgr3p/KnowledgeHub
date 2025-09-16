# UnlockApi

Класс для работы с Unlock Protocol смарт-контрактами в блокчейне Optimism. Позволяет получать информацию о владельцах NFT-ключей и времени истечения их действия.

## Конструкторы

### `UnlockApi(IZennoPosterProjectModel project, bool log = false)`

Создает новый экземпляр UnlockApi для работы с Unlock Protocol.

**Параметры:**
- `project` - ссылка на проект ZennoPoster для логирования
- `log` - включить дополнительное логирование (по умолчанию false)

**Пример:**
```csharp
//создать экземпляр UnlockApi
var unlockApi = new UnlockApi(project, true);
```

## Методы

### `string keyExpirationTimestampFor(string addressTo, int tokenId, bool decode = true)`

**Возвращает:** string - временную метку истечения ключа

Получает временную метку истечения действия NFT-ключа по его ID.

**Параметры:**
- `addressTo` - адрес смарт-контракта Unlock
- `tokenId` - ID токена (ключа)
- `decode` - декодировать результат в читаемый формат (по умолчанию true)

**Пример:**
```csharp
//получить время истечения ключа с ID 1
string expiration = unlockApi.keyExpirationTimestampFor("0x1234567890abcdef", 1);
project.SendInfoToLog($"Ключ истекает: {expiration}");
```

### `string ownerOf(string addressTo, int tokenId, bool decode = true)`

**Возвращает:** string - адрес владельца ключа

Получает адрес владельца NFT-ключа по его ID.

**Параметры:**
- `addressTo` - адрес смарт-контракта Unlock
- `tokenId` - ID токена (ключа)
- `decode` - декодировать результат в читаемый формат (по умолчанию true)

**Пример:**
```csharp
//получить владельца ключа с ID 1
string owner = unlockApi.ownerOf("0x1234567890abcdef", 1);
project.SendInfoToLog($"Владелец ключа: {owner}");
```

### `string Decode(string toDecode, string function)`

**Возвращает:** string - декодированное значение

Декодирует результат вызова функции смарт-контракта из HEX формата в читаемый вид.

**Параметры:**
- `toDecode` - HEX строка для декодирования
- `function` - имя функции для определения типа декодирования

**Пример:**
```csharp
//декодировать HEX результат
string result = unlockApi.Decode("0x000000000000000000000000a1b2c3d4e5f6", "ownerOf");
project.SendInfoToLog($"Декодированный результат: {result}");
```

### `Dictionary<string, string> Holders(string contract)`

**Возвращает:** Dictionary<string, string> - словарь где ключ это адрес владельца, значение - время истечения

Получает всех держателей NFT-ключей контракта и время истечения их ключей.

**Параметры:**
- `contract` - адрес смарт-контракта Unlock

**Пример:**
```csharp
//получить всех держателей ключей
var holders = unlockApi.Holders("0x1234567890abcdef");
foreach(var holder in holders)
{
    //вывести информацию о каждом держателе
    project.SendInfoToLog($"Владелец: {holder.Key}, истекает: {holder.Value}");
}
```