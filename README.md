# NotifyHub
Мультиарендная дырявая SaaS для управления уведомлениями. Используется в учебных целях CI/CD, AppSec и немного 
разработки.

Пользователь создаёт "кампании", шаблоны сообщений, список получателей, webhook endpoints и API keys; затем запускает 
тестовую отправку и смотрит историю delivery attempts.

## Цели проекта
1. Демонстрация пути **код → CI/CD → сканирование → деплой**
2. Отработка на практике:
   1. Внедрение SAST, SCA, DAST, сканирование секретов и контейнеров.
   2. Работа с OWASP API Security Top 10 - создание и устранение уязвимостей
   3. Docker-харденинг, k8s (RBAC, NetworkPolicy), IaC, мониторинг и логирование.

## Используемый стек
- Backend: Python/FastAPI
- БД: PostgreSQL
- ORM: SQLAlchemy
- Обработчик очередей: Redis
- Docker Compose

## Компоненты
- `apps/api` - FastAPI API, HTTP-эндпоинты, аутентификация/авторизация.
- `apps/worker` - фоновая обработка отправок
- `db` - миграции (Alembic), схема (SQLAlchemy).
- `infra/` - Docker, k8s манифесты.
- `docs/` - вся документация по проекту от ТЗ и модели угроз до 

## Потоковая диаграмма 

```mermaid
flowchart TD
    A([User / Client]) --> B[POST /auth/login]
    B --> C{Credentials valid?}

    C -- No --> C1[401 Unauthorized<br/>audit: login_failed]
    C -- Yes --> D[Issue access token]
    D --> E[Issue refresh token]
    E --> F[Create session / store refresh token hash]
    F --> G[Client stores tokens]
    G --> H[Call protected API]

    H --> I{Access token valid?}
    I -- Yes --> J[Return protected resource]
    I -- No --> K[POST /auth/refresh]

    K --> L{Refresh token valid<br/>and active session?}
    L -- No --> L1[401 Unauthorized<br/>force re-login]
    L -- Yes --> M[Rotate refresh token]
    M --> N[Invalidate old refresh token]
    N --> O[Issue new access + refresh]
    O --> H

    J --> P[Create test-send / launch campaign]
    P --> Q[Queue delivery job]
    Q --> R[Worker sends webhook]
    R --> S[Sign payload with secret + timestamp]
    S --> T[POST webhook to target]

    T --> U{HTTP response}
    U -- 2xx --> V[Mark delivery success]
    U -- 4xx --> W[Mark permanent failure]
    U -- 5xx / timeout --> X[Retry with backoff]
    X --> Y{Retries exhausted?}
    Y -- No --> R
    Y -- Yes --> Z[Move to DLQ / failed deliveries]

    T --> AA[Receiver verifies signature]
    AA --> AB{Signature valid?}
    AB -- No --> AC[401/403 reject webhook]
    AB -- Yes --> AD{Timestamp fresh?}
    AD -- No --> AE[Reject replay / expired request]
    AD -- Yes --> AF{Event already processed?}
    AF -- Yes --> AG[Return 200 idempotent ack]
    AF -- No --> AH[Process event]
    AH --> AI[Store event result + audit/security log]
```

## Итеративный подход
Проект должен содержать три итерации:
1. Реализация основного функционала, но содержит демонстрационные уязвимости.
2. Добавление базового контроля безопасности, устранение ключевых уязвимостей.
3. Харденинг и безопасность платформы.

## О применении ИИ
В данном проекте ИИ используется для формирования и формализации общей идеи и спецификации технического задания. 
При этом, ИИ не используется для описания модели угроз, RBAC-матрицы, описания контроля безопасности.

ИИ не применяется для разработки исходного кода и пайплайнов для прозрачности демонстрации и честности самого обучения.
Исключение: дизайн и фронтенд, которые могут использоваться для демонстрации