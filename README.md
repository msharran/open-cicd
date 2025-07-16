# open-cicd

An alternate open source CI/CD tool to Jenkins with modern architecture and better UX.

## Key Features

- **Better UI/UX**: Modern web interface built with SvelteKit
- **GitOps Native**: Jobs defined as YAML files, version-controlled
- **Self-Hosted**: Control plane runs anywhere, connects to any SCM (GitHub, GitLab, etc.)
- **HTTP-First**: Simple REST API for easy integration and extensibility
- **High Performance**: Built with Zig and Zap HTTP server for blazing speed

## Architecture

### Control Plane and Data Plane Separation

**Control Plane** (HTTP Server):
- Zap-based HTTP server written in Zig
- PostgreSQL for metadata storage
- Redis for caching and real-time updates
- S3-compatible storage for artifacts
- SvelteKit web interface for modern UI

**Data Plane** (Agents):
- Docker containers with HTTP endpoints
- Ephemeral agents (one job at a time)
- Self-registering via `/register` endpoint
- Health monitoring via `/health` endpoint
- Build caching through shared volumes

### Communication Flow

```
┌─────────────────┐    HTTP/JSON    ┌─────────────────┐
│   Control       │ ────────────────▶│     Agent       │
│   Plane         │                 │   (Docker)      │
│   (Zap Server)  │◀──────────────── │                 │
└─────────────────┘    Status       └─────────────────┘
        │               Updates              │
        │                                   │
        ▼                                   ▼
┌─────────────────┐                ┌─────────────────┐
│   PostgreSQL    │                │   Job Execution │
│   Redis         │                │   Log Streaming │
│   S3 Storage    │                │   Artifact Gen  │
└─────────────────┘                └─────────────────┘
```

### API Endpoints

**Control Plane**:
- `POST /register` - Agent registration
- `GET /agents` - List active agents
- `POST /jobs` - Create new job
- `GET /jobs/{id}` - Job status and logs
- `POST /jobs/{id}/status` - Status updates from agents

**Agent**:
- `GET /health` - Health check endpoint
- `POST /jobs` - Receive job assignment
- `GET /jobs/{id}` - Job status query
- `DELETE /jobs/{id}` - Cancel running job

## Technology Stack

- **Backend**: Zig with Zap HTTP server
- **Database**: PostgreSQL + Redis
- **Storage**: MinIO/S3 compatible
- **Frontend**: SvelteKit + TailwindCSS
- **Agents**: Docker containers with HTTP servers
- **Serialization**: JSON for HTTP API

## Development Phases

### Phase 1: Core HTTP Infrastructure (4-6 weeks)
- [x] Project setup and architecture planning
- [ ] Zap HTTP server with basic endpoints
- [ ] Agent registration and health monitoring
- [ ] PostgreSQL integration
- [ ] Basic job API

### Phase 2: Job Execution (4-6 weeks)
- [ ] YAML pipeline parser
- [ ] Job distribution algorithm
- [ ] Real-time status updates
- [ ] Log streaming and collection
- [ ] SvelteKit web interface with real-time features

### Phase 3: Production Features (6-8 weeks)
- [ ] GitHub/GitLab integration
- [ ] Artifact storage system
- [ ] Authentication and authorization
- [ ] Build caching
- [ ] Agent auto-scaling

## Benefits

- **Simple Integration**: Standard HTTP/JSON API
- **Language Agnostic**: Agents can be written in any language
- **Easy Debugging**: Standard HTTP tools work out of the box
- **High Performance**: Zap provides excellent HTTP performance
- **Extensible**: Users can build custom agents and integrations
    
