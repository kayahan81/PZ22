---
# Практическое задание 22

## ЭФМО-02-25 

## Алиев Каяхан Командар оглы
---
# Тема работы
Браузерные угрозы CSRF/XSS и безопасная работа с cookies.

## Цели занятия
Научиться безопасно использовать cookies в серверном приложении и внедрить практические меры защиты от CSRF и XSS.

## Структура проекта
```
C:.
│   .gitattributes
│   .gitignore
│   go.mod
│   go.sum
│   README.md
│   testdata.bat
│
├───.vs
│   │   ProjectSettings.json
│   │   slnx.sqlite
│   │   VSWorkspaceState.json
│   │
│   └───tech-ip-sem2
│       ├───FileContentIndex
│       │       2019e4a7-05d4-4380-9757-192646eef486.vsidx
│       │
│       └───v17
├───deploy
│   ├───monitoring
│   │       docker-compose.yml
│   │       prometheus.yml
│   │
│   └───tls
│           cert.pem
│           docker-compose.yml
│           key.pem
│           nginx.conf
│
├───docs
│       pz17_api.md
│       pz17_diagram.md
│
├───img
├───proto
│       auth.proto
│
├───services
│   ├───auth
│   │   ├───cmd
│   │   │   └───auth
│   │   │           main.go
│   │   │
│   │   ├───internal
│   │   │   ├───config
│   │   │   ├───grpc
│   │   │   │       server.go
│   │   │   │
│   │   │   ├───handler
│   │   │   │       auth.go
│   │   │   │
│   │   │   ├───http
│   │   │   ├───middleware
│   │   │   └───service
│   │   └───pkg
│   │       └───authpb
│   │               auth.pb.go
│   │               auth_grpc.pb.go
│   │
│   └───tasks
│       │   Dockerfile
│       │
│       ├───cmd
│       │   └───tasks
│       │           main.go
│       │
│       └───internal
│           ├───client
│           │   ├───authclient
│           │   │       client.go
│           │   │
│           │   └───authgrpc
│           │           client.go
│           │
│           ├───handler
│           │       tasks.go
│           │
│           ├───http
│           ├───middleware
│           │       auth.go
│           │       auth_cookie.go
│           │       csrf.go
│           │       metric.go
│           │       security_headers.go
│           │
│           ├───migration
│           │       001_create_tasks.sql
│           │
│           ├───models
│           │       tasks.go
│           │
│           ├───repository
│           │       postgres.go
│           │
│           ├───service
│           └───storage
│                   memory.go
│
└───shared
    ├───csrf
    │       generator.go
    │
    ├───httpx
    │       client.go
    │
    ├───logger
    │       logger.go
    │
    └───middleware
            accesslog.go
            requestid.go
```

## Коды статуса:
-	200 OK — успешный ответ
-	201 Created — ресурс создан
-	204 No Content — успешно, без тела
-	400 Bad Request — неверные данные
-	404 Not Found — ресурс не найден
-	422 Unprocessable Entity — некорректные данные по смыслу
-	500 Internal Server Error — внутренняя ошибка

# Примечания по конфигурации и требования

Для запуска требуется:

Go: версия 1.25.1

<img width="841" height="232" alt="Установка Git и Go" src="https://github.com/user-attachments/assets/8e01d831-5a7f-4376-8348-9052b240aec9" />


# Команды запуска/сборки
## 1) Клонировать данный репозиторий в удобную для вас папку:
```Powershell
git clone https://github.com/kayahan81/pz22
```
## 2) Перейти в папку pz19:
```Powershell
cd pz22
```
## 3) Загрузка зависимостей:
```Powershell
go mod tidy
```
## 4) Команда запуска
В первом окне
```Powershell
cd services/auth
$env:AUTH_PORT="8081"
$env:AUTH_GRPC_PORT="50051"
$env:ENV="development"
go run ./cmd/auth
```
Во втором окне
```Powershell
cd deploy/tls
docker-compose up -d
```

# Проверка работоспособности
## Получаем cookies (login)
<img width="846" height="352" alt="image" src="https://github.com/user-attachments/assets/19984d48-0cb6-4061-b640-4d3a43ad552f" />
## Проверка запроса без cookies (POST 403)
<img width="803" height="351" alt="image" src="https://github.com/user-attachments/assets/5b0f368d-5c4d-4085-b021-69adce52de34" />
## Корректный запрос (POST 201)
<img width="795" height="451" alt="image" src="https://github.com/user-attachments/assets/b5d63822-b5e3-43ca-a1fe-4e1f4ec841f8" />



# Отчёт
1.	Какие cookies используются и какие флаги установлены (HttpOnly/Secure/SameSite/Max-Age).
session_id (HttpOnly, Secure, SameSite=Lax), csrf_token (Secure, SameSite=Lax, НЕ HttpOnly)
2.	Какой CSRF подход выбран и как он работает (double submit).
Double Submit Cookie — сервер выдаёт csrf_token в cookie, клиент отправляет его в заголовке X-CSRF-Token
3.	Примеры запросов:
См. пункт "Проверка работоспособности"
4.	Что сделано для XSS (правило обработки description и/или заголовки безопасности).
Приняты меры: экранирование через html.EscapeString + заголовки X-Content-Type-Options, CSP, X-Frame-Options


# Ответы на вопросы
1.	Почему CSRF возможен при использовании cookies?
CSRF возможен, потому что браузер автоматически прикрепляет cookies к запросам на ваш домен, даже если запрос инициирован с вредоносного сайта, что позволяет злоумышленнику «заставить» браузер жертвы отправить аутентифицированный запрос без её ведома.
2.	Что делает флаг SameSite и какие есть режимы?
Флаг SameSite ограничивает отправку cookies при кросс-сайтовых запросах; режим Lax — разумный минимум для большинства сценариев, Strict — максимально строгий (может ломать UX), а None — разрешает отправку всегда, но требует флага Secure.
3.	Чем HttpOnly защищает от XSS и почему он не “лечит” XSS полностью?
HttpOnly запрещает JavaScript читать cookie, что снижает риск кражи сессии при XSS, но не лечит XSS полностью, потому что злоумышленник всё равно может выполнить вредоносный код от имени пользователя (например, сделать запросы от его лица), просто не сможет украсть саму cookie.
4.	Почему Secure обязателен, если cookie несёт сессию?
Secure обязателен для сессионной cookie, потому что иначе она будет передаваться по незашифрованному HTTP-соединению, и злоумышленник может перехватить её (например, при атаке Man-in-the-Middle).
5.	Как работает double-submit CSRF защита?
Double-submit CSRF защита работает так: сервер выдаёт CSRF cookie и одновременно отдаёт его значение клиенту; на каждый опасный запрос клиент отправляет заголовок X-CSRF-Token с этим значением, а сервер сравнивает его со значением в cookie — вредоносный сайт не может прочитать cookie другого домена и сформировать правильный заголовок.
6.	Что такое XSS и какие базовые меры защиты применимы на backend?
XSS — это атака, при которой злоумышленник внедряет вредоносный код в веб-страницу; на backend базовые меры включают экранирование/санитизацию пользовательского ввода (например, замена <script> на безопасные сущности) и добавление заголовков безопасности вроде Content-Security-Policy.
