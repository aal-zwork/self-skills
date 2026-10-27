---
name: ganievrf-self-ansible
description: >-
  Структура Ansible-ролей и плейбуков для ADM-машины ganievrf: provider
  abstraction (замена Yandex Cloud на другого провайдера без правки общих
  ролей), устройство agent_runtime_scaffold и точка подключения ecc,
  единый GNU-тулчейн Makefile на macOS/Ubuntu.
---

# ganievrf-self-ansible

## Структура ролей

- `roles/base/` — пакеты, базовые утилиты.
- `roles/providers/<name>/` — всё провайдер-специфичное (install CLI,
  аутентификация, получение kubeconfig, S3 CLI). Сейчас только `yandex_cloud`.
- `roles/gnupg_pass/` — импорт (не генерация!) уже доставленного VM GPG-ключа,
  инициализация `pass`.
- `roles/ssh_agent_session/`, `roles/ephemeral_runtime_dir/` — per-session
  ssh-agent и tmpfs runtime dir.
- `roles/github_access/` — github deploy-key через `pass`+`ssh-agent`.
- `roles/k8s_cluster_access/` — provider-agnostic, делегирует в
  `roles/providers/{{ cloud_provider }}/`.
- `roles/agent_runtime_scaffold/` — generic k8s-скелет под рантайм агентов
  (namespace/RBAC/NetworkPolicy/S3-secret), без привязки к конкретному
  рантайму.

## Provider abstraction

Переключатель — `group_vars/all.yml: cloud_provider`. Контракт провайдера —
4 task-файла (`install_cli.yml`, `authenticate.yml`, `get_k8s_credentials.yml`,
`s3_config.yml`). Новый провайдер = новая `roles/providers/<name>/` с тем же
набором + зеркальный `secrets/providers/<name>/`, без правки общих ролей.

## agent_runtime_scaffold + ecc

Скелет generic, рантайм (кандидат — [ecc](https://github.com/aal-zwork/ecc))
подключается отдельно. Открытые вопросы (запуск агентов, доставка образов
без интернета, авторизация в S3, доступ к GitHub) зафиксированы как TODO —
не блокируют скелет, но блокируют реальную интеграцию.

## Makefile / тулчейн

- `Makefile` ничего не знает про ADM, не перебрасывает команды по ssh —
  выполняет задачи там, где вызван.
- Пишется под Ubuntu/GNU Make; на macOS — через `gmake`/`gsed`/`ggrep`
  (Homebrew), выбор через `_detect-os` (простая цель, безопасно гонять на
  каждый вызов).
- Мутирующие цели (`workspace`/`update`/`upgrade`/`secrets`) спрашивают
  подтверждение; `GANIEVRF_MAKE_ASK_SKIP=yes` — пропустить для автоматизации.

## Terraform (будущее)

Появится отдельным слоем, должен брать креды из того же
`secrets/providers/<name>/` layout — не вводить третий способ хранения.
