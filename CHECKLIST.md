# OtterDog Migration POC — Master Checklist

**Goal:** Create a repeatable plan to migrate existing GitHub orgs and repos to OtterDog configs.

**Status key:**

- [ ] Not started
- [x] Done
- [~] In progress
- [!] Blocked

---

## Phase 1: Decisions

- [x] Decide: use `releng-otterdog-poc` as the target org
- [x] Decide: mirror a real org or create fake repos to simulate migration — **mirror a real org**
- [x] Decide: credential provider — **`pass`**
- [x] Identify which existing orgs/repos the final migration will cover — **all LF Releng managed orgs**
- [x] Decide: minimum number of PR reviewers required before merge — **0 for POC, 1 Releng sign-off for production**
- [x] Decide: change freeze windows — **none for POC, production decision deferred until after POC**
- [x] Decide: who has admin access to `otterdog-configs` repo — **POC: Vanessa only, production: TBD after POC**
- [x] Decide: who is the accountable owner for completed migrations — **LF Releng team**
- [x] Decide: is a security review required before going live? — **yes, security review required at every phase of POC and production**

---

## Phase 2: Local Environment Setup

- [ ] Check Python version — need 3.11 or later
- [ ] Install **Git**
- [ ] Install **pipx**
- [ ] Install **Poetry 2.0.1+** via pipx: `pipx install poetry>=2.0.1`
- [ ] Clone OtterDog repo and run `make init` to set up environment
- [ ] Verify: `./otterdog.sh --version`
- [ ] Set up shell completion (optional):
  - Bash: add `eval "$(_OTTERDOG_COMPLETE=bash_source otterdog)"` to `~/.bashrc`
  - Zsh: add `eval "$(_OTTERDOG_COMPLETE=zsh_source otterdog)"` to `~/.zshrc`
- [ ] Install `pass` (if using pass as credential provider)

---

## Phase 3: GitHub Token Setup

- [ ] Create a **dedicated service account** for OtterDog — do not use a personal account
- [ ] Create a GitHub PAT (Personal Access Token) for that service account
- [ ] Add these scopes:
  - [ ] `repo`
  - [ ] `workflow`
  - [ ] `admin:org`
  - [ ] `admin:org_hook`
  - [ ] `delete_repo`
- [ ] Confirm token uses **minimum required scopes only**
- [ ] Store token in `pass`
- [ ] Define token **rotation policy** (e.g. rotate every 90 days)
- [ ] Verify token works with OtterDog

---

## Phase 4: Config Repo Setup

- [ ] Create `otterdog-configs` repo in the POC org — set visibility to **private**
- [ ] Enable **secret scanning** on the config repo
- [ ] Define and document who has **admin access** to this repo
- [ ] Add a **CODEOWNERS** file — define who must review config changes
- [ ] Create `otterdog.json` — register the target org
- [ ] Set credential provider reference in `otterdog.json`
- [ ] Choose base template (Eclipse CSI default or custom)
- [ ] Create `orgs/` directory structure

---

## Phase 5: Simulate Existing Org

- [ ] Create test repos in the target org
- [ ] Set some org-level settings (e.g. default branch, visibility)
- [ ] Set branch protection rules on test repos
- [ ] Add webhooks if testing that too
- [ ] Document the expected state before import

---

## Phase 6: Import

- [ ] Run `otterdog import <org-id>`
- [ ] Confirm `.bak` backup file was auto-created by OtterDog
- [ ] Verify sensitive values (webhooks, secrets) were preserved from previous config
- [ ] Review generated `<org-id>.jsonnet` file
- [ ] Verify the `.jsonnet` file contains **no plaintext secrets** before committing
- [ ] Verify all repos are captured
- [ ] Verify all org settings are captured
- [ ] Verify branch protection rules are captured
- [ ] Commit config to `otterdog-configs` repo

---

## Phase 7: Validate

- [ ] Run `otterdog validate`
- [ ] Review warnings and errors
- [ ] Fix any issues in the jsonnet config
- [ ] Re-run validate until clean

---

## Phase 8: Test Apply

- [ ] **Backup current org state** — run `otterdog import <org-id>` to snapshot before first apply
- [ ] Run `otterdog apply` without `--force` — OtterDog will prompt for interactive approval
- [ ] Require **two-person sign-off** on plan output before approving
- [ ] Record the apply in a **ticket or issue** for audit trail
- [ ] Make a test change in the config (e.g. change a repo description)
- [ ] Run `otterdog apply` again and approve the prompt
- [ ] If removing resources — use `--delete-resources` flag explicitly
- [ ] If updating secrets — use `--update-secrets` flag explicitly
- [ ] If updating webhooks — use `--update-webhooks` flag explicitly
- [ ] **Post-apply verification** — confirm GitHub state matches config
- [ ] Verify change applied correctly in GitHub

---

## Phase 9: GitOps Workflow Setup

- [ ] Add branch protection to `otterdog-configs` repo
- [ ] Require PR review before merge — enforce minimum reviewer count
- [ ] Require **signed commits** on the config repo
- [ ] Define who can run `otterdog apply`
- [ ] Set up audit log — record who ran `otterdog apply` and when
- [ ] Define rollback trigger — document when and how to roll back
- [ ] Document the PR → review → apply workflow

---

## Phase 10: Documentation

- [ ] Write migration runbook
- [ ] Document prerequisites
- [ ] Document credential setup steps
- [ ] Document import process
- [ ] Document apply process
- [ ] Document rollback steps if apply goes wrong
- [ ] Document rollback trigger criteria
- [ ] Document token rotation procedure
