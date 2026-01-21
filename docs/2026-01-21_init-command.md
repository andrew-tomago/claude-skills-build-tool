# Plan: `cbt init` CLI Tool

## Goal
Build a Python CLI tool (`cbt`) that scaffolds the `.claude/` directory structure from README.md, inspired by dbt's init command.

## Package Structure

```
claude-skills-build-tool/
├── pyproject.toml                 # Entry point: cbt = "cbt.cli.main:cli"
├── src/
│   └── cbt/
│       ├── __init__.py            # __version__ = "0.1.0"
│       ├── __main__.py            # python -m cbt support
│       ├── cli/
│       │   ├── __init__.py
│       │   ├── main.py            # Click group
│       │   └── commands/
│       │       ├── __init__.py
│       │       └── init.py        # cbt init command
│       ├── scaffold/
│       │   ├── __init__.py
│       │   ├── builder.py         # ScaffoldBuilder class
│       │   └── templates/         # Static template files
│       │       ├── commands/{_dev,act,chain}/.gitkeep
│       │       ├── skills/.gitkeep
│       │       ├── shared/.gitkeep
│       │       └── docs/{index.md,getting-started.md,reference/,guides/}
│       └── events.py              # Color-coded CLI output
└── tests/
    ├── conftest.py
    ├── test_cli.py
    └── test_scaffold.py
```

## Critical Path (Implementation Order)

### Step 1: Create `pyproject.toml`
- Dependencies: `click>=8.0`
- Entry point: `cbt = "cbt.cli.main:cli"`
- Package data: include `templates/**/*`

### Step 2: Create `src/cbt/__init__.py` and `__main__.py`
- Version metadata
- Entry point delegation

### Step 3: Create `src/cbt/events.py`
- `EventEmitter` class with `info()`, `success()`, `warning()`, `error()`, `created()`
- Color-coded output via Click

### Step 4: Create `src/cbt/scaffold/builder.py`
Core logic:

```python
class ScaffoldBuilder:
    def preflight_check(self, force: bool = False) -> PreflightResult:
        # Check 1: Inside .claude directory? → Abort
        # Check 2: .claude exists and not --force? → Abort

    def scaffold(self) -> None:
        # Copy templates from package resources
        # Create directories with .gitkeep
```

### Step 5: Create template files in `src/cbt/scaffold/templates/`
- Mirror README structure exactly
- Include starter content in docs files
- Use `.gitkeep` for empty directories

### Step 6: Create `src/cbt/cli/main.py`
```python
@click.group()
@click.version_option()
def cli():
    """Claude Build Tool"""

cli.add_command(init)
```

### Step 7: Create `src/cbt/cli/commands/init.py`
```python
@click.command()
@click.option("--path", default=".")
@click.option("--force", is_flag=True)
def init(path, force):
    # 1. Create ScaffoldBuilder
    # 2. Run preflight_check()
    # 3. If ok: scaffold()
    # 4. Emit success + next steps
```

### Step 8: Write tests
- `test_scaffold.py`: Unit tests for preflight checks and scaffolding
- `test_cli.py`: Integration tests using Click's `CliRunner`

## Init Command Flow

```
cbt init [--path DIR] [--force]
    │
    ├─ Is user inside .claude/? ──YES──► Abort: "Cannot initialize inside .claude"
    │
    ├─ Does .claude/ exist?
    │   ├─ YES + no --force ──────────► Abort: "Already exists. Use --force"
    │   └─ YES + --force ─────────────► Remove existing, continue
    │
    └─ Copy templates → Create dirs → Emit "Created" for each file
                                     → Success message + next steps
```

## Files to Create

| File | Purpose |
|------|---------|
| `pyproject.toml` | Package config, entry points |
| `src/cbt/__init__.py` | Version metadata |
| `src/cbt/__main__.py` | `python -m cbt` support |
| `src/cbt/events.py` | CLI output formatting |
| `src/cbt/cli/__init__.py` | Package marker |
| `src/cbt/cli/main.py` | Click group |
| `src/cbt/cli/commands/__init__.py` | Package marker |
| `src/cbt/cli/commands/init.py` | Init command |
| `src/cbt/scaffold/__init__.py` | Package marker |
| `src/cbt/scaffold/builder.py` | Scaffolding logic |
| `src/cbt/scaffold/templates/**` | Template files |
| `tests/conftest.py` | Test fixtures |
| `tests/test_scaffold.py` | Builder tests |
| `tests/test_cli.py` | CLI tests |

## Verification

1. **Install**: `pip install -e .`
2. **Test init**:
   ```bash
   cd /tmp && mkdir test-project && cd test-project
   cbt init
   ls -la .claude/  # Verify structure
   ```
3. **Test existence check**:
   ```bash
   cbt init  # Should fail: "already exists"
   cbt init --force  # Should succeed
   ```
4. **Test inside-check**:
   ```bash
   cd .claude/commands
   cbt init  # Should fail: "inside a .claude directory"
   ```
5. **Run tests**: `pytest tests/ -v`
