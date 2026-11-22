Спецификация REST API:
| Метод и адрес | Роли | Тело запроса | Успешный ответ | Ошибки |
| :--- | :--- | :--- | :--- | :--- |
| POST /api/auth/login | все | LoginRequestDto | 200, LoginResponseDto | 400, 401, 429 |
| GET /api/clients?search= | вошедшие | — | 200, список ClientDto | 401 |
| GET /api/clients/{id} | вошедшие | — | 200, ClientDto | 401, 404 |
| POST /api/clients | Admin, Operator | CreateClientDto | 201, ClientDto | 400, 401, 403 |
| GET /api/orders?status=&courierId=&search= | вошедшие | — | 200, список OrderDto | 401 |
| GET /api/orders/{id} | вошедшие | — | 200, OrderDto | 401, 404 |
| POST /api/orders | Admin, Operator | CreateOrderDto | 201, OrderDto | 400, 401, 403, 404 |
| PUT /api/orders/{id}/status | вошедшие (повар/курьер — только свои) | ChangeStatusDto | 200, OrderDto | 400, 401, 403, 404 |
| PUT /api/orders/{id}/courier | Admin, Operator | AssignCourierDto | 200, OrderDto | 400, 401, 403, 404 |
| GET /api/orders/statistics?courierId= | вошедшие | — | 200, RequestStatisticsDto | 401 |
| GET /api/users/couriers | Admin, Operator | — | 200, список UserDto | 401, 403 |
| GET /api/users, POST /api/users | Admin | CreateUserDto | 200 / 201 | 400, 401, 403 |
| POST /api/exchange/menu/import | Admin, Operator | ImportCsvDto | 200, ImportResultDto | 400, 401, 403 |
| GET /api/exchange/orders/export?format= | Admin, Operator | — | 200, файл csv / json / xml | 400, 401, 403 |
| GET /health | все | — | 200, «Healthy» | 503 |


Пример сообщения. Запрос на создание заказа:
POST /api/orders
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1Ni...

{
  "clientId": 1,
  "deliveryAddress": "ул. Ленина, д. 10, кв. 5",
  "items": [
    { "dishId": 12, "quantity": 2 },
    { "dishId": 5, "quantity": 1 }
  ],
  "comment": "Без лука, пожалуйста"
}


Ответ с ошибкой бизнес-правила (формат ProblemDetails):
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "title": "Нарушено бизнес-правило",
  "status": 400,
  "detail": "Нельзя перевести заказ из статуса «Готов к доставке» в статус «Доставлен» без назначения курьера.",
  "instance": "/api/orders/6/status"
}


Соглашения формата данных:
- Имена свойств JSON — camelCase.
- Перечисления — строками ("status": "ReadyForDelivery").
- Даты — ISO 8601 в UTC ("2026-09-16T18:00:00Z").
- Дробные числа — с точкой.
- Кодировка — UTF-8.

Импорт меню (блюд) из CSV:

| Колонка CSV | Поле Dish | Правило преобразования | Ошибка, если |
| :--- | :--- | :--- | :--- |
| Название | Name | обрезка пробелов | короче 3 символов |
| Цена | Price | только цифры и точка, приведение к decimal | меньше или равно 0 |
| Категория | Category | пустое значение -> "Разное" | - |


Требования к файлу: 
первая строка — заголовок Название;Цена;Категория, разделитель «;», кодировка UTF-8 (допускается Windows-1251 из Excel).
Если блюдо с таким названием уже есть в базе — обновляется его цена и категория.

Форматы выгрузки заказов:
- CSV (UTF-8 с BOM, чтобы Excel правильно открыл кириллицу; разделитель «;»; стоимость с точкой).
- JSON (массив объектов).
- XML (корень <Orders>, элементы <Order id="...">).