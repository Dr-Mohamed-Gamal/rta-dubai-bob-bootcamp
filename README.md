<p align="center">
  <img src="rta.jpg" alt="RTA Dubai Logo" width="250"/>
</p>

# IBM Bob × watsonx Orchestrate — Bootcamp Lab
### Roads & Transport Authority Dubai | AI Agent Development Bootcamp

> **Goal:** By the end of this lab, you will use IBM Bob as your AI development partner to build, deploy, and test an **Expense Report Agent** on watsonx Orchestrate — without writing a single line of code manually.

---

## Table of Contents

1. [What is IBM Bob?](#1-what-is-ibm-bob)
2. [What is watsonx Orchestrate?](#2-what-is-watsonx-orchestrate)
3. [Architecture Overview](#3-architecture-overview)
4. [Prerequisites](#4-prerequisites)
5. [Step 1 — Configure MCP Servers in Bob](#step-1--configure-mcp-servers-in-bob)
6. [Step 2 — Create a Bob Rule for wxO Best Practices](#step-2--create-a-bob-rule-for-wxo-best-practices)
7. [Step 3 — Generate the Implementation Plan](#step-3--generate-the-implementation-plan)
8. [Step 4 — Implement the Agent & Workflow](#step-4--implement-the-agent--workflow)
9. [Step 5 — Deploy to watsonx Orchestrate](#step-5--deploy-to-watsonx-orchestrate)
10. [Step 6 — Test the Agent](#step-6--test-the-agent)
11. [Summary & Next Steps](#summary--next-steps)

---

## Bob Modes — Quick Reference

> **You control which mode Bob operates in at all times. Bob will never switch modes on its own — you must do it manually.**

| Mode | Icon | When to use |
|------|------|-------------|
| **Ask** | 💬 | Questions, explanations, viewing diagrams |
| **Plan** | 📝 | Creating structured plans — no code written |
| **Advanced** | ⚡ | Writing code, accessing MCP servers, deploying |



---

## 1. What is IBM Bob?

**IBM Bob** is an AI-powered coding assistant (IDE extension) that acts as your **developer partner**. It can:

- Understand your requirements in plain English
- Generate project structures, code, and configuration files
- Access external documentation via **MCP (Model Context Protocol) servers**
- Plan, implement, and deploy end-to-end solutions

---

## 2. What is watsonx Orchestrate?

**IBM watsonx Orchestrate (wxO)** is an open, hybrid enterprise platform for **agentic AI**. It lets you build intelligent agents that can:

- Reason and make decisions
- Call external tools and APIs
- Process documents automatically
- Run structured, multi-step workflows

### Development approaches

| Approach | Description |
|----------|-------------|
| No-code | Drag-and-drop UI agent builder |
| Chat to build | Create agents via natural language prompting |
| **Pro-code (ADK)** | Full control via the Agent Development Kit — *what we use today* |
| Flow-builder | Visual workflow builder with Langflow |

---

## 3. Architecture Overview

We will build an **Transport Invoice Agent** that extracts structured data from RTA transport invoices.

```
User
 │
 │  uploads invoice (PDF / image)
 ▼
┌─────────────────────────────┐
│   Expense Report Agent      │  ← watsonx Orchestrate
│                             │
└────────────┬────────────────┘
             │ invokes
             ▼
┌─────────────────────────────┐
│   Invoice Processing Flow   │  ← Agentic Workflow (ADK)
└──────┬──────────────┬───────┘
       │ retrieves    │ processes
       ▼              ▼
┌──────────────┐  ┌──────────────────────┐
│ KVP Schema   │  │ Extract Structured   │
│ (inline)     │  │ Data (DocProc Node)  │
└──────────────┘  └──────────┬───────────┘
                             │ returns
                             ▼
                  ┌──────────────────────┐
                  │  Structured JSON     │
                  │  Output              │
                  └──────────────────────┘
```

### What gets extracted?

| Category | Fields |
|----------|--------|
| Invoice Info | Invoice Number, Invoice Date, Transaction Mode |
| Passenger Info | Passenger Name, Emirates ID |
| Transport Services | Service Type, Route, Zone, Fare per trip |
| Fee Info | Subtotal, VAT, Total Amount, Currency |

---

## 4. Prerequisites

Before starting, make sure you have:

- [ ] **IBM Bob IDE** installed
  1. Go to [bob.ibm.com](https://bob.ibm.com)
  2. Download the installer for your operating system (Mac / Windows / Linux)
  3. Run the installer and follow the on-screen instructions
  4. Once installed, launch Bob and **log in with your Gmail account**
- [ ] **watsonx Orchestrate SaaS environment** provisioned and accessible, with your **environment URL** and **API key** ready — no account yet? [Provision a free trial here](https://www.ibm.com/account/reg/us-en/signup?formid=urx-52753&cm_sp=ibmdev-_-developer-_-trial&utm_source=ibm_developer&utm_content=in_content_link&utm_id=tutorials_develop-agents-no-code-watsonx-orchestrate)

#### How to get your API key and Service instance URL

1. Log in to your watsonx Orchestrate environment
2. Click your **Profile icon** in the top-right corner
3. Click **Settings**
4. Click the **API details** tab
5. Click **Generate API key** — copy and save it somewhere safe
6. Copy the **Service instance URL** shown below the button

```
Service instance URL:
https://api.dl.watson-orchestrate.ibm.com/instances/<your-instance-id>
```

> You will need both the **API key** and the **Service instance URL** when activating your environment in Step 5.
- [ ] **Python 3.11** installed

---

### Install the watsonx Orchestrate ADK Extension

1. In Bob, click the **Extensions** icon in the left sidebar (or press `Cmd+Shift+X` on Mac / `Ctrl+Shift+X` on Windows)
2. In the search bar, type **`watsonx`**
3. Find **IBM watsonx Orchestrate** in the results and click **Install**
4. Wait for the installation to complete — you will see the **watsonx Orchestrate icon** appear in the left sidebar

> **You are now ready to start the lab.**

---

## Step 1 — Configure MCP Servers in Bob

> 🔧 **No specific Bob mode needed for this step — you are creating files manually.**

### What are MCP Servers?

**MCP (Model Context Protocol) servers** are external programs that extend IBM Bob's capabilities by providing:

- **Access to documentation** — Bob can search and retrieve information from IBM watsonx Orchestrate docs
- **Development tools** — Bob can list agents, export configurations, manage tools, and deploy to watsonx Orchestrate
- **Real-time information** — Bob gets up-to-date API references and code examples

Think of MCP servers as plugins that give Bob superpowers for specific tasks.

### The Two Servers We'll Configure

| Server Name | Type | Purpose |
|------------|------|---------|
| `watsonx-orchestrate-adk-docs` | Global | Search IBM watsonx Orchestrate documentation |
| `watsonx-orchestrate-adk` | Local | Access watsonx Orchestrate development tools (CLI commands) |

---

### 1.0 Install Required Dependencies

MCP servers require specific Python packages to run. We'll use Bob to install them.

Open Bob and send this message:

```
Please install the following dependencies needed for MCP servers:
1. Install uv package manager if not already installed
2. Install ibm-watsonx-orchestrate version 2.7.0
3. Install ibm-watsonx-orchestrate-mcp-server version 2.7.0
4. Install mcp-proxy
```

Bob will install:
- `uv/uvx` — Fast Python package manager
- `ibm-watsonx-orchestrate` (v2.7.0) — Core IBM watsonx Orchestrate library
- `ibm-watsonx-orchestrate-mcp-server` (v2.7.0) — MCP server for development tools
- `mcp-proxy` — Proxy for remote MCP servers

---

### 1.1 Configure the Global MCP Server

The global MCP server (`watsonx-orchestrate-adk-docs`) provides documentation search capabilities across all your projects.

1. In Bob, click the **Bob Settings icon** (⚙️) in the top right corner of the chat
2. In the Settings menu, click **MCP** in the left sidebar
3. Under "Global MCP", click the **Open** button
4. Paste the following configuration into the file and save (`Cmd+S` on Mac / `Ctrl+S` on Windows):

```json
{
  "mcpServers": {
    "watsonx-orchestrate-adk-docs": {
      "command": "uvx",
      "args": [
        "mcp-proxy",
        "--transport",
        "streamablehttp",
        "https://developer.watson-orchestrate.ibm.com/mcp"
      ],
      "alwaysAllow": [
        "search_ibm_watsonx_orchestrate_adk",
        "get_page_ibm_watsonx_orchestrate_adk"
      ],
      "disabled": false
    }
  }
}
```

---

### 1.2 Configure the Local MCP Server

The local MCP server (`watsonx-orchestrate-adk`) provides development tools specific to your current project.

In your project workspace, create a folder named **`.bob`**, then inside it create a file named **`mcp.json`** with the following content:

```json
{
  "mcpServers": {
    "watsonx-orchestrate-adk": {
      "command": "uvx",
      "args": [
        "--with",
        "ibm-watsonx-orchestrate==2.7.0",
        "ibm-watsonx-orchestrate-mcp-server==2.7.0"
      ],
      "env": {
        "WXO_MCP_WORKING_DIRECTORY": "/absolute/path/to/your/workspace",
        "WXO_MCP_DEBUG": ""
      },
      "disabled": false,
      "timeout": 300
    }
  }
}
```

> **Important:** Replace `/absolute/path/to/your/workspace` with the full absolute path to your project folder.

---

### 1.3 Test Both Servers (Optional)

→ **[View full testing steps](TESTING_MCP_SERVERS.md)**

---

## Step 2 — Create a Bob Rule for wxO Best Practices

> 🔧 **No specific Bob mode needed for this step — you are creating files manually.**

Bob rules are instructions that Bob **always follows** across all modes. We use them to make Bob an expert in watsonx Orchestrate patterns.

### 2.1 Create the rules directory

Inside `.bob`, create a subfolder named **`rules`**

### 2.2 Create the development rule file

Inside `.bob/rules`, create **`wxo-development.md`** with the following:

```markdown
## watsonx Orchestrate Development Rule

When working with IBM watsonx Orchestrate or watsonx Orchestrate ADK projects:

1. **Reference Guide**: Always check `wxo-implementation-guide.md` for:
   - Implementation patterns (Document Processing, User Activity, etc.)
   - Code examples and best practices
   - Mermaid diagram requirements

2. **Project Structure**: Follow ADK conventions:
   - `tools/` for flows and Python tools
   - `agents/` for YAML configurations
   - `import-all.sh` for deployment
   - `README.md` with architecture + workflow diagrams

3. **Key Patterns**:
   - Use `@flow` decorator for flows
   - Use `@tool` decorator for Python tools
   - Include inline KVP schemas for document processing
   - Create native agents for document handling
   - Use Command Line Interface to import agents and tools

4. **MCP Server Integration**:
   - Use `wxo-docs` MCP server to search IBM watsonx Orchestrate ADK documentation
   - Leverage MCP tools for finding API references, code examples, and implementation guides
   - Query the knowledge base when uncertain about ADK features or best practices
```

### Your `.bob` folder structure should now look like this:

```
.bob/
├── mcp.json
└── rules/
    └── wxo-development.md
```

---

## Step 3 — Generate the Implementation Plan

> 📝 **Switch Bob to PLAN mode before starting this step.**
>
> Click the mode selector at the bottom of the Bob panel and select **Plan**.
> Make sure **Auto-approval is OFF**.

### 3.1 Send this prompt to Bob

Copy and paste the following into the Bob chat:

```
Create a watsonx Orchestrate agent that will help a user create an expense report.

The agent will use a flow that:
1. Accepts an uploaded document file (PDF or image)
2. Extracts the expense fields below, validated with a KVP schema (Key-Value Pair Schema)
3. Returns the output in structured JSON format

Required Fields to Extract:

**Invoice Information:**
- Invoice Date
- Transaction Mode (e.g., Credit Card, Bank Transfer, Cash)

**Passenger Information:**
- Passenger Name
- Emirates ID

**Transport Services:**
- Service Type (Metro, Taxi, Tram, Bus)
- Route
- Zone
- Fare per trip

**Fee Information:**
- Subtotal
- VAT (5%)
- Total Amount
- Currency
```

### 3.2 Approve Bob's requests

Bob will ask permission to read files and access MCP servers. Click **Approve** for each:

| Request | Action |
|---------|--------|
| Read `wxo-implementation-guide.md` | ✅ Approve |
| Access watsonx Orchestrate ADK docs MCP | ✅ Approve |
| Review the generated task list | ✅ Approve if it looks correct |

### 3.3 Answer Bob's clarification questions

Bob may ask some clarifying questions like these:

**1. LLM Model:** Which LLM would you like to use for the agent?
(e.g., watsonx/ibm/granite-3-8b-instruct, watsonx/mistralai/mistral-small-3-1-24b-instruct-2503)

**2. Agent Style:** Which agent style would you prefer?
(default, react, or planner)

**3. Output Format:** Should the flow return:
- A document reference URL (best for large documents)
- An inline JSON object (best for small documents and immediate processing)

**4. Additional Features:** Would you like any of these features?
- Welcome message and starter prompts for the agent
- Guidelines for specific behaviors
- Error handling and validation messages

**5. Deployment:** Are you using watsonx Orchestrate Developer Edition locally, or a cloud instance?

> **Answer with this:** `gpt-oss` for the LLM model, and select the **default** option for all remaining questions.

### 3.4 Review the plan

Bob will generate a complete plan. Once it appears, **stay in Plan mode** and move to the next step.

---

## Step 3b — View the Workflow Diagram

> 💬 **Switch Bob to ASK mode now.**
>
> Click the mode selector and select **Ask**.

Type this into Bob:

```
Show me the workflow diagram
```

Bob will render a Mermaid diagram of the full flow. Review it and confirm the architecture matches what you expect before proceeding.

> 💡 If Mermaid diagrams don't render in your IDE, install the **Mermaid Preview** VS Code extension.

---

## Step 4 — Implement the Agent & Workflow

> ⚡ **Switch Bob to ADVANCED mode now.**
>
> Click the mode selector and select **Advanced**. This gives Bob access to both MCP servers for code generation and deployment.

### 4.1 Send this implementation prompt to Bob

```
Implement the approved plan and follow the instructions below:

**Requirements:**
1. Create a native agent with this specific LLM model: groq/openai/gpt-oss-120b
2. Build a document processing flow using a docproc node
3. Define a KVP schema for the fields I need to extract
4. For simplicity, include the KVP schema inline in the flow file
5. Ensure all imports are relative (using dot notation)

**Important Implementation Details:**
- Use plain functions for schema helpers (no @tool decorator)
- Use relative imports in all Python files
- Keep the flow simple with just a docproc node
- Use native agent type for better document handling
- Use JSON output format (not file reference)
```

### 4.2 What Bob will create

Bob will generate the following project structure automatically:

```
expense-report-agent/
├── __init__.py                        # Package initialization
├── README.md                          # Documentation with diagrams
├── flow_main.py                       # Programmatic testing script
├── import-all.sh                      # CLI deployment script
├── tools/
│   ├── __init__.py
│   └── expense_extraction_flow.py    # Document processing flow + inline KVP schema
├── agents/
│   └── expense_report_agent.yaml     # Native agent
└── generated/                         # Auto-generated flow specs
    └── .gitkeep
```

### 4.3 Review key files as Bob creates them

For each file Bob creates, review it then click **Save**:

| File | What to check |
|------|--------------|
| `expense_extraction_flow.py` | KVP schema fields are all present, `@flow` decorator used, `docproc` node configured |
| `expense_report_agent.yaml` | `kind: native`, flow is listed under `tools` |
| `import-all.sh` | Both `tools import` and `agents import` commands are present |


---

## Step 5 — Deploy to watsonx Orchestrate

> ⚡ **Stay in ADVANCED mode for this step.**

### 5.0 Set up Python virtual environment

> ⚡ **Stay in Advanced mode for this step.**

Send this prompt to Bob:

```
Please help me:
1. Install a standalone Python 3.11 (if not existing) (no conda)
2. Set up a standard Python virtual environment using that Python version
```

Bob will run the necessary terminal commands to install Python 3.11 and create a virtual environment in your project directory.

---

### 5.1 Activate your wxO environment

1. Click the **watsonx Orchestrate extension tile** in the left sidebar
2. Click **Initialize the Workspace**
3. In the **Environment Manager**, find your environment and click **Activate**
4. Enter your **API key** when prompted in the command bar at the top

You should see: *"Environment is now active!"*

### 5.2 Send this deployment prompt to Bob

```
Run the import script to deploy the flow and agent to my watsonx Orchestrate environment.
If the orchestrate command line is not installed, install ibm-watsonx-orchestrate with pip.
```

Bob will:
1. Install `ibm-watsonx-orchestrate` if needed
2. Run `import-all.sh`
3. Import `expense_extraction_flow` tool → wxO
4. Import `expense_report_agent` → wxO

**Expected terminal output:**
```
✅ Flow imported successfully
✅ Agent imported successfully
➜ Expense Report Agent deployed successfully!
```

---

## Step 6 — Test the Agent

> 💬 **Switch Bob to ASK mode now** (or close Bob — this step is done in the wxO UI).

### 6.1 Access watsonx Orchestrate

1. Log in to **IBM Cloud**
2. Go to **Resource List** → AI/Machine Learning
3. Find **Watson Orchestrate-itz** → click **Launch watsonx Orchestrate**

### 6.2 Find your agent

1. Go to **Manage Agents** (top navigation)
2. Search for: `expense_report_agent`
3. Confirm:
   - [ ] Model is configured
   - [ ] Tool **Expense Extraction Flow** is attached under Toolset

### 6.3 Run a test

In the **Preview** chat panel on the right, type:

```
Extract my transport invoice details from rta_transport_invoice.pdf and output the results in table format
```

The agent will prompt you to upload a file. Upload the **`rta_transport_invoice.pdf`** file included in this repo and click **Send**.

**Expected JSON output:**

```json
{
  "invoice_number": "RTA-INV-2026-00472",
  "invoice_date": "05 April 2026",
  "transaction_mode": "NOL Card",
  "currency": "AED",
  "passenger_name": "Ahmed Al Mansoori",
  "emirates_id": "784-1990-7654321-2",
  "services": [
    { "date": "05 Apr 2026", "service": "Dubai Metro", "route": "Union to DIFC", "zone": "Zone 2", "fare": "5.80" },
    { "date": "05 Apr 2026", "service": "RTA Taxi",    "route": "DIFC to RTA HQ", "zone": "N/A",    "fare": "18.50" },
    { "date": "06 Apr 2026", "service": "Dubai Metro", "route": "BurJuman to Airport", "zone": "Zone 3", "fare": "8.50" },
    { "date": "06 Apr 2026", "service": "Dubai Tram",  "route": "Al Sufouh to JBR", "zone": "Zone 1", "fare": "3.00" }
  ],
  "subtotal": "35.80",
  "vat": "1.79",
  "total_amount": "37.59"
}
```

The agent also returns a **human-readable summary** alongside the JSON.

---

## Mode Summary — What You Used and When

| Step | Bob Mode | Why |
|------|----------|-----|
| Steps 1 & 2 — File setup | None (manual) | Creating config files directly |
| Step 3 — Implementation plan | 📝 **Plan** | Bob plans without writing code |
| Step 3b — View diagram | 💬 **Ask** | Bob answers questions, no side effects |
| Step 4 — Write all code | ⚡ **Advanced** | Bob needs MCP access to generate & write files |
| Step 5 — Deploy | ⚡ **Advanced** | Bob runs terminal commands via MCP |
| Step 6 — Test | 💬 **Ask** (or none) | Testing happens in wxO UI |

---

## Summary & Next Steps

### What you built today

| Component | Description |
|-----------|-------------|
| **Bob Rules** | Guided Bob with watsonx Orchestrate best practices |
| **Implementation Plan** | Bob designed the full architecture |
| **Expense Extraction Flow** | Document processing flow with inline KVP schema |
| **Expense Report Agent** | Native agent on watsonx Orchestrate |
| **Deployment Script** | One-click deploy via `import-all.sh` |
| **Live Agent** | Running in watsonx Orchestrate, tested with a real invoice |

### Key takeaways

- **You control Bob's mode** — always switch manually, never let Bob switch for you
- **Bob Rules** ensure consistent patterns and reduce hallucinations
- **MCP servers** give Bob live access to wxO documentation and deployment tools
- **ADK (Agent Development Kit)** enables pro-code, production-ready agent development
- The full workflow — from requirements to deployed agent — took **minutes, not days**

### Ideas to extend this lab

- [ ] Add a **hotel invoice** extraction test
- [ ] Connect the agent to an **expense management system** via API tool
- [ ] Add a **validation step** that flags missing or suspicious fields
- [ ] Schedule the agent to **process invoices automatically** on arrival
- [ ] Deploy to **Slack or Microsoft Teams** as a channel

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| MCP server not connecting | Check `WXO_MCP_WORKING_DIRECTORY` is an absolute path; verify JSON files have no typos |
| Global server not found | Open Bob Settings → MCP → verify the global config was saved correctly |
| `orchestrate` command not found | Run `pip install ibm-watsonx-orchestrate` |
| Dependencies not installed | Ask Bob to install dependencies again; check your internet connection |
| Local server shows no tools | Normal before authentication — activate your wxO environment in Step 5 |
| Agent not found after deploy | Re-run `import-all.sh` and refresh the wxO UI |
| API key rejected | Ensure you're using the correct TechZone account |
| Flow import fails | Check relative imports in `expense_extraction_flow.py` |

---

## Resources

| Resource | Link |
|----------|------|
| IBM Bob documentation | Available via MCP server in Bob IDE |
| watsonx Orchestrate ADK docs | Via `watsonx-orchestrate-adk-docs` MCP server |
| IBM Developer tutorials | IBM Developer portal |
| wxO ADK GitHub | IBM GitHub organization |

---

*Lab prepared for RTA Dubai AI Agent Bootcamp*  

