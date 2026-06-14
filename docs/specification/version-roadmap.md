## Версионность и дорожная карта

NotifyHub развивается в три итерации, каждая — отдельная ветка/тэг:

- **v1 – Intentionally Vulnerable Baseline**
  - Рабочий API и базовый UI.
  - Намеренно оставлены уязвимости:
    - BOLA/IDOR на кампаниях (нет жёсткой привязки к `organization_id`).
    - Broken function-level auth на webhook management.
    - SSRF-риск в `POST /webhooks/validate-target`.
    - Нет rate limiting на auth и test-send.
    - Secrets в `.env.example` и простые dev-секреты в compose.
    - Одна уязвимая зависимость в requirements.
    - Небезопасный Dockerfile (root, лишние пакеты).
    - Болтливые ошибки.
  - Используется для демонстрации SAST/SCA/DAST и triage.

- **v2 – Remediation & Basic Controls**
  - Исправляем BOLA/IDOR, function-level auth, SSRF, rate limiting.
  - Добавляем:
    - Policy слой для RBAC и object-level auth.
    - DTO/Schema allowlists против mass assignment.
    - Безопасную валидацию webhook URL (allowlist, block private ranges, no redirects).
    - Rate limiting на login/refresh/test-send/API.
    - Secret scanning в CI, вынесение секретов из репо.
    - SCA + image scanning, quality gates в CI.
    - Базовый audit на security-sensitive действия.

- **v3 – Hardening & Platform Security**
  - Более строгая session security (refresh rotation, device inventory, logout-all, MFA stub).
  - Scopes, quotas и abuse detection per tenant.
  - Egress-restrictions для worker (NetworkPolicy, firewall).
  - Шифрование чувствительных полей (webhook secrets, API ключи).
  - DLQ, retries c exponential backoff.
  - Security dashboard и alerts.
  - IaC + policy-as-code (Terraform, OPA/Kyverno/Conftest).