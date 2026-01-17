# Токены авторизации: Практическое руководство для автоматизации

## 1. Типы токенов

### JWT (JSON Web Token)

Самодостаточный токен, содержащий всю информацию о пользователе в закодированном виде. Сервер может валидировать токен без обращения к БД — достаточно проверить подпись.

**Формат:** три части через точку `header.payload.signature`, каждая часть — Base64URL encoded.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.Gfx6VO9tcxwk6xqx9yYzSfebfeakZp5JYIgP_edcw_A
```

**Хранение:** клиентская сторона (localStorage, sessionStorage, cookie, память приложения).

### Bearer Token

Не формат, а **способ передачи** любого токена в HTTP заголовке. Bearer означает "предъявитель" — кто владеет токеном, тот авторизован.

```http
Authorization: Bearer <любой_токен>
```

Bearer токеном может быть JWT, opaque token, или любая другая строка.

### OAuth Tokens

OAuth 2.0 выдаёт два типа токенов:

**Access Token** — короткоживущий (минуты-часы), даёт доступ к API. Может быть JWT или opaque string.

**Refresh Token** — долгоживущий (дни-месяцы), используется только для получения новых access token. Всегда opaque, хранится на сервере авторизации.

### Session Token / Session ID

Opaque идентификатор сессии на сервере. Сам по себе не содержит данных — это ключ к записи в серверной БД/Redis.

```
Set-Cookie: PHPSESSID=abc123def456; HttpOnly; Secure
Set-Cookie: ASP.NET_SessionId=xyz789; HttpOnly
```

**Хранение:** Cookie (клиент) + серверное хранилище (Redis, БД, память).

### API Keys

Статические ключи для идентификации приложения/пользователя. Не привязаны к сессии, обычно долгоживущие.

**Типичные форматы:**
```
sk-proj-abc123...         # OpenAI style
ghp_xxxxxxxxxxxx          # GitHub PAT
AKIA...                   # AWS Access Key
```

**Передача:** Header (`X-API-Key`, `Authorization`), query param (`?api_key=xxx`), тело запроса.

---

## 2. JWT детально

### Структура

```
HEADER.PAYLOAD.SIGNATURE
```

**Header (алгоритм + тип):**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload (claims — утверждения о пользователе):**
```json
{
  "iss": "https://auth.example.com",
  "sub": "user_12345",
  "aud": "https://api.example.com",
  "iat": 1704067200,
  "exp": 1704070800,
  "nbf": 1704067200,
  "name": "John Doe",
  "role": "admin"
}
```

**Signature:**
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

### Стандартные Claims (RFC 7519)

| Claim | Название | Описание |
|-------|----------|----------|
| `iss` | Issuer | Кто выдал токен (URL сервера авторизации) |
| `sub` | Subject | ID пользователя/сущности |
| `aud` | Audience | Для кого предназначен (URL API) |
| `iat` | Issued At | Время выдачи (Unix timestamp) |
| `exp` | Expiration | Время истечения (Unix timestamp) |
| `nbf` | Not Before | Не использовать до (Unix timestamp) |
| `jti` | JWT ID | Уникальный идентификатор токена |

### Декодирование в C#

```csharp
// Без валидации подписи — просто чтение payload
public static string DecodeJwtPayload(string jwt)
{
    var parts = jwt.Split('.');
    if (parts.Length != 3) 
        throw new ArgumentException("Invalid JWT format");
    
    // Base64URL -> Base64 -> bytes -> string
    var payload = parts[1]
        .Replace('-', '+')
        .Replace('_', '/');
    
    // Padding
    switch (payload.Length % 4)
    {
        case 2: payload += "=="; break;
        case 3: payload += "="; break;
    }
    
    var bytes = Convert.FromBase64String(payload);
    return Encoding.UTF8.GetString(bytes);
}

// Проверка expiration
public static bool IsJwtExpired(string jwt)
{
    var payload = DecodeJwtPayload(jwt);
    var json = JsonDocument.Parse(payload);
    
    if (json.RootElement.TryGetProperty("exp", out var expElement))
    {
        var exp = expElement.GetInt64();
        var expTime = DateTimeOffset.FromUnixTimeSeconds(exp);
        return DateTimeOffset.UtcNow > expTime;
    }
    return false; // Нет exp — не истекает
}
```

### Signing Algorithms

**Симметричные (shared secret):**
- `HS256` — HMAC + SHA-256 (самый частый)
- `HS384`, `HS512` — варианты с большей длиной хэша

**Асимметричные (public/private key):**
- `RS256` — RSA + SHA-256 (Google, Microsoft используют)
- `ES256` — ECDSA + SHA-256 (компактнее RSA)

**None (опасно):**
- `none` — без подписи. Никогда не принимать такие токены!

```csharp
// Пример валидации HS256 в .NET
using System.IdentityModel.Tokens.Jwt;
using Microsoft.IdentityModel.Tokens;

var handler = new JwtSecurityTokenHandler();
var validationParams = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidIssuer = "https://auth.example.com",
    ValidateAudience = true,
    ValidAudience = "https://api.example.com",
    ValidateLifetime = true,
    IssuerSigningKey = new SymmetricSecurityKey(
        Encoding.UTF8.GetBytes("your-256-bit-secret-key-here!!")
    ),
    ClockSkew = TimeSpan.FromMinutes(1) // Допуск на рассинхрон часов
};

try
{
    var principal = handler.ValidateToken(jwt, validationParams, out var validatedToken);
    // Токен валиден
}
catch (SecurityTokenExpiredException)
{
    // Токен истёк
}
catch (SecurityTokenInvalidSignatureException)
{
    // Подпись не совпадает
}
```

---

## 3. Refresh Tokens

### Механизм работы

```
┌─────────┐                    ┌─────────────┐                    ┌─────────┐
│ Client  │                    │ Auth Server │                    │   API   │
└────┬────┘                    └──────┬──────┘                    └────┬────┘
     │                                │                                 │
     │  1. Login (credentials)        │                                 │
     │ ─────────────────────────────► │                                 │
     │                                │                                 │
     │  2. Access Token (15min)       │                                 │
     │     + Refresh Token (7d)       │                                 │
     │ ◄───────────────────────────── │                                 │
     │                                │                                 │
     │  3. API Request + Access Token │                                 │
     │ ────────────────────────────────────────────────────────────────►│
     │                                │                                 │
     │  4. Response                   │                                 │
     │ ◄────────────────────────────────────────────────────────────────│
     │                                │                                 │
     │  ... Access Token expires ...  │                                 │
     │                                │                                 │
     │  5. API Request + Expired Token│                                 │
     │ ────────────────────────────────────────────────────────────────►│
     │                                │                                 │
     │  6. 401 Unauthorized           │                                 │
     │ ◄────────────────────────────────────────────────────────────────│
     │                                │                                 │
     │  7. Refresh (refresh_token)    │                                 │
     │ ─────────────────────────────► │                                 │
     │                                │                                 │
     │  8. New Access Token           │                                 │
     │     (+ New Refresh Token)      │                                 │
     │ ◄───────────────────────────── │                                 │
     │                                │                                 │
     │  9. Retry API Request          │                                 │
     │ ────────────────────────────────────────────────────────────────►│
```

### HTTP примеры

**Получение токенов (OAuth 2.0 Password Grant):**
```http
POST /oauth/token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=user@example.com&password=secret123&client_id=myapp
```

**Ответ:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 900,
  "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

**Обновление токена:**
```http
POST /oauth/token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...&client_id=myapp
```

### Когда использовать

**Refresh нужен когда:**
- API требует короткоживущие access tokens (< 1 час)
- Нужна долгая сессия без повторного ввода пароля
- Требуется возможность отзыва доступа (revoke refresh token на сервере)

**Refresh НЕ нужен когда:**
- API выдаёт долгоживущие токены (API keys)
- Сессия привязана к cookies с автообновлением
- Каждый запрос требует свежую авторизацию (редко)

---

## 4. Где токены передаются

### Authorization Header (рекомендуемый способ)

```http
GET /api/users/me HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Плюсы:** стандартно, не кэшируется прокси, не сохраняется в логах URL.

**Минусы:** нужно вручную добавлять к каждому запросу.

### Cookie

```http
GET /api/users/me HTTP/1.1
Host: api.example.com
Cookie: access_token=eyJhbGciOiJIUzI1NiIs...; session_id=abc123
```

**Плюсы:** автоматически отправляется браузером, можно защитить HttpOnly (недоступен из JS).

**Минусы:** уязвим к CSRF (нужен CSRF token), привязан к домену.

**Важные флаги Cookie:**
```http
Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```
- `HttpOnly` — недоступен из JavaScript
- `Secure` — только по HTTPS
- `SameSite=Strict` — защита от CSRF
- `Path=/` — для всех путей домена

### Query Parameters (избегать для токенов)

```http
GET /api/users/me?access_token=eyJhbGciOiJIUzI1NiIs... HTTP/1.1
Host: api.example.com
```

**Проблемы:**
- Токен в логах сервера/прокси
- Токен в Referer header при переходах
- Токен в истории браузера

**Когда допустимо:** одноразовые токены (email confirmation), WebSocket connections (нет заголовков), signed URLs для файлов.

### Custom Headers

```http
GET /api/data HTTP/1.1
Host: api.example.com
X-API-Key: sk_live_abc123
X-Auth-Token: xyz789
```

**Когда использовать:** legacy API, специфические требования сервиса.

---

## 5. Схемы авторизации в HTTP

### Bearer (OAuth 2.0)

```http
Authorization: Bearer <token>
```

Самая распространённая схема. Токен передаётся как есть, без дополнительной обработки.

### Basic

```http
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

Base64 от `username:password`. Используется для API keys или простой авторизации.

```csharp
var credentials = Convert.ToBase64String(
    Encoding.UTF8.GetBytes($"{username}:{password}")
);
var header = $"Basic {credentials}";
```

### Digest (редко)

Challenge-response схема. Сервер отправляет nonce, клиент хэширует credentials с nonce.

```http
# Первый запрос
GET /protected HTTP/1.1

# Ответ сервера
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Digest realm="api", nonce="abc123", qop="auth"

# Повторный запрос с Digest
Authorization: Digest username="user", realm="api", nonce="abc123", 
               uri="/protected", response="6629fae49393a05397450978507c4ef1"
```

### AWS Signature v4

Сложная схема с подписью запроса. Каждый запрос подписывается secret key.

```http
Authorization: AWS4-HMAC-SHA256 
    Credential=AKIAIOSFODNN7EXAMPLE/20230101/us-east-1/s3/aws4_request,
    SignedHeaders=host;x-amz-date,
    Signature=calculated_signature
```

### Custom / Proprietary

```http
# Twitter OAuth 1.0
Authorization: OAuth oauth_consumer_key="...", oauth_signature="...", ...

# Custom API
X-Signature: hmac_sha256(request_body + timestamp, secret_key)
X-Timestamp: 1704067200
```

---

## 6. Алгоритм отладки

### Перехват токена из браузера

**DevTools → Network tab:**
1. Открыть DevTools (F12)
2. Network → включить "Preserve log"
3. Выполнить действие (логин, запрос к API)
4. Найти запрос → Headers → Authorization / Cookie
5. Request/Response для анализа flow

**Фильтрация в DevTools:**
- `method:POST` — только POST запросы
- `domain:api.example.com` — конкретный домен
- `-status-code:200` — исключить успешные

**Copy as cURL:**
ПКМ на запросе → Copy → Copy as cURL → получаешь готовую команду со всеми headers.

### Перехват через прокси (Fiddler, Charles, mitmproxy)

```bash
# mitmproxy для HTTPS трафика
mitmproxy --listen-port 8080

# Экспорт в HAR
mitmdump -w traffic.har --set hardump=true
```

**ZennoPoster:** Traffic → HTTP Analyzer, или встроенный Fiddler.

### Анализ токена

**JWT decode онлайн:** jwt.io, jwt.ms

**В коде:**
```csharp
public static void AnalyzeToken(string token)
{
    // Проверяем формат
    var parts = token.Split('.');
    
    if (parts.Length == 3)
    {
        Console.WriteLine("=== JWT Token ===");
        Console.WriteLine($"Header: {DecodeBase64Url(parts[0])}");
        Console.WriteLine($"Payload: {DecodeBase64Url(parts[1])}");
        Console.WriteLine($"Signature: {parts[2].Substring(0, 20)}...");
        
        // Проверяем expiration
        var payload = JsonDocument.Parse(DecodeBase64Url(parts[1]));
        if (payload.RootElement.TryGetProperty("exp", out var exp))
        {
            var expTime = DateTimeOffset.FromUnixTimeSeconds(exp.GetInt64());
            var remaining = expTime - DateTimeOffset.UtcNow;
            Console.WriteLine($"Expires: {expTime:u} ({remaining.TotalMinutes:F0} min remaining)");
        }
    }
    else
    {
        Console.WriteLine("=== Opaque Token ===");
        Console.WriteLine($"Length: {token.Length}");
        Console.WriteLine($"Preview: {token.Substring(0, Math.Min(50, token.Length))}...");
    }
}
```

### Диагностика: токен протух?

```csharp
public enum TokenStatus
{
    Valid,
    Expired,
    NotYetValid,
    InvalidFormat,
    MissingExpiration
}

public static TokenStatus CheckTokenStatus(string jwt)
{
    try
    {
        var payload = DecodeJwtPayload(jwt);
        var json = JsonDocument.Parse(payload);
        var now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
        
        // Проверка exp
        if (json.RootElement.TryGetProperty("exp", out var expEl))
        {
            if (expEl.GetInt64() < now)
                return TokenStatus.Expired;
        }
        else
        {
            return TokenStatus.MissingExpiration;
        }
        
        // Проверка nbf (not before)
        if (json.RootElement.TryGetProperty("nbf", out var nbfEl))
        {
            if (nbfEl.GetInt64() > now)
                return TokenStatus.NotYetValid;
        }
        
        return TokenStatus.Valid;
    }
    catch
    {
        return TokenStatus.InvalidFormat;
    }
}
```

### Обновление токена

```csharp
public async Task<string> EnsureValidToken()
{
    var status = CheckTokenStatus(_accessToken);
    
    if (status == TokenStatus.Valid)
        return _accessToken;
    
    if (status == TokenStatus.Expired && !string.IsNullOrEmpty(_refreshToken))
    {
        // Пробуем обновить
        var newTokens = await RefreshTokens(_refreshToken);
        if (newTokens != null)
        {
            _accessToken = newTokens.AccessToken;
            _refreshToken = newTokens.RefreshToken;
            return _accessToken;
        }
    }
    
    // Refresh не сработал — нужен полный reauth
    throw new AuthenticationRequiredException("Token expired, re-login required");
}
```

---

## 7. Частые проблемы

### CORS (Cross-Origin Resource Sharing)

**Проблема:** браузер блокирует запрос к другому домену.

```
Access to fetch at 'https://api.example.com' from origin 'https://myapp.com' 
has been blocked by CORS policy
```

**Решение на сервере:**
```http
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Credentials: true
```

**В автоматизации:** CORS — проблема только браузеров. HttpClient/WebClient игнорируют CORS.

### Preflight Requests

**Проблема:** браузер отправляет OPTIONS запрос перед основным.

```http
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Origin: https://myapp.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization
```

**Когда возникает:**
- Методы кроме GET, HEAD, POST
- Кастомные заголовки (Authorization, X-Custom-Header)
- Content-Type кроме text/plain, multipart/form-data, application/x-www-form-urlencoded

**В автоматизации:** если повторяешь браузерный flow — отправляй OPTIONS если видел его в traffic.

### Token Expiration

**401 Unauthorized с разными причинами:**

```json
// Token expired
{"error": "token_expired", "error_description": "Access token has expired"}

// Token invalid
{"error": "invalid_token", "error_description": "Token signature verification failed"}

// Token revoked
{"error": "invalid_token", "error_description": "Token has been revoked"}
```

**Обработка:**
```csharp
public async Task<HttpResponseMessage> SendWithRetry(HttpRequestMessage request)
{
    var response = await _httpClient.SendAsync(CloneRequest(request));
    
    if (response.StatusCode == HttpStatusCode.Unauthorized)
    {
        var content = await response.Content.ReadAsStringAsync();
        
        if (content.Contains("expired"))
        {
            // Пробуем refresh
            if (await RefreshAccessToken())
            {
                // Повторяем с новым токеном
                request = CloneRequest(request);
                request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", _accessToken);
                return await _httpClient.SendAsync(request);
            }
        }
        
        // Token invalid/revoked — нужен полный reauth
        throw new AuthenticationRequiredException();
    }
    
    return response;
}
```

### Signature Validation Failed

**Причины:**
- Токен изменён (подделка)
- Неправильный secret/key на сервере
- Токен для другого environment (dev vs prod)
- Алгоритм не совпадает

**Диагностика:**
```csharp
// Проверить алгоритм в header
var header = DecodeBase64Url(jwt.Split('.')[0]);
// {"alg":"RS256","typ":"JWT"} — нужен public key
// {"alg":"HS256","typ":"JWT"} — нужен shared secret
```

### 403 Forbidden (не путать с 401)

**401 Unauthorized** — нет авторизации или токен невалиден. Можно попробовать refresh.

**403 Forbidden** — авторизация есть, но нет прав. Refresh не поможет.

```csharp
if (response.StatusCode == HttpStatusCode.Forbidden)
{
    // Не пытаемся refresh — это проблема прав, а не токена
    throw new InsufficientPermissionsException("Access denied");
}
```

---

## 8. Практические паттерны

### Retry с автоматическим Refresh

```csharp
public class AuthenticatedHttpClient
{
    private readonly HttpClient _client;
    private readonly SemaphoreSlim _refreshLock = new(1, 1);
    private string _accessToken;
    private string _refreshToken;
    private DateTimeOffset _tokenExpiry;
    
    public async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request)
    {
        // Проверяем expiry до запроса (оптимизация)
        if (DateTimeOffset.UtcNow >= _tokenExpiry.AddMinutes(-1))
        {
            await RefreshTokenAsync();
        }
        
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", _accessToken);
        var response = await _client.SendAsync(await CloneRequestAsync(request));
        
        // 401 — пробуем refresh и retry
        if (response.StatusCode == HttpStatusCode.Unauthorized)
        {
            if (await RefreshTokenAsync())
            {
                var retryRequest = await CloneRequestAsync(request);
                retryRequest.Headers.Authorization = new AuthenticationHeaderValue("Bearer", _accessToken);
                return await _client.SendAsync(retryRequest);
            }
        }
        
        return response;
    }
    
    private async Task<bool> RefreshTokenAsync()
    {
        // Один refresh за раз (избегаем race condition)
        await _refreshLock.WaitAsync();
        try
        {
            // Double-check — может другой поток уже обновил
            if (DateTimeOffset.UtcNow < _tokenExpiry.AddMinutes(-1))
                return true;
            
            var refreshRequest = new HttpRequestMessage(HttpMethod.Post, "/oauth/token")
            {
                Content = new FormUrlEncodedContent(new Dictionary<string, string>
                {
                    ["grant_type"] = "refresh_token",
                    ["refresh_token"] = _refreshToken
                })
            };
            
            var response = await _client.SendAsync(refreshRequest);
            if (!response.IsSuccessStatusCode)
                return false;
            
            var json = await response.Content.ReadFromJsonAsync<TokenResponse>();
            _accessToken = json.AccessToken;
            _refreshToken = json.RefreshToken ?? _refreshToken;
            _tokenExpiry = DateTimeOffset.UtcNow.AddSeconds(json.ExpiresIn);
            
            return true;
        }
        finally
        {
            _refreshLock.Release();
        }
    }
    
    private async Task<HttpRequestMessage> CloneRequestAsync(HttpRequestMessage request)
    {
        var clone = new HttpRequestMessage(request.Method, request.RequestUri);
        
        foreach (var header in request.Headers)
            clone.Headers.TryAddWithoutValidation(header.Key, header.Value);
        
        if (request.Content != null)
        {
            var content = await request.Content.ReadAsByteArrayAsync();
            clone.Content = new ByteArrayContent(content);
            
            foreach (var header in request.Content.Headers)
                clone.Content.Headers.TryAddWithoutValidation(header.Key, header.Value);
        }
        
        return clone;
    }
}
```

### Хранение токенов в БД

```csharp
public class TokenStorage
{
    // Структура для хранения
    public class StoredToken
    {
        public string AccountId { get; set; }
        public string AccessToken { get; set; }
        public string RefreshToken { get; set; }
        public DateTimeOffset AccessTokenExpiry { get; set; }
        public DateTimeOffset? RefreshTokenExpiry { get; set; }
        public string TokenType { get; set; } // "Bearer", "OAuth1", etc.
        public string Scope { get; set; }
        public DateTimeOffset UpdatedAt { get; set; }
    }
    
    // SQLite пример
    public async Task SaveTokenAsync(StoredToken token)
    {
        const string sql = @"
            INSERT OR REPLACE INTO tokens 
            (account_id, access_token, refresh_token, access_token_expiry, 
             refresh_token_expiry, token_type, scope, updated_at)
            VALUES 
            (@AccountId, @AccessToken, @RefreshToken, @AccessTokenExpiry,
             @RefreshTokenExpiry, @TokenType, @Scope, @UpdatedAt)";
        
        await _connection.ExecuteAsync(sql, token);
    }
    
    // Получение с проверкой
    public async Task<StoredToken> GetValidTokenAsync(string accountId)
    {
        var token = await GetTokenAsync(accountId);
        
        if (token == null)
            return null;
        
        // Access token валиден
        if (token.AccessTokenExpiry > DateTimeOffset.UtcNow.AddMinutes(1))
            return token;
        
        // Refresh token есть и валиден
        if (!string.IsNullOrEmpty(token.RefreshToken) && 
            (token.RefreshTokenExpiry == null || token.RefreshTokenExpiry > DateTimeOffset.UtcNow))
        {
            // Нужен refresh
            return token; // Вызывающий код должен сделать refresh
        }
        
        // Всё протухло
        return null;
    }
}
```

### Определение когда нужен новый auth flow

```csharp
public enum AuthAction
{
    UseExisting,      // Токен валиден, используем
    RefreshRequired,  // Нужен refresh
    ReauthRequired    // Нужна полная переавторизация
}

public AuthAction DetermineAuthAction(StoredToken token)
{
    if (token == null)
        return AuthAction.ReauthRequired;
    
    var now = DateTimeOffset.UtcNow;
    
    // Access token валиден (с запасом 1 минута)
    if (token.AccessTokenExpiry > now.AddMinutes(1))
        return AuthAction.UseExisting;
    
    // Есть refresh token
    if (!string.IsNullOrEmpty(token.RefreshToken))
    {
        // Refresh token без expiry или ещё валиден
        if (token.RefreshTokenExpiry == null || token.RefreshTokenExpiry > now)
            return AuthAction.RefreshRequired;
    }
    
    // Ничего не работает
    return AuthAction.ReauthRequired;
}

// Использование
public async Task<string> GetAccessTokenAsync(string accountId)
{
    var token = await _storage.GetTokenAsync(accountId);
    var action = DetermineAuthAction(token);
    
    switch (action)
    {
        case AuthAction.UseExisting:
            return token.AccessToken;
            
        case AuthAction.RefreshRequired:
            var newToken = await _authService.RefreshAsync(token.RefreshToken);
            await _storage.SaveTokenAsync(newToken);
            return newToken.AccessToken;
            
        case AuthAction.ReauthRequired:
            throw new ReauthorizationRequiredException(accountId);
    }
}
```

### Паттерн для массовых операций (фермы аккаунтов)

```csharp
public class BulkTokenManager
{
    private readonly ConcurrentDictionary<string, SemaphoreSlim> _accountLocks = new();
    
    public async Task<List<(string AccountId, string Token, Exception Error)>> 
        EnsureValidTokensAsync(IEnumerable<string> accountIds)
    {
        var tasks = accountIds.Select(async id =>
        {
            // Один refresh на аккаунт одновременно
            var lockObj = _accountLocks.GetOrAdd(id, _ => new SemaphoreSlim(1, 1));
            await lockObj.WaitAsync();
            
            try
            {
                var token = await GetAccessTokenAsync(id);
                return (id, token, (Exception)null);
            }
            catch (Exception ex)
            {
                return (id, (string)null, ex);
            }
            finally
            {
                lockObj.Release();
            }
        });
        
        return (await Task.WhenAll(tasks)).ToList();
    }
    
    // Пакетная обработка с rate limiting
    public async IAsyncEnumerable<(string AccountId, string Token)> 
        GetTokensWithRateLimit(IEnumerable<string> accountIds, int maxConcurrent = 10)
    {
        var semaphore = new SemaphoreSlim(maxConcurrent);
        var accounts = accountIds.ToList();
        
        foreach (var batch in accounts.Chunk(maxConcurrent))
        {
            var tasks = batch.Select(async id =>
            {
                await semaphore.WaitAsync();
                try
                {
                    return (id, await GetAccessTokenAsync(id));
                }
                finally
                {
                    semaphore.Release();
                }
            });
            
            foreach (var result in await Task.WhenAll(tasks))
            {
                yield return result;
            }
            
            // Rate limit между батчами
            await Task.Delay(1000);
        }
    }
}
```

---

## Быстрая справка

### Проверка типа токена

```
eyJ... (starts with eyJ) → JWT (base64 encoded JSON starts with {"alg...)
ghp_... → GitHub PAT
sk-... → OpenAI API Key
AKIA... → AWS Access Key
xoxb-... → Slack Bot Token
random alphanumeric → Opaque token (session/refresh)
```

### HTTP статусы авторизации

```
200 OK          → Всё хорошо
400 Bad Request → Неправильный формат запроса (не auth проблема)
401 Unauthorized → Нет токена / токен невалиден / токен истёк
403 Forbidden   → Токен валиден, но нет прав
429 Too Many    → Rate limit (не auth, но часто путают)
```

### Checklist отладки 401

1. Токен вообще есть в запросе?
2. Правильный header? (`Authorization: Bearer` vs `Authorization: Basic`)
3. Токен не истёк? (проверить `exp` claim)
4. Токен для правильного environment? (dev/staging/prod)
5. Токен для правильного audience? (проверить `aud` claim)
6. Refresh token работает?
7. Сервер вообще принимает этот тип токена?

### Минимальный код для дебага

```csharp
// Быстрый decode JWT
string payload = Encoding.UTF8.GetString(
    Convert.FromBase64String(
        jwt.Split('.')[1]
            .Replace('-', '+')
            .Replace('_', '/')
            .PadRight((jwt.Split('.')[1].Length + 3) & ~3, '=')
    )
);
Console.WriteLine(payload);

// Быстрая проверка expiry
var exp = JsonDocument.Parse(payload).RootElement.GetProperty("exp").GetInt64();
var expiresAt = DateTimeOffset.FromUnixTimeSeconds(exp);
Console.WriteLine($"Expires: {expiresAt:u}, Remaining: {(expiresAt - DateTimeOffset.UtcNow).TotalMinutes:F0} min");
```