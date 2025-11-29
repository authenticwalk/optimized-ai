# Optimized-AI: Architecture & Design

## Project Philosophy

**Core Principles**:
1. Clean code - no bloat, no orphaned files
2. Small LLM context - don't pollute with irrelevant data
3. Self-learning over planning - the real gains are in learning from mistakes
4. Shared plans - orchestrator and subagents see the same file, "you're on step X"

---

## Database Decision: Firebase vs Alternatives

### Option 1: Firebase (Firestore + Realtime Database)

**Pros**:
- ✅ Works everywhere (machine, phone, web, any portal)
- ✅ Real-time listeners - dashboard shows live updates
- ✅ No server to manage
- ✅ Generous free tier (50K reads/day, 20K writes/day)
- ✅ Offline support with sync when reconnected
- ✅ Authentication built-in (useful for multi-device)
- ✅ Cloud Functions for scheduled tasks (nightly learning)

**Cons**:
- ~~⚠️ No native vector search~~ **RESOLVED: Firebase now has native vector search!**
- ⚠️ Query limitations (no full-text search, limited joins)
- ⚠️ Costs scale with reads (embedding lookups = many reads)
- ⚠️ Vendor lock-in to Google

---

## Firebase Vector Search (Native Support)

**Source**: [Firebase Vector Search Docs](https://firebase.google.com/docs/firestore/vector-search)

Firebase Firestore now supports native K-nearest neighbor (KNN) vector search. This eliminates the need for a separate vector database.

### Storing Embeddings

```javascript
import { FieldValue } from 'firebase-admin/firestore';

// Store a learning with its embedding
const learningsRef = db.collection('learnings');
await learningsRef.add({
  lesson: "When refactoring auth, check JWT expiry logic first",
  context: "auth refactor",
  keywords: ["auth", "jwt", "refactor"],
  confidence: 0.9,
  createdAt: FieldValue.serverTimestamp(),

  // Vector embedding (384 dimensions from Xenova)
  embedding: FieldValue.vector([0.12, -0.34, 0.56, ...])  // 384 floats
});
```

### Creating Vector Index

Before querying, create an index via gcloud CLI:

```bash
gcloud firestore indexes composite create \
  --collection-group=learnings \
  --field-config field-path=embedding,vector-config='{"dimension":"384", "flat": "{}"}' \
  --database="(default)"
```

**Note**: Dimension must match your embedding model (Xenova = 384).

### Querying with find_nearest

```javascript
// Find similar learnings
const queryEmbedding = await xenova.embed("auth refactor task");

const similarLearnings = await db.collection('learnings')
  .findNearest({
    vectorField: 'embedding',
    queryVector: queryEmbedding,
    limit: 5,
    distanceMeasure: 'COSINE'  // or 'EUCLIDEAN', 'DOT_PRODUCT'
  });

// Can combine with filters
const filteredResults = await db.collection('learnings')
  .where('confidence', '>=', 0.7)
  .findNearest({
    vectorField: 'embedding',
    queryVector: queryEmbedding,
    limit: 5,
    distanceMeasure: 'COSINE'
  });
```

### Distance Measures

| Measure | Best For | Notes |
|---------|----------|-------|
| COSINE | Most embeddings | Normalized, recommended default |
| EUCLIDEAN | Absolute distance | Good for spatial data |
| DOT_PRODUCT | Already normalized | Slightly faster if pre-normalized |

### Embedding Strategy

We'll use **Xenova/all-MiniLM-L6-v2** (384 dimensions):
- Runs locally (no API costs)
- Fast (~50ms per embedding)
- Good quality for semantic similarity

```javascript
// hooks/lib/embeddings.js
import { pipeline } from '@xenova/transformers';

let embedder = null;

export async function getEmbedding(text) {
  if (!embedder) {
    embedder = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
  }

  const output = await embedder(text, { pooling: 'mean', normalize: true });
  return Array.from(output.data);  // 384-dim float array
}

### Option 2: Supabase (PostgreSQL)

**Pros**:
- ✅ Real PostgreSQL (full SQL, pgvector for embeddings)
- ✅ Real-time subscriptions
- ✅ Self-hostable (no vendor lock-in)
- ✅ Row-level security
- ✅ pgvector = native vector similarity search

**Cons**:
- ⚠️ Less mature than Firebase
- ⚠️ Slightly more complex setup
- ⚠️ Free tier is smaller

### Option 3: Hybrid (Firebase + Pinecone/Qdrant)

**Pros**:
- ✅ Firebase for real-time state
- ✅ Dedicated vector DB for embeddings
- ✅ Best-of-breed for each concern

**Cons**:
- ⚠️ Two services to manage
- ⚠️ More complex sync logic
- ⚠️ Higher cost

### Recommendation

**Firebase Firestore** with these considerations:
1. Store structured data (sessions, learnings, plans) in Firestore
2. For embeddings: Use Google's Vertex AI Vector Search OR store embeddings as arrays and filter in Cloud Functions
3. Cache aggressively to reduce read costs

**Rationale**: Your requirements prioritize real-time sync across devices and live dashboards. Firebase excels at this. The embedding limitation can be worked around.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         YOUR DEVICES                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐    │
│  │ Local    │  │  Phone   │  │ Claude   │  │ Other Portals    │    │
│  │ Machine  │  │  App     │  │ Code Web │  │ (Cursor, etc)    │    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬─────────┘    │
│       │             │             │                  │              │
│       └─────────────┴──────┬──────┴──────────────────┘              │
│                            │                                         │
│                    ┌───────▼───────┐                                │
│                    │  Local Agent  │  ← Runs on your machine        │
│                    │  (CLI/Hook)   │    Intercepts Claude Code      │
│                    └───────┬───────┘                                │
└────────────────────────────┼────────────────────────────────────────┘
                             │
                             │ HTTPS/WebSocket
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│                        FIREBASE                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     Firestore                                │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │   │
│  │  │   sessions   │  │  learnings   │  │   plans (live)   │  │   │
│  │  │              │  │              │  │                  │  │   │
│  │  │ - id         │  │ - id         │  │ - id             │  │   │
│  │  │ - startTime  │  │ - embedding  │  │ - steps[]        │  │   │
│  │  │ - status     │  │ - lesson     │  │ - currentStep    │  │   │
│  │  │ - outcome    │  │ - confidence │  │ - agentSees      │  │   │
│  │  │ - summary    │  │ - appliesTo  │  │ - lastUpdated    │  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘  │   │
│  │                                                              │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │   │
│  │  │command_cache │  │  dashboard   │  │   skill_patterns │  │   │
│  │  │              │  │   (live)     │  │                  │  │   │
│  │  │ - hash       │  │ - active     │  │ - pattern        │  │   │
│  │  │ - safe:bool  │  │ - running[]  │  │ - successRate    │  │   │
│  │  │ - reason     │  │ - metrics    │  │ - usageCount     │  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   Cloud Functions                            │   │
│  │                                                              │   │
│  │  ┌──────────────────┐  ┌────────────────────────────────┐  │   │
│  │  │ securityCheck()  │  │ nightlyLearner()               │  │   │
│  │  │                  │  │                                │  │   │
│  │  │ - Calls Gemini   │  │ - Consolidates learnings       │  │   │
│  │  │   Flash          │  │ - Prunes low-confidence        │  │   │
│  │  │ - Returns safe/  │  │ - Extracts skill patterns      │  │   │
│  │  │   unsafe         │  │ - Runs on schedule             │  │   │
│  │  │ - Caches result  │  │                                │  │   │
│  │  └──────────────────┘  └────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Component Design

### 1. Security Check (Command Approval)

**Goal**: Never prompt for approval. Use fast LLM to assess risk, cache results.

```
┌─────────────────────────────────────────────────────────┐
│                   PreToolUse Hook                        │
│                                                          │
│  Command: "rm -rf node_modules"                         │
│                    │                                     │
│                    ▼                                     │
│  ┌──────────────────────────────────┐                   │
│  │  1. Hash the command             │                   │
│  │  2. Check Firebase cache         │──── HIT ────────► │ Allow/Block
│  │  3. If miss, call Gemini Flash   │                   │
│  └──────────────────────────────────┘                   │
│                    │ MISS                               │
│                    ▼                                     │
│  ┌──────────────────────────────────┐                   │
│  │  Gemini Flash 2.0 (fast, cheap)  │                   │
│  │                                  │                   │
│  │  Prompt:                         │                   │
│  │  "Is this command safe?          │                   │
│  │   Command: {command}             │                   │
│  │   Context: {cwd}, {user}         │                   │
│  │                                  │                   │
│  │   Reply: SAFE or RISKY: reason"  │                   │
│  └──────────────────────────────────┘                   │
│                    │                                     │
│                    ▼                                     │
│  ┌──────────────────────────────────┐                   │
│  │  Cache result in Firebase        │                   │
│  │  command_cache/{hash}            │                   │
│  │  { safe: true, reason: "..." }   │                   │
│  └──────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

**Cache Strategy**:
- Hash: `sha256(command + cwd_pattern)`
- Normalize paths (replace specific paths with patterns)
- TTL: 7 days (commands don't become dangerous over time)
- Warm cache with common commands on first run

---

### 2. Session Start: Learning Retrieval

**Goal**: Before starting work, spawn Opus subagent to find relevant learnings.

```
┌─────────────────────────────────────────────────────────┐
│                   Session Start                          │
│                                                          │
│  User: "Help me refactor the auth module"               │
│                    │                                     │
│                    ▼                                     │
│  ┌──────────────────────────────────┐                   │
│  │  Orchestrator (Haiku - fast)     │                   │
│  │                                  │                   │
│  │  1. Create plan document         │                   │
│  │  2. Spawn Learning Retriever     │                   │
│  │  3. Wait for learnings           │                   │
│  │  4. Inject into context          │                   │
│  │  5. Begin work                   │                   │
│  └──────────────────────────────────┘                   │
│                    │                                     │
│                    ▼                                     │
│  ┌──────────────────────────────────┐                   │
│  │  Learning Retriever (Opus)       │  ← Deep thinking  │
│  │                                  │                   │
│  │  1. Parse task intent            │                   │
│  │  2. Generate search queries      │                   │
│  │     - Keyword: "auth refactor"   │                   │
│  │     - Embedding: semantic search │                   │
│  │  3. Query Firebase learnings     │                   │
│  │  4. Evaluate applicability       │                   │
│  │     "Does this apply to current  │                   │
│  │      task? Why or why not?"      │                   │
│  │  5. Return curated learnings     │                   │
│  └──────────────────────────────────┘                   │
│                    │                                     │
│                    ▼                                     │
│  ┌──────────────────────────────────┐                   │
│  │  Injected Context                │                   │
│  │                                  │                   │
│  │  "Before you start, here are     │                   │
│  │   relevant learnings from past   │                   │
│  │   sessions:                      │                   │
│  │                                  │                   │
│  │   1. [HIGH RELEVANCE] When       │                   │
│  │      refactoring auth, always    │                   │
│  │      check JWT expiry logic...   │                   │
│  │                                  │                   │
│  │   2. [MEDIUM RELEVANCE] The      │                   │
│  │      project uses bcrypt 5.x..." │                   │
│  └──────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
```

---

### 3. Shared Plan Document

**Goal**: Orchestrator and subagents see the same live plan. Less back-and-forth.

```
┌─────────────────────────────────────────────────────────┐
│                Firebase: plans/{planId}                  │
│                                                          │
│  {                                                       │
│    "id": "plan_2024_auth_refactor",                     │
│    "goal": "Refactor auth module for JWT refresh",      │
│    "createdAt": "2024-...",                             │
│    "status": "in_progress",                             │
│                                                          │
│    "steps": [                                           │
│      {                                                  │
│        "id": 1,                                         │
│        "description": "Analyze current auth flow",      │
│        "status": "completed",                           │
│        "outcome": "Found 3 security issues",            │
│        "assignedTo": "subagent-1"                       │
│      },                                                 │
│      {                                                  │
│        "id": 2,                                         │
│        "description": "Design new token refresh",       │
│        "status": "in_progress",  ← LIVE UPDATES        │
│        "assignedTo": "subagent-2",                      │
│        "notes": "Considering sliding window approach"   │
│      },                                                 │
│      {                                                  │
│        "id": 3,                                         │
│        "description": "Implement refresh endpoint",     │
│        "status": "pending",                             │
│        "dependencies": [2]                              │
│      }                                                  │
│    ],                                                   │
│                                                          │
│    "context": {                                         │
│      "relevantLearnings": ["learn_123", "learn_456"],   │
│      "projectType": "typescript",                       │
│      "constraints": ["no breaking changes"]             │
│    },                                                   │
│                                                          │
│    "agentPrompt": "You are working on step {current}.   │
│      The goal is: {goal}. Previous steps completed:     │
│      {completedSteps}. Your specific task: {stepDesc}"  │
│  }                                                       │
└─────────────────────────────────────────────────────────┘
```

**How subagents use this**:
1. Subagent spawns with `planId`
2. Reads current plan from Firebase
3. Sees its step, context, and what's done
4. Updates `notes` field as it works (live)
5. Marks step complete when done

**Dashboard sees**: Real-time plan status, which agents are working, progress.

---

### 4. Quick Question Mode

**Goal**: Not everything needs deep planning. Quick questions skip the ceremony.

```
┌─────────────────────────────────────────────────────────┐
│                   Intent Detection                       │
│                                                          │
│  User Input Analysis:                                   │
│                                                          │
│  "Refactor the auth module"                             │
│       └─► COMPLEX TASK → Full planning flow             │
│                                                          │
│  "What's the syntax for TypeScript generics?"           │
│       └─► QUICK QUESTION → Direct answer, no plan       │
│                                                          │
│  "Fix the bug in line 42 of auth.ts"                    │
│       └─► FOCUSED TASK → Minimal plan (1-2 steps)       │
│                                                          │
│  Signals for QUICK QUESTION:                            │
│  - "What is", "How do I", "Syntax for"                  │
│  - No file references or code changes implied           │
│  - Informational, not transformational                  │
│                                                          │
│  Signals for COMPLEX TASK:                              │
│  - "Refactor", "Build", "Implement", "Design"           │
│  - Multiple files likely affected                        │
│  - Requires understanding of system state               │
└─────────────────────────────────────────────────────────┘
```

---

### 5. Self-Learning Loop

**Goal**: Learn from mistakes, don't repeat them. This is the core value.

```
┌─────────────────────────────────────────────────────────┐
│                   Learning Loop                          │
│                                                          │
│  DURING TASK:                                           │
│  ┌────────────────────────────────┐                     │
│  │  1. Task starts                │                     │
│  │  2. Relevant learnings loaded  │                     │
│  │  3. Work proceeds...           │                     │
│  │  4. Error occurs!              │ ← CAPTURE THIS     │
│  │  5. Claude fixes it            │                     │
│  │  6. Task completes             │                     │
│  └────────────────────────────────┘                     │
│                    │                                     │
│  AFTER TASK:       ▼                                     │
│  ┌────────────────────────────────┐                     │
│  │  Self-Audit (Opus subagent)    │                     │
│  │                                │                     │
│  │  Questions:                    │                     │
│  │  - Did we encounter errors?    │                     │
│  │  - What caused them?           │                     │
│  │  - How were they fixed?        │                     │
│  │  - Would past learnings have   │                     │
│  │    prevented this?             │                     │
│  │  - What should we remember?    │                     │
│  └────────────────────────────────┘                     │
│                    │                                     │
│  LEARNING STORED:  ▼                                     │
│  ┌────────────────────────────────┐                     │
│  │  Firebase: learnings/{id}      │                     │
│  │                                │                     │
│  │  {                             │                     │
│  │    "lesson": "When refactoring │                     │
│  │      auth, check if tests mock │                     │
│  │      the token validation",    │                     │
│  │    "context": "auth refactor", │                     │
│  │    "errorType": "test failure",│                     │
│  │    "confidence": 0.9,          │                     │
│  │    "appliesTo": ["auth",       │                     │
│  │      "testing", "refactor"],   │                     │
│  │    "embedding": [0.1, 0.3...]  │                     │
│  │  }                             │                     │
│  └────────────────────────────────┘                     │
│                    │                                     │
│  NIGHTLY:          ▼                                     │
│  ┌────────────────────────────────┐                     │
│  │  Consolidation (Cloud Function)│                     │
│  │                                │                     │
│  │  - Merge similar learnings     │                     │
│  │  - Increase confidence if      │                     │
│  │    pattern repeats             │                     │
│  │  - Prune low-confidence (<0.5) │                     │
│  │  - Extract skill patterns      │                     │
│  └────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────┘
```

---

## Data Models

### Session
```typescript
interface Session {
  id: string;
  userId: string;
  startTime: Timestamp;
  endTime?: Timestamp;
  status: 'active' | 'completed' | 'failed';

  // What happened
  taskDescription: string;
  outcome?: string;
  errorsEncountered: Error[];
  filesModified: string[];

  // Learning
  learningsApplied: string[];  // IDs of learnings used
  learningsCreated: string[];  // IDs of new learnings

  // Meta
  planId?: string;  // If complex task
  mode: 'quick' | 'planned';
}
```

### Learning
```typescript
interface Learning {
  id: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;

  // Content
  lesson: string;           // Human-readable lesson
  context: string;          // When this applies
  errorType?: string;       // What kind of error this prevents

  // Search
  keywords: string[];       // For keyword search
  embedding: number[];      // For semantic search (384 dims)
  appliesTo: string[];      // Categories: ["auth", "testing"]

  // Quality
  confidence: number;       // 0-1, increases with confirmations
  usageCount: number;       // Times this learning was applied
  successRate: number;      // Did applying it help?

  // Source
  sourceSessionId: string;
  sourceError?: string;     // The original error that taught us
}
```

### CommandCache
```typescript
interface CommandCache {
  hash: string;             // sha256(normalized command)
  command: string;          // Original command
  pattern: string;          // Normalized pattern

  safe: boolean;
  reason: string;

  checkedAt: Timestamp;
  checkedBy: 'gemini-flash' | 'manual';

  // Stats
  hitCount: number;
}
```

### Plan
```typescript
interface Plan {
  id: string;
  goal: string;
  status: 'planning' | 'in_progress' | 'completed' | 'failed';

  steps: PlanStep[];
  currentStepId: number;

  context: {
    relevantLearnings: string[];
    projectType: string;
    constraints: string[];
  };

  // For subagent injection
  agentPromptTemplate: string;

  // Real-time
  lastUpdated: Timestamp;
  activeAgents: string[];
}

interface PlanStep {
  id: number;
  description: string;
  status: 'pending' | 'in_progress' | 'completed' | 'failed';
  assignedTo?: string;
  dependencies: number[];
  notes?: string;
  outcome?: string;
}
```

---

## Decisions Made

1. ~~**Embedding Storage**~~: ✅ **SOLVED** - Firebase now has native vector search with `findNearest()`. Using Xenova (384 dims) for local embeddings.

2. **Subagent Model Selection** ✅ **DECIDED**:
   - Learning Retriever: **Opus** (included in Claude Max)
   - Security Check: **Gemini Flash** (fast, cheap, simple task)
   - Self-Audit: **Opus** (included in Claude Max)
   - Step Execution: **Sonnet** (balanced cost/quality)

3. **Plan Document Sync**: Real-time listeners for dashboard, polling (30s) for agents.

4. **Embeddings**: **Xenova/all-MiniLM-L6-v2** - runs locally, 384 dimensions, no API cost.

---

## Hook Architecture (Node.js)

Based on patterns from claude-flow and agentic-flow analysis.

### How Claude Code Hooks Work

Claude Code hooks receive JSON on stdin with tool information:

```json
// PreToolUse receives:
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf node_modules"
  }
}

// PostToolUse receives:
{
  "tool_name": "Bash",
  "tool_input": { "command": "..." },
  "tool_output": "..."  // Result of the command
}
```

### settings.json Hook Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "node .claude/hooks/pre-bash.js"
        }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash|Write|Edit",
        "hooks": [{
          "type": "command",
          "command": "node .claude/hooks/post-tool.js"
        }]
      }
    ],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "node .claude/hooks/session-end.js"
      }]
    }]
  }
}
```

### Directory Structure

```
.claude/
├── settings.json           # Hook configuration
├── hooks/
│   ├── pre-bash.js        # Security check before bash commands
│   ├── post-tool.js       # Log tool usage, detect errors
│   ├── session-end.js     # Self-audit, store learnings
│   └── lib/
│       ├── firebase.js    # Firebase client
│       ├── embeddings.js  # Xenova embedding
│       ├── security.js    # Gemini Flash security check
│       └── learnings.js   # Learning retrieval/storage
└── CLAUDE.md              # Project instructions
```

### Hook Scripts

#### pre-bash.js (Security Check)

```javascript
#!/usr/bin/env node
/**
 * Pre-Bash Hook: Security check before command execution
 *
 * Flow:
 * 1. Parse command from stdin
 * 2. Check Firebase cache for previous approval
 * 3. If miss, call Gemini Flash for assessment
 * 4. Cache result
 * 5. Exit 0 (allow) or exit 1 (block)
 */

import { createHash } from 'crypto';
import { getFirestore } from './lib/firebase.js';
import { checkCommandSafety } from './lib/security.js';

async function main() {
  // Read stdin
  const chunks = [];
  for await (const chunk of process.stdin) chunks.push(chunk);
  const input = JSON.parse(Buffer.concat(chunks).toString());

  const command = input.tool_input?.command;
  if (!command) process.exit(0);  // No command, allow

  // Normalize and hash command
  const normalized = normalizeCommand(command);
  const hash = createHash('sha256').update(normalized).digest('hex');

  // Check cache
  const db = getFirestore();
  const cacheRef = db.collection('command_cache').doc(hash);
  const cached = await cacheRef.get();

  if (cached.exists) {
    const data = cached.data();
    await cacheRef.update({ hitCount: data.hitCount + 1 });

    if (data.safe) {
      console.log(`[CACHED] Command approved: ${command.substring(0, 50)}...`);
      process.exit(0);
    } else {
      console.log(`[BLOCKED] ${data.reason}`);
      process.exit(1);
    }
  }

  // Cache miss - call Gemini Flash
  const result = await checkCommandSafety(command, process.cwd());

  // Store in cache
  await cacheRef.set({
    hash,
    command,
    pattern: normalized,
    safe: result.safe,
    reason: result.reason,
    checkedAt: new Date(),
    checkedBy: 'gemini-flash',
    hitCount: 0
  });

  if (result.safe) {
    console.log(`[APPROVED] ${result.reason}`);
    process.exit(0);
  } else {
    console.log(`[BLOCKED] ${result.reason}`);
    process.exit(1);
  }
}

function normalizeCommand(cmd) {
  // Replace specific paths with patterns
  return cmd
    .replace(/\/Users\/[^\/]+/g, '/Users/<user>')
    .replace(/\/home\/[^\/]+/g, '/home/<user>')
    .replace(/[0-9a-f]{8,}/gi, '<hash>')  // Git hashes, etc.
    .trim();
}

main().catch(err => {
  console.error('[ERROR]', err.message);
  process.exit(0);  // On error, allow (fail open)
});
```

#### post-tool.js (Logging & Error Detection)

```javascript
#!/usr/bin/env node
/**
 * Post-Tool Hook: Log tool usage, detect errors
 *
 * Updates the current session document with:
 * - Tools used
 * - Files modified
 * - Errors detected
 */

import { getFirestore, FieldValue } from './lib/firebase.js';

async function main() {
  const chunks = [];
  for await (const chunk of process.stdin) chunks.push(chunk);
  const input = JSON.parse(Buffer.concat(chunks).toString());

  const db = getFirestore();
  const sessionId = process.env.CLAUDE_SESSION_ID || 'unknown';
  const sessionRef = db.collection('sessions').doc(sessionId);

  const toolName = input.tool_name;
  const toolOutput = input.tool_output || '';

  // Detect errors in output
  const errorPatterns = [
    /error:/i,
    /failed/i,
    /exception/i,
    /traceback/i,
    /cannot find/i,
    /permission denied/i
  ];

  const hasError = errorPatterns.some(p => p.test(toolOutput));

  // Update session
  const update = {
    lastActivity: FieldValue.serverTimestamp(),
    [`toolsUsed.${toolName}`]: FieldValue.increment(1)
  };

  if (hasError) {
    update.errors = FieldValue.arrayUnion({
      tool: toolName,
      output: toolOutput.substring(0, 500),
      timestamp: new Date().toISOString()
    });
  }

  // Track file modifications
  if (toolName === 'Write' || toolName === 'Edit') {
    const filePath = input.tool_input?.file_path || input.tool_input?.path;
    if (filePath) {
      update.filesModified = FieldValue.arrayUnion(filePath);
    }
  }

  await sessionRef.set(update, { merge: true });
}

main().catch(console.error);
```

#### session-end.js (Self-Audit)

```javascript
#!/usr/bin/env node
/**
 * Session End Hook: Self-audit and learning extraction
 *
 * Spawns Opus subagent to:
 * 1. Review what happened in the session
 * 2. Identify learnings from errors
 * 3. Store learnings with embeddings
 */

import { getFirestore, FieldValue } from './lib/firebase.js';
import { getEmbedding } from './lib/embeddings.js';

async function main() {
  const db = getFirestore();
  const sessionId = process.env.CLAUDE_SESSION_ID || 'unknown';

  // Mark session as completed
  const sessionRef = db.collection('sessions').doc(sessionId);
  const session = await sessionRef.get();

  if (!session.exists) {
    console.log('[SESSION-END] No session to audit');
    return;
  }

  const data = session.data();

  // Update session status
  await sessionRef.update({
    status: 'completed',
    endTime: FieldValue.serverTimestamp()
  });

  // If there were errors, extract learnings
  if (data.errors?.length > 0) {
    console.log(`[SESSION-END] Found ${data.errors.length} errors, extracting learnings...`);

    // This would trigger an Opus subagent in the real implementation
    // For now, we'll create a learning request document
    await db.collection('learning_requests').add({
      sessionId,
      errors: data.errors,
      filesModified: data.filesModified || [],
      taskDescription: data.taskDescription || 'Unknown',
      status: 'pending',
      createdAt: FieldValue.serverTimestamp()
    });

    console.log('[SESSION-END] Learning request created for Opus review');
  }

  // Update dashboard with session summary
  await db.collection('dashboard').doc('current').set({
    lastSession: {
      id: sessionId,
      status: 'completed',
      errorsCount: data.errors?.length || 0,
      filesModified: data.filesModified?.length || 0,
      endTime: new Date().toISOString()
    }
  }, { merge: true });

  console.log('[SESSION-END] Audit complete');
}

main().catch(console.error);
```

### Patterns Learned from claude-flow/agentic-flow

1. **JSON parsing from stdin**: Use `cat | jq` or Node.js stdin reading
2. **Matcher patterns**: `"Bash"`, `"Write|Edit|MultiEdit"` for targeting specific tools
3. **Exit codes**: `exit 0` allows, `exit 1` blocks (for PreToolUse)
4. **Environment variables**: Pass context via env (session ID, project path)
5. **Fail open**: On errors, allow command to prevent blocking work

---

## Dashboard Design (Samsung Fold 4)

### Device Specifications

- **Folded (Cover)**: 904 x 2316 px @ 120Hz (23.1:9 aspect)
- **Unfolded (Main)**: 1812 x 2176 px @ 120Hz (almost square)
- **Key insight**: Design for BOTH modes

### Layout Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    FOLDED (Cover Screen)                         │
│                    Single-column, glanceable                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  🔵 optimized-ai          2 tasks running                  │ │
│  │  └─ Refactoring auth      ████████░░ 80%                   │ │
│  │  └─ Writing tests         ██░░░░░░░░ 20%                   │ │
│  ├────────────────────────────────────────────────────────────┤ │
│  │  🟢 client-project        Idle                             │ │
│  │  └─ Last: Fixed login bug (2h ago)                         │ │
│  ├────────────────────────────────────────────────────────────┤ │
│  │  🟡 data-pipeline         1 task blocked                   │ │
│  │  └─ Waiting: API key needed                                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  [Quick Stats: 5 sessions today | 12 learnings | 98% safe]      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    UNFOLDED (Main Screen)                        │
│                    Two-column, detailed                          │
│                                                                  │
│  ┌──────────────────────┬──────────────────────────────────────┐│
│  │   PROJECTS           │   SELECTED PROJECT DETAIL             ││
│  │                      │                                        ││
│  │  🔵 optimized-ai    │   ## optimized-ai                      ││
│  │     2 running        │                                        ││
│  │                      │   Current Tasks:                       ││
│  │  🟢 client-project  │   ┌──────────────────────────────────┐ ││
│  │     idle             │   │ 1. Refactoring auth             │ ││
│  │                      │   │    Step 3/5: Update middleware  │ ││
│  │  🟡 data-pipeline   │   │    Agent: claude-opus            │ ││
│  │     blocked          │   │    ████████░░ 80%                │ ││
│  │                      │   └──────────────────────────────────┘ ││
│  │                      │   ┌──────────────────────────────────┐ ││
│  │                      │   │ 2. Writing tests                 │ ││
│  │                      │   │    Step 1/4: Setup test env      │ ││
│  │                      │   │    Agent: claude-sonnet          │ ││
│  │                      │   │    ██░░░░░░░░ 20%                │ ││
│  │                      │   └──────────────────────────────────┘ ││
│  │                      │                                        ││
│  │                      │   Recent Learnings:                    ││
│  │                      │   • JWT refresh needs sliding window  ││
│  │                      │   • Test mocks must match prod types  ││
│  └──────────────────────┴──────────────────────────────────────┘│
│                                                                  │
│  [Today: 5 sessions | 12 learnings | 47 commands | 2 errors]    │
└─────────────────────────────────────────────────────────────────┘
```

### Firebase Real-time Structure for Dashboard

```typescript
// /dashboard/{userId}
interface DashboardState {
  projects: {
    [projectId: string]: {
      name: string;
      status: 'active' | 'idle' | 'blocked';
      currentTasks: {
        id: string;
        description: string;
        progress: number;  // 0-100
        currentStep: number;
        totalSteps: number;
        agent: string;
      }[];
      lastActivity: Timestamp;
      errorCount: number;
    }
  };
  stats: {
    todaySessions: number;
    todayLearnings: number;
    todayCommands: number;
    todayErrors: number;
    safetyRate: number;  // % of commands approved
  };
}
```

### Real-time Listener (React/Next.js)

```typescript
import { onSnapshot, doc } from 'firebase/firestore';
import { useEffect, useState } from 'react';

function useDashboard(userId: string) {
  const [dashboard, setDashboard] = useState<DashboardState | null>(null);

  useEffect(() => {
    const unsubscribe = onSnapshot(
      doc(db, 'dashboard', userId),
      (doc) => setDashboard(doc.data() as DashboardState)
    );
    return unsubscribe;
  }, [userId]);

  return dashboard;
}
```

### Tech Stack for Dashboard

- **Framework**: Next.js 14 (App Router)
- **Hosting**: Vercel (free tier)
- **Styling**: Tailwind CSS (responsive utilities)
- **State**: Firebase onSnapshot (real-time)
- **PWA**: Add to home screen on Fold 4

---

## Next Steps

1. ✅ Database: Firebase with native vector search
2. ✅ Embeddings: Xenova local (384 dims)
3. ✅ Models: Opus (Claude Max), Gemini Flash (security)
4. 🔲 Create hooks/ directory with Node.js scripts
5. 🔲 Set up Firebase project and indexes
6. 🔲 Create dashboard Next.js app
7. 🔲 Define security check prompts
8. 🔲 Design learning retrieval algorithm (MMR)

What should we tackle next?
