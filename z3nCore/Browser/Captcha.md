# Captcha

Статический класс для работы с различными видами captcha, включая Cloudflare Turnstile и интеграцию с сервисом Cap.Guru.

## Методы

### CFSolve

```csharp
public static void CFSolve(this Instance instance)
```

**Возвращаемое значение:** `void`

**Описание:** Автоматически решает Cloudflare Turnstile captcha путем клика по защитному элементу на странице.

**Пример использования:**
```csharp
//решить Cloudflare captcha
instance.CFSolve();
```

---

### CFToken

```csharp
public static string CFToken(this Instance instance, int deadline = 60, bool strict = false)
```

**Возвращаемое значение:** `string` - токен от решённой Cloudflare captcha

**Описание:** Получает токен от решённой Cloudflare Turnstile captcha с возможностью настройки времени ожидания.

**Параметры:**
- `deadline` (`int`) - максимальное время ожидания решения в секундах (по умолчанию 60)
- `strict` (`bool`) - строгий режим проверки (по умолчанию false)

**Пример использования:**
```csharp
//получить токен с ожиданием 120 секунд
string token = instance.CFToken(120);
project.SendInfoToLog($"Получен токен: {token}");
```

---

### CapGuru

```csharp
public static bool CapGuru(this IZennoPosterProjectModel project)
```

**Возвращаемое значение:** `bool` - результат выполнения плагина Cap.Guru (true - успешно, false - ошибка)

**Описание:** Запускает плагин Cap.Guru для решения различных типов captcha. Автоматически использует API ключ из базы данных проекта.

**Пример использования:**
```csharp
//использовать Cap.Guru для решения captcha
bool result = project.CapGuru();
if (result)
{
    project.SendInfoToLog("Cap.Guru успешно решил captcha");
}
else
{
    project.SendErrorToLog("Ошибка при решении captcha через Cap.Guru");
}
```