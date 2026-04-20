
# HR Workflow Designer

 
> A visual drag-and-drop workflow builder for HR admins — design, configure, and simulate internal HR workflows like onboarding, leave approval, and document verification.

---

## Features

| Feature | Status |
|---|---|
| Drag-and-drop canvas (React Flow) | ✅ |
| 5 custom node types with distinct forms | ✅ |
| Node configuration / edit forms | ✅ |
| Mock API layer (`GET /automations`, `POST /simulate`) | ✅ |
| Workflow sandbox with step-by-step execution log | ✅ |
| Real-time validation with error indicators on nodes | ✅ |
| Export workflow as JSON | ✅ |
| Import workflow from JSON | ✅ bonus |
| Undo / Redo (30-step history) | ✅ bonus |
| Keyboard shortcuts (⌘Z, ⌘Y, Delete) | ✅ bonus |
| Right-click context menu (duplicate, delete) | ✅ bonus |
| MiniMap + zoom controls | ✅ bonus |

---

## Quick Start

```bash
git clone https://github.com/YOUR_USERNAME/hr-workflow-designer.git
cd hr-workflow-designer
npm install
npm run dev
# → http://localhost:5173
```

**Requirements:** Node.js 18+

---

## How to Use

1. **Add nodes** — drag from the left sidebar onto the canvas, or click a node type to auto-place it
2. **Connect nodes** — drag from the blue handle on the right of a node to another node
3. **Edit nodes** — click any node to open its config form in the right panel
4. **Validate** — the Sandbox tab shows real-time validation errors
5. **Simulate** — click "Run Simulation" to see a step-by-step execution log
6. **Export** — save your workflow as JSON and import it back later

**Keyboard shortcuts:** `⌘Z` undo · `⌘Y` redo · `Delete` remove selected node · `Right-click` context menu

---

## Project Structure

```
src/
├── types/index.ts              # All TypeScript interfaces (NodeData union, API types, etc.)
├── api/mockApi.ts              # GET /automations + POST /simulate (local mocks)
├── store/workflowStore.ts      # Zustand store — nodes, edges, selectedNodeId, undo/redo history
├── hooks/index.ts              # useNodeFactory, useWorkflowValidation, useSimulation,
│                               # useWorkflowExport, useKeyboardShortcuts
├── lib/
│   ├── nodeConfig.ts           # Node type metadata (colors, icons, labels)
│   └── nanoid.ts               # Tiny ID generator
└── components/
    ├── nodes/
    │   ├── BaseNode.tsx         # Shared node shell (badge, title, handles, error dot)
    │   └── index.tsx            # 5 node components + nodeTypes map for React Flow
    ├── panels/
    │   ├── Sidebar.tsx          # Node palette, drag + click-to-add, canvas stats
    │   ├── RightPanel.tsx       # Edit / Sandbox tabs, timeline log
    │   └── NodeForms.tsx        # Per-type config forms
    ├── canvas/
    │   ├── WorkflowCanvas.tsx   # React Flow wrapper, toolbar, drop handler
    │   └── ContextMenu.tsx      # Right-click context menu
    └── ui/index.tsx             # Input, Select, Textarea, Toggle, KVEditor, etc.
```

---

## Architecture & Design Decisions

### State — Zustand
Flat store with selector subscriptions. Each component subscribes only to the slice it needs. The entire workflow state (nodes, edges, selectedNodeId, undo/redo history) lives in one store.

### Undo/Redo — Manual History Stack
`past: Snapshot[]` and `future: Snapshot[]` in the store. Every destructive action pushes to `past`. Undo pops from `past`, pushes current to `future`. Capped at 30 snapshots.

### React Flow Integration
Nodes/edges live in Zustand as controlled state, passed as props to React Flow. Base `Node[]` type used for React Flow props to avoid generic variance issues; typed `WorkflowNode` stays in the store.

### Discriminated Union Node Data
Each node type has its own interface with a `nodeType` discriminant (e.g. `StartNodeData`, `TaskNodeData`). All extend `Record<string, unknown>` to satisfy React Flow's generic constraint. The discriminant enables exhaustive type switching in forms and simulation.

### Mock API
`mockApi.ts` exports two async functions with simulated delays:
- `getAutomations()` — returns 5 actions, each with a `params` array that drives dynamic field generation
- `simulateWorkflow()` — validates structure, traverses the graph, returns typed `SimulateResponse`

Fully isolated — swap for real HTTP calls by only changing `mockApi.ts`.

---

## Node Types

| Node | Purpose | Key fields |
|------|---------|------------|
| Start | Entry point | Title, metadata key-value pairs |
| Task | Human task | Title, description, assignee, due date, custom fields |
| Approval | Approval gate | Title, approver role, auto-approve threshold (days) |
| Automated Step | System action | Title, action from API, dynamic action parameters |
| End | Completion | End message, generate summary report toggle |

---

## Mock API Reference

### GET /automations
```json
[
  { "id": "send_email",    "label": "Send Email",         "params": ["to", "subject", "body"] },
  { "id": "generate_doc",  "label": "Generate Document",  "params": ["template", "recipient"] },
  { "id": "notify_slack",  "label": "Notify Slack",       "params": ["channel", "message"] },
  { "id": "update_hris",   "label": "Update HRIS Record", "params": ["employee_id", "field", "value"] },
  { "id": "create_ticket", "label": "Create Ticket",      "params": ["title", "assignee", "priority"] }
]
```

### POST /simulate
```json
{
  "success": true,
  "steps": [
    { "nodeType": "start", "title": "Onboarding Start", "status": "success", "detail": "Workflow initiated", "durationMs": 52 }
  ],
  "errors": [],
  "totalDurationMs": 2540
}
```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| React 19 + TypeScript 5 | UI + type safety |
| Vite 6 | Build tool |
| @xyflow/react 12 | Canvas, nodes, edges |
| Zustand 5 | Global state + undo/redo |
| Tailwind CSS 4 | Utility styling |

---

## What I Would Add With More Time

- Conditional edges — branch on approval outcome (approved / rejected)
- Auto-layout — Dagre-based automatic node positioning
- Node templates — pre-configured bundles (e.g. "Leave Approval" workflow)
- Real backend — FastAPI + PostgreSQL to persist and version workflows
- Workflow versioning — named snapshots

---


