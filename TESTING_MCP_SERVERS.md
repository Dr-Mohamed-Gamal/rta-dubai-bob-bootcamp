<p align="center">
  <img src="rta.jpg" alt="RTA Dubai Logo" width="250"/>
</p>

# Testing MCP Servers
### Roads & Transport Authority Dubai | AI Agent Development Bootcamp

> **Goal:** Verify that both MCP servers are working correctly before proceeding with the lab.

> ⚡ **Switch Bob to ADVANCED mode now.**

---

## Step 1 — Restart IBM Bob

1. Close IBM Bob completely (not just the window — quit the application)
2. Reopen IBM Bob
3. Open your project folder in Bob

> **Why restart?** Bob reads MCP configuration files only when it starts. Restarting ensures Bob loads your new configurations.

---

## Step 2 — Test the Global Documentation Server

In Bob's chat, type:

```
Can you search the watsonx Orchestrate documentation for "flow decorator"?
```

**Expected result:**

Bob should return search results from IBM's watsonx Orchestrate documentation, showing links and snippets about the `@flow` decorator.

**Example output:**
```
I found several results about flow decorators:

1. Building agentic workflows
   Link: https://developer.watson-orchestrate.ibm.com/tools/flows/building_flow
   Content: Use the @flow decorator to define a workflow...

2. Flow examples
   Link: https://developer.watson-orchestrate.ibm.com/examples/flows
   Content: Python function with the @flow decorator...
```

---

## Step 3 — Test the Local Development Server

In Bob's chat, type:

```
Can you list the available watsonx Orchestrate development tools you have access to?
```

**Expected result:**

Bob should list the tools from the `watsonx-orchestrate-adk` server, such as:
- `list_agents`
- `export_agent`
- `list_tools`
- `list_toolkits`
- `check_version`

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "No servers available" | Check that `.bob/mcp.json` is in the correct project folder and restart Bob |
| Global server not responding | Open Bob Settings → MCP → verify the global config was saved correctly |
| Local server shows no tools | Normal before authentication — activate your wxO environment in Step 5 of the main lab |
| JSON parse error | Use a JSON validator to check for typos in your config files |

---

[← Back to Main Lab](README.md)
