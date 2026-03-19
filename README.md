---
# Практическое задание 19

## ЭФМО-02-25 

## Алиев Каяхан Командар оглы
---
# Тема работы
Структурированное логирование в серверных приложениях.

## Цели занятия
Научиться внедрять структурированные логи в сервис и применять единый стандарт логирования для диагностики и эксплуатации.

## Структура проекта
<img width="423" height="299" alt="image" src="https://github.com/user-attachments/assets/2a469ff7-c8d2-416e-b788-100d3422b63b" />
<p><img width="284" height="645" alt="image" src="https://github.com/user-attachments/assets/dd7de4e3-5417-49ba-87ca-a984fdef9594" /></p>

## Ключевая идея работы
Правильные логи — это не “печать строк”. Это события + поля, которые можно фильтровать, искать и анализировать. Поэтому в рамках задания требуется:
1.	Структурированный формат (желательно JSON).
2.	Единый набор полей во всех сервисах.
3.	Логирование жизненного цикла запроса: вход → обработка → выход.
4.	Корреляция через X-Request-ID.

Для реализации логирования был выбран zap, потому что на проде он чаще благодаря своей скорости и JSON по умолчанию

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
git clone https://github.com/kayahan81/pz19
```
## 2) Перейти в папку pz19:
```Powershell
cd pz19
```
## 3) Загрузка зависимостей:
```Powershell
go mod tidy
```
## 4) Команда запуска
В первом окне
```Powershell
$env:AUTH_PORT="8081"
$env:AUTH_GRPC_PORT="50051"
go run ./services/ayth/cmd/auth
```
Во втором окне
```Powershell
$env:TASKS_PORT="8082"
$env:AUTH_GRPC_ADDR="localhost:50051"
go run ./services/tasks/cmd/tasks
```

# Проверка работоспособности
## Получение токена
<img width="939" height="361" alt="image" src="https://github.com/user-attachments/assets/156398bb-a014-4390-9420-779a02febff0" />

### Логи auth
2026-03-18T18:26:20.855+0300    ←[34mINFO←[0m   handler/auth.go:54      login attempt   {"service": "auth", "request_id": "122f45540b754db7", "username": "student"}
2026-03-18T18:26:20.855+0300    ←[34mINFO←[0m   handler/auth.go:64      login successful        {"service": "auth", "request_id": "122f45540b754db7", "username": "student"}
2026-03-18T18:26:20.855+0300    ←[34mINFO←[0m   middleware/accesslog.go:27      request completed       {"service": "auth", "request_id": "122f45540b754db7", "method": "POST", "path": "/v1/auth/login", "status": 200, "duration_ms": 0, "remote_ip": "[::1]:49501", "user_agent": "PostmanRuntime/7.52.0"}

## Запрос с токеном
<img width="830" height="473" alt="image" src="https://github.com/user-attachments/assets/fc80bb5a-8000-49df-8fe6-90371d965219" />

### Логи auth
2026-03-18T18:28:51.280+0300    ←[34mINFO←[0m   grpc/server.go:43       gRPC: token verified    {"service": "auth", "subject": "student"}
2026-03-18T18:28:57.468+0300    ←[34mINFO←[0m   handler/auth.go:54      login attempt   {"service": "auth", "request_id": "86bc13cde0bb4a4e", "username": "student"}
2026-03-18T18:28:57.468+0300    ←[34mINFO←[0m   handler/auth.go:64      login successful        {"service": "auth", "request_id": "86bc13cde0bb4a4e", "username": "student"}
2026-03-18T18:28:57.468+0300    ←[34mINFO←[0m   middleware/accesslog.go:27      request completed       {"service": "auth", "request_id": "86bc13cde0bb4a4e", "method": "POST", "path": "/v1/auth/login", "status": 200, "duration_ms": 0, "remote_ip": "[::1]:56357", "user_agent": "PostmanRuntime/7.52.0"}

### Логи tasks
2026-03-18T18:28:51.281+0300    ←[34mINFO←[0m   authgrpc/client.go:76   auth response   {"service": "tasks", "request_id": "test-log-001", "component": "auth_client", "valid": true, "subject": "student", "duration_ms": 0.01387}
2026-03-18T18:28:51.281+0300    ←[34mINFO←[0m   middleware/auth.go:68   token validated {"service": "tasks", "request_id": "test-log-001", "component": "auth_middleware", "subject": "student"}
2026-03-18T18:28:51.281+0300    ←[34mINFO←[0m   handler/tasks.go:45     task created    {"service": "tasks", "request_id": "test-log-001", "handler": "CreateTask", "task_id": "t_177384", "title": "Логирование"}
2026-03-18T18:28:51.283+0300    ←[34mINFO←[0m   middleware/accesslog.go:27      request completed       {"service": "tasks", "request_id": "test-log-001", "method": "POST", "path": "/v1/tasks", "status": 201, "duration_ms": 15, "remote_ip": "[::1]:56288", "user_agent": "PostmanRuntime/7.52.0"}

## Запрос без токена
<img width="628" height="331" alt="image" src="https://github.com/user-attachments/assets/57bd6645-07de-4567-a894-bd68afdc35c3" />

### Логи auth
2026-03-18T18:29:55.557+0300    ←[33mWARN←[0m   grpc/server.go:50       gRPC: invalid token     {"service": "auth"}

### Логи tasks
2026-03-18T18:29:55.557+0300    ←[34mINFO←[0m   authgrpc/client.go:76   auth response   {"service": "tasks", "request_id": "test-log-002", "component": "auth_client", "valid": false, "subject": "", "duration_ms": 0}
2026-03-18T18:29:55.558+0300    ←[33mWARN←[0m   middleware/auth.go:50   invalid token   {"service": "tasks", "request_id": "test-log-002", "component": "auth_middleware"}
2026-03-18T18:29:55.558+0300    ←[34mINFO←[0m   middleware/accesslog.go:27      request completed       {"service": "tasks", "request_id": "test-log-002", "method": "GET", "path": "/v1/tasks", "status": 401, "duration_ms": 0, "remote_ip": "[::1]:56288", "user_agent": "PostmanRuntime/7.52.0"}

## Auth остановлен
<img width="441" height="325" alt="image" src="https://github.com/user-attachments/assets/a55462f2-3db7-4403-a986-f036ce602939" />

### Логи tasks
2026-03-18T18:31:23.200+0300    ←[31mERROR←[0m  authgrpc/client.go:65   auth unavailable        {"service": "tasks", "request_id": "test-log-004", "component": "auth_client", "duration_ms": 0.0005144}


# Ответы на вопросы
1.	Почему структурированные логи удобнее строковых?
Структурированные логи удобнее, потому что они представляют данные в машиночитаемом формате (например, JSON), что позволяет их легко фильтровать, искать и анализировать с помощью специализированных инструментов, в отличие от простого текста.
2.	Что такое request-id и как он помогает при диагностике?
Request-ID — это уникальный идентификатор запроса, который пробрасывается через все сервисы, позволяя связать воедино все события и логи, относящиеся к одному пользовательскому запросу, что критически упрощает отладку распределенных систем.
3.	Какие поля вы считаете обязательными для access log?
Обязательные поля для access log: request_id, method, path, status, duration_ms, remote_ip, user_agent.
4.	Почему нельзя писать токены и пароли в логи?
Нельзя писать токены и пароли в логи, потому что это прямая утечка секретных данных и угроза безопасности, которая может привести к компрометации учетных записей и системы в целом.
5.	Что логировать в ERROR, а что в INFO/WARN?
В ERROR логируются критическое проблемы, требующие вмешательства (ошибки БД, недоступность сервисов), а в INFO/WARN — штатные события (запросы, валидации) и нештатные ситуации, которые не ломают систему, но важны для мониторинга.
