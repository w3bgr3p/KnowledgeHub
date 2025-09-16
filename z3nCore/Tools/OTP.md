## OTP

Статический класс для получения одноразовых паролей (OTP) различными способами. Позволяет генерировать TOTP коды в автономном режиме и получать их из сервиса FirstMail.

---

### Методы

#### Offline

**Возвращаемое значение:** `string` - сгенерированный TOTP код

Генерирует одноразовый пароль TOTP в автономном режиме по секретному ключу. Если до истечения текущего кода осталось мало времени, метод дожидается генерации нового кода для повышения надежности.

**Параметры:**
- `keyString` *(string)* - секретный ключ в формате Base32
- `waitIfTimeLess` *(int, необязательный)* - если до истечения кода осталось меньше указанных секунд, метод дожидается нового кода (по умолчанию 5)

**Пример использования:**
```csharp
string secret = "JBSWY3DPEHPK3PXP";
string otpCode = OTP.Offline(secret);
project.SendInfoToLog($"Сгенерированный OTP код: {otpCode}");

//дождаться нового кода если осталось меньше 10 секунд
string otpCode2 = OTP.Offline(secret, 10);
project.SendInfoToLog($"Новый OTP код: {otpCode2}");
```

#### FirstMail

**Возвращаемое значение:** `string` - OTP код, полученный из сервиса FirstMail

Получает одноразовый пароль из сервиса FirstMail для указанного email адреса. Использует внутренний механизм для извлечения кода из почтовых сообщений.

**Параметры:**
- `project` *(IZennoPosterProjectModel)* - экземпляр проекта ZennoPoster
- `email` *(string)* - email адрес для получения OTP кода

**Пример использования:**
```csharp
string email = "test@example.com";
string otpCode = OTP.FirstMail(project, email);
project.SendInfoToLog($"OTP код из FirstMail: {otpCode}");
```