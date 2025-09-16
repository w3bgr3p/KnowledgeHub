## Класс Rnd

Статический класс для генерации различных случайных значений: строк, чисел, никнеймов и других данных, используемых в автоматизации.

---

## Методы

### Seed()

**Возвращаемое значение:** `string`  
**Описание:** Генерирует мнемоническую фразу из 12 слов на английском языке

```csharp
string mnemonic = Rnd.Seed();
project.SendInfoToLog($"Generated mnemonic: {mnemonic}");
```

---

### RndHexString(int length)

**Возвращаемое значение:** `string`  
**Описание:** Генерирует случайную hex-строку заданной длины с префиксом "0x"

**Параметры:**
- `length` (int) - длина генерируемой hex-строки (без учета префикса "0x")

```csharp
string hexString = Rnd.RndHexString(8);
project.SendInfoToLog($"Generated hex: {hexString}"); // Например: 0x1a2b3c4d
```

---

### RndString(int length)

**Возвращаемое значение:** `string`  
**Описание:** Генерирует случайную строку из букв и цифр заданной длины

**Параметры:**
- `length` (int) - длина генерируемой строки

```csharp
string randomString = Rnd.RndString(10);
project.SendInfoToLog($"Random string: {randomString}"); // Например: Kj8Xm2Qr9Z
```

---

### RndNickname()

**Возвращаемое значение:** `string`  
**Описание:** Генерирует случайный никнейм из прилагательного, существительного и суффикса (максимум 15 символов)

```csharp
string nickname = Rnd.RndNickname();
project.SendInfoToLog($"Generated nickname: {nickname}"); // Например: MysticWolfX
```

---

### RndInvite(this IZennoPosterProjectModel project, object limit = null, bool log = false)

**Возвращаемое значение:** `string`  
**Описание:** Получает случайный реферальный код из базы данных или использует существующий из переменной cfgRefCode

**Параметры:**
- `limit` (object, опционально) - максимальный ID записи для выборки
- `log` (bool) - включить логирование (по умолчанию false)

```csharp
string refCode = project.RndInvite(100, true);
project.SendInfoToLog($"Using ref code: {refCode}");
```

---

### RndPercent(decimal input, double percent, double maxPercent)

**Возвращаемое значение:** `double`  
**Описание:** Вычисляет процент от числа с случайным уменьшением до максимального процента

**Параметры:**
- `input` (decimal) - исходное число
- `percent` (double) - процент от числа (0-100)
- `maxPercent` (double) - максимальный процент случайного уменьшения (0-100)

```csharp
double result = Rnd.RndPercent(1000m, 50, 10);
project.SendInfoToLog($"Result: {result}"); // 50% от 1000 с уменьшением до 10%
```

---

### RndDecimal(this IZennoPosterProjectModel project, string Var)

**Возвращаемое значение:** `decimal`  
**Описание:** Получает случайное decimal число из переменной проекта (поддерживает диапазон через дефис)

**Параметры:**
- `Var` (string) - имя переменной проекта

```csharp
// Переменная содержит "10.5-20.7"
decimal randomDecimal = project.RndDecimal("myRange");
project.SendInfoToLog($"Random decimal: {randomDecimal}");
```

---

### RndInt(this IZennoPosterProjectModel project, string Var)

**Возвращаемое значение:** `int`  
**Описание:** Получает случайное целое число из переменной проекта (поддерживает диапазон через дефис)

**Параметры:**
- `Var` (string) - имя переменной проекта

```csharp
// Переменная содержит "5-15"
int randomInt = project.RndInt("myIntRange");
project.SendInfoToLog($"Random integer: {randomInt}");
```

---

### RndBool(this int truePercent)

**Возвращаемое значение:** `bool`  
**Описание:** Возвращает случайное булево значение с заданной вероятностью true в процентах

**Параметры:**
- `truePercent` (int) - вероятность получить true в процентах (0-100)

```csharp
bool result = 75.RndBool(); // 75% вероятность получить true
project.SendInfoToLog($"Random bool: {result}");
```