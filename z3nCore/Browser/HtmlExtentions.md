# HtmlExtensions

Статический класс для работы с HTML элементами. Предоставляет методы для декодирования QR-кодов и извлечения транзакционных хешей из ссылок.

## Методы

### DecodeQr
**Возвращает:** string  
**Описание:** Декодирует QR-код из HTML элемента в текстовую строку.

**Параметры:**
- `element` (HtmlElement) - HTML элемент, содержащий QR-код

**Возвращаемые значения:**
- Текст QR-кода при успешном декодировании
- "qrIsNull" если QR-код не найден или пустой  
- "qrError" в случае ошибки при декодировании

**Пример:**
```csharp
// найти элемент с QR-кодом
HtmlElement qrElement = tab.FindElementByTag("img", 0);

// декодировать QR-код
string qrText = HtmlExtensions.DecodeQr(qrElement);

if (qrText == "qrIsNull")
{
    //QR-код не найден
    project.SendWarningToLog("QR-код не обнаружен");
}
else if (qrText == "qrError") 
{
    //ошибка декодирования
    project.SendErrorToLog("Ошибка при декодировании QR-кода");
}
else
{
    //успешное декодирование  
    project.SendInfoToLog($"QR-код: {qrText}");
}
```

### GetTxHash
**Возвращает:** string  
**Описание:** Извлекает транзакционный хеш из href атрибута ссылки. Возвращает последнюю часть URL после символа '/'.

**Параметры:**
- `element` (HtmlElement) - HTML элемент ссылки с атрибутом href

**Исключения:**
- Exception - если элемент не содержит атрибута href или произошла другая ошибка

**Пример:**
```csharp
// найти ссылку на транзакцию
HtmlElement linkElement = tab.FindElementByAttribute("a", "href", "*tx*", "text", 0);

try 
{
    // извлечь хеш транзакции
    string txHash = HtmlExtensions.GetTxHash(linkElement);
    
    //использовать полученный хеш
    project.SendInfoToLog($"Хеш транзакции: {txHash}");
}
catch (Exception ex)
{
    //обработать ошибку
    project.SendErrorToLog($"Ошибка получения хеша: {ex.Message}");
}
```