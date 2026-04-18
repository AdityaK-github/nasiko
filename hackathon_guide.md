Background

Nasiko currently:

Validates uploaded agents and metadata
Generates AgentCard.json if absent
Injects Arize Phoenix + OpenTelemetry observability at deploy time
Builds/deploys container images
Registers agents with Kong
Assumes LangChain / CrewAI as baseline frameworks
Unchanged Agent Structure Contract
my-agent/
├── docker-compose.yml
├── Dockerfile
└── src/
    └── main.py

This exact top-level structure is mandatory for all supported frameworks.

Track 1: MCP Server Publishing and Agent Integration
Goal

Add first-class support for publishing MCP servers as deployable artifacts with:

Validation
Manifest generation
Observability
Capability discovery
End-to-end deploy + callability parity
Build Requirements
Automatic artifact-type detection:
Agent vs MCP server
Transport detection
Loud validation failures on ambiguity
Stdio transport
Primary target
Requires stdio → HTTP bridge at deploy time
MCP manifest generation (if absent):
Tools
Resources
Prompts
Extracted from FastMCP decorators or equivalent
Capability discovery:
Agents can consume MCP tools
No code changes required
Stretch Goal:
Remote MCP registration via URL (HTTP/SSE)
Must Impact
Manifest generation
nasiko/app/utils/agentcard_generator/
Add MCP manifest generator
Observability injection
injector.py
instrumentation_injector.py
Orchestrator wiring
agent_builder.py
redis_stream_listener.py
Include stdio → HTTP bridge
Kong routing for MCP artifacts
Upload validation (auto-detection)
Agent ↔ MCP association (no code changes)
Developer documentation
Must NOT Impact
Existing agent upload flows (LangChain, CrewAI, etc.)
AgentCard output (must remain byte-identical)
Existing APIs:
/api/v1/agents/upload
/agents/upload-directory
Project structure contract
Kong routing for existing agents
Redis stream contract
Acceptance Criteria
Stdio MCP server:
Fully deployable end-to-end
Manifest generated
Observability injected
MCP server:
Appears in web + CLI
Has non-empty manifest
Agents:
Can use MCP tools without code changes
Observability:
Tool calls produce correlated spans in Phoenix
Backward compatibility:
Existing agents remain unchanged
Stretch:
Remote MCP server via URL works fully
Track 2: LLM Router Gateway Integration
Goal

Introduce a platform-managed LLM gateway using:

LiteLLM or
Portkey

So agents use:

Platform endpoint
Virtual key

Instead of hardcoded provider keys

Build Requirements
Gateway deploys within existing infra/orchestrator
Legacy agents must continue working
Provider credentials:
Centrally managed
Must document:
LiteLLM vs Portkey trade-offs
Warning against hardcoding keys
Must Impact
Infra deployment:
Orchestrator
CLI setup
docker-compose.local.yml
Agent runtime:
Inject gateway URL + virtual key
Observability:
Trace correlation between agent ↔ gateway
Documentation updates
At least one sample agent uses gateway
Must NOT Impact
Upload/build/deploy pipeline
Agent structure contract
Existing trace/metric format
Legacy agents using direct keys
Kong routing behavior
Acceptance Criteria
Gateway auto-deploys via:
make start-nasiko
Sample agent:
Works without provider key in code
Provider switching:
Done via gateway config only
Legacy agents:
Continue working unchanged
Observability:
Gateway calls linked to agent traces
Required Integration Tests
Track 1 Tests
Valid MCP server upload → success + deploy + callable
Missing src/main.py → validation error
Ambiguous artifact → validation error
Manifest:
Contains tools, resources, prompts
CLI vs Web behavior:
Must match
Stretch
Remote MCP URL:
Works fully
Invalid remote URL:
Fails clearly
Track 2 Tests
Platform boot:
Gateway reachable
Sample agent:
Works via gateway without provider key
Provider rotation:
No agent code changes required
Observability:
Gateway span linked to agent
How to Contribute
Fork the repository
Pick a track
Open a design issue first
Submit focused PR with:
Feature code
Integration tests
Documentation
Behavior notes
Keep:
CI green
Changes scoped
Fixed Design Decisions
Already Fixed
Track 1:
Detection via imports/manifests
Track 2:
Gateway runs in-cluster
Gateway failure:
Requests fail clearly
No fallback/queueing
You Must Design
Virtual key system
Minting
Storage
Rotation policy