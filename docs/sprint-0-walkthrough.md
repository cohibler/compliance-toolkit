# Sprint 0 Walkthrough: Repo & Environment Foundation
### Security Engineering Self-Study Track — `compliance-toolkit`

---

## Why This Sprint Matters

Sprint 0 looks like nothing — no scanning logic, no NIST mapping, no drift detection. That's exactly why it's the sprint most likely to stall. There's no dopamine in `mkdir` and `git init`. But every professional Python repo you've ever seen that looked competent to a hiring manager got that way because someone did this boring part *deliberately*, not by accident.

Treat this sprint as a skill in itself: **how a security engineer initializes a project so that day 90 of the repo looks as intentional as day 1.** That's a real, evaluable competency — interviewers notice repo hygiene before they read a single line of your scan logic.

Target time: 60–90 minutes, one sitting. Definition of Done is at the bottom — check it, don't guess it.

---

## Prerequisites Checklist

- [ ] GitHub account (you have this)
- [ ] Git installed locally — check with `git --version`
- [ ] Python 3.11+ installed — check with `python3 --version`
- [ ] SSH key pair for GitHub auth (if you don't have one, Step 3 covers it)

---

## Step 1: Create the GitHub Repo

Go to GitHub → New Repository.

- **Name:** `compliance-toolkit`
- **Visibility:** Public. This is a portfolio artifact, not a private experiment — it needs to be visible from commit #1, not polished and revealed later. A recruiter or hiring manager looking at your commit history over time is *more* credible than a repo that appears fully-formed.
- **Initialize with:** a `README.md` and a `.gitignore` (Python template) — GitHub can scaffold both for you at creation time. Do **not** add a license yet; decide that deliberately later (MIT is the conventional default for a portfolio piece, but it's a two-second decision to make once, not now).

**Why public now, not later:** Employers increasingly discount repos that appear polished with no history — it reads as staged. A messy Sprint 0 commit followed by visible iteration is *more* credible signal than a clean repo that materializes fully-formed. Let the timeline be real.

---

## Step 2: Clone It Locally

```bash
git clone git@github.com:<your-username>/compliance-toolkit.git
cd compliance-toolkit
```

If `git clone` over SSH fails with a permission error, you don't have an SSH key registered with GitHub yet — that's Step 3, do it now rather than falling back to HTTPS + password, which GitHub has deprecated for git operations anyway.

---

## Step 3: Git Identity & Commit Signing

This step is the one most self-taught developers skip, and it's the one that actually differentiates a security engineer's repo from a hobbyist's.

**Basic identity** (if not already set globally):
```bash
git config --global user.name "Corbin <Last Name>"
git config --global user.email "<your-github-email>"
```

**SSH key for GitHub auth**, if you don't have one:
```bash
ssh-keygen -t ed25519 -C "corbin-github"
```
Add the resulting public key (`~/.ssh/id_ed25519.pub`) under GitHub → Settings → SSH and GPG keys.

**Commit signing — the differentiator.** GitHub marks signed commits "Verified." Unsigned commits are trivially spoofable — anyone can `git commit --author="Linus Torvalds"` and it'll look legitimate in the log. A security engineer's portfolio repo running unsigned commits is a small but real inconsistency between what you claim to value and what your tooling demonstrates.

You can sign with SSH keys directly (no separate GPG key needed, as of Git 2.34+):
```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```
Then register the same public key under GitHub → Settings → SSH and GPG keys → **also** add it as a *signing key* (GitHub lets a single key serve both roles).

**Tie-in to your Polaris build:** once the YubiKey 5C NFC arrives, you can move the signing key onto hardware (`ssh-keygen` with a FIDO2 resident key, or GPG-on-YubiKey) so commit signing is backed by a physical security key instead of a key sitting on disk. Not required for Sprint 0 — flag it as a Sprint 3+ hardening task once the YubiKey is in hand.

---

## Step 4: Python Environment

Don't install packages globally. Use a virtual environment scoped to this repo:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Your prompt should now show `(.venv)`. Confirm with `which python3` — it should point inside `.venv/`, not `/usr/bin` or a system path.

**A note on tooling choice:** `venv` (stdlib) is the right call here — no opinion needed. You'll see `conda`, `pyenv`, `poetry`, and `uv` recommended elsewhere; those solve problems (multiple Python versions, complex dependency resolution, publishing packages) you don't have yet. Reach for them if this repo's needs actually demand it later, not preemptively. `uv` in particular is worth knowing exists — it's become the fast-moving standard for dependency management in 2025–2026 Python tooling — but it's an optimization, not a Sprint 0 blocker.

---

## Step 5: Project Scaffold

```bash
mkdir -p scripts/warmup tests docs
touch requirements.txt
```

Resulting structure:
```
compliance-toolkit/
├── .venv/              # not committed — see .gitignore
├── scripts/
│   └── warmup/         # Sprint 1-2 lands here
├── tests/               # pytest lives here
├── docs/
├── requirements.txt
├── README.md
└── .gitignore
```

**Why this shape:** a flat `scripts/` + `tests/` layout, rather than a `src/` package layout, is deliberate for this stage. `src/`-layout packaging (with `pyproject.toml`, installable via `pip install -e .`) is the more "correct" modern standard for anything that will eventually be distributed as a package — worth knowing that's the industry direction — but it adds packaging overhead you don't need until the capstone (Sprint 4–6) has real internal imports across modules. Revisit the layout decision then, not now.

---

## Step 6: `.gitignore`

If GitHub's repo-creation wizard already added a Python `.gitignore`, open it and confirm it includes at minimum:

```
.venv/
__pycache__/
*.pyc
.pytest_cache/
.env
*.egg-info/
```

If you're writing it from scratch, don't hand-roll it — pull GitHub's canonical Python template: https://github.com/github/gitignore/blob/main/Python.gitignore

**The line that matters most for this specific repo:** `.env`. You're building a tool that ingests scan and STIG data — at some point you may test against real or realistic-looking data, credentials, or API keys. `.env` in `.gitignore` from commit #1 means you never have to think about it again. Retroactively scrubbing a secret from git history after the fact is a genuinely painful, error-prone process — prevention here is nearly free.

---

## Step 7: README v0

This is the file a recruiter reads before any code. Right now it only needs a few sentences — but they should already state the *point* of the repo, not just its contents:

```markdown
# compliance-toolkit

A working implementation of point-in-time compliance evidence validation
and drift detection — the tooling side of NIST 800-53 control assessment
and STIG compliance work.

Built as a hands-on complement to certification study (CISA/CISSP),
demonstrating the Python, containerization, and scanning-tool skills
behind compliance program work typically done manually or via
closed-source GRC platforms.

**Status:** Sprint 0 — environment and repo setup.

## Roadmap
- [x] Sprint 0 — Environment setup
- [ ] Sprint 1-2 — Python fundamentals (file/API handling, CLI, testing, logging)
- [ ] Sprint 3 — Docker + Kubernetes + Trivy/Grype
- [ ] Sprint 4-6 — Capstone: Compliance Evidence Validator
```

The roadmap-with-checkboxes pattern is doing real work here: it shows a hiring manager scanning your GitHub that this is a structured, in-progress body of work — not an abandoned weekend project. Update the checkboxes as you go; a stale roadmap is worse than no roadmap.

---

## Step 8: Pre-commit Hooks *(recommended, not blocking)*

This is the "real insight" item most self-taught tutorials skip entirely, and it's cheap to set up now while the repo is empty.

[`pre-commit`](https://pre-commit.com/) runs checks automatically before every commit — formatting, linting, and (relevantly, for a *compliance* tool) secret-scanning. Setting this up in Sprint 0 means every commit from here forward is clean by construction, rather than you retrofitting linting in Sprint 4 across a messier codebase.

```bash
pip install pre-commit
```

Create `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.9
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks
```

Then:
```bash
pre-commit install
```

**Why `ruff`:** it's replaced the old `flake8` + `black` + `isort` stack for most new Python projects as of the last couple years — same job (lint + format), an order of magnitude faster, one tool instead of three. Worth knowing this is where the ecosystem has moved, since a reviewer familiar with current Python tooling will register it as a small but real signal that you're not working from a five-year-old tutorial.

**Why `gitleaks`:** it's a secret-scanner — it blocks commits containing anything that looks like an API key, token, or credential. For a repo whose entire purpose is *compliance evidence handling*, shipping a secret-scanning pre-commit hook from day one is thematically correct, not just good practice. It's a small detail an interviewer who actually opens the repo will notice.

Pin versions (`rev:`) rather than tracking `main` — check current release tags on each repo if the ones above have moved by the time you set this up.

---

## Step 9: First Commit — Conventional Commits

Stage and commit using the [Conventional Commits](https://www.conventionalcommits.org/) format — `type(scope): description`. It's become close to a de facto standard in professional repos because it makes history skimmable and enables automated changelog generation later.

```bash
git add .
git commit -m "chore: initial repo scaffold and environment setup"
```

Common types you'll use across this project: `feat` (new capability), `fix`, `chore` (tooling/config), `docs`, `test`, `refactor`. You'll use most of these across Sprints 1–6 — starting the convention now means your capstone's commit log reads as a coherent build narrative, which is itself part of what an interviewer is evaluating when they open the "commits" tab.

---

## Step 10: Push & Verify

```bash
git push origin main
```

Then check on GitHub:
- Commit shows a green **Verified** badge (confirms signing worked)
- `.venv/` is *not* visible in the repo (confirms `.gitignore` worked)
- README renders correctly on the repo homepage

---

## Definition of Done — Sprint 0

- [ ] Repo exists on GitHub, public, with README and .gitignore
- [ ] Cloned locally, venv created and activated
- [ ] Git identity configured; commits are signed and show "Verified" on GitHub
- [ ] Folder scaffold (`scripts/warmup/`, `tests/`, `docs/`) committed
- [ ] `.gitignore` confirmed working (`.venv/` not tracked)
- [ ] README v0 states the repo's purpose and roadmap, not just its name
- [ ] (Recommended) pre-commit installed with ruff + gitleaks
- [ ] At least one commit pushed to `main` using conventional commit format

If every box is checked, Sprint 0 is done — not "mostly done," not "good enough for now." Move to Sprint 1.

---

## If You Get Stuck — Reference Resources

| Problem | Resource |
|---|---|
| SSH key / GitHub auth issues | https://docs.github.com/en/authentication/connecting-to-github-with-ssh |
| Commit signing setup | https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits |
| `venv` usage | https://docs.python.org/3/library/venv.html |
| `.gitignore` templates | https://github.com/github/gitignore |
| Conventional Commits spec | https://www.conventionalcommits.org/ |
| pre-commit framework | https://pre-commit.com/ |
| ruff (linter/formatter) | https://docs.astral.sh/ruff/ |
| gitleaks (secret scanning) | https://github.com/gitleaks/gitleaks |

If a command fails and none of the above resolves it, paste the exact error back here rather than troubleshooting solo for more than ~15 minutes — that's the ADHD-aware version of "don't let Sprint 0 eat your whole evening on a solvable error."

---

## Recap: What Makes This "Sprint 0" Instead of Just "Setup"

The difference between a tutorial-follower and an engineer is that the tutorial-follower runs the commands; the engineer knows *why each one is there* and can defend the choices in an interview. Every step above was chosen, not defaulted to:

- **Public repo** → credibility through visible iteration
- **Signed commits** → integrity claim backed by tooling, not just résumé language
- **`.gitignore` before any code exists** → secret hygiene by construction
- **Pre-commit with gitleaks** → the toolkit practices the discipline it's built to audit
- **Conventional commits** → a build narrative, not just a file history

That's the whole point of this track: not "I read about compliance tooling," but "here's a repo where you can watch me build it the way a security engineer actually would."
