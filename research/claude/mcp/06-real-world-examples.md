# Real-World MCP Server Examples

## Production MCP Servers & Use Cases

This document catalogs real-world MCP server implementations to learn from.

## 🏢 Official Production Servers

### 1. GitHub MCP Server
- **Repository**: https://github.com/github/github-mcp-server
- **Maintainer**: GitHub (Official)
- **Purpose**: Connect AI to GitHub platform

**Capabilities:**
- Repository management (clone, search, browse code)
- Issue and PR operations
- GitHub Projects integration
- Code analysis
- File operations
- Organization management

**Key Features:**
- Server instructions for guiding model behavior
- Multi-tool workflows for complex operations
- OAuth authentication
- Rate limiting and error handling

**Tools Provided:**
- `create_or_update_file` - Create/update repository files
- `search_repositories` - Search GitHub repos
- `create_issue` - Create issues
- `create_pull_request` - Create PRs
- `push_files` - Push multiple files
- `search_code` - Search code across repos

**Learnings:**
- ✅ Clear tool naming conventions
- ✅ Comprehensive error messages
- ✅ Server instructions guide usage
- ✅ Interdependent tool workflows
- ✅ OAuth for enterprise security

---

### 2. PostgreSQL MCP Server
- **Repository**: https://github.com/modelcontextprotocol/servers (postgres)
- **Maintainer**: MCP Community
- **Purpose**: Database operations for PostgreSQL

**Capabilities:**
- Query execution
- Schema inspection
- Table management
- Data analysis

**Tools:**
- `query` - Execute SQL queries
- `list_tables` - List all tables in database
- `describe_table` - Get table schema
- `get_table_data` - Retrieve table data with pagination

**Resources:**
- `schema://{database}` - Database schema
- `table://{table_name}` - Table structure

**Learnings:**
- ✅ Connection pooling for performance
- ✅ SQL injection prevention
- ✅ Pagination for large results
- ✅ Schema as resources, not tools

---

### 3. Filesystem MCP Server
- **Repository**: https://github.com/modelcontextprotocol/servers (filesystem)
- **Maintainer**: Anthropic (Official Reference)
- **Purpose**: Secure file operations

**Security Features:**
- Configurable access controls
- Path validation
- Sandboxing
- Permission checks

**Tools:**
- `read_file` - Read file contents
- `write_file` - Write to file
- `list_directory` - List directory contents
- `create_directory` - Create new directory
- `move_file` - Move/rename files
- `search_files` - Search by pattern

**Learnings:**
- ✅ Security-first design (path validation)
- ✅ Configurable allowed directories
- ✅ Comprehensive error handling
- ✅ Cross-platform path handling

---

## 🚀 Enterprise Production Servers

### 4. Slack MCP Server
- **Provider**: Official Slack integration
- **Purpose**: Team communication integration

**Tools:**
- `send_message` - Send messages to channels
- `list_channels` - Get available channels
- `search_messages` - Search message history
- `get_channel_info` - Channel metadata

**Resources:**
- `channel://{channel_id}/messages` - Channel history
- `user://{user_id}/profile` - User profiles

**Use Cases:**
- AI agents posting status updates
- Automated notifications
- Team collaboration workflows

---

### 5. Sentry MCP Server
- **Repository**: Official Sentry MCP integration
- **Purpose**: Error monitoring and debugging

**Tools:**
- `get_issues` - Retrieve error issues
- `get_issue_details` - Get detailed issue info
- `resolve_issue` - Mark issues as resolved
- `get_stack_trace` - Retrieve stack traces

**Use Cases:**
- AI-assisted debugging
- Automated error triage
- Root cause analysis

---

### 6. CentralMind Gateway
- **Repository**: https://github.com/CentralMind/Gateway
- **Purpose**: Production-ready API generation from database schema

**Key Features:**
- Automatic API from DB schema
- Supports PostgreSQL, MySQL, ClickHouse, Snowflake, BigQuery, Supabase
- Production-grade scalability
- RESTful API generation

**Learnings:**
- ✅ Schema-driven development
- ✅ Multi-database support
- ✅ Auto-generation reduces boilerplate

---

## 💡 Community Innovations

### 7. Last9 MCP Server
- **Provider**: Last9
- **Purpose**: Production observability (logs, metrics, traces)

**Innovation:**
- Brings real-time production context into local environment
- Helps auto-fix code faster
- Integrates observability data with AI coding

**Use Cases:**
- Debug production issues locally
- AI-assisted incident response
- Performance optimization

**Learnings:**
- ✅ Real-time data integration
- ✅ Production/development bridge
- ✅ Context-aware debugging

---

### 8. QuantConnect MCP Server
- **Purpose**: Algorithmic trading platform integration

**Tools:**
- `update_project` - Update trading strategies
- `write_strategy` - Write new strategies
- `run_backtest` - Test strategies on historical data
- `deploy_live` - Deploy to live trading

**Use Cases:**
- AI-assisted strategy development
- Automated backtesting
- Risk analysis

**Learnings:**
- ✅ High-stakes environments need extra validation
- ✅ Sandbox testing before production
- ✅ Financial data security

---

### 9. Needle (RAG Platform)
- **Provider**: Needle
- **Purpose**: Production-ready RAG for document search

**Features:**
- Out-of-the-box RAG
- Search and retrieve from documents
- Semantic search
- Vector embeddings

**Use Cases:**
- Knowledge base integration
- Document Q&A
- Enterprise search

**Learnings:**
- ✅ RAG as a service via MCP
- ✅ Semantic search integration
- ✅ Document pipeline automation

---

### 10. ApeRAG (ApeCloud)
- **Repository**: https://github.com/apecloud/ApeRAG
- **Purpose**: Production RAG combining multiple search methods

**Architecture:**
- Graph RAG + Vector search + Full-text search
- Hybrid search for better results
- Production-ready scaling

**Learnings:**
- ✅ Multi-modal search strategies
- ✅ Graph + vector combination
- ✅ Enterprise-grade RAG

---

## 🎮 Specialized Use Cases

### 11. Fantasy Premier League MCP
- **Repository**: https://github.com/rishijatia/fantasy-pl-mcp
- **Purpose**: Real-time Fantasy Premier League data

**Tools:**
- `get_player_stats` - Player performance data
- `get_fixtures` - Upcoming matches
- `get_team_info` - Team information
- `analyze_transfers` - Transfer recommendations

**Learnings:**
- ✅ Real-time sports data integration
- ✅ API wrapper as MCP server
- ✅ Domain-specific tools

---

### 12. Lunar Gateway (TheLunarCompany)
- **Repository**: https://github.com/TheLunarCompany/lunar#mcpx
- **Purpose**: Production MCP gateway at scale

**Enterprise Features:**
- Centralized tool discovery
- Access controls
- Call prioritization
- Usage tracking
- Multi-tenant support

**Learnings:**
- ✅ Gateway pattern for scale
- ✅ Centralized management
- ✅ Enterprise access control
- ✅ Usage analytics

---

## 📊 Data & Analytics Servers

### 13. Snowflake MCP Server
- **Purpose**: Cloud data warehouse integration

**Tools:**
- `execute_query` - Run SQL on Snowflake
- `list_databases` - List available databases
- `get_table_schema` - Retrieve table schemas
- `analyze_data` - Run analysis queries

**Use Cases:**
- AI-driven data analysis
- Automated reporting
- Business intelligence

---

### 14. BigQuery MCP Server
- **Purpose**: Google BigQuery integration

**Similar capabilities to Snowflake**

**Learnings:**
- ✅ Cloud warehouse patterns
- ✅ Cost-aware query execution
- ✅ Large dataset handling

---

## 🔧 Developer Tools

### 15. Memory MCP Server (Official Reference)
- **Repository**: https://github.com/modelcontextprotocol/servers (memory)
- **Purpose**: Knowledge graph-based persistent memory

**Architecture:**
- Graph database for relationships
- Persistent across sessions
- Semantic connections

**Tools:**
- `store_memory` - Store information
- `recall_memory` - Retrieve related info
- `connect_concepts` - Link related ideas
- `search_knowledge` - Semantic search

**Use Cases:**
- Long-term AI memory
- Cross-session learning
- Relationship mapping

**Learnings:**
- ✅ Graph for relationships
- ✅ Persistent state management
- ✅ Semantic connections

---

### 16. Sequential Thinking MCP (Official Reference)
- **Purpose**: Dynamic, reflective problem-solving

**Approach:**
- Thought sequences
- Iterative refinement
- Meta-reasoning

**Learnings:**
- ✅ AI reasoning transparency
- ✅ Iterative problem solving
- ✅ Step-by-step thinking

---

## 🌐 API Integration Patterns

### 17. Stripe MCP Server
- **Purpose**: Payment processing

**Security Focus:**
- API key management
- PCI compliance considerations
- Webhook handling

**Tools:**
- `create_customer` - Create customer records
- `create_payment` - Process payments
- `get_transactions` - Retrieve transaction history
- `refund_payment` - Issue refunds

---

### 18. AWS Services MCP
- **Purpose**: AWS service integration

**Services Covered:**
- S3 (storage)
- Lambda (functions)
- DynamoDB (database)
- CloudWatch (monitoring)

**Learnings:**
- ✅ Multi-service integration
- ✅ IAM role management
- ✅ Cross-service workflows

---

## 🎓 Key Patterns from Real-World Servers

### Pattern 1: Gateway/Proxy Architecture
**Example**: Lunar Gateway

```
Client → Gateway → Multiple MCP Servers
         ↓
    - Auth
    - Rate Limiting
    - Routing
    - Logging
```

**Benefits:**
- Centralized access control
- Usage tracking
- Server discovery
- Load balancing

---

### Pattern 2: Hybrid Search (RAG)
**Example**: ApeRAG

```
Query → [Vector Search + Graph RAG + Full-text] → Merged Results
```

**Benefits:**
- Better result quality
- Multi-modal retrieval
- Fallback strategies

---

### Pattern 3: Production Observability Bridge
**Example**: Last9

```
Production Logs/Metrics/Traces → MCP Server → Local Dev Environment
```

**Benefits:**
- Debug with real data
- Faster issue resolution
- Context-aware development

---

### Pattern 4: Schema-Driven API Generation
**Example**: CentralMind Gateway

```
Database Schema → Auto-generate API → MCP Tools
```

**Benefits:**
- Reduced boilerplate
- Schema changes auto-reflected
- Consistent API design

---

## 📈 Awesome Lists for More Examples

### Comprehensive Collections

1. **punkpeye/awesome-mcp-servers**
   - https://github.com/punkpeye/awesome-mcp-servers
   - 150+ servers cataloged

2. **wong2/awesome-mcp-servers**
   - https://github.com/wong2/awesome-mcp-servers
   - Curated quality list

3. **appcypher/awesome-mcp-servers**
   - https://github.com/appcypher/awesome-mcp-servers
   - Enterprise focus

4. **TensorBlock/awesome-mcp-servers**
   - https://github.com/TensorBlock/awesome-mcp-servers
   - Comprehensive collection

### Categories Covered
- **Databases**: PostgreSQL, MySQL, MongoDB, Redis, Supabase
- **Cloud**: AWS, Azure, Google Cloud, Alibaba Cloud
- **SaaS**: Slack, Jira, Salesforce, HubSpot
- **Dev Tools**: GitHub, GitLab, Sentry, Linear
- **Data**: Snowflake, BigQuery, Airtable
- **AI/ML**: OpenAI, Anthropic, Cohere
- **Analytics**: Mixpanel, Segment, PostHog
- **CMS**: WordPress, Strapi, Contentful

---

## 🎯 Recommendations for Optimized AI

### Servers to Study

1. **GitHub MCP** - Complex workflows, server instructions
2. **Filesystem MCP** - Security patterns, path validation
3. **Memory MCP** - State management, persistence
4. **PostgreSQL MCP** - Database patterns, connection pooling
5. **Lunar Gateway** - Enterprise scale, access control

### Patterns to Adopt

1. ✅ **Server instructions** - Guide model usage (from GitHub MCP)
2. ✅ **Security-first** - Path validation, sandboxing (from Filesystem)
3. ✅ **Connection pooling** - Performance (from PostgreSQL)
4. ✅ **Gateway pattern** - Scale (from Lunar)
5. ✅ **Observability** - Logs, metrics (from Last9)

### Anti-Patterns to Avoid

1. ❌ **Monolithic servers** - Keep single responsibility
2. ❌ **No auth** - Always consider security
3. ❌ **Synchronous long operations** - Use async + progress
4. ❌ **No error handling** - Comprehensive error responses
5. ❌ **No pagination** - Always paginate large results

---

## 📊 Server Categories Statistics

From awesome-mcp-servers lists (approximate):

- **Databases & Storage**: 30+ servers
- **API Integrations**: 40+ servers
- **Developer Tools**: 25+ servers
- **Cloud Services**: 20+ servers
- **Data & Analytics**: 15+ servers
- **Communication**: 10+ servers
- **AI/ML Services**: 10+ servers
- **Specialized/Domain**: 20+ servers

**Total**: 150+ production MCP servers

---

## 🔗 Quick Links

- Official Servers: https://github.com/modelcontextprotocol/servers
- GitHub MCP: https://github.com/github/github-mcp-server
- Awesome Lists: Search "awesome-mcp-servers" on GitHub
- MCP Directory: https://www.mcplist.ai/
