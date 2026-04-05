# IBM Bob × watsonx Orchestrate — Bootcamp Lab
### RTA Dubai | AI Agent Development Bootcamp

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

## 1. What is IBM Bob?

**IBM Bob** is an AI-powered coding assistant (IDE extension) that acts as your **developer partner**. It can:

- Understand your requirements in plain English
- Generate project structures, code, and configuration files
- Access external documentation via **MCP (Model Context Protocol) servers**
- Plan, implement, and deploy end-to-end solutions

Bob works in three modes:

| Mode | What it does |
|------|-------------|
| **Ask** | Answer questions, explain code, show diagrams |
| **Plan** | Create structured implementation plans |
| **Advanced** | Full coding + MCP server access for deployment |

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
│ Tool         │  │ Data (DocProc Node)  │
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
| Airline Info | Airline Name, Passenger Name, Ticket Number, Flight Details |
| Hotel Info | Hotel Name, Customer Name, City |
| Fee Info | Base Fare, Taxes (breakdown), Total Amount, Currency |

---

## 4. Prerequisites

Before starting, make sure you have:

- [ ] **IBM Bob IDE** installed and open (VS Code extension)
- [ ] A **watsonx Orchestrate** environment activated (SaaS IBM Cloud or local Developer Edition)
- [ ] **Python 3.10+** installed
- [ ] **`uv`** package manager installed (`pip install uv`)
- [ ] Your **wxO API key** ready

> **Tip:** Your trainer will provide the wxO environment URL and API key.

---

## Step 1 — Configure MCP Servers in Bob

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

### Your `.bob` folder structure should look like this:

```
.bob/
├── mcp.json
└── rules/
    └── wxo-development.md
```

---

## Step 3 — Generate the Implementation Plan

Now we tell Bob what to build. Bob will create a complete plan before writing any code.

### 3.1 Switch Bob to Plan mode

In the Bob panel (bottom right of your IDE), click the mode selector and choose **Plan**.

Make sure **Auto-approval is OFF** so you can review each step.

### 3.2 Send this prompt to Bob

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

### 3.3 Approve Bob's requests

Bob will ask permission to read files and access MCP servers. Click **Approve** for each:

| Request | Action |
|---------|--------|
| Read `wxo-implementation-guide.md` | Approve |
| Access watsonx Orchestrate ADK docs MCP | Approve |
| Review the generated task list | Approve if it looks correct |

### 3.4 Answer Bob's clarification questions

Bob may ask questions like:

- *Receipt upload method?* → **Direct document input through the flow**
- *Output format?* → **Formatted JSON response in chat**
- *Deployment target?* → **watsonx Orchestrate SaaS (production)**

> Choose the **simplest option** for each question in this bootcamp.

### 3.5 Review the plan

Bob will generate:
- A task list (e.g., 7–10 tasks)
- An architecture design
- A workflow diagram (Mermaid format)

To view the diagram, switch to **Ask mode** and type:
```
Show me the workflow diagram
```

---

## Step 4 — Implement the Agent & Workflow

Now Bob writes all the code. Switch to **Advanced mode** (for MCP access).

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

### 4.3 Review key files

**`tools/expense_extraction_flow.py`** — the heart of the workflow:
- Defines the KVP schema (all fields to extract)
- Uses `docproc` node for document understanding
- Returns structured JSON

**`agents/expense_report_agent.yaml`** — the agent config:
- Sets the LLM model to `groq/openai/gpt-oss-120b`
- Links the flow as a tool
- Defines agent instructions and behavior

**`import-all.sh`** — the deployment script:
- Imports the flow tool to wxO
- Imports the agent YAML to wxO

> **Review each file before approving.** Click **Save** to accept each file Bob creates.

---

## Step 5 — Deploy to watsonx Orchestrate

### 5.1 Activate your wxO environment

1. Click the **watsonx Orchestrate extension tile** in the left sidebar
2. Click **Initialize the Workspace**
3. In the **Environment Manager**, find your environment and click **Activate**
4. Enter your **API key** when prompted in the command bar

You should see: *"Environment is now active!"*

### 5.2 Ask Bob to deploy

Send this prompt to Bob (still in Advanced mode):

```
Run the import script to deploy the flow and agent to my watsonx Orchestrate environment.
If the orchestrate command line is not installed, install ibm-watsonx-orchestrate with pip.
```

Bob will:
1. Install `ibm-watsonx-orchestrate` package if needed
2. Run `import-all.sh`
3. Import `expense_extraction_flow` tool → wxO
4. Import `expense_report_agent` → wxO

**Expected output:**
```
✅ Flow imported successfully
✅ Agent imported successfully
➜ Expense Report Agent deployed successfully!
```

---

## Step 6 — Test the Agent

### 6.1 Access watsonx Orchestrate

**SaaS (IBM Cloud):**
1. Log in to IBM Cloud
2. Go to **Resource List** → AI/Machine Learning
3. Find **Watson Orchestrate-itz** → click **Launch watsonx Orchestrate**

**Local Developer Edition:**
```bash
orchestrate chat start
# Opens http://localhost:3000/chat-lite
```

### 6.2 Find your agent

1. Go to **Manage Agents** (top navigation)
2. Search for: `expense_report_agent`
3. Confirm:
   - [ ] Model is **GPT-OSS 120B – OpenAI (via Groq)**
   - [ ] Tool **Expense Extraction Flow** is attached

### 6.3 Run a test

In the **Preview** chat panel on the right, type:

```
Extract my invoice details from flight.pdf
```

The agent will ask you to upload a file. Upload a sample airline invoice PDF.

**Expected JSON output:**

```json
{
  "invoice_date": "2026-02-06",
  "transaction_mode": "Credit Card",
  "currency": "USD",
  "airline_name": "Global Airways",
  "passenger_name": "Jane Smith",
  "ticket_number": "TKT-987654321098",
  "flight_details": "GA 2045, Cairo (CAI) → New Delhi (DEL)",
  "base_fare": "850.00",
  "fuel_surcharge": "125.00",
  "airport_taxes": "78.50",
  "total_amount": "1202.58"
}
```

The agent also returns a **human-readable summary** alongside the JSON.

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

- Bob acts as a **developer partner**, not just a code generator
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
*Based on IBM Bob T3 Paris 2025 materials by Allen Chan, Ahmed Azraq, Syeda Ameena Begum & Jukka Juselius*
