# User Guide — gallo-claudio

This guide explains how to use the **gallo-claudio** plugin in Claude Code. You don't need prior programming experience to take advantage of it — the plugin works like a team of virtual specialists that respond to plain language requests.

---

## What is gallo-claudio

gallo-claudio is a plugin for **Claude Code** (Anthropic's AI-powered software development tool). Once installed, it adds:

- **16 specialist agents** that activate automatically based on the topic of your request
- **16 quick commands** that you can invoke directly for specific tasks

Think of the plugin as a team of consultants: each one has their area of expertise, and Claude knows who to ask depending on what you need.

---

## Part 1 — Agents

### How they work

Agents are specialists that **activate on their own**. You don't need to invoke them manually — simply describe what you need and Claude detects which agent is best suited to help.

### Practical example

Imagine you type in Claude Code:

> "I need to design a camera trap sampling plan for a 200-hectare forest"

Claude detects that your query is about camera trap survey design and automatically activates the **field-ecologist** agent, which knows about:
- Sampling grid design
- Camera placement strategies
- Field protocols
- Deployment logistics

You don't need to know the agent is called "field-ecologist" — just describe your need.

### The 16 available agents

#### Engineering agents (11)

These agents help with technical software development and infrastructure tasks.

| Agent | When it activates | Example question |
|---|---|---|
| **web-developer** | When working with web APIs, web pages, or authentication | "Create an endpoint to filter species by region" |
| **data-manager** | When working with databases or migrations | "Add a coordinates field to the sites table" |
| **data-quality-engineer** | When you need to validate data or create cleaning pipelines | "Create validation rules for the detections data" |
| **data-engineer** | When working with cloud data flows (AWS) | "Design a pipeline to process CSV files from S3" |
| **devops-aws** | When configuring servers, containers, or AWS infrastructure | "Create Terraform infrastructure to deploy the API" |
| **devops-multicloud** | When working with Azure, GCP, or multiple clouds | "Migrate this service from AWS to Azure" |
| **platform-architect** | When designing distributed systems or job queues | "Design the architecture to process 10,000 images per hour" |
| **databricks-specialist** | When using Databricks, Spark, or Delta Lake | "Create a DLT pipeline to process sensor data" |
| **data-scientist** | When working with ML models or statistical analysis | "Design a model to predict species presence" |
| **eo-cv-specialist** | When processing satellite imagery, camera traps, or bird audio | "Set up MegaDetector to process 50,000 camera trap photos" |
| **gis-specialist** | When working with maps, PostGIS, QGIS, or OGC services | "Publish the land cover layers as a WMS service" |

#### Domain advisor agents (5)

These agents provide expert advice on ecological and field topics. They don't write code — they give recommendations.

| Agent | When it activates | Example question |
|---|---|---|
| **field-ecologist** | Survey design, field protocols, camera traps | "How should I distribute 30 camera traps across a 500 km2 area?" |
| **bioacoustics-expert** | Acoustic monitoring, recorder selection, soundscape ecology | "What recorder should I use to monitor birds in a tropical rainforest?" |
| **conservation-biologist** | Biodiversity assessment, IUCN criteria, ecological indicators | "We have 2 years of data — how do we assess mammal community health?" |
| **remote-sensing-analyst** | Satellite data selection, land cover classification, change detection | "Which spectral index should I use to detect deforestation in lowland forest?" |
| **field-data-specialist** | KoboCollect, ODK, XLSForm, mobile data collection | "Design a KoboCollect form to record bird sightings" |

### How to delegate a task to an agent

There is no special process — simply **write what you need** in Claude Code. Here are the steps:

1. **Open Claude Code** in your project
2. **Type your request** in plain language. Be as specific as you can:
   - Instead of: *"Help with data"*
   - Better: *"I need to create a PostgreSQL table to store mammal detections with coordinates, date, species, and model confidence score"*
3. **Claude selects the agent** automatically and tells you which one it's using
4. **The agent works** on the task and shows you the results
5. **You review and approve** — Claude will ask for confirmation before making permanent changes

### Tips for getting better results

- **Be specific**: instead of "create a model", say "create a Bayesian occupancy model to estimate tapir presence probability using camera trap data from 40 sites"
- **Provide context**: mention what data you have, what format it uses, where it's stored
- **Describe the end goal**: "I need a species distribution map" is more useful than "process these rasters"
- **You can ask questions**: domain agents don't just execute tasks — they also advise. Ask "what approach do you recommend?" before jumping into implementation

---

## Part 2 — Commands

### How they work

Unlike agents (which activate on their own), commands are **invoked manually** by typing their name preceded by `/`. Each command runs a step-by-step workflow for a specific task.

### How to use a command

1. In Claude Code, type `/` followed by the command name
2. Claude will ask you questions to understand exactly what you need
3. Once it has all the information, it executes the full workflow

### Example: /scaffold-api

You type:
```
/scaffold-api
```

Claude asks:
- What resource is the endpoint for? (e.g., species, deployments, detections)
- What operations do you need? (list, create, edit, delete)
- What authentication does it require?

Then it automatically generates everything needed: data model, validations, views, routes, and tests.

### The 16 available commands

#### Web development

| Command | What it does | When to use it |
|---|---|---|
| `/scaffold-api` | Generates a complete API endpoint (model, view, route, tests) | When you need to add a new resource to your API |
| `/review-frontend` | Reviews React/TypeScript code for security and performance | Before deploying frontend changes |
| `/db-migrate` | Generates a safe, reversible database migration | When you need to change a table's structure |
| `/query-design` | Designs and optimizes SQL queries | When a query is slow or you need a new one |

#### Infrastructure and DevOps

| Command | What it does | When to use it |
|---|---|---|
| `/terraform-provision` | Generates Terraform modules for AWS resources | When you need to create new infrastructure |
| `/ansible-configure` | Generates Ansible roles and playbooks for server configuration | When you need to configure or update servers |
| `/controller-script` | Generates a controller script with start/stop/deploy/logs commands | When you need a single script to operate a service |
| `/aws-infra` | Reviews or designs AWS architecture with security and cost analysis | Before building or modifying infrastructure |
| `/docker-setup` | Generates Dockerfile and docker-compose configuration | When you need to containerize a service |
| `/iam-review` | Audits IAM policies for least privilege | When you want to verify permissions are secure |
| `/s3-data-flow` | Designs S3 data lake architecture | When you need to organize data in the cloud |

#### Data science and ML

| Command | What it does | When to use it |
|---|---|---|
| `/model-design` | Designs an ML model from problem framing to deployment plan | When you need to create a new predictive model |
| `/bayesian-network` | Designs and implements a Bayesian hierarchical model in PyMC | When the problem requires probabilistic modeling |

#### Ecological data processing

| Command | What it does | When to use it |
|---|---|---|
| `/eo-pipeline` | Generates a satellite imagery processing pipeline | When you need to process Sentinel-2 or similar data |
| `/camera-trap` | Sets up batch processing with MegaDetector | When you need to process thousands of camera trap photos |
| `/birdnet-audio` | Sets up batch processing with BirdNET-Analyzer | When you need to analyze audio recordings to identify birds |

---

## Part 3 — Planning mode

### What it is

Planning mode is a Claude Code feature (not exclusive to gallo-claudio) that lets you **design a plan before executing**. It's useful for complex tasks where you want to review and approve the approach before Claude starts writing code or making changes.

### When to use it

- Before a task that involves many files or steps
- When you're not sure which approach is best
- When you want to discuss options before committing to one
- For tasks where a mistake would be costly to revert

### How to activate it

You can request it in several ways using plain language:

> "Before making any changes, let's design a plan"

> "I want to plan how we're going to implement this"

> "Don't make changes yet — let's plan first"

Claude will enter planning mode: it presents a step-by-step plan, discusses options, and waits for your approval before executing any changes.

### Example planning workflow

1. **You**: "I need to add hydrophone data support to the system. Let's plan first."

2. **Claude** (in planning mode):
   - "Here's my proposed plan:
     1. Create a `HydrophoneRecording` model with fields for location, depth, and metadata
     2. Add an API endpoint to upload recordings
     3. Integrate with the BirdNET pipeline for marine mammal detection
     4. Create the database migration
   - Would you like me to modify anything in the plan?"

3. **You**: "Yes, I also need to store the water temperature for each recording"

4. **Claude**: Updates the plan and waits for your approval

5. **You**: "Approved, go ahead"

6. **Claude**: Executes the plan step by step, checking with you at key points

### Combining planning with agents

You can combine planning mode with specialist agents. For example:

> "Plan how we should design a multi-species occupancy model for the camera trap data from Project X"

Claude will activate the **data-scientist** agent in planning mode: it first presents the methodological approach, modeling options, and data requirements, and only implements once you give the green light.

---

## Part 4 — Quick reference

| I want to... | I do this |
|---|---|
| Ask a general question | Type my question directly — Claude picks the right agent |
| Run a specific workflow | Type `/command-name` (e.g., `/scaffold-api`) |
| Plan before acting | Ask "let's plan first" before the task |
| Get advice without code | Ask a question to a domain agent (e.g., about survey design) |
| Delegate a complete task | Describe the task with full context and let Claude work |

### Requirements

- [Claude Code](https://claude.ai/claude-code) installed
- gallo-claudio installed (see the project README)
- Be in a project directory so agents have context

### Getting help

If something doesn't work as expected:
- Verify the plugin is installed: type `/plugin` in Claude Code and look for `gallo-claudio@local`
- If an agent doesn't activate, try being more specific in your request
- To report issues: https://github.com/omaranda/gallo-claudio/issues
