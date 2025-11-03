# Web Tools

**Tools Covered**: WebFetch, WebSearch

## Overview

Web tools enable Claude to access external information beyond its training data. Understanding the differences and costs is critical for efficient usage.

## WebFetch Tool

### How It Works

**Purpose**: Retrieve and answer questions from a specific URL

**Technical Implementation**:
1. **Validation & Normalization**: Enforces URL length limits (≤2000 chars), upgrades HTTP to HTTPS, removes credentials
2. **Domain Safety Check**: Validates domain against deny-list via `domain_info` API
3. **Content Fetching**: Follows same-host redirects automatically, caches for 15 minutes, caps at ~10 MB
4. **HTML Processing**: Converts HTML to Markdown using Turndown library, truncates to 100 KB
5. **LLM Summarization**: Claude 3.5 Haiku processes content with strict instructions

**Why Haiku?**
- Cost control: Haiku is ~10x cheaper than Sonnet
- Pre-filters content before reaching main model
- Reduces token consumption dramatically

### Parameters

```typescript
{
  url: string;            // REQUIRED: URL to fetch (≤2000 chars)
  prompt: string;         // REQUIRED: Question to answer about content
}
```

### Design Rationale

**Required Prompt (not optional)**:

The tool does NOT return raw HTML/Markdown. You MUST provide a prompt. This design serves three purposes:

1. **Cost Control**: Haiku pre-processes content instead of sending ~100KB to expensive Sonnet
2. **Injection Resistance**: System prompt constrains Haiku to answer only from provided content
3. **Copyright Protection**: Enforces strict quote limits (125 chars max), prohibits exact lyrics

### Usage Examples

```typescript
// Basic information extraction
WebFetch({
  url: "https://docs.example.com/api/authentication",
  prompt: "What authentication methods are supported?"
})

// Technical documentation
WebFetch({
  url: "https://github.com/example/repo",
  prompt: "What is this repository about and what are the main features?"
})

// API documentation
WebFetch({
  url: "https://api.service.com/docs",
  prompt: "How do I authenticate API requests? Provide example code."
})
```

### Output Format

**Concise answers**:
- Brief, paraphrased summary
- Direct quotes limited to 125 characters maximum
- Focuses on answering the specific prompt
- May omit details not relevant to prompt

**Example**:
```typescript
WebFetch({
  url: "https://docs.anthropic.com/claude/reference",
  prompt: "What are the rate limits for Claude API?"
})

// Response (paraphrased):
"Claude API rate limits vary by tier. Free tier allows 1000 requests/day.
Pro tier supports 10,000 requests/day with 60 requests/minute. Enterprise
has custom limits."
```

### Caching Behavior

**15-minute TTL**:
- Same URL + prompt cached for 15 minutes
- Subsequent requests return cached result instantly
- Reduces costs for repeated access
- Cache is per URL (different prompts still hit cache)

**Implications**:
- ✅ Free to retry same URL within 15 minutes
- ✅ Good for iterative research on same page
- ❌ Won't see updates within cache window

### Redirect Handling

**Same-host redirects**: Followed automatically

**Cross-host redirects**:
- Tool returns redirect information
- You must make new WebFetch request with redirect URL

```typescript
// Original request
WebFetch({
  url: "https://short.url/abc",
  prompt: "What is this about?"
})

// Response if cross-host redirect:
"This URL redirects to https://different-domain.com/full-url"

// Make new request:
WebFetch({
  url: "https://different-domain.com/full-url",
  prompt: "What is this about?"
})
```

### Content Limits

**Fetching**:
- Maximum fetch size: ~10 MB
- Content truncated if larger

**Processing**:
- HTML → Markdown conversion
- Truncated to 100 KB text
- Sent to Haiku for summarization

**Output**:
- Brief answer (typically 100-500 tokens)
- Not full page content

### Best Practices for Token Efficiency

**DO**:
- ✅ Use specific, focused prompts
- ✅ Ask for exactly what you need
- ✅ Leverage 15-minute cache for follow-up questions
- ✅ Use WebSearch first if you don't have the URL

**DON'T**:
- ❌ Use vague prompts ("tell me about this page")
- ❌ Expect full page content (use for Q&A only)
- ❌ Fetch when information is in codebase
- ❌ Use for downloading files (not designed for that)

### Token Cost Analysis

**Approximate costs**:

```typescript
// Single WebFetch call
WebFetch({ url: "...", prompt: "..." })
// Input (to main model): ~100 tokens (tool call)
// Internal (to Haiku): ~20,000 tokens (100KB content) - cheap
// Output (from Haiku): ~300 tokens (answer)
// Total to main model: ~400 tokens

// Cost breakdown:
// Main model (Sonnet): ~400 tokens (~$0.001)
// Haiku processing: ~20,000 tokens (~$0.005)
// Total: ~$0.006 per fetch
```

**Comparison**:
- WebFetch: ~$0.006 + concise answer
- Pasting 100KB docs into chat: ~20,000 Sonnet tokens (~$0.06)
- **Savings**: 90%

### Security Considerations

**Domain Safety**:
- Deny-listed domains blocked automatically
- Removes credentials from URLs
- HTTPS enforced (HTTP upgraded)

**Content Safety**:
- Haiku constrained by system prompt
- Only answers from provided content
- Cannot execute JavaScript or active content

**Best practices**:
- Verify URLs before fetching
- Don't fetch untrusted domains
- Be cautious with user-provided URLs

### Known Limitations

**Cannot do**:
- Execute JavaScript (gets static HTML only)
- Interact with pages (no clicking, form submission)
- Access authenticated content (no cookies/sessions)
- Download large binary files
- Process real-time/streaming content

**Workarounds**:
- For authenticated content: Use API directly via Bash/curl
- For dynamic content: Use API endpoints instead of web pages
- For large files: Use Bash to wget/curl

## WebSearch Tool

### How It Works

**Purpose**: Discover relevant URLs for a search query

**Technical Implementation**:
- Sends query to Anthropic's server-side WebSearch API
- Same API used by Claude chat interface
- Returns only titles and URLs
- No content retrieval (use WebFetch for that)

### Parameters

```typescript
{
  query: string;                  // REQUIRED: Search terms (≥2 chars)
  allowed_domains?: string[];     // Optional: Whitelist domains
  blocked_domains?: string[];     // Optional: Exclude domains
}
```

### Output Format

**Minimal metadata**:
```typescript
WebSearch({ query: "Claude API documentation" })

// Returns:
[
  {
    title: "Claude API Reference - Anthropic",
    url: "https://docs.anthropic.com/claude/reference"
  },
  {
    title: "Getting Started with Claude API",
    url: "https://docs.anthropic.com/claude/docs/getting-started"
  }
  // ... more results
]
```

**No content included**:
- API returns additional fields (page_age, encrypted_content)
- Claude Code discards these to keep results lightweight
- Must use WebFetch to get actual content

### Domain Filtering

**Whitelist specific domains**:
```typescript
WebSearch({
  query: "authentication best practices",
  allowed_domains: ["owasp.org", "auth0.com", "okta.com"]
})
// Returns only results from these domains
```

**Blacklist domains**:
```typescript
WebSearch({
  query: "TypeScript tutorial",
  blocked_domains: ["w3schools.com", "tutorialspoint.com"]
})
// Excludes these domains from results
```

**Combined filtering**:
```typescript
WebSearch({
  query: "API security",
  allowed_domains: ["owasp.org", "cwe.mitre.org"],
  blocked_domains: ["spam-site.com"]
})
```

### Platform Limitations

**US Only**:
- Only available via Anthropic's direct API
- NOT available on AWS Bedrock
- NOT available on Google Cloud Vertex AI

**Implication**: International users may not have access

### Best Practices for Token Efficiency

**DO**:
- ✅ Use specific search queries
- ✅ Filter by domain when you know source
- ✅ Combine with WebFetch for content
- ✅ Search once, fetch multiple results

**DON'T**:
- ❌ Use overly broad queries
- ❌ Expect content in results (only titles/URLs)
- ❌ Search when you already have the URL
- ❌ Make multiple similar searches

### Token Cost Analysis

**Approximate costs**:

```typescript
// Single WebSearch call
WebSearch({ query: "Claude API docs" })
// Input: ~40 tokens
// Output (10 results): ~200 tokens
// Total: ~240 tokens (~$0.0007)

// Very cheap compared to WebFetch
```

**Efficient workflow**:
```typescript
// 1. Search for sources
WebSearch({ query: "Firebase authentication tutorial" })
// ~240 tokens

// 2. Fetch top 3 results
WebFetch({ url: result1, prompt: "How to set up Firebase auth?" })
WebFetch({ url: result2, prompt: "How to set up Firebase auth?" })
WebFetch({ url: result3, prompt: "How to set up Firebase auth?" })
// ~1200 tokens (3 × 400)

// Total: ~1440 tokens

// vs. Manual web browsing and copy-paste: Much more
```

## WebSearch + WebFetch Pattern

### Recommended Workflow

**1. Discover sources**:
```typescript
const results = WebSearch({
  query: "TypeScript generics best practices",
  allowed_domains: ["typescriptlang.org", "stackoverflow.com"]
})
```

**2. Fetch content from top results**:
```typescript
// Parallel fetch for efficiency
Promise.all([
  WebFetch({
    url: results[0].url,
    prompt: "What are the best practices for TypeScript generics?"
  }),
  WebFetch({
    url: results[1].url,
    prompt: "What are the best practices for TypeScript generics?"
  }),
  WebFetch({
    url: results[2].url,
    prompt: "What are the best practices for TypeScript generics?"
  })
])
```

**3. Synthesize information**:
```typescript
// Claude combines answers from all sources
// Provides comprehensive guidance
```

### When to Use Web Tools

**Use WebSearch + WebFetch when**:
- ✅ Need current information (after training cutoff)
- ✅ Looking for specific documentation
- ✅ Researching best practices
- ✅ Finding examples or tutorials
- ✅ Checking latest API versions

**Don't use when**:
- ❌ Information likely in training data
- ❌ Looking for general programming concepts
- ❌ Question answerable from codebase
- ❌ Need guaranteed accuracy (verify sources)

## Web Tools vs MCP Servers

### When to Use Web Tools

**Built-in WebFetch/WebSearch**:
- Public web content
- One-off lookups
- General research
- No authentication needed

### When to Use MCP

**Custom MCP servers for**:
- Authenticated APIs (require API keys)
- Specialized data sources
- Frequent/repeated access
- Real-time data streams
- Complex query logic

**Example MCP use cases**:
- Jira API for tickets
- GitHub API for issues/PRs
- Internal documentation systems
- Custom databases

## Summary: Web Tools Optimization

### Token Minimization

1. **Specific prompts**: Ask exactly what you need from WebFetch
2. **Cache awareness**: Re-use URLs within 15 minutes
3. **Domain filtering**: Limit WebSearch results
4. **Parallel fetching**: Fetch multiple URLs simultaneously
5. **Search then fetch**: Don't fetch blindly

### Reliability

1. **Verify URLs**: Check results before fetching
2. **Handle redirects**: Follow cross-host redirect instructions
3. **Expect summarization**: WebFetch returns answers, not full content
4. **Check availability**: WebSearch US-only

### Cost Efficiency

1. **WebSearch is cheaper**: Use to discover, then fetch
2. **Haiku pre-processing**: Keeps main model costs low
3. **Batch questions**: Ask comprehensive questions to reduce fetches
4. **Prefer built-in over MCP**: Unless you need authentication

---

**Next**: [Task Management Tools](./05-TASK-MANAGEMENT.md)
