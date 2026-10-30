# QA Portfolio: AresMarket

Репозиторий содержит артефакты обеспечения качества (QA) для учебного веб-приложения «Межпланетный маркетплейс» (AresMarket).

## Стек и инструменты
- **Ручное тестирование:** Markdown, GitHub Issues, Test Design.
- **API:** Postman, Swagger UI, JSON.
- **Автоматизация:** Python, Playwright, Pytest (в процессе).
- **CI/CD:** GitHub Actions (в процессе).
- **Окружение:** Windows 11, Docker, WSL2.

## Локальный запуск среды
Тестируемое приложение (AresMarket) запускается в Docker-контейнерах.
1. Склонировать [репозиторий приложения](https://github.com/OsintsevE-QA/ares-market).
2. Выполнить `docker compose up --build`.
3. UI: `http://localhost:8000` | Swagger: `http://localhost:8000/api/docs`.

## Структура репозитория
- `manual/` — чек-листы и тест-кейсы.
- `api/` — коллекции Postman и стратегия тестирования API.
- `auto/` — код автотестов.
- `defects/` — витрина найденных багов (Bug Log).
- `docs/` — тест-план, стратегия и troubleshooting.

## Трекинг дефектов
Все найденные баги регистрируются в [GitHub Issues](../../issues).
В папке `defects/` представлена витрина наиболее сложных и интересных дефектов.
Перейти к [Индексной таблице багов](defects/bug-log.md).