# Bible Graph Architecture Proposal

## Your Goals (Front and Center)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PRIMARY GOAL                                          │
│   Build a graph database of the Bible to assist translation into            │
│   languages with little/no LLM representation                               │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                        SPECIFIC REQUIREMENTS                                 │
│                                                                              │
│  DATA RELATIONSHIPS:                                                         │
│  ├── Verse → Strong's words                                                 │
│  ├── Strong's word → translations in N languages                            │
│  ├── Verse → concepts/topics (kinsman redeemer, forgiveness, etc.)          │
│  ├── Verse → grammar constructions                                          │
│  ├── Grammar construction → words involved                                  │
│  └── Word → language features (plural, number, mood, tense, gender)         │
│                                                                              │
│  TRANSLATION PREDICTIONS:                                                    │
│  ├── Given 1000 languages, predict words for language 1001                  │
│  ├── Discover relationships between new language and existing ones          │
│  ├── Learn grammar patterns from similar languages                          │
│  ├── word → []potential_words cipher with AI sorting                        │
│  └── Store exceptions to improve predictions over time                      │
│                                                                              │
│  UNIQUE CHALLENGES:                                                          │
│  ├── Target languages NOT in any LLM training data                          │
│  ├── Bible is often FIRST literature in these languages                     │
│  ├── Embeddings may fail on unknown words                                   │
│  └── Scunthorpe-like problems with look-alike words                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BIBLE GRAPH SYSTEM                                   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                     LAYER 4: TRANSLATION API                            ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────────┐  ││
│  │  │ Predict Word │ │ Add Word     │ │ Get Patterns │ │ Query Verses   │  ││
│  │  │ in New Lang  │ │ + Learn      │ │ + Exceptions │ │ by Concept     │  ││
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └────────────────┘  ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                     LAYER 3: LEARNING ENGINE                            ││
│  │  ┌──────────────────────┐  ┌──────────────────────────────────────────┐ ││
│  │  │ SONA (Pattern Learn) │  │ LLM Integration (Context-Enhanced)       │ ││
│  │  │ - MicroLoRA          │  │ - Exception-aware prompts                │ ││
│  │  │ - ReasoningBank      │  │ - Grammar pattern injection              │ ││
│  │  │ - EWC++              │  │ - Multi-language examples                │ ││
│  │  └──────────────────────┘  └──────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                     LAYER 2: QUERY ENGINE                               ││
│  │  ┌──────────────────────┐  ┌──────────────────────────────────────────┐ ││
│  │  │ Hybrid Query         │  │ Embedding Service                        │ ││
│  │  │ (Graph + Vector)     │  │ - Relational embeddings                  │ ││
│  │  │ - Cypher patterns    │  │ - Definition-based (for unknowns)        │ ││
│  │  │ - Semantic search    │  │ - Context aggregation                    │ ││
│  │  └──────────────────────┘  └──────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                     LAYER 1: DATA STORAGE                               ││
│  │  ┌──────────────────────┐  ┌──────────────────────────────────────────┐ ││
│  │  │ Graph Database       │  │ Vector Database                          │ ││
│  │  │ - Nodes/Edges        │  │ - HNSW Index                             │ ││
│  │  │ - Hyperedges         │  │ - Semantic vectors                       │ ││
│  │  │ - Properties/Labels  │  │ - Definition vectors                     │ ││
│  │  └──────────────────────┘  └──────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                     LAYER 0: DATA IMPORT                                ││
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────────────────┐││
│  │  │ Strong's   │ │ USFM/USX   │ │ Alignments │ │ Morphology Parsers    │││
│  │  │ Concordance│ │ Bibles     │ │ (1000 lang)│ │ (Hebrew/Greek)        │││
│  │  └────────────┘ └────────────┘ └────────────┘ └────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Layer 1: Data Storage

### Decision 1.1: Graph Database

#### Option A: RuVector GraphDB (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Use RuVector GraphDB                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ REUSE:                                                          │
│   ✓ ruvector-graph crate (full)                                │
│   ✓ GraphDB struct with DashMap storage                        │
│   ✓ Hyperedge + HyperedgeWithRoles                             │
│   ✓ LabelIndex, PropertyIndex, AdjacencyIndex                  │
│   ✓ Transaction support (IsolationLevel)                       │
│                                                                  │
│ BUILD:                                                          │
│   • Custom indexes for Strong's lookup                         │
│   • Language family index                                       │
│   • Grammar pattern index                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| Hyperedges native (perfect for verse alignments) | Less mature than Neo4j |
| Integrated with vector search | Limited Cypher subset |
| Rust-native, no network overhead | No visual browser |
| SONA integration built-in | Smaller community |
| Can run in WASM for edge | Documentation sparse |

**Alternatives Considered:**

| Alternative | Why Not |
|-------------|---------|
| **Neo4j** | No native hyperedges (would need workaround nodes), separate service, no SONA integration |
| **SurrealDB** | No hyperedges, newer/less proven, would need separate vector DB |
| **DGraph** | GraphQL-focused, no native vector search, heavy setup |
| **Custom SQLite** | No graph traversal, would rebuild everything |

#### Option B: Hybrid with Neo4j (NOT RECOMMENDED)

Would require: Separate Neo4j service + RuVector for vectors + custom SONA integration
Complexity: 3x components to maintain

---

### Decision 1.2: Vector Database

#### Option A: RuVector VectorDB (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Use RuVector VectorDB                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ REUSE:                                                          │
│   ✓ ruvector-core crate (full)                                 │
│   ✓ HnswIndex with hnsw_rs backend                             │
│   ✓ SIMD distance via simsimd                                  │
│   ✓ redb persistence                                           │
│   ✓ Batch operations with rayon                                │
│                                                                  │
│ BUILD:                                                          │
│   • Multiple vector spaces (verses, words, concepts)           │
│   • Definition-based embedding fallback                        │
│   • Language-clustered indexes                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| Same codebase as graph | HNSW delete is soft (graph remains) |
| Hybrid queries native | No product quantization yet |
| SIMD optimized | Memory usage with many vectors |
| Persists to redb | Single-node only |

**Alternatives Considered:**

| Alternative | Why Not |
|-------------|---------|
| **Qdrant** | Separate service, no graph integration, would lose SONA |
| **Pinecone** | Cloud-only, expensive at scale, no graph |
| **Milvus** | Heavy infrastructure, overkill for this |
| **pgvector** | Slower than HNSW, no graph features |

---

### Decision 1.3: Storage Backend

#### Option A: redb + Memory-Mapped Files (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Use redb with memmap2                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ REUSE:                                                          │
│   ✓ redb (already in ruvector)                                 │
│   ✓ memmap2 (already in ruvector)                              │
│   ✓ VectorStorage struct                                       │
│   ✓ GraphStorage struct                                        │
│                                                                  │
│ BUILD:                                                          │
│   • Sharded storage for 1000+ language data                    │
│   • Incremental backup strategy                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| Embedded, no separate DB | Single-writer limitation |
| ACID transactions | Not distributed |
| Fast memory-mapped reads | redb is newer (less battle-tested) |
| Already integrated | |

**Alternatives:**

| Alternative | Why Not |
|-------------|---------|
| **RocksDB** | More complex, would need new integration |
| **SQLite** | Slower for this workload, less flexible |
| **LMDB** | Similar to redb but would need integration |

---

## Layer 2: Query Engine

### Decision 2.1: Hybrid Query System

#### Option A: RuVector Hybrid Queries (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Use RuVector Hybrid Query System                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ REUSE:                                                          │
│   ✓ ruvector-graph/hybrid module                               │
│   ✓ HybridIndex, HybridQuery, HybridResult                     │
│   ✓ SemanticSearch struct                                      │
│   ✓ VectorCypherParser (limited)                               │
│   ✓ RAG integration (RagEngine)                                │
│                                                                  │
│ BUILD:                                                          │
│   • Extended Cypher parser for Bible-specific patterns         │
│   • Multi-hop translation path queries                         │
│   • Language similarity scoring                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Query Examples Your System Will Support:**

```cypher
// 1. "What words in language X are used for Strong's H7225?"
MATCH (s:Strongs {id: "H7225"})-[:TRANSLATED_AS]->(w:Word {lang: "X"})
RETURN w.form, w.verse_contexts, w.frequency

// 2. "Find verses about 'forgiveness' similar to John 3:16"
MATCH (v:Verse)-[:HAS_CONCEPT]->(c:Concept {name: "forgiveness"})
WHERE vector_similar(v.embedding, verse_embedding("john:3:16"), 0.8)
RETURN v

// 3. "What grammar pattern does language X use for construct chains?"
MATCH (v:Verse)-[:HAS_GRAMMAR]->(g:Grammar {type: "construct_chain"})
WHERE v.lang = "X"
RETURN g.pattern, count(*) as frequency
ORDER BY frequency DESC

// 4. "Trace translation path from Hebrew to language X via similar languages"
MATCH path = (src:Word {lang: "hebrew"})-[:SIMILAR_IN*1..3]->(tgt:Word {lang: "X"})
WHERE src.strongs = "H7225"
RETURN path, [r in relationships(path) | r.confidence]
```

| Pros | Cons |
|------|------|
| Graph + Vector in one query | Cypher subset only |
| Semantic paths built-in | May need custom extensions |
| RAG ready | Learning curve |

---

### Decision 2.2: Embedding Strategy

#### THE CRITICAL CHALLENGE

```
┌─────────────────────────────────────────────────────────────────┐
│ PROBLEM: Standard embeddings FAIL for unknown languages        │
│                                                                  │
│ • Word "xyz123" in language L has no embedding                 │
│ • LLM-based embedders produce garbage for unknown tokens       │
│ • Sentence embeddings may randomly match due to subwords       │
│ • Scunthorpe problem: "shitake" matches inappropriate content  │
└─────────────────────────────────────────────────────────────────┘
```

#### Option A: Multi-Strategy Embedding (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Use Relational Embeddings with Fallback Chain        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ STRATEGY:                                                       │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ For KNOWN languages (in embedding model):               │   │
│   │                                                          │   │
│   │   word/verse → multilingual-e5-large → embedding        │   │
│   │                                                          │   │
│   │ REUSE: HuggingFace sentence-transformers               │   │
│   │   Model: intfloat/multilingual-e5-large (100+ langs)   │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ For UNKNOWN languages (not in any model):               │   │
│   │                                                          │   │
│   │   word_embedding = COMBINE([                            │   │
│   │     embed(strongs_definition),      // "beginning"      │   │
│   │     embed(concept_description),     // "creation start" │   │
│   │     grammar_feature_vector,         // [noun, sing, ...]│   │
│   │     avg(similar_lang_translations)  // neighbor average │   │
│   │   ])                                                    │   │
│   │                                                          │   │
│   │ BUILD: RelationalEmbedder service                      │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ For VERSES (any language):                              │   │
│   │                                                          │   │
│   │   verse_embedding = COMBINE([                           │   │
│   │     avg(word_embeddings),          // word-level        │   │
│   │     concept_centroid,              // topic cluster     │   │
│   │     grammar_signature,             // structure vector  │   │
│   │     aligned_verse_embedding        // from known lang   │   │
│   │   ])                                                    │   │
│   │                                                          │   │
│   │ BUILD: VerseEmbedder service                           │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| Works for unknown languages | More complex pipeline |
| Leverages Strong's as anchor | Relational embeds less accurate |
| Graceful degradation | Need to tune combination weights |
| Scunthorpe-resistant (definition-based) | Multiple embedding calls |

**Embedding Model Decision:**

| Model | Langs | Dim | Why/Why Not |
|-------|-------|-----|-------------|
| **multilingual-e5-large** ✓ | 100+ | 1024 | Best multilingual coverage, open |
| LaBSE | 109 | 768 | Good but slightly worse quality |
| mBERT | 104 | 768 | Older, less effective |
| OpenAI ada-002 | ~100 | 1536 | Costly, API dependency |
| Cohere multilingual | 100+ | 1024 | API dependency, cost |

**REUSE:** `sentence-transformers` Python library or `candle` Rust bindings
**BUILD:** RelationalEmbedder wrapper that falls back through strategies

---

## Layer 3: Learning Engine

### Decision 3.1: Pattern Learning

#### Option A: SONA from RuVector (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Use SONA for Translation Pattern Learning            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ REUSE:                                                          │
│   ✓ sona crate (full)                                          │
│   ✓ SonaEngine                                                 │
│   ✓ MicroLoRA (rank 1-2, per-translation)                      │
│   ✓ BaseLoRA (rank 4-16, background consolidation)             │
│   ✓ ReasoningBank (K-means++ pattern clustering)               │
│   ✓ EWC++ (prevent forgetting good patterns)                   │
│   ✓ TrajectoryBuilder                                          │
│                                                                  │
│ BUILD:                                                          │
│   • TranslationTrajectory (extends TrajectoryBuilder)          │
│   • LanguageFamilyPatterns (group similar languages)           │
│   • ExceptionTracker (stores prediction failures)              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**How SONA Fits Your Translation Workflow:**

```
┌─────────────────────────────────────────────────────────────────┐
│              TRANSLATION LEARNING FLOW                          │
│                                                                  │
│  USER ACTION                      SONA RESPONSE                 │
│  ───────────                      ─────────────                 │
│                                                                  │
│  1. Predict word for H7225        → find_patterns(query_embed)  │
│     in language L                   Returns: similar successful │
│                                     translations from bank      │
│                                                                  │
│  2. User provides correct         → begin_trajectory()          │
│     translation "palavra_X"         add_step(word_embed, 0.9)   │
│                                     end_trajectory(quality=0.9) │
│                                                                  │
│  3. Pattern accumulates           → MicroLoRA updates instantly │
│     (100+ similar translations)     BaseLoRA consolidates hourly│
│                                     ReasoningBank clusters      │
│                                                                  │
│  4. Next prediction for H7226     → Patterns now include        │
│     in language L                   learned L characteristics   │
│                                     Better prediction!          │
│                                                                  │
│  5. Prediction wrong              → Store as Exception          │
│                                     Feed to LLM as context      │
│                                     EWC++ preserves good patterns│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| Learns without retraining LLM | Needs enough data to cluster |
| Per-request adaptation | SONA is newer, less proven |
| Prevents forgetting | Configuration tuning needed |
| Exception-aware | |

**Alternatives:**

| Alternative | Why Not |
|-------------|---------|
| **Fine-tune LLM** | Expensive, catastrophic forgetting, can't do per-request |
| **Traditional ML** | Would need to build clustering, no LoRA benefits |
| **RAG only** | No learning, just retrieval |
| **Custom implementation** | SONA already does this well |

---

### Decision 3.2: LLM Integration

#### Option A: Context-Enhanced LLM Calls (RECOMMENDED)

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: LLM as Final Ranker with Graph-Enhanced Context      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ WHY NOT LLM-FIRST:                                              │
│   • Target language not in training data                       │
│   • LLM will hallucinate words                                 │
│   • No way to debug/improve LLM's internal model              │
│   • Expensive for every prediction                             │
│                                                                  │
│ OUR APPROACH:                                                   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 1. Graph generates candidates:                          │   │
│   │    word → []potential_words (your cipher idea)          │   │
│   │                                                          │   │
│   │ 2. SONA patterns rank candidates:                       │   │
│   │    Apply learned patterns from similar translations     │   │
│   │                                                          │   │
│   │ 3. LLM makes final selection WITH context:              │   │
│   │    - Top 5 candidates from graph                        │   │
│   │    - Grammar patterns from similar languages            │   │
│   │    - Known exceptions for this word/verse              │   │
│   │    - Example translations from related languages        │   │
│   │                                                          │   │
│   │ 4. Exception stored if user corrects:                   │   │
│   │    Feeds back into SONA + future LLM context           │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│ REUSE:                                                          │
│   ✓ RagEngine from ruvector-graph/hybrid                       │
│   ✓ Claude API / OpenAI API / Local LLM                        │
│                                                                  │
│ BUILD:                                                          │
│   • TranslationPromptBuilder                                   │
│   • ExceptionContextInjector                                   │
│   • CandidateRanker                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Prompt Structure:**

```
┌─────────────────────────────────────────────────────────────────┐
│ TRANSLATION PROMPT TEMPLATE                                     │
│                                                                  │
│ You are translating a Bible verse into {target_language}.       │
│                                                                  │
│ VERSE: {verse_reference}                                        │
│ HEBREW/GREEK: {source_text}                                     │
│                                                                  │
│ STRONG'S WORD: {strongs_id} - {definition}                      │
│                                                                  │
│ CANDIDATE TRANSLATIONS (from graph):                            │
│ 1. {word_1} (confidence: 0.85, used in {N} similar contexts)   │
│ 2. {word_2} (confidence: 0.72, used in {M} similar contexts)   │
│ ...                                                             │
│                                                                  │
│ LANGUAGE {target_language} CHARACTERISTICS:                     │
│ - Similar to: {similar_languages}                               │
│ - Word order: {observed_pattern}                                │
│ - Grammar: {grammar_patterns}                                   │
│                                                                  │
│ KNOWN EXCEPTIONS (where patterns didn't work):                  │
│ - In verse X, predicted Y but correct was Z because {reason}   │
│                                                                  │
│ EXAMPLES FROM RELATED LANGUAGES:                                │
│ - {lang_1}: {translation_1}                                     │
│ - {lang_2}: {translation_2}                                     │
│                                                                  │
│ Select the best candidate or suggest alternative. Explain why.  │
└─────────────────────────────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| LLM debuggable (we see context) | Still need some LLM calls |
| Exceptions improve over time | Prompt engineering needed |
| Works with unknown languages | Context can get long |
| Candidates bound predictions | |

**LLM Selection:**

| LLM | Why/Why Not |
|-----|-------------|
| **Claude (Anthropic)** ✓ | Best reasoning, good with structured context |
| GPT-4 | Good but expensive, less controllable |
| Llama 3 (local) | Free, but less capable at ranking |
| Mistral | Good balance, can run locally |

**Recommendation:** Claude API for quality, with Llama 3 fallback for cost control.

---

### Decision 3.3: Exception Storage

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Graph-Native Exception Storage                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ SCHEMA:                                                         │
│                                                                  │
│   (:Word)-[:HAS_EXCEPTION]->(:Exception {                       │
│     verse_context: "gen:1:1",                                   │
│     predicted: "word_A",                                        │
│     actual: "word_B",                                           │
│     reason: "user provided",                                    │
│     grammar_context: "construct_chain",                         │
│     similar_exceptions: ["exception_id_1", ...],                │
│     created_at: timestamp                                       │
│   })                                                            │
│                                                                  │
│   // Also link exception to relevant concepts                   │
│   (:Exception)-[:RELATED_TO]->(:Concept)                        │
│   (:Exception)-[:IN_GRAMMAR]->(:Grammar)                        │
│                                                                  │
│ REUSE:                                                          │
│   ✓ GraphDB node/edge storage                                  │
│   ✓ PropertyIndex for exception lookup                         │
│                                                                  │
│ BUILD:                                                          │
│   • ExceptionManager service                                   │
│   • Exception clustering (similar exceptions)                  │
│   • Auto-exception detection (repeated corrections)            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Layer 4: Translation API

### Decision 4.1: API Design

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Rust API with Optional HTTP/WASM Interfaces          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ CORE API (Rust):                                                │
│                                                                  │
│   pub struct BibleGraph {                                       │
│       graph: GraphDB,                                           │
│       vectors: VectorDB,                                        │
│       sona: SonaEngine,                                         │
│       embedder: RelationalEmbedder,                             │
│       llm: LlmClient,                                           │
│   }                                                             │
│                                                                  │
│   impl BibleGraph {                                             │
│       // Core translation functions                             │
│       fn predict_word(&self, strongs: &str, lang: &str)        │
│           -> Vec<WordCandidate>;                                │
│                                                                  │
│       fn add_translation(&self, word: WordTranslation)         │
│           -> Result<()>;  // Learns pattern                    │
│                                                                  │
│       fn get_exceptions(&self, strongs: &str, lang: &str)      │
│           -> Vec<Exception>;                                    │
│                                                                  │
│       // Query functions                                        │
│       fn verses_by_concept(&self, concept: &str)               │
│           -> Vec<Verse>;                                        │
│                                                                  │
│       fn similar_verses(&self, verse_id: &str, k: usize)       │
│           -> Vec<(Verse, f32)>;                                │
│                                                                  │
│       fn word_translations(&self, strongs: &str)               │
│           -> HashMap<Language, Vec<WordForm>>;                  │
│                                                                  │
│       fn grammar_patterns(&self, lang: &str)                   │
│           -> Vec<GrammarPattern>;                               │
│                                                                  │
│       fn language_similarity(&self, lang: &str)                │
│           -> Vec<(Language, f32)>;                             │
│   }                                                             │
│                                                                  │
│ REUSE:                                                          │
│   ✓ All ruvector crates as dependencies                        │
│   ✓ axum or actix-web for HTTP (optional)                      │
│   ✓ wasm-bindgen for WASM (optional)                           │
│                                                                  │
│ BUILD:                                                          │
│   • BibleGraph facade struct                                   │
│   • API endpoint handlers                                      │
│   • TypeScript types (for WASM)                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Layer 0: Data Import

### Decision 5.1: Import Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│ DECISION: Modular Import Pipeline                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ IMPORTERS NEEDED:                                               │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 1. STRONG'S CONCORDANCE                                 │   │
│   │    Source: OpenScriptures Hebrew/Greek lexicons        │   │
│   │    Format: JSON/XML                                     │   │
│   │    Creates: :Word nodes with morphology properties     │   │
│   │                                                          │   │
│   │    REUSE: serde_json, quick-xml                        │   │
│   │    BUILD: StrongsImporter                              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 2. BIBLE TEXTS (1000+ languages)                        │   │
│   │    Source: eBible, Paratext exports, USFM files        │   │
│   │    Format: USFM, USX (XML), plain text                 │   │
│   │    Creates: :Verse nodes, :Word nodes per language     │   │
│   │                                                          │   │
│   │    REUSE: usfm-parser crate (or build minimal)         │   │
│   │    BUILD: BibleTextImporter                            │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 3. WORD ALIGNMENTS                                      │   │
│   │    Source: unfoldingWord alignment data, Paratext      │   │
│   │    Format: TSV, JSON                                    │   │
│   │    Creates: :TRANSLATED_AS edges, hyperedges           │   │
│   │                                                          │   │
│   │    REUSE: csv crate                                    │   │
│   │    BUILD: AlignmentImporter                            │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 4. MORPHOLOGY (Hebrew/Greek)                            │   │
│   │    Source: OpenScriptures morphology, OSHB, OGNT       │   │
│   │    Format: TSV, XML                                     │   │
│   │    Creates: Morphology properties on :Word nodes       │   │
│   │                                                          │   │
│   │    REUSE: csv, quick-xml                               │   │
│   │    BUILD: MorphologyImporter                           │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 5. CONCEPTS/TOPICS                                      │   │
│   │    Source: Treasury of Scripture Knowledge, manual     │   │
│   │    Format: Custom (you define)                         │   │
│   │    Creates: :Concept nodes, :HAS_CONCEPT edges         │   │
│   │                                                          │   │
│   │    REUSE: serde                                        │   │
│   │    BUILD: ConceptImporter                              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Complete Component Reuse Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPONENT REUSE SUMMARY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ ✅ REUSE AS-IS (from ruvector):                                             │
│ ───────────────────────────────                                             │
│ • ruvector-core          VectorDB, HNSW, SIMD distance, storage            │
│ • ruvector-graph         GraphDB, hyperedges, indexes, Cypher              │
│ • sona                   SonaEngine, LoRA, EWC++, ReasoningBank            │
│ • ruvector-graph/hybrid  HybridQuery, SemanticSearch, RagEngine            │
│                                                                              │
│ ✅ REUSE AS-IS (third-party):                                               │
│ ────────────────────────────                                                │
│ • hnsw_rs               HNSW algorithm                                      │
│ • simsimd               SIMD distance calculations                          │
│ • redb                  Embedded storage                                    │
│ • dashmap               Concurrent hashmaps                                 │
│ • rayon                 Parallelism                                         │
│ • sentence-transformers Python embedding models (via FFI or service)       │
│ • serde, serde_json     Serialization                                       │
│ • quick-xml, csv        Import parsing                                      │
│                                                                              │
│ 🔧 EXTEND (modify ruvector components):                                     │
│ ──────────────────────────────────────                                      │
│ • Cypher parser         Add Bible-specific query patterns                   │
│ • PropertyIndex         Add Strong's-optimized index                        │
│ • HybridQuery           Add language similarity scoring                     │
│                                                                              │
│ 🏗️ BUILD NEW:                                                               │
│ ────────────                                                                │
│ • BibleGraph            Main facade orchestrating all components            │
│ • RelationalEmbedder    Multi-strategy embedding for unknown languages     │
│ • TranslationPredictor  Combines graph + SONA + LLM                        │
│ • ExceptionManager      Stores and retrieves translation exceptions        │
│ • StrongsImporter       Imports Strong's concordance                        │
│ • BibleTextImporter     Imports USFM/USX Bible texts                       │
│ • AlignmentImporter     Imports word alignments                            │
│ • MorphologyImporter    Imports morphological data                         │
│ • ConceptImporter       Imports concepts/topics                            │
│ • PromptBuilder         Constructs LLM prompts with context                │
│ • HTTP API              REST endpoints (optional)                          │
│ • WASM bindings         Browser interface (optional)                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Risk Assessment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RISK ASSESSMENT                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ HIGH RISK:                                                                   │
│ ──────────                                                                  │
│ ⚠️  Embedding quality for unknown languages                                  │
│     Mitigation: Relational embedding fallback, definition-based approach    │
│                                                                              │
│ ⚠️  SONA pattern quality with sparse data                                   │
│     Mitigation: Start with well-represented languages, build up patterns   │
│                                                                              │
│ ⚠️  Import data quality/alignment accuracy                                  │
│     Mitigation: Validate imports, start with high-quality sources          │
│                                                                              │
│ MEDIUM RISK:                                                                 │
│ ────────────                                                                │
│ ⚡ RuVector maturity (newer project)                                        │
│    Mitigation: Comprehensive testing, fallback to simpler approaches       │
│                                                                              │
│ ⚡ Memory usage with 1000+ languages                                        │
│    Mitigation: Sharded storage, lazy loading, memory profiling            │
│                                                                              │
│ ⚡ LLM API costs at scale                                                   │
│    Mitigation: Local LLM fallback, aggressive caching, batch requests      │
│                                                                              │
│ LOW RISK:                                                                    │
│ ─────────                                                                   │
│ ✓ Graph/vector storage (proven patterns)                                   │
│ ✓ Import pipeline (standard parsing)                                       │
│ ✓ API design (straightforward)                                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Implementation Phases

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      IMPLEMENTATION PHASES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ PHASE 1: Foundation                                                         │
│ ────────────────────                                                        │
│ • Set up ruvector as dependency                                            │
│ • Create BibleGraph facade                                                 │
│ • Import Strong's concordance (Hebrew + Greek)                             │
│ • Import 1 English Bible (baseline)                                        │
│ • Basic queries working                                                    │
│                                                                              │
│ PHASE 2: Multi-Language                                                     │
│ ─────────────────────                                                       │
│ • Import 100 aligned languages                                             │
│ • Build alignment hyperedges                                               │
│ • Implement RelationalEmbedder                                             │
│ • Test vector search across languages                                      │
│                                                                              │
│ PHASE 3: Learning                                                           │
│ ────────────────                                                            │
│ • Integrate SONA for pattern learning                                      │
│ • Implement exception storage                                              │
│ • Build TranslationPredictor                                               │
│ • Test predict → correct → learn cycle                                     │
│                                                                              │
│ PHASE 4: LLM Integration                                                    │
│ ─────────────────────                                                       │
│ • Build PromptBuilder with context injection                               │
│ • Integrate LLM API                                                        │
│ • Exception-aware prompts                                                  │
│ • Full translation workflow                                                │
│                                                                              │
│ PHASE 5: Scale                                                              │
│ ────────────                                                                │
│ • Import remaining 900 languages                                           │
│ • Concepts/topics import                                                   │
│ • Grammar pattern extraction                                               │
│ • Performance optimization                                                 │
│ • API/WASM interfaces                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Decision Summary Table

| Decision | Choice | Key Reason |
|----------|--------|------------|
| Graph DB | RuVector GraphDB | Native hyperedges, integrated |
| Vector DB | RuVector VectorDB | Hybrid queries, SONA integration |
| Storage | redb + memmap2 | Already integrated, embedded |
| Hybrid Queries | RuVector hybrid module | Vector + graph in one |
| Embeddings (known langs) | multilingual-e5-large | 100+ languages, open |
| Embeddings (unknown) | Relational (definition-based) | Only option that works |
| Pattern Learning | SONA | LoRA + EWC++ + clustering |
| LLM Role | Final ranker with context | Debuggable, bounded |
| Exception Storage | Graph-native | Queryable, linkable |
| Primary Language | Rust | Performance, ruvector native |

---

## Next Steps

1. **Create Rust project** with ruvector dependencies
2. **Implement BibleGraph facade** with basic operations
3. **Build Strong's importer** and import lexicons
4. **Test basic graph queries** before adding complexity
5. **Iterate** based on real data characteristics
