# Класс Converer

## Описание
Статический класс для преобразования данных между различными форматами: hex (шестнадцатеричные строки), base64, bech32 (адреса блокчейна), bytes и обычный текст. Поддерживает двустороннее преобразование между любыми из этих форматов, что полезно при работе с криптографическими данными, адресами кошельков и кодированием информации.

## Публичные методы

**ConvertFormat(IZennoPosterProjectModel project, string toProcess, string input, string output, bool log = false)**

**Возвращает:** string - преобразованная строка или null в случае ошибки

**Описание:** Преобразует строку из одного формата в другой. Поддерживает форматы: hex, base64, bech32, bytes, text.

**Параметры:**
- project - модель проекта для логирования
- toProcess - строка для преобразования
- input - исходный формат (hex/base64/bech32/bytes/text)
- output - целевой формат (hex/base64/bech32/bytes/text)
- log - включить логирование операции (по умолчанию false)

**Пример:**
```csharp
// преобразование hex в base64
string result = Converer.ConvertFormat(project, "48656c6c6f", "hex", "base64", true);
project.SendInfoToLog($"Результат: {result}");

// преобразование текста в hex
string hexResult = Converer.ConvertFormat(project, "Hello World", "text", "hex");
project.SendInfoToLog($"Текст в hex: {hexResult}");

// преобразование bech32 адреса в hex
string bech32ToHex = Converer.ConvertFormat(project, "init1qqqsyqcyq5rqwzqfpg9scrgwpugpzysnpswf", "bech32", "hex");
//результат преобразования
if (bech32ToHex != null)
{
    project.SendInfoToLog($"Bech32 в hex: {bech32ToHex}");
}
else
{
    project.SendErrorToLog("Ошибка преобразования bech32");
}
```