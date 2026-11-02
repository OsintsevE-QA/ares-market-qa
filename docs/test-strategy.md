# Test Strategy: AresMarket

## Цели тестирования

- Проверить соответствие поведения приложения `TECHNICAL_REQUIREMENTS.md` v1.1.
- Покрыть функциональные, UI, API, ролевые и локализационные сценарии.
- Найти и задокументировать дефекты для портфолио.

## Уровни тестирования

- Smoke — базовая работоспособность ключевых флоу.
- Functional — проверка каждого FR-* из требований.
- UI / UX — адаптив, верстка, поведение форм.
- API — прямые запросы через Swagger / Postman.
- Roles — матрица доступа guest / colonist / admin.
- Localization — RU / EN / Martian (включая 500 на `/api/locale/mars/crash`).
- Security (базовый) — JWT, 401/403, SQL-инъекции в формах.

## Объекты тестирования

- UI: http://localhost:8000
- API: http://localhost:8000/api/docs
- БД: SQLite, проверки описательные (см. `docs/db-checks.md`), прямые SQL-запросы не выполняются.

## Риски

- Намеренные дефекты в приложении могут маскироваться под архитектурные решения.
- Martian locale возвращает 200 с битыми строками и отдельно 500 на crash-эндпоинте — это два разных дефекта.
- JWT без сохранения состояния: после выхода из системы (logout) токен формально остаётся действительным до истечения срока его действия.

## Инструменты

- Ручное тестирование: Chrome DevTools, VS Code, Markdown.
- API: Swagger UI, Postman.
- Трекинг дефектов: GitHub Issues + витрина в `defects/`.
- Автоматизация (v2.0): Python, Playwright, Pytest.
- CI: GitHub Actions.

## Критерии завершения

- 100% покрытие critical path: AUTH, CAT, CART/ORD, LOC, BUG, FB.
- Матрица ролей: colonist получает 403 на admin API.
- Минимум 7 багов в витрине `defects/`.
- Test Report с residual risks.
  