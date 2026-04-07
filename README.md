# galloClaudio

A [Claude Code](https://claude.ai/claude-code) plugin providing specialized AI agents and workflow slash commands for full-stack development teams working with Python backends, React frontends, AWS infrastructure, and scientific/geospatial data processing.

## What's included

### 6 Auto-Delegation Agents

Claude reads each agent's description and automatically routes work to the right specialist — no manual invocation needed.

| Agent | Triggers when you work on... |
|---|---|
| `web-developer` | Django REST views, FastAPI endpoints, React/TypeScript, Auth0, serializers |
| `data-manager` | PostgreSQL schemas, Alembic/Django migrations, QuestDB time-series, data quality |
| `data-engineer` | S3/SQS pipelines, Lambda functions, boto3 ETL jobs, data lake architecture |
| `devops-aws` | Terraform, Ansible playbooks, Dockerfiles, CI/CD, controller scripts |
| `data-scientist` | ML models, Bayesian networks (PyMC), scikit-learn pipelines, model evaluation |
| `eo-cv-specialist` | EO processing (GDAL/rasterio), camera traps (MegaDetector), BirdNET audio |

### 16 Slash Commands

Workflow commands you invoke explicitly — each loads a focused, step-by-step prompt.

| Command | Purpose |
|---|---|
| `/scaffold-api` | Scaffold a new Django/FastAPI endpoint with model, serializer, view, URL, and tests |
| `/review-frontend` | Review React/TypeScript code for type safety, performance, and security |
| `/db-migrate` | Design and generate a safe, reversible Alembic or Django migration |
| `/query-design` | Design and optimize PostgreSQL or QuestDB queries |
| `/terraform-provision` | Scaffold Terraform modules for AWS resources (ECS, RDS, S3, SQS, VPC, IAM) |
| `/ansible-configure` | Generate Ansible roles and playbooks for server configuration |
| `/controller-script` | Generate a `controller.sh` with `stop|start|deploy|destroy|build|logs|connect` |
| `/aws-infra` | Review or design AWS architecture with security and cost analysis |
| `/docker-setup` | Generate Dockerfile and docker-compose configuration |
| `/iam-review` | Audit IAM policies for least-privilege and security issues |
| `/model-design` | Design an ML model from problem framing to deployment plan |
| `/bayesian-network` | Design and implement a Bayesian hierarchical model in PyMC |
| `/eo-pipeline` | Scaffold a satellite imagery processing pipeline (COG, S3, QuestDB) |
| `/camera-trap` | Set up MegaDetector batch processing with SQS and PostgreSQL |
| `/birdnet-audio` | Set up BirdNET-Analyzer batch processing from S3 to PostgreSQL |
| `/s3-data-flow` | Design S3 data lake architecture with event notifications and lifecycle policies |

### MCP Servers

Configured in `.mcp.json` — give Claude direct access to your infrastructure:

| Server | Capability |
|---|---|
| PostgreSQL | Query your database directly from Claude |
| AWS S3 | Browse and manage S3 buckets and objects |
| AWS Core | Access AWS resources and services |
| AWS Cost Explorer | Analyze and optimize AWS spending |
| GitHub | Manage repos, issues, PRs, and code search |
| Atlassian | Access Jira issues and Confluence pages |

---

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed
- Git
- [jq](https://jqlang.github.io/jq/) (`brew install jq` on macOS)

### Quick install

```bash
git clone git@github.com:omaranda/galloClaudio.git ~/GitHub/galloClaudio
cd ~/GitHub/galloClaudio
./install.sh
```

That's it. Restart Claude Code and run `/plugin` to verify.

To uninstall: `./install.sh --uninstall`

### Configure MCP servers (optional but recommended)

The plugin ships with MCP server configs for PostgreSQL and AWS. Set the required environment variables in your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
# PostgreSQL direct access
export DATABASE_URL="postgresql://user:password@host:5432/dbname"

# AWS access
export AWS_PROFILE="your-aws-profile"
export AWS_REGION="eu-central-1"

# GitHub (requires a PAT with repo scope)
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_..."

# Atlassian / Jira / Confluence
export JIRA_URL="https://your-domain.atlassian.net"
export JIRA_USERNAME="your-email@example.com"
export JIRA_API_TOKEN="your-jira-api-token"
export CONFLUENCE_URL="https://your-domain.atlassian.net"
export CONFLUENCE_USERNAME="your-email@example.com"
export CONFLUENCE_API_TOKEN="your-confluence-api-token"
```

### Step 5 — Restart Claude Code

Close and reopen Claude Code. The plugin loads at session start.

### Step 6 — Verify

Run `/plugin` in Claude Code — `galloClaudio@local` should appear in the list.

Test an agent by opening any project file and asking Claude to make a change. The relevant specialist agent should activate automatically.

Test a slash command by typing `/scaffold-api` or `/controller-script`.

---

## Keeping up to date

Pull the latest changes from the repo:

```bash
cd ~/GitHub/galloClaudio && git pull
```

No reinstall required — Claude Code reads the plugin directory live.

---

## Contributing

Agents live in `agents/`, slash commands in `commands/`. Each is a Markdown file with YAML frontmatter.

**To add a new agent**, create `agents/your-agent-name.md` following the existing format:

```markdown
---
name: your-agent-name
description: Use this agent when... Examples: <example>...</example>
model: inherit
color: blue
---

You are a specialist in...
```

**To add a new slash command**, create `commands/your-command.md`:

```markdown
---
name: your-command
description: What this command does
---

Step-by-step instructions for Claude...
```

Changes take effect on the next Claude Code session — no reinstall needed.

---

## Stack reference

This plugin is tuned for the following technology stack:

- **Backend**: Django REST Framework, FastAPI, Python 3.10+
- **Frontend**: React, TypeScript
- **Auth**: Auth0, JWT / JWKS
- **Databases**: PostgreSQL + PostGIS, QuestDB
- **Cloud**: AWS (ECS Fargate, Lambda, RDS, S3, SQS, VPC, IAM)
- **IaC**: Terraform + Ansible
- **EO processing**: GDAL, rasterio, xarray, rio-cogeo, Sentinel-2, Planet
- **Wildlife detection**: MegaDetector v5, DeepFaune, BirdNET-Analyzer
- **ML**: scikit-learn, PyMC, XGBoost, PyTorch
