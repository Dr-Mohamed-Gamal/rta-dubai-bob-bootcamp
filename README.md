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

We will build an **Expense Report Agent** that extracts structured data from airline and hotel invoices.

```
User
 │
 │  uploads invoice (PDF / image)
 ▼
┌─────────────────────────────┐
│   Expense Report Agent      │  ← watsonx Orchestrate
│   (LLM: GPT-OSS 120B/Groq) │
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
| Invoice Info | Invoice Date, Transaction Mode |
| Airline Info | Airline Name, Passenger Name, Ticket Number, Ticket Date, Flight Details |
| Hotel Info | Hotel Name, Customer Name, City |
| Fee Info | Base Fare, Taxes (breakdown), Total Amount, Currency |

---

## 4. Prerequisites

Before starting, make sure you have:

- [ ] **IBM Bob IDE** installed and open (VS Code extension)
- [ ] A **watsonx Orchestrate** environment ready (SaaS IBM Cloud or local Developer Edition)
- [ ] **Python 3.10+** installed
- [ ] **`uv`** package manager installed (`pip install uv`)
- [ ] Your **wxO API key** ready

> **Tip:** Your trainer will provide the wxO environment URL and API key.

---

## Step 1 — Configure MCP Servers in Bob

> 🔧 **No specific Bob mode needed for this step — you are creating files manually.**

MCP servers give Bob direct access to watsonx Orchestrate documentation and commands.

### 1.1 Create the configuration folder

In your project workspace, create a folder named **`.bob`**

### 1.2 Create the MCP config file

Inside `.bob`, create a file named **`mcp.json`** with the following content:

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
      ]
    },
    "watsonx-orchestrate-adk": {
      "command": "uvx",
      "args": [
        "ibm-watsonx-orchestrate-mcp-server"
      ],
      "env": {
        "WXO_MCP_WORKING_DIRECTORY": "/absolute/path/to/your/workspace",
        "WXO_MCP_DEBUG": ""
      },
      "timeout": 300
    }
  }
}
```

> **Important:** Replace `/absolute/path/to/your/workspace` with the full path to your project folder.

### What each server does

| Server | Purpose |
|--------|---------|
| `watsonx-orchestrate-adk-docs` | Gives Bob access to wxO developer documentation |
| `watsonx-orchestrate-adk` | Gives Bob direct CLI access to create/deploy agents & tools |

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
2. Extracts the expense fields below, validated with a KVP schema
3. Returns the output in structured JSON format

Required Fields to Extract:

**Invoice Information:**
- Invoice Date
- Transaction Mode (e.g., Credit Card, Bank Transfer, Cash)

**Airline and Passenger Information (if airline invoice):**
- Airline Name
- Passenger Name
- Ticket Number
- Ticket Date
- Flight Details

**Hotel Information (if accommodation invoice):**
- Hotel Name
- Customer Name
- City

**Fee Information:**
- Base Fare / Charges
- Taxes (with breakdown if available)
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

Bob may ask questions like:

- *Receipt upload method?* → **Direct document input through the flow**
- *Output format?* → **Formatted JSON response in chat**
- *Deployment target?* → **watsonx Orchestrate SaaS (production)**
- *Unknown invoice type handling?* → **Extract only common fields and return a note**

> Choose the **simplest option** for each question.

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
│   └── expense_report_agent.yaml     # Native agent with GPT-OSS 120B
└── generated/                         # Auto-generated flow specs
    └── .gitkeep
```

### 4.3 Review key files as Bob creates them

For each file Bob creates, review it then click **Save**:

| File | What to check |
|------|--------------|
| `expense_extraction_flow.py` | KVP schema fields are all present, `@flow` decorator used, `docproc` node configured |
| `expense_report_agent.yaml` | `llm: groq/openai/gpt-oss-120b`, `kind: native`, flow is listed under `tools` |
| `import-all.sh` | Both `tools import` and `agents import` commands are present |


---

## Step 5 — Deploy to watsonx Orchestrate

> ⚡ **Stay in ADVANCED mode for this step.**

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
   - [ ] Model shows **GPT-OSS 120B – OpenAI (via Groq)**
   - [ ] Tool **Expense Extraction Flow** is attached under Toolset

### 6.3 Run a test

In the **Preview** chat panel on the right, type:

```
Extract my invoice details from flight_invoice.pdf
```

The agent will prompt you to upload a file. Upload the **`flight_invoice.pdf`** file included in this repo and click **Send**.

**Expected JSON output:**

```json
{
  "invoice_date": "2026-02-06",
  "transaction_mode": "Credit Card",
  "currency": "USD",
  "airline_name": "Global Airways",
  "passenger_name": "Jane Smith",
  "ticket_number": "TKT-987654321098",
  "ticket_date": "2026-02-05",
  "flight_details": "GA 2045, Cairo (CAI) → New Delhi (DEL)",
  "base_fare": "850.00",
  "taxes": "203.58",
  "total_amount": "1202.58"
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
| **Expense Report Agent** | Native agent using GPT-OSS 120B via Groq |
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
| MCP server not connecting | Check `WXO_MCP_WORKING_DIRECTORY` is an absolute path |
| `orchestrate` command not found | Run `pip install ibm-watsonx-orchestrate` |
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
*Consolidated by Dr. Mohamed Gamal - IBM Client Engineering*
