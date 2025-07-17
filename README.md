# open-cicd

An alternate open source CI/CD tool to Jenkins with modern architecture and better UX.

## Key Features

- **Better UI/UX**: Modern web interface built with SvelteKit
- **GitOps Native**: Jobs defined as YAML files, version-controlled
- **Self-Hosted**: Control plane runs anywhere, connects to any SCM (GitHub, GitLab, etc.)
- **HTTP-First**: Pure HTTP/JSON API for all communication - no gRPC complexity
- **High Performance**: Built with Zig and Zap HTTP server for blazing speed

## Architecture

### Control Plane and Data Plane Separation

**Control Plane** (HTTP Server):
- Zap-based HTTP server written in Zig for blazing performance
- PostgreSQL for metadata storage with connection pooling
- Redis for caching and real-time updates
- S3-compatible storage for artifacts
- SvelteKit web interface with Server-Sent Events (SSE)

**Data Plane** (Agents):
- Docker containers with HTTP endpoints
- Ephemeral agents (one job at a time)
- Self-registering via `POST /api/v1/agents/register`
- Health monitoring via `POST /api/v1/agents/{id}/heartbeat`
- Job polling via `GET /api/v1/agents/{id}/jobs`
- Build caching through shared volumes

### Communication Flow

```
┌─────────────────┐   HTTP/JSON     ┌─────────────────┐
│   Control       │ ────────────────▶│     Agent       │
│   Plane         │   Job Polling   │   (Docker)      │
│   (Zap Server)  │◀──────────────── │                 │
└─────────────────┘   Status/Logs   └─────────────────┘
        │               Updates              │
        │                                   │
        ▼                                   ▼
┌─────────────────┐                ┌─────────────────┐
│   PostgreSQL    │                │   Job Execution │
│   Redis         │                │   Log Streaming │
│   S3 Storage    │                │   Artifact Gen  │
└─────────────────┘                └─────────────────┘
```

### HTTP API Endpoints

**Control Plane**:
- `POST /api/v1/agents/register` - Agent registration
- `GET /api/v1/agents` - List active agents
- `GET /api/v1/agents/{id}/jobs` - Get job assignments for agent
- `POST /api/v1/jobs` - Create new job
- `GET /api/v1/jobs/{id}` - Job status and logs
- `GET /api/v1/jobs/{id}/logs/stream` - Stream job logs (SSE)
- `PUT /api/v1/jobs/{id}/status` - Status updates from agents
- `POST /api/v1/agents/{id}/heartbeat` - Agent heartbeat

**Agent Client Actions**:
- Poll for jobs via `GET /api/v1/agents/{id}/jobs`
- Send status updates via `PUT /api/v1/jobs/{id}/status`
- Send heartbeat via `POST /api/v1/agents/{id}/heartbeat`
- Stream logs via `POST /api/v1/jobs/{id}/logs`

## Technology Stack

- **Backend**: Zig with Zap HTTP server (microsecond response times)
- **Database**: PostgreSQL + Redis (async connections)
- **Storage**: MinIO/S3 compatible
- **Frontend**: SvelteKit + TailwindCSS + Server-Sent Events
- **Agents**: Docker containers with HTTP clients
- **Communication**: HTTP/JSON only (no gRPC)
- **Real-time**: Server-Sent Events (SSE) for live updates
- **Architecture**: Domain-driven design with clear boundaries

## Development Phases

### Phase 1: Core HTTP Infrastructure (4-6 weeks)
- [x] Project setup and architecture planning
- [x] Architecture documentation updated for Zig + HTTP-only
- [ ] Zap HTTP server with domain-driven structure
- [ ] Agent registration and heartbeat system
- [ ] PostgreSQL + Redis integration
- [ ] Basic job API with state machines

### Phase 2: Job Execution (4-6 weeks)
- [ ] YAML pipeline parser
- [ ] Job distribution algorithm
- [ ] Real-time status updates via HTTP
- [ ] Log streaming via Server-Sent Events
- [ ] SvelteKit web interface with SSE integration

### Phase 3: Production Features (6-8 weeks)
- [ ] GitHub/GitLab webhook integration
- [ ] Artifact storage system
- [ ] JWT authentication and authorization
- [ ] Build caching
- [ ] Agent auto-scaling
- [ ] Monitoring and metrics

## Benefits

- **Simple Integration**: Standard HTTP/JSON API - no gRPC complexity
- **Language Agnostic**: Agents can be written in any language with HTTP support
- **Easy Debugging**: Standard HTTP tools (curl, Postman) work out of the box
- **High Performance**: Zap provides microsecond HTTP response times
- **Extensible**: Users can build custom agents and integrations
- **Domain-Driven**: Clear separation of concerns with domain boundaries
- **Memory Safety**: Zig's compile-time safety prevents common bugs
- **Real-time**: Server-Sent Events provide live updates without WebSocket complexity
