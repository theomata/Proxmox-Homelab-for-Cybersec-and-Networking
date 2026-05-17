# Git Commit Conventions

Consistent commit messages make the repository history readable as a learning timeline.

---

## Format

```
type(scope): short description

Optional longer explanation if needed.
```

---

## Types

| Type | Use For |
|------|---------|
| `docs` | Adding or updating documentation |
| `lab` | New lab session log |
| `config` | Configuration files or infrastructure changes |
| `fix` | Correcting errors in existing docs |
| `add` | New files or sections |
| `refactor` | Reorganizing without changing content |

## Scopes

| Scope | Meaning |
|-------|---------|
| `proxmox` | Proxmox-related docs |
| `networking` | Networking docs |
| `linux` | Linux administration docs |
| `security` | Cybersecurity docs |
| `vm` | VM-specific documentation |
| `day-X` | Daily lab log |

---

## Examples

```bash
git commit -m "lab(day-1): add Day 1 session log with Proxmox setup and first VM"

git commit -m "docs(networking): add ip-route deep dive with routing table explanation"

git commit -m "docs(security): add ss-tulnp port auditing guide"

git commit -m "add(linux): systemctl service management reference"

git commit -m "docs(proxmox): document vmbr0 bridge architecture"

git commit -m "fix(networking): correct subnet notation in troubleshooting guide"

git commit -m "add(templates): lab session and command reference templates"
```

---

## Daily Workflow

```bash
# After each lab session
git add .
git status                          # Review what changed
git commit -m "lab(day-X): [summary of session]"
git push
```

## Batch Commits (when adding multiple docs at once)

```bash
git add networking/
git commit -m "docs(networking): add ip-route, ss-tulnp, and troubleshooting methodology"

git add linux/
git commit -m "docs(linux): add systemctl and apt package management references"

git push
```
