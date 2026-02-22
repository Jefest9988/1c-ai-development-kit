---
name: 1c-project-init
description: Initialize a new 1C project from a server database - create folder structure, copy skills, auto-detect 1C version, generate .1c-devbase.bat, dump configuration and extensions, setup OpenSpec and Git. Use when user requests "создать проект 1С", "инициализировать проект", "развернуть проект из базы", "init 1C project", or needs to bootstrap a new 1C development workspace from an existing database.
---

# 1c-project-init

Automated 1C project initialization from a server database. Creates a complete development workspace in one command.

## What It Does (14 steps)

1. Creates project structure (`src/cf`, `src/cfe`, `mcp/base`, `mcp/ext`, `openspec`, `.cursor/skills`, `.cursor/rules`, `.cursor/agents`, `docs`)
2. **Creates MCP data folders** (`mcp/base/report`, `mcp/base/src`, `mcp/ext/report`, `mcp/ext/src`)
3. **Creates symlinks** (`mcp/base/src` → `src/cf/`, `mcp/ext/src` → `src/cfe/`)
4. **Copies all skills** (10 skills):
   - **Core skills** (4): 1c-batch, 1c-feature-dev-enhanced, 1c-help-mcp, auto-skill-bootstrap
   - **Extended skills** (5): 1c-forms, 1c-mxl, 1c-roles, 1c-bsp, 1c-query-optimization
   - **Total**: 30+ sub-skills with JSON DSL support
5. **Copies 1C agents** (12 agents: onec-*, onec-admin, mcp-deploy, dev-optimizer)
6. Auto-detects 1C version by server port
7. Generates `.1c-devbase.bat` from 1c-batch template (or uses existing one)
8. Dumps configuration to `src/cf/`
9. Dumps extensions to `src/cfe/<Name>/` (if specified via `-Extensions`)
10. **Copies AI rules** (7 rules: skills-first, no-roi-estimates, context-management, model-selection, bsl-lsp-integration, rlm-toolkit-autoload, mcp-tools-usage)
11. **Copies optimization docs** (.cursorignore, CURSOR-CONTEXT-LIMITS.md, docs/cursor-optimization.md)
12. Creates OpenSpec structure (AGENTS.md, project.md, templates)
13. **Creates project MCP configuration** (`.cursor/mcp.json`, `docker-compose.yml`, management scripts)
14. Creates README.md with MCP deployment instructions
15. **Creates MCP deployment guide** (configs/mcp-setup.md with instructions for Configuration Report export)
16. Initializes Git with 2 commits (configuration + extensions)

## Personal Skill

This skill is also installed as a **personal skill** at:
`C:\Users\YOUR_USERNAME\.cursor\skills\1c-project-init\SKILL.md`

This makes it available in any project (including empty folders).

## Prerequisites

- PowerShell 5.1+
- Git installed and in PATH
- 1C:Enterprise installed (auto-detected from Program Files)
- Server database accessible (1C server running)
- Console encoding: UTF-8 (chcp 65001) - automatically set by script

## Cyrillic Encoding

This skill uses **UTF-8 BOM encoding** for all generated files to ensure proper Cyrillic display:
- README.md, openspec/*.md, configs/*.md - UTF-8 BOM
- .cursorrules, .gitignore, .1c-devbase.bat - UTF-8 BOM
- Console output - UTF-8 (chcp 65001)

See `.cursor/rules/powershell-cyrillic.mdc` for encoding rules.

Test encoding: `.cursor/skills/1c-project-init/scripts/test-cyrillic.ps1`

### Check prerequisites

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\check-prerequisites.ps1" `
  -Server "localhost:1641" `
  -Database "MyBase" `
  -Username "Admin"
```

---

## Usage

### Simple case (ASCII username or no auth)

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\MyProject" `
  -Server "localhost:1641" `
  -Database "MyBase" `
  -Username "Admin" `
  -Force
```

### Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `-ProjectPath` | Yes | — | Absolute path to new project directory |
| `-Server` | Yes | — | 1C server `host:port` (e.g. `localhost:1641`) |
| `-Database` | Yes | — | Database name on server |
| `-Username` | No | `""` | 1C user name |
| `-Password` | No | `""` | 1C user password |
| `-ProjectName` | No | folder name | Display name for README/OpenSpec |
| `-OneCPath` | No | auto-detect | Explicit path to `1cv8.exe` |
| `-Extensions` | No | none | Array of extension names to dump (ask user if they have extensions) |
| `-SkipExtensions` | No | `false` | Switch to skip extensions entirely |
| `-Force` | No | `false` | Overwrite if path exists (**always use in agent mode**) |
| `-GitUserName` | No | `"Developer"` | Git commit author name |
| `-GitUserEmail` | No | `"dev@local"` | Git commit author email |
| `-SkillsSource` | No | Home_Infrastructure | Source directory for skills to copy |

### Port-to-Version Auto-Detection

| Server Port | 1C Version | EDT project suffix |
|-------------|------------|--------------------|
| 1540, 1541  | 8.3.25     | `_25`              |
| 1640, 1641  | 8.3.24     | `_24`              |
| 1740, 1741  | 8.3.27     | `_27`              |
| Other       | Latest installed | —            |

---

## CRITICAL: Cyrillic Usernames

PowerShell terminal in Cursor **corrupts Cyrillic** in command-line arguments.
The agent has the `Write` tool which writes files in perfect UTF-8.

**For Cyrillic usernames, use this 2-step approach:**

### Step 1: Create `.1c-devbase.bat` via Write tool

Use the Write tool to create the config file directly. Cyrillic will be preserved perfectly.

**Path:** `{ProjectPath}/.1c-devbase.bat`

**Content template:**

```bat
@echo off
set "ONEC_PATH=C:\Program Files\1cv8\{version}\bin\1cv8.exe"
set "ONEC_SERVER={server}"
set "ONEC_BASE={database}"
set "ONEC_USER={username_cyrillic}"
set "ONEC_PASSWORD={password}"
```

**Example** for Бухгалтер on localhost:1641 / Minim_kg:

```bat
@echo off
set "ONEC_PATH=C:\Program Files\1cv8\8.3.24.1808\bin\1cv8.exe"
set "ONEC_SERVER=localhost:1641"
set "ONEC_BASE=Minim_kg"
set "ONEC_USER=Бухгалтер"
set "ONEC_PASSWORD="
```

**Determine ONEC_PATH:** Use the port-to-version table above, then find the installed version:

```powershell
Get-ChildItem "C:\Program Files\1cv8\" -Directory | Where-Object { $_.Name -like "8.3.24*" }
```

### Step 2: Run init script (it detects existing config)

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\MinimKG" `
  -Server "localhost:1641" `
  -Database "Minim_kg" `
  -Force
```

The script sees `.1c-devbase.bat` already exists → reads credentials from it → uses them for dump.
**No `-Username` needed** — it's already in the config file.

---

## Extensions

Extensions **cannot be auto-discovered** for server databases (ibcmd doesn't support it).

**Agent behavior:**
1. If user mentions extensions (e.g. "расширение Minim") → pass them via `-Extensions @("Minim")`
2. If user doesn't mention extensions → ask: "Есть ли расширения в базе? Если да, укажите названия."
3. If user says "без расширений" or "нет" → use `-SkipExtensions`
4. Extensions can be added later by re-running `dump-extension.bat` manually

### Dump extensions after initialization

```powershell
cd C:\Users\YOUR_USERNAME\workspace\MyProject
.cursor\skills\1c-batch\scripts\dump-extension.bat src\cfe\Minim Minim
```

---

## Examples

### ASCII username (one command)

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\ERP_Dev" `
  -Server "localhost:1540" `
  -Database "erp_dev" `
  -Username "Admin" `
  -Force
```

### Cyrillic username (two steps)

**Step 1** — Agent creates `.1c-devbase.bat` via Write tool:

Write file `C:\Users\YOUR_USERNAME\workspace\MinimKG\.1c-devbase.bat` with content:
```bat
@echo off
set "ONEC_PATH=C:\Program Files\1cv8\8.3.24.1808\bin\1cv8.exe"
set "ONEC_SERVER=localhost:1641"
set "ONEC_BASE=Minim_kg"
set "ONEC_USER=Бухгалтер"
set "ONEC_PASSWORD="
```

**Step 2** — Agent runs init:
```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\MinimKG" `
  -Server "localhost:1641" `
  -Database "Minim_kg" `
  -Force
```

### With extensions

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\MinimKG" `
  -Server "localhost:1641" `
  -Database "Minim_kg" `
  -Extensions @("Minim") `
  -Force
```

### No auth (empty username)

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\TestProject" `
  -Server "localhost:1540" `
  -Database "test_db" `
  -Force
```

### Explicit 1C path

```powershell
& "C:\Users\YOUR_USERNAME\workspace\Home_Infrastructure\.cursor\skills\1c-project-init\scripts\init-1c-project.ps1" `
  -ProjectPath "C:\Users\YOUR_USERNAME\workspace\MyProject" `
  -Server "localhost:1540" `
  -Database "MyBase" `
  -OneCPath "C:\Program Files\1cv8\8.3.25.1257\bin\1cv8.exe" `
  -Force
```

---

## Result Structure

```
MyProject/
├── .1c-devbase.bat              # Connection config (NOT in git)
├── .cursorignore                # Cursor cache optimization
├── .gitignore
├── README.md
├── CURSOR-CONTEXT-LIMITS.md     # Context limits guide
├── src/
│   ├── cf/                      # Configuration XML (dumped)
│   └── cfe/
│       └── ExtName/             # Extension XML (if any)
├── mcp/                         # Data for MCP servers
│   ├── base/                    # Configuration data
│   │   ├── report/              # Configuration report (export from 1C)
│   │   └── src/                 # Symlink → src/cf/
│   └── ext/                     # Extension data
│       ├── report/              # Extension report
│       └── src/                 # Symlink → src/cfe/
├── openspec/
│   ├── AGENTS.md
│   ├── README.md
│   ├── project.md
│   └── changes/
│       └── _template/
│           └── spec.md
├── configs/
│   └── mcp-setup.md             # MCP deployment guide
├── docs/
│   └── cursor-optimization.md   # Optimization guide
└── .cursor/
    ├── agents/                  # 12 specialized agents
    │   ├── onec-code-architect.md
    │   ├── onec-code-explorer.md
    │   ├── onec-code-reviewer.md
    │   ├── onec-code-writer.md
    │   ├── onec-code-simplifier.md
    │   ├── onec-form-generator.md
    │   ├── onec-metadata-helper.md
    │   ├── onec-query-optimizer.md
    │   ├── onec-test-generator.md
    │   ├── onec-admin.md
    │   ├── mcp-deploy.md
    │   └── dev-optimizer.md
    ├── rules/
    │   ├── skills-first.mdc
    │   ├── no-roi-estimates.mdc
    │   ├── context-management.mdc
    │   ├── model-selection.mdc
    │   ├── bsl-lsp-integration.mdc
    │   ├── rlm-toolkit-autoload.mdc
    │   └── mcp-tools-usage.mdc
    └── skills/
        ├── 1c-batch/
        ├── 1c-feature-dev-enhanced/
        ├── 1c-help-mcp/
        └── auto-skill-bootstrap/
```

---

## After Initialization

### Immediate (ready to code)
1. `cd <ProjectPath>` and open in Cursor
2. Start development via `openspec/changes/` (SDD workflow)
3. Use `1c-batch` skill for dump/load operations
4. Use `1c-feature-dev-enhanced` skill for full development cycle
5. Use `1c-help-mcp` skill for 1C documentation search

### EDT Integration (optional but recommended)

[EDT-MCP](https://github.com/DitriXNew/EDT-MCP) (by DitriX) enables AI to read live BSL modules, search code, find references, and validate queries directly in EDT.

**Setup:**
1. Install [EDT-MCP plugin](https://github.com/DitriXNew/EDT-MCP) (DitriX) in 1C:EDT
2. Configure MCP server port in plugin settings (default: 8765)
3. Add `edt-mcp` to project `.cursor/mcp.json` (see `templates/mcp.json`)
4. Import project into EDT workspace — name must be Latin, convention: `ProjectName_<version_suffix>` (e.g. `MyProject_25`)

**Key tools:** `list_projects`, `read_module_source`, `search_in_code`, `find_references`, `validate_query`, `get_project_errors`

### Optional (for full AI capabilities)
6. **Export Configuration Report** (for MCP servers):
   - Open 1C Designer → Configuration → Reports → Configuration Report
   - Select all objects
   - Export to: `mcp/base/report/` (format: Text files)
   - If extensions: Export extension report to `mcp/ext/report/`
7. **Deploy project-specific MCP servers** (see `configs/mcp-setup.md`):
   - **IMPORTANT:** Ask user which objects need MCP:
     - Configuration (src/cf/)? → `{project}-cf-metadata` + `{project}-cf-codesearch`
     - Extensions (src/cfe/)? → `{project}-{extname}-metadata` + `{project}-{extname}-codesearch`
   - Each object gets separate MCP servers (separate ports)
   - Port allocation: cf=7500-7502, ext1=7510-7512, ext2=7520-7522, etc.
7. Wait for indexing (~40-60 minutes per object)
8. Use specialized agents (onec-code-explorer, onec-metadata-helper, etc.)

---

## Troubleshooting

### "Пользователь ИБ не идентифицирован"
Username is wrong or corrupted. For Cyrillic — create `.1c-devbase.bat` via Write tool first (see above).

### 1C not found
Specify `-OneCPath` explicitly pointing to `1cv8.exe`.

### Configuration dump fails
1. Run `check-prerequisites.ps1` to verify database connectivity
2. Ensure 1C Designer is not already open (script runs `taskkill /F /IM 1cv8.exe`)
3. Check `.1c-devbase.bat` has correct values

### Extension dump fails
1. Verify extension name matches exactly (case-sensitive)
2. Check extension exists in the database via 1C Designer

### Path already exists
Script fails without `-Force`. Always pass `-Force` when running from agent.

---

## Project MCP Management

After initialization, each project gets its own **project-specific MCP configuration** (`.cursor/mcp.json`).

### Key Concepts

1. **Workspace-specific MCP** — Each project has its own `.cursor/mcp.json`
2. **Common MCP always available** — 1c-help, 1c-ssl, 1c-templates, syntax-checker, code-checker, forms, rlm-toolkit, bsl-lsp-bridge
3. **Project MCP only in their projects** — KAF MCP only in KAF project, MCParqa24 MCP only in MCParqa24 project
4. **Cloud indexing → Local model** — Start with cloud (qwen 72b), switch to local (qwen3:8b) after indexing

### Workflow

#### 1. Initial Setup (Cloud Indexing)

After `init-1c-project.ps1` completes:

```powershell
cd C:\Users\YOUR_USERNAME\workspace\MyProject

# 1. Export Configuration Report
#    Designer → Configuration → Reports → Configuration Report
#    Save to: mcp/base/report/

# 2. Export Extension Report (if extensions)
#    Designer → Extensions → <Name> → Reports → Configuration Report
#    Save to: mcp/ext/report/

# 3. Start MCP with CLOUD indexing
.\start-mcp-cloud.ps1

# 4. Wait for indexing (check logs)
docker-compose logs -f
```

**Cloud model:** `qwen/qwen-2.5-72b-instruct` via OpenRouter (fast indexing)

#### 2. Switch to Local Model

After indexing completes (~40-60 minutes):

```powershell
# 1. Switch Docker containers to local model
.\switch-to-local.ps1

# 2. Update Cursor MCP config
& .cursor\skills\1c-project-init\scripts\manage-project-mcp.ps1 `
  -Action switch-local `
  -ProjectPath .

# 3. Restart Cursor
# Ctrl+Shift+P → "Reload Window"
```

**Local model:** `qwen3:8b` via Ollama @ CT XXX (YOUR_RLM_SERVER:11434)

#### 3. Enable/Disable Project MCP

```powershell
# Check status
& .cursor\skills\1c-project-init\scripts\manage-project-mcp.ps1 `
  -Action status `
  -ProjectPath .

# Disable project MCP (use only common MCP)
& .cursor\skills\1c-project-init\scripts\manage-project-mcp.ps1 `
  -Action disable `
  -ProjectPath .

# Enable project MCP
& .cursor\skills\1c-project-init\scripts\manage-project-mcp.ps1 `
  -Action enable `
  -ProjectPath .
```

### Files Created

```
MyProject/
├── .cursor/
│   └── mcp.json              # Project-specific MCP config
├── docker-compose.yml         # MCP containers (codemetadata + graph)
├── start-mcp-cloud.ps1       # Start with cloud indexing
└── switch-to-local.ps1       # Switch to local model
```

### MCP Servers per Project

Each project gets 2 MCP servers:

1. **{project}-codemetadata** — Semantic search (ChromaDB + embeddings)
2. **{project}-graph** — Graph search (Neo4j)

**Ports:** Auto-assigned starting from 8000 (codemetadata), 8001 (graph)

### Benefits

- ✅ **No MCP pollution** — Only relevant MCP servers in each project
- ✅ **Fast cloud indexing** — qwen 72b via OpenRouter
- ✅ **Free local inference** — qwen3:8b via Ollama after indexing
- ✅ **Easy management** — Enable/disable per project
- ✅ **Isolated configs** — Each project independent

---

## Important

- **Run from any directory** — `ProjectPath` is absolute
- **Do NOT read scripts** — treat them as black boxes, just run
- **`.1c-devbase.bat` is NOT committed** — contains credentials, in `.gitignore`
- **Skills are COPIED** — each project gets its own independent copy
- **Execution time** — ~2-3 minutes depending on configuration size
- **MCP config is project-specific** — `.cursor/mcp.json` in each project (not global)
