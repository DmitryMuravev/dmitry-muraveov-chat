# Dmitry Muraveov Chat Project
***
## Инструкция по запуску.

### 1. Необходимое ПО.
1. Java Development Kit 11;
2. Apache Maven Framework;
3. Docker;
4. База данных Postgres (опционально).
### 2. Сборка проекта.
1. Скачать проект с сайта, либо склонировать его на локальную машину с использованием Git:
```
    git clone https://github.com/DmitryMuraveov/dmitry-muraveov-chat.git
```
2. Проект создан с использованием фреймворка Apache maven. Для сборки необходимо из 
корневой папки проекта выполнить в терминале команду:
```
    mvn clean package
```
После чего появится директория *target* с jar-файлом 'chat-1.0.0.jar'.

3. При необходимости можно создать Docker image сервиса командой:
```
    docker build .
```

### 3. Запуск проекта с использованием Docker
Для запуска проекта с использованием Docker достаточно выполнить следующие команды:
```
    docker compose build
    docker compose up
```
В результате будут запущены контейнеры с текущим сервисом и с базой данных Postgres. 
Доступ к сервису с локальной машины можно получить по адресу: http://localhost:8081.
### 4. Запуск проекта без использования Docker
Если есть необходимость запустить сервис непосредственно на локальной машине, 
необходимо выполнить следующие действия:
1. Установить Postgres и создать базу данных с которой будет работать сервис;
2. Из директории *target* выполнить команду:
```
    java -DDATABASE_URL=jdbc:postgresql://localhost:5432/chat_db -DDATABASE_USERNAME=postgres -DDATABASE_PASSWORD=postgres -jar chat-1.0.0.jar
```
Где:

- DATABASE_URL - адрес созданной базы данных;
- DATABASE_USERNAME - имя пользователя, под которым сервис будет выполнять операции;
- DATABASE_PASSWORD - пароль для указанного пользователя.

Также можно указать дополнительные параметры, список которых указан в CHANGELOG.md

## Инструкция по работе с сервисом.

Сервис предоставляет 3 эндпоинта:
- /register - регистрация пользователя; поддерживаемые методы: POST;
- /login - авторизация пользователя; поддерживаемые методы: POST;
- /messages - работа с сообщениями; поддерживаемые методы: POST, GET.

Примеры curl запросов можно найти в файле CurlRequests.txt 

### 1. POST /register
Эндпоинт для регистрации нового пользователя сервиса.

Структура запроса:
```json
{
  "login": "NewUserLogin",
  "password": "NewUserPassword_32131"
}
```
- login - имя пользователя, под которым он будет зарегистрирован;
- password - пароль от учётной записи пользователя.

При успешном выполнении сервис отправит пустой ответ со статусом "200 OK".

При возникновении ошибки сервис отправит ответ со следующей структурой:
```json
{
  "errorMessage": "User with login username already exists"
}
```
- errorMessage - Сообщение об ошибке.

### 1. POST /login
Эндпоинт для авторизации зарегистрированного пользователя сервиса.

Структура запроса:
```json
{
  "login": "NewUserLogin",
  "password": "NewUserPassword_32131"
}
```

- login - имя пользователя, под которым он зарегистрирован;
- password - пароль от учётной записи пользователя.

При успешном выполнении сервис вернёт ответ, содержащий токен доступа. Структура ответа:

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoicXdlIiwiaWF0IjoxNjY5NTc0NzEzLCJleHAiOjE2Njk1ODkxMTN9.aavD8Ie8ZCl8aXRR_tVKDnRa7h3egL1yVqxMP7a5fi8"
}
```

- token - токен доступа пользователя.

### 3. POST /messages
Эндпоинт для отправки нового сообщения от лица зарегистрированного пользователя. 
Доступен только при предоставлении токена в заголовке запроса *Authorization*. 
Формат заголовка: 
```
Authorization: Bearer YOUR_TOKEN
```

Структура запроса:
```json
{
  "message": "Hello!"
}
```

- message - содержимое отправляемого сообщения.

В результате сервис вернёт ответ со статусом "200 OK" и следующим содержимым:
```json
{
  "statusMessage": "Message send successfully"
}
```

- statusMessage - информация о статусе отправленного сообщения.

Если предоставленный токен невалиден, то пользователю вернётся пустой ответ со статусом "FORBIDDEN 403".

### 3. GET /messages
Эндпоинт для получения списка сообщений.
Доступен только при предоставлении токена в заголовке запроса *Authorization*.
Формат заголовка:
```
Authorization: Bearer YOUR_TOKEN
```

Информация по выборке сообщений содержится в параметрах запроса.
- page - номер страницы выборки, число;
- size - размер выборки, число;
- sort - информация по сортировке данных, (имя_поля,(asc|desc)).

Пример запроса на получение последних 10 сообщений:

```
    http://localhost:8081/messages?page=0&size=10&sort=createdAt,desc
```

В результате сервис вернёт ответ со статусом 200 следующего формата:
```json
{
  "messages": [
    {
      "createdAt": "2022-11-27T18:55:23.166Z",
      "message": "Hello!",
      "user": "username"
    }
  ]
}
```

- messages - список запрошенных сообщений;
- createdAt - дата отправки сообщения;
- message - содержимое сообщения;
- user - имя пользователя, отправившего сообщение.

Если предоставленный токен невалиден, то пользователю вернётся пустой ответ со статусом "FORBIDDEN 403".