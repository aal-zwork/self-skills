# AGENTS.md — skills/

This directory is a draft of a future standalone repository holding the
`ganievrf-self-*` skills: documentation (for both humans and agents) on how
to operate the infrastructure defined by the sibling `ansible/` and
`secrets/` repositories.

## Structure

- `ganievrf-self-common/` — hub document, not tied to this specific issue.
  Lists every `ganievrf-self-*` skill, what it is for, and how they relate
  (dependency tree). Purpose: when a *new* issue shows up later, this is the
  entry point to quickly pull in already-accumulated architecture knowledge
  instead of re-deriving it.
- `ganievrf-self-adm/` — operating the ADM machine: ssh alias, `make
  workspace`/`update`/`upgrade`, full disaster-recovery flow.
- `ganievrf-self-secrets/` — the secrets model: pass + S3 + per-VM GPG keys,
  how to add/rotate a secret (cloud or external), provider layout, what
  never leaves the local machine (the master key).
- `ganievrf-self-ansible/` — role/playbook structure, how to add a cloud
  provider, how `agent_runtime_scaffold` is laid out and where ecc plugs in.
- `ganievrf-self-skills/` — meta-skill: conventions for this skill set itself
  (language rule, naming, when to extend).

## Conventions

- Skill content itself (the body of each `ganievrf-self-*` skill) is written
  in Russian. This `AGENTS.md` file is in English — see
  `ganievrf-self-skills/` for the authoritative language rule.
- Keep `ganievrf-self-common/` generic: it must stay usable from a brand new,
  unrelated issue — do not bake ISSUE-GANIEVRF-0-specific task details into
  it. Task-specific detail belongs in the other four skills.
- When a new `ganievrf-self-*` skill is added, update
  `ganievrf-self-common/` tree listing in the same change — do not let it
  drift out of sync.
