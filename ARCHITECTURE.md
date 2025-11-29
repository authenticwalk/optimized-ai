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
- ⚠️ No native vector search (need extension or Vertex AI)
- ⚠️ Query limitations (no full-text search, limited joins)
- ⚠️ Costs scale with reads (embedding lookups = many reads)
- ⚠️ Vendor lock-in to Google

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

## Open Questions

1. **Embedding Storage**: Firebase doesn't have native vector search. Options:
   - Store embeddings as arrays, search in Cloud Function (works for <10k learnings)
   - Use Vertex AI Vector Search (Google, adds cost)
   - Use Pinecone/Qdrant separately (more complexity)

   **Recommendation**: Start with array storage + Cloud Function. Migrate when scale demands.

2. **Subagent Model Selection**:
   - Learning Retriever: Opus (needs deep thinking about applicability)
   - Security Check: Gemini Flash (fast, cheap, simple task)
   - Self-Audit: Opus (needs reflection capability)
   - Step Execution: Sonnet (balanced cost/quality)

   **Question**: What's your budget tolerance? Opus is expensive.

3. **Plan Document Sync**:
   - How often should subagents poll the plan document?
   - Should we use Firebase real-time listeners or polling?

   **Recommendation**: Real-time listeners for dashboard, polling (30s) for agents.

4. **Error Capture**:
   - How do we detect when Claude encounters an error during work?
   - Options: PostToolUse hook checks output, explicit error markers, AI detection

   **Question**: Do you have access to tool outputs in hooks?

---

## Next Steps

1. Finalize database choice (Firebase confirmed?)
2. Design the hook architecture in detail
3. Define the exact prompts for each subagent role
4. Design the dashboard UI/data flows
5. Create the learning retrieval algorithm
6. Define the security check prompt and caching strategy

What aspect would you like to dig into first?
