# Ansible role: ${{ role_name }}

[![prek](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/j178/prek/master/docs/assets/badge-v0.json)](https://github.com/j178/prek)
[![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)](https://github.com/OWNER/REPO/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/OWNER/REPO)](LICENSE)

> [!IMPORTANT]
> Before you start: Replace `OWNER`, `REPO` and `${{ role_name }}` with your GitHub username/organization, repository name and the actual role name throughout this file, the badge URLs, `HEADER.md` and `meta/main.yml`.

Describe what this role does.

---

## Prerequisites

| Tool                                  | Required         | Purpose                                                                    |
| ------------------------------------- | ---------------- | -------------------------------------------------------------------------- |
| [uv](https://docs.astral.sh/uv/)      | yes              | Python package and project manager (manages Python versions automatically) |
| [Podman](https://podman.io/)          | yes              | Container runtime for Molecule tests                                       |
| [prek](https://github.com/j178/prek) | no (recommended) | Fast Git hook manager (drop-in pre-commit alternative)                     |
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
uv sync --groups dev
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
| `handlers/`           | Handlers triggered by tasks           |
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

### Git hooks (prek)

After installing dev dependencies, install the hooks:

```sh
prek install
```

The hooks (`trailing-whitespace`, `end-of-file-fixer`, `ansible-lint`, `yamllint`) now run automatically on every `git commit`.

### README generation

This template includes [ansible-doctor](https://ansible-doctor.geekdocs.de/). It generates the `README.md` from `HEADER.md` and variable documentation:

- **Variable documentation** is maintained as `@var` annotations in `defaults/main.yml`
- **`HEADER.md`** is prepended automatically (configured in `.ansibledoctor.yml`)
- **`meta/argument_specs.yml`** stores role-level metadata (`author`, `description`) and is only populated for runtime argument validation when needed

```sh
uv run ansible-doctor
```

A [prek hook](.pre-commit-config.yaml) runs `ansible-doctor` automatically on changes to `defaults/`, `meta/`, `HEADER.md` or `.ansibledoctor.yml`.

---

## CI/CD (GitHub Actions)

Two workflows are provided:

### CI Pipeline (`.github/workflows/ci.yml`)

1. **Lint** – runs `yamllint` and `ansible-lint`
2. **Test** (depends on lint) – runs `molecule test` across:
   - Distros: Debian 13, Rocky Linux 9, Ubuntu 24.04
   - Ansible 2.20 on Python 3.14

No manual changes are required – the role name is derived from the repository name automatically.

### Release Pipeline (`.github/workflows/release.yml`)

Creates a GitHub release on every push to `main` when the commit message starts with a semantic prefix:

| Prefix   | Bump   |
| -------- | ------ |
| `major:` | Major  |
| `feat:`  | Minor  |
| `fix:`   | Patch  |

After creating the release, the role is imported to Ansible Galaxy – but only if `GALAXY_API_KEY` is configured. Missing the key skips the step gracefully, so the pipeline works without it.

### Secrets and Variables

Configure these in your repository under **Settings → Secrets and variables → Actions**:

| Name                | Type     | Required | Description                                             |
| ------------------- | -------- | -------- | ------------------------------------------------------- |
| `GALAXY_API_KEY`    | Secret   | no       | Ansible Galaxy API token (step is skipped if unset) |
| `GALAXY_NAMESPACE`  | Variable | no       | Alternate Galaxy namespace (defaults to repo owner)  |

---

## License

MIT
