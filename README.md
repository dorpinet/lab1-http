# Практична робота № 1
 
**Тема:** Дослідження мережевих протоколів засобами інструментів розробника
 
## Хід роботи
 
використано: Git Bash, Postman, Google Chrome. Команди виконуються в Git Bash у папці проєкту:
 
```bash
cd lab1-http
curl --version
```
 
![Середовище](screenshots/Screenshot%202026-10-07%20213036.png)
 
## Рівень 1
 
### Postman
 
| Метод | URL | Тіло | Результат |
|---|---|---|---|
| GET | `https://dummyjson.com/products/1` | немає | 200 OK |
| POST | `https://dummyjson.com/products/add` | `{"title":"test bibibobo","price":123}` | 201 Created |
| PUT | `https://dummyjson.com/products/1` | `{"title":"new bibu"}` | 200 OK |
| DELETE | `https://dummyjson.com/products/1` | немає | 200 OK |
 
![GET](screenshots/Screenshot%202026-10-07%20213812.png)
![POST](screenshots/Screenshot%202026-10-07%20214006.png)
![PUT](screenshots/Screenshot%202026-10-07%20214151.png)
![DELETE](screenshots/Screenshot%202026-10-07%20214213.png)
 
### DevTools: `GET https://dummyjson.com/products`
 
| Розділ | Поле | Значення |
|---|---|---|
| General | Request URL | `https://dummyjson.com/products` |
| General | Request Method | `GET` |
| General | Status Code | `200 OK` |
| General | Remote Address | `104.21.61.23:443` |
| Response | content-type | `application/json; charset=utf-8` |
| Response | date | `Wed, 07 Oct 2026 18:44:48 GMT` |
| Response | server | `cloudflare` |
| Response | etag | `W/"ac3a-Q0j5X7Zb/GG4CpZwhP3POutAwN4"` |
| Request | accept | `text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7` |
| Request | accept-encoding | `gzip, deflate, br, zstd` |
| Request | user-agent | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36` |
| Timing | DNS Lookup, Initial connection | відсутні (з'єднання використано повторно) |
| Timing | Waiting (TTFB) | 35.23 мс |
| Timing | Content Download | 0.73 мс |
| Timing | Разом | 37.23 мс |
 
![Headers](screenshots/Screenshot%202026-10-07%20214516.png)
![Timing](screenshots/Screenshot%202026-10-08%20161440.png)
 
## Рівень 2
 
```bash
curl "https://dummyjson.com/users?limit=2&select=firstName,email"
```
![](screenshots/Screenshot%202026-10-09%20141755.png)
 
```bash
curl -X POST https://dummyjson.com/posts/add -H "Content-Type: application/json" -d '{"title":"My post","userId":5}'
```
![](screenshots/Screenshot%202026-10-09%20142111.png)
 
```bash
curl -I https://dummyjson.com/products
```
![](screenshots/Screenshot%202026-10-09%20142136.png)
 
```bash
curl -i https://httpbin.org/status/404
curl -i https://httpbin.org/status/500
```
![](screenshots/Screenshot%202026-10-09%20142157.png)
![](screenshots/Screenshot%202026-10-09%20142212.png)
 
```bash
curl -X POST https://dummyjson.com/auth/login -H "Content-Type: application/json" -d '{"username":"emilys","password":"emilyspass"}'
```
![](screenshots/Screenshot%202026-10-09%20142227.png)
 
```bash
TOKEN=$(curl -s -X POST https://dummyjson.com/auth/login -H "Content-Type: application/json" -d '{"username":"emilys","password":"emilyspass","expiresInMins":60}' | sed -E 's/.*"accessToken":"([^"]+)".*/\1/')
curl https://dummyjson.com/auth/me -H "Authorization: Bearer $TOKEN"
```
![](screenshots/Screenshot%202026-10-09%20142733.png)
 
```bash
curl -s -o /dev/null -w "DNS: %{time_namelookup}s\nTCP Connect: %{time_connect}s\nTLS Handshake: %{time_appconnect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" https://dummyjson.com/products
```
![](screenshots/Screenshot%202026-10-09%20142847.png)
 
## Рівень 3
 
### TLS
 
```bash
curl -v https://dummyjson.com/products/1
```
![](screenshots/Screenshot%202026-10-11%20185017.png)
 
curl на Windows використовує schannel і не показує деталі рукостискання, тому TLS-дані отримано через `openssl`:
 
```bash
openssl s_client -connect dummyjson.com:443 -servername dummyjson.com </dev/null 2>&1 | grep -E "Protocol|Cipher|issuer|subject|Verif"
openssl s_client -connect dummyjson.com:443 -servername dummyjson.com </dev/null 2>/dev/null | openssl x509 -noout -dates -ext subjectAltName
```
![](screenshots/Screenshot%202026-10-11%20185613.png)
 
| Параметр | Значення |
|---|---|
| TCP | `172.67.205.42:443` |
| Протокол | TLSv1.3 |
| Шифронабір | TLS_AES_256_GCM_SHA384 |
| Subject | `CN=dummyjson.com` |
| Issuer | `C=US, O=Google Trust Services, CN=WE1` |
| Дійсний з | `Oct 6 18:02:33 2026 GMT` |
| Дійсний до | `Jan 4 19:01:08 2027 GMT` |
| SAN | `DNS:dummyjson.com`, `DNS:*.dummyjson.com` |
| Перевірка | `Verification: OK` |
 
```bash
curl -I http://dummyjson.com/products/1
```
Результат: `301 Moved Permanently`, `Location: https://dummyjson.com/products/1`.
 
![](screenshots/Screenshot%202026-10-11%20185710.png)
 
### Cookies
 
```bash
curl -c cookies.txt "https://httpbin.org/cookies/set?user_role=student&session_key=lab1_token"
cat cookies.txt
curl -b cookies.txt https://httpbin.org/cookies
```
![](screenshots/Screenshot%202026-10-11%20185727.png)
![](screenshots/Screenshot%202026-10-11%20185747.png)
![](screenshots/Screenshot%202026-10-11%20185812.png)
 
Сервер розпізнав надіслані cookies (`session_key`, `user_role`) і повернув їх у тілі відповіді.
 
### Атрибути Cookie
 
```bash
curl -i "https://httpbin.org/response-headers?Set-Cookie=session_id=xyz789;%20Path=/;%20Secure;%20HttpOnly;%20SameSite=Strict"
```
![](screenshots/Screenshot%202026-10-11%20185832.png)
 
| Атрибут | Захищає від |
|---|---|
| HttpOnly | XSS (JavaScript не читає cookie) |
| Secure | перехоплення трафіку (передача лише по HTTPS) |
| SameSite | CSRF (cookie не йде з міжсайтових запитів) |
 
### CORS
 
```bash
curl -i -X OPTIONS https://httpbin.org/post -H "Origin: https://my-college-app.edu" -H "Access-Control-Request-Method: POST" -H "Access-Control-Request-Headers: Content-Type, Authorization"
```
Результат: статус `200 OK`. Сервер повернув:
 
- `Access-Control-Allow-Origin: https://my-college-app.edu`
- `Access-Control-Allow-Methods: GET, POST, PUT, DELETE, PATCH, OPTIONS`
- `Access-Control-Allow-Headers: Content-Type, Authorization`
- `Access-Control-Max-Age: 3600`
Тобто сервер дозволив запитаний origin, метод POST і обидва заголовки.
 
![](screenshots/Screenshot%202026-10-11%20190012.png)
 
## Висновки
 
Досліджено HTTP-методи, статуси та заголовки через Postman, DevTools і curl, TLS 1.3 з'єднання із сертифікатом Google Trust Services, cookies та їхні атрибути безпеки, а також preflight-запит CORS.
