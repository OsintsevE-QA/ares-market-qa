# Тестовые данные: Authentication and profile (секция 5.1)

## Scope

Покрытие секции 5.1 `TECHNICAL_REQUIREMENTS.md` v1.1: регистрация, логин, профиль, logout, валидация токена, отображение роли.
Связанные требования: FR-AUTH-01 - FR-AUTH-09.
Включены проверки профиля (`/api/auth/me`, FR-AUTH-08), logout (FR-AUTH-07) и валидации токена (FR-AUTH-06) — все они входят в секцию 5.1 «Authentication and profile».

В область не входят: инъекции, XSS, SQLi, обход авторизации, матрица доступа других секций, локализация, корзина, заказы, отзывы, feedback, bug reports, admin-разделы.

## Легенда типов данных

- `valid` - данные, которые по документации должны приниматься системой.
- `invalid` - данные, которые по документации должны отклоняться.
- `boundary` - граничные значения, проверяющие края допустимого диапазона.

---

## Блок 1. Регистрация - UI

Форма регистрации содержит поля: email, username, full_name (опционально), colony (select), password.

| ID | Проверяемое поле/сценарий | Тестовые данные | Тип | Ожидаемый результат по документации | Источник/FR | Примечание |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TD-AUTH-REG-UI-001 | Валидная регистрация, все поля заполнены | email: `lyra@valles.mars`, username: `lyra_valles`, full_name: `Lyra Okafor`, colony: `Valles Base`, password: `mars2026` | valid | Пользователь зарегистрирован, UI сообщает об успехе и перенаправляет на вход | FR-AUTH-01 | Требуется проверка в БД: запись в таблице users |
| TD-AUTH-REG-UI-002 | Валидная регистрация без опциональных полей | email: `kai@phobos.mars`, username: `kai_phobos`, full_name: (пусто), colony: `Phobos Dock`, password: `dock42pass` | valid | Пользователь зарегистрирован | FR-AUTH-01 | full_name опционален по FR-AUTH-01 |
| TD-AUTH-REG-UI-003 | Пустое поле email | email: (пусто), username: `test003`, password: `pass1234` | invalid | Форма не отправляется / отображается ошибка обязательного поля | FR-AUTH-01 | email указан как обязательный в FR-AUTH-01 |
| TD-AUTH-REG-UI-004 | Пустое поле username | email: `test004@ares.mars`, username: (пусто), password: `pass1234` | invalid | Форма не отправляется / отображается ошибка обязательного поля | FR-AUTH-01 | username указан как обязательный в FR-AUTH-01 |
| TD-AUTH-REG-UI-005 | Пустое поле password | email: `test005@ares.mars`, username: `test005`, password: (пусто) | invalid | Форма не отправляется / отображается ошибка обязательного поля | FR-AUTH-01 | password указан как обязательный в FR-AUTH-01 |
| TD-AUTH-REG-UI-006 | Пароль ровно 4 символа (нижняя граница) | email: `test006@ares.mars`, username: `test006`, password: `abcd` | boundary | Регистрация успешна | FR-AUTH-03 | "at least 4 characters" |
| TD-AUTH-REG-UI-007 | Пароль 3 символа (ниже границы) | email: `test007@ares.mars`, username: `test007`, password: `abc` | boundary | Ошибка: пароль слишком короткий | FR-AUTH-03 | |
| TD-AUTH-REG-UI-008 | Дубликат email (существующий `nova@aresmarket.mars`) | email: `nova@aresmarket.mars`, username: `unique_nova_copy`, password: `pass1234` | invalid | Четкая ошибка о том, что email уже занят | FR-AUTH-02 | Требуется проверка в БД: второй пользователь не создан |
| TD-AUTH-REG-UI-009 | Дубликат username (существующий `nova`) | email: `unique@ares.mars`, username: `nova`, password: `pass1234` | invalid | Четкая ошибка о том, что username занят | FR-AUTH-02 | Требуется проверка в БД: второй пользователь не создан |
| TD-AUTH-REG-UI-010 | Невалидный формат email (без @) | email: `not-an-email`, username: `test010`, password: `pass1234` | invalid | Ошибка валидации: email не соответствует базовому формату (отсутствует символ `@` и/или доменная часть). (Общепринятая практика: проверка наличия `@` и домена с точкой.) | FR-AUTH-01 | Документация явно не фиксирует формат, но базовая валидация email - общепринятое ожидание для любого поля с названием "email" |
| TD-AUTH-REG-UI-011 | Невалидный формат email (без домена) | email: `user@`, username: `test011`, password: `pass1234` | invalid | Ошибка валидации: email не содержит доменной части. | FR-AUTH-01 | Общепринятая практика |
| TD-AUTH-REG-UI-012 | Невалидный формат email (только локальная часть) | email: `justtext`, username: `test012`, password: `pass1234` | invalid | Ошибка валидации: email не содержит `@` и домена. | FR-AUTH-01 | Общепринятая практика |
| TD-AUTH-REG-UI-013 | Колония из списка UI (Olympus City) | email: `selene@olympus.mars`, username: `selene_olympus`, full_name: (пусто), colony: `Olympus City`, password: `olympus77` | valid | Регистрация успешна | FR-AUTH-01 | Уникальные email/username, чтобы не пересекаться с TD-AUTH-REG-UI-001/002. Значение `colony` — из списка на форме |
| TD-AUTH-REG-UI-014 | Пароль ровно 128 символов (верхняя граница) | email: `test014@ares.mars`, username: `test014`, password: `a` × 128 | boundary | Требуется уточнение (общепринятая практика: регистрация успешна, верхняя граница пароля обычно 128 символов) | FR-AUTH-03 | В документации верхняя граница длины пароля не задана |
| TD-AUTH-REG-UI-015 | Пароль 129 символов (превышение верхней границы) | email: `test015@ares.mars`, username: `test015`, password: `a` × 129 | boundary | Требуется уточнение (общепринятая практика: отклонение с ошибкой валидации) | FR-AUTH-03 | |
| TD-AUTH-REG-UI-016 | Username ровно 50 символов (верхняя граница) | email: `test016@ares.mars`, username: `u` × 50, password: `pass1234` | boundary | Требуется уточнение (общепринятая практика: регистрация успешна, верхняя граница username обычно 50 символов) | FR-AUTH-01 | В документации длина username не задана |
| TD-AUTH-REG-UI-017 | Username 51 символ (превышение верхней границы) | email: `test017@ares.mars`, username: `u` × 51, password: `pass1234` | boundary | Требуется уточнение (общепринятая практика: отклонение с ошибкой валидации) | FR-AUTH-01 | |
| TD-AUTH-REG-UI-018 | Email ровно 254 символа (верхняя граница) | email: `t` × 64 + `@` + `d` × 184 + `.mars`, username: `test018`, password: `pass1234` | boundary | Требуется уточнение (общепринятая практика: регистрация успешна, максимальная длина email 254 символа) | FR-AUTH-01 | |
| TD-AUTH-REG-UI-019 | Email 255 символов (превышение верхней границы) | email: `t` × 64 + `@` + `d` × 185 + `.mars`, username: `test019`, password: `pass1234` | boundary | Требуется уточнение (общепринятая практика: отклонение с ошибкой валидации) | FR-AUTH-01 | |

---

## Блок 2. Регистрация - API

Метод: `POST /api/auth/register`
Формат тела: `application/json`
Поля по документации: email, username, password (обязательные); full_name — опционален (FR-AUTH-01). Обязательность `colony` по FR-AUTH-01 явно не зафиксирована — см. «Требуется уточнение», п. 8.

| ID | Проверяемое поле/сценарий | Тестовые данные | Тип | Ожидаемый результат по документации | Источник/FR | Примечание |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TD-AUTH-REG-API-001 | Валидный JSON, все поля | `{"email":"rex@hellas.mars","username":"rex_hellas","full_name":"Rex Tanaka","password":"hellas909","colony":"Hellas Outpost"}` | valid | HTTP 200/201, в ответе данные пользователя | FR-AUTH-01 | Требуется проверка в БД: запись создана |
| TD-AUTH-REG-API-002 | JSON без `full_name` и без `colony` | `{"email":"io@ares.mars","username":"io_colonist","password":"io2026"}` | boundary | Требуется уточнение: `full_name` опционален по FR-AUTH-01, обязательность `colony` не зафиксирована. Если `colony` необязателен — HTTP 200/201; если обязателен — HTTP 400/422 | FR-AUTH-01 | `full_name` опционален по FR-AUTH-01. По `colony` — см. «Требуется уточнение», п. 8 |
| TD-AUTH-REG-API-003 | Отсутствует email | `{"username":"api_test003","password":"pass1234"}` | invalid | HTTP 400/422, ошибка валидации | FR-AUTH-01 | |
| TD-AUTH-REG-API-004 | Отсутствует username | `{"email":"t4@ares.mars","password":"pass1234"}` | invalid | HTTP 400/422 | FR-AUTH-01 | |
| TD-AUTH-REG-API-005 | Отсутствует password | `{"email":"t5@ares.mars","username":"api_test005"}` | invalid | HTTP 400/422 | FR-AUTH-01 | |
| TD-AUTH-REG-API-006 | Пароль ровно 4 символа | `{"email":"api006@ares.mars","username":"api_test006","password":"abcd"}` | boundary | HTTP 200/201 | FR-AUTH-03 | |
| TD-AUTH-REG-API-007 | Пароль 3 символа | `{"email":"t7@ares.mars","username":"api_test007","password":"abc"}` | boundary | HTTP 400/422, ошибка длины пароля | FR-AUTH-03 | |
| TD-AUTH-REG-API-008 | Дубликат email | `{"email":"nova@aresmarket.mars","username":"unique_xyz","password":"pass1234"}` | invalid | HTTP 400, четкая ошибка "email already registered" (или аналогичная по смыслу) | FR-AUTH-02 | Требуется проверка в БД: новая запись не создана |
| TD-AUTH-REG-API-009 | Дубликат username | `{"email":"unique_api@ares.mars","username":"nova","password":"pass1234"}` | invalid | HTTP 400, четкая ошибка "username already taken" (или аналогичная) | FR-AUTH-02 | Требуется проверка в БД: новая запись не создана |
| TD-AUTH-REG-API-010 | Невалидный формат email (без @) | `{"email":"not-an-email-api","username":"api_test010","password":"pass1234"}` | invalid | HTTP 400/422, ошибка валидации email. (Общепринятая практика: проверка наличия `@` и домена.) | FR-AUTH-01 | Документация не фиксирует формат явно, но базовая проверка email - ожидаемое поведение для поля с названием "email" |
| TD-AUTH-REG-API-011 | Невалидный формат email (без домена) | `{"email":"user-api@","username":"api_test011","password":"pass1234"}` | invalid | HTTP 400/422, ошибка валидации email. | FR-AUTH-01 | Общепринятая практика |
| TD-AUTH-REG-API-012 | Пустое тело запроса | `{}` | invalid | HTTP 400/422 | FR-AUTH-01 | |
| TD-AUTH-REG-API-013 | Не-JSON Content-Type | form-urlencoded вместо JSON | invalid | Требуется уточнение (документация не фиксирует поведение при неверном Content-Type для этого эндпоинта) | FR-AUTH-01 | |
| TD-AUTH-REG-API-014 | Пустая строка email | `{"email":"","username":"api_test014","password":"pass1234"}` | invalid | Требуется уточнение (общепринятая практика: HTTP 400/422, ошибка "field cannot be empty") | FR-AUTH-01 | В документации non-empty для email не зафиксировано явно |
| TD-AUTH-REG-API-015 | Пустая строка username | `{"email":"t15@ares.mars","username":"","password":"pass1234"}` | invalid | Требуется уточнение (общепринятая практика: HTTP 400/422, ошибка "field cannot be empty") | FR-AUTH-01 | |
| TD-AUTH-REG-API-016 | Пустая строка password | `{"email":"t@ares.mars","username":"test016","password":""}` | invalid | HTTP 400, ошибка длины пароля | FR-AUTH-01, FR-AUTH-03 | Пустой `password` — 0 символов, что прямо нарушает FR-AUTH-03 («at least 4 characters») |
| TD-AUTH-REG-API-017 | Пароль ровно 128 символов (верхняя граница) | `{"email":"t17@ares.mars","username":"test017","password":"a"*128}` | boundary | Требуется уточнение (общепринятая практика: HTTP 200/201, регистрация успешна) | FR-AUTH-03 | Верхняя граница длины пароля в документации не задана |
| TD-AUTH-REG-API-018 | Пароль 129 символов (превышение верхней границы) | `{"email":"t18@ares.mars","username":"api_test018","password":"a"*129}` | boundary | Требуется уточнение (общепринятая практика: HTTP 400/422, ошибка длины) | FR-AUTH-03 | |
| TD-AUTH-REG-API-019 | Username ровно 50 символов (верхняя граница) | `{"email":"t19@ares.mars","username":"v"*50,"password":"pass1234"}` | boundary | Требуется уточнение (общепринятая практика: HTTP 200/201) | FR-AUTH-01 | Длина username в документации не задана |
| TD-AUTH-REG-API-020 | Username 51 символ (превышение верхней границы) | `{"email":"t20@ares.mars","username":"v"*51,"password":"pass1234"}` | boundary | Требуется уточнение (общепринятая практика: HTTP 400/422) | FR-AUTH-01 | |
| TD-AUTH-REG-API-021 | Email ровно 254 символа (верхняя граница) | `{"email":"s"*64+"@"+"d"*184+".mars","username":"test021","password":"pass1234"}` | boundary | Требуется уточнение (общепринятая практика: HTTP 200/201) | FR-AUTH-01 | |
| TD-AUTH-REG-API-022 | Email 255 символов (превышение верхней границы) | `{"email":"s"*64+"@"+"d"*185+".mars","username":"test022","password":"pass1234"}` | boundary | Требуется уточнение (общепринятая практика: HTTP 400/422) | FR-AUTH-01 | |

---

## Блок 3. Логин - UI

Форма входа: username, password.

| ID | Проверяемое поле/сценарий | Тестовые данные | Тип | Ожидаемый результат по документации | Источник/FR | Примечание |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TD-AUTH-LOGIN-UI-001 | Валидный вход colonist | username: `nova`, password: `nova123` | valid | Успешный вход, UI показывает username и badge `colonist` | FR-AUTH-04, FR-AUTH-09 | |
| TD-AUTH-LOGIN-UI-002 | Валидный вход admin | username: `admin`, password: `admin123` | valid | Успешный вход, UI показывает username и badge `admin` | FR-AUTH-04, FR-AUTH-09 | |
| TD-AUTH-LOGIN-UI-003 | Валидный вход под свежезарегистрированным (пароль 4 символа) | username: `test006`, password: `abcd` | boundary | Успешный вход | FR-AUTH-04, FR-AUTH-03 | **Предусловие**: TD-AUTH-REG-UI-006 выполнен успешно и пользователь `test006` создан. Если TD-AUTH-REG-UI-006 не создал пользователя, кейс блокируется и требует явной подготовки данных |
| TD-AUTH-LOGIN-UI-004 | Неверный пароль при верном username | username: `nova`, password: `wrongpass` | invalid | Ошибка входа, текст не раскрывает, что именно неверно | FR-AUTH-05 | |
| TD-AUTH-LOGIN-UI-005 | Несуществующий username | username: `ghost_user`, password: `any1234` | invalid | Ошибка входа, текст не раскрывает, что username не существует | FR-AUTH-05 | Текст ошибки должен совпадать по смыслу с TD-AUTH-LOGIN-UI-004 |
| TD-AUTH-LOGIN-UI-006 | Пустой username | username: (пусто), password: `nova123` | invalid | Форма не отправляется / ошибка обязательного поля | FR-AUTH-04 | |
| TD-AUTH-LOGIN-UI-007 | Пустой password | username: `nova`, password: (пусто) | invalid | Форма не отправляется / ошибка обязательного поля | FR-AUTH-04 | |
| TD-AUTH-LOGIN-UI-008 | Logout (клиентская часть) | Предусловие: выполнен вход как `nova`. Действие: нажать «Выход» | valid | UI отбрасывает токен (FR-AUTH-07). (Общепринятая практика: возврат на главную, исчезновение badge роли, замена меню на «Вход»/«Регистрация» — FR-AUTH-09) | FR-AUTH-07, FR-AUTH-09 | Документация явно фиксирует только «client discards token». Остальные ожидания — общепринятая практика, требуют уточнения при прогоне |
| TD-AUTH-LOGIN-UI-009 | Повторный вход после logout | Предусловие: выполнен logout. Действие: вход как `nova` / `nova123` | valid | Успешный вход | FR-AUTH-04, FR-AUTH-07 | |

---

## Блок 4. Логин - API и связанные проверки (токен, профиль, logout)

Метод логина: `POST /api/auth/login`
Формат тела: `application/x-www-form-urlencoded` (по документации, раздел 6)
Поля: `username`, `password`.

| ID | Проверяемое поле/сценарий | Тестовые данные | Тип | Ожидаемый результат по документации | Источник/FR | Примечание |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TD-AUTH-LOGIN-API-001 | Валидный логин colonist | form-urlencoded: `username=nova&password=nova123` | valid | HTTP 200, в ответе `access_token`, `token_type: bearer`, объект `user` с полем `role` | FR-AUTH-04 | |
| TD-AUTH-LOGIN-API-002 | Валидный логин admin | form-urlencoded: `username=admin&password=admin123` | valid | HTTP 200, `user.role == "admin"` | FR-AUTH-04 | |
| TD-AUTH-LOGIN-API-003 | Неверный пароль | `username=nova&password=wrong` | invalid | HTTP 401, текст ошибки не раскрывает, что именно неверно | FR-AUTH-05 | |
| TD-AUTH-LOGIN-API-004 | Несуществующий username | `username=ghost&password=any` | invalid | HTTP 401, текст ошибки совпадает по смыслу с TD-AUTH-LOGIN-API-003 | FR-AUTH-05 | |
| TD-AUTH-LOGIN-API-005 | Отсутствует password | `username=nova` | invalid | HTTP 400/422 | FR-AUTH-04 | |
| TD-AUTH-LOGIN-API-006 | Отсутствует username | `password=nova123` | invalid | HTTP 400/422 | FR-AUTH-04 | |
| TD-AUTH-LOGIN-API-007 | Проверка `/api/auth/me` с валидным токеном | Предусловие: получен токен из TD-AUTH-LOGIN-API-001. Запрос: `GET /api/auth/me` с `Authorization: Bearer <token>` | valid | HTTP 200, в ответе профиль пользователя, включая поле `role` | FR-AUTH-08 | |
| TD-AUTH-LOGIN-API-007a | Проверка `/api/auth/me` с токеном admin | Предусловие: получен токен из TD-AUTH-LOGIN-API-002. Запрос: `GET /api/auth/me` с `Authorization: Bearer <token>` | valid | HTTP 200, в ответе профиль admin, `user.role == "admin"` | FR-AUTH-08 | Дополняет TD-AUTH-LOGIN-API-007 (colonist), покрывает FR-AUTH-08 для роли admin |
| TD-AUTH-LOGIN-API-008 | `/api/auth/me` без токена | `GET /api/auth/me` без заголовка Authorization | invalid | HTTP 401 | FR-AUTH-06 | "Expired or invalid tokens shall result in HTTP 401" |
| TD-AUTH-LOGIN-API-009 | `/api/auth/me` с невалидным токеном | `Authorization: Bearer garbage_token_string` | invalid | HTTP 401 | FR-AUTH-06 | |
| TD-AUTH-LOGIN-API-010 | `/api/auth/me` с просроченным токеном | Токен с истёкшим `exp` | invalid | HTTP 401 | FR-AUTH-06 | **Требуется уточнение**: TTL токена в документации не задан, «дождаться истечения» нереализуемо как тест-кейс. Способ получения просроченного токена (ручная генерация JWT с `exp` в прошлом или настройка короткого TTL) должен быть согласован отдельно. См. «Требуется уточнение», п. 9 |
| TD-AUTH-LOGIN-API-011 | Logout - серверная часть | Действие: клиент отбрасывает токен после logout (из TD-AUTH-LOGIN-UI-008). Проверка: повторный вызов `/api/auth/me` со старым токеном | boundary | Требуется уточнение: документация фиксирует только клиентскую часть logout («client discards token»). Поведение сервера при повторном использовании старого токена не описано — возможны оба исхода: HTTP 200 (токен действует до истечения `exp`) или HTTP 401 (сервер инвалидирует) | FR-AUTH-07 | См. «Требуется уточнение», п. 1 |
| TD-AUTH-LOGIN-API-012 | Пустая строка username | form-urlencoded: `username=&password=nova123` | invalid | Требуется уточнение (общепринятая практика: HTTP 400/422, ошибка "field cannot be empty") | FR-AUTH-04 | В документации non-empty для username не зафиксировано явно |
| TD-AUTH-LOGIN-API-013 | Пустая строка password | form-urlencoded: `username=nova&password=` | invalid | Требуется уточнение (общепринятая практика: HTTP 400/422, ошибка "field cannot be empty") | FR-AUTH-04 | |
| TD-AUTH-LOGIN-API-014 | Неверный Content-Type (JSON вместо form-urlencoded) | `POST /api/auth/login` с заголовком `Content-Type: application/json`, тело `{"username":"nova","password":"nova123"}` | invalid | Требуется уточнение: раздел 6 фиксирует формат `application/x-www-form-urlencoded` для login, но не описывает реакцию на неверный Content-Type. (Общепринятая практика: HTTP 415 Unsupported Media Type или HTTP 422) | FR-AUTH-04 | Аналог TD-AUTH-REG-API-013 для регистрации. См. «Требуется уточнение», п. 2 |

---

## Покрытие требований FR-AUTH-*

| Требование | IDs тестовых данных | Статус покрытия |
| :--- | :--- | :--- |
| FR-AUTH-01 (регистрация с полями) | TD-AUTH-REG-UI-001, 002, 003, 004, 005, 010, 011, 012, 013, 016, 017, 018, 019; TD-AUTH-REG-API-001, 002, 003, 004, 005, 010, 011, 012, 014, 015, 019, 020, 021, 022 | Покрыто |
| FR-AUTH-02 (уникальность email/username) | TD-AUTH-REG-UI-008, 009; TD-AUTH-REG-API-008, 009 | Покрыто |
| FR-AUTH-03 (пароль >= 4 символа) | TD-AUTH-REG-UI-006, 007, 014, 015; TD-AUTH-REG-API-006, 007, 016, 017, 018; TD-AUTH-LOGIN-UI-003 | Частично: нижняя граница покрыта, верхняя - "Требуется уточнение" |
| FR-AUTH-04 (логин + JWT) | TD-AUTH-LOGIN-UI-001, 002, 006, 007, 009; TD-AUTH-LOGIN-API-001, 002, 005, 006, 012, 013, 014 | Покрыто |
| FR-AUTH-05 (не раскрывать что неверно) | TD-AUTH-LOGIN-UI-004, 005; TD-AUTH-LOGIN-API-003, 004 | Покрыто |
| FR-AUTH-06 (токен -> 401) | TD-AUTH-LOGIN-API-008, 009, 010 | Покрыто |
| FR-AUTH-07 (logout) | TD-AUTH-LOGIN-UI-008, 009; TD-AUTH-LOGIN-API-011 | Частично: клиентская часть покрыта, серверная - "Требуется уточнение" |
| FR-AUTH-08 (`/api/auth/me` с role) | TD-AUTH-LOGIN-API-007, 007a | Покрыто (colonist и admin) |
| FR-AUTH-09 (UI badge роли) | TD-AUTH-LOGIN-UI-001, 002, 008 | Покрыто |

---

## Требуется уточнение

1. **Поведение сервера после logout.** FR-AUTH-07 описывает только клиентскую часть ("client discards token"). Остается ли старый токен валидным на стороне сервера до истечения `exp` - не зафиксировано. Строка TD-AUTH-LOGIN-API-011.
2. **Поведение при неверном Content-Type на `/api/auth/register` и `/api/auth/login`.** Документация фиксирует JSON для register и form-urlencoded для login, но не описывает реакцию на неверный Content-Type. Строки TD-AUTH-REG-API-013 и TD-AUTH-LOGIN-API-014.
3. **Точные тексты ошибок для дубликатов.** FR-BUG-02 требует "clear error", но конкретные формулировки не заданы. В таблицах указаны ожидаемые по смыслу ("email already registered" / "username already taken") - требуют сверки с реальной реализацией при прогоне.
4. **Верхняя граница длины пароля.** FR-AUTH-03 задаёт минимум 4 символа, но не задаёт максимум. Строки TD-AUTH-REG-UI-014, 015 и TD-AUTH-REG-API-017, 018.
5. **Верхняя граница длины username.** Документация не задаёт длину username. Строки TD-AUTH-REG-UI-016, 017 и TD-AUTH-REG-API-019, 020.
6. **Верхняя граница длины email.** Документация не задаёт длину email. Строки TD-AUTH-REG-UI-018, 019 и TD-AUTH-REG-API-021, 022.
7. **Обработка пустых строк (не отсутствующих полей) в API.** Документация фиксирует обязательность email/username/password, но не уточняет, считается ли пустая строка `""` валидным значением. Строки TD-AUTH-REG-API-014, 015 и TD-AUTH-LOGIN-API-012, 013.
8. **Обязательность поля `colony` при регистрации.** FR-AUTH-01 перечисляет `colony` вместе с обязательными полями, а слово «optional» относится только к `full_name`. В документации нет явного требования «colony обязателен» и нет явного «colony опционален». Строка TD-AUTH-REG-API-002.
9. **TTL токена и способ получения просроченного токена.** FR-AUTH-06 требует «Expired tokens shall result in HTTP 401», но срок жизни токена (`exp`) в документации не задан. Как подготовить просроченный токен для теста — не зафиксировано. Строка TD-AUTH-LOGIN-API-010.

---

## Требуется проверка в БД

1. **TD-AUTH-REG-UI-001, TD-AUTH-REG-API-001** - после успешной регистрации проверить, что в таблице `users` появилась запись с соответствующими `email`, `username`, `role` (по умолчанию `colonist`), `colony`.
2. **TD-AUTH-REG-UI-008, TD-AUTH-REG-API-008** - после попытки регистрации с дубликатом email убедиться, что количество записей в `users` не изменилось.
3. **TD-AUTH-REG-UI-009, TD-AUTH-REG-API-009** - аналогично для дубликата username.
4. **TD-AUTH-REG-UI-007, TD-AUTH-REG-API-007** - после отклонения короткого пароля убедиться, что запись в `users` не создана.