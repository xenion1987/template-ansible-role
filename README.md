# Ansible role: ${{ role_name }}

[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://github.com/pre-commit/pre-commit)
[![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)](https://github.com/OWNER/REPO/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/OWNER/REPO)](LICENSE)

> **Before you start:** Replace `OWNER`, `REPO` and `${{ role_name }}` with your GitHub username/organization, repository name and the actual role name throughout this file, the badge URLs, `HEADER.md` and `meta/main.yml`.

Describe what this role does.

---

## Prerequisites

| Tool                                  | Required         | Purpose                                                                    |
| ------------------------------------- | ---------------- | -------------------------------------------------------------------------- |
| [uv](https://docs.astral.sh/uv/)      | yes              | Python package and project manager (manages Python versions automatically) |
| [Podman](https://podman.io/)          | yes              | Container runtime for Molecule tests                                       |
| [pre-commit](https://pre-commit.com/) | no (recommended) | Local lint hooks                                                           |
| [direnv](https://direnv.net/)         | no               | Auto-load environment variables                                            |

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
uv sync --dev
```

### 4. Update metadata

- **`meta/main.yml`** – author, description, platforms, galaxy_tags
- **`meta/argument_specs.yml`** – document all role variables ([docs](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-argument-validation))

### 5. (Optional) Editor settings

```sh
cp .vscode/settings.json.example .vscode/settings.json
```

Then adjust `ansible.python.interpreterPath` to your local Python path.

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

| Path                 | Purpose                               |
| -------------------- | ------------------------------------- |
| `tasks/`             | Main role tasks                       |
| `handlers/           | Handlers triggered by tasks           |
| `defaults/`          | Default variables (lowest precedence) |
| `vars/`              | OS-specific or internal variables     |
| `files/`             | Static files to deploy                |
| `templates/`         | Jinja2 templates (`.j2`)              |
| `meta/`              | Role metadata and argument specs      |
| `molecule/`          | Test scenarios                        |
| `.github/workflows/` | CI pipeline                           |

### Linting

```sh
uv run ansible-lint        # Best-practices for Ansible playbooks/roles
uv run yamllint .          # YAML syntax and style
```

### Testing

```sh
uv run molecule test       # Full test: create, converge, verify, destroy
uv run molecule converge   # Only apply the role (keep container running)
uv run molecule verify     # Run verification steps
uv run molecule destroy    # Tear down containers
```

### Pre-commit hooks

After installing dev dependencies, activate the hooks:

```sh
uv run pre-commit install
```

The hooks (`trailing-whitespace`, `end-of-file-fixer`, `ansible-lint`, `yamllint`) now run automatically on every `git commit`.

### README generation

This template includes [ansible-doctor](https://ansible-doctor.geekdocs.de/). It generates a `README.md` from `meta/argument_specs.yml` and a header file:

```sh
uv run ansible-doctor
```

The `HEADER.md` is prepended automatically (configured in `.ansibledoctor.yml`).

---

## CI/CD (GitHub Actions)

The workflow in `.github/workflows/ci.yml`:

1. **Lint** – runs `yamllint` and `ansible-lint`
2. **Test** (depends on lint) – runs `molecule test` across:
   - Distros: Debian 13, Rocky Linux 9, Ubuntu 24.04
   - Ansible versions: latest on all, older versions on a subset

No manual changes to the CI file are required – the role name is derived from the repository name automatically.

---

## License

MIT
