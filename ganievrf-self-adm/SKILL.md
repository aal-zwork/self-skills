---
name: ganievrf-self-adm
description: >-
  Как работать с ADM-машиной ganievrf в Yandex Cloud: ssh-доступ, целевые
  команды make workspace/update/upgrade, порядок disaster recovery с нуля.
  Дополняет ganievrf-self-common; детали ролей — в ganievrf-self-ansible,
  детали секретов — в ganievrf-self-secrets.
---

# ganievrf-self-adm

## Доступ

- `ssh yandex-cloud-adm` — ssh alias на ADM-машину.
- Сервис-аккаунт инстанса уже имеет admin-права в Yandex Cloud — `yc`
  аутентифицируется автоматически, без отдельного секрета для самого `yc`.

## Основные команды (репозиторий `ansible`)

Выполняются **на самой ADM-машине** (зайти по ssh, затем `make`) —
`Makefile` не знает про удалённое исполнение и никуда не перебрасывает
команды сам.

- `make workspace` — полный бутстрап с нуля (пакеты, `yc`/`kubectl`/`helm`,
  gpg-agent/ssh-agent инфраструктура). Входная точка disaster recovery.
- `make update` — лёгкий рефреш: подтянуть роли и pass-store, пересобрать
  сессию, без переустановки пакетов.
- `make upgrade` — поднять закреплённые версии инструментов и
  переустановить бинари.

Каждая из этих целей спрашивает подтверждение перед применением;
`GANIEVRF_MAKE_ASK_SKIP=yes` — пропустить вопрос (режим автоматизации).

На macOS те же команды — через `gmake` (GNU Make из Homebrew), см.
`ganievrf-self-ansible` → раздел про `os-toolchain`.

## Сессия

- `ansible-playbook playbooks/session_open.yml` — разворачивает
  tmpfs-секреты и ssh-agent/gpg-agent на время сессии (kubeconfig, deploy-key).
- `ansible-playbook playbooks/session_close.yml` — подчищает всё явно
  (помимо естественной очистки tmpfs при перезагрузке).

## Disaster recovery (кратко)

1. Новая VM → `ssh yandex-cloud-adm` → `make workspace`.
2. Ручная доставка VM GPG-ключа (локально выпущен мастер-ключом) — см.
   `ganievrf-self-secrets`.
3. `make secrets` (репозиторий `secrets`) — создаёт/подтягивает облачные и
   внешние секреты.
4. `session_open.yml` — рабочее состояние готово.

Полная версия с проверками на каждом шаге — в `docs/disaster-recovery.md`
задачи, где эта инфраструктура была спроектирована (`ISSUE-GANIEVRF-0`).
