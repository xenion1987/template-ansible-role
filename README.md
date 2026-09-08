# Ansible role: ROLE_NAME

[![prek](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/j178/prek/master/docs/assets/badge-v0.json)](https://github.com/j178/prek)
[![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)](https://github.com/OWNER/REPO/actions/workflows/ci.yml)
[![Molecule](https://img.shields.io/badge/tested%20with-Molecule-blue.svg)](https://github.com/ansible-community/molecule)
[![Galaxy downloads](https://img.shields.io/ansible/role/d/OWNER/ROLE_NAME?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/OWNER/ROLE_NAME)
[![License](https://img.shields.io/github/license/OWNER/REPO)](https://github.com/OWNER/REPO/blob/main/LICENSE)

> [!IMPORTANT]
> **`README.md` is generated - do not edit it.** The handwritten part lives in
> `HEADER.md`; the variable reference is built from the `@var` annotations in
> `defaults/main.yml`. Run `uv run ansible-doctor` to regenerate it, or just
> commit - a prek hook does it for you.
>
> Before you start, replace the placeholders `OWNER`, `REPO` and `ROLE_NAME`
> with your GitHub username/organization, repository name and role name in:
>
> - `HEADER.md` (this file - title and badge URLs)
> - `meta/main.yml` (`namespace`, `role_name`, `author`, `company`, `description`)
> - `meta/argument_specs.yml` (top-level spec key and `author`)
> - `pyproject.toml` (`name`, `description`)

Describe what this role does.

---

## Prerequisites

| Tool                                 | Required         | Purpose                                                                    |
| ------------------------------------ | ---------------- | -------------------------------------------------------------------------- |
| [uv](https://docs.astral.sh/uv/)     | yes              | Python package and project manager (manages Python versions automatically) |
| [Podman](https://podman.io/)         | yes              | Container runtime for Molecule tests                                       |
| [prek](https://github.com/j178/prek) | no (recommended) | Fast Git hook manager (drop-in pre-commit alternative)                     |
| [direnv](https://direnv.net/)        | no               | Auto-load environment variables                                            |

---

## Getting Started

### 1. Create a repository from this template

Click **"Use this template"** on GitHub or run:

```sh
gh repo create my-ansible-role --template https://github.com/OWNER/template-ansible-role
```

### 2. Clone and enter

```sh
git clone https://github.com/OWNER/my-ansible-role.git
cd my-ansible-role
```

### 3. Install dependencies

```sh
uv sync --group dev
```

### 4. Update metadata

Replace the placeholders listed in the note at the top of this file, then fill in:

- **`meta/main.yml`** - author, description, platforms, galaxy_tags
- **`meta/argument_specs.yml`** - document all role variables ([docs](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-argument-validation))

### 5. (Optional) Editor settings

```sh
cp .vscode/settings.example.json .vscode/settings.json
```

The default `ansible.python.interpreterPath` points at `.venv/bin/python`, which
is where `uv sync` creates the environment. Adjust it if you deviate from that.

### 6. (Optional) Environment auto-load with direnv

```sh
cp .example.envrc .envrc
direnv allow
```

This activates the virtual environment automatically when entering the directory.

### 7. Run the tests

```sh
uv run molecule test
```

---

## Development Workflow

### Directory structure

| Path                 | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| `tasks/`             | Main role tasks                                     |
| `handlers/`          | Handlers triggered by tasks                         |
| `defaults/`          | Default variables (lowest precedence) + `@var` docs |
| `vars/`              | OS-specific or internal variables                   |
| `files/`             | Static files to deploy                              |
| `templates/`         | Jinja2 templates (`.j2`)                            |
| `meta/`              | Role metadata and argument specs                    |
| `molecule/`          | Test scenarios                                      |
| `HEADER.md`          | Handwritten part of `README.md`                     |
| `.github/workflows/` | CI and release pipelines                            |

### Linting

```sh
uv run yamllint .          # YAML syntax and style (incl. .github/workflows)
uv run ansible-lint        # Best practices for the role and the Molecule playbooks
```

### Testing

```sh
uv run molecule test       # Full test: create, converge, idempotence, verify, destroy
uv run molecule converge   # Only apply the role (keep container running)
uv run molecule verify     # Run verification steps
uv run molecule destroy    # Tear down containers
```

Pick the target distribution with `MOLECULE_DISTRO` (default `debian13`):

```sh
MOLECULE_DISTRO=rockylinux9 uv run molecule test
```

Assertions belong in `molecule/default/verify.yml`, which runs as part of
`molecule test`. Keep at least one real assertion there - an empty verify stage
makes CI pass without checking anything.

### Git hooks (prek)

After installing dev dependencies, install the hooks:

```sh
prek install
```

The hooks then run on every `git commit`:

| Hook                  | Does                                                   |
| --------------------- | ------------------------------------------------------ |
| `trailing-whitespace` | Strips trailing whitespace                             |
| `end-of-file-fixer`   | Ensures a single trailing newline                      |
| `ansible-doctor`      | Regenerates `README.md` from `HEADER.md` + `defaults/` |
| `ansible-lint`        | Lints the role and the Molecule playbooks              |
| `yamllint`            | Lints all YAML                                         |

The lint hooks run the same repo-wide command as CI, so a green commit means a
green lint job.

### Documenting variables

`README.md` is assembled by [ansible-doctor](https://ansible-doctor.geekdocs.de/)
from `HEADER.md` plus `@var` annotations in `defaults/main.yml`:

```yaml
# @var my_variable:description: What this variable controls
# @var my_variable:type: str
# @var my_variable:required: false
# @var my_variable:example: >
# my_variable: some-value
my_variable: "default-value"
```

Valid subtypes are `description`, `type`, `required`, `value`, `example` and
`deprecated`. Two things to avoid:

- There is **no `:default:` subtype** - the default value is read from the YAML
  below the annotation.
- Do **not** use the short form `# @var my_variable: some text`. It is parsed as
  `:value:` and collides with the auto-detected value, which makes
  `ansible-doctor` exit non-zero and blocks the commit.

`meta/argument_specs.yml` carries role-level metadata (`author`, `description`)
and is only populated with `options` when runtime argument validation
(type checks, required fields, choices) is needed.

Regenerate manually with:

```sh
uv run ansible-doctor
```

---

## CI/CD (GitHub Actions)

### CI Pipeline (`.github/workflows/ci.yml`)

1. **Lint** - runs `yamllint` and `ansible-lint` across the whole repository
2. **Test** (depends on lint) - runs `molecule test` on Debian 13, Rocky Linux 9
   and Ubuntu 24.04, against each supported Ansible/Python pair:

   | ansible-core | Python |
   | ------------ | ------ |
   | 2.19         | 3.13   |
   | 2.21         | 3.14   |

   The pairs are explicit because each `ansible-core` release supports a
   specific range of controller Python versions. Keep the oldest pair in sync
   with `min_ansible_version` in `meta/main.yml` and the `ansible-core` floor in
   `pyproject.toml` - only tested versions should be advertised as supported.

3. **test-matrix** - a tiny aggregate job that fails unless every matrix job
   succeeded

Branch protection should require exactly two checks: **`lint`** and
**`test-matrix`**. Both names are stable, so adding a distribution or an
ansible-core version never means editing the required-check list again. Do not
require the individual `test (...)` contexts - renaming the matrix silently
leaves them "expected forever" and blocks every pull request.

No manual changes are required - the role name is derived from the repository
name automatically.

### Release Pipeline (`.github/workflows/release.yml`)

Tag and GitHub release are always created when a commit carries a release
prefix. Publishing to Ansible Galaxy is the only part that needs credentials and
is skipped without them, so the pipeline is useful in a repository that has no
Galaxy presence at all.

A push to `main` cuts a release automatically. The bump is derived from **every
commit since the last release tag**, not just the pushed commit - so a
multi-commit push, a merge-commit strategy, or a run that GitHub drops from the
concurrency queue cannot silently lose a release. The strongest signal wins:

| Commit subject (or message body)                      | Bump  |
| ----------------------------------------------------- | ----- |
| `major:` , any type with `!` (`feat!:`, `fix(api)!:`) | Major |
| `BREAKING CHANGE` / `BREAKING-CHANGE` in the body     | Major |
| `feat:` , `feat(scope):`                              | Minor |
| `fix:` , `fix(scope):`                                | Patch |
| anything else (`chore:`, `docs:`, `ci:`, `refactor:`) | none  |

Type prefixes match case-insensitively; `BREAKING CHANGE` is matched
case-sensitively, as the Conventional Commits spec defines it. Only the commit
*subject* decides the type, so the `* fix: ...` bullets that squash merges put
in the body do not trigger releases of their own.

The base version is the **highest** `v*` semver tag in the repository, not the
closest reachable one, so out-of-order or branch-local tags cannot bump from the
wrong base. Non-semver tags (`nightly-...`) are filtered out before any
arithmetic. Pre-release tags are deliberately not supported - if you need `-rc`
versions, replace this workflow with a dedicated tool rather than extending it.

**Every run writes a plan to the job summary** - base version, how many commits
were inspected, the resulting bump, the target version and which commits drove
it. A run that deliberately produces no release says so, instead of being an
indistinguishable green check.

**Manual runs**: *Actions -> Release Pipeline -> Run workflow* takes two inputs:

| Input     | Default | Purpose                                                      |
| --------- | ------- | ------------------------------------------------------------ |
| `bump`    | `auto`  | Force `patch`/`minor`/`major`, e.g. to recover a lost release |
| `dry_run` | `true`  | Print the plan without tagging or publishing                  |

`dry_run` defaults to on, so a manual trigger can never release by accident.

Release notes are generated by GitHub and shaped by `.github/release.yml`, which
groups PRs into categories and excludes dependency bumps.

> [!NOTE]
> The release workflow is not gated on CI by itself - both workflows trigger
> independently, so the protection comes from the branch. A ruleset on `main`
> requiring the `lint` and `test-matrix` status checks, linear history, and no
> force-push or deletion covers it. Add a pull-request requirement on top if you
> want to rule out direct pushes entirely.

After the release, the role is imported to Ansible Galaxy - but only if
`GALAXY_API_KEY` is configured. Without it the step logs why it is skipping and
succeeds, so tag and GitHub release still happen. The import references the
default branch because Galaxy discovers a role's versions from the repository's
tags.

Both workflows pin their actions to commit SHAs rather than moving tags; the
release job holds `contents: write` and sees the Galaxy token. Dependabot
updates SHA pins just as it does version tags.

### Secrets and Variables

Configure these under **Settings -> Secrets and variables -> Actions**:

| Name               | Type     | Required | Description                                                              |
| ------------------ | -------- | -------- | ------------------------------------------------------------------------ |
| `GALAXY_API_KEY`   | Secret   | no       | Ansible Galaxy API token; only the Galaxy import is skipped without it   |
| `GALAXY_NAMESPACE` | Variable | no       | Alternate Galaxy namespace (defaults to the repository owner)            |

### Dependency updates

`.github/dependabot.yml` keeps the GitHub Actions pins and the `uv.lock` dev
dependencies current. Adjust the schedule or reviewers to taste.

---

A template to create Ansible roles

## Table of contents

- [Requirements](#requirements)
- [Default Variables](#default-variables)
  - [example_variable](#example_variable)
- [Dependencies](#dependencies)
- [License](#license)
- [Author](#author)

---

## Requirements

- Minimum Ansible version: `2.19`

## Default Variables

### example_variable

Example variable for demonstration purposes

**_Required:_** false<br />
**_Type:_** str<br />

#### Default value

```YAML
example_variable: hello_world
```

#### Example usage

```YAML
example_variable: hello_world
```

## Dependencies

None.

## License

MIT

## Author

xenion1987
